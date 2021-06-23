---
title: "Flawfinder开源C/C++静态扫描分析工具安装与使用"
date:   2019-11-15  12:00:00 +0800
layout: post
tag:
- Flawfinder
categories:
- Security
- Code Audit
---

Flawfinder开源C/C++静态扫描分析工具安装与使用
------
## flawfinder的介绍
Flawfinder是一款开源的关于C/C++静态扫描分析工具，其根据内部字典数据库进行静态搜索，匹配简单的缺陷与漏洞，flawfinder工具不需要编译C/C++代码，可以直接进行扫描分析。简单快速，最大的有点就是免费，不需要编译。flawfinder工具可以在官网进行下载。
[https://dwheeler.com/flawfinder/#downloading](https://dwheeler.com/flawfinder/#downloading)

## flawfinder的安装
### 在线安装
flawfinder安装比较简单，由于其是基于Python实现的一款工具，所以需要首先安装Python环境，并配置环境变量。flawfinder下载之后解压既可使用。flawfinder目前之前python2和python3，简单的方法是使用pip工具，执行以下指令进行安装。
```
pip install flawfinder
```
### 离线安装
直接从flawfinder下载程序压缩包，解压完成以后，使用python直接加载flawfiner程序即可，或者直接下载flawfinder-*.whl，使用pip工具离线安装，这种通常是在一些甲方审计项目中出现，甲方客户无法协调审计设备上的管理员权限，又无法连接外网，只能使用一些免安装的工具。对于这种情况，python直接使用免安装版本，使用离线的方式使用pip安装或者使用python直接运行flawfinder,给各位审计人员一个建议，原理这些不靠谱的甲方企业。


## flawfinder的使用
方式一：`flawfinder  --csv  > test-result.csv   test.c`
这种方式根据缺陷库生成一个 .csv文件  ，你只需要根据这个.csv文件就可以转换为正常Excel文件使用，转换方法自行百度。
方式二：`flawfinder  --html > test-result.html test.c`

## 案例讲解
由于在日常审计过程中，项目中有其他格式的文件，通常使用linux`find`工具批量筛选.c或者.cpp文件，然后使用flawfinder进行扫描
```
cp --parents `find 程序目录/-name *.c`  指定扫描目录
```
  * 增加--parents目录主要作用是在拷贝的时候，会在目标路径中创建源文件参数中的所有父目录层级(不止是一层父目录)，然后将源文件拷贝进去。这样做的目的主要是清晰展示目录结构，方便写报告。

```
flawfinder --csv > result.csv 指定扫描目录
```
导出csv文件内容展示如下
![flawfinder-csv.png](/images/flawfinder-csv.png)

## flawfinder分析
Flawfinder 不是类似于fortify那样复杂的工具,它是一个简单并有意义工具。Flawfinder通过使用内置的C/C++函数数据库来工作，该数据库具有众所周知的安全风险，例如缓冲区溢出风险（例如strcpy()，strcat()，gets()，sprintf()和scanf()），格式字符串问题（printf()，snprintf()和syslog()），竞争条件（例如access()，chown()，chgrp()，chmod()，tmpfile()，tmpnam()，tempnam()和mktemp()），潜在的远程命令执行风险（大多数exec()系列，system()，popen()）和较差的随机数获取方法（例如random()）。
Flawfinder的好处是不必创建相关数据库，自身就拥有相关数据库。Flawfinder获取源代码，并将源代码文本与这些名称匹配，同时忽略注释和字符串中的文本。Flawfinder还支持gettext（国际化程序的公共库），并且会将通过gettext传递的常量字符串当作常量字符串对待。这减少了国际化程序中的错误命中次数。
Flawfinder生成按风险分类的（潜在安全漏洞）列表；默认情况下，最危险的匹配项将首先显示。风险级别不仅取决于功能，还取决于功能的参数值。例如：在许多情况下，常量字符串通常比完全可变字符串的风险要小。在某些情况下，代码审计人员可能能够确定该结构体完全没有风险，从而减少了误报。与仅在源代码上运行“ grep”相比，Flawfinder提供了更好的信息和更好的优先级。flawfinder可以忽略注释和字符串内部，并且还将检查参数以估计风险水平。但是，从根本上来说，flawfinder仅仅是一个简单的python程序。它甚至不知道函数参数的数据类型，并且当然也不进行控制流或数据流分析。由于Flawfinder很简单，因此不会被宏定义和更复杂的工具遇到的其他奇怪问题所混淆。Flawfinder可以分析无法构建的软件；在某些情况下，它可以分析甚至无法在本地编译的文件。但是需要主要一点儿，并非发现的每个问题都是一个安全漏洞，也不一定能找到所有安全漏洞。如上所述，flawfinder不能真正理解代码的语义，它主要完成简单的文本模式匹配（忽略注释和字符串），不执行数据流或控制流分析，尽管如此，flawfinder在实际代码审计项目中也可以协助安全人员发现和消除安全漏洞。

## 错误修复
### UnicodeDecodeError: 'utf-8' codec can't decode byte 0xff in position 0: invalid start byte
在运行过程中，会出现解码出错，官方给出的建议是通过强制转换扫描文档的格式为utf-8，我们可以直接忽略
`UnicodeDecodeError: 'utf-8' codec can't decode byte 0xff in position 0: invalid start byte`
![flawfinder-error.png](/images/flawfinder-error.png)

#### 官方修复建议
![flawfinder-office-advice.png](/images/flawfinder-office-advice.png)
将操作系统的编码格式设置成`utf-8`，将程序编码格式强制转换为utf-8，官方推荐的工具为`
cvt2utf`，可以根据实际情况自行修改。

#### 个人修复建议
![flawfinder-persional-advice1.png](/images/flawfinder-persional-advice1.png)
个人这个就有点儿暴力，直接在打开文件的那一步设定，如果出现错误直接忽略。flawfinder如果使用pip安装，安装的位置位于`/usr/local/bin/flawfinder`，其他安装方式，请根据实际情况进行查找。
![flawfinder-persional-advice2.png](/images/flawfinder-persional-advice2.png)

### UnicodeDecodeError: 'ascii' codec can't decode byte 0xe6 in position 29: ordinal not in range(128)
#### 错误原因
提示中的“ordinal not in range(128)”，意思是，字符不在128范围内，即说明不是普通的ASCII字符，超出处理能力了。

#### 解决方法
找到flawfinder程序文件，用文本编辑器打开，在文件抬头加入以下代码。
```
import sys
reload(sys)
sys.setdefaultencoding("utf-8")
```

## 参考
* [https://dwheeler.com/flawfinder/](https://dwheeler.com/flawfinder/)【flawfinder官网】
* [https://github.com/david-a-wheeler/flawfinder](https://github.com/david-a-wheeler/flawfinder)【github】
