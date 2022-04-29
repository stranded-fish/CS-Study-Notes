# Nginx 安装与配置

Nginx 是一个高性能的 Web Server，同时可以用作 反向代理、负载均衡 和 Web 缓存。

本文主要介绍在 CentOS 7 环境下 Nginx 的安装与配置。

目录：

- [Nginx 安装与配置](#nginx-安装与配置)
  - [安装 Nginx](#安装-nginx)
  - [配置 Nginx](#配置-nginx)
  - [操作 Nginx](#操作-nginx)
  - [参考链接](#参考链接)

## 安装 Nginx

**Step 1.** 检查软件源

```bash
yum list | grep nginx
```

可根据列表选择所需的 Nginx 版本安装，默认安装软件源最新版本。

**Step 2.** 安装 Nginx

```bash
yum install nginx
```

默认配置文件路径为：`/etc/nginx/nginx.conf`。

**Step 3.** 安装检验

```bash
nginx -version
```

## 配置 Nginx

**Step 1.** 修改配置文件 `/etc/nginx/nginx.conf`，强制 https 访问

```txt
server {
    listen       80;
    listen       [::]:80;
    server_name  your.site www.your.site;    # 网站域名（可以有多个，用空格分隔）
    rewrite ^(.*) https://$host$1 permanent; # 将 http 请求重定向到 https
}

server {
    listen       443 ssl http2;
    listen       [::]:443 ssl http2;
    server_name  your.site www.your.site;    # 网站域名（可以有多个，用空格分隔）
    
    # 设置响应资源路径（即网页静态内容）
    location / {
        root  /your/path/to/site;
    }

    ssl_certificate "/your/path/to/ssl_pem/********.pem";     # 设置 SSL pem 证书路径
    ssl_certificate_key "/your/path/to/ssl_key/********.key"; # 设置 SSL key 证书路径
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  5m;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    error_page 404 /404.html;
        location = /404.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}
```

关于 SSL 证书，可自行购买或申请免费证书，详见：[阿里云 SSL 证书](https://www.aliyun.com/product/cas)。

**Step 2.** 检查防火墙端口 `80` 与 `443`

```bash
firewall-cmd --list-ports                                     # 查看已开放端口
firewall-cmd --zone=public --add-port=80/tcp --permanent      # 开放指定端口
firewall-cmd --zone=public --remove-port=80/tcp --permanent   # 关闭指定端口
```

参数说明：

* `--zone` 作用域
* `--add-port=80/tcp` 添加端口，格式为：端口/通讯协议
* `--permanent` 永久生效，没有此参数重启后失效

**注意：**

* 开放/关闭指定端口后，必须通过命令 `systemctl restart firewalld` 或 `firewall-cmd --reload` 重启防火墙生效。
* 如果使用的是云服务器，在进行防火墙设置时，还需要登录对应的云服务器控制台，检查其防火墙设置是否与自身服务器防火墙设置冲突。

## 操作 Nginx

```bash
# 启动 Nginx
systemctl start nginx

# 停止 Nginx
systemctl stop nginx

# 重启 Nginx
systemctl restart nginx

# 查看 Nginx 状态
systemctl status nginx

# 启用开机自启动 Nginx
systemctl enable nginx

# 禁用开机自启动 Nginx
systemctl disable nginx
```

## 参考链接

* [Nginx - wiki](https://zh.m.wikipedia.org/wiki/Nginx)
