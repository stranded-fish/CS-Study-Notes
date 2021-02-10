# Markdown 学习

* Markdown 是一种轻量级标记语言，它允许人们使用易读易写的纯文本格式编写文档。
* Markdown 编写的文档可以导出 HTML 、Word、图像、PDF、Epub 等多种格式的文档。

该教程主要针对：**Markdown 语法**、**VS Code 相关设置** 及 **快捷键** 等学习。

目录：

- [Markdown 学习](#markdown-学习)
  - [标题](#标题)
  - [段落](#段落)
  - [字体](#字体)
  - [分割线](#分割线)
  - [删除线](#删除线)
  - [脚注](#脚注)
  - [列表](#列表)
    - [无序列表](#无序列表)
    - [有序列表](#有序列表)
    - [嵌套列表](#嵌套列表)
  - [区块](#区块)
    - [单一区块](#单一区块)
    - [嵌套区块](#嵌套区块)
    - [区块中使用列表](#区块中使用列表)
    - [列表中使用区块](#列表中使用区块)
  - [代码](#代码)
    - [代码片段](#代码片段)
    - [代码区块](#代码区块)
  - [链接](#链接)
  - [图片](#图片)
  - [表格](#表格)
  - [高级技巧](#高级技巧)
    - [HTML元素](#html元素)
    - [转义](#转义)
    - [公式](#公式)
    - [目录](#目录)
    - [自定义CSS样式](#自定义css样式)
  - [相关插件](#相关插件)
  - [相关设置](#相关设置)
  - [快捷键](#快捷键)
  - [格式转换 pandoc todo](#格式转换-pandoc-todo)
  - [参考链接](#参考链接)

## 标题

使用 = 标记一级标题、- 标记二级标题：

* 只需要单个符号 `=`、`-` 即可，符号与内容之间 **不能有空行**。
* 一般情况为了避免歧义（与列表、分隔符混淆），通常避免使用这种标题设置方式。

```markdown
一级标题
============

二级标题
------------
```

使用 # 号标记：

* `#` 号前不能有除空格以外内容（可与序号嵌套）。
* `#` 后无空格要求，可直接添加内容，从书写风格考虑，一般要求 **单个** 空格。

```markdown
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```

## 段落

- 换行：在该行末尾使用两个以上的空格实现换行新的一行。
  VS Code 可在setting.json中添加设置：`"markdown.preview.breaks": true`  实现回车换行。
- 段落：通过在行文字后面用一个空行来实现空行新的一行（此时会空出一行）。
  
## 字体

VS Code 快捷键：

- 斜体：<kbd>Ctrl</kbd> + <kbd>I</kbd>
- 粗体：<kbd>Ctrl</kbd> + <kbd>B</kbd>
- 粗斜体：<kbd>Ctrl</kbd> + <kbd>I</kbd> + <kbd>Ctrl</kbd> + <kbd>B</kbd>
  
注：符号必须 **紧邻**，不能有空格。

```markdown
*斜体文本*
_斜体文本_
**粗体文本**
__粗体文本__
***粗斜体文本***
___粗斜体文本___
==高亮文本==
```

*斜体文本*  
**粗体文本**
***粗斜体文本***
==高亮文本==

更多设置可通过内嵌 html 元素实现，但 Markdown 哲学要求尽可能地避免使用 html 标签，以确保可读性与可移植性。

## 分割线

可以通过在一行中使用三个以上星号、减号、底线来实现一条分割线（行内可以有空格，但不能有其他东西）。

注：减号 `-` 上方紧接上方内容时，将会被识别为标题符号，故需上方空行。

```markdown
***

* * *

*****

- - -

----------
```

## 删除线

文字两端添加两个波浪线 ~~

注：符号须 **紧邻**，不能有空格。

```markdown
~~test~~
```

~~test~~

## 脚注

文本中脚注内部须 **紧邻**，不能有空格，其周围可以有空格，脚注解释须有 **英文冒号**。

```markdown
[^要注明的文本]
[^要注明的文本]：脚注内容
```

创建脚注 [^脚注]
[^脚注]:测试脚注

## 列表

### 无序列表

无序列表使用 `*`、`+` 或是 `-` 作为列表标记，标号后紧跟一个 **空格**。

```markdown
* 第一项
* 第二项
* 第三项

+ 第一项
+ 第二项
+ 第三项


- 第一项
- 第二项
- 第三项
```

示例：

- 第一项
- 第二项
- 第三项

### 有序列表

有序列表使用数字并加上 `.` 表示。

```markdown
1. 第一项
2. 第二项
3. 第三项
```

1. 第一项
2. 第二项
3. 第三项

### 嵌套列表

在子列表中的选项前面添加 **四个空格** 或 **<kbd>Tab</kbd>**。

```markdown
1. 第一项：
    - 第一项嵌套的第一个元素
    - 第一项嵌套的第二个元素
2. 第二项：
    - 第二项嵌套的第一个元素
    - 第二项嵌套的第二个元素
```

1. 第一项：
    - 第一项嵌套的第一个元素
    - 第一项嵌套的第二个元素
2. 第二项：
    - 第二项嵌套的第一个元素
    - 第二项嵌套的第二个元素
  
## 区块

### 单一区块

区块引用是在段落开头使用 `>` 符号，同时可以为了方便，仅在整个段落的第一行添加 `>`。

```markdown
> 区块引用
> 菜鸟教程
```

> 区块引用
> 菜鸟教程

```markdown
> 区块引用
仅在段落的第一行测试
```

> 区块引用
仅在段落的第一行测试

### 嵌套区块

```markdown
> 最外层
> > 第一层嵌套
> > > 第二层嵌套
```

> 最外层
> > 第一层嵌套
> > > 第二层嵌套

### 区块中使用列表

```markdown
> 区块中使用列表
> 1. 第一项
> 2. 第二项
> + 第一项
> + 第二项
```

> 区块中使用列表
>
> 1. 第一项
> 2. 第二项
>
> - 第一项
> - 第二项

### 列表中使用区块

需要在 `>` 前添加 **四个空格** 的缩进 或 **<kbd>Tab</kbd>**

```markdown
* 第一项
    > 菜鸟教程
    > 学的不仅是技术更是梦想
* 第二项
```

- 第一项
    > 菜鸟教程
    > 学的不仅是技术更是梦想
- 第二项

## 代码

### 代码片段

通过反引号包围 <kbd>`</kbd>

```markdown
`printf()`函数
```

`printf()`函数

### 代码区块

eg 1. 代码区块使用 **4个空格** 或者 一个 **<kbd>Tab</kbd>** （该方法无法指定语言类型）。

通常在代码量不大的情况下使用该方法，增强源文本可读性。

for example:

    echo hello world

eg 2. 通过` ``` `包围一段代码，并指定语言（也可不指定）。

```java
public static void main(String args[]) {
    System.out.println("Hello World");
}
```

## 链接

eg 1 与 eg 2 均需要 **指定http协议**，否则将作为当前路径下文件名读取，可通过该方法，设置相对路径，指定相关静态文件。

```markdown
eg 1. [链接名称](链接地址) // 链接地址，以 # 开头可以设置为文件标题用于导航，如：[Markdown学习](#markdown-学习)

eg 2. 高级链接，通过变量来设置一个链接，变量赋值在文档末尾进行
这个链接用 1 作为网址变量 [Google][1]
[1]: http://www.google.com/

eg 3. <链接地址>
```

eg 1:
这是一个链接 [菜鸟教程](http://www.runoob.com)
这是一个页内导航 [Markdown学习](#markdown-学习)
  
eg 2:  
这个链接用 1 作为网址变量 [Google][1]  
测试链接 [baidu][2]

[1]: http://www.google.com/
[2]: http://www.baidu.com/

eg 3:  
<https://www.runoob.com>

## 图片

默认将图片放在 **图床** 中，以便分享传播。
  
```markdown
![alt 属性文本](图片地址)
![alt 属性文本](图片地址 "可选标题")
```

![RUNOOB 图标](http://static.runoob.com/images/runoob-logo.png)

![RUNOOB 图标](http://static.runoob.com/images/runoob-logo.png "RUNOOB")

通过标签设置图片高度与宽度。
`<img src="http://static.runoob.com/images/runoob-logo.png" width="50%">`
<img src="http://static.runoob.com/images/runoob-logo.png" width="50%">

## 表格

使用 | 来分隔不同的单元格，使用 - 来分隔表头和其他行。

注：无相关空格要求。
  
```markdown
表格可以不用开头和结尾的 | 可以简单一点
| 表头   | 表头   |
|------|------|
| 单元格 | 单元格 |
| 单元格 | 单元格 |
```

| 表头   | 表头   |
|------|------|
| 单元格 | 单元格 |
| 单元格 | 单元格 |

设置对齐方式：

- -:  设置内容和标题栏居右对齐
- :-  设置内容和标题栏居左对齐
- :-: 设置内容和标题栏居中对齐

左对齐 | 右对齐 | 居中对齐
----|-----|-----
单元格 | 单元格 | 单元格
单元格 | 单元格 | 单元格

## 高级技巧

### HTML元素

元素                                                                             | 效果
:--------------------------------------------------------------------------------|:--------------------------------------------------------------------------
\<u>下划线文本\</u>                                                              | <u>下划线文本</u>
\<kbd>Ctrl\</kbd>                                                                | <kbd>Ctrl</kbd>
\<font color=#0099ff size=5 face="黑体">color=#0099ff size=5 face="黑体"\</font> | <font color=#0099ff size=5 face="黑体">color=#0099ff size=5 face="黑体"</font>
\<table>\<tr>\<td bgcolor=orange>背景色是：orange\</td>\</tr>\</table>            | <table><tr><td bgcolor=orange>背景色是：orange</td></tr></table>
篮球 \<br> 羽毛球                                                                | 篮球 <br> 羽毛球

### 转义

通过反斜杠 \ 转义特殊字符。

可通过转义一些特殊字符达到使 Markdown 语法失效的效果，从而实现在一些情况下屏蔽 markdown 自身设定，得到自己想要的显示效果。

1. 有序标题
2. 自动缩进

```markdown
1\. 转义字符
2\. 缩进失效
```

1\. 转义字符
2\. 缩进失效

### 公式

使用两个美元符 $$ 包裹 TeX 或 LaTeX 格式的数学公式来实现。

```LaTex
$$
\mathbf{V}_1 \times \mathbf{V}_2 =  \begin{vmatrix}
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 \\
\end{vmatrix}
${$tep1}{\style{visibility:hidden}{(x+1)(x+1)}}
$$
```

### 目录

通过 `[TOC]` 自动生成目录。

注：有些渲染器无法解析 `[TOC]`，故为了确保可移植性，可使用 Markdown All in One 插件来自动生成目录。

### 自定义CSS样式

可以通过添加自定义 CSS 样式，实现高级的布局功能。

通过CSS实现标题自动编号：

* 直接在 `.md` 文件中内嵌CSS代码。
* 将CSS代码，保存到一个独立的CSS文件中，并通过代码引用。

      <link rel="stylesheet" type="text/css" href="auto-number-title.css" />

## 相关插件

1. Markdown All in One (相关文本编辑功能、快捷键等)
2. Markdown Preview Enhanced (文本高级预览功能)
3. Markdown Shortcuts (提供快捷键功能)
4. markdownlint (提供语法检查、提示与自动修复功能)
5. Markdown Table Prettifier (提供表格美化功能)
6. Markdown PDF（提供 Markdown 打印功能）

## 相关设置

1. 回车换行设置：
  VS Code：settings.json -> 添加 `"markdown.preview.breaks": true,`
2. 关于图床的相关设置
  使用PicGo作为图床软件，快捷键设置：<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>A</kbd>。
3. 关于markdownlint 规则的自定义设置
  在setting.json中添加配置数据：

   ```JSON
   // markdownlint 语法配置
   "markdownlint.config":
    {
       "default": true,
       "MD033":false, // 允许内嵌 html 元素
       "MD040": false // 允许代码块 ``` 不指定语言
     },
   ```

## 快捷键

VS Code 环境下：
  
key                                                             | command
:---------------------------------------------------------------|:--------------
<kbd>Ctrl</kbd> + <kbd>B</kbd>                                  | 加粗
<kbd>Ctrl</kbd> + <kbd>I</kbd>                                  | 斜体
<kbd>Alt</kbd> + <kbd>S</kbd>                                   | 删除线
<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>]</kbd>               | 当前标题升级
<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>[</kbd>               | 当前标题降级
<kbd>Ctrl</kbd> + <kbd>L</kbd>                                  | 添加链接
<kbd>Ctrl</kbd> + <kbd>M</kbd> + <kbd>C</kbd>                   | 生成代码块 \``` ```
<kbd>Ctrl</kbd> + <kbd>M</kbd> + <kbd>I</kbd>                   | 生成行内代码块 \` `
<kbd>Ctrl</kbd> + <kbd>M</kbd> + <kbd>B</kbd>                   | 生成无序标题
<kbd>Ctrl</kbd> + <kbd>M</kbd> + <kbd>1</kbd>                   | 生成有序标题
<kbd>Ctrl</kbd> + <kbd>M</kbd> + <kbd>X</kbd>                   | 生成可选框
<kbd>Ctrl</kbd> + <kbd>M</kbd> + <kbd>M</kbd>                   | 显示出所有功能
<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>M</kbd>               | 显示所有错误信息
<kbd>Alt</kbd> + <kbd>F8</kbd>                                  | 显示当前错误信息
<kbd>Ctrl</kbd> + <kbd>.</kbd>                                  | 自动修复
<kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>F</kbd> \  <kbd>L</kbd> | 格式化所有表格
<kbd>Shift</kbd> + <kbd>Alt</kbd> + <kbd>→</kbd>                | 内容选取

命令面板操作（ <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd> ）：

key              | command
:----------------|:---------------------
create table ... | 创建表格
export ...       | 打印为 pdf、html、png  ...

## 格式转换 pandoc todo

## 参考链接

* [Markdown 教程 | 菜鸟教程](https://www.runoob.com/markdown/md-tutorial.html)
* [MarkDown标题自动添加编号](https://yanwei.github.io/misc/markdown-auto-number-title.html)
