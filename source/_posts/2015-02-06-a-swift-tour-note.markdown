---
layout: post
title: "A Swift Tour 笔记"
date: 2015-02-06 23:06
comments: true
categories: swift
---

最近看完了 The Swift Programming Language 中的 A Swift Tour 一节，对 Swift 有了一点初步的了解，在此记录一下学习过程中遇到的一些问题。

## 可变参数列表

Swift 的函数可以接受多个参数，把它们收集到一个数组中。这本来应该是一个不错的语言特性，但是在写对应的 experiment 时就发现问题了，Swift 只能把多个参数收集到数组中，却不能将一个数组打散成多个参数。Ruby 中可以用 `sumOf(*numbers)` 这样的方式实现将数据打散成多个参数，Swift 目前没有这样的功能，所以无法用下面的代码实现 `average` 函数。如果实在想用 `sumOf` 函数来实现 `average`，那只能将 `sumOf` 的参数改为数组类型的。只能期待Swift能在以后实现这样的功能了，目前来说尽量少用这样的蹩脚的语言特性吧。

```
func sumOf(numbers: Int...) -> Int {
    var sum = 0
    for number in numbers {
        sum += number
    }
    return sum
}
sumOf()
sumOf(42, 597, 12)

func average(numbers: Int...) -> Double {
    let count = numbers.count
    // The code below do not work!
    // let avg = sumOf(*numbers) / count
    return avg
}
```

## 枚举类型遍历

Swift 可以使用 `enum` 定义枚举类型，但是目前貌似没有非常方便的方法返回所有的枚举值。希望以后能添加这样的方法吧。


## 命名空间
Swift 有命名空间的功能，不过很少有地方提到。在解决下面与命名空间有关的一个问题时找到了 Lattner 大神的 Twitter，提到了显式使用命名空间。链接：[Chris Lattner suggests](https://twitter.com/clattner_llvm/status/474772713739792384)

与命名空间有关的问题代码如下：

```
extension Int {

    func restrictToRange(#minValue: Int, maxValue: Int) -> Int {
        var ret: Int

        // 'Int' does not have a member named 'min'
        //ret  = min(max(self, minValue), maxValue)

        // use explicit namespace
        ret = Swift.min(Swift.max(self, minValue), maxValue)
        return ret
    }

}
```

我想用 `extension` 向 `Int` 中添加一个方法，在其中用到了 `min` 和 `max` 两个全局函数，但是直接使用会有 `'Int' does not have a member named 'min'` 这样的错误提示，在函数名前加上显式的命名空间 `Swift` 后问题解决，但是引起该错误的原因没有找到，可能与作用域和符号查找有关吧，希望在以后能找出该问题的具体原因。
