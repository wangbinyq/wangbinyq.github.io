---
title: "在 .NET 中运行后台任务"
date: 2021-09-24T09:36:50+08:00
description: "在 .NET 中运行后台任务"
tags: [".NET", "后台任务", "定时任务", "队列任务", "Generic Host"]
categories: [".NET", "ASP.NET"]
---

在一个 `ASP.NET` 程序中我们常常需要进行比较耗时间的任务, 通常我们不会在一个请求中等待这些任务完成后再返回响应, 比如发邮件, 短信等. 这时候就需要我们将任务放到后台运行, 然后直接返回响应, 告诉用户任务已经再进行了. 本文将教会大家如何在 `.NET` 中运行后台任务, 示例代码请点击[链接](https://github.com/wangbinyq/net-background-task-example)
<!-- more -->

## 创建 `Worker` 程序

`.NET` 自带了一个后台任务模板, 我们执行 `dotnet new worker -o BackgroundTask` 就可以创建一个通用的 `console` 后台任务程序. 该模板中包含一个 `Worker` 类型, 继承了 `BackgroundService`, 然后在 `HostBuilder` 中通过 `AddHostedService` 注册. 最后调用 `host.RunAsync()` 执行该后台任务. 该示例程序每隔一秒钟打印当前时间.

```c#
namespace BackgroundTask;

public class Worker : BackgroundService
{
    private readonly ILogger<Worker> _logger;

    public Worker(ILogger<Worker> logger)
    {
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            _logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);
            await Task.Delay(1000, stoppingToken);
        }
    }
}
```

```c#
using BackgroundTask;

IHost host = Host.CreateDefaultBuilder(args)
    .ConfigureServices(services =>
    {
        services.AddHostedService<Worker>();
    })
    .Build();

await host.RunAsync();
```

`AddHostedService` 接受一个 `IHostedService` 的泛型参数, 其有两个方法 `StartAsync` 和 `StopAsync`. 一个 `IHostedService` 跟一个单例 `Service` 相似, 只是启动时会调用 `StartAsync` 方法, 结束时调用 `StopAsync` 方法.
`BackgroundService` 实际上是一个 `IHostedService` 长时间运行的实现. 

现在我们添加一个 `SimpleWorker` 类:

```c#
namespace BackgroundTask;

public class SimpleWorker : IHostedService
{
    private ILogger<SimpleWorker> _logger;

    public SimpleWorker(ILogger<SimpleWorker> logger)
    {
        _logger = logger;
    }

    public Task StartAsync(CancellationToken cancellationToken)
    {
        _logger.LogInformation("Simple Worker Start");
        return Task.CompletedTask;
    }

    public Task StopAsync(CancellationToken cancellationToken)
    {
        _logger.LogInformation("Simple Worker Stop");
        return Task.CompletedTask;
    }
}
```
我们还需要在 `Program.cs` 中添加注册.

```c#
    services.AddHostedService<SimpleWorker>();
```

现在在启动和结束时会分别打印信息.

![打印信息](/images/background-task/task-1.png)


## 使用 `Timer` 进行定时任务

默认的 `Worker` 实现定时每秒钟打印执行的效果, 我们也可以使用 `Timer` 可以获得同样的效果.

```c#
namespace BackgroundTask;

public class TimerWorker : IHostedService
{
    private int _count = 0;
    private ILogger<TimerWorker> _logger;
    private Timer? _timer;

    public TimerWorker(ILogger<TimerWorker> logger)
    {
        _logger = logger;
    }

    public Task StartAsync(CancellationToken cancellationToken)
    {
        _logger.LogInformation("Timer Worker Start");
        _timer = new Timer(DoWork, null, TimeSpan.Zero, TimeSpan.FromSeconds(5));

        return Task.CompletedTask;
    }

    public Task StopAsync(CancellationToken cancellationToken)
    {
        _logger.LogInformation("Timer Worker Stop");

        return Task.CompletedTask;
    }

    private void DoWork(object? state)
    {
        var count = Interlocked.Increment(ref _count);

        _logger.LogInformation($"count = {count}, time: {DateTimeOffset.Now}");
    }
}
```

同时别忘了注册该 `Service`.

![定时任务](/images/background-task/task-2.png)

## 使用 `Channel` 创建任务队列

更常见的使用后台任务的方式是使用队列, `.NET` 为我们提供了 `System.Threading.Channel` 类. 这里我们需要两个 `Worker`: 一个作为生产者, 一个作为消费者:

```c#
namespace BackgroundTask;

using System.Threading.Channels;

public class ProducerWorker : BackgroundService
{
    private int _count = 0;
    private Channel<int> _queue;

    public ProducerWorker(Channel<int> queue)
    {
        _queue = queue;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            var count = Interlocked.Increment(ref _count);
            await _queue.Writer.WriteAsync(count, stoppingToken);
            await Task.Delay(1000, stoppingToken);
        }
    }
}
```

```c#
namespace BackgroundTask;

using System.Threading;
using System.Threading.Channels;
using System.Threading.Tasks;

public class ConsumerWorker : BackgroundService
{
    private Channel<int> _queue;
    private ILogger<ConsumerWorker> _logger;

    public ConsumerWorker(Channel<int> queue, ILogger<ConsumerWorker> logger)
    {
        _queue = queue;
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            var count = await _queue.Reader.ReadAsync(stoppingToken);

            _logger.LogInformation($"do work for: {count}");
        }
    }
}
```

同时我们需要通过依赖注入注入一个 `Channel<int>` 的单例:

```c#
var queue = Channel.CreateBounded<int>(10);
services.AddSingleton(queue);
services.AddHostedService<ProducerWorker>();
services.AddHostedService<ConsumerWorker>();
```

![队列任务](/images/background-task/task-3.png)