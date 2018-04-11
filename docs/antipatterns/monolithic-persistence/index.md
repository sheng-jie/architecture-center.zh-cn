---
title: 整体持久性对立模式
description: 将应用程序的所有数据放入单个数据存储可能会损害性能。
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 7f04b9f0805c281068b6b2edaf040683773e6f6e
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="monolithic-persistence-antipattern"></a><span data-ttu-id="459c8-103">整体持久性对立模式</span><span class="sxs-lookup"><span data-stu-id="459c8-103">Monolithic Persistence antipattern</span></span>

<span data-ttu-id="459c8-104">将应用程序的所有数据放入单个数据存储可能会降低性能，原因是这会导致资源争用，或者数据存储不很适合某些数据。</span><span class="sxs-lookup"><span data-stu-id="459c8-104">Putting all of an application's data into a single data store can hurt performance, either because it leads to resource contention, or because the data store is not a good fit for some of the data.</span></span>

## <a name="problem-description"></a><span data-ttu-id="459c8-105">问题描述</span><span class="sxs-lookup"><span data-stu-id="459c8-105">Problem description</span></span>

<span data-ttu-id="459c8-106">一直以来，不管应用程序需要存储哪些不同类型的数据，往往都只使用单个数据存储。</span><span class="sxs-lookup"><span data-stu-id="459c8-106">Historically, applications have often used a single data store, regardless of the different types of data that the application might need to store.</span></span> <span data-ttu-id="459c8-107">这样做的原因通常是为了简化应用程序设计，或者受限于开发团队的现有技能组合。</span><span class="sxs-lookup"><span data-stu-id="459c8-107">Usually this was done to simplify the application design, or else to match the existing skill set of the development team.</span></span> 

<span data-ttu-id="459c8-108">现代基于云的系统往往附带其他功能性和非功能性要求，需要存储许多异构类型的数据，例如文档、图像、缓存数据、排队消息、应用程序日志和遥测数据。</span><span class="sxs-lookup"><span data-stu-id="459c8-108">Modern cloud-based systems often have additional functional and nonfunctional requirements, and need to store many heterogenous types of data, such as documents, images, cached data, queued messages, application logs, and telemetry.</span></span> <span data-ttu-id="459c8-109">遵循传统方法将所有这些信息放入同一个数据存储可能会损害性能，主要原因有两个：</span><span class="sxs-lookup"><span data-stu-id="459c8-109">Following the traditional approach and putting all of this information into the same data store can hurt performance, for two main reasons:</span></span>

- <span data-ttu-id="459c8-110">在同一个数据存储中存储和检索大量不相关的数据可能导致资源争用，从而导致响应时间增加和连接失败。</span><span class="sxs-lookup"><span data-stu-id="459c8-110">Storing and retrieving large amounts of unrelated data in the same data store can cause contention, which in turn leads to slow response times and connection failures.</span></span>
- <span data-ttu-id="459c8-111">不管选择哪个数据存储，它都不一定最适合所有不同类型的数据，或者未针对应用程序执行的操作进行优化。</span><span class="sxs-lookup"><span data-stu-id="459c8-111">Whichever data store is chosen, it might not be the best fit for all of the different types of data, or it might not be optimized for the operations that the application performs.</span></span> 

<span data-ttu-id="459c8-112">以下示例演示了一个向数据库添加新记录，并将结果记录到日志的 ASP.NET Web API 控制器。</span><span class="sxs-lookup"><span data-stu-id="459c8-112">The following example shows an ASP.NET Web API controller that adds a new record to a database and also records the result to a log.</span></span> <span data-ttu-id="459c8-113">日志保存在业务数据所在的同一个数据库中。</span><span class="sxs-lookup"><span data-stu-id="459c8-113">The log is held in the same database as the business data.</span></span> <span data-ttu-id="459c8-114">可在[此处][sample-app]找到完整示例。</span><span class="sxs-lookup"><span data-stu-id="459c8-114">You can find the complete sample [here][sample-app].</span></span>

