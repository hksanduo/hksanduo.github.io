---
title: "openwrt 配置ipv6"
date:   2020-03-13  10:03:00 +0800
layout: post
tag:
- Network
- Openwrt
categories:
- Linux
---

openwrt 配置ipv6

-------
## 背景
之前一直只有教育网或者局部地区测试的IPv6现在已经大范围推开，大部分地区的ISP均正确部署了IPv6。通常来说，Openwrt获取IPv6的方式有三种：中继、穿透和NAT，由于ISP已经提供了IPv6和某些方案的缺陷，择优采取中继的方案。

## 准备
首先将光猫的模式调为桥接或者混合模式，然后通过电脑拨号确认ISP是否已经正确配置IPv6。
然后升级路由器的Openwrt的版本，最好不要低于17.01，否则odhcpd可能会出现问题，当然更老的版本也能正确获取IPv6，不过可能需要每隔一段时间就重启一次odhcpd。

## 配置
从Openwrt 15.xx（即CC版本）开始，默认的初始设置中就会含有wan6，无需安装其它软件包。
由于Openwrt默认分配IPv6私网网段，首先应该删除网络>接口页面内IPv6 ULA 前缀配置自动生成的fd开头的/64随机IPv6地址段并保存生效。其实这个时候，在较新版本的Openwrt上面应该已经成功获取了IPv6。
然后我们需要修改/etc/config/dhcp文件，使用无状态地址自动配置（SLAAC）IPv6，而不是DHCPv6。
为了保险期间，首先需要备份dhcp配置文件，以便遇到问题进行回滚。
```
cp /etc/config/dhcp /etc/config/dhcp.backup
```
然后修改dhcp的配置示例如下：
```
config dhcp 'lan'
        option interface 'lan'
        option start '100'
        option limit '150'
        option leasetime '12h'
        option dhcpv6 'relay'
        option ra 'relay'
        option ndp 'relay'
        option ra_management '1'

config dhcp 'wan'
        option interface 'wan'
        option ignore '1'
        option dhcpv6 'disabled'
        option ndp 'relay'
        option ra 'relay'
        option master '1'

config dhcp 'wan6'
        option dhcpv6 'relay'
        option ra 'relay'
        option ndp 'relay'
        option master '1'
```
配置完成之后需要重启network服务，以便接入终端获取IPv6地址：

```
/etc/init.d/network restart
```
至此所有的客户端包括路由器均可获得可用的IPv6地址，不过在部分操作系统上dhcp不会马上获取到ipv6，需要手动刷新一下：
Windows下面，需要在CMD中执行如下命令：
```
ipconfig /release6
ipconfig /renew6
```

在Linux下面，可以重启dhcp或者NetworkManager服务
```
sudo systemctl restart dhcpcd
```
或者
```
sudo systemctl restart NetworkManager
```
每个发行版本重启网络的方式不一样，请根据实际情况刷新网络
![20200313-ip-addr.png](/images/20200313-ip-addr.png)

## 体验
体验较好，可以访问ipv6站点。

## 参考内容
- [http://blog.kompaz.win/2017/02/22/OpenWRT%20IPv6%20%E9%85%8D%E7%BD%AE/](http://blog.kompaz.win/2017/02/22/OpenWRT%20IPv6%20%E9%85%8D%E7%BD%AE/)
- [https://openwrt.org/docs/guide-user/network/ipv6/start](https://openwrt.org/docs/guide-user/network/ipv6/start)
- [https://linkthis.me/2018/12/04/ipv6-on-openwrt/](https://linkthis.me/2018/12/04/ipv6-on-openwrt/)
