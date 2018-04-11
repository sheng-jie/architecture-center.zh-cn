---
title: 无缓存对立模式
description: 反复提取相同的数据可能会降低性能和可伸缩性。
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 8a2bc3b473a30536cc1bef9e1dcad87acb46c4a9
ms.sourcegitcommit: c93f1b210b3deff17cc969fb66133bc6399cfd10
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/05/2018
---
# <a name="no-caching-antipattern"></a><span data-ttu-id="0b9b5-103">无缓存对立模式</span><span class="sxs-lookup"><span data-stu-id="0b9b5-103">No Caching antipattern</span></span>

<span data-ttu-id="0b9b5-104">在处理许多并发请求的云应用程序中，反复提取相同的数据可能会降低性能和可伸缩性。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-104">In a cloud application that handles many concurrent requests, repeatedly fetching the same data can reduce performance and scalability.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="0b9b5-105">问题描述</span><span class="sxs-lookup"><span data-stu-id="0b9b5-105">Problem description</span></span>

<span data-ttu-id="0b9b5-106">在不缓存数据的情况下，可能会导致一些意外的行为，包括：</span><span class="sxs-lookup"><span data-stu-id="0b9b5-106">When data is not cached, it can cause a number of undesirable behaviors, including:</span></span>

- <span data-ttu-id="0b9b5-107">反复从访问开销较高（体现在 I/O 开销或延迟方面）的资源提取相同的信息。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-107">Repeatedly fetching the same information from a resource that is expensive to access, in terms of I/O overhead or latency.</span></span>
- <span data-ttu-id="0b9b5-108">为多个请求反复构造相同的对象或数据结构。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-108">Repeatedly constructing the same objects or data structures for multiple requests.</span></span>
- <span data-ttu-id="0b9b5-109">向具有服务配额并对超过特定限制的客户端施加约束的远程服务发出过多的调用。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-109">Making excessive calls to a remote service that has a service quota and throttles clients past a certain limit.</span></span>

<span data-ttu-id="0b9b5-110">这些问题可能进一步导致响应时间不佳、数据存储中资源争用加剧，以及可伸缩性不佳。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-110">In turn, these problems can lead to poor response times, increased contention in the data store, and poor scalability.</span></span>

<span data-ttu-id="0b9b5-111">以下示例使用实体框架连接到数据库。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-111">The following example uses Entity Framework to connect to a database.</span></span> <span data-ttu-id="0b9b5-112">即使多个请求提取完全相同的数据，每个客户端请求也会导致调用数据库。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-112">Every client request results in a call to the database, even if multiple requests are fetching exactly the same data.</span></span> <span data-ttu-id="0b9b5-113">重复请求的成本（体现在 I/O 开销和数据访问费用方面）可能会迅速累积。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-113">The cost of repeated requests, in terms of I/O overhead and data access charges, can accumulate quickly.</span></span>

```csharp
public class PersonRepository : IPersonRepository
{
    public async Task<Person> GetAsync(int id)
    {
        using (var context = new AdventureWorksContext())
        {
            return await context.People
                .Where(p => p.Id == id)
                .FirstOrDefaultAsync()
                .ConfigureAwait(false);
        }
    }
}
```

<span data-ttu-id="0b9b5-114">可在[此处][sample-app]找到完整示例。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-114">You can find the complete sample [here][sample-app].</span></span>

