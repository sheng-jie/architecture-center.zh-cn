---
title: 缓存端
description: 将数据按需从数据存储加载到缓存中
keywords: 设计模式
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- performance-scalability
ms.openlocfilehash: 1536a33884c9c9faa1e3702c951067249e691bf8
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/23/2018
---
# <a name="cache-aside-pattern"></a><span data-ttu-id="58da6-104">缓存端模式</span><span class="sxs-lookup"><span data-stu-id="58da6-104">Cache-Aside pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="58da6-105">将数据按需从数据存储加载到缓存中。</span><span class="sxs-lookup"><span data-stu-id="58da6-105">Load data on demand into a cache from a data store.</span></span> <span data-ttu-id="58da6-106">这可提升性能，并且有助于在缓存中保存的数据与基础数据存储中的数据之间保持一致性。</span><span class="sxs-lookup"><span data-stu-id="58da6-106">This can improve performance and also helps to maintain consistency between data held in the cache and data in the underlying data store.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="58da6-107">上下文和问题</span><span class="sxs-lookup"><span data-stu-id="58da6-107">Context and problem</span></span>

<span data-ttu-id="58da6-108">应用程序使用缓存来改善对数据存储中保存的信息的重复访问。</span><span class="sxs-lookup"><span data-stu-id="58da6-108">Applications use a cache to improve repeated access to information held in a data store.</span></span> <span data-ttu-id="58da6-109">但是，期望缓存的数据始终与数据存储中的数据完全一致是不切实际的。</span><span class="sxs-lookup"><span data-stu-id="58da6-109">However, it's impractical to expect that cached data will always be completely consistent with the data in the data store.</span></span> <span data-ttu-id="58da6-110">应用程序应实施策略，帮助尽可能确保缓存中的数据为最新状态，并且检测和处理当缓存中的数据过期时所出现的情况。</span><span class="sxs-lookup"><span data-stu-id="58da6-110">Applications should implement a strategy that helps to ensure that the data in the cache is as up-to-date as possible, but can also detect and handle situations that arise when the data in the cache has become stale.</span></span>

## <a name="solution"></a><span data-ttu-id="58da6-111">解决方案</span><span class="sxs-lookup"><span data-stu-id="58da6-111">Solution</span></span>

<span data-ttu-id="58da6-112">许多商业缓存系统提供直读和直写/后写操作。</span><span class="sxs-lookup"><span data-stu-id="58da6-112">Many commercial caching systems provide read-through and write-through/write-behind operations.</span></span> <span data-ttu-id="58da6-113">在这些系统中，应用程序通过引用缓存来检索数据。</span><span class="sxs-lookup"><span data-stu-id="58da6-113">In these systems, an application retrieves data by referencing the cache.</span></span> <span data-ttu-id="58da6-114">如果数据不在缓存中，则将从数据存储中检索数据并将其添加到缓存。</span><span class="sxs-lookup"><span data-stu-id="58da6-114">If the data isn't in the cache, it's retrieved from the data store and added to the cache.</span></span> <span data-ttu-id="58da6-115">对缓存中保存的数据进行的任何修改还会自动写回到数据存储。</span><span class="sxs-lookup"><span data-stu-id="58da6-115">Any modifications to data held in the cache are automatically written back to the data store as well.</span></span>

<span data-ttu-id="58da6-116">对于不提供此功能的缓存，使用缓存的应用程序将负责保存数据。</span><span class="sxs-lookup"><span data-stu-id="58da6-116">For caches that don't provide this functionality, it's the responsibility of the applications that use the cache to maintain the data.</span></span>

<span data-ttu-id="58da6-117">通过实施缓存端策略，应用程序可以模拟直读缓存的功能。</span><span class="sxs-lookup"><span data-stu-id="58da6-117">An application can emulate the functionality of read-through caching by implementing the cache-aside strategy.</span></span> <span data-ttu-id="58da6-118">此策略可按需将数据加载到缓存。</span><span class="sxs-lookup"><span data-stu-id="58da6-118">This strategy loads data into the cache on demand.</span></span> <span data-ttu-id="58da6-119">下图演示使用缓存端模式在缓存中存储数据。</span><span class="sxs-lookup"><span data-stu-id="58da6-119">The figure illustrates using the Cache-Aside pattern to store data in the cache.</span></span>