```csharp
public class MonoController : ApiController
{
    private static readonly string ProductionDb = ...;

    public async Task<IHttpActionResult> PostAsync([FromBody]string value)
    {
        await DataAccess.InsertPurchaseOrderHeaderAsync(ProductionDb);
        await DataAccess.LogAsync(ProductionDb, LogTableName);
        return Ok();
    }
}
```

<span data-ttu-id="459c8-115">日志记录的生成速率可能会影响业务操作的性能。</span><span class="sxs-lookup"><span data-stu-id="459c8-115">The rate at which log records are generated will probably affect the performance of the business operations.</span></span> <span data-ttu-id="459c8-116">如果另一个组件（例如应用程序进程监视器）定期读取和处理日志数据，则它们也可能影响业务操作。</span><span class="sxs-lookup"><span data-stu-id="459c8-116">And if another component, such as an application process monitor, regularly reads and processes the log data, that can also affect the business operations.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="459c8-117">如何解决问题</span><span class="sxs-lookup"><span data-stu-id="459c8-117">How to fix the problem</span></span>

<span data-ttu-id="459c8-118">根据数据的用法隔离数据。</span><span class="sxs-lookup"><span data-stu-id="459c8-118">Separate data according to its use.</span></span> <span data-ttu-id="459c8-119">对于每个数据集，选择最符合数据集用法的数据存储。</span><span class="sxs-lookup"><span data-stu-id="459c8-119">For each data set, select a data store that best matches how that data set will be used.</span></span> <span data-ttu-id="459c8-120">在前面的示例中，应用程序应将日志记录到与保存业务数据的数据库不同的存储：</span><span class="sxs-lookup"><span data-stu-id="459c8-120">In the previous example, the application should be logging to a separate store from the database that holds business data:</span></span> 

```csharp
public class PolyController : ApiController
{
    private static readonly string ProductionDb = ...;
    private static readonly string LogDb = ...;

    public async Task<IHttpActionResult> PostAsync([FromBody]string value)
    {
        await DataAccess.InsertPurchaseOrderHeaderAsync(ProductionDb);
        // Log to a different data store.
        await DataAccess.LogAsync(LogDb, LogTableName);
        return Ok();
    }
}
```

## <a name="considerations"></a><span data-ttu-id="459c8-121">注意事项</span><span class="sxs-lookup"><span data-stu-id="459c8-121">Considerations</span></span>

- <span data-ttu-id="459c8-122">根据数据的使用方式及其访问方式隔离数据。</span><span class="sxs-lookup"><span data-stu-id="459c8-122">Separate data by the way it is used and how it is accessed.</span></span> <span data-ttu-id="459c8-123">例如，不要将日志信息和业务数据存储在同一个数据存储中。</span><span class="sxs-lookup"><span data-stu-id="459c8-123">For example, don't store log information and business data in the same data store.</span></span> <span data-ttu-id="459c8-124">这些数据类型的要求和访问模式明显不同。</span><span class="sxs-lookup"><span data-stu-id="459c8-124">These types of data have significantly different requirements and patterns of access.</span></span> <span data-ttu-id="459c8-125">日志记录原生是连续的，而业务数据更有可能需要随机访问，因此通常是关系型的。</span><span class="sxs-lookup"><span data-stu-id="459c8-125">Log records are inherently sequential, while business data is more likely to require random access, and is often relational.</span></span>

- <span data-ttu-id="459c8-126">考虑每种数据类型的数据访问模式。</span><span class="sxs-lookup"><span data-stu-id="459c8-126">Consider the data access pattern for each type of data.</span></span> <span data-ttu-id="459c8-127">例如，将带有格式的报告和文档存储在 [Cosmos DB][CosmosDB] 等文档数据库中，但使用 [Azure Redis 缓存][Azure-cache]来缓存临时数据。</span><span class="sxs-lookup"><span data-stu-id="459c8-127">For example, store formatted reports and documents in a document database such as [Cosmos DB][CosmosDB], but use [Azure Redis Cache][Azure-cache] to cache temporary data.</span></span>

