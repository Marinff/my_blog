---
title: ssh
date: '2025-05-05 09:34:43'
updated: '2025-05-08 23:53:20'
permalink: /post/ssh-z11cd36.html
comments: true
toc: true
---



# ssh

## 笔记

最近学长在学**Stratoshark**，有关ssh和sysdig相关的知识点，没学过，学一下

### ssh

ssh全称secure shell，安全shell协议
是一种计算机网络协议，默认端口号位22

通过ssh协议可以在客户端安全的远程连接linux服务器或其他设备

ssh的核心功能是

1. 安全远程登录：替代传统的明文协议，提供加密的终端访问
2. 文件传输：通过scp或sftp安全传输文件
3. 端口转发/隧道
4. 远程命令执行

#### openssh

openssh是广泛使用的一种实现ssh的软件

openssh分为两个部分：client端和server端

openssh server端主要提供远程登录和文件传输功能，可以被其他用户通过ssh协议连接并访问，进行命令执行、上传或下载文件

openssh client端是用来连接服务端的工具
客户端连接服务器端时需要身份认证，常用有密码认证和密钥认证

#### ssh命令

ssh远程连接命令

```bash
ssh username@hostname -p port(默认是22)

ssh -i ./密钥文件 username@host
```

在远程主机上执行命令

```bash
ssh username@hostname command
```

> - `-l user`：指定要登录的用户。
> - `-p port`：指定连接到远程主机的端口号，默认是22。
> - `-i identity_file`：指定身份验证文件（私钥文件）。
> - `-v`：详细模式，可以显示调试信息。
> - `-C`：启用压缩。
> - `-N`：不执行远程命令，只进行端口转发。
> - `-f`：后台运行。
> - `-L local_port:remote_host:remote_port`：本地端口转发。
> - `-R remote_port:local_host:local_port`：远程端口转发。
> - `-D [bind_address:]port`：动态应用程序级端口转发。

使用linux时，开机第一件事就是登录用户，输入用户名和密码，这个是存储在 `/etc/passwd` 文件中的
使用ssh远程登录的时候是类似的，在登录命令中，都要传入username，这个username要求远程机器中是创建的，就是使用这个username去登录远程机器，hostname就是指定这个机器的地址，然后如果这个用户有设置密码，就会要求输入密码

### sysdig

使用`Stratoshark`获取ssh的时候需要用到这个工具为基础（大概是吧

sysdig是一个在linux平台上进行系统监控、分析和排除障碍的工具

参考

[ssh详解–让你彻底学会ssh-CSDN博客](https://blog.csdn.net/m0_51720581/article/details/131796669)