![使用缓存端模式在缓存中存储数据](./_images/cache-aside-diagram.png)


<span data-ttu-id="58da6-121">如果应用程序更新了信息，则可按照直写策略操作，方法是修改数据存储和使缓存中的相应项无效。</span><span class="sxs-lookup"><span data-stu-id="58da6-121">If an application updates information, it can follow the write-through strategy by making the modification to the data store, and by invalidating the corresponding item in the cache.</span></span>

<span data-ttu-id="58da6-122">如果下一步需要该项，使用缓存端策略将导致可从数据存储检索更新后的数据，并将其添加回缓存。</span><span class="sxs-lookup"><span data-stu-id="58da6-122">When the item is next required, using the cache-aside strategy will cause the updated data to be retrieved from the data store and added back into the cache.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="58da6-123">问题和注意事项</span><span class="sxs-lookup"><span data-stu-id="58da6-123">Issues and considerations</span></span>

<span data-ttu-id="58da6-124">在决定如何实现此模式时，请考虑以下几点：</span><span class="sxs-lookup"><span data-stu-id="58da6-124">Consider the following points when deciding how to implement this pattern:</span></span> 

<span data-ttu-id="58da6-125">**已缓存数据的生存期**。</span><span class="sxs-lookup"><span data-stu-id="58da6-125">**Lifetime of cached data**.</span></span> <span data-ttu-id="58da6-126">许多缓存实施过期策略，如果未在指定期间访问数据，则数据将失效并从缓存中删除。</span><span class="sxs-lookup"><span data-stu-id="58da6-126">Many caches implement an expiration policy that invalidates data and removes it from the cache if it's not accessed for a specified period.</span></span> <span data-ttu-id="58da6-127">若要使缓存端有效，请确保过期策略与使用数据的应用程序的访问模式相匹配。</span><span class="sxs-lookup"><span data-stu-id="58da6-127">For cache-aside to be effective, ensure that the expiration policy matches the pattern of access for applications that use the data.</span></span> <span data-ttu-id="58da6-128">过期期限不宜太短，因为可能导致应用程序不断从数据存储检索数据并将其添加到缓存。</span><span class="sxs-lookup"><span data-stu-id="58da6-128">Don't make the expiration period too short because this can cause applications to continually retrieve data from the data store and add it to the cache.</span></span> <span data-ttu-id="58da6-129">同样，过期期限不宜太长，否则缓存数据可能会过期。</span><span class="sxs-lookup"><span data-stu-id="58da6-129">Similarly, don't make the expiration period so long that the cached data is likely to become stale.</span></span> <span data-ttu-id="58da6-130">请记住，缓存最适用于相对静态的数据或经常读取的数据。</span><span class="sxs-lookup"><span data-stu-id="58da6-130">Remember that caching is most effective for relatively static data, or data that is read frequently.</span></span>

<span data-ttu-id="58da6-131">**逐出数据**。</span><span class="sxs-lookup"><span data-stu-id="58da6-131">**Evicting data**.</span></span> <span data-ttu-id="58da6-132">与数据起源的数据存储相比，大多数缓存的大小有限，并且在必要时逐出数据。</span><span class="sxs-lookup"><span data-stu-id="58da6-132">Most caches have a limited size compared to the data store where the data originates, and they'll evict data if necessary.</span></span> <span data-ttu-id="58da6-133">大多数缓存采用最近最少使用策略来选择要逐出的项，但这也可自定义。</span><span class="sxs-lookup"><span data-stu-id="58da6-133">Most caches adopt a least-recently-used policy for selecting items to evict, but this might be customizable.</span></span> <span data-ttu-id="58da6-134">配置缓存的全局过期属性和其他属性，以及每个已缓存项的过期属性，以确保缓存具有成本效益。</span><span class="sxs-lookup"><span data-stu-id="58da6-134">Configure the global expiration property and other properties of the cache, and the expiration property of each cached item, to ensure that the cache is cost effective.</span></span> <span data-ttu-id="58da6-135">将全局逐出策略应用于缓存中的每个项并不总是适用。</span><span class="sxs-lookup"><span data-stu-id="58da6-135">It isn't always appropriate to apply a global eviction policy to every item in the cache.</span></span> <span data-ttu-id="58da6-136">例如，如果从数据存储检索某个已缓存项的代价很高，则将该项保留在缓存中是有益的，尽管访问更频繁，但减少了成本高的项。</span><span class="sxs-lookup"><span data-stu-id="58da6-136">For example, if a cached item is very expensive to retrieve from the data store, it can be beneficial to keep this item in the cache at the expense of more frequently accessed but less costly items.</span></span>

