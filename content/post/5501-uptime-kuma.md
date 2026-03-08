+++
title = "监控工具 Uptime Kuma"
date = "2026-01-17"
tags = [
    "monitor"
]
[params]
  no_toc = true
  no_date = true
+++

Uptime Kuma 是一个开源免费的轻量级监控工具，可以随意添加要监控的网站，每隔一段时间，自动监测网站是否正常在线。

- 官网：https://uptimekuma.org/
- Github：https://github.com/louislam/uptime-kuma
- 数据库依赖：内置 SQLite，或专门的 MariaDB
- 中文版：有
- 开销：
	- CPU < 4%
	- 内存 120~150 MB
	- Docker Image：600 MB
	- 本地存储：1 MB
- 默认 docker 端口号：3001
- 本网站分配的端口号：5501

**docker-compose.yml**
```
services:
  uptime-kuma:
    image: louislam/uptime-kuma:2
    restart: unless-stopped
    volumes:
      - ./data:/app/data
    ports:
      - "5501:3001"
```

**在 Nginx conf 配置文件中添加：**
```
server {
    server_name monitor.example.com;

    listen 443 ssl;
    ssl_certificate /ADMIN/https-certs/all.example.com.public.pem;
    ssl_certificate_key /ADMIN/https-certs/all.example.com.private.pem;

	location / {
		# 基础代理设置
		proxy_pass http://localhost:5501/;
		
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

初次运行后，会询问你要使用的数据库，对于个人用户，选择 SQLite 就可以了。然后按提示创建管理员账号后，就可以使用。
