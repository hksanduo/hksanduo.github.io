---
title: "解决Python3 控制台输出InsecureRequestWarning的问题"
date:   2018-05-28 12:00 +0800
layout: post
tag: 
- Python
categories:
- Python
- Web
---

# google hacking web常用语法收集
------
解决Python3 控制台输出InsecureRequestWarning的问题
------
问题：

使用Python3 requests发送HTTPS请求，已经关闭认证（verify=False）情况下，控制台会输出以下错误：
```
InsecureRequestWarning: Unverified HTTPS request is being made. Adding certificate verification is strongly advised. See: https://urllib3.readthedocs.io/en/latest/advanced-usage.html#ssl-warnings
```
解决方法：

在代码中添加以下代码即可解决：
```
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
```
Python2添加如下代码即可解决：
```
from requests.packages.urllib3.exceptions import InsecureRequestWarning
# 禁用安全请求警告
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
```
摘自：（https://www.cnblogs.com/ernana/p/8601789.html）
