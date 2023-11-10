---
title: "在nexus添加pipy代理源"
date:   2023-11-10  17:00:00 +0800
layout: post
tag:
- ML
categories:
- Linux
---

# 安装nexus并且添加pipy代理源

------

## 背景
使用机器学习，测试不同项目时，依赖都不同，之前也尝试使用复制conda 环境等方式来加快环境的部署，但是torch等包动辄几百兆，使用tuna等代理源也比较耗费时间并且占用带宽，为了应对这个问题，需要在内部搭建一个代理源，方便数据的共用，减少带宽的占用，写这篇文章主要原因是网络上对于配置pypi代理源的资料太少，博主在配置的过程中处处碰壁，这里记录一下，方便以后查阅。我这里会从nexus的安装讲起，方便大家安装和使用。

## nexus安装
我这里使用的是docker进行部署，方便配置和运维。docker-compose.yaml如下：
```
version: '3.8'

services:
  nexus:
    image: 'sonatype/nexus3'
    container_name: 'nexus'
    restart: always
    privileged: true
    environment: 
      - TZ=Asia/Shanghai
    ports:
      - 48081:8081
    volumes:
      - './nexus-data:/nexus-data'
    networks:
      - nexus

networks:
  nexus:
    driver: bridge
```
我这里做了数据持久化操作，将数据保存到本地。
启用直接运行命令 `docker-compose up -d` 容器会在后台进行运行。稍等片刻，访问：http://{your server ip}:48081
![nexus](/img/20231110-01.png)

默认用户名为admin，密码为：admin123，如果发现密码不对，`cat ./nexus-data/admin.password`,首次进入系统后，会提示修改密码，注意修改即可。

## 增加pypi代理源

仓库类型说明：
* hosted: 本地仓库，通常我们会部署自己的构件到这一类型的仓库。比如公司的私有库。    
* proxy: 代理仓库，它们被用来代理远程的公共仓库，如maven中央仓库。      
* group:仓库组，用来合并多个hosted/proxy仓库，当你的项目希望在多个repository使用资源时就不需要多次引用了，只需要引用一个group即可。     

blob stores
在创建repository之前，还需要先指定文件存储目录，便于统一管理。就需要创建Blob Stores，不创建则使用的是default，blob stores相当于一个存储空间，如果没有特殊要求，使用默认的即可。

点击创建源
![创建源](/img/20231110-02.png)

选择pypi proxy
![创建源](/img/20231110-03.png)

设置代理源的名称，选择blob stores，我这里直接默认，各位师傅根据实际情况配置，点击`create repository`创建源。
![创建源](/img/20231110-04.png)

点击我们新创建的源pypi
![创建源](/img/20231110-05.png)

进入源配置页面，我们将tuna源(`https://pypi.tuna.tsinghua.edu.cn/`)加进来，加入源需要注意，需要去掉`simple`，代理源地址nexus已经默认生成，默认是：`http://ip:port/repository/pypi/`，pip配置的时候，需要将`simple`拼接到最后，最后配置的代理源地址为：`http://ip:port/repository/pypi/simple` 我们只需要修改远程的存储库或者镜像源即可，点击保存即可完成配置。
![配置源](/img/20231110-06.png)

如果新人使用默认配置，到这里应该没问题了，但是对于部分单位，存在多个用户，多个私有源，我们可能访问失败，这里需要配置权限。由于是代理源，直接使用匿名用户权限修改即可。
![配置源](/img/20231110-07.png)

添加权限，修改完成，点击保存即可。
![添加权限](/img/20231110-08.png)

## 本地pip配置
linux下，使用代理源，我们通常有临时方法和永久方案(写入配置文件当中)，比如：`pip install -i https://pypi.tuna.tsinghua.edu.cn/simple some-package`,但是对于我们内网配置的代理源会有一个小问题，pip信任这个源，所以无法安装，这个问题就是pip对于非https的源不会进行安装。
![ not a trusted or secure host](/img/20231110-09.png)
解决方法，可以增加参数，`--trusted-host`，如下：
![ 临时使用源命令 ](/img/20231110-10.png)
修改配置文件，linux位于：~/.config/pip/pip.conf，配置文件举例如下：
```
[global]
index-url = http://192.168.3.199:48081/repository/pypi/simple
[install]
trusted-host = 192.168.3.199
```
也可以直接使用命令行：
```
pip config set global.index-url http://192.168.3.199:48081/repository/pypi/simple
pip config set install.trusted-host 192.168.3.199
```
![ 配置效果 ](/img/20231110-11.png)

## 总结

1、安装正确
2、选取合适的镜像源，比如：tuna，ustc，商业的都行
3、权限配置正确
4、注意pip配置

祝大家炼丹快乐。

## 参考
- [https://mirrors.tuna.tsinghua.edu.cn/help/pypi/](https://mirrors.tuna.tsinghua.edu.cn/help/pypi/)【tuna mirrors pypi】
- [https://zhuanlan.zhihu.com/p/526011309](https://zhuanlan.zhihu.com/p/526011309)【docker安装nexus3，搭建私人maven仓库】
- [https://blog.csdn.net/BThinker/article/details/123688143](https://blog.csdn.net/BThinker/article/details/123688143)