- <span data-ttu-id="459c8-128">如果遵循了此指导原则，但仍达到了数据库限制，可能需要纵向扩展数据库。</span><span class="sxs-lookup"><span data-stu-id="459c8-128">If you follow this guidance but still reach the limits of the database, you may need to scale up the database.</span></span> <span data-ttu-id="459c8-129">此外，请考虑横向扩展，并将负载分区到不同的数据库服务器。</span><span class="sxs-lookup"><span data-stu-id="459c8-129">Also consider scaling horizontally and partitioning the load across database servers.</span></span> <span data-ttu-id="459c8-130">但是，分区可能需要重新设计应用程序。</span><span class="sxs-lookup"><span data-stu-id="459c8-130">However, partitioning may require redesigning the application.</span></span> <span data-ttu-id="459c8-131">有关详细信息，请参阅[数据分区][DataPartitioningGuidance]。</span><span class="sxs-lookup"><span data-stu-id="459c8-131">For more information, see [Data partitioning][DataPartitioningGuidance].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="459c8-132">如何检测问题</span><span class="sxs-lookup"><span data-stu-id="459c8-132">How to detect the problem</span></span>

<span data-ttu-id="459c8-133">随着系统耗尽数据库连接等资源，系统的运行速度可能明显下降并最终发生故障。</span><span class="sxs-lookup"><span data-stu-id="459c8-133">The system will likely slow down dramatically and eventually fail, as the system runs out of resources such as database connections.</span></span>

<span data-ttu-id="459c8-134">可执行以下步骤来帮助确定原因。</span><span class="sxs-lookup"><span data-stu-id="459c8-134">You can perform the following steps to help identify the cause.</span></span>

1. <span data-ttu-id="459c8-135">检测系统以记录关键性能统计信息。</span><span class="sxs-lookup"><span data-stu-id="459c8-135">Instrument the system to record the key performance statistics.</span></span> <span data-ttu-id="459c8-136">捕获每个操作的计时信息，以及应用程序读取和写入数据的位置。</span><span class="sxs-lookup"><span data-stu-id="459c8-136">Capture timing information for each operation, as well as the points where the application reads and writes data.</span></span>
1. <span data-ttu-id="459c8-137">如果可能，请在生产环境中监视运行了几天的系统，以获得有关系统使用方式的真实视图。</span><span class="sxs-lookup"><span data-stu-id="459c8-137">If possible, monitor the system running for a few days in a production environment to get a real-world view of how the system is used.</span></span> <span data-ttu-id="459c8-138">如果无法进行这种监视，请配合实际数量的虚拟用户（这些用户执行一系列典型操作）运行脚本化负载测试。</span><span class="sxs-lookup"><span data-stu-id="459c8-138">If this is not possible, run scripted load tests with a realistic volume of virtual users performing a typical series of operations.</span></span>
2. <span data-ttu-id="459c8-139">使用遥测数据来识别性能不佳的时段。</span><span class="sxs-lookup"><span data-stu-id="459c8-139">Use the telemetry data to identify periods of poor performance.</span></span>
3. <span data-ttu-id="459c8-140">识别在这些时段访问了哪些数据存储。</span><span class="sxs-lookup"><span data-stu-id="459c8-140">Identify which data stores were accessed during those periods.</span></span>
4. <span data-ttu-id="459c8-141">识别可能发生争用的数据存储资源。</span><span class="sxs-lookup"><span data-stu-id="459c8-141">Identify data storage resources that might be experiencing contention.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="459c8-142">示例诊断</span><span class="sxs-lookup"><span data-stu-id="459c8-142">Example diagnosis</span></span>

<span data-ttu-id="459c8-143">以下部分将这些步骤应用到前面所述的示例应用程序。</span><span class="sxs-lookup"><span data-stu-id="459c8-143">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="instrument-and-monitor-the-system"></a><span data-ttu-id="459c8-144">检测和监视系统</span><span class="sxs-lookup"><span data-stu-id="459c8-144">Instrument and monitor the system</span></span>

<span data-ttu-id="459c8-145">下图显示了对上述示例应用程序执行负载测试的结果。</span><span class="sxs-lookup"><span data-stu-id="459c8-145">The following graph shows the results of load testing the sample application described earlier.</span></span> <span data-ttu-id="459c8-146">该测试使用了包含多达 1000 个并发用户的阶跃负载。</span><span class="sxs-lookup"><span data-stu-id="459c8-146">The test used a step load of up to 1000 concurrent users.</span></span>

