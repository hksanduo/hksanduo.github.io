---
title: "mqtts 介绍"
date:   2024-06-24  14:36:00 +0800
layout: post
tag:
- Car
categories:
- Security
---

# mqtts 介绍

------

## 背景介绍
最近在学习车联网相关知识，碰巧媳妇公司CTF题目中有一道车联网的题目，题目方向是MQTT，正好可以深入研究一下。

## mqtss介绍
mqtss是一款mqtt安全测试工具，github地址：https://github.com/SPuerBRead/mqtts

主要功能包括如下：

* 匿名登陆 (批量)
* emqx embox_plugin_template
* 任意用户名密码登陆 (批量)
* 用户名密码爆破 (批量)
* 获取服务端信息
* 尽可能获取所有topic信息
* 获取证书信息

## 使用
支持协议类型
-----------
* TCP
* SSL
* WS
* WSS

使用说明
-----------
自动探测（包含匿名登陆、任意用户名密码登陆、用户名密码爆破，此处爆破使用的是内置密码字典）

`./mqtts -t 127.0.0.1 -p 1883 -au`
比如EXMQ默认允许匿名登录，测试结果如下：
![匿名登录测试](/img/20240624-03.png)

指定字典，不指定字典会从本地自动加载username.txt和password.txt两个文件。
`./mqtts -t 127.0.0.1 -p 1883 -b -nf username.txt -pf password.txt`

获取服务端信息

`./mqtts -t 127.0.0.1 -p 1883 -s`

获取topic信息

`./mqtts -t 127.0.0.1 -p 1883 -ts -w 60`

批量测试

`./mqtts -tf ./target.txt -au`

其他参数见 `./mqtts -h`

批量扫描文件格式(空格分割，*必填项)

*host *port protocol clientId username password

## 实际测试
1、通过端口扫描，发现目标开放1883端口，使用NMAP等进行端口扫描
> 功能强大的nmap是支持MQTT协议的识别的，可以直接通过nmap进行识别MQTT协议。另外，除上面提到的默认端口外，有的管理员会修改默认端口，这时也可以尝试1884，8084，8884等临近端口以进行快速探测，或前面增加数字等作为组合，如果针对单个目标，则可以探测全部端口。如果进行大规模的扫描或者提升扫描效率，则可以使用masscan、zmap、RustScan等先进性端口扫描，在使用nmap进行协议识别即可。
> nmap举例命令如下：`sudo nmap -p1883,8083,8883 -sS -sV --version-intensity 9 -Pn --open target_ip`

![nmap scan example](/img/20240624-02.png)

2、使用mqtts测试目标
![mqtts实际测试效果](/img/20240624-01.jpg)

爆破没啥可以讲解的了，就看谁的字典强大了，也可以使用mqtts内置的用户名和密码字典。

3、该项目多年没有维护了，对于topic，获取服务信息已经无效，个人测试的mqtt服务端是EMQX，仅供参考。


## 参考
* https://github.com/SPuerBRead/mqtts
* https://www.anquanke.com/post/id/212335
