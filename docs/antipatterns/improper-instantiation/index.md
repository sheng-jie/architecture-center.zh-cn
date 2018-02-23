---
title: "不当实例化对立模式"
description: "避免连续创建本应一次性创建，然后共享的对象的新实例。"
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 4d5ef9ad9e675b46df94b51e81d7a4bd4c1b25e9
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/23/2018
---
# <a name="improper-instantiation-antipattern"></a><span data-ttu-id="cb601-103">不当实例化对立模式</span><span class="sxs-lookup"><span data-stu-id="cb601-103">Improper Instantiation antipattern</span></span>

<span data-ttu-id="cb601-104">连续创建本应一次性创建，然后共享的对象的新实例可能会损害性能。</span><span class="sxs-lookup"><span data-stu-id="cb601-104">It can hurt performance to continually create new instances of an object that is meant to be created once and then shared.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="cb601-105">问题描述</span><span class="sxs-lookup"><span data-stu-id="cb601-105">Problem description</span></span>

<span data-ttu-id="cb601-106">许多库提供外部资源的抽象。</span><span class="sxs-lookup"><span data-stu-id="cb601-106">Many libraries provide abstractions of external resources.</span></span> <span data-ttu-id="cb601-107">在内部，这些类通常管理其自身与资源之间的连接，充当可由客户端用来访问资源的中转站。</span><span class="sxs-lookup"><span data-stu-id="cb601-107">Internally, these classes typically manage their own connections to the resource, acting as brokers that clients can use to access the resource.</span></span> <span data-ttu-id="cb601-108">下面是与 Azure 应用程序相关的中转站类的一些示例：</span><span class="sxs-lookup"><span data-stu-id="cb601-108">Here are some examples of broker classes that are relevant to Azure applications:</span></span>

- <span data-ttu-id="cb601-109">`System.Net.Http.HttpClient`。</span><span class="sxs-lookup"><span data-stu-id="cb601-109">`System.Net.Http.HttpClient`.</span></span> <span data-ttu-id="cb601-110">使用 HTTP 来与 Web 服务通信。</span><span class="sxs-lookup"><span data-stu-id="cb601-110">Communicates with a web service using HTTP.</span></span>
- <span data-ttu-id="cb601-111">`Microsoft.ServiceBus.Messaging.QueueClient`。</span><span class="sxs-lookup"><span data-stu-id="cb601-111">`Microsoft.ServiceBus.Messaging.QueueClient`.</span></span> <span data-ttu-id="cb601-112">向服务总线队列发布和接收消息。</span><span class="sxs-lookup"><span data-stu-id="cb601-112">Posts and receives messages to a Service Bus queue.</span></span> 
- <span data-ttu-id="cb601-113">`Microsoft.Azure.Documents.Client.DocumentClient`。</span><span class="sxs-lookup"><span data-stu-id="cb601-113">`Microsoft.Azure.Documents.Client.DocumentClient`.</span></span> <span data-ttu-id="cb601-114">连接到 Cosmos DB 实例</span><span class="sxs-lookup"><span data-stu-id="cb601-114">Connects to a Cosmos DB instance</span></span>
- <span data-ttu-id="cb601-115">`StackExchange.Redis.ConnectionMultiplexer`。</span><span class="sxs-lookup"><span data-stu-id="cb601-115">`StackExchange.Redis.ConnectionMultiplexer`.</span></span> <span data-ttu-id="cb601-116">连接到 Redis，包括 Azure Redis 缓存。</span><span class="sxs-lookup"><span data-stu-id="cb601-116">Connects to Redis, including Azure Redis Cache.</span></span>

