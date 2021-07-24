# AutoHotkey 快速入门

AutoHotkey 是一种免费的、开源的 Windows 脚本语言，它允许用户轻松地为各种任务创建简单或复杂的脚本，例如：表单填充、自动单击、宏等。

该教程主要针对 **AutoHotkey 的基础用法** 学习。更多的内容可见 [AutoHotkey 官方文档](https://www.autohotkey.com/docs/AutoHotkey.htm)。

目录：

- [AutoHotkey 快速入门](#autohotkey-快速入门)
  - [基础](#基础)
    - [安装 AutoHotkey](#安装-autohotkey)
    - [创建脚本](#创建脚本)
  - [快捷键 & 热字串](#快捷键--热字串)
    - [快捷键修饰符 & 常规按键](#快捷键修饰符--常规按键)
    - [上下文相关的快捷键](#上下文相关的快捷键)
  - [命令 & 函数](#命令--函数)
    - [命令](#命令)
    - [函数](#函数)
  - [参考链接](#参考链接)

## 基础

### 安装 AutoHotkey

* 访问 [AutoHotkey 主页](https://autohotkey.com/);
* 点击 [下载](https://autohotkey.com/download/ahk-install.exe)。

> **注意：** 在安装 Autohotkey 过程中，需要选择安装 UNICODE 或者 ANSI 版本。如果需要它支持非英文字符和数字, 就选择安装 UNICODE 版。

安装完成后，可在其安装目录下找到帮助文件 `AutoHotkey.chm`。

### 创建脚本

1. 新建 `.ahk` 脚本文件；
2. 编辑该文件并保存；
3. 双击该文件启动脚本。

脚本启动后会显示在系统托盘区，右击可对其进行暂停、重启、编辑等各种操作。

## 快捷键 & 热字串

* 快捷键：用于触发某些动作的按键或组合按键。
  * 按键名或组合按键名在 `::` 左边；
  * 代码跟在下面；
  * 最后以 Ruturn 结尾。

  ```autohotkey
  ^j::
    Send, My First Script
    MsgBox Wow!
    Run, Notepad.exe
  Return
  ```

* 热字串：用于扩展缩写（自动替换），也可用于启动任何脚本动作。
  * 在要触发替换的文本两边添加 `::`；
  * 替换后的文本在第二对冒号 `::` 的右边。

  ```autohotkey
  ; eg 1 扩展缩写
  ::ftw::Free the whales

  ; eg 2 启动脚本动作
  ::btw::
    MsgBox You typed "btw".
    Run, Notepad.exe
  Return
  ```

### 快捷键修饰符 & 常规按键

符号/名称 | 描述
:-----:|:---
**#** |	<kbd>Win</kbd> (Windows home)
**!** / Alt |	<kbd>Alt</kbd>
**^** / Ctrl |	<kbd>Ctrl</kbd>
**+** / Shift |	<kbd>Shift</kbd>
**<** |	使用成对按键中左边的那个
**>** |	使用成对按键中右边的那个
CapsLock | <kbd>CapsLock</kbd>（大小写锁定键）
Space | <kbd>Space</kbd>（空格键）
Tab | <kbd>Tab</kbd>（Tab 键）
Enter | <kbd>Enter</kbd>（回车键）
Escape（或 Esc） | </kbd>Esc</kbd>（退出键）
Backspace（或 BS） | <kbd>Backspace</kbd>（退格键）

部分常规按键需要 \` 符号进行转义，如：;（分号） ,（逗号） 等。

**快捷键名称** 之间可通过 `&` 符号进行组合，形成自定义组合键。（**快捷键修饰符** 无需 `&` 连接，直接组合即可）

> 完整快捷键修饰符列表 [中文](https://wyagd001.github.io/zh-cn/docs/Hotkeys.htm#Symbols) / [English](https://www.autohotkey.com/docs/Hotkeys.htm#Symbols)。
> 完整按键名称列表 [中文](https://wyagd001.github.io/zh-cn/docs/KeyList.htm) / [Engli](https://www.autohotkey.com/docs/KeyList.htm)。

### 上下文相关的快捷键

`\#IfWinActive/Exist` 和 `#If` 指令可以用来让热键根据不同的条件执行不同的动作（或什么都不做）。

```autohotkey
; 以下快捷键将仅在记事本中生效
#IfWinActive, ahk_class Notepad
^a::MsgBox 你在记事本中按下了 Ctrl-A. 而在其他窗口中按下 Ctrl-A 将原样发送.
#c::MsgBox 你在记事本中按下了 Win-C 组合键.
```

## 命令 & 函数

### 命令

```autohotkey
Command, 参数1, 参数2, 参数3
```

**命令特点：**

* 不能将多条命令放在同一行（IfEqual 除外）。
* 不能将一个命令作为参数插入到另一个命令中。
* 命令使用传统语法，需要在变量前后添加 `%`，如 `%val%`。
* 命令的参数不能参与运算。

**常用命令：**

1\. 发送按键

`Send` 命令，表示发送按键，用于模拟打字或按键操作。

```autohotkey
^j::
  Send, hello
Return
```

> **注意①：** `Send` 命令会将 `#`、`!`、`^` 等符号视为快捷键修饰符，如需打印输出，需添加 `{}` 将符号括起来进行转义。
>
> ```autohotkey
> Send, This text has been typed{!}
> ; 注意 {} 中的 !(感叹号)? 这是因为, 如果没有 {}，AHK 将按下 Alt 键。
> ```
>
> **注意②：** 常规按键与快捷键修饰符相反，默认直接打印输出，如果需要执行按键，需要添加 `{}` 符号。
>
> ```autohotkey
> Send, Multiple Enter lines have Enter been sent. ; 直接输出
> Send, Multiple{Enter}lines have{Enter}been sent. ; 回车换行
> ```

2\. 运行外部程序

`Run` 命令，可用于打开指定程序或网址。

```autohotkey
; 运行一个程序。注意：大部分的程序可能需要完整路径。
Run, %A_ProgramFiles%\Some_Program\Program.exe

; 打开一个网址
Run, https://autohotkey.com
```

### 函数

```autohotkey
Function(参数1, 参数2, 参数3)
```

**函数特点：**

* 能够使用运算，如：`SubStr(37 * 12, 1, 2)`。
* 变量不需要添加 `%`。
* 函数可以嵌套。
* 文本前后需要添加双引号。

**常用函数：**

1\. `GetKeyState()`

检查键盘、鼠标或操纵杆按键是否按下。

```autohotkey
; API
KeyIsDown := GetKeyState(KeyName , Mode)

; eg
if GetKeyState("control") = 0 {
      if GetKeyState("shift") = 0
          Send, {Left}
      else
          Send, +{Left}
      return
  } else {
      if GetKeyState("shift") = 0
          Send, ^{Left}
      else
          Send, +^{Left}
      return
  }
```

> 完整命令和内置函数列表 [中文](https://wyagd001.github.io/zh-cn/docs/commands/index.htm) / [English](https://www.autohotkey.com/docs/commands/index.htm)。

## 参考链接

* [AutoHotkey Beginner Tutorial](https://www.autohotkey.com/docs/Tutorial.htm)
* [AutoHotkey 初学者向导](https://wyagd001.github.io/zh-cn/docs/Tutorial.htm)
