---
title: "C# 语言 String 和 string 的区别"
date: 2021-09-22T13:24:20+08:00
description: "C# 中 String 和 string 类型的区别和适合的使用方式"
tags: ["C#"]
categories: ["C# 语言"]
---


在 C# 语言中 `string` 是 `System.String` 的别名. 因此从技术上来说, 这两者没有任何区别, 就像 `int` 和 `System.Int32`.
<!-- more -->

通常我们推荐使用 `string` 类型声明一个对象.

```c#
string place = "world";
```

同时, 推荐使用 `String` 来引用字符串类型中的方法, 属性等, 比如:

```c#
string greet = String.Format("Hello {0}!", place);
```