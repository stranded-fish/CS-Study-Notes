# Git 快速入门

该教程主要针对：Git 常用操作 学习。

目录：

- [Git 快速入门](#git-快速入门)
  - [新建仓库](#新建仓库)
  - [配置仓库](#配置仓库)
  - [操作文件](#操作文件)
  - [提交代码](#提交代码)
  - [分支管理](#分支管理)
  - [标签设置](#标签设置)
  - [查看信息](#查看信息)
    - [查看状态](#查看状态)
    - [查看历史](#查看历史)
    - [查看差异](#查看差异)
  - [远程同步](#远程同步)
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

# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track [branch] [remote-branch]

# 建立追踪关系，在现有分支与指定的远程分支之间
$ git branch --set-upstream [branch] [remote-branch]

# 合并指定分支到当前分支
$ git merge [branch]

# 选择一个 commit，合并进当前分支
$ git cherry-pick [commit]

# 删除分支
$ git branch -d [branch-name]

# 删除远程分支
$ git push origin --delete [branch-name]
$ git branch -dr [remote/branch]
```

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

`git fetch` 和 `git pull` 区别：

* `git fetch` 只会将本地库所关联的远程库的 commit id 更新至最新；
* `git pull` 会将本地库更新至远程库的最新状态，由于本地库进行了更新，`HEAD` 也会相应的指向最新的 commit id。

从结果来看，git pull 类似于 git fetch + git merge。出于安全考虑，**应尽量使用 git fetch + git merge。**

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
