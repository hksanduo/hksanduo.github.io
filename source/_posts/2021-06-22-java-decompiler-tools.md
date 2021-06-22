---
title: "常见java反编译工具"
date:   2021-06-22  17:58:00 +0800
layout: post
tag:
- Java
categories:
- Security
---

常见java反编译工具

------
在源代码审计或者进行漏洞挖掘，有时会碰到war包，jar包，或者直接打包的class文件，需要通过反编译进行源代码审计。以下是个人常用的几种java反编译工具，个人比较推荐使用IDEA自带的反编译组件，直接调用IDEA的java-decompiler组件进行反编译，也方便进行调试分析。部分工具不仅限于反编译，其他功能请自行摸索。萝卜青菜，各有所爱，各位依据自身情况选择工具。

## jd-gui
JD-GUI 是一个用 C++ 开发的 Java 反编译工具，由 Pavel Kouznetsov开发，支持Windows、Linux和苹果Mac Os三个平台。而且提供了Eclipse平台下的插件JD-Eclipse、IntelliJ的插件JD-IntelliJ。JD-GUI不需要安装，直接点击运行，可以反编译jar,class文件。
![20210622-01.png](/images/20210622-01.png)
点击Save All Sources 来保存反编译以后的结果，方便查询和阅读。
![20210622-02.png](/images/20210622-02.png)

官方地址：http://java-decompiler.github.io/

## jadx
 jadx是一款Android反编译gui工具，它支持apk、dex、jar、class、zip、aar等文件。jadx操作方便，反编译后的代码可读性高，同时还拥有较完善的gui界面，除去混淆部分的代码，jadx已经非常接近源代码了。
![20210622-03.png](/images/20210622-03.png)

官方地址：https://github.com/skylot/jadx/

## fernflower
对于未做混淆的war包和jar包，使用fernflower是一个比较好的反编译软件，比jd-gui好用，把所用工程从class反编译成java可以很好的进行字符串搜索和匹配，非常适合纯代码审计（当无法进行动态调试的时候是一个不错的选择）

命令格式为：
```
java -jar java-decompiler.jar [-<option>=<value>]* [<source>]+ <destination>
```
source 表示jar包所在目录，可以填写单个jar包，也可以填写一个目录(将解压目录下所有jar包)destination 表示反编译的java源码生成目录

官方地址：https://github.com/fesh0r/fernflower

## idea decompiler
fernflower是 IDEA 采用的反编译工具，在IDEA打开class文件时，就是通过该组件的反编译能力。java-decompiler 是IDEA中的插件名称，实际上来源于 fernflower 工具。

### 1、直接使用IDEA打开我们需要分析的war包工程，IDEA会自动帮助我们将class进行反编译并展示出来，节约了我们好多工作。

![20210622-04.png](/images/20210622-04.png)

对于jar包，将jar所在的目录设置成library

![20210622-05.png](/images/20210622-05.png)

jar包可以直接打开，方便进行分析。

![20210622-06.png](/images/20210622-06.png)

也可以在项目结构(Project Structure)中配置Libraray

![20210622-07.png](/images/20210622-07.png)


### 2、直接调用IDEA java-decompiler.jar进行反编译
IDEA中java-decompiler.jar默认是安装的，位于IDEA安装目录\plugins\java-decompiler\lib\，博主本人java-decompiler.jar目录为：D:\Program\IntelliJ IDEA 2018.1.6\plugins\java-decompiler\lib ，主要是接下来会用到。

![20210622-08.png](/images/20210622-08.png)

创建一个目录，用来存储反编译后的文件，如test文件夹。我这里拿FrontDemoController.class来举例：
```
java -cp "D:\Program\IntelliJ IDEA 2018.1.6\plugins\java-decompiler\lib\java-decompiler.jar" org.jetbrains.java.decompiler.main.decompiler.ConsoleDecompiler -dgs=true .\FrontDemoController.class .\test\
```
![20210622-09.png](/images/20210622-09.png)

这样可以将FrontDemoController.class直接反编译成FrontDemoController.java。对于jar包，反编译结果也打包成jar包，这点儿需要注意，你直接使用压缩工具解压即可。这里拿dom4j这个jar来举例，以下截图是反编译以后使用压缩工具解压jar的结果，可以明显看出jar包中的class文件均被反编译成java。

![20210622-10.png](/images/20210622-10.png)

## JAD
JAD 是一款老牌的、经典的、使用起来简单的 Java 反编译工具,可以通过命令行把Java的class文件反编译成源代码。
下载地址：http://www.varaneckas.com/jad

![20210622-11.png](/images/20210622-11.png)

使用方法：
1、反编译一个class文件：jad.exe -sjava example.class，会生成example.java，用文本编辑器打开就是java源代码
2、把源代码文件输出到指定的目录：jad -dnewdir -sjava example.class，在newdir目录下生成example.java
3、把packages目录下的class文件全部反编译：jad -sjava packages/.class
4、把packages目录以及子目录下的文件全部反编译：jad -sjava packages/**/.class，不过你仍然会发现所有的源代码文件被放到了同一个文件中，没有按照class文件的包路径建立起路径
5、把packages目录以及子目录下的文件全部反编译并建立和java包一致的文件夹路径，可以使用-r命令：jad -r -sjava packages/*/.class常用的也就是第5条，各位根据自己需求进行操作，一些特殊的需求详见参考中的jad命令总结链接。

![20210622-12.png](/images/20210622-12.png)

注意：jad不支持中文，如果反编译结果中出现部分unicode乱码，请自行解码分析。jad不能直接反编译jar包，直接解压jar包后再反编译。

## bytecode-viewer
bytecodeviewer是一款基于图形界面的Java反编译器，Java字节码编辑器，APK编辑器，Dex编辑器，APK反编译器，DEX反编译器，其集成了6个Java反编译库（包含Fernflower和CFR），Andorid反编译类库和字节码类库。不仅如此，它还是一款Hex查看器，代码搜索器和代码调试器。除此之外，它还具备Smali和Baksmali等汇编器的相关功能(这段是百度上抄的)。

其实有很多大佬改jar包的时候都会使用smali和baksmali ，然后在dex2jar。这确实可以适应大部分情况，但有些情况dex2jar后的jar包出问题了，这时候你可能会用到这款工具。

简单分享一下它对jar包的逆向，包括分析，以及直接修改java字节码，通过平时的使用，发现用这款工具对各种各样的SDK修改，简直方便的不行。

注意：建议直接编译源码，因为直接使用可能会有莫名其妙的问题。
![20210622-13.png](/images/20210622-13.png)

官方地址：https://github.com/Konloch/bytecode-viewer


## 参考：
- [https://bbs.pediy.com/thread-254141.htm](https://bbs.pediy.com/thread-254141.htm)【bytecodeviewer使用】
- [https://hksanduo.github.io/2021/04/08/2021-04-08-vulnerability-mining-within-pentest/](https://hksanduo.github.io/2021/04/08/2021-04-08-vulnerability-mining-within-pentest/)【在渗透测试过程中通过源代码审计进行漏洞挖掘 】
- [https://blog.csdn.net/yannqi/article/details/80847354](https://blog.csdn.net/yannqi/article/details/80847354)【2020年支持java8的Java反编译工具汇总】
- [https://www.zhihu.com/question/20264247](https://www.zhihu.com/question/20264247)【最好的java反编译工具是哪个？】

