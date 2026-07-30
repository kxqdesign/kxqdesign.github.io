# kxqdesign.github.io

个人博客源码，Hugo + [PaperMod](https://github.com/kxqdesign/hugo-PaperMod-kxq)，部署在 <https://kxqdesign.com>。

推送到 `main` 会自动构建并发布。

## 写文章

见 **[docs/写作指南.md](docs/写作指南.md)** —— 文章放哪里、怎么写、本地怎么预览、怎么发布。

快速开始：

```bash
hugo new content research/my-new-post/index.md   # 或 making/
hugo server --renderToMemory                      # 预览：http://localhost:1313
```

## 目录

| 路径 | 说明 |
| --- | --- |
| `content/research/` | 研究类文章（首页显示） |
| `content/making/` | 做东西类文章 |
| `layouts/` | 覆盖主题的模板 |
| `assets/css/extended/` | 覆盖主题的样式 |
| `themes/PaperMod/` | 主题（submodule，不要直接改） |
| `docs/` | 写作指南与设计文档 |
