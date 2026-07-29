# 博客分栏与字体改造 · 设计文档

日期：2026-07-29
状态：待实施

## 背景

站点目前只有一个内容目录 `content/posts/`，4 篇已发布文章混在同一条时间线上：

| 文章 | 日期 | 性质 |
| --- | --- | --- |
| FlowFocus更新，增加预置日历和云同步功能 | 2026-07-12 | 产品 / AI 编程（设计大爆炸） |
| 2026年XAI可解释人工智能学习资源 | 2026-07-23 | 研究（XAI小探） |
| Forecasting: Getting Started | 2024-11-14 | 研究 |
| 如何使用word模版准备提交版论文 | 2024-09-07 | 学术工具 |
| Hello World | 2024-05-11 | `draft: true` |

两个目标：

1. 把「做东西」类的内容和「研究」类的内容分成两个独立栏目，不再混排。
2. 全站换用 Maple Mono 字体。

两件事互不依赖：Part A 只动 `content/`、`hugo.yaml` 和一个 partial，Part B 只动 `assets/css/extended/` 和另一个 partial，没有共同文件。可以分两次实施、分两次验证、分两次上线；任一部分回滚不影响另一部分。

## 已确定的决策

| 决策 | 选择 | 理由 |
| --- | --- | --- |
| 栏目命名 | `making` / `research` | 不用公众号名（设计大爆炸 / XAI小探）——那是分发渠道，会变；也不用「设计」——太窄，装不下 AI 编程和工具。`造物 / making` 取自作者自己在 FlowFocus 一文中用过的「AI 造物指南」。 |
| 实现方式 | Hugo content sections | 见下方「方案取舍」。 |
| 首页内容 | 只显示 research | 站点主体形象是研究，making 是副线。 |
| 导航形态 | research + making 两个 tab 都保留 | 视觉对称；接受 `/` 与 `/research/` 列表重复。 |
| 导航语言 | 小写英文 | 与现有的 `search` / `archives` 一致。 |
| 字体范围 | 全站含中文 | 通过 ZeoSeven FontsAPI 加载 Maple Mono NF CN。 |
| 标题字重 | 400 | 该 CDN 端点只有 400 一个字重，标题用字号和颜色做层级，避免浏览器合成粗体。 |

## 方案取舍（分栏）

| | A · sections | B · categories | C · 混合 |
| --- | --- | --- | --- |
| 首页只显示 research | 1 行配置 | 每篇加标记 | 1 行配置 |
| 栏目 URL | `/making/` | `/categories/making/` | `/making/` |
| 现有 URL | 2 个失效，需 alias | 不变 | 不变 |
| 栏目页样式 | 完整列表页 | tag-entry 窄样式 | 完整列表页 |
| 分栏 RSS | 免费获得 | 无 | 免费获得 |

选 **A**。「首页只显示 research」正是 Hugo `mainSections` 的语义，A 用一行配置拿到，B 需要绕开主题逻辑。代价是两个已上线 URL 变动，用 `aliases` 解决。

## 主题现状（已核对源码）

实施前逐条确认过，不是推测：

| 位置 | 事实 | 影响 |
| --- | --- | --- |
| `hugo.yaml` | **未定义** `mainSections` | Hugo 当前自动选取页面最多的 section。一旦出现第二个 section，这个自动选择会变得不可预测，文章会静默从首页消失。**必须显式配置。** |
| `layouts/_default/list.html:44` | 首页按 `mainSections` 过滤 | 配 `mainSections: ["research"]` 即实现「首页只显示 research」。 |
| `layouts/_default/archives.html:30,32` | 按 `mainSections` 过滤，但支持 `ShowAllPagesInArchive` | 需开启该参数，否则归档页会漏掉 making。 |
| `layouts/partials/post_nav_links.html:1` | **也**按 `mainSections` 过滤 | 不处理的话，making 文章底部的上一篇/下一篇会整块消失。需覆盖。 |
| `layouts/_default/index.json:2` | 用 `site.RegularPages` | 搜索天然覆盖全站，无需改动。 |
| `layouts/_default/rss.xml:33-37` | 忽略 `mainSections` | 主 feed `/index.xml` 仍包含两个栏目；另外免费获得分栏 feed。 |
| `layouts/partials/head.html:64-68` | 拼接顺序 `license + core + extended` | `assets/css/extended/*.css` 在主题 CSS 之后加载，是干净的覆盖点。**但 `@import` 会因为不在文件开头而被浏览器丢弃。** |
| `layouts/partials/extend_head.html` | 空占位，由 `head.html:149` 调用 | 站点级同名文件即可覆盖，用来插入字体 `<link>`。 |
| `assets/css/core/reset.css:26` | `body { font-family: ... }` | 字体覆盖的目标选择器。 |