<span data-ttu-id="58da6-137">**填充缓存**。</span><span class="sxs-lookup"><span data-stu-id="58da6-137">**Priming the cache**.</span></span> <span data-ttu-id="58da6-138">很多解决方案使用可能用作应用程序启动处理一部分的数据来预填充缓存。</span><span class="sxs-lookup"><span data-stu-id="58da6-138">Many solutions prepopulate the cache with the data that an application is likely to need as part of the startup processing.</span></span> <span data-ttu-id="58da6-139">如果一些数据已过期或已逐出，则缓存端模式仍然十分有用。</span><span class="sxs-lookup"><span data-stu-id="58da6-139">The Cache-Aside pattern can still be useful if some of this data expires or is evicted.</span></span>

<span data-ttu-id="58da6-140">**一致性**。</span><span class="sxs-lookup"><span data-stu-id="58da6-140">**Consistency**.</span></span> <span data-ttu-id="58da6-141">实现缓存端模式并不能保证数据存储与缓存之间的一致性。</span><span class="sxs-lookup"><span data-stu-id="58da6-141">Implementing the Cache-Aside pattern doesn't guarantee consistency between the data store and the cache.</span></span> <span data-ttu-id="58da6-142">外部进程可随时更改数据存储中的项，并且在下次加载该项之前，此更改不会反映在缓存中。</span><span class="sxs-lookup"><span data-stu-id="58da6-142">An item in the data store can be changed at any time by an external process, and this change might not be reflected in the cache until the next time the item is loaded.</span></span> <span data-ttu-id="58da6-143">在跨数据存储复制数据的系统中，如果经常发生同步，则此问题可能会变得严重。</span><span class="sxs-lookup"><span data-stu-id="58da6-143">In a system that replicates data across data stores, this problem can become serious if synchronization occurs frequently.</span></span>

<span data-ttu-id="58da6-144">**本地（内存中）缓存**。</span><span class="sxs-lookup"><span data-stu-id="58da6-144">**Local (in-memory) caching**.</span></span> <span data-ttu-id="58da6-145">缓存可能是应用程序实例的本地缓存，并且存储在内存中。</span><span class="sxs-lookup"><span data-stu-id="58da6-145">A cache could be local to an application instance and stored in-memory.</span></span> <span data-ttu-id="58da6-146">如果应用程序反复访问相同的数据，则缓存端在此环境中很有用。</span><span class="sxs-lookup"><span data-stu-id="58da6-146">Cache-aside can be useful in this environment if an application repeatedly accesses the same data.</span></span> <span data-ttu-id="58da6-147">但是，因为本地缓存是专用的，因此不同的应用程序实例可各自拥有相同缓存数据的副本。</span><span class="sxs-lookup"><span data-stu-id="58da6-147">However, a local cache is private and so different application instances could each have a copy of the same cached data.</span></span> <span data-ttu-id="58da6-148">缓存之间的此数据可快速变得不一致，因此有必要使专用缓存中保存的数据过期并更加频繁地刷新该数据。</span><span class="sxs-lookup"><span data-stu-id="58da6-148">This data could quickly become inconsistent between caches, so it might be necessary to expire data held in a private cache and refresh it more frequently.</span></span> <span data-ttu-id="58da6-149">在这些情况下，请考虑研究共享或分布式缓存机制的使用。</span><span class="sxs-lookup"><span data-stu-id="58da6-149">In these scenarios, consider investigating the use of a shared or a distributed caching mechanism.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="58da6-150">何时使用此模式</span><span class="sxs-lookup"><span data-stu-id="58da6-150">When to use this pattern</span></span>

<span data-ttu-id="58da6-151">在以下情况下使用此模式：</span><span class="sxs-lookup"><span data-stu-id="58da6-151">Use this pattern when:</span></span>

