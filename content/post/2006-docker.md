
+++
title = "初始安装 2. Docker 的安装、设置、基本操作"
date = "2026-03-20"
tags = [
    "docker","setup","security"
]
[params]
  no_toc = false
  no_date = true
+++

## 安装 Docker

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

## Docker 暴露端口的问题

Docker 程序有一个很严重的问题：docker 对外暴露的端口号，会绕过 ufw 防火墙的规则。例如，即使防火墙禁止访问 5566 端口，但如果运行一个 docker 容器，对外映射了端口 5566:80，那么，这个容器可以从 vps 外部，通过 ip:5566 来访问，造成了严重的网络安全隐患。

以前这个问题解决起来比较复杂，如今有了更简单的解决方案是，将 docker 端口仅绑定到本地的回环 ip，即可。

```
    ports:
      - "127.0.0.1:3306:3306"  # 这样设置，外界无法访问 3306 端口
      - "3307:3307"            # 这样设置，外界可以访问 3307 端口
```

本站之后介绍的各种服务，会根据是否需要外界访问端口，而做好相应的设置。

## Docker 常用命令

待续
