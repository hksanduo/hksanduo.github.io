---
title: "shell编程技巧之判断用户权限"
date:   2018-05-14 12:00 +0800
layout: post
tag: 
- shell
categories:
- Security
- Linux
---

# shell编程技巧之判断用户权限
------

## UID,GID,EUID,EGID,SUID,SGID

| 名称 | 介绍 |
| :------: | :------: |
| UID/GID | 实际用户ID和实际用户组ID,登录时用户对应的ID |
| EUDI/EGID| 有效的用户ID和有效的组ID,主要指定了访问目标的权限 |
| SUID/SGID| 针对文件而讲述的概念,他可以修改当前进程的EUDI/EGID |

## linux权限s/t
这里s/t是针对执行权限来说的.
s权限,是为了让使用者临时具有该文件的所属用户或组的执行权限,0755最前面的0表示不使用任何特殊
权限，该位上的数字可以是```0,1(--t),2(-s-),3(-st),4(s--),5(s-t),6(ss-),7(sst)```,那个t权限只针对
目录生效，它表示只能让所属主以及root可以删除（重命名/移动）该目录下的文件。比如/tmp目录本
来就是任何用户都可以读写，如果别人可以任意删除（重命名/移动）自己的文件，那岂不是很危险。
所以这个t权限就是为了解决这个麻烦的。

## 判断当前有效用户的权限是否为root

    [[ $EUID -ne 0 ]] && echo -e "[Error] This script must be run as root!" && exit 1

可以使用ANSI转义代码图形再现序列(SGR sequence)输出彩色警示文字

    # Color
    red='\033[0;31m'
    green='\033[0;32m'
    yellow='\033[0;33m'
    plain='\033[0m'

    # Make sure only root can run our script
    [[ $EUID -ne 0 ]] && echo -e "[${red}Error${plain}] This script must be run as root!" && exit 1

## 附ANSI转义代码图形再现序列(SGR sequence)
### ANSI escape sequences - CSI
```Control Sequence Introducer```, CSI是ANSI转义序列中最有用处的序列，用两字符序列```ESC [```表示，
这个序列是由```控制字符ESC```（通常用```^[```或```<ESC>```表示），加上后面的左方括号字符```[```组
成，即```^[[```。因为大多数文本编辑器将键盘上的ESC键解释为其它功能，所以不能仅敲击ESC键，比如在
xterm终端中要输出^[这个字符，你需要先按Ctrl + v，然后按ESC键

在bash中，控制字符ESC也支持```\e、\033或\x1b```三种转义字符的写法，大写字母也行

### 设置显示属性 - SGR
要控制显示格式，必须使用```Set Graphic Rendition```, SGR转义序列```ESC [ parameters m```，其中```m```
表示这是SGR序列，```parameters```是控制代码，可以有多个代码组合，中间用分号```;```隔开，如果不指定
代码```ESC [ m```相当于```ESC [ 0 m```（重置所有显示控制属性为默认设置）。

显示控制代码有3类：

    * 效果控制代码
    * 前景色控制代码（即字体颜色）
    * 背景色控制代码

#### 效果控制代码

| 代码 | 效果 | 备注 |
| :------: | :------: | :------: |
| 0 | 重置所有显示属性为默认设置 | reset all attributes to their defaults |
| 1 | 字体加粗 | set bold |
| 4 | 字体加下划线 | set underscore |
| 5 | 字体闪烁 | set blink |
| 7 | 前景色与背景色调转 | set reverse video |

#### 字体颜色和背景颜色控制代码
前景色控制代码和背景色控制代码都使用两位数表示，前景色使用3开头，而背景色使用4开头，第二位数字表示具体颜色 <br />
字体颜色：30:黑 31:红 32:绿 33:黄 34:蓝色 35:紫色 36:深绿 37:白色 <br /> 
背景：40:黑 41:深红 42:绿 43:黄色 44:蓝色 45:紫色 46:深绿 47:白色。<br />
因此，要设置红色前景则发送代码```ESC [31m```，要设置黄色背景可以使用代码```ESC [43m```；也可以组合使用，比如要
设置字体颜色为红色、背景为黄色、且字体加粗，则使用代码```ESC [31;43;1m```.

### bash修改字符颜色
想在bash命令行或脚本中使用带颜色的字符，可以使用echo命令，像发送普通文本一样，将ANSI转义字符序列发送到终端会话：
```
1. 使用`^[`这个字符，你需要先按`Ctrl + v`，然后按`ESC`键（你不能直接复制我的哦~）
# echo ^[[31m红色字体

2. 使用bash的ESC转义控制字符，注意echo要使用-e选项，且后面的字符要用引号包括起来，单引号或双引号都可以
# echo -e "\e[31m红色字体" 
# echo -e "\033[31m红色字体"
# echo -e "\x1b[31m红色字体"

# echo -e "\E[31m红色字体"  # 大写字母
# echo -e "\x1B[31m红色字体"
```
上述代码都可以显示红色字体，但是你会注意到在shell打印出echo命令中的文本之后，新的提示符仍然使用该颜色效果，需要
使用重置控制码0（即ESC [0m，）将终端重置为正常显示：
```
# echo ^[[31m红色字体^[[0m
# echo -e "\e[31m红色字体\e[0m"
# echo -e "\033[31m红色字体\033[0m"
# echo -e "\x1b[31m红色字体\x1b[0m"
```
同时设置多个控制代码，用;隔开，顺序无关：
```
# echo -e "\e[31;42;1m绿底红字加粗\e[0m"
# echo -e "\e[1;42;31m绿底红字加粗\e[0m"
# echo -e "\e[1;42;31;5m绿底红字加粗，且字体闪烁\e[0m"
# echo -e "\e[1;42;31;5;4m绿底红字加粗，且字体闪烁、带下划线\e[0m"
```
参考链接: [如何修改PS1命令行提示符的颜色](http://www.madmalls.com/blog/post/how-to-change-the-output-color-of-echo-in-linux/) 
