---
title: "超量提取对立模式"
description: "检索超出业务运营需要的数据可能会导致不必要的 I/O 开销，并降低响应能力。"
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 8a808dce62a1c80c126b7b1df536f74c46726ea1
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="extraneous-fetching-antipattern"></a><span data-ttu-id="afc34-103">超量提取对立模式</span><span class="sxs-lookup"><span data-stu-id="afc34-103">Extraneous Fetching antipattern</span></span>

<span data-ttu-id="afc34-104">检索超出业务运营需要的数据可能会导致不必要的 I/O 开销，并降低响应能力。</span><span class="sxs-lookup"><span data-stu-id="afc34-104">Retrieving more data than needed for a business operation can result in unnecessary I/O overhead and reduce responsiveness.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="afc34-105">问题描述</span><span class="sxs-lookup"><span data-stu-id="afc34-105">Problem description</span></span>

<span data-ttu-id="afc34-106">如果应用程序尝试通过检索所有数据（超过它可能需要的数据量）来尽量减少 I/O 请求数量，则可能会发生这种对立模式。</span><span class="sxs-lookup"><span data-stu-id="afc34-106">This antipattern can occur if the application tries to minimize I/O requests by retrieving all of the data that it *might* need.</span></span> <span data-ttu-id="afc34-107">这通常是过度补偿[琐碎 I/O][chatty-io] 对立模式的结果。</span><span class="sxs-lookup"><span data-stu-id="afc34-107">This is often a result of overcompensating for the [Chatty I/O][chatty-io] antipattern.</span></span> <span data-ttu-id="afc34-108">例如，应用程序可能会提取数据库中每个产品的详细信息。</span><span class="sxs-lookup"><span data-stu-id="afc34-108">For example, an application might fetch the details for every product in a database.</span></span> <span data-ttu-id="afc34-109">但用户可能只需要一部分详细信息（有些信息与客户无关），因此不需要一次性查看所有产品。</span><span class="sxs-lookup"><span data-stu-id="afc34-109">But the user may need just a subset of the details (some may not be relevant to customers), and probably doesn't need to see *all* of the products at once.</span></span> <span data-ttu-id="afc34-110">即使用户浏览的是整个目录，也最好是将结果分页 &mdash; 例如，每次显示 20 条结果。</span><span class="sxs-lookup"><span data-stu-id="afc34-110">Even if the user is browsing the entire catalog, it would make sense to paginate the results &mdash; showing 20 at a time, for example.</span></span>

<span data-ttu-id="afc34-111">出现此问题的另一个原因是遵循了不合理的编程或设计做法。</span><span class="sxs-lookup"><span data-stu-id="afc34-111">Another source of this problem is following poor programming or design practices.</span></span> <span data-ttu-id="afc34-112">例如，以下代码使用实体框架来提取每个产品的完整详细信息。</span><span class="sxs-lookup"><span data-stu-id="afc34-112">For example, the following code uses Entity Framework to fetch the complete details for every product.</span></span> <span data-ttu-id="afc34-113">然后，它会筛选结果以返回一部分字段，并丢弃剩余的字段。</span><span class="sxs-lookup"><span data-stu-id="afc34-113">Then it filters the results to return only a subset of the fields, discarding the rest.</span></span> <span data-ttu-id="afc34-114">可在[此处][sample-app]找到完整示例。</span><span class="sxs-lookup"><span data-stu-id="afc34-114">You can find the complete sample [here][sample-app].</span></span>

```csharp
public async Task<IHttpActionResult> GetAllFieldsAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Execute the query. This happens at the database.
        var products = await context.Products.ToListAsync();

        // Project fields from the query results. This happens in application memory.
        var result = products.Select(p => new ProductInfo { Id = p.ProductId, Name = p.Name });
        return Ok(result);
    }
}
```

<span data-ttu-id="afc34-115">在以下示例中，应用程序会检索数据来执行聚合（也可以由数据库执行该聚合）。</span><span class="sxs-lookup"><span data-stu-id="afc34-115">In the next example, the application retrieves data to perform an aggregation that could be done by the database instead.</span></span> <span data-ttu-id="afc34-116">该应用程序通过获取所有销售订单的每条记录来计算总销售额，然后对这些记录求和。</span><span class="sxs-lookup"><span data-stu-id="afc34-116">The application calculates total sales by getting every record for all orders sold, and then computing the sum over those records.</span></span> <span data-ttu-id="afc34-117">可在[此处][sample-app]找到完整示例。</span><span class="sxs-lookup"><span data-stu-id="afc34-117">You can find the complete sample [here][sample-app].</span></span>

```csharp
public async Task<IHttpActionResult> AggregateOnClientAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Fetch all order totals from the database.
        var orderAmounts = await context.SalesOrderHeaders.Select(soh => soh.TotalDue).ToListAsync();

        // Sum the order totals in memory.
        var total = orderAmounts.Sum();
        return Ok(total);
    }
}
```

