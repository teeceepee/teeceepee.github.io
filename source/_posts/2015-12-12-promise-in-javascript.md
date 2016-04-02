---
layout: post
title: "Promise in JavaScript"
date: 2015-12-12 19:59
comments: true
categories:
---

## Promise

JavaScript 的并发编程模型是依赖回调函数的异步事件驱动模型。这种模型通过只使用单线程避免了多线程模型的诸多弊端，同时还可以充分利用单核 CPU，达到不错的性能表现。但是如果业务逻辑比较复杂，大量的异步回调函数会使代码非常难读难调试。Promise 是对回调函数的进一步抽象，力图在保留原有优势的情况下解决回调函数带来的问题。

## Promise in jQuery

jQuery 可能是使用最多的 JavaScript 库，jQuery 的 Ajax 部分显然是异步的、依赖回调函数的，自然也就有上面说的各种问题。jQuery 从 1.5 版本开始引入了 Deferred 对象，Deferred 对象和相关的 Promise 对象是 jQuery 对 Promise 的实现。Deferred 包含两类方法，一类是修改内部状态的，一类是注册回调函数的，Promise 隐藏了 Deffered 的修改内部状态的方法，只保留了注册回调函数的接口。jQuery 1.5 的 Ajax 全部使用 Deferred 对象重写了，所以很多人可能已经通过 Ajax 操作使用过 Deferred 对象了。下面的例子是 Ajax 的两种写法：

``` javascript
// callback
$.ajax({
  url: '/foo',
  success: function(data) {
    console.log(data);
  },
  error: function() {
    console.error('error');
  }
});
```
``` javascript
// Promise object
xhr = $.ajax({
  url: '/foo'
});

xhr.done(function(data) {
  console.log(data);
});

xhr.fail(function() {
  console.error('error');
});

// chainable Promise object
$.ajax({
  url: '/foo'
}).done(function(data) {
  console.log(data);
}).fail(function() {
  console.error('error');
});

```

`jQuery.ajax` 的返回值是一个 Promise 对象，`done` 和 `fail` 都是 Promise 对象的方法，而且返回值还是 Promise 对象自身。两种写法看上去好像差不多，`done` 对应 `success`，`fail` 对应 `error`。但是 Promise 对象的写法可以把把回调单独拿出来写，同时又因为可以使用链式写法，可读性上貌似好了一些。但是使用 Promise 的写法还有其他好处。

首先，Promise 对象的 `done` 和 `fail` 方法可以调用多次，也就是可以自由关联多个回调函数，传统写法如果想关联多个回调，只能把多个函数用一个函数包裹后才可以。

其次，我认为是最重要的，Promise 对象可以解耦事件注册和事件触发的顺序。在传统的 JavaScript 事件编程中，事件注册一定要在事件触发之前，否则可能会漏掉事件。看下面的例子：

``` javascript
// correct
var httpRequest = new XMLHttpRequest();

httpRequest.onreadystatechange = function() {
  console.log(arguments);
};

httpRequest.open('GET', '/foo/bar', true);
httpRequest.send(null);


// wrong!
var httpRequest = new XMLHttpRequest();

httpRequest.open('GET', '/foo/bar', true);
httpRequest.send(null);

httpRequest.onreadystatechange = function() {
  console.log(arguments);
};
```

第二段代码是错误的，在 `send` 之后再使用 `onreadystatechange` 注册回调函数有可能会漏掉一些事件。jQuery 的 `ajax` 函数通过把回调限制为参数的一部分来避免这个问题。


Promise 则不一定需要保证这样的顺序，因为 Promise 对象内部保存了事件的状态，所以即使注册回调函数发生在事件触发之后，注册的回调函数仍然可以执行。下面的例子演示了这个过程。

``` javascript
var slow_func = function() {
  for (var i = 0; i < 1000000; i++) {
    1 + 1;
  }
};

xhr = $.ajax({
  url: '/foo'
});

slow_func();

// xhr completed here!

slow_func();


xhr.done(function(data) {
  // still executed!
  console.log(data);
});
```


## TBC Promise in ES2015
