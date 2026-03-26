
+++
title = "adminer，从网页访问和管理 vps 数据库"
date = "2026-03-21"
tags = [
    "admin","database"
]
[params]
  no_toc = false
  no_date = true
+++

[Adminer](https://www.adminer.org/)，曾经的 phpMinAdmin，是一个通过网页来访问本机数据库的工具。可以不登入 vps 的数据库命令行，就能执行一些基本的操作。但总体来说，更适合对于已经有一定数据库经验的人，作为便捷的辅助工具。对于新人，则很容易在各种界面选项中迷失，所以可能还是按照本站其它教程，在命令行界面复制粘贴命令，更靠谱一些。

- 官网：https://www.adminer.org/
- 数据库依赖：无
- 中文版：有
- 开销：
	- 本地存储：< 1 MB
- 当前最新版本：5.4.2
- （如果使用 docker ）默认 docker 端口号：8080 or 9000
- 本网站分配的端口号：5506

注意：使用 adminer 这类程序，增加了被人从网页侵入数据库的风险！请在明白此类风险的情况下谨慎使用！并且

- 相关的数据库密码（包括根用户密码和普通用户密码）应足够复杂；
- adminer 所在的网址，最好足够隐秘。譬如藏在已有网址的深层子路径下。

## 安装

Adminer 是一个单独的 .php 文件，所以，可以放在 VPS 外层网络服务器 Nginx 的任何支持 php 的已有网站的任何子文件夹下。当然，如果你在外层 Nginx 还完全没有任何网站，也可以使用和其它工具类似的方法，启动一个 docker 容器。

### 方法 1. 直接放在 vps 已有的支持 php 的网站里

如何让 Nginx 的网站支持数据库，请参见 [Nginx 和 php 的配置文章](/post/2003-install-nginx/)。

```
# 在任何已有网站的下创建任意子目录，用来放置 adminer 程序
mkdir -p /WEBSITES/any_website.example.com/any/path/to

wget -O /WEBSITES/any_website.example.com/any/path/to/index.php https://github.com/vrana/adminer/releases/download/v5.4.2/adminer-5.4.2.php
```

然后，就可以通过类似下面的网址，来访问 adminer 工具：

```
https://any_website.example.com/any/path/to/
```

### 方法 2. 使用 docker 容器

**docker compose 目录结构**

```
/DOCKERS/adminer       # docker compose 项目目录
└── docker-compose.yml # 配置文件
```

**docker-compose.yml**

```
services:
  vps-adminer:
    image: adminer
    restart: always
    ports:
      - 5506:8080
    container_name: vps-adminer
    networks:
      - network_database #加入预设的数据库共享网络
networks:
  network_database:
    external: true
```

**在 Nginx conf 配置文件中添加：**

```
server {
    server_name adminer.example.com;

    listen 443 ssl;
    ssl_certificate /ADMIN/https-certs/all.example.com.public.pem;
    ssl_certificate_key /ADMIN/https-certs/all.example.com.private.key;

    # 方法 2.1. 把整个网站的根目录作为 adminer 程序的入口
    # 如 https://adminer.example.com
    location / {
        proxy_pass http://127.0.0.1:5506/; 
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    # 方法 2.2. 在任何已有的网站配置中，用任意路径作为 adminer 程序的入口
    # 如 https://adminer.example.com/any/path/to
    location /any/path/to/ {
        proxy_pass http://127.0.0.1:5506/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    location = /any/path/to {
        return 301 $scheme://$host/any/path/to/;
    }
}
```

## 使用

登录后，选择

- 系统：数据库类型，MySQL/MariaDB 或 PostgreSQL
- 服务器：
	- 方法 1. adminer 部署在 VPS 网络服务器上，地址为：127.0.0.1（默认的 localhost 可能不起作用）
	- 方法 2. adminer 部署在 docker 容器里，按照本站的设定，地址为：
		- MySQL / MariaDB：vps-mariadb
		- PostgreSQL：vps-postgresql
- 用户名
- 密码
- 数据库：可以选择直接进入某个已有的数据库。如果是通过根用户（root 或 postgres）登入，用来创建其它数据库，这里留空即可。

可以通过 adminer 执行的操作，以 MySQL / MariaDB 为例：

- 创建数据库，
	- 点击创建的下一步，设置数据库名称，在 collation（校对）选项中，建议选择 utf8mb4_general_ci，获得更好的多语言支持
- 删除数据库
- 创建用户：Privilege（权限）菜单，设置用户名，在 Server（服务器）一栏填入 %
	- 点击编辑用户后，可以看到 Drop（删除用户）的操作
- 将某个数据库的权限授权给某个用户
	- 太复杂了，懒得写了……
- 查看、修改数据表
- 执行任意 SQL 语句

