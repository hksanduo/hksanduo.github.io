---
title: "grub命令"
date:   2018-05-15 12:00 +0800
layout: post
tag: 
- grub
categories:
- Linux
- OS
---

grub命令
------


## 一、菜单命令

菜单命令只能用于grub配置文件的全局配置部分，不能用在grub命令行交互界面，菜单命令在配置文件中应放在其它命令之前。
```           
1.default //设置默认启动的菜单项
2.fallback //设置启动某菜单项失败后反回的菜单项
3.hiddenmenu //隐藏菜单界面
4.timeout //设置菜单自动启动的延时时间
5.title //开始一个菜单项
```
## 二、常规命令

常规命令可以应该于配置文件和grub命令行交互界面，可使用的常规命令有
```
1.bootp //通过bootp初始化网络设备
2.color //设置菜单界面的颜色
3.device //指定设备文件作为驱动器
4.dhcp //通过DHCP初始化网络设备
5.hide //隐藏某分区
6.ifconfig //手工配置网络设备
7.pager //改变内部页程序的状态
8.partnew //新建一个主分区
9.parttype //改变分区的类型
10.password 为菜单界面设置口令
11.rarp //通过RARP初始化网络设置
12.serial //设置串口设备
13.setkey //设置键盘映射
14.splashimage //设置GRUB启动时的背景图片文件
15.termainal //选择终端类型
16.tftpserver //指定TFTP服务器
17.unhide //还原某隐藏分区
```
## 三、命令行和菜单项命令

命令行和菜单项命令可应该于GRUB配置文件的菜单项设置中，也可以用在GRUB命令交互界面。
```
1.bolcklist //显示某文件所在分区位置（block list notation）
2.boot //启动操作系统
3.cat //显示文件内容
4.chainloader //把启动控制权软交给另外的启动引导器
5.cmp //比较两个文件
6.configfile //加载已存在的GRUB配置文件
7.debug //设置为debug模式
8.displayapm //显示APM BIOS信息
9.displaymem //显示内存配置
10.embed //嵌入Stage 1.5文件
11.find //查找包括某文件的所有设备
12.fstest //测试文件系统
13.geometry //显示某驱动器的物理信息
14.halt //停止计算机运行（软件关机）
15.help //显示GRUB的命令帮助信息
16.impsprobe //查询对称多处理器（SMP）的信息
17.initrd //加载initrd文件
18.install //安装GRUB
19.ioprobe //查询某驱动器的输入输出（I/O）端口
20.kernel //引导操作系统内核
21.lock //锁定某GRUB导菜单项，使其输入密码后才可启动
22.makeactive //激活某主分区
23.map //虚拟映射某驱动器
24.md5crypt //使用MD5加密口令
25.module //加载模块
26.modulenounzip //加载模块不进行解压
27.pause //暂停并等待按键
28.quit //退出GRUB
29.reboot //重新启动计算机
30.read //读取内存中的内容
31.root //设置GRUB的root设备
32.rootnoverify //设备GRUB的root设备但不装载文件系统
33.savedefault //保存当前的启动菜单项为默认启动
34.setup //自动安装GRUB
35.testload //从文件系统中测试读取某文件
36.testvbe //测试VESA BIOS EXTENSION
37.uppermem //强制设置主机上位内存的大小
38.vbeprobe //查询VESA BIOS EXTENSION信息 
```
