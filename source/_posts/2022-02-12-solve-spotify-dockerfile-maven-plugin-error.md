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


## 更新
###  repository does not exist or may require 'docker login': denied: requested access to the resource is denied
之前使用的镜像文件一直是dockerhub上的，最近需要使用自行构建的镜像，重新编辑dockerfile，使用mvn命令编译项目，打包镜像过程中遇到了以下问题：
```
[ERROR] pull access denied for hksanduo/oracle-jdk-8, repository does not exist or may require 'docker login': denied: requested access to the resource is denied
[WARNING] An attempt failed, will retry 1 more times
org.apache.maven.plugin.MojoExecutionException: Could not build image
    at com.spotify.plugin.dockerfile.BuildMojo.buildImage (BuildMojo.java:247)
    at com.spotify.plugin.dockerfile.BuildMojo.execute (BuildMojo.java:135)
    at com.spotify.plugin.dockerfile.AbstractDockerMojo.tryExecute (AbstractDockerMojo.java:265)
    at com.spotify.plugin.dockerfile.AbstractDockerMojo.execute (AbstractDockerMojo.java:254)
    at org.apache.maven.plugin.DefaultBuildPluginManager.executeMojo (DefaultBuildPluginManager.java:137)
    at org.apache.maven.lifecycle.internal.MojoExecutor.execute (MojoExecutor.java:210)
    at org.apache.maven.lifecycle.internal.MojoExecutor.execute (MojoExecutor.java:156)
    at org.apache.maven.lifecycle.internal.MojoExecutor.execute (MojoExecutor.java:148)
    at org.apache.maven.lifecycle.internal.LifecycleModuleBuilder.buildProject (LifecycleModuleBuilder.java:117)
    at org.apache.maven.lifecycle.internal.LifecycleModuleBuilder.buildProject (LifecycleModuleBuilder.java:81)
    at org.apache.maven.lifecycle.internal.builder.singlethreaded.SingleThreadedBuilder.build (SingleThreadedBuilder.java:56)
    at org.apache.maven.lifecycle.internal.LifecycleStarter.execute (LifecycleStarter.java:128)
    at org.apache.maven.DefaultMaven.doExecute (DefaultMaven.java:305)
    at org.apache.maven.DefaultMaven.doExecute (DefaultMaven.java:192)
    at org.apache.maven.DefaultMaven.execute (DefaultMaven.java:105)
    at org.apache.maven.cli.MavenCli.execute (MavenCli.java:972)
    at org.apache.maven.cli.MavenCli.doMain (MavenCli.java:293)
    at org.apache.maven.cli.MavenCli.main (MavenCli.java:196)
    at jdk.internal.reflect.NativeMethodAccessorImpl.invoke0 (Native Method)
    at jdk.internal.reflect.NativeMethodAccessorImpl.invoke (NativeMethodAccessorImpl.java:62)
    at jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke (DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke (Method.java:566)
    at org.codehaus.plexus.classworlds.launcher.Launcher.launchEnhanced (Launcher.java:282)
    at org.codehaus.plexus.classworlds.launcher.Launcher.launch (Launcher.java:225)
    at org.codehaus.plexus.classworlds.launcher.Launcher.mainWithExitCode (Launcher.java:406)
    at org.codehaus.plexus.classworlds.launcher.Launcher.main (Launcher.java:347)
Caused by: com.spotify.docker.client.exceptions.DockerException: pull access denied for hksanduo/oracle-jdk-8, repository does not exist or may require 'docker login': denied: requested access to the resource is denied
```
通过错误，可以看出，从公共镜像中找不到hksanduo/oracle-jdk-8,登录失败，也无法获取私有镜像，直接抛出异常，博主本人本地镜像源中是有hksanduo/oracle-jdk-8这个镜像的，这个镜像是博主自行打包的，但是尚未提交的dockerhub，相当于在当前情况下，使用spotify dockerfile maven plugin无法使用本地的镜像源，每次只获取最新的镜像源进行构建。   
通过翻阅资料和代码，找到一个配置属性，可以跳过，在pom.xml中增加配置信息，具体配置属性名称为：```pullNewerImage```，将值设置成 **false** 即可，配置参考：
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
                    <repository>hksanduo/${project.artifactId}</repository>
                    <tag>${project.version}</tag>
                    <dockerfile>src/main/docker/Dockerfile</dockerfile>
                    <buildArgs>
                        <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
                        <SCA_CLIENT>client/sca-client</SCA_CLIENT>
                    </buildArgs>
                    <pullNewerImage>false</pullNewerImage>
                </configuration>
            </plugin>
```

## 参考
- [https://github.com/spotify/dockerfile-maven](https://github.com/spotify/dockerfile-maven)【dockerfile-maven】
- [https://hjwjw.github.io/posts/9d44b524/](https://hjwjw.github.io/posts/9d44b524/)
- [https://spring.io/blog/2020/01/27/creating-docker-images-with-spring-boot-2-3-0-m1](https://spring.io/blog/2020/01/27/creating-docker-images-with-spring-boot-2-3-0-m1)
- [https://www.baeldung.com/spring-boot-docker-images](https://www.baeldung.com/spring-boot-docker-images)

