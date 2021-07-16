---
title: "Xjar加密破解"
date:   2021-07-16  10:13:00 +0800
layout: post
tag:
- Code Audit
categories:
- Security
---

Xjar加密破解

------
## 背景
研发人员通常在客户侧部署，为了防止更多源代码信息泄露，通常会将源码打包成二进制包，其中参杂一些混淆和加密的操作，安全人员在做测试，或者从一些泄露出来的二进制包进行漏洞挖掘时，总会遇到这些拦路虎，博主在逆向分析某系统jar包时就发现可以自动获取class文件，但是class内数据均被加密，打开全是乱码，但是spring boot启动相关数据是可以获取到的，第一次遇到，挺奇怪的，通过分析包名发现了io.xjar，后经过检索，发现是知名jar包加密工具xjar的特征。既然查到了，就顺便分析一下。

![20210716-01.png](/images/20210716-01.png)

## xjar介绍
xjar安全加密运行工具, 同时支持的原生JAR.基于对JAR包内资源的加密以及拓展ClassLoader来构建的一套程序加密启动, 动态解密运行的方案, 避免源码泄露以及反编译.
### 功能特性
- 无代码侵入, 只需要把编译好的JAR包通过工具加密即可.
- 完全内存解密, 降低源码以及字节码泄露或反编译的风险.
- 支持所有JDK内置加解密算法.
- 可选择需要加解密的字节码或其他资源文件.
- 支持Maven插件, 加密更加便捷.
- 动态生成Go启动器, 保护密码不泄露.

## 破解
### xjar加密分析
对于破解一些需要授权的系统，通常破解的思路是动态加载重写后的方法，或者直接替换 jar 或 class ，但是 xjar 使用一个用 go 写的加载器，并且原项目所有 class 均被加密了，并且做了 md5 和 sha1 的校验，无法通过替换来绕过。
后续分析 xjar ,发现使用对称加密算法加密 class 字节码，每次使用 xjar 加密源码，都会自动生成相关 go 的源码，包括 jar 包 hash 和密码，用户需要自行编译启动器。每次系统启动前，通过启动器，将秘钥通过标准输入流写到 jvm 中，然后通过 jvm 的 classloader 动态解密字节码，所以是纯 jvm 内存的解密。

### 破解思路
通过获取加解密秘钥，自行构建启动器(启动器在xjar项目源码中已存在，每次只会自动添加加密密钥，文件hash等数据，可以自行构建，移除hash检测，然后重新编译加载项目即可，这样就绕过了hash检测，我可以直接通过替换class，达到破解。但是当我做完这一步，发现我是真的笨，xjar提供了解密的方法，通过解密jar包，后续操作不就是水到渠成，何至于我还停留在修改启动器上。

### 破解步骤
通过分析启动器xjar.go，发现只检测参数中是否包含-jar参数，目的是获取jar的名称，然后通过获取到的参数，拼接命令行，启动java项目。

![20210716-02.png](/images/20210716-02.png)

可以写一个简单的程序将xjar启动器传给jvm的数据使用标准输出流打印出来。以下提供了php和go的两种写法，请自行参考或根据实际情况自行构建。
go版本:
```
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	input := bufio.NewScanner(os.Stdin) //初始化一个扫表对象
	for input.Scan() {                  //扫描输入内容
		line := input.Text() //把输入内容转换为字符串
		fmt.Println(line)    //输出到标准输出
	}
}

```
输出输入流命令行如下,我这里构建完成的程序名为xjarCrack，是在linux上构建运行的：
```
./xjar ./xjarCrack java -jar xjarTest-1.0-SNAPSHOT.xjar
```

![20210716-03.png](/images/20210716-03.png)

第一行是加密算法，第二行是密码长度，第三行是iv长度，最优一行是密码。

我这里也提供一个php版本，原理相同，如果碰到，根据自己熟练的语言自行构建即可:
```
<?php var_dump(file_get_contents('php://stdin'));
```
输出输入流命令行如下,我这里将php文件命名为file.php：
```
./xjar php file.php java -jar xjarTest-1.0-SNAPSHOT.xjar
```

通过获取到的密码，自行构建java项目，调用xjar解密方法，解密jar包即可。核心代码如下：

spring-boot jar包解密
```
// Spring-Boot Jar包解密
String password = "io.xjar";
XKey xKey = XKit.key(password);
XBoot.decrypt("/path/to/read/encrypted.jar", "/path/to/save/decrypted.jar", xKey);
```

jar包解密
```
// Jar包解密
String password = "io.xjar";
XKey xKey = XKit.key(password);
XJar.decrypt("/path/to/read/encrypted.jar", "/path/to/save/decrypted.jar", xKey);
```
使用idea自动加载jar包，可以查看配置文件，伪代码相关信息。

![20210716-04.png](/images/20210716-04.png)

## 总结
本次破解能够成功，主要归根于作者将xjar的源码进行开源，否则博主也没这么方便就破解掉了，通过分析xjar源码，我也发现了xjar在加密jar包，对jar包无损加密上做的确实厉害，虽然 xjar 在安全性上没办法起到更大的保障，但在很多地方是值得学习，致敬每一位开源大佬。

## 参考
- [https://github.com/core-lib/xjar](https://github.com/core-lib/xjar)【xjar github】
- [https://learnku.com/articles/56253](https://learnku.com/articles/56253)【记录一次破解xjar加密的经历】
- [https://www.jianshu.com/p/614e1d5358b2](https://www.jianshu.com/p/614e1d5358b2)【XJar: Spring-Boot JAR 包加/解密工具，避免源码泄露以及反编译】
