---
layout: post
title: "Debian禁用不必要的开机自启动服务"
date: 2013-12-30 20:00
comments: true
categories: 
---

经常为了尝试安装各种软件，比如Mongodb、Redis、Nginx等，但实际上用到的时候不多。这些软件都是用apt-get方式安装的，安装完默认都是开机启动的，慢慢开始影响开机启动的速度了。不让他们自启动可以直接去/etc/rc?.d修改，把S改成K就可以，但是手动改太麻烦了。Debian有一个命令，但是每次改都要现查，老是记不住。写一篇博客加深一下印象吧。

这个命令是`update-rc.d`，看名字就知道是修改rc?.d的。根据`man update-rc.d`，用法如下：
``` bash
update-rc.d [-n] [-f] name remove

update-rc.d [-n] name defaults

update-rc.d [-n] name

update-rc.d [-n] name disable|enable [ S|2|3|4|5 ]
```
用到的是最后一个，如果最后面的运行级别不指定则修改所有级别。
```
# 禁止Redis开机启动
sudo update-rc.d redis-server disable
# Redis开机启动
sudo update-rc.d redis-server enable
```

写这篇博客时顺便学会了一个查看所有服务运行状态的命令，+表示正在运行，-表示没有运行，?表示不详（？），可能是不支持用status查询状态吧。

``` bash
sudo service --status-all
# ==>
 [ + ]  acpi-fakekey
 [ - ]  acpi-support
 [ + ]  acpid
 [ ? ]  alsa-utils
 [ - ]  anacron
...
```
