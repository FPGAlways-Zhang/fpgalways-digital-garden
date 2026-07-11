# Cloudflare Pages Migration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Keep the Quartz blog source and history on GitHub while moving all production builds and hosting from GitHub Pages to Cloudflare Pages Git integration.

**Architecture:** The local Obsidian/Quartz repository continues to push the `v5` branch to GitHub. Cloudflare Pages connects to that repository through its GitHub App, runs the Quartz build for every production push, and serves the generated `public` directory. GitHub Pages remains available as a temporary fallback until the Cloudflare deployment passes public verification, then it is deleted.

**Tech Stack:** Obsidian Markdown, Quartz 5, Git, GitHub, Cloudflare Pages, Node.js 24, npm, PowerShell

---

## File Map

- Modify: `quartz.config.yaml` — change the canonical production host from GitHub Pages to Cloudflare Pages.
- Delete: `.github/workflows/deploy-pages.yaml` — remove the project-owned GitHub Pages deployment workflow.
- Modify: `content/项目与作品/FPGAlways数字花园.md` — replace GitHub Pages setup and update instructions with Cloudflare Pages Git integration instructions.
- Read only: `docs/superpowers/specs/2026-07-11-cloudflare-pages-migration-design.md` — source of approved requirements.
- External configuration: Cloudflare Pages project `fpgalways-digital-garden` and GitHub Pages settings for `FPGAlways-Zhang/fpgalways-digital-garden`.

### Task 1: Verify the migration baseline

**Files:**
- Read: `quartz.config.yaml`
- Read: `.github/workflows/deploy-pages.yaml`
- Read: `content/项目与作品/FPGAlways数字花园.md`

- [ ] **Step 1: Inspect the current worktree without discarding the existing documentation edit**

Run:

```powershell
cd F:\QuartzObsidianBlog
git -c core.quotepath=false status -sb
git log -2 --oneline
```

Expected:

```text
## v5...origin/v5 [ahead 1]
 M content/项目与作品/FPGAlways数字花园.md
9dbf0fb 记录 Cloudflare Pages 迁移设计
```

- [ ] **Step 2: Confirm the old GitHub Pages site still exists**

Run:

```powershell
F:\Tools\GitHubCLI\bin\gh.exe api repos/FPGAlways-Zhang/fpgalways-digital-garden/pages
```

Expected: JSON contains both `"build_type":"workflow"` and the old `github.io/fpgalways-digital-garden` address.

- [ ] **Step 3: Confirm the current blog builds before migration changes**

Run:

```powershell
npx.cmd quartz build
```

Expected: exit code `0`, 13 Markdown input files are parsed, and files are emitted to `public`.

### Task 2: Replace GitHub Pages configuration with Cloudflare Pages configuration

**Files:**
- Modify: `quartz.config.yaml`
- Delete: `.github/workflows/deploy-pages.yaml`
- Modify: `content/项目与作品/FPGAlways数字花园.md`

- [ ] **Step 1: Change the Quartz production host**

In `quartz.config.yaml`, replace:

```yaml
  baseUrl: FPGAlways-Zhang.github.io/fpgalways-digital-garden
```

with:

```yaml
  baseUrl: fpgalways-digital-garden.pages.dev
```

- [ ] **Step 2: Remove the project-owned GitHub Pages workflow**

Delete:

```text
.github/workflows/deploy-pages.yaml
```

Do not delete the Quartz upstream workflows. Their repository guards prevent the upstream deployment jobs from publishing this personal blog.

- [ ] **Step 3: Update the technology and project information in the blog note**

In `content/项目与作品/FPGAlways数字花园.md`, use these values:

```markdown
- 代码托管：GitHub
- 免费部署：Cloudflare Pages
```

and:

```markdown
- Cloudflare Pages 项目：`fpgalways-digital-garden`
- 博客地址：[FPGAlways的数字花园](https://fpgalways-digital-garden.pages.dev/)
```

- [ ] **Step 4: Replace the automatic deployment section**

Use this content for the deployment section:

