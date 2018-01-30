---
title: "不当实例化对立模式"
description: "避免连续创建本应一次性创建，然后共享的对象的新实例。"
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 4b217f7fc644901eb5c3e77319d151caed30eef1
ms.sourcegitcommit: cf207fd10110f301f1e05f91eeb9f8dfca129164
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/29/2018
---
# <a name="improper-instantiation-antipattern"></a>不当实例化对立模式

连续创建本应一次性创建，然后共享的对象的新实例可能会损害性能。 

## <a name="problem-description"></a>问题描述

许多库提供外部资源的抽象。 在内部，这些类通常管理其自身与资源之间的连接，充当可由客户端用来访问资源的中转站。 下面是与 Azure 应用程序相关的中转站类的一些示例：

- `System.Net.Http.HttpClient`。 使用 HTTP 来与 Web 服务通信。
- `Microsoft.ServiceBus.Messaging.QueueClient`。 向服务总线队列发布和接收消息。 
- `Microsoft.Azure.Documents.Client.DocumentClient`。 连接到 Cosmos DB 实例
- `StackExchange.Redis.ConnectionMultiplexer`。 连接到 Redis，包括 Azure Redis 缓存。

这些类应该只实例化一次，并在应用程序的整个生存期内重复使用。 但是，一个常见的误解是，只能在有必要时才获取这些类，并应快速将其释放。 （此处正好列出了 .NET 库，但模式不是 .NET 特有的）。

以下 ASP.NET 示例创建一个 `HttpClient` 实例来与远程服务通信。 可在[此处][sample-app]找到完整示例。

```csharp
public class NewHttpClientInstancePerRequestController : ApiController
{
    // This method creates a new instance of HttpClient and disposes it for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        using (var httpClient = new HttpClient())
        {
            var hostName = HttpContext.Current.Request.Url.Host;
            var result = await httpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
            return new Product { Name = result };
        }
    }
}
```

在 Web 应用程序中，此技术不可缩放。 为每个用户请求创建了一个新的 `HttpClient` 对象。 在重负载下，Web 服务器可能会耗尽可用的套接字，从而导致 `SocketException` 错误。

此问题并不局限于 `HttpClient` 类。 用于包装资源或者创建开销较高的其他类可能导致类似问题。 以下示例创建 `ExpensiveToCreateService` 类的实例。 此处的问题不一定是套接字耗尽问题，而只是创建每个实例需要花费多长时间。 连续创建再销毁此类的实例可能对系统的可伸缩性造成不利影响。

```csharp
public class NewServiceInstancePerRequestController : ApiController
{
    public async Task<Product> GetProductAsync(string id)
    {
        var expensiveToCreateService = new ExpensiveToCreateService();
        return await expensiveToCreateService.GetProductByIdAsync(id);
    }
}

public class ExpensiveToCreateService
{
    public ExpensiveToCreateService()
    {
        // Simulate delay due to setup and configuration of ExpensiveToCreateService
        Thread.SpinWait(Int32.MaxValue / 100);
    }
    ...
}
```

## <a name="how-to-fix-the-problem"></a>如何解决问题

如果用于包装外部资源的类可共享且是线程安全的，可创建该类的共享单一实例或可重用实例池。

以下示例使用静态 `HttpClient` 实例，因此在所有请求之间共享了连接。

```csharp
public class SingleHttpClientInstanceController : ApiController
{
    private static readonly HttpClient HttpClient;

    static SingleHttpClientInstanceController()
    {
        HttpClient = new HttpClient();
    }

    // This method uses the shared instance of HttpClient for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        var hostName = HttpContext.Current.Request.Url.Host;
        var result = await HttpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
        return new Product { Name = result };
    }
}
```

## <a name="considerations"></a>注意事项

- 此对立模式的关键要素是重复创建和销毁可共享对象的实例。 如果某个类不可共享（不是线程安全的），则此对立模式不适用。

- 共享资源的类型可能决定了是要使用单一实例还是创建池。 `HttpClient` 类旨在进行共享而不是入池。 其他对象可能支持入池，使系统能够将工作负荷分散到多个实例。

- 在多个请求之间共享的对象必须是线程安全的。 应该以这种方式使用 `HttpClient` 类，但其他类可能不支持并发请求，因此需查看可用文档。

- 某些资源类型很消耗资源，必要时应将其放弃。 数据库连接就是一个例子。 保留打开一个不需要的数据库连接可能导致其他并发用户无法访问数据库。

- 在 .NET Framework 中，与外部资源建立连接的许多对象是使用管理这些连接的其他类的静态工厂方法创建的。 应该保存并重复使用这些工厂对象，而不是将其释放再重新创建。 例如，在 Azure 服务总线中，`QueueClient` 对象是通过 `MessagingFactory` 对象创建的。 在内部，`MessagingFactory` 管理连接。 有关详细信息，请参阅[有关使用服务总线消息传送提高性能的最佳做法][service-bus-messaging]。

## <a name="how-to-detect-the-problem"></a>如何检测问题

此问题的症状包括吞吐量下降或错误率增多，以及以下一种或多种症状： 

- 表明套接字、数据库连接、文件句柄等资源耗尽的异常增多。 
- 内存用量和垃圾回收次数增多。
- 网络、磁盘或数据库活动增多。

可执行以下步骤来帮助识别此问题：