<span data-ttu-id="0b9b5-115">出现此对立模式的原因通常是：</span><span class="sxs-lookup"><span data-stu-id="0b9b5-115">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="0b9b5-116">不使用缓存的解决方案更容易实施，在低负载下可正常运转。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-116">Not using a cache is simpler to implement, and it works fine under low loads.</span></span> <span data-ttu-id="0b9b5-117">缓存使代码变得更复杂。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-117">Caching makes the code more complicated.</span></span> 
- <span data-ttu-id="0b9b5-118">不能清楚地了解使用缓存的优点和缺点。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-118">The benefits and drawbacks of using a cache are not clearly understood.</span></span>
- <span data-ttu-id="0b9b5-119">关心保持缓存数据的准确性和全新性所涉及的开销。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-119">There is concern about the overhead of maintaining the accuracy and freshness of cached data.</span></span>
- <span data-ttu-id="0b9b5-120">应用程序是从本地系统迁移的，其中的网络延迟不是问题，并且系统在昂贵的高性能硬件上运行，因此，原始设计中未考虑缓存。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-120">An application was migrated from an on-premises system, where network latency was not an issue, and the system ran on expensive high-performance hardware, so caching wasn't considered in the original design.</span></span>
- <span data-ttu-id="0b9b5-121">开发人员未意识到缓存在给定方案中是一个可行的选项。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-121">Developers aren't aware that caching is a possibility in a given scenario.</span></span> <span data-ttu-id="0b9b5-122">例如，开发人员在实现 Web API 时可能不会考虑使用 Etag。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-122">For example, developers may not think of using ETags when implementing a web API.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="0b9b5-123">如何解决问题</span><span class="sxs-lookup"><span data-stu-id="0b9b5-123">How to fix the problem</span></span>

<span data-ttu-id="0b9b5-124">最流行的缓存策略是按需策略或缓存端策略。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-124">The most popular caching strategy is the *on-demand* or *cache-aside* strategy.</span></span>

- <span data-ttu-id="0b9b5-125">读取时，应用程序尝试从缓存中读取数据。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-125">On read, the application tries to read the data from the cache.</span></span> <span data-ttu-id="0b9b5-126">如果数据不在缓存中，应用程序会从数据源中检索数据，并将其添加到缓存。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-126">If the data isn't in the cache, the application retrieves it from the data source and adds it to the cache.</span></span>
- <span data-ttu-id="0b9b5-127">写入时，应用程序将更改直接写入数据源，并从缓存中删除旧值。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-127">On write, the application writes the change directly to the data source and removes the old value from the cache.</span></span> <span data-ttu-id="0b9b5-128">下一次有需要时，会在缓存中检索和添加数据。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-128">It will be retrieved and added to the cache the next time it is required.</span></span>

<span data-ttu-id="0b9b5-129">此方法适合经常更改的数据。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-129">This approach is suitable for data that changes frequently.</span></span> <span data-ttu-id="0b9b5-130">下面将前一示例更新为使用 [缓存端][cache-aside] 模式。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-130">Here is the previous example updated to use the [Cache-Aside][cache-aside] pattern.</span></span>  

```csharp
public class CachedPersonRepository : IPersonRepository
{
    private readonly PersonRepository _innerRepository;

    public CachedPersonRepository(PersonRepository innerRepository)
    {
        _innerRepository = innerRepository;
    }

    public async Task<Person> GetAsync(int id)
    {
        return await CacheService.GetAsync<Person>("p:" + id, () => _innerRepository.GetAsync(id)).ConfigureAwait(false);
    }
}

public class CacheService
{
    private static ConnectionMultiplexer _connection;

    public static async Task<T> GetAsync<T>(string key, Func<Task<T>> loadCache, double expirationTimeInMinutes)
    {
        IDatabase cache = Connection.GetDatabase();
        T value = await GetAsync<T>(cache, key).ConfigureAwait(false);
        if (value == null)
        {
            // Value was not found in the cache. Call the lambda to get the value from the database.
            value = await loadCache().ConfigureAwait(false);
            if (value != null)
            {
                // Add the value to the cache.
                await SetAsync(cache, key, value, expirationTimeInMinutes).ConfigureAwait(false);
            }
        }
        return value;
    }
}
```

