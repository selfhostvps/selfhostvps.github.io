
+++
title = "数据库 MySQL / MariaDB"
date = "2026-03-16"
tags = [
    "docker","database","backup"
]
[params]
  no_toc = false
  no_date = true
+++

## MySQL / MariaDB 介绍

[MySQL](https://www.mysql.com/) 是最常用的关系性数据库之一，是一些 VPS 常用自建软件（如 Wordpress blog、NextCloud…）依赖的基础。MariaDB 是 MySQL 的一个分支。MySQL 在 2009 年被收购，逐渐变得商业化后，包括 MySQL 创始人在内的一些开发者，为了保持开源项目的纯洁，而创建了 MariaDB，作为独立的开源项目来维护。

MariaDB 保持了对 MySQL 的兼容性，甚至在一些功能上更加便利。对于需要使用 MySQL 的绝大多数软件（至少是本站介绍的所有软件），选择 MariaDB 还是 MySQL，没有任何区别。本站推荐使用 MariaDB。

但是，从称谓上，MySQL 更习惯被用作这种类型数据库的统称，尤其是和其它类型的数据库做对比时（如 PostgreSQL、SQLite…）。所以，虽然大家实际安装的是 MariaDB，可能有时仍然会把它叫做 MySQL。

## MariaDB 的安装

**docker compose 目录结构**

```
/DOCKERS/mariadb       # docker compose 项目目录
├── backup             # 动态备份使用的目录
├── data               # 数据存储
├── tmp                # 临时交互目录
└── docker-compose.yml # 配置文件
```

按照本站 [docker 一文的设定](/post/2006-docker/#%E6%9C%AC%E7%AB%99%E7%9A%84%E4%B8%80%E4%BA%9B%E7%BA%A6%E5%AE%9A%E8%AE%BE%E7%BD%AE)，预先创建 docker 网络，让其它容器和外部网站，共享同一个数据库系统。

```
# 如果还没有运行过的话，先创建 docker 网络，为需要数据库的容器共享内部网络
sudo docker network create network_database
```

**docker-compose.yml**，本站使用 MariaDB 11.4 的[长期支持版本](https://mariadb.com/resources/blog/announcing-yearly-lts-releases-for-mariadb-community-server/)。

```
services:
  vps-mariadb:
    image: mariadb:11.4-ubi # 这是长期稳定支持的版本
    container_name: vps-mariadb
    restart: always
    # 如果忘记了 root 密码，取消下一行的注释，重新启动 docker-compose
    # command: --skip-grant-tables --skip-networking
    environment:
      # 必须：设置根用户 root 的密码
      - MARIADB_ROOT_PASSWORD=root_password_example
      # 可选项：启动时直接创建一个数据库，和
      # - MARIADB_DATABASE=db_example
      # - MARIADB_USER=user_example
      # - MARIADB_PASSWORD=user_password_example
    ports:
      - "3306:3306"
    volumes:
      - ./data:/var/lib/mysql
      - ./backup:/var/mariadb/backup
      - ./tmp:/tmp
    networks:
      - network_database #加入预设的数据库共享网络
networks:
  network_database:
    external: true
```

注意，

1. 如果你暂时只是为了单个服务而创建数据库，可以在第一次启动时，在 docker-compose.yml 里直接创建新的数据库名称和用户密码，可以省略下文专门进入数据库创建的步骤；
2. 在docker-compose.yml 里设置的 root 用户密码，只在第一次启动时有用。如果你以后按照下文的指令，更改了 root 密码，那么原来设置的密码再也不会有效，也不能给你任何提示作用。请自行记住新的密码！

## 进入 MariaDB，创建数据库，创建用户

在 vps 命令行下，进入数据库命令行模式，需要运行：

```
sudo docker exec -it vps-mariadb mariadb -u root -p
```

然后在下一行输入密码。也可以把密码写在下面的命令里，一次性进入（注意，-p 和密码之间没有空格）。但这样会让一些辅助记录历史命令的软件，记住你的明文密码，所以建议尽量不要这样做。

```
sudo docker exec -it vps-mariadb mariadb -u root -pPASSWORD
```

然后会看到这样的，和 vps 不同的提示符，表明你已经进入了数据库管理界面：

```
MariaDB [(none)]>
```

### 创建新的数据库和用户

很多使用 MySQL / MariaDB 的工具，都需要知道你的数据库密码，并把它保存下来。所以，一直使用根用户 root，是很不安全的。推荐创建新的用户，在 MariaDB 管理界面下，输入下面的命令。注意，把用户名和密码改成你自己的：

```
# 创建新的用户 vps_mariadb_user
CREATE USER 'vps_mariadb_user'@'%' IDENTIFIED VIA mysql_native_password USING PASSWORD('password_example') OR unix_socket;
```

创建一个新的数据库

```
# 创建新的数据库 database_new_1
CREATE DATABASE database_new_1;
```

将这个数据库的使用权限，赋给你新创建的用户

```
# 将数据库 database_new_1 的使用权限，赋给用户 vps_mariadb_user
GRANT ALL ON database_new_1.* TO 'vps_mariadb_user';
```

刷新数据库的权限。所有涉及到权限的操作，最后都要执行这条命令才生效。

```
# 刷新权限
FLUSH PRIVILEGES;

# 最后，退出数据库管理界面
QUIT;
```

然后，就可以在其它的软件设置里，填入相应的数据库信息了。通常是这些

- 数据库名称：database_new_1
- 数据库用户名：vps_mariadb_user
- 数据库密码：password_example
- 数据库地址：
	- 如果软件在 docker 容器外部：localhost 或 127.0.0.1
	- 如果是 network_database 网络里的另一个 docker 容器：vps-mariadb
- 数据库端口号：3306

数据库和用户之间，是多对多的关系。

- 可以把多个数据库的权限，都赋给一个用户，然后在所有工具的设置里，都填写同样的用户名和密码；
- 可以为每个数据库，创建一个新的用户，用户看不到不属于自己的数据库的内容；
- 也可以把一个数据库，同时赋给多个用户。

你可以随意选择适合自己的方案，更方便的或更安全的。本站在之后的工具介绍里，统一以 vps_mariadb_user 作为用户名的例子。

## 常用数据库操作

```
# 列出所有数据库
SHOW DATABASES;

# 列出所有用户
SELECT User, Host FROM mysql.user;

# 显示用户 user_name_example 的所有权限
SHOW GRANTS FOR 'user_name_example'@'%';

# 创建新的用户
CREATE USER 'user_name_example'@'%' IDENTIFIED VIA mysql_native_password USING PASSWORD('password_example') OR unix_socket;

# 修改用户的密码
ALTER USER 'user_name_example'@'%' IDENTIFIED VIA mysql_native_password USING PASSWORD('new_password') OR unix_socket;

# 修改 root 根用户的密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';

# 删除用户
DROP USER user_name_example;

# 创建新的数据库
CREATE DATABASE database_new_1;

# 删除数据库
DROP DATABASE database_new_1;

# 将数据库的使用权限，赋给用户
GRANT ALL ON database_new_1.* TO 'user_name_example'@'%';

# 取消用户对于数据库的使用权限
REVOKE ALL PRIVILEGES ON database_new_1.* FROM 'user_name_example'@'%';

# 刷新权限
FLUSH PRIVILEGES;

# 退出数据库
QUIT;
```

## 数据库的备份

### 物理备份：mariadb-backup

待续

### 逻辑备份 / SQL 导出：mariadb-dump / mysqldump

待续

## 其它

### 是否需要同时安装 MySQL 和 PostgreSQL？

另一种常见的，配合自建软件的数据库，是 PostgreSQL。有些自建工具，原生支持多种数据库；有些仅支持一种；也有些能找到第三方开发者提供的，对其它数据库的支持方案，但可能不大稳定，需要对技术更有自信，才适合使用。

正常情况下，单独的 MySQL 或者 PostgreSQL，运行两三份个人软件的数据库，需要占用的服务器内存，从 200MB 到 500+MB，甚至更多。如果你的 VPS 内存充足，同时安装两种数据库，会更加便利。但如果你是只有 1~2 GB 内存的 VPS，又想跑多个服务，就需要斟酌一下，能不能只装一种数据库，就能满足自己的工具需求？

| 自建软件      | MySQL | PostgreSQL | SQLite |
| --------- | ----- | ---------- | ------ |
| Wordpress | ✓     | 第三方 | 第三方 |
| Nextcloud | ✓     | ✓          | x      |
| Mastodon  | x     | ✓          | x      |
| Wallabag  | ✓     | ✓          | ✓      |
| Miniflux  | x     | ✓          | x      |

### 忘记 root 密码怎么办？

停止 MariaDB 容器

```
cd /DOCKERS/mariadb
sudo docker-compose down
```

编辑 docker-compose.yml，添加下面的 command（如果是使用本文的 docker-compose.yml，只需取消这一行前面的注释，注意对齐缩进）

```
  command: --skip-grant-tables --skip-networking
```

重启 docker 容器，然后就可以无需密码进入数据库：

```
sudo docker-compose up -d
sudo docker exec -it vps-mariadb mariadb -u root
```

在 MariaDB 提示符下，输入下面的命令，设置新密码：

```
FLUSH PRIVILEGES;
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
QUIT;
```

退出数据库，停止 docker-compose 容器，再次编辑 docker-compose.yml，把 --skip-grant-tables 那行命令删除或注释掉，再次启动 docker-compose 容器，就可以使用新的 root 密码进入 MariaDB 数据库了。

```
# 停止容器
sudo docker-compose down

# 把 docker-compose.yml 里这一行注释掉
# command: --skip-grant-tables --skip-networking

# 再次启动容器
sudo docker-compose up -d
```