1. 对生产系统执行进程监视，识别响应时间变长，或系统因缺少资源而发生故障的位置。
2. 检查在这些位置捕获到的遥测数据，确定哪些操作可能在创建和销毁消耗资源的对象。
3. 在受控的测试环境而不是生产系统中针对每个可疑的操作执行负载测试。
4. 查看源代码，检查中转站对象的管理方式。

查看缓慢运行的，或者在系统承受负载时生成异常的操作的堆栈跟踪。 此信息可帮助识别这些操作如何利用资源。 异常可帮助确定错误是否因共享资源耗尽而导致。 

## <a name="example-diagnosis"></a>示例诊断

以下部分将这些步骤应用到前面所述的示例应用程序。

### <a name="identify-points-of-slow-down-or-failure"></a>识别速度减慢或发生故障的位置

下图显示使用 [New Relic APM][new-relic] 生成的结果，其中显示了响应时间不佳的操作。 在本例中，值得进一步调查 `NewHttpClientInstancePerRequest` 控制器中的 `GetProductAsync` 方法。 请注意，在运行这些操作时，错误率也会增多。 

![New Relic 监视仪表板，其中显示示例应用程序正在针对每个请求创建 HttpClient 对象的新实例][dashboard-new-HTTPClient-instance]

### <a name="examine-telemetry-data-and-find-correlations"></a>检查遥测数据并找出关联

下图显示了在上图的对应时段内，使用线程分析捕获的数据。 系统花费了大量的时间来打开套接字连接，而关闭这些连接和处理套接字异常所花费的时间甚至更长。

![New Relic 线程探查器，其中显示示例应用程序正在针对每个请求创建 HttpClient 对象的新实例][thread-profiler-new-HTTPClient-instance]

### <a name="performing-load-testing"></a>执行负载测试

使用负载测试模拟用户可能执行的典型操作。 这可以帮助识别系统的哪些部分在承受不同负载的情况下，受到了资源耗尽的影响。 请在受控环境而不是生产系统中执行这些测试。 下图显示了在用户负载增大到 100 个并发用户时，`NewHttpClientInstancePerRequest` 控制器处理的请求吞吐量。

![针对每个请求创建 HttpClient 对象新实例的示例应用程序的吞吐量][throughput-new-HTTPClient-instance]

一开始，每秒处理的请求数量随着工作负荷的增大而增加。 但是，在负载变为大约 30 个用户后，成功请求的数量达到限制，并且系统开始生成异常。 从那时起，异常数量随着用户负载的增大而逐渐增加。 

负载测试将这些故障报告为 HTTP 500（内部服务器）错误。 查看遥测数据发现，这些错误的原因是随着创建的 `HttpClient` 对象越来越多，系统逐渐耗尽套接字资源。

下图显示了针对一个创建自定义 `ExpensiveToCreateService` 对象的控制器执行的类似测试。

![针对每个请求创建 ExpensiveToCreateService 新实例的示例应用程序的吞吐量][throughput-new-ExpensiveToCreateService-instance]

此时，控制器未生成任何异常，但吞吐量仍保持平稳状态，同时，平均响应时间以 20 为系数增加。 （该图对响应时间和吞吐量使用了对数刻度。）遥测数据显示，创建 `ExpensiveToCreateService` 的新实例是问题的主要原因。

### <a name="implement-the-solution-and-verify-the-result"></a>实施解决方案并验证结果

将 `GetProductAsync` 方法切换为共享单个 `HttpClient` 实例之后，第二次负载测试表明性能有所改进。 未报告任何错误，并且系统能够处理更高的负载：每秒最多可处理 500 个请求。 与前面的测试相比，平均响应时间下降了一半。

![针对每个请求重复使用 HttpClient 对象新实例的示例应用程序的吞吐量][throughput-single-HTTPClient-instance]

下图显示了堆栈跟踪遥测数据用于比较。 此时，系统将大部分时间花费在执行实际工作上，而不是花费在打开和关闭套接字上。

![New Relic 线程探查器，其中显示示例应用程序正在针对所有请求创建 HttpClient 对象的单个实例][thread-profiler-single-HTTPClient-instance]

下图显示使用 `ExpensiveToCreateService` 对象的共享实例执行的类似负载测试。 同样，处理的请求数量随着用户负载的增大而增加，同时，平均响应时间保持较低水平。 

![针对每个请求重复使用 HttpClient 对象新实例的示例应用程序的吞吐量][throughput-single-ExpensiveToCreateService-instance]



[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ImproperInstantiation
[service-bus-messaging]: /azure/service-bus-messaging/service-bus-performance-improvements
[new-relic]: https://newrelic.com/application-monitoring
[throughput-new-HTTPClient-instance]: _images/HttpClientInstancePerRequest.jpg
[dashboard-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestWebTransactions.jpg
[thread-profiler-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestThreadProfile.jpg
[throughput-new-ExpensiveToCreateService-instance]: _images/ServiceInstancePerRequest.jpg
[throughput-single-HTTPClient-instance]: _images/SingleHttpClientInstance.jpg
[throughput-single-ExpensiveToCreateService-instance]: _images/SingleServiceInstance.jpg
[thread-profiler-single-HTTPClient-instance]: _images/SingleHttpClientInstanceThreadProfile.jpg