<span data-ttu-id="afc34-118">以下示例演示实体框架对 LINQ to Entities 的使用方式所造成的微妙问题。</span><span class="sxs-lookup"><span data-stu-id="afc34-118">The next example shows a subtle problem caused by the way Entity Framework uses LINQ to Entities.</span></span> 

```csharp
var query = from p in context.Products.AsEnumerable()
            where p.SellStartDate < DateTime.Now.AddDays(-7) // AddDays cannot be mapped by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

<span data-ttu-id="afc34-119">该应用程序尝试查找 `SellStartDate` 超过一周的产品。</span><span class="sxs-lookup"><span data-stu-id="afc34-119">The application is trying to find products with a `SellStartDate` more than a week old.</span></span> <span data-ttu-id="afc34-120">在大多数情况下，LINQ to Entities 会将 `where` 子句转换为可由数据库执行的 SQL 语句。</span><span class="sxs-lookup"><span data-stu-id="afc34-120">In most cases, LINQ to Entities would translate a `where` clause to a SQL statement that is executed by the database.</span></span> <span data-ttu-id="afc34-121">但是，在这种情况下，LINQ to Entities 无法将 `AddDays` 方法映射到 SQL。</span><span class="sxs-lookup"><span data-stu-id="afc34-121">In this case, however, LINQ to Entities cannot map the `AddDays` method to SQL.</span></span> <span data-ttu-id="afc34-122">而是返回 `Product` 表中的每行，并在内存中筛选结果。</span><span class="sxs-lookup"><span data-stu-id="afc34-122">Instead, every row from the `Product` table is returned, and the results are filtered in memory.</span></span> 

<span data-ttu-id="afc34-123">调用 `AsEnumerable` 暗示着存在一个问题。</span><span class="sxs-lookup"><span data-stu-id="afc34-123">The call to `AsEnumerable` is a hint that there is a problem.</span></span> <span data-ttu-id="afc34-124">此方法将结果转换为 `IEnumerable` 接口。</span><span class="sxs-lookup"><span data-stu-id="afc34-124">This method converts the results to an `IEnumerable` interface.</span></span> <span data-ttu-id="afc34-125">尽管 `IEnumerable` 支持筛选，但筛选是在客户端而不是数据库上执行的。</span><span class="sxs-lookup"><span data-stu-id="afc34-125">Although `IEnumerable` supports filtering, the filtering is done on the *client* side, not the database.</span></span> <span data-ttu-id="afc34-126">默认情况下，LINQ to Entities 使用 `IQueryable`，该接口将筛选责任传递给数据源。</span><span class="sxs-lookup"><span data-stu-id="afc34-126">By default, LINQ to Entities uses `IQueryable`, which passes the responsibility for filtering to the data source.</span></span> 

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="afc34-127">如何解决问题</span><span class="sxs-lookup"><span data-stu-id="afc34-127">How to fix the problem</span></span>

<span data-ttu-id="afc34-128">避免提取可能很快就会过时或被丢弃的大量数据，只提取所执行操作需要的数据。</span><span class="sxs-lookup"><span data-stu-id="afc34-128">Avoid fetching large volumes of data that may quickly become outdated or might be discarded, and only fetch the data needed for the operation being performed.</span></span> 

<span data-ttu-id="afc34-129">不要从表中获取每个列并对其进行筛选，而是从数据库中选择所需的列。</span><span class="sxs-lookup"><span data-stu-id="afc34-129">Instead of getting every column from a table and then filtering them, select the columns that you need from the database.</span></span>

```csharp
public async Task<IHttpActionResult> GetRequiredFieldsAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Project fields as part of the query itself
        var result = await context.Products
            .Select(p => new ProductInfo {Id = p.ProductId, Name = p.Name})
            .ToListAsync();
        return Ok(result);
    }
}
```

<span data-ttu-id="afc34-130">同样，请在数据库而不是应用程序内存中执行聚合。</span><span class="sxs-lookup"><span data-stu-id="afc34-130">Similarly, perform aggregation in the database and not in application memory.</span></span>

```csharp
public async Task<IHttpActionResult> AggregateOnDatabaseAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Sum the order totals as part of the database query.
        var total = await context.SalesOrderHeaders.SumAsync(soh => soh.TotalDue);
        return Ok(total);
    }
}
```

<span data-ttu-id="afc34-131">使用实体框架时，请确保使用 `IQueryable` 接口而不是 `IEnumerable` 来解析 LINQ 查询。</span><span class="sxs-lookup"><span data-stu-id="afc34-131">When using Entity Framework, ensure that LINQ queries are resolved using the `IQueryable`interface and not `IEnumerable`.</span></span> <span data-ttu-id="afc34-132">可能需要调整查询，以便只使用可映射到数据源的函数。</span><span class="sxs-lookup"><span data-stu-id="afc34-132">You may need to adjust the query to use only functions that can be mapped to the data source.</span></span> <span data-ttu-id="afc34-133">可以重构前面的示例，以便从查询中删除 `AddDays` 方法，使筛选由数据库执行。</span><span class="sxs-lookup"><span data-stu-id="afc34-133">The earlier example can be refactored to remove the `AddDays` method from the query, allowing filtering to be done by the database.</span></span>

```csharp
DateTime dateSince = DateTime.Now.AddDays(-7); // AddDays has been factored out. 
var query = from p in context.Products
            where p.SellStartDate < dateSince // This criterion can be passed to the database by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

