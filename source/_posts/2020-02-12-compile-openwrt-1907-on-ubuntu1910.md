---
title: "在ubuntu19.10上编译openwrt19.07"
date:   2020-02-12  18:00:00 +0800
layout: post
tag:
- Openwrt
- Kernel
categories:
- IOT
- Linux
---

手上有一块任子行的NET110(88F6281)的路由器,还有一块环硕科技(公司已倒闭)的路由器,同样基于88F6281 A1,由于较为小众,固件需要自行编译,本文内容完全是作者自己在编译过程中的血泪史,如果你遇到同样的问题,可以一起交流.
------
## 准备
之前使用archlinux编译openwrt的估计,但是出现各种各样玄学问题,再加上梯子不稳定,后续直接转移到梯子上进行编译,虽说速度有点儿慢.
### 安装编译环境
```
apt update && apt distupgrade -y
apt install gcc g++ binutils patch bzip2 flex bison make autoconf gettext texinfo unzip sharutils subversion libncurses5-dev ncurses-term zlib1g-dev gawk  libssl-dev -y
```
### 下载openwrt并更新相关组件
```
https://github.com/openwrt/openwrt.git
```    
进入openwrt目录执行    
```
make prereq
./scripts/feeds update -a
./scripts/feeds install -a
```
### 配置编译内容
```
make defconfig
```
### * 增加自定义patch
部分平台比较小众,openwrt并为提供相关编译选项,需要自定添加patch,patch的位置在 **openwrt/target/linux/平台/patches-xx/** 目录下,这个需要根据实际情况进行更改,我这里的板子是kirkwood,在patches-4.14目录添加自己板子的patch,其他配置信息需要修改kirkwood目录下的base-files和image,这里我会在接下来的博文详细说明,这里不赘述,你可以直接将我配置好的patches添加到编译目录下,地址:

然后更新平台信息
```
make target/linux/clean V=99
make target/linux/prepare V=99
make target/linux/update V=99
```

感到奇怪,使用```make target/linux/refresh V=s -j99```总会出现使用4.19内核编译出现问题

## 编译
如果使用root权限编译,需要用到:
```
make V=s -j99 IGNORE_ERRORS=1 FORCE_UNSAFE_CONFIGURE=1
```

## 遇到的错误
### 1.WARNING: Makefile 'package/xx/xx' has a dependency on 'libxxx', which does not exist
不太清楚为何该lib包没有通过脚本自动安装,需要更新feeds手动安装
```
./scripts/feeds updata -a
 ./scripts/feeds install libxxx
```
终端显示 **Installing package 'libxxx' from packages** 完成安装

### 2.gcc: fatal error: Killed signal terminated program cc1
编译过程中gcc报错:gcc: fatal error: Killed signal terminated program cc1
大体上是因为内存不足,临时使用交换分区来解决吧
```
sudo dd if=/dev/zero of=/swapfile bs=64M count=16
sudo mkswap /swapfile
sudo swapon /swapfile
```
编译完成，可以取消交换分区：
```
sudo swapoff /swapfile
sudo rm /swapfile
```

### 3.compile devicetree.dts error: Unable to parse input tree
这个错误很可能是我们在写节点时多加了或者少写了“}”符号造成的，特别是节点里面包含了一些子节点的情况下，很容易少加“}”符号。

解决思路：由于gedit打开dts文件，dts内容的层次结构不好一眼看出，需要把dts内容copy到sourceinsight、UlterEdit或者其他文本工具里编辑，总之一点能让dts内容层次分明。这样一下就可以定位是那个节点存在少加或者多加‘}’符号。

## 参考
* [https://forum.openwrt.org/t/solved-build-error-make-toplevel-mk-world-error-2/47478](https://forum.openwrt.org/t/solved-build-error-make-toplevel-mk-world-error-2/47478)
* [https://blog.csdn.net/walker0411/article/details/51916959](https://blog.csdn.net/walker0411/article/details/51916959)
* [https://blog.csdn.net/flexman09/article/details/51862858?utm_source=itdadao&utm_medium=referral](https://blog.csdn.net/flexman09/article/details/51862858?utm_source=itdadao&utm_medium=referral)
* [https://www.cnblogs.com/hubery/p/4633863.html](https://www.cnblogs.com/hubery/p/4633863.html)
* [https://blog.csdn.net/renlonggg/article/details/53784509](https://blog.csdn.net/renlonggg/article/details/53784509)
* [https://openwrt.org/docs/guide-developer/quickstart-build-images](https://openwrt.org/docs/guide-developer/quickstart-build-images)