<span data-ttu-id="0b9b5-131">请注意，`GetAsync` 方法现在调用 `CacheService` 类，而不是直接调用数据库。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-131">Notice that the `GetAsync` method now calls the `CacheService` class, rather than calling the database directly.</span></span> <span data-ttu-id="0b9b5-132">`CacheService` 类首先尝试从 Azure Redis 缓存中获取项。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-132">The `CacheService` class first tries to get the item from Azure Redis Cache.</span></span> <span data-ttu-id="0b9b5-133">如果在 Redis 缓存中未找到值，`CacheService` 会调用由调用方传递给它的 lambda 函数。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-133">If the value isn't found in Redis Cache, the `CacheService` invokes a lambda function that was passed to it by the caller.</span></span> <span data-ttu-id="0b9b5-134">该 lambda 函数负责从数据库提取数据。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-134">The lambda function is responsible for fetching the data from the database.</span></span> <span data-ttu-id="0b9b5-135">此实现将存储库与特定的缓存解决方案分离开来，并将 `CacheService` 与数据库分离开来。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-135">This implementation decouples the repository from the particular caching solution, and decouples the `CacheService` from the database.</span></span> 

## <a name="considerations"></a><span data-ttu-id="0b9b5-136">注意事项</span><span class="sxs-lookup"><span data-stu-id="0b9b5-136">Considerations</span></span>

- <span data-ttu-id="0b9b5-137">如果缓存不可用（可能是暂时性故障造成的），请不要向客户端返回错误。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-137">If the cache is unavailable, perhaps because of a transient failure, don't return an error to the client.</span></span> <span data-ttu-id="0b9b5-138">而应该从原始数据源提取数据。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-138">Instead, fetch the data from the original data source.</span></span> <span data-ttu-id="0b9b5-139">但请注意，在恢复缓存时，原始数据存储可能忙于处理请求，导致超时和连接失败。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-139">However, be aware that while the cache is being recovered, the original data store could be swamped with requests, resulting in timeouts and failed connections.</span></span> <span data-ttu-id="0b9b5-140">（毕竟这是首先使用缓存的动机之一。）使用[断路器模式][circuit-breaker]等技术可避免数据源瘫痪。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-140">(After all, this is one of the motivations for using a cache in the first place.) Use a technique such as the [Circuit Breaker pattern][circuit-breaker] to avoid overwhelming the data source.</span></span>

- <span data-ttu-id="0b9b5-141">缓存非静态数据的应用程序应设计为支持最终一致性。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-141">Applications that cache nonstatic data should be designed to support eventual consistency.</span></span>

- <span data-ttu-id="0b9b5-142">对于 Web API，可以通过在请求和响应消息中包含 Cache-Control 标头并使用 Etag 标识对象版本，来支持客户端缓存。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-142">For web APIs, you can support client-side caching by including a Cache-Control header in request and response messages, and using ETags to identify versions of objects.</span></span> <span data-ttu-id="0b9b5-143">有关详细信息，请参阅 [API 实现][api-implementation]。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-143">For more information, see [API implementation][api-implementation].</span></span>

- <span data-ttu-id="0b9b5-144">不需要缓存整个实体。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-144">You don't have to cache entire entities.</span></span> <span data-ttu-id="0b9b5-145">如果某个实体的大部分内容是静态的，但只有一小部分经常更改，请缓存静态元素，并从数据源检索动态元素。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-145">If most of an entity is static but only a small piece changes frequently, cache the static elements and retrieve the dynamic elements from the data source.</span></span> <span data-ttu-id="0b9b5-146">此方法有助于减少针对数据源执行的 I/O 数量。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-146">This approach can help to reduce the volume of I/O being performed against the data source.</span></span>

- <span data-ttu-id="0b9b5-147">在某些情况下，如果易失性数据的生存期较短，将它缓存可能会有帮助。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-147">In some cases, if volatile data is short-lived, it can be useful to cache it.</span></span> <span data-ttu-id="0b9b5-148">例如，假设某个设备持续发送状态更新。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-148">For example, consider a device that continually sends status updates.</span></span> <span data-ttu-id="0b9b5-149">有利的做法可能是在此信息抵达时将其缓存，而完全不将它写入持久性存储。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-149">It might make sense to cache this information as it arrives, and not write it to a persistent store at all.</span></span>  

