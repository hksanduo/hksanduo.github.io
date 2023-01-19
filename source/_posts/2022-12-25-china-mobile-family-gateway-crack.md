---
title: "中国移动光猫h5-9家庭网关重置管理员密码并桥接网络"
date:   2022-12-25  16:57:00 +0800
layout: post
tag:
- Iot
categories:
- Security
---

中国移动光猫h5-9家庭网关重置管理员密码并桥接网络

------

## 背景
首先吐槽一下中国移动的宽带服务，这里基于帝都其他省份未知，对比联通而言，联通资费较贵，每个月资费大概比移动贵三四十元，但是网络稳定，访问国际网络较为流畅，并且可以获取公网ipv4/ipv6地址，这点相当给力，并且套餐比较灵活，随用随开，随走随停，反观移动，如果使用移动赠送的宽带，会有每个月75的最低消费限制，并且强制要求在网两年，如果使用正常资费的宽带，价格对比联通略微便宜，但是安装费用跑不了，无公网ipv4，但是支持ipv6,对于一个搞IT，这点就很难受，也挺恶心，想换网，还得再忍受两年垃圾带宽。之前提供的移动光猫只给了user密码，无管理员密码，联系客服和安装师傅无果后，只能用软路由做二级路由进行管理，期间也尝试使用网上爆出的超级管理员用户名和密码进行测试，均失败，终于通过一个未修复的漏洞，成功重置了管理员密码，破解方案参考网友enihsyou，并进行了光猫桥接，访问米国测速站点，成功将300+ms的ping值降到300ms以下，这里记录一下，方便其他网友参考。       
这里不得不再吐槽一下，这个光猫厂家可能给电信也做过，电信的界面和相关功能也没删除，测试的时候给调试出来了，看着天翼的logo当场懵逼。

## 获取光猫管理员密码
### 启动telnet
1、访问192.168.3.1，默认用户名为user，密码如果没修改，自行查阅光猫背后的密码进行登录        
![光猫信息](/img/20221225-01.png) 
2、在当前页面地址栏输入 `http://192.168.1.1/usr=CMCCAdmin&psw=aDm8H%25MdA&cmd=1&telnet.gch` 点开启telnet，点保存
3、使用telnet并用 `telnet 192.168.1.1` 配合用户名 `CMCCAdmin` 密码 `aDm8H%MdA` 登录     
![telnet](/img/20221225-02.png) 

### 重置下发的CMCCAdmin管理员密码
再输入 `su` ，密码为user的密码，切换到超级用户,输入以下代码覆盖下发的超级用户用户名和密码（其实只要密码的部分就够了）
```shell
sidbg 1 DB set DevAuthInfo 0 User CMCCAdmin
sidbg 1 DB set DevAuthInfo 0 Pass aDm8H%MdA
```
会回显带`*`的XML，无视它，密码已经修改成功了，直接去192.168.1.1使用用户名 `CMCCAdmin` 密码 `aDm8H%MdA` 进行登录。     
![密码重置](/img/20221225-03.png) 

另外可以直接解密看原文
```shell
sidbg 1 DB decry /userconfig/cfg/db_user_cfg.xml
grep Password /tmp/debug-decry-cfg
```

## 光猫桥接
我们在网络菜单中，找到网络配置，记录一下vlan id，       
![光猫PPPOE默认配置](/img/20221225-04.png) 

光猫桥接配置如下：

修改完成点击保存即可，这里需要注意，光猫桥接以后，就无法通过二级路由进行访问光猫的管理界面，这点需要注意，一定确认好再保存，这里需要绑定光猫的lan1口，可以通过其他lan口管理光猫。

## openwrt 配置
之前配置openwrt为二级路由，这里需要重新配置，我这里就不在网页上配置了，直接修改配置文件，其中需要注意，默认固件中会有一个wan接口，还有一个wan6，方便ipv4和ipv6管理，但是如果使用pppoe进行桥接拨号，会生成一个wan_6的虚拟动态接口，这里冲突了，需要将WAN6移除。
网络配置如下，修改 `/etc/config/network `:
```
config interface 'lan'
        option device 'br-lan'
        option proto 'static'
        option ipaddr '192.168.3.1'
        option netmask '255.255.255.0'
        option ip6assign '64'

config interface 'wan'
        option device 'eth1'
        option proto 'pppoe'
        option username 'xxxxx'
        option password 'xxxxx'
        option ipv6 'auto'
```
DHCP配置如下，修改` /etc/config/dhcp`
```
config dhcp 'lan'
        option interface 'lan'
        option start '100'
        option limit '150'
        option ra_management '1'
        list dhcp_option '6,192.168.3.1,223.5.5.5'
        option leasetime '12h'
        option ra 'server'
        option dhcpv6 'server'

config dhcp 'wan'
        option interface 'wan'
        option ignore '1'
        option start '100'
        option limit '150'
        option leasetime '12h'
```
DHCP修改，主要是为了配置ipv6相关内容，ipv4使用默认配置即可，这里配置ipv6的ra,dhcp为服务器模式，禁用NDP。使用以下命令重启网络：
```
/etc/init.d/network restart
```
自此，光猫桥接和ipv6的配置已完成，所有设备可以获取到ipv6地址，可以使用：https://test-ipv6.com/ 和https://ipw.cn/来测试，也可以使用使用以下命令：```curl ipv6.ip.sb ```，看是否可以获取到ipv6地址，或者直接ping。

