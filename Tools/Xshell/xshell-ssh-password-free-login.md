# Xshell 配置免密码登录

目录：

- [Xshell 配置免密码登录](#xshell-配置免密码登录)
  - [生成密钥对](#生成密钥对)
  - [服务器上安装公钥](#服务器上安装公钥)
  - [配置密钥登录功能](#配置密钥登录功能)
  - [Xshell 指定私钥](#xshell-指定私钥)
  - [参考链接](#参考链接)

## 生成密钥对

在本地电脑或者远程服务器输入以下命令生成密匙对：

```bash
ssh-keygen

# 输入保存文件名，直接 Enter 则为默认密钥名 id_rsa
Enter file in which to save the key (C:\Users\*****/.ssh/id_rsa): my_id_rsa # Windows

# 输入密码，直接 Enter 留空
Enter passphrase (empty for no passphrase):

# 再次输入密码，直接 Enter 留空
Enter same passphrase again:

# 生成公钥与私钥
Your identification has been saved in my_id_rsa. # 私钥
Your public key has been saved in my_id_rsa.pub. # 公钥
```

生成成功后，可在 `C:\Users\*****/.ssh/` 文件夹下（此为 Windows 系统，Linux 系统一般为 `/root/.ssh/`）找到生成的公钥（.pub）与私钥。

## 服务器上安装公钥

* 若选择在本地电脑生成密钥，则先将生成的 `my_id_rsa.pub` 公钥，上传到服务器；
* 然后将其添加到 `authorized_keys` 文件中。

```bash
cat ~/my_id_rsa.pub >> ~/.ssh/authorized_keys
```

## 配置密钥登录功能

1\. 打开 SSH 配置文件

```bash
sudo vim /etc/ssh/sshd_config
```

2\. 确认以下两项配置

```bash
RSAAuthentication yes
PubkeyAuthentication yes
```

> 默认不需要修改配置

3\. 禁用密码登录

```bash
PasswordAuthentication no
```

> **注意：** 该项设置需在密钥登录成功后修改，是否禁止密码登录根据个人需要而定。

4\. 重启 SSH

```bash
sudo service sshd restart
```

## Xshell 指定私钥

**Step 1.** 打开 Xshell，选择 文件 > 新建

![文件 > 新建](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/20220102150615.png)

**Step 2.** 填写名称、主机等信息

![填写信息](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/20220102150628.png)

**Step 3.** 选择 Public Key 方式验证

![选择验证方法](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/20220102150645.png)

**Step 4.** 指定私钥路径

![指定私钥路径](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/20220102150708.png)

## 参考链接

* [XShell 6 设置免密登录](https://blog.csdn.net/weixin_39534467/article/details/79359520)
