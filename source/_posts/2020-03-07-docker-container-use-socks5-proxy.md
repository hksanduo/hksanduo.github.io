---
title: "docker容器使用socks5做全局代理"
date:   2020-03-07  15:13:00 +0800
layout: post
tag:
- Docker
- Proxy
categories:
- Linux
---

docker容器使用socks5做全局代理

-------
## 背景
博主自行构建ubuntu容器来编译openwrt，部分组件构建需要获取墙外资源，博主使用的代理工具，支持的协议
为socks5，但是容器中部分工具如：**wget** 只支持http代理，所以在容器中需要配置socks5转http全局代理。  

## 宿主机代理配置
需要修改宿主机代理客户端配置，方便局域网其他主机连接代理，容器使用的网络类型为桥接。之前也使用过host网
络，但是在容器中测试并未成功。为了能迅速编译openwrt，只能使用默认桥接网络进行代理。     

首先配置客户端，允许局域网中其他主机进行连接，我这里直接配置成“0.0.0.0”，虽说这个不安全，但是在局域网中
风险暂时可以接受。这里需要注意，需要使用firewalld或者iptables启用本地代理端口。firewalld配置指令如下：
```
firewall-cmd --permanent --add-port=6666/tcp
```    

个人本地代理服务器配置如下：
![20200307-proxy-client.png](https://hksanduo.github.io/images/20200307-proxy-client.png)    
宿主机本地的代理端口为：6666，未设置验证用户名和密码    
可以使用局域网中其他主机测试一下，宿主机本地代理服务器是否配置成功，测试过程这里就不在赘述了。

## 容器构建配置
以下Dockerfile配置文件仅供参考，这是我为了编译openwrt自行构建的。除了openwrt编译需要的基础环境，
我增加了polipo，我使用polipo这个工具进行全局代理。设置http和https代理地址。
```
FROM ubuntu:latest

ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8

RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

RUN echo 'Asia/Shanghai' >/etc/timezone

RUN apt-get update -qq && \
    apt-get upgrade -qqy && \
    apt-get install -qqy build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib wget iputils-ping curl polipo  && \
    apt-get clean && \
    rm -rf /tmp/* /var/tmp/*

ADD config /etc/polipo/config

ENV http_proxy "http://127.0.0.1:8183"
ENV https_proxy  "http://127.0.0.1:8183"
```
polipo配置文件如下：
```
logSyslog = true
logFile = /var/log/polipo/polipo.log
socksParentProxy = "192.168.3.200:6666"
socksProxyType = socks5
proxyPort = 8183
proxyAddress = "0.0.0.0"
allowedClients = 127.0.0.1
```
宿主机的ip为192.168.3.200，宿主机代理服务器启用的端口为：6666，polipo全局代理的端口为8183   
使用以下指令进行构建
```
docker build -t openwrt-build-env .
```

## 启用容器并进行测试
执行以下指令映射本地目录到容器中去
```
docker run -itd --name openwrt-build-env -v ～/openwrt/openwrt:/home/user/openwrt openwrt-build-env
```     
使用以下指令进入容器
```
docker exec -it openwrt-build-env /bin/bash
```
使用以下指令测试代理是否成功      
```
curl cip.cc
```
![20200307-proxy-test.png](https://hksanduo.github.io/images/20200307-proxy-test.png)
显示得ip位于国外，代理成功，可以开心编译openwrt了。

## 参考内容
- [https://blog.denghaihui.com/2017/10/10/shadowsocks-polipo/](https://blog.denghaihui.com/2017/10/10/shadowsocks-polipo/)
- [https://wiki.archlinux.org/index.php/Polipo (简体中文))](https://wiki.archlinux.org/index.php/Polipo_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
- [https://juejin.im/post/5c91ff5ee51d4534446edb9a](https://juejin.im/post/5c91ff5ee51d4534446edb9a)
- [https://milkice.me/2019/08/07/docker-network-tunnel/](https://milkice.me/2019/08/07/docker-network-tunnel/)
- [https://zhanghongtong.github.io/2019/06/27/Ubuntu%E5%92%8Cdocker%E4%BD%BF%E7%94%A8shadowsocks%E5%AE%A2%E6%88%B7%E7%AB%AF%E7%BF%BB%E5%A2%99/](https://zhanghongtong.github.io/2019/06/27/Ubuntu%E5%92%8Cdocker%E4%BD%BF%E7%94%A8shadowsocks%E5%AE%A2%E6%88%B7%E7%AB%AF%E7%BF%BB%E5%A2%99/)
- [https://wiki.archlinux.org/index.php/Docker#Proxy_configuration](https://wiki.archlinux.org/index.php/Docker#Proxy_configuration)
- [https://docs.docker.com/network/proxy/#use-environment-variables](https://docs.docker.com/network/proxy/#use-environment-variables)
- [https://kebingzao.com/2019/02/14/centos7-ss-proxy/](https://kebingzao.com/2019/02/14/centos7-ss-proxy/)
- [https://kebingzao.com/2019/02/22/docker-container-proxy/](https://kebingzao.com/2019/02/22/docker-container-proxy/)
