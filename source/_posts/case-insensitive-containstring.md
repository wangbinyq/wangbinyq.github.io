---
title: "C# 如何在字符串中查找大小写不敏感子字符串"
date: 2021-09-22T13:24:20+08:00
description: "在 C# 中如何查找大小写不敏感的子字符串"
tags: ["C#"]
categories: ["C# 语言"]
---

在 C# 中如何查找大小写不敏感的子字符串, 比如我们想要下面的例子返回 `true`:

```c#
string title = "ASTRINGTOTEST";
title.Contains("string") == true;
```
<!-- more -->

## 对于英语

一种解决方案是将两个字符串都转换成大写或小写, 然后再进行比较:

```c#
title.ToUpper().Contains("string".ToUpper()) == true;
```

另一种方案是使用 [`IndexOf` 方法](https://msdn.microsoft.com/en-us/library/ms224425(v=vs.110).aspx), 该方法有一个重载方法可以忽略字符串的大小写, 即 [`StringComparison.OrdinalIgnoreCase`](https://msdn.microsoft.com/en-us/library/system.stringcomparer.ordinalignorecase(v=vs.110).aspx) 作为第二个参数:

```c#
string title = "STRING";
bool contains = title.IndexOf("string", StringComparison.OrdinalIgnoreCase) >= 0;
```

我们还可以为 `String` 定义一个扩展方法, 重载 `Contains`

```c#
public static class StringExtensions
{
    public static bool Contains(this string source, string toCheck, StringComparison comp)
    {
        return source?.IndexOf(toCheck, comp) >= 0;
    }
}
```

使用方法:

```c#
string title = "STRING";
bool contains = title.Contains("string", StringComparison.OrdinalIgnoreCase);
```

## 非英语语言解决方案
不同语言对于大小写的定义是不一样的, 比如在英语中 `I` 和 `i` 是第九个字符的大小写形式, 然而在土耳其语中, 这两个字符分别是[第 19 和 20 个字符](http://en.wikipedia.org/wiki/Dotted_and_dotless_I), 土耳其语中 `i` 的大写形式为 `İ`. 因此 `tin` 和 `TIN` 在英语中是同一个单词, 在土耳其语中前一个词表示 `精神`, 第二个是一个拟声词.

因此上面提供的方法只适用于英语语言, 为了兼容各种不同语言, C# 中提供了 [`CultureInfo`](http://msdn.microsoft.com/en-gb/library/system.globalization.cultureinfo(v=vs.110).aspx) API, 我们可以通过下面方法进行判断:

```c#
culture.CompareInfo.IndexOf(paragraph, word, CompareOptions.IgnoreCase) >= 0
```

其中 `culture` 是一个 `CultureInfo` 对象.