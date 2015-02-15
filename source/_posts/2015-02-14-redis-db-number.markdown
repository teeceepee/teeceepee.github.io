---
layout: post
title: "Redis Database Number"
date: 2015-02-14 22:00
comments: true
categories: Redis
---

Redis 使用 DB number 实现类似关系型数据库中 schema 的功能。不同 DB number 表示的数据库是隔离的，但是目前只能使用数字来表示一个数据库，Ubuntu 默认的配置文件配置了16个数据库，DB number 是从0开始的，并且默认连接0号数据库。

### 使用命令行连接其他数据库

可以使用 `-n` 选项指定连接的数据库号。
``` bash
$ redis-cli -n <dbnumber>

$ redis-cli -n 2
127.0.0.1:6379[2]>
```

如果已经执行 `redis-cli` 进入了 Redis 控制台，可以使用 `SELECT` 命令选择其他数据库。
``` bash
127.0.0.1:6379> SELECT 1
OK
127.0.0.1:6379[1]>
```

在 Redis 控制台中，如果当前连接的不是0号数据库，提示符中会在方括号内显示当前连接的数据库号。

### 使用URL连接其他数据库

在连接 URL 后面添加数据库号即可。
```
redis://127.0.0.1:6379/<dbnumber>

redis://127.0.0.1:6379/1
```

### 连接到不存在的数据库号
在命令行中，如果指定的数据库号超出了配置文件中的范围，貌似都会连接到0号数据库。使用 `redis-cli -n 10000` 这样的方式连错误都没有提示，而在Redis控制台中用 `SELECT 10000` 这样的方式会有错误提示，不过依然可以正常使用。不知道这算不算是 BUG。

```
$ redis-cli -n 10000
127.0.0.1:6379[10000]>
```
```
127.0.0.1:6379> SELECT 10000
(error) ERR invalid DB index
127.0.0.1:6379[10000]> set number 10000
OK
127.0.0.1:6379[10000]>
```
