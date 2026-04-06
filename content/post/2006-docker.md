
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

然后进行——

### 本站的一些约定设置

```
# 创建 docker 网络 network_expose，提前占据内部 ip 地址 172.18.120.*
sudo docker network create --subnet=172.18.120.0/24 network_expose

# 将 network_expose 对应的 docker 容器 ip 加入防火墙允许范围
sudo ufw route allow proto tcp from any to 172.18.120.0/24
sudo ufw route allow proto udp from any to 172.18.120.0/24

# 创建 docker 网络 network_database，为需要数据库的容器共享内部网络
sudo docker network create network_database
```

关于预设 network_expose 的目的，详见后文：[让一些容器可以从外部访问](#%E8%AE%A9%E4%B8%80%E4%BA%9B%E5%AE%B9%E5%99%A8%E5%8F%AF%E4%BB%A5%E4%BB%8E%E5%A4%96%E9%83%A8%E8%AE%BF%E9%97%AE)

## Docker 暴露端口的问题

现有的 docker 程序的一个很严重的问题是：docker 对外暴露的端口号，会绕过 ufw 防火墙的规则。例如，即使防火墙禁止访问 5566 端口，但如果运行一个 docker 容器，对外映射了端口 5566:80，那么，这个容器可以从 vps 外部，通过 ip:5566 来访问，造成了严重的网络安全隐患。

在各种解决方案中，本站采用 [chaifeng 的方案](https://github.com/chaifeng/ufw-docker?tab=readme-ov-file#solving-ufw-and-docker-issues)，在 Ubuntu 24.04 上测试通过。编辑防火墙配置文件

```
sudo nano /etc/ufw/after.rules
```

在文件的末尾，添加：

```
# BEGIN UFW AND DOCKER
*filter
:ufw-user-forward - [0:0]
:ufw-docker-logging-deny - [0:0]
:DOCKER-USER - [0:0]
-A DOCKER-USER -j ufw-user-forward

-A DOCKER-USER -m conntrack --ctstate RELATED,ESTABLISHED -j RETURN
-A DOCKER-USER -m conntrack --ctstate INVALID -j DROP
-A DOCKER-USER -i docker0 -o docker0 -j ACCEPT

-A DOCKER-USER -j RETURN -s 10.0.0.0/8
-A DOCKER-USER -j RETURN -s 172.16.0.0/12
-A DOCKER-USER -j RETURN -s 192.168.0.0/16

-A DOCKER-USER -j ufw-docker-logging-deny -m conntrack --ctstate NEW -d 10.0.0.0/8
-A DOCKER-USER -j ufw-docker-logging-deny -m conntrack --ctstate NEW -d 172.16.0.0/12
-A DOCKER-USER -j ufw-docker-logging-deny -m conntrack --ctstate NEW -d 192.168.0.0/16

-A DOCKER-USER -j RETURN

-A ufw-docker-logging-deny -m limit --limit 3/min --limit-burst 10 -j LOG --log-prefix "[UFW DOCKER BLOCK] "
-A ufw-docker-logging-deny -j DROP

COMMIT
# END UFW AND DOCKER
```

保存文件后，重启 ufw 防火墙。但是，由于某些莫名的系统原因，可能要重启服务器才能生效。

```
# 重启防火墙
sudo systemctl restart ufw
# 或者直接重启服务器
sudo reboot
```

### 让一些容器可以从外部访问

本节的命令已经[在前面执行过了](#%E6%9C%AC%E7%AB%99%E7%9A%84%E4%B8%80%E4%BA%9B%E7%BA%A6%E5%AE%9A%E8%AE%BE%E7%BD%AE)，此处只是介绍原理。

进行上文的防火墙配置，解决了 docker 向外暴露端口的问题。但随之而来的问题是，一些我们本来就打算向外暴露端口的容器，如自建的 reality 翻墙程序、bt 下载程序……也统统不能都访问了。

按照 [chaifeng 的方案](https://github.com/chaifeng/ufw-docker?tab=readme-ov-file#solving-ufw-and-docker-issues)，可以使用类似

```
ufw route allow proto tcp from any to 容器内部ip port 容器内部端口
``` 

的指令，让 ufw 防火墙允许特定的容器。但每次都要获取或指定容器的 ip 信息，也很麻烦。所以本站预先创建 docker 网络 network_expose，预留一段 ip，以后把需要从外部访问的容器，都加入这个网络。

```
# 创建 docker 网络 network_expose，提前占据内部 ip 地址 172.18.120.*
sudo docker network create --subnet=172.18.120.0/24 network_expose

# 将 network_expose 对应的 docker 容器 ip 加入防火墙允许范围
sudo ufw route allow proto tcp from any to 172.18.120.0/24
sudo ufw route allow proto udp from any to 172.18.120.0/24

# 重启防火墙
sudo systemctl restart ufw
```

ps，因为本站教程是基于新安装的 vps，所以 172.18.120.* 的地址，应该还是空闲的。如果在创建 docker 网络时，遇到 Pool overlaps with other one on this address space 的报错信息，说明这段 ip 地址已经被占用，可以运行下面的命令，寻找空闲的 ip 地址。

```
sudo docker network inspect $(sudo docker network ls -q) --format '{{.Name}}: {{range .IPAM.Config}}{{.Subnet}}{{end}}'

```

## Docker 常用命令

待续
