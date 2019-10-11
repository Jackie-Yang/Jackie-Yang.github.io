---
title: OpenSSH交叉编译
date: 2016-09-04 14:03:42
tags:
 - OpenSSH
---

学习了{% post_link openssh-base OpenSSH的基本使用 %}之后，有时候需要在嵌入式设备中使用OpenSSH，这个时候就需要从官网下载源码进行移植了。

# 准备工作
* 交叉编译器
* [openssh-7.3p1](http://www.openssh.com/portable.html)
* [openssl-1.0.2g](http://www.openssl.org)
* [zlib-1.2.8](http://www.zlib.net)

*OpenSSL最新版本为1.1.1，但是在编译OpenSSH的时候发现编译不过，在[OpenSSH Installation instructions](http://ftp.openbsd.org/pub/OpenBSD/OpenSSH/portable/INSTALL)中看到说明，最终改用1.0.2（同样也是长期维护的版本）*
>Note that because of API changes,
OpenSSL 1.1.x is not currently supported.

# 交叉编译
源代码通过config/configure进行配置，并生成Makefile，具体的配置参数可以参考源码网站，或者查看config/configure文件。配置完成后，使用make进行编译

```bash
# prefix 指定Install安装目录
# CC,AR等环境变量配置为相应的交叉编译器

#zlib
CHOST=arm-linux ./configure --prefix=${ZLIB_INSTALL_PATH}
make
make install

#openssl
CC=${CC} AR=${AR} RANLIB=${RANLIB} NM=${NM} ./config os/compiler:arm-linux-gcc no-asm shared --prefix=${OPENSSL_INSTALL_PATH}
make NM=${NM}
make install


#openssh
#调用参数指定zlib和OpenSSL安装路径
./configure --host=arm-linux --with-zlib=${ZLIB_INSTALL_PATH} --with-ssl-dir=${OPENSSL_INSTALL_PATH} --disable-etc-default-login CC=${CC} AR=${AR} RANLIB=${RANLIB}
make
# 不需要Install，我们将手动将OpenSSH拷贝到嵌入式设备
```
可能缺失依赖：lib64z1/lib32z1，使用apt-get安装即可

# 安装
将以下文件拷贝到设备相应目录(若使用过程中发现库文件缺失，还需要将相应的库文件拷贝到平台的库文件目录中，一般是/lib)：
```
├── bin
│   ├── scp
│   ├── sftp
│   ├── ssh
│   ├── ssh-add
│   ├── ssh-agent
│   ├── ssh-keygen
│   └── ssh-keyscan
├── etc
│   ├── moduli
│   ├── ssh_config
│   └── sshd_config
├── libexec
│   ├── sftp-server
│   └── ssh-keysign
└── sbin
    └── sshd
```

# 配置sshd服务
sshd为ssh服务端程序，需要在平台上进行相应的配置

```bash
# /etc/passwd 添加用户
sshd::103:103::/var/run/sshd:/bin/sh

# /etc/group 添加用户组
sshd:*:103:

# 配置服务器密钥
cd /usr/local/openssh/etc
ssh-keygen -t rsa -f ssh_host_rsa_key -N ""  
ssh-keygen -t dsa -f ssh_host_dsa_key -N ""  
ssh-keygen -t ecdsa -f ssh_host_ecdsa_key -N "" 
ssh-keygen -t ed25519 -f ssh_host_ed25519_key -N "" 

# 建立目录
mkdir -p /var/empty/sshd
mkdir -p /var/run/sshd

# 在/etc/init.c/rcS 中追加开机启动 /sbin/sshd
```



# 参考
[OpenSSH Installation instructions](http://ftp.openbsd.org/pub/OpenBSD/OpenSSH/portable/INSTALL)

[OpenSSL Compilation and Installation](https://wiki.openssl.org/index.php/Compilation_and_Installation)

[sshd_config(5) - OpenBSD manual pages](https://man.openbsd.org/sshd_config)

[OpenSSH Manual Pages](http://www.openssh.com/manual.html)