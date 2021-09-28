---
title: 在 ASP.NET 中使用生成 Swagger 文档和 Typescript 客户端程序
date: 2021-09-23T10:30:20+08:00
tags: ["ASP.NET", "Swagger", "Angular", "客户端"]
categories: ["ASP.NET"]
---

本文将使用 ASP.NET 创建一个 WebApi 程序, 并使用 [`Swashbuckle`](https://www.nuget.org/packages?q=Swashbuckle) 生成 Swagger 文档和对应的 Typescript 客户端程序, 同时创建一个 `Angular` 客户端程序调用生成的 Api. 示例代码在 [aspnet-swagger-codegen-example](https://github.com/wangbinyq/aspnet-swagger-codegen-example).
<!-- more -->

## WebApi 程序

首先使用 `dotnet cli` 创建一个 WebApi 程序:

```bash
$ dotnet new webapi -o Swagger
```

可以看到 WebApi 的模板已经自动帮我们设置好了 Swagger.

```C#
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new() { Title = "Swagger", Version = "v1" });
});

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(c => c.SwaggerEndpoint("/swagger/v1/swagger.json", "Swagger v1"));
}
```

在开始之前, 先对 `appsettings.json` 添加 `Urls` 配置:

```json
{
  "Urls": "http://localhost:5001"
}
```

这样我们固定 Api 程序的监听端口, 方便后面客户端使用. 现在我们通过命令 `dotnet watch run` 启动程序. NET6 中使用 `watch` 命令可以获得 `hot reload` 支持. 在浏览器中打开 `http://localhost:5001/swagger/index.html` 可以看到自带的 `WeatherForecast` 的接口文档了.

点击接口可以看到接口定义的出参入参, 再点击 `Try it out` 然后点击 `Execute`, `Responses` 中就显示接口返回的数据.

![接口返回数据]("/image/aspnet-openapi/swagger-1.png)

### 添加接口

在 `Models` 文件夹中定义新接口的模型: `CountResponse.cs` 和 `CountPostResponse.cs`

```c#
namespace Swagger.Models;

public class CountResponse
{
    public int Count { get; set; }
}
```

```c#
namespace Swagger.Models;

public class CountPostRequest
{
    public int Add { get; set; }
}
```

在 `Controllers` 文件夹中添加 `CountController.cs`

```c#
using Microsoft.AspNetCore.Mvc;
using Swagger.Models;

namespace Swagger.Controllers;

[ApiController]
[Route("[controller]")]
public class CountController
{
    private static int _count = 0;

    [HttpGet]
    public CountResponse Get()
    {
        return new CountResponse
        {
            Count = _count
        };
    }

    [HttpPost]
    public CountResponse Post(CountPostRequest req)
    {
        Interlocked.Add(ref _count, req.Add);

        return new CountResponse
        {
            Count = _count
        };
    }
}
```

刷新浏览器, 就可以看到新添加的接口已经存在了

![新接口]("/image/aspnet-openapi/swagger-2.png)

## 客户端程序

首先我们修改 `cjproj` 文件, 添加如下字段

```xml
<ItemGroup>
  <Watch Remove="Client\**\*"></Watch>
</ItemGroup>
```
这样 `watch` 命令就不会监听客户端程序文件的变动, 然后输入命令 `ng new Client` 添加一个 `Angular` 程序. 接下来我们配置 `Angular` 的代理服务器. 在 `Client` 根目录下新建文件 `proxy.conf.json`:

```json
{
  "/api": {
    "target": "http://localhost:5001",
    "secure": false,
    "pathRewrite": {
      "^/api": ""
    }
  }
}
```

修改 `angular.json`, 添加 `proxyConfig` 项:

```json
{
  "architect": {
    "serve": {
      "builder": "@angular-devkit/build-angular:dev-server",
      "options": {
        "proxyConfig": "proxy.conf.json"
      },
    }
  }
}
```

然后通过 `yarn start` 启动.

## Swagger CodeGen
生成 Api 客户端代码的方式有许多, 可以使用工具从 OpenAPI 文档生成, 另外就是使用 [`NSwag.CodeGeneration.TypeScript`](https://github.com/RicoSuter/NSwag/wiki/TypeScriptClientGenerator) 直接从代码生成.

先来看工具生成的方式, 我们选择的是 [`NSwagStudio`](https://github.com/RicoSuter/NSwag/wiki/NSwagStudio) 来生成 Api 的客户端代码. 从该[链接](https://github.com/RicoSuter/NSwag/wiki/NSwagStudio)下载并安装 `NSwagStudio`. 打开 `NSwagStduio` 后, 左边是 OpenAPI 定义的输入, 右边是客户端生成程序的输出. 我们选择使用 `OpenAPI/Swagger Specification` 作为输入, 在 `URL` 中填写我们的 OpenAPI 定义 json 文件路径 `http://localhost:5001/swagger/v1/swagger.json`, 然后勾选 `Typescript Client` 作为输出, `Template` 选择 `Angular`. 最后点击 `Generate Output`


![NSwagStudio]("/image/aspnet-openapi/swagger-3.png)


然后复制右边 `Output` 标签中的内容到 `Client/client.ts` 中保存.

在我们调用接口之前还需要做一点设置, 即注入 `API_BASE_URL` 和 `Client`, 修改 `app.module.ts`

```ts
@NgModule({
  declarations: [
    AppComponent,
  ],
  imports: [
    BrowserModule,
    HttpClientModule,
  ],
  providers: [
    { provide: API_BASE_URL, useValue: '/api' },
    { provide: Client }
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

现在我们就可以在 `Angular` 中调用 `WebApi` 的接口了. 修改 `app.component.html` 和 `app.component.ts` 为:

```html
<div>
  count: {{ (count$ | async)?.count }}

  <button (click)="postCount(1)">+1</button>
  <button (click)="postCount(10)">+10</button>
</div>
```

```ts
import { Component } from '@angular/core';
import { Observable } from 'rxjs';
import {  tap } from 'rxjs/operators';
import { Client, CountResponse } from 'src/client';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})
export class AppComponent {
  title = 'Client';

  count$?: Observable<CountResponse>

  constructor(private client: Client) {

  }

  ngOnInit() {
    this.count$ = this.client.countGET()
  }

  getCount() {
    this.count$ = this.client.countGET()
  }

  postCount(add: number) {
    this.client.countPOST({
      add,
    }).pipe(
      tap(() => this.getCount())
    ).subscribe()
  }
}
```

这样我们就获得类型安全的客户端 Api 代码.