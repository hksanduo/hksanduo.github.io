---
title: "ubuntu 安装php7.0 xdebug"
date:   2018-05-18 12:00 +0800
layout: post
tag: 
- php
categories:
- code
- php
---

# ubuntu 安装php xdebug
------

环境:ubuntu 17.04
php:7.0
xdebug:2.7.0
官方指导教程:(https://xdebug.org/wizard.php)

## 安装
(xdebug 官网https://xdebug.org/)
需要下载xdebug进行编译安装

	$ https://xdebug.org/files/xdebug-2.7.0alpha1.tgz
    #我这里下载的是2.7.0版本,可视情况下载
    
解压文件
	
    $ tar -zxvf xdebug-2.7.0alpha1.tgz 
    
使用phpize进行编译(在xdebug目录下执行)

	$ phpize
    # 如果没有phpize,请安装php7.0-dev
    $ ./configure --enable-xdebug 
    $ make
    $ sudo make install
    
## 配置
官方给出的配置信息太模糊,需要根据实际情况进行配置
不要编辑``` /etc/php/7.0/cli/php.ini```和添加 ```zend_extension = /usr/lib/php/20151012/xdebug.so```

创建xdebug.ini

	$sudo vim /etc/php/7.0/mods-available/xdebug.ini

```
zend_extension=xdebug.so
 
xdebug.remote_enable = 1
xdebug.remote_connect_back=1
xdebug.remote_port = 9000
xdebug.scream=1
xdebug.show_local_vars=1
xdebug.idekey=netbeans-xdebug
 
;To remove limits for xdebug_var_dump()
 
;xdebug.var_display_max_depth = 5
;xdebug.var_display_max_children = 256
;xdebug.var_display_max_data = 1024 
 
xdebug.var_display_max_depth = -1 
xdebug.var_display_max_children = -1
xdebug.var_display_max_data = -1
```
启用xdebug

	$sudo phpenmod xdebug
    
如果你使用composer,或许需要禁用xdebug在cli下:

	sudo phpdismod -s cli xdebug
