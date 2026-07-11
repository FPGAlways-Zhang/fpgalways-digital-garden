# Explorer 一级目录对齐 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 让桌面端 Explorer 的一级目录箭头与“探索”标题左边缘对齐，并移除标题与第一项之间的额外空白。

**Architecture:** 仅在 `quartz/styles/custom.scss` 增加桌面端覆盖规则，不修改 Quartz 插件源码。使用本地 Quartz 预览中的 DOM 几何数据建立失败断言和修复后断言，再运行项目测试与正式构建。

**Tech Stack:** Quartz v5、SCSS、浏览器 DOM/Computed Style、Node.js/npm、Git

---

### Task 1: 建立失败的布局断言

**Files:**
- Read: `quartz/styles/custom.scss:64-170`
- Read: `.quartz/plugins/explorer/src/components/styles/explorer.scss:21-100`
- Read: `quartz/styles/base.scss:628-652`

- [ ] **Step 1: 启动或确认本地预览**

Run:

```powershell
cd F:\QuartzObsidianBlog
npx.cmd quartz build --serve
```

Expected: Quartz 在 `http://localhost:8080/` 提供本地页面。

- [ ] **Step 2: 在文章页面采集修复前几何数据**

在浏览器页面上下文中运行：

```js
(() => {
  const header = document.querySelector("button.desktop-explorer h2")
  const firstFolder = document.querySelector(
    ".explorer-content > ul.explorer-ul > li:not(.overflow-end) > .folder-container",
  )
  const firstIcon = firstFolder?.querySelector("svg.folder-icon")
  const headerButton = document.querySelector("button.desktop-explorer")

  if (!header || !firstFolder || !firstIcon || !headerButton) {
    throw new Error("Explorer geometry targets are missing")
  }

  const headerRect = header.getBoundingClientRect()
  const iconRect = firstIcon.getBoundingClientRect()
  const buttonRect = headerButton.getBoundingClientRect()

  return {
    horizontalOffset: iconRect.left - headerRect.left,
    verticalGap: iconRect.top - buttonRect.bottom,
  }
})()
```

Expected before the fix: `horizontalOffset` is about `6.8px`, and `verticalGap` is about `27.9px`.

- [ ] **Step 3: 确认断言失败**

在同一页面上下文中运行：

```js
(() => {
  const header = document.querySelector("button.desktop-explorer h2")
  const firstFolder = document.querySelector(
    ".explorer-content > ul.explorer-ul > li:not(.overflow-end) > .folder-container",
  )
  const firstIcon = firstFolder?.querySelector("svg.folder-icon")
  const headerButton = document.querySelector("button.desktop-explorer")

  if (!header || !firstIcon || !headerButton) return false

  const horizontalOffset =
    firstIcon.getBoundingClientRect().left - header.getBoundingClientRect().left
  const verticalGap =
    firstIcon.getBoundingClientRect().top - headerButton.getBoundingClientRect().bottom

  return Math.abs(horizontalOffset) < 1 && verticalGap < 18
})()
```

Expected: `false`.

### Task 2: 实施最小 SCSS 修复

**Files:**
- Modify: `quartz/styles/custom.scss`，在 `.page .explorer` 规则结束后、移动端媒体查询前加入桌面端覆盖

- [ ] **Step 1: 添加桌面端根目录对齐规则**

在 `quartz/styles/custom.scss` 中加入：

```scss
@media all and (min-width: 801px) {
  .page .explorer .explorer-content {
    margin-top: 0;
  }

  .page .explorer .explorer-content > ul.explorer-ul > li.overflow-end {
    height: 0;
    margin: 0;
    padding: 0;
  }

  .page
    .explorer
    .explorer-content
    > ul.explorer-ul
    > li
    > .folder-container {
    border-left-width: 0;
    padding-left: 0;
  }
}
```

- [ ] **Step 2: 检查格式和变更范围**

Run:

```powershell
npm.cmd exec prettier -- --check quartz/styles/custom.scss
git diff --check
git diff -- quartz/styles/custom.scss
```

Expected: Prettier 和空白检查通过，差异只包含上述桌面端 Explorer 覆盖规则。

### Task 3: 验证布局和交互

**Files:**
- Test: 本地文章页面 `http://localhost:8080/项目与作品/fpgalways数字花园`

- [ ] **Step 1: 刷新本地页面并重新运行几何断言**

运行 Task 1 Step 3 的断言。

Expected: `true`；一级箭头与标题左边缘误差小于 `1px`，垂直间隔小于 `18px`。

- [ ] **Step 2: 验证嵌套目录没有被拉平**

在“项目与作品”展开状态下采集根目录箭头和子笔记链接的左坐标：

```js
(() => {
  const rootIcon = document.querySelector(
    ".explorer-content > ul.explorer-ul > li:not(.overflow-end) svg.folder-icon",
  )
  const nestedLink = document.querySelector(
    ".explorer-content .folder-outer.open a.nav-file-title",
  )

  if (!rootIcon || !nestedLink) return false
  return nestedLink.getBoundingClientRect().left > rootIcon.getBoundingClientRect().left
})()
```

Expected: `true`，子笔记仍然保持缩进。

- [ ] **Step 3: 验证展开、折叠和主题模式**

依次执行：

1. 折叠并重新展开“探索”。
2. 切换深色模式，检查一级箭头对齐和顶部间距。
3. 切回浅色模式。
4. 确认选中笔记的蓝色背景和左侧强调线保持正常。

Expected: 两种主题和两种 Explorer 状态均无裁切、错位或层级丢失。

### Task 4: 回归测试、提交和发布

**Files:**
- Modify: `quartz/styles/custom.scss`
- Preserve: `.gitignore` 的现有未提交修改

- [ ] **Step 1: 运行项目测试**

Run:

```powershell
npm.cmd test
```

Expected: 109 项测试全部通过。

- [ ] **Step 2: 运行正式构建**

Run:

```powershell
npx.cmd quartz build
```

Expected: Quartz 成功生成 `public`，无构建错误。

- [ ] **Step 3: 只提交本任务的样式文件**

Run:

```powershell
git add quartz/styles/custom.scss
git commit -m "style: 对齐左侧一级目录"
```

Expected: 提交不包含 `.gitignore`。

- [ ] **Step 4: 推送并核对远端分支**

Run:

```powershell
git push origin v5
git status -sb
```

Expected: `v5` 与 `origin/v5` 同步；工作区只保留用户原有的 `.gitignore` 修改。
