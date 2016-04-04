---
layout: post
title: "尝试 Let's Encrypt"
date: 2016-04-02 20:22
comments: true
categories:
---

在计算机网络的远古时期，HTTP 的设计者一开始并没有在安全方面做太多的考虑，他们把 HTTP 设计为一个明文传输的文本协议，简单、直观。HTTP 出现后互联网经历了爆炸性的成长，HTTP 成为了使用最广泛的一种应用层网络协议，这与它简单直观的特点不无关系。然而，在互联网高速发展了几十年后，支撑 HTTP 广泛应用的几个特性却正在慢慢制约互联网的继续发展。其中一个特性，明文传输，越来越不适合存在于当今的互联网环境。明文传输，意味着在数据链路上的任何人都有可能会窥探和篡改数据，在数据往返的途中，路由器、公共 WiFi、电信运营商、GFW 等都有可能造成潜在的危害。当前人们对互联网安全性的要求越来越高，使用加密的传输协议势在必行。

好在有 HTTPS，它在 HTTP 的基础上增加了一层加密的协议，应用层的代码几乎不需要任何修改，只要部署好相关的设施就能无缝迁移到安全的 HTTPS 协议了。但是部署 HTTPS 需要向证书认证机构申请证书，这对个人来说是一件非常繁琐的事情，而且证书通常需要购买，更增加了个人网站使用 HTTPS 的难度。

然而在 2015 年，情况发生了变化，一家新的证书机构 [Let's Encrypt][letsencrypt_homepage] 开始提供免费、自动化的证书服务。它的官网是这么描述自己的：

> Let’s Encrypt is a new Certificate Authority:

> It’s free, automated, and open.

免费，Let's Encrypt 提供的证书是完全免费的，完全不需要任何支付费用。自动化的，使用它提供的工具几部操作就可以完成证书的获取，更进一步，还可以把证书的获取、更新、部署通过程序完全自动化。开放的，整个服务开发与更新都是开放的。当然，Let's Encrypt 提供的证书是安全的。


## 获取证书

在刚了解 Let's Encrypt 时，我以为需要到官网注册账号，之后才能获取到证书。后来发现完全不需要，只要证明证书绑定的域名属于你自己就可以了，非常简便。

Let's Encrypt 提供了工具来实现证书的获取和更新，可以通过 Git 下载：

``` bash
$ git clone https://github.com/letsencrypt/letsencrypt
$ cd letsencrypt
$ ./letsencrypt-auto --help
```

运行 `letsencrypt-auto` 会自动安装依赖包并且把自身升级到最新版，所以第一次执行上面最后一条命令会有一点慢。执行过命令后会创建 `/etc/letsencrypt/` 目录，证书以及其他的数据都会保存在这个目录，因此执行命令需要有 root 权限。

当 `letsencrypt-auto` 把依赖安装和升级都做完后就可以使用了。`letsencrypt-auto` 提供了好几种方式来获取证书，下面是 `letsencrypt-auto --help` 输出的一部分，描述了提供的几个子命令。

```
  (default) run        Obtain & install a cert in your current webserver
  certonly             Obtain cert, but do not install it (aka "auth")
  install              Install a previously obtained cert in a server
  renew                Renew previously obtained certs that are near expiry
  revoke               Revoke a previously obtained certificate
  rollback             Rollback server configuration changes made during install
  config_changes       Show changes made to server config during installation
  plugins              Display information about installed plugins
```

`certonly` 子命令只用来获取证书，证书的安装，也就是在 Web Server 里的配置需要手动完成。`renew` 更新已经获取到的证书，获取到的证书只有 90 天有效期，通过这个命令可以将证书的剩余有效期更新为 90 天。

`letsencrypt-auto` 使用插件机制实现证书的获取和安装，下面介绍自带的三种插件。

### Apache

`letsencrypt-auto` 提供了针对 Apache 的自动获取和安装插件，只要一条命令 `letsencrypt-auto --apache` 就完成获取和安装，但是我使用的并不是 Apache，因此无法验证这种方法。

### Webroot

`letsencrypt-auto` 提供的第二种获取证书的插件叫 Webroot。这种方式需要配置服务器上已运行的 Web Server 来实现。比如 Nginx，需要在配置文件里添加下面的配置，使得 `/.well-known/*` 这样的路径可以被访问到。修改好配置文件后不要忘记 `restart` 或 `reload` Nginx。

```
        location ~ /.well-known {
                allow all;
        }
```

第二步，需要搞清楚 Web Server 提供静态文件的根路径。基本是对应 Nginx `root` 指令指定的位置，比如 `/usr/share/nginx/html` 或 `/var/www/`，需要根据具体的配置文件来确定，我们假设这个路径叫 `$WEBROOT_PATH`，同时假设证书绑定的域名叫 `$DOMAIN` 和 `$WWW_DOMAIN`。

然后执行下面的命令就可以获取到证书了。证书以及其他数据存储在 `/etc/letsencrypt/` 目录。

```
./letsencrypt-auto certonly --webroot --webroot-path $WEBROOT_PATH -d $DOMAIN -d $WWW_DOMAIN
```

Webroot 插件的原理是配置已有的 Web Server，使 `/.well-known/*` 这样的路径可以从外部访问，同时在 Web Server 的根路径放一个 `.well-known/xxx...` 文件。这个文件是在执行上面的命令时生成的，然后把访问路径告诉 Let's Encrypt 的服务器，Let's Encrypt 服务器访问后就可以验证待绑定域名的有效性。所以在执行完上面的命令后，Web Server 的日志里应该会有几条类似这样的访问记录。

```
"GET /.well-known/acme-challenge/kiEt5Rn8rAQqZBQ9-ve6-xxxxxx HTTP/1.1" "Mozilla/5.0 (compatible; Let's Encrypt validation server; +https://www.letsencrypt.org)"
```

### Standalone

Webroot 需要配置已有的 Web Server，步骤稍显繁琐，standalone 插件可以把这些事情自动化，但前提是如果已有 Web Server 在监听 80 端口，需要暂时停掉，否则 standalone 插件不能启动自带的服务器程序。获取到的证书同样存储在 `/etc/letsencrypt/`。

```
./letsencrypt-auto certonly --standalone -d $DOMAIN -d $WWW_DOMAIN
```

## 使用证书配置 HTTPS

配置 HTTPS 已经与 Let's Encrypt 没有关系了。比如 Nginx 可以使用下面的片段配置 HTTPS，其中的域名需要修改成自己的。

```
        listen 443 ssl;

        server_name example.com www.example.com;

        ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
```

## 续签证书

上面提到过，获取到的证书有效期只有 90 天，因此需要在过期之前续签，将证书有效期重新设置为 90 天。使用 renew 子命令实现续签。

```
./letsencrypt-auto renew
```

如果想要实现自动续签，只要在一定时间间隔内重新执行上面的命令就可以了。网上的大部分教程使用 Linux 的 Cron 实现自动续签。



## 参考

[Homepage][letsencrypt_homepage]

[Getting Started](https://letsencrypt.org/getting-started/)

[Configure Nginx on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-14-04)


[letsencrypt_homepage]: https://letsencrypt.org/
