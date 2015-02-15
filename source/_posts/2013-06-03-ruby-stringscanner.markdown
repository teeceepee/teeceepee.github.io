---
layout: post
title: "Ruby StringScanner"
date: 2013-06-03 20:09
comments: true
categories: Ruby
---

想写一个简单的解释器，源代码用字符串表示。本来想自己写一个 scanner，后来发现 Ruby 库中有一个 StringScanner 类，但不是用 Ruby 写的，不能做源码浏览了。在这里记录一下 StringScanner 的用法。

# \<b\> strscan \</b\>
要用 StringScanner 类需要 `require 'strscan'` ，以前老是忘了这个，这次一定要记住。

按照 StringScanner [官方文档](http://ruby-doc.org/stdlib-1.9.3/libdoc/strscan/rdoc/StringScanner.html)的说法，它的用处是对字符串进行词法扫描操作。我用的最多的是 `scan` 方法，它与 `String#scan` 最大的不同是内部维护一个 position 变量，使得每次扫描都从position开始而不是开头。

# 常用方法
`StringScanner.new` 构造一个 StringScanner 对象，参数是待扫描的字符串。
``` ruby StringScanner.new
s = StringScanner.new '(+ a 2)'
```
`StringScanner#eos?` 判断是否扫描完毕。

`StringScanner#scan` 参数 pattern 为一个正则表达式，尝试从当前位置与 pattern 匹配，如果成功那么 scanner 前进到下一个待扫描位置并返回匹配的字符串，前进的数目与匹配字符串的长度相等。匹配失败的话返回 `nil`。
``` ruby StringScanner#scan
s = StringScanner.new('test string')
p s.scan(/\w+/)   # -> "test"
p s.scan(/\w+/)   # -> nil
p s.scan(/\s+/)   # -> " "
p s.scan(/\w+/)   # -> "string"
p s.scan(/./)     # -> nil
```

`StringScanner#peek` 接受一个数字类型的参数 len，返回 `string[pos, len]`，也就是即将被扫描的字符串，不推进扫描指针。

`StringScanner#check` 与 `scan` 类似，但是不推进扫描指针。