## <a name="considerations"></a><span data-ttu-id="afc34-134">注意事项</span><span class="sxs-lookup"><span data-stu-id="afc34-134">Considerations</span></span>

- <span data-ttu-id="afc34-135">在某些情况下，可以通过将数据水平分区来提高性能。</span><span class="sxs-lookup"><span data-stu-id="afc34-135">In some cases, you can improve performance by partitioning data horizontally.</span></span> <span data-ttu-id="afc34-136">如果不同的操作访问不同的数据属性，水平分区可以减少资源争用。</span><span class="sxs-lookup"><span data-stu-id="afc34-136">If different operations access different attributes of the data, horizontal partitioning may reduce contention.</span></span> <span data-ttu-id="afc34-137">大部分操作往往是针对少部分数据运行的，因此，分散此负载可提高性能。</span><span class="sxs-lookup"><span data-stu-id="afc34-137">Often, most operations are run against a small subset of the data, so spreading this load may improve performance.</span></span> <span data-ttu-id="afc34-138">请参阅[数据分区][data-partitioning]。</span><span class="sxs-lookup"><span data-stu-id="afc34-138">See [Data partitioning][data-partitioning].</span></span>

- <span data-ttu-id="afc34-139">对于必须支持无界限查询的操作，请实现分页，每次只提取有限数量的实体。</span><span class="sxs-lookup"><span data-stu-id="afc34-139">For operations that have to support unbounded queries, implement pagination and only fetch a limited number of entities at a time.</span></span> <span data-ttu-id="afc34-140">例如，如果客户正在浏览产品目录，则每次可向其显示一页结果。</span><span class="sxs-lookup"><span data-stu-id="afc34-140">For example, if a customer is browsing a product catalog, you can show one page of results at a time.</span></span>

- <span data-ttu-id="afc34-141">如果可能，请利用数据存储内置的功能。</span><span class="sxs-lookup"><span data-stu-id="afc34-141">When possible, take advantage of features built into the data store.</span></span> <span data-ttu-id="afc34-142">例如，SQL 数据库通常提供聚合函数。</span><span class="sxs-lookup"><span data-stu-id="afc34-142">For example, SQL databases typically provide aggregate functions.</span></span> 

- <span data-ttu-id="afc34-143">如果使用的数据存储不支持某个特定函数（例如聚合），你可以将计算结果存储在其他位置，并在添加或更新记录后更新该值，这样，每当应用程序需要该值时，不必要重新计算。</span><span class="sxs-lookup"><span data-stu-id="afc34-143">If you're using a data store that doesn't support a particular function, such as aggregration, you could store the calculated result elsewhere, updating the value as records are added or updated, so the application doesn't have to recalculate the value each time it's needed.</span></span>

- <span data-ttu-id="afc34-144">如果发现请求正在检索大量字段，请检查源代码，以确定是否真正需要所有这些字段。</span><span class="sxs-lookup"><span data-stu-id="afc34-144">If you see that requests are retrieving a large number of fields, examine the source code to determine whether all of these fields are actually necessary.</span></span> <span data-ttu-id="afc34-145">有时，这些请求是设计不佳的 `SELECT *` 查询造成的。</span><span class="sxs-lookup"><span data-stu-id="afc34-145">Sometimes these requests are the result of poorly designed `SELECT *` query.</span></span> 

- <span data-ttu-id="afc34-146">同样，检索大量实体的请求可能意味着应用程序未正确筛选数据。</span><span class="sxs-lookup"><span data-stu-id="afc34-146">Similarly, requests that retrieve a large number of entities may be sign that the application is not filtering data correctly.</span></span> <span data-ttu-id="afc34-147">请确认是否真正需要所有这些实体。</span><span class="sxs-lookup"><span data-stu-id="afc34-147">Verify that all of these entities are actually needed.</span></span> <span data-ttu-id="afc34-148">尽量使用数据库端筛选，例如，在 SQL 中使用 `WHERE` 子句。</span><span class="sxs-lookup"><span data-stu-id="afc34-148">Use database-side filtering if possible, for example, by using `WHERE` clauses in SQL.</span></span> 

