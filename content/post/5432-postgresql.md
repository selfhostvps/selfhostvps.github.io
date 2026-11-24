
+++
title = "数据库 PostgreSQL"
date = "2026-10-16"
tags = [
    "docker","database","backup"
]
[params]
  no_toc = false
  no_date = true
+++

## PostgreSQL 介绍

和 MariaDB / MySQL 一样，PostgreSQL（或者叫 Postgres）也是常用的关系性数据库之一，被一些常用的 VPS 服务使用（如 Miniflux、Mastodon…）。现有的 PostgreSQL 项目创建于 1996 年，作为独立的开源项目被长期维护。

## PostgreSQL 的安装

**docker compose 目录结构**

```
/DOCKERS/mariadb       # docker compose 项目目录
├── data               # 本地数据文件
└── docker-compose.yml # 配置文件
```

按照本站 [docker 一文的设定](/post/2006-docker/#%E6%9C%AC%E7%AB%99%E7%9A%84%E4%B8%80%E4%BA%9B%E7%BA%A6%E5%AE%9A%E8%AE%BE%E7%BD%AE)，预先创建 docker 网络，让其它容器和外部网站，共享同一个数据库系统。

```
# 如果还没有运行过的话，先创建 docker 网络，为需要数据库的容器共享内部网络
sudo docker network create network_database
```

PostgreSQL 没有特定的长期稳定版本（Long Term Support），而是每个大版本有五年的[支持期](https://endoflife.date/postgresql)。

**docker-compose.yml**

```
services:
  vps-postgres:
    image: postgres:18
    container_name: vps-postgres
    restart: always
    # set shared memory limit when using docker compose
    shm_size: 128mb # 或者 256mb
    environment:
      POSTGRES_PASSWORD: example_password
    volumes:
      - ./data:/var/lib/postgresql
    networks:
      - network_database # 加入预设的数据库共享网络
#   ports:
#     - "127.0.0.1:5432:5432"
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "postgres"]
networks:
  network_database:
    external: true
```

注意：
- POSTGRES_PASSWORD 的密码是必须的，可以设置的复杂一些。和 MariaDB 不同，PostgreSQL 在使用 docker 进行管理和维护时，基本不会在命令行输入密码，所以对个人用户而言，也不是很需要用专门的 secret 来隐藏密码。
- 默认的超级用户名为 postgres，也可以用 POSTGRES_USER 参数自行指定。
- 端口号（ports）的部分，暂时注释掉了。因为通常不会有什么机会，从容器外部访问 PostgreSQL（譬如使用非 docker 的 adminer 管理界面）。如果需要的话，可以把注释去掉。

## PostgreSQL 数据库的使用，创建数据库，创建用户

进入 psql 命令行管理页面的方式如下。退出 psql 的命令为 `\q 然后 Enter` ，或者直接按 `Ctrl d`

```
sudo docker exec -it vps-postgres psql -U postgres
```

在 psql 命令行界面里，常用的管理命令：

```
# 创建新用户
CREATE USER user_example WITH PASSWORD 'password_example' CREATEDB;
# 修改用户密码，
ALTER USER user_example WITH PASSWORD 'password_new';
# 删除用户
DROP USER IF EXISTS user_example;
# 创建数据库，并把它归到用户 user_example 名下
CREATE DATABASE database_example OWNER user_example; 
# 删除数据库
DROP DATABASE IF EXISTS database_example;
```

也可以使用  [Adminer](/post/5506-adminer/) 从网页访问管理数据库。

## 数据库的备份

和 MySQL / MariaDB 类似，数据库的备份方式，通常分为这几种：

1. 逻辑备份：导出通用的 .sql 文件，可以方便地把单个或多个数据库的内容，导入到其它服务器。执行速度比物理备份慢。对于本站的个人规模的数据库，更推荐使用逻辑备份；
2. 物理备份：迅速复制当前数据库的全部内容文件。速度更快，但通常是对数据库整体进行备份。备份和恢复都和当前的数据库版本紧密相关。
3. Docker 备份：在停止数据库后，打包备份整个 docker 项目文件夹，和相应的 docker 数据卷（volume），也是另一种可能更简单的 “物理备份”。

简单起见，本文只介绍单个数据库的 .sql 格式的逻辑备份，其它方式参见[官方文档](https://www.postgresql.org/docs/8.1/backup.html)的更详细介绍。

**备份**：

将单个数据库 database_1 的所有内容，备份到 backup_1.sql 文件：

```
sudo docker exec vps-postgres pg_dump -U postgres database_1 > backup_1.sql

# 如果未来导入的数据库的用户名可能会改变，那么建议导出时添加 --no-owner 参数
sudo docker exec vps-postgres pg_dump -U postgres --no-owner database_1 > backup_1.sql
```

**恢复**：

将前面生成的数据库备份文件 backup_1.sql，恢复到数据库 database_1 的命令：
（注意：docker exec 后面的参数是 -i ，而不是常用的  -it）

```
sudo docker exec -i vps-postgres psql -U postgres -d database_1 < backup_1.sql
```

**注意：**

- 虽然备份文件里，包含了把每个数据表（table）删除后重新创建的命令，但是，为了稳妥起见，建议先删除原有的数据库（drop database），再进行导入；
