---
layout: post
title: "Install Emacs on OSX"
date: 2014-12-06 16:24
comments: true
categories: Emacs OSX
---

原来在 Linux 和 Windows 系统都是用 Emacs，换了 MacBook Pro 后就考虑如何在 OSX 上安装 Emacs。其实 OSX 默认就安装了 Emacs，只不过版本太老，还是22的，而且貌似没有图形界面，所以还是需要安装新版本的有图形界面的Emacs。OSX 上貌似有好几种 Emacs，因为我使用 Homebrew 管理软件包，所以选择了用 Homebrew 安装 Emacs。在这里记录一下安装配置时遇到的问题。

## 使用 Homebrew 安装 Emacs

Homebrew 有 Emacs 的 formula，那就像其他软件一样安装吧。

``` bash
 # Do not use this command, it does not work well!
brew install emacs
```

通过命令行的输出可以看到 Homebrew 默认选择 bottle 的方式安装 Emacs，bottle 方式就是直接下载已经编译好的软件包。一会儿安装完成，赶紧在命令行输入 `emacs --version`，确认是版本是新的，以为大功告成了。但是找了半天没找到怎么启动图形界面的 Emacs，遂上网查找问题。查找的结果是用上面的命令安装的 Emacs 只能在终端使用，根本就没有图形界面。想用图形界面，只能用编译的方式安装 Emacs。命令如下：

``` bash
brew install emacs --cocoa
```

这样就会编译 Cocoa 版本的 Emacs，也就是有图形界面的。在写这篇文章时，Homebrew 中的 Emacs 版本为 24.4，安装这个版本不需要再加 `--srgb` 选项，貌似已经是默认行为，不需要写在命令中了。网上查到的资料大多都有 `--srgb` 选项，是24.4版本之前需要的选项，现在已经不需要了。

安装完后仍然是不能直接启动图形界面的，需要做一个符号链接。

``` bash
ln -s /usr/local/Cellar/emacs/24.4/Emacs.app /Applications/Emacs.app
```

需要注意链接的路径，如果修改了 Homebrew 的安装路径，需要修改成对应的。到此，终于大功告成了。

## 修改 Emacs 修饰键

MacBook Pro 的键盘有一点让我非常不习惯，Control 键竟然只有一个！Command、Option、Shift 键都有两个，Control 键竟然只在左边有，右边的位置被方向键霸占了。OSX 的快捷键大多以 Command 开头，影响不大，但是 Emacs 可是大量使用 Control 键啊。原来的解决方案是使用外接键盘，现在不用外接键盘了，只能考虑其他方式了。用 Command 代替 Control 的功能是个不错的方法，经过上网查找，找到了几个 Emacs 的变量，可以实现该功能，只需添加下面的代码到配置文件即可。

``` cl
(if (eq system-type 'darwin)
    (progn
      ;(setq mac-control-modifier 'super)
      (setq mac-command-modifier 'control)))
```

`system-type` 变量表示当前系统类型，`darwin` 表示 OSX，`mac-command-modifier` 变量表示按下 Command 键输入哪个前缀。注释掉的那一行表示，用 Control 键实现原来 Command 键的功能，因为发生过误操作，所以注释不用了。

## 总结

这篇文章介绍了如何用 Homebrew 安装有图形界面的 Emacs，安装版本为24.4。原来安装过24.3版本的，有一个非常严重的BUG，一旦触发用 x-popup-dialog 打开的对话框，整个 Emacs 就没有响应了，只能通过系统的强制退出功能关掉程序。这其中就包括 `cmd + p` 这样的组合键，有时想按 `ctrl +p`，结果按成了 `cmd + p`，弹出和打印有关的对话框，然后整个 Emacs 就没有响应了，这也是更改 Command 键的功能的部分原因。现在重新安装了24.4，BUG没有了，使用起来更顺畅了。


## 参考

<http://emacsredux.com/blog/2014/01/11/a-peek-at-emacs-24-dot-4-srgb-colours-on-os-x/>

<http://stackoverflow.com/questions/1817257/how-to-determine-operating-system-in-elisp>
