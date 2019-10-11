---
title: 出国学习指南
date: 2019-03-24 14:32:04
tags:
 - Network
---

# 简述
Google一下，你就知道

方案思路
1. dnsmasq+china-list+ss-tunnel 解决DNS污染的问题
2. iptables+ipset+APNIC+ss-redir 分流国内外流量并代理
<!--more-->

网络架构：
![网络](network.jpeg)

数据逻辑：
![数据](data.jpeg)

# 服务器搭建

## 准备工作
* 境外服务器
* 该方案使用的是CentOS 7，Ubuntu 18也挺方便，以下工具可以通过apt-get直接安装

## 环境依赖

```bash
yum-config-manager --enable epel
yum install epel-release -y
# 以下工具根据实际情况选择
yum install git vim wget -y
yum install gcc gettext autoconf libtool automake make pcre-devel asciidoc xmlto c-ares-devel libev-devel libsodium-devel mbedtls-devel -y
yum install zlib-devel openssl-devel -y
```

## 安装shadowsocks-libev
shadowsocks有python版本的，但是很久没维护了，shadowsocks-libev功能更强大，资源占用更少。
```bash
#添加软件源
cd /etc/yum.repos.d/
wget https://copr.fedorainfracloud.org/coprs/librehat/shadowsocks/repo/epel-7/librehat-shadowsocks-epel-7.repo
yum update
#安装
yum install shadowsocks-libev
```

## 编译安装simple-obfs

```bash
git clone https://github.com/shadowsocks/simple-obfs.git
cd simple-obfs
git submodule update --init --recursive
./autogen.sh
./configure && make
make install
```

## 配置
```bash
vim /etc/shadowsocks-libev/config.json
```

```json
{
    "server":"0.0.0.0",
    "server_port":8388,
    "local_port":1080,
    "password":"xxxx",
    "timeout":600,
    "method":"aes-256-gcm",
    "mode":"tcp_and_udp",
    "plugin":"obfs-server",
    "plugin_opts":"obfs=http"
}
```

## 配置服务
目前的shadowsocks-libev会在/lib/systemd/system/下生成各个服务的模版
```bash
vim /lib/systemd/system/shadowsocks-libev-server\@.service
```
配置大概如下：
```
[Unit]
Description=Shadowsocks-Libev Custom Server Service for %I
Documentation=man:ss-server(1)
After=network.target

[Service]
Type=simple
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
ExecStart=/usr/bin/ss-server -c /etc/shadowsocks-libev/%i.json
Restart=on-abort
#User=nobody
#Group=nobody
LimitNOFILE=32768

[Install]
WantedBy=multi-user.target
```

可见它会到/etc/shadowsocks-libev/文件夹下加载配置文件夹，通过配置传入配置名即可，以下指令在后面客户端的配置同样适用

```bash
# 设置开机自启服务
systemctl enable  shadowsocks-libev-xxx@配置名

# 移除开机自启服务
systemctl disable  shadowsocks-libev-xxx@配置名

# 启动
systemctl start  shadowsocks-libev-xxx@配置名

# 停止
systemctl stop  shadowsocks-libev-xxx@配置名

#查看状态
systemctl status  shadowsocks-libev-xxx@配置名
```

# 客户端配置

客户端基于Ubuntu server18.04搭建，工具如果没有特别说明都是通过apt-get 获取。
与服务器端一样，首先需要安装`shadowsocks-libev` 和 `simple-obfs`


## DNS代理

这部分主要解决DNS污染问题

* dnsmasq，监听53端口，相当于本地DNS服务器，建立DNS缓存。
* ss-tunnel 在本地作为dnsmasq的上层DNS服务器，监听5353端口。
* dnsmasq将DNS请求发往本地5353端口，ss-tunnel 通过隧道转发到ss-server端，由服务器向8.8.8.8:53发送DNS请求得到纯净的dns结果，并返回

> DNS请求->dnsmasq(:53)->ss-tunnel(:5353) —> ss-server ~~-隧道-~~>8.8.8.8:53

* china-list增加国内域名白名单：对于国内域名，指定国内的DNS服务器或直接向本地DNS服务器请求

### 配置ss-tunnel：
```bash
vim /etc/shadowsocks-libev/tunnel.json
```
```json
{
    "server":"xxx.xxx.xxx.xxx",
    "server_port":xxx,
    "local_address":"0.0.0.0",
    "local_port":5353,
    "password":"password",
    "timeout":60,
    "method":"aes-256-gcm",
    "mode":"tcp_and_udp",
    "tunnel_address":"8.8.8.8:53",
    "plugin":"obfs-local",
    "plugin_opts":"obfs=http;obfs-host=www.xxxx.com"
}
```

启动服务
```bash
systemctl enable shadowsocks-libev-tunnel@tunnel
systemctl start shadowsocks-libev-tunnel@tunnel
```


### 配置dnsmasq
```bash
sudo apt-get install dnsmasq
```

配置默认生成的 /etc/dnsmasq.conf
```makefile
no-resolv
表示不通过/etc/resolv.conf或者其他文件来获取配置

server=127.0.0.1#5353
表示上层dns服务器是本地的5353端口，这个是我们配置ss-tunnel的监听端口
```

### 配置china-list：
```bash
wget https://raw.githubusercontent.com/felixonmars/dnsmasq-china-list/master/install.sh

# 把SERVERS改成国内DNS服务器，默认是114，可以加上本地DNS服务器
sudo ./install.sh
# 自动重启dnsmasq生效配置
```

### 验证：
dig @192.168.1.100 www.taobao.com