![对基于 SQL 的控制器执行的负载测试性能结果][MonolithicScenarioLoadTest]

<span data-ttu-id="459c8-148">随着负载提高到 700 个用户，吞吐量随之上升。</span><span class="sxs-lookup"><span data-stu-id="459c8-148">As the load increases to 700 users, so does the throughput.</span></span> <span data-ttu-id="459c8-149">但此时，吞吐量保持稳定，系统似乎以最大的容量运行。</span><span class="sxs-lookup"><span data-stu-id="459c8-149">But at that point, throughput levels off, and the system appears to be running at its maximum capacity.</span></span> <span data-ttu-id="459c8-150">平均响应时间随着用户负载的提高而逐渐增加，表明系统无法跟上需求。</span><span class="sxs-lookup"><span data-stu-id="459c8-150">The average response gradually increases with user load, showing that the system can't keep up with demand.</span></span>

### <a name="identify-periods-of-poor-performance"></a><span data-ttu-id="459c8-151">识别性能不佳的时段</span><span class="sxs-lookup"><span data-stu-id="459c8-151">Identify periods of poor performance</span></span>

<span data-ttu-id="459c8-152">监视生产系统时，可能会看到一些模式。</span><span class="sxs-lookup"><span data-stu-id="459c8-152">If you are monitoring the production system, you might notice patterns.</span></span> <span data-ttu-id="459c8-153">例如，响应时间可能在每天的相同时间明显下降。</span><span class="sxs-lookup"><span data-stu-id="459c8-153">For example, response times might drop off significantly at the same time each day.</span></span> <span data-ttu-id="459c8-154">这可能是定期工作负荷或计划的批处理作业造成的，或只是因为系统在某些时候包含更多的用户。</span><span class="sxs-lookup"><span data-stu-id="459c8-154">This could be caused by a regular workload or scheduled batch job, or just because the system has more users at certain times.</span></span> <span data-ttu-id="459c8-155">应该重点关注这些事件的遥测数据。</span><span class="sxs-lookup"><span data-stu-id="459c8-155">You should focus on the telemetry data for these events.</span></span>

<span data-ttu-id="459c8-156">找出响应时间增加与数据库活动增加或者对共享资源发出的 I/O 请求增加之间的关联。</span><span class="sxs-lookup"><span data-stu-id="459c8-156">Look for correlations between increased response times and increased database activity or I/O to shared resources.</span></span> <span data-ttu-id="459c8-157">如果存在关联，则意味着数据库可能是瓶颈。</span><span class="sxs-lookup"><span data-stu-id="459c8-157">If there are correlations, it means the database might be a bottleneck.</span></span>

### <a name="identify-which-data-stores-are-accessed-during-those-periods"></a><span data-ttu-id="459c8-158">识别在这些时段访问了哪些数据存储</span><span class="sxs-lookup"><span data-stu-id="459c8-158">Identify which data stores are accessed during those periods</span></span>

<span data-ttu-id="459c8-159">下图显示了负载测试期间数据库吞吐量单位 (DTU) 的利用率。</span><span class="sxs-lookup"><span data-stu-id="459c8-159">The next graph shows the utilization of database throughput units (DTU) during the load test.</span></span> <span data-ttu-id="459c8-160">（DTU 用于度量可用容量，是 CPU 利用率、内存分配和 I/O 速率的组合。）DTU 利用率很快达到 100%。</span><span class="sxs-lookup"><span data-stu-id="459c8-160">(A DTU is a measure of available capacity, and is a combination of CPU utilization, memory allocation, I/O rate.) Utilization of DTUs quickly reached 100%.</span></span> <span data-ttu-id="459c8-161">这大概就是上图中出现吞吐量峰值的位置。</span><span class="sxs-lookup"><span data-stu-id="459c8-161">This is roughly the point where throughput peaked in the previous graph.</span></span> <span data-ttu-id="459c8-162">在测试完成之前，数据库利用率一直很高。</span><span class="sxs-lookup"><span data-stu-id="459c8-162">Database utilization remained very high until the test finished.</span></span> <span data-ttu-id="459c8-163">在测试快要结束时略微下降，原因可能是实施了限制、数据库连接争用或其他因素。</span><span class="sxs-lookup"><span data-stu-id="459c8-163">There is a slight drop toward the end, which could be caused by throttling, competition for database connections, or other factors.</span></span>