需要注意，如果只能单向ping通，通常是防火墙禁止转发，目前只能调通icmp相关规则    
![禁止ipv6 ping 防火墙规则](/img/20221225-05.png)    
启用以后就会无法从公网进行ping，ping的结果为：Destination unreachable: Port unreachable   

![禁止ipv6 ping ](/img/20221225-06.png) 

## ddns
ddns这块可以有很多种方法，基本原理相同，通过获取公网IP地址，通过dns服务商接口修改域名解析的ip，个人写的python脚本，shell脚本，或者其他同学写的插件，这里推荐使用luci-app-ddns-go这个插件，主要是比较方便，直接可以从openwrt的软件源进行安装，也可以自行编译安装，说明文档也比较详细，个人比较推荐。

群晖cloudflare的脚本需要进行修改，具体可以参考，joshuaavalon的脚本，参考连接在最后，这里发现一个bug，群晖的ddns管理模块有时候无法获取到ipv6地址，这时候就歇菜了，个人直接修改脚本，使用以下命令，直接获取ipv6地址，修改后的脚本如下：
```
#!/bin/bash
set -e;

ipv4Regex="((25[0-5]|(2[0-4]|1{0,1}[0-9]){0,1}[0-9])\.){3,3}(25[0-5]|(2[0-4]|1{0,1}[0-9]){0,1}[0-9])"

proxy="false"

# DSM Config
username="$1"
password="$2"
hostname="$3"
#ipAddr="$4"

ipAddr=$(curl -s -X GET http://ipv6.ip.sb)

if [[ $ipAddr =~ $ipv4Regex ]]; then
    recordType="A";
        echo "nochg";
        exit 0 ;
else
    recordType="AAAA";
fi

listDnsApi="https://api.cloudflare.com/client/v4/zones/${username}/dns_records?type=${recordType}&name=${hostname}"
createDnsApi="https://api.cloudflare.com/client/v4/zones/${username}/dns_records"

res=$(curl -s -X GET "$listDnsApi" -H "Authorization: Bearer $password" -H "Content-Type:application/json")
resSuccess=$(echo "$res" | jq -r ".success")

if [[ $resSuccess != "true" ]]; then
    echo "badauth";
    exit 1;
fi

recordId=$(echo "$res" | jq -r ".result[0].id")
recordIp=$(echo "$res" | jq -r ".result[0].content")

if [[ $recordIp = "$ipAddr" ]]; then
    echo "nochg";
    exit 0;
fi

if [[ $recordId = "null" ]]; then
    # Record not exists
    res=$(curl -s -X POST "$createDnsApi" -H "Authorization: Bearer $password" -H "Content-Type:application/json" --data "{\"type\":\"$recordType\",\"name\":\"$hostname\",\"content\":\"$ipAddr\",\"proxied\":$proxy}")
else
    # Record exists
    updateDnsApi="https://api.cloudflare.com/client/v4/zones/${username}/dns_records/${recordId}";
    res=$(curl -s -X PUT "$updateDnsApi" -H "Authorization: Bearer $password" -H "Content-Type:application/json" --data "{\"type\":\"$recordType\",\"name\":\"$hostname\",\"content\":\"$ipAddr\",\"proxied\":$proxy}")
fi

resSuccess=$(echo "$res" | jq -r ".success")

if [[ $resSuccess = "true" ]]; then
    echo "good";
else
    echo "badauth";
fi
```
脚本修改了两点，一是禁用了ipv4解析，二是通过ipv6.ip.sb获取nas公网ipv6地址，仅供参考。

## todo
- [] ipv6防火墙配置
- [] 

## 参考：
- [https://ipw.cn/](https://ipw.cn/)【ipv6测试】
- [https://test-ipv6.com/](https://test-ipv6.com/)【ipv6测试】
- [https://gist.github.com/enihsyou/24bdff2d1e19dd332de0493ee491ff04](https://gist.github.com/enihsyou/24bdff2d1e19dd332de0493ee491ff04)【获取移动光猫 H2-2 管理员密码 ver.2022-04】
- [https://github.com/sirpdboy/luci-app-ddns-go](https://github.com/sirpdboy/luci-app-ddns-go)【luci-app-ddns-go】
- [https://www.ioiox.com/archives/105.html](https://www.ioiox.com/archives/105.html)【群晖cloudflare ddns配置】
- [https://raw.githubusercontent.com/joshuaavalon/SynologyCloudflareDDNS/master/cloudflareddns.sh](https://raw.githubusercontent.com/joshuaavalon/SynologyCloudflareDDNS/master/cloudflareddns.sh)【cloudfalre ddns 脚本】

