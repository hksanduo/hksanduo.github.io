---
title: "grub vs grub2 bootloader"
date:   2019-11-12  23:00:00 +0800
layout: post
tag:
- grub
categories:
- Linux
- Security
- OS
---

grub和grub2对比
------
# 前言
在本文中，我提供了一些对Linux引导加载程序GRUB（统一引导加载程序）的一些了解。如果您了解GRUB的工作模式，那么这篇文章可以帮助您更好的了解操作系统。如果您对在Linux上工作充满信心，那么您应该掌握GRUB引导加载程序。GRUB可以轻松地与DOS，Windows，Linux或任何BSD操作系统一起使用。

Grub引导加载程序可以动态配置，这意味着用户可以选择在引导时进行更改。甚至用户也可以轻松地更改当前的引导条目，他们可以添加新条目，选择多个内核，甚至可以修改initrd。GRUB还获得了逻辑块地址的支持。GRUB可以从任何类型的设备（例如硬盘，CD和USB）安装和执行

# GRUB和GRUB2是两个不同的版本。
GRUB2被视为Ubuntu的默认引导加载程序，而GRUB通常用于RHEL较旧的版本中。启动时，GRUB2主要显示一个菜单并等待用户的一些输入。它通常将控制权转移到我们的操作系统内核。GRUB2的主要设计目的是为当今的操作系统提供灵活性和性能。

# GRUB和GRUB2
GRUB2的默认菜单看起来与GRUB非常相似，但是其中进行了一些更改。以下是个人总结的一些异同点,仅供参考:
  * Grub有两个配置文件，即menu.lst和grub.conf，而Grub2只有一个主要配置文件，即grub.cfg，它看起来非常接近完整的脚本语言。每当添加或删除内核或用户运行update-grub时，此配置文件都会被某些Grub 2软件包更新所覆盖。对于任何配置更改，我们都需要运行update-grub来使更改生效。
  * 在Grub1中，普通用户确实很难修改配置。但是Grub2更加用户友好，Grub-mkconfig将自动更改配置。
  * 在grub1分区号从0开始，而在Grub2中，分区号从1开始。第一个设备仍用hd0标识。如果需要，可以通过对/ etc / grub文件夹的device.map文件进行一些更改来更改这些更改。
  * Grub1使用物理和逻辑地址来寻址磁盘，甚至无法从新的分区读取它，而Grub2使用UUID来标识磁盘，因此更加可靠。它支持LVM和RAID设备。
  * 在包括（Ubuntu 和RHEL ）的当今Linux发行版中，GRUB2现在将直接显示登录提示，并且现在不显示菜单。
  * 如果要在引导过程中查看菜单，则需要按住SHIFT键。即使有时按ESC也可以显示菜单。
  * 用户现在还可以选择创建自定义文件，在其中可以放置自己的菜单项。可以使用/etc/grub.d文件夹中的40_custom文件进行配置。
  * 现在，用户甚至也可以更改菜单显示设置。这是通过修改/ etc / default文件夹下的grub文件实现的。

------
参考：https://zh.wikipedia.org/wiki/GNU_GRUB
