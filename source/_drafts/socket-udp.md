---
title: 网络编程之UDP
date: 2019-01-29 15:44:17
tags:
 - Network
 - Socket
---

# 基本用法



## UDP 中的`connect`和`bind`
* connect: 虽然UDP是无连接的，但是connect可以指定目的IP/端口，在发送数据时即可不指定地址直接使用read/recv, send/write 进行数据的接收和发送

* bind: 如果不使用bind，系统会随机分配一个空闲的端口，对于客户端来说，可以不绑定端口

[TODO]:
EAGAIN