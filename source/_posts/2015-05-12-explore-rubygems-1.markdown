---
layout: post
title: "Explore RubyGems 1"
date: 2015-05-12 21:58
comments: true
categories: RubyGems
---

最近对 Ruby 的常用工具的使用以及实现原理产生了兴趣，因为前一阵子发布了一个试验性质的 gem，所以对 gem 相关的东西学习了一下。寻根溯源，找到了一个叫做 RubyGems 的概念。

# Basics

RubyGems 是什么？这几个字母的组合会在许多地方出现，Gemfile 的第一行一般是 `source 'https://rubygems.org'` ，对应有一个域名为 rubygems.org 的网站，在一些很多年前的 Ruby 代码中会有 `require 'rubygems'` 。基本可以确定 rubygems.org 是 RubyGems 的官网，那就从官网入手吧。下面是[官网教程][guides]的说法：

> The RubyGems software allows you to easily download, install, and use ruby software packages on your system. The software package is called a “gem” and contains a package Ruby application or library.

所以 RubyGems 是一套软件，这套软件的目标是让用户方便的下载、安装和使用 Ruby 的软件包。同时从这段话中可以看到，在 RubyGems 的范畴内，Ruby 的软件包叫做 gem。当然，有一些 Ruby 使用经验的人都知道还有一个叫 `gem` 的命令，这个命令当然也是 RubyGems 提供的。`gem` 命令是 RubyGems 提供的使用接口。RubyGems 需要一个中心服务器存储管理所有的 gem，默认使用的服务器地址就是 https://rubygems.org 。

## 被重写的 Kernel#require 方法

`Kernel#require` 方法是用来干什么的？用来加载 Ruby 代码，它会从全局变量 `$LOAD_PATH` 中查找对应名字的文件并加载。然而上面的描述在 RubyGems 存在的情况下并不准确。RubyGems 重写了 `Kernel#require` 方法，扩展了它的功能。

``` ruby kernel_require.rb https://github.com/ruby/ruby/blob/trunk/lib/rubygems/core_ext/kernel_require.rb#L38

  ##
  # When RubyGems is required, Kernel#require is replaced with our own which
  # is capable of loading gems on demand.
  #
  # When you call <tt>require 'x'</tt>, this is what happens:
  # * If the file can be loaded from the existing Ruby loadpath, it
  #   is.
  # * Otherwise, installed gems are searched for a file that matches.
  #   If it's found in gem 'y', that gem is activated (added to the
  #   loadpath).
  #
  # The normal <tt>require</tt> functionality of returning false if
  # that file has already been loaded is preserved.

  def require path
    # ...
  end
```

从注释中可以看到，重写后的 `Kernel#require` 引入文件时会分两步：

1. 如果文件能在 $LOAD_PATH 中找到，那就引入它。
2. 否则，在所有已安装的 gem 中查找，如果找到，那么把对应的 gem 激活，也就是把该 gem 的 `lib` 目录加入 $LOAD_PATH。

需要注意的是，`require` 的参数永远都对应一个文件名（可以省略 `.rb` 扩展名），而不是 gem 的名字。比如 awesome_print 这个 gem 的 `lib` 目录是这样的：

``` bash

% tree -L 1 lib                                                              ➜
lib
├── ap.rb
├── awesome_print
└── awesome_print.rb

1 directory, 2 files

```

那么可以用下面的方式引入：

``` ruby
require 'ap'
require 'ap.rb'
require 'awesome_print'
require 'awesome_print.rb'
```

个人猜测，如果在其他 gem 中有同名的文件，可能会导致错误的引入。为了避免这样的问题，RubyGems 给出了一些命名和目录结构的建议。

## 概念

- RubyGems，软件包管理软件
- gem，RubyGems 中的软件包
- `gem`，RubyGems 提供的命令行接口
- https://rubygems.org ，RubyGems 的一个中心服务器


## 参考

[guides][]

[requiring_code][]

[guides]: http://guides.rubygems.org
[requiring_code]: http://guides.rubygems.org/rubygems-basics/#requiring-code
