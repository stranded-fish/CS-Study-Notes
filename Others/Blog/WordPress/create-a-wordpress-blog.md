# 创建一个 WordPress 博客

WordPress 是一个以 PHP 和 MySQL 为平台的自由开源的博客软件和内容管理系统。

该教程主要针对：WordPress 系统的 **环境准备**、**系统配置** 与 **安装**。

目录：

- [创建一个 WordPress 博客](#创建一个-wordpress-博客)
  - [系统准备](#系统准备)
  - [环境配置](#环境配置)
  - [WordPress 安装](#wordpress-安装)
    - [下载并解压 wordpress 安装包](#下载并解压-wordpress-安装包)
    - [创建数据库和用户](#创建数据库和用户)
    - [设置 wp-config.php](#设置-wp-configphp)
    - [上传文件](#上传文件)
    - [运行安装脚本](#运行安装脚本)
  - [参考链接](#参考链接)

## 系统准备

* 阿里云控制台：<https://swas.console.aliyun.com/>
* aliyun-轻量应用服务器

  * CPU：1 核
  * 内存：2 GB
  * 系统盘：40 GB SSD

* 系统镜像：Ubuntu 20.04
  * 重置系统后，登录控制台；
  * 选择服务器运维、远程连接；
  * 设置服务器 root 账号密码登录。

## 环境配置

在安装 Wordpress 之前，需要安装一些必须的软件，同时设置相关的远程访问支持。

需要主机支持的软件如下：

* Server 需支持 PHP 和 MySQL（建议：Apache 或 Nginx）。
* MySQL version 5.6 及以上 或 MariaDB version 10.1 及以上。
* PHP version 7.4 及以上。

以上软件通常会一起安装，合称为 **LAMP**（**L**inux operating system, **A**pache web server, **M**ySQL database, **P**HP），以使服务器能够托管用 PHP 编写的动态网站和 Web 应用程序。该站点数据存储在 MySQL 数据库中，动态内容由 PHP 处理。

LAMP 安装步骤如下：

* [安装 Apache 并更新 Firewall](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-20-04#step-1-%E2%80%94-installing-apache-and-updating-the-firewall)
* [安装 MySQL](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-20-04#step-2-%E2%80%94-installing-mysql)
* [安装 PHP](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-20-04#step-3-%E2%80%94-installing-php)
* [创建虚拟主机](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-20-04#step-4-%E2%80%94-creating-a-virtual-host-for-your-website)

相关问题整理：

* Apache:
  * [AH00558: Could not reliably determine the server's fully qualified domain name](https://www.digitalocean.com/community/tutorials/apache-configuration-error-ah00558-could-not-reliably-determine-the-server-s-fully-qualified-domain-name)
  未使用全局 ServerName 指令配置 Apache。
* MySQL:
  * [ERROR 2003 (HY000): Can't connect to MySQL server on 'XXX.XX.XX.XX' (110)](https://www.digitalocean.com/community/questions/error-2003-hy000-can-t-connect-to-mysql-server-on-xxx-xx-xx-xx-110)
  * [在Ubuntu/Linux环境下使用MySQL：开放/修改3306端口、开放访问权限](https://blog.csdn.net/freezingxu/article/details/77088506)
  MySQL 本地可连接，但无法远程连接 `mysql -h hostname -u root -p`。

## WordPress 安装

### 下载并解压 wordpress 安装包

* 在 wordpress 官网 [下载](https://cn.wordpress.org/download/) 安装包并上传到服务器。
* 登录到服务器，通过如下命令直接在服务器上下载并解压：
  * `wget https://wordpress.org/latest.tar.gz`
  * `tar -xzvf latest.tar.gz`

### 创建数据库和用户

登录 MySQL 为 WordPress 创建专有数据库并授予用户权限：

    授权格式：grant 权限 on 数据库.* to 用户名@登录主机 identified by "密码";

注意：由于从 MySQL 8 开始，不能够再（隐式）使用 GRANT 命令创建用户，故需改用 CREATE USER 命令，再使用 GRANT 命令：

```mysql
mysql> CREATE DATABASE databasename;

// 创建用户
mysql> CREATE USER 'root'@'%' IDENTIFIED BY 'password';

// 赋予该用户对 databasename 数据库的所有权限，若需要所有数据库权限，则改为 *.*
// '%' 表示对所有非本地主机授权，不包括 localhost，若需要包括 localhost，则需要单独授权
mysql> GRANT ALL PRIVILEGES ON databasename.* TO 'root'@'%' WITH GRANT OPTION;
mysql> FLUSH PRIVILEGES;
```

若创建用户时出现 `ERROR 1396 (HY000): Operation CREATE USER failed for 'XXXX'@'XXXX'` ，可通过以下方法进行排查：

* 首先查看是否存在该用户：

  ```mysql
  mysql> use mysql;
  mysql> select user, host from user;
  ```

* 若不存在该用户且上次删除过该用户，则可能没有刷新权限：

  ```mysql
  mysql> flush privileges;
  ```

* 若不存在该用户且刷新后仍然报错，则可以删除一次，再重新创建：

  ```mysql
  mysql> drop user 'root'@'hostname';
  mysql> flush privileges;
  ```

### 设置 wp-config.php

* 复制 wp-config-sample.php 文件到同级目录，并重命名为 wp-config.php；
* 修改其中的数据库配置信息：
  * define( 'DB_NAME', 'wordpress' );
  * define( 'DB_USER', 'root' );
  * define( 'DB_PASSWORD', 'passoword' );
  * define( 'DB_HOST', 'localhost' );
  * define( 'DB_CHARSET', 'utf8mb4' );
  * $table_prefix = 'wp_';

### 上传文件

将 wordpress 文件，上传到 Apache 服务器中的指定位置（默认为：/var/www/html，可通过创建虚拟主机修改），以决定域名显示样式：

* 存放到服务器根目录中。（<http://example.com/>）
* 存放到服务器子目录中。（<http://example.com/blog/>）

### 运行安装脚本

打开浏览器，访问安装域名，运行安装脚本。

* 根目录：
  <http://example.com/wp-admin/install.php>
* 子目录：
  <http://example.com/blog/wp-admin/install.php>

根据 web 安装指引，完成网站搭建。

## 参考链接

* [How to install WordPress](https://wordpress.org/support/article/how-to-install-wordpress/#detailed-instructions)
* [How To Install Linux, Apache, MySQL, PHP (LAMP) stack on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-20-04)
* [How to grant all privileges to root user in MySQL 8.0](https://stackoverflow.com/questions/50177216/how-to-grant-all-privileges-to-root-user-in-mysql-8-0)
* [mysql-创建用户报错ERROR 1396 (HY000)](https://blog.csdn.net/u011575570/article/details/51438841)
* [mysql-授予用户新建数据库的权限](https://blog.csdn.net/lindiwo/article/details/81708166)