![Azure 经典门户中的数据库监视器显示数据库的资源利用率][MonolithicDatabaseUtilization]

### <a name="examine-the-telemetry-for-the-data-stores"></a><span data-ttu-id="459c8-165">检查数据存储的遥测数据</span><span class="sxs-lookup"><span data-stu-id="459c8-165">Examine the telemetry for the data stores</span></span>

<span data-ttu-id="459c8-166">检测数据存储以捕获活动的低级详细信息。</span><span class="sxs-lookup"><span data-stu-id="459c8-166">Instrument the data stores to capture the low-level details of the activity.</span></span> <span data-ttu-id="459c8-167">在示例应用程序中，数据访问统计信息显示针对 `PurchaseOrderHeader` 表和 `MonoLog` 表执行了大量插入操作。</span><span class="sxs-lookup"><span data-stu-id="459c8-167">In the sample application, the data access statistics showed a high volume of insert operations performed against both the `PurchaseOrderHeader` table and the `MonoLog` table.</span></span> 

![示例应用程序的数据访问统计信息][MonolithicDataAccessStats]

### <a name="identify-resource-contention"></a><span data-ttu-id="459c8-169">识别资源争用</span><span class="sxs-lookup"><span data-stu-id="459c8-169">Identify resource contention</span></span>

<span data-ttu-id="459c8-170">此时，可以检查源代码，并重点检查应用程序在哪些位置访问了争用的资源。</span><span class="sxs-lookup"><span data-stu-id="459c8-170">At this point, you can review the source code, focusing on the points where contended resources are accessed by the application.</span></span> <span data-ttu-id="459c8-171">找到如下所述的情况：</span><span class="sxs-lookup"><span data-stu-id="459c8-171">Look for situations such as:</span></span>

- <span data-ttu-id="459c8-172">正在将逻辑隔离的数据写入相同的存储。</span><span class="sxs-lookup"><span data-stu-id="459c8-172">Data that is logically separate being written to the same store.</span></span> <span data-ttu-id="459c8-173">日志、报告和排队消息等数据不应保存在业务信息所在的同一个数据库中。</span><span class="sxs-lookup"><span data-stu-id="459c8-173">Data such as logs, reports, and queued messages should not be held in the same database as business information.</span></span>
- <span data-ttu-id="459c8-174">所选数据存储与数据类型（例如关系型数据库中的大型 Blob 或 XML 文档）之间不匹配。</span><span class="sxs-lookup"><span data-stu-id="459c8-174">A mismatch between the choice of data store and the type of data, such as large blobs or XML documents in a relational database.</span></span>
- <span data-ttu-id="459c8-175">具有明显不同使用模式的数据共享同一个存储，例如将 high-write/low-read 数据与 low-write/high-read 数据存储在一起。</span><span class="sxs-lookup"><span data-stu-id="459c8-175">Data with significantly different usage patterns that share the same store, such as high-write/low-read data being stored with low-write/high-read data.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="459c8-176">实施解决方案并验证结果</span><span class="sxs-lookup"><span data-stu-id="459c8-176">Implement the solution and verify the result</span></span>

<span data-ttu-id="459c8-177">应用程序已更改为将日志写入独立的数据存储。</span><span class="sxs-lookup"><span data-stu-id="459c8-177">The application was changed to write logs to a separate data store.</span></span> <span data-ttu-id="459c8-178">下面是负载测试结果：</span><span class="sxs-lookup"><span data-stu-id="459c8-178">Here are the load test results:</span></span>

![使用 Polyglot 控制器的负载测试性能结果][PolyglotScenarioLoadTest]