- <span data-ttu-id="afc34-149">将处理工作量卸载到数据库并不总是最佳选择。</span><span class="sxs-lookup"><span data-stu-id="afc34-149">Offloading processing to the database is not always the best option.</span></span> <span data-ttu-id="afc34-150">仅当数据库在设计上或优化为执行这些处理时，才使用此策略。</span><span class="sxs-lookup"><span data-stu-id="afc34-150">Only use this strategy when the database is designed or optimized to do so.</span></span> <span data-ttu-id="afc34-151">大部分数据库系统已针对某些函数进行高度优化，但并不旨在充当通用的应用程序引擎。</span><span class="sxs-lookup"><span data-stu-id="afc34-151">Most database systems are highly optimized for certain functions, but are not designed to act as general-purpose application engines.</span></span> <span data-ttu-id="afc34-152">有关详细信息，请参阅[繁忙数据库对立模式][BusyDatabase]。</span><span class="sxs-lookup"><span data-stu-id="afc34-152">For more information, see the [Busy Database antipattern][BusyDatabase].</span></span>


## <a name="how-to-detect-the-problem"></a><span data-ttu-id="afc34-153">如何检测问题</span><span class="sxs-lookup"><span data-stu-id="afc34-153">How to detect the problem</span></span>

<span data-ttu-id="afc34-154">超量提取的症状包括高延迟和低吞吐量。</span><span class="sxs-lookup"><span data-stu-id="afc34-154">Symptoms of extraneous fetching include high latency and low throughput.</span></span> <span data-ttu-id="afc34-155">如果数据是从数据存储中检索的，则还可能会加剧资源争用。</span><span class="sxs-lookup"><span data-stu-id="afc34-155">If the data is retrieved from a data store, increased contention is also probable.</span></span> <span data-ttu-id="afc34-156">最终用户可能会反映响应时间延长，或服务超时导致失败。这些失败可能返回 HTTP 500（内部服务器）错误或 HTTP 503（服务不可用）错误。</span><span class="sxs-lookup"><span data-stu-id="afc34-156">End users are likely to report extended response times or failures caused by services timing out. These failures could return HTTP 500 (Internal Server) errors or HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="afc34-157">检查 Web 服务器的事件日志，其中可能包含有关错误原因和情况的更详细信息。</span><span class="sxs-lookup"><span data-stu-id="afc34-157">Examine the event logs for the web server, which are likely to contain more detailed information about the causes and circumstances of the errors.</span></span>

<span data-ttu-id="afc34-158">此对立模式的症状和获取的某些遥测数据可能与[整体持久性对立模式][MonolithicPersistence]非常类似。</span><span class="sxs-lookup"><span data-stu-id="afc34-158">The symptoms of this antipattern and some of the telemetry obtained might be very similar to those of the [Monolithic Persistence antipattern][MonolithicPersistence].</span></span> 

<span data-ttu-id="afc34-159">可执行以下步骤来帮助确定原因：</span><span class="sxs-lookup"><span data-stu-id="afc34-159">You can perform the following steps to help identify the cause:</span></span>

1. <span data-ttu-id="afc34-160">执行负载测试、进程监视或其他检测数据捕获方法，识别速度缓慢的工作负荷或事务。</span><span class="sxs-lookup"><span data-stu-id="afc34-160">Identify slow workloads or transactions by performing load-testing, process monitoring, or other methods of capturing instrumentation data.</span></span>
2. <span data-ttu-id="afc34-161">观察系统表现的所有行为模式。</span><span class="sxs-lookup"><span data-stu-id="afc34-161">Observe any behavioral patterns exhibited by the system.</span></span> <span data-ttu-id="afc34-162">在每秒事务数或用户量方面是否存在特定的限制？</span><span class="sxs-lookup"><span data-stu-id="afc34-162">Are there particular limits in terms of transactions per second or volume of users?</span></span>
3. <span data-ttu-id="afc34-163">将慢速工作负荷的实例与行为模式相关联。</span><span class="sxs-lookup"><span data-stu-id="afc34-163">Correlate the instances of slow workloads with behavioral patterns.</span></span>
4. <span data-ttu-id="afc34-164">识别正在使用的数据存储。</span><span class="sxs-lookup"><span data-stu-id="afc34-164">Identify the data stores being used.</span></span> <span data-ttu-id="afc34-165">对于每个数据源，运行较低级别的遥测来观察操作行为。</span><span class="sxs-lookup"><span data-stu-id="afc34-165">For each data source, run lower level telemetry to observe the behavior of operations.</span></span>
6. <span data-ttu-id="afc34-166">识别引用这些数据源的所有缓慢运行的查询。</span><span class="sxs-lookup"><span data-stu-id="afc34-166">Identify any slow-running queries that reference these data sources.</span></span>
7. <span data-ttu-id="afc34-167">针对缓慢运行的查询执行特定于资源的分析，并确定数据的使用和消耗方式。</span><span class="sxs-lookup"><span data-stu-id="afc34-167">Perform a resource-specific analysis of the slow-running queries and ascertain how the data is used and consumed.</span></span>

<span data-ttu-id="afc34-168">确定是否存在以下任何症状：</span><span class="sxs-lookup"><span data-stu-id="afc34-168">Look for any of these symptoms:</span></span>

