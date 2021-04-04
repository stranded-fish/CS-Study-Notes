# VS Code 远程开发环境配置

本文主要针对 **VS Code 远程开发环境配置** 的学习与总结。

目标是最终在本地搭建一个 VS Code 开发环境，能够在本地开发与调试远程服务器端 Linux C 程序。

目录：

- [VS Code 远程开发环境配置](#vs-code-远程开发环境配置)
  - [远程服务器配置](#远程服务器配置)
  - [本地主机配置](#本地主机配置)
  - [远程调试](#远程调试)
  - [参考链接](#参考链接)

## 远程服务器配置

以下命令针对 CentOS 操作系统。

**Step 1.** 安装 gdb

```bash
yum install gdb
```

**Step 2.** 安装 gdbserver

```bash
yum install gdb-gdbserver
```

**Step 3.** 确保指定端口已开放

* 确保 ssh 服务端监听端口已开放

  由于后续 VS Code 需要通过 ssh 协议连接服务器端，故需要保证 ssh 服务端监听端口正常开放。

  ssh 默认监听端口为 22，该端口通常情况下为开放状态。

* 确保后续 gdbserver 服务端监听端口已开放

  由于后续在服务器端启动 gdbserver 服务时，需要指定监听端口号（通常选择 2333 端口），故需要保证服务器已将该端口正常开放。

**注意：** 各大云厂商服务器（如：阿里云、腾讯云等）的防火墙配置默认仅开启 ssh 22 、http 80 以及 https 443 等极少数端口，**故大多数情况，在需要监听某端口时，应事先检查该端口的开放情况，若没有开放，则需手动开放。**

## 本地主机配置

**Step 1.** VS Code 配置 Remote - SSH 插件

**Step 1.1** 安装 Remote - SSH 插件

VS Code 应用商店搜索 Remote - SSH 插件，并安装

![搜索 Remote - SSH 插件](https://i.loli.net/2021/04/04/XWPHTxrwUMR5Il6.png)

**Step 1.2** 新建 SSH Host

* <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd> 打开命令面板；
* 输入 remote-ssh；
* 选择 Remote-SSH: Add New SSH Host；
  ![选择 Remote-SSH: Add New SSH Host](https://i.loli.net/2021/04/04/x3dHBuiUQtPK9qC.png)
* 输入 SSH Host 信息；
  ![输入 SSH Host](https://i.loli.net/2021/04/04/Q9voeXxGMspiqRw.png)
* 将新增 SSH Host 信息更新到本地 `config` 文件。
  ![更新本地 config 文件](https://i.loli.net/2021/04/04/tHzan76DejMmcSf.png)

如需修改之前新建的 SSH Host 信息，可打开之前保存更新的本地 `config` 文件，并进行修改即可。

**Step 2.** VS Code 登录 Remote Host

* <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd> 打开命令面板；
* 输入 remote-ssh；
* 选择 Remote-SSH: Connect to Host；
* 选择之前新建的 SSH Host 登录；

登录完成后，同本地访问一样，正常打开远程服务器工程文件即可。

**注意：** VS Code 上方会弹出输入框提示输入密码。可通过**设置公钥认证实现免密码登录。**

**Step 3.** VS Code 配置 C/C++ 插件

VS Code 应用商店搜索 C/C++ 插件，并选择在 `SSH: ***.***.***.***` 中安装。

**Step 4.** VS Code gdb 配置

点击左侧 debug 按钮，并选择创建 `launch.json` 文件，`launch.json` 文件会被新建到远程服务器工程文件中的 `.vscode` 目录中。

将其修改为以下内容：

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "gdb Remote Launch",
            "type": "cppdbg",
            "request": "launch",
            "program": "program_name",
            "args": ["arg1", "arg2", "arg3", "arg4"],
            "stopAtEntry": true,
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "gdb",
            "miDebuggerArgs": "gdb",
            "linux": {
                "MIMode": "gdb",
                "miDebuggerPath": "/usr/bin/gdb",
                "miDebuggerServerAddress": "***.***.***.***:2333",
            },
            "logging": {
                "moduleLoad": false,
                "engineLogging": false,
                "trace": false
            },
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "cwd": "${workspaceFolder}",
        }
    ]
}
```

属性说明：

* `"type": "cppdbg"` 调试器类型。若提示不支持该类型，可检查是否已经将 C/C++ 插件安装到服务器端。
* `"program": "program_name"` 调试的可执行程序。需保证能够访问到该可执行程序，如果没有将程序添加到 `/usr/bin` 或者设置环境变量，则需指定全路径，如 `/root/home/path/hello`
* `"args": ["arg1", "arg2", "arg3", "arg4"]` 程序运行参数。多个参数以 json 数组形式表示。
* `"miDebuggerServerAddress": "***.***.***.***:2333"` 远程服务器主机 IP 与 gdbserver 监听端口号。

## 远程调试

**Step 1.** 远程服务器准备

运行 gdbserver 服务。

```bash
gdbserver :2333 program_name arg1 arg2 arg3 arg4
```

**Step 2.** 本地主机准备

* VS Code 点击 debug 窗口中的绿色三角型 或 快捷键 <kbd>F5</kbd>，启动调试;
* 连接成功后，即可在本地 VS Code 使用调试功能，调试远端程序。

![调试截图](https://i.loli.net/2021/04/04/Q2nsV6gxr1KNobl.png)

## 参考链接

* [使用 VSCode 远程访问代码以及远程 GDB 调试](https://warmgrid.github.io/2019/05/21/remote-debug-in-vscode-insiders.html)
* [VS Code远程调试Linux C指南](https://zhuanlan.zhihu.com/p/98801522)
