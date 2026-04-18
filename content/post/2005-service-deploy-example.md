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

登录 Cloudflare（或者其它域名管理商的界面），选择要设置的域名，进入 DNS 页面，添加 Record

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

## 3. 申请 https / SSL 证书

1. 如果你一直按照本站的设定，

- 使用 cloudflare 管理解析域名
- 已经[获取了 cloudflare 的 15 年证书](/post/2004-cloudflare/)
- 在本文第 1 步设置 A 类记录时，选择了 Proxied（小黄云）

那么这一步可以直接跳过。

2. 如果不满足上述条件，那么，按照本站《[申请和使用 https / SSL 证书](/post/2007-https/#%E6%96%B9%E6%B3%95-2-%E4%BD%BF%E7%94%A8-certbot-%E4%B8%BA%E6%AF%8F%E4%B8%AA%E5%AD%90%E5%9F%9F%E5%90%8D%E5%8D%95%E7%8B%AC%E7%94%B3%E8%AF%B7%E8%AF%81%E4%B9%A6)》的步骤，为 monitor.example.com 申请单独的 https / SSL 证书。

并且，在下一步，设置 nginx 网站配置文件中，修改 SSL 证书的文件地址：

```
server {
    server_name monitor.example.com;

    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/monitor.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/monitor.example.com/privkey.pem;

	# 其它设置....
}
```

## 4. 设置 nginx

Nginx 的网站配置文件，默认位于 /etc/nginx/sites-available 可以选择把新网站的配置代码，

- 写入这个文件夹已有的配置文件中（譬如 default）；
- 也可以选择新建一个配置文件（文件名任意）。建议把功能或域名相似的站点的配置，放在同一个文件中，便于管理。

```
# 创建新的配置文件
sudo touch /etc/nginx/sites-available/new_website.conf

# 将新的配置文件，添加到 Nginx 的有效目录
sudo ln -s /etc/nginx/sites-available/new_website.conf /etc/nginx/sites-enabled/new_website.conf
```

编辑你选定的配置文件（现有的或新建的），将 [Uptime Kuma 帖子](/post/5501-uptime-kuma/)中 Nginx conf 的相关代码，复制到编辑页面中（可能需要修改其中的 https / SSL 证书文件地址），Ctrl+x 确认保存。

> 注意，每一个独立的站点，都是在一个独立的 server { } 内。复制配置代码时，不要复制到其它站点的 { } 内部！

```
# 编辑 Nginx 站点配置文件，Ctrl+x 确认保存
sudo nano /etc/nginx/sites-available/new_website.conf
```

刷新 Nginx 服务
```
sudo systemctl reload nginx
```

## 5. 完成

此时，访问 https://monitor.example.com ，就可以看到你部署好的网站服务了。

本站的大多数服务，都可以用这种方式部署。但每个服务都有各自的细微不同，譬如如何设置管理员账户。需要看每个服务各自的帖子，进行调整。
