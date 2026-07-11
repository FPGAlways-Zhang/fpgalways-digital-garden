# Cloudflare Pages 迁移设计

## 背景

FPGAlways 数字花园目前将源码保存在 GitHub，并通过 GitHub Actions 构建后部署到 GitHub Pages。新的目标是继续使用 GitHub 进行版本追踪，同时让 Cloudflare Pages 直接连接 GitHub 仓库、构建 Quartz 并提供网站访问服务。

## 目标

- GitHub 继续保存 Quartz 源码、Obsidian 内容和完整提交历史。
- Cloudflare Pages 直接监听 GitHub 仓库的 `v5` 分支。
- 每次推送后由 Cloudflare 自动安装依赖、构建并发布网站。
- 移除项目自建的 GitHub Pages 部署工作流。
- Cloudflare 网站验证成功后关闭现有 GitHub Pages 站点。
- 更新 Quartz 的正式站点地址和博客操作说明。

## 非目标

- 不使用 GitHub Actions 向 Cloudflare 上传构建产物。
- 不在 GitHub Secrets 中保存 Cloudflare API Token。
- 不改变 Obsidian Vault 位置、文章目录结构或 Quartz 默认布局。
- 不删除 GitHub 仓库、提交历史、`origin` 或 `upstream`。

## 目标架构

```text
Obsidian 写作
  → 本地 Git 提交
  → 推送到 GitHub v5 分支
  → Cloudflare Pages 拉取仓库
  → Cloudflare 构建 Quartz
  → 发布到 pages.dev
```

GitHub 只负责源码托管和版本管理。Cloudflare Pages 负责依赖安装、Quartz 构建、部署、HTTPS 和 CDN 分发。

## Cloudflare Pages 配置

- 项目名称：`fpgalways-digital-garden`
- GitHub 仓库：`FPGAlways-Zhang/fpgalways-digital-garden`
- 生产分支：`v5`
- 根目录：仓库根目录
- 构建命令：`npm ci && npx quartz plugin install && npx quartz build`
- 输出目录：`public`
- Node.js 版本：`24`
- 默认域名：`https://fpgalways-digital-garden.pages.dev`

Cloudflare 通过 GitHub App 读取仓库并接收推送事件，不依赖 GitHub Actions。

## 仓库修改

1. 删除 `.github/workflows/deploy-pages.yaml`，停止项目自建的 GitHub Pages 自动部署。
2. 将 `quartz.config.yaml` 中的 `baseUrl` 改为 `fpgalways-digital-garden.pages.dev`。
3. 更新 `content/项目与作品/FPGAlways数字花园.md`，用 Cloudflare Pages 流程替换 GitHub Pages 流程。
4. Quartz 上游自带的工作流保留；其中受官方仓库条件限制的任务不负责本博客部署。

## 迁移顺序

1. 在本地完成仓库修改并验证 Quartz 构建。
2. 提交并推送到 GitHub 的 `v5` 分支。
3. 在 Cloudflare Pages 中连接 GitHub 仓库并填写构建配置。
4. 等待首次构建完成。
5. 验证 Cloudflare 首页、分类页、中文文章路径、静态资源和站内链接。
6. Cloudflare 网站确认正常后，通过 GitHub API 删除现有 GitHub Pages 站点。
7. 再次确认 GitHub 仓库仍可正常提交和推送，且后续推送会触发 Cloudflare 部署。

该顺序保留现有 GitHub Pages 网站直到 Cloudflare 部署可用，避免迁移过程中出现无站点可访问的时间窗口。

## 验证标准

- 本地执行 `npx.cmd quartz build` 返回成功。
- Cloudflare 首次构建完成且部署状态成功。
- `https://fpgalways-digital-garden.pages.dev` 返回 HTTP 200。
- 页面标题为“FPGAlways的数字花园”。
- 首页和中文文章链接不再包含 `/fpgalways-digital-garden/` 仓库子路径。
- GitHub Pages API 在迁移完成后返回未启用状态。
- 本地 `v5` 分支与 `origin/v5` 保持同步。

## 失败处理与回退

- Cloudflare 构建失败时，保留 GitHub Pages，不执行关闭操作。
- 优先根据 Cloudflare 构建日志修正 Node.js 版本、构建命令或输出目录。
- 如果新域名下链接异常，检查 Quartz `baseUrl` 是否与 Cloudflare 域名一致。
- 在 Cloudflare 验证完成前，原 GitHub Pages 网站继续作为临时回退地址。

## 后续更新流程

日常写作流程保持简单：

```powershell
git add content
git commit -m "更新博客文章"
git push
```

推送完成后，Cloudflare Pages 自动拉取 `v5` 分支并重新发布，无需执行额外部署命令。
