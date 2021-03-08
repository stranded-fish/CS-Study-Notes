# VS Code 学习

该教程主要针对：**VS Code 配置**、**功能** 与 **快捷键** 等学习。

目录：

- [VS Code 学习](#vs-code-学习)
  - [VS Code 设置](#vs-code-设置)
    - [工作区层次](#工作区层次)
    - [新建工作区](#新建工作区)
    - [工作区配置](#工作区配置)
    - [工作区插件管理](#工作区插件管理)
    - [VS Code settings.json](#vs-code-settingsjson)
    - [设置同步](#设置同步)
  - [快捷键](#快捷键)
    - [快捷键设置](#快捷键设置)
    - [主命令框](#主命令框)
    - [窗口管理](#窗口管理)
    - [代码编辑](#代码编辑)
  - [参考链接](#参考链接)

## VS Code 设置

### 工作区层次

VS Code 层级：**系统全局默认设置（不可修改）-> 用户设置 -> 工作区设置 -> 文件夹设置**
**优先级依次升高**，后者的设置会覆盖前者的设置，若没有设置某一项，将继续使用前者的设置。

注：系统全局默认设置（`defaultSettings.json`），位于安装目录中的特定位置，只读。

* **用户设置 (`settings.json`)：** 全局设置，若无相关设置则使用默认设置。
* **工作区设置 (`工作区.code-workspace`)：** 对不同的工作环境使用不同的配置，若某项无设定，即使用上一层用户设置。
* **文件夹设置 (`.vscode/settings.json`)：** 项目设置，将一个文件夹作为一个项目，对同一个工作区的不同项目，使用不同的配置，若某项无设定，即使用上一层工作区设置。

### 新建工作区

新建工作区：文件(F) -> 将工作区另存为...

### 工作区配置

.../工作区.code-workspace

```JSON
{
    "folders": [

        // 设置工作区包含的文件夹
        {
            // 将当前工作区文件所在的文件夹加入到工作区中
            "path": "."
        },

        {
            // 自行将其他路径的文件夹加入到工作区中
            // 可通过为加入的不同文件夹设置独立的settings.json文件，实现同一工作区，不同项目，不同配置
            "path": "D:\\Workspaces\\VSCode\\Python"
        }
    ],

    // 用于设置当前工作区调用的程序路径
    "settings": {
         "python.pythonPath": "C:\\Program Files\\Python37\\python.exe"
    }
}
```

### 工作区插件管理

选择需要管理的插件：

* "禁用"，为用户设置
* "禁用（工作区）"，为工作区设置

### VS Code settings.json

* `settings.json` 文件为VS Code配置文件之一，用于控制工作项的配置。
* 文件为 JSON 形式，包含多条配置数据（键值对形式，以逗号分隔）并与 **UI 设置界面同步**。
* 打开方式：<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd>  并输入 `settings.json`。
* settings.json文件有 **用户设置**、**文件夹设置** 两个分支，分别对应了 **不同位置的配置文件：**
  * **用户设置：** 位于与用户相关联的特定文件夹中 `C:\Users\User\AppData\Roaming\Code\User\settings.json`
  * **文件夹设置：** 位于工作区配置下的文件夹中的.vscode子目录下 `工作区路径\.vscode\settings.json`
  ![UI配置](https://i.loli.net/2020/10/21/CTanFbRPjGVH2Yw.png)
* 优先级：**文件夹设置 ＞ 用户设置** ，**（高优先级可覆盖低优先级）** 可以为不同的项目，做不同的配置。
* 同一工作区，多个文件夹（项目）配置示例：
  * 修改 **`工作区.code-workspace`** 将其他文件夹 path 加入到工作区中。
  * 在新添加文件夹中新建 `.vscode` 文件夹，并在 `.vscode` 文件夹中新建 `settings.json` 文件。
  * 在 `settings.json` 中，添加相应的配置，覆盖全局用户配置。

### 设置同步

通过扩展插件 `Settings Sync` 实现利用 GitHub Gist 跨设备同步设置、片段、主题、文件图标、启动、按键绑定、工作空间和扩展。

key                                              | command
:------------------------------------------------|--------
sync（命令面板下）                                 | 显示所有命令
<kbd>Shift</kbd> + <kbd>Alt</kbd> + <kbd>U</kbd> | 更新\上传配置
<kbd>Shift</kbd> + <kbd>Alt</kbd> + <kbd>D</kbd> | 下载配置

## 快捷键

### 快捷键设置

**eg 1. 图形化界面**

1. <kbd>F1</kbd> / <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd> 打开命令面板；
2. type Shortcuts；
3. 搜索相关快捷键操作，并修改。

**eg 2. 修改 keybindings.json**

1. <kbd>F1</kbd> / <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd> 打开命令面板；
2. type Shortcuts；
3. 点击右上方图标，打开 `keybindings.json`。

![打开 keybindings.json](https://i.loli.net/2021/03/01/6LnrmXWFRa4ixOD.png)

### 主命令框

key                                                               | command
:-----------------------------------------------------------------|:-------
<kbd>F1</kbd> / <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd> | 打开命令面板
<kbd>Ctrl</kbd> + <kbd>P</kbd>                                    | 选择操作模式
<kbd>Ctrl</kbd> + <kbd>P</kbd> 常用操作如下：
> ?   列出当前可执行的动作
  !   显示 Errors或 Warnings，也可以 Ctrl+Shift+M
  :   跳转到行数，也可以 Ctrl+G 直接进入
  @   跳转到 symbol（搜索变量或者函数），也可以 Ctrl+Shift+O 直接进入
  @   根据分类跳转 symbol，查找属性或函数，也可以 Ctrl+Shift+O 后输入" : "进入

### 窗口管理

key                            | command
:------------------------------|:-------
<kbd>Ctrl</kbd> + <kbd>N</kbd> | 新建文件
<kbd>Ctrl</kbd> + <kbd>W</kbd> | 关闭当前窗口
<kbd>Ctrl</kbd> + <kbd>`</kbd>                    | 打开终端
<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>`</kbd> | 新建终端
<kbd>Ctrl</kbd> + <kbd>Tab</kbd>                  | 切换到最近使用文件
<kbd>Ctrl</kbd> + <kbd>R</kbd>                    | 切换工作区
<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>N</kbd> | 新建编辑器
<kbd>Ctrl</kbd> + <kbd>K</kbd> + <kbd>O</kbd>     | 将当前文件，在新建编辑器中打开
<kbd>Ctrl</kbd> + <kbd>K</kbd> + <kbd>W</kbd>     | 关闭所有编辑器
<kbd>Ctrl</kbd> + <kbd>\\</kbd>                   | 拆分窗口
<kbd>Ctrl</kbd> + <kbd>1</kbd> + <kbd>2</kbd> ... | 选中不同的窗口
<kbd>Ctrl</kbd> + <kbd>End</kbd>                  | 移动到文件结尾
<kbd>Ctrl</kbd> + <kbd>Home</kbd>                 | 移动到文件开头
<kbd>Alt</kbd> + <kbd>1</kbd>                     | 显示侧边栏
<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>E</kbd> | 聚焦侧边栏
<kbd>Insert</kbd>                                 | 新建文件
<kbd>Alt</kbd> + <kbd>Insert</kbd>                | 新建文件夹
<kbd>Shift</kbd> + <kbd>Alt</kbd> + <kbd>R</kbd>  | 选中侧边栏文件后，新建文件资源管理器并显示

### 代码编辑

key                                                             | command
:---------------------------------------------------------------|:----------------
<kbd>Ctrl</kbd> + <kbd>[</kbd> / <kbd>]</kbd>                   | 代码行向 左 / 右 缩进
<kbd>Alt</kbd> + <kbd>↑</kbd> / <kbd>↓</kbd>                    | 向 上 / 下 移动当前行
<kbd>Alt</kbd> + <kbd>←</kbd> / <kbd>→</kbd>                    | 后退 / 前进
<kbd>Shift</kbd> + <kbd>Alt</kbd> + <kbd>↑</kbd> / <kbd>↓</kbd> | 向 上 / 下 移动复制当前行
<kbd>Ctrl</kbd> + <kbd>C</kbd>                                  | 复制光标所在行
<kbd>Ctrl</kbd> + <kbd>Enter</kbd>                              | 在当前行下方插入一行，并移动光标
<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>Enter</kbd>           | 在当前行上方插入一行，并移动光标
<kbd>Ctrl</kbd> + <kbd>K</kbd> + <kbd>Q</kbd>                   | 返回上一编辑位置
<kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>L</kbd>                 | 格式化代码

## 参考链接

* [VS Code 添加新建文件夹快捷键（配置 when 属性）](https://blog.csdn.net/u011511756/article/details/85058990)
