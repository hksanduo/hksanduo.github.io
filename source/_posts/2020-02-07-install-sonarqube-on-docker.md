---
title: "使用docker安装sonarqube"
date:   2020-02-07  15:23:00 +0800
layout: post
tag:
- Docker
categories:
- Security
- Code Audit
---

使用docker安装sonarqube
------
## 背景
使用docker部署相关应用,方便,省事,对系统影响较小,拥有开箱即用等优点,总之很香,对比互联网上找到的教程,或多或少有些问题,
有的甚至存在误导的嫌疑,扯这么多,其实看的最多也可能是自己.废话不多说了,开始安装.

## 准备
个人使用的平台是archlinux,版本号为:5.4.15-arch1-1,使用的docker版本为19.03.5-ce,需要安装的Sonarqube和PostgreSQL版本均为latest(截止到今天),Sonarqube对应的版本为7.9.2,PostgreSQL对应的版本为:12.1,参考的同学注意文章的实效性.
* 注意:如果在获取docker镜像时速度缓慢,尝试使用中科大的镜像站,或者使用厂商的加速器均可.

## 安装
### 安装sonarqube和postgres
在终端中输入以下执行,获取相应的docker镜像.
```
docker pull postgres:latest
docker pull sonarqube:latest
```
### 创建一个PostgreSQL Docker容器
Sonarqube依赖于数据库才能正常工作，在这里，我们选择PostgreSQL。下面的命令作用是创建用户名为sonar，密码为sonar的PostgreSQL实例并运行，在mynet容器网络中将主机端口5432与容器端口5432绑定在一起。
```
docker run --name sonar-postgres -e POSTGRES_USER=sonar -e POSTGRES_PASSWORD=sonar -d -p 5432:5432  postgres
```

### 创建一个Sonar Qube Docker容器
配置系统：
```
sysctl -w vm.max_map_count=262144
sysctl -w fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
```
这里我们使用使用宿主机的IP加上容器暴露出的端口号来通信,我的宿主机ip为192.168.3.200
以下命令的作用主要是通过JDBC连接PostgreSQL数据库创建并运行SonarQube实例。将主机端口9000绑定到mynet容器网络内部的容器端口9000。
```
docker run --name sonarqube -d -p 9000:9000 -e SONARQUBE_JDBC_USERNAME=sonar -e SONARQUBE_JDBC_PASSWORD=sonar -e SONARQUBE_JDBC_URL=jdbc:postgresql://192.168.3.200:5432/sonar sonarqube
```
### 查看是否安装成功
执行```docker ps```
![docker-ps.png](/img/20200207-docker-ps.png)
查看sonar和postgres实例进程是否正常

## 访问
在宿主机上直接访问  http://localhost:9000, 账户和密码均为admin.如果需要开放宿主机端口供其他主机访问,请使用iptables或者firewall-cmd自行增加防火墙规则,这里就不在赘述了.
![sonarqube-login.png](/img/20200207-sonar-login.png)

## 遇到的坑
### 1.使用容器自定义网络,sonarqube jdbc连接失败.
我最初使用官方提供的方式,设置容器网络进行通信,但是sonarqube使用jdbc链接postgres数据库时总会出现连接数据库失败的提示,目前我没有找到原因,只能使用固定ip进行访问,有点儿失败.
![sonarqube-connect-error.png](/img/20200207-sonarqube-connect-error.png)

以下是我安装的步骤,如果哪位大佬有解决方法,烦请不吝赐教,毕竟使用固定ip访问数据库不是一件长久的事情.
#### 为sonarqube和postgres创建相应的容器网络
为了提高Sonar和Postgres容器之间的通信，我们创建一个Docker Network。以下命令将创建一个名为mynet的容器网络。
```
docker network create mynet
```

#### 创建一个PostgreSQL Docker容器
Sonarqube依赖于数据库才能正常工作，在这里，我们选择PostgreSQL。下面的命令作用是创建用户名为sonar，密码为sonar的PostgreSQL实例并运行，在mynet容器网络中将主机端口5432与容器端口5432绑定在一起。
```
docker run --name sonar-postgres -e POSTGRES_USER=sonar -e POSTGRES_PASSWORD=sonar -d -p 5432:5432 --network mynet postgres
```

#### 创建一个Sonar Qube Docker容器
以下命令的作用主要是通过JDBC连接PostgreSQL数据库创建并运行SonarQube实例。将主机端口9000绑定到mynet容器网络内部的容器端口9000。
```
docker run --name sonarqube -p 9000:9000 -e SONARQUBE_JDBC_USERNAME=sonar -e SONARQUBE_JDBC_PASSWORD=sonar -e SONARQUBE_JDBC_URL=jdbc:postgresql://sonar-postgres:5432/sonar -d --network mynet sonarqube
```

