---
title: "spotify dockerfile-maven-plugin ‘Could not acquire image ID or digest following build’错误解决"
date:   2022-02-12  17:51:00 +0800
layout: post
tag:
- Java
categories:
- Code
---

spotify dockerfile-maven-plugin ‘Could not acquire image ID or digest following build’错误解决

------
## 背景
使用spotify dockerfile-maven-plugin 在使用maven编译项目的过程中自动构建docker镜像文件，在构建过程中遇到“Could not acquire image ID or digest following build”错误，无法定位错误原因，全网检索以后，发现部分网友也遇到过，网友提出的错误原因有：
* Dockerfile配置错误，解决方法是检查dockerfile文件，或者单独调用docker构建命令进行测试
* .dockerignore配置错误，如果项目中需要配置.dockerignore,请认真检查，dockerfile-maven-plugin的[issues](https://github.com/spotify/dockerfile-maven/issues/25)中有提到
* dockerfile-maven-plugin版本较低，从1.3.×升级到1.4.×即可

博主开发环境：
jdk:1.8.0_291
dockerfile-maven-plugin:1.4.12
docker：Docker version 20.10.12, build e91ed5707e

博主的项目名为：engine,Dockerfile位于：engine/src/main/docker，构建的命令为：```mvn clean package dockerfile:build -Dmaven.test.skip=true ```
pom.xml docker-maven-plugin配置如下：
```
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>dockerfile-maven-plugin</artifactId>
    <version>1.4.12</version>
    <executions>
        <execution>
            <id>default</id>
            <phase>package</phase>
            <goals>
                <goal>build</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <repository>blingsec/${project.artifactId}</repository>
        <tag>${project.version}</tag>
        <dockerfile>src/main/docker</dockerfile>
        <buildArgs>
            <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
            <SCA_CLIENT>client/sca-client</SCA_CLIENT>
        </buildArgs>
    </configuration>
</plugin>
```

错误如下：
```
[ERROR] Failed to execute goal com.spotify:dockerfile-maven-plugin:1.4.12:build (default) on project engine: Execution default of goal com.spotify:dockerfile-maven-plugin:1.4.12:build failed: Could not acquire image ID or digest following build -> [Help 1]
```
![maven build error](/img/20220212-01.png)

mvn调试输出内容：
![maven build debug error](/img/20220212-02.png)


## 解决方案
解决方案仅限于我这种情况，dockerfile-maven-plugin配置如上所示。我进过多次尝试并翻了翻代码，发现dockerfile-maven-plugin  dockerfile参数如果没有指定默认是当前项目目录，我的Dockerfile文件位于engine/src/main/docker目录下，由于pom.xml配置是参考了docker-maven-plugin，dockerfile参数内容发生了变化,docker-maven-plugin中dockerfile参数是Dockerfile所在的目录，dockerfile-maven-plugin中dockerfile参数是Dockerfile文件的相对路径，对于我这种情况，一种就是把Dockerfile移动到项目根目录下，或者将dockerfile更改成src/main/docker/Dockerfile即可

执行以下命令进行构建：
```
mvn clean package dockerfile:build -Dmaven.test.skip=true 
```     
![maven build success](/img/20220212-03.png)


## 参考
- [https://github.com/spotify/dockerfile-maven](https://github.com/spotify/dockerfile-maven)【dockerfile-maven】
- [https://hjwjw.github.io/posts/9d44b524/](https://hjwjw.github.io/posts/9d44b524/)
- [https://spring.io/blog/2020/01/27/creating-docker-images-with-spring-boot-2-3-0-m1](https://spring.io/blog/2020/01/27/creating-docker-images-with-spring-boot-2-3-0-m1)
- [https://www.baeldung.com/spring-boot-docker-images](https://www.baeldung.com/spring-boot-docker-images)