- <span data-ttu-id="afc34-169">频繁地向同一资源或数据存储发出大型 I/O 请求。</span><span class="sxs-lookup"><span data-stu-id="afc34-169">Frequent, large I/O requests made to the same resource or data store.</span></span>
- <span data-ttu-id="afc34-170">共享资源或数据存储中发生资源争用。</span><span class="sxs-lookup"><span data-stu-id="afc34-170">Contention in a shared resource or data store.</span></span>
- <span data-ttu-id="afc34-171">某个操作频繁通过网络接收大量数据。</span><span class="sxs-lookup"><span data-stu-id="afc34-171">An operation that frequently receives large volumes of data over the network.</span></span>
- <span data-ttu-id="afc34-172">应用程序和服务花费大量时间等待 I/O 完成。</span><span class="sxs-lookup"><span data-stu-id="afc34-172">Applications and services spending significant time waiting for I/O to complete.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="afc34-173">示例诊断</span><span class="sxs-lookup"><span data-stu-id="afc34-173">Example diagnosis</span></span>    

<span data-ttu-id="afc34-174">以下部分对前面的示例应用这些步骤。</span><span class="sxs-lookup"><span data-stu-id="afc34-174">The following sections apply these steps to the previous examples.</span></span>

### <a name="identify-slow-workloads"></a><span data-ttu-id="afc34-175">识别速度缓慢的工作负荷</span><span class="sxs-lookup"><span data-stu-id="afc34-175">Identify slow workloads</span></span>

<span data-ttu-id="afc34-176">此图显示了某项负载测试的性能结果。该测试模拟多达 400 个运行前面所示 `GetAllFieldsAsync` 方法的并发用户。</span><span class="sxs-lookup"><span data-stu-id="afc34-176">This graph shows performance results from a load test that simulated up to 400 concurrent users running the `GetAllFieldsAsync` method shown earlier.</span></span> <span data-ttu-id="afc34-177">随着负载的增大，吞吐量缓慢下降。</span><span class="sxs-lookup"><span data-stu-id="afc34-177">Throughput diminishes slowly as the load increases.</span></span> <span data-ttu-id="afc34-178">随着工作负荷的增大，平均响应时间不断提高。</span><span class="sxs-lookup"><span data-stu-id="afc34-178">Average response time goes up as the workload increases.</span></span> 

![GetAllFieldsAsync 方法的负载测试结果][Load-Test-Results-Client-Side1]

<span data-ttu-id="afc34-180">针对 `AggregateOnClientAsync` 操作执行的负载测试显示了类似的模式。</span><span class="sxs-lookup"><span data-stu-id="afc34-180">A load test for the `AggregateOnClientAsync` operation shows a similar pattern.</span></span> <span data-ttu-id="afc34-181">请求量适度稳定。</span><span class="sxs-lookup"><span data-stu-id="afc34-181">The volume of requests is reasonably stable.</span></span> <span data-ttu-id="afc34-182">平均响应时间随着工作负荷的增大而提高，不过，提高的速度比前一图表中要慢。</span><span class="sxs-lookup"><span data-stu-id="afc34-182">The average response time increases with the workload, although more slowly than the previous graph.</span></span>

![AggregateOnClientAsync 方法的负载测试结果][Load-Test-Results-Client-Side2]

### <a name="correlate-slow-workloads-with-behavioral-patterns"></a><span data-ttu-id="afc34-184">将慢速工作负荷与行为模式相关联</span><span class="sxs-lookup"><span data-stu-id="afc34-184">Correlate slow workloads with behavioral patterns</span></span>

<span data-ttu-id="afc34-185">定期的高使用率与缓慢性能之间的任何关联可以指明关注区域。</span><span class="sxs-lookup"><span data-stu-id="afc34-185">Any correlation between regular periods of high usage and slowing performance can indicate areas of concern.</span></span> <span data-ttu-id="afc34-186">仔细检查怀疑运行速度缓慢的功能的性能配置文件，确定它是否与前面执行的负载测试匹配。</span><span class="sxs-lookup"><span data-stu-id="afc34-186">Closely examine the performance profile of functionality that is suspected to be slow running, to determine whether it matches the load testing performed earlier.</span></span>

<span data-ttu-id="afc34-187">使用基于步骤的用户负载对同一功能进行负载测试，找出性能明显下降或完全失败的位置。</span><span class="sxs-lookup"><span data-stu-id="afc34-187">Load test the same functionality using step-based user loads, to find the point where performance drops significantly or fails completely.</span></span> <span data-ttu-id="afc34-188">如果该位置在预期的实际使用边界内，请检查该功能的实现方式</span><span class="sxs-lookup"><span data-stu-id="afc34-188">If that point falls within the bounds of your expected real-world usage, examine how the functionality is implemented.</span></span>

