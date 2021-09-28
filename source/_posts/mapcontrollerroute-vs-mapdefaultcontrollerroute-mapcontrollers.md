---
title: "ASP.NET 中 MapControllerRoute, MapDefaultControllerRoute 和 MapControllers 的区别"
date: 2021-09-24T12:52:02+08:00
description: "ASP.NET 中 MapControllerRoute, MapDefaultControllerRoute 和 MapControllers 的区别"
tags: ["ASP.NET", "路由"]
categories: ["ASP.NET"]
---

`MapControllerRoute` 定义了默认的 `Controller` 的路由映射关系, 就像我们在大部分的教程中看到的:

```c#
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

`MapDefaultControllerRoute` 是上面代码的缩写形式, 也就是 `pattern` 等于 `{controller=Home}/{action=Index}/{id?}`.

`MapControllers` 没有定义默认的 `Controller` 的路由, 我们需要在 `Controller` 或 `Action` 上使用 `[Route]` 来定义对应的路由, 大部分时候在 `ApiController` 中使用.
<!-- more -->