<span data-ttu-id="cb601-117">这些类应该只实例化一次，并在应用程序的整个生存期内重复使用。</span><span class="sxs-lookup"><span data-stu-id="cb601-117">These classes are intended to be instantiated once and reused throughout the lifetime of an application.</span></span> <span data-ttu-id="cb601-118">但是，一个常见的误解是，只能在有必要时才获取这些类，并应快速将其释放。</span><span class="sxs-lookup"><span data-stu-id="cb601-118">However, it's a common misunderstanding that these classes should be acquired only as necessary and released quickly.</span></span> <span data-ttu-id="cb601-119">（此处正好列出了 .NET 库，但模式不是 .NET 特有的）。以下 ASP.NET 示例创建一个 `HttpClient` 实例来与远程服务通信。</span><span class="sxs-lookup"><span data-stu-id="cb601-119">(The ones listed here happen to be .NET libraries, but the pattern is not unique to .NET.) The following ASP.NET example creates an instance of `HttpClient` to communicate with a remote service.</span></span> <span data-ttu-id="cb601-120">可在[此处][sample-app]找到完整示例。</span><span class="sxs-lookup"><span data-stu-id="cb601-120">You can find the complete sample [here][sample-app].</span></span>

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

<span data-ttu-id="cb601-121">在 Web 应用程序中，此技术不可缩放。</span><span class="sxs-lookup"><span data-stu-id="cb601-121">In a web application, this technique is not scalable.</span></span> <span data-ttu-id="cb601-122">为每个用户请求创建了一个新的 `HttpClient` 对象。</span><span class="sxs-lookup"><span data-stu-id="cb601-122">A new `HttpClient` object is created for each user request.</span></span> <span data-ttu-id="cb601-123">在重负载下，Web 服务器可能会耗尽可用的套接字，从而导致 `SocketException` 错误。</span><span class="sxs-lookup"><span data-stu-id="cb601-123">Under heavy load, the web server may exhaust the number of available sockets, resulting in `SocketException` errors.</span></span>

<span data-ttu-id="cb601-124">此问题并不局限于 `HttpClient` 类。</span><span class="sxs-lookup"><span data-stu-id="cb601-124">This problem is not restricted to the `HttpClient` class.</span></span> <span data-ttu-id="cb601-125">用于包装资源或者创建开销较高的其他类可能导致类似问题。</span><span class="sxs-lookup"><span data-stu-id="cb601-125">Other classes that wrap resources or are expensive to create might cause similar issues.</span></span> <span data-ttu-id="cb601-126">以下示例创建 `ExpensiveToCreateService` 类的实例。</span><span class="sxs-lookup"><span data-stu-id="cb601-126">The following example creates an instances of the `ExpensiveToCreateService` class.</span></span> <span data-ttu-id="cb601-127">此处的问题不一定是套接字耗尽问题，而只是创建每个实例需要花费多长时间。</span><span class="sxs-lookup"><span data-stu-id="cb601-127">Here the issue is not necessarily socket exhaustion, but simply how long it takes to create each instance.</span></span> <span data-ttu-id="cb601-128">连续创建再销毁此类的实例可能对系统的可伸缩性造成不利影响。</span><span class="sxs-lookup"><span data-stu-id="cb601-128">Continually creating and destroying instances of this class might adversely affect the scalability of the system.</span></span>

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

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="cb601-129">如何解决问题</span><span class="sxs-lookup"><span data-stu-id="cb601-129">How to fix the problem</span></span>

<span data-ttu-id="cb601-130">如果用于包装外部资源的类可共享且是线程安全的，可创建该类的共享单一实例或可重用实例池。</span><span class="sxs-lookup"><span data-stu-id="cb601-130">If the class that wraps the external resource is shareable and thread-safe, create a shared singleton instance or a pool of reusable instances of the class.</span></span>

<span data-ttu-id="cb601-131">以下示例使用静态 `HttpClient` 实例，因此在所有请求之间共享了连接。</span><span class="sxs-lookup"><span data-stu-id="cb601-131">The following example uses a static `HttpClient` instance, thus sharing the connection across all requests.</span></span>

```csharp
public class SingleHttpClientInstanceController : ApiController
{
    private static readonly HttpClient httpClient;

    static SingleHttpClientInstanceController()
    {
        httpClient = new HttpClient();
    }

    // This method uses the shared instance of HttpClient for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        var hostName = HttpContext.Current.Request.Url.Host;
        var result = await httpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
        return new Product { Name = result };
    }
}
```

## <a name="considerations"></a><span data-ttu-id="cb601-132">注意事项</span><span class="sxs-lookup"><span data-stu-id="cb601-132">Considerations</span></span>

