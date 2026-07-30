# Blog Section Split + Maple Mono Font — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Split `content/posts/` into two Hugo sections (`research/`, `making/`) with the home page showing only research, and switch the site to the Maple Mono NF CN webfont.

**Architecture:** Two independent phases. Phase A (Tasks 1–3) moves content into Hugo content sections and drives the home page off `mainSections`. Phase B (Tasks 4–5) loads a subsetted CJK webfont from ZeoSeven's CDN via a `<link>` in an overridden head partial, plus a CSS override in `assets/css/extended/`. The phases share no files and can ship or roll back independently.

**Tech Stack:** Hugo extended v0.123.0, PaperMod theme (git submodule — never edited), YAML config, plain CSS, Go HTML templates.

**Source spec:** `docs/superpowers/specs/2026-07-29-sections-and-font-design.md`

## Global Constraints

- **Never edit `themes/PaperMod/`.** It is a git submodule pointing at `kxqdesign/hugo-PaperMod-kxq`. All overrides go in site-level `layouts/` and `assets/`, which Hugo loads with higher priority than the theme.
- **Never build into the repo's `public/`.** Every build in this plan uses `--destination "$OUT"` pointing outside the repo. `public/` is tracked by git; building into it produces a large spurious diff. If it ever gets dirtied: `git checkout -- public && git clean -fd public resources`.
- **Hugo binary:** the CI pins extended **v0.123.0** (`.github/workflows/hugo.yaml:35`). Use the same locally. If `hugo` is not on PATH, use the scratchpad copy at `C:/Users/s398876/AppData/Local/Temp/claude/C--Users-s398876-Documents-kxqdesign/6b6741be-6c09-4f0e-9a16-e80d41e1ea27/scratchpad/hugo/hugo.exe`.
- **Standard build+verify preamble** — every verification step in this plan assumes these two shell variables:

```bash
HUGO=hugo                                              # or the scratchpad hugo.exe path above
OUT="C:/Users/s398876/AppData/Local/Temp/kxq-verify"   # forward slashes work in Git Bash, Hugo and Python
```

- **Move files with `git mv`,** never delete-and-recreate. These posts have history worth keeping.
- **Do not push.** The user pushes when they choose. Commit only.
- Repo currently sits 4 commits ahead of `origin/main` on branch `main`.

---

### Task 1: Move content into `research/` and `making/` sections

Creates the two sections, relocates all five posts, adds section landing pages, and adds redirects for the two URLs that are already live.

**Files:**
- Create: `content/research/_index.md`
- Create: `content/making/_index.md`
- Move: `content/posts/Forecasting.md` → `content/research/forecasting.md`
- Move: `content/posts/xai-learning-resources/` → `content/research/xai-learning-resources/`
- Move: `content/posts/如何使用word模版准备提交版论文/` → `content/research/如何使用word模版准备提交版论文/`
- Move: `content/posts/flowfocus-ai-rebuild/` → `content/making/flowfocus-ai-rebuild/`
- Move: `content/posts/hello-world/hello-world.md` → `content/making/hello-world/index.md`
- Move: `content/posts/hello-world/testfigure.webp` → `content/making/hello-world/testfigure.webp`
- Modify: `content/research/forecasting.md` (add `aliases`)
- Modify: `content/research/如何使用word模版准备提交版论文/index.md` (add `aliases`)
- Modify: `content/making/hello-world/index.md` (fix cover path)

**Interfaces:**
- Consumes: nothing (first task).
- Produces: two Hugo sections named exactly `research` and `making`. Task 2 references these names in `mainSections` and in menu URLs `/research/` and `/making/`. Task 3's override relies on every post living inside one of them.

- [ ] **Step 1: Verify the current state, so the change is provable**

```bash
rm -rf "$OUT" && $HUGO --gc --minify --destination "$OUT"
python - "$OUT" <<'PY'
import os, sys
out = sys.argv[1]
for p in ("posts/forecasting", "posts/xai-learning-resources",
          "research/forecasting", "making/flowfocus-ai-rebuild"):
    print(("EXISTS " if os.path.isfile(os.path.join(out, p, "index.html")) else "absent ") + p)
PY
```

