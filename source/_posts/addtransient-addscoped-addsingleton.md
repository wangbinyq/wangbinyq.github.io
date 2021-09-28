---
title: "ASP.NET 中 AddTransient AddScoped AddSingleton 的区别"
date: 2021-09-24T18:40:15+08:00
description: "ASP.NET 中 AddTransient AddScoped AddSingleton 的区别"
tags:
  - ASP.NET
  - 依赖注入
  - 生命周期
categories:
  - ASP.NET
---

在 `.NET` 中依赖注入有三种生命周期:

- `Singleton` 在程序运行时只创建一个对象实例. 它会在第一次使用该对象时创建, 之后的所有请求获取的都是同一个对象. 由于单例是全局存在的, 如果这种 `Service` 里面发生了内存泄漏, 会影响到整个程序.
- `Scoped` 我们称 `ASP.NET` 每一个请求创建了一个 `scope`, 因此对于 `Scoped` 生命周期会在当前请求生效. 即每一个请求都会创建一个新的实例, 但是在一个请求中所有实例都相同. 适用于一个请求中保存状态.
- `Transient` 在每次获取实例时都创建一个新的实例. 所以它会占用比较多的内存和资源, 对性能产生影响. 只适合无状态的轻量级的 `Service`.

<!-- more -->
