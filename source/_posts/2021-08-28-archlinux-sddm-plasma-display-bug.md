---
title: "plasma使用扩展屏幕无法加载任务栏bug"
date:   2021-08-28  11:54:00 +0800
layout: post
tag:
- Plasma
categories:
- Linux
---

plasma使用扩展屏幕无法加载任务栏bug

------
## 背景
由于笔记本的屏幕过于垃圾,并且有点儿小,所以就把外接的显示器当作主屏幕,并在plasma的配置中将笔记本屏幕关闭,然后就会出现一个玄学的问题,每次开机进入桌面,任务栏消失了,也无法通过win键唤起Application Launcher,这就很尴尬,之前以为是plasma自身的bug导致的,网上也没找到相关方法,当时由于工作比较忙,也没有时间来捣鼓它.过去了很长时间...此处省略一万字,主要是比较懒,plasma更新了好几个版本,这个问题还是没有解决,个人感觉可能这个是自己配置的问题,所以周末看看,捣鼓一下,个人设备配置如下:

![20210828-01.png](/images/20210828-01.png)

bug相关截图如下:

sddm加载桌面截图:
![20210828-07.jpg](/images/20210828-07.jpg)

进入plasma,无法加载任务栏,可以使用快捷键.
![20210828-02.jpg](/images/20210828-02.jpg)

使用截图工具,截取整个屏幕,仍然是一张桌面图片,无任务栏,和分辨率显示应该没问题
![20210828-06.png](/images/20210828-06.png)

系统配置中已将笔记本屏幕关闭
![20210828-04.png](/images/20210828-04.png)

## 原因分析
通过分析xorg日志信息,系统加载日志信息,sddm日志信息,均未发现有任何异常.每次启动,如果移除外接显示器,直接使用笔记本屏幕,plasma桌面加载正常.如果同时使用笔记本和外接显示器,两个屏幕都会加载sddm登录界面,在一个屏幕输入完用户名和密码进入plasma桌面,与此同时笔记本屏幕关闭,外界显示器显示主界面,但是无法显示任务栏,个人感觉可能是在加载桌面的过程中笔记本屏幕和外接显示器的配置出现了冲突,导致plasma无法正常加载任务栏.

## 尝试解决
在sddm的启动脚本中,使用xrandr命令强制指定外接显示器的分辨率并关闭笔记本屏幕
首先使用xrander命令列举当前显示器信息
```
xrandr --listmonitors
```
![20210828-03.png](/images/20210828-03.png)

在:/usr/share/sddm/scripts/Xsetup脚本中新增以下内容
```
xrandr --output *HDMI1 --mode 1920x1080
xrandr --output eDP1 --off
```
重新加载桌面,发现plasma加载正常
![20210828-05.png](/images/20210828-05.png)

## 总结
到这里,我只能通过在sddm中禁用笔记本屏幕,目前只是规避了这个问题,具体原因尚未可知,哪位大佬如果清楚可以私聊讨论一下.

## 参考
- [https://wiki.archlinux.org/title/KDE](https://wiki.archlinux.org/title/KDE)【archlinux wiki--plasma】
- [https://wiki.archlinux.org/title/Xrandr](https://wiki.archlinux.org/title/Xrandr)【archlinux wiki--xrandr】