<span data-ttu-id="afc34-189">如果速度缓慢的操作不是在系统承受压力的情况下执行的、不是时间关键的，且不会对其他重要操作的性能造成负面影响，则该操作就不一定是个问题。</span><span class="sxs-lookup"><span data-stu-id="afc34-189">A slow operation is not necessarily a problem, if it is not being performed when the system is under stress, is not time critical, and does not negatively affect the performance of other important operations.</span></span> <span data-ttu-id="afc34-190">例如，生成每月操作统计信息可能是一个长时间运行的操作，但它可能是以批处理的形式执行的，并以低优先级作业的形式运行。</span><span class="sxs-lookup"><span data-stu-id="afc34-190">For example, generating monthly operational statistics might be a long-running operation, but it can probably be performed as a batch process and run as a low priority job.</span></span> <span data-ttu-id="afc34-191">另一方面，客户查询产品目录是一个关键的业务操作。</span><span class="sxs-lookup"><span data-stu-id="afc34-191">On the other hand, customers querying the product catalog is a critical business operation.</span></span> <span data-ttu-id="afc34-192">专注于这些关键操作生成的遥测数据可以了解重度使用期间的性能变化。</span><span class="sxs-lookup"><span data-stu-id="afc34-192">Focus on the telemetry generated by these critical operations to see how the performance varies during periods of high usage.</span></span>

### <a name="identify-data-sources-in-slow-workloads"></a><span data-ttu-id="afc34-193">识别慢速工作负荷中的数据源</span><span class="sxs-lookup"><span data-stu-id="afc34-193">Identify data sources in slow workloads</span></span>

<span data-ttu-id="afc34-194">如果怀疑某个服务由于检索数据的方式原因而性能不佳，请调查应用程序与其使用的存储库之间的交互方式。</span><span class="sxs-lookup"><span data-stu-id="afc34-194">If you suspect that a service is performing poorly because of the way it retrieves data, investigate how the application interacts with the repositories it uses.</span></span> <span data-ttu-id="afc34-195">监视实时系统，确定在性能不佳期间访问了哪些源。</span><span class="sxs-lookup"><span data-stu-id="afc34-195">Monitor the live system to see which sources are accessed during periods of poor performance.</span></span> 

<span data-ttu-id="afc34-196">对于每个数据源，检测系统以捕获以下信息：</span><span class="sxs-lookup"><span data-stu-id="afc34-196">For each data source, instrument the system to capture the following:</span></span>

- <span data-ttu-id="afc34-197">访问每个数据存储的频率。</span><span class="sxs-lookup"><span data-stu-id="afc34-197">The frequency that each data store is accessed.</span></span>
- <span data-ttu-id="afc34-198">进入和退出数据存储的数据量。</span><span class="sxs-lookup"><span data-stu-id="afc34-198">The volume of data entering and exiting the data store.</span></span>
- <span data-ttu-id="afc34-199">这些操作的计时，尤其是请求延迟。</span><span class="sxs-lookup"><span data-stu-id="afc34-199">The timing of these operations, especially the latency of requests.</span></span>
- <span data-ttu-id="afc34-200">在承受典型负载的情况下访问每个数据存储时发生的所有错误的性质和比率。</span><span class="sxs-lookup"><span data-stu-id="afc34-200">The nature and rate of any errors that occur while accessing each data store under typical load.</span></span>

<span data-ttu-id="afc34-201">将此信息与应用程序返回给客户端的数据量进行比较。</span><span class="sxs-lookup"><span data-stu-id="afc34-201">Compare this information against the volume of data being returned by the application to the client.</span></span> <span data-ttu-id="afc34-202">根据返回给客户端的数据量跟踪数据存储返回的数据量比率。</span><span class="sxs-lookup"><span data-stu-id="afc34-202">Track the ratio of the volume of data returned by the data store against the volume of data returned to the client.</span></span> <span data-ttu-id="afc34-203">如果有较大的差异，请调查确定应用程序是否在提取不需要的数据。</span><span class="sxs-lookup"><span data-stu-id="afc34-203">If there is any large disparity, investigate to determine whether the application is fetching data that it doesn't need.</span></span>

<span data-ttu-id="afc34-204">可以通过观察实时系统并跟踪每个用户请求的生命周期来捕获此数据，或者，可为一系列合成工作负荷建模，并针对测试系统运行这些工作负荷。</span><span class="sxs-lookup"><span data-stu-id="afc34-204">You may be able to capture this data by observing the live system and tracing the lifecycle of each user request, or you can model a series of synthetic workloads and run them against a test system.</span></span>

<span data-ttu-id="afc34-205">下图显示了在对 `GetAllFieldsAsync` 方法执行负载测试期间，使用 [New Relic APM][new-relic] 捕获的遥测数据。</span><span class="sxs-lookup"><span data-stu-id="afc34-205">The following graphs show telemetry captured using [New Relic APM][new-relic] during a load test of the `GetAllFieldsAsync` method.</span></span> <span data-ttu-id="afc34-206">请注意从数据库收到的数据量与相应 HTTP 响应之间的差异。</span><span class="sxs-lookup"><span data-stu-id="afc34-206">Note the difference between the volumes of data received from the database and the corresponding HTTP responses.</span></span>

![“GetAllFieldsAsync”方法的遥测数据][TelemetryAllFields]

