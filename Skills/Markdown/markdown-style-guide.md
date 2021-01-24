# Markdown 风格指南

**Markdown 哲学：** 埏埴以为器，当其无，有器之用；凿户牖以为室，当其无，有室之用。故有之以为利，无之以为用。

为了让 Markdown 可简洁、可读性更强，我们追求满足以下三个目标：

* 源文本可读且可移植。
* Markdown 文件可随时跨团队维护。
* 语法简单易记。

辅助工具：Markdownlint 插件，该插件可以帮助 VS Code 完成大部分代码风格检测与自动修复。

该风格指南主要针对：Markdownlint 插件无法处理的一些排版问题。

目录：

- [Markdown 风格指南](#markdown-风格指南)
  - [文件名](#文件名)
  - [文件布局](#文件布局)
  - [空格](#空格)
  - [标点符号](#标点符号)
  - [强调](#强调)
  - [单行限制](#单行限制)
  - [嵌套列表缩进](#嵌套列表缩进)
  - [代码](#代码)
    - [inline](#inline)
    - [Codeblocks](#codeblocks)
      - [声明编程语言](#声明编程语言)
      - [使用更简洁的缩进代码](#使用更简洁的缩进代码)
      - [转义换行符](#转义换行符)
  - [表格](#表格)
  - [参考链接](#参考链接)
  - [TODO](#todo)

## 文件名

文件名建议使用如下风格：

* 使用小写字母（专有名词除外）。
* 使用连字符 - 作为分隔符。

解释：连字符为当前网址最流行的分隔符，并且 Markdown 文件在上下文中经常使用：

* 可能在同一个项目中有连字符分割的HTML文件使用Markdown相同的目录。
* 文件名会被直接使用到 URL 中，比如: GitHub blobs。

## 文件布局

文件总体布局：

```markdown
# Document Title | 文件标题

Short introduction

[TOC]

## Topic | 文章主题

Content

## see also | 参考链接

* https://link-to-more-info
```

1. `# Document Title`： 第一个标题应当是 一级标题，并且尽可能地与文件名相同或者类似。同时一级标题将会作用于页面。一级标题只应有一个，也不能够编号，正文均从二级标题开始。
2. `author`：可选。如果需要添加作者信息，可添加至一级标题下方。
3. `Short introduction`：1-3 个句子介绍该文件主题。
4. `[TOC]`：在简单介绍主题后，添加目录。注意，部分 Markdown 渲染器不支持 TOC自动生成目录（如 Github），故最好手动添加目录（可使用 Markdown All in one 插件实时更新目录）。
5. `## Topic`：其余标题应从 2 级标题开始。
6. `## See also`：在文章底部，添加参考链接。

注：尽量不使用 五级标题 与 六级标题，考虑用 有序列表 与 无序列表 代替。

## 空格

* 中文文字与 **英文**、**数字** 及 **@ # $ % ^ & * . ( ) 等符号** 之间 **加** 空格。
    > 在 LeanCloud 上，数据存储是围绕 AVObject 进行的。
    今天出去买菜花了 5000 元。
    我家的光纤入屋宽带有 10 Gbps，SSD 一共有 20 TB

* 中文标点（全角标点），与其他字符之间 **不加** 空格。
    > 刚刚买了一部 iPhone，好开心！

* 链接之间增 **加** 空格。
    > 访问我们网站的最新动态，请 点击这里 进行订阅！

* 加粗、斜体、高亮文本前后 **加** 空格，以避免解析失败。
    > 修复了一个 **内存泄露** 问题，该问题由 someone 在 版本 ==v0.1.1== 中引入。

* 行内代码的两端各添 **加** 一个空格。若行内代码紧邻标点符号，则其与标点之间 **不加** 空格。
    > 打开 Linux 虚拟终端，输入 `echo 'Hello World'`。

## 标点符号

标点通用规则：

* 中文排版时应全文使用中文全角标点，无论内容中是否包含英文词语。
* 若内容中包含完整的英文句子或段落，则使用英文半角标点。
* 有序列表和无序列表，除表示标题或短语之外，均需要正常标点符号。
  * 如果是完整句子，则末尾使用 `。` 。
  * 若不是完整句子，前后有联系，则除最后一个列表使用 `。` 其余列表使用 `；`。

其他规则：

* 通常使用直角引号，单引号「」，双引号『』。
    > 因为在使用 弯引号 时，虽然为中文语境，但却会显示为英文样式，而在Unicode编码中，仅通过中/西字体来判断使用的标点符号，故会被显示为英文标点（半角符号）与其他中文标点极不协调。
* 当半角符号 / 表示「或者」之意时，与前后的字符之间均不加空格。其他时候前后添加空格。

## 强调

中文需要强调某处内容时用粗体。中文排版中不使用斜体。在英文排版中可用斜体表示强调，书名或题目。

## 单行限制

在任何时候都需要遵循单行字符数限制。过长的 URLs 和 表格 可能会破坏这个限制。因此需要额外的改进。

```markdown
Lorem ipsum dolor sit amet, nec eius volumus patrioque cu, nec et commodo
hendrerit, id nobis saperet fuisset ius.

*   Malorum moderatius vim eu. In vix dico persecuti. Te nam saperet percipitur
    interesset. See the [foo docs](https://gerrit.googlesource.com/gitiles/+/master/Documentation/markdown.md).
```

通常，在插入长链接的时候换行，以保持可读性，并且最大程度减少溢出。

```markdown
Lorem ipsum dolor sit amet. See the
[foo docs](https://gerrit.googlesource.com/gitiles/+/master/Documentation/markdown.md)
for details.
```

## 嵌套列表缩进

使用空格缩进，使换行文本布局保持一致。

```markdown
* Foo,
  wrapped. // 2 个空格缩进，保持与上一级布局一致

1. 1 spaces
   and 3 space indenting. // 3 个空格缩进，保持与上一级布局一致
2. 2 spaces again.
```

## 代码

### inline

使用 inline code <kbd>``</kbd> ，修饰 **短代码引用** 与 **文件名**：

```markdown
You'll want to run `really_cool_script.sh arg`.

Pay attention to the `foo_bar_whammy` field in that table.

Be sure to update your `README.md`!
```

通常还可以使用 <kbd>``</kbd> 完成转义功能以代替 <kbd>\\</kbd>。

```markdown
\##     测试
`##     测试`
```

### Codeblocks

对于超过单行的代码引用，需使用代码块 <kbd>\``` \```</kbd> ：

```Python
def Foo(self, bar):
  self.bar = bar
```

#### 声明编程语言

已知编程语言类型的情况下，需明确指定语言类型以获取语法高亮。

#### 使用更简洁的缩进代码

4 个空格缩进将会被解析为代码块，出于代码简洁与增强源文本可读性的角度，在写较短的代码片段时，可以使用这种方法。

```markdown
You'll need to run:

    bazel run :thing -- --foo

And then:

    bazel run :another_thing -- --bar

And again:

    bazel run :yet_again -- --baz
```

#### 转义换行符

由于大多数命令行片段，都需要直接复制粘贴到控制台中运行，故最好的做法是将太长的命令行，分为两段后，在行尾转义换行符。

```Shell
bazel run :target -- --flag --foo=longlonglonglonglongvalue \
--bar=anotherlonglonglonglonglonglonglonglonglonglongvalue
```

## 表格

* 表格应省略 行首 与 行尾 处 `|` 。
* 并且表格源文本需要保持对齐以保证可读性（当表格格式满足要求后，可通过 Markdown Table Prettifier 插件实现表格自动格式化）。
* 表格标题与内容需保持同种对齐方式。

```markdown
key                                               | command
:-------------------------------------------------|:--------------
<kbd>Ctrl</kbd> + <kbd>B</kbd>                    | 加粗
<kbd>Ctrl</kbd> + <kbd>I</kbd>                    | 斜体
```

上方表格内容默认左对齐，故需额外设置左对齐，以满足标题与内容对齐方式相同。

## 参考链接

* [Markdown styleguide](https://google.github.io/styleguide/docguide/style.html)
* [Markdown Philosophy](https://google.github.io/styleguide/docguide/philosophy.html)
* [Markdown 书写风格指南](https://einverne.github.io/markdown-style-guide/zh.html)
* [基于 Markdown 的中文文档排版规范](https://xie.infoq.cn/article/69feb60ca6fba4ae0c8adeef6)
* [Markdown 编写规范](https://github.com/fex-team/styleguide/blob/master/markdown.md)
* [中文里直角引号与弯引号的使用](https://www.zhihu.com/question/20110888)

## TODO

整理各种细节排版风格（主要是序号排序问题，而不选择用下一级标题的方法）

1. 比如 centos IP config 文件中 关于显示步骤，而不使用有序标题缩进的布局方式
2. 比如 apis 关于显示非常多的无需标题，但不使用有序标题缩进的布局方式
