+++
title = "一个基本的 Docker - Nginx 服务的样例"
date = "2026-01-17"
tags = [
    "service","docker"
]
[params]
  no_toc = true
  no_date = true
+++

这篇文章，以「[监控软件 Uptime Kuma](/post/5501-uptime-kuma/)」为例，介绍如何把一个 Docker 服务，架设到 VPS 上。

## 1. 为这个服务指定子域名

假设你为这个服务指定的子域名为 monitor.example.com

登录 Cloudflare，在域名配置文件中，进入 DNS 页面，添加 Record

- 类型 Type：A
- 子域名：monitor
- 地址：123.123.123.123 （你的 VPS 的 ip 地址）
- 推荐使用 Proxied

## 2. 创建 docker compose 服务

```
# 创建新的 docker compose 文件夹
mkdir -p ~/DOCKERS/uptime-kuma

# 进入创建的文件夹
cd ~/DOCKERS/uptime-kuma
```

编辑 docker compose 配置文件，将 [Uptime Kuma 帖子](/post/5501-uptime-kuma/)中 docker-compose.yml 下面的代码，复制到编辑页面中，Ctrl+x 确认保存。

```
# 编辑 docker compose 配置文件
nano docker-compose.yml
```

启动 docker compose 服务

```
# 启动 docker compose 服务
sudo docker compose up -d
```

## 3. 设置 nginx

Nginx 的网站配置文件，默认位于 /etc/nginx/sites-available 可以选择把新网站的配置代码，

- 写入这个文件夹已有的配置文件中（譬如 default）；
- 也可以选择新建一个配置文件（文件名任意）。建议把功能或域名相似的站点的配置，放在同一个文件中，便于管理。

```
# 创建新的配置文件
sudo touch /etc/nginx/sites-available/new_website.conf

# 将新的配置文件，添加到 Nginx 的有效目录
sudo ln -s /etc/nginx/sites-available/new_website.conf /etc/nginx/sites-enabled/new_website.conf
```

编辑你选定的配置文件（现有的或新建的），将 [Uptime Kuma 帖子](/post/5501-uptime-kuma/)中 Nginx conf 的相关代码，复制到编辑页面中，Ctrl+x 确认保存。

> 注意，每一个独立的站点，都是在一个独立的 server { } 内。复制配置代码时，不要复制到其它站点的 { } 内部！

```
# 编辑 Nginx 站点配置文件，Ctrl+x 确认保存
sudo nano /etc/nginx/sites-available/new_website.conf
```

刷新 Nginx 服务
```
sudo systemctl reload nginx
```

## 4. 完成

此时，访问 https://monitor.example.com ，就可以看到你部署好的网站服务了。

本站的大多数服务，都可以用这种方式部署。但每个服务都有各自的细微不同，譬如如何设置管理员账户。需要看每个服务各自的帖子，进行调整。
