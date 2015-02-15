---
layout: post
title: "Explore ActiveSupport 1"
date: 2014-11-15 20:57
comments: true
categories: ActiveSupport
---

Ruby on Rails 框架的 Active Support 组件提供了很多方便使用的方法，有些方法是通过 Monkey Patch 的方式添加到 Ruby 原有的类中的。

## Object#blank?
只截取了主要部分的代码。看过源码后发现，#blank? 不仅添加到 Object 类，还添加到了 NilClass，FalseClass，TrueClass，String，Numeric，Array，Hash 等类中。
``` ruby
# File activesupport/lib/active_support/core_ext/object/blank.rb
class Object
  def blank?
    respond_to?(:empty?) ? !!empty? : !self
  end
end

class String
  BLANK_RE = /\A[[:space:]]*\z/

  def blank?
    BLANK_RE === self
  end
end

```

判断一个对象是否是空白的。空白的定义是如果一个对象是 false，empty 或空白字符，那么就是空白的。比较的特殊的是对 String 类的处理，只有零个或多个空格字符的字符串都认为是空白的。根据注释，制表符换行符等都算作空白字符，还支持 Unicode 的空白字符。

``` ruby
''.blank?       # => true
'   '.blank?    # => true
"\t\n\r".blank? # => true
' blah '.blank? # => false

# Unicode whitespace is supported:
"\u00a0".blank? # => true
```

所有的数字都不是空白的，因为向 Numeric 类添加的 blank? 方法永远返回 false。

## Object#present?
`Object#present?`非常常用，实现非常简单，所有不是空白的对象都返回 true。

``` ruby
def present?
  !blank?
end
```

## ActiveSupport::HashWithIndifferentAccess

随着 Ruby 版本的发展，String 和 Symbol 对象的差别越来越小，在 Ruby 和 Rails 中很多时候都是可以混用的，但是有一个地方是不能混用的，那就是 Hash 的键。`:foo.hash != 'foo'.hash`，Active Support 中的 HashWithIndifferentAccess 类就是解决这个问题的。

``` ruby
# File activesupport/lib/active_support/hash_with_indifferent_access.rb
rgb = ActiveSupport::HashWithIndifferentAccess.new

rgb[:black] = '#000000'
rgb[:black]  # => '#000000'
rgb['black'] # => '#000000'

rgb['white'] = '#FFFFFF'
rgb[:white]  # => '#FFFFFF'
rgb['white'] # => '#FFFFFF'
```

好了，再也不用担心搞错了，不过这名字也太长了。Active Support还添加了`Hash#with_different_access`方法，可以通过已有的 Hash 对象转化。

``` ruby
rgb = { black: '#000000', white: '#FFFFFF' }.with_indifferent_access
```
算是短一点了。