```markdown
### 5. 配置 Cloudflare Pages 自动部署

Cloudflare Pages 直接连接 GitHub 仓库，不使用 GitHub Actions 上传构建产物。项目配置如下：

- GitHub 仓库：`FPGAlways-Zhang/fpgalways-digital-garden`
- 生产分支：`v5`
- 根目录：仓库根目录
- 构建命令：`npm ci && npx quartz plugin install && npx quartz build`
- 输出目录：`public`
- 环境变量：`NODE_VERSION=24`

每次向 `v5` 分支执行 `git push` 后，Cloudflare Pages 会自动拉取最新代码、安装 Quartz 插件、构建网站并发布。
```

- [ ] **Step 5: Replace the deployment-result instructions**

Use this content:

````markdown
### 4. 查看部署结果

推送后可以在 Cloudflare 控制台的 Workers & Pages 项目中查看部署进度。最新生产部署显示成功后，刷新博客地址即可看到更新：

```text
https://fpgalways-digital-garden.pages.dev/
```
````

- [ ] **Step 6: Check that active instructions no longer describe GitHub Pages deployment**

Run:

```powershell
rg -n "GitHub Pages|deploy-pages.yaml|FPGAlways-Zhang.github.io" quartz.config.yaml content\项目与作品\FPGAlways数字花园.md
```

Expected: no active setup or update instructions reference GitHub Pages. A short historical note that the site migrated away from GitHub Pages is acceptable.

### Task 3: Validate and commit the local migration

**Files:**
- Modify: `quartz.config.yaml`
- Delete: `.github/workflows/deploy-pages.yaml`
- Modify: `content/项目与作品/FPGAlways数字花园.md`

- [ ] **Step 1: Run the production build**

Run:

```powershell
npx.cmd quartz build
```

Expected: exit code `0`; the blog is emitted to `public` without missing-plugin errors.

- [ ] **Step 2: Check whitespace and scope**

Run:

```powershell
git diff --check
git -c core.quotepath=false status --short
git -c core.quotepath=false diff --stat
```

Expected changed paths:

```text
D  .github/workflows/deploy-pages.yaml
M  content/项目与作品/FPGAlways数字花园.md
M  quartz.config.yaml
```

The already committed design and plan documents may appear in branch history but must not be modified by this task.

- [ ] **Step 3: Stage only the intended migration files**

Run:

```powershell
git add -- quartz.config.yaml content/项目与作品/FPGAlways数字花园.md .github/workflows/deploy-pages.yaml
git -c core.quotepath=false diff --cached --stat
```

Expected: the cached diff contains only the three migration paths.

- [ ] **Step 4: Commit the migration**

Run:

```powershell
git commit -m "迁移博客部署配置到 Cloudflare Pages"
```

Expected: one commit containing the configuration change, workflow deletion, and updated operating instructions.

### Task 4: Push the migration to GitHub

**Files:**
- No additional file changes.

- [ ] **Step 1: Verify GitHub authentication and remote target**

Run:

```powershell
F:\Tools\GitHubCLI\bin\gh.exe auth status
git remote -v
```

Expected: active account is `FPGAlways-Zhang`; `origin` points to `FPGAlways-Zhang/fpgalways-digital-garden.git`.

- [ ] **Step 2: Push the `v5` branch**

Run:

```powershell
git push origin v5
```

Expected: the design, plan, documentation, and migration commits are uploaded to `origin/v5`.

- [ ] **Step 3: Verify branch synchronization**

Run:

```powershell
git status -sb
```

Expected:

```text
## v5...origin/v5
```

### Task 5: Create the Cloudflare Pages Git integration

**Files:**
- External Cloudflare configuration only.

- [ ] **Step 1: Open Cloudflare and create a Pages project**

In the Cloudflare dashboard:

1. Open **Workers & Pages**.
2. Select **Create application** or **Create**.
3. Choose **Pages** and **Connect to Git**.
4. Connect or authorize the GitHub account `FPGAlways-Zhang`.
5. Grant access to `FPGAlways-Zhang/fpgalways-digital-garden`.
6. Select that repository.

Expected: Cloudflare displays the build configuration page for the selected repository.

