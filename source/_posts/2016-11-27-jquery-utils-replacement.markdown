---
layout: post
title: "jQuery Utils Replacement"
date: 2016-11-27 16:25
comments: true
categories: JavaScript
---

曾几何时，编写跨浏览器的 JavaScript 代码是噩梦一般的存在。在那个浏览器差异巨大、标准提供的功能有限的时代，jQuery 出现后，依靠跨浏览器、功能丰富、接口简洁优雅的特性，迅速成为了前端开发的事实标准。直到现在，jQuery 可能依然是最流行的 JavaScript 库。但是到了现在，现代浏览器不再各自为战，开始遵循相同的标准，而标准本身也在前进，不断加入新的接口。很多逻辑已经可以使用原生的接口实现，不再依赖第三方库。

在我看来，jQuery 的 DOM 操作接口依旧比原生接口好用，还是很难丢弃掉。但很多实用函数已经可以使用原生接口替代。下面整理了几个 jQuery 常用的实用函数，以及对应的原生接口。

## $.each() 和 $.fn.each()

分别使用 `Array.prototype.forEach()` 和 `NodeList.forEach()` 替代。

曾经 JavaScript 只能不停地使用循环来实现遍历，ECMAScript 5 新增了数组等类型的 `forEach` 方法，因此已经可以不使用 jQuery 的遍历函数了。而且现在最新版本的浏览器也开始支持 NodeList 实例的 `forEach` 方法。需要注意的是参数顺序是不一样的，`forEach` 的参数顺序更方便一些，因为很多时候并不需要下标。

``` javascript
$.each(array, function (i, item) {

}

array.forEach(function (item, i) {

}

$(selector).each(function (i, el) {

}

document.querySelectorAll(selector).forEach(function (item, i) {

}
```

[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)

[https://developer.mozilla.org/en-US/docs/Web/API/NodeList/forEach](https://developer.mozilla.org/en-US/docs/Web/API/NodeList/forEach)

## $.extend()

使用 `Object.assign()` 替代。

很多库和框架都有自己的对象合并方法，方法名称一般是 `extend` 。ECMAScript 2015 新增了原生的对象合并方法，只不过名字叫 `assgin`。

``` javascript
$.extend({}, objA, objB)

Object.assign({}, objA, objB)
```

[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)

[https://github.com/nzakas/understandinges6/issues/99](https://github.com/nzakas/understandinges6/issues/99)

## $.map()

使用 `Array.prototype.map()` 替代。

同样需要注意参数顺序。

``` javascript
$.map(array, function(value, index) {

}

array.map(function (value, index) {

}
```

[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)

## $.proxy()

使用 `Function.prototype.bind()` 替代。

函数的 `bind` 方法是 EMCAScript 5 中新增的非常重要的一个方法，有了它可以尽量避免 `that`, `self` 之类的愚蠢变量名。需要注意的是 `bind` 每次调用都返回一个新的函数，而 `$.proxy` 在内部做了缓存处理，多次调用返回的是同一个函数。

``` javascript
$.proxy(fn, context)

fn.bind(context)
```

[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)

[http://stackoverflow.com/questions/18848343/underscore-bind-vs-jquery-proxy-vs-native-bind#answer-22860661](http://stackoverflow.com/questions/18848343/underscore-bind-vs-jquery-proxy-vs-native-bind#answer-22860661)

## $.trim()

使用 `String.prototype.trim()` 替代。

MDN 中给出了一个 polyfill 的方法，jQuery 的实现几乎与其一样，`$.trim()` 不是字符串类型的方法，所以对 `null` 做了特殊处理，`$.trim(null) === ‘’`。

``` javascript
$.trim(string)

string.trim()
```

[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/Trim](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/Trim)

## 总结

本文是阅读 [You Might Not Need jQuery](http://youmightnotneedjquery.com/) 后写的，页面上有一个选择支持 IE 最老版本的按钮，通过切换不同的开关，可以看到越新的浏览器版本，实现相同功能的代码越简洁统一。相信在浏览器厂商和标准的共同努力下，可以让我们更好得使用原生接口编写前端代码。
