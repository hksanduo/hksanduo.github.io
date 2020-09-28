---
title: "导入centos7虚拟机出现dracut-initqueue timeout"
date:   2020-09-10  10:37:00 +0800
layout: post
tag:
- VM
categories:
- Linux
---

导入centos7虚拟机出现dracut-initqueue timeout

-------
## 背景
拷贝centos7虚拟机导入到客户的虚拟集群，导入过程不多讲，一起正常，在启动过程中，出现 **dracut-initqueue timeout**，通过分析启动日志，发现内核无法加载硬盘，dracut提供shell中工具不多，也无法定位问题具体出在那里，由于磁盘使用lvm的形式，个人猜测可能是由于硬件发生变化，缺乏驱动，导致无法启动。重启以后使用单用户模式登录系统，一些正常，可以直接挂在磁盘并引导系统，可能某些模块无法加载导致的，个人太菜，实在无法定位。

![dracut-initqueue timeout](/images/20200910-01.png)

## 修复
重启系统，grub选择救援模式（recure)选项，进入救援模式，然后执行
```
dracut --force --no-hostonly
```
后续在官方的文档上发现：
> The dracut command can be used to modify the contents of your initramfs. For example, if you are going to move your hard drive to a new computer, you might want to temporarily include all drivers in the initramfs to be sure that the operating system can load on the new computer. To do so, you would run the following command:
> ```
> # dracut --force --no-hostonly
> ```
> The force parameter tells dracut that it is OK to overwrite the existing initramfs archive. The no-hostonly parameter overrides the default behavior of including only drivers that are germane to the currently-running computer and causes dracut to instead include all drivers in the initramfs.

移动硬盘到一台新的设备上，可能驱动出现问题，使用force参数覆盖initramfs，使用no-hostonly参数，仅加载与当前系统运行相关驱动模块。我使用以上指令，重构initramfs,重启以后发现仍然无法正常启动。我这里使用了加密模块，对磁盘进行加密，开机自动解密磁盘，可能未加载解密密钥。我执行一下指令，重构一下initramfs，重启后可以正常进入系统
```
# dracut --force
```
## 注意
对于dracut-initqueue timeout错误，互联网检索到的都是加载centos系统安装盘出现问题，和本文提到现象相同但是不是同一个问题，所以解决方法也不同。

## 参考
- [https://fedoramagazine.org/initramfs-dracut-and-the-dracut-emergency-shell/](https://fedoramagazine.org/initramfs-dracut-and-the-dracut-emergency-shell/)【initramfs dracut and the dracut emergency shell】

