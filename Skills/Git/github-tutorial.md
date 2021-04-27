# Github 教程

GitHub 是通过 Git 进行版本控制的软件源代码托管服务平台。

该教程主要针对：**Github 相关设置** 与 **远程仓库管理** 等学习。

目录：

- [Github 教程](#github-教程)
  - [配置 Git](#配置-git)
  - [多人协作](#多人协作)
  - [参考链接](#参考链接)

## 配置 Git

**Step 1.** 本地创建 SSH Key

```powershell
$ ssh-keygen -t rsa -C "your_email@youremail.com"
```

your_email@youremail.com 为 Github 上绑定的邮箱，之后会要求输入保存路径与密码，在这使用默认设置（直接回车）即可。成功生成后，会在 `~/` 目录（用户目录）下生成 `.ssh` 文件夹，其中 `id_rsa.pub` 中的内容，即为 key。

**Step 2.** Github 添加 SSH Key

1. 进入 Github，选择 Settings -> SSH and GPG keys -> New SSH key;
2. 输入自定义 Title；
3. 将本地生成的 `id_rsa.pub` 内容（Linux: cat ~/.ssh/id_rsa.pub）复制到 Key 中，并完成添加。

![添加 SSH Key](https://i.loli.net/2021/02/14/hYLWAZydQDnRoKt.png)

**Step 3.** 验证 SSH Key

在 git bash 中输入：

```powershell
$ ssh -T git@github.com
```

如果是第一次登录会提示是否 continue，输入 yes 或 第二次登录就会收到提示：`You've successfully authenticated, but GitHub does not provide shell access` 。这就表示已成功连上 Github。

**Step 4.** 设置用户名/邮箱

Github 每次 commit 都会记录提交用户名与邮箱，并作为贡献依据。

```git
# 设置用户名/邮箱
$ git config --global user.name "your name"
$ git config --global user.email "your_email@youremail.com"

# 检查用户名/邮箱
$ git config --get user.name
$ git config --get user.email
```

**Step 5.** 添加远程仓库

在 Github web 端新建仓库，并记录 SSH 地址 `git@github.com:yourName/yourRepo.git`。

命令行添加远程仓库：

```git
$ git remote add origin git@github.com:yourName/yourRepo.git
```

yourname 和 yourRepo 表示 Github 用户名 和 仓库名，origin 表示远程仓库别名，添加完成之后，在本地 config 文件中，会新增 `[remote "origin"]` 内容，用于记录之前添加的内容。也可以通过修改该文件，重新配置远程地址。

**Step 6.** 推送至远程仓库

```git
$ git push origin main
```

origin 代表远程分支，main 代表本地分支（可以将其替换为任意想要推送的本地分支）。

## 多人协作

TODO 提交 PR [参考内容](https://juejin.cn/post/6844903821521469448)

## 参考链接

* [Github 简明教程](https://www.runoob.com/w3cnote/git-guide.html)
