---
title: "在星际蜗牛上安装ESXI 6.7 及openwrt"
date:   2021-11-06  19:50:00 +0800
layout: post
tag:
- openwrt
categories:
- Linux
---

在星际蜗牛J1900平台上上安装ESXI 6.7及openwrt

------
## 背景
之前做NAS，替换出一块电源和一块星际蜗牛的主板，从咸鱼上淘了一个千兆USB网卡，就组成了一个简单的X86软路由，由于固态容量只有16G，安装一个openwrt虚拟机磁盘就占的的差不多了，最近想扩容一下固态，在重新安装ESXI和openwrt过程中，之前遇到的坑又走了一遍，所以记录下来，供以后参考。

## 准备

* ESXI 6.7 安装镜像(我这里用的是VMware-VMvisor-Installer-6.7.0.update03-14320388.x86_64.iso)
* Ventoy安装优盘一个，也可以是烧录的光盘和优盘
* openwrt vmdk 虚拟磁盘一个
* ESXI 6.7 RTL8157 驱动

> Ventoy是一个制作可启动U盘的开源工具。有了Ventoy你就无需反复地格式化U盘，你只需要把 ISO/WIM/IMG/VHD(x)/EFI 等类型的文件直接拷贝到U盘里面就可以启动了，无需其他操作。可以一次性拷贝很多个不同类型的镜像文件，Ventoy 会在启动时显示一个菜单来供你进行选择。Ventoy 安装之后，同一个U盘可以同时支持 x86 Legacy BIOS、IA32 UEFI、x86_64 UEFI、ARM64 UEFI 和 MIPS64EL UEFI 模式，同时还不影响U盘的日常使用。Ventoy 支持大部分常见类型的操作系统 （Windows/WinPE/Linux/ChromeOS/Unix/VMware/Xen ...）

## ESXI6.7 安装
### 引导安装
引导ventoy，选择esxi安装镜像
![20211106-01.jpg](/images/20211106-01.jpg)

进入esxi安装引导
![20211106-04.jpg](/images/20211106-04.jpg)

进入VMware加载界面，倒计时5s的时候，按 Shift+O 键添加启动参数：ignoreHeadless=TRUE
![20211106-05.png](/images/20211106-05.png)

![20211106-02.jpg](/images/20211106-02.jpg)
接着直接按回车，完成ESXI的安装，直到重启。安装完重启之后使用同样的方法插入启动参数来进入系统，然后永久配置。
![20211106-03.jpg](/images/20211106-03.jpg)

> 这里的主要解决卡顿的坑：
> J1900/N3150因为核显的问题不能直接装ESXI，开机会卡在Relocating modules and starting up the kernel，忘了截图了，图片来源于互联网。
> ![20211106-06.jpg](/images/20211106-06.jpg)
> 增加ignoreHeadless=TRUE即可，临时解决本次引导卡顿问题，后续安装完毕ESXI，还需要永久配置。

注意一点：可以先将ESXI接入一个私有网络下面，给ESXI配置一个静态IP，方便以后管理；也可以给ESXI配置一个静态IP，用网线把ESXI主机和自己的电脑连接起来，比如我的ESXI配置的IP是192.168.3.110/24 网关是：192.168.3.1，个人PC配置的ip是192.168.3.100/24,网关是192.168.3.1，这样在PC上直接访问: https://192.168.3.110 ,即可访问到ESXI的管理界面。
![20211106-08.png](/images/20211106-08.png)

### 永久配置ignoreHeadless
1、启用ssh
点击 操作-》服务-》启用安全SHELL(SSH) 选项，启用ssh   
![20211106-07.png](/images/20211106-07.png)   
使用XSHELL访问对应的ip，输入root用户名及密码登录上去。   
![20211106-09.png](/images/20211106-09.png)   

2、配置
执行以下命令进行配置：
```
esxcfg-advcfg -k TRUE ignoreHeadless
```
执行以下命令判断是否配置成功：
```
esxcfg-advcfg -j ignoreHeadless
```
返回TRUE说明成功了
![20211106-10.png](/images/20211106-10.png)
我这里直接通过启用ssh，远程进行配置，也可以直接启用ESXI SHELL进行配置。 

### 安装ESXI 6.7 RTL8157 驱动
我这块板子上只有一个千兆网卡，别问我为什么不使用双网口的板子，问就是穷，穷人家的板子早当家。    
![20211106-11.jpg](/images/20211106-11.jpg)  

ESXI默认不包含8157的驱动，需要自行安装。下载链接详见附件，根据ESXI的版本进行下载。
![20211106-12.png](/images/20211106-12.png)

我这里使用tftp进行上传，也可以使用scp，上传上去即可，放到/tmp目录下。
![20211106-13.png](/images/20211106-13.png)

