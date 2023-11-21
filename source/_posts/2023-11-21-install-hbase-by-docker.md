---
title: "在docker中安装hbase-3(本地模式)"
date:   2023-11-21  17:00:00 +0800
layout: post
tag:
- BigData
categories:
- Linux
---

# 在docker中安装hbase-3(本地模式)

------

## 背景
最近在处理数据，传统的模式已经无法支撑千万级的任务分析和存储，被迫得升级一下当前的技术栈，为了方便和易于部署(图省事)，构造hbase-3(本地模式)容器来继研究学习。

## 构造hbase容器
参考harisekhon/hbase的脚本以及配置文件，结合hbase最新版本特征，重新修改的配置文件和脚本目录树如下：
![hbase容器目录](/img/20231121-01.png)
### 配置
#### Dockerfile
其中，Dockerfile如下：
```
FROM harisekhon/alpine-java:jre8

ARG HBASE_VERSION=3.0.0-alpha-4

ENV PATH $PATH:/hbase/bin

ENV JAVA_HOME=/usr

WORKDIR / 

# bash - needed for entrypoint.sh
RUN apk add --no-cache bash

RUN set -eux && \
    apk add --no-cache wget tar && \
    url="http://www.apache.org/dyn/closer.lua?filename=hbase/$HBASE_VERSION/hbase-$HBASE_VERSION-bin.tar.gz&action=download"; \
    url_archive="http://archive.apache.org/dist/hbase/$HBASE_VERSION/hbase-$HBASE_VERSION-bin.tar.gz"; \
    wget -t 10 --max-redirect 1 --retry-connrefused -O "hbase-$HBASE_VERSION-bin.tar.gz" "$url" || \
    wget -t 10 --max-redirect 1 --retry-connrefused -O "hbase-$HBASE_VERSION-bin.tar.gz" "$url_archive" && \
    mkdir "hbase-$HBASE_VERSION" && \
    tar zxf "hbase-$HBASE_VERSION-bin.tar.gz" -C "hbase-$HBASE_VERSION" --strip 1 && \
    test -d "hbase-$HBASE_VERSION" && \
    ln -sv "hbase-$HBASE_VERSION" hbase && \
    rm -fv "hbase-$HBASE_VERSION-bin.tar.gz" && \
    rm -rf hbase/docs hbase/src && \
    mkdir /hbase-data && \
    apk del tar wget

# Needed for HBase 2.0+ hbase-shell
# asciidoctor solves 'NotImplementedError: fstat unimplemented unsupported or native support failed to load'
RUN bash -c ' \
    set -euxo pipefail && \
    apk add --no-cache jruby jruby-irb asciidoctor && \
    echo exit | hbase shell \
    # jruby-maven jruby-minitest jruby-rdoc jruby-rake jruby-testunit && \
    '

VOLUME /hbase-data

COPY entrypoint.sh /
COPY conf/hbase-site.xml /hbase/conf/

# Stargate  8080  / 8085
# Thrift    9090  / 9095
# HMaster   16000 / 16010
# RS        16201 / 16301
EXPOSE 2181 8080 8085 9090 9095 16000 16010 16030 16201 16301

ENTRYPOINT ["/entrypoint.sh"]
```
Dockerfile说明：
如果觉得harisekhon/alpine-java:jre8比较陈旧，可以自行编译构建，参考：alpine-java

#### hbase-site.xml
conf/hbase-site.xml 配置如下:
```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <name>hbase.rootdir</name>
    <value>/hbase-data</value>
  </property>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>zookeeper:2181</value>
  </property>
  <property>
    <name>hbase.tmp.dir</name>
    <value>${env.HBASE_HOME:-.}/tmp</value>
  </property>
  <property>
    <name>hbase.unsafe.stream.capability.enforce</name>
    <value>false</value>
  </property>
</configuration>

```
hbase-site.xml 配置说明：
这个配置文件主要是针对本地模式进行修改的，并不适配集群。

#### docker-compose.yaml
docker-compose.yaml 如下：
```
version: '3'

services:
    hbase:
        image: hksanduo/hbase:3.0.0-alpha-4
        container_name: hbase
        hostname: master
        restart: always
        logging:
          driver: "json-file"
          options:
            max-size: "500m"
        ports:
          - 2181:2181
          - 8080:8080
          - 8085:8085
          - 9090:9090
          - 9095:9095
          - 16000:16000
          - 16010:16010
          - 16020:16020
          - 16030:16030
        volumes:
          - ./hbase-data:/hbase-data
        environment:
          - TZ=Asia/Shanghai
        privileged: true

```
这里将当前目录的hbase-data目录进行映射，做到数据持久化。

