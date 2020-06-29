---
title: "run格式文件破解"
date:   2020-05-28  14:10:00 +0800
layout: post
tag:
- Rever Engineering
categories:
- Linux
- Security
---

run格式文件破解

-------
## run文件介绍

在RUN格式的文件是一个可执行的基于Linux的应用程序，包含了为Linux平台开发的具体方案相关的代码。此代码通常初始化应用程序本身，虽然在某些情况下，在文件.RUN扩展指程序安装用于Linux操作系统开发的应用程序。设备驱动软件为Linux常常被捆绑并分发.RUN包中包含的文件。这些.RUN文件初始化这些设备驱动程序软件应用程序的安装，所以Linux用户可以快速，轻松地安装或更新自己的设备驱动程序软件。

## 如何运行run文件
一个.RUN失败初始化文件是最有可能缺乏来自Linux操作系统相应的可执行权限。使用`chmod + X filenamehere.run` 这可以通过通过这个chmod命令添加所需的权限予以纠正。用户可事后初始化.RUN文件使用此命令：`./filenamehere.run`。

## 如果构建run文件

run程序安装包实质上是一个安装脚本加要安装的程序，如下图所示：
![20200528-crack-run-program-01.png](/images/20200528-crack-run-program-01.png)

通常为了方便，只放一个程序或者是一个压缩包，理论上可以放多个，但是会很恶心的。

这样整个run安装包结构就一目了然了，实际上因为实际需要结构多少有点变动但这个无关紧要，只需要明白原理就行了。
制作run安装包以下举个实际的例子：
为了简单起见，要安装的程序就是helloworld程序，安装它的过程就是把它拷贝到/bin目录下。
```
$ ls
install.sh helloworld
$ cat install.sh
#!/bin/bash
cp helloworld /bin
```

现在有一个安装脚本了，名为install.sh，有一个要安装的程序helloworld.因为要安装的程序一般都是用.tar.bz2来做的。我们这儿也做一下：
```
$ tar zcvf helloworld.tar.gz helloworld
```
现在修改一个安装脚本install.sh，改为：
```
#!/bin/bash
tail -n +6 "$0" >/tmp/helloworld.tar.gz # $0表示脚本本身，这个命令用来把从$lines开始的内容写入一个/tmp目录的helloworld.tar.gz文件里。
tar jxvf /tmp/hellowrold.tar.gz
cp helloworld /bin
exit 0
```
然后使用cat命令连接安装脚本install.sh和helloworld.tar.gz。
```
$ cat install.sh helloworld.tar.gz > myinstall.run
```
这样就得到了myinstall.run文件，它的结构如下：

![20200528-crack-run-program-05.png](/images/20200528-crack-run-program-05.png)

我们在notepad++中打开我们生成的安装文件
![20200528-crack-run-program-02.png](/images/20200528-crack-run-program-02.png)

运行myinstall.run时，运行到第5行的exit 0脚本就退出了，所以不会去运行第6行以下的二进制数据(即helloworld.tar.gz文件)，而我们用了tail巧妙地把第7行以下的数据重新生成了一个helloworld.tar.gz文件。再执行安装。run安装包制作较小的程序包是很好的选择，但是它也有缺点，做逻辑比较复杂的安装包，写的安装脚本将会很麻烦。

## 解压run文件
怎么样构建的run文件，怎么样解压run文件，但是加压之前你得需要明确文件的结构，也就是文件拆分的行数。我们这里以之前构建的myinstall.run文件为例来讲解。

### 确定文件结构
使用notepadd++，winhex等文本编辑工具打开run文件，获取安装shell脚本，小文件可以直接用notepad++查看，大文件建议先用winhex,010editor等工具打开，将shell文件拷贝出来再进行分析

![20200528-crack-run-program-03.png](/images/20200528-crack-run-program-03.png)

通过分析myinstall.run文件中的shell脚本，重点关注第二行，通过这行我们能确认具体安装程序从第6行开始。

### 解压文件
通过tail工具，我们可以直接获取我们helloworld.tar.gz，进而查看程序内容。

![20200528-crack-run-program-04.png](/images/20200528-crack-run-program-04.png)

命令可以参考以下，根据实际情况修改。
```
tail -n +6 myinstall.run>/tmp/helloworld.tar.gz
```
### 注意
文件解压不一定通过tail来获取，理论上所有可以操作文件的指令都可以，比如：sed，对于myinstall.run也可以使用以下指令来解压run文件
```
sed -n -e '1,/^exit 0$/!p' ./myinstall.run > ./helloworld.tar.gz 2>/dev/null
```

## 参考
- [https://blog.51cto.com/linuxcgi/1965299](https://blog.51cto.com/linuxcgi/1965299)【run文件介绍】
