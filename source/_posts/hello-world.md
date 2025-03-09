---
title: 配置 GitHub Action 发布网站
tags: [blog，github]
date: 2025-03-09
permalink: post-20250309.html
---

从昨天晚上到今天上午，才把这个网站给掰扯明白，更换了主题，并且由原来的git提交静态文件，改成了 Actions Workflow 的方式部署。

### 更换主题

原来用的 tranquilty 主题，今天换了一个更简洁的主题
[minimalism](https://minimalism.codeover.cn/docs/introduction)

### Github Action Workflow
我的理解这个东西就是 jekenis 的部署应用。

#### 过程
期间各种失败，网络问题、yml文件代码错误、文件问题等等，能遇到的全遇到了。

1. 网络问题

   不多说了，墙内网络，提交代码错误，但是很奇怪，通过 vscode 自带的 git 管理工具就可以，这个也算有解决方式。


2. 文件安全问题
![20250309012032.png](https://s2.loli.net/2025/03/09/i3MPKDIrta8CTgO.png)

> 解决方案 是点击错误信息中的链接，在页面中选择运行即可。

3. workflow代码配置错误
一开始是缺少deploy-pages的步骤，导致发布了十几次，但是访问页面始终无变化，通过 deepseek一步步的问才最终解决。

然后是报下面这个提示错误，最后才知道，github 默认部署的jekyll应用，需要声明改工程不是 jekyll，通过增加 touch public/.nojekyll 才解决。

``` bash
Error: Artifact could not be deployed. Please ensure the content does not contain any hard links, symlinks and total size is less than 10GB.
```

#### workflow文件
``` yml
name: Deploy Hexo to GitHub Pages
# 触发条件：当代码推送到主分支（默认 main）时自动运行
on:
  push:
    branches:
      - main
jobs:
  build:
    runs-on: ubuntu-latest  # 使用最新版 Ubuntu 系统
    steps:
      # 步骤 1：拉取仓库代码
      - name: Checkout repository
        uses: actions/checkout@v3
        with:
          submodules: true  # 如果使用 Git 子模块（如主题），需启用

      # 步骤 2：安装 Node.js 环境
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 20  # 指定 Node.js 版本（与本地一致）

      # 步骤 3：缓存 node_modules 加速后续构建
      - name: Cache node modules
        uses: actions/cache@v3
        id: cache-node-modules
        with:
          path: node_modules
          key: ${{ runner.os }}-node-${{ hashFiles('package-lock.json') }}

      # 步骤 4：安装 Hexo 及依赖
      - name: Install dependencies
        run: |
          npm install -g hexo-cli  # 全局安装 Hexo CLI
          npm install  # 安装项目依赖

      # 步骤 5：生成静态文件（public 目录）
      - name: Generate static files
        run: |
          hexo clean  # 清理旧文件
          hexo generate  # 生成静态文件
          touch public/.nojekyll   # 在构建后执行此命令

      # 步骤 7：部署到 GitHub Pages（gh-pages 分支）
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3  # 使用第三方部署 Action
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}  # 自动注入的 Token，无需手动配置
          publish_dir: ./public  # 静态文件目录
          publish_branch: gh-pages  # 目标分支（GitHub Pages 默认分支）
          commit_message: 'Deploy via GitHub Actions'  # 提交信息

      # 步骤 8：上传 GitHub Pages artifact
      - name: Upload GitHub Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          name: github-pages
          path: ./public

  deploy:
    needs: build
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      # 步骤 9：启动 GitHub Pages 部署
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
        with:
          artifact_name: github-pages
```