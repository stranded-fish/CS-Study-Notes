# Git 学习

* Git 是一个开源的 分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。
* Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。
* Git 与常用的版本控制工具 CVS, Subversion 等不同，它采用了分布式版本库的方式，不必服务器端软件支持。

该教程主要针对：**Git 配置**、**操作**、**远程仓库** 与 **分支管理** 等系统性的学习。

目录：

- [Git 学习](#git-学习)
  - [版本控制系统分类](#版本控制系统分类)
    - [本地版本控制系统](#本地版本控制系统)
    - [集中式版本控制系统](#集中式版本控制系统)
    - [分布式版本控制系统](#分布式版本控制系统)
  - [基本操作](#基本操作)
    - [获取仓库](#获取仓库)
      - [在已存在的目录中初始化仓库](#在已存在的目录中初始化仓库)
      - [克隆现有的仓库](#克隆现有的仓库)
    - [记录更新](#记录更新)
      - [查看文件状态](#查看文件状态)
      - [跟踪新文件](#跟踪新文件)
      - [暂存已修改文件](#暂存已修改文件)
      - [忽略文件](#忽略文件)
      - [提交更新](#提交更新)
      - [移除文件](#移除文件)
      - [移动文件](#移动文件)
    - [查看历史](#查看历史)
    - [撤销操作](#撤销操作)
      - [修改提交说明](#修改提交说明)
      - [取消暂存的文件](#取消暂存的文件)
      - [撤销对文件的修改](#撤销对文件的修改)
    - [远程仓库 todo](#远程仓库-todo)
      - [查看远程仓库](#查看远程仓库)
      - [添加远程仓库](#添加远程仓库)
      - [远程仓库中抓取与拉取](#远程仓库中抓取与拉取)
      - [推送到远程仓库](#推送到远程仓库)
      - [远程仓库的重命名与移除](#远程仓库的重命名与移除)
    - [设置标签](#设置标签)
      - [列出标签](#列出标签)
      - [创建标签](#创建标签)
    - [Git 别名 todo](#git-别名-todo)
  - [参考链接](#参考链接)

## 版本控制系统分类

版本控制系统（Version Control Systems, VCS）：一种记录一个或若干文件内容变化，以便将来查阅特定版本修订情况的系统

### 本地版本控制系统

大多采用简单的数据库来记录文件的历次更新差异

工作原理：在硬盘上保存补丁集（补丁是指文件修订前后的变化）；通过应用所有的补丁，可以重新计算出各个版本的文件内容

  <!-- ![Figure 1. 本地版本控制系统](https://i.loli.net/2020/10/23/dCBQAt2wk7jNTga.png) -->
  常见本地版本控制系统：RCS

### 集中式版本控制系统

Centralized Version Control Systems, CVCS

通过单一的集中管理的服务器，保存所有文件的修订版本，而协同工作的人们都通过客户端连到这台服务器，取出最新的文件或者提交更新
<!-- ![Figure 2. 集中式版本控制系统](https://i.loli.net/2020/10/23/gKz1HpPVRut9QWJ.png) -->

* 优点：
  * 文件夹级权限控制，权限控制粒度较小，并可轻易控制单个开发者权限
  * 仅管理单个CVCS，较管理各个客户端并维护本地数据库更容易
  * 对客户端配置要求较低，客户端无需存储所有代码

* 缺点：
  * 网络环境要求高，必须联网才能工作
  * 中央服务器发生单点故障，将使所有人都无法提交更新与协同工作
  * 中央服务器数据库所在磁盘损坏，且又无备份，将丢失所有数据（只剩下各自客户端机器上的快照）

常见集中式版本控制系统：CVS、SVN

### 分布式版本控制系统

Distributed Version Control System, DVCS

 客户端并不只提取最新版本的文件快照， 而是把代码仓库完整地镜像下来，包括完整的历史记录。 这么一来，任何一处协同工作用的服务器发生故障，事后都可以用任何一个镜像出来的本地仓库恢复。 因为每一次的克隆操作，实际上都是一次对代码仓库的完整备份
<!-- ![Figure 3. 分布式版本控制系统](https://i.loli.net/2020/10/23/bUKuC7wsE2WvHBo.png) -->

* 优点：
  * 版本库本地化，版本库的完整克隆，包括标签、分支、版本记录等，任何一处协同工作用的服务器发生故障，都可以用任何一个镜像的本地仓库进行恢复
  * 支持离线提交，适合跨地域协同开发
  * 分支切换快速高效，创建和销毁分支廉价

* 缺点：
  * 只能针对整个仓库创建分支，无法根据目录建立层次性的分支

## 基本操作

### 获取仓库

仓库又名版本库，repository，通常有两种方式获取 Git 项目仓库：

* 将尚未进行版本控制的本地目录转换为 Git 仓库
* 从其它服务器 克隆 一个已存在的 Git 仓库

#### 在已存在的目录中初始化仓库

1. 定位到待创建仓库目录
2. 通过 `git init` 命令将该目录变为Git可以管理的仓库

   ```git
   $ git init
   Initialized empty Git repository in D:/Temporary Files/GitTest/.git/
   ```

创建完成后，该目录下将新增一个.git文件夹，该文件夹含有初始化 Git 仓库中所有的必须文件，用于跟踪管理版本库（隐藏文件夹，可通过`ls -a`查看）

注：此时项目中的文件还未被追踪，需要后续通过 `git add` 命令来指定所需跟踪的文件

#### 克隆现有的仓库

使用 `git clone` 命令，实现对已经存在了的 Git 仓库的拷贝。

用法：`git clone <url>`

```git
$ git clone https://github.com/libgit2/libgit2
```

自定义本地仓库名字：

```git
$ git clone https://github.com/libgit2/libgit2 mylibgit
```

注：Git 不同于其他版本控制系统，Git 克隆的是该 Git 仓库服务器上几乎所有的数据，而不仅仅是工作所需文件

* 在指定目录初始化一个 `.git` 文件夹，从远程仓库拉去所有的数据并放入到该文件夹中
* 从 .git 中读取最新版本文件的拷贝

Git 支持多种数据传输协议。如 `https://` ， `git://` ， SSH 传输协议

### 记录更新

Git 目录下的文件，只有两种状态：**已跟踪** 或 **未跟踪**

* 已跟踪：指已经被纳入版本控制的文件，在上一次快照中有其记录，所处的状态有：
  * 未修改 Unmodified：提交过后，尚未修改
  * 已修改 Modified：自上次提交后，对文件做出了修改
  * 已暂存 Staged：可以在工作中，选择性的将修改后的文件放入到暂存区

![文件状态周期变化](https://i.loli.net/2020/11/13/SvuGiMEtO5wUpHx.png)

* 未跟踪：既未存在上次快照的记录中，也没被放入到暂存区

#### 查看文件状态

使用 `git status` 命令查看文件目前所处状态

```git
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
nothing to commit, working directory clean
```

状态简览 `git status -s` 或 `git status --short`：

```git
$ git status -s
 M README           // 已修改但未暂存
MM Rakefile         // 已修改且已暂存，之后又做了修改
A  lib/git.rb       // 新添加到暂存区
M  lib/simplegit.rb // 已修改且已暂存
?? LICENSE.txt      // 未跟踪的文件
```

输出中有两栏（1、2列），左栏表示暂存区的状态，右栏表示工作区的状态

#### 跟踪新文件

使用 `git add` 命令开始跟踪一个文件。该命令使用文件或目录的路径作为参数，如果参数是路径，则会递归地跟踪该目录下的所有文件

```git
$ git add README
```

此时运行 `git status` 命令，可得知目前 `README` 文件已被跟踪，并处于暂存状态：

```git
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)

    new file:   README
```

#### 暂存已修改文件

使用 `git add` 命令将已跟踪的文件，放到暂存区

注：`git add` 命令暂存的是当运行命令时的版本，而非工作目录中的当前版本。`git add` 之后再次作出修改后，需要重新执行命令暂存

#### 忽略文件

忽略特定文件，使其无需纳入 `Git` 的管理，且也不出现在未跟踪文件列表中。通常使用该方法忽略一些自动生成的日志或临时文件

* 在工程目录创建 `.gitignore` 文件
* 在其中添加自定义内容
* 将 `.gitignore` 文件提交到 `Git`

文件 `.gitignore` 的格式规范如下：

* 所有空行或者以 # 开头的行都会被 `Git` 忽略。
* 可以使用标准的 `glob` 模式匹配，它会递归地应用在整个工作区中。(标准的glob模式是指shell使用的简化了的正则表达式)
* 匹配模式可以以（/）开头防止递归。
* 匹配模式可以以（/）结尾指定目录。
* 要忽略指定模式以外的文件或目录，可以在模式前加上叹号（!）取反。

忽略文件的原则是：

* 忽略操作系统自动生成的文件，比如缩略图等
* 忽略编译生成的中间文件、可执行文件等，也就是如果一个文件是通过另一个文件自动生成的，那自动生成的文件就没必要放进版本库，比如Java编译产生的.class文件，.o或者.a文件
* 忽略你自己的带有敏感信息的配置文件，比如存放口令的配置文件

// 详细规则todo

#### 提交更新

```git
$ git commit
```

不加参数情况下，会打开默认文本编辑器，来输入提交说明。

另外，也可以通过添加参数 `-m` ，附带说明信息。

```git
$ git commit -m "Story 182: Fix benchmarks for speed"
[master 463dc4f] Story 182: Fix benchmarks for speed
 2 files changed, 2 insertions(+)
 create mode 100644 README
```

提交完成后，控制台会提示，当前是在哪个分支 `(master)` 提交的，本次提交的完整的 SHA-1 效验和是什么 `(463dc4f)` ，以及在本次提交中，有多少文件修订过，多少行添加和删改过。

跳过暂存区域

提交时，给 `git commit` 添加 `-a` 参数，Git 将自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过git add步骤：

```git
$ git commit -a -m 'added new benchmarks'
```

#### 移除文件

通过 git rm 命令移除文件：

```git
$ git rm PROJECTS.md
rm 'PROJECTS.md'
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    deleted:    PROJECTS.md
```

下一次提交时，该文件就不再纳入版本管理了。如果要删除之前修改过或已经放到暂存区的文件，则必须使用强制删除选项 -f（即 force 的首字母）。

另一种情况，如果想要把 Git 从仓库中删除 (亦即从暂存区域移除)，但仍然希望保留在当前工作目录中 (即不让 git 跟踪)，使用 `--cached` 参数：

```git
$ git rm --cached README
```

#### 移动文件

通过 `git mv` 命令实现移动文件与重命名。

```git
$ git mv file_from file_to
```

Git 将会记录到该次重命名操作。

```git
$ git mv README.md README
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README
```

### 查看历史

通过 `git log` 命令，查看历史。

```git
$ git log
commit e7584476deff64b1e6129134dd3cfa02888fd82c
Author: stranded-fish <zhouyulancq@foxmail.com>
Date:   Fri Nov 13 15:11:30 2020 +0800

    测试文件

commit e7198d6bc0aca48ccee5e8d64dd39c45c26402eb
Author: stranded-fish <zhouyulancq@foxmail.com>
Date:   Fri Nov 13 15:10:52 2020 +0800

    add ignore file
```

无参数情况，`git log` 会按照时间先后顺序列出所有的提交，最近的更新排在最上面。该命令会列出每个提交的 SHA-1效验和、作者的名字和电子邮箱地址、提交时间以及提交说明

`git log` 常用参数：

选项            | 说明
----------------|----------------------------------------------------------------------
-p              | 按补丁格式显示每个提交引入的差异。
--stat          | 显示每次提交的文件修改统计信息。
--shortstat     | 只显示 --stat 中最后的行数修改添加移除统计。
--name-only     | 仅在提交信息后显示已修改的文件清单。
--name-status   | 显示新增、修改、删除的文件清单。
--abbrev-commit | 仅显示 SHA-1 校验和所有 40 个字符中的前几个字符。
--relative-date | 使用较短的相对时间而不是完整格式显示日期（比如“2 weeks ago”）。
--graph         | 在日志旁以 ASCII 图形显示分支与合并历史。
--pretty        | 使用其他格式显示历史提交信息。可用的选项包括 oneline、short、full、fuller 和 format（用来定义自己的格式）。
--oneline       | --pretty=oneline --abbrev-commit 合用的简写。

限制 `git log` 输出的选项：

选项              | 说明
------------------|----------------------
-<n>              | 仅显示最近的 n 条提交。
--since, --after  | 仅显示指定时间之后的提交。
--until, --before | 仅显示指定时间之前的提交。
--author          | 仅显示作者匹配指定字符串的提交。
--committer       | 仅显示提交者匹配指定字符串的提交。
--grep            | 仅显示提交说明中包含指定字符串的提交。
-S                | 仅显示添加或删除内容匹配指定字符串的提交。

### 撤销操作

#### 修改提交说明

当提交完成后，若想修改提交信息，可通过带有 `--amend` 参数的提交命令来重新提交：

```git
$ git commit --amend
```

不加参数情况下，会打开默认文本编辑器，来输入提交说明。
另外，也可以通过添加参数 `-m` ，附带说明信息。

#### 取消暂存的文件

通过 `git reset HEAD <file>` 命令取消暂存。

```git
$ git reset HEAD CONTRIBUTING.md
Unstaged changes after reset:
M CONTRIBUTING.md
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md
```

注：git reset 命令，较为危险，特别是加上了 `--hard` 选项，若源文件没有修改则相对安全一些。

#### 撤销对文件的修改

通过 `git checkout -- <file>` 命令，来撤销对文件的修改，即将他还原成上次提交时的状态。

todo

### 远程仓库 todo

#### 查看远程仓库

通过 `git remote` 命令，列出远程服务器的缩写。

```git
$ git remote
origin
```

可以指定参数 `-v` ，显示需要读写远程仓库使用的 Git 保存的简写与其对应的 URL。

```git
$ git remote -v
origin  https://github.com/stranded-fish/Java-learning (fetch)
origin  https://github.com/stranded-fish/Java-learning (push)
cho45     https://github.com/cho45/grit (fetch)
cho45     https://github.com/cho45/grit (push)
```

通过 `git remote show <remote>` 查看指定远程仓库的详细信息。

```git
$ git remote show origin
* remote origin
  Fetch URL: https://github.com/stranded-fish/Java-learning
  Push  URL: https://github.com/stranded-fish/Java-learning
  HEAD branch: master
  Remote branch:
    master tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (up to date)
```

#### 添加远程仓库

eg 1. 通过 `git alone <url>` 命令自动添加远程仓库；
eg 2. 通过 `git remote add <shortname> <url>` 添加远程仓库，并指定一个缩写。

#### 远程仓库中抓取与拉取

从远程仓库中获取数据

```git
$ git fetch <remote>
```

#### 推送到远程仓库

```git
$ git push origin master
```

#### 远程仓库的重命名与移除

通过 `git remote rename` 命令来修改远程仓库的简写名。

例如将 pb 重命名为 paul:

```git
$ git remote rename pb paul
$ git remote
origin
paul
```

通过 `git remote remove` 或 `git remote rm` 命令移除一个远程仓库。

```git
$ git remote remove paul
```

一旦你使用这种方式删除了一个远程仓库，那么所有和这个远程仓库相关的远程跟踪分支以及配置信息也会一起被删除。

### 设置标签

Git 可以给仓库历史中的某个提交打上标签，以示重要。如使用这个功能来标记发布节点（v1.0、v2.0 等）。

#### 列出标签

通过 `git tag` 命令列出标签（可带上可选的 `-l` 选项 `--list`）:

```git
$ git tag
v1.0
v2.0
```

还可以通过特定模式查找标签：

```git
$ git tag -l "v1.8.5*"
v1.8.5
```

#### 创建标签

Git 支持两种标签：轻量标签（lightweight）与附注标签（annotated）。

轻量标签：某个特定提交的引用。

附注标签：存储在 Git 数据库中的一个完整的对象，可以被效验，并且包含一定的信息。

### Git 别名 todo

## 参考链接

* [Git 教程 | 廖雪峰](https://www.liaoxuefeng.com/wiki/896043488029600)
* [Git - Book](https://git-scm.com/book/zh/v2)
