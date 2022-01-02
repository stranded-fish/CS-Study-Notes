# Git Bash 美化

该教程主要针对 **Git Bash 美化** 与 **风格修改**。

修改后样式：

![修改后样式](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/20220102143842.png)

目录：

- [Git Bash 美化](#git-bash-美化)
  - [主题修改](#主题修改)
  - [换行修改](#换行修改)
  - [参考链接](#参考链接)

## 主题修改

1. 打开 git bash；
2. 执行命令 `code ~/.minttyrc` 用 VS Code 打开用户根目录的 `.minttyrc` 文件，若没有该文件则新建（使用 notepad、vim 等其他编辑器均可）；
3. `.minttyrc` 文件内容填写如下：

```config
Font=Consolas
FontHeight=14

ForegroundColour=131,148,150
BackgroundColour=4,32,40
CursorColour=0,255,0

Black=4,32,40
BoldBlack=7,54,66
Red=128,128,0
BoldRed=203,75,22
Green=131,148,150
BoldGreen=88,110,117
Yellow=181,137,0
BoldYellow=101,123,131
Blue=38,139,210
BoldBlue=128,128,255
Magenta=221,54,130
BoldMagenta=108,113,196
Cyan=42,161,152
BoldCyan=147,161,161
White=238,232,213
BoldWhite=253,246,227
BoldAsFont=-1
FontSmoothing=full
FontWeight=700
FontIsBold=yes
Locale=C
Charset=UTF-8
CursorType=block
```

## 换行修改

Git Bash 运行时 `$` 符号将默认换行，可通过修改 `~/.bash_profile` 文件，统一为 Linux Bash 风格。

在 `~/.bash_profile` 文件中添加如下语句取消换行：

```.bash_profile
export PS1="\[\e[37;40m\]\[\e[32;40m\]\u\[\e[37;40m\]@\W\[\e[33;40m\]\$(__git_ps1 ['%s'])\[\e[32;40m\]\$\[\e[0m\] "
```

此外，还可以添加以下语句显示当前 Git 所处分支：

```.bash_profile
export PS1="\\u@windows:\w\$(__git_ps1 '(%s)')\$ "
```

## 参考链接

* [Git Bash 修改主题](https://blog.csdn.net/hustcw98/article/details/78979800)
* [git bash 界面修改成linux界面](https://www.cnblogs.com/heqiyoujing/p/10023084.html)