Expected before the change:
```
EXISTS posts/forecasting
EXISTS posts/xai-learning-resources
absent research/forecasting
absent making/flowfocus-ai-rebuild
```

- [ ] **Step 2: Create the section directories and move every post**

```bash
cd C:/Users/s398876/Documents/kxqdesign/kxqdesign.github.io
mkdir -p content/research content/making/hello-world
git mv content/posts/Forecasting.md                      content/research/forecasting.md
git mv content/posts/xai-learning-resources              content/research/xai-learning-resources
git mv content/posts/如何使用word模版准备提交版论文        content/research/如何使用word模版准备提交版论文
git mv content/posts/flowfocus-ai-rebuild                content/making/flowfocus-ai-rebuild
git mv content/posts/hello-world/hello-world.md          content/making/hello-world/index.md
git mv content/posts/hello-world/testfigure.webp         content/making/hello-world/testfigure.webp
rmdir content/posts/hello-world content/posts
```

- [ ] **Step 3: Create `content/research/_index.md`**

```yaml
---
title: 'research'
description: '研究以及做研究的道术器'
---
```

- [ ] **Step 4: Create `content/making/_index.md`**

```yaml
---
title: 'making'
description: '把想法做成能用的东西：产品、工具，以及过程中的取舍。'
---
```

- [ ] **Step 5: Add the alias to `content/research/forecasting.md`**

Replace the whole front matter block (the first `---` … `---`) with:

```yaml
---
title: 'Forecasting: Getting Started'
date: 2024-11-14T21:50:51+08:00
aliases:
  - /posts/forecasting/
---
```

- [ ] **Step 6: Add the alias to `content/research/如何使用word模版准备提交版论文/index.md`**

Replace the whole front matter block with:

```yaml
---
title: '如何使用word模版准备提交版论文'
date: 2024-09-07T21:50:51+08:00
aliases:
  - /posts/如何使用word模版准备提交版论文/
---
```

- [ ] **Step 7: Fix the `hello-world` cover path**

It is now a page bundle (`index.md` beside its image), so the cover must be a relative page resource. Replace the front matter block of `content/making/hello-world/index.md` with:

```yaml
---
title: 'Hello World'
date: 2024-05-11T10:32:20+01:00
cover:
    image: "testfigure.webp"
    caption: ""
    alt: ""
    relative: true
draft: true
---
```

Leave everything below the front matter untouched.

- [ ] **Step 8: Rebuild and verify the moves and redirects**

```bash
rm -rf "$OUT" && $HUGO --gc --minify --destination "$OUT"
python - "$OUT" <<'PY'
import os, re, sys
out = sys.argv[1]
ok = True

def page(*parts):
    return os.path.join(out, *parts, "index.html")

for p in (("research", "forecasting"),
          ("research", "xai-learning-resources"),
          ("research", "如何使用word模版准备提交版论文"),
          ("making", "flowfocus-ai-rebuild"),
          ("research",), ("making",)):
    hit = os.path.isfile(page(*p))
    print(("PASS " if hit else "FAIL ") + "/".join(p))
    ok &= hit

# old URLs must now be redirect stubs, not real pages
for p in (("posts", "forecasting"), ("posts", "如何使用word模版准备提交版论文")):
    f = page(*p)
    if not os.path.isfile(f):
        print("FAIL alias missing: " + "/".join(p)); ok = False; continue
    html = open(f, encoding="utf-8").read()
    m = re.search(r'http-equiv=["\']?refresh', html, re.I)
    print(("PASS " if m else "FAIL ") + "alias redirects: " + "/".join(p))
    ok &= bool(m)

# section pages must show their descriptions
for sec, want in (("research", "道术器"), ("making", "把想法做成能用的东西")):
    html = open(page(sec), encoding="utf-8").read()
    hit = want in html
    print(("PASS " if hit else "FAIL ") + f"{sec} description rendered")
    ok &= hit

print("\nRESULT:", "OK" if ok else "PROBLEMS")
sys.exit(0 if ok else 1)
PY
```

