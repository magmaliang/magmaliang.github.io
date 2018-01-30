layout: post
title:  "shadowsocks 配置 ubuntu"
date:   2017-11-17
author: 梁龙飞
tags: 工具
---

在很久之前自己花了一个通宵部署过一台shadowsocks server,当时本想写一遍攻略来着，因为从网上找到各种带坑的（因为随着时间的推移，环境和软件都有所变化）。但是坐在那沉思了一会，觉得整个过程了然于胸，没有写的必要了。

纵然此情景坑过我无数次，但总归于我是个性情中人，十分愿意审时度势、活在当下。

然后就到了今天，新买了一台虚机，结果一个ss硬是搞了一个下午——一部分原因是自己装逼没有选择自己一直熟悉的ubuntu，装了centos。所以最终是吃了个大亏，所以今天拼着周五不提前下班，也要给这玩意写了……最后还是认怂换成了ubuntu。。。

## shadowsocks与shadowsocks-libev
我记忆不大清楚，以为shadowsocks升级了，照着shadowsocks-libev装，结果在centos上装了一下午没成，多数是因为各种说 package 不存在。

原来shadowsocks-libev是几个新的人贱贱地弄出来的……

以后一定要不图急，收集了解好信息再下手。（关键我搜shadowsocks排在第一的竟然是shadowsocks-libev）


## 使用pip安装

```
pip install shadowsocks;
```

使用pip装最省事，我使用其他都特别费力，还有让我下包下来自己编译执行的，然后我照着做又出来什么autoconf需要升级，autoconf升级不了。。。反正是连环阵。

以后我就记着，装shadowsocks我只认**pip**

## 常用命令

shadowsocks安装后，会注册一个ssserver的命令，除了要写下配置文件，基本这个命令就搞定一切了:
```
ssserver -c 配置文件 -d start/stop/reload
```

## 同步文件命令： rsync

我是vim渣，修改几个字符还可以，写整个文件这种事我实在干不出来——就算是一个shadowsocks.json

```
rsync -rv /usr/local/var/www/* root@xx.xx.xx.xx:/usr/share/nginx/www/mytest/
```

## 免密登录
每次登录同步远程机器也是麻烦，以前的时候是手动copy rsa_pub去远程机器，这几天发现了更高级的玩意：专属命令：

```
# -p 是 port
ssh-copy-id -p 8129 user@remote
ssh-copy-id user@remote
```










