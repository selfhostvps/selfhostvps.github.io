+++
title = "在 vps 部署静态网站：Hugo、JekyII、Hexo……"
date = "2026-03-28"
tags = [
    "nginx","blog"
]
[params]
  no_toc = false
  no_date = true
+++

把各种静态网站生成器（[Hugo](https://gohugo.io/)、[Jekyll](https://jekyllrb.com/)、[Hexo](https://hexo.io/)……）生成的网站，部署到 vps 上，大致有 3 种方法：

1. 最简单地，在你们自己的电脑上运行静态生成器，然后，使用 [SFTP 客户端](/post/2002-vps-linux-commands/#%E9%80%9A%E8%BF%87-sftp-%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%92%8C-vps-%E4%BA%92%E4%BC%A0%E6%96%87%E4%BB%B6)，把生成的 html 静态网站，复制到 vps 的 Nginx 网络服务器的对应网站的文件夹。
2. 在 vps 上创建整个项目，使用 SFTP 客户端复制最新的文章，在 vps 上生成新的网站。
3. 在 github 之类的托管网站上创建项目，更新文章，然后使用 github action 自动发布到 vps。——这种方法配置过于繁琐，安全风险也更高，通常有能力做到这个层级的用户，也没必要这样做了，建个 wordpress 更加方便……所以本站不介绍这种方式了。

以 Hugo 为例，

## 方法 1. 在本机创建 hugo 项目，把生成的静态网站复制到 vps

也可以在你们自己的电脑上创建、维护 hugo 项目，然后把生成的静态网站 public/ 直接复制到 vps 的网站文件夹位置，而不是在 vps 上运行 hugo。

**在 Nginx conf 配置文件中添加：**

```
server {
    server_name hugo.example.com;      # 网站域名
    root /WEBSITES/hugo.example.com/;  # 网站所在文件夹

    listen 443 ssl;  # https 证书所在位置
    ssl_certificate /ADMIN/https-certs/all.example.com.public.pem;
    ssl_certificate_key /ADMIN/https-certs/all.example.com.private.key;

    index index.html index.htm;
}
```

## 方法 2. 在 VPS 服务器上创建 hugo 项目

```
# 在 VPS 安装 hugo
sudo apt install -y hugo

# 进入本站设定的，存放网站的文件夹
cd /WEBSITES

# 创建 hugo 项目
hugo new site hugo.example.com

# ......然后你们应该都知道 hugo 怎么玩吧？？
# 添加主题、文章什么的......
```

每次发布文章时，使用 [SFTP 客户端](/post/2002-vps-linux-commands/#%E9%80%9A%E8%BF%87-sftp-%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%92%8C-vps-%E4%BA%92%E4%BC%A0%E6%96%87%E4%BB%B6)，把 content/ 里的新文章，复制到 vps 的相应目录里，然后运行 hugo 命令，生成新的静态网站。

```
cd /WEBSITES/hugo.example.com
hugo
```

**在 Nginx conf 配置文件中添加：**

```
server {
    server_name hugo.example.com;      # 网站域名
    root /WEBSITES/hugo.example.com/public/;  # 网站所在文件夹

    listen 443 ssl;  # https 证书所在位置
    ssl_certificate /ADMIN/https-certs/all.example.com.public.pem;
    ssl_certificate_key /ADMIN/https-certs/all.example.com.private.key;

    index index.html index.htm;
}
```