- <span data-ttu-id="58da6-152">缓存不提供本机直读和直写操作。</span><span class="sxs-lookup"><span data-stu-id="58da6-152">A cache doesn't provide native read-through and write-through operations.</span></span>
- <span data-ttu-id="58da6-153">资源需求不可预知。</span><span class="sxs-lookup"><span data-stu-id="58da6-153">Resource demand is unpredictable.</span></span> <span data-ttu-id="58da6-154">此模式可使应用程序按需加载数据。</span><span class="sxs-lookup"><span data-stu-id="58da6-154">This pattern enables applications to load data on demand.</span></span> <span data-ttu-id="58da6-155">无法提前假设应用程序将需要哪些数据。</span><span class="sxs-lookup"><span data-stu-id="58da6-155">It makes no assumptions about which data an application will require in advance.</span></span>

<span data-ttu-id="58da6-156">此模式可能不适用：</span><span class="sxs-lookup"><span data-stu-id="58da6-156">This pattern might not be suitable:</span></span>

- <span data-ttu-id="58da6-157">缓存的数据集为静态。</span><span class="sxs-lookup"><span data-stu-id="58da6-157">When the cached data set is static.</span></span> <span data-ttu-id="58da6-158">如果数据可融入可用的缓存空间，则在启动时用数据填充缓存，并应用可防止数据过期的策略。</span><span class="sxs-lookup"><span data-stu-id="58da6-158">If the data will fit into the available cache space, prime the cache with the data on startup and apply a policy that prevents the data from expiring.</span></span>
- <span data-ttu-id="58da6-159">在 Web 场中托管的 Web 应用程序中的缓存会话状态信息。</span><span class="sxs-lookup"><span data-stu-id="58da6-159">For caching session state information in a web application hosted in a web farm.</span></span> <span data-ttu-id="58da6-160">在此环境中，应避免引入基于客户端服务器相关性的依赖关系。</span><span class="sxs-lookup"><span data-stu-id="58da6-160">In this environment, you should avoid introducing dependencies based on client-server affinity.</span></span>

## <a name="example"></a><span data-ttu-id="58da6-161">示例</span><span class="sxs-lookup"><span data-stu-id="58da6-161">Example</span></span>

<span data-ttu-id="58da6-162">在 Microsoft Azure 中，可以使用 Azure Redis 缓存来创建可由应用程序的多个实例共享的分布式缓存。</span><span class="sxs-lookup"><span data-stu-id="58da6-162">In Microsoft Azure you can use Azure Redis Cache to create a distributed cache that can be shared by multiple instances of an application.</span></span> 

<span data-ttu-id="58da6-163">若要连接到 Azure Redis 缓存实例，请调用静态 `Connect` 方法并传入连接字符串。</span><span class="sxs-lookup"><span data-stu-id="58da6-163">To connect to an Azure Redis Cache instance, call the static `Connect` method and pass in the connection string.</span></span> <span data-ttu-id="58da6-164">该方法返回表示连接的 `ConnectionMultiplexer`。</span><span class="sxs-lookup"><span data-stu-id="58da6-164">The method returns a `ConnectionMultiplexer` that represents the connection.</span></span> <span data-ttu-id="58da6-165">共享应用程序中的 `ConnectionMultiplexer` 实例的一个方法是，拥有返回连接示例的静态属性（与下列示例类似）。</span><span class="sxs-lookup"><span data-stu-id="58da6-165">One approach to sharing a `ConnectionMultiplexer` instance in your application is to have a static property that returns a connected instance, similar to the following example.</span></span> <span data-ttu-id="58da6-166">此方法是一种线程安全方法，仅初始化连接的一个实例。</span><span class="sxs-lookup"><span data-stu-id="58da6-166">This approach provides a thread-safe way to initialize only a single connected instance.</span></span>

```csharp
private static ConnectionMultiplexer Connection;

// Redis Connection string info
private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
{
    string cacheConnection = ConfigurationManager.AppSettings["CacheConnection"].ToString();
    return ConnectionMultiplexer.Connect(cacheConnection);
});

public static ConnectionMultiplexer Connection => lazyConnection.Value;
```

<span data-ttu-id="58da6-167">以下代码示例中的 `GetMyEntityAsync` 方法演示对基于 Azure Redis 缓存的缓存端模式的实现。</span><span class="sxs-lookup"><span data-stu-id="58da6-167">The `GetMyEntityAsync` method in the following code example shows an implementation of the Cache-Aside pattern based on Azure Redis Cache.</span></span> <span data-ttu-id="58da6-168">此方法使用直读方法从缓存检索对象。</span><span class="sxs-lookup"><span data-stu-id="58da6-168">This method retrieves an object from the cache using the read-though approach.</span></span>

