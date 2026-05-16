+++
title = "Gitea，自建 git 服务器"
date = "2026-05-08"
tags = [
    "server","manage","git"
]
[params]
  no_toc = true
  no_date = false
+++

本篇教程适用于熟悉 git 操作的用户。关于如何使用 git / gitea，请参阅[其它教程](https://www.runoob.com/git/git-tutorial.html)。

[Gitea](https://about.gitea.com/products/gitea/) 是一款开源的、轻量级、自托管的 Git 代码托管平台，类似于 GitHub 或 GitLab。用户将代码或者文档托管在自己的服务器上，更好地保护了安全隐私。

- 官网：https://gitea.com/
- 数据库依赖：MySQL、PostgreSQL、SQLite
- 中文界面：有
- 开销：
	- CPU < 1%
	- 内存 < 200 MB
	- Docker Image：250 MB
	- 本地存储：0
- 当前最新版本：1.26.1
- 默认 docker 端口号：3000 & 22
- 本网站分配的端口号：5511 & 5512
- 备份重要性：高
- 备份方式：打包整个 docker compose 文件夹即可
	- 如果使用非 SQLite 的数据库：额外导出数据库记录

本文的安装配置，使用 SQLite 数据库，对于个人使用足够了。更复杂的配置，可以参见官网的 [docker 安装教程](https://docs.gitea.com/installation/install-with-docker)。

常见的 2 种使用 git 的方式：https 和 ssh。如果只用到 https，配置相对更简单一些。本文会在后面介绍使用 ssh 需要的额外操作。

**docker compose 目录结构**

```
/DOCKERS/gitea
├── gitea              # 数据文件夹
└── docker-compose.yml # 配置文件
```

**docker-compose.yml**

```
services:
  server:
    image: docker.gitea.com/gitea:1.26.1
    container_name: gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
    restart: always
    volumes:
      - ./gitea:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "5511:3000"
# 如果不需要通过 ssh 使用 git，可以省略下面的代码
      - "5512:22"
    networks:
      - network_expose
networks:
  network_expose:
    external: true
```

**Nginx conf 配置文件（[官网样例](https://docs.gitea.com/administration/reverse-proxies#nginx)）：**

假设为 gitea 分配的域名为 gitea.example.com

```
server {
    server_name gitea.example.com;

    listen 443 ssl;
    ssl_certificate /ADMIN/https-certs/all.example.com.public.pem;
    ssl_certificate_key /ADMIN/https-certs/all.example.com.private.key;

    location / {
        # 基础代理设置
        proxy_pass http://localhost:5511;

        client_max_body_size 512M;
        proxy_set_header Connection $http_connection;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

启动 docker compose 服务，重启 nginx 网络服务器。然后，从浏览器进入网站，**初次运行会进入设置界面**，重点修改下列选项：

- 数据库设置，如果使用 SQLite，使用默认设置即可
- Server Domain，选定的域名 gitea.example.com
- Gitea Base URL，主页地址，https://gitea.example.com/
- Optional Settings - Server and Third-Party Service Settings - Disable Self-Registration，禁止新用户注册
- Optional Settings - Administrator Account Settings，创建网址的管理员账号

然后点击 Install Gitea，安装后，自动进入管理员界面，就可以使用了。

更多配置，位于 docker compose 文件夹下的 gitea/gitea/conf/app.ini 文件，修改后重启 gitea（重启 docker compose）生效。

```
cd /DOCKERS/gitea
sudo docker-compose down
sudo nano ./gitea/gitea/conf/app.ini
sudo docker-compose up -d
```

### 使用 ssh 连接 git 服务器

#### 服务器端设置

在 ./gitea/gitea/conf/app.ini 配置文件中，设置 SSH_DOMAIN。默认的 SSH_DOMAIN 和 https 使用的 DOMAIN 相同（譬如 gitea.example.com）。这个 SSH_DOMAIN 唯一的作用，是在页面一键复制项目的 git 地址时，比较方便而已。可以将 SSH_DOMAIN 改成任何别名，只需要和下文客户端里配置的 Host 名称一致即可。

app.ini 里的相关配置，修改后重启 gitea 生效：

```
[server]
DOMAIN = gitea.example.com      # https 使用的域名
SSH_DOMAIN = hostname-sample      # ssh 使用的别名
HTTP_PORT = 3000
DISABLE_SSH = false             # 默认开启 SSH 功能
SSH_PORT = 22
SSH_LISTEN_PORT = 22
```

然后可以在网页里得到这样的 ssh 项目地址：

```
git@hostname-sample:repository-name.git
```

#### 客户端设置

从客户端访问 gitea 服务器的真实地址，可以使用下面几种方式：

- VPS 的 ip 地址，如 123.123.123.123
- 任何 A- 指向 VPS 的域名，但不要在 cloudflare 里勾选 Proxied
 
其实这两种方式都会暴露 VPS 的 ip 地址，所以在需要考虑隐私安全的场合，尽量在个人范围内谨慎使用。也有一些可以隐藏 VPS 真实 ip 的方式，譬如使用 Nginx 的 stream 模块，把另一个指向 443 端口的 Proxied 域名请求分流到 gitea 的 ssh 端口；或者使用 cloudflared 来隐藏 ip。但是都需要复杂的设置，这里就不多介绍了，如果不想向外人泄露 VPS ip 地址，还是建议只用 https 使用 gitea。

客户端 ssh 连接 gitea 的步骤（以 Linux 为例）：

1. 在客户端本地生成密钥，譬如

```
ssh-keygen -t rsa -f ~/.ssh/vps-gitea
```

2. 在 gitea 网页的 User Settings - SSH/GPG Keys 页面，添加 SSH key，复制上一步生成的公钥内容（ ~/.ssh/vps-gitea.pub ）

3. 编辑本地的 ~/.ssh/config 文件，添加：

```
Host hostname-sample              # 任意别名，最好和 gitea 的 SSH_DOMAIN 一致
    HostName 123.123.123.123      # VPS ip 或指向它的域名
    Port 5512                     # 本文 docker-compose.yml 里使用的 ssh 端口
    User git
    IdentityFile ~/.ssh/vps-gitea # 第 1 步生成的密钥
```

4. 可以用如下命令进行测试

```
ssh -T git@hostname-sample
```
