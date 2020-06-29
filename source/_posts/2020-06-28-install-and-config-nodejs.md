---
title: "windows安装配置nodejs"
date:   2020-06-28  14:10:00 +0800
layout: post
tag:
- Nodejs
categories:
- Dev
---

windows 安装配置nodejs

-------
## 背景
最近碰到几个nodejs的代码审计项目，之前在archlinux上配置过nodejs环境，基本上就是一句命令的事，由于疫情
出差就带了一个笔记本，懒得远程折腾了，种种原因，还是在自己平时工作的windows笔记本搭建一下nodejs环境，本文
纯属扯淡，大家略过就行。
## 下载
首先到官网上下载nodejs，这里以最新的为例。
![20200628-nodejs-01.png](/images/20200628-nodejs-01.png)
## 安装
没有明显需要注意的地方，一直下一步就行，注意安装位置
## 配置
### 1、修改NodeJs数据存放的路径
![20200628-nodejs-02.png](/images/20200628-nodejs-02.png)
创建两个文件夹：node_cache、node_data
![20200628-nodejs-03.png](/images/20200628-nodejs-03.png)
### 2、配置存储路径
打开CMD执行以下命令
```
npm config set prefix D:\Program\nodejs\node_data
npm config set cache D:\Program\nodejs\node_cache
```
![20200628-nodejs-04.png](/images/20200628-nodejs-04.png)
如果什么不显示,则表示修改成功

### 3、配置环境变量
修改用户变量把系统的npm指定改为如下```C:\Users\Administrator\AppData\Roaming\npm 改为 D:\Program\nodejs\node_data```
![20200628-nodejs-05.png](/images/20200628-nodejs-05.png)

添加系统变量 NODE_PATH 路径为 D:\Program\nodejs\node_data\node_modules
![20200628-nodejs-06.png](/images/20200628-nodejs-06.png)

注意这俩个指向需要改为我们刚刚新建的俩个文件夹 , 路径不要拷贝上面的 ,写你自己保存的地方，配置完成后点击保存。
测试，判断是否配置陈工
执行以下指令判断是否配置成功
```
node -v
npm -v
```
![20200628-nodejs-07.png](/images/20200628-nodejs-07.png)

输入 node -v 与 npm -v 这一次我们没有在 nodejs 文件夹中运行这俩个命令 ,如果命令能够正确输出则表示环境配置成功

### 4、配置NodeJs的国内镜像
运行以下指令，为nodejs配置阿里的镜像源
```
npm install ‐g cnpm ‐‐registry=https://registry.npm.taobao.org
```
![20200628-nodejs-08.png](/images/20200628-nodejs-08.png)
检查是否配置成功
![20200628-nodejs-09.png](/images/20200628-nodejs-09.png)



## 参考
- [https://nodejs.org/zh-cn/](https://nodejs.org/zh-cn/)【nodejs官网】
- [https://blog.csdn.net/qq_40646143/article/details/103237095](https://blog.csdn.net/qq_40646143/article/details/103237095)【NodeJs卸载,安装与配置国内镜像】
