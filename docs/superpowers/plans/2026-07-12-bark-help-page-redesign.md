# Bark Help Page Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the packaged Bark help page with a polished, responsive interface whose examples and copy actions always use the current LazyCat application origin.

**Architecture:** Keep the help experience as one dependency-free `dist/tours.html` file served by the existing `/help` file upstream. CSS owns layout and motion; a small inline script derives example URLs from `window.location.origin`, updates every bound text node, and provides Clipboard API plus selection fallback behavior.

**Tech Stack:** Semantic HTML5, modern CSS, vanilla browser JavaScript, LazyCat LPK v2 content packaging.

## Global Constraints

- Load no remote fonts, scripts, images, or styles.
- Derive every Bark push example from `window.location.origin`; no `api.day.app` push example may remain.
- Select Chinese for `zh*` browser locales and English otherwise, including copy feedback and example path segments.
- Keep interaction transitions below 250ms and animate only transform, opacity, color, background, border, and box-shadow.
- Support `prefers-reduced-motion`, visible keyboard focus, touch layouts, and clipboard fallback.
- Preserve the existing `/help` upstream and launcher entry.
- Remove the manifest-level `usage` field.

---

### Task 1: Replace the static help interface

**Files:**
- Modify: `dist/tours.html`

**Interfaces:**
- Consumes: `window.location.origin`, browser Clipboard API, the existing `/help` static route.
- Produces: `buildPushURL(pathSegments: string[], query?: URLSearchParams): string`, `copyText(text: string): Promise<void>`, and DOM elements with `data-example` and `data-copy-target` identifiers.

- [ ] **Step 1: Record failing content checks**

Run:

```bash
rg -n 'https://api\.day\.app/YourKey|window\.location\.origin|data-copy-target|prefers-reduced-motion' dist/tours.html
```

Expected before implementation: the official `api.day.app` examples are present, while copy controls and reduced-motion handling are absent.

- [ ] **Step 2: Implement the semantic page structure**

Replace `dist/tours.html` with a complete document containing:

```html
<main class="shell">
  <header class="hero reveal">...</header>
  <section class="steps" aria-labelledby="setup-title">...</section>
  <section class="examples" aria-labelledby="examples-title">
    <article class="code-card">
      <code id="basic-example" data-example="basic"></code>
      <button type="button" class="copy-button" data-copy-target="basic-example">复制</button>
    </article>
    <article class="code-card">
      <code id="advanced-example" data-example="advanced"></code>
      <button type="button" class="copy-button" data-copy-target="advanced-example">复制</button>
    </article>
  </section>
  <section class="parameters" aria-labelledby="parameters-title">...</section>
  <aside class="security-note">...</aside>
  <p class="sr-only" id="copy-status" aria-live="polite"></p>
</main>
```

The complete page must retain the App Store and source-code links, explain `YourKey`, title, and content, and list `title`, `body`, `badge`, `sound`, `icon`, `group`, and `url` parameters.

- [ ] **Step 3: Implement instance-aware examples and copying**

Add one inline script with these exact behaviors:

```js
const origin = window.location.origin.replace(/\/$/, "");

function buildPushURL(pathSegments, query) {
  const encodedPath = pathSegments.map((segment) => encodeURIComponent(segment)).join("/");
  const suffix = query && query.toString() ? `?${query.toString()}` : "";
  return `${origin}/${encodedPath}${suffix}`;
}

async function copyText(text) {
  if (navigator.clipboard && window.isSecureContext) {
    await navigator.clipboard.writeText(text);
    return;
  }
  const input = document.createElement("textarea");
  input.value = text;
  input.setAttribute("readonly", "");
  input.style.position = "fixed";
  input.style.opacity = "0";
  document.body.appendChild(input);
  input.select();
  const copied = document.execCommand("copy");
  input.remove();
  if (!copied) throw new Error("copy failed");
}

const basicExample = buildPushURL(["YourKey", "消息标题", "消息内容"]);
const advancedQuery = new URLSearchParams({ icon: "https://day.app/assets/images/avatar.jpg" });
const advancedExample = buildPushURL(["YourKey", "消息标题", "消息内容"], advancedQuery);
```

Set the visible instance label and both code elements from these values. Bind each copy button to its target code element, change its label to `已复制` for 1600ms on success or `复制失败` on error, update `#copy-status`, and always restore the original label.

