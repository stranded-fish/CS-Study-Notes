# CentOS 防火墙与端口管理

本文主要针对 **CentOS 防火墙** 与 **端口** 管理等相关操作的总结。

目录：

- [CentOS 防火墙与端口管理](#centos-防火墙与端口管理)
  - [防火墙管理](#防火墙管理)
  - [端口管理](#端口管理)
  - [云服务器](#云服务器)
  - [参考链接](#参考链接)

## 防火墙管理

CentOS 7 中，引入了新服务 Firewalld，若没有默认安装，可通过命令 `yum install firewalld` 安装。

基本操作：

```bash
systemctl status firewalld        # 查看状态

systemctl start firewalld         # 启动防火墙
systemctl enable firewalld        # 设置开机启动（并不会启动当前已停止的防火墙）

systemctl stop firewalld          # 停止防火墙
systemctl disable firewalld       # 禁止开机启动（并不会停止当前已启动的防火墙）

systemctl restart firewalld       # 重启防火墙
```

若在开发中想彻底关闭防火墙、启动所有端口，可执行 `stop` 与 `disable` 命令，以彻底关闭防火墙并禁止开机启动。（若为云服务器还需关闭其提供的额外的云防火墙）

## 端口管理

```bash
firewall-cmd --list-ports                                       # 查看已开放端口
firewall-cmd --zone=public --add-port=3306/tcp --permanent      # 开放指定端口
firewall-cmd --zone=public --remove-port=3306/tcp --permanent   # 关闭指定端口
```

参数说明：

* `--zone` 作用域
* `--add-port=80/tcp` 添加端口，格式为：端口/通讯协议
* `--permanent` 永久生效，没有此参数重启后失效

**注意：** 开放/关闭指定端口后，必须通过命令 `systemctl restart firewalld` 或 `firewall-cmd --reload` 重启防火墙生效。

## 云服务器

各大云服务器厂商提供的云服务器，通常有自己的独立的云防火墙，**对服务器访问的请求，会先被云防火墙拦截，然后才会到达服务器本身的防火墙。**

故如果使用的是云服务器，在进行防火墙设置时，建议先登录对应的云服务器控制台，检查其防火墙设置是否与自身服务器防火墙设置冲突。

## 参考链接

* [Centos防火墙设置与端口开放的方法](https://blog.csdn.net/u011846257/article/details/54707864)
* [CentOS防火墙设置](https://juejin.cn/post/6844904116443938824)
* [Centos 7.x 防火墙端口管理](https://juejin.cn/post/6844903589257691149)
