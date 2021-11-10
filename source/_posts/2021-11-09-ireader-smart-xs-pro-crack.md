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

通过分析,发现在安装软件过程中,通过伪造响应包,替换响应包中appName和apk下载路径,由于ireader安装程序未校验下载的apk及相关内容,因此可以通过篡改响应数据诱导ireader安装程序安装自定义url的软件包.

### 构造apk下载服务HTTP
我这里直接使用docker启动了一个apache的容器,搭建了一个文件下载服务器,相关命令如下:
```
docker run -dit --name apache2 -p 12345:80 -v /data/apache:/usr/local/apache2/htdocs/ httpd
```
> 这里映射宿主机的目录为:/data/apache,如果新人搭建,注意目录是否拥有权限,搭建成功的标识是你访问: http://your ip address:12345 会列举出:/data/apache目录下的所有文件
也可以搭建其他http的文件下载服务,类型不限,总之能通过url下载apk包即可,这里需要注意的是apk包需要压缩成zip格式.
![20211109-12.png](/images/20211109-12.png)


### 安装第三方软件
点击burpsuite拦截按钮,点击ireader设置-工具,在工具界面下,点击右上方的应用市场的按钮
![20211109-06.jpg](/images/20211109-06.jpg)

你的burpsuite代理会捕获到一个请求包,你需要右击,选择Do intercept-> Reponse to this request,这是一个很典型的修改相应数据包的操作.
![20211109-08.png](/images/20211109-08.png)

点击Forward,转发数据包.将响应的数据包中的appName进行替换,替换成你想安装的安卓软件包的名称,如何获取软件包的名称,请参考aapt工具的使用,我这里简洁带过.
![20211109-09.png](/images/20211109-09.png)

> 比如我们要安装创建快捷方式这个apk,查看apk的命令是:``` aapt dump badging ShortcutCreator_*.apk ```
![20211109-10.png](/images/20211109-10.png)
创建快捷方式apk的appName为:com.x7890.shortcutcreator,替换以后的格式为
![20211109-11.png](/images/20211109-11.png)
点击forward,我们将伪造好的数据包转发给ireader.这时候ireader存储的微信读书的appName已经被篡改了.
![20211109-13.jpg](/images/20211109-13.jpg)

> 注意:这里有个小技巧,如果操作不熟练,可以将整个json数据在记事本中修改好了再复制到burpsuite中

点击下载按钮,弹出提示对话框,点击确定,这时候burpsuite会捕获到一个请求包,和上述同样的操作,需要篡改响应包
![20211109-14.jgp](/images/20211109-14.jpg)
burpsuite请求包截图
![20211109-15.png](/images/20211109-15.png)
burpsuite响应包:
![20211109-16.png](/images/20211109-16.png)

burpsuite响应包中将appName替换成 ```com.x7890.shortcutcreator```,appurl替换成我们实现构造好的apk url地址,我这里的地址为:```http://192.168.3.209:12345/ShortcutCreator_1.17-199043-o_1d2ncpumt4f4tu0daf1qkb15i310-uid-906093.zip``` 
![20211109-17.png](/images/20211109-17.png)
点击forward,转发数据包,在次期间,如果速度比较慢,可能会有多个相同的请求,需要执行相同的步骤伪造相应包,相应包的内容均相同.
如果伪造成功了,你的ireader就会自动下载apk包并进行安装,如果失败,查找原因,再尝试一下以上步骤.


最终结果:
![20211109-07.jpg](/images/20211109-07.jpg)


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