Expected: every line `PASS`, final `RESULT: OK`.

- [ ] **Step 9: Confirm git recorded moves rather than delete+add**

```bash
git add -A
git status --short
```

Expected: lines beginning with `R` (rename) for the moved posts, `A` for the two new `_index.md` files. If you see `D` + `??` pairs instead, the moves were not staged as renames — that is acceptable to git but re-check that no file was lost.

- [ ] **Step 10: Commit**

```bash
git add -A
git commit -m "Split content/posts into research and making sections

Moves all five posts into two Hugo content sections and adds section
landing pages. The two already-published URLs keep working via aliases.

hello-world becomes a proper page bundle; its cover pointed at
posts/hello-world/testfigure.webp with relative:false, which the move
would have broken."
```

---

### Task 2: Point the home page at `research` and rebuild the menu

Makes the home page research-only, keeps archives complete, and adds the two content tabs.

**Files:**
- Modify: `hugo.yaml:18-28` (menu block)
- Modify: `hugo.yaml:30` (add two params at the top of `params:`)

**Interfaces:**
- Consumes: section names `research` and `making` from Task 1.
- Produces: `site.Params.mainSections == ["research"]`. Task 3 exists specifically because this value breaks the theme's prev/next partial for pages outside it.

- [ ] **Step 1: Verify the home page currently shows all four posts**

```bash
rm -rf "$OUT" && $HUGO --gc --minify --destination "$OUT"
python - "$OUT" <<'PY'
import os, re, sys
html = open(os.path.join(sys.argv[1], "index.html"), encoding="utf-8").read()
titles = [re.sub(r"<[^>]+>", "", t).strip()
          for t in re.findall(r'<h2 class=entry-hint-parent>(.*?)</h2>', html, re.S)]
print("home lists %d posts:" % len(titles))
for t in titles: print("   ", t)
PY
```

Expected before the change: **4** posts, including `FlowFocus更新预置日历和云同步功能`.

- [ ] **Step 2: Replace the menu block at `hugo.yaml:18-28`**

Current content:

```yaml
menu:
  main:
    - identifier: search
      name: search
      url: search
      weight: 1

    - identifier: archives
      name: archives
      url: archives/
      weight: 2
```

Replace with:

```yaml
menu:
  main:
    - identifier: research
      name: research
      url: /research/
      weight: 1

    - identifier: making
      name: making
      url: /making/
      weight: 2

    - identifier: archives
      name: archives
      url: archives/
      weight: 3

    - identifier: search
      name: search
      url: search
      weight: 4
```

- [ ] **Step 3: Add the two params directly under the `params:` line**

`hugo.yaml:30` currently reads `params:` followed by `  homeInfoParams:`. Insert two lines between them so it becomes:

```yaml
params:
  mainSections: ["research"]
  ShowAllPagesInArchive: true
  homeInfoParams:
```

`mainSections` is load-bearing: without it Hugo silently auto-selects the section with the most pages, which is why this must be explicit now that a second section exists. `ShowAllPagesInArchive` stops the archive page from hiding everything outside `mainSections`.

- [ ] **Step 4: Rebuild and verify home, sections, archives, search and menu**

