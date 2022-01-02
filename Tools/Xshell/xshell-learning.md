# Xshell 学习

Xshell 是一款功能强大的终端模拟器，支持 SSH2，SSH3，SFTP，TELNET，RLOGIN 和 SERIAL。

该教程主要针对：**Xshell 个性化设置** 与 **常用快捷键** 的学习

目录：

- [Xshell 学习](#xshell-学习)
  - [个性化设置](#个性化设置)
    - [修改字体与大小](#修改字体与大小)
    - [个性化配色](#个性化配色)
  - [常用快捷键](#常用快捷键)
  - [参考链接](#参考链接)

## 个性化设置

### 修改字体与大小

1. 打开 Xshell 点击菜单栏 -> 文件 -> （当前会话/默认会话）属性
2. 选择外观
3. 修改字体与大小

**注：**

* 当前会话指当前选定的连接会话，修改后，之后新建的该连接会话都会应用修改属性。
* 默认会话指系统默认终端会话，即通过点击标签栏 + 号直接新建的会话。

![修改字体与大小](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/20220102150514.png)

### 个性化配色

1. 新建文件命名为：`Solarized-dark.xcs`。
2. 复制以下内容到新建文件中：

    <br>

    > [Solarized Dark]
      text=839496
      cyan(bold)=93a1a1
      text(bold)=408080
      magenta=dd3682
      green=859900
      green(bold)=586e75
      background=042028
      cyan=2aa198
      red(bold)=cb4b16
      yellow=b58900
      magenta(bold)=6c71c4
      yellow(bold)=657b83
      red=808000
      white=eee8d5
      blue(bold)=8080ff
      white(bold)=fdf6e3
      black=002b36
      blue=268bd2
      black(bold)=073642
      [Names]
      name0=Solarized Dark
      count=1

3. 打开 Xshell 点击菜单栏 -> 工具 -> 配色方案 -> 导入 -> 选择刚才新建的 `.xcs` 文件。
4. 选择刚才导入的配色方案，点击确定。

Solarized Dark 配色效果展示：

![Solarized Dark 配色效果展示](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/20220102150541.png)

## 常用快捷键

key                                               | command
:-------------------------------------------------|:----------
<kbd>Alt</kbd> + {0...9}                          | 转到特定标签
<kbd>Ctrl</kbd> + <kbd>Tab</kbd>                  | 标签间跳转
<kbd>Ctrl</kbd> + <kbd>L</kbd>                    | 清屏
<kbd>Alt</kbd> + <kbd>Enter</kbd>                 | 全屏
<kbd>Alt</kbd> + <kbd>N</kbd>                     | 新建会话
<kbd>Alt</kbd> + <kbd>O</kbd>                     | 打开会话
<kbd>Alt</kbd> + <kbd>P</kbd>                     | 会话属性
<kbd>Alt</kbd> + <kbd>R</kbd>                     | 透明
<kbd>Alt</kbd> + <kbd>S</kbd>                     | 简单模式
<kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>F</kbd>   | 传输新建文件
<kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>T</kbd>   | 新建终端
<kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>N</kbd>   | 新建窗口
<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>C</kbd> | 复制（自定义）
<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>V</kbd> | 粘贴（自定义）
<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>W</kbd> | 关闭选项卡（自定义）

## 参考链接

* [几款xshell绝佳配色方案](https://blog.csdn.net/hxspace/article/details/79851144?utm_medium=distribute.pc_relevant.none-task-blog-searchFromBaidu-2.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-searchFromBaidu-2.control)
