+++
title = "在 Cloudflare 设置自己的域名（未完成）"
date = "2026-01-17"
tags = [
    "domain","cloudflare","https"
]
[params]
  no_toc = true
  no_date = true
+++

（未写完）

## 1. 购买域名

本站非常不建议从国内的提供商（阿里、腾讯……）购买域名，光是实名验证就非常麻烦，且未来不确定会不会对域名持有者进行更多的折腾，总之后患无穷。

NameSilo（高性价比）、Dynadot、Porkbun、Namecheap 和 GoDaddy

https://tld-list.com/tld/org

## 2. 注册 cloudflare 账号，添加你的域名

注册 cloudflare，添加你自己的域名，初期选择免费方案就足够了

在添加域名的过程中，根据 cloudflare 的提示，在域名注册商的管理界面，把 NS（Name Server）改成 cloudflare 提供的两个地址。

## 3. 生成 cloudflare 的十年 TLS / https 证书

登录 Cloudflare，在你的域名管理界面中，进入 SSL/TLS - Origin Server 界面。

点击 Create Certificate，在下一步菜单中，确认要生成证书的有效期（默认是最长的 15 年），点击生成。

下一个页面中，会显示生成证书的公钥（Origin Certificate / Public Key）和私钥（Private Key），

然后，把这两个密钥保持到 VPS 中，用什么名字无所谓，但之后会在 Nginx 的配置文件中，频繁引用这两个文件。本站约定使用下列文件名（可以将 example.com 改成你自己的域名）：

```
# 编辑公钥文件，粘贴后 Ctrl+x 保存，y 确认
nano /ADMIN/https-certs/all.example.com.public.pem

# 编辑私钥文件
nano /ADMIN/https-certs/all.example.com.private.key
```
## 4. 更改 Cloudflare 的 SSL/TLS 连接模式

登录 Cloudflare，在你的域名管理界面中，进入 SSL/TLS - Overview 界面。

默认的 SSL/TLS encryption 模式，应该是 Automatic (Flexible), 点击 Configure，点击 Custom SSL/TLS 一栏下的 Select，选择 Full 模式，确认保存。

