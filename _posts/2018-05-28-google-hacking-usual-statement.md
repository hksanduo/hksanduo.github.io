---
title: "google hacking web常用语法收集"
date:   2018-05-28 12:00 +0800
layout: post
tag: 
- google
categories:
- security
- php
- web
---

# google hacking web常用语法收集
------

# 谷歌hacking语法常用信息收集
## 目录遍历漏洞
语法为: site:域名 intitle:index.of
## 配置文件泄露  
语法为: site:域名 ext:xml | ext:conf | ext:cnf | ext:reg | ext:inf | ext:rdp | ext:cfg | ext:txt | ext:ora | ext:ini
## 数据库文件泄露  
site:域名 ext:sql | ext:dbf | ext:mdb
## 日志文件泄露
site:域名 ext:log
## 备份和历史文件 
site:域名 ext:bkf | ext:bkp | ext:bak | ext:old | ext:backup
## SQL错误  
site:域名 intext:”sql syntax near” | intext:”syntax error has occurred” | intext:”incorrect syntax near” | intext:”unexpected end of SQL command” | intext:”Warning: mysql_connect()” | intext:”Warning: mysql_query()” | intext:”Warning: pg_connect()”
## 公开文件信息  
site:域名 ext:doc | ext:docx | ext:odt | ext:pdf | ext:rtf | ext:sxw | ext:psw | ext:ppt | ext:pptx | ext:pps | ext:csv | ext:xml
## phpinfo()  
site:域名 ext:php intitle:phpinfo “published by the PHP Group”

------
以上仅仅是自己学习总结的资料，有什么不足请及时联系本人，欢迎一起交流。

