---
layout: post
title: "安装docker并手工构建一个image"
date: 2013-12-28 20:40
comments: true
categories: 
---

最近貌似docker这玩意儿比较火，有可能是虚拟化的新趋势。比较感兴趣，正好有时间就玩了玩，遇到一些问题就记下来了。

[docker](https://www.docker.io)的官网做得不错，对于新手能很方便的找到所需信息。最让人喜欢的是那个[Get started!](https://www.docker.io/gettingstarted/)交互式命令行教程，简直就是“手把手得教，一对一得学”。通过`docker version`、`docker search tutorial`等一步步深入，看完教程就能熟悉个大概了。

首先是安装docker。我的系统是Debian unstable，官方有Ubuntu的安装教程，我是按照Ubuntu的教程来的，但是并没有执行`sudo apt-get install linux-image-extra-\`uname -r\``，因为Debian貌似没有linux-image-extra-*这个软件包。安装docker只用下面几行命令即可。

``` bash
sudo sh -c "wget -qO- https://get.docker.io/gpg | apt-key add -"

sudo sh -c "echo deb http://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list"
sudo apt-get update
sudo apt-get install lxc-docker
```

安装好docker我就按[Get started!](https://www.docker.io/gettingstarted/)的命令操作，但不凑巧的是我在本机执行这些命令时可没有在线教程那么顺利，`docker pull ubuntu`并不能正常下载image，卡了一会儿输出个错误，超时了。原因猜也能猜个差不多，只能说句“F*K GFW”然后找其他办法了。还好我有个VPN，连上VPN，再试，结果还是不行，错误信息和不连VPN的还不太一样，具体原因还不清楚，再找其他解决方案吧。最终在一篇[博客](http://www.blogjava.net/yongboy/archive/2013/12/12/407498.html)的评论里找到了一种解决方案，有一点麻烦，但确实可行。

这个解决方案就是自己在本地构建一个image，然后导入到docker中，最终就和用`docker pull`从远程下载的效果一样了。该方案只能在Debian或Ubuntu上操作，制作的image也只能是Debian或Ubuntu。

首先，确认已经安装了`debootstrap`这个工具，如果没安装用下面命令安装。

``` bash
sudo apt-get update
sudo apt-get install debootstrap
```

`debootstrap`貌似是一个构建Debian系统的工具，可以从指定的源获取deb安装包并安装。过程应该和安装系统差不多，只不过是将文件都安装到某一个目录下。根据`man debootstrap`，用法如下，用一句话解释就是："debootstrap  bootstraps a basic Debian system of SUITE into TARGET from MIRROR by running SCRIPT."
```
debootstrap [OPTION...]  SUITE TARGET [MIRROR [SCRIPT]]
```

`TARGET`这里应该是写一个路径，最终构建的系统就在这个路径中。如果该路径不存在会自动创建。

`MIRROR`指定deb包的获取路径，与`sources.list`文件中写的路径一样，比如`http://mirrors.sohu.com/ubuntu/`

`SUITE`是一个名字，起初我以为可以随便写，经过测试发现必须与`/usr/share/debootstrap/scripts/`目录中的文件名对应，并且需要与MIRROR对应，下面有说明。在我的机器上这个目录有如下内容：
``` bash
$ ls /usr/share/debootstrap/scripts/
breezy            intrepid          potato            stable
dapper            jaunty            precise           testing
edgy              jessie            quantal           trusty
etch              karmic            raring            unstable
etch-m68k         lenny             sarge             warty
feisty            lucid             sarge.buildd      warty.buildd
gutsy             maverick          sarge.fakechroot  wheezy
hardy             natty             saucy             woody
hoary             oldstable         sid               woody.buildd
hoary.buildd      oneiric           squeeze
```
显然，`SUITE`这一项只能写Debian或Ubuntu的代号。否则错误提示为：
``` bash
$ sudo debootstrap abc target http://mirrors.sohu.com/ubuntu
E: No such script: /usr/share/debootstrap/scripts/abc
```
但只满足了本机的要求也不够，如果该suite在MIRROR对应源中不存在也无法执行，毕竟源中没有对应版本的deb包那什么也干不了。比如：
``` bash
$ sudo debootstrap karmic target http://mirrors.sohu.com/ubuntu
I: Retrieving Release 
E: Failed getting release file http://mirrors.sohu.com/ubuntu/dists/karmic/Release
```

例子，从[http://mirrors.sohu.com/ubuntu](http://mirrors.sohu.com/ubuntu)下载并构建Ubuntu saucy 13.10，存放在`./saucy`中：
``` bash
# Example:
sudo debootstrap saucy ./saucy http://mirrors.sohu.com/ubuntu
```

构建好系统就可以用tar打包了。需要注意的是路径问题，要保证tar包里面的/目录要对应上面的TARGET目录。简单的办法就是先进入TARGET目录，再执行tar命令：
``` bash
cd ./saucy
tar -cf ../ubuntu-saucy.tar .
cd ..
```

有了tar包就可以导入到docker中了，用以下命令导入：
``` bash
cat ./ubuntu-saucy.tar | sudo docker import - saucy
```
docker的import子命令接受两种形式的参数，一种是URL，另一种是标准输入。上面的命令中的短横杠`-`表示从标准输入导入，`saucy`是给这个image起的名字，类似`learn/tutorial`、`ubuntu`。因为要从标准输入导入，所以用cat命令将tar包内容输出到标准输出，再用管道连到docker的标准输入。

到这儿就完成了，可以试用一下刚导入的image
``` bash
$ sudo docker run -i -t saucy cat /etc/issue
WARNING: IPv4 forwarding is disabled.
Ubuntu 13.10 \n \l
```
最后一行就是image的输出。

最后再说一遍“F*K GFW”，搞这么多就为了替代`docker pull`一行命令。
