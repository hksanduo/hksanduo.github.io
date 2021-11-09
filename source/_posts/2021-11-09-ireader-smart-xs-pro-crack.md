---
title: "掌阅ireader smart xs pro 安装第三方软件"
date:   2021-11-09  22:24:00 +0800
layout: post
tag:
- Android
categories:
- Security
---

掌阅ireader smart xs pro 安装第三方软件

------
## 背景
给媳妇买了一个掌阅ireader smart xs pro,比普通版本多花1k,就为了一个手写,这个败家娘们,不过喜欢就行,千金难买美人一笑,这就扯远了,主要是这玩意只能安装自己应用商城里面的软件,并且只有一个微信读书,这就很难受,网上查了一下,看到有人利用设备重启不断使用adb安装命令,部分设备可以安装成功,这个也是溜,不过我在smart xs上尝试了一下,有反应,但是报错了,有些玄学,这里就不折腾了,发现ireader设备就是一个安卓设备魔改的,并且可以设置代理,正好抓个包看看.

![20211109-01.jpg](/images/20211109-01.jpg)

注意:
	1.本文描述的操作对安全测试人员来说很简单,对普通用户有一定的技术门槛,请注意.
	2.本次绕过系统限制安装第三方软件仅限于ireader smart xs pro 型号为:SR801 系统版本号为:10.6.0.90 软件版本号为:6.10010.1090(109969),对于其他版本和平台并未测试
	3.绕过系统限制安装第三方软件这个bug已和官方同步,但是没啥反馈,估计官方也懒得修了,正好造福各位网友.

## 抓包分析
### 抓包配置
和正常安卓测试配置代理一样,通过wifi选项配置代理,接入个人无线网络,我这里代理服务器的ip为:192.168.3.209,代理端口为:8080
![20211109-02.jpg](/images/20211109-02.jpg)

![20211109-03.jpg](/images/20211109-03.jpg)

burpsuite配置
在代理服务器的burpsuite配置,监听192.168.3.209,端口为8080.
![20211109-04.png](/images/20211109-04.png)
点击http history,然后在ireader上点击几个需要网络同步的功能,看burpsuite上是否能截获ireader的请求包,以下截图就是获取到http相关请求包.
![20211109-05.png](/images/20211109-05.png)
如果获取不到,检查是否启用防火墙,配置是否正确等等,这里不过多阐述.

### 安装第三方软件
点击burpsuite拦截按钮,点击ireader设置-工具,在工具界面下,点击右上方的应用市场的按钮,你的burpsuite代理会捕获到一个请求包,你需要点击
![20211109-06.jpg](/images/20211109-06.jpg)


## 参考
- [http://store.ireader.com/product/index?type=Smart&id=383890&source=](http://store.ireader.com/product/index?type=Smart&id=383890&source=)【 ireader smart xs pro 】
- [https://www.coolapk.com/apk/com.x7890.shortcutcreator](https://www.coolapk.com/apk/com.x7890.shortcutcreator)【 创建快捷方式apk 】
- []()【 】
- []()【 】
- []()【 】
- []()【 】
- []()【 】
- []()【 】
- []()【 】
