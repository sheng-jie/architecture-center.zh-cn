---
title: "缓存指南"
description: "有关配置缓存以提高性能和伸缩性的指南。"
author: dragon119
ms.date: 05/24/2017
pnp.series.title: Best Practices
ms.openlocfilehash: f8bc25ef10847e8308e830b745e87a176438d200
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="caching"></a><span data-ttu-id="a00ee-103">缓存</span><span class="sxs-lookup"><span data-stu-id="a00ee-103">Caching</span></span>

<span data-ttu-id="a00ee-104">缓存是一种常见的技术，目标是提高系统的性能和伸缩性。</span><span class="sxs-lookup"><span data-stu-id="a00ee-104">Caching is a common technique that aims to improve the performance and scalability of a system.</span></span> <span data-ttu-id="a00ee-105">为此，它会暂时会经常访问的数据复制到位置靠近应用程序的快速存储。</span><span class="sxs-lookup"><span data-stu-id="a00ee-105">It does this by temporarily copying frequently accessed data to fast storage that's located close to the application.</span></span> <span data-ttu-id="a00ee-106">如果这种快速数据存储比原始源更靠近应用程序，则缓存可以通过更快速提供数据，大幅改善客户端应用程序的响应时间。</span><span class="sxs-lookup"><span data-stu-id="a00ee-106">If this fast data storage is located closer to the application than the original source, then caching can significantly improve response times for client applications by serving data more quickly.</span></span>

<span data-ttu-id="a00ee-107">如果客户端实例重复读取同一数据，则缓存是最有效的方式，尤其是原始数据存储存在以下情况时：</span><span class="sxs-lookup"><span data-stu-id="a00ee-107">Caching is most effective when a client instance repeatedly reads the same data, especially if all the following conditions apply to the original data store:</span></span>

* <span data-ttu-id="a00ee-108">保持相对静态。</span><span class="sxs-lookup"><span data-stu-id="a00ee-108">It remains relatively static.</span></span>
* <span data-ttu-id="a00ee-109">相对于缓存速度而言较慢时。</span><span class="sxs-lookup"><span data-stu-id="a00ee-109">It's slow compared to the speed of the cache.</span></span>
* <span data-ttu-id="a00ee-110">受限于激烈的资源争用。</span><span class="sxs-lookup"><span data-stu-id="a00ee-110">It's subject to a high level of contention.</span></span>
* <span data-ttu-id="a00ee-111">由于距离遥远，网络延迟会造成访问速度缓慢。</span><span class="sxs-lookup"><span data-stu-id="a00ee-111">It's far away when network latency can cause access to be slow.</span></span>

## <a name="caching-in-distributed-applications"></a><span data-ttu-id="a00ee-112">分布式应用程序中的缓存</span><span class="sxs-lookup"><span data-stu-id="a00ee-112">Caching in distributed applications</span></span>
<span data-ttu-id="a00ee-113">在缓存数据时，分布式应用程序通常会实施以下一种或两种策略：</span><span class="sxs-lookup"><span data-stu-id="a00ee-113">Distributed applications typically implement either or both of the following strategies when caching data:</span></span>

* <span data-ttu-id="a00ee-114">使用专用缓存，其中的数据保存在运行应用程序或服务实例的计算机本地。</span><span class="sxs-lookup"><span data-stu-id="a00ee-114">Using a private cache, where data is held locally on the computer that's running an instance of an application or service.</span></span>
* <span data-ttu-id="a00ee-115">使用共享缓存，充当可由多个进程和/或计算机访问的公用源。</span><span class="sxs-lookup"><span data-stu-id="a00ee-115">Using a shared cache, serving as a common source which can be accessed by multiple processes and/or machines.</span></span>

<span data-ttu-id="a00ee-116">在这两种情况下，缓存可在客户端和/或服务器端执行。</span><span class="sxs-lookup"><span data-stu-id="a00ee-116">In both cases, caching can be performed client-side and/or server-side.</span></span> <span data-ttu-id="a00ee-117">通过为系统提供用户界面（例如 Web 浏览器或桌面应用程序）的进程来实现客户端缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-117">Client-side caching is done by the process that provides the user interface for a system, such as a web browser or desktop application.</span></span>
<span data-ttu-id="a00ee-118">通过远程运行的提供业务服务的进程来实现服务器端缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-118">Server-side caching is done by the process that provides the business services that are running remotely.</span></span>

### <a name="private-caching"></a><span data-ttu-id="a00ee-119">专用缓存</span><span class="sxs-lookup"><span data-stu-id="a00ee-119">Private caching</span></span>
<span data-ttu-id="a00ee-120">最基本类型的缓存是内存中存储。</span><span class="sxs-lookup"><span data-stu-id="a00ee-120">The most basic type of cache is an in-memory store.</span></span> <span data-ttu-id="a00ee-121">这种缓存保留在单个进程的地址空间中，可由该进程中运行的代码直接访问。</span><span class="sxs-lookup"><span data-stu-id="a00ee-121">It's held in the address space of a single process and accessed directly by the code that runs in that process.</span></span> <span data-ttu-id="a00ee-122">此缓存类型可进行非常快速的访问。</span><span class="sxs-lookup"><span data-stu-id="a00ee-122">This type of cache is very quick to access.</span></span> <span data-ttu-id="a00ee-123">此外，还可提供极其有效的方式用于存储适度的静态数据量，因为缓存大小通常受限于托管进程的计算机上可用内存量。</span><span class="sxs-lookup"><span data-stu-id="a00ee-123">It can also provide an extremely effective means for storing modest amounts of static data, since the size of a cache is typically constrained by the volume of memory that's available on the machine hosting the process.</span></span>

<span data-ttu-id="a00ee-124">如果缓存的信息需要超过内存中实际可用的信息，可以将缓存数据写入本地文件系统。</span><span class="sxs-lookup"><span data-stu-id="a00ee-124">If you need to cache more information than is physically possible in memory, you can write cached data to the local file system.</span></span> <span data-ttu-id="a00ee-125">这比访问保留在内存中的数据更慢，但应该仍比通过网络检索数据更快速且更可靠。</span><span class="sxs-lookup"><span data-stu-id="a00ee-125">This will be slower to access than data that's held in-memory, but should still be faster and more reliable than retrieving data across a network.</span></span>

<span data-ttu-id="a00ee-126">如果有多个并行运行的、使用此模型的应用程序实例，则每个应用程序实例将有自身的独立缓存用于保存自身的数据副本。</span><span class="sxs-lookup"><span data-stu-id="a00ee-126">If you have multiple instances of an application that uses this model running concurrently, each application instance has its own independent cache holding its own copy of the data.</span></span>

<span data-ttu-id="a00ee-127">应该将缓存视为过去某个时间点原始数据的快照。</span><span class="sxs-lookup"><span data-stu-id="a00ee-127">Think of a cache as a snapshot of the original data at some point in the past.</span></span> <span data-ttu-id="a00ee-128">如果此数据不是静态的，则有可能不同的应用程序实例会在其缓存中保存不同版本的数据。</span><span class="sxs-lookup"><span data-stu-id="a00ee-128">If this data is not static, it is likely that different application instances hold different versions of the data in their caches.</span></span> <span data-ttu-id="a00ee-129">因此，这些实例执行的同一查询可能会返回不同的结果，如图 1 所示。</span><span class="sxs-lookup"><span data-stu-id="a00ee-129">Therefore, the same query performed by these instances can return different results, as shown in Figure 1.</span></span>

![在不同的应用程序实例中使用内存中缓存](./images/caching/Figure1.png)

<span data-ttu-id="a00ee-131">*图 1：在不同的应用程序实例中使用内存中缓存*</span><span class="sxs-lookup"><span data-stu-id="a00ee-131">*Figure 1: Using an in-memory cache in different instances of an application*</span></span>

### <a name="shared-caching"></a><span data-ttu-id="a00ee-132">共享缓存</span><span class="sxs-lookup"><span data-stu-id="a00ee-132">Shared caching</span></span>
<span data-ttu-id="a00ee-133">使用共享缓存有助于缓解每个缓存中可能存在不同数据的忧虑，这种情况可能会发生于内存中缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-133">Using a shared cache can help alleviate concerns that data might differ in each cache, which can occur with in-memory caching.</span></span> <span data-ttu-id="a00ee-134">共享缓存可确保不同的应用程序实例看到同一缓存数据视图。</span><span class="sxs-lookup"><span data-stu-id="a00ee-134">Shared caching ensures that different application instances see the same view of cached data.</span></span> <span data-ttu-id="a00ee-135">为此，缓存将定位在不同的位置，通常作为不同服务的一部分托管，如图 2 所示。</span><span class="sxs-lookup"><span data-stu-id="a00ee-135">It does this by locating the cache in a separate location, typically hosted as part of a separate service, as shown in Figure 2.</span></span>

![使用共享缓存](./images/caching/Figure2.png)

<span data-ttu-id="a00ee-137">*图 2：使用共享缓存*</span><span class="sxs-lookup"><span data-stu-id="a00ee-137">*Figure 2: Using a shared cache*</span></span>

<span data-ttu-id="a00ee-138">伸缩性是共享缓存的一个重要优势。</span><span class="sxs-lookup"><span data-stu-id="a00ee-138">An important benefit of the shared caching approach is the scalability it provides.</span></span> <span data-ttu-id="a00ee-139">许多共享缓存服务是使用服务器群集实施的，并以透明方式利用将数据分散到群集的软件。</span><span class="sxs-lookup"><span data-stu-id="a00ee-139">Many shared cache services are implemented by using a cluster of servers, and utilize software that distributes the data across the cluster in a transparent manner.</span></span> <span data-ttu-id="a00ee-140">应用程序实例只会将请求发送到缓存服务。</span><span class="sxs-lookup"><span data-stu-id="a00ee-140">An application instance simply sends a request to the cache service.</span></span>
<span data-ttu-id="a00ee-141">底层基础结构负责确定缓存数据在群集中的位置。</span><span class="sxs-lookup"><span data-stu-id="a00ee-141">The underlying infrastructure is responsible for determining the location of the cached data in the cluster.</span></span> <span data-ttu-id="a00ee-142">可以轻松地通过添加更多服务器来扩展缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-142">You can easily scale the cache by adding more servers.</span></span>

<span data-ttu-id="a00ee-143">共享缓存方法有两个主要缺点：</span><span class="sxs-lookup"><span data-stu-id="a00ee-143">There are two main disadvantages of the shared caching approach:</span></span>

* <span data-ttu-id="a00ee-144">缓存的访问速度较慢，因为它不再保留在每个应用程序实例的本地。</span><span class="sxs-lookup"><span data-stu-id="a00ee-144">The cache is slower to access because it is no longer held locally to each application instance.</span></span>
* <span data-ttu-id="a00ee-145">为了满足实施不同缓存服务的要求，可能会增大解决方案的复杂性。</span><span class="sxs-lookup"><span data-stu-id="a00ee-145">The requirement to implement a separate cache service might add complexity to the solution.</span></span>

## <a name="considerations-for-using-caching"></a><span data-ttu-id="a00ee-146">使用缓存时的注意事项</span><span class="sxs-lookup"><span data-stu-id="a00ee-146">Considerations for using caching</span></span>
<span data-ttu-id="a00ee-147">以下部分更详细地说明了设计和使用缓存时的注意事项。</span><span class="sxs-lookup"><span data-stu-id="a00ee-147">The following sections describe in more detail the considerations for designing and using a cache.</span></span>

### <a name="decide-when-to-cache-data"></a><span data-ttu-id="a00ee-148">确定何时缓存数据</span><span class="sxs-lookup"><span data-stu-id="a00ee-148">Decide when to cache data</span></span>
<span data-ttu-id="a00ee-149">缓存可大幅提高性能、伸缩性和可用性。</span><span class="sxs-lookup"><span data-stu-id="a00ee-149">Caching can dramatically improve performance, scalability, and availability.</span></span> <span data-ttu-id="a00ee-150">当数据越多且需要访问此数据的用户越多，缓存的优点也就越大。</span><span class="sxs-lookup"><span data-stu-id="a00ee-150">The more data that you have and the larger the number of users that need to access this data, the greater the benefits of caching become.</span></span> <span data-ttu-id="a00ee-151">这是因为在原始数据存储中处理大量并发请求时，可以减少相关的延迟和争用。</span><span class="sxs-lookup"><span data-stu-id="a00ee-151">That's because caching reduces the latency and contention that's associated with handling large volumes of concurrent requests in the original data store.</span></span>

<span data-ttu-id="a00ee-152">例如，数据库可以支持有限数目的并发连接。</span><span class="sxs-lookup"><span data-stu-id="a00ee-152">For example, a database might support a limited number of concurrent connections.</span></span> <span data-ttu-id="a00ee-153">但从共享缓存而不是底层数据库检索数据可让客户端应用程序访问此数据，即使当前可用的连接数已用尽。</span><span class="sxs-lookup"><span data-stu-id="a00ee-153">Retrieving data from a shared cache, however, rather than the underlying database, makes it possible for a client application to access this data even if the number of available connections is currently exhausted.</span></span> <span data-ttu-id="a00ee-154">此外，如果数据库变得不可用，客户端应用程序也许可以使用缓存中保存的数据继续运行。</span><span class="sxs-lookup"><span data-stu-id="a00ee-154">Additionally, if the database becomes unavailable, client applications might be able to continue by using the data that's held in the cache.</span></span>

<span data-ttu-id="a00ee-155">考虑使用经常读取但很少修改的缓存（数据读取操作的比例要高于写入操作）。</span><span class="sxs-lookup"><span data-stu-id="a00ee-155">Consider caching data that is read frequently but modified infrequently (for example, data that has a higher proportion of read operations than write operations).</span></span> <span data-ttu-id="a00ee-156">但是，不应将缓存用作关键信息的权威存储。</span><span class="sxs-lookup"><span data-stu-id="a00ee-156">However, we don't recommend that you use the cache as the authoritative store of critical information.</span></span> <span data-ttu-id="a00ee-157">应确保应用程序不可丢失的所有更改始终存储到永久性数据存储中。</span><span class="sxs-lookup"><span data-stu-id="a00ee-157">Instead, ensure that all changes that your application cannot afford to lose are always saved to a persistent data store.</span></span> <span data-ttu-id="a00ee-158">这意味着，当缓存不可用时，应用程序仍可以使用数据存储继续操作，用户不会丢失重要信息。</span><span class="sxs-lookup"><span data-stu-id="a00ee-158">This means that if the cache is unavailable, your application can still continue to operate by using the data store, and you won't lose important information.</span></span>

### <a name="determine-how-to-cache-data-effectively"></a><span data-ttu-id="a00ee-159">确定如何有效缓存数据</span><span class="sxs-lookup"><span data-stu-id="a00ee-159">Determine how to cache data effectively</span></span>
<span data-ttu-id="a00ee-160">有效使用缓存的关键在于确定最适合缓存的数据，以及最适合缓存的时间。</span><span class="sxs-lookup"><span data-stu-id="a00ee-160">The key to using a cache effectively lies in determining the most appropriate data to cache, and caching it at the appropriate time.</span></span> <span data-ttu-id="a00ee-161">数据可能在第一次由应用程序检索时随选添加到缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-161">The data can be added to the cache on demand the first time it is retrieved by an application.</span></span> <span data-ttu-id="a00ee-162">这意味着，应用程序仅需从数据存储检索一次数据，而后续访问可通过使用缓存来满足。</span><span class="sxs-lookup"><span data-stu-id="a00ee-162">This means that the application needs to fetch the data only once from the data store, and that subsequent access can be satisfied by using the cache.</span></span>

<span data-ttu-id="a00ee-163">或者，可以事先在缓存中部分或完全填充数据，这通常发生在应用程序启动时（此方法称为种子设定）。</span><span class="sxs-lookup"><span data-stu-id="a00ee-163">Alternatively, a cache can be partially or fully populated with data in advance, typically when the application starts (an approach known as seeding).</span></span> <span data-ttu-id="a00ee-164">但是，不建议对大型缓存实施种子设定，因为这种方法在应用程序开始运行时，可能会在原始数据存储上造成突发性的高负载。</span><span class="sxs-lookup"><span data-stu-id="a00ee-164">However, it might not be advisable to implement seeding for a large cache because this approach can impose a sudden, high load on the original data store when the application starts running.</span></span>

<span data-ttu-id="a00ee-165">使用模式分析通常可以帮助确定是否要完整或部分预先填充缓存，以及选择要缓存的数据。</span><span class="sxs-lookup"><span data-stu-id="a00ee-165">Often an analysis of usage patterns can help you decide whether to fully or partially prepopulate a cache, and to choose the data to cache.</span></span> <span data-ttu-id="a00ee-166">例如，对于定期（也许是每天）使用应用程序的客户，使用静态用户配置文件数据设定缓存种子可能相当实用，但不适用于一周仅使用一次应用程序的客户。</span><span class="sxs-lookup"><span data-stu-id="a00ee-166">For example, it can be useful to seed the cache with the static user profile data for customers who use the application regularly (perhaps every day), but not for customers who use the application only once a week.</span></span>

<span data-ttu-id="a00ee-167">缓存通常适用于不会变化或很少变化的数据。</span><span class="sxs-lookup"><span data-stu-id="a00ee-167">Caching typically works well with data that is immutable or that changes infrequently.</span></span> <span data-ttu-id="a00ee-168">示例包含引用信息，例如电子商务应用程序中的产品和价格信息，或构建成本高昂的共享静态资源。</span><span class="sxs-lookup"><span data-stu-id="a00ee-168">Examples include reference information such as product and pricing information in an e-commerce application, or shared static resources that are costly to construct.</span></span> <span data-ttu-id="a00ee-169">此数据的部分或全部可在应用程序启动时加载到缓存，以便将资源需求降到最低并提高性能。</span><span class="sxs-lookup"><span data-stu-id="a00ee-169">Some or all of this data can be loaded into the cache at application startup to minimize demand on resources and to improve performance.</span></span> <span data-ttu-id="a00ee-170">拥有定期更新缓存中引用数据的后台进程可能也是适当的方式，可确保其处于最新状态，或在引用数据更改时刷新缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-170">It might also be appropriate to have a background process that periodically updates reference data in the cache to ensure it is up to date, or that refreshes the cache when reference data changes.</span></span>

<span data-ttu-id="a00ee-171">缓存可能较不适合动态数据，但这种考虑因素有一些例外情况（请参阅本文后面的“缓存高动态数据”部分以了解详细信息）。</span><span class="sxs-lookup"><span data-stu-id="a00ee-171">Caching is less useful for dynamic data, although there are some exceptions to this consideration (see the section Cache highly dynamic data later in this article for more information).</span></span> <span data-ttu-id="a00ee-172">如果原始数据定期更改，缓存的信息可能很快就会过时，或者为了保持与原始数据存储的缓存同步而产生开销，导致降低缓存的效率。</span><span class="sxs-lookup"><span data-stu-id="a00ee-172">When the original data changes regularly, either the cached information becomes stale very quickly or the overhead of synchronizing the cache with the original data store reduces the effectiveness of caching.</span></span>

<span data-ttu-id="a00ee-173">请注意，缓存中不一定会包含实体的完整数据。</span><span class="sxs-lookup"><span data-stu-id="a00ee-173">Note that a cache does not have to include the complete data for an entity.</span></span> <span data-ttu-id="a00ee-174">例如，如果数据项代表多值对象（例如具有名称、地址和帐户余额的银行客户），则其中某些元素可以保持静态（例如名称和地址），而有些元素（例如帐户余额）则可能更加动态。</span><span class="sxs-lookup"><span data-stu-id="a00ee-174">For example, if a data item represents a multivalued object such as a bank customer with a name, address, and account balance, some of these elements might remain static (such as the name and address), while others (such as the account balance) might be more dynamic.</span></span> <span data-ttu-id="a00ee-175">在这种情况下，缓存数据的静态部分，并只在需要时检索（或计算）剩余信息可能相当有用。</span><span class="sxs-lookup"><span data-stu-id="a00ee-175">In these situations, it can be useful to cache the static portions of the data and retrieve (or calculate) only the remaining information when it is required.</span></span>

<span data-ttu-id="a00ee-176">建议执行性能测试和使用情况分析来确定缓存的预先填充和/或按需加载是否适当。</span><span class="sxs-lookup"><span data-stu-id="a00ee-176">We recommend that you carry out performance testing and usage analysis to determine whether pre-population or on-demand loading of the cache, or a combination of both, is appropriate.</span></span> <span data-ttu-id="a00ee-177">这种判断应该基于数据易变性和使用模式。</span><span class="sxs-lookup"><span data-stu-id="a00ee-177">The decision should be based on the volatility and usage pattern of the data.</span></span> <span data-ttu-id="a00ee-178">在会遇到重度负载且必须高度可缩放的应用程序中，缓存利用和性能分析特别重要。</span><span class="sxs-lookup"><span data-stu-id="a00ee-178">Cache utilization and performance analysis is particularly important in applications that encounter heavy loads and must be highly scalable.</span></span> <span data-ttu-id="a00ee-179">例如，在高度可缩放的方案中，有时可以设定缓存种子，以在高峰期降低数据存储的负载。</span><span class="sxs-lookup"><span data-stu-id="a00ee-179">For example, in highly scalable scenarios it might make sense to seed the cache to reduce the load on the data store at peak times.</span></span>

<span data-ttu-id="a00ee-180">缓存还可用于在应用程序运行时避免重复计算。</span><span class="sxs-lookup"><span data-stu-id="a00ee-180">Caching can also be used to avoid repeating computations while the application is running.</span></span> <span data-ttu-id="a00ee-181">如果操作会转换数据或执行复杂计算，则可以在缓存中保存操作的结果。</span><span class="sxs-lookup"><span data-stu-id="a00ee-181">If an operation transforms data or performs a complicated calculation, it can save the results of the operation in the cache.</span></span> <span data-ttu-id="a00ee-182">如果后续需要相同的计算，应用程序只需从缓存中检索结果。</span><span class="sxs-lookup"><span data-stu-id="a00ee-182">If the same calculation is required afterward, the application can simply retrieve the results from the cache.</span></span>

<span data-ttu-id="a00ee-183">应用程序可以修改保存在缓存中的数据。</span><span class="sxs-lookup"><span data-stu-id="a00ee-183">An application can modify data that's held in a cache.</span></span> <span data-ttu-id="a00ee-184">但是，我们建议将缓存视为可能随时消失的暂时性数据存储。</span><span class="sxs-lookup"><span data-stu-id="a00ee-184">However, we recommend thinking of the cache as a transient data store that could disappear at any time.</span></span> <span data-ttu-id="a00ee-185">请勿只在缓存中存储重要数据，而是确保同时在原始数据存储中保留信息。</span><span class="sxs-lookup"><span data-stu-id="a00ee-185">Do not store valuable data in the cache only; make sure that you maintain the information in the original data store as well.</span></span> <span data-ttu-id="a00ee-186">这意味着，当缓存不可用时，可以最大程度地减少数据丢失。</span><span class="sxs-lookup"><span data-stu-id="a00ee-186">This means that if the cache becomes unavailable, you minimize the chance of losing data.</span></span>

### <a name="cache-highly-dynamic-data"></a><span data-ttu-id="a00ee-187">缓存高动态数据</span><span class="sxs-lookup"><span data-stu-id="a00ee-187">Cache highly dynamic data</span></span>
<span data-ttu-id="a00ee-188">在永久性数据存储中存储快速变化的信息时，可能会给系统造成开销。</span><span class="sxs-lookup"><span data-stu-id="a00ee-188">When you store rapidly-changing information in a persistent data store, it can impose an overhead on the system.</span></span> <span data-ttu-id="a00ee-189">例如，假设有一个会持续报告状态或其他度量的设备。</span><span class="sxs-lookup"><span data-stu-id="a00ee-189">For example, consider a device that continually reports status or some other measurement.</span></span> <span data-ttu-id="a00ee-190">在缓存信息几乎一直处于过期状态的情况下，如果应用程序选择不要缓存此数据，则在数据存储中存储和检索此信息时，同样存在这种考虑因素。</span><span class="sxs-lookup"><span data-stu-id="a00ee-190">If an application chooses not to cache this data on the basis that the cached information will nearly always be outdated, then the same consideration could be true when storing and retrieving this information from the data store.</span></span> <span data-ttu-id="a00ee-191">在保存和提取此数据时它可能已经更改。</span><span class="sxs-lookup"><span data-stu-id="a00ee-191">In the time it takes to save and fetch this data, it might have changed.</span></span>

