---
title: "批处理bat echo中文乱码解决方法"
date:   2019-09-27  11:00:00 +0800
layout: post
tag:
- Note
- bat
categories:
- Windows
---

批处理bat echo中文乱码解决方法
------
# 问题
运行批处理bat文件，中文输出乱码，网上找到的解决方案，通过指定输出编码来解决：`chcp 65001`   
以下是测试用例
```
@echo off
REM 声明采用UTF-8编码
chcp 65001
echo test
echo 中文测试
pause
```
