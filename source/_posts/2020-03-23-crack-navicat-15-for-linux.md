---
title: "Linux下激活navicat15"
date:   2020-03-23  22:36:00 +0800
layout: post
tag:
- SQL
- Crack
categories:
- Linux
---

Linux下激活navicat15

-------
## 背景
> Navicat 是香港卓软数字科技有限公司生产的一系列 MySQL、MariaDB、MongoDB、Oracle、SQLite、PostgreSQL 及 Microsoft SQL Server 的图形化数据库管理及发展软件。它有一个类似浏览器的图形用户界面，支持多重连线到本地和远程数据库。它的设计合乎各种用户的需求，从数据库管理员和程序员，到各种为客户服务并与合作伙伴共享信息的不同企业或公司。   --wikipedia

Navicat这个工具很强大,方便IT人员日常对数据库进行管理，但是由于自己太穷，所以买不起正版授权，从网上看到有大佬放出patch，根据大佬提供的wiki和patch源码，在自己的本机成功激活Navicat15,这里只是在记录自己激活的步骤，有条件还是去买个永久版吧。

## 激活
### 下载navicat。
从[官方网站](https://www.navicat.com/en/download/navicat-premium)下载navicat,你会得到一个AppImage文件。例如 navicat15-premium-en.AppImage。
我假定这个AppImage文件在 ~/Desktop 文件夹下。

### 提取AppImage文件
提取AppImage文件里的所有文件到一个文件夹。例如：
```
$ mkdir ~/Desktop/navicat15-premium-en
$ sudo mount -o loop ~/Desktop/navicat15-premium-en.AppImage ~/Desktop/navicat15-premium-en
$ cp -r ~/Desktop/navicat15-premium-en ~/Desktop/navicat15-premium-en-patched
$ sudo umount ~/Desktop/navicat15-premium-en
$ rm -rf ~/Desktop/navicat15-premium-en
```
这里主要目的是从AppImage中提取文件，放到~/Desktop/navicat15-premium-en-patched目录中

### 编译patcher和keygen
### 编译准备
1、请确保你安装了下面几个库：　
  - capstone
  - keystone
  - rapidjson

作者wiki上使用的Linux版本为Ubunut，我个人使用的是Archlinux，我这里就以Archlinux为例进行讲解patcher和keygen的编译,执行以下指令，直接安装对应依赖库：
```
sudo pacman -S capstone keystone rapidjson
```
2、确定你的gcc支持C++17特性。

### 编译
```
$ git clone -b linux --single-branch https://github.com/DoubleLabyrinth/navicat-keygen.git
$ cd navicat-keygen
$ make all
```
如果编译成功，在navicat-keygen的bin目录下会生成navicat-keygen和navicat-patcher两个文件

###　使用 navicat-patcher 替换官方公钥
使用 navicat-patcher 替换官方公钥。
```
Usage:
    navicat-patcher [--dry-run] <Navicat Installation Path> [RSA-2048 Private Key File]

        [--dry-run]                   Run patcher without applying any patches.
                                      This parameter is optional.

        <Navicat Installation Path>   Path to a directory where Navicat locates
                                      This parameter must be specified.

        [RSA-2048 Private Key File]   Path to a PEM-format RSA-2048 private key file.
                                      This parameter is optional.
```
例如：
```
$ ./bin/navicat-patcher ~/Desktop/navicat15-premium-en-patched
```
Navicat Premium 15.0.8 Linux 英文版 已经通过测试。

下面是一份样例输出：
```
**********************************************************
*       Navicat Patcher (Linux) by @DoubleLabyrinth      *
*                  Version: 1.0                          *
**********************************************************

Press ENTER to continue or Ctrl + C to abort.

[+] Try to open libcc.so ... Ok!

[+] PatchSolution0 ...... Ready to apply
    RefSegment      =  1
    MachineCodeRva  =  0x000000000141fbf0
    PatchMarkOffset = +0x0000000002a25648

[*] Generating new RSA private key, it may take a long time...
[*] Your RSA private key:
    -----BEGIN RSA PRIVATE KEY-----
    MIIEowIBAAKCAQEAwGklHmDx9hacVUjT94Ydpy/1mTHJ7lJy6aGu84MjQDHBw4Ni
    iNG+axcv0gi5RATceD0DTZdF/Mt2dklWwMfGi3Ztk3Axbnof/byDCEeriQ79bCEb
    1rPqiVmXH54wwS/6kM8d+rQW/xx6WWndo8JvasPYApRjW9moxnOT4ylzvjw/AMzA
    Nw0dPfNqRtdYcOflIvP3PvwFhaYqb9mk7LhnfBqUF4fKwnPwtnC+g2L8V2gPlHQb
    NIOuxP7krX6lreEn1vK6E03doV1ZGs74ZYwcQGcb7RFPt/gVATzN/E8CILBq8pBt
    O19Cpv44cvXY2fBDL9q5UauS4dqvI9EjAjFRvwIDAQABAoIBAEUQeMZivfc7PnpO
    XednOJWeXWXTvUvSRHUgGBBIbgrI0WhAbMn3n4YJGJ0njHih1hFCtUDQn8qRrb/f
    q0gfbWD57XMSvmuNYpZNaCs8rpHP059QcxGqGvGaOuiae52cfzAjZ/tpUSfZLQGT
    Qn9Zd2y3R34FjXSWuEIjkl3jrywFDCoxtsVQKRBhJhprUgytRT4qlQlDkG4OcZ+T
    GJP+TU1tqvv7bcP7vIEMTnkrIjyfYfUPNK+HIoV8obGHDWxVhlcvvxTvtd1IsFfF
    hqvfeVdWWl5krWDCL1wME7ipY3N240rIXAlR9WQTMkNerr83IYM0OPeXja/vYPSz
    8gDPQQECgYEA8PNcCw/5ok5iltWlvsdw/7MHx/wIpo8a37Q6+Q2KZKeVnwEFgItd
    6QGSiEeVzlC8LcIMdpxzGET8Ky+IUMs6d0u+GkF7L2gAHOKmAf+8N2zMIGBn255m
    8uvMmo8ZDRqdookJytNq9FhfBJ4XnuQ05AixG4OBjOvO+G6bAZ44NOECgYEAzG2q
    KCBA5lGHF3v5RG4j8194XCSEQ5CnEUt6ijCBh/KnQSkR64ARzWlKhQzm9L5DbPBi
    Zn2OlcDWBzaQnQVxbF43YrIwln1JaU+7oc29CDq7OEQFTeiOkEcGbOv1RTJciUaI
    FXAHeR5XOM8+DnPnrWT/9NLnO/zIA8xyWLV9up8CgYAFricIV7sR2Xk3hxfeNIN0
    c7sGOunVS2Bdz7joMCqIDu9XDUYc7qwrFw9mSRG+CGc3SPDURwHrm4y6U+eJyBC7
    yTxVECAgUPpXs/wn5eiBAf6Z8MviAIz6wxZSunbjuTHoKATkFDHcSs0KpdN2uniI
    u6+5L1N5iAGcF7dtxVYb4QKBgFeLt00L0v9Pds0F2JAHovm8ruy27rWIoyNy8X1w
    nGg98IalMflqlTUknDcyeU3ilTl4EIvXxVI4jO/13CSCijpgdtObf9dSvDePX8lB
    NGzryfWkm6jrqPH8mLHYsub5VEutmuWjXm/uIGhByE+kL1lAYaTVFRXJgVavHWEE
    CF39AoGBAIni+wquohFgYTrCs8jhvHwK+llYqnDVjFCm4EetHPQpnAfr17/WSpI+
    dJQHXd1mnLPSE0GxMa7uZSkQ82Ph/HBCJY+Ht/7czo31Jws0nUUtOOCjTuoYHV5b
    bYNak7aYKLsn1vDYSr9BqZp56p8ZLD9ObWXCJ6KifFd6n6iWKnz+
    -----END RSA PRIVATE KEY-----
[*] Your RSA public key:
    -----BEGIN PUBLIC KEY-----
    MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAwGklHmDx9hacVUjT94Yd
    py/1mTHJ7lJy6aGu84MjQDHBw4NiiNG+axcv0gi5RATceD0DTZdF/Mt2dklWwMfG
    i3Ztk3Axbnof/byDCEeriQ79bCEb1rPqiVmXH54wwS/6kM8d+rQW/xx6WWndo8Jv
    asPYApRjW9moxnOT4ylzvjw/AMzANw0dPfNqRtdYcOflIvP3PvwFhaYqb9mk7Lhn
    fBqUF4fKwnPwtnC+g2L8V2gPlHQbNIOuxP7krX6lreEn1vK6E03doV1ZGs74ZYwc
    QGcb7RFPt/gVATzN/E8CILBq8pBtO19Cpv44cvXY2fBDL9q5UauS4dqvI9EjAjFR
    vwIDAQAB
    -----END PUBLIC KEY-----

*******************************************************
*                   PatchSolution0                    *
*******************************************************
[*] Previous:
+0x0000000000000070                          01 00 00 00 05 00 00 00          ........
+0x0000000000000080  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000000000090  00 00 00 00 00 00 00 00 48 56 a2 02 00 00 00 00  ........HV......
+0x00000000000000a0  48 56 a2 02 00 00 00 00 00 10 00 00 00 00 00 00  HV..............
[*] After:
+0x0000000000000070                          01 00 00 00 05 00 00 00          ........
+0x0000000000000080  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000000000090  00 00 00 00 00 00 00 00 d8 57 a2 02 00 00 00 00  .........W......
+0x00000000000000a0  d8 57 a2 02 00 00 00 00 00 10 00 00 00 00 00 00  .W..............

[*] Previous:
+0x0000000002a25640                          00 00 00 00 00 00 00 00          ........
+0x0000000002a25650  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a25660  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a25670  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a25680  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a25690  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a256a0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a256b0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a256c0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a256d0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a256e0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a256f0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a25700  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a25710  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a25720  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a25730  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a25740  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a25750  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a25760  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a25770  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a25780  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a25790  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a257a0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a257b0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a257c0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000002a257d0  00 00 00 00 00 00 00 00                          ........        
[*] After:
+0x0000000002a25640                          ef be ad de 4d 49 49 42          ....MIIB
+0x0000000002a25650  49 6a 41 4e 42 67 6b 71 68 6b 69 47 39 77 30 42  IjANBgkqhkiG9w0B
+0x0000000002a25660  41 51 45 46 41 41 4f 43 41 51 38 41 4d 49 49 42  AQEFAAOCAQ8AMIIB
+0x0000000002a25670  43 67 4b 43 41 51 45 41 77 47 6b 6c 48 6d 44 78  CgKCAQEAwGklHmDx
+0x0000000002a25680  39 68 61 63 56 55 6a 54 39 34 59 64 70 79 2f 31  9hacVUjT94Ydpy/1
+0x0000000002a25690  6d 54 48 4a 37 6c 4a 79 36 61 47 75 38 34 4d 6a  mTHJ7lJy6aGu84Mj
+0x0000000002a256a0  51 44 48 42 77 34 4e 69 69 4e 47 2b 61 78 63 76  QDHBw4NiiNG+axcv
+0x0000000002a256b0  30 67 69 35 52 41 54 63 65 44 30 44 54 5a 64 46  0gi5RATceD0DTZdF
+0x0000000002a256c0  2f 4d 74 32 64 6b 6c 57 77 4d 66 47 69 33 5a 74  /Mt2dklWwMfGi3Zt
+0x0000000002a256d0  6b 33 41 78 62 6e 6f 66 2f 62 79 44 43 45 65 72  k3Axbnof/byDCEer
+0x0000000002a256e0  69 51 37 39 62 43 45 62 31 72 50 71 69 56 6d 58  iQ79bCEb1rPqiVmX
+0x0000000002a256f0  48 35 34 77 77 53 2f 36 6b 4d 38 64 2b 72 51 57  H54wwS/6kM8d+rQW
+0x0000000002a25700  2f 78 78 36 57 57 6e 64 6f 38 4a 76 61 73 50 59  /xx6WWndo8JvasPY
+0x0000000002a25710  41 70 52 6a 57 39 6d 6f 78 6e 4f 54 34 79 6c 7a  ApRjW9moxnOT4ylz
+0x0000000002a25720  76 6a 77 2f 41 4d 7a 41 4e 77 30 64 50 66 4e 71  vjw/AMzANw0dPfNq
+0x0000000002a25730  52 74 64 59 63 4f 66 6c 49 76 50 33 50 76 77 46  RtdYcOflIvP3PvwF
+0x0000000002a25740  68 61 59 71 62 39 6d 6b 37 4c 68 6e 66 42 71 55  haYqb9mk7LhnfBqU
+0x0000000002a25750  46 34 66 4b 77 6e 50 77 74 6e 43 2b 67 32 4c 38  F4fKwnPwtnC+g2L8
+0x0000000002a25760  56 32 67 50 6c 48 51 62 4e 49 4f 75 78 50 37 6b  V2gPlHQbNIOuxP7k
+0x0000000002a25770  72 58 36 6c 72 65 45 6e 31 76 4b 36 45 30 33 64  rX6lreEn1vK6E03d
+0x0000000002a25780  6f 56 31 5a 47 73 37 34 5a 59 77 63 51 47 63 62  oV1ZGs74ZYwcQGcb
+0x0000000002a25790  37 52 46 50 74 2f 67 56 41 54 7a 4e 2f 45 38 43  7RFPt/gVATzN/E8C
+0x0000000002a257a0  49 4c 42 71 38 70 42 74 4f 31 39 43 70 76 34 34  ILBq8pBtO19Cpv44
+0x0000000002a257b0  63 76 58 59 32 66 42 44 4c 39 71 35 55 61 75 53  cvXY2fBDL9q5UauS
+0x0000000002a257c0  34 64 71 76 49 39 45 6a 41 6a 46 52 76 77 49 44  4dqvI9EjAjFRvwID
+0x0000000002a257d0  41 51 41 42 ad de ef be                          AQAB....        

[*] Previous:
+0x000000000141fbf0  44 0f b6 24 18 48 8b 44 24 28 8b 50 f8 85 d2 79  D..$.H.D$(.P...y
+0x000000000141fc00  6f                                               o               
[*] After:
+0x000000000141fbf0  45 31 e4 48 8d 05 52 5a 60 01 90 90 90 90 90 90  E1.H..RZ`.......
+0x000000000141fc00  90                                               .               

[*] New RSA-2048 private key has been saved to
    /home/hksanduo/Downloads/navicat-keygen/RegPrivateKey.pem

*******************************************************
*           PATCH HAS BEEN DONE SUCCESSFULLY!         *
*                  HAVE FUN AND ENJOY~                *
*******************************************************

```

### 将文件重新打包成AppImage
具体指令可以参考以下：
```
$ wget 'https://github.com/AppImage/AppImageKit/releases/download/continuous/appimagetool-x86_64.AppImage'
$ chmod +x appimagetool-x86_64.AppImage
$ ./appimagetool-x86_64.AppImage ~/Desktop/navicat15-premium-en-patched ~/Desktop/navicat15-premium-en-patched.AppImage
```
### 运行刚生成的AppImage：
```
$ chmod +x ~/Desktop/navicat15-premium-en-patched.AppImage
$ ~/Desktop/navicat15-premium-en-patched.AppImage
```
### 使用 navicat-keygen 来生成 序列号 和 激活码。
```
Usage:
    navicat-keygen <--bin|--text> [--adv] <RSA-2048 Private Key File>

        <--bin|--text>    Specify "--bin" to generate "license_file" used by Navicat 11.
                          Specify "--text" to generate base64-encoded activation code.
                          This parameter must be specified.

        [--adv]                       Enable advance mode.
                                      This parameter is optional.

        <RSA-2048 Private Key File>   A path to an RSA-2048 private key file.
                                      This parameter must be specified.
```
例如：
```
$ ./bin/navicat-keygen --text ./RegPrivateKey.pem
```
你会被要求选择Navicat产品类别、Navicat语言版本和填写主版本号。之后一个随机生成的 序列号 将会给出。
```
$ ./bin/navicat-keygen --text ./RegPrivateKey.pem
**********************************************************
*       Navicat Keygen (Linux) by @DoubleLabyrinth       *
*                   Version: 1.0                         *
**********************************************************

[*] Select Navicat product:
0. DataModeler
1. Premium
2. MySQL
3. PostgreSQL
4. Oracle
5. SQLServer
6. SQLite
7. MariaDB
8. MongoDB
9. ReportViewer

(Input index)> 1

[*] Select product language:
0. English
1. Simplified Chinese
2. Traditional Chinese
3. Japanese
4. Polish
5. Spanish
6. French
7. German
8. Korean
9. Russian
10. Portuguese

(Input index)> 0

[*] Input major version number:
(range: 0 ~ 15, default: 12)> 15

[*] Serial number:
NAVM-RTVJ-EO42-IODD

[*] Your name:
你可以使用这个 序列号 来暂时激活Navicat。

之后你会被要求填写 用户名 和 组织名。你可以随意填写，但别太长。

[*] Your name: DoubleLabyrinth
[*] Your organization: DoubleLabyrinth

[*] Input request code in Base64: (Double press ENTER to end)
之后你会被要求填写请求码。注意不要关闭keygen。

断开网络. 找到注册窗口，填写keygen给你的 序列号，然后点击 激活。

通常在线激活会失败，所以在弹出的提示中选择 手动激活。

复制 请求码 到keygen，连按两次回车结束。

[*] Input request code in Base64: (Double press ENTER to end)
OaGPC3MNjJ/pINbajFzLRkrV2OaSXYLr2tNLDW0fIthPOJQFXr84OOroCY1XN8R2xl2j7epZ182PL6q+BRaSC6hnHev/cZwhq/4LFNcLu0T0D/QUhEEBJl4QzFr8TlFSYI1qhWGLIxkGZggA8vMLMb/sLHYn9QebBigvleP9dNCS4sO82bilFrKFUtq3ch8r7V3mbcbXJCfLhXgrHRvT2FV/s1BFuZzuWZUujxlp37U6Y2PFD8fQgsgBUwrxYbF0XxnXKbCmvtgh2yaB3w9YnQLoDiipKp7io1IxEFMYHCpjmfTGk4WU01mSbdi2OS/wm9pq2Y62xvwawsq1WQJoMg==

[*] Request Info:
{"K":"NAVMRTVJEO42IODD", "DI":"4A12F84C6A088104D23E", "P":"linux"}

[*] Response Info:
{"K":"NAVMRTVJEO42IODD","DI":"4A12F84C6A088104D23E","N":"DoubleLabyrinth","O":"DoubleLabyrinth","T":1575543648}

[*] Activation Code:
i45HIr7T1g69Cm9g3bN1DBpM/Zio8idBw3LOFGXFQjXj0nPfy9yRGuxaUBQkWXSOWa5EAv7S9Z1sljlkZP6cKdfDGYsBb/4N1W5Oj1qogzNtRo5LGwKe9Re3zPY3SO8RXACfpNaKjdjpoOQa9GjQ/igDVH8r1k+Oc7nEnRPZBm0w9aJIM9kS42lbjynVuOJMZIotZbk1NloCodNyRQw3vEEP7kq6bRZsQFp2qF/mr+hIPH8lo/WF3hh+2NivdrzmrKKhPnoqSgSsEttL9a6ueGOP7Io3j2lAFqb9hEj1uC3tPRpYcBpTZX7GAloAENSasFwMdBIdszifDrRW42wzXw==
```
最终你会得到一个base64编码的 激活码。将之复制到 手动激活 的窗口，然后点击 **激活**。如果没有什么意外，应该可以成功激活。

### 清理
```
$ rm ~/Desktop/navicat15-premium-en.AppImage
$ rm -rf ~/Desktop/navicat15-premium-en-patched
$ mv ~/Desktop/navicat15-premium-en-patched.AppImage ~/Desktop/navicat15-premium-en.AppImage
```
## 后续使用
你可以将激活的navicat15放置于/opt目录下或者/usr/local/share目录下，设置一个软连接，方便在终端及命令行调用，我的navicat15-premium-en.AppImage放置的目录为：/usr/local/share/,使用以下命令建立软连接
```
sudo chmod +x /usr/local/share/navicat15-premium-en.AppImage
sudo ln -sf /usr/local/share/navicat15-premium-en.AppImage /usr/local/bin/navicat15
```
![20200323-crack-navicat-15-for-linux.png](/img/20200323-crack-navicat-15-for-linux.png)

## 参考内容
- [https://gitee.com/yangzhuoming/navicat-keygen](https://gitee.com/yangzhuoming/navicat-keygen)
- [https://github.com/DoubleLabyrinth/navicat-keygen.git](https://github.com/DoubleLabyrinth/navicat-keygen.git)
- [https://github.com/AppImage/AppImageKit/](https://github.com/AppImage/AppImageKit/)