```bash
rm -rf "$OUT" && $HUGO --gc --minify --destination "$OUT"
python - "$OUT" <<'PY'
import json, os, re, sys
out = sys.argv[1]
ok = True

def titles(path):
    html = open(os.path.join(out, path), encoding="utf-8").read()
    return [re.sub(r"<[^>]+>", "", t).strip()
            for t in re.findall(r'<h2 class=entry-hint-parent>(.*?)</h2>', html, re.S)]

home = titles("index.html")
ok &= (len(home) == 3);            print(("PASS " if len(home)==3 else "FAIL ") + f"home lists 3 posts (got {len(home)})")
no_ff = not any("FlowFocus" in t for t in home)
ok &= no_ff;                       print(("PASS " if no_ff else "FAIL ") + "FlowFocus absent from home")

mk = titles("making/index.html")
hit = len(mk) == 1 and "FlowFocus" in mk[0]
ok &= hit;                         print(("PASS " if hit else "FAIL ") + f"/making/ lists FlowFocus (got {mk})")

rs = titles("research/index.html")
ok &= (len(rs) == 3);              print(("PASS " if len(rs)==3 else "FAIL ") + f"/research/ lists 3 (got {len(rs)})")

arch = open(os.path.join(out, "archives", "index.html"), encoding="utf-8").read()
n = len(re.findall(r'class="?archive-entry"?', arch))
ok &= (n == 4);                    print(("PASS " if n==4 else "FAIL ") + f"archives lists all 4 (got {n})")

idx = json.load(open(os.path.join(out, "index.json"), encoding="utf-8"))
ok &= (len(idx) >= 4);             print(("PASS " if len(idx)>=4 else "FAIL ") + f"search index has >=4 entries (got {len(idx)})")

nav = re.findall(r'<a href=[^>]*>\s*<span[^>]*>([^<]+)</span>', arch)
menu = [m for m in nav if m.strip() in ("research", "making", "archives", "search")]
hit = {"research", "making"} <= set(x.strip() for x in menu)
ok &= hit;                         print(("PASS " if hit else "FAIL ") + f"menu has research+making (got {menu})")

for feed in ("index.xml", "research/index.xml", "making/index.xml"):
    e = os.path.isfile(os.path.join(out, feed))
    ok &= e;                       print(("PASS " if e else "FAIL ") + f"feed {feed}")

main = open(os.path.join(out, "index.xml"), encoding="utf-8").read()
hit = "FlowFocus" in main
ok &= hit;                         print(("PASS " if hit else "FAIL ") + "main feed still carries making posts")

print("\nRESULT:", "OK" if ok else "PROBLEMS")
sys.exit(0 if ok else 1)
PY
```

Expected: every line `PASS`, final `RESULT: OK`.

- [ ] **Step 5: Confirm existing tag pages still resolve**

```bash
python - "$OUT" <<'PY'
import os, sys
out = sys.argv[1]
for t in ("xai", "flowfocus", "产品设计", "可解释性"):
    p = os.path.join(out, "tags", t, "index.html")
    print(("PASS " if os.path.isfile(p) else "FAIL ") + "/tags/%s/" % t)
PY
```

Expected: all `PASS`.

- [ ] **Step 6: Commit**

```bash
git add hugo.yaml
git commit -m "Drive home page off mainSections and add section tabs

Sets mainSections explicitly to research so the home page shows only
research posts, and turns on ShowAllPagesInArchive so the archive still
covers both sections. Menu gains research and making ahead of the
utility links.

Before this, mainSections was never configured and Hugo auto-picked the
section with the most pages - fine with one section, unpredictable with
two."
```

---

### Task 3: Keep prev/next working on `making` posts

`layouts/partials/post_nav_links.html:1` in the theme filters by `mainSections`. After Task 2 a making post is not in that set, so the guard `(in $pages .)` is false and the entire prev/next nav disappears. This override scopes it to the current section instead.

**Files:**
- Create: `layouts/partials/post_nav_links.html`

**Interfaces:**
- Consumes: `mainSections: ["research"]` from Task 2 (this task is only necessary because of it).
- Produces: nothing other tasks consume.

- [ ] **Step 1: Prove the bug exists**

Drafts are included with `-D` so `hello-world` gives the FlowFocus post a neighbour inside `making`.

```bash
rm -rf "$OUT" && $HUGO -D --gc --minify --destination "$OUT"
python - "$OUT" <<'PY'
import os, sys
out = sys.argv[1]
for sec, slug in (("making", "flowfocus-ai-rebuild"), ("research", "forecasting")):
    html = open(os.path.join(out, sec, slug, "index.html"), encoding="utf-8").read()
    print("%-9s %-24s paginav present: %s" % (sec, slug, 'class=paginav' in html))
PY
```

Expected before the fix:
```
making    flowfocus-ai-rebuild     paginav present: False
research  forecasting              paginav present: True
```

- [ ] **Step 2: Create `layouts/partials/post_nav_links.html`**

Only line 1 differs from the theme's copy.

