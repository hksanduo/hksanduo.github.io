---
title: "centos7 svn 客户端错误svn: E210007: Unable to connect to a repository at URL的解决方法"
date:   2019-07-08  10:00:00 +0800
layout: post
tag:
- Note
categories:
- Linux
---

centos7 svn 客户端错误svn: E210007: Unable to connect to a repository at URL的解决方法
------
# 问题
centos7 使用svn同步代码，同步过程中报错
```
svn: E210007: Unable to connect to a repository at URL 'svn://git.oschina.net/cqcqphper/taskPHP'
svn: E210007: Cannot negotiate authentication mechanism
```
# 解决方法
服务器缺少cyrus-sasl cyrus-sasl-plain cyrus-sasl-ldap 组件。
运行以下命令，成功解决
`sudo yum install cyrus-sasl cyrus-sasl-plain cyrus-sasl-ldap`