<span data-ttu-id="a00ee-192">在这种情况下，请考虑直接在缓存而不是永久性数据存储中存储动态信息的优点。</span><span class="sxs-lookup"><span data-stu-id="a00ee-192">In a situation such as this, consider the benefits of storing the dynamic information directly in the cache instead of in the persistent data store.</span></span> <span data-ttu-id="a00ee-193">如果数据不太重要且不需要审核，则偶尔丢失更改的数据就无关紧要。</span><span class="sxs-lookup"><span data-stu-id="a00ee-193">If the data is non-critical and does not require auditing, then it doesn't matter if the occasional change is lost.</span></span>

### <a name="manage-data-expiration-in-a-cache"></a><span data-ttu-id="a00ee-194">管理缓存中的数据过期</span><span class="sxs-lookup"><span data-stu-id="a00ee-194">Manage data expiration in a cache</span></span>
<span data-ttu-id="a00ee-195">在大多数情况下，缓存中保存的数据是保存在原始数据存储中的数据的副本。</span><span class="sxs-lookup"><span data-stu-id="a00ee-195">In most cases, data that's held in a cache is a copy of data that's held in the original data store.</span></span> <span data-ttu-id="a00ee-196">原始数据存储中的数据可能在缓存后更改，导致缓存的数据过时。</span><span class="sxs-lookup"><span data-stu-id="a00ee-196">The data in the original data store might change after it was cached, causing the cached data to become stale.</span></span> <span data-ttu-id="a00ee-197">许多缓存系统允许将缓存配置为使数据过期，以及减少数据可以过期的时间长短。</span><span class="sxs-lookup"><span data-stu-id="a00ee-197">Many caching systems enable you to configure the cache to expire data and reduce the period for which data may be out of date.</span></span>

<span data-ttu-id="a00ee-198">过期的缓存数据将从缓存中删除，应用程序必须从原始数据存储中检索数据（它可以将新提取的信息放回缓存）。</span><span class="sxs-lookup"><span data-stu-id="a00ee-198">When cached data expires, it's removed from the cache, and the application must retrieve the data from the original data store (it can put the newly-fetched information back into cache).</span></span> <span data-ttu-id="a00ee-199">在配置缓存时，可以设置默认的过期策略。</span><span class="sxs-lookup"><span data-stu-id="a00ee-199">You can set a default expiration policy when you configure the cache.</span></span> <span data-ttu-id="a00ee-200">在许多缓存服务中，当以编程方式将单个对象存储在缓存中时，还可以规定这些对象的过期时间。</span><span class="sxs-lookup"><span data-stu-id="a00ee-200">In many cache services, you can also stipulate the expiration period for individual objects when you store them programmatically in the cache.</span></span>
<span data-ttu-id="a00ee-201">某些缓存可让你将过期时间指定为绝对值，或者，如果并未在指定的时间内访问，则从缓存中删除项的滑动值。</span><span class="sxs-lookup"><span data-stu-id="a00ee-201">Some caches enable you to specify the expiration period as an absolute value, or as a sliding value that causes the item to be removed from the cache if it is not accessed within the specified time.</span></span> <span data-ttu-id="a00ee-202">此设置将重写任何缓存范围的过期策略，但只适用于指定的对象。</span><span class="sxs-lookup"><span data-stu-id="a00ee-202">This setting overrides any cache-wide expiration policy, but only for the specified objects.</span></span>

> [!NOTE]
> <span data-ttu-id="a00ee-203">请慎重考虑缓存的过期时段及其包含的对象。</span><span class="sxs-lookup"><span data-stu-id="a00ee-203">Consider the expiration period for the cache and the objects that it contains carefully.</span></span> <span data-ttu-id="a00ee-204">如果设置的时段太短，则对象很快就会过期，因此就减少了使用缓存带来的优势。</span><span class="sxs-lookup"><span data-stu-id="a00ee-204">If you make it too short, objects will expire too quickly and you will reduce the benefits of using the cache.</span></span> <span data-ttu-id="a00ee-205">如果设置的时段太长，则会面临数据过时的风险。</span><span class="sxs-lookup"><span data-stu-id="a00ee-205">If you make the period too long, you risk the data becoming stale.</span></span>
> 
> 

<span data-ttu-id="a00ee-206">此外，如果允许数据长时间驻留，则缓存有可能会填满。</span><span class="sxs-lookup"><span data-stu-id="a00ee-206">It's also possible that the cache might fill up if data is allowed to remain resident for a long time.</span></span> <span data-ttu-id="a00ee-207">在此情况下，将新项添加到缓存的任何请求可能会导致某些项被强行删除，这个过程称为逐出。</span><span class="sxs-lookup"><span data-stu-id="a00ee-207">In this case, any requests to add new items to the cache might cause some items to be forcibly removed in a process known as eviction.</span></span> <span data-ttu-id="a00ee-208">缓存服务通常按最近最少使用 (LRU) 原则逐出数据，但通常可以替代此策略并防止逐出项。</span><span class="sxs-lookup"><span data-stu-id="a00ee-208">Cache services typically evict data on a least-recently-used (LRU) basis, but you can usually override this policy and prevent items from being evicted.</span></span> <span data-ttu-id="a00ee-209">但是，如果采用这种方法，则会面临缓存超过可用内存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-209">However, if you adopt this approach, you risk exceeding the memory that's available in the cache.</span></span> <span data-ttu-id="a00ee-210">应用程序尝试将项添加到缓存时会失败并发生异常。</span><span class="sxs-lookup"><span data-stu-id="a00ee-210">An application that attempts to add an item to the cache will fail with an exception.</span></span>

<span data-ttu-id="a00ee-211">某些缓存的实施可能会提供其他逐出策略。</span><span class="sxs-lookup"><span data-stu-id="a00ee-211">Some caching implementations might provide additional eviction policies.</span></span> <span data-ttu-id="a00ee-212">有多种类型的逐出策略。</span><span class="sxs-lookup"><span data-stu-id="a00ee-212">There are several types of eviction policies.</span></span> <span data-ttu-id="a00ee-213">其中包括：</span><span class="sxs-lookup"><span data-stu-id="a00ee-213">These include:</span></span>

* <span data-ttu-id="a00ee-214">最近使用的策略（预期不再需要数据）。</span><span class="sxs-lookup"><span data-stu-id="a00ee-214">A most-recently-used policy (in the expectation that the data will not be required again).</span></span>
* <span data-ttu-id="a00ee-215">先进先出策略（先逐出最旧的数据）。</span><span class="sxs-lookup"><span data-stu-id="a00ee-215">A first-in-first-out policy (oldest data is evicted first).</span></span>
* <span data-ttu-id="a00ee-216">或基于触发事件显式删除（例如，正在修改数据）。</span><span class="sxs-lookup"><span data-stu-id="a00ee-216">An explicit removal policy based on a triggered event (such as the data being modified).</span></span>

### <a name="invalidate-data-in-a-client-side-cache"></a><span data-ttu-id="a00ee-217">使客户端缓存中的数据失效</span><span class="sxs-lookup"><span data-stu-id="a00ee-217">Invalidate data in a client-side cache</span></span>
<span data-ttu-id="a00ee-218">保存在客户端缓存中的数据通常被视为不受向客户端提供数据的服务的支持。</span><span class="sxs-lookup"><span data-stu-id="a00ee-218">Data that's held in a client-side cache is generally considered to be outside the auspices of the service that provides the data to the client.</span></span> <span data-ttu-id="a00ee-219">服务不能直接强制客户端添加或删除来自客户端缓存的信息。</span><span class="sxs-lookup"><span data-stu-id="a00ee-219">A service cannot directly force a client to add or remove information from a client-side cache.</span></span>

<span data-ttu-id="a00ee-220">这意味着，使用配置不当的缓存的客户端可能继续使用过时的本地缓存信息。</span><span class="sxs-lookup"><span data-stu-id="a00ee-220">This means that it's possible for a client that uses a poorly configured cache to continue using outdated information.</span></span> <span data-ttu-id="a00ee-221">例如，如果未正确实施过期策略，当原始数据源中的信息已更改时，客户端可能使用本地缓存的已过时信息。</span><span class="sxs-lookup"><span data-stu-id="a00ee-221">For example, if the expiration policies of the cache aren't properly implemented, a client might use outdated information that's cached locally when the information in the original data source has changed.</span></span>

<span data-ttu-id="a00ee-222">如果要构建通过 HTTP 连接提供数据的 Web 应用程序，可以隐式强制 Web 客户端（例如浏览器或 Web 代理）提取最新的信息。</span><span class="sxs-lookup"><span data-stu-id="a00ee-222">If you are building a web application that serves data over an HTTP connection, you can implicitly force a web client (such as a browser or web proxy) to fetch the most recent information.</span></span> <span data-ttu-id="a00ee-223">当资源通过更改该资源的 URI 更新时，可以执行此操作。</span><span class="sxs-lookup"><span data-stu-id="a00ee-223">You can do this if a resource is updated by a change in the URI of that resource.</span></span> <span data-ttu-id="a00ee-224">Web 客户端通常使用资源的 URI 作为客户端缓存中的键，因此更改 URI 会导致 Web 客户端忽略任何先前缓存的资源版本，并改为提取新的版本。</span><span class="sxs-lookup"><span data-stu-id="a00ee-224">Web clients typically use the URI of a resource as the key in the client-side cache, so if the URI changes, the web client ignores any previously cached versions of a resource and fetches the new version instead.</span></span>

## <a name="managing-concurrency-in-a-cache"></a><span data-ttu-id="a00ee-225">管理缓存中的并发</span><span class="sxs-lookup"><span data-stu-id="a00ee-225">Managing concurrency in a cache</span></span>
<span data-ttu-id="a00ee-226">缓存通常设计为由应用程序的多个实例共享。</span><span class="sxs-lookup"><span data-stu-id="a00ee-226">Caches are often designed to be shared by multiple instances of an application.</span></span> <span data-ttu-id="a00ee-227">每个应用程序实例可以读取和修改缓存中的数据。</span><span class="sxs-lookup"><span data-stu-id="a00ee-227">Each application instance can read and modify data in the cache.</span></span> <span data-ttu-id="a00ee-228">因此，任何共享数据存储中会出现的并发问题，在缓存中同样也会出现。</span><span class="sxs-lookup"><span data-stu-id="a00ee-228">Consequently, the same concurrency issues that arise with any shared data store also apply to a cache.</span></span> <span data-ttu-id="a00ee-229">在应用程序需要修改缓存中保存的数据的情况下，可能需要确保应用程序的一个实例所做的更新不会盲目地覆盖另一个实例所做的更改。</span><span class="sxs-lookup"><span data-stu-id="a00ee-229">In a situation where an application needs to modify data that's held in the cache, you might need to ensure that updates made by one instance of the application do not overwrite the changes made by another instance.</span></span>

<span data-ttu-id="a00ee-230">根据数据的性质和冲突的可能性，可以采用以下两种并发方式之一：</span><span class="sxs-lookup"><span data-stu-id="a00ee-230">Depending on the nature of the data and the likelihood of collisions, you can adopt one of two approaches to concurrency:</span></span>

* <span data-ttu-id="a00ee-231">**乐观并发。**</span><span class="sxs-lookup"><span data-stu-id="a00ee-231">**Optimistic.**</span></span> <span data-ttu-id="a00ee-232">应用程序检查以确定缓存中的数据自检索之后、更新之前是否已更改。</span><span class="sxs-lookup"><span data-stu-id="a00ee-232">Immediately prior to updating the data, the application checks to see whether the data in the cache has changed since it was retrieved.</span></span> <span data-ttu-id="a00ee-233">如果数据保持相同，则可以进行更改。</span><span class="sxs-lookup"><span data-stu-id="a00ee-233">If the data is still the same, the change can be made.</span></span> <span data-ttu-id="a00ee-234">否则，应用程序必须确定是否要进行更新。</span><span class="sxs-lookup"><span data-stu-id="a00ee-234">Otherwise, the application has to decide whether to update it.</span></span> <span data-ttu-id="a00ee-235">（促使做出此决定的业务逻辑特定于应用程序。）这种方法适合不常更新或不太可能发生冲突的情况。</span><span class="sxs-lookup"><span data-stu-id="a00ee-235">(The business logic that drives this decision will be application-specific.) This approach is suitable for situations where updates are infrequent, or where collisions are unlikely to occur.</span></span>
* <span data-ttu-id="a00ee-236">**悲观并发。**</span><span class="sxs-lookup"><span data-stu-id="a00ee-236">**Pessimistic.**</span></span> <span data-ttu-id="a00ee-237">应用程序在检索缓存中的数据时锁定数据，以避免另一个实例更改数据。</span><span class="sxs-lookup"><span data-stu-id="a00ee-237">When it retrieves the data, the application locks it in the cache to prevent another instance from changing it.</span></span> <span data-ttu-id="a00ee-238">此过程可确保不发生冲突，但可能阻止其他需要处理同一数据的实例。</span><span class="sxs-lookup"><span data-stu-id="a00ee-238">This process ensures that collisions cannot occur, but they can also block other instances that need to process the same data.</span></span> <span data-ttu-id="a00ee-239">悲观并发可能会影响解决方案的伸缩性，建议只对短期操作使用。</span><span class="sxs-lookup"><span data-stu-id="a00ee-239">Pessimistic concurrency can affect the scalability of a solution and is recommended only for short-lived operations.</span></span> <span data-ttu-id="a00ee-240">这种方法可能适用于很可能发生冲突的情况，特别是当应用程序更新缓存中的多个项，且必须确保这些更改一致应用时。</span><span class="sxs-lookup"><span data-stu-id="a00ee-240">This approach might be appropriate for situations where collisions are more likely, especially if an application updates multiple items in the cache and must ensure that these changes are applied consistently.</span></span>

### <a name="implement-high-availability-and-scalability-and-improve-performance"></a><span data-ttu-id="a00ee-241">实现高可用性和伸缩性并提高性能</span><span class="sxs-lookup"><span data-stu-id="a00ee-241">Implement high availability and scalability, and improve performance</span></span>
<span data-ttu-id="a00ee-242">避免使用缓存作为数据的主存储库；主存储库应该是从中填充缓存的原始数据存储的角色。</span><span class="sxs-lookup"><span data-stu-id="a00ee-242">Avoid using a cache as the primary repository of data; this is the role of the original data store from which the cache is populated.</span></span> <span data-ttu-id="a00ee-243">原始数据存储负责确保数据的持久性。</span><span class="sxs-lookup"><span data-stu-id="a00ee-243">The original data store is responsible for ensuring the persistence of the data.</span></span>

<span data-ttu-id="a00ee-244">请小心不要将共享缓存服务可用性的重要依赖性引入解决方案。</span><span class="sxs-lookup"><span data-stu-id="a00ee-244">Be careful not to introduce critical dependencies on the availability of a shared cache service into your solutions.</span></span> <span data-ttu-id="a00ee-245">如果提供共享缓存的服务不可用，应用程序应能继续工作。</span><span class="sxs-lookup"><span data-stu-id="a00ee-245">An application should be able to continue functioning if the service that provides the shared cache is unavailable.</span></span> <span data-ttu-id="a00ee-246">应用程序应该不会在等待缓存服务恢复时停止响应或失败。</span><span class="sxs-lookup"><span data-stu-id="a00ee-246">The application should not hang or fail while waiting for the cache service to resume.</span></span>