- <span data-ttu-id="cb601-133">此对立模式的关键要素是重复创建和销毁可共享对象的实例。</span><span class="sxs-lookup"><span data-stu-id="cb601-133">The key element of this antipattern is repeatedly creating and destroying instances of a *shareable* object.</span></span> <span data-ttu-id="cb601-134">如果某个类不可共享（不是线程安全的），则此对立模式不适用。</span><span class="sxs-lookup"><span data-stu-id="cb601-134">If a class is not shareable (not thread-safe), then this antipattern does not apply.</span></span>

- <span data-ttu-id="cb601-135">共享资源的类型可能决定了是要使用单一实例还是创建池。</span><span class="sxs-lookup"><span data-stu-id="cb601-135">The type of shared resource might dictate whether you should use a singleton or create a pool.</span></span> <span data-ttu-id="cb601-136">`HttpClient` 类旨在进行共享而不是入池。</span><span class="sxs-lookup"><span data-stu-id="cb601-136">The `HttpClient` class is designed to be shared rather than pooled.</span></span> <span data-ttu-id="cb601-137">其他对象可能支持入池，使系统能够将工作负荷分散到多个实例。</span><span class="sxs-lookup"><span data-stu-id="cb601-137">Other objects might support pooling, enabling the system to spread the workload across multiple instances.</span></span>

- <span data-ttu-id="cb601-138">在多个请求之间共享的对象必须是线程安全的。</span><span class="sxs-lookup"><span data-stu-id="cb601-138">Objects that you share across multiple requests *must* be thread-safe.</span></span> <span data-ttu-id="cb601-139">应该以这种方式使用 `HttpClient` 类，但其他类可能不支持并发请求，因此需查看可用文档。</span><span class="sxs-lookup"><span data-stu-id="cb601-139">The `HttpClient` class is designed to be used in this manner, but other classes might not support concurrent requests, so check the available documentation.</span></span>

- <span data-ttu-id="cb601-140">某些资源类型很消耗资源，必要时应将其放弃。</span><span class="sxs-lookup"><span data-stu-id="cb601-140">Some resource types are scarce and should not be held onto.</span></span> <span data-ttu-id="cb601-141">数据库连接就是一个例子。</span><span class="sxs-lookup"><span data-stu-id="cb601-141">Database connections are an example.</span></span> <span data-ttu-id="cb601-142">保留打开一个不需要的数据库连接可能导致其他并发用户无法访问数据库。</span><span class="sxs-lookup"><span data-stu-id="cb601-142">Holding an open database connection that is not required may prevent other concurrent users from gaining access to the database.</span></span>

- <span data-ttu-id="cb601-143">在 .NET Framework 中，与外部资源建立连接的许多对象是使用管理这些连接的其他类的静态工厂方法创建的。</span><span class="sxs-lookup"><span data-stu-id="cb601-143">In the .NET Framework, many objects that establish connections to external resources are created by using static factory methods of other classes that manage these connections.</span></span> <span data-ttu-id="cb601-144">应该保存并重复使用这些工厂对象，而不是将其释放再重新创建。</span><span class="sxs-lookup"><span data-stu-id="cb601-144">These factories  objects are intended to be saved and reused, rather than disposed and recreated.</span></span> <span data-ttu-id="cb601-145">例如，在 Azure 服务总线中，`QueueClient` 对象是通过 `MessagingFactory` 对象创建的。</span><span class="sxs-lookup"><span data-stu-id="cb601-145">For example, in Azure Service Bus, the `QueueClient` object is created through a `MessagingFactory` object.</span></span> <span data-ttu-id="cb601-146">在内部，`MessagingFactory` 管理连接。</span><span class="sxs-lookup"><span data-stu-id="cb601-146">Internally, the `MessagingFactory` manages connections.</span></span> <span data-ttu-id="cb601-147">有关详细信息，请参阅[有关使用服务总线消息传送提高性能的最佳做法][service-bus-messaging]。</span><span class="sxs-lookup"><span data-stu-id="cb601-147">For more information, see [Best Practices for performance improvements using Service Bus Messaging][service-bus-messaging].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="cb601-148">如何检测问题</span><span class="sxs-lookup"><span data-stu-id="cb601-148">How to detect the problem</span></span>

