title: "linux下的split 命令（将一个大文件根据行数平均分成若干个小文件）"
date:   2020-02-22  21:26:00 +0800
layout: post
tag:
- Split
categories:
- Linux
---

在树莓派3B上安装安卓TV   
-------
# 背景
将一个大文件分成若干个小文件方法

例如将一个BLM.txt文件分成前缀为 BLM_ 的1000个小文件，后缀为系数形式，且后缀为4位数字形式

先利用

wc -l BLM.txt       读出 BLM.txt 文件一共有多少行

再利用 split 命令

split -l 2482 ../BLM/BLM.txt -d -a 4 BLM_

将 文件 BLM.txt 分成若干个小文件，每个文件2482行(-l 2482)，文件前缀为BLM_ ，系数不是字母而是数字（-d），后缀系数为四位数（-a 4）



linux下文件分割可以通过split命令来实现，可以指定按行数分割和安大小分割两种模式。Linux下文件合并可以通过cat命令来实现，非常简单。

　　在Linux下用split进行文件分割：

　　模式一：指定分割后文件行数

　　对与txt文本文件，可以通过指定分割后文件的行数来进行文件分割。

　　命令：split -l 300 large_file.txt new_file_prefix

　　模式二：指定分割后文件大小

   split -b 10m server.log waynelog

   对二进制文件我们同样也可以按文件大小来分隔。



在Linux下用cat进行文件合并：

　　命令：cat small_files* > large_file

将a.txt的内容输入到b.txt的末尾

cat a.txt >> b.txt

## 参考内容
* [https://blog.csdn.net/mxgsgtc/article/details/12048919](https://blog.csdn.net/mxgsgtc/article/details/12048919)