dig @192.168.1.100 -p 5353 www.taobao.com


检验两次DNS请求返回是否不一致，192.168.1.100 为搭建客户端的设备地址，当然也可以在客户端中对本地端口发起请求

直接进行请求，访问的是53端口，淘宝是国内域名 ，会通过本地DNS服务器进行解析
指定5353端口，即绕过dnsmasq直接通过ss-tunnel进行DNS请求，得到海外淘宝ip地址

验证完成，DNS配置完毕

## 分流及透明代理
DNS解析得到IP后，如果境内ip也通过代理访问，速度较慢且浪费流量，

处理思路：
* 从APNIC（亚太互联网络信息中心 Asia-Pacific Network Information Center）获取境内ip列表，通过iptable分流境内及境外流量
* 国内ip直接访问即可，海外IP则进行端口转发给ss-redir进行透明代理。
* 透明代理，用户无需进行代理服务器配置，通过拦截策略（这里使用iptables防火墙进行NAT实现）拦截用户报文，像服务器发送请求，接收回传，实现全局的透明代理。

>*一般的正向代理需要客户端应用配置代理服务器，向代理服务器发送访问请求，代理服务器再访问目标服务器。若用户不进行配置，则不进行代理。透明代理对客户来说是无感的，只需要网络配置网关即可。*

### 准备工作
1. 开启ipv4端口转发
```bash
vim /etc/sysctl.conf

# 增加一行
net.ipv4.ip_forward = 1

#运行命令使之生效
sysctl -p
```
2. 安装ipset
```bash
sudo apt-get install ipset
```
### 配置ss-redir
```bash
vim /etc/shadowsocks-libev/redir.json
systemctl enable shadowsocks-libev-redir@redir
systemctl start shadowsocks-libev-redir@redir
```
```json
{
    "server":"xxx.xxx.xxx.xxx",
    "server_port":xxx,
    "local_address":"0.0.0.0",
    "local_port":2080,
    "password":"password",
    "timeout":60,
    "method":"aes-256-gcm",
    "mode":"tcp_and_udp",
    "plugin":"obfs-local",
    "plugin_opts":"obfs=http;obfs-host=www.xxxx.com"
}
```

### APNIC获取国内ip
```bash
wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > /etc/shadowsocks-libev/apnic_cn.txt
```

### ip转发策略
* 导入策略
```bash
vim /usr/local/bin/ss_up.sh
```
```bash
#! /bin/bash
ss_server_ip=xx.xx.xx.xxx
ss_redir_port=2080
china_ip_list=/etc/shadowsocks-libev/apnic_cn.txt

BYPASS_RESERVED_IPS=" \
    0.0.0.0/8 \
    10.0.0.0/8 \
    127.0.0.0/8 \
    169.254.0.0/16 \
    172.16.0.0/12 \
    192.168.0.0/16 \
    224.0.0.0/4 \
    240.0.0.0/4 \
    ${ss_server_ip}/32 \
"

ipset create ss_bypass hash:net

for ip in $BYPASS_RESERVED_IPS; do
    ipset -! add ss_bypass $ip
done

ipset create apnic_cn hash:net

if [ -f $china_ip_list ]; then
    for ip in $(cat "$china_ip_list"); do
        ipset -! add apnic_cn $ip
    done
else
    echo "no china ip file"
fi

#iptables -t nat -A POSTROUTING -s 192.168.50.0/24 -j MASQUERADE

iptables -t nat -N SHADOWSOCKS
iptables -t nat -A SHADOWSOCKS -p tcp -m set --match-set ss_bypass dst -j RETURN
iptables -t nat -A SHADOWSOCKS -p tcp -m set --match-set apnic_cn dst -j RETURN
iptables -t nat -A SHADOWSOCKS -p icmp -m set --match-set apnic_cn dst -j RETURN

iptables -t nat -A SHADOWSOCKS -p tcp -j REDIRECT --to-ports $ss_redir_port
iptables -t nat -A SHADOWSOCKS -p udp -j REDIRECT --to-ports $ss_redir_port

iptables -t nat -A PREROUTING -p tcp -j SHADOWSOCKS
iptables -t nat -A PREROUTING -p icmp -j SHADOWSOCKS
iptables -t nat -A OUTPUT -p tcp -j SHADOWSOCKS
```

* 移除策略
```bash
vim /usr/local/bin/ss_down.sh
```
```bash
#!/bin/bash
iptables -t nat -D OUTPUT -p icmp -j SHADOWSOCKS
iptables -t nat -D OUTPUT -p tcp -j SHADOWSOCKS
iptables -t nat -D PREROUTING -p icmp -j SHADOWSOCKS
iptables -t nat -D PREROUTING -p tcp -j SHADOWSOCKS
iptables -t nat -F SHADOWSOCKS
iptables -t nat -X SHADOWSOCKS
ipset destroy apnic_cn
ipset destroy ss_bypass
```

* 创建开机自动配置服务：
vim /etc/systemd/system/transparent-proxy.service
```
[Unit]
Description=Packet Filtering Framework and Shadowsocks-chnroute
#Before=network-pre.target
Wants=network-pre.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/ss_up.sh
ExecStop=/usr/local/bin/ss_down.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```
```bash
systemctl enable  transparent-proxy
systemctl start  transparent-proxy
```

# 参考
* 指导 - **楠哥**
* [利用shadowsocks打造局域网翻墙透明网关](https://medium.com/@oliviaqrs/利用shadowsocks打造局域网翻墙透明网关-fb82ccb2f729)

* [ss透明代理](https://www.zfl9.com/ss-redir.html)