#### entrypoint.sh 
entrypoint.sh 如下：
```
#!/usr/bin/env bash

set -x
set -euo pipefail
[ -n "${DEBUG:-}" ] && set -x


export HBASE_HOME="/hbase"
export JAVA_HOME="${JAVA_HOME:-/usr}"

echo "================================================================================"
echo "                              HBase Docker Container"
echo "================================================================================"
echo
# shell breaks and doesn't run zookeeper without this
mkdir -pv "$HBASE_HOME/logs"

start_zookeeper(){
    # tries to run zookeepers.sh distributed via SSH, run zookeeper manually instead now
    #RUN sed -i 's/# export HBASE_MANAGES_ZK=true/export HBASE_MANAGES_ZK=true/' "$HBASE_HOME/conf/hbase-env.sh"
    echo
    echo "Starting Zookeeper..."
    "$HBASE_HOME/bin/hbase" zookeeper &>"$HBASE_HOME/logs/zookeeper.log" &
    echo
}

start_master(){
    echo "Starting HBase Master..."
    "$HBASE_HOME/bin/hbase-daemon.sh" start master
    echo
}

start_regionserver(){
    echo "Starting HBase RegionServer..."
    # HBase versions < 1.0 fail to start RegionServer without SSH being installed
    if [ "$(echo /hbase-* | sed 's,/hbase-,,' | cut -c 1)" = 0 ]; then
        "$HBASE_HOME/bin/local-regionservers.sh" start 1
    else
        "$HBASE_HOME/bin/hbase-daemon.sh" start regionserver
    fi
    echo
}

start_stargate(){
    # kill any pre-existing rest instances before starting new ones
    pgrep -f proc_rest && pkill -9 -f proc_rest
    echo "Starting HBase Stargate Rest API server..."
    "$HBASE_HOME/bin/hbase-daemon.sh" start rest
    echo
}

start_thrift(){
    # kill any pre-existing thrift instances before starting new ones
    pgrep -f proc_thrift && pkill -9 -f proc_thrift
    echo "Starting HBase Thrift API server..."
    #"$HBASE_HOME/bin/hbase-daemon.sh" start thrift
    "$HBASE_HOME/bin/hbase-daemon.sh" start thrift2
    echo
}

start_hbase_shell(){
    echo "
Example Usage:

create 'table1', 'columnfamily1'

put 'table1', 'row1', 'columnfamily1:column1', 'value1'

get 'table1', 'row1'


Now starting HBase Shell...
"
    "$HBASE_HOME/bin/hbase" shell
}

trap_func(){
    echo -e '\n\nShutting down HBase:'
    "$HBASE_HOME/bin/hbase-daemon.sh" stop rest || :
    "$HBASE_HOME/bin/hbase-daemon.sh" stop thrift || :
    "$HBASE_HOME/bin/local-regionservers.sh" stop 1 || :
    # let's not confuse users with superficial errors in the Apache HBase scripts
    "$HBASE_HOME/bin/stop-hbase.sh" |
        grep -v -e "ssh: command not found" \
                -e "kill: you need to specify whom to kill" \
                -e "kill: can't kill pid .*: No such process"
    sleep 2
    pgrep -fla org.apache.hadoop.hbase |
        grep -vi org.apache.hadoop.hbase.zookeeper |
            awk '{print $1}' |
                xargs kill 2>/dev/null || :
    sleep 3
    pkill -f org.apache.hadoop.hbase.zookeeper 2>/dev/null || :
    sleep 2
}
trap trap_func INT QUIT TRAP ABRT TERM EXIT

if [ -n "$*" ]; then
    if [ "$1" = master ] || [ "$1" = m ]; then
        start_master
        tail -f /dev/null "$HBASE_HOME/logs/"* &
    elif [ "$1" = regionserver ] || [ "$1" = rs ]; then
        start_regionserver
        tail -f /dev/null "$HBASE_HOME/logs/"* &
    elif [ "$1" = rest ] || [ "$1" = stargate ]; then
        start_stargate
        tail -f /dev/null "$HBASE_HOME/logs/"* &
    elif [ "$1" = thrift ]; then
        start_thrift
        tail -f /dev/null "$HBASE_HOME/logs/"* &
    elif [ "$1" = shell ]; then
        "$HBASE_HOME/bin/hbase" shell
    elif [ "$1" = bash ]; then
        bash
    else
        echo "usage:  must specify one of: master, regionserver, thrift, rest, shell, bash"
    fi
else
    sed -i 's/zookeeper:2181/localhost:2181/' "$HBASE_HOME/conf/hbase-site.xml"
    start_zookeeper
    start_master
    start_regionserver
    start_stargate
    start_thrift
    if [ -t 0 ]; then
        start_hbase_shell
    else
        echo "
    Running non-interactively, will not open HBase shell

    For HBase shell start this image with 'docker run -t -i' switches
    "
        tail -f /dev/null "$HBASE_HOME/logs/"* &
        # this shuts down from Control-C but exits prematurely, even when +euo pipefail and doesn't shut down HBase
        # so I rely on the sig trap handler above
    fi
fi
wait || :
```
说明：entrypoint.sh 中默认启用的是thrift2，如果需要thrift，请自行修改

### 构建运行镜像
将以上内容，按照上述目录结构进行保存，需要注意，提前赋予entrypoint.sh执行权限，然后进行构建，构建命令如下：

``` docker build -t hksanduo/hbase:3.0.0-alpha-4 . ```

构建成功如下：
![构建成功](/img/20231121-02.png)

如果映射端口有所调整，请自行修改docker-compose.yaml，如果没有，直接执行``` docker-compose up -d ```

稍等片刻，访问：http://{your server ip}:10610/ 如果能够正常加载HBase Master web 界面证明运行成功。  
![ master status ](/img/20231121-03.png)


## 参考
- [https://github.com/harisekhon/Dockerfiles](https://github.com/harisekhon/Dockerfiles)【harisekhon hbase github】
- [https://hub.docker.com/r/harisekhon/hbase](https://hub.docker.com/r/harisekhon/hbase)【harisekhon hbase】
- [https://github.com/apache/hbase](https://github.com/apache/hbase)【hbase】
- [https://github.com/harisekhon/Dockerfiles/tree/master/alpine-java](https://github.com/harisekhon/Dockerfiles/tree/master/alpine-java)【alpine java】