<span data-ttu-id="a00ee-247">因此，应用程序必须准备好检测缓存服务的可用性，并在无法访问缓存时回退到原始数据存储。</span><span class="sxs-lookup"><span data-stu-id="a00ee-247">Therefore, the application must be prepared to detect the availability of the cache service and fall back to the original data store if the cache is inaccessible.</span></span> <span data-ttu-id="a00ee-248">[断路器模式](http://msdn.microsoft.com/library/dn589784.aspx)可用于处理这种情况。</span><span class="sxs-lookup"><span data-stu-id="a00ee-248">The [Circuit-Breaker pattern](http://msdn.microsoft.com/library/dn589784.aspx) is useful for handling this scenario.</span></span> <span data-ttu-id="a00ee-249">提供缓存的服务可以恢复，当服务可用时，缓存会在从原始数据存储读取数据时，遵循[缓存端模式](http://msdn.microsoft.com/library/dn589799.aspx)等策略重新填充。</span><span class="sxs-lookup"><span data-stu-id="a00ee-249">The service that provides the cache can be recovered, and once it becomes available, the cache can be repopulated as data is read form the original data store, following a strategy such as the [Cache-aside pattern](http://msdn.microsoft.com/library/dn589799.aspx).</span></span>

<span data-ttu-id="a00ee-250">但是，在缓存暂时不可用的情况下应用程序回退到原始数据存储可能会影响系统的伸缩性。</span><span class="sxs-lookup"><span data-stu-id="a00ee-250">However, there might be a scalability impact on the system if the application falls back to the original data store when the cache is temporarily unavailable.</span></span>
<span data-ttu-id="a00ee-251">在恢复数据存储时，原始数据存储可能忙于处理数据请求，导致超时和连接失败。</span><span class="sxs-lookup"><span data-stu-id="a00ee-251">While the data store is being recovered, the original data store could be swamped with requests for data, resulting in timeouts and failed connections.</span></span>

<span data-ttu-id="a00ee-252">考虑在每个应用程序实例中实施本地专用缓存，以及所有应用程序实例访问的共享缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-252">Consider implementing a local, private cache in each instance of an application, together with the shared cache that all application instances access.</span></span> <span data-ttu-id="a00ee-253">当应用程序检索项时，可能会先后在本地缓存、共享缓存和原始数据存储中检查。</span><span class="sxs-lookup"><span data-stu-id="a00ee-253">When the application retrieves an item, it can check first in its local cache, then in the shared cache, and finally in the original data store.</span></span> <span data-ttu-id="a00ee-254">共享缓存不可用时，本地缓存可以使用共享缓存或数据库中的数据来填充。</span><span class="sxs-lookup"><span data-stu-id="a00ee-254">The local cache can be populated using the data in either the shared cache, or in the database if the shared cache is unavailable.</span></span>

<span data-ttu-id="a00ee-255">采用此方法需要经过慎重的配置，以防止本地缓存相对于共享缓存而言太过时。</span><span class="sxs-lookup"><span data-stu-id="a00ee-255">This approach requires careful configuration to prevent the local cache from becoming too stale with respect to the shared cache.</span></span> <span data-ttu-id="a00ee-256">但在无法访问共享缓存时，它可以充当缓冲区。</span><span class="sxs-lookup"><span data-stu-id="a00ee-256">However, the local cache acts as a buffer if the shared cache is unreachable.</span></span> <span data-ttu-id="a00ee-257">图 3 显示了此结构。</span><span class="sxs-lookup"><span data-stu-id="a00ee-257">Figure 3 shows this structure.</span></span>

<span data-ttu-id="a00ee-258">![将本地、专用和共享缓存配合使用](./images/caching/Caching3.png)
*图 3：将本地、专用和共享缓存配合使用*</span><span class="sxs-lookup"><span data-stu-id="a00ee-258">![Using a local, private cache with a shared cache](./images/caching/Caching3.png)
*Figure 3: Using a local, private cache with a shared cache*</span></span>

<span data-ttu-id="a00ee-259">为了支持保存相对长期数据的大型缓存，某些缓存服务在缓存不可用时，提供实施自动故障转移的高可用性选项。</span><span class="sxs-lookup"><span data-stu-id="a00ee-259">To support large caches that hold relatively long-lived data, some cache services provide a high-availability option that implements automatic failover if the cache becomes unavailable.</span></span> <span data-ttu-id="a00ee-260">这种方法通常涉及到将存储在主缓存服务器上的缓存数据复制到辅助缓存服务器，并在主服务器故障或断开连接时切换到辅助服务器。</span><span class="sxs-lookup"><span data-stu-id="a00ee-260">This approach typically involves replicating the cached data that's stored on a primary cache server to a secondary cache server, and switching to the secondary server if the primary server fails or connectivity is lost.</span></span>

<span data-ttu-id="a00ee-261">为了减少与写入多个目标相关的延迟，当数据写入主服务器上的缓存时，复制到辅助服务器的操作可以异步发生。</span><span class="sxs-lookup"><span data-stu-id="a00ee-261">To reduce the latency that's associated with writing to multiple destinations, the replication to the secondary server might occur asynchronously when data is written to the cache on the primary server.</span></span> <span data-ttu-id="a00ee-262">此方法可能会导致某些缓存的信息在发生故障时丢失，但是此数据的比例应该小于缓存的总体大小。</span><span class="sxs-lookup"><span data-stu-id="a00ee-262">This approach leads to the possibility that some cached information might be lost in the event of a failure, but the proportion of this data should be small compared to the overall size of the cache.</span></span>

<span data-ttu-id="a00ee-263">如果共享缓存很大，则在节点上分区缓存数据可能很有帮助，这可减少争用的可能性，并提高伸缩性。</span><span class="sxs-lookup"><span data-stu-id="a00ee-263">If a shared cache is large, it might be beneficial to partition the cached data across nodes to reduce the chances of contention and improve scalability.</span></span> <span data-ttu-id="a00ee-264">许多共享缓存支持动态添加（与删除）节点，以及重新平衡分区之间的数据的功能。</span><span class="sxs-lookup"><span data-stu-id="a00ee-264">Many shared caches support the ability to dynamically add (and remove) nodes and rebalance the data across partitions.</span></span> <span data-ttu-id="a00ee-265">这种方法可能涉及到群集，其中，节点集合将作为无缝单一缓存向客户端应用程序呈现。</span><span class="sxs-lookup"><span data-stu-id="a00ee-265">This approach might involve clustering, in which the collection of nodes is presented to client applications as a seamless, single cache.</span></span> <span data-ttu-id="a00ee-266">但在内部，数据分散在节点之间并遵循某种预定义的分配策略，以便平均地平衡负载。</span><span class="sxs-lookup"><span data-stu-id="a00ee-266">Internally, however, the data is dispersed between nodes following a predefined distribution strategy that balances the load evenly.</span></span> <span data-ttu-id="a00ee-267">Microsoft 网站上的[数据分区指南文档](http://msdn.microsoft.com/library/dn589795.aspx)提供了有关可行分区策略的详细信息。</span><span class="sxs-lookup"><span data-stu-id="a00ee-267">The [Data partitioning guidance document](http://msdn.microsoft.com/library/dn589795.aspx) on the Microsoft website provides more information about possible partitioning strategies.</span></span>

<span data-ttu-id="a00ee-268">群集还可以提高缓存的可用性。</span><span class="sxs-lookup"><span data-stu-id="a00ee-268">Clustering can also increase the availability of the cache.</span></span> <span data-ttu-id="a00ee-269">如果节点发生故障，仍可访问缓存的剩余部分。</span><span class="sxs-lookup"><span data-stu-id="a00ee-269">If a node fails, the remainder of the cache is still accessible.</span></span>
<span data-ttu-id="a00ee-270">群集经常与复制和故障转移结合使用。</span><span class="sxs-lookup"><span data-stu-id="a00ee-270">Clustering is frequently used in conjunction with replication and failover.</span></span> <span data-ttu-id="a00ee-271">每个节点都可复制且副本在节点故障时可快速联机。</span><span class="sxs-lookup"><span data-stu-id="a00ee-271">Each node can be replicated, and the replica can be quickly brought online if the node fails.</span></span>

<span data-ttu-id="a00ee-272">许多读取和写入操作可能会涉及到单个数据值或对象。</span><span class="sxs-lookup"><span data-stu-id="a00ee-272">Many read and write operations are likely to involve single data values or objects.</span></span> <span data-ttu-id="a00ee-273">但是，有时可能需要快速存储或检索大量数据。</span><span class="sxs-lookup"><span data-stu-id="a00ee-273">However, at times it might be necessary to store or retrieve large volumes of data quickly.</span></span>
<span data-ttu-id="a00ee-274">例如，设定缓存种子可能涉及到将数百或数千个项写入到缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-274">For example, seeding a cache could involve writing hundreds or thousands of items to the cache.</span></span> <span data-ttu-id="a00ee-275">应用程序还可能需要从缓存中检索属于同一请求的大量相关项。</span><span class="sxs-lookup"><span data-stu-id="a00ee-275">An application might also need to retrieve a large number of related items from the cache as part of the same request.</span></span>

<span data-ttu-id="a00ee-276">许多大型缓存针对这些目的提供了批处理操作。</span><span class="sxs-lookup"><span data-stu-id="a00ee-276">Many large-scale caches provide batch operations for these purposes.</span></span> <span data-ttu-id="a00ee-277">这样，客户端应用程序便可以将大量的项打包成单个请求，并减少执行大量小型请求时的相关开销。</span><span class="sxs-lookup"><span data-stu-id="a00ee-277">This enables a client application to package up a large volume of items into a single request and reduces the overhead that's associated with performing a large number of small requests.</span></span>

## <a name="caching-and-eventual-consistency"></a><span data-ttu-id="a00ee-278">缓存和最终一致性</span><span class="sxs-lookup"><span data-stu-id="a00ee-278">Caching and eventual consistency</span></span>
<span data-ttu-id="a00ee-279">要使缓存端模式正常工作，填充缓存的应用程序实例必须有权访问最新且一致的数据版本。</span><span class="sxs-lookup"><span data-stu-id="a00ee-279">For the cache-aside pattern to work, the instance of the application that populates the cache must have access to the most recent and consistent version of the data.</span></span> <span data-ttu-id="a00ee-280">在实施最终一致性的系统（例如复制的数据存储）中，情况可能不是这样。</span><span class="sxs-lookup"><span data-stu-id="a00ee-280">In a system that implements eventual consistency (such as a replicated data store) this might not be the case.</span></span>

<span data-ttu-id="a00ee-281">应用程序的一个实例可以修改数据项，使该项的缓存版本失效。</span><span class="sxs-lookup"><span data-stu-id="a00ee-281">One instance of an application could modify a data item and invalidate the cached version of that item.</span></span> <span data-ttu-id="a00ee-282">应用程序的另一个实例可以尝试从导致缓存未命中的缓存读取此项，因此它将从数据存储中读取数据，并将它添加到缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-282">Another instance of the application might attempt to read this item from a cache, which causes a cache-miss, so it reads the data from the data store and adds it to the cache.</span></span> <span data-ttu-id="a00ee-283">但是，如果数据存储没有完全与其他副本同步，则应用程序实例可能会使用旧值来读取并填充缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-283">However, if the data store has not been fully synchronized with the other replicas, the application instance could read and populate the cache with the old value.</span></span>

<span data-ttu-id="a00ee-284">有关处理数据一致性的详细信息，请参阅 [Data consistency primer](http://msdn.microsoft.com/library/dn589800.aspx)（数据一致性入门）。</span><span class="sxs-lookup"><span data-stu-id="a00ee-284">For more information about handling data consistency, see the [Data consistency primer](http://msdn.microsoft.com/library/dn589800.aspx).</span></span>

### <a name="protect-cached-data"></a><span data-ttu-id="a00ee-285">保护缓存的数据</span><span class="sxs-lookup"><span data-stu-id="a00ee-285">Protect cached data</span></span>
<span data-ttu-id="a00ee-286">无论使用的缓存服务为何，都应该考虑如何防范缓存中保存的数据遭到未经授权的访问。</span><span class="sxs-lookup"><span data-stu-id="a00ee-286">Irrespective of the cache service you use, consider how to protect the data that's held in the cache from unauthorized access.</span></span> <span data-ttu-id="a00ee-287">有两个主要考虑因素：</span><span class="sxs-lookup"><span data-stu-id="a00ee-287">There are two main concerns:</span></span>

* <span data-ttu-id="a00ee-288">缓存中数据的隐私性。</span><span class="sxs-lookup"><span data-stu-id="a00ee-288">The privacy of the data in the cache.</span></span>
* <span data-ttu-id="a00ee-289">数据在缓存与使用缓存的应用程序之间流动时的隐私性。</span><span class="sxs-lookup"><span data-stu-id="a00ee-289">The privacy of data as it flows between the cache and the application that's using the cache.</span></span>

<span data-ttu-id="a00ee-290">若要保护缓存中的数据，缓存服务可以实施一种身份验证机制，要求应用程序指定：</span><span class="sxs-lookup"><span data-stu-id="a00ee-290">To protect data in the cache, the cache service might implement an authentication mechanism that requires that applications specify the following:</span></span>

* <span data-ttu-id="a00ee-291">哪些标识可以访问缓存中的数据。</span><span class="sxs-lookup"><span data-stu-id="a00ee-291">Which identities can access data in the cache.</span></span>
* <span data-ttu-id="a00ee-292">允许这些标识执行的操作（读取和写入）。</span><span class="sxs-lookup"><span data-stu-id="a00ee-292">Which operations (read and write) that these identities are allowed to perform.</span></span>

<span data-ttu-id="a00ee-293">为了减少读取和写入数据时的相关开销，当标识已获得写入和/或读取缓存的权限时，该标识可以使用缓存中的任何数据。</span><span class="sxs-lookup"><span data-stu-id="a00ee-293">To reduce overhead that's associated with reading and writing data, after an identity has been granted write and/or read access to the cache, that identity can use any data in the cache.</span></span>

<span data-ttu-id="a00ee-294">如果需要限制对缓存数据子集的访问权限，可以执行以下操作之一：</span><span class="sxs-lookup"><span data-stu-id="a00ee-294">If you need to restrict access to subsets of the cached data, you can do one of the following:</span></span>

* <span data-ttu-id="a00ee-295">将缓存拆分成分区（使用不同的缓存服务器），并只向标识授予他们有权使用的分区的访问权限。</span><span class="sxs-lookup"><span data-stu-id="a00ee-295">Split the cache into partitions (by using different cache servers) and only grant access to identities for the partitions that they should be allowed to use.</span></span>
* <span data-ttu-id="a00ee-296">使用不同的密钥来加密每个子集中的数据，并只向应该具有每个子集访问权限的标识提供加密密钥。</span><span class="sxs-lookup"><span data-stu-id="a00ee-296">Encrypt the data in each subset by using different keys, and provide the encryption keys only to identities that should have access to each subset.</span></span> <span data-ttu-id="a00ee-297">客户端应用程序可能仍然能够检索缓存中的所有数据，但它只能够解密具有密钥的数据。</span><span class="sxs-lookup"><span data-stu-id="a00ee-297">A client application might still be able to retrieve all of the data in the cache, but it will only be able to decrypt the data for which it has the keys.</span></span>

<span data-ttu-id="a00ee-298">还必须在数据流入或流出缓存时保护数据。</span><span class="sxs-lookup"><span data-stu-id="a00ee-298">You must also protect the data as it flows in and out of the cache.</span></span> <span data-ttu-id="a00ee-299">为此，可以依赖于客户端应用程序用来连接缓存的网络基础结构所提供的安全功能。</span><span class="sxs-lookup"><span data-stu-id="a00ee-299">To do this, you depend on the security features provided by the network infrastructure that client applications use to connect to the cache.</span></span> <span data-ttu-id="a00ee-300">如果在托管客户端应用程序的同一组织中使用现场服务器来实施缓存，则网络本身的隔离可能不需要用户采取任何其他措施。</span><span class="sxs-lookup"><span data-stu-id="a00ee-300">If the cache is implemented using an on-site server within the same organization that hosts the client applications, then the isolation of the network itself might not require you to take  additional steps.</span></span> <span data-ttu-id="a00ee-301">如果缓存位于远程，且需要基于公共网络（例如 Internet）的 TCP 或 HTTP 连接，请考虑实施 SSL。</span><span class="sxs-lookup"><span data-stu-id="a00ee-301">If the cache is located remotely and requires a TCP or HTTP connection over a public network (such as the Internet), consider implementing SSL.</span></span>

## <a name="considerations-for-implementing-caching-with-microsoft-azure"></a><span data-ttu-id="a00ee-302">使用 Microsoft Azure 实现缓存的注意事项</span><span class="sxs-lookup"><span data-stu-id="a00ee-302">Considerations for implementing caching with Microsoft Azure</span></span>
<span data-ttu-id="a00ee-303">Azure 提供 Azure Redis 缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-303">Azure provides the Azure Redis Cache.</span></span> <span data-ttu-id="a00ee-304">这是开源 Redis 缓存的一种实现，可在 Azure 数据中心作为服务运行。</span><span class="sxs-lookup"><span data-stu-id="a00ee-304">This is an implementation of the open source Redis cache that runs as a service in an Azure datacenter.</span></span> <span data-ttu-id="a00ee-305">它提供可从任何 Azure 应用程序访问的缓存服务，无论应用程序是实施为云服务、网站，还是在 Azure 虚拟机中。</span><span class="sxs-lookup"><span data-stu-id="a00ee-305">It provides a caching service that can be accessed from any Azure application, whether the application is implemented as a cloud service, a website, or inside an Azure virtual machine.</span></span> <span data-ttu-id="a00ee-306">拥有适当访问密钥的客户端应用程序可以共享缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-306">Caches can be shared by client applications that have the appropriate access key.</span></span>

<span data-ttu-id="a00ee-307">Azure Redis 缓存是高性能缓存解决方案，提供可用性、伸缩性和安全性。</span><span class="sxs-lookup"><span data-stu-id="a00ee-307">Azure Redis Cache is a high-performance caching solution that provides availability, scalability and security.</span></span> <span data-ttu-id="a00ee-308">它通常作为分散在一个或多个专用计算机上的服务运行，</span><span class="sxs-lookup"><span data-stu-id="a00ee-308">It typically runs as a service spread across one or more dedicated machines.</span></span> <span data-ttu-id="a00ee-309">并尝试在内存中存储尽量多的信息以确保快速访问。</span><span class="sxs-lookup"><span data-stu-id="a00ee-309">It attempts to store as much information as it can in memory to ensure fast access.</span></span> <span data-ttu-id="a00ee-310">这种体系结构旨在通过减少执行缓慢 I/O 操作的需要，提供低延迟和高吞吐量。</span><span class="sxs-lookup"><span data-stu-id="a00ee-310">This architecture is intended to provide low latency and high throughput by reducing the need to perform slow I/O operations.</span></span>

 <span data-ttu-id="a00ee-311">Azure Redis 缓存与客户端应用程序使用的多种 API 兼容。</span><span class="sxs-lookup"><span data-stu-id="a00ee-311">Azure Redis Cache is compatible with many of the various APIs that are used by client applications.</span></span> <span data-ttu-id="a00ee-312">如果现有应用程序已使用运行本地的 Azure Redis 缓存，Azure Redis 缓存可在云中提供缓存的快速迁移路径。</span><span class="sxs-lookup"><span data-stu-id="a00ee-312">If you have existing applications that already use Azure Redis Cache running on-premises, the Azure Redis Cache provides a quick migration path to caching in the cloud.</span></span>

> [!NOTE]
> <span data-ttu-id="a00ee-313">Azure 还提供托管缓存服务。</span><span class="sxs-lookup"><span data-stu-id="a00ee-313">Azure also provides the Managed Cache Service.</span></span> <span data-ttu-id="a00ee-314">此服务基于 Azure Service Fabric 缓存引擎。</span><span class="sxs-lookup"><span data-stu-id="a00ee-314">This service is based on the Azure Service Fabric Cache engine.</span></span> <span data-ttu-id="a00ee-315">使用它可以创建可由松散耦合应用程序共享的分布式缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-315">It enables you to create a distributed cache that can be shared by loosely-coupled applications.</span></span> <span data-ttu-id="a00ee-316">缓存托管在 Azure 数据中心内运行的高性能服务器上。</span><span class="sxs-lookup"><span data-stu-id="a00ee-316">The cache is hosted on high-performance servers running in an Azure datacenter.</span></span>
> <span data-ttu-id="a00ee-317">但是，不再建议使用此选项，提供此选项只是为了支持构建为使用此选项的现有应用程序。</span><span class="sxs-lookup"><span data-stu-id="a00ee-317">However, this option is no longer recommended and is only provided to support existing applications that have been built to use it.</span></span> <span data-ttu-id="a00ee-318">针对所有新的开发，请改用 Azure Redis 缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-318">For all new development, use Azure Redis Cache instead.</span></span>
> 
> <span data-ttu-id="a00ee-319">此外，Azure 支持角色中缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-319">Additionally, Azure supports in-role caching.</span></span> <span data-ttu-id="a00ee-320">可使用此功能创建特定于云服务的缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-320">This feature enables you to create a cache that's specific to a cloud service.</span></span>
> <span data-ttu-id="a00ee-321">缓存由 Web 角色或辅助角色的实例托管，只能由以同一云服务部署单位的一部分来操作的角色进行访问。</span><span class="sxs-lookup"><span data-stu-id="a00ee-321">The cache is hosted by instances of a web or worker role, and can only be accessed by roles that are operating as part of the same cloud service deployment unit.</span></span> <span data-ttu-id="a00ee-322">（部署单位是作为云服务部署到特定区域的角色实例集合。）缓存已组建群集，托管缓存的同一部署单位中的所有角色实例将成为同一缓存群集的一部分。</span><span class="sxs-lookup"><span data-stu-id="a00ee-322">(A deployment unit is the set of role instances that are deployed as a cloud service to a specific region.) The cache is clustered, and all instances of the role within the same deployment unit that hosts the cache become part of the same cache cluster.</span></span> <span data-ttu-id="a00ee-323">但是，不再建议使用此选项，提供此选项只是为了支持构建为使用此选项的现有应用程序。</span><span class="sxs-lookup"><span data-stu-id="a00ee-323">However, this option is no longer recommended and is only provided to support existing applications that have been built to use it.</span></span> <span data-ttu-id="a00ee-324">针对所有新的开发，请改用 Azure Redis 缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-324">For all new development, use Azure Redis Cache instead.</span></span>
> 
> <span data-ttu-id="a00ee-325">Azure 托管缓存服务和 Azure 角色中缓存目前已预定于 2016 年 11 月 16 日停用。</span><span class="sxs-lookup"><span data-stu-id="a00ee-325">Both Azure Managed Cache Service and Azure In-Role Cache are currently slated for retirement on November 16th, 2016.</span></span>
> <span data-ttu-id="a00ee-326">建议迁移到 Azure Redis 缓存，以便为这次停用做好准备。</span><span class="sxs-lookup"><span data-stu-id="a00ee-326">It is recommended that you migrate to Azure Redis Cache in preparation for this retirement.</span></span> <span data-ttu-id="a00ee-327">有关详细信息，请参阅[应使用哪种 Azure Redis 缓存产品和大小？](/azure/redis-cache/cache-faq#what-redis-cache-offering-and-size-should-i-use)。</span><span class="sxs-lookup"><span data-stu-id="a00ee-327">For more information, see [What is Azure Redis Cache offering and what size should I use?](/azure/redis-cache/cache-faq#what-redis-cache-offering-and-size-should-i-use).</span></span>
> 
> 

### <a name="features-of-redis"></a><span data-ttu-id="a00ee-328">Redis 的功能</span><span class="sxs-lookup"><span data-stu-id="a00ee-328">Features of Redis</span></span>
 <span data-ttu-id="a00ee-329">Redis 不仅是简单的缓存服务器。</span><span class="sxs-lookup"><span data-stu-id="a00ee-329">Redis is more than a simple cache server.</span></span> <span data-ttu-id="a00ee-330">它还提供分布式内存中数据库，其中包含用于支持许多常见方案的广泛命令集。</span><span class="sxs-lookup"><span data-stu-id="a00ee-330">It provides a distributed in-memory database with an extensive command set that supports many common scenarios.</span></span> <span data-ttu-id="a00ee-331">本文档后面的“使用 Redis 缓存”部分中将提供介绍。</span><span class="sxs-lookup"><span data-stu-id="a00ee-331">These are described later in this document, in the section Using  Redis caching.</span></span> <span data-ttu-id="a00ee-332">本部分汇总了 Redis 提供的一些重要功能。</span><span class="sxs-lookup"><span data-stu-id="a00ee-332">This section summarizes some of the key features that Redis provides.</span></span>

### <a name="redis-as-an-in-memory-database"></a><span data-ttu-id="a00ee-333">Redis 用作内存中数据库</span><span class="sxs-lookup"><span data-stu-id="a00ee-333">Redis as an in-memory database</span></span>
<span data-ttu-id="a00ee-334">Redis 支持读取和写入操作。</span><span class="sxs-lookup"><span data-stu-id="a00ee-334">Redis supports both read and write operations.</span></span> <span data-ttu-id="a00ee-335">在 Redis 中，写入操作将定期存储在本地快照文件或仅限附加的日志文件中，从而写入操作可在系统故障时得到保护。</span><span class="sxs-lookup"><span data-stu-id="a00ee-335">In Redis, writes can be protected from system failure either by being stored  periodically in a local snapshot file or in an append-only log file.</span></span> <span data-ttu-id="a00ee-336">而其他许多缓存（应被视为暂时性数据存储）中并非如此。</span><span class="sxs-lookup"><span data-stu-id="a00ee-336">This is not the case in many caches (which should be considered  transitory data stores).</span></span>

 <span data-ttu-id="a00ee-337">所有写入都是异步的，不会阻止客户端读取和写入数据。</span><span class="sxs-lookup"><span data-stu-id="a00ee-337">All writes are asynchronous and do not block clients from reading and writing data.</span></span> <span data-ttu-id="a00ee-338">当 Redis 开始运行时，将从快照或日志文件中读取数据，并使用它来构建内存中缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-338">When Redis starts running, it reads the data from the snapshot or log file and uses it to construct the in-memory cache.</span></span> <span data-ttu-id="a00ee-339">有关详细信息，请参阅 Redis 网站上的 [Redis persistence](http://redis.io/topics/persistence)（Redis 持久性）。</span><span class="sxs-lookup"><span data-stu-id="a00ee-339">For more information, see [Redis persistence](http://redis.io/topics/persistence) on the Redis website.</span></span>

> [!NOTE]
> <span data-ttu-id="a00ee-340">Redis 不保证所有写入在发生灾难性故障时都会得到保存，但在最糟的情况下，只会丢失几秒钟的数据。</span><span class="sxs-lookup"><span data-stu-id="a00ee-340">Redis does not guarantee that all writes will be saved in the event of a catastrophic failure, but at worst you might lose only a few seconds worth of data.</span></span> <span data-ttu-id="a00ee-341">请记住，缓存并不适合用作权威数据源，应用程序负责使用缓存来确保成功将关键数据保存到适当的数据存储。</span><span class="sxs-lookup"><span data-stu-id="a00ee-341">Remember that a cache is not intended to act as an authoritative data source, and it is the responsibility of the applications using the cache to ensure that critical data is saved successfully to an appropriate data store.</span></span> <span data-ttu-id="a00ee-342">有关详细信息，请参阅[缓存端模式](http://msdn.microsoft.com/library/dn589799.aspx)。</span><span class="sxs-lookup"><span data-stu-id="a00ee-342">For more information, see the [cache-aside pattern](http://msdn.microsoft.com/library/dn589799.aspx).</span></span>
> 
> 

#### <a name="redis-data-types"></a><span data-ttu-id="a00ee-343">Redis 数据类型</span><span class="sxs-lookup"><span data-stu-id="a00ee-343">Redis data types</span></span>
<span data-ttu-id="a00ee-344">Redis 属于键-值存储，其中的值可以包含简单类型或复杂数据结构，例如哈希、列表和集。</span><span class="sxs-lookup"><span data-stu-id="a00ee-344">Redis is a key-value store, where values can contain simple types or complex data structures such as hashes, lists, and sets.</span></span> <span data-ttu-id="a00ee-345">Redis 支持对这些数据类型执行原子操作。</span><span class="sxs-lookup"><span data-stu-id="a00ee-345">It supports a set of atomic operations on these data types.</span></span> <span data-ttu-id="a00ee-346">键可以是永久性的，或者标记了一个有限的生存时间，到了该时间后，键及其对应的值会自动从缓存中删除。</span><span class="sxs-lookup"><span data-stu-id="a00ee-346">Keys can be permanent or tagged with a limited time-to-live, at which point the key and its corresponding value are automatically removed from the cache.</span></span> <span data-ttu-id="a00ee-347">有关 Redis 键和值的详细信息，请访问 Redis 网站上的 [An introduction to Redis data types and abstractions](http://redis.io/topics/data-types-intro)（Redis 数据类型和抽象简介）页。</span><span class="sxs-lookup"><span data-stu-id="a00ee-347">For more information about Redis keys and values, visit the page [An introduction to Redis data types and abstractions](http://redis.io/topics/data-types-intro) on the Redis website.</span></span>

#### <a name="redis-replication-and-clustering"></a><span data-ttu-id="a00ee-348">Redis 复制和群集</span><span class="sxs-lookup"><span data-stu-id="a00ee-348">Redis replication and clustering</span></span>
<span data-ttu-id="a00ee-349">Redis 支持主/从复制，以帮助确保可用性并保持吞吐量。</span><span class="sxs-lookup"><span data-stu-id="a00ee-349">Redis supports master/subordinate replication to help ensure availability and maintain throughput.</span></span> <span data-ttu-id="a00ee-350">Redis 主节点的写入操作将复制到一个或多个从属节点。</span><span class="sxs-lookup"><span data-stu-id="a00ee-350">Write operations to a Redis master node are replicated to one or more subordinate nodes.</span></span> <span data-ttu-id="a00ee-351">读取操作可由主节点或任何从属节点提供。</span><span class="sxs-lookup"><span data-stu-id="a00ee-351">Read operations can be served by the master or any of the subordinates.</span></span>

<span data-ttu-id="a00ee-352">如果执行了网络分区，从属节点可以继续提供数据，并在重新建立连接时以透明方式与主节点重新同步。</span><span class="sxs-lookup"><span data-stu-id="a00ee-352">In the event of a network partition, subordinates can continue to serve data and then transparently resynchronize with the master when the connection is reestablished.</span></span> <span data-ttu-id="a00ee-353">有关详细信息，请访问 Redis 网站上的 [Replication](http://redis.io/topics/replication)（复制）页。</span><span class="sxs-lookup"><span data-stu-id="a00ee-353">For further details, visit the [Replication](http://redis.io/topics/replication) page on the Redis website.</span></span>

<span data-ttu-id="a00ee-354">Redis 还提供群集，可让用户以透明方式在服务器之间将数据分区成分片并分散负载。</span><span class="sxs-lookup"><span data-stu-id="a00ee-354">Redis also provides clustering, which enables  you to transparently partition data into shards across servers and spread the load.</span></span> <span data-ttu-id="a00ee-355">此功能提高了伸缩性，因为可以添加新 Redis 服务器，并且随着缓存大小的增加，数据将重新分区。</span><span class="sxs-lookup"><span data-stu-id="a00ee-355">This feature improves scalability, because new Redis servers can be added and the data repartitioned as the size of the cache increases.</span></span>

<span data-ttu-id="a00ee-356">此外，群集中的每一台服务器可以使用主/从复制进行复制。</span><span class="sxs-lookup"><span data-stu-id="a00ee-356">Furthermore, each server in the cluster can be replicated by using master/subordinate replication.</span></span> <span data-ttu-id="a00ee-357">这可确保整个群集中每个节点的可用性。</span><span class="sxs-lookup"><span data-stu-id="a00ee-357">This ensures availability across each node in the cluster.</span></span> <span data-ttu-id="a00ee-358">有关群集和分片的详细信息，请访问 Redis 网站上的 [Redis 群集教程](http://redis.io/topics/cluster-tutorial)页。</span><span class="sxs-lookup"><span data-stu-id="a00ee-358">For more information about clustering and sharding, visit the [Redis cluster tutorial page](http://redis.io/topics/cluster-tutorial) on the Redis website.</span></span>

### <a name="redis-memory-use"></a><span data-ttu-id="a00ee-359">Redis 内存使用</span><span class="sxs-lookup"><span data-stu-id="a00ee-359">Redis memory use</span></span>
<span data-ttu-id="a00ee-360">Redis 缓存具有有限的大小，具体取决于主机计算机上可用的资源。</span><span class="sxs-lookup"><span data-stu-id="a00ee-360">A Redis cache has a finite size that depends on the resources available on the host computer.</span></span> <span data-ttu-id="a00ee-361">在配置 Redis 服务器时，可以指定服务器可使用的最大内存量。</span><span class="sxs-lookup"><span data-stu-id="a00ee-361">When you configure a Redis server, you can specify the maximum amount of memory it can use.</span></span> <span data-ttu-id="a00ee-362">可为 Redis 缓存中的键配置过期时间，到时它会自动从缓存中删除。</span><span class="sxs-lookup"><span data-stu-id="a00ee-362">You can also configure a key in a Redis cache to have an expiration time, after which it is automatically removed from the cache.</span></span> <span data-ttu-id="a00ee-363">此功能可帮助避免内存中缓存填满陈旧或过时的数据。</span><span class="sxs-lookup"><span data-stu-id="a00ee-363">This feature can help prevent the in-memory cache from filling with old or stale data.</span></span>

<span data-ttu-id="a00ee-364">当内存填满时，Redis 可以遵循一些策略自动逐出键及其值。</span><span class="sxs-lookup"><span data-stu-id="a00ee-364">As memory fills up, Redis can automatically evict keys and their values by following a number of policies.</span></span> <span data-ttu-id="a00ee-365">默认策略是 LRU（最近最少使用），但你也可以选择其他策略，例如，随机逐出键，或完全关闭逐出（在此情况下，当缓存已满时，尝试将项添加到缓存会失败）。</span><span class="sxs-lookup"><span data-stu-id="a00ee-365">The default is LRU (least recently used), but you can also select other policies such as evicting keys at random or turning off eviction altogether (in which, case attempts to add items to the cache fail if it is full).</span></span> <span data-ttu-id="a00ee-366">[Using Redis as an LRU cache](http://redis.io/topics/lru-cache)（使用 Redis 作为 LRU 缓存）页提供了详细信息。</span><span class="sxs-lookup"><span data-stu-id="a00ee-366">The page [Using Redis as an LRU cache](http://redis.io/topics/lru-cache) provides more information.</span></span>

### <a name="redis-transactions-and-batches"></a><span data-ttu-id="a00ee-367">Redis 事务和批处理</span><span class="sxs-lookup"><span data-stu-id="a00ee-367">Redis transactions and batches</span></span>
<span data-ttu-id="a00ee-368">Redis 可让客户端应用程序提交一系列的操作，用于在缓存中以原子事务的形式读取和写入数据。</span><span class="sxs-lookup"><span data-stu-id="a00ee-368">Redis enables a client application to submit a series of operations that read and write data in the cache as an atomic transaction.</span></span> <span data-ttu-id="a00ee-369">保证事务中的所有命令按顺序运行，其他并发客户端所发出的命令将不在两者之间交互编排。</span><span class="sxs-lookup"><span data-stu-id="a00ee-369">All the commands in the transaction are guaranteed to run sequentially, and no commands issued by other concurrent clients will be interwoven between them.</span></span>

<span data-ttu-id="a00ee-370">但是，这不是真正的事务，因为关系数据库将执行这些事务。</span><span class="sxs-lookup"><span data-stu-id="a00ee-370">However, these are not true transactions as a relational database would perform them.</span></span> <span data-ttu-id="a00ee-371">事务处理包括两个阶段 -- 在第一个阶段将命令排队，在第二个阶段运行命令。</span><span class="sxs-lookup"><span data-stu-id="a00ee-371">Transaction processing consists of two stages--the first is when the commands are queued, and the second is when the commands are run.</span></span> <span data-ttu-id="a00ee-372">在命令排队阶段，客户端将提交构成事务的命令。</span><span class="sxs-lookup"><span data-stu-id="a00ee-372">During the command queuing stage, the commands that comprise the transaction are submitted by the client.</span></span> <span data-ttu-id="a00ee-373">如果此时发生某种形式的错误（例如语法错误，或参数数目不正确），Redis 将拒绝处理整个事务并将其丢弃。</span><span class="sxs-lookup"><span data-stu-id="a00ee-373">If some sort of error occurs at this point (such as a syntax error, or the wrong number of parameters) then Redis refuses to process the entire transaction and discards it.</span></span>

<span data-ttu-id="a00ee-374">在运行阶段，Redis 将按顺序执行每个队列中的命令。</span><span class="sxs-lookup"><span data-stu-id="a00ee-374">During the run phase, Redis performs each queued command in sequence.</span></span> <span data-ttu-id="a00ee-375">如果在此阶段命令失败，Redis 将继续执行下一个队列中的命令，且它不会回滚任何已运行命令的结果。</span><span class="sxs-lookup"><span data-stu-id="a00ee-375">If a command fails during this phase, Redis continues with the next queued command and does not roll back the effects of any commands that have already been run.</span></span> <span data-ttu-id="a00ee-376">这种简化的事务形式有助于保持性能，并避免争用所造成的性能问题。</span><span class="sxs-lookup"><span data-stu-id="a00ee-376">This simplified form of transaction helps to maintain performance and avoid performance problems that are caused by contention.</span></span>

<span data-ttu-id="a00ee-377">Redis 实施某种形式的乐观锁定，以帮助保持一致性。</span><span class="sxs-lookup"><span data-stu-id="a00ee-377">Redis does implement a form of optimistic locking to assist in maintaining consistency.</span></span> <span data-ttu-id="a00ee-378">有关事务和使用 Redis 进行锁定的详细信息，请访问 Redis 网站上的[事务](http://redis.io/topics/transactions)页。</span><span class="sxs-lookup"><span data-stu-id="a00ee-378">For detailed information about transactions and locking with Redis, visit the [Transactions page](http://redis.io/topics/transactions) on the Redis website.</span></span>

<span data-ttu-id="a00ee-379">Redis 还支持非事务式的请求批处理。</span><span class="sxs-lookup"><span data-stu-id="a00ee-379">Redis also supports non-transactional batching of requests.</span></span> <span data-ttu-id="a00ee-380">客户端用于将命令发送到 Redis 服务器的 Redis 协议可让客户端以同一请求的一部分来发送一系列操作。</span><span class="sxs-lookup"><span data-stu-id="a00ee-380">The Redis protocol that clients use to send commands to a Redis server enables a client to send a series of operations as part of the same request.</span></span> <span data-ttu-id="a00ee-381">这有助于减少网络上的数据包分段。</span><span class="sxs-lookup"><span data-stu-id="a00ee-381">This can help to reduce packet fragmentation on the network.</span></span> <span data-ttu-id="a00ee-382">处理批时，将执行每个命令。</span><span class="sxs-lookup"><span data-stu-id="a00ee-382">When the batch is processed, each command is performed.</span></span> <span data-ttu-id="a00ee-383">如果其中任一命令的格式不当，则将遭到拒绝（对于事务不会发生这种情况），但会执行剩余的命令。</span><span class="sxs-lookup"><span data-stu-id="a00ee-383">If any of these commands are malformed, they will be rejected (which doesn't happen with a transaction), but the remaining commands will be performed.</span></span> <span data-ttu-id="a00ee-384">此外，不保证批中命令的处理顺序。</span><span class="sxs-lookup"><span data-stu-id="a00ee-384">There is also no guarantee about the order in which the commands in the batch will be processed.</span></span>

### <a name="redis-security"></a><span data-ttu-id="a00ee-385">Redis 安全性</span><span class="sxs-lookup"><span data-stu-id="a00ee-385">Redis security</span></span>
<span data-ttu-id="a00ee-386">Redis 专门注重于提供数据快速访问，设计为在受信任的环境中运行，且只能由受信任的客户端访问。</span><span class="sxs-lookup"><span data-stu-id="a00ee-386">Redis is focused purely on providing fast access to data, and is designed to run inside a trusted environment that can be accessed only by trusted clients.</span></span> <span data-ttu-id="a00ee-387">Redis 支持基于密码身份验证的有限安全模型。</span><span class="sxs-lookup"><span data-stu-id="a00ee-387">Redis supports a limited security model based on password authentication.</span></span> <span data-ttu-id="a00ee-388">（可以完全删除身份验证，但不建议这样做。）</span><span class="sxs-lookup"><span data-stu-id="a00ee-388">(It is possible to remove authentication completely, although we don't recommend this.)</span></span>

<span data-ttu-id="a00ee-389">所有已经过身份验证的客户端共享同一个全局密码，并有权访问相同的资源。</span><span class="sxs-lookup"><span data-stu-id="a00ee-389">All authenticated clients share the same global password and have access to the same resources.</span></span> <span data-ttu-id="a00ee-390">如果需要更全面的登录安全性，必须在 Redis 服务器前面实施自己的安全层，并且所有客户端请求应通过此附加层。</span><span class="sxs-lookup"><span data-stu-id="a00ee-390">If you need more comprehensive sign-in security, you must implement your own security layer in front of the Redis server, and all client requests should pass through this additional layer.</span></span> <span data-ttu-id="a00ee-391">不应直接向不受信任或未经身份验证的客户端公开 Redis。</span><span class="sxs-lookup"><span data-stu-id="a00ee-391">Redis should not be directly exposed to untrusted or unauthenticated clients.</span></span>

<span data-ttu-id="a00ee-392">可以通过禁用命令或重命名命令（仅提供有权限的客户端使用新的名称）来限制对命令的访问。</span><span class="sxs-lookup"><span data-stu-id="a00ee-392">You can restrict access to commands by disabling them or renaming them (and by providing only privileged clients with the new names).</span></span>

<span data-ttu-id="a00ee-393">Redis 不直接支持任何形式的数据加密，因此所有编码必须由客户端应用程序执行。</span><span class="sxs-lookup"><span data-stu-id="a00ee-393">Redis does not directly support any form of data encryption, so all encoding must be performed by client applications.</span></span> <span data-ttu-id="a00ee-394">此外，Redis 不提供任何形式的传输安全性。</span><span class="sxs-lookup"><span data-stu-id="a00ee-394">Additionally, Redis does not provide any form of transport security.</span></span> <span data-ttu-id="a00ee-395">如果数据在网络上流动时需要保护数据，建议实施 SSL 代理。</span><span class="sxs-lookup"><span data-stu-id="a00ee-395">If you need to protect data as it flows across the network, we recommend implementing an SSL proxy.</span></span>

<span data-ttu-id="a00ee-396">有关详细信息，请访问 Redis 网站上的 [Redis security](http://redis.io/topics/security)（Redis 安全性）页。</span><span class="sxs-lookup"><span data-stu-id="a00ee-396">For more information, visit the [Redis security](http://redis.io/topics/security) page on the Redis website.</span></span>

> [!NOTE]
> <span data-ttu-id="a00ee-397">Azure Redis 缓存通过连接的客户端提供自身的安全层。</span><span class="sxs-lookup"><span data-stu-id="a00ee-397">Azure Redis Cache provides its own security layer through which clients connect.</span></span> <span data-ttu-id="a00ee-398">底层 Redis 服务器不向公共网络公开。</span><span class="sxs-lookup"><span data-stu-id="a00ee-398">The underlying Redis servers are not exposed to the public network.</span></span>
> 
> 

### <a name="azure-redis-cache"></a><span data-ttu-id="a00ee-399">Azure Redis 缓存</span><span class="sxs-lookup"><span data-stu-id="a00ee-399">Azure Redis cache</span></span>
<span data-ttu-id="a00ee-400">Azure Redis 缓存提供对 Azure 数据中心托管的 Redis 服务器的访问。</span><span class="sxs-lookup"><span data-stu-id="a00ee-400">Azure Redis Cache provides access to Redis servers that are hosted at an Azure datacenter.</span></span> <span data-ttu-id="a00ee-401">它充当提供访问控制与安全性的机制。</span><span class="sxs-lookup"><span data-stu-id="a00ee-401">It acts as a façade that provides access control and security.</span></span> <span data-ttu-id="a00ee-402">可以使用 Azure 门户来预配缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-402">You can provision a cache by using the Azure  portal.</span></span>

<span data-ttu-id="a00ee-403">此门户提供了许多预定义的配置。</span><span class="sxs-lookup"><span data-stu-id="a00ee-403">The portal provides a number of predefined configurations.</span></span> <span data-ttu-id="a00ee-404">这些配置包括支持 SSL 通信（适用于隐私性）以及主/从复制配合 99.9% 可用性的 SLA 的作为专用服务运行的 53 GB 缓存，以及共享硬件上运行不含复制（无可用性保证）的 250 MB 缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-404">These range from a 53 GB cache running as a dedicated service that supports SSL communications (for privacy) and master/subordinate replication with an SLA of 99.9% availability, down to a 250 MB cache without replication (no availability guarantees) running on shared hardware.</span></span>

<span data-ttu-id="a00ee-405">使用 Azure 门户还可以配置缓存的逐出策略，并通过将用户添加到提供的角色来控制缓存的访问权限。</span><span class="sxs-lookup"><span data-stu-id="a00ee-405">Using the Azure portal, you can also configure the eviction policy of the cache, and control access to the cache by adding users to the roles provided.</span></span>  <span data-ttu-id="a00ee-406">这些角色定义成员可以执行的操作，包括所有者、参与者和读取者。</span><span class="sxs-lookup"><span data-stu-id="a00ee-406">These roles, which  define the operations that members can perform, include Owner, Contributor, and Reader.</span></span> <span data-ttu-id="a00ee-407">例如，所有者角色成员拥有缓存（包含安全性）及其内容的完全控制权，参与者角色成员可以在缓存中读取和写入信息，而读取者角色成员只能从缓存检索数据。</span><span class="sxs-lookup"><span data-stu-id="a00ee-407">For example, members of the Owner role have complete control over the cache (including security) and its contents, members of the Contributor role can read and write information in the cache, and members of the Reader role can only retrieve data from the cache.</span></span>

<span data-ttu-id="a00ee-408">大多数管理任务可通过 Azure 门户来执行。</span><span class="sxs-lookup"><span data-stu-id="a00ee-408">Most administrative tasks are performed through the Azure portal.</span></span> <span data-ttu-id="a00ee-409">出于此原因，许多 Redis 标准版中的管理命令都不可用，包括以编程方式修改配置、关闭 Redis 服务器、配置其他从属服务器，或强制将数据存储到磁盘等功能。</span><span class="sxs-lookup"><span data-stu-id="a00ee-409">For this reason, many of the administrative commands that are available in the standard version of Redis are not available, including the ability to modify the configuration programmatically, shut down the Redis server, configure additional subordinates, or forcibly save data to disk.</span></span>

<span data-ttu-id="a00ee-410">Azure 门户拥有便利的图形显示，可通过它监视缓存性能。</span><span class="sxs-lookup"><span data-stu-id="a00ee-410">The Azure portal includes a convenient graphical display that enables you to monitor the performance of the cache.</span></span> <span data-ttu-id="a00ee-411">例如，可以查看创建的连接数、执行的请求数、读取和写入次数，以及缓存命中与缓存未命中次数。</span><span class="sxs-lookup"><span data-stu-id="a00ee-411">For example, you can view the number of connections being made, the number of requests being performed, the volume of reads and writes, and the number of cache hits versus cache misses.</span></span> <span data-ttu-id="a00ee-412">可以使用此信息来确定缓存的效率，并可根据需要切换到不同的配置，或更改逐出策略。</span><span class="sxs-lookup"><span data-stu-id="a00ee-412">Using this information, you can determine the effectiveness of the cache and if necessary, switch to a different configuration or change the eviction policy.</span></span>

<span data-ttu-id="a00ee-413">此外，如果一个或多个关键度量值超过预期范围，可以创建将电子邮件消息发送给管理员的警报。</span><span class="sxs-lookup"><span data-stu-id="a00ee-413">Additionally, you can create alerts that send email messages to an administrator if one or more critical metrics fall outside of an expected range.</span></span> <span data-ttu-id="a00ee-414">例如，如果缓存失误次数在最后一小时超过指定的值，则可能要提醒管理员，因为这意味着缓存可能太小或数据可能逐出得太快。</span><span class="sxs-lookup"><span data-stu-id="a00ee-414">For example, you might want to alert an administrator if the number of cache misses exceeds a specified value in the last hour, because it means the cache might be too small or data might be being evicted too quickly.</span></span>

<span data-ttu-id="a00ee-415">还可以监视缓存的 CPU、内存和网络使用量。</span><span class="sxs-lookup"><span data-stu-id="a00ee-415">You can also monitor the CPU, memory, and network usage for the cache.</span></span>

<span data-ttu-id="a00ee-416">有关说明如何创建和配置 Azure Redis 缓存的更多信息和示例，请访问 Redis 博客上的 [Lap around Azure Redis Cache](https://azure.microsoft.com/blog/2014/06/04/lap-around-azure-redis-cache-preview/)（浏览 Azure Redis 缓存）页。</span><span class="sxs-lookup"><span data-stu-id="a00ee-416">For further information and examples showing how to create and configure an Azure Redis Cache, visit the page [Lap around Azure Redis Cache](https://azure.microsoft.com/blog/2014/06/04/lap-around-azure-redis-cache-preview/) on the Azure blog.</span></span>

## <a name="caching-session-state-and-html-output"></a><span data-ttu-id="a00ee-417">缓存会话状态和 HTML 输出</span><span class="sxs-lookup"><span data-stu-id="a00ee-417">Caching session state and HTML output</span></span>
<span data-ttu-id="a00ee-418">如果要构建通过使用 Azure Web 角色运行的 ASP.NET Web 应用程序，可以会话状态信息和 HTML 输出保存在 Azure Redis 缓存中。</span><span class="sxs-lookup"><span data-stu-id="a00ee-418">If you're building ASP.NET web applications that run by using Azure web roles, you can save session state information and HTML output in an Azure Redis Cache.</span></span> <span data-ttu-id="a00ee-419">Azure Redis 缓存的会话状态提供程序使用户能够在 ASP.NET Web 应用程序的不同实例之间共享会话信息，且它在以下 web 场情况中十分有用：客户端与服务器的相关性不可用并且缓存内存中的会话数据这一行为恰当。</span><span class="sxs-lookup"><span data-stu-id="a00ee-419">The session state provider for Azure Redis Cache enables you to share session information between different instances of an ASP.NET web application, and is very useful in web farm situations where client-server affinity is not available and caching session data in-memory would not be appropriate.</span></span>

<span data-ttu-id="a00ee-420">配合 Azure Redis 缓存使用会话状态提供程序可带来几个好处，包括：</span><span class="sxs-lookup"><span data-stu-id="a00ee-420">Using the session state provider with Azure Redis Cache delivers several benefits, including:</span></span>

* <span data-ttu-id="a00ee-421">在 ASP.NET Web 应用程序的大量实例之间共享会话状态。</span><span class="sxs-lookup"><span data-stu-id="a00ee-421">Sharing session state with a large number of instances of ASP.NET web applications.</span></span>
* <span data-ttu-id="a00ee-422">提供更高的伸缩性。</span><span class="sxs-lookup"><span data-stu-id="a00ee-422">Providing improved scalability.</span></span>
* <span data-ttu-id="a00ee-423">针对多个读取者和单个写入者的同一会话状态数据支持受控的并发访问权限。</span><span class="sxs-lookup"><span data-stu-id="a00ee-423">Supporting controlled, concurrent access to the same session state data for multiple readers and a single writer.</span></span>
* <span data-ttu-id="a00ee-424">使用压缩来节省内存，并提高网络性能。</span><span class="sxs-lookup"><span data-stu-id="a00ee-424">Using compression to save memory and improve network performance.</span></span>

<span data-ttu-id="a00ee-425">有关详细信息，请参阅 [Azure Redis 缓存的 ASP.NET 会话状态提供程序](/azure/redis-cache/cache-aspnet-session-state-provider/)。</span><span class="sxs-lookup"><span data-stu-id="a00ee-425">For more information, see [ASP.NET session state provider for Azure Redis Cache](/azure/redis-cache/cache-aspnet-session-state-provider/).</span></span>

> [!NOTE]
> <span data-ttu-id="a00ee-426">不要针对在 Azure 环境外部运行的 ASP.NET 应用程序使用 Azure Redis 缓存的会话状态提供程序。</span><span class="sxs-lookup"><span data-stu-id="a00ee-426">Do not use the session state provider for Azure Redis Cache with ASP.NET applications that run outside of the Azure environment.</span></span> <span data-ttu-id="a00ee-427">从 Azure 外部访问缓存的延迟会抵消缓存数据带来的性能优势。</span><span class="sxs-lookup"><span data-stu-id="a00ee-427">The latency of accessing the cache from outside of Azure can eliminate the performance benefits of caching data.</span></span>
> 
> 

<span data-ttu-id="a00ee-428">同样地，Azure Redis 缓存的输出缓存提供程序使用户能够保存由 ASP.NET Web 应用程序生成的 HTTP 响应。</span><span class="sxs-lookup"><span data-stu-id="a00ee-428">Similarly, the output cache provider for Azure Redis Cache enables you to save the HTTP responses generated by an ASP.NET web application.</span></span> <span data-ttu-id="a00ee-429">配合 Azure Redis 缓存使用输出缓存提供程序可以针对呈现复杂 HTML 输出的应用程序改善响应时间。</span><span class="sxs-lookup"><span data-stu-id="a00ee-429">Using the output cache provider with Azure Redis Cache can improve the response times of applications that render complex HTML output.</span></span> <span data-ttu-id="a00ee-430">生成类似响应的应用程序实例可以使用缓存中的共享输出段，而不用重新生成此 HTML 输出。</span><span class="sxs-lookup"><span data-stu-id="a00ee-430">Application instances that generate similar responses can make use of the shared output fragments in the cache rather than generating this HTML output afresh.</span></span> <span data-ttu-id="a00ee-431">有关详细信息，请参阅 [Azure Redis 缓存的 ASP.NET 输出缓存提供程序](/azure/redis-cache/cache-aspnet-output-cache-provider/)。</span><span class="sxs-lookup"><span data-stu-id="a00ee-431">For more information, see [ASP.NET output cache provider for Azure Redis Cache](/azure/redis-cache/cache-aspnet-output-cache-provider/).</span></span>

## <a name="building-a-custom-redis-cache"></a><span data-ttu-id="a00ee-432">构建自定义 Redis 缓存</span><span class="sxs-lookup"><span data-stu-id="a00ee-432">Building a custom Redis cache</span></span>
<span data-ttu-id="a00ee-433">Azure Redis 缓存充当底层 Redis 服务器的机制。</span><span class="sxs-lookup"><span data-stu-id="a00ee-433">Azure Redis Cache acts as a façade to the underlying Redis servers.</span></span> <span data-ttu-id="a00ee-434">目前它支持固定的一组配置，但没有 Redis 群集提供配置。</span><span class="sxs-lookup"><span data-stu-id="a00ee-434">Currently it supports a fixed set of configurations but does not provide for Redis clustering.</span></span> <span data-ttu-id="a00ee-435">如果需要 Azure Redis 缓存未涵盖的高级配置（例如大于 53 GB 的缓存），可以使用 Azure 虚拟机来构建和托管自己的 Redis 服务器。</span><span class="sxs-lookup"><span data-stu-id="a00ee-435">If you require an advanced configuration that is not covered by the Azure Redis cache (such as a cache bigger than 53 GB) you can build and host your own Redis servers by using Azure virtual machines.</span></span>

<span data-ttu-id="a00ee-436">你在实施复制时可能需要创建多个 VM 作为主节点和从属节点，这可能是一个复杂的过程。</span><span class="sxs-lookup"><span data-stu-id="a00ee-436">This is a potentially complex process because you might need to create several VMs to act as master and subordinate nodes if you want to implement replication.</span></span> <span data-ttu-id="a00ee-437">此外，如果想要创建群集，需要多个主服务器和从属服务器。</span><span class="sxs-lookup"><span data-stu-id="a00ee-437">Furthermore, if you wish to create a cluster, then you need multiple masters and subordinate servers.</span></span> <span data-ttu-id="a00ee-438">一个可以提供高度可用性和伸缩性，并且至少包含 6 个 VM 并组织成 3 对主/从服务器（一个群集必须至少包含 3 个主节点）的精简群集复制拓扑。</span><span class="sxs-lookup"><span data-stu-id="a00ee-438">A minimal clustered replication topology that provides a high degree of availability and scalability comprises at least six VMs organized as three pairs of master/subordinate servers (a cluster must contain at least three master nodes).</span></span>

<span data-ttu-id="a00ee-439">每个主/从对应彼此靠近以降低延迟。</span><span class="sxs-lookup"><span data-stu-id="a00ee-439">Each master/subordinate pair should be located close together to minimize latency.</span></span> <span data-ttu-id="a00ee-440">但如果想要找出靠近的应用程序（该应用程序很可能会使用缓存数据），每一组对可以在位于不同区域的不同 Azure 数据中心运行。</span><span class="sxs-lookup"><span data-stu-id="a00ee-440">However, each set of pairs can be running in different Azure datacenters located in different regions, if you wish to locate cached data close to the applications that are most likely to use it.</span></span>  <span data-ttu-id="a00ee-441">有关生成和配置作为 Azure VM 运行的 Redis 节点的示例，请参阅 [Running Redis on a CentOS Linux VM in Azure](http://blogs.msdn.com/b/tconte/archive/2012/06/08/running-redis-on-a-centos-linux-vm-in-windows-azure.aspx)（在 Azure 中的 CentOS Linux VM 上运行 Redis）。</span><span class="sxs-lookup"><span data-stu-id="a00ee-441">For an example of building and configuring a Redis node running as an Azure VM, see [Running Redis on a CentOS Linux VM in Azure](http://blogs.msdn.com/b/tconte/archive/2012/06/08/running-redis-on-a-centos-linux-vm-in-windows-azure.aspx).</span></span>

> [!NOTE]
> <span data-ttu-id="a00ee-442">请注意，如果以这种方式实施自己的 Redis 缓存，则需要负责监视、管理和保护服务。</span><span class="sxs-lookup"><span data-stu-id="a00ee-442">Please note that if you implement your own Redis cache in this way, you are responsible for monitoring, managing, and securing the service.</span></span>
> 

## <a name="partitioning-a-redis-cache"></a><span data-ttu-id="a00ee-443">将 Redis 缓存分区</span><span class="sxs-lookup"><span data-stu-id="a00ee-443">Partitioning a Redis cache</span></span>
<span data-ttu-id="a00ee-444">将缓存分区涉及到在多台计算机之间拆分缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-444">Partitioning the cache involves splitting the cache across multiple computers.</span></span> <span data-ttu-id="a00ee-445">此结构使用单个缓存服务器，可以提供多种优势，包括：</span><span class="sxs-lookup"><span data-stu-id="a00ee-445">This structure gives you several advantages over using a single cache server, including:</span></span>

* <span data-ttu-id="a00ee-446">创建的缓存比单个服务器上存储的缓存要大得多。</span><span class="sxs-lookup"><span data-stu-id="a00ee-446">Creating a cache that is much bigger than can be stored on a single server.</span></span>
* <span data-ttu-id="a00ee-447">将数据分散到多个服务器，从而提高可用性。</span><span class="sxs-lookup"><span data-stu-id="a00ee-447">Distributing data across servers, improving availability.</span></span> <span data-ttu-id="a00ee-448">如果一台服务器发生故障或不可访问，该服务器保存的数据将不可用，但剩余服务器上的数据仍可访问。</span><span class="sxs-lookup"><span data-stu-id="a00ee-448">If one server fails or becomes inaccessible, the data that it holds is unavailable, but the data on the remaining servers can still be accessed.</span></span> <span data-ttu-id="a00ee-449">对于缓存而言这并不重要，因为缓存数据只是数据库中暂时保存的数据副本。</span><span class="sxs-lookup"><span data-stu-id="a00ee-449">For a cache, this is not crucial because the cached data is only a transient copy of the data that's held in a database.</span></span> <span data-ttu-id="a00ee-450">不可访问的服务器上的缓存数据可以改为在不同的服务器上缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-450">Cached data on a server that becomes inaccessible can be cached on a different server instead.</span></span>
* <span data-ttu-id="a00ee-451">在服务器之间分散负载，从而提高性能和伸缩性。</span><span class="sxs-lookup"><span data-stu-id="a00ee-451">Spreading the load across servers, thereby improving performance and scalability.</span></span>
* <span data-ttu-id="a00ee-452">将数据放置在靠近用户访问的地理位置以降低延迟。</span><span class="sxs-lookup"><span data-stu-id="a00ee-452">Geolocating data close to the users that access it, thus reducing latency.</span></span>

<span data-ttu-id="a00ee-453">对于缓存，最常见的分区形式是分片。</span><span class="sxs-lookup"><span data-stu-id="a00ee-453">For a cache, the most common form of partitioning is sharding.</span></span> <span data-ttu-id="a00ee-454">在此策略中，每个分区（或分片）本身是一个 Redis 缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-454">In this strategy, each partition (or shard) is a Redis cache in its own right.</span></span> <span data-ttu-id="a00ee-455">数据使用分片逻辑定向到特定的分区，该逻辑可以使用各种方法来分布数据。</span><span class="sxs-lookup"><span data-stu-id="a00ee-455">Data is directed to a specific partition by using sharding logic, which can use a variety of approaches to distribute the data.</span></span> <span data-ttu-id="a00ee-456">[Sharding pattern](http://msdn.microsoft.com/library/dn589797.aspx)（分片模式）提供了有关实施分片的详细信息。</span><span class="sxs-lookup"><span data-stu-id="a00ee-456">The [Sharding pattern](http://msdn.microsoft.com/library/dn589797.aspx) provides more information about implementing sharding.</span></span>

<span data-ttu-id="a00ee-457">若要在 Redis 缓存中实施分区，可以采用以下方法之一：</span><span class="sxs-lookup"><span data-stu-id="a00ee-457">To implement partitioning in a Redis cache, you can take one of the following approaches:</span></span>

* <span data-ttu-id="a00ee-458">*服务器端查询路由。*</span><span class="sxs-lookup"><span data-stu-id="a00ee-458">*Server-side query routing.*</span></span> <span data-ttu-id="a00ee-459">使用此方法时，客户端应用程序会将请求发送到构成缓存的任何 Redis 服务器（可能是最靠近的服务器）。</span><span class="sxs-lookup"><span data-stu-id="a00ee-459">In this technique, a client application sends a request to any of the Redis servers that comprise the cache (probably the closest server).</span></span> <span data-ttu-id="a00ee-460">每个 Redis 服务器将存储用于描述它所保存的分区的元数据，同时还包含有关哪些分区位于其他服务器上的信息。</span><span class="sxs-lookup"><span data-stu-id="a00ee-460">Each Redis server stores metadata that describes the partition that it holds, and also contains information about which partitions are located on other servers.</span></span> <span data-ttu-id="a00ee-461">Redis 服务器检查客户端请求。</span><span class="sxs-lookup"><span data-stu-id="a00ee-461">The Redis server examines the client request.</span></span> <span data-ttu-id="a00ee-462">如果可以在本地解决，则执行请求的操作。</span><span class="sxs-lookup"><span data-stu-id="a00ee-462">If it can be resolved locally, it will perform the requested operation.</span></span> <span data-ttu-id="a00ee-463">否则将请求转发到相应的服务器。</span><span class="sxs-lookup"><span data-stu-id="a00ee-463">Otherwise it will forward the request on to the appropriate server.</span></span> <span data-ttu-id="a00ee-464">此模型是通过 Redis 群集实施的，Redis 网站上的 [Redis 群集教程](http://redis.io/topics/cluster-tutorial)页上提供了更详细的说明。</span><span class="sxs-lookup"><span data-stu-id="a00ee-464">This model is implemented by Redis clustering, and is described in more detail on the [Redis cluster tutorial](http://redis.io/topics/cluster-tutorial) page on the Redis website.</span></span> <span data-ttu-id="a00ee-465">Redis 群集对客户端应用程序而言是透明的，其他 Redis 服务器可以添加到群集（数据将重新分区），而无需重新配置客户端。</span><span class="sxs-lookup"><span data-stu-id="a00ee-465">Redis clustering is transparent to client applications, and additional Redis servers can be added to the cluster (and the data re-partitioned) without requiring that you reconfigure the clients.</span></span>
* <span data-ttu-id="a00ee-466">*客户端分区。*</span><span class="sxs-lookup"><span data-stu-id="a00ee-466">*Client-side partitioning.*</span></span> <span data-ttu-id="a00ee-467">在此模型中，客户端应用程序包含将请求路由到适当 Redis 服务器的逻辑（可能以库的形式）。</span><span class="sxs-lookup"><span data-stu-id="a00ee-467">In this model, the client application contains logic (possibly in the form of a library) that routes requests to the appropriate Redis server.</span></span> <span data-ttu-id="a00ee-468">这种方法可以配合 Azure Redis 缓存使用。</span><span class="sxs-lookup"><span data-stu-id="a00ee-468">This approach can be used with Azure Redis Cache.</span></span> <span data-ttu-id="a00ee-469">创建多个 Azure Redis 缓存（每个数据分区一个缓存），并实施将请求路由到正确缓存的客户端逻辑。</span><span class="sxs-lookup"><span data-stu-id="a00ee-469">Create multiple Azure Redis Caches (one for each data partition) and implement the client-side logic that routes the requests to the correct cache.</span></span> <span data-ttu-id="a00ee-470">如果分区方案发生更改（例如，如果已创建其他 Azure Redis 缓存），则可能需要重新配置客户端应用程序。</span><span class="sxs-lookup"><span data-stu-id="a00ee-470">If the partitioning scheme changes (if additional Azure Redis Caches are created, for example), client applications might need to be reconfigured.</span></span>
* <span data-ttu-id="a00ee-471">*代理辅助分区。*</span><span class="sxs-lookup"><span data-stu-id="a00ee-471">*Proxy-assisted partitioning.*</span></span> <span data-ttu-id="a00ee-472">在此方案中，客户端应用程序将请求发送到一个知道如何数据分区方式的中间代理服务，然后将请求路由到适当的 Redis 服务器。</span><span class="sxs-lookup"><span data-stu-id="a00ee-472">In this scheme, client applications send requests to an intermediary proxy service which understands how the data is partitioned and then routes the request to the appropriate Redis server.</span></span> <span data-ttu-id="a00ee-473">此方法也可以配合 Azure Redis 缓存使用；代理服务可以实施为 Azure 云服务。</span><span class="sxs-lookup"><span data-stu-id="a00ee-473">This approach can also be used with Azure Redis Cache; the proxy service can be implemented as an Azure cloud service.</span></span> <span data-ttu-id="a00ee-474">使用此方法实施服务需要提高复杂性，并且执行请求的时间可能比使用客户端分区更长。</span><span class="sxs-lookup"><span data-stu-id="a00ee-474">This approach requires an additional level of complexity to implement the service, and requests might take longer to perform than using client-side partitioning.</span></span>

<span data-ttu-id="a00ee-475">Redis 网站上的 [Partitioning: how to split data among multiple Redis instances](http://redis.io/topics/partitioning)（分区：如何在多个 Redis 实例之间拆分数据）页提供了有关使用 Redis 实施分区的更多信息。</span><span class="sxs-lookup"><span data-stu-id="a00ee-475">The page [Partitioning: how to split data among multiple Redis instances](http://redis.io/topics/partitioning) on the Redis website provides further information about implementing partitioning with Redis.</span></span>

### <a name="implement-redis-cache-client-applications"></a><span data-ttu-id="a00ee-476">实施 Redis 缓存客户端应用程序</span><span class="sxs-lookup"><span data-stu-id="a00ee-476">Implement Redis cache client applications</span></span>
<span data-ttu-id="a00ee-477">Redis 支持以多种编程语言编写的客户端应用程序。</span><span class="sxs-lookup"><span data-stu-id="a00ee-477">Redis supports client applications written in numerous programming languages.</span></span> <span data-ttu-id="a00ee-478">如果要使用.NET Framework 构建新的应用程序，建议的方法是使用 StackExchange.Redis 客户端库。</span><span class="sxs-lookup"><span data-stu-id="a00ee-478">If you are building new applications by using the .NET Framework, the recommended approach is to use the StackExchange.Redis client library.</span></span> <span data-ttu-id="a00ee-479">此库提供 .NET Framework 对象模型，用于抽象连接到 Redis 服务器连接、发送命令和接收响应所需的详细信息。</span><span class="sxs-lookup"><span data-stu-id="a00ee-479">This library provides a .NET Framework object model that abstracts the details for connecting to a Redis server, sending commands, and receiving responses.</span></span> <span data-ttu-id="a00ee-480">在 Visual Studio 中，它以 NuGet 包的形式提供。</span><span class="sxs-lookup"><span data-stu-id="a00ee-480">It is available in Visual Studio as a NuGet package.</span></span> <span data-ttu-id="a00ee-481">可以使用同一个库连接到 Azure Redis 缓存，或者 VM 上托管的自定义 Redis 缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-481">You can use this same library to connect to an Azure Redis Cache, or a custom Redis cache hosted on a VM.</span></span>

<span data-ttu-id="a00ee-482">若要连接到 Redis 服务器，可以使用 `ConnectionMultiplexer` 类的静态 `Connect` 方法。</span><span class="sxs-lookup"><span data-stu-id="a00ee-482">To connect to a Redis server you use the static `Connect` method of the `ConnectionMultiplexer` class.</span></span> <span data-ttu-id="a00ee-483">此方法创建的连接可在客户端应用程序的整个生存期内使用，同一个连接可由多个并发线程使用。</span><span class="sxs-lookup"><span data-stu-id="a00ee-483">The connection that this method creates is designed to be used throughout the lifetime of the client application, and the same connection can be used by multiple concurrent threads.</span></span> <span data-ttu-id="a00ee-484">每次执行 Redis 操作时，请不要重新连接和断开连接，因为这可能会降低性能。</span><span class="sxs-lookup"><span data-stu-id="a00ee-484">Do not reconnect and disconnect each time you perform a Redis operation because this can degrade performance.</span></span>

<span data-ttu-id="a00ee-485">可以指定连接参数，例如 Redis 主机的地址和密码。</span><span class="sxs-lookup"><span data-stu-id="a00ee-485">You can specify the connection parameters, such as the address of the Redis host and the password.</span></span> <span data-ttu-id="a00ee-486">如果使用 Azure Redis 缓存，密码可能是使用 Azure 管理门户针对 Azure Redis 缓存生成的主密钥或辅助密钥。</span><span class="sxs-lookup"><span data-stu-id="a00ee-486">If you are using Azure Redis Cache, the password  is either the primary or secondary key that is generated for Azure Redis Cache by using the Azure Management portal.</span></span>

<span data-ttu-id="a00ee-487">在已连接到 Redis 服务器后，可以在用作缓存的 Redis 数据库上获取句柄。</span><span class="sxs-lookup"><span data-stu-id="a00ee-487">After you have connected to the Redis server, you can obtain a handle on the Redis database that acts as the cache.</span></span> <span data-ttu-id="a00ee-488">Redis 连接提供了 `GetDatabase` 方法来执行此操作。</span><span class="sxs-lookup"><span data-stu-id="a00ee-488">The Redis connection provides the `GetDatabase` method to do this.</span></span> <span data-ttu-id="a00ee-489">然后，可以使用 `StringGet` 和 `StringSet` 方法，从缓存中检索项并在缓存中存储数据。</span><span class="sxs-lookup"><span data-stu-id="a00ee-489">You can then retrieve items from the cache and store data in the cache by using the `StringGet` and `StringSet` methods.</span></span> <span data-ttu-id="a00ee-490">这些方法需要将键用作参数，并返回缓存中具有匹配值的项 (`StringGet`)，或者将项添加到具有此键的缓存 (`StringSet`)。</span><span class="sxs-lookup"><span data-stu-id="a00ee-490">These methods expect a key as a parameter, and return the item either in the cache that has a matching value (`StringGet`) or add the item to the cache with this key (`StringSet`).</span></span>

<span data-ttu-id="a00ee-491">根据 Redis 服务器的位置，在将请求传输到服务器以及将响应返回给客户端时，许多操作可能会造成一些延迟。</span><span class="sxs-lookup"><span data-stu-id="a00ee-491">Depending on the location of the Redis server, many operations might incur some latency while a request is transmitted to the server and a response is returned to the client.</span></span> <span data-ttu-id="a00ee-492">StackExchange 库公开了许多方法的异步版本，用于帮助客户端应用程序保持响应。</span><span class="sxs-lookup"><span data-stu-id="a00ee-492">The StackExchange library provides asynchronous versions of many of the methods that it exposes to help client applications remain responsive.</span></span> <span data-ttu-id="a00ee-493">这些方法支持 .NET Framework 中的[基于任务的异步模式](http://msdn.microsoft.com/library/hh873175.aspx)。</span><span class="sxs-lookup"><span data-stu-id="a00ee-493">These methods support the [Task-based Asynchronous Pattern](http://msdn.microsoft.com/library/hh873175.aspx) in the .NET Framework.</span></span>

<span data-ttu-id="a00ee-494">以下代码片段显示了名为 `RetrieveItem` 的方法。</span><span class="sxs-lookup"><span data-stu-id="a00ee-494">The following code snippet shows a method named `RetrieveItem`.</span></span> <span data-ttu-id="a00ee-495">其中演示了基于 Redis 和 StackExchange 库的缓存端模式的实现。</span><span class="sxs-lookup"><span data-stu-id="a00ee-495">It illustrates an implementation of the cache-aside pattern based on Redis and the StackExchange library.</span></span> <span data-ttu-id="a00ee-496">该方法采用字符串键值，并通过调用 `StringGetAsync` 方法（`StringGet` 的异步版本）尝试从 Redis 缓存中检索相应的项。</span><span class="sxs-lookup"><span data-stu-id="a00ee-496">The method takes a string key value and attempts to retrieve the corresponding item from the Redis cache by calling the `StringGetAsync` method (the asynchronous version of `StringGet`).</span></span>

<span data-ttu-id="a00ee-497">如果找不到该项，则使用 `GetItemFromDataSourceAsync` 方法（这是一个本地方法，它不是 StackExchange 库的一部分）从底层数据源提取该项。</span><span class="sxs-lookup"><span data-stu-id="a00ee-497">If the item is not found, it is fetched from the underlying data source using the `GetItemFromDataSourceAsync` method (which is a local method and not part of the StackExchange library).</span></span> <span data-ttu-id="a00ee-498">然后，使用 `StringSetAsync` 方法将该项添加到缓存，以便下一次可以更快地检索。</span><span class="sxs-lookup"><span data-stu-id="a00ee-498">It's then added to the cache by using the `StringSetAsync` method so it can be retrieved more quickly next time.</span></span>

```csharp
// Connect to the Azure Redis cache
ConfigurationOptions config = new ConfigurationOptions();
config.EndPoints.Add("<your DNS name>.redis.cache.windows.net");
config.Password = "<Redis cache key from management portal>";
ConnectionMultiplexer redisHostConnection = ConnectionMultiplexer.Connect(config);
IDatabase cache = redisHostConnection.GetDatabase();
...
private async Task<string> RetrieveItem(string itemKey)
{
    // Attempt to retrieve the item from the Redis cache
    string itemValue = await cache.StringGetAsync(itemKey);

    // If the value returned is null, the item was not found in the cache
    // So retrieve the item from the data source and add it to the cache
    if (itemValue == null)
    {
        itemValue = await GetItemFromDataSourceAsync(itemKey);
        await cache.StringSetAsync(itemKey, itemValue);
    }

    // Return the item
    return itemValue;
}
```

<span data-ttu-id="a00ee-499">`StringGet` 和 `StringSet` 方法不是只能检索或存储字符串值。</span><span class="sxs-lookup"><span data-stu-id="a00ee-499">The `StringGet` and `StringSet` methods are not restricted to retrieving or storing string values.</span></span> <span data-ttu-id="a00ee-500">它们可以采用任何序列化为字节数组的项。</span><span class="sxs-lookup"><span data-stu-id="a00ee-500">They can take any item that is serialized as an array of bytes.</span></span> <span data-ttu-id="a00ee-501">如果需要保存 .NET 对象，可以将它序列化为字节流，然后使用 `StringSet` 方法将它写入缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-501">If you need to save a .NET object, you can serialize it as a byte stream and use the `StringSet` method to write it to the cache.</span></span>

<span data-ttu-id="a00ee-502">同样地，可以使用 `StringGet` 方法从缓存中读取对象，并将其反序列化为 .NET 对象。</span><span class="sxs-lookup"><span data-stu-id="a00ee-502">Similarly, you can read an object from the cache by using the `StringGet` method and deserializing it as a .NET object.</span></span> <span data-ttu-id="a00ee-503">以下代码演示了 IDatabase 接口的一组扩展方法（Redis 连接的 `GetDatabase` 方法返回 `IDatabase` 对象），使用这些方法的某些示例代码可以在缓存中读取和写入 `BlogPost` 对象：</span><span class="sxs-lookup"><span data-stu-id="a00ee-503">The following code shows a set of extension methods for the IDatabase interface (the `GetDatabase` method of a Redis connection returns an `IDatabase` object),  and some sample code that uses these methods to read and write a `BlogPost` object to the cache:</span></span>

```csharp
public static class RedisCacheExtensions
{
    public static async Task<T> GetAsync<T>(this IDatabase cache, string key)
    {
        return Deserialize<T>(await cache.StringGetAsync(key));
    }

    public static async Task<object> GetAsync(this IDatabase cache, string key)
    {
        return Deserialize<object>(await cache.StringGetAsync(key));
    }

    public static async Task SetAsync(this IDatabase cache, string key, object value)
    {
        await cache.StringSetAsync(key, Serialize(value));
    }

    static byte[] Serialize(object o)
    {
        byte[] objectDataAsStream = null;

        if (o != null)
        {
            BinaryFormatter binaryFormatter = new BinaryFormatter();
            using (MemoryStream memoryStream = new MemoryStream())
            {
                binaryFormatter.Serialize(memoryStream, o);
                objectDataAsStream = memoryStream.ToArray();
            }
        }

        return objectDataAsStream;
    }

    static T Deserialize<T>(byte[] stream)
    {
        T result = default(T);

        if (stream != null)
        {
            BinaryFormatter binaryFormatter = new BinaryFormatter();
            using (MemoryStream memoryStream = new MemoryStream(stream))
            {
                result = (T)binaryFormatter.Deserialize(memoryStream);
            }
        }

        return result;
    }
}
```

<span data-ttu-id="a00ee-504">以下代码演示了一个名为 `RetrieveBlogPost` 的方法，该方法使用这些扩展方法，遵循缓存端模式在缓存中读取和写入可序列化的 `BlogPost` 对象：</span><span class="sxs-lookup"><span data-stu-id="a00ee-504">The following code illustrates a method named `RetrieveBlogPost` that uses these extension methods to read and write a serializable `BlogPost` object to the cache following the cache-aside pattern:</span></span>

```csharp
// The BlogPost type
[Serializable]
public class BlogPost
{
    private HashSet<string> tags;

    public BlogPost(int id, string title, int score, IEnumerable<string> tags)
    {
        this.Id = id;
        this.Title = title;
        this.Score = score;
        this.tags = new HashSet<string>(tags);
    }

    public int Id { get; set; }
    public string Title { get; set; }
    public int Score { get; set; }
    public ICollection<string> Tags => this.tags;
}
...
private async Task<BlogPost> RetrieveBlogPost(string blogPostKey)
{
    BlogPost blogPost = await cache.GetAsync<BlogPost>(blogPostKey);
    if (blogPost == null)
    {
        blogPost = await GetBlogPostFromDataSourceAsync(blogPostKey);
        await cache.SetAsync(blogPostKey, blogPost);
    }

    return blogPost;
}
```

<span data-ttu-id="a00ee-505">如果客户端应用程序发送了多个异步请求，Redis 将支持命令管道。</span><span class="sxs-lookup"><span data-stu-id="a00ee-505">Redis supports command pipelining if a client application sends multiple asynchronous requests.</span></span> <span data-ttu-id="a00ee-506">Redis 可以使用同一连接来多路复用请求，而不是按照严格的顺序来接收和响应命令。</span><span class="sxs-lookup"><span data-stu-id="a00ee-506">Redis can multiplex the requests using the same connection rather than receiving and responding to commands in a strict sequence.</span></span>

<span data-ttu-id="a00ee-507">此方法可以更有效地使用网络来帮助降低延迟。</span><span class="sxs-lookup"><span data-stu-id="a00ee-507">This approach helps to reduce latency by making more efficient use of the network.</span></span> <span data-ttu-id="a00ee-508">以下代码段演示了并行检索两个客户的详细信息的示例。</span><span class="sxs-lookup"><span data-stu-id="a00ee-508">The following code snippet shows an example that retrieves the details of two customers concurrently.</span></span> <span data-ttu-id="a00ee-509">该代码将提交两个请求，再执行其他某种处理（未显示），然后等待接收结果。</span><span class="sxs-lookup"><span data-stu-id="a00ee-509">The code submits two requests and then performs some other processing (not shown) before waiting to receive the results.</span></span> <span data-ttu-id="a00ee-510">缓存对象的 `Wait` 方法类似于 .NET Framework `Task.Wait` 方法：</span><span class="sxs-lookup"><span data-stu-id="a00ee-510">The `Wait` method of the cache object is similar to the .NET Framework `Task.Wait` method:</span></span>

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
var task1 = cache.StringGetAsync("customer:1");
var task2 = cache.StringGetAsync("customer:2");
...
var customer1 = cache.Wait(task1);
var customer2 = cache.Wait(task2);
```

<span data-ttu-id="a00ee-511">有关编写可使用 Azure Redis 缓存的客户端应用程序的其他信息，请参阅 [Azure Redis 缓存文档](https://azure.microsoft.com/documentation/services/cache/)。</span><span class="sxs-lookup"><span data-stu-id="a00ee-511">For additional information on writing client applications that can the Azure Redis Cache, see [Azure Redis Cache documentation](https://azure.microsoft.com/documentation/services/cache/).</span></span> <span data-ttu-id="a00ee-512">请在 [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Basics.md) 处查看详细信息。</span><span class="sxs-lookup"><span data-stu-id="a00ee-512">More information is also available at [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Basics.md).</span></span>

<span data-ttu-id="a00ee-513">同一网站上的 [Pipelines and multiplexers](https://stackexchange.github.io/StackExchange.Redis/PipelinesMultiplexers)（管道与多路复用器）页提供了有关使用 Redis 和 StackExchange 库执行异步操作和管道传输的详细信息。</span><span class="sxs-lookup"><span data-stu-id="a00ee-513">The page [Pipelines and multiplexers](https://stackexchange.github.io/StackExchange.Redis/PipelinesMultiplexers) on the same website provides more information about asynchronous operations and pipelining with Redis and the StackExchange library.</span></span>  <span data-ttu-id="a00ee-514">本文的下一部分“使用 Redis 缓存”提供了一些更高级技巧的示例，用户可以对 Redis 缓存中保存的数据运用这些技巧。</span><span class="sxs-lookup"><span data-stu-id="a00ee-514">The next section in this article, Using  Redis Caching, provides examples of some of the more advanced techniques that you can apply to data that's held in a Redis cache.</span></span>

## <a name="using-redis-caching"></a><span data-ttu-id="a00ee-515">使用 Redis 缓存</span><span class="sxs-lookup"><span data-stu-id="a00ee-515">Using Redis caching</span></span>
<span data-ttu-id="a00ee-516">Redis 缓存的最简单用法包括存储键-值对，其中的值是未解释的字符串，该字符串具有任意长度，可以包含任何二进制数据。</span><span class="sxs-lookup"><span data-stu-id="a00ee-516">The simplest use of Redis for caching concerns is key-value pairs where the value is an uninterpreted string of arbitrary length that can contain any binary data.</span></span> <span data-ttu-id="a00ee-517">（本质上是可视为字符串的字节数组。）</span><span class="sxs-lookup"><span data-stu-id="a00ee-517">(It is essentially  an array of bytes that can be treated as a string).</span></span> <span data-ttu-id="a00ee-518">本文前面的“实施 Redis 缓存客户端应用程序”部分中已演示这种方案。</span><span class="sxs-lookup"><span data-stu-id="a00ee-518">This scenario was illustrated in the section Implement Redis Cache client applications earlier in this article.</span></span>

<span data-ttu-id="a00ee-519">请注意，键还包含未解释的数据，因此，可以使用任何二进制信息作为键。</span><span class="sxs-lookup"><span data-stu-id="a00ee-519">Note that keys also contain uninterpreted data, so you can use any binary information as the key.</span></span> <span data-ttu-id="a00ee-520">但键越长，存储花费的空间就越多，执行查找操作所需的时间也越长。</span><span class="sxs-lookup"><span data-stu-id="a00ee-520">The longer the key is, however, the more space it will take to store, and the longer it will take to perform lookup operations.</span></span> <span data-ttu-id="a00ee-521">为了实现可用性和易维护性，请认真设计键空间并使用有意义（但非详细）的键。</span><span class="sxs-lookup"><span data-stu-id="a00ee-521">For usability and ease of maintenance, design your keyspace carefully and use meaningful (but not verbose) keys.</span></span>

<span data-ttu-id="a00ee-522">例如，使用类似于“customer:100”的结构化键来表示 ID 为 100 的客户的键，而不是简单地使用“100”。</span><span class="sxs-lookup"><span data-stu-id="a00ee-522">For example, use structured keys such as "customer:100" to represent the key for the customer with ID 100 rather than simply "100".</span></span> <span data-ttu-id="a00ee-523">使用此方案可以轻松区分存储不同数据类型的值。</span><span class="sxs-lookup"><span data-stu-id="a00ee-523">This scheme enables you to easily distinguish between values that store different data types.</span></span> <span data-ttu-id="a00ee-524">例如，也可以使用键“orders:100”来表示 ID为 100 的订单的键。</span><span class="sxs-lookup"><span data-stu-id="a00ee-524">For example, you could also use the key "orders:100" to represent the key for the order with ID 100.</span></span>

<span data-ttu-id="a00ee-525">除了一维二进制字符串以外，Redis 键-值对中的值还可以包含更结构化的信息，包括列表、集（已排序和未排序）和哈希。</span><span class="sxs-lookup"><span data-stu-id="a00ee-525">Apart from one-dimensional binary strings, a value in a Redis key-value pair can also hold more structured information, including lists, sets (sorted and unsorted), and hashes.</span></span> <span data-ttu-id="a00ee-526">Redis 提供全面的命令集用于处理这些类型，其中的许多命令可以通过 StackExchange 等客户端库用于 .NET Framework 应用程序。</span><span class="sxs-lookup"><span data-stu-id="a00ee-526">Redis provides a comprehensive command set that can manipulate these types, and many of these commands are available to .NET Framework applications through a client library such as StackExchange.</span></span> <span data-ttu-id="a00ee-527">Redis 网站上的 [An introduction to Redis data types and abstractions](http://redis.io/topics/data-types-intro)（Redis 数据类型和抽象简介）页更详细地概述了这些类型以及可用于处理这些类型的命令。</span><span class="sxs-lookup"><span data-stu-id="a00ee-527">The page [An introduction to Redis data types and abstractions](http://redis.io/topics/data-types-intro) on the Redis website provides a more detailed overview of these types and the commands that you can use to manipulate them.</span></span>

<span data-ttu-id="a00ee-528">本部分汇总了这些数据类型和命令的一些常见用例。</span><span class="sxs-lookup"><span data-stu-id="a00ee-528">This section summarizes some common use cases for these data types and commands.</span></span>

### <a name="perform-atomic-and-batch-operations"></a><span data-ttu-id="a00ee-529">执行原子操作和批处理操作</span><span class="sxs-lookup"><span data-stu-id="a00ee-529">Perform atomic and batch operations</span></span>
<span data-ttu-id="a00ee-530">Redis 支持对字符串值执行一系列原子性“获取和设置”操作。</span><span class="sxs-lookup"><span data-stu-id="a00ee-530">Redis supports a series of atomic get-and-set operations on string values.</span></span> <span data-ttu-id="a00ee-531">这些操作将删除使用单独的 `GET` 和 `SET` 命令时可能发生的争用风险。</span><span class="sxs-lookup"><span data-stu-id="a00ee-531">These operations remove the possible race hazards that might occur when using separate `GET` and `SET` commands.</span></span> <span data-ttu-id="a00ee-532">可用的操作包括：</span><span class="sxs-lookup"><span data-stu-id="a00ee-532">The operations that are available include:</span></span>

* <span data-ttu-id="a00ee-533">`INCR`、`INCRBY`、`DECR` 和 `DECRBY`，用于对整数数字数据值执行原子递增和递减操作。</span><span class="sxs-lookup"><span data-stu-id="a00ee-533">`INCR`, `INCRBY`, `DECR`, and `DECRBY`, which perform atomic increment and decrement operations on integer numeric data values.</span></span> <span data-ttu-id="a00ee-534">StackExchange 库提供了 `IDatabase.StringIncrementAsync` 和 `IDatabase.StringDecrementAsync` 方法的重载版本，用于执行这些操作并返回存储在缓存中的结果值。</span><span class="sxs-lookup"><span data-stu-id="a00ee-534">The StackExchange library provides overloaded versions of the `IDatabase.StringIncrementAsync` and `IDatabase.StringDecrementAsync` methods to perform these operations and return the resulting value that is stored in the cache.</span></span> <span data-ttu-id="a00ee-535">以下代码段演示了如何使用这些方法：</span><span class="sxs-lookup"><span data-stu-id="a00ee-535">The following code snippet illustrates how to use these methods:</span></span>
  
  ```csharp
  ConnectionMultiplexer redisHostConnection = ...;
  IDatabase cache = redisHostConnection.GetDatabase();
  ...
  await cache.StringSetAsync("data:counter", 99);
  ...
  long oldValue = await cache.StringIncrementAsync("data:counter");
  // Increment by 1 (the default)
  // oldValue should be 100
  
  long newValue = await cache.StringDecrementAsync("data:counter", 50);
  // Decrement by 50
  // newValue should be 50
  ```
* <span data-ttu-id="a00ee-536">`GETSET` 用于检索与键关联的值，并将其更改为新值。</span><span class="sxs-lookup"><span data-stu-id="a00ee-536">`GETSET`, which retrieves the value that's associated with a key and changes it to a new value.</span></span> <span data-ttu-id="a00ee-537">StackExchange 库通过 `IDatabase.StringGetSetAsync` 方法使此操作可供使用。</span><span class="sxs-lookup"><span data-stu-id="a00ee-537">The StackExchange library makes this operation available through the `IDatabase.StringGetSetAsync` method.</span></span> <span data-ttu-id="a00ee-538">以下代码段演示了此方法的示例。</span><span class="sxs-lookup"><span data-stu-id="a00ee-538">The code snippet below shows an example of this method.</span></span> <span data-ttu-id="a00ee-539">此代码从前一示例返回与键 "data:counter" 关联的当前值。</span><span class="sxs-lookup"><span data-stu-id="a00ee-539">This code returns the current value that's associated with the key "data:counter" from the previous example.</span></span> <span data-ttu-id="a00ee-540">然后将此键的值重置为零，这些都是同一操作的一部分：</span><span class="sxs-lookup"><span data-stu-id="a00ee-540">Then it resets the value for this key back to zero, all as part of the same operation:</span></span>
  
  ```csharp
  ConnectionMultiplexer redisHostConnection = ...;
  IDatabase cache = redisHostConnection.GetDatabase();
  ...
  string oldValue = await cache.StringGetSetAsync("data:counter", 0);
  ```
* <span data-ttu-id="a00ee-541">`MGET` 和 `MSET` 可以作为单个操作返回或更改一组字符串值。</span><span class="sxs-lookup"><span data-stu-id="a00ee-541">`MGET` and `MSET`, which can return or change a set of string values as a single operation.</span></span> <span data-ttu-id="a00ee-542">`IDatabase.StringGetAsync` 和 `IDatabase.StringSetAsync` 已重载以支持此功能，如以下示例中所示：</span><span class="sxs-lookup"><span data-stu-id="a00ee-542">The `IDatabase.StringGetAsync` and `IDatabase.StringSetAsync` methods are overloaded to support this functionality, as shown in the following example:</span></span>
  
  ```csharp
  ConnectionMultiplexer redisHostConnection = ...;
  IDatabase cache = redisHostConnection.GetDatabase();
  ...
  // Create a list of key-value pairs
  var keysAndValues =
      new List<KeyValuePair<RedisKey, RedisValue>>()
      {
          new KeyValuePair<RedisKey, RedisValue>("data:key1", "value1"),
          new KeyValuePair<RedisKey, RedisValue>("data:key99", "value2"),
          new KeyValuePair<RedisKey, RedisValue>("data:key322", "value3")
      };
  
  // Store the list of key-value pairs in the cache
  cache.StringSet(keysAndValues.ToArray());
  ...
  // Find all values that match a list of keys
  RedisKey[] keys = { "data:key1", "data:key99", "data:key322"};
  // values should contain { "value1", "value2", "value3" }
  RedisValue[] values = cache.StringGet(keys);

  ```

<span data-ttu-id="a00ee-543">也可以将多个操作合并成单个 Redis 事务，如本文前面的“Redis 事务和批处理”部分中所述。</span><span class="sxs-lookup"><span data-stu-id="a00ee-543">You can also combine multiple operations into a single Redis transaction as described in the Redis transactions and batches section earlier in this article.</span></span> <span data-ttu-id="a00ee-544">StackExchange 库通过 `ITransaction` 接口提供事务支持。</span><span class="sxs-lookup"><span data-stu-id="a00ee-544">The StackExchange library provides support for transactions through the `ITransaction` interface.</span></span>

<span data-ttu-id="a00ee-545">使用 `IDatabase.CreateTransaction` 方法创建 `ITransaction` 对象。</span><span class="sxs-lookup"><span data-stu-id="a00ee-545">You create an `ITransaction` object by using the `IDatabase.CreateTransaction` method.</span></span> <span data-ttu-id="a00ee-546">使用 `ITransaction` 对象提供的方法调用对事务的命令。</span><span class="sxs-lookup"><span data-stu-id="a00ee-546">You invoke commands to the transaction by using the methods provided by the `ITransaction` object.</span></span>

<span data-ttu-id="a00ee-547">`ITransaction` 接口可用于访问 `IDatabase` 接口所访问的类似一组方法，不过，所有方法是异步的。</span><span class="sxs-lookup"><span data-stu-id="a00ee-547">The `ITransaction` interface provides access to a set of methods that's similar to those accessed by the `IDatabase` interface, except that all the methods are asynchronous.</span></span> <span data-ttu-id="a00ee-548">这意味着，这些方法仅在调用 `ITransaction.Execute` 方法时执行。</span><span class="sxs-lookup"><span data-stu-id="a00ee-548">This means that they are only performed when the `ITransaction.Execute` method is invoked.</span></span> <span data-ttu-id="a00ee-549">`ITransaction.Execute` 方法返回的值指示事务创建是成功 (true) 还是失败 (false)。</span><span class="sxs-lookup"><span data-stu-id="a00ee-549">The value that's returned by the `ITransaction.Execute` method indicates whether the transaction was created successfully (true) or if it failed (false).</span></span>

<span data-ttu-id="a00ee-550">以下代码段显示的示例会在执行同一事务期间递增和递减两个计数器：</span><span class="sxs-lookup"><span data-stu-id="a00ee-550">The following code snippet shows an example that increments and decrements two counters as part of the same transaction:</span></span>

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
ITransaction transaction = cache.CreateTransaction();
var tx1 = transaction.StringIncrementAsync("data:counter1");
var tx2 = transaction.StringDecrementAsync("data:counter2");
bool result = transaction.Execute();
Console.WriteLine("Transaction {0}", result ? "succeeded" : "failed");
Console.WriteLine("Result of increment: {0}", tx1.Result);
Console.WriteLine("Result of decrement: {0}", tx2.Result);
```

<span data-ttu-id="a00ee-551">请记住，Redis 事务不同于关系数据库中的事务。</span><span class="sxs-lookup"><span data-stu-id="a00ee-551">Remember that Redis transactions are unlike transactions in relational databases.</span></span> <span data-ttu-id="a00ee-552">`Execute` 方法只是将构成运行事务的所有命令排入队列，如果其中任何一个命令格式不当，则停止事务。</span><span class="sxs-lookup"><span data-stu-id="a00ee-552">The `Execute` method simply queues all the commands that comprise the transaction to be run, and if any of them is malformed then the transaction is stopped.</span></span> <span data-ttu-id="a00ee-553">如果已成功将所有命令排入队列，以异步方式运行每个命令。</span><span class="sxs-lookup"><span data-stu-id="a00ee-553">If all the commands have been queued successfully, each command runs asynchronously.</span></span>

<span data-ttu-id="a00ee-554">如果任何命令失败，其他命令仍将继续处理。</span><span class="sxs-lookup"><span data-stu-id="a00ee-554">If any command fails, the others still continue processing.</span></span> <span data-ttu-id="a00ee-555">如果需要验证命令是否已成功完成，必须使用相应任务的 **Result** 属性来提取命令的结果，如上述示例中所示。</span><span class="sxs-lookup"><span data-stu-id="a00ee-555">If you need to verify that a command has completed successfully, you must fetch the results of the command by using the **Result** property of the corresponding task, as shown in the example above.</span></span> <span data-ttu-id="a00ee-556">读取 **Result** 属性会阻塞调用线程，直到任务完成。</span><span class="sxs-lookup"><span data-stu-id="a00ee-556">Reading the **Result** property will block the calling thread until the task has completed.</span></span>

<span data-ttu-id="a00ee-557">有关详细信息，请参阅 StackExchange.Redis 网站上的 [Transactions in Redis](https://stackexchange.github.io/StackExchange.Redis/Transactions)（Redis 中的事务）页。</span><span class="sxs-lookup"><span data-stu-id="a00ee-557">For more information, see the [Transactions in Redis](https://stackexchange.github.io/StackExchange.Redis/Transactions) page on the StackExchange.Redis website.</span></span>

<span data-ttu-id="a00ee-558">执行批处理操作时，可以使用 StackExchange 库的 `IBatch` 接口。</span><span class="sxs-lookup"><span data-stu-id="a00ee-558">When performing batch operations, you can use the `IBatch` interface of the StackExchange library.</span></span> <span data-ttu-id="a00ee-559">此接口可用于访问 `IDatabase` 接口所访问的类似一组方法，不过，所有方法是异步的。</span><span class="sxs-lookup"><span data-stu-id="a00ee-559">This interface provides access to a set of methods similar to those accessed by the `IDatabase` interface, except that all the methods are asynchronous.</span></span>

<span data-ttu-id="a00ee-560">可以使用 `IDatabase.CreateBatch` 方法来创建 `IBatch` 对象，并使用 `IBatch.Execute` 方法来运行批处理，如以下示例所示。</span><span class="sxs-lookup"><span data-stu-id="a00ee-560">You create an `IBatch` object by using the `IDatabase.CreateBatch` method, and then run the batch by using the `IBatch.Execute` method, as shown in the following example.</span></span> <span data-ttu-id="a00ee-561">这段代码仅设置字符串值，递增和递减前面示例中使用的相同计数器，并显示结果：</span><span class="sxs-lookup"><span data-stu-id="a00ee-561">This code simply sets a string value, increments and decrements the same counters used in the previous example, and displays the results:</span></span>

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
IBatch batch = cache.CreateBatch();
batch.StringSetAsync("data:key1", 11);
var t1 = batch.StringIncrementAsync("data:counter1");
var t2 = batch.StringDecrementAsync("data:counter2");
batch.Execute();
Console.WriteLine("{0}", t1.Result);
Console.WriteLine("{0}", t2.Result);
```

<span data-ttu-id="a00ee-562">必须知道，这不同于事务，如果因为格式不当而导致批中的命令失败，其他命令仍可运行。</span><span class="sxs-lookup"><span data-stu-id="a00ee-562">It is important to understand that unlike a transaction, if a command in a batch fails because it is malformed, the other commands might still run.</span></span> <span data-ttu-id="a00ee-563">`IBatch.Execute` 方法不返回成功或失败的任何指示。</span><span class="sxs-lookup"><span data-stu-id="a00ee-563">The `IBatch.Execute` method does not return any indication of success or failure.</span></span>

### <a name="perform-fire-and-forget-cache-operations"></a><span data-ttu-id="a00ee-564">执行即发即弃缓存操作</span><span class="sxs-lookup"><span data-stu-id="a00ee-564">Perform fire and forget cache operations</span></span>
<span data-ttu-id="a00ee-565">Redis 通过使用命令标志来支持即发即弃操作。</span><span class="sxs-lookup"><span data-stu-id="a00ee-565">Redis supports fire and forget operations by using command flags.</span></span> <span data-ttu-id="a00ee-566">在此情况下，客户端仅启动操作，但不关注结果，且并不会等待命令完成。</span><span class="sxs-lookup"><span data-stu-id="a00ee-566">In this situation, the client simply initiates an operation but has no interest in the result and does not wait for the command to be completed.</span></span> <span data-ttu-id="a00ee-567">以下示例演示了如何以即发即弃操作的形式执行 INCR 命令：</span><span class="sxs-lookup"><span data-stu-id="a00ee-567">The example below shows how to perform the INCR command as a fire and forget operation:</span></span>

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
await cache.StringSetAsync("data:key1", 99);
...
cache.StringIncrement("data:key1", flags: CommandFlags.FireAndForget);
```

### <a name="specify-automatically-expiring-keys"></a><span data-ttu-id="a00ee-568">指定密钥自动过期</span><span class="sxs-lookup"><span data-stu-id="a00ee-568">Specify automatically expiring keys</span></span>
<span data-ttu-id="a00ee-569">在 Redis 缓存中存储项时，可以指定超时，超时过后，会自动从缓存中删除该项。</span><span class="sxs-lookup"><span data-stu-id="a00ee-569">When you store an item in a Redis cache, you can specify a timeout after which the item will be automatically removed from the cache.</span></span> <span data-ttu-id="a00ee-570">还可以在密钥过期之前，使用 `TTL` 命令来查询剩余时间。</span><span class="sxs-lookup"><span data-stu-id="a00ee-570">You can also query how much more time a key has before it expires by using the `TTL` command.</span></span> <span data-ttu-id="a00ee-571">StackExchange 应用程序可通过 `IDatabase.KeyTimeToLive` 方法使用此命令。</span><span class="sxs-lookup"><span data-stu-id="a00ee-571">This command is available to StackExchange applications by using the `IDatabase.KeyTimeToLive` method.</span></span>

<span data-ttu-id="a00ee-572">以下代码段演示了如何将密钥过期时间设置为 20 秒，并查询密钥剩余生存期：</span><span class="sxs-lookup"><span data-stu-id="a00ee-572">The following code snippet shows how to set an expiration time of 20 seconds on a key, and query the remaining lifetime of the key:</span></span>

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
// Add a key with an expiration time of 20 seconds
await cache.StringSetAsync("data:key1", 99, TimeSpan.FromSeconds(20));
...
// Query how much time a key has left to live
// If the key has already expired, the KeyTimeToLive function returns a null
TimeSpan? expiry = cache.KeyTimeToLive("data:key1");
```

<span data-ttu-id="a00ee-573">还可以使用 StackExchange 库中作为 `KeyExpireAsync` 方法提供的 EXPIRE 命令将过期时间设置为特定的日期和时间：</span><span class="sxs-lookup"><span data-stu-id="a00ee-573">You can also set the expiration time to a specific date and time by using the EXPIRE command, which is available in the StackExchange library as the `KeyExpireAsync` method:</span></span>

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
// Add a key with an expiration date of midnight on 1st January 2015
await cache.StringSetAsync("data:key1", 99);
await cache.KeyExpireAsync("data:key1",
    new DateTime(2015, 1, 1, 0, 0, 0, DateTimeKind.Utc));
...
```

> [!TIP] 
> <span data-ttu-id="a00ee-574">可以使用 DEL 命令手动从缓存中删除项，该命令在 StackExchange 库中作为 `IDatabase.KeyDeleteAsync` 方法提供。</span><span class="sxs-lookup"><span data-stu-id="a00ee-574">You can manually remove an item from the cache by using the DEL command, which is available through the StackExchange library as the `IDatabase.KeyDeleteAsync` method.</span></span>

### <a name="use-tags-to-cross-correlate-cached-items"></a><span data-ttu-id="a00ee-575">使用标记来交叉关联项</span><span class="sxs-lookup"><span data-stu-id="a00ee-575">Use tags to cross-correlate cached items</span></span>
<span data-ttu-id="a00ee-576">Redis 集是共享单个键的多个项集合。</span><span class="sxs-lookup"><span data-stu-id="a00ee-576">A Redis set is a collection of multiple items that share a single key.</span></span> <span data-ttu-id="a00ee-577">可以使用 SADD 命令来创建集。</span><span class="sxs-lookup"><span data-stu-id="a00ee-577">You can create a set by using the SADD command.</span></span> <span data-ttu-id="a00ee-578">可以使用 SMEMBERS 命令来检索集中的项。</span><span class="sxs-lookup"><span data-stu-id="a00ee-578">You can retrieve the items in a set by using the SMEMBERS command.</span></span> <span data-ttu-id="a00ee-579">StackExchange 库通过 `IDatabase.SetAddAsync` 方法实现 SADD 命令，并使用 `IDatabase.SetMembersAsync` 方法实现 SMEMBERS 命令。</span><span class="sxs-lookup"><span data-stu-id="a00ee-579">The StackExchange library implements the SADD command with the `IDatabase.SetAddAsync` method, and the SMEMBERS command with the `IDatabase.SetMembersAsync` method.</span></span>

<span data-ttu-id="a00ee-580">还可以使用 SDIFF（差集）、SINTER（交集）和 SUNION（并集）命令来合并现有集以创建新的集。</span><span class="sxs-lookup"><span data-stu-id="a00ee-580">You can also combine existing sets to create new sets by using the SDIFF (set difference), SINTER (set intersection), and SUNION (set union) commands.</span></span> <span data-ttu-id="a00ee-581">StackExchange 库在 `IDatabase.SetCombineAsync` 方法中统一了这些操作。</span><span class="sxs-lookup"><span data-stu-id="a00ee-581">The StackExchange library unifies these operations in the `IDatabase.SetCombineAsync` method.</span></span> <span data-ttu-id="a00ee-582">此方法的第一个参数指定要执行的设置操作。</span><span class="sxs-lookup"><span data-stu-id="a00ee-582">The first parameter to this method specifies the set operation to perform.</span></span>

<span data-ttu-id="a00ee-583">以下代码段演示了如何使用集来快速存储和检索相关项的集合。</span><span class="sxs-lookup"><span data-stu-id="a00ee-583">The following code snippets show how sets can be useful for quickly storing and retrieving collections of related items.</span></span> <span data-ttu-id="a00ee-584">此代码使用本文前面的“实施 Redis 缓存客户端应用程序”部分中所述的 `BlogPost` 类型。</span><span class="sxs-lookup"><span data-stu-id="a00ee-584">This code uses the `BlogPost` type that was described in the section Implement Redis Cache Client Applications earlier in this article.</span></span>

<span data-ttu-id="a00ee-585">`BlogPost` 对象包含四个字段：ID、标题、排名分数和标记集合。</span><span class="sxs-lookup"><span data-stu-id="a00ee-585">A `BlogPost` object contains four fields—an ID, a title, a ranking score, and a collection of tags.</span></span> <span data-ttu-id="a00ee-586">以下第一个代码片段演示了用于填充 `BlogPost` 对象的 C# 列表的示例数据：</span><span class="sxs-lookup"><span data-stu-id="a00ee-586">The first code snippet below shows the sample data that's used for populating a C# list of `BlogPost` objects:</span></span>

```csharp
List<string[]> tags = new List<string[]>
{
    new[] { "iot","csharp" },
    new[] { "iot","azure","csharp" },
    new[] { "csharp","git","big data" },
    new[] { "iot","git","database" },
    new[] { "database","git" },
    new[] { "csharp","database" },
    new[] { "iot" },
    new[] { "iot","database","git" },
    new[] { "azure","database","big data","git","csharp" },
    new[] { "azure" }
};

List<BlogPost> posts = new List<BlogPost>();
int blogKey = 1;
int numberOfPosts = 20;
Random random = new Random();
for (int i = 0; i < numberOfPosts; i++)
{
    blogKey++;
    posts.Add(new BlogPost(
        blogKey,                  // Blog post ID
        string.Format(CultureInfo.InvariantCulture, "Blog Post #{0}",
            blogKey),             // Blog post title
        random.Next(100, 10000),  // Ranking score
        tags[i % tags.Count]));   // Tags--assigned from a collection
                                  // in the tags list
}
```

<span data-ttu-id="a00ee-587">可以在 Redis 缓存中针对每个 `BlogPost` 对象将标记存储为集，并将每个集与 `BlogPost` ID关联。</span><span class="sxs-lookup"><span data-stu-id="a00ee-587">You can store the tags for each `BlogPost` object as a set in a Redis cache and associate each set with the ID of the `BlogPost`.</span></span> <span data-ttu-id="a00ee-588">这样，应用程序便可以快速查找属于特定博客文章的所有标记。</span><span class="sxs-lookup"><span data-stu-id="a00ee-588">This enables an application to quickly find all the tags that belong to a specific blog post.</span></span> <span data-ttu-id="a00ee-589">若要启用反向搜索并查找所有共享特定标记的博客文章，可以创建另一个集，用于保存引用键中标记 ID 的博客文章：</span><span class="sxs-lookup"><span data-stu-id="a00ee-589">To enable searching in the opposite direction and find all blog posts that share a specific tag, you can create another set that holds the blog posts referencing the tag ID in the key:</span></span>

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
// Tags are easily represented as Redis Sets
foreach (BlogPost post in posts)
{
    string redisKey = string.Format(CultureInfo.InvariantCulture,
        "blog:posts:{0}:tags", post.Id);
    // Add tags to the blog post in Redis
    await cache.SetAddAsync(
        redisKey, post.Tags.Select(s => (RedisValue)s).ToArray());

    // Now do the inverse so we can figure how which blog posts have a given tag
    foreach (var tag in post.Tags)
    {
        await cache.SetAddAsync(string.Format(CultureInfo.InvariantCulture,
            "tag:{0}:blog:posts", tag), post.Id);
    }
}
```

<span data-ttu-id="a00ee-590">这些结构允许高效执行许多常见查询。</span><span class="sxs-lookup"><span data-stu-id="a00ee-590">These structures enable you to perform many common queries very efficiently.</span></span> <span data-ttu-id="a00ee-591">例如，可以按如下所示查找并显示博客文章 1 的所有标记：</span><span class="sxs-lookup"><span data-stu-id="a00ee-591">For example, you can find and display all of the tags for blog post 1 like this:</span></span>

```csharp
// Show the tags for blog post #1
foreach (var value in await cache.SetMembersAsync("blog:posts:1:tags"))
{
    Console.WriteLine(value);
}
```

<span data-ttu-id="a00ee-592">可以通过执行交集操作，查找博客文章 1 和博客文章 2 公用的所有标记，如下所示：</span><span class="sxs-lookup"><span data-stu-id="a00ee-592">You can find all tags that are common to blog post 1 and blog post 2 by performing a set intersection operation, as follows:</span></span>

```csharp
// Show the tags in common for blog posts #1 and #2
foreach (var value in await cache.SetCombineAsync(SetOperation.Intersect, new RedisKey[]
    { "blog:posts:1:tags", "blog:posts:2:tags" }))
{
    Console.WriteLine(value);
}
```

<span data-ttu-id="a00ee-593">可以查找包含特定标记的所有博客文章：</span><span class="sxs-lookup"><span data-stu-id="a00ee-593">And you can find all blog posts that contain a specific tag:</span></span>

```csharp
// Show the ids of the blog posts that have the tag "iot".
foreach (var value in await cache.SetMembersAsync("tag:iot:blog:posts"))
{
    Console.WriteLine(value);
}
```

### <a name="find-recently-accessed-items"></a><span data-ttu-id="a00ee-594">查找最近访问的项</span><span class="sxs-lookup"><span data-stu-id="a00ee-594">Find recently accessed items</span></span>
<span data-ttu-id="a00ee-595">许多应用程序需要执行的常见任务是查找最近访问的项。</span><span class="sxs-lookup"><span data-stu-id="a00ee-595">A common task required of many applications is to find the most recently accessed items.</span></span> <span data-ttu-id="a00ee-596">例如，博客站点可能要显示有关最近读过的博客文章的信息。</span><span class="sxs-lookup"><span data-stu-id="a00ee-596">For example, a blogging site might want to display information about the most recently read blog posts.</span></span>

<span data-ttu-id="a00ee-597">可以使用 Redis 列表来实现此功能。</span><span class="sxs-lookup"><span data-stu-id="a00ee-597">You can implement this functionality by using a Redis list.</span></span> <span data-ttu-id="a00ee-598">Redis 列表包含共享同一个键的多个项。</span><span class="sxs-lookup"><span data-stu-id="a00ee-598">A Redis list contains multiple items that share the same key.</span></span> <span data-ttu-id="a00ee-599">该列表充当双端队列。</span><span class="sxs-lookup"><span data-stu-id="a00ee-599">The list acts as a double-ended queue.</span></span> <span data-ttu-id="a00ee-600">可以使用 LPUSH（左推）和 RPUSH（右推）命令将项推送到列表一端。</span><span class="sxs-lookup"><span data-stu-id="a00ee-600">You can push items to either end of the list by using the LPUSH (left push) and RPUSH (right push) commands.</span></span> <span data-ttu-id="a00ee-601">可以使用 LPOP 和 RPOP 命令从列表的一端检索项。</span><span class="sxs-lookup"><span data-stu-id="a00ee-601">You can retrieve items from either end of the list by using the LPOP and RPOP commands.</span></span> <span data-ttu-id="a00ee-602">还可以使用 LRANGE 和 RRANGE 命令返回一组元素。</span><span class="sxs-lookup"><span data-stu-id="a00ee-602">You can also return a set of elements by using the LRANGE and RRANGE commands.</span></span>

<span data-ttu-id="a00ee-603">以下代码段演示了如何使用 StackExchange 库来执行这些操作。</span><span class="sxs-lookup"><span data-stu-id="a00ee-603">The code snippets below show how you can perform these operations by using the StackExchange library.</span></span> <span data-ttu-id="a00ee-604">此代码使用前面示例中的 `BlogPost` 类型。</span><span class="sxs-lookup"><span data-stu-id="a00ee-604">This code uses the `BlogPost` type from the previous examples.</span></span> <span data-ttu-id="a00ee-605">当用户阅读博客文章时，`IDatabase.ListLeftPushAsync` 方法将博客文章的标题推送到与 Redis 缓存中的“blog:recent_posts”键关联的列表。</span><span class="sxs-lookup"><span data-stu-id="a00ee-605">As a blog post is read by a user, the `IDatabase.ListLeftPushAsync` method pushes the title of the blog post onto a list that's associated with the key "blog:recent_posts" in the Redis cache.</span></span>

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
string redisKey = "blog:recent_posts";
BlogPost blogPost = ...; // Reference to the blog post that has just been read
await cache.ListLeftPushAsync(
    redisKey, blogPost.Title); // Push the blog post onto the list
```

<span data-ttu-id="a00ee-606">随着阅读的博客文章越来越多，其标题将推送到同一列表。</span><span class="sxs-lookup"><span data-stu-id="a00ee-606">As more blog posts are read, their titles are pushed onto the same list.</span></span> <span data-ttu-id="a00ee-607">列表已根据其添加顺序进行排序。</span><span class="sxs-lookup"><span data-stu-id="a00ee-607">The list is ordered by the sequence in which the titles have been added.</span></span> <span data-ttu-id="a00ee-608">最近阅读的博客文章朝向列表左端。</span><span class="sxs-lookup"><span data-stu-id="a00ee-608">The most recently read blog posts are towards the left end of the list.</span></span> <span data-ttu-id="a00ee-609">（如果同一博客文章阅读了一次以上，则它在列表中有多个条目。）</span><span class="sxs-lookup"><span data-stu-id="a00ee-609">(If the same blog post is read more than once, it will have multiple entries in the list.)</span></span>

<span data-ttu-id="a00ee-610">可以使用 `IDatabase.ListRange` 方法显示最近阅读的文章的标题。</span><span class="sxs-lookup"><span data-stu-id="a00ee-610">You can display the titles of the most recently read posts by using the `IDatabase.ListRange` method.</span></span> <span data-ttu-id="a00ee-611">此方法采用包含列表、起点和终点的键。</span><span class="sxs-lookup"><span data-stu-id="a00ee-611">This method takes the key that contains the list, a starting point, and an ending point.</span></span> <span data-ttu-id="a00ee-612">以下代码将从列表的最左端检索 10 篇博客文章的标题（项为 0 到 9）：</span><span class="sxs-lookup"><span data-stu-id="a00ee-612">The following code retrieves the titles of the 10 blog posts (items from 0 to 9) at the left-most end of the list:</span></span>

```csharp
// Show latest ten posts
foreach (string postTitle in await cache.ListRangeAsync(redisKey, 0, 9))
{
    Console.WriteLine(postTitle);
}
```

<span data-ttu-id="a00ee-613">请注意，`ListRangeAsync` 方法不会从列表中删除项。</span><span class="sxs-lookup"><span data-stu-id="a00ee-613">Note that the `ListRangeAsync` method does not remove items from the list.</span></span> <span data-ttu-id="a00ee-614">为此，可以使用 `IDatabase.ListLeftPopAsync` 和 `IDatabase.ListRightPopAsync` 方法。</span><span class="sxs-lookup"><span data-stu-id="a00ee-614">To do this, you can use the `IDatabase.ListLeftPopAsync` and `IDatabase.ListRightPopAsync` methods.</span></span>

<span data-ttu-id="a00ee-615">若要防止列表无限增长，可以通过修剪列表来定期删除项。</span><span class="sxs-lookup"><span data-stu-id="a00ee-615">To prevent the list from growing indefinitely, you can periodically cull items by trimming the list.</span></span> <span data-ttu-id="a00ee-616">以下代码段演示了如何只保留列表中位于最左端的 5 个项并删除其他所有项：</span><span class="sxs-lookup"><span data-stu-id="a00ee-616">The code snippet below shows you how to remove all but the five left-most items from the list:</span></span>

```csharp
await cache.ListTrimAsync(redisKey, 0, 5);
```

### <a name="implement-a-leader-board"></a><span data-ttu-id="a00ee-617">实施排行榜</span><span class="sxs-lookup"><span data-stu-id="a00ee-617">Implement a leader board</span></span>
<span data-ttu-id="a00ee-618">默认情况下，集中的项不以任何特定顺序保存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-618">By default, the items in a set are not held in any specific order.</span></span> <span data-ttu-id="a00ee-619">可以使用 ZADD 命令（StackExchange 库中的 `IDatabase.SortedSetAdd` 方法）来创建排序集合。</span><span class="sxs-lookup"><span data-stu-id="a00ee-619">You can create an ordered set by using the ZADD command (the `IDatabase.SortedSetAdd` method in the StackExchange library).</span></span> <span data-ttu-id="a00ee-620">系统使用一个名为 score（作为命令的参数提供）的数字值来为项排序。</span><span class="sxs-lookup"><span data-stu-id="a00ee-620">The items are ordered by using a numeric value called a score, which is provided as a parameter to the command.</span></span>

<span data-ttu-id="a00ee-621">以下代码段将博客文章的标题添加到排序列表。</span><span class="sxs-lookup"><span data-stu-id="a00ee-621">The following code snippet adds the title of a blog post to an ordered list.</span></span> <span data-ttu-id="a00ee-622">在示例中，每篇博客文章还有包含博客文章排名的评分字段。</span><span class="sxs-lookup"><span data-stu-id="a00ee-622">In this example, each blog post also has a score field that contains the ranking of the blog post.</span></span>

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
string redisKey = "blog:post_rankings";
BlogPost blogPost = ...; // Reference to a blog post that has just been rated
await cache.SortedSetAddAsync(redisKey, blogPost.Title, blogPost.Score);
```

<span data-ttu-id="a00ee-623">可以使用 `IDatabase.SortedSetRangeByRankWithScores` 方法以评分递增顺序来检索博客文章标题和评分：</span><span class="sxs-lookup"><span data-stu-id="a00ee-623">You can retrieve the blog post titles and scores in ascending score order by using the `IDatabase.SortedSetRangeByRankWithScores` method:</span></span>

```csharp
foreach (var post in await cache.SortedSetRangeByRankWithScoresAsync(redisKey))
{
    Console.WriteLine(post);
}
```

> [!NOTE]
> <span data-ttu-id="a00ee-624">StackExchange 库还提供了 `IDatabase.SortedSetRangeByRankAsync` 方法，用于以评分顺序返回数据，但不返回评分。</span><span class="sxs-lookup"><span data-stu-id="a00ee-624">The StackExchange library also provides the `IDatabase.SortedSetRangeByRankAsync` method, which returns the data in score order, but does not return the scores.</span></span>
> 
> 

<span data-ttu-id="a00ee-625">也可以使用评分递减顺序来检索项，并通过将额外参数提供给 `IDatabase.SortedSetRangeByRankWithScoresAsync` 方法来限制返回项的数目。</span><span class="sxs-lookup"><span data-stu-id="a00ee-625">You can also retrieve items in descending order of scores, and limit the number of items that are returned by providing additional parameters to the `IDatabase.SortedSetRangeByRankWithScoresAsync` method.</span></span> <span data-ttu-id="a00ee-626">以下示例演示了排名前 10 位博客文章的标题和评分：</span><span class="sxs-lookup"><span data-stu-id="a00ee-626">The next example displays the titles and scores of the top 10 ranked blog posts:</span></span>

```csharp
foreach (var post in await cache.SortedSetRangeByRankWithScoresAsync(
                               redisKey, 0, 9, Order.Descending))
{
    Console.WriteLine(post);
}
```

<span data-ttu-id="a00ee-627">以下示例使用了 `IDatabase.SortedSetRangeByScoreWithScoresAsync` 方法，该方法可用于限制返回给那些处于给定评分范围内的项：</span><span class="sxs-lookup"><span data-stu-id="a00ee-627">The next example uses the `IDatabase.SortedSetRangeByScoreWithScoresAsync` method, which you can use to limit the items that are returned to those that fall within a given score range:</span></span>

```csharp
// Blog posts with scores between 5000 and 100000
foreach (var post in await cache.SortedSetRangeByScoreWithScoresAsync(
                               redisKey, 5000, 100000))
{
    Console.WriteLine(post);
}
```

### <a name="message-by-using-channels"></a><span data-ttu-id="a00ee-628">使用通道进行消息传送</span><span class="sxs-lookup"><span data-stu-id="a00ee-628">Message by using channels</span></span>
<span data-ttu-id="a00ee-629">Redis 服务器除了可用作数据缓存以外，还可通过高性能发布者/订阅者机制提供消息传送。</span><span class="sxs-lookup"><span data-stu-id="a00ee-629">Apart from acting as a data cache, a Redis server provides messaging through a high-performance publisher/subscriber mechanism.</span></span> <span data-ttu-id="a00ee-630">客户端应用程序可以订阅通道，其他应用程序或服务可以将消息发布到通道。</span><span class="sxs-lookup"><span data-stu-id="a00ee-630">Client applications can subscribe to a channel, and other applications or services can publish messages to the channel.</span></span> <span data-ttu-id="a00ee-631">订阅应用程序随后会接收这些消息，并可以处理消息。</span><span class="sxs-lookup"><span data-stu-id="a00ee-631">Subscribing applications will then receive these messages and can process them.</span></span>

<span data-ttu-id="a00ee-632">Redis 提供 SUBSCRIBE 命令来让客户端应用程序订阅通道。</span><span class="sxs-lookup"><span data-stu-id="a00ee-632">Redis provides the SUBSCRIBE command for client applications to use to subscribe to channels.</span></span> <span data-ttu-id="a00ee-633">此命令需要一个或多个可供应用程序接受消息的通道的名称。</span><span class="sxs-lookup"><span data-stu-id="a00ee-633">This command expects the name of one or more channels on which the application will accept messages.</span></span> <span data-ttu-id="a00ee-634">StackExchange 库包含 `ISubscription` 接口，可让 .NET Framework 应用程序订阅和发布到通道。</span><span class="sxs-lookup"><span data-stu-id="a00ee-634">The StackExchange library includes the `ISubscription` interface, which enables a .NET Framework application to subscribe and publish to channels.</span></span>

<span data-ttu-id="a00ee-635">使用 Redis 服务器连接的 `GetSubscriber` 方法创建 `ISubscription` 对象。</span><span class="sxs-lookup"><span data-stu-id="a00ee-635">You create an `ISubscription` object by using the `GetSubscriber` method of the connection to the Redis server.</span></span> <span data-ttu-id="a00ee-636">然后使用此对象的 `SubscribeAsync` 方法在通道上侦听消息。</span><span class="sxs-lookup"><span data-stu-id="a00ee-636">Then you listen for messages on a channel by using the `SubscribeAsync` method of this object.</span></span> <span data-ttu-id="a00ee-637">以下代码示例演示了如何订阅名为“messages:blogPosts”的通道：</span><span class="sxs-lookup"><span data-stu-id="a00ee-637">The following code example shows how to subscribe to a channel named "messages:blogPosts":</span></span>

```csharp
ConnectionMultiplexer redisHostConnection = ...;
ISubscriber subscriber = redisHostConnection.GetSubscriber();
...
await subscriber.SubscribeAsync("messages:blogPosts", (channel, message) => Console.WriteLine("Title is: {0}", message));
```

<span data-ttu-id="a00ee-638">`Subscribe` 方法的第一个参数为通道的名称。</span><span class="sxs-lookup"><span data-stu-id="a00ee-638">The first parameter to the `Subscribe` method is the name of the channel.</span></span> <span data-ttu-id="a00ee-639">此名称遵循缓存中键使用的相同约定。</span><span class="sxs-lookup"><span data-stu-id="a00ee-639">This name follows the same conventions that are used by keys in the cache.</span></span> <span data-ttu-id="a00ee-640">该名称可以包含任何二进制数据，但建议最好使用相对较短且有意义的字符串，以帮助确保良好的性能和易维护性。</span><span class="sxs-lookup"><span data-stu-id="a00ee-640">The name can contain any binary data, although it is advisable to use relatively short, meaningful strings to help ensure good performance and maintainability.</span></span>

<span data-ttu-id="a00ee-641">另请注意，通道使用的命名空间与键使用的不同。</span><span class="sxs-lookup"><span data-stu-id="a00ee-641">Note also that the namespace used by channels is separate from that used by keys.</span></span> <span data-ttu-id="a00ee-642">这意味着，通道和键可以同名，不过，这可能会导致更难以维护应用程序代码。</span><span class="sxs-lookup"><span data-stu-id="a00ee-642">This means you can have channels and keys that have the same name, although this may make your application code more difficult to maintain.</span></span>

<span data-ttu-id="a00ee-643">第二个参数是 Action 委派。</span><span class="sxs-lookup"><span data-stu-id="a00ee-643">The second parameter is an Action delegate.</span></span> <span data-ttu-id="a00ee-644">每当新的消息出现在通道上时，此委派就会以异步方式运行。</span><span class="sxs-lookup"><span data-stu-id="a00ee-644">This delegate runs asynchronously whenever a new message appears on the channel.</span></span> <span data-ttu-id="a00ee-645">此示例仅显示了控制台上的消息（消息将包含博客文章的标题）。</span><span class="sxs-lookup"><span data-stu-id="a00ee-645">This example simply displays the message on the console (the message will contain the title of a blog post).</span></span>

<span data-ttu-id="a00ee-646">若要发布到通道，应用程序可以使用 Redis PUBLISH 命令。</span><span class="sxs-lookup"><span data-stu-id="a00ee-646">To publish to a channel, an application can use the Redis PUBLISH command.</span></span> <span data-ttu-id="a00ee-647">StackExchange 库提供了 `IServer.PublishAsync` 方法来执行此操作。</span><span class="sxs-lookup"><span data-stu-id="a00ee-647">The StackExchange library provides the `IServer.PublishAsync` method to perform this operation.</span></span> <span data-ttu-id="a00ee-648">以下代码片段演示了如何将消息发布到“messages:blogPosts”通道：</span><span class="sxs-lookup"><span data-stu-id="a00ee-648">The next code snippet shows how to publish a message to the "messages:blogPosts" channel:</span></span>

```csharp
ConnectionMultiplexer redisHostConnection = ...;
ISubscriber subscriber = redisHostConnection.GetSubscriber();
...
BlogPost blogPost = ...;
subscriber.PublishAsync("messages:blogPosts", blogPost.Title);
```

<span data-ttu-id="a00ee-649">关于发布/订阅机制，应该了解几个要点：</span><span class="sxs-lookup"><span data-stu-id="a00ee-649">There are several points you should understand about the publish/subscribe mechanism:</span></span>

* <span data-ttu-id="a00ee-650">多个订阅者可以订阅同一个通道，他们都将接收发布到该通道的消息。</span><span class="sxs-lookup"><span data-stu-id="a00ee-650">Multiple subscribers can subscribe to the same channel, and they will all receive the messages that are published to that channel.</span></span>
* <span data-ttu-id="a00ee-651">订阅者仅接收订阅后发布的消息。</span><span class="sxs-lookup"><span data-stu-id="a00ee-651">Subscribers only receive messages that have been published after they have subscribed.</span></span> <span data-ttu-id="a00ee-652">通道不会缓冲，一旦发布消息，Redis 基础结构就会将消息推送到每个订阅者，然后删除消息。</span><span class="sxs-lookup"><span data-stu-id="a00ee-652">Channels are not buffered, and once a message is published, the Redis infrastructure pushes the message to each subscriber and then removes it.</span></span>
* <span data-ttu-id="a00ee-653">默认情况下，订阅者根据发送顺序来接收消息。</span><span class="sxs-lookup"><span data-stu-id="a00ee-653">By default, messages are received by subscribers in the order in which they are sent.</span></span> <span data-ttu-id="a00ee-654">在具有大量消息和许多订阅者与发布者的高度活跃系统中，保证依序传送消息可能会降低系统性能。</span><span class="sxs-lookup"><span data-stu-id="a00ee-654">In a highly active system with a large number of messages and many subscribers and publishers, guaranteed sequential delivery of messages can slow performance of the system.</span></span> <span data-ttu-id="a00ee-655">如果每个消息各自独立且顺序并不重要，则可以通过 Redis 系统启用并发处理，这有助于提高响应度。</span><span class="sxs-lookup"><span data-stu-id="a00ee-655">If each message is independent and the order is unimportant, you can enable concurrent processing by the Redis system, which can help to improve responsiveness.</span></span> <span data-ttu-id="a00ee-656">可以在 StackExchange 客户端中，通过将订阅者使用的连接的 PreserveAsyncOrder 设置为 false 来实现此目的：</span><span class="sxs-lookup"><span data-stu-id="a00ee-656">You can achieve this in a StackExchange client by setting the PreserveAsyncOrder of the connection used by the subscriber to false:</span></span>

```csharp
ConnectionMultiplexer redisHostConnection = ...;
redisHostConnection.PreserveAsyncOrder = false;
ISubscriber subscriber = redisHostConnection.GetSubscriber();
```

### <a name="serialization-considerations"></a><span data-ttu-id="a00ee-657">序列化注意事项</span><span class="sxs-lookup"><span data-stu-id="a00ee-657">Serialization considerations</span></span>

<span data-ttu-id="a00ee-658">选择序列化格式时，请考虑在性能、互操作性、版本控制、与现有系统的兼容性、数据压缩和内存开销之间作出权衡。</span><span class="sxs-lookup"><span data-stu-id="a00ee-658">When you choose a serialization format, consider tradeoffs between performance, interoperability, versioning, compatibility with existing systems, data compression, and memory overhead.</span></span> <span data-ttu-id="a00ee-659">评估性能时，请记住基准高度依赖于上下文。</span><span class="sxs-lookup"><span data-stu-id="a00ee-659">When you are evaluating performance, remember that benchmarks are highly dependent on context.</span></span> <span data-ttu-id="a00ee-660">它们可能不反映实际工作负载，也可能不考虑较新的库或版本。</span><span class="sxs-lookup"><span data-stu-id="a00ee-660">They may not reflect your actual workload, and may not consider newer libraries or versions.</span></span> <span data-ttu-id="a00ee-661">不存在适用于所有方案的“最快”序列化程序。</span><span class="sxs-lookup"><span data-stu-id="a00ee-661">There is no single "fastest" serializer for all scenarios.</span></span> 

<span data-ttu-id="a00ee-662">应考虑的一些选项包括：</span><span class="sxs-lookup"><span data-stu-id="a00ee-662">Some options to consider include:</span></span>

- <span data-ttu-id="a00ee-663">[Protocol Buffers](https://github.com/google/protobuf)（也称为 protobuf）是由 Google 开发的序列化格式，用于对结构化数据进行高效序列化。</span><span class="sxs-lookup"><span data-stu-id="a00ee-663">[Protocol Buffers](https://github.com/google/protobuf) (also called protobuf) is a serialization format developed by Google for serializing structured data efficiently.</span></span> <span data-ttu-id="a00ee-664">它使用强类型定义文件定义消息结构。</span><span class="sxs-lookup"><span data-stu-id="a00ee-664">It uses strongly-typed definition files to define message structures.</span></span> <span data-ttu-id="a00ee-665">然后，将这些定义文件编译为用于对消息进行序列化和反序列化的特定语言代码。</span><span class="sxs-lookup"><span data-stu-id="a00ee-665">These definition files are then compiled to language-specific code for serializing and deserializing messages.</span></span> <span data-ttu-id="a00ee-666">Protobuf 可用于现有的 RPC 机制，还可以生成 RPC 服务。</span><span class="sxs-lookup"><span data-stu-id="a00ee-666">Protobuf can be used over existing RPC mechanisms, or it can generate an RPC service.</span></span>

- <span data-ttu-id="a00ee-667">[Apache Thrift](https://thrift.apache.org/) 使用类似的方法，即使用强类型定义文件和编译步骤生成序列化代码和 RPC 服务。</span><span class="sxs-lookup"><span data-stu-id="a00ee-667">[Apache Thrift](https://thrift.apache.org/) uses a similar approach, with strongly typed definition files and a compilation step to generate the serialization code and RPC services.</span></span>  

- <span data-ttu-id="a00ee-668">[Apache Avro](https://avro.apache.org/) 向 Protocol Buffers 和 Thrift 提供类似的功能，但不提供编译步骤。</span><span class="sxs-lookup"><span data-stu-id="a00ee-668">[Apache Avro](https://avro.apache.org/) provides similar functionality to Protocol Buffers and Thrift, but there is no compilation step.</span></span> <span data-ttu-id="a00ee-669">相反，序列化数据始终包含描述结构的架构。</span><span class="sxs-lookup"><span data-stu-id="a00ee-669">Instead, serialized data always includes a schema that describes the structure.</span></span> 

- <span data-ttu-id="a00ee-670">[JSON](http://json.org/) 是使用可人工读取的文本字段的开放标准。</span><span class="sxs-lookup"><span data-stu-id="a00ee-670">[JSON](http://json.org/) is an open standard that uses human-readable text fields.</span></span> <span data-ttu-id="a00ee-671">它提供广泛的跨平台支持。</span><span class="sxs-lookup"><span data-stu-id="a00ee-671">It has broad cross-platform support.</span></span> <span data-ttu-id="a00ee-672">JSON 不使用消息架构。</span><span class="sxs-lookup"><span data-stu-id="a00ee-672">JSON does not use message schemas.</span></span> <span data-ttu-id="a00ee-673">它是基于文本的格式，在网络上的效率不高。</span><span class="sxs-lookup"><span data-stu-id="a00ee-673">Being a text-based format, it is not very efficient over the wire.</span></span> <span data-ttu-id="a00ee-674">但是，在某些情况下，可通过 HTTP 直接向客户端返回缓存项，此时，存储 JSON 可以节省从另一格式反序列化然后再序列化为 JSON 的成本。</span><span class="sxs-lookup"><span data-stu-id="a00ee-674">In some cases, however, you may be returning cached items directly to a client via HTTP, in which case storing JSON could save the cost of deserializing from another format and then serializing to JSON.</span></span>

- <span data-ttu-id="a00ee-675">[BSON](http://bsonspec.org/) 是一种二进制序列化格式，使用的结构与 JSON 类似。</span><span class="sxs-lookup"><span data-stu-id="a00ee-675">[BSON](http://bsonspec.org/) is a binary serialization format that uses a structure similar to JSON.</span></span> <span data-ttu-id="a00ee-676">相对于 JSON，BSON 旨在轻量、易于扫描以及可快速进行序列化和反序列化。</span><span class="sxs-lookup"><span data-stu-id="a00ee-676">BSON was designed to be lightweight, easy to scan, and fast to serialize and deserialize, relative to JSON.</span></span> <span data-ttu-id="a00ee-677">其有效负载的大小与 JSON 相当。</span><span class="sxs-lookup"><span data-stu-id="a00ee-677">Payloads are comparable in size to JSON.</span></span> <span data-ttu-id="a00ee-678">BSON 有效负载可能大于或小于 JSON 有效负载，具体取决于数据。</span><span class="sxs-lookup"><span data-stu-id="a00ee-678">Depending on the data, a BSON payload may be smaller or larger than a JSON payload.</span></span> <span data-ttu-id="a00ee-679">BSON 拥有 JSON 中不支持的一些其他数据类型，尤其是 BinData（用于字节数组）和日期。</span><span class="sxs-lookup"><span data-stu-id="a00ee-679">BSON has some additional data types that are not available in JSON, notably BinData (for byte arrays) and Date.</span></span>

- <span data-ttu-id="a00ee-680">[MessagePack](http://msgpack.org/) 是旨在压缩网络传输的二进制序列化格式。</span><span class="sxs-lookup"><span data-stu-id="a00ee-680">[MessagePack](http://msgpack.org/) is a binary serialization format that is designed to be compact for transmission over the wire.</span></span> <span data-ttu-id="a00ee-681">没有消息架构或消息类型检查。</span><span class="sxs-lookup"><span data-stu-id="a00ee-681">There are no message schemas or message type checking.</span></span>

- <span data-ttu-id="a00ee-682">[Bond](https://microsoft.github.io/bond/) 是用于系统化数据的跨平台框架。</span><span class="sxs-lookup"><span data-stu-id="a00ee-682">[Bond](https://microsoft.github.io/bond/) is a cross-platform framework for working with schematized data.</span></span> <span data-ttu-id="a00ee-683">它支持跨语言序列化和反序列化。</span><span class="sxs-lookup"><span data-stu-id="a00ee-683">It supports cross-language serialization and deserialization.</span></span> <span data-ttu-id="a00ee-684">此处列出的与其他系统的显著差异包括对继承、类型别名和泛型的支持。</span><span class="sxs-lookup"><span data-stu-id="a00ee-684">Notable differences from other systems listed here are support for inheritance, type aliases, and generics.</span></span> 

- <span data-ttu-id="a00ee-685">[gRPC](http://www.grpc.io/) 是由 Google 开发的开放源 RPC 系统。</span><span class="sxs-lookup"><span data-stu-id="a00ee-685">[gRPC](http://www.grpc.io/) is an open source RPC system developed by Google.</span></span> <span data-ttu-id="a00ee-686">默认情况下，它将 Protocol Buffers 用作其定义语言和基础消息交换格式。</span><span class="sxs-lookup"><span data-stu-id="a00ee-686">By default, it uses Protocol Buffers as its definition language and underlying message interchange format.</span></span>

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="a00ee-687">相关模式和指南</span><span class="sxs-lookup"><span data-stu-id="a00ee-687">Related patterns and guidance</span></span>

<span data-ttu-id="a00ee-688">在应用程序中实施缓存时，以下模式也可能与方案相关：</span><span class="sxs-lookup"><span data-stu-id="a00ee-688">The following pattern might also be relevant to your scenario when you implement caching in your applications:</span></span>

* <span data-ttu-id="a00ee-689">[缓存端模式](http://msdn.microsoft.com/library/dn589799.aspx)：此模式描述如何按需将数据从数据存储载入缓存。</span><span class="sxs-lookup"><span data-stu-id="a00ee-689">[Cache-aside pattern](http://msdn.microsoft.com/library/dn589799.aspx): This pattern describes how to load data on demand into a cache from a data store.</span></span> <span data-ttu-id="a00ee-690">此模式还有助于在缓存中保存的数据与原始数据存储中的数据之间保持一致性。</span><span class="sxs-lookup"><span data-stu-id="a00ee-690">This pattern also helps to maintain consistency between data that's held in the cache and the data in the original data store.</span></span>
* <span data-ttu-id="a00ee-691">[分片模式](http://msdn.microsoft.com/library/dn589797.aspx)提供了有关实施水平分区，以帮助在存储和访问大量数据时提高伸缩性的信息。</span><span class="sxs-lookup"><span data-stu-id="a00ee-691">The [Sharding pattern](http://msdn.microsoft.com/library/dn589797.aspx) provides information about implementing horizontal partitioning to help improve scalability when storing and accessing large volumes of data.</span></span>

## <a name="more-information"></a><span data-ttu-id="a00ee-692">详细信息</span><span class="sxs-lookup"><span data-stu-id="a00ee-692">More information</span></span>
* <span data-ttu-id="a00ee-693">Microsoft 网站上的 [MemoryCache class](http://msdn.microsoft.com/library/system.runtime.caching.memorycache.aspx)（MemoryCache 类）页</span><span class="sxs-lookup"><span data-stu-id="a00ee-693">The [MemoryCache class](http://msdn.microsoft.com/library/system.runtime.caching.memorycache.aspx) page on the Microsoft website</span></span>
* <span data-ttu-id="a00ee-694">Microsoft 网站上的 [Azure Redis Cache documentation](https://azure.microsoft.com/documentation/services/cache/)（Azure Redis 缓存文档）页</span><span class="sxs-lookup"><span data-stu-id="a00ee-694">The [Azure Redis Cache documentation](https://azure.microsoft.com/documentation/services/cache/) page on the Microsoft website</span></span>
* <span data-ttu-id="a00ee-695">Microsoft 网站上的 [Azure Redis Cache FAQ](/azure/redis-cache/cache-faq)（Azure Redis 缓存常见问题）页</span><span class="sxs-lookup"><span data-stu-id="a00ee-695">The [Azure Redis Cache FAQ](/azure/redis-cache/cache-faq) page on the Microsoft website</span></span>
* <span data-ttu-id="a00ee-696">Microsoft 网站上的 [Configuration model](http://msdn.microsoft.com/library/windowsazure/hh914149.aspx)（配置模型）页</span><span class="sxs-lookup"><span data-stu-id="a00ee-696">The [Configuration model](http://msdn.microsoft.com/library/windowsazure/hh914149.aspx) page on the Microsoft website</span></span>
* <span data-ttu-id="a00ee-697">Microsoft 网站上的 [Task-based Asynchronous Pattern](http://msdn.microsoft.com/library/hh873175.aspx)（基于任务的异步模式）页</span><span class="sxs-lookup"><span data-stu-id="a00ee-697">The [Task-based Asynchronous Pattern](http://msdn.microsoft.com/library/hh873175.aspx) page on the Microsoft website</span></span>
* <span data-ttu-id="a00ee-698">StackExchange.Redis GitHub 存储库上的 [Pipelines and multiplexers](https://stackexchange.github.io/StackExchange.Redis/PipelinesMultiplexers)（管道和多路复用器）页</span><span class="sxs-lookup"><span data-stu-id="a00ee-698">The [Pipelines and multiplexers](https://stackexchange.github.io/StackExchange.Redis/PipelinesMultiplexers) page on the StackExchange.Redis GitHub repo</span></span>
* <span data-ttu-id="a00ee-699">Redis 网站上的 [Redis persistence](http://redis.io/topics/persistence)（Redis 持久性）页</span><span class="sxs-lookup"><span data-stu-id="a00ee-699">The [Redis persistence](http://redis.io/topics/persistence) page on the Redis website</span></span>
* <span data-ttu-id="a00ee-700">Redis 网站上的 [Replication](http://redis.io/topics/replication)（复制）页</span><span class="sxs-lookup"><span data-stu-id="a00ee-700">The [Replication page](http://redis.io/topics/replication) on the Redis website</span></span>
* <span data-ttu-id="a00ee-701">Redis 网站上的 [Redis cluster tutorial](http://redis.io/topics/cluster-tutorial)（Redis 群集教程）页</span><span class="sxs-lookup"><span data-stu-id="a00ee-701">The [Redis cluster tutorial](http://redis.io/topics/cluster-tutorial) page on the Redis website</span></span>
* <span data-ttu-id="a00ee-702">Redis 网站上的 [Partitioning: how to split data among multiple Redis instances](http://redis.io/topics/partitioning)（分区：如何在多个 Redis 实例之间拆分数据）页</span><span class="sxs-lookup"><span data-stu-id="a00ee-702">The [Partitioning: how to split data among multiple Redis instances](http://redis.io/topics/partitioning) page on the Redis website</span></span>
* <span data-ttu-id="a00ee-703">Redis 网站上的 [Using Redis as an LRU Cache](http://redis.io/topics/lru-cache)（使用 Redis 作为 LRU 缓存）页</span><span class="sxs-lookup"><span data-stu-id="a00ee-703">The [Using Redis as an LRU Cache](http://redis.io/topics/lru-cache) page on the Redis website</span></span>
* <span data-ttu-id="a00ee-704">Redis 网站上的 [Transactions](http://redis.io/topics/transactions)（事务）页</span><span class="sxs-lookup"><span data-stu-id="a00ee-704">The [Transactions](http://redis.io/topics/transactions) page on the Redis website</span></span>
* <span data-ttu-id="a00ee-705">Redis 网站上的 [Redis security](http://redis.io/topics/security)（Redis 安全性）页</span><span class="sxs-lookup"><span data-stu-id="a00ee-705">The [Redis security](http://redis.io/topics/security) page on the Redis website</span></span>
* <span data-ttu-id="a00ee-706">Azure 博客上的 [Lap around Azure Redis Cache](https://azure.microsoft.com/blog/2014/06/04/lap-around-azure-redis-cache-preview/)（浏览 Azure Redis 缓存）页</span><span class="sxs-lookup"><span data-stu-id="a00ee-706">The [Lap around Azure Redis Cache](https://azure.microsoft.com/blog/2014/06/04/lap-around-azure-redis-cache-preview/) page on the Azure blog</span></span>
* <span data-ttu-id="a00ee-707">Microsoft 网站上的 [Running Redis on a CentOS Linux VM in Azure](http://blogs.msdn.com/b/tconte/archive/2012/06/08/running-redis-on-a-centos-linux-vm-in-windows-azure.aspx)（在 Azure 中的 CentOS Linux VM 上运行 Redis）页</span><span class="sxs-lookup"><span data-stu-id="a00ee-707">The [Running Redis on a CentOS Linux VM in Azure](http://blogs.msdn.com/b/tconte/archive/2012/06/08/running-redis-on-a-centos-linux-vm-in-windows-azure.aspx) page on the Microsoft website</span></span>
* <span data-ttu-id="a00ee-708">Microsoft 网站上的 [ASP.NET session state provider for Azure Redis Cache](/azure/redis-cache/cache-aspnet-session-state-provider)（Azure Redis 缓存的 ASP.NET 会话状态提供程序）页</span><span class="sxs-lookup"><span data-stu-id="a00ee-708">The [ASP.NET session state provider for Azure Redis Cache](/azure/redis-cache/cache-aspnet-session-state-provider) page on the Microsoft website</span></span>
* <span data-ttu-id="a00ee-709">Microsoft 网站上的 [ASP.NET output cache provider for Azure Redis Cache](/azure/redis-cache/cache-aspnet-output-cache-provider)（Azure Redis 缓存的 ASP.NET 输出缓存提供程序）页</span><span class="sxs-lookup"><span data-stu-id="a00ee-709">The [ASP.NET output cache provider for Azure Redis Cache](/azure/redis-cache/cache-aspnet-output-cache-provider) page on the Microsoft website</span></span>
* <span data-ttu-id="a00ee-710">Redis 网站上的 [An Introduction to Redis data types and abstractions](http://redis.io/topics/data-types-intro)（Redis 数据类型和抽象简介）页</span><span class="sxs-lookup"><span data-stu-id="a00ee-710">The [An Introduction to Redis data types and abstractions](http://redis.io/topics/data-types-intro) page on the Redis website</span></span>
* <span data-ttu-id="a00ee-711">StackExchange.Redis 网站上的 [Basic usage](https://stackexchange.github.io/StackExchange.Redis/Basics)（基本用法）页</span><span class="sxs-lookup"><span data-stu-id="a00ee-711">The [Basic usage](https://stackexchange.github.io/StackExchange.Redis/Basics) page on the StackExchange.Redis website</span></span>
* <span data-ttu-id="a00ee-712">StackExchange.Redis 存储库上的 [Transactions in Redis](https://stackexchange.github.io/StackExchange.Redis/Transactions)（Redis 中的事务）页</span><span class="sxs-lookup"><span data-stu-id="a00ee-712">The [Transactions in Redis](https://stackexchange.github.io/StackExchange.Redis/Transactions) page on the StackExchange.Redis repo</span></span>
* <span data-ttu-id="a00ee-713">Microsoft 网站上的 [Data partitioning guide](http://msdn.microsoft.com/library/dn589795.aspx)（数据分区指南）</span><span class="sxs-lookup"><span data-stu-id="a00ee-713">The [Data partitioning guide](http://msdn.microsoft.com/library/dn589795.aspx) on the Microsoft website</span></span>

