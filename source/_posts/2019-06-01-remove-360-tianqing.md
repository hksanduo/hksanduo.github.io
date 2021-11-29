---
title: "移除360天擎"
date:   2019-06-01  08:00:00 +0800
layout: post
tag: 
- OS
categories:
- Security
---

# 无密码卸载360天擎
------
不少甲方爸爸使用360天擎这款终端安全软件，为了接入客户的内网，被迫安装。安装后，电脑各种卡顿，卸载还需要密码，这就难受了，琢磨了一下，找出了两个卸载方法，供各位参考

## 方法一
### 找到360的安装目录
360天擎安装目录很好找，通常都安装在C盘的Program Files或者Program Files (x86)，如果你在安装的时候指定了安装目录请绕过此步
### 准备一个PE系统或者linux系统
由于360安装目录下的文件禁止被更改，个人学艺不精，不清楚如何在windows上修改对应的文件，只能通过PE或者linux系统对360的配置文件进行修改
### 修改配置文件
360的配置文件位于360\360Safe\EntClient\conf\ExtBase.dat，将其中的uipass置空即可
![360sage-config.png](/img/360sage-config.png)
## 方法二
### 粉碎相关配置文件
使用第三方的文件粉碎器，强行移除360\360Safe\EntClient\conf\ExtBase.dat文件
## 卸载
1、重新进入windows系统后，进入360的安装目录，找到360Safe\uninst.exe，点击运行，然后进行卸载即可
![360-uninstall.png](/img/360-uninstall.png)
2、或者直接使用控制面板里的程序和功能模块下的卸载或更改程序进行卸载
![360-uninstall-control.png](/img/360-uninstall-control.png)
## 重启
卸载完成以后需要重启，重启以后才能移除剩余文件

------
附：本文只是提供网友一个卸载方法，如有任何侵权，请及时联系本人。