- <span data-ttu-id="0b9b5-150">为了防止数据过时，许多缓存解决方案支持可配置的失效期，以便在达到指定的间隔后，从缓存中自动删除数据。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-150">To prevent data from becoming stale, many caching solutions support configurable expiration periods, so that data is automatically removed from the cache after a specified interval.</span></span> <span data-ttu-id="0b9b5-151">可能需要根据方案优化过期时间。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-151">You may need to tune the expiration time for your scenario.</span></span> <span data-ttu-id="0b9b5-152">高度静态的数据在缓存中的保留时间可以长于可能很快过时的易失性数据。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-152">Data that is highly static can stay in the cache for longer periods than volatile data that may become stale quickly.</span></span>

- <span data-ttu-id="0b9b5-153">如果缓存解决方案未提供内置过期，你可能需要实现一个后台进程，以不时地扫描缓存，防止它不受限制地扩大。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-153">If the caching solution doesn't provide built-in expiration, you may need to implement a background process that occasionally sweeps the cache, to prevent it from growing without limits.</span></span> 

- <span data-ttu-id="0b9b5-154">除了缓存来自外部数据源的数据以外，还可以使用缓存来保存复杂计算的结果。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-154">Besides caching data from an external data source, you can use caching to save the results of complex computations.</span></span> <span data-ttu-id="0b9b5-155">但是，在执行此操作之前，请检测应用程序，确定应用程序是否真正受 CPU 的约束。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-155">Before you do that, however, instrument the application to determine whether the application is really CPU bound.</span></span>

- <span data-ttu-id="0b9b5-156">在应用程序启动时准备好缓存可能很有帮助。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-156">It might be useful to prime the cache when the application starts.</span></span> <span data-ttu-id="0b9b5-157">在缓存中填充最有可能会用到的数据。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-157">Populate the cache with the data that is most likely to be used.</span></span>

- <span data-ttu-id="0b9b5-158">始终包含检测机制来检测缓存命中数和缓存未命中数。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-158">Always include instrumentation that detects cache hits and cache misses.</span></span> <span data-ttu-id="0b9b5-159">使用此信息来优化缓存策略，例如，要缓存哪些数据，以及在数据过期之前要在缓存中保存数据多长时间。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-159">Use this information to tune caching policies, such what data to cache, and how long to hold data in the cache before it expires.</span></span>

- <span data-ttu-id="0b9b5-160">如果缺少缓存造成了瓶颈，则添加缓存可能会大幅增加请求数量，导致 Web 前端过载。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-160">If the lack of caching is a bottleneck, then adding caching may increase the volume of requests so much that the web front end becomes overloaded.</span></span> <span data-ttu-id="0b9b5-161">客户端可能开始收到 HTTP 503（服务不可用）错误。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-161">Clients may start to receive HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="0b9b5-162">这些问题表明需要横向扩展前端。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-162">These are an indication that you should scale out the front end.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="0b9b5-163">如何检测问题</span><span class="sxs-lookup"><span data-stu-id="0b9b5-163">How to detect the problem</span></span>

<span data-ttu-id="0b9b5-164">可以执行以下步骤来帮助识别缺少缓存是否导致了性能问题：</span><span class="sxs-lookup"><span data-stu-id="0b9b5-164">You can perform the following steps to help identify whether lack of caching is causing performance problems:</span></span>