<span data-ttu-id="cb601-149">此问题的症状包括吞吐量下降或错误率增多，以及以下一种或多种症状：</span><span class="sxs-lookup"><span data-stu-id="cb601-149">Symptoms of this problem include a drop in throughput or an increased error rate, along with one or more of the following:</span></span> 

- <span data-ttu-id="cb601-150">表明套接字、数据库连接、文件句柄等资源耗尽的异常增多。</span><span class="sxs-lookup"><span data-stu-id="cb601-150">An increase in exceptions that indicate exhaustion of resources such as sockets, database connections, file handles, and so on.</span></span> 
- <span data-ttu-id="cb601-151">内存用量和垃圾回收次数增多。</span><span class="sxs-lookup"><span data-stu-id="cb601-151">Increased memory use and garbage collection.</span></span>
- <span data-ttu-id="cb601-152">网络、磁盘或数据库活动增多。</span><span class="sxs-lookup"><span data-stu-id="cb601-152">An increase in network, disk, or database activity.</span></span>

<span data-ttu-id="cb601-153">可执行以下步骤来帮助识别此问题：</span><span class="sxs-lookup"><span data-stu-id="cb601-153">You can perform the following steps to help identify this problem:</span></span>

1. <span data-ttu-id="cb601-154">对生产系统执行进程监视，识别响应时间变长，或系统因缺少资源而发生故障的位置。</span><span class="sxs-lookup"><span data-stu-id="cb601-154">Performing process monitoring of the production system, to identify points when response times slow down or the system fails due to lack of resources.</span></span>
2. <span data-ttu-id="cb601-155">检查在这些位置捕获到的遥测数据，确定哪些操作可能在创建和销毁消耗资源的对象。</span><span class="sxs-lookup"><span data-stu-id="cb601-155">Examine the telemetry data captured at these points to determine which operations might be creating and destroying resource-consuming objects.</span></span>
3. <span data-ttu-id="cb601-156">在受控的测试环境而不是生产系统中针对每个可疑的操作执行负载测试。</span><span class="sxs-lookup"><span data-stu-id="cb601-156">Load test each suspected operation, in a controlled test environment rather than the production system.</span></span>
4. <span data-ttu-id="cb601-157">查看源代码，检查中转站对象的管理方式。</span><span class="sxs-lookup"><span data-stu-id="cb601-157">Review the source code and examine the how broker objects are managed.</span></span>

<span data-ttu-id="cb601-158">查看缓慢运行的，或者在系统承受负载时生成异常的操作的堆栈跟踪。</span><span class="sxs-lookup"><span data-stu-id="cb601-158">Look at stack traces for operations that are slow-running or that generate exceptions when the system is under load.</span></span> <span data-ttu-id="cb601-159">此信息可帮助识别这些操作如何利用资源。</span><span class="sxs-lookup"><span data-stu-id="cb601-159">This information can help to identify how these operations are utilizing resources.</span></span> <span data-ttu-id="cb601-160">异常可帮助确定错误是否因共享资源耗尽而导致。</span><span class="sxs-lookup"><span data-stu-id="cb601-160">Exceptions can help to determine whether errors are caused by shared resources being exhausted.</span></span> 

## <a name="example-diagnosis"></a><span data-ttu-id="cb601-161">示例诊断</span><span class="sxs-lookup"><span data-stu-id="cb601-161">Example diagnosis</span></span>

<span data-ttu-id="cb601-162">以下部分将这些步骤应用到前面所述的示例应用程序。</span><span class="sxs-lookup"><span data-stu-id="cb601-162">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="identify-points-of-slow-down-or-failure"></a><span data-ttu-id="cb601-163">识别速度减慢或发生故障的位置</span><span class="sxs-lookup"><span data-stu-id="cb601-163">Identify points of slow down or failure</span></span>