```go-html-template
{{- $pages := .CurrentSection.RegularPages.ByDate.Reverse }}
{{- if and (gt (len $pages) 1) (in $pages . ) }}
<nav class="paginav">
  {{- with $pages.Next . }}
  <a class="prev" href="{{ .Permalink }}">
    <span class="title">« {{ i18n "prev_page" }}</span>
    <br>
    <span>{{- .Name -}}</span>
  </a>
  {{- end }}
  {{- with $pages.Prev . }}
  <a class="next" href="{{ .Permalink }}">
    <span class="title">{{ i18n "next_page" }} »</span>
    <br>
    <span>{{- .Name -}}</span>
  </a>
  {{- end }}
</nav>
{{- end }}
```

- [ ] **Step 3: Rebuild and verify prev/next now works in both sections**

```bash
rm -rf "$OUT" && $HUGO -D --gc --minify --destination "$OUT"
python - "$OUT" <<'PY'
import os, re, sys
out = sys.argv[1]
ok = True
for sec, slug in (("making", "flowfocus-ai-rebuild"), ("research", "forecasting")):
    html = open(os.path.join(out, sec, slug, "index.html"), encoding="utf-8").read()
    hit = 'class=paginav' in html
    ok &= hit
    print(("PASS " if hit else "FAIL ") + f"{sec}/{slug} has prev/next")
    for href in re.findall(r'<a class=(?:prev|next) href=([^ >]+)', html):
        inside = ("/%s/" % sec) in href
        ok &= inside
        print(("  PASS " if inside else "  FAIL ") + f"link stays in section: {href}")
print("\nRESULT:", "OK" if ok else "PROBLEMS")
sys.exit(0 if ok else 1)
PY
```

Expected: every line `PASS`. Each post's prev/next links point within its own section.

- [ ] **Step 4: Confirm the submodule was not touched**

```bash
git status --short themes/
```

Expected: no output.

- [ ] **Step 5: Commit**

```bash
git add layouts/partials/post_nav_links.html
git commit -m "Scope prev/next links to the current section

The theme partial filters by mainSections, so after scoping the home
page to research the prev/next nav vanished entirely on making posts.
Walking CurrentSection instead keeps navigation inside each stream,
which is better than the original behaviour either way."
```

---

### Task 4: Load Maple Mono NF CN and apply it site-wide

Adds the webfont via ZeoSeven's subsetted CDN and points `body` at it.

**Files:**
- Create: `layouts/partials/extend_head.html`
- Create: `assets/css/extended/fonts.css`

**Interfaces:**
- Consumes: nothing from Phase A. This task is independent and could run first.
- Produces: CSS family name `"Maple Mono NF CN"`, referenced again in Task 5.

- [ ] **Step 1: Confirm the site currently uses the system font stack**

```bash
rm -rf "$OUT" && $HUGO --gc --minify --destination "$OUT"
python - "$OUT" <<'PY'
import glob, os, sys
css = open(glob.glob(os.path.join(sys.argv[1], "assets", "css", "stylesheet*.css"))[0],
           encoding="utf-8").read()
print("Maple referenced:", "Maple Mono NF CN" in css)
print("zeoseven link in home html:",
      "fontsapi.zeoseven.com" in open(os.path.join(sys.argv[1], "index.html"), encoding="utf-8").read())
PY
```

Expected before the change: both `False`.

- [ ] **Step 2: Create `layouts/partials/extend_head.html`**

This overrides the theme's empty stub, which `head.html:149` already calls.

```html
{{- /* Maple Mono NF CN, served as ~191 unicode-range subsets by ZeoSeven (item 442). */ -}}
{{- /* Must be a <link>, not @import: Hugo concatenates assets/css/extended/*.css after */ -}}
{{- /* the core CSS, and CSS silently discards any @import that is not at the top of the file. */ -}}
<link rel="preconnect" href="https://fontsapi.zeoseven.com" crossorigin>
<link rel="stylesheet" href="https://fontsapi.zeoseven.com/442/main/result.css">
```

- [ ] **Step 3: Create `assets/css/extended/fonts.css`**