<span data-ttu-id="58da6-169">通过将整数 ID 用作密钥来识别对象。</span><span class="sxs-lookup"><span data-stu-id="58da6-169">An object is identified by using an integer ID as the key.</span></span> <span data-ttu-id="58da6-170">`GetMyEntityAsync` 方法尝试使用此密钥从缓存检索项。</span><span class="sxs-lookup"><span data-stu-id="58da6-170">The `GetMyEntityAsync` method tries to retrieve an item with this key from the cache.</span></span> <span data-ttu-id="58da6-171">如果找到匹配项，则它将返回。</span><span class="sxs-lookup"><span data-stu-id="58da6-171">If a matching item is found, it's returned.</span></span> <span data-ttu-id="58da6-172">如果缓存中没有匹配项，`GetMyEntityAsync` 方法将从数据存储检索对象、将其添加到缓存中，然后将其返回。</span><span class="sxs-lookup"><span data-stu-id="58da6-172">If there's no match in the cache, the `GetMyEntityAsync` method retrieves the object from a data store, adds it to the cache, and then returns it.</span></span> <span data-ttu-id="58da6-173">从数据存储实际读取数据的代码取决于数据存储，所以它未在此处显示。</span><span class="sxs-lookup"><span data-stu-id="58da6-173">The code that actually reads the data from the data store is not shown here, because it depends on the data store.</span></span> <span data-ttu-id="58da6-174">请注意，缓存的项已配置为过期，以防止在其他位置对其进行更新后它会变得陈旧。</span><span class="sxs-lookup"><span data-stu-id="58da6-174">Note that the cached item is configured to expire to prevent it from becoming stale if it's updated elsewhere.</span></span>


```csharp
// Set five minute expiration as a default
private const double DefaultExpirationTimeInMinutes = 5.0;

public async Task<MyEntity> GetMyEntityAsync(int id)
{
  // Define a unique key for this method and its parameters.
  var key = $"MyEntity:{id}";
  var cache = Connection.GetDatabase();
  
  // Try to get the entity from the cache.
  var json = await cache.StringGetAsync(key).ConfigureAwait(false);
  var value = string.IsNullOrWhiteSpace(json) 
                ? default(MyEntity) 
                : JsonConvert.DeserializeObject<MyEntity>(json);
  
  if (value == null) // Cache miss
  {
    // If there's a cache miss, get the entity from the original store and cache it.
    // Code has been omitted because it's data store dependent.  
    value = ...;

    // Avoid caching a null value.
    if (value != null)
    {
      // Put the item in the cache with a custom expiration time that 
      // depends on how critical it is to have stale data.
      await cache.StringSetAsync(key, JsonConvert.SerializeObject(value)).ConfigureAwait(false);
      await cache.KeyExpireAsync(key, TimeSpan.FromMinutes(DefaultExpirationTimeInMinutes)).ConfigureAwait(false);
    }
  }

  return value;
}
```

