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
本教程安装环境为ubuntu 19.04

# 准备
在构建libbtbb和Ubertooth工具之前，需要预先安装一下编译环境，其中许多都可以从您的操作系统的安装源中中获得，例如：
## Debian/Ubuntu
```
sudo apt-get install cmake libusb-1.0-0-dev make gcc g++ libbluetooth-dev pkg-config libpcap-dev python-numpy python-pyside python-qt4
```
## Fedora / Red Hat
```
su -c "yum install libusb1-devel make gcc wget tar bluez-libs-devel"
```
## MAC OS
Mac OS X用户可以使用MacPorts或Homebrew来安装所需的软件包
```
brew install libusb wget cmake pkg-config libpcap
or
sudo port install libusb wget cmake python27 py27-numpy py27-pyside
```
## FreeBSD
FreeBSD用户可以直接从ports和package系统安装主机工具和库
```
sudo pkg install ubertooth
```
# 安装libbtbb
接下来需要为Ubertooth工具构建蓝牙基带库(libbtbb)进而解析蓝牙数据包。
```
wget https://github.com/greatscottgadgets/libbtbb/archive/2018-12-R1.tar.gz -O libbtbb-2018-12-R1.tar.gz
tar -xf libbtbb-2018-12-R1.tar.gz
cd libbtbb-2018-12-R1
mkdir build
cd build
cmake ..
make
sudo make install
```
注意：Linux用户如果是第一次安装或者出现无法找到libbtbb库的错误，请执行：
```
sudo ldconfig
```

## 安装ubertooth工具
Ubertooth存储库包含用于嗅探蓝牙数据包，配置Ubertooth和更新固件的主机代码。默认情况下，使用以下三种方法构建和安装：

## 实体编译
```
wget https://github.com/greatscottgadgets/ubertooth/releases/download/2018-12-R1/ubertooth-2018-12-R1.tar.xz
tar xf ubertooth-2018-12-R1.tar.xz
cd ubertooth-2018-12-R1/host
mkdir build
cd build
cmake ..
make
sudo make install
```
注意：Linux用户如果是第一次安装或者出现找不到库的错误，请执行：
```
sudo ldconfig
```

## wireshark
Wireshark版本1.12及更新版本默认包含Ubertooth BLE插件。也可以通将Ubertooth的BLE直接捕获到Wireshark中。

Wireshark BTBB和BR/EDR插件允许使用Kismet捕获的蓝牙基带流量在Wireshark GUI中进行分析和检测。它们与Ubertooth和libbtbb软件的其余部分分开构建。

传递给cmake的目录MAKE_INSTALL_LIBDIR因系统而异，但它应该是现有Wireshark插件的位置，例如asn1.so和ethercat.so。在macOS上目录可能位于/Applications/Wireshark.app/Contents/PlugIns/wireshark/
```
sudo apt-get install wireshark wireshark-dev libwireshark-dev cmake
cd libbtbb-2018-12-R1/wireshark/plugins/btbb
mkdir build
cd build
cmake -DCMAKE_INSTALL_LIBDIR=/usr/lib/x86_64-linux-gnu/wireshark/libwireshark3/plugins ..
make
sudo make install
```
BT BR/EDR组件重复相同的步骤进行安装
```
sudo apt-get install wireshark wireshark-dev libwireshark-dev cmake
cd libbtbb-2018-12-R1/wireshark/plugins/btbredr
mkdir build
cd build
cmake -DCMAKE_INSTALL_LIBDIR=/usr/lib/x86_64-linux-gnu/wireshark/libwireshark3/plugins ..
make
sudo make install
```
## 第三方软件
有许多支持Ubertooth的第三方软件。有些人支持Ubertooth开箱即用，而其他人则需要建立插件。
# 安装kismet
```
sudo apt install kismet
```
# 安装BLE解密工具crackle
crackle的地址是：https://github.com/mikeryan/crackle.git
## 安装
```
git clone https://github.com/mikeryan/crackle.git
cd crackle
make
make install
```

# 结尾
文章主要是个人参考官方安装文档女装常见的工具，为蓝牙测试做准备。

------
参考：https://github.com/greatscottgadgets/ubertooth/wiki/Build-Guide
