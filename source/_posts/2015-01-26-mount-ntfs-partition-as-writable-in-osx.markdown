---
layout: post
title: "OSX 中挂载 NTFS 分区"
date: 2015-01-26 22:29
comments: true
categories: OSX
---

我的移动硬盘是在用 Mac 之前买的，格式化为 NTFS 格式，可以在 Windows 和 Linux 中正常使用。换用 Mac 后，一开始只是从移动硬盘中读文件，没发觉有什么问题，一度以为网上说的 OSX 不支持 NTFS 的说法是针对老版本。直到又一次想要向移动硬盘中存文件，发现连新建目录都不行，这才了解到 OSX 默认把 NTFS 分区挂载为只读。

原来 OSX 从 Snow Leopard 开始就内置了对 NTFS 分区的读写支持，但是默认将 NTFS 分区挂载为只读分区，具体原因不详。既然是内置读写支持，那就有办法将 NTFS 分区挂载为可读可写的分区。

实现这个功能的命令是 `/sbin/mount_ntfs`，系统在实现移动硬盘自动挂载功能时也是间接使用该命令，所以网上有修改该文件来实现自动挂载为可读可写分区的方法。个人觉得修改系统原文件不是一个很好的方法，没有采用。不过也学到了使用该命令的方法。

``` bash
mount_ntfs -o rw,nobrowse DEVICE MOUNT_POINT
```

其中 `DEVICE` 是硬盘分区对应的设备文件， 比如 `/dev/disk2s1`，`MOUNT_POINT` 是挂载点，比如 `/Volumes/data`。通过 `-o rw,nobrowse` 这样的参数，可以将 NTFS 分区挂载为可读可写分区，其中的 `rw` 顾名思义，`nobrowse` 表示不在桌面和 Finder 中显示。不在桌面和 Finder 中显示很不方便，于是尝试只是用 `-o rw` 这样的参数，挂载的分区出现在了桌面和 Finder中，但是依然是只读分区，尝试失败。可能是系统做了限制，挂载为可读可写分区时不允许在桌面和 Finder 中显示。目前我的做法是将 `/Volumes` 目录放到 Finder 的个人收藏中，然后手动挂载分区到这个目录下的某个子目录。
