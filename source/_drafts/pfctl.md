---
title: PF防火墙
date: 2019-03-21 17:12:22
tags:
 - Network
---
# 简述
PF(Packet Filter) 防火墙是 OpenBSD上进行 TCP/IP 流量过滤和网络地址转换的软件系统。类似于iptables，PF 同样也能提供 TCP/IP 流量的整形和控制，并且提供带宽控制和数据包优先集控制。

<!--more-->
# 基本指令
```bash
#启动/关闭
pfctl -e/-d
# 载入 pf.conf 文件
pfctl -f /etc/pf.conf  
# 解析文件，但不载入
pfctl -nf /etc/pf.conf 
# 只载入文件中的NAT规则
pfctl -Nf /etc/pf.conf 
# 只载入文件中的过滤规则
pfctl -Rf /etc/pf.conf

#显示当前的NAT规则
pfctl -sn 
#显示当前的过滤规则
pfctl -sr 
#显示当前的状态表
pfctl -ss 
#显示过滤状态和计数
pfctl -si 
#显示任何可显示的
pfctl -sa 

#载入默认配置，清除其他添加的规则
pfctl -F all -f /etc/pf.conf
```

# 规则编写

## 文件格式
系统引导到在rc脚本文件运行PF时PF从/etc/pf.conf文件载入配置规则。注意当/etc/pf.conf文件是默认配置文件，在系统调用rc脚本文件时，它仅仅是作为文本文件由pfctl装入并解释和插入pf的。

对于一些应用来说，其他自定义规则集可以在系统引导后由其他应用载入。对于一些设计的非常好的unix程序，PF提供了足够的灵活性。
pf.conf 文件有7个部分:

* 宏:              用户定义的变量，包括IP地址，接口名称等等
* 表:              一种用来保存IP地址列表的结构
* 选项:          控制PF如何工作的变量
* 整形:          重新处理数据包，进行正常化和碎片整理
* 排队:          提供带宽控制和数据包优先级控制.
* 转换:          控制网络地址转换和数据包重定向.
* 过滤规则:   在数据包通过接口时允许进行选择性的过滤和阻止
除去宏和表，其他的段在配置文件中也应该按照这个顺序出现，尽管对于一些特定的应用并不是所有的段都是必须的。
空行会被忽略，以＃开头的行被认为是注释.

*示例：
[Simple firewalling and traffic shaping with PF](http://kestas.kuliukas.com/pf.conf/)*

## 基本语法
一般而言，最简单的过滤规则语法是这样的：
```
action direction [log] [quick] on interface [af] [proto protocol] \
from src_addr [port src_port] to dst_addr [port dst_port] \
[tcp_flags] [state]
```
### action
数据包匹配规则时执行的动作，放行或者阻塞。放行动作把数据包传递给核心进行进一步出来，阻塞动作根据block-policy 选项指定的方法进行处理。默认的动作可以修改为阻塞丢弃或者阻塞返回。

### direction (out/in)
数据包传递的方向，进或者出

### [log]
指定匹配的数据包应该被pflogd（8）进行日志记录 
（如果规则指定了keep state, modulate state, or synproxy state 选项，则只有建立了连接的状态被记录。要记录所有的日志，使用log-all）

### [quick]
如果数据包匹配的规则指定了quick关键字，则这条规则被认为时最终的匹配规则，指定的动作会立即执行。

### interface
数据包通过的网络接口的名称或组,例如lo0。组是接口的名称但没有最后的整数。比如ppp或fxp，会使得规则分别匹配任何ppp或者fxp接口上的任意数据包。

### af (inet,inet6)
数据包的地址类型，inet代表Ipv4，inet6代表Ipv6。通常PF能够根据源或者目标地址自动确定这个参数。

### protocol (tcp,udp,icmp,icmp6)
数据包的4层协议


## 指定地址
除了使用IP地址来指定主机外，也可以使用主机名。当主机名被解析成IP地址时，IPv4和IPv6地址都被插进规则中。IP地址也可以通过合法的接口名称或者self关键字输入表中，这样的表会分别包含接口或者机器上（包括loopback地址）上配置的所有IP地址。

*地址0.0.0.0/0 以及 0/0在表中不能工作。替代方法是明确输入该地址或者使用宏。*

* any 代表所有地址
```
block quick from <banned> to any
```
* all 是 from any to any的缩写。
```
block log all
```


## 列表

在载入规则集碰到列表时，会产生多个规则，每条规则对于列表中的一个条目。例如：
```
block out on fxp0 from { 192.168.0.1, 10.5.32.6 } to any
```
展开后:
```
block out on fxp0 from 192.168.0.1 to any
block out on fxp0 from 10.5.32.6 to any
```
**逗号在列表中可有可无**

## 表
表是用来保存一组IPv4或者IPv6地址。在表中进行查询是非常快的，并且比列表消耗更少的内存和cpu时间。因此，表是保存大量地址的最好方法。

* 表通过table关键字生成
* 可使用！进行排除
```
table <banned> { 192.168.0.0/16, 127.0.0.0/8, 0.0.0.0/8, !172.16.1.0/24}
```

* const: 这类表的内容一旦创建出来就不能被改变。如果这个属性没有指定，可以使用pfctl添加和删除表里的地址，即使系统是运行在2或者更高得安全级别上。
```
table <private> const { 10/8, 172.16/12, 192.168/16 }
```
* persist: 即使没有规则引用这类表，内核也会把它保留在内存中。如果这个属性没有指定，当最后引用它的规则被取消后内核自动把它移出内存。
```
table <badhosts> persist
```

## 宏
宏是用户定义变量用来指定IP地址，端口号，接口名称等等。宏可以降低PF规则集的复杂度并且使得维护规则集变得容易。

宏名称必须以字母开头，可以包括字母，数字和下划线。宏名称不能包括保留关键字如：`pass, out, queue ...`


```
ext_if = "fxp0"
block in on $ext_if from any to any
```
这生成了一个宏名称为ext_if, 当一个宏在它产生以后被引用时，它的名称前面以$字符开头。

宏也可以展开成列表，如：
```
friends = "{ 192.168.1.1, 10.0.2.5, 192.168.43.53 }"
```

由于宏不能在引号内被扩展，因此在列表中使用必须使用下面得语法：
```
host1 = "192.168.1.1"
host2 = "192.168.1.2"
all_hosts = "{" $host1 $host2 "}"
```
宏 $all_hosts 现在会展开成 192.168.1.1, 192.168.1.2.


# 防火墙配置


# Reference
```bash
$ man pf.conf
$ man pfctl
```
* [PF中文手册](https://www.freebsdchina.org/forum/topic_24641.html)
* [Wikipedia - PF (firewall)](https://en.wikipedia.org/wiki/PF_(firewall))
* [OpenBSD PF - User's Guide](https://www.openbsd.org/faq/pf/)
* [Simple firewalling and traffic shaping with PF](http://kestas.kuliukas.com/pf.conf/)