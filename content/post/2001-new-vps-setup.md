+++
title = "新购买 VPS 的初始登录和安全配置"
date = "2026-01-16"
tags = [
    "VPS","security"
]
+++

这篇文章会介绍一些，在刚刚购买 VPS 后，需要做的初始设置。此刻，你刚刚购买了 VPS 服务器，你拥有

- VPS 的 ip 地址：例如 123.123.123.123
- 根用户（root）的密码，例如 rootpassword

本站的设置和命令格式，都是基于 Ubuntu 24.04 LTS 的版本。

#### 1. 使用 ssh 登录到 VPS

 在你的电脑上打开命令行界面，使用 ssh 命令。（也可以下载专门的 ssh 软件，譬如 Windows 下的 [PuTTY](https://putty.org/index.html)）

```
ssh root@123.123.123.123
# 或者
ssh root@123.123.123.123 -p 22
```

（这里的 -p 22 是登录用的端口号，可以忽略。以后如果改变登录的端口号，需要添加 -p 参数指明。）

然后，根据提示输入密码，回车键确认。输入密码的过程中，屏幕上通常不会显示任何东西，既不会显示你输入的密码，也不会显示 *** 表示你输入了几位字符。

第一次登录时，会询问是否将密钥登录到当前设备上，输入 yes 确认。（详见文末）

#### 2. 用 root 登录后，首先更新系统软件

```
apt update && apt upgrade -y
```

#### 3. 创建新的用户，然后使用新用户登录系统
强烈建议，创建一个新的用户，而不是一直使用 root 用户。

```
# 创建用户，按提示输入密码
adduser new_user_name

# 将超级用户权限赋给新用户
usermod -aG sudo new_user_name
```

然后，修改 VPS 上的 ssh 设置，禁止使用 root 根用户远程登录 VPS。

输入命令，编辑设置文件
```
nano /etc/ssh/sshd_config
```

在文件中修改 PermitRootLogin 选项，（如果选项前面有注释符号 # ，把 # 删掉）

```
# 禁止使用 root 用户登录
PermitRootLogin no

# 也可以添加 AllowUsers，只允许你指定的用户登录
AllowUsers new_user_name
```

在 nano 编辑页面下，按 Ctrl + x 保存编辑后的文件，再按 y + 回车，确认保存并退回到命令行。输入命令，重启 ssh 服务。

```
# 重启 ssh 服务
systemctl reload ssh

# 如果上面的命令报错，可以尝试下面的命令：（早期版本的 Ubuntu 的服务的名字会有变化）
systemctl restart sshd
```

然后退出当前登录的用户：可以直接关闭程序窗口，也可以在命令行输入 logout 命令。

再次使用你的电脑上的命令行（或 ssh 登录软件），使用新的用户登录 VPS。

```
ssh new_user_name@123.123.123.123
```

使用新用户登录后，运行一些系统管理员级别的命令时，需要在命令前面加上 sudo，然后输入你的当前用户密码（不是 root 密码），才能执行。系统在几分钟内，不会连续要求每次都输入 sudo 密码。

#### 4. 安装 UFW 防火墙

```
# 安装 ufw 防火墙（可能 Ubuntu 已经安装了）
sudo apt install ufw -y

# 只允许这些端口接收互联网访问
sudo ufw allow 22,80,443/tcp

# 启动防火墙
sudo ufw enable
```

- 22，ssh 登录使用的端口
- 80，http 访问端口
- 443，https 访问端口

#### 5. 安装 Fail2Ban

如果一个外部 ip 频繁地使用错误密码尝试登录你的 VPS，Fail2Ban 会自动把这个 ip 暂时封禁。

```
sudo apt install fail2ban -y
```

#### 6. 一些并不是必须，但推荐进行的设置

设置 VPS 的时区，输入命令后在界面中选择地区：

```
sudo dpkg-reconfigure tzdata
```

给 VPS 设置一个别名

```
sudo hostnamectl set-hostname SHORT_NAME
```

---

### 高级知识

#### 两种登录方法：输入密码 vs 使用密钥文件

一些 VPS 设置指南，会建议使用 “更安全” 的登录方式：不是每次都输入 ssh 的用户密码，而是使用更复杂的密钥文件。和密码登录相比，使用密钥文件登录，确实可以减少密码被暴力破解的可能，以及因为输入密码时，键盘被黑客监控，甚至被人从背后直接看到密码，而造成的风险。但相应地，也有一些不足：

- 配置对于新人而言，过于繁琐（有兴趣可以）；
- 彻底禁用了密码登录方式后，不能便捷地在其它机器上进行登录；
- 密钥文件丢失，会非常麻烦；
- 如果你使用的电脑被人侵入，甚至行政上扣押，则可以无需密码直接进入你的 VPS。

所以，我们更推荐新人用户，使用密码登录的方式。有兴趣使用密钥登录的，可以自行[查看相关攻略](https://www.runoob.com/w3cnote/set-ssh-login-key.html)。

#### ssh 登录时，会在本地设备上留下痕迹

无论使用命令行的 ssh 命令，还是 PuTTY 之类的 ssh 软件，一些验证密钥会保存在你使用的电脑上。如果别人侵入你的电脑，就可以得知你的 VPS 的 ip 地址（但是，在密码登录的模式下，对方不知道密码，并不能侵入你的 VPS）。通常可以在 user_HOME/.ssh/known_hosts 把历史密钥删除。

另外，如果你把 VPS 重装了系统，再次用 ssh user@ip 登录时，会显示密钥验证错误。这时，同样需要在上述位置删掉之前的密钥。