1. <span data-ttu-id="0b9b5-165">检查应用程序设计。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-165">Review the application design.</span></span> <span data-ttu-id="0b9b5-166">盘点应用程序使用的所有数据存储。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-166">Take an inventory of all the data stores that the application uses.</span></span> <span data-ttu-id="0b9b5-167">对于每个数据存储，确定应用程序是否使用了缓存。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-167">For each, determine whether the application is using a cache.</span></span> <span data-ttu-id="0b9b5-168">在可能的情况下，确定数据更改频率。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-168">If possible, determine how frequently the data changes.</span></span> <span data-ttu-id="0b9b5-169">适合缓存的初始候选项包括缓慢更改的数据，以及频繁读取的静态引用数据。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-169">Good initial candidates for caching include data that changes slowly, and static reference data that is read frequently.</span></span> 

2. <span data-ttu-id="0b9b5-170">检测应用程序并监视实时系统，找出应用程序检索数据或计算信息的频率。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-170">Instrument the application and monitor the live system to find out how frequently the application retrieves data or calculates information.</span></span>

3. <span data-ttu-id="0b9b5-171">在测试环境中分析应用程序，捕获有关与数据访问操作或其他经常执行的计算关联的开销的低级指标。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-171">Profile the application in a test environment to capture low-level metrics about the overhead associated with data access operations or other frequently performed calculations.</span></span>

4. <span data-ttu-id="0b9b5-172">在测试环境中执行负载测试，识别系统在承受正常工作负荷和重度负载的情况下如何响应。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-172">Perform load testing in a test environment to identify how the system responds under a normal workload and under heavy load.</span></span> <span data-ttu-id="0b9b5-173">负载测试应使用真实工作负荷模拟在生产环境中观察到的数据访问模式。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-173">Load testing should simulate the pattern of data access observed in the production environment using realistic workloads.</span></span> 

5. <span data-ttu-id="0b9b5-174">检查底层数据存储的数据访问统计信息，并检查相同数据请求的重复频率。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-174">Examine the data access statistics for the underlying data stores and review how often the same data requests are repeated.</span></span> 


## <a name="example-diagnosis"></a><span data-ttu-id="0b9b5-175">示例诊断</span><span class="sxs-lookup"><span data-stu-id="0b9b5-175">Example diagnosis</span></span>

<span data-ttu-id="0b9b5-176">以下部分将这些步骤应用到前面所述的示例应用程序。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-176">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="instrument-the-application-and-monitor-the-live-system"></a><span data-ttu-id="0b9b5-177">检测应用程序并监视实时系统</span><span class="sxs-lookup"><span data-stu-id="0b9b5-177">Instrument the application and monitor the live system</span></span>

<span data-ttu-id="0b9b5-178">应用程序已部署到生产环境时，检测并监视应用程序，以获取有关用户发出的特定请求的信息。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-178">Instrument the application and monitor it to get information about the specific requests that users make while the application is in production.</span></span> 

<span data-ttu-id="0b9b5-179">下图显示了在负载测试期间 [New Relic][NewRelic] 捕获的监视数据。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-179">The following image shows monitoring data captured by [New Relic][NewRelic] during a load test.</span></span> <span data-ttu-id="0b9b5-180">在本例中，执行唯一的 HTTP GET 操作是 `Person/GetAsync`。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-180">In this case, the only HTTP GET operation performed is `Person/GetAsync`.</span></span> <span data-ttu-id="0b9b5-181">但在实时生产环境中，了解每个请求的相对执行频率可以深入分析应该缓存哪些资源。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-181">But in a live production environment, knowing the relative frequency that each request is performed can give you insight into which resources should be cached.</span></span>

![显示 CachingDemo 应用程序服务器请求的 New Relic][NewRelic-server-requests]

<span data-ttu-id="0b9b5-183">如果需要更深入的分析，可以在测试环境（而不是生产系统）中使用探查器捕获低级性能数据。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-183">If you need a deeper analysis, you can use a profiler to capture low-level performance data in a test environment (not the production system).</span></span> <span data-ttu-id="0b9b5-184">查看 I/O 请求速率、内存使用率和 CPU 利用率等指标。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-184">Look at metrics such as I/O request rates, memory usage, and CPU utilization.</span></span> <span data-ttu-id="0b9b5-185">这些指标可以显示向数据存储或服务发出的大量请求，或者执行相同计算的重复处理。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-185">These metrics may show a large number of requests to a data store or service, or repeated processing that performs the same calculation.</span></span> 

