title: "学习kernel patch"
date:   2020-02-13  22:17:00 +0800
layout: post
tag:
- Openwrt
- Kernel
categories:
- IOT
- Linux
---

学习kernel patch    
-------
# 背景
编译openwrt固件,对于小众板子,需要自行构建patch,在构建过程中遇到不少坑,在这里记录一下,主要是自己
太菜,好多知识不明白

## patch文件格式解说
patch    
> patch命令跟diff配合使用，把生成的补丁应用到现有代码上。常用命令行选项：
>
>　　patch [命令行选项] [待patch的文件[patch]]
>
>　　-pn patch level(n是数字) -b[后缀] 生成备份，缺省是.orig
>
> 为了说明什么是patch level，这里看一个patch文件的头标记。

　　diff -ruNa xc.orig/config/cf/Imake.cf xc.bsd/config/cf/Imake.cf

　　--- xc.orig/config/cf/Imake.cf Fri Jul 30 12:45:47 1999

　　+++ xc.new/config/cf/Imake.cf Fri Jan 21 13:48:44 2000

　　这个patch如果直接应用，它会去找xc.orig/config/cf目录下的Imake.cf文件，假如你的源码树的根目录是缺省的xc而不是xc.orig，除了mv xc xc.orig之外，有无简单的方法应用此patch呢？patch level就是为此而设：patch会把目标路径名砍去开头patch level个节(由/分开的部分)。在本例中，可以用下述命令：cd xc; patch _p1 < /pathname/xxx.patch 完成操作。注意，由于没有指定patch文件，patch程序默认从stdin读入，所以用了输入重定向。

　　如果patch成功，缺省是不建备份文件的(注：FreeBSD下的patch工具缺省是保存备份)，如果你需要，可以加上 b 开关。这样把修改前的文件以“原文件名.orig”的名字做备份。如果你喜欢其它后缀名，也可以用“b 后缀”来指定。

　　如果patch失败，patch会把成功的patch行给patch上，同时（无条件）生成备份文件和一个.rej文件。.rej文件里是没有成功提交的patch行，需要手工patch上去。这种情况在原码升级的时候有可能会发生。

　　关于二进制文件的说明：binary文件可以原始方式存入patch文件。diff可以生成(加-a选项),patch也可以识别。如果觉得这样的patch文件太难看，解决方法之一是用uuencode处理该binary文件。

## 错误
### 1.arch/arm/boot/dts/kirkwood-butong.dtb: ERROR (phandle_references): Reference to non-existent node or label "pmx_sdio_cd"
主要是在配置过程中未检测变量是否定义,胡乱粘贴就编译

一个最简单的patch的格式

--- 旧文件
+++ 新文件
@@  —旧行号开始，旧行号结束 +新行号开始，新行号结束 @@                      
 不改动的文件内容
 不改动的文件内容
 不改动的文件内容
+增加的行
-删除的行
 不改动的文件内容
 不改动的文件内容
 不改动的文件内容 

## 参考内容
* [https://www.cnblogs.com/the-tops/p/6068669.html](https://www.cnblogs.com/the-tops/p/6068669.html)
