+++
title = "常用 Linux 命令和工具"
date = "2026-01-16"
tags = [
    "vps","linux","tools"
]
[params]
  no_toc = false
  no_date = true
+++

## 常用命令

网上有很多 Linux 常用命令的教程（[英文](https://www.digitalocean.com/community/tutorials/linux-commands)、[中文](https://www.runoob.com/w3cnote/linux-common-command-2.html)），这里就不重复阐述了。只是简单列出一些 vps 管理时最常用的命令，以便让读者看到后面的具体教程时，大致明白每条命令都是在做什么。具体如何使用各种命令，以及更详尽的参数，请去查阅其它教程和文档。

- 列出文件 ls、创建目录 mkdir、进入目录 cd、显示当前位置 pwd
- 复制 cp、移动 mv、删除 rm、创建链接 ln
- 以超级用户权限执行命令 sudo
- 查看文件内容 cat、less
- 更改文件权限 chmod、更改文件所有者 chown
- 安装软件 apt
- 编辑文件 nano：编辑后按 Ctrl + x，确认是否保存
- 查看硬盘使用情况：df -hl
- 查看当前目录大小：du -sh
- 生成 32 位随机密码：openssl rand -hex 32

## 常用工具

### 通过 SFTP 客户端，和 VPS 互传文件

常用的，通过图形界面，和 VPS 互传文件的 SFTP 软件。使用时，就[像 ssh 登录](/post/2001-new-vps-setup/#1-%E4%BD%BF%E7%94%A8-ssh-%E7%99%BB%E5%BD%95%E5%88%B0-vps)时那样，输入 vps 的 ip 地址、端口号（默认 22）、用户名、密码。

- Windows：[FileZilla](https://filezilla-project.org/download.php?type=client)、[CyberDuck](https://cyberduck.io/download/)、[WinSCP](https://winscp.net/eng/download.php)
- MacOS：FileZilla、CyberDuck
- Linux 桌面：FileZilla、以及很多常用文件管理器会自带 SFTP 支持
