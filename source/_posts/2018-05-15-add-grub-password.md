---
title: "如何为GRUP系统引导管理器添加密码"
date:   2018-05-15 12:00 +0800
layout: post
tag: 
- grub
categories:
- Linux
- os
---

如何为GRUB系统引导管理器加上密码
摘要：本文主要是讲述就如何为GRUB系统引导管理器加上密码，只要输入密码才能使用GRUB来引导系统；仅限于桌面系统上的应用，不能用 于远程管理的服务器上；我们总不会为了系统安全，重启服务器后，要跑到机房输入GRUB的密码吧；GRUB有两种加密方法，一种是明口令，另一种是md5 口令加密；
------

## 一、GRUB 明口令加密；

比如我没有设置密码之前/etc/grub是如下的样子：
```
default=1
timeout=10
splashimage=(hd0,7)/boot/grub/splash.xpm.gz
title Fedora Core (2.4.22-1.2061.nptl)
root (hd0,7)
kernel /boot/vmlinuz-2.4.22-1.2061.nptl ro root=LABEL=/
initrd /boot/initrd-2.4.22-1.2061.nptl.img
title WindowsXP
rootnoverify (hd0,0)
chainloader +1
```
加入以后就是下面这样的：
```
default=1
timeout=10
splashimage=(hd0,7)/boot/grub/splash.xpm.gz
password=123456
title Fedora Core (2.4.22-1.2061.nptl)
lock
root (hd0,7)
kernel /boot/vmlinuz-2.4.22-1.2061.nptl ro root=LABEL=/
initrd /boot/initrd-2.4.22-1.2061.nptl.img

title WindowsXP
rootnoverify (hd0,0)
chainloader +1
```
从上面的可以看出，GRUB的密码是123456，lock的意思就是把Redhat Fedora锁住了。如果启动时会提示错误。这时就应该按P键，然后输入密码就行了。我设置的是123456，当然应该输入123456了，输入别的密码肯定不能通过，这样是不是做到保密了呢？？


## 二、GRUB 的md5加密方法；

经jerboa兄指教，我又读了一下GRUB文档，的确感觉到用md5加密校验GRUB密码比较安全。为了也能让和我一样菜的弟兄，也能知道如何通过md5进行GRUB密码加密，我不得不把这个教程写出来。哈哈，高手就是免读了，此文为菜鸟弟兄所准备。
用md5加密校码GRUB密码，这样会更安全。


## 1、用grub-md5-crypt成生GRUB的md5密码；

通过grub-md5-crypt对GRUB的密码进行加密码运算，比如我们想设置grub的密码是123456，所以我们先要用md5进行对123456这个密码进行加密
```
[root@linux01 beinan]# /sbin/grub-md5-crypt
Password: 在这里输入123456
Retype password: 再输入一次123456
$1$7uDL20$eSB.XRPG2A2Fv8AeH34nZ0
```
$1$7uDL20$eSB.XRPG2A2Fv8AeH34nZ0 就是通过grub-md5-crypt进行加密码后产生的值。这个值我们要记下来，还是有点用。


## 2、更改 /etc/grub.conf

比如我原来的/etc/grub.conf文件的内容是下面的。
```
default=1
timeout=10
splashimage=(hd0,7)/boot/grub/splash.xpm.gz
title Fedora Core (2.4.22-1.2061.nptl)
root (hd0,7)
kernel /boot/vmlinuz-2.4.22-1.2061.nptl ro root=LABEL=/
initrd /boot/initrd-2.4.22-1.2061.nptl.img
title WindowsXP
rootnoverify (hd0,0)
chainloader +1
```
所以我要在/etc/grub.conf中加入 password --md5 $1$7uDL20$eSB.XRPG2A2Fv8AeH34nZ0 这行，以及lock，应该加到哪呢，请看下面的更改实例；
```
timeout=10
splashimage=(hd0,7)/boot/grub/splash.xpm.gz
password --md5 $1$7uDL20$eSB.XRPG2A2Fv8AeH34nZ0
title Fedora Core (2.4.22-1.2061.nptl)
lock
root (hd0,7)
kernel /boot/vmlinuz-2.4.22-1.2061.nptl ro root=LABEL=/
initrd /boot/initrd-2.4.22-1.2061.nptl.img

title WindowsXP
rootnoverify (hd0,0)
chainloader +1
```
我们仔细看一下，从上面的我们改过的/etc/grub.conf中是不是已经用到了我们在第一步通过/grub-md5-crypt所产生的密码呢？？是不是有点安全感了？？