>  <span data-ttu-id="58da6-175">此示例使用 Azure Redis 缓存 API 访问存储并从缓存中检索信息。</span><span class="sxs-lookup"><span data-stu-id="58da6-175">The examples use the Azure Redis Cache API to access the store and retrieve information from the cache.</span></span> <span data-ttu-id="58da6-176">有关详细信息，请参阅[使用 Microsoft Azure Redis 缓存](https://docs.microsoft.com/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache)和[如何使用 Redis 缓存创建 Web 应用](https://docs.microsoft.com/azure/redis-cache/cache-web-app-howto)</span><span class="sxs-lookup"><span data-stu-id="58da6-176">For more information, see [Using Microsoft Azure Redis Cache](https://docs.microsoft.com/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache) and [How to create a Web App with Redis Cache](https://docs.microsoft.com/azure/redis-cache/cache-web-app-howto)</span></span>

<span data-ttu-id="58da6-177">如下所示，`UpdateEntityAsync` 方法演示如何在应用程序更改值时使缓存中的对象无效。</span><span class="sxs-lookup"><span data-stu-id="58da6-177">The `UpdateEntityAsync` method shown below demonstrates how to invalidate an object in the cache when the value is changed by the application.</span></span> <span data-ttu-id="58da6-178">代码更新原始数据存储，然后从缓存中删除缓存的项。</span><span class="sxs-lookup"><span data-stu-id="58da6-178">The code updates the original data store and then removes the cached item from the cache.</span></span>

```csharp
public async Task UpdateEntityAsync(MyEntity entity)
{
    // Update the object in the original data store.
    await this.store.UpdateEntityAsync(entity).ConfigureAwait(false); 

    // Invalidate the current cache object.
    var cache = Connection.GetDatabase();
    var id = entity.Id;
    var key = $"MyEntity:{id}"; // The key for the cached object.
    await cache.KeyDeleteAsync(key).ConfigureAwait(false); // Delete this key from the cache.
}
```

> [!NOTE]
> <span data-ttu-id="58da6-179">该步骤的顺序非常重要。</span><span class="sxs-lookup"><span data-stu-id="58da6-179">The order of the steps is important.</span></span> <span data-ttu-id="58da6-180">请先更新数据存储，然后再从缓存中删除项。</span><span class="sxs-lookup"><span data-stu-id="58da6-180">Update the data store *before* removing the item from the cache.</span></span> <span data-ttu-id="58da6-181">如果首先删除缓存的项，在短时间内，客户端可能会在数据存储更新前提取该项。</span><span class="sxs-lookup"><span data-stu-id="58da6-181">If you remove the cached item first, there is a small window of time when a client might fetch the item before the data store is updated.</span></span> <span data-ttu-id="58da6-182">这将造成缓存失误（因为从缓存中删除了项），导致要从数据存储提取该项的早期版本并将其添加回缓存。</span><span class="sxs-lookup"><span data-stu-id="58da6-182">That will result in a cache miss (because the item was removed from the cache), causing the earlier version of the item to be fetched from the data store and added back into the cache.</span></span> <span data-ttu-id="58da6-183">造成缓存数据陈旧。</span><span class="sxs-lookup"><span data-stu-id="58da6-183">The result will be stale cache data.</span></span>


## <a name="related-guidance"></a><span data-ttu-id="58da6-184">相关指南</span><span class="sxs-lookup"><span data-stu-id="58da6-184">Related guidance</span></span> 

<span data-ttu-id="58da6-185">实现此模式时，以下信息可能相关：</span><span class="sxs-lookup"><span data-stu-id="58da6-185">The following information may be relevant when implementing this pattern:</span></span>

- <span data-ttu-id="58da6-186">[Caching Guidance](https://docs.microsoft.com/azure/architecture/best-practices/caching)（缓存指南）。</span><span class="sxs-lookup"><span data-stu-id="58da6-186">[Caching Guidance](https://docs.microsoft.com/azure/architecture/best-practices/caching).</span></span> <span data-ttu-id="58da6-187">提供有关如何在云解决方案中缓存数据的其他信息，以及实现缓存时应考虑的问题。</span><span class="sxs-lookup"><span data-stu-id="58da6-187">Provides additional information on how you can cache data in a cloud solution, and the issues that you should consider when you implement a cache.</span></span>

- <span data-ttu-id="58da6-188">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx)（数据一致性入门）。</span><span class="sxs-lookup"><span data-stu-id="58da6-188">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span> <span data-ttu-id="58da6-189">云应用程序通常使用遍布数据存储的数据。</span><span class="sxs-lookup"><span data-stu-id="58da6-189">Cloud applications typically use data that's spread across data stores.</span></span> <span data-ttu-id="58da6-190">系统的一个重要方面是在此环境中管理和维护数据一致性，特别是可能出现的并发性和可用性问题。</span><span class="sxs-lookup"><span data-stu-id="58da6-190">Managing and maintaining data consistency in this environment is a critical aspect of the system, particularly the concurrency and availability issues that can arise.</span></span> <span data-ttu-id="58da6-191">此入门介绍了有关跨分布式数据的一致性问题，并总结了应用程序实现最终一致性以维持数据的可用性的方法。</span><span class="sxs-lookup"><span data-stu-id="58da6-191">This primer describes issues about consistency across distributed data, and summarizes how an application can implement eventual consistency to maintain the availability of data.</span></span>
