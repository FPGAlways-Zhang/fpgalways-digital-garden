---
title: FPGAlways 数字花园
tags:
  - 项目
  - Quartz
---

这是本站的项目记录页，用于整理数字花园从搭建到发布的全过程。

## 技术方案

- 内容编辑：Obsidian
- 网站生成：Quartz
- 代码托管：GitHub
- 免费部署：Cloudflare Pages

## 项目信息

- 本地项目目录：`F:\QuartzObsidianBlog`
- Obsidian Vault：`F:\QuartzObsidianBlog\content`
- GitHub 分支：`v5`
- GitHub 仓库：[FPGAlways-Zhang/fpgalways-digital-garden](https://github.com/FPGAlways-Zhang/fpgalways-digital-garden)
- Cloudflare Pages 项目：`fpgalways-digital-garden`
- 博客地址：[FPGAlways的数字花园](https://203605.xyz/)
- WWW 地址：`https://www.203605.xyz/`，自动 301 跳转到主域名
- Cloudflare 默认地址：`https://fpgalways-digital-garden.pages.dev/`

## 首次配置流程

### 1. 安装依赖和 Quartz 插件

进入项目目录：

```powershell
cd F:\QuartzObsidianBlog
npm.cmd install
npx.cmd quartz plugin install
```

Quartz 5 的插件会安装到项目中的 `.quartz` 目录。如果构建时出现 `Could not resolve "../../.quartz/plugins"`，通常是插件还没有安装，重新运行插件安装命令即可。

### 2. 配置博客

主要配置文件是 `quartz.config.yaml`，本站使用的关键设置包括：

- 网站名称：`FPGAlways的数字花园`
- 界面语言：`zh-CN`
- 网站地址：`203605.xyz`
- 左侧区域：搜索、深色模式和文件浏览器
- 右侧区域：关系图谱、目录和反向链接

文章和分类都放在 `content` 目录中。当前主要分为：

- 技术与 AI
- 生活随笔
- 读书笔记
- 项目与作品

### 3. 本地预览

```powershell
cd F:\QuartzObsidianBlog
npx.cmd quartz build --serve
```

然后访问：

```text
http://localhost:8080/
```

本地预览不要添加 `--baseDir`。正式站点地址由 `baseUrl` 和部署环境处理，本地直接从根路径预览更稳定。

### 4. 关联 GitHub 仓库

GitHub CLI 登录命令：

```powershell
F:\Tools\GitHubCLI\bin\gh.exe auth login --hostname github.com --git-protocol https --web --clipboard --skip-ssh-key
```

检查登录状态：

```powershell
F:\Tools\GitHubCLI\bin\gh.exe auth status
```

本地 Git 仓库保留两个远程地址：

```text
origin    https://github.com/FPGAlways-Zhang/fpgalways-digital-garden.git
upstream  https://github.com/jackyzha0/quartz.git
```

- `origin` 用于上传自己的博客。
- `upstream` 用于以后获取 Quartz 官方更新。

### 5. 配置 Cloudflare Pages 自动部署

Cloudflare Pages 直接连接 GitHub 仓库，不使用 GitHub Actions 上传构建产物。项目配置如下：

- GitHub 仓库：`FPGAlways-Zhang/fpgalways-digital-garden`
- 生产分支：`v5`
- 根目录：仓库根目录
- 构建命令：`npm ci && npx quartz plugin install && npx quartz build`
- 输出目录：`public`
- 环境变量：`NODE_VERSION=24`

每次向 `v5` 分支执行 `git push` 后，Cloudflare Pages 会自动拉取最新代码、安装 Quartz 插件、构建网站并发布。

GitHub 只负责保存源码和版本记录，GitHub Pages 已关闭，正式网站由 Cloudflare Pages 提供。

### 6. 绑定自定义域名

在 Cloudflare Pages 项目的“自定义域”页面，依次添加：

```text
203605.xyz
www.203605.xyz
```

Cloudflare 会自动创建并代理下面的 DNS 记录，同时自动申请 SSL 证书：

```text
CNAME  @    fpgalways-digital-garden.pages.dev
CNAME  www  fpgalways-digital-garden.pages.dev
```

等待两个域名都显示“活动”和“SSL 已启用”后，即可通过 HTTPS 访问。Quartz 配置文件中的正式站点地址应保持为：

```yaml
configuration:
  baseUrl: 203605.xyz
```

为了避免根域名和 WWW 域名产生两份重复内容，在 `203605.xyz` 的 Cloudflare 控制台中进入“规则 → 重定向规则”，使用“从 WWW 重定向到根”模板：

- 请求 URL：`https://www.*`
- 目标 URL：`https://${1}`
- 状态代码：`301 - Permanent Redirect`
- 开启“保留查询字符串”

部署规则后，路径和查询参数都会保留，例如：

```text
https://www.203605.xyz/项目与作品/?from=www
→ https://203605.xyz/项目与作品/?from=www
```

日常分享和站内配置统一使用主域名 `https://203605.xyz/`，`pages.dev` 地址只作为 Cloudflare 提供的备用访问地址。

## 后续更新博客

### 1. 开始前同步代码

如果可能在其他电脑或 GitHub 网页上修改过内容，写作前先同步：

```powershell
cd F:\QuartzObsidianBlog
git pull --rebase origin v5
```

### 2. 写文章

使用 Obsidian 打开下面的 Vault：

```text
F:\QuartzObsidianBlog\content
```

在对应分类文件夹中创建或修改 Markdown 笔记。需要在网站左侧显示的文件，应放在 `content` 内，并避免放入 `private` 或 `templates` 目录。

### 3. 本地检查

发布前启动本地预览：

```powershell
cd F:\QuartzObsidianBlog
npx.cmd quartz build --serve
```

确认首页、左侧文件列表、文章链接和中文路径都能正常打开后，在运行预览的终端中按 `Ctrl+C` 停止服务。

如果只想检查能否成功构建，不需要打开本地网站，可以运行：

```powershell
npx.cmd quartz build
```

### 4. 提交文章

```powershell
cd F:\QuartzObsidianBlog
git status -sb
git add content
git commit -m "更新博客文章"
git push origin v5
```

提交说明可以写得更具体，例如：

```powershell
git commit -m "新增 Quartz 博客搭建记录"
```

如果同时修改了 Quartz 配置，可以这样暂存：

```powershell
git add content quartz.config.yaml
```

提交前可运行 `git diff --check`，检查是否存在空白符等格式问题。如果 Git 提示没有可提交的修改，就不需要执行 `git commit` 和 `git push`。

### 5. 查看部署结果

推送后可以在 Cloudflare 控制台的 Workers & Pages 项目中查看部署进度。最新生产部署显示成功后，刷新博客地址即可看到更新：

```text
https://203605.xyz/
```

建议同时检查：

- 首页是否正常打开
- 新文章和中文路径是否返回正常页面
- 左侧文件浏览器和搜索是否可用
- 深色模式是否能切换
- `https://www.203605.xyz/` 是否跳转到主域名

### 日常更新命令速查

```powershell
cd F:\QuartzObsidianBlog
git pull --rebase origin v5

# 在 Obsidian 中完成文章编辑后
npx.cmd quartz build
git status -sb
git add content
git commit -m "更新博客文章"
git push origin v5
```

如果修改了博客配置，则将暂存命令改为：

```powershell
git add content quartz.config.yaml
```

## 常见检查

查看当前修改：

```powershell
git status -sb
```

查看远程仓库：

```powershell
git remote -v
```

只执行一次正式构建，不启动本地服务器：

```powershell
npx.cmd quartz build
```

不要提交 `node_modules`、`public`、`.quartz` 或 `.npm-cache`，这些目录已经通过 `.gitignore` 排除。

更详细的搭建过程记录在 [[Quartz博客搭建记录]]。

返回分类：[[项目与作品]]
