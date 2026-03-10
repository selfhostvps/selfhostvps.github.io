+++
title = "密码管理软件 Bitwarden / Vaultwarden"
date = "2026-03-08"
tags = [
    "security","service"
]
[params]
  no_toc = true
  no_date = true
+++

[Bitwarden](https://bitwarden.com/) 是著名的密码管理软件，而 Vaultwarden（曾经叫做 Bitwarden_RS）是一个著名的非官方的自建版本。——虽然 Bitwarden 也提供免费的[官方自建方案](https://bitwarden.com/help/self-host-bitwarden/)，但是需要更大的系统开销，和专门的数据库，更适合有大量用户同时使用的场景。Vaultwarden 使用 SQLite 数据库，开销很小，更适合少量用户使用，且支持 Bitwarden 官方的 APP。

注：官方的 Bitwarden 也提供免费的官方注册服务。但是，和收费版相比，免费账号不支持 2FA 服务，也不允许附加文件，而自建系统则不存在这些限制。

- Github：https://github.com/dani-garcia/vaultwarden
- 数据库依赖：内置 SQLite
- 中文版：有
- 开销：
	- CPU < 1%
	- 内存 < 100 MB
	- Docker Image：600 MB
	- 本地存储：<10 MB
- 当前最新版本：1.35.4
- 默认 docker 端口号：3001
- 本网站分配的端口号：5502

注意：

1. 为安全起见，建议在 docker 设置文件中，指定为这项服务的子域名，譬如 vaultwarden.example.com
2. 默认的配置是允许新用户注册的。可以在初次启动 docker 服务，注册用户后，停止 docker 服务，使配置文件中的 SIGNUPS_ALLOWED: false 生效，再次启动 docker，则不再允许新用户注册。
3. 密码管理软件的数据，**需要完善的备份方案！** 请参见数据备份篇（还没写）；
4. 更多配置，请看官方文档。

**docker-compose.yml**
```
services:
  vaultwarden:
    image: vaultwarden/server:1.35.4
    container_name: vaultwarden
    restart: unless-stopped
    environment:
      # 为这项服务指定的子域名
      DOMAIN: "https://vaultwarden.example.com"
      # 删除下一行的 #，不允许新用户注册
      # SIGNUPS_ALLOWED: false
    volumes:
      - ./vw-data/:/data/
    ports:
      - 127.0.0.1:8000:80
```



**在 Nginx conf 配置文件中添加：**
```
server {
    server_name vaultwarden.example.com;

    listen 443 ssl;
    ssl_certificate /ADMIN/https-certs/all.example.com.public.pem;
    ssl_certificate_key /ADMIN/https-certs/all.example.com.private.key;

	location / {
		# 基础代理设置
		proxy_pass http://localhost:5502/;
		
		# HTTP 版本和 WebSocket 支持
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection "upgrade";
		
		# 传递客户端信息
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header X-Forwarded-Proto $scheme;
		proxy_set_header X-Forwarded-Host $host;
		proxy_set_header X-Forwarded-Port $server_port;
		
		# 超时控制
		proxy_connect_timeout 60s;
		proxy_send_timeout 60s;
		proxy_read_timeout 60s;
		
		# 缓冲优化
		proxy_buffering on;
		proxy_buffer_size 4k;
		proxy_buffers 8 4k;
		proxy_busy_buffers_size 8k;
		
		# 其他优化
		proxy_redirect off;
		proxy_cache_bypass $http_upgrade;  # WebSocket 时不缓存
	}
}
```

