+++
title = "申请和使用 https / SSL 证书"
date = "2026-04-08"
tags = [
    "https","nginx"
]
[params]
  no_toc = false
  no_date = true
+++

## 介绍. http vs https

在浏览器里输入的网站地址，按照安全性，分为 2 种（参见《[什么是 HTTPS？](https://www.cloudflare.com/zh-cn/learning/ssl/what-is-https/)》
）：

- http://example.com ，数据在网络传输时不加密，非常不安全，导致数据被窃取乃至更改；
- https://example.com ，加密传输，更加安全，需要申请专门的 SSL 证书。

现代网站通常都已经是 https 的模式，很多浏览器都会警告乃至默认拦截不安全的 http 网站。本站介绍的所有自建服务，都是更安全的 https 网站。

配置 https 网站，需要向第三方权威机构，申请 SSL 加密证书。本站介绍 2 种获取证书的方式：

**方法 1**. 可以从 cloudflare.com 一次性获取最高 15 年的免费 SSL 证书，应用于根域名和所有二级子域名（example.com & \*.example.com），不必每次生成新的网站，都要去申请 SSL 证书。

前提条件：域名必须托管在 Cloudflare 解析并开启 CDN（ 设置子域名的 A 记录时，开启 Proxied 小黄云）。

**方法 2**. 使用 certbot 工具，从 Let's Encrypt 机构获取并管理证书。每个网站在最初配置的时候，都需要专门申请一次证书。和第 1 种方法相比，稍微繁琐，适用性却更加广泛，所有指向 VPS 的子域名，无论是否通过 cloudflare 管理，无论是否使用 CDN proxied，都可以用这种方法申请 SSL 证书。

## 申请 https / SSL 证书

### 方法 1. 申请 cloudflare 15 年证书

参见本站《[cloudflare 配置](/post/2004-cloudflare/)》一文。只需操作一次，以后一直生效。（当然，15 年后要记得更新证书 ^^）

### 方法 2. 使用 certbot 为每个子域名单独申请证书

如果是初次使用，需要先安装 certbot：

```
# 安装 certbot
sudo snap install --classic certbot

# 设置命令的快捷方式
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

假设要添加的网站地址为 sub.example.com

1. 首先，在 cloudflare 或者其它域名商管理界面下，添加子域名的 A 记录到 VPS 的 ip 地址。可能要稍等一段时间（几分钟到一两个小时……），才能生效。如果 A 记录还没生效，接下来的命令会报错，稍候重新尝试运行即可；

2. 运行命令，生成 SSL 证书。如果是第一次运行 certbot，会要求输入 email（可回车跳过）

```
# 为 sub.example.com 域名申请 https 证书
sudo certbot certonly --nginx -d sub.example.com
```

然后 SSL 证书就申请好了。这样申请的证书，有效期有 90 天。VPS 会自动定时通过 certbot 检查每个证书是否到期，即将到期的证书会自动进行更新。

## 在 nginx 网络服务器中，配置 SSL 证书

用上面的 2 种方法得到的 SSL 证书，存放的地址是不同的。所以，在 Nginx 网络服务器的每个网站的配置文件中，会有细微的不同。

### 方法 1. cloudflare 15 年证书

配置文件如下，所有 example.com 的二级子域名，都使用同一份证书文件：

```
server {
    server_name sub.example.com;

    listen 443 ssl;
    ssl_certificate /ADMIN/https-certs/all.example.com.public.pem;
    ssl_certificate_key /ADMIN/https-certs/all.example.com.private.key;

	# 其它设置....
}
```

### 方法 2. 用 certbot 申请的证书

配置文件如下，每个网站的证书都是不同的文件：

```
server {
    server_name sub.example.com;

    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/sub.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/sub.example.com/privkey.pem;

	# 其它设置....
}
```
