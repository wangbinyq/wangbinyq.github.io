---
title: "C# 中如何遍历 Enum 枚举值"
date: 2021-09-22T15:47:20+08:00
description: "在 C# 中遍历 Enum 枚举值"
tags: ["C#"]
categories: ["C# 语言"]
---

假如我们有如下 `Shape` 枚举类型, 我们要遍历其中所有的枚举值
```c#
public enum Shape
{
    Circle,
    Triangle,
    Rect
}
```

可以通过 [`GetValues`](https://docs.microsoft.com/en-us/dotnet/api/system.enum.getvalues) 方法实现:
<!-- more -->

```c#
var shapes = Enum.GetValues(typeof(Shape));

foreach (var shape in shapes)
{
    DoSomething(shape);
}
```

另外我们还可以做一点封装

```c#
public static class EnumUtil {
    public static IEnumerable<T> GetValues<T>() {
        return Enum.GetValues(typeof(T)).Cast<T>();
    }
}

var shapes = EnumUtil.GetValues<Shape>();
```