<span data-ttu-id="afc34-208">对于每个请求，数据库返回了 80,503 字节，但返回给客户端的响应仅包含 19,855 字节，大约为数据库响应大小的 25%。</span><span class="sxs-lookup"><span data-stu-id="afc34-208">For each request, the database returned 80,503 bytes, but the response to the client only contained 19,855 bytes, about 25% of the size of the database response.</span></span> <span data-ttu-id="afc34-209">返回给客户端的数据大小可能会根据格式的不同而异。</span><span class="sxs-lookup"><span data-stu-id="afc34-209">The size of the data returned to the client can vary depending on the format.</span></span> <span data-ttu-id="afc34-210">在此负载测试中，客户端请求了 JSON 数据。</span><span class="sxs-lookup"><span data-stu-id="afc34-210">For this load test, the client requested JSON data.</span></span> <span data-ttu-id="afc34-211">使用 XML 执行的单独测试（未显示）返回的响应大小为 35,655 字节，即数据库响应大小的 44%。</span><span class="sxs-lookup"><span data-stu-id="afc34-211">Separate testing using XML (not shown) had a response size of 35,655 bytes, or 44% of the size of the database response.</span></span>

<span data-ttu-id="afc34-212">针对 `AggregateOnClientAsync` 方法的负载测试显示了更极端的结果。</span><span class="sxs-lookup"><span data-stu-id="afc34-212">The load test for the `AggregateOnClientAsync` method shows more extreme results.</span></span> <span data-ttu-id="afc34-213">在本例中，每项测试执行了一个从数据库中检索超过 280Kb 数据的查询，但 JSON 响应只有 14 个字节。</span><span class="sxs-lookup"><span data-stu-id="afc34-213">In this case, each test performed a query that retrieved over 280Kb of data from the database, but the JSON response was a mere 14 bytes.</span></span> <span data-ttu-id="afc34-214">之所以出现这么大的差异，是因为该方法基于大量数据计算了聚合结果。</span><span class="sxs-lookup"><span data-stu-id="afc34-214">The wide disparity is because the method calculates an aggregated result from a large volume of data.</span></span>

![“AggregateOnClientAsync”方法的遥测数据][TelemetryAggregateOnClient]

### <a name="identify-and-analyze-slow-queries"></a><span data-ttu-id="afc34-216">识别和分析慢速查询</span><span class="sxs-lookup"><span data-stu-id="afc34-216">Identify and analyze slow queries</span></span>

<span data-ttu-id="afc34-217">找出消耗最多资源且执行时间最长的数据库查询。</span><span class="sxs-lookup"><span data-stu-id="afc34-217">Look for database queries that consume the most resources and take the most time to execute.</span></span> <span data-ttu-id="afc34-218">可以添加检测来找到许多数据库操作的开始时间和完成时间。</span><span class="sxs-lookup"><span data-stu-id="afc34-218">You can add instrumentation to find the start and completion times for many database operations.</span></span> <span data-ttu-id="afc34-219">许多数据存储还提供有关查询执行和优化方式的深入信息。</span><span class="sxs-lookup"><span data-stu-id="afc34-219">Many data stores also provide in-depth information on how queries are performed and optimized.</span></span> <span data-ttu-id="afc34-220">例如，可以通过 Azure SQL 数据库管理门户中的“查询性能”窗格选择某个查询，并查看详细的运行时性能信息。</span><span class="sxs-lookup"><span data-stu-id="afc34-220">For example, the Query Performance pane in the Azure SQL Database management portal lets you select a query and view detailed runtime performance information.</span></span> <span data-ttu-id="afc34-221">下面是 `GetAllFieldsAsync` 操作生成的查询：</span><span class="sxs-lookup"><span data-stu-id="afc34-221">Here is the query generated by the `GetAllFieldsAsync` operation:</span></span>

![Windows Azure SQL 数据库管理门户中的“查询详细信息”窗格][QueryDetails]


## <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="afc34-223">实施解决方案并验证结果</span><span class="sxs-lookup"><span data-stu-id="afc34-223">Implement the solution and verify the result</span></span>

<span data-ttu-id="afc34-224">在数据库端将 `GetRequiredFieldsAsync` 方法更改为使用 SELECT 语句后，负载测试显示了以下结果。</span><span class="sxs-lookup"><span data-stu-id="afc34-224">After changing the `GetRequiredFieldsAsync` method to use a SELECT statement on the database side, load testing showed the following results.</span></span>

![GetRequiredFieldsAsync 方法的负载测试结果][Load-Test-Results-Database-Side1]

