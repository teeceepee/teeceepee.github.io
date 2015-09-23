---
layout: post
title: "设置 MiniMagick 查找命令的路径"
date: 2015-09-23 21:21
comments: true
categories: MiniMagick
---

MiniMagick 是 Ruby 中常用的图片处理库，它是基于 ImageMagick 这个跨平台图片处理库实现的。比较特别的是，MiniMagick 是通过调用 ImageMagick 提供的一系列命令来实现图片处理功能的，也就是只要操作系统中有对应的 ImageMagick 命令，比如 `identify`，`convert` ，MiniMagick 就能正常工作。

只要涉及到命令调用，就一定会有命令查找路径的问题。MiniMagick 默认只使用命令名来调用命令，也就是从环境变量的 PATH 中查找。这一般没有问题，但如果 ImageMagick 的可执行程序没有安装在默认的 PATH 中，那么可以通过 `MiniMagick.cli_path=` 这个方法来指定命令查找的路径。例如在 Rails 中，可以创建一个 `config/initializers/mini_magick.rb` 初始化文件，写入如下内容：

``` ruby
# config/initializers/mini_magick.rb

MiniMagick.configure do |config|
  # 如果不能正常调用 ImageMagick 的命令，则使用指定的路径。
  begin
    # 返回命令 `identify -version` 输出，如果查找不到命令则会抛出异常。
    MiniMagick.cli_version
  rescue
    config.cli_path= '/root/.linuxbrew/bin'
  end
end

```

上面的初始化文件首先尝试调用 ImageMagick 的命令，如果调用失败则使用指定的路径查找命令。