Hugo concatenates `license + core + extended`, so this lands after the theme's `reset.css:26` and wins at equal specificity. No `!important` needed.

```css
body {
  font-family: "Maple Mono NF CN",
    -apple-system, BlinkMacSystemFont, 'Segoe UI', 'PingFang SC',
    'Microsoft YaHei', sans-serif;
}
```

- [ ] **Step 4: Rebuild and verify the font is wired up**

```bash
rm -rf "$OUT" && $HUGO --gc --minify --destination "$OUT"
python - "$OUT" <<'PY'
import glob, os, re, sys
out = sys.argv[1]; ok = True
css = open(glob.glob(os.path.join(out, "assets", "css", "stylesheet*.css"))[0], encoding="utf-8").read()

hit = "Maple Mono NF CN" in css
ok &= hit; print(("PASS " if hit else "FAIL ") + "family present in stylesheet")

m = re.search(r'body\{[^}]*font-family:([^;}]+)', css)
first = m.group(1).strip().split(',')[0] if m else None
hit = first == '"Maple Mono NF CN"'
ok &= hit; print(("PASS " if hit else "FAIL ") + f"body first font is Maple (got {first})")

hit = "sans-serif" in (m.group(1) if m else "")
ok &= hit; print(("PASS " if hit else "FAIL ") + "system fallback retained")

# the override must come after the theme rule, otherwise it loses
hit = css.rindex("Maple Mono NF CN") > css.rindex("BlinkMacSystemFont, 'Segoe UI', Roboto")
ok &= hit; print(("PASS " if hit else "FAIL ") + "override ordered after theme reset.css")

for page in ("index.html", "posts/index.html"):
    p = os.path.join(out, page)
    if not os.path.isfile(p): continue
    html = open(p, encoding="utf-8").read()
    a = "fontsapi.zeoseven.com/442/main/result.css" in html
    b = 'rel=preconnect' in html or 'rel="preconnect"' in html
    ok &= a and b
    print(("PASS " if a else "FAIL ") + f"{page}: stylesheet link")
    print(("PASS " if b else "FAIL ") + f"{page}: preconnect hint")

print("\nRESULT:", "OK" if ok else "PROBLEMS")
sys.exit(0 if ok else 1)
PY
```

Expected: every line `PASS`.

- [ ] **Step 5: Verify the CDN actually serves the font**

```bash
curl -s -o /dev/null -w "css   http=%{http_code} bytes=%{size_download}\n" \
  -A "Mozilla/5.0" https://fontsapi.zeoseven.com/442/main/result.css
```

Expected: `http=200` and roughly 150000 bytes. If this fails, the site still renders — the system fallback in Step 3 covers it — but stop and report rather than continuing.

- [ ] **Step 6: Look at it in a browser**

```bash
$HUGO server --renderToMemory
```

Open <http://localhost:1313>. Confirm Chinese *and* Latin text both render in Maple Mono (rounded, fixed-width) rather than only the Latin changing. `--renderToMemory` keeps `public/` clean — plain `hugo server` writes into it.

Then stop the server and confirm nothing leaked:

```bash
git status --short
```

Expected: only the two new files from this task.

- [ ] **Step 7: Commit**

```bash
git add layouts/partials/extend_head.html assets/css/extended/fonts.css
git commit -m "Switch site font to Maple Mono NF CN

Loaded from ZeoSeven's FontsAPI (item 442), which serves the CJK variant
as ~191 unicode-range subsets - a real post pulls roughly 1.1 MB across
~21 requests, cached immutable for a year and shared between posts.
Upstream ships no WOFF2 build of the CN variant at all; the only official
web build is Latin-only.

Uses <link> rather than @import because Hugo concatenates extended CSS
after the core, and CSS drops any @import that is not at the top of the
file - silently, with no build error."
```

---

### Task 5: Stop headings from using synthetic bold

Endpoint 442 ships weight 400 only, so any bold text is faux-bolded by the browser. Headings switch to 400 and rely on size for hierarchy.

**Files:**
- Modify: `assets/css/extended/fonts.css` (append)

