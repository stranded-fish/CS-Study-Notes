# Github Actions 自动化部署 Hugo 网站至 ECS

Github Actions 是 Github 推出的持续集成（continuous integration，CI）和持续交付（continuous delivery，CD）服务，利用该服务可以实现项目的自动化构建、测试和部署。更多可参见：[GitHub Actions 文档](https://docs.github.com/cn/actions/learn-github-actions/understanding-github-actions)。

本文章主要介绍如何使用 Github Actions 自动化部署 Hugo 网站至 ECS（云服务器），以达到最简化 Hugo 部署 workflow，专注于写作本身的目的。

* 原始 workflow：本地编辑文章 -> build & test -> 连接服务器 -> 将文件上传至服务器指定位置。
* 目标 workflow：本地编辑文章 -> git 推送 至 Github（实现保存网站资源的同时，自动部署到指定云服务器）。

目录：

- [Github Actions 自动化部署 Hugo 网站至 ECS](#github-actions-自动化部署-hugo-网站至-ecs)
  - [Github 操作](#github-操作)
  - [本地操作](#本地操作)
  - [参考链接](#参考链接)

## Github 操作

**Step 1.** 新建 Github 库

新建一个 Github 库，用于保存网站资源，以避免本地设备遗失或损坏所造成数据丢失。同时也用于后续自动化部署。

**Step 2.** 创建 Actions secrets

创建 Actions secrets 来保存 Actions 访问云服务器所需要用到的一些敏感信息。

![创建 Actions secrets](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/202204292127125.png)

根据不同的云服务器访问方式，需要设置不同的 secrets，本例使用 `user/password` 方式访问云服务器，故需设置如下 secrets：

![创建的 secrets](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/202204292320850.png)

* `HOST`：云服务器域名 或 IP 地址
* `SSH_USERNAME`：登录用户
* `SSH_PASSWORD`：用户密码
* `SSH_PORT`：服务端口号

## 本地操作

**Step 1.** 新增 Hugo 网站资源

可以执行 `hugo new site your-site` 命令新建 hugo 网站，或是将自己已经建好的网站资源作为 git 库。

**Step 2.** 初始化 git 库并关联 Github 新建库

```bash
# 初始化 git 库，并添加网站资源
git init
git add .
git commit . -m "Initial commit"

# 关联 Github 远程库
git remote add origin git@github.com:yourID/yourRepo.git
git branch -M main
git push -u origin main
```

**Step 3.** 新增 workflow

本地目录新建 `/.github/workflows/main.yml`，内容如下：

```yml
name: github pages

on:
  push:
    branches:
      - main  # push 到 main 分支时触发 jobs

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # 更新子模块（hugo themes）
          fetch-depth: 0

      # 配置 Hugo 环境
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: latest
          extended: true

      # 构建 Hugo 静态页面
      - name: Build
        run: hugo --minify

      # 将静态页面部署至 ECS
      - name: Deploy to ECS
        uses: garygrossgarten/github-action-scp@release # 使用 scp 工具将静态页面远程复制到 ECS
        with:
          local: public                                 # 待复制的本地文件路径
          remote: /web/hugo/public                      # ECS 路径
          host: ${{ secrets.HOST }}                     # ECS 域名 或 IP 地址
          username: ${{ secrets.SSH_USERNAME }}         # 登录用户名
          password: ${{ secrets.SSH_PASSWORD }}         # 登录密码
          port: ${{ secrets.SSH_PORT }}                 # scp 端口号（默认 22）
```

该 workflow 使用 `user/password` 作为验证登录 ECS，并通过 `scp` 命令将 Github 虚拟机上构建的静态页面，远程复制到 ECS 完成部署。

**Step 4.** 推送至 Github 远程库

首先提交本地新增的 `main.yml`，然后执行 `push` 命令将当前修改推送至远程库：

```bash
git push origin main
```

Github 将自动检测并运行 `.github/workflows/` 目录下的所有 `.yml` 文件，触发 Actions。

可登录 Github 网页端查看 Actions 具体执行情况：

![查看 Actions 执行情况](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/202204292303019.png)

![查看 Actions 执行细节](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/202204292304301.png)

Actions 执行成功后，可访问网站域名或 IP 地址，检查服务部署情况。

## 参考链接

* [GitHub Actions 文档](https://docs.github.com/cn/actions/learn-github-actions/understanding-github-actions)
* [GitHub Action SCP](https://github.com/garygrossgarten/github-action-scp)
