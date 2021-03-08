# Git 快速入门

该教程主要针对：**Git 常用操作** 学习。

目录：

- [Git 快速入门](#git-快速入门)
  - [新建仓库](#新建仓库)
  - [配置仓库](#配置仓库)
  - [忽略文件](#忽略文件)
  - [操作文件](#操作文件)
  - [提交代码](#提交代码)
  - [分支管理](#分支管理)
    - [基本操作](#基本操作)
    - [clone 远程仓库](#clone-远程仓库)
    - [推送](#推送)
    - [跟踪分支](#跟踪分支)
    - [拉取](#拉取)
    - [解决合并冲突](#解决合并冲突)
    - [删除远程分支](#删除远程分支)
  - [远程同步](#远程同步)
  - [标签设置](#标签设置)
  - [查看信息](#查看信息)
    - [查看状态](#查看状态)
    - [查看历史](#查看历史)
    - [查看差异](#查看差异)
  - [撤销操作](#撤销操作)
    - [撤销工作区文件修改](#撤销工作区文件修改)
    - [撤销暂存区文件修改](#撤销暂存区文件修改)
    - [撤销本地提交](#撤销本地提交)
    - [撤销远程仓库提交](#撤销远程仓库提交)
  - [配置别名](#配置别名)
  - [参考链接](#参考链接)

## 新建仓库

```git
# 在当前目录新建一个 Git 仓库
# 注意：只有当首次 commit 之后，才会真正建立分支
$ git init

# 新建一个名为 project-name 的目录，并将其初始化为 Git 仓库
$ git init [project-name]

# 下载一个项目及其整个代码历史
$ git clone [url]
```

## 配置仓库

Git 配置有 `system`（系统）、`global`（用户级别）和 `local` （当前仓库）三个级别，底层配置会优先覆盖高层配置，当底层没有配置时，会使用高层配置。

参数：

    --global              use global config file
    --system              use system config file
    --local               use repository config file
    --worktree            use per-worktree config file
    -f, --file <file>     use given config file
    --blob <blob-id>      read config from given blob object

默认为 `local`（当前仓库）级别。

```git
# 用户级别需要添加 --global 参数

# 显示所有配置信息
$ git config --list / -l

# 显示用户名/邮箱 
$ git config --get user.name
$ git config --get user.email

# 修改用户名/邮箱 
$ git config user.name "[new name]"
$ git config user.email "[new email address]"

# 添加变量
$ git config --add user.newKey newValue

# 删除变量
$ git config --unset user.newKey

# 编辑 Git 配置文件 
$ git config --edit / -e 

# 修改 git init 默认分支名
$ git config --add init.defaultBranch main
# [init]
#   defaultBranch = main
```

## 忽略文件

忽略文件原则：

* 忽略操作系统自动生成的文件，比如缩略图等。
* 忽略编译生成的中间文件、可执行文件等，也就是如果一个文件是通过另一个文件自动生成的，那自动生成的文件就没必要放进版本库，比如 Java 编译产生的 .class 文件。
* 忽略你自己的带有敏感信息的配置文件，比如存放口令的配置文件。

实现方法：

1. 在 Git 仓库工作区 根目录（即 `.git` 文件夹同级目录） 新建一个名为 `.gitignore` 的文件；
1. 将要忽略的文件填进去，Git 即可自动忽略这些文件。

注意：无需从头写 `.gitignore` 文件，GitHub 已经为我们准备了各种配置文件，只需要组合一下就可以使用了。所有配置文件可以直接在线浏览：[github/gitignore](https://github.com/github/gitignore)。

示例：

如果项目为 Java 项目，.java 文件编译后会生成 .class 文件，这些文件多数情况是不需要传到仓库中的，这时可以选择使用 Github 中的 [Java.gitignore](https://github.com/github/gitignore/blob/master/Java.gitignore) 模板，将该模板内容复制到本地 .gitignore 文件中：

```gitignore
# Compiled class file
*.class

# Log file
*.log

# BlueJ files
*.ctxt

# Mobile Tools for Java (J2ME)
.mtj.tmp/

# Package Files #
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

# virtual machine crash logs, see http://www.java.com/en/download/help/error_hotspot.xml
hs_err_pid*
```

保存后，通过 `git status` 命令即可检查是否已经忽略指定文件。

若有些文件已经被忽略了，但仍想强制添加到 git 中去，需要加上 `-f` 参数：

```git
git add -f HelloWorld.class 
```

## 操作文件

Git 本地数据管理，分为三个区：

* 工作区（Working Directory）：可以直接编辑的地方。
* 暂存区（Stage/Index）：数据暂时存放的区域。
* 版本库（commit History）：存放已经提交的数据。

工作区的文件 `git add` 后到暂存区，暂存区的文件 `git commit` 后到版本库。

```git
# 添加指定 文件、目录、当前目录所有文件 到 暂存区
$ git add [file1] [dir] .  ...

# 删除 工作区 文件（仅删除了 工作区 文件，若想进一步删除 版本库 文件，还需 add 至 暂存区、最后 commit 提交至 版本库）
$ rm [file1] [file2] ...

# 删除 工作区 文件，并将这次删除放入 暂存区
# 注意：待删除文件需要与当前版本库文件相同，即未被修改
$ git rm [file1] [file2] ...

# 删除 工作区 和 暂存区 文件，并将这次删除放入 暂存区
$ git rm -f [file1] [file2] ...

# 删除 暂存区 文件，但保留 工作区 文件，并将这次删除放入 暂存区（同时会将文件取消跟踪）
# 可通过该命令将文件取消跟踪，并配置 .gitignore 忽略该文件
$ git rm --cached [file]

# 修改 文件名，并将改名操作放入 暂存区
$ git mv [file-original] [file-renamed]
```

## 提交代码

```git
# 提交 暂存区 所有文件到 版本库
$ git commit -m [message]

# 提交 暂存区 指定文件到 版本库
$ git commit [file1] [file2] ... -m [message]

# 提交 工作区 自上次 commit 之后的变化，直接到 版本库
$ git commit -a

# 提交时显示所有 diff 信息
$ git commit -v

# 使用一次新的 commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次 commit 的提交信息
$ git commit --amend -m [message]
```

## 分支管理

由于自 2020 年 10 月 1 日起，Gihub 中的默认分支 master 将更改为 main，本文使用 Github 作为远程仓库示例，故出现的 master 直接视为 main 即可。

### 基本操作

```git
# 列出所有本地分支\远程分支\本地和远程分支
$ git branch [-r] [-a]

# 新建一个分支，但依然停留在当前分支
$ git branch [branch-name]

# 新建一个分支，指向指定commit
$ git branch [branch-name] [commit]

# 新建一个分支，并切换到该分支
$ git switch -c [branch-name]

# 切换到指定分支，并更新工作区
$ git switch [branch-name]

# 合并指定分支到当前分支
$ git merge [branch]

# 选择一个 commit，合并进当前分支
$ git cherry-pick [commit]

# 删除分支
$ git branch -d [branch-name]
```

### clone 远程仓库

从远程 Git 服务器克隆代码 `git clone [url]`：

* Git 的 `clone` 命令会自动将该远程 Git 服务器仓库命名为 `origin`，并拉取它的所有数据；
* 同时创建一个指向它（远程仓库）的 `master` 分支的指针，并且在本地将其命名为 `origin/master`；
* 最后 Git 也会创建一个与 `origin/master` 分支指向同一个地方的本地 `master` 分支，用户可通过本地 `master` 分支进行相关工作。

**本地分支：`master` 跟踪 远程分支 `origin/master`。**

![克隆之后的服务器与本地仓库](https://i.loli.net/2021/02/15/rCe1VdqI24hlDL6.png)

**本地 `master` 分支 与 远程分支 `origin/master` 分支互不影响。** 如果在本地 `master` 分支做了一些工作，在同一时间有其他人推送到 Git 服务器并更新了 `master` 分支。即，提交历史已走向不同的方向。即便这样，只要自身不与 `origin` 服务器连接（并拉取数据），本地 `origin/master` 指针就不会移动。

![本地与远程的工作可以交叉](https://i.loli.net/2021/02/15/wUO5uFt8WqcaLYX.png)

**通过 `git fetch <remote>` 命令与给定的远程仓库同步数据。** 该命令会查找 “origin” 对应的服务器，并从中抓取本地没有的数据，并更新本地数据库，移动 `origin/master` 指针到更新后的位置。

![git fetch 更新你的远程跟踪分支](https://i.loli.net/2021/02/15/tRHIPosSD1MqVyl.png)

### 推送

示例：

```git
# git push <remote> <branch>
$ git push origin serverfix
```

该命令简化了部分工作。Git 自动将 `serverfix` 分支名字展开为 `refs/heads/serverfix:refs/heads/serverfix`，即，推送本地的 `serverfix` 分支来更新远程仓库上的 `serverfix` 分支。

可以通过这种格式来指定待推送的本地分支与远程分支。如执行：

```git
$ git push origin serverfix:awesomebranch
```

表示将本地的 `serverfix` 分支推送到远程仓库上的 `awesomebranch` 分支。

下一次其他协作者从服务器上抓取数据时，他们会在本地生成一个远程分支 `origin/serverfix`，指向服务器的 `serverfix` 分支的引用。

```git
$ git fetch origin
remote: Counting objects: 7, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 3 (delta 0)
Unpacking objects: 100% (3/3), done.
From https://github.com/schacon/simplegit
 * [new branch]      serverfix    -> origin/serverfix
```

注意：当抓取到新的远程跟踪分支时，本地不会自动生成一份可编辑的副本（拷贝）。 即，这种情况下，不会有一个新的 `serverfix` 分支——只有一个不可以修改的 `origin/serverfix` 指针。

若要生成可编辑的副本：

* 可以运行 `git merge origin/serverfix` 将这些工作合并到当前所在的分支。
* 如果想要在自己的 `serverfix` 分支上工作，可以将其建立在跟踪 远程分支 之上：

  ```git
  $ git checkout -b serverfix origin/serverfix
  Branch serverfix set up to track remote branch serverfix from origin.
  Switched to a new branch 'serverfix'
  ```

  该命令会创建一个用于工作的本地分支 `serverfix`，起点位于远程分支 `origin/serverfix`，并会跟踪该远程分支。

### 跟踪分支

从一个远程跟踪分支检出一个本地分支会自动创建所谓的“跟踪分支”（它跟踪的分支叫做“上游分支”）。 跟踪分支是与远程分支有直接关系的本地分支。 如果在一个跟踪分支上输入 git pull，Git 能自动地识别去哪个服务器上抓取、合并到哪个分支。当克隆一个仓库时，它通常会自动地创建一个跟踪 origin/master 的 master 分支。

创建跟踪分支：

```git
$ git checkout -b <branch> <remote>/<branch>

# --track 快捷模式
$ git checkout --track origin/serverfix
# Branch serverfix set up to track remote branch serverfix from origin.
# Switched to a new branch 'serverfix'

# 设置别名，将本地分支与远程分支设置为不同的名字
$ git checkout -b sf origin/serverfix
```

设置已有的本地分支跟踪远程分支：

```git
$ git branch -u origin/serverfix
# Branch serverfix set up to track remote branch serverfix from origin.
```

查看本地分支关联（跟踪）的远程分支：

```git
$ git branch -vv
```

上游快捷方式：

当设置好跟踪分支后，可以通过简写 `@{upstream}` 或 `@{u}` 来引用它的上游分支。 所以在 `master` 分支时并且它正在跟踪 `origin/master` 时，如果愿意的话可以使用 `git merge @{u}` 来取代 `git merge origin/master`。

### 拉取

```git
# 更新与远程仓库关联的版本号 commit id，但不修改本地内容（需要之后手动 merge）
$ git fetch [remote]

# 直接将本地代码与 commit id 更新为远程仓库最新版本
$ git pull [remote] [branch]
```

`git fetch` 和 `git pull` 区别：

* `git fetch` 从服务器上抓取本地没有的数据，但不会修改工作目录中的内容，只会修改远程跟踪分支，如：`origin/master`。
* `git pull` 会查找当前分支所跟踪的远程仓库与分支，从服务器上抓取数据，并尝试合并入当前分支。

从结果来看，`git pull` 类似于 `git fetch` + `git merge`。出于安全考虑，**应尽量使用 `git fetch` + `git merge`。**

### 解决合并冲突

当待合并的两个分支都分别有新的提交时，如下图所示：

![冲突分支](https://i.loli.net/2021/02/15/iuJQ2YhpIDXoSTk.png)

Git 无法快速合并，只能试图将各自修改合并起来，但这种合并就可能会有冲突：

```git
$ git merge feature1
Auto-merging readme.txt
CONFLICT (content): Merge conflict in readme.txt
Automatic merge failed; fix conflicts and then commit the result.
```

当发生冲突时，需要手动合并发生冲突的文件，同时可以通过 `git status` 显示冲突文件：

```git
$ git status
On branch master
Your branch is ahead of 'origin/master' by 2 commits.
  (use "git push" to publish your local commits)

You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)

  both modified:   readme.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

Git 通过 `<<<<<<<`，`=======`，`>>>>>>>` 标记出不同分支的内容。

```git
Git is a distributed version control system.
Git is free software distributed under the GPL.
Git has a mutable index called stage.
Git tracks changes of files.
<<<<<<< HEAD
Creating a new branch is quick & simple.
=======
Creating a new branch is quick AND simple.
>>>>>>> feature1
```

解决冲突后，再提交：

```git
$ git add readme.txt
$ git commit -m "conflict fixed"
[master cf810e4] conflict fixed
```

最终分支状态变为：

![合并分支](https://i.loli.net/2021/02/15/R75DSAduGvhp69P.png)

### 删除远程分支

```git
$ git push origin --delete serverfix
```

该命令只是从服务器上移除这个指针。Git 服务器通常会保留数据一段时间直到垃圾回收运行，所以如果不小心删除掉了，通常可以恢复。

## 远程同步

```git
# 增加一个新的远程仓库，并命名
$ git remote add [shortname] [url]

# 显示所有远程仓库
$ git remote -v

# 显示指定远程仓库的信息
$ git remote show [remote]

# 更新与远程仓库关联的版本号 commit id，但不修改本地内容（需要之后手动 merge）
$ git fetch [remote]

# 直接将本地代码与 commit id 更新为远程仓库最新版本
$ git pull [remote] [branch]

# 上传本地指定分支到远程仓库
$ git push [remote] [branch]

# 强行推送当前分支到远程仓库，即使有冲突
$ git push [remote] --force

# 推送所有分支到远程仓库
$ git push [remote] --all
```

![图解](https://i.loli.net/2021/02/09/uCTdrBA3w6l12qD.png)

## 标签设置

```git
# 列出所有 tag
$ git tag

# 新建一个 tag 在当前 commit
$ git tag [tag]

# 新建一个 tag 在指定 commit
$ git tag [tag] [commit]

# 删除本地 tag
$ git tag -d [tag]

# 删除远程 tag
$ git push origin :refs/tags/[tagName]

# 查看 tag 信息
$ git show [tag]

# 提交指定 tag
$ git push [remote] [tag]

# 提交所有 tag
$ git push [remote] --tags

# 新建一个分支，指向某个 tag
$ git checkout -b [branch] [tag]
```

## 查看信息

### 查看状态

```git
# 显示当前 工作区 状态
$ git status
```

### 查看历史

```git
# 显示当前分支 commit 历史
$ git log

# 显示所有分支的所有操作记录（包括 commit 和 reset 的操作）
# 可以通过该命名查看当前 commit id 之前的操作
# 如：通过 reset 命名回退版本后，git log 将只会显示到当前命名版本号，之后的版本号只能通过 git reflog 命令显示
$ git reflog

# 显示过去 5 次提交
$ git log -5 --pretty --oneline

# 显示当前分支 commit 历史，以及每次 commit 发生变更的信息
$ git log --stat

# 通过关键词，搜索提交历史
$ git log -S [keyword]

# 显示某个 commit 之后的所有变动，每个 commit 占据一行
$ git log [tag] HEAD --pretty=format:%s

# 显示某个 commit 之后的所有变动，其"提交说明"必须符合搜索条件
$ git log [tag] HEAD --grep feature

# 显示某个文件的版本历史，包括文件改名
$ git log --follow [file]
$ git whatchanged [file]

# 显示指定文件相关的每一次 diff
$ git log -p [file]

# 显示某次提交的元数据和内容变化
$ git show [commit]

# 显示某次提交发生变化的文件
$ git show --name-only [commit]

# 显示某次提交时，某个文件的内容
$ git show [commit]:[file]
```

### 查看差异

![状态差异查看](https://i.loli.net/2021/02/11/pzGtnFD2y36PZ4e.png)

```git
# 显示 工作区 和 暂存区 的差异
$ git diff [file]

# 显示 工作区 和 版本库 的差异
$ git diff [commit] [file]

# 显示 暂存区 和 版本库 的差异
$ git diff --staged [file]

# 显示 任意两个 版本库 版本（即任意两次提交的差异）
$ git diff [commit] [commit]

# 显示今天的修改内容
$ git diff --shortstat "@{0 day ago}"
```

`HEAD` 指代当前分支的最新提交，当使用 `git switch` 命令切换分支时，`HEAD` 重新指向新的分支。

* 通过 `git diff HEAD test.txt`，实现当前 工作区 文件与最新 commit 版本库 中文件的差异比较。
* `HEAD^` 表示 `HEAD` 分支上一分支（`HEAD^^` 同理）。

## 撤销操作

### 撤销工作区文件修改

若 工作区 的某个文件被修改了，但没有提交，可以通过 `git restore` 命令恢复本次修改之前的文件。

该命令会首先搜索 暂存区，若 暂存区 有该文件，则恢复该文件，否则恢复至上一次提交版本。

Changes not staged for commit:

```git
# 撤销 工作区 文件修改
$ git restore [file]
```

**注意：工作区的文件变化一旦被撤销，将无法恢复。**

### 撤销暂存区文件修改

Changes to be committed:

```git
$ git restore --staged [file]
```

注意：该命令不会将 暂存区 内容覆盖至 工作区。（即工作区 内容将会保存，暂存区 内容将会被丢弃）

### 撤销本地提交

```git
# 工作区 文件不会被修改，重置当前分支的 HEAD 为指定 commit（--mixed）
$ git reset [commit]
```

参数：

* --soft：工作区 文件不会被修改，原 版本库 内容会被保存到 暂存区；
* --mixed：工作区 文件不会被修改，暂存区 与 版本库 都将被修改为指定 commit id 版本内容；
* --hard：工作区 文件将会被修改，工作区、暂存区 与 版本库 都将被修改为指定 commit id 版本内容。

示例：

连续执行 3 次 commit：

```git
d13f1fa HEAD@{5}: commit: add 3
69fdbf5 HEAD@{6}: commit: add 2
fd93bdd HEAD@{7}: commit: add 1
```

file.txt 内容：

```txt
1 // commit: add 1
2 // commit: add 2
3 // commit: add 3
4 // 工作区新增，未提交
```

当前 `HEAD` 指向 d13f1fa 执行命令 `git reset fd93bdd [参数]`：

参数              | 工作区 | 暂存区 | 现版本库 | 原版本库
------------------|--------|--------|----------|-----
--soft            | 1234   | 123    | 1        | 123
--mixed (default) | 1234   | 1      | 1        | 123
--hard            | 1      | 1      | 1        | 123

### 撤销远程仓库提交

```git
# step 1 - 本地回退到相应版本，并保留当前 工作区 修改
$ git reset --soft [commit id]

# step 2 - 推送当前分支到远程仓库
# 由于本地仓库版本号低于远程仓库版本号，为了覆盖掉远端的版本信息，使远端的仓库也回退到相应的版本，需要加上参数 --force
$ git push origin [branch-name] --force
```

## 配置别名

```git
$ git config --global alias.s status
$ git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
$ git config --global alias.last 'log -1'
```

## 参考链接

* [常用 Git 命令清单](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)
* [Git：移除文件----git rm 命令的使用](https://blog.csdn.net/qq_42780289/article/details/98353792)
* [详解 git pull 和 git fetch 的区别](https://blog.csdn.net/weixin_41975655/article/details/82887273?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.control)
* [git 不同阶段撤回](http://einverne.github.io/post/2017/12/git-reset.html)
* [配置别名](https://www.liaoxuefeng.com/wiki/896043488029600/898732837407424)
* [Git 撤销commit文件 和 回退push的文件](https://www.jianshu.com/p/491a14d414f6)
* [Git忽略文件.gitignore的使用](https://www.jianshu.com/p/a09a9b40ad20)
* [廖雪峰-解决冲突](https://www.liaoxuefeng.com/wiki/896043488029600/900004111093344)
* [Git 分支 - 远程分支](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E8%BF%9C%E7%A8%8B%E5%88%86%E6%94%AF)
