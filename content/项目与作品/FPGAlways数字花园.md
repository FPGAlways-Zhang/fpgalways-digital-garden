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

## 后续更新博客

### 1. 写文章

使用 Obsidian 打开下面的 Vault：

```text
F:\QuartzObsidianBlog\content
```

在对应分类文件夹中创建或修改 Markdown 笔记。需要在网站左侧显示的文件，应放在 `content` 内，并避免放入 `private` 或 `templates` 目录。

### 2. 本地检查

发布前启动本地预览：

```powershell
cd F:\QuartzObsidianBlog
npx.cmd quartz build --serve
```

确认首页、左侧文件列表、文章链接和中文路径都能正常打开后，在运行预览的终端中按 `Ctrl+C` 停止服务。

### 3. 提交文章

```powershell
cd F:\QuartzObsidianBlog
git status
git add content
git commit -m "更新博客文章"
git push
```

提交说明可以写得更具体，例如：

```powershell
git commit -m "新增 Quartz 博客搭建记录"
```

如果同时修改了 Quartz 配置，可以这样暂存：

```powershell
git add content quartz.config.yaml
```

### 4. 查看部署结果

推送后可以在 Cloudflare 控制台的 Workers & Pages 项目中查看部署进度。最新生产部署显示成功后，刷新博客地址即可看到更新：

```text
https://203605.xyz/
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

如果在其他电脑或 GitHub 网页上修改过仓库，开始写作前先同步：

```powershell
git pull --rebase
```

不要提交 `node_modules`、`public`、`.quartz` 或 `.npm-cache`，这些目录已经通过 `.gitignore` 排除。

更详细的搭建过程记录在 [[Quartz博客搭建记录]]。

返回分类：[[项目与作品]]