- [ ] **Step 2: Enter the production settings**

Use exactly:

```text
Project name: fpgalways-digital-garden
Production branch: v5
Framework preset: None
Root directory: leave blank (repository root)
Build command: npm ci && npx quartz plugin install && npx quartz build
Build output directory: public
Environment variable: NODE_VERSION = 24
```

Expected default production domain:

```text
https://fpgalways-digital-garden.pages.dev
```

- [ ] **Step 3: Start the first deployment**

Select **Save and Deploy**.

Expected Cloudflare build stages:

1. Repository clone succeeds.
2. `npm ci` succeeds.
3. Quartz plugins install successfully.
4. Quartz build emits the `public` directory.
5. Deployment status becomes **Success**.

If any build stage fails, stop here and keep GitHub Pages enabled while correcting the Cloudflare build settings or repository configuration.

### Task 6: Verify the Cloudflare production site

**Files:**
- No file changes.

- [ ] **Step 1: Verify the homepage over the public internet**

Run:

```powershell
$response = Invoke-WebRequest -Uri 'https://fpgalways-digital-garden.pages.dev/' -MaximumRedirection 5 -TimeoutSec 30
$response.StatusCode
([regex]::Match($response.Content, '<title>(.*?)</title>', 'IgnoreCase').Groups[1].Value)
```

Expected:

```text
200
FPGAlways的数字花园
```

- [ ] **Step 2: Verify a Chinese article path**

Run:

```powershell
$article = Invoke-WebRequest -Uri 'https://fpgalways-digital-garden.pages.dev/项目与作品/FPGAlways数字花园' -MaximumRedirection 5 -TimeoutSec 30
$article.StatusCode
```

Expected:

```text
200
```

- [ ] **Step 3: Verify repository-subpath links are gone**

Run:

```powershell
($response.Content -match 'href="/fpgalways-digital-garden/')
```

Expected:

```text
False
```

- [ ] **Step 4: Perform a browser smoke test**

Open the Cloudflare URL and verify:

- The left explorer expands and opens Chinese notes.
- Search opens normally.
- Dark mode works.
- Article navigation does not add `/fpgalways-digital-garden/` to the path.
- Static styles, fonts, and graph assets load without visible errors.

### Task 7: Disable the old GitHub Pages site

**Files:**
- External GitHub Pages configuration only.

- [ ] **Step 1: Apply the safety gate**

Proceed only if every step in Task 6 passed. If Cloudflare verification failed, do not run the deletion command.

- [ ] **Step 2: Delete the GitHub Pages site configuration**

Run:

```powershell
F:\Tools\GitHubCLI\bin\gh.exe api --method DELETE repos/FPGAlways-Zhang/fpgalways-digital-garden/pages
```

Expected: HTTP `204 No Content` and no error output.

- [ ] **Step 3: Verify GitHub Pages is disabled**

Run:

```powershell
F:\Tools\GitHubCLI\bin\gh.exe api repos/FPGAlways-Zhang/fpgalways-digital-garden/pages
```

Expected: HTTP `404 Not Found`, showing that GitHub no longer has an active Pages site for the repository.

### Task 8: Run final end-to-end checks

**Files:**
- No additional file changes.

- [ ] **Step 1: Verify the branch and worktree**

Run:

```powershell
git status -sb
git log -4 --oneline
```

Expected: `v5` is synchronized with `origin/v5`, with no uncommitted files.

- [ ] **Step 2: Verify the final hosting state**

Run:

```powershell
(Invoke-WebRequest -Uri 'https://fpgalways-digital-garden.pages.dev/' -TimeoutSec 30).StatusCode
F:\Tools\GitHubCLI\bin\gh.exe api repos/FPGAlways-Zhang/fpgalways-digital-garden/pages
```

Expected:

- Cloudflare URL returns `200`.
- GitHub Pages API returns `404`.

- [ ] **Step 3: Confirm the ongoing publishing workflow**

Future blog updates use:

```powershell
git add content
git commit -m "更新博客文章"
git push
```

Expected: GitHub records the new commit and Cloudflare Pages automatically starts a new deployment from the `v5` branch.
