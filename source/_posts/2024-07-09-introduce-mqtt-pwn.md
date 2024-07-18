---
title: "mqtt-pwn 介绍"
date:   2024-07-09  14:36:00 +0800
layout: post
tag:
- Car
categories:
- Security
---

# mqtt-pwn 介绍

------

## 背景介绍
最近在学习车联网相关知识，碰巧媳妇公司CTF题目中有一道车联网的题目，题目方向是MQTT，发现一个mqtt测试工具：mqtt-pwn，这里做记录和学习一下。

## mqtt 安全风险相关
* 授权：匿名连接问题，匿名访问则代表任何人都可以发布或订阅消息。如果存在敏感数据或指令，将导致信息泄漏或者被恶意攻击者发起恶意指令；
* 传输：默认未加密，则可被中间人攻击。可获取其验证的用户名和密码；
* 认证：弱口令问题，由于可被爆破，设置了弱口令，同样也会存在安全风险；
* 应用：订阅端明文配置导致泄漏其验证的用户名和密码；
* 漏洞：服务端软件自身存在缺陷可被利用，或者订阅端或服务端解析内容不当产生安全漏洞，这将导致整个系统。

| id | 漏洞详情 |
| -- | --- |
| 1 | 1、Eclipse Mosquitto 1.0～1.5.5 存在授权问题漏洞 漏洞公告：https://www.anquanke.com/vul/id/1514742 |
| 2 | 1、Eclipse Mosquitto <= 1.4.15 存在拒绝服务漏洞 漏洞公告：http://www.nsfocus.net/vulndb/40015 |
| 3 | 2、 Eclipse Mosquitto 1.0～1.5.5存在访问控制漏洞 漏洞公告：https://www.anquanke.com/vul/id/1514740 |
| 4 | 3、 MQTT 3.4.6之前版本和4.0.5之前的4.0.x版本存在拒绝服务漏洞 漏洞公告：https://www.anquanke.com/vul/id/1132531 |
| 5 | 4、 MQTT protocol 3.1.1版本中存在安全漏洞 漏洞公告：https://www.anquanke.com/vul/id/2051251 |

## mqtt-pwn 介绍
MQTT-PWN是Akamai Threat Research 开发的一个针对物联网（IoT）Broker进行渗透测试和安全评估的强大工具集。但是项目当年未维护，无法直接运行，重新修改后地址如下：https://github.com/hksanduo/mqtt-pwn


### 构建
```
docker-compose up --build --detach
```

### 使用

#### 客户端运行
```
docker-compose run cli
```
![启动界面](/img/20240709-01.png)

#### MQTT-PWN 使用
功能模块支持：
* 爆破（Credentials Brute Force）
* 执行命令（Command & Control）
* 连接（Connect to a Broker）
* 信息收集（Infromation Grabber）
* GPS泄露检测（Owntracks(GPS Tracker)）
* Sonofff测试（Sonoff Exploiter）
* topic探测枚举（Enumeration）


![help](img/20240709-02.png)

#### 使用注意事项

1、MQTT-PWN不支持EMQX，主要原因是EMQX对返回的数据做了处理，如果获取不到MQTT Server数据的小伙伴不要慌张，对于EMQX可以使用的功能是凭证爆破，也可以用mqtts工具测试，上一篇博客有介绍。 

2、MQTT-PWN的原理也较为简单，测试EMQX之外的服务，尽可能工具扫描和手动测试结合，主要是探测Topic。 

3、此处就不一一介绍mqtt-pwn的使用了，各位可以参考官方wiki，写的很详细：[https://mqtt-pwn.readthedocs.io/en/latest/intro.html](https://mqtt-pwn.readthedocs.io/en/latest/intro.html)


## 参考
* https://github.com/SPuerBRead/mqtts
* https://www.anquanke.com/post/id/212335
* https://mqtt-pwn.readthedocs.io/en/latest/intro.html
* https://www.secpulse.com/archives/160231.html