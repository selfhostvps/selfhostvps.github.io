+++
title = "几种自建翻墙的方案（高级篇）"
date = "2026-04-10"
tags = [
    "vps"
]
[params]
  no_toc = false
  no_date = true
+++

## 介绍

本文介绍一些自建梯子翻墙的方式。和之前的《[自建翻墙 - 新手篇](/post/3001-selfhost-gfw/)》不同，

- 《[新手篇](/post/3001-selfhost-gfw/)》的目的，是让缺乏技术经验的新人，通过尽量简洁的操作，购买 VPS 自建翻墙服务，VPS 并不做其它用途。
- 本文《高级篇》，让大家在使用 VPS 运行各种自建服务的同时，顺便在同一台 VPS 上翻墙。
	- 按照本站介绍的各种服务的统一设置，本文介绍的翻墙服务，都是运行在 docker compose 的容器内，（如果需要的话）外层用 Nginx 通过已经配置好 https 证书的域名进行反向代理。

### 几种梯子的比较

翻墙的技术有很多种，其中也有很多种已经被封杀了。目前比较好用的，本文主要介绍 2 种。

- Reality 协议，由 [xray-core 团队开发](https://github.com/XTLS/REALITY)
	- 特点：无需域名和 https 证书，省略了很多麻烦的步骤
	- 缺点（或者说 不便之处）：会在翻墙账号里，暴露服务器的 ip 地址。因此，更适合自用或者小规模私密分享，不利于大规模的传播。
- WebSocket (WS) 技术，通过伪装成标准网页浏览流量，能有效绕过 GFW 检测。
	- 需要一个没有被封禁的子域名，或者服务商为 VPS 配置的子域名（但这种域名很可能已经被封了）；
	- 支持 Cloudflare 等 CDN，可以隐藏 VPS 的真实 ip 地址，但网速可能会因此受影响。

### 本篇涉及的前置知识

如果采用 reality 方式翻墙，只需了解：

- [新购买 VPS 的初始登录和安全配置](/post/2001-new-vps-setup/)
- [初始安装 2. Docker 的安装、设置、基本操作](https://selfhostvps.github.io/post/2006-docker/) 

如果采用 Websocket 方式翻墙，还需了解：

- [购买域名、Cloudflare 配置、https 证书](https://selfhostvps.github.io/post/2004-cloudflare/)
- [初始安装 1. 网站服务器 Nginx & PHP](https://selfhostvps.github.io/post/2003-install-nginx/)
- [一个基本的 Docker - Nginx 服务的样例](https://selfhostvps.github.io/post/2005-service-deploy-example/)

## 使用 Reality 自建翻墙

推荐使用 [wulabing 封装的 docker 容器](https://github.com/wulabing/xray_docker)，省略了手动设置配置文件的繁琐过程。

- Github：https://github.com/wulabing/xray_docker
- 数据库依赖：无
- 中文版：无
- 开销：
	- CPU < 1%
	- 内存 < 20 MB
	- Docker Image：80 MB
	- 本地存储：0
- 当前最新版本：0.0.34
- 默认 docker 端口号：443
- 本网站分配的端口号：5507

**docker compose 目录结构**

```
/DOCKERS/xray_reality_5507
└── docker-compose.yml # 配置文件
```

**docker-compose.yml**

“#” 下方的代码，是按照本站 docker 篇的设定，[修复了 docker 暴露端口的问题](/post/2006-docker/#docker-%E6%9A%B4%E9%9C%B2%E7%AB%AF%E5%8F%A3%E7%9A%84%E9%97%AE%E9%A2%98)后，专门添加的对外暴露端口的网络。如果你的 vps 没有进行这方面的设置，可以直接忽略 # 下面的代码。

```
services:
  xray_reality_5507:
    image: wulabing/xray_docker_reality:latest
    container_name: xray_reality_5507
    restart: always
    ports:
      - 5507:443
    environment:
      - EXTERNAL_PORT=5507
# --- 以下的代码，让容器处于可以向外暴露端口的 docker 网络 ---
    networks:
      - network_expose
networks:
  network_expose:
    external: true
```

启动 docker compose 后，允许下面的命令，就可以看到生成的 vless reality 翻墙账号了。

```
sudo docker exec -it xray_reality_5507 cat /config_info.txt
```

## 使用 WebSocket (WS) 自建翻墙

推荐使用 [Project X 封装的 docker 容器](https://github.com/xtls/xray-core)，这个 docker 容器也可以用于 Reality 翻墙，但是配置过程更繁琐一些。本文参照了 [Leohoo 的教程](https://blog.leoho.dev/posts/docker-compose-deploy-xray/)。

- Github：https://github.com/xtls/Xray-core/pkgs/container/xray-core
- 数据库依赖：无
- 中文版：无
- 开销：
	- CPU < 1%
	- 内存 < 20 MB
	- Docker Image：70 MB
	- 本地存储：0
- 当前最新版本：26.4.15
- 默认 docker 端口号：自定义
- 本网站分配的端口号：5508

**docker compose 目录结构**

```
/DOCKERS/xray_core_5508
├── config.json        # 账号配置文件
└── docker-compose.yml # docker 配置文件
```

首先，运行下面的命令，生成随机的用户账号（uuid：类似 12345678-1234-1234-1234-123456789012 的 32 位字符）。

```
# 生成 uuid
sudo docker run --rm ghcr.io/xtls/xray-core:latest uuid
```

创建 **config.json** 配置文件。注意替换下面代码里的：

- uuid 账号
- 端口号
- /yourpath ：隐藏在域名下方的路径
	- 你的 any.example.com/yourpath 网址，没有其它用途
	- 需要和后面 Nginx 的配置文件一致
	- 最好不要用 /vpn 这种特征太明显的名字

**config.json**

```
{
  "log": {
    "loglevel": "warning"
  },
  "inbounds": [
    {
      "listen": "0.0.0.0",
      "port": 5508,
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "替换成你生成的 uuid"
          }
        ],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "ws",
        "security": "none",
        "wsSettings": {
          "path": "/yourpath"
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ],
  "dns": {
    "servers": [
      "1.1.1.1",
      "8.8.8.8",
      "223.5.5.5"
    ]
  }
}
```

**docker-compose.yml**

“#” 下方的代码，是按照本站 docker 篇的设定，[修复了 docker 暴露端口的问题](/post/2006-docker/#docker-%E6%9A%B4%E9%9C%B2%E7%AB%AF%E5%8F%A3%E7%9A%84%E9%97%AE%E9%A2%98)后，专门添加的对外暴露端口的网络。如果你的 vps 没有进行这方面的设置，可以直接忽略 # 下面的代码。

```
services:
  xray_core_5508:
    image: ghcr.io/xtls/xray-core:latest
    container_name: xray_core_5508
    restart: unless-stopped
    command: ["run", "-c", "/usr/local/etc/xray/config.json"]
    volumes:
      - ./config.json:/usr/local/etc/xray/config.json:ro
    ports:
      - "5508:5508/tcp"
# --- 以下的代码，让容器处于可以向外暴露端口的 docker 网络 ---
    networks:
      - network_expose
networks:
  network_expose:
    external: true
```

在 Nginx 的任何已有域名（也可以新分配一个域名）的配置文件内部，添加

```
server {
    server_name any.example.com;
    
    listen 443 ssl;
    ssl_certificate /ADMIN/https-certs/all.example.com.public.pem;
    ssl_certificate_key /ADMIN/https-certs/all.example.com.private.key;
# ... 原有的 nginx 配置文件 ... 

# 添加的 Web Socket 配置
# 注意：和 config.json 里的 /your-path 一致

    location /yourpath {
        proxy_redirect off;
        proxy_pass http://127.0.0.1:5508;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;

        # Show realip in v2ray access.log
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
}
```

启动 docker compose，重启 nginx，然后生成的 Web socket 翻墙地址，格式如下：

```
vless://12345678-1234-1234-1234-123456789012@any.example.com:443?encryption=none&security=tls&type=ws&path=%2Fyourpath#any-account-name
```

替换其中的

- uuid
- 域名，443 不要变
- yourpath （前面的 %2F 就是 / 的替换符）
- any-account-name，账号标题，随意
