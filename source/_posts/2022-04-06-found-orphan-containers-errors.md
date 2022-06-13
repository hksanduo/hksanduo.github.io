---
title: "Found orphan containers 错误"
date:   2022-04-06  16:33:00 +0800
layout: post
tag:
- Docker
categories:
- Develop
---

Found orphan containers 错误

------

## 背景
项目新需求，需要增加mongodbd数据库的支持，做一些数据的爬取工作，在docker目录下面新建一个mongodb的docker-compose配置文件：docker-compose-mongodb.yml，使用以下命令执行：
```
docker-compose -f docker-compose-mongodb.yml up 
```
系统提示：
```
WARN[0000] Found orphan containers ([sca-mysql docker-frontend-1]) for this project. If you removed or renamed this service in your compose file, you can run this command with the --remove-orphans flag to clean it up. 
[+] Running 1/0
 ⠿ Container sca-mongodb Created       0.1s
Attaching to docker-frontend-1, sca-mongodb, sca-mysql
no such service: docker-apiserver
```
也没有在意问题原因，但是感到奇怪，随后把无用的项目从系统中移除掉，只保留系统需要的容器镜像，在导入的过程中每次指定 docker-compose-mongodb.yml 会把 docker-compose-mysql.yml 的容器进行加载运行。

## 原因
Compose 在内部使用项目名称（默认为项目目录的基本名称，但可以明确指定）来隔离项目。项目名称用于为所有项目的容器和其他资源创建唯一标识符，Compose将其所在目录的名称作为默认项目名称。例如，如果您的项目名称是myapp并且它包含两个服务db和web，那么 Compose 会分别启动名为myapp_db_1和myapp_web_1两个容器。如果遇到"Found orphan containers"警告是因为Compose检测到一些属于另一个同名项目的容器。

## 解决
### docker-compose解决方法
为了防止不同的项目相互干扰，可以使用-p命令行选项或COMPOSE_PROJECT_NAME环境变量设置自定义项目名称。环境变量也可以通过环境文件设置（.env默认在当前工作目录中）。

增加运行参数：-p project_name / --project-name project_name
比如：
```
docker-compose -f docker-compose-mongodb.yml up -p project_name
```
在bash中执行,指定compose project name：
```
export COMPOSE_PROJECT_NAME
```
指定环境变量，只能对当前docker-compose*.yml起作用，如果两个docker-compose*.yml都执行相同的命令，他们的compose_project_name相同，仍然存在镜像冲突，或者相互影响的情况。

如果在同一个目录下，运行两个不同的docker-compose.yml，通过在其中一条命令下指定和当前目录名不同的project_name即可。

## 注意
在处理这种情况，谨慎使用```--remove-orphans```参数,该参数会停用并移除冲突的容器

## 参考
- [https://stackoverflow.com/questions/50947938/docker-compose-orphan-containers-warning](https://stackoverflow.com/questions/50947938/docker-compose-orphan-containers-warning)
- [https://docs.docker.com/compose/reference/](https://docs.docker.com/compose/reference/)