<span data-ttu-id="afc34-226">此负载测试使用了与前面相同的部署以及包含 400 个并发用户的模拟工作负荷。</span><span class="sxs-lookup"><span data-stu-id="afc34-226">This load test used the same deployment and the same simulated workload of 400 concurrent users as before.</span></span> <span data-ttu-id="afc34-227">图中显示的延迟要低得多。</span><span class="sxs-lookup"><span data-stu-id="afc34-227">The graph shows much lower latency.</span></span> <span data-ttu-id="afc34-228">随着负载的增大，响应时间提升到大约 1.3 秒，而在前一案例中，响应时间为 4 秒。</span><span class="sxs-lookup"><span data-stu-id="afc34-228">Response time rises with load to approximately 1.3 seconds, compared to 4 seconds in the previous case.</span></span> <span data-ttu-id="afc34-229">吞吐量也已提高，为每秒 350 个请求，而在前一案例中为 100。</span><span class="sxs-lookup"><span data-stu-id="afc34-229">The throughput is also higher at 350 requests per second compared to 100 earlier.</span></span> <span data-ttu-id="afc34-230">从数据库中检索的数据量现在与 HTTP 响应消息的大小非常接近。</span><span class="sxs-lookup"><span data-stu-id="afc34-230">The volume of data retrieved from the database now closely matches the size of the HTTP response messages.</span></span>

![“GetRequiredFieldsAsync”方法的遥测数据][TelemetryRequiredFields]

<span data-ttu-id="afc34-232">使用 `AggregateOnDatabaseAsync` 方法执行的负载测试生成了以下结果：</span><span class="sxs-lookup"><span data-stu-id="afc34-232">Load testing using the `AggregateOnDatabaseAsync` method generates the following results:</span></span>

![AggregateOnDatabaseAsync 方法的负载测试结果][Load-Test-Results-Database-Side2]

<span data-ttu-id="afc34-234">现在，平均响应时间极短。</span><span class="sxs-lookup"><span data-stu-id="afc34-234">The average response time is now minimal.</span></span> <span data-ttu-id="afc34-235">这是数量级的性能改善，主要原因是来自数据库的 I/O 大幅减少。</span><span class="sxs-lookup"><span data-stu-id="afc34-235">This is an order of magnitude improvement in performance, caused primarily by the large reduction in I/O from the database.</span></span> 

<span data-ttu-id="afc34-236">下面是 `AggregateOnDatabaseAsync` 方法的相应遥测数据。</span><span class="sxs-lookup"><span data-stu-id="afc34-236">Here is the corresponding telemetry for the `AggregateOnDatabaseAsync` method.</span></span> <span data-ttu-id="afc34-237">从数据库中检索的数据量大大减少，从每个事务 280Kb 以上缩减到 53 个字节。</span><span class="sxs-lookup"><span data-stu-id="afc34-237">The amount of data retrieved from the database was vastly reduced, from over 280Kb per transaction to 53 bytes.</span></span> <span data-ttu-id="afc34-238">因此，每分钟的最大持续请求数已从大约 2,000 提升到 25,000 以上。</span><span class="sxs-lookup"><span data-stu-id="afc34-238">As a result, the maximum sustained number of requests per minute was raised from around 2,000 to over 25,000.</span></span>

![“AggregateOnDatabaseAsync”方法的遥测数据][TelemetryAggregateInDatabaseAsync]


## <a name="related-resources"></a><span data-ttu-id="afc34-240">相关资源</span><span class="sxs-lookup"><span data-stu-id="afc34-240">Related resources</span></span>

- <span data-ttu-id="afc34-241">[繁忙数据库对立模式][BusyDatabase]</span><span class="sxs-lookup"><span data-stu-id="afc34-241">[Busy Database antipattern][BusyDatabase]</span></span>
- <span data-ttu-id="afc34-242">[琐碎 I/O 对立模式][chatty-io]</span><span class="sxs-lookup"><span data-stu-id="afc34-242">[Chatty I/O antipattern][chatty-io]</span></span>
- <span data-ttu-id="afc34-243">[数据分区最佳做法][data-partitioning]</span><span class="sxs-lookup"><span data-stu-id="afc34-243">[Data partitioning best practices][data-partitioning]</span></span>


[BusyDatabase]: ../busy-database/index.md
[chatty-io]: ../chatty-io.md
[data-partitioning]: ../../best-practices/data-partitioning.md
[new-relic]: https://newrelic.com/application-monitoring

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ExtraneousFetching

[chatty-io]: ../chatty-io/index.md
[MonolithicPersistence]: ../monolithic-persistence/index.md
[Load-Test-Results-Client-Side1]:_images/LoadTestResultsClientSide1.jpg
[Load-Test-Results-Client-Side2]:_images/LoadTestResultsClientSide2.jpg
[Load-Test-Results-Database-Side1]:_images/LoadTestResultsDatabaseSide1.jpg
[Load-Test-Results-Database-Side2]:_images/LoadTestResultsDatabaseSide2.jpg
[QueryPerformanceZoomed]: _images/QueryPerformanceZoomed.jpg
[QueryDetails]: _images/QueryDetails.jpg
[TelemetryAllFields]: _images/TelemetryAllFields.jpg
[TelemetryAggregateOnClient]: _images/TelemetryAggregateOnClient.jpg
[TelemetryRequiredFields]: _images/TelemetryRequiredFields.jpg
[TelemetryAggregateInDatabaseAsync]: _images/TelemetryAggregateInDatabase.jpg