<span data-ttu-id="cb601-164">下图显示使用 [New Relic APM][new-relic] 生成的结果，其中显示了响应时间不佳的操作。</span><span class="sxs-lookup"><span data-stu-id="cb601-164">The following image shows results generated using [New Relic APM][new-relic], showing operations that have a poor response time.</span></span> <span data-ttu-id="cb601-165">在本例中，`NewHttpClientInstancePerRequest` 控制器中的 `GetProductAsync` 方法值得进一步调查。</span><span class="sxs-lookup"><span data-stu-id="cb601-165">In this case, the `GetProductAsync` method in the `NewHttpClientInstancePerRequest` controller is worth investigating further.</span></span> <span data-ttu-id="cb601-166">请注意，在运行这些操作时，错误率也会增多。</span><span class="sxs-lookup"><span data-stu-id="cb601-166">Notice that the error rate also increases when these operations are running.</span></span> 

![New Relic 监视仪表板，其中显示示例应用程序正在针对每个请求创建 HttpClient 对象的新实例][dashboard-new-HTTPClient-instance]

### <a name="examine-telemetry-data-and-find-correlations"></a><span data-ttu-id="cb601-168">检查遥测数据并找出关联</span><span class="sxs-lookup"><span data-stu-id="cb601-168">Examine telemetry data and find correlations</span></span>

<span data-ttu-id="cb601-169">下图显示了在上图的对应时段内，使用线程分析捕获的数据。</span><span class="sxs-lookup"><span data-stu-id="cb601-169">The next image shows data captured using thread profiling, over the same period corresponding as the previous image.</span></span> <span data-ttu-id="cb601-170">系统花费了大量的时间来打开套接字连接，而关闭这些连接和处理套接字异常所花费的时间甚至更长。</span><span class="sxs-lookup"><span data-stu-id="cb601-170">The system spends a significant time opening socket connections, and even more time closing them and handling socket exceptions.</span></span>

![New Relic 线程探查器，其中显示示例应用程序正在针对每个请求创建 HttpClient 对象的新实例][thread-profiler-new-HTTPClient-instance]

### <a name="performing-load-testing"></a><span data-ttu-id="cb601-172">执行负载测试</span><span class="sxs-lookup"><span data-stu-id="cb601-172">Performing load testing</span></span>

<span data-ttu-id="cb601-173">使用负载测试模拟用户可能执行的典型操作。</span><span class="sxs-lookup"><span data-stu-id="cb601-173">Use load testing to simulate the typical operations that users might perform.</span></span> <span data-ttu-id="cb601-174">这可以帮助识别系统的哪些部分在承受不同负载的情况下，受到了资源耗尽的影响。</span><span class="sxs-lookup"><span data-stu-id="cb601-174">This can help to identify which parts of a system suffer from resource exhaustion under varying loads.</span></span> <span data-ttu-id="cb601-175">请在受控环境而不是生产系统中执行这些测试。</span><span class="sxs-lookup"><span data-stu-id="cb601-175">Perform these tests in a controlled environment rather than the production system.</span></span> <span data-ttu-id="cb601-176">下图显示了在用户负载增大到 100 个并发用户时，`NewHttpClientInstancePerRequest` 控制器处理的请求吞吐量。</span><span class="sxs-lookup"><span data-stu-id="cb601-176">The following graph shows the throughput of requests handled by the `NewHttpClientInstancePerRequest` controller as the user load increases to 100 concurrent users.</span></span>

![针对每个请求创建 HttpClient 对象新实例的示例应用程序的吞吐量][throughput-new-HTTPClient-instance]

<span data-ttu-id="cb601-178">一开始，每秒处理的请求数量随着工作负荷的增大而增加。</span><span class="sxs-lookup"><span data-stu-id="cb601-178">At first, the volume of requests handled per second increases as the workload increases.</span></span> <span data-ttu-id="cb601-179">但是，在负载变为大约 30 个用户后，成功请求的数量达到限制，并且系统开始生成异常。</span><span class="sxs-lookup"><span data-stu-id="cb601-179">At about 30 users, however, the volume of successful requests reaches a limit, and the system starts to generate exceptions.</span></span> <span data-ttu-id="cb601-180">从那时起，异常数量随着用户负载的增大而逐渐增加。</span><span class="sxs-lookup"><span data-stu-id="cb601-180">From then on, the volume of exceptions gradually increases with the user load.</span></span> 

