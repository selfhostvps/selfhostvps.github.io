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

- Nginx 网络服务器
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

## 3. 安装 Docker

去看官网[安装文档](https://docs.docker.com/engine/install/ubuntu/)……通常新的机器，可以只执行 “Install using the apt repository” 的部分。

```
# 添加 Docker 官方密钥
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# 让自动安装工具识别 Docker
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update

# 安装 Docker
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

