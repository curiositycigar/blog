---
title: css position
date: 2018-06-19 13:25:03
tags:
---
# CSS position取值

## 常用取值
```
  position: static
  position: relative
  position: absolute
  position: fixed
```
> `static`：默认定位方式，正常的文档流

> `relative`：相对于自身在文档流中的位置定位

> `absolute`：相对于第一个非static定位的父元素定位

> `fixed`：绝对定位，相对于浏览器窗口定位

## 其他取值
```
  position: inherit
  position: initial
  position: unset
```
> `inherit`：从父元素继承

> `initial`：设置为默认值

> `unset`：如果属性为继承属性，等同于inherit。否则等同于initial

## 还有一个
```
  position: sticky
```
粘性布局

[MDN介绍](https://developer.mozilla.org/zh-CN/docs/Web/CSS/position)