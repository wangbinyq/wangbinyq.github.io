---
title: "C# 获取运行时内存占用"
date: 2021-09-28T14:22:18+08:00
description: "C# 获取运行时内存占用"
tags: ["C#", "内存占用"]
categories: ["C#"]
---

## 获取 GC 分配的内存

这里输出的是 `GC` 管理的堆上内存.

```c#
Console.WriteLine("Total Memory: {0}", GC.GetTotalMemory(false));
```
<!-- more -->

## 获取进程分配的所有内存

这里输出的是进程从操作系统申请的所有内存, 包括堆, 栈, 静态变量, 本地库申请的内存等, .

```c#
Process proc = Process.GetCurrentProcess();
System.Console.WriteLine($"Current Memory Usage: {proc.PrivateMemorySize64}");
```
