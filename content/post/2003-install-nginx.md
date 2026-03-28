+++
title = "VPS 初始安装设置：1. Nginx & Docker"
date = "2026-01-17"
tags = [
    "VPS","webserver","docker"
]
+++

本站向大家介绍的 VPS 管理方式，是——

- 在 VPS 上安装 Nginx 网络服务器和 php；
- 域名通过 Cloudflare 定向到 VPS；
- 使用 Cloudflare 自己的 https 证书；
- 所有其它服务，都使用 docker compose 运行在容器中，然后通过 Nginx，将每个 docker 容器的端口，对应到相应域名的 443 端口。

初期需要安装的软件包括

- Nginx 网络服务器 + php 脚本语言
- Docker 容器体系

## 1. 一些本站常用的路径设置

首先，为了以后写教程方便，在这里约定一些常用的文件夹的位置：在用户目录下创建几个文件夹，并把它们映射到根目录下。

使用上一篇创建的用户（而不是 root）登录服务器后，执行：

```
mkdir ~/ADMIN ~/WEBSITES ~/DOCKERS ~/tmp
sudo ln -s ~/ADMIN /ADMIN
sudo ln -s ~/WEBSITES /WEBSITES
sudo ln -s ~/DOCKERS /DOCKERS
mkdir ~/ADMIN/https-certs
```

- ~/ADMIN，管理用的程序，如 https 证书、日常维护用的脚本程序
- ~/WEBSITES，直接放在 Nginx网络服务器下的各个网站
- ~/DOCKERS，放置不同的 docker compose 项目

## 2. 安装 Nginx 网络服务器

```
# 安装 Nginx 网络服务器
sudo apt install nginx

# 可以用下面的命令，检查 Nginx 的状态是否正常 active (running)
sudo systemctl status nginx
```

此时，在浏览器里输入你的 ip 地址，应该就可以看到 "Welcome to nginx" 的欢迎页面了。（注意，ip 地址前面是 http，而不是 https）

```
# vps 的 ip 地址
http://123.123.123.123
```

## 3. 安装 php

PHP 是一个通用[开源的脚本语言](https://zh.wikipedia.org/wiki/PHP)，是很多网络项目使用的编程语言，如 Wordpress blog。因为 PHP 和普通 HTML 的嵌入搭配很好，可以被 Nginx 网络服务器直接调用，有时比放在 docker 容器中更加便利。 

本站目前安装的是 [php 8.4 的版本](https://www.php.net/supported-versions.php)，

```
# 在 VPS 的更新系统中加入 PHP 的订阅地址，执行后需要按回车键确认
sudo LC_ALL=C.UTF-8 add-apt-repository ppa:ondrej/php

# 更新系统
sudo apt update

# 安装 php 的基本功能
sudo apt install -f php8.4-cli php8.4-fpm

# 安装 php 的常用扩展支持
sudo apt install -f php8.4-common php8.4-{bcmath,bz2,curl,gd,gmp,intl,mbstring,opcache,readline,xml,zip}

# 安装 php 的数据库支持（也可以根据需要，只选择其中的一两项）
sudo apt install -f php8.4-{mysql,pgsql,sqlite3}
```

### 设置从 php 网页上传文件的最大限制

在 php 的配置文件 /etc/php/8.4/fpm/php.ini 里，默认只能上传最大 2M 的文件

```
upload_max_filesize = 2M
post_max_size = 8M
```

可以把这两个参数修改成 200M，或者任何你希望的数值。可以手动修改上面的配置文件，也可以用下面的命令自动更改：

```
sudo sed -i 's/^\(upload_max_filesize\s*=\s*\).*/\1100M/' /etc/php/8.4/fpm/php.ini
sudo sed -i 's/^\(post_max_size\s*=\s*\).*/\1100M/' /etc/php/8.4/fpm/php.ini

# 修改后，重启 php 服务，才会生效
sudo systemctl restart php8.4-fpm.service
```

### 更换默认的 php 版本

VPS 内可同时安装多个版本的 php，可以执行下面的命令，切换默认的 php 版本：

```
sudo update-alternatives --config php
```

## 4. 安装 Docker

此处内容已挪至专门的一篇，[关于 docker 的更详细的安装和设置](/post/2006-docker/)。