- [ ] **Step 4: Implement polished responsive styling and motion**

Define tokens including:

```css
:root {
  --accent: #22c55e;
  --accent-strong: #15803d;
  --ink: #152019;
  --muted: #657169;
  --surface: rgba(255, 255, 255, 0.88);
  --line: rgba(21, 32, 25, 0.1);
  --ease-out: cubic-bezier(0.23, 1, 0.32, 1);
}
```

Use a centered responsive shell, soft green ambient background, compact hero badge, numbered step cards, code cards with horizontally scrollable code, a responsive parameter grid, and a distinct security panel. Buttons must use exact-property transitions, `transform: scale(0.97)` on `:active`, visible `:focus-visible`, and hover elevation only inside `@media (hover: hover) and (pointer: fine)`.

Add short initial reveal animation with 30–60ms stagger increments and this reduced-motion override:

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    scroll-behavior: auto !important;
  }
  .reveal {
    animation: none;
  }
  .step-card,
  .code-card,
  .copy-button {
    transition-duration: 1ms;
  }
}
```

- [ ] **Step 5: Run content and dependency checks**

Run:

```bash
! rg -n 'https://api\.day\.app/YourKey|transition:\s*all|scale\(0\)|ease-in\b' dist/tours.html
rg -n 'window\.location\.origin|data-copy-target|prefers-reduced-motion|aria-live' dist/tours.html
! rg -n '<script[^>]+src=|<link[^>]+href=|@import' dist/tours.html
```

Expected: all negative checks return no matches; all required behavior checks match.

### Task 2: Remove manifest usage and verify the packaged artifact

**Files:**
- Modify: `lzc-manifest.yml`
- Verify: `lzc-build.yml`, `dist/tours.html`, `dist/startup.sh`

**Interfaces:**
- Consumes: existing LPK v2 build configuration and content directory.
- Produces: `/tmp/bark-server-redesign.lpk` containing the redesigned help page with no manifest `usage` field.

- [ ] **Step 1: Remove usage**

Delete this top-level mapping from `lzc-manifest.yml`:

```yaml
usage: "请打开应用的“使用帮助”入口，或通过应用域名的 /help 路径查看 Bark Server 使用说明。"
```

- [ ] **Step 2: Validate source files**

Run:

```bash
sh -n dist/startup.sh
git diff --check
go run github.com/rhysd/actionlint/cmd/actionlint@latest .github/workflows/lazycat.yml
```

Expected: all commands exit successfully.

- [ ] **Step 3: Build and inspect the LPK**

Run:

```bash
lzc-cli project build -f lzc-build.yml -o /tmp/bark-server-redesign.lpk
tar -xOf /tmp/bark-server-redesign.lpk manifest.yml > /tmp/bark-manifest.yml
tar -xOf /tmp/bark-server-redesign.lpk content.tar > /tmp/bark-content.tar
tar -xOf /tmp/bark-content.tar tours.html > /tmp/bark-tours.html
! rg -n '^usage:' /tmp/bark-manifest.yml
rg -n 'window\.location\.origin|data-copy-target|prefers-reduced-motion' /tmp/bark-tours.html
```

Expected: build succeeds, packaged manifest has no usage field, and packaged help contains all required dynamic behavior.

### Task 3: Commit, push, and publish

**Files:**
- Modify: repository history and remote workflow state only.

**Interfaces:**
- Consumes: verified source tree and existing `.github/workflows/lazycat.yml`.
- Produces: a pushed commit and a completed manually dispatched dual-store workflow.

- [ ] **Step 1: Commit implementation**

Run:

```bash
git add dist/tours.html lzc-manifest.yml docs/superpowers/plans/2026-07-12-bark-help-page-redesign.md
git commit -m "feat: redesign Bark help experience"
git push origin main
```

Expected: `main` is pushed successfully.

- [ ] **Step 2: Trigger the workflow**

Run:

```bash
gh workflow run .github/workflows/lazycat.yml --ref main
gh run list --workflow .github/workflows/lazycat.yml --limit 1
```

Expected: a new `workflow_dispatch` run appears for the implementation commit.

- [ ] **Step 3: Verify release completion**

Run:

```bash
gh run watch <run-id> --exit-status
```

Expected: image automation, Release asset, MiaoMiao private store, and LazyCat official store steps complete successfully.