### <a name="load-test-the-application"></a><span data-ttu-id="0b9b5-186">对应用程序进行负载测试</span><span class="sxs-lookup"><span data-stu-id="0b9b5-186">Load test the application</span></span>

<span data-ttu-id="0b9b5-187">下图显示了对示例应用程序执行负载测试的结果。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-187">The following graph shows the results of load testing the sample application.</span></span> <span data-ttu-id="0b9b5-188">该负载测试模拟一个包含多达 800 个用户的阶跃负载，这些用户执行一系列典型操作。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-188">The load test simulates a step load of up to 800 users performing a typical series of operations.</span></span> 

![针对未缓存方案执行的性能负载测试结果][Performance-Load-Test-Results-Uncached]

<span data-ttu-id="0b9b5-190">每秒成功执行的测试数达到平稳状态，因此，其他请求的速度减慢。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-190">The number of successful tests performed each second reaches a plateau, and additional requests are slowed as a result.</span></span> <span data-ttu-id="0b9b5-191">平均测试时间随着工作负荷的增大而平稳增加。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-191">The average test time steadily increases with the workload.</span></span> <span data-ttu-id="0b9b5-192">达到用户负载峰值后，响应时间趋向平稳。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-192">The response time levels off once the user load peaks.</span></span>

### <a name="examine-data-access-statistics"></a><span data-ttu-id="0b9b5-193">检查数据访问统计信息</span><span class="sxs-lookup"><span data-stu-id="0b9b5-193">Examine data access statistics</span></span>

<span data-ttu-id="0b9b5-194">数据存储提供有用的数据访问统计信息和其他信息，例如，哪些查询的重复最频繁。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-194">Data access statistics and other information provided by a data store can give useful information, such as which queries are repeated most frequently.</span></span> <span data-ttu-id="0b9b5-195">例如，在 Microsoft SQL Server 中，`sys.dm_exec_query_stats` 管理视图提供最近执行的查询的统计信息。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-195">For example, in Microsoft SQL Server, the `sys.dm_exec_query_stats` management view has statistical information for recently executed queries.</span></span> <span data-ttu-id="0b9b5-196">`sys.dm_exec-query_plan` 视图中提供每个查询的文本。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-196">The text for each query is available in the `sys.dm_exec-query_plan` view.</span></span> <span data-ttu-id="0b9b5-197">可以使用 SQL Server Management Studio 等工具运行以下 SQL 查询，并确定查询的执行频率。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-197">You can use a tool such as SQL Server Management Studio to run the following SQL query and determine how frequently queries are performed.</span></span>

```SQL
SELECT UseCounts, Text, Query_Plan
FROM sys.dm_exec_cached_plans
CROSS APPLY sys.dm_exec_sql_text(plan_handle)
CROSS APPLY sys.dm_exec_query_plan(plan_handle)
```

<span data-ttu-id="0b9b5-198">结果中的 `UseCount` 列指示每个查询的运行频率。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-198">The `UseCount` column in the results indicates how frequently each query is run.</span></span> <span data-ttu-id="0b9b5-199">下图显示第三个查询已运行 250,000 次以上，远远超过其他任何查询。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-199">The following image shows that the third query was run more than 250,000 times, significantly more than any other query.</span></span>

![在 SQL Server 管理服务器中查询动态管理视图后的结果][Dynamic-Management-Views]

<span data-ttu-id="0b9b5-201">下面是导致发出过多数据库请求的 SQL 查询：</span><span class="sxs-lookup"><span data-stu-id="0b9b5-201">Here is the SQL query that is causing so many database requests:</span></span> 

