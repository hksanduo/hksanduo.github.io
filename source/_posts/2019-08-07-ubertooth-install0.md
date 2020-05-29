---
title: "ubertooth安装"
date:   2019-08-08  17:32:00 +0800
layout: post
tag:
- bluetooth
categories:
- Security
- IOT
---

Ubertooth安装教程
------
# 前言
本教程安装环境为ubuntu 19.04，ubertooth固件为2018-12-R1，由于ubertooth首次安装使用，
为了更好的适配最新的应用程序，请及时更新对应的固件

# 准备
获取最新的ubertooth发行版，地址：[https://github.com/greatscottgadgets/ubertooth/releases](https://github.com/greatscottgadgets/ubertooth/releases)
解压以后，点击进入`ubertooth-one-firmware-bin`目录
![image](https://hksanduo.github.io/images/ubertooth-one-firmware.png)
# 更新固件
在对应目录下执行以下命令`ubertooth-dfu -d bluetooth_rxtx.dfu -r`
```
$ ubertooth-dfu -d bluetooth_rxtx.dfu -r
Checking firmware signature
........................................
........................................
........................................
.
Detached
```
设备将自动进入DFU模式并刷新固件。

如果最后看到control message unsupported，这意味着重置设备失败。您可以通过运行ubertooth-util -r或从Ubertooth拔下USB电缆并重新连接来解决。

# 检查是否更新成功
在非DFU模式下，您可以使用以下方式获取固件信息ubertooth-util -v。请注意，显示的版本应与您刚刚安装的版本匹配：
```
$ ubertooth-util -v
Firmware version: 2018-12-R1 (API:1.06)
$ ubertooth-util -V
ubertooth 2018-12-R1 (mikeryan@steel) Tue Dec  4 22:23:44 PST 2018
```

# 结尾
文章主要是个人参考官方安装文档女装常见的工具，为蓝牙测试做准备。至于固件的研发和更新固件遇到错误，请读者参考官方文档和解决方案。

------
参考：[https://github.com/greatscottgadgets/ubertooth/wiki/Firmware](https://github.com/greatscottgadgets/ubertooth/wiki/Firmware)
