---
title: "archlinux docker 容器dns服务失效问题"
date:   2020-03-05  22:41:00 +0800
layout: post
tag:
- Docker
categories:
- Linux
---

archlinux docker 容器dns服务失效问题

-------
## 背景
博主一台archlinux服务器上运行docker服务，在docker容器中dns域名解析总是失败，具体现象是可以ping通
DNS服务器，但是无法解析相关域名，已在容器中配置DNS，宿主机网络一切正常。具体现象如下：  

![20200305-docker-dns-not-working.png](https://hksanduo.github.io/images/20200305-docker-dns-not-working.png)

```
nslookup: write to '8.8.8.8': No route to host
;; connection timed out; no servers could be reached
```
同样问题存在docker镜像构建过程中，博主在构建ubuntu镜像，同步软件源会出现以下问题，导致构建失败   
```
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic/InRelease  Temporary failure resolving 'archive.ubuntu.com'
```

![20200305-dockerfile-build-failed.png](https://hksanduo.github.io/images/20200305-dockerfile-build-failed.png)

## 解决方法
### 配置docker dns
使用vim或者其他文本工具打开 **/etc/docker/daemon.json** 在daemon.json中设置DNS，新增```"dns": ["114.114.114.114","8.8.8.8"]```如下所示：  
![20200305-docker-dns-config.png](https://hksanduo.github.io/images/20200305-docker-dns-config.png)

设置DNS重启docker，docker容器中DNS服务仍然存在问题，通过博主不断尝试，发现系统启用firewalld防火墙，
导致DNS服务未生效，通过以下指令将docker0网卡添加到信任区，发现DNS服务生效了。
```
sudo firewall-cmd --permanent --zone=trusted --change-interface=docker0
sudo firewall-cmd --reload
sudo systemctl restart docker
```
目前并未探明之前的防火墙策略为何会阻断DNS服务，哪位大佬可以指导一下，我将不胜感激。

### 测试
```
docker run --name box1 -it --rm busybox sh
```
我们使用busybox 启动一个容器服务来测试网络。    
![20200305-docker-dns-working.png](https://hksanduo.github.io/images/20200305-docker-dns-working.png)   
DNS服务正常。可以使用

## 参考内容
- [https://github.com/moby/moby/issues/36151](https://github.com/moby/moby/issues/36151)
- [https://askubuntu.com/questions/881843/dns-issue-with-docker-image](https://askubuntu.com/questions/881843/dns-issue-with-docker-image)
