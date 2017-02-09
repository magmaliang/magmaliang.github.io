---
layout: post
title:  "nginx中的rewrite和location"
date:   2017-01-17
author: 梁龙飞
tags: 编程
---

>在开发环境，经常使用nginx将首页的静态资料重定向到本地，用的多了，总结下各种重定向的手艺。

## location
location 并不能完成重定向，它只是一个辅助，主要起过滤作用，所以它可以多层嵌套。可以使用正则来匹配url。比较重要的是多规则匹配遵循以下优先级：

## proxy_pass
proxy_pass 可以重定向域名。例如：
```perl
# 将所有js后缀的请求代理到http://www.proxy.com下
location /*.js {
	proxy_pass http://www.proxy.com;
}
```
原始url: http://www.source.com/rs/js/main.js
实际访问地址：http://www.proxy.com/rs/js/main.js

## return
```
Syntax:	return code [text];
		return code URL;
		return URL;
Default:	—
Context:	server, location, if
```
Context表示当前命令可以使用的环境。return 主要是返回状态码，虽然也可以直接return url，但这不是它的特色。


## rewrite
```
rewrite regex url [flag]
```
regex匹配上$url时，则执行重写。url中可以使用regex中捕获的子组。

## try_files
```
try_files url...url;
```
try_files会从左到右地尝试所有路径，直到它找到一个实际存在的资源，将它返回。这可以方便地设置优先级相关的逻辑。

## alias
```
Syntax:	alias path;
Default:	—
Context:	location
```

alias其实是一个replace，它在location命中后，直接返回指定的path资源。

## 正则

nginx编译的时候需要 pcre（Perl Compatible Regular Expressions），这个库由 Philip Hazel开发，它作为非常完善的正则表达式模块被很多其他工具和语言所使用。

下面这个网站记录了正则的工业标准和各种语言的支持程度。

[http://www.regular-expressions.info/](http://www.regular-expressions.info/)

pcre的语法与各语言相似，如果你精通其他语言的正则，那么只需要一个小时就能掌握pcre的正则。因为我对javascript的正则较了解，所以只拿这二者作了下比较。

pcre比js中的正则多出的功能有以下几点：

- 自定义捕获组名，(?P<name>aaa),如果这个正则捕获成功，那么在下文中可以使用$name进行读取（在perl中你得使用$+{name}，我也不明白为啥nginx中这样可以）。

- Lookbehinds ,/(?<=b)aaa/,匹配baaa中的aaa。javascript中只有向前查找。

- 多出的环境变量，如$`等。

## 参考

- [创建nginx重写规则](https://www.nginx.com/blog/creating-nginx-rewrite-rules/)

一篇超级棒的英文，我准备抽时间翻译下。

## 备注

这是nginx官方的一个[config full example](https://www.nginx.com/resources/wiki/start/topics/examples/fullexample2/)。

```perl
user  www www;
worker_processes  2;
pid /var/run/nginx.pid;

# [ debug | info | notice | warn | error | crit ]
error_log  /var/log/nginx.error_log  info;

events {
  worker_connections   2000;
  # use [ kqueue | rtsig | epoll | /dev/poll | select | poll ] ;
  use kqueue;
}

http {
  include       conf/mime.types;
  default_type  application/octet-stream;

  log_format main      '$remote_addr - $remote_user [$time_local]  '
    '"$request" $status $bytes_sent '
    '"$http_referer" "$http_user_agent" '
    '"$gzip_ratio"';

  log_format download  '$remote_addr - $remote_user [$time_local]  '
    '"$request" $status $bytes_sent '
    '"$http_referer" "$http_user_agent" '
    '"$http_range" "$sent_http_content_range"';

  client_header_timeout  3m;
  client_body_timeout    3m;
  send_timeout           3m;

  client_header_buffer_size    1k;
  large_client_header_buffers  4 4k;

  gzip on;
  gzip_min_length  1100;
  gzip_buffers     4 8k;
  gzip_types       text/plain;

  output_buffers   1 32k;
  postpone_output  1460;

  sendfile         on;
  tcp_nopush       on;

  tcp_nodelay      on;
  send_lowat       12000;

  keepalive_timeout  75 20;

  # lingering_time     30;
  # lingering_timeout  10;
  # reset_timedout_connection  on;


  server {
    listen        one.example.com;
    server_name   one.example.com  www.one.example.com;

    access_log   /var/log/nginx.access_log  main;

    location / {
      proxy_pass         http://127.0.0.1/;
      proxy_redirect     off;

      proxy_set_header   Host             $host;
      proxy_set_header   X-Real-IP        $remote_addr;
      # proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;

      client_max_body_size       10m;
      client_body_buffer_size    128k;

      client_body_temp_path      /var/nginx/client_body_temp;

      proxy_connect_timeout      90;
      proxy_send_timeout         90;
      proxy_read_timeout         90;
      proxy_send_lowat           12000;

      proxy_buffer_size          4k;
      proxy_buffers              4 32k;
      proxy_busy_buffers_size    64k;
      proxy_temp_file_write_size 64k;

      proxy_temp_path            /var/nginx/proxy_temp;

      charset  koi8-r;
    }

    error_page  404  /404.html;

    location /404.html {
      root  /spool/www;

      charset         on;
      source_charset  koi8-r;
    }

    location /old_stuff/ {
      rewrite   ^/old_stuff/(.*)$  /new_stuff/$1  permanent;
    }

    location /download/ {
      valid_referers  none  blocked  server_names  *.example.com;

      if ($invalid_referer) {
        #rewrite   ^/   http://www.example.com/;
        return   403;
      }

      # rewrite_log  on;
      # rewrite /download/*/mp3/*.any_ext to /download/*/mp3/*.mp3
      rewrite ^/(download/.*)/mp3/(.*)\..*$ /$1/mp3/$2.mp3 break;

      root         /spool/www;
      # autoindex    on;
      access_log   /var/log/nginx-download.access_log  download;
    }

    location ~* ^.+\.(jpg|jpeg|gif)$ {
      root         /spool/www;
      access_log   off;
      expires      30d;
    }
  }
}
```
