# Hugo Quick Start

Hugo 是一个用 Go 编写的开源的静态网站生成器，通过其惊人的速度和灵活性，Hugo 使网站建设再次变得有趣。

Hugo 特点：

* 极快的构建时间（每页 < 1 ms）
* 跨平台，易于在 macOS、Linux、Windows 等上安装
* 在开发过程中可以利用 LiveReload 实时呈现变化
* 拥有大量的主题
* 可以在任何地方托管您的站点

目录：

- [Hugo Quick Start](#hugo-quick-start)
  - [安装 Hugo](#安装-hugo)
  - [创建 Hugo 网站](#创建-hugo-网站)
  - [参考链接](#参考链接)

## 安装 Hugo

从 [Hugo Releases](https://github.com/gohugoio/hugo/releases) 下载适合自身操作系统的二进制版本：

* Linux：`hugo_x.xx.x_Linux-64bit.tar.gz`
* Windows：`hugo_x.xx.x_Windows-64bit.zip`

下载完成后解压，即可运行 `hugo` 可执行文件：

```bash
hugo version
```

建议将 `hugo` 移动到特定位置，并设置环境变量 `PATH`，以便于使用。

## 创建 Hugo 网站

**Step 1.** 创建新网站

```bash
hugo new site quickstart
```

该命令将在当前目录下名为 `quickstart` 的文件夹中创建一个新的 Hugo 站点。

**Step 2.** 添加主题

可通过 [Hugo Themes](https://themes.gohugo.io) 寻找喜欢的主题，然后将其作为子模块添加到新建站点中。

```bash
cd quickstart
git init
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
```

然后，将主题添加到站点配置文件中：

```bash
echo theme = \"ananke\" >> config.toml
```

**Step 3.** 添加内容

**eg 1.** 手动创建文件 `/content/posts/my-first-post.md`，并添加元数据。

**eg 2.** 通过 `new` 命令创建，并自动生成部分元数据：

```bash
hugo new posts/my-first-post.md
```

自动生成元数据如下：

```markdown
---
title: "My First Post"
date: 2022-04-28T16:20:20+08:00
draft: true
---
```

可打开新创建文件进一步编辑。

> `draft: true` 表示当前内容为草稿，不会部署。文章完成后，可将帖子的标题更新为 `draft: false`。

**Step 4.** 启动 Hugo server

在站点根目录（即 `config.toml` 文件所在目录）执行以下命令：

```bash
# -D, --buildDrafts 同时构建被标记为 draft 草稿的内容
hugo server -D
```

构建完成后，可访问 [<http://localhost:1313/>](http://localhost:1313/) 浏览新建站点。

在服务器运行期间，可随意编辑或新增内容，Hugo 将呈现实时变化，刷新浏览器即可查看。

**Step 5.** 网站配置

修改 `config.toml` 配置文件，如：`title` 和 `baseURL` 等。

**Step 6.** 构建静态网站

```bash
hugo
```

默认将输出至 `./public/` 目录中（可通过 `-d` / `--destination` 参数指定输出路径，或者在配置文件中设置`publishdir`）。

## 参考链接

* [Hugo Documentation](https://gohugo.io/documentation/)
