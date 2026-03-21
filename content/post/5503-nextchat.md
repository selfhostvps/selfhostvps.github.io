+++
title = "NextChat 个人 AI 浏览器界面"
date = "2026-03-08"
tags = [
    "AI"
]
[params]
  no_toc = true
  no_date = true
+++

[NextChat](https://github.com/chatgptnextweb/nextchat) 可以让你架设一个网站，在自己的网站上，使用各家 AI 的 API key，支持 Claude、ChatGPT、Gemini、Deepseek 等模型。

- Github：https://github.com/chatgptnextweb/nextchat
- 数据库依赖：无
- 中文版：有
- 开销：
	- CPU < 1%
	- 内存 < 100 MB
	- Docker Image：65 MB
	- 本地存储：0
- 当前最新版本：2.16.1
- 默认 docker 端口号：3000
- 本网站分配的端口号：5503

**docker compose 目录结构**

```
/DOCKERS/nextchat
└── docker-compose.yml # 配置文件
```

**docker-compose.yml**，需要在配置里明文写出你使用的 AI 公司的 API key，和访问你的网站的密码。除非 VPS 没有被人侵入，这些信息不会被看到。更多环境变量，可以参考[官网的说明](https://github.com/ChatGPTNextWeb/NextChat/tree/main#environment-variables)。

```
services:
  chatgpt-next-web:
    container_name: chatgpt-next-web
    image: yidadaa/chatgpt-next-web:v2.16.1
    restart: always
    ports:
      - 5503:3000
    environment:
      # 可以设置多个密码，用逗号分隔
      - CODE=password1,password2,password3
      # 各家的 API key
      - OPENAI_API_KEY=123456
      - GOOGLE_API_KEY=123456
      - AZURE_API_KEY=123456
      - ANTHROPIC_API_KEY=123456
      - DEEPSEEK_API_KEY=123456
      - BAIDU_SECRET_KEY=123456
      - BYTEDANCE_API_KEY=123456
      - ALIBABA_API_KEY=123456
      # 默认使用的模型
      - DEFAULT_MODEL=gpt-4o-mini
```

**在 Nginx conf 配置文件中添加：**

```
server {
    server_name nextchat.example.com;

    listen 443 ssl;
    ssl_certificate /ADMIN/https-certs/all.example.com.public.pem;
    ssl_certificate_key /ADMIN/https-certs/all.example.com.private.key;

	location / {
		# 基础代理设置
        proxy_pass http://localhost:5503;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header REMOTE-HOST $remote_addr;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;
        add_header X-Cache $upstream_cache_status;
        add_header Strict-Transport-Security "max-age=31536000";
        add_header Cache-Control no-cache;

        # 不缓存，支持流式输出
        proxy_cache off;  # 关闭缓存
        proxy_buffering off;  # 关闭代理缓冲
        chunked_transfer_encoding on;  # 开启分块传输编码
        tcp_nopush on;  # 开启TCP NOPUSH选项，禁止Nagle算法
        tcp_nodelay on;  # 开启TCP NODELAY选项，禁止延迟ACK算法
        keepalive_timeout 300;  # 设定keep-alive超时时间为65秒
	}
}
```

架设网站后，使用浏览器打开网站，在左下方的设置界面里，输入在 docker-compose.yml 中设定的密码，就可以一直在这个浏览器上，使用自己的 AI 了。——如果不是自己的电脑，使用后注意清空密码！
