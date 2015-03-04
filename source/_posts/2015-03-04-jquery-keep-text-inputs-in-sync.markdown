---
layout: post
title: "用 jQuery 同步输入框内容"
date: 2015-03-04 22:08
comments: true
categories:
---

前几天在项目中发现有不能正确同步输入框内容的页面，经过上网查找后找到了一种比较好的办法。

要想同步输入框内容，就要监听各种可能使输入框内容发生改变的事件，第一个想到的就是监听键盘输入的事件，与键盘输入相关的事件有 `keydown`, `keypress`, `keyup` 等，`keydown` 和 `keypress` 发生在输入框内容改变之前，应该使用 `keyup` 事件。除了键盘输入外，通过粘贴的方式也会使输入框内容发生变化。 使用键盘快捷键粘贴的方式也会触发 `keyup` 事件，不过用鼠标右键粘贴的方式不触发 `keyup` 事件，鼠标右键粘贴会触发 `paste` 事件，但是该事件发生在输入框内容改变之前，不能正确同步内容。

针对鼠标右键粘贴等方式没有找到太好的办法，只想出了一个折衷的办法：当输入框失去焦点时同步内容，这样虽然不能保持实时同步，但能保证最终同步。输入框失去焦点的事件是 `blur` 。

``` html
<div>
  <label>Input:</label>
  <input id="input-field" type="text">
  <br>
  <label>Output:</label>
  <span id="output-field"></span>
</div>
<script>
  $('#input-field').on('keyup blur', function() {
    console.log('asdf');
    $('#output-field').text($(this).val());
  });
</script>
```

<div>
  <label>Input:</label>
  <input id="input-field" type="text">
  <br>
  <label>Output:</label>
  <span id="output-field"></span>
</div>
<script>
  $('#input-field').on('keyup blur', function() {
    console.log('asdf');
    $('#output-field').text($(this).val());
  });
</script>