```SQL
(@p__linq__0 int)SELECT TOP (2)
[Extent1].[BusinessEntityId] AS [BusinessEntityId],
[Extent1].[FirstName] AS [FirstName],
[Extent1].[LastName] AS [LastName]
FROM [Person].[Person] AS [Extent1]
WHERE [Extent1].[BusinessEntityId] = @p__linq__0
```

<span data-ttu-id="0b9b5-202">这是实体框架在前面所示的 `GetByIdAsync` 方法中生成的查询。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-202">This is the query that Entity Framework generates in `GetByIdAsync` method shown earlier.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="0b9b5-203">实施解决方案并验证结果</span><span class="sxs-lookup"><span data-stu-id="0b9b5-203">Implement the solution and verify the result</span></span>

<span data-ttu-id="0b9b5-204">合并缓存后，请重复负载测试，并将结果与前面未使用缓存执行的负载测试进行比较。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-204">After you incorporate a cache, repeat the load tests and compare the results to the earlier load tests without a cache.</span></span> <span data-ttu-id="0b9b5-205">下面是将缓存添加到示例应用程序后的负载测试结果。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-205">Here are the load test results after adding a cache to the sample application.</span></span>

![针对缓存方案执行的性能负载测试结果][Performance-Load-Test-Results-Cached]

<span data-ttu-id="0b9b5-207">成功测试数量仍保持平稳状态，但用户负载更高。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-207">The volume of successful tests still reaches a plateau, but at a higher user load.</span></span> <span data-ttu-id="0b9b5-208">在承受此负载时，请求速率明显高于前面的测试。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-208">The request rate at this load is significantly higher than earlier.</span></span> <span data-ttu-id="0b9b5-209">平均测试时间仍然随着负载的增大而增加，但最大响应时间为 0.05 毫秒，而前面的测试中为 1 毫秒 &mdash; 有了 20&times; 倍的改善。</span><span class="sxs-lookup"><span data-stu-id="0b9b5-209">Average test time still increases with load, but the maximum response time is 0.05 ms, compared with 1ms earlier &mdash; a 20&times; improvement.</span></span> 

## <a name="related-resources"></a><span data-ttu-id="0b9b5-210">相关资源</span><span class="sxs-lookup"><span data-stu-id="0b9b5-210">Related resources</span></span>

- <span data-ttu-id="0b9b5-211">[有关 API 实现的最佳做法][api-implementation]</span><span class="sxs-lookup"><span data-stu-id="0b9b5-211">[API implementation best practices][api-implementation]</span></span>
- <span data-ttu-id="0b9b5-212">[缓存端模式][cache-aside-pattern]</span><span class="sxs-lookup"><span data-stu-id="0b9b5-212">[Cache-Aside Pattern][cache-aside-pattern]</span></span>
- <span data-ttu-id="0b9b5-213">[有关缓存的最佳做法][caching-guidance]</span><span class="sxs-lookup"><span data-stu-id="0b9b5-213">[Caching best practices][caching-guidance]</span></span>
- <span data-ttu-id="0b9b5-214">[断路器模式][circuit-breaker]</span><span class="sxs-lookup"><span data-stu-id="0b9b5-214">[Circuit Breaker pattern][circuit-breaker]</span></span>

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/NoCaching
[cache-aside-pattern]: /azure/architecture/patterns/cache-aside
[caching-guidance]: ../../best-practices/caching.md
[circuit-breaker]: ../../patterns/circuit-breaker.md
[api-implementation]: ../../best-practices/api-implementation.md#optimizing-client-side-data-access
[NewRelic]: http://newrelic.com/azure
[NewRelic-server-requests]: _images/New-Relic.jpg
[Performance-Load-Test-Results-Uncached]:_images/InitialLoadTestResults.jpg
[Dynamic-Management-Views]: _images/SQLServerManagementStudio.jpg
[Performance-Load-Test-Results-Cached]: _images/CachedLoadTestResults.jpg