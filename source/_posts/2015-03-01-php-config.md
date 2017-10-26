---
layout: post
title:  "mac php+nginx"
date:   2015-03-01 
author: "梁龙飞"
tags: 环境初始化
---

从笔记里面移出了这份配置，并且做了一个新的tag.原因在于这些初始化弄过很多次，但都忘记了，虽然是些基础的东西，还是留存下吧。

[使用homebrew安装](!https://easyengine.io/tutorials/mac/osx-brew-php-mysql-nginx/)

## php-fpm
php 5.3.3之后集成了php-fpm，无需另外安装。

- 启动 php-fpm  
> sudo php-fpm

## nginx 配置

```
server {
    listen       80;
    server_name  localhost;
    root /usr/local/var/www;
    index index.php;

    location / { 
        index index.php;
        autoindex on; 
    }
    location ~ \.php$ {
        include /usr/local/etc/nginx/fastcgi.conf;
        fastcgi_intercept_errors on; 
        fastcgi_pass   127.0.0.1:9000; 
    }   

}
```
## mysql
- 启动服务 mysql.server start

- php连接时问题
  mac上经常有mysql_connect找不到，解决方案如下：
http://stackoverflow.com/questions/4219970/warning-mysql-connect-2002-no-such-file-or-directory-trying-to-connect-vi
