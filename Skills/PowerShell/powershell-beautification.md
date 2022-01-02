# PowerShell 美化

该教程主要利用 oh-my-posh 实现对 PowerShell 的美化。

最终效果：

Windows Terminal:
![Windows Terminal](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/20220102145721.png)

VS Code:
![VS Code](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/20220102145736.png)

IDEA:
![IDEA](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/20220102145748.png)

目录：

- [PowerShell 美化](#powershell-美化)
  - [安装 Windows Terminal](#安装-windows-terminal)
  - [安装 oh-my-posh](#安装-oh-my-posh)
  - [安装依赖字体](#安装依赖字体)
  - [设置主题](#设置主题)
  - [自定义主题](#自定义主题)
  - [参考链接](#参考链接)

## 安装 Windows Terminal

出于视觉以及功能性角度（原 PowerShell 默认终端添加新字体较为麻烦，而美化主题必须用到第三方字体），建议使用第三方更美观的 Terminal，如 Terminus、cmder、Windows Terminal 等。**若仅考虑在各 IDE（如：VS Code、IDEA）中使用嵌入式终端可忽略该步骤。**

本教程选择使用 Windows Terminal。该终端是一款新式、快速、高效且强大的终端应用程序，适用于命令行工具和命令提示符，PowerShell 和 WSL 等 Shell 用户。该终端尚无 GUI 配置界面，须手动修改 JSON 配置文件完成配置修改。

Windows Terminal 安装步骤：

1. 打开微软应用商店；
2. 搜索 Windows Terminal 并安装。

## 安装 oh-my-posh

**eg 1.** PowerShell 下命令行安装

1. 以管理员权限运行 PowerShell；
2. 执行以下命令，安装 posh-git（oh-my-posh 依赖）：

   ```powershell
   Install-Module posh-git -Scope CurrentUser
   ```

3. 执行以下命令，安装 oh-my-posh：

   ```powershell
   Install-Module oh-my-posh -Scope CurrentUser
   ```

注意：

* 安装 oh-my-posh 时将会提示正在从不受信任的存储库安装模块，选择 全是（A）即可。
* 出于网络原因，oh-my-posh 可能无法通过命令行完成下载（一直卡在初始状态），此时建议选择 eg 2 方法手动下载。

**eg 2.** Github 手动下载安装

1. Github 下进入 [oh-my-posh2](https://github.com/JanDeDobbeleer/oh-my-posh2) 项目主页，点击右方 Releases，下载最新版本 `oh-my-posh.zip`；
2. 在 `C:\Program Files\WindowsPowerShell\Modules` 目录下新建 `oh-my-posh` 文件夹，并将之前下载的 `oh-my-posh.zip` 压缩包解压到该文件夹。

## 安装依赖字体

由于 oh-my-posh 主题将会用到很多特有符号，而默认字体通常无法正常显示该符号（会显示乱码或方框），所以需要下载第三方支持其特有符号的字体（如：cascadia-code、Nerd 等）。

本教程选择使用 cascadia-code 字体。这是一种新的、有趣的等宽字体，包含了很多编程用符号，旨在增强 Windows 终端的现代化与外观。

cascadia-code 字体安装步骤：

1. Github 下进入 [cascadia-code](https://github.com/microsoft/cascadia-code) 项目主页，点击右方 Releases，下载最新版本 `CascadiaCode-2102.25.zip`（截至 2021年3月5日），并解压；
2. 选择安装 `CascadiaCodePL.ttf` 或 `CascadiaMonoPL.ttf` 字体。

>Windows 字体安装步骤:
>
>1. <kbd>win</kbd> + <kbd>Q</kbd> 搜索字体设置；
>2. 将待安装字体文件 `.ttf` 拖入到提示区域中完成安装。

切换字体：

Windows Terminal:

1. 点击上方状态栏下拉菜单，选择设置；
2. 在 "profiles" 下的 "defaults" 数据中添加：`"fontFace":"Cascadia Mono PL"`。

VS Code:

1. 打开 `settings.json`；
2. 添加数据：`"terminal.integrated.fontFamily": "Cascadia Mono PL"`。

IDEA:

1. 打开设置；
2. 搜索 Console Font；
3. 将 Font 修改为 `Cascadia Mono PL`。

## 设置主题

预览主题：

```powershell
Set-Theme 主题名
```

可通过 <kbd>Tab</kbd> 键自动补全切换。全部已有主题样式可在 oh-my-posh 项目 [README](https://github.com/JanDeDobbeleer/oh-my-posh2/blob/master/README.md) 中预览。

设置主题：

1. PowerShell 执行 `code $PROFILE`；
2. 在打开文件中，填写以下内容：

   ```ps1
   Import-Module posh-git
   Import-Module oh-my-posh
   Set-Theme themeName 
   ```

## 自定义主题

以下为该教程开头的自定义主题的配置方法：

1\. 在 `C:\Program Files\WindowsPowerShell\Modules\oh-my-posh\Themes` 目录下新建主题文件 `yourThemeName.psm1`，并将以下内容复制到该文件中：

```psm1
#requires -Version 2 -Modules posh-git

function Write-Theme {
    
    param(
        [bool]
        $lastCommandFailed,
        [string]
        $with
    )

    #check the last command state and indicate if failed
    If ($lastCommandFailed) {
        $prompt = Write-Prompt -Object "$($sl.PromptSymbols.FailedCommandSymbol) " -ForegroundColor $sl.Colors.CommandFailedIconForegroundColor
    }

    #check for elevated prompt
    If (Test-Administrator) {
        $prompt += Write-Prompt -Object "$($sl.PromptSymbols.ElevatedSymbol) " -ForegroundColor $sl.Colors.AdminIconForegroundColor
    }

    # Writes the drive portion
    $temp = Split-Path -leaf -path (Get-Location)
    $prompt += Write-Prompt -Object "$temp" -ForegroundColor $sl.Colors.DriveForegroundColor
    $prompt += Write-Prompt -Object ":" -ForegroundColor $sl.Colors.AccentColor

    $status = Get-VCSStatus
    if ($status) {
        $themeInfo = Get-VcsInfo -status ($status)
        $prompt += Write-Prompt -Object "git" -ForegroundColor $sl.Colors.PromptForegroundColor
        $prompt += Write-Prompt -Object ":" -ForegroundColor $sl.Colors.AccentColor
        $prompt += Write-Prompt -Object "$($themeInfo.VcInfo) " -ForegroundColor $themeInfo.BackgroundColor
    }

    # write virtualenv
    if (Test-VirtualEnv) {
        $prompt += Write-Prompt -Object 'env:' -ForegroundColor $sl.Colors.PromptForegroundColor
        $prompt += Write-Prompt -Object "$(Get-VirtualEnvName) " -ForegroundColor $themeInfo.VirtualEnvForegroundColor
    }

    if ($with) {
        $prompt += Write-Prompt -Object "$($with.ToUpper()) " -BackgroundColor $sl.Colors.WithBackgroundColor -ForegroundColor $sl.Colors.WithForegroundColor
    }

    # Writes the postfixes to the prompt
    $prompt += Write-Prompt -Object $sl.PromptSymbols.PromptIndicator -ForegroundColor $sl.Colors.CommandFailedIconForegroundColor
    $prompt += ' '
    $prompt
}

# local settings
$sl = $global:ThemeSettings 
$sl.PromptSymbols.PromptIndicator = [char]::ConvertFromUtf32(0x276F)
$sl.Colors.PromptForegroundColor = [ConsoleColor]::White
$sl.Colors.PromptSymbolColor = [ConsoleColor]::White
$sl.Colors.PromptHighlightColor = [ConsoleColor]::DarkBlue
$sl.Colors.WithForegroundColor = [ConsoleColor]::DarkRed
$sl.Colors.WithBackgroundColor = [ConsoleColor]::Magenta
$sl.Colors.AccentColor = [System.ConsoleColor]::DarkGray
```

2\. 设置主题（方法同上，主题名为新建主题文件的文件名 `yourThemeName`）。

主题效果：

![VS Code 效果展示](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/20220102145804.png)

主题特点：

* 省略全路径，仅保留当前所处文件夹信息。
* 可显示当前是否处在 Git 仓库中以及所处分支。
* 可显示当前 Git 仓库状态（分为暂存区、工作区，并依次显示增改删信息）。
* 可显示当前 Git 分支领先远程跟踪分支的 commit 数。

## 参考链接

* [将美化进行到底，把 PowerShell 做成 oh-my-zsh 的样子](https://blog.walterlv.com/post/beautify-powershell-like-zsh.html#%E5%AE%89%E8%A3%85%E5%AD%97%E4%BD%93%E5%AE%89%E8%A3%85%E7%AC%AC%E4%B8%89%E6%96%B9-powershell)
* [How to set up an awesome prompt with your Git Branch, Windows Terminal, PowerShell, + Cascadia Code!](https://www.youtube.com/watch?v=lu__oGZVT98)
