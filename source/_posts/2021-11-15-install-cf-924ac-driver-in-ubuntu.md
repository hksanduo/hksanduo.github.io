---
title: "Ubuntu 安装 COMFAST CF-924AC V2 USB网卡驱动"
date:   2021-11-15  16:57:00 +0800
layout: post
tag:
- Driver
categories:
- Linux
---

Ubuntu 安装 COMFAST CF-924AC V2 USB网卡驱动 

------
## 背景
公司只有无线网络，服务器一直用一个2.4GHz的无线网卡进行访问，但是网络速度和质量有些垃圾，网上买了个USB千兆无线网卡，支持2.4GHz和5.8Ghz，在安装的过程中发现官方源中找不到相关的linux驱动，全网检索，也找不到这张网卡linux驱动，去官网查了一下，发现官方就不提供linux版本驱动，这就有点儿尴尬了。
![20211115-01.jpg](/img/20211115-01.jpg)
![20211115-03.jpg](/img/20211115-03.jpg)
ubuntu自动识别这个USB网卡为：RTL88x2bu [AC1200 Techkey]，
![20211115-02.png](/img/20211115-02.png)
我尝试直接编译安装RTL88x2bu相关驱动模块，发现可以识别这个网卡。以下是相关安装步骤：

## 编译安装
```
git clone https://github.com/cilynx/rtl88x2bu
```
编译安装
```
make 
sudo make install
sudo modprobe 88x2bu
```
执行以上命令，未遇到相关错误，到这一步，你的USB网卡已经可以加载上去了，你可以通过命令行查看：
```
ifconfig
```
![20211115-04.png](/img/20211115-04.png)
也可以直接通过配置图形界面去查看，我这里没截图，就这样吧。

## 后续
尝试加载到dkms，失败了，感觉短时间解决不了，可以成功加载网卡，使服务器可以连接到网络，已经满足我的要求了，无线质量不错，但是网速一般，有空得排查一下原因。

## 参考
- [https://github.com/Tencent/secguide](https://github.com/Tencent/secguide)【腾讯安全编码指南】
- [https://github.com/cilynx/rtl88x2bu](https://github.com/cilynx/rtl88x2bu)【rtl88x2bu】