<span data-ttu-id="459c8-180">吞吐量模式类似于前面的图形，但性能峰值位置出现在每秒大约发出 500 个或更多的请求时。</span><span class="sxs-lookup"><span data-stu-id="459c8-180">The pattern of throughput is similar to the earlier graph, but the point at which performance peaks is approximately 500 requests per second higher.</span></span> <span data-ttu-id="459c8-181">平均响应时间略微下降。</span><span class="sxs-lookup"><span data-stu-id="459c8-181">The average response time is marginally lower.</span></span> <span data-ttu-id="459c8-182">但是，这些统计信息不能反映整体形式。</span><span class="sxs-lookup"><span data-stu-id="459c8-182">However, these statistics don't tell the full story.</span></span> <span data-ttu-id="459c8-183">业务数据库的遥测数据显示，DTU 利用率峰值大约为 75% 而不是 100%。</span><span class="sxs-lookup"><span data-stu-id="459c8-183">Telemetry for the business database shows that DTU utilization peaks at around 75%, rather than 100%.</span></span>

![Azure 经典门户中的数据库监视器显示 polyglot 方案中数据库的资源利用率][PolyglotDatabaseUtilization]

<span data-ttu-id="459c8-185">同样，日志数据库的最大 DTU 利用率只达到了大约 70%。</span><span class="sxs-lookup"><span data-stu-id="459c8-185">Similarly, the maximum DTU utilization of the log database only reaches about 70%.</span></span> <span data-ttu-id="459c8-186">数据库不再是系统性能的制约因素。</span><span class="sxs-lookup"><span data-stu-id="459c8-186">The databases are no longer the limiting factor in the performance of the system.</span></span>

![Azure 经典门户中的数据库监视器显示 polyglot 方案中日志数据库的资源利用率][LogDatabaseUtilization]


## <a name="related-resources"></a><span data-ttu-id="459c8-188">相关资源</span><span class="sxs-lookup"><span data-stu-id="459c8-188">Related resources</span></span>

- <span data-ttu-id="459c8-189">[选择适当的数据存储][data-store-overview]</span><span class="sxs-lookup"><span data-stu-id="459c8-189">[Choose the right data store][data-store-overview]</span></span>
- <span data-ttu-id="459c8-190">[有关选择数据存储的准则][data-store-comparison]</span><span class="sxs-lookup"><span data-stu-id="459c8-190">[Criteria for choosing a data store][data-store-comparison]</span></span>
- <span data-ttu-id="459c8-191">[高度可缩放解决方案的数据访问：使用 SQL、NoSQL 和 Polyglot 持久性][Data-Access-Guide]</span><span class="sxs-lookup"><span data-stu-id="459c8-191">[Data Access for Highly-Scalable Solutions: Using SQL, NoSQL, and Polyglot Persistence][Data-Access-Guide]</span></span>
- <span data-ttu-id="459c8-192">[数据分区][DataPartitioningGuidance]</span><span class="sxs-lookup"><span data-stu-id="459c8-192">[Data partitioning][DataPartitioningGuidance]</span></span>

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/MonolithicPersistence
[CosmosDB]: http://azure.microsoft.com/services/cosmos-db/
[Azure-cache]: /azure/redis-cache/
[Data-Access-Guide]: https://msdn.microsoft.com/library/dn271399.aspx
[DataPartitioningGuidance]: ../../best-practices/data-partitioning.md
[data-store-overview]: ../../guide/technology-choices/data-store-overview.md
[data-store-comparison]: ../../guide/technology-choices/data-store-comparison.md

[MonolithicScenarioLoadTest]: _images/MonolithicScenarioLoadTest.jpg
[MonolithicDatabaseUtilization]: _images/MonolithicDatabaseUtilization.jpg
[MonolithicDataAccessStats]: _images/MonolithicDataAccessStats.jpg
[PolyglotScenarioLoadTest]: _images/PolyglotScenarioLoadTest.jpg
[PolyglotDatabaseUtilization]: _images/PolyglotDatabaseUtilization.jpg
[LogDatabaseUtilization]: _images/LogDatabaseUtilization.jpg
