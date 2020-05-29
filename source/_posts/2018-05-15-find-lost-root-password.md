---
title: "Linux root密码丢失解决方法"
date:   2018-05-15 12:00 +0800
layout: post
tag: 
- grub
categories:
- Linux
- os
---

Linux root密码丢失的解决办法

和UNIX系统相同，Linux超级用户root拥有系统的最高权限。当由于用户的疏忽，遗忘了root 密码，或者系统受到黑客的入侵，无法用root 账号登录系统时，可以通过下列办法来恢复root 的密码。
------
## 一、进入单用户模式

### 1.使用Linux 系统启动软盘

如果你已创建了Linux 系统的启动软盘，而且设置计算机系统从软盘启动，当显示boot 提示符后输入：

	boot: linux single

系统进入了提示符为“#”的单用户模式，计算机引导的运行级别为1，本地文件系统被挂载，很多系统服务没有运行，跳过所有系统认证，是一个系统管理员使用特定的机器，把 root 文件系统挂为读写，此时可以使用：

(1)passwd 程序来设置root的新密码
```
# passwd root
# reboot
```
重启系统后，root 密码已被更新。

(2)通过修改 /etc/shadow 文件，删除root 的密码
```
# cd /etc
# vi shadow
```
将以root 开头的一行中“root：”后和下一个“ ：”前的内容删除，
第一行将类似于“root ：：****”，保存后重启系统，root 密码置为空。

## 2.以LILO 多系统引导程序启动

当系统以LILO 引导程序启动时，在出现LILO 提示符时输入：

	LILO: linux single

进入单用户后，更改password 的方法同1。

4.3.以GRUB 多系统引导程序启动
用GRUB引导系统进入单用户步骤：

(1) 启动GRUB   ，然后键入 e 来编辑；

(2) 选择以kernel开头的一行，再按e 键，在此行的末尾，按空格键后输入single，以回车键来退出编辑模式；

(3) 回到了 GRUB 屏幕后，键入 b 来引导进入单用户模式。

进入单用户后，更改password 的方法同1。

## 二、使用Linux 系统安装盘

如果你既没做系统启动软盘，同时多系统的引导LILO 和GRUB 又被删除(如重装了Windows 系统后)，那么只能使用Linux 系统安装盘来恢复root 的密码。

用第一张Linux 系统安装盘启动，出现boot 提示符后输入：

	oot: linux rescue

此时系统进入救援模式，然后根据提示完成：

　　1.选择语言和键盘格式；

　　2.选择是否配置网卡，一般系统因网络不需要，所以可以选择否跳过网卡配置；

　　3 . 选择是否让系统查找硬盘上的Redhat Linux 系统，选择继续；

　　4.系统显示硬盘上的系统已经被找到，并挂载在/mnt/sysimage 下；

　　5.进入拯救状态，可重新设置root 的密码：
	
	# chroot/mnt/sysimage (让系统成为根环境)
	# cd /mnt/sysimage
	# passwd root
