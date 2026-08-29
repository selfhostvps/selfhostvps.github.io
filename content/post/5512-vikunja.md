+++
title = "Vikunja 任务管理系统"
date = "2026-08-21"
tags = [
    "office","manage"
]
[params]
  no_toc = true
  no_date = true
+++

[Vikunja](https://vikunja.io/)，是一个自建的任务管理服务。这样服务有很多，几乎所有的在线 office 套件，都有类似的服务：Microsoft To-Do、Google Tasks、或者独立的 any.do、todoist……但是几乎所有这类服务（包括 Vikunja），都不提供端到端加密，数据对于服务商都是可见的。所以，如果对日常任务的保密性要求比较高，自建的任务管理服务，成为自己的服务商，还是很有必要的。

- 官网：https://vikunja.io
- 数据库依赖：SQLite / PostgreSQL / MySQL
- 中文版：有
- 开销：
	- 内存：~ 30 MB
- 当前最新版本：2.5.0
- 本网站分配的端口号：5512
- 数据备份级别：高

## 安装

Vikunja 官网提供了非常详细的[自建安装指引](https://vikunja.io/install/)，根据不同的数据库、网络服务器、是否使用 docker……生成相应的配置文件。

**docker compose 目录结构**

```
/DOCKERS/vikunja       # docker compose 项目目录
├── files              # 储存附件的文件夹
├── root_pw.txt        # 储存 SQLite 数据库的文件夹
└── docker-compose.yml # 配置文件
```

按照官网的安装文档，需要为 docker compose 目录下的数据文件夹指定用户权限：

```
cd /DOCKERS/vikunja
mkdir files db
sudo chown 1000 files db
```

**docker-compose.yml** 配置文件

```
services:
  vikunja:
    image: vikunja/vikunja:2.5.0
    container_name: vps-vikunja
    user: 1000:1000
    environment:
    # 随机的长密钥，可以用命令 openssl rand -hex 32 生成
    VIKUNJA_SERVICE_JWTSECRET: 233ee0405171cd1e6bc21df719eff92fcbf4
    # 为 vikunja 分配的域名网址，要以 / 结尾
    VIKUNJA_SERVICE_PUBLICURL: https://vikunja.example.com/
    # 是否允许新用户注册
    VIKUNJA_SERVICE_ENABLEREGISTRATION: 0
    VIKUNJA_DATABASE_PATH: /db/vikunja.db
    ports:
    - 127.0.0.1:5512:3456
    volumes:
    - ./files:/app/vikunja/files
    - ./db:/db
    restart: unless-stopped
```

**在 Nginx conf 配置文件中添加：**

```
server {
    server_name vikunja.example.com;

    listen 443 ssl;
    ssl_certificate /ADMIN/https-certs/all.example.com.public.pem;
    ssl_certificate_key /ADMIN/https-certs/all.example.com.private.key;

    location / {
        proxy_pass http://localhost:5512;
        client_max_body_size 20M;
    }
```

## 使用

### 创建用户

自建的 vikunja 默认不开放新用户注册，要创建新用户，可以先更改 docker-compose.yml 里允许注册的参数，重启 docker 容器，在网页注册用户后，再把参数改回去，重启 docker 容器。

```
VIKUNJA_SERVICE_ENABLEREGISTRATION=1
```

更安全的方法，是通过 docker 内部的 vikunja 命令，直接创建用户（参见官方的[命令行文档](https://vikunja.io/docs/cli/#user-create)）：

```
sudo docker exec vps-vikunja /app/vikunja/vikunja user create --username user1 --email mail@example.com --password password1
```

所有用户都是普通的平级用户，除了使用命令行外，没有能够管理其它用户的管理员用户。更高级的管理员和用户群组功能，需要付费购买 [Vikunja Pro](https://vikunja.io/docs/pro/) License 才能使用。

### 手机 app

手机 app 有两种方案。一种是官方的 Vikunja，有苹果和安卓版本。但不能断网后脱机使用，这和直接使用浏览器也没区别了。

另一种方案是用第三方的 app，通过 CalDAV 接口传送任务信息，断网时也可以添加任务，网络恢复后自动同步。安卓上口碑很好的是 Tasks.org，可以从 F-Droid [免费下载](https://f-droid.org/en/packages/org.tasks/)（在 Google Play 发布的好像是内购收费版），在 vikunja 的【设置 - CalDAV】界面可以看到接口地址和 token 设置。缺点是不支持任务分类的多层结构。
