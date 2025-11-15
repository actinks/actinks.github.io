---
title: "Hugo + GitHub Pages 搭建个人博客（Ubuntu）"
date: 2025-11-15T10:00:00+08:00
draft: false
---

## 创建 GitHub Pages 仓库

1. 登录 GitHub，创建新仓库（New repository）。
2. 仓库名称必须为：`<你的GitHub用户名>.github.io`（例如：`alice.github.io`）。
3. 选择 **Public**，**不要初始化 README**。
4. 创建完成后，仓库应为空。

## 安装 Hugo

> 请前往 [https://github.com/gohugoio/hugo/releases](https://github.com/gohugoio/hugo/releases) 查看最新版本号。以下以 `v0.152.2` 为例。

```bash
VERSION="0.152.2"
wget "https://github.com/gohugoio/hugo/releases/download/v${VERSION}/hugo_extended_${VERSION}_linux-amd64.deb"

# 安装
sudo dpkg -i "hugo_extended_${VERSION}_linux-amd64.deb"

# 验证
hugo version
# 应输出：hugo v0.152.2-... extended ...
```

## 创建 Hugo 站点

```bash
# 创建博客目录
mkdir -p ~/projects/blog && cd ~/projects/blog

# 初始化 Hugo 站点
hugo new site .

# 初始化 Git 并添加 PaperMod 主题（轻量、快速、开发者友好）
git init
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

## 配置 `hugo.toml`

```bash
vim hugo.toml
```

内容如下（请将 `yourusername` 和个人信息替换为你的实际信息）：

```toml
baseURL = "https://yourusername.github.io/"
languageCode = "en-us"
title = "My's Blog"
theme = "PaperMod"

[params]
  author = "Your Name"
  description = "A blog about ..."
```


## 写第一篇文章

```bash
hugo new posts/my-first-post.md
```

生成的文件内容类似：

```markdown
---
title: "My First Post"
date: 2025-11-15T10:00:00+08:00
draft: true
---

Hello, this is my blog powered by Hugo and GitHub Pages!
```

> 将 `draft: true` 改为 `draft: false` 才会发布！

## 本地预览

```bash
hugo server -D
```

- `-D` 表示包含草稿（drafts）。
- 打开浏览器访问：[http://localhost:1313](http://localhost:1313)
- 修改文章后自动热重载。

## 部署到 GitHub Pages

使用 `gh-pages` 分支 + GitHub Actions 自动部署，这是最干净、主流的方式（源码和生成文件分离）。

### 步骤：

1. **确保你刚创建的 `<username>.github.io` 仓库是空的。**
2. **将 Hugo 源码推送到 `main` 分支：**

```bash
# 在你的博客目录下
git add .
git commit -m "Initial commit with Hugo site"
git branch -M main
git remote add origin https://github.com/yourusername/yourusername.github.io.git
git push -u origin main
```

3. **配置 GitHub Actions 自动构建并部署到 `gh-pages` 分支**

在博客目录下创建 `.github/workflows/gh-pages.yml`：

```yaml
# .github/workflows/gh-pages.yml
name: github pages

on:
  push:
    branches:
      - main  # 当 main 分支有更新时触发

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pages: write
      id-token: write

    steps:
      - uses: actions/checkout@v4
        with:
          submodules: true  # 必须，因为主题是 submodule
          fetch-depth: 0    # 获取所有历史（可选）

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

4. **在 GitHub 仓库中启用 GitHub Pages**

- 进入你的 `<username>.github.io` 仓库。
- 点击 **Settings → Pages**。
- 在 **Build and deployment → Source** 中选择 **GitHub Actions**。
- 保存。

5. **推送代码触发部署**

```bash
git add .
git commit -m "Add GitHub Actions workflow"
git push
```

等待几分钟，Actions 成功运行后，访问：  
👉 [https://yourusername.github.io/](https://yourusername.github.io/)

## 验证是否成功

- 检查 GitHub Actions 是否显示绿色 ✅。
- 访问 `https://yourusername.github.io` 应看到博客首页。
- 文章列表应包含你写的 **My First Post**。

## 日常写作流程

**写新文章：**

```bash
hugo new posts/another-post.md
```

编辑内容，设置 `draft: false`。

**本地预览：**

```bash
hugo server -D
```

**满意后提交并推送：**

```bash
git add .
git commit -m "Add new post: ..."
git push
```

GitHub Actions 会自动部署，几分钟后更新上线！
