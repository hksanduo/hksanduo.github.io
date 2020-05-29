---
title: "memcached 安全加固"
date:   2018-03-02 12:00:00 +0800
layout: post
tag: 
- Security
categories:
- Web
---

# Memcached安全加固
------
## Memcached用户
如果您正在使用memcached，请在不使用UDP的情况下禁用UDP。在memcached启动时，您可以指定`--listen 127.0.0.1`仅侦听本地主机并`-U 0`完全禁用UDP。默认情况下，memcached侦听**INADDR_ANY**，并在UDP支持ENABLED的情况下运行。文档：
[https://github.com/memcached/memcached/wiki/ConfiguringServer#udp](https://github.com/memcached/memcached/wiki/ConfiguringServer#udp)
运行以下命令可以轻松测试服务器是否易受攻击：

    $ echo -en "\x00\x00\x00\x00\x00\x01\x00\x00stats\r\n" | nc -q1 -u 127.0.0.1 11211
    STAT pid 21357
    STAT uptime 41557034
    STAT time 1519734962
    ...

如果您看到非空的响应（如上所示），则您的服务器很脆弱。
## 系统管理员
请确保您的memcached服务器从互联网受到防火墙限制！为了测试它们是否可以使用UDP访问，运行nmap来检测：

    $ nmap TARGET -p 11211 -sU -sS --script memcached-info
    Starting Nmap 7.30 ( https://nmap.org ) at 2018-02-27 12:44 UTC
    Nmap scan report for xxxx
    Host is up (0.011s latency).
    PORT      STATE         SERVICE
    11211/tcp open          memcache
    | memcached-info:
    |   Process ID           21357
    |   Uptime               41557524 seconds
    |   Server time          2018-02-27T12:44:12
    |   Architecture         64 bit
    |   Used CPU (user)      36235.480390
    |   Used CPU (system)    285883.194512
    |   Current connections  11
    |   Total connections    107986559
    |   Maximum connections  1024
    |   TCP Port             11211
    |   UDP Port             11211
    |_  Authentication       no
    11211/udp open|filtered memcache

## 互联网服务提供商
**分布式缓存反射器**
为了在未来击败此类攻击，我们需要修复易受攻击的协议以及IP欺骗。只要互联网上允许IP欺骗，我们就会陷入困境。
通过跟踪这些攻击背后的人来帮助我们。我们必须知道谁不是有问题的memcached服务器，而是首先向他们发送查询的人。没有你的帮助，我们无法做到这一点！
## 开发商
请停止使用UDP。如果您必须，请不要默认启用它。如果你不知道什么是放大攻击，我特此禁止你**SOCK_DGRAM**在编辑器中输入内容。
我们已经遇到过这么多次了。DNS，NTP，Chargen，SSDP和现在的memcached。如果使用UDP，则必须始终以严格较小的数据包大小响应请求。否则你的协议将被滥用。另外请记住，人们会忘记设置防火墙。做一个开发人员。不要发明缺乏任何类型认证的基于UDP的协议。

## 加固方式总结
### 配置访问控制。
建议用户不要将服务发布到互联网上而被黑客利用，可以通过ECS安全组规则或IPtables配置访问控制规则。
例如，在Linux环境中运行命令`iptables -A INPUT -p tcp -s 192.168.0.2 —dport 11211 -j ACCEPT`，在IPtables中添加此规则只允许192.168.0.2这个IP对11211端口进行访问。

### 绑定监听IP。
如果Memcached没有在公网开放的必要，可在Memcached启动时指定绑定的IP地址为 127.0.0.1。例如，在Linux环境中运行以下命令：

    memcached -d -m 1024 -u memcached -l 127.0.0.1 -p 11211 -c 1024 -P /tmp/memcached.pid

### 最小化权限运行。
使用普通权限账号运行，指定Memcached用户。例如，在Linux环境中运行以下命令来运行Memcached：

    memcached -d -m 1024 -u memcached -l 127.0.0.1 -p 11211 -c 1024 -P /tmp/memcached.pid

### 修改默认端口。
修改默认11211监听端口为11222端口。在Linux环境中运行以下命令：

    memcached -d -m 1024 -u memcached -l 127.0.0.1 -p 11222 -c 1024 -P /tmp/memcached.pid

Memcached命令参数说明

    -d 是指启动一个守护进程。
    -m 是指分配给Memcached使用的内存数量，单位是MB，以上为1024MB。
    -u 是指运行Memcached的用户，推荐使用单独普通权限用户memcached，而不要使用root权限账户。
    -l 是指监听的服务器IP地址，例如指定服务器的IP地址为127.0.0.1。
    -p 是用来设置Memcached的监听端口，默认端口为11211。建议设置1024以上的端口。
    -c 是指最大运行的并发连接数，默认是1024。可按照您服务器的负载量来设定。
    -P 是指设置保存Memcached的pid文件，例如保存在 /tmp/memcached.pid 位置。

**文章参考**
[https://blog.cloudflare.com/memcrashed-major-amplification-attacks-from-port-11211/](https://blog.cloudflare.com/memcrashed-major-amplification-attacks-from-port-11211/)