### 2.elasticsearch 无法启动
bootstrap checks failed主要原因是elasticsearch启动失败,elasticsearch需要的vm.max_map_count至少为262144
![20200207-bootstrap-checks-failed.png](/img/20200207-bootstrap-checks-failed.png)

```
max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```
解决方法,通过在root权限用户下执行以下指令:
```
sysctl -w vm.max_map_count=262144
```

### 3.防火墙
docker会在firewalld的规则列表中增加一个docker的域（zone），并且docker0网卡默认是关联到这个docker域下面的，部分系统如果安装docker,默认启用的域会被修改成docker,如果需要开放端口，方便其他人远程访问，需要使用指定域为docker。可参考以下指令：

```
firewall-cmd --add-port=9000/tcp --permanent --zone=docker
firewall-cmd --reload
```
需要注意：在启动sonarqube服务器过程中，如果未设置数据库，sonarqube的web服务会正常启动，如果配置了数据库，但是由于种种原因，数据库无法被访问到，sonarqube的web服务是失效的，在调试防火墙的过程中先确定当前sonarqube是否成功运行，可以使用指令 ``` docker logs ${sonarqube容器名称} ``` 来查看sonarqube容器启动日志。连接错误日志如下：

![20200207-fail-to-connect-to-database.png](/img/20200207-fail-to-connect-to-database.png)

## 进阶
最近为了方便，也免得之前的做法扰乱大家的思路，我直接构建了一个docker-componse.yml，方便大家直接构建。docker-componse.yml如下：
```
version: '3'

services:
  postgres-db:
    image: postgres:latest
    container_name: sonar-postgres
    ports:
      - 5432:5432
    volumes:
      - ./data/postgres:/data/postgres
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: 1qaz@WSX
      PGDATA: /data/postgres
    networks:
      - sonarqube
    restart: unless-stopped

  sonarqube:
    image: sonarqube:latest
    container_name: sonarqube
    ports:
      - 9000:9000
    depends_on:
      - postgres-db
    environment:
      SONARQUBE_JDBC_USERNAME: sonar
      SONARQUBE_JDBC_PASSWORD: 1qaz@WSX
      SONARQUBE_JDBC_URL: jdbc:postgresql://postgres-db:5432/sonar
    volumes:
      - ./data/sonarqube/sonarqube_data:/opt/sonarqube/data
      - ./data/sonarqube/sonarqube_extensions:/opt/sonarqube/extensions
      - ./data/sonarqube/sonarqube_logs:/opt/sonarqube/logs
    networks:
      - sonarqube
    restart: unless-stopped

networks:
  sonarqube:
    driver: bridge

```
使用sonarqube和postgres结合的方式，数据库的用户名为：sonar,密码各位看官自行配置即可，数据已做了本地映射

## 参考
* [https://hub.docker.com/_/sonarqube](https://hub.docker.com/_/sonarqube)
* [https://birdben.github.io/2017/05/02/Docker/Docker%E5%AE%9E%E6%88%98%EF%BC%88%E4%BA%8C%E5%8D%81%E4%B8%83%EF%BC%89Docker%E5%AE%B9%E5%99%A8%E4%B9%8B%E9%97%B4%E7%9A%84%E9%80%9A%E4%BF%A1/](https://birdben.github.io/2017/05/02/Docker/Docker%E5%AE%9E%E6%88%98%EF%BC%88%E4%BA%8C%E5%8D%81%E4%B8%83%EF%BC%89Docker%E5%AE%B9%E5%99%A8%E4%B9%8B%E9%97%B4%E7%9A%84%E9%80%9A%E4%BF%A1/)
* [https://s0docs0docker0com.icopy.site/engine/reference/commandline/network_create/](https://s0docs0docker0com.icopy.site/engine/reference/commandline/network_create/)
* [https://www.cnkirito.moe/docker-network-bridge/](https://www.cnkirito.moe/docker-network-bridge/)
* [https://gist.github.com/ceduliocezar/b3bf93125024482b5f2f479696842046](https://gist.github.com/ceduliocezar/b3bf93125024482b5f2f479696842046)
* [https://github.com/SonarSource/docker-sonarqube/issues/282](https://github.com/SonarSource/docker-sonarqube/issues/282)
* [https://docs.sonarqube.org/latest/setup/get-started-2-minutes/](https://docs.sonarqube.org/latest/setup/get-started-2-minutes/)
