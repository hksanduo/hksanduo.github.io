---
title: "go windows程序使用管理员权限运行"
date:   2021-04-28  16:21:00 +0800
layout: post
tag:
- Go
categories:
- Coding
---

go windows程序使用管理员权限运行

-------

## 背景
写了一个windows日志收集的小工具，部分操作需要管理员权限，从网上找了几篇文章，思路都是通过Windows的Manifest.xml文件构建syso，然后打包运行即可，在此参考了几篇文章，CSDN居多，不止百度，谷歌上CSDN的文章也比较考前，但是部分文章思路没问题，但是你们的manifest.xml配置文件都没有闭合，导致博主在实验的时候走了不少弯路，对你们各种不负责转载的现象进行批评。以下是搜索引擎截图，部分manifest文件内容缺失，会导致程序抛出异常，对于新手，这很不友好。
![20210428-01](/images/20210428-01.png)
## windows manifest文件介绍
> 引用微软官方docs
> An application manifest is an XML file that describes and identifies the shared and private side-by-side assemblies that an application should bind to at run time. These should be the same assembly versions that were used to test the application. Application manifests may also describe metadata for files that are private to the application.

通俗来讲，Manifest事实上就是一个以.manifest为后缀的XML文件，用于组织和描述隔离应用程序及并行组件，其内部的信息如<assemblyIdentity>元素则标识着一个唯一的程序集，和其他信息一起，他们用于COM类、接口及库的绑定和激活，而这些信息，以往都是存储在注册表中的。另外，Manifests也制定了组成程序集的文件及Windows类。

## go windows程序使用管理员权限运行
### 1、获取rsrc
```
go get get github.com/akavel/rsrc 
```
在项目根目录中创建*.manifest文件（文件名没有过多局限，不过按照要求来也没错）
文件内容如下：
```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
    <trustInfo xmlns="urn:schemas-microsoft-com:asm.v3">
        <security>
            <requestedPrivileges>
                <requestedExecutionLevel level="requireAdministrator" />
            </requestedPrivileges>
        </security>
    </trustInfo>
</assembly>
```
网上部分转载摘抄的文章中有的少抄了最后一行，会导致程序抛出异常，`报错：应用程序无法启动，因为应用程序的并行配置不正确。有关详细信息，请参阅应用程序事件日志，或使用命令行 sxstrace.exe 工具。`
![20210428-02](/images/20210428-02.png)

### 2、编译 *.syso
编译命令如下:
```
rsrc -manifest .\test.manifest -o test.syso
```
文件名没啥要求

### 3、go编译打包
```
go build
```
然后测试运行一下，看是否提示申请管理员权限。

### 4、优化
如果需要做到windows全系列兼容，可以在`<assembly>`标签下面增加以下内容：
```
<!-- OSVersion -->
    <compatibility xmlns="urn:schemas-microsoft-com:compatibility.v1">   
        <application>   
            <!-- Windows 10 -->   
            <supportedos id="{8e0f7a12-bfb3-4fe8-b9a5-48fd50a15a9a}"></supportedos>  
            <!-- Windows 8.1/Windows Blue/Server 2012 R2 -->  
            <supportedos id="{1f676c76-80e1-4239-95bb-83d0f6d0da78}"></supportedos>  
            <!-- Windows Vista/Server 2008 -->  
            <supportedos id="{e2011457-1546-43c5-a5fe-008deee3d3f0}"></supportedos>   
            <!-- Windows 7/Server 2008 R2 -->  
            <supportedos id="{35138b9a-5d96-4fbd-8e2d-a2440225f93a}"></supportedos>  
            <!-- Windows 8/Server 2012 -->  
            <supportedos id="{4a2f28e3-53b9-4441-ba9c-d69d4a4a6e38}"></supportedos>  
        </application>   
    </compatibility>  
```
支持Windows 6.0界面库、支持管理员权限、兼容WIN8/WIN10下取系统版本、兼容DPI Aware，完整manifest可以参考[https://blog.csdn.net/cometnet/article/details/52995192](https://blog.csdn.net/cometnet/article/details/52995192)

## 参考
- [https://docs.microsoft.com/en-us/windows/win32/sbscs/application-manifests](https://docs.microsoft.com/en-us/windows/win32/sbscs/application-manifests)【Application Manifests】
- [https://blog.csdn.net/cometnet/article/details/52995192](https://blog.csdn.net/cometnet/article/details/52995192)【比较完整的Windows应用程序清单文件 manifest.xml】
- [https://github.com/akavel/rsrc](https://github.com/akavel/rsrc)