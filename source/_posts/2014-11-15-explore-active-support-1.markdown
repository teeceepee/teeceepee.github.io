---
layout: post
title: "explore active support 1"
date: 2014-11-15 20:57
comments: true
categories: active_support
---

Ruby on Rails框架的Active Support组件提供了很多方便使用的方法，有些方法是通过Monkey Patch的方式添加到Ruby原有的类中的。

## Object#blank?
只截取了主要部分的代码。看过源码后发现，#blank? 不仅添加到Object类，还添加到了NilClass，FalseClass，TrueClass，String，Numeric，Array，Hash等类中。
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

判断一个对象是否是空白的。空白的定义是如果一个对象是false，empty或空白字符，那么就是空白的。比较的特殊的是对String类的处理，只有零个或多个空格字符的字符串都认为是空白的。根据注释，制表符换行符等都算作空白字符，还支持Unicode的空白字符。

``` ruby
''.blank?       # => true
'   '.blank?    # => true
"\t\n\r".blank? # => true
' blah '.blank? # => false

# Unicode whitespace is supported:
"\u00a0".blank? # => true
```

所有的数字都不是空白的，因为向Numeric类添加的blank?方法永远返回false。

## Object#present?
`Object#present?`非常常用，实现非常简单，所有不是空白的对象都返回true。

``` ruby
def present?
  !blank?
end
```

## ActiveSupport::HashWithIndifferentAccess

随着Ruby版本的发展，String和Symbol对象的差别越来越小，在Ruby和Rails中很多时候都是可以混用的，但是有一个地方是不能混用的，那就是Hash的键。`:foo.hash != 'foo'.hash`，Active Support中得HashWithIndifferentAccess类就是解决这个问题的。

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

好了，再也不用担心搞错了，不过这名字也太长了。Active Support还添加了`Hash#with_different_access`方法，可以通过已有的Hash对象转化。

``` ruby
rgb = { black: '#000000', white: '#FFFFFF' }.with_indifferent_access
```
算是短一点了。