主题是 submodule（`themes/PaperMod` → `kxqdesign/hugo-PaperMod-kxq`），所有改动走站点级 `layouts/` 与 `assets/`，不碰 submodule。

## Part A · 分栏改造

### A1 目录结构

```
content/
├── research/
│   ├── _index.md                          新建
│   ├── forecasting.md                     ← posts/Forecasting.md
│   ├── xai-learning-resources/            ← 整个 bundle 移动
│   └── 如何使用word模版准备提交版论文/       ← 整个 bundle 移动
├── making/
│   ├── _index.md                          新建
│   ├── flowfocus-ai-rebuild/              ← 整个 bundle 移动
│   └── hello-world/                       ← 草稿，移动并修好封面路径
├── archives.md                            不变
└── search.md                              不变
```

`content/posts/` 整个删除。图片随 bundle 整体移动，文章内的图片引用是相对路径，无需修改。

用 `git mv` 移动，保留文件历史。

### A2 URL 与重定向

| 文章 | 旧 URL | 新 URL | 处理 |
| --- | --- | --- | --- |
| Forecasting | `/posts/forecasting/` | `/research/forecasting/` | 加 alias（已上线） |
| word模版 | `/posts/如何使用word模版准备提交版论文/` | `/research/如何使用word模版准备提交版论文/` | 加 alias（已上线） |
| XAI 资源 | — | `/research/xai-learning-resources/` | 无需（尚未 push） |
| FlowFocus | — | `/making/flowfocus-ai-rebuild/` | 无需（尚未 push） |

前两篇的 front matter 加：

```yaml
aliases:
  - /posts/forecasting/
```

Hugo 会生成跳转页，旧链接和搜索引擎收录继续可用。

### A3 `hello-world` 草稿

现状是 `content/posts/hello-world/hello-world.md` 加同目录一张图，**不是** page bundle（缺 `index.md`），封面写的是 `posts/hello-world/testfigure.webp` 且 `relative: false`——移动后必然失效。

改为规范 bundle：`content/making/hello-world/index.md`，封面改为 `image: "testfigure.webp"` + `relative: true`。它是草稿（`buildDrafts: false`），现在不影响构建，但此刻顺手修好，免得将来发布时踩坑。

### A4 `hugo.yaml`

```yaml
params:
  mainSections: ["research"]     # 新增
  ShowAllPagesInArchive: true    # 新增

menu:
  main:
    - { identifier: research, name: research, url: /research/, weight: 1 }
    - { identifier: making,   name: making,   url: /making/,   weight: 2 }
    - { identifier: archives, name: archives, url: archives/,  weight: 3 }
    - { identifier: search,   name: search,   url: search,     weight: 4 }
```

内容栏目排在工具栏目之前（现状是 `search` weight 1）。

### A5 栏目落地页

```yaml
# content/research/_index.md
---
title: 'research'
description: '可解释性、预测，以及做研究本身的方法与工具。'
---
```

```yaml
# content/making/_index.md
---
title: 'making'
description: '用 AI 把想法做成能用的东西：产品、工具，以及过程中的取舍。'
---
```

`list.html:26` 会把 description 渲染在标题下方。

### A6 模板覆盖：上一篇/下一篇

新建 `layouts/partials/post_nav_links.html`，内容复制主题原文件，仅改第一行：

```go-html-template
{{- $pages := .CurrentSection.RegularPages.ByDate.Reverse }}
{{- if and (gt (len $pages) 1) (in $pages . ) }}
```

原本按 `mainSections` 过滤，配了 `mainSections: ["research"]` 之后 making 文章不在集合里，`(in $pages .)` 为假，整个导航块不渲染。改为按当前 section 取，上一篇/下一篇在各自栏目内跳转——这比原行为更合理。

## Part B · 字体改造

### B1 为什么是 ZeoSeven

- Maple Mono **只有等宽**，没有比例字体变体。
- 官方 WOFF2 构建**只含拉丁字符**（`@fontsource/maple-mono@5.3.0` 只有 `maple-mono-latin-400-normal.woff2`）。
- 中文变体没有 web 构建：`MapleMono-CN.zip` 134 MB、`MapleMono-NF-CN.zip` 152 MB，只有 TTF/OTF。
- 直接用官方拉丁版会得到「拉丁 Maple + 中文系统字体」的混排效果。

ZeoSeven（item 442）用 `cn-font-split` 把 CN 版切成 **191 个 unicode-range 分片**，浏览器只下载页面用到的分片。这是目前唯一现成的中文 web 方案。

许可证 OFL-1.1。

### B2 实测成本