**Interfaces:**
- Consumes: the `"Maple Mono NF CN"` family loaded in Task 4.
- Produces: nothing other tasks consume. Final task.

- [ ] **Step 1: Confirm headings are currently bold**

```bash
rm -rf "$OUT" && $HUGO --gc --minify --destination "$OUT"
python - "$OUT" <<'PY'
import glob, os, sys
css = open(glob.glob(os.path.join(sys.argv[1], "assets", "css", "stylesheet*.css"))[0],
           encoding="utf-8").read()
print(".post-title weight override present:", ".post-title" in css and "font-weight:400" in css)
PY
```

Expected before the change: `False`. `.post-title` (`post-single.css:6`) and `.post-content h1`–`h6` (`post-single.css:39-62`) set only size and margins — their bold comes from the browser's UA stylesheet, so it must be overridden explicitly. `.logo a` (`header.css:25`) does set `font-weight: 700`.

- [ ] **Step 2: Append the heading rules to `assets/css/extended/fonts.css`**

```css
/* ZeoSeven endpoint 442 ships weight 400 only, so anything bold is
   synthesised by the browser. Headings use size for hierarchy instead. */
.post-title,
.post-content h1, .post-content h2, .post-content h3,
.post-content h4, .post-content h5, .post-content h6,
.entry-header h2,
.first-entry .entry-header h1,
.logo a {
  font-weight: 400;
}
```

- [ ] **Step 3: Rebuild and verify**

```bash
rm -rf "$OUT" && $HUGO --gc --minify --destination "$OUT"
python - "$OUT" <<'PY'
import glob, os, re, sys
css = open(glob.glob(os.path.join(sys.argv[1], "assets", "css", "stylesheet*.css"))[0],
           encoding="utf-8").read()
ok = True
for sel in (".post-title", ".post-content h1", ".entry-header h2", ".logo a"):
    m = re.search(re.escape(sel) + r'[^{}]*\{[^}]*font-weight:400', css)
    ok &= bool(m); print(("PASS " if m else "FAIL ") + f"{sel} -> 400")
# theme's .logo a{font-weight:700} must be overridden, i.e. ours comes later
hit = css.rindex("font-weight:400") > css.find("font-weight:700")
ok &= hit; print(("PASS " if hit else "FAIL ") + "override ordered after theme")
print("\nRESULT:", "OK" if ok else "PROBLEMS")
sys.exit(0 if ok else 1)
PY
```

Expected: every line `PASS`.

- [ ] **Step 4: Judge it by eye**

```bash
$HUGO server --renderToMemory
```

Open a post. Headings should read as clean 400-weight Maple, distinguished by size rather than weight.

While there, look at inline `<strong>` in the body — the XAI post has 24 instances, FlowFocus 14. Those are **still** synthetic bold; this task deliberately only covers headings. If they look smeared, report back rather than fixing it here; it is an open item in the spec, not part of this plan.

- [ ] **Step 5: Commit**

```bash
git add assets/css/extended/fonts.css
git commit -m "Use weight 400 for headings

The ZeoSeven endpoint serves a single weight, so bold headings were being
synthesised by the browser. Headings now rely on size for hierarchy.
Inline <strong> is deliberately left alone pending a look at real pages."
```

---

## Final verification

After all five tasks, run the full build the way CI does and re-check everything at once:

```bash
cd C:/Users/s398876/Documents/kxqdesign/kxqdesign.github.io
rm -rf "$OUT" && $HUGO --gc --minify --destination "$OUT"
git status --short          # expect: clean
git log --oneline -6
```

Expected: build reports no `ERROR`, working tree clean, five new commits on top of `acbf344`.

Then confirm `public/` was never dirtied:

```bash
git status --short public resources
```

Expected: no output. If not: `git checkout -- public && git clean -fd public resources`.

The user pushes when ready — this plan never pushes.

## Out of scope

- Treating inline `<strong>` (spec open item 3 — decide after seeing real pages)
- Self-hosting the font instead of using ZeoSeven (spec open item 4 — only if the CDN proves unreliable)
- Adding a `.gitignore`, or removing `public/` from version control
- Any change to `themes/PaperMod/`
