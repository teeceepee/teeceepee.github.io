---
layout: post
title: "metaprogramming ruby notes 1"
date: 2013-07-19 13:55
comments: true
categories: metaprogramming-ruby
---

这些天在看[《ruby元编程》](http://book.douban.com/)。其中有一个关于block的小测验，要模仿C#中的using关键字。

在Ruby中，using可以通过内核方法写一个。内核方法即添加到Kernel模块中的方法，因此可以在程序的任何地方使用，有点像一个关键字。书中给的答案可以通过测试，但是仔细研究发现书中实现的using貌似有些问题，而且是大问题。实现using的目的在于在block中使用资源，使用完毕后自动将资源释放。因此using后面的block一定有一个参数代表资源，using的实现也应该将资源传递给block，所以正确的实现应该如下。

``` ruby using.rb
module Kernel
  def using(resource)
    yield resource
  ensure
    resource.dispose
  end
end
```

书中的实现在`yield`后没有resource，无法将资源传递给block。不能在block中使用资源，写这玩意儿有啥用？我看的版本是中文版，不清楚原版是否也有这个错误。另外，书中使用的是`begin ... ensure ... end`，在用`def`定义方法时`begin`是可以省略的。

之所以会出现这个问题，我认为是单元测试的用例写的不够严格。下面是与书中原测试等价的测试用例。

``` ruby using_test.rb
require 'test/unit'
require './using.rb'

class Resource
  def dispose
    @dispose = true
  end
  def disposed?
    @dispose
  end
end

class TestUsing < Test::Unit::TestCase
  def setup
    @r = Resource.new
  end

  def test_disposes_of_resources
    using(@r) { }
    assert @r.disposed?
  end

  def test_disposes_of_resources_in_case_of_exception
    assert_raises(Exception) do
      using(@r) do
        raise Exception
      end
    end
  end
end
```

可以看到，两个测试在block中都没有使用资源。测试的不严谨导致了这个using的大bug。至少应该再添加一个测试，比如：

``` ruby using_test_add.rb
def test_with_block_argument
  using(@r) do |r|
    r.disposed?
  end
end
```

添加这个测试后，书中原代码会无法通过测试，因此可以发现这一bug。

# 后记

刚写完上面的，正在想怎么总结结尾的时候，一个想法忽然在脑海中闪现出来：我是否真的理解了书中设计的using的用法。按照我自己的想法，using的参数是一个实现了dispose方法的资源，后面的block是单形参的。这样using将资源传递给block，在block中使用形参代表资源。后来想了想，using后面的block其实也可以访问到外面的资源，没有必要设置一个参数来传递资源。比如：

``` ruby using_test.rb
def test_using
  using(@r) do
    # something using @r
    # ...
    @r.disposed?
  end
  assert @r.disposed?
end
```

改完测试，运行通过，那这篇博文怎么办...
删还是不删，这是一个问题...

# 后记的后记

终于想出一个不用删掉这篇文章的理由，那就是通过参数传递资源的方式要灵活那么一点。书上的写法因为没有使用参数，所以using后的block必须定义在要立即使用资源的地方，当然C#中的using大约也是这样使用的。但如果资源是通过block的参数传递的，那么这个block就可以通过 `Proc.new` 或者 `lambda` 定义在任何地方。比如：

``` ruby using_define_anywhere.rb
def test_define_anywhere
  p = Proc.new do |r|
    r.disposed?
  end
  using(@r, &p)
  assert @r.disposed?
end
```