解析全部 191 个 `unicode-range` 与文章正文字符取交集：

| 文章 | 不重复字符数 | 需要分片 | 字体下载量 |
| --- | --- | --- | --- |
| FlowFocus | 714 | 22 / 191 | 1151 KB |
| XAI 资源 | 647 | 20 / 191 | 1083 KB |

缓解因素（均已核实）：

- `font-display: swap`——正文立即以后备字体渲染，字体到达后替换，无空白期。
- `cache-control: public, immutable, max-age=31536000`——缓存一年；分片跨文章复用，第二篇几乎不产生新下载。
- Cloudflare 承载，`access-control-allow-origin: *`，实测单请求约 150 ms。不是仅限国内的 CDN。

作为参照：XAI 一文本身已有 2.9 MB 图片。

### B3 实施

```html
<!-- layouts/partials/extend_head.html  新建 -->
<link rel="preconnect" href="https://fontsapi.zeoseven.com" crossorigin>
<link rel="stylesheet" href="https://fontsapi.zeoseven.com/442/main/result.css">
```

```css
/* assets/css/extended/fonts.css  新建 */
body {
  font-family: "Maple Mono NF CN",
    -apple-system, BlinkMacSystemFont, 'Segoe UI', 'PingFang SC',
    'Microsoft YaHei', sans-serif;
}

/* 该端点只有 400 字重；标题靠字号与颜色区分，避免浏览器合成粗体 */
.post-title,
.post-content h1, .post-content h2, .post-content h3,
.post-content h4, .post-content h5, .post-content h6,
.entry-header h2,
.first-entry .entry-header h1,
.logo a {
  font-weight: 400;
}
```

选择器依据（已核对）：`.post-title`（`post-single.css:6`）和 `.post-content h1`–`h6`（`post-single.css:39-62`）只定义了字号与间距，**粗体来自浏览器 UA 默认样式**，因此必须显式覆盖。`.logo a` 在 `header.css:25` 显式写了 `font-weight: 700`；`.entry-header h2` 见 `post-entry.css:54`。extended CSS 在主题之后加载，同特异性下后者生效，无需 `!important`。

**必须用 `<link>`，不能用 `@import`。** Hugo 把 extended CSS 拼接在 core 之后，`@import` 会落在文件中部；CSS 规范要求 `@import` 位于所有规则之前，浏览器会静默丢弃它，且不报任何错误。

后备字体链保留系统字体：ZeoSeven 不可达时，站点退化为当前外观而非损坏。

## 验证方式

构建通过不算验证。逐项检查：

**分栏**

- [ ] `hugo --gc --minify` 无 ERROR
- [ ] 首页恰好列出 3 篇 research，FlowFocus 不出现
- [ ] `/making/` 列出 FlowFocus；`/research/` 列出 3 篇
- [ ] `/posts/forecasting/` 与 `/posts/如何使用word模版准备提交版论文/` 仍可跳转（alias 页面存在）
- [ ] 归档页显示全部 4 篇
- [ ] 搜索索引 `index.json` 包含全部 4 篇
- [ ] making 文章底部有上一篇/下一篇（用 `-D` 让草稿充当邻居来测）
- [ ] 现有 `/tags/...` 页面仍可访问
- [ ] `/index.xml` 含两个栏目；`/research/index.xml`、`/making/index.xml` 存在

**字体**

- [ ] 页面实际计算样式的 `font-family` 首选为 `Maple Mono NF CN`
- [ ] 中文与拉丁字符均由 Maple 渲染（非中文回落系统字体）
- [ ] 字体 CSS 与分片请求返回 200
- [ ] 标题计算字重为 400
- [ ] 断网/屏蔽 fontsapi 时页面仍正常渲染（后备链生效）

## 已知取舍

1. **`/` 与 `/research/` 列表重复**——Option 2 的代价，两者仅首页多一段简介。个人博客可接受。
2. **正文等宽中文**——长文可读性弱于比例字体。这是明确选择的风格，不是疏忽。
3. **`<strong>` 仍为合成粗体**——本次只把标题改为 400。两篇文章正文里 `<strong>` 用得不少，预览时需要实际看一眼；若观感不佳再单独处理。
4. **依赖第三方 CDN**——ZeoSeven 若长期不可用，需要改为自托管 subset（`cn-font-split` 自建）。后备字体链保证不会开天窗。

## 不在本次范围

- 不动 `public/`（仓库虽然跟踪它，但 CI 自行构建部署，本地副本已过期）
- 不改标签体系；tag 页跨栏目聚合是既有行为，保持
- 不引入 `.gitignore`（仓库现状无此文件，属独立议题）
- 不动 PaperMod submodule
