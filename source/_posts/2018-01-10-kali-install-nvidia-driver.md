---
title: "KaliLinux 安装Nvidia闭源驱动"
date:   2018-01-9 22:05:41 +0800
layout: post
tag: 
- Kali
categories:
- Kali
- Linux
---

## 查看主机是否存在nvidia显卡
```
lspci -k | grep -A 2 -E "(VGA|3D)"
```

>00:02.0 VGA compatible controller: Intel Corporation 3rd Gen Core processor Graphics Controller (rev 09)
	Subsystem: Lenovo 3rd Gen Core processor Graphics Controller
	Kernel driver in use: i915

>01:00.0 3D controller: NVIDIA Corporation GK208M [GeForce GT 740M] (rev a1)
	Subsystem: Lenovo GK208M [GeForce GT 740M]
	Kernel driver in use: nvidia

## 更新系统并重启
```
apt update && apt dist-upgrade -y
reboot
```
## 禁用nvidia开源驱动nouveau
```
vim /etc/modprobe.d/blacklist-libnfc.conf
```
在**blacklist-libnfc.conf**最后添加
```
blacklist nouveau
options nouveau modeset=0
```
## 安装 linux-headers
```
apt install linux-headers-$(uname -r)
```
## 安装nvidia驱动
```
apt-get install nvidia-kernel-dkms nvidia-cuda-toolkit nvidia-driver
```
## 安装bumblebee primus
> "Bumblebee 致力于使 NVIDIA Optimus 在 GNU/Linux 系统上可用，实现两块不同的供电配置的显卡同时插入使用，共享同一个 framebuffer。" 
```
apt install bumblebee-nvidia primus
```
开机自启动bumblebee服务
```
#systemctl enable bumblebeed
```
添加当前用户到bumblebee组
```
adduser root bumblebee
```
改变以下 bumblebee.conf 设置:
```
vim /etc/bumblebee/bumblebee.conf 
```
```
KeepUnusedXServer=true
Driver=nvidia
```
接下来修改 vim /etc/bumblebee/xorg.conf.nvidia
下面这条指令可以获取显卡的总线的ID
```
lspci -k | grep -A 2 -E "(VGA|3D)"
```
>00:02.0 VGA compatible controller: Intel Corporation 3rd Gen Core processor Graphics Controller (rev 09)
	Subsystem: Lenovo 3rd Gen Core processor Graphics Controller
	Kernel driver in use: i915

>01:00.0 3D controller: NVIDIA Corporation GK208M [GeForce GT 740M] (rev a1)
	Subsystem: Lenovo GK208M [GeForce GT 740M]
	Kernel driver in use: nvidia

我的笔记本nvidia的总线ID为01:00.0
```
Section "ServerLayout" 
    Identifier  "Layout0" 
    Option      "AutoAddDevices" "false" 
    Option      "AutoAddGPU" "false" 
EndSection 

Section "Device" 
    Identifier  "DiscreteNvidia" 
    Driver      "nvidia" 
    VendorName  "NVIDIA Corporation" 
 
#  If the X server does not automatically detect your VGA device, 
#  you can manually set it here. 
#  To get the BusID prop, run `lspci | egrep 'VGA|3D'` and input the data 
#  as you see in the commented example. 
#  This Setting may be needed in some platforms with more than one 
#  nvidia card, which may confuse the proprietary driver (e.g., 
#  trying to take ownership of the wrong device). Also needed on Ubuntu 13.04. 
    BusID "PCI:01:00:0" 
 
#  Setting ProbeAllGpus to false prevents the new proprietary driver 
#  instance spawned to try to control the integrated graphics card, 
#  which is already being managed outside bumblebee. 
#  This option doesn't hurt and it is required on platforms running 
#  more than one nvidia graphics card with the proprietary driver. 
#  (E.g. Macbook Pro pre-2010 with nVidia 9400M + 9600M GT). 
#  If this option is not set, the new Xorg may blacken the screen and 
#  render it unusable (unless you have some way to run killall Xorg). 
    Option "ProbeAllGpus" "false" 
 
    Option "NoLogo" "true" 
    Option "UseEDID" "false" 
    Option "UseDisplayDevice" "none" 
EndSection 
```
把BusID这一行的注释去掉,修改里面的PCI号(你获取到独显BUSID号码),值得注意的是,获取到的ID号最后一位必须为冒号.

也就是**PCI:01:00:0**
## 重启检测驱动是否安装成功
我这里主要使用hashcat破解密码，这里使用hashcat测试nvidia驱动是否安装成功
```
hashcat -I
```
也可以安装 **mesa-demos** 并使用 **glxgears** 测试 **Bumblebee** 是否工作：
```
$ optirun glxgears -info
```
如果失败，尝试下列命令:
64位系统:
```
$ optirun glxspheres64
```
32位系统:
```
$ optirun glxspheres32
```
如果一个内有动画的窗口出现，那么 Optimus 和 Bumblebee 正在工作。 
