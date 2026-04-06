+++
title = "Wordpress 博客系统"
date = "2026-03-31"
tags = [
    "blog","docker","php","mysql"
]
[params]
  no_toc = false
  no_date = true
+++

[Wordpress](https://wordpress.org/) 大概是最常用的 blog 系统了。（省略介绍若干）

本篇涉及的前置知识：

- [新购买 VPS 的初始登录和安全配置](/post/2001-new-vps-setup/)
- [购买域名、Cloudflare 配置](https://selfhostvps.github.io/post/2004-cloudflare/)
- [初始安装 1. 网站服务器 Nginx & PHP](https://selfhostvps.github.io/post/2003-install-nginx/)
- [初始安装 2. Docker 的安装、设置、基本操作](https://selfhostvps.github.io/post/2006-docker/) 
- [数据库安装：MySQL / MariaDB](https://selfhostvps.github.io/post/3306-mysql-mariadb/)

## 安装 Wordpress 的三种方式

### 1. 直接放在 VPS 的目录下

#### 创建数据库

参照[本站数据库管理一节的内容](/post/3306-mysql-mariadb/#%E8%BF%9B%E5%85%A5-mariadb%E5%88%9B%E5%BB%BA%E6%95%B0%E6%8D%AE%E5%BA%93%E5%88%9B%E5%BB%BA%E7%94%A8%E6%88%B7)，为 wordpress 创建一个新的数据库

```
# 进入 MariaDB 数据库界面
sudo docker exec -it vps-mariadb sh -c 'mariadb -u root -p"$(cat $MARIADB_ROOT_PASSWORD_FILE)"'

# 在数据库提示符下，创建数据库 db_wordpress_1
CREATE DATABASE db_wordpress_1 CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

# 创建数据库用户 user_wordpress_1，如果要使用已有的用户，这一步可以跳过
CREATE USER 'user_wordpress_1'@'%' IDENTIFIED VIA mysql_native_password USING PASSWORD('password_1');

# 将新建数据库的使用权限授予用户
GRANT ALL ON db_wordpress_1.* TO 'user_wordpress_1'@'%';

# 刷新权限
FLUSH PRIVILEGES;

# 退出 MariaDB 数据库
QUIT;
```

#### 下载 wordpress 程序

- 假设，网站域名为 wordpress.example.com
- 放置网站的文件夹为 /WEBSITES/wordpress.example.com/

```
# 确认临时文件夹不存在
rm /tmp/wordpress -rf

# 从官网下载最新的 wordpress 程序
curl -o /tmp/wordpress.zip https://wordpress.org/latest.zip

# 解压程序包
7z x /tmp/wordpress.zip -o/tmp/

# 移动 wordpress 程序到网站位置
mv /tmp/wordpress /WEBSITES/wordpress.example.com
```

#### 配置 Nginx 网络服务器

**在 Nginx conf 配置文件中添加：**

```
server {
    server_name wordpress.example.com;      # 网站域名
    root /WEBSITES/wordpress.example.com/;  # 网站所在文件夹

    listen 443 ssl;  # https 证书所在位置
    ssl_certificate /ADMIN/https-certs/all.example.com.public.pem;
    ssl_certificate_key /ADMIN/https-certs/all.example.com.private.key;

    index index.php;
    location / {
        try_files $uri $uri/ /index.php?$args;
    }
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.4-fpm.sock;
    }
}
```

#### 在浏览器初始化 wordpress

在浏览器打开 https://wordpress.example.com ，进入初始设置页面，点击 Let's go，输入之前创建的数据库的信息：

- Database Name : db_wordpress_1
- Username : user_wordpress_1
- Password : password_1
- Database Host : 127.0.0.1
- Table Prefix: wp_

点击 Submit 提交后，选择网站的界面语言，然后输入网站的管理信息（这些都可以在建站后，在管理界面随意更改）

- 标题
- 管理员账户名
- 管理员密码
- 管理员 email
- 是否允许网站被搜索引擎收录

然后提交，wordpress 网站就建好了。

### 2. 使用 docker，但是使用 vps 统一的数据库

待补充。

### 3. 使用包含自身数据库的 docker

这里是一个最快的，用 docker 创建一个 wordpress 站点的步骤。如果你只想临时建个站测试一下，这样会更方便。数据库用的也是独立的，和 vps 原有的数据库并无关联。


- 数据库依赖：MySQL / MariaDB
- 中文版：有
- 开销：
	- 内存：~ 220 MB
		- wordpress：75 MB
		- MariaDB：145 MB
	- Docker Image：~ 420 MB
		- wordpress：260 MB
		- MariaDB：160 MB
	- 本地存储：210 MB（不包括用户使用 wordpress 时添加的媒体文件）
- 当前最新版本：6.9.4
- 默认 docker 端口号：3000
- 本网站分配的端口号：5505

**docker compose 目录结构**

```
/DOCKERS/wordpress_5505  # docker compose 项目目录
├── database-data        # 数据库存储文件夹
├── wordpress            # wordpress 存储文件夹
└── docker-compose.yml   # 配置文件
```

**docker-compose.yml**

```
services:

  wordpress:
    image: wordpress
    restart: always
    ports:
      - 5505:80
    environment:
      WORDPRESS_DB_HOST: wordpress_db
      WORDPRESS_DB_USER: example_user
      WORDPRESS_DB_PASSWORD: example_user_password
      WORDPRESS_DB_NAME: example_database
    volumes:
      - ./wordpress:/var/www/html
    depends_on:
      - wordpress_db

  wordpress_db:
    image: mariadb:11.4-ubi
    restart: always
    environment:
      - MARIADB_ROOT_PASSWORD=expmple_root_password
      - MARIADB_DATABASE=example_database
      - MARIADB_USER=example_user
      - MARIADB_PASSWORD=example_user_password
    volumes:
      - ./database-data:/var/lib/mysql
```

**在 Nginx conf 配置文件中添加：**

```
server {
    server_name wordpress.example.com;

    listen 443 ssl;
    ssl_certificate /ADMIN/https-certs/all.example.com.public.pem;
    ssl_certificate_key /ADMIN/https-certs/all.example.com.private.key;

    location / {
        proxy_pass http://localhost:5505;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

然后在浏览器打开 https://wordpress.example.com ，会跳过输入数据库信息的部分，而直接进入选择语言和管理员信息的页面。