使用unzip命令解压驱动压缩包
![20211106-14.png](/images/20211106-14.png)
使用以下指令安装驱动。
```
esxcli software vib install -v /tmp/vib20/vmkusb-nic-fling/VMW_bootbank_vmkusb-nic-fling_2.1-6vmw.670.2.48.39203948.vib --no-sig-check
```
![20211106-16.png](/images/20211106-16.png)

> 这里有个坑，离线安装不加--no-sig-check参数，可能存在签名验证失败导致安装失败的问题，这里由于是直接从官网上下载的，使用 --no-sig-check 参数忽略掉。
> ![20211106-15.png](/images/20211106-15.png)

安装完成以后需要重启系统，重启系统以后，在网络-》物理网卡界面下发现USB网卡已经成功加载
![20211106-17.png](/images/20211106-17.png)

### 配置网络
1、增加一个虚拟交换机，点击网络-》虚拟交换机-》添加标准虚拟交换机，增加一个名为vSwitch1的虚拟交换机，上行链路选择vusb0
![20211106-18.png](/images/20211106-18.png) 
点击添加
![20211106-19.png](/images/20211106-19.png)

2、增加一个端口组，点击网络-》端口组-》添加端口组，添加一个名为VM Network1的端口组，虚拟交换机选择vSwitch1。
![20211106-20.png](/images/20211106-20.png)

至此，ESXI网络已经配置完成，主板上的网口准备配置成lan，USB网口准备配置成wan。

## 安装Openwrt虚拟机
openwrt x86虚拟机镜像我是从其他地方下载的，这里就不提供下载地址了，建议使用openwrt官方的，可以使用工具将img格式的镜像转换成ESXI支持的vmdk格式。这里就不提供详细的安装步骤了，重点强调一下几个需要注意的地方：
1、配置磁盘
首先需要移除系统默认添加的磁盘，然后点击添加硬盘-》现有硬盘，选择合适的位置，点击上载，将openwrt虚拟磁盘文件(vmdk)上传上来并选择。
![20211106-22.png](/images/20211106-22.png)

![20211106-23.png](/images/20211106-23.png)

2、配置网络
需要再增加一个网络适配器，选择VM Network1端口组。
![20211106-21.png](/images/20211106-21.png)

配置完成，启动虚拟机，我这里的虚拟机默认的管理ip是192.168.1.1，但是我需要配置192.168.3.1，这是需要从ESXI提供的控制台进入openwrt的虚拟终端进行配置，配置位置位于：/etc/config/network

![20211106-24.png](/images/20211106-24.png)
使用以下命令重启网络，使配置生效：
```
/etc/init.d/network restart
```
浏览器访问：192.168.3.1，可以成功访问到openwrt的管理界面：
![20211106-25.png](/images/20211106-25.png)

## 总结
以前玩软路由，都是基于各种各样的板子编译对应的估计，我花费时间最长的一次是编译88F6281的一块板子固件，花费了大概有两个多月，主要原因包括：板子资料太少，编译不熟练，网络编译环境不兼容等等问题，最终结果是好的，但是总感觉投入与产出负相关，这大概是自己转入X86平台的原因。贴一张自己软路由的照片，加了一个风扇，主要是夏天有些热，怕散热不行影响性能，当时直接拿亚克力板用简易的工具做了一个支撑壳，将就能用。
![20211106-26.png](/images/20211106-26.png)

## 参考
- [https://blog.jemnluo.top/%E8%BD%AF%E8%B7%AF%E7%94%B1%EF%BC%88J1900-N3150%EF%BC%89%E5%AE%89%E8%A3%85ESXI%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%8D%A1%E5%B1%8F%E8%A7%A3%E5%86%B3/#%E6%B7%BB%E5%8A%A0%E5%90%AF%E5%8A%A8%E5%8F%82%E6%95%B0](https://blog.jemnluo.top/%E8%BD%AF%E8%B7%AF%E7%94%B1%EF%BC%88J1900-N3150%EF%BC%89%E5%AE%89%E8%A3%85ESXI%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%8D%A1%E5%B1%8F%E8%A7%A3%E5%86%B3/#%E6%B7%BB%E5%8A%A0%E5%90%AF%E5%8A%A8%E5%8F%82%E6%95%B0)【软路由（J1900/N3150）安装ESXI虚拟机卡屏解决】
- [https://flings.vmware.com/usb-network-native-driver-for-esxi](https://flings.vmware.com/usb-network-native-driver-for-esxi)【USB Network Native Driver for ESXi】
- [https://www.right.com.cn/forum/thread-506950-1-1.html](https://www.right.com.cn/forum/thread-506950-1-1.html)【矿渣之蜗牛星际(j1900 4盘位NAS)在ESXI6.7安装软路由LEDE 保姆级教程】
- [https://linhongbo.com/posts/openwrt-on-esxi/](https://linhongbo.com/posts/openwrt-on-esxi/)【ESXi 安装&配置 OpenWrt】