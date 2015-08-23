---
layout: post
title: "Bootstrap 输入框效果实现"
date: 2015-07-27 21:42
comments: true
---

当我第一次用 [Bootstrap](http://getbootstrap.com/) 时，印象最深的是加在表单的文本输入框上的效果。浅灰的圆角边框，似有似无的边框阴影，以及获得焦点时的蓝色边框效果。但是一直以来我并不知道这些效果是如何实现的，最近终于下决心研究一下 Bootstrap，看看它是如何写成的。首先想到的便是输入框的效果。

我是用 `bootstrap-sass` 这个 Ruby 的 gem 研究 Bootstrap 源码的。与输入框相关的代码在 `assets/stylesheets/bootstrap/_forms.scss`。

查看网页的源码发现，实现这些输入框效果的是 `.form-control` 这个 class。`.form-control` 的源码如下：

``` scss
.form-control {
  display: block;
  width: 100%;
  height: $input-height-base; // Make inputs at least the height of their button counterpart (base line-height + padding + border)
  padding: $padding-base-vertical $padding-base-horizontal;
  font-size: $font-size-base;
  line-height: $line-height-base;
  color: $input-color;
  background-color: $input-bg;
  background-image: none; // Reset unusual Firefox-on-Android default style; see https://github.com/necolas/normalize.css/issues/214
  border: 1px solid $input-border;
  border-radius: $input-border-radius; // Note: This has no effect on <select>s in some browsers, due to the limited stylability of <select>s in CSS.
  @include box-shadow(inset 0 1px 1px rgba(0,0,0,.075));
  @include transition(border-color ease-in-out .15s, box-shadow ease-in-out .15s);

  // Customize the `:focus` state to imitate native WebKit styles.
  @include form-control-focus;

  // Placeholder
  @include placeholder;

  // Disabled and read-only inputs
  //
  // HTML5 says that controls under a fieldset > legend:first-child won't be
  // disabled if the fieldset is disabled. Due to implementation difficulty, we
  // don't honor that edge case; we style them as disabled anyway.
  &[disabled],
  &[readonly],
  fieldset[disabled] & {
    background-color: $input-bg-disabled;
    opacity: 1; // iOS fix for unreadable disabled content; see https://github.com/twbs/bootstrap/issues/11655
  }

  &[disabled],
  fieldset[disabled] & {
    cursor: $cursor-disabled;
  }

  // [converter] extracted textarea& to textarea.form-control
}
```

圆角用 `border-radius` 属性实现，边框颜色是在 `border` 属性中设置的。最重要的是下面两行：

``` scss
@include box-shadow(inset 0 1px 1px rgba(0,0,0,.075));
@include transition(border-color ease-in-out .15s, box-shadow ease-in-out .15s);

```

用到了 Sass 的 Mixin 功能，实际的实现就是设置 `box-shadow` 和 `transition` 属性。也就是说，输入框的效果主要是靠这两个属性实现的。

## `box-shadow`

`box-shadow` 用来设置盒阴影。

## `transition`

`transition` 用来设置过渡效果。

TODO uncompleted


