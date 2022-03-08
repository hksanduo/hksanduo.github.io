---
title: "多线程压缩数据"
date:   2022-03-08  15:56:00 +0800
layout: post
tag:
- Compress
categories:
- Os
---

多线程压缩数据

------
## 背景
最近需要和同事分享代码样本数据，数据集大概有几十个G，使用tar压缩时，发现只会调用一个核心进行压缩，速度有些鸡肋，所以查询了一些多线程压缩的资料，这里总结一下，供以后参考：

## 指定压缩程序
```
tar cf - your-archive-path | pigz > archive.tar.gz
```
或者写成：
```
tar -cf --use-compress-program=pigz -f archive.tar.gz your-archive-path
```
![调用所有核心进行压缩](/img/20220308-01.png)
默认情况下，pigz使用系统所有的核心，如果无法获取到，默认为8。 可以使用-p n请求更多信息，例如： -p 32. pigz与gzip具有相同的选项，因此您可以使用-9请求更好的压缩。 例如。
```
tar cf - your-archive-path | pigz -9 -p 32> archive.tar.gz
```

## pigz
pigz(parallel implementation of gzip)是一个并行执行的压缩工具，解压缩比gzip快，同时CPU消耗是gzip的好几倍，在对短时间内CPU消耗较高不受影响的场景下，可以使用pigz。

使用pigz解压数据，有个有趣的点就是，解压只会调用4个CPU核心
```
pigz -k -d data.tar.gz
```
![多线程解压缩](/img/20220308-02.png)
使用-p指定核心数目也不可以，有点儿奇怪，暂时没时间去深究原因。

本文提到的只是一个小技巧，个人单纯记录一下，仅做参考。

## 参考
- [https://zlib.net/pigz/](https://zlib.net/pigz/)
- [https://segmentfault.com/a/1190000038347566](https://segmentfault.com/a/1190000038347566)