<span data-ttu-id="cb601-181">负载测试将这些故障报告为 HTTP 500（内部服务器）错误。</span><span class="sxs-lookup"><span data-stu-id="cb601-181">The load test reported these failures as HTTP 500 (Internal Server) errors.</span></span> <span data-ttu-id="cb601-182">查看遥测数据发现，这些错误的原因是随着创建的 `HttpClient` 对象越来越多，系统逐渐耗尽套接字资源。</span><span class="sxs-lookup"><span data-stu-id="cb601-182">Reviewing the telemetry showed that these errors were caused by the system running out of socket resources, as more and more `HttpClient` objects were created.</span></span>

<span data-ttu-id="cb601-183">下图显示了针对一个创建自定义 `ExpensiveToCreateService` 对象的控制器执行的类似测试。</span><span class="sxs-lookup"><span data-stu-id="cb601-183">The next graph shows a similar test for a controller that creates the custom `ExpensiveToCreateService` object.</span></span>

![针对每个请求创建 ExpensiveToCreateService 新实例的示例应用程序的吞吐量][throughput-new-ExpensiveToCreateService-instance]

<span data-ttu-id="cb601-185">此时，控制器未生成任何异常，但吞吐量仍保持平稳状态，同时，平均响应时间以 20 为系数增加。</span><span class="sxs-lookup"><span data-stu-id="cb601-185">This time, the controller does not generate any exceptions, but throughput still reaches a plateau, while the average response time increases by a factor of 20.</span></span> <span data-ttu-id="cb601-186">（该图对响应时间和吞吐量使用了对数刻度。）遥测数据显示，创建 `ExpensiveToCreateService` 的新实例是问题的主要原因。</span><span class="sxs-lookup"><span data-stu-id="cb601-186">(The graph uses a logarithmic scale for response time and throughput.) Telemetry showed that creating new instances of the `ExpensiveToCreateService` was the main cause of the problem.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="cb601-187">实施解决方案并验证结果</span><span class="sxs-lookup"><span data-stu-id="cb601-187">Implement the solution and verify the result</span></span>

<span data-ttu-id="cb601-188">将 `GetProductAsync` 方法切换为共享单个 `HttpClient` 实例之后，第二次负载测试表明性能有所改进。</span><span class="sxs-lookup"><span data-stu-id="cb601-188">After switching the `GetProductAsync` method to share a single `HttpClient` instance, a second load test showed improved performance.</span></span> <span data-ttu-id="cb601-189">未报告任何错误，并且系统能够处理更高的负载：每秒最多可处理 500 个请求。</span><span class="sxs-lookup"><span data-stu-id="cb601-189">No errors were reported, and the system was able to handle an increasing load of up to 500 requests per second.</span></span> <span data-ttu-id="cb601-190">与前面的测试相比，平均响应时间下降了一半。</span><span class="sxs-lookup"><span data-stu-id="cb601-190">The average response time was cut in half, compared with the previous test.</span></span>

![针对每个请求重复使用 HttpClient 对象新实例的示例应用程序的吞吐量][throughput-single-HTTPClient-instance]

<span data-ttu-id="cb601-192">下图显示了堆栈跟踪遥测数据用于比较。</span><span class="sxs-lookup"><span data-stu-id="cb601-192">For comparison, the following image shows the stack trace telemetry.</span></span> <span data-ttu-id="cb601-193">此时，系统将大部分时间花费在执行实际工作上，而不是花费在打开和关闭套接字上。</span><span class="sxs-lookup"><span data-stu-id="cb601-193">This time, the system spends most of its time performing real work, rather than opening and closing sockets.</span></span>

![New Relic 线程探查器，其中显示示例应用程序正在针对所有请求创建 HttpClient 对象的单个实例][thread-profiler-single-HTTPClient-instance]

<span data-ttu-id="cb601-195">下图显示使用 `ExpensiveToCreateService` 对象的共享实例执行的类似负载测试。</span><span class="sxs-lookup"><span data-stu-id="cb601-195">The next graph shows a similar load test using a shared instance of the `ExpensiveToCreateService` object.</span></span> <span data-ttu-id="cb601-196">同样，处理的请求数量随着用户负载的增大而增加，同时，平均响应时间保持较低水平。</span><span class="sxs-lookup"><span data-stu-id="cb601-196">Again, the volume of handled requests increases in line with the user load, while the average response time remains low.</span></span> 

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
