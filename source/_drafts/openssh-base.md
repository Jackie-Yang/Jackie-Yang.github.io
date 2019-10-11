---
title: OpenSSH的基本使用
date: 2016-09-04 11:03:42
tags:
 - OpenSSH
---

# 简介
OpenSSH 是 SSH(Secure SHell) 协议的免费开源实现。SSH协议族可以用来进行远程控制，及文件传输。而实现此功能的传统方式，如telnet(终端仿真协议)、 rcp ftp、 rlogin、rsh都是不安全的，并且会使用明文传送密码。OpenSSH提供了服务端后台程序和客户端工具，用来加密远程控制和文件传输过程中的数据，并由此来代替原来的类似服务。

<!--more-->

# 安装服务
```bash
# 服务端
sudo apt-get install openssh-server
# 客户端
sudo apt-get install openssh-client
```

# 基本使用
```bash
#登录
ssh user@ip
#指定端口登陆
ssh -p port user@ip

#远程执行指令
ssh user@ip  "command" 

#复制文件 用法类似cp
scp user@ip:source target
scp source user@ip:target
```

# SSH服务器配置
在


# 参考
[OpenSSH Manual Pages](http://www.openssh.com/manual.html)