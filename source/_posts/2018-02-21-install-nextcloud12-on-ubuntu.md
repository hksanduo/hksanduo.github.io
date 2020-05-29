---
title: "ubuntu server 安装nextcloud12"
date:   2018-02-21 14:05:41 +0800
layout: post
tag: 
- Linux
categories:
- Linux
---

# ubuntu server 安装nextcloud12
------
## 1.更新系统安装必要的依赖包
```
[user]$ sudo apt-get update && sudo apt-get -y upgrade
[user]$ sudo apt-get install software-properties-common nano wget
```
## 2.安装ＭariaDB10以上版本，如果不清楚安装的版本是否符合nextcloud要求，可以到官网上查询
nextcloud官网：[https://nextcloud.com/install/#](https://nextcloud.com/install/#)
MariaDB安装与配置:
```
[user]$ sudo apt-get install -y mariadb-server
```
安装完毕以后，运行：
```
[user]$ mysql_secure_installation
```
这个的主要目的是更新和配置数据库，同时设置相应的root密码，我在安装的时候遇到一个问题就是在普通用户下无法成功连接到数据库，需要切换到root进行，目前没有解决这个问题，如果哪位大佬可以解决，请给我留言．
重启ＭariaDB服务：
```
[user]$ sudo service mysql restart
```
登录数据库并且创建相应的数据库
```
[user]$ mysql -uroot -p
```
***注意这一步如果登录失败，请切换root用户执行
创建数据库：
```
MariaDB [(none)]> CREATE DATABASE nextcloud;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'localhost' IDENTIFIED BY 'strong_password';
MariaDB [(none)]> FLUSH PRIVILEGES;
MariaDB [(none)]> \q
```
## 3.安装php
```
[user]$ sudo apt-get -y install php-fpm php-cli php-json php-curl php-imap php-gd php-mysql php-xml php-zip php-intl php-mcrypt php-imagick php-mbstring
```
这里设置的**php memory**为512MB，**upload_max_filesize** 和 **post_max_size** 为200MB
```
[user]$ sed -i "s/memory_limit = .*/memory_limit = 512M/" /etc/php/7.0/fpm/php.ini
[user]$ sed -i "s/;date.timezone.*/date.timezone = UTC/" /etc/php/7.0/fpm/php.ini
[user]$ sed -i "s/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=1/" /etc/php/7.0/fpm/php.ini
[user]$ sed -i "s/upload_max_filesize = .*/upload_max_filesize = 200M/" /etc/php/7.0/fpm/php.ini
[user]$ sed -i "s/post_max_size = .*/post_max_size = 200M/" /etc/php/7.0/fpm/php.ini
```
***配置文件的位置**：
```
php:/etc/php/7.0/fpm/php.ini
```
配置**PHP-FPM**
配置文件使用默认的就好，如果有问题，请参考下面关于环境变量的配置
```
[user]$ sudo vim /etc/php/7.0/fpm/pool.d/www.conf
```
**环境变量**：
```
env[HOSTNAME] = $HOSTNAME
env[PATH] = /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp
```
重启**PHP-FPM**服务：
```
[user]$ sudo service php7.0-fpm restart
```
## 4.下载并且安装ＮextCloud12
这里将Nextcloud安装到**/var/www/nextcloud/**目录下，安装目录根据个人爱好就行
```
[user]$ wget https://download.nextcloud.com/server/releases/nextcloud-12.0.0.zip
[user]$ unzip nextcloud-12.0.0.zip
[user]$ sudo mkdir /var/www/
[user]$ sudo mv nextcloud /var/www/
[user]$ rm -f nextcloud-12.0.0.zip
[user]$ sudo chown -R www-data: /var/www/nextcloud
```
## 5.安装和配置nginx
```
[user]$ sudo apt-get install nginx-extras nginx
```
## 6.这里创建相应的ssl证书，方便大家访问，如果有相应的证书，将证书放置/etc/nginx/ssl目录下，如果是新手，请注意证书的名字
```
[user]$ sudo mkdir -p /etc/nginx/ssl
[user]$ cd /etc/nginx/ssl
[user]$ sudo openssl genrsa -des3 -passout pass:x -out nextcloud.pass.key 2048
[user]$ sudo openssl rsa -passin pass:x -in nextcloud.pass.key -out nextcloud.key
[user]$ sudo rm nextcloud.pass.key
[user]$ sudo openssl req -new -key nextcloud.key -out nextcloud.csr
[user]$ sudo openssl x509 -req -days 365 -in nextcloud.csr -signkey nextcloud.key -out nextcloud.crt
```
## 7.为nginx服务器创建相应的配置文件：
```
[user]$ sudo nano /etc/nginx/sites-available/nextcloud
```
文件内容：
```
server {
    listen 80;
    server_name my.nextcloud.com;
    return 301 https://$server_name$request_uri;
}
server {
    listen 443 ssl http2;
    server_name my.nextcloud.com;
    root /var/www/nextcloud;

    ssl on;
    ssl_certificate     /etc/nginx/ssl/nextcloud.crt;
    ssl_certificate_key /etc/nginx/ssl/nextcloud.key;
    ssl_session_timeout 5m;
    ssl_ciphers               'AES128+EECDH:AES128+EDH:!aNULL';
    ssl_protocols              TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;

    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header X-Download-Options noopen;
    add_header X-Permitted-Cross-Domain-Policies none;

    access_log  /var/log/nginx/nextcloud.access.log;
    error_log   /var/log/nginx/nextcloud.error.log;

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    location = /.well-known/carddav { 
        return 301 $scheme://$host/remote.php/dav; 
    }
    location = /.well-known/caldav { 
        return 301 $scheme://$host/remote.php/dav; 
    }

    client_max_body_size 512M;
    fastcgi_buffers 64 4K;
    gzip off;

    error_page 403 /core/templates/403.php;
    error_page 404 /core/templates/404.php;

    location / {
        rewrite ^ /index.php$uri;
    }

    location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)/ {
        deny all;
    }

    location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console) {
        deny all;
    }

    location ~^/(?:index|remote|public|cron|core/ajax/update|status|ocs/v[12]|updater/.+|ocs-provider/.+|core/templates/40[34])\.php(?:$|/) {
        include fastcgi_params;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param HTTPS on;
        #Avoid sending the security headers twice
        fastcgi_param modHeadersAvailable true;
        fastcgi_param front_controller_active true;
        fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
        fastcgi_intercept_errors on;
        fastcgi_request_buffering off;
    }

    location ~ ^/(?:updater|ocs-provider)(?:$|/) {
        try_files $uri/ =404;
        index index.php;
    }

    location ~* \.(?:css|js)$ {
        try_files $uri /index.php$uri$is_args$args;
        add_header Cache-Control "public, max-age=7200";
        add_header X-Content-Type-Options nosniff;
        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Robots-Tag none;
        add_header X-Download-Options noopen;
        add_header X-Permitted-Cross-Domain-Policies none;
        # Optional: Don't log access to assets
        access_log off;
    }

    location ~* \.(?:svg|gif|png|html|ttf|woff|ico|jpg|jpeg)$ {
        try_files $uri /index.php$uri$is_args$args;
        access_log off;
    }

    location ~ /\.ht {
        deny all;
    }

}
```
***注意：这里需要更改相应的域名，不要直接复制**
链接相应的配置文件：
```
[user]$ sudo ln -s /etc/nginx/sites-available/nextcloud /etc/nginx/sites-enabled/nextcloud
```
重启nginx服务：
```
[user]$ sudo nginx -t
[user]$ sudo service nginx restart
```
## 8.到此，安装已经完成，接下来需要登录nextcloud并且配置相应的文件存储位置：
打开浏览器输入：https://my.nextcloud.com/install.php

>注意这里设置的域名，如果没有设置域名，在相同的网段下使用ＩＰ进行访问，我这里在同一个路由下访问nextcloud,主机的ip为192.168.1.105，怎么查看自己主机的ip方法有很多，命令也行，路由也行，怎么方便怎么来．
**https://192.168.1.105/install.php**

打开网页后，设置相应的管理员用户名和密码及相应的文件存储位置，这个位置可以随便设置，但是需要更改位置的组和拥有者为**www-data**,

***注意**：上传和下载大文件时可能会提示文件太大，需要更改上传和下载文件最大限制：

配置文件需要修改：

1. /etc/php/7.0/fpm/php.ini　

    ```
php memory;
upload_max_filesize;
post_max_size;
    ```

2. /etc/nginx/sites-available/nextcloud

    ```
client_max_body_size;
    ```

3. /var/www/nextcloud/.htaccess

    ```
php_value upload_max_filesize; 
php_value post_max_size;
php_value memory_limit;
    ```

***具体大小根据个人情况设置**

配置完成以后重启相应的服务：

```
sudo systemctl restart php7.0-fpm 
sudo systemctl restart nginx
```
