---
title: 琐碎 I/O 对立模式
description: 发出大量的 I/O 请求可能会损害性能和响应能力。
author: dragon119
ms.openlocfilehash: 4f0e0e455ceb58317d3029d8ab4631d476802499
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/23/2018
---
# <a name="chatty-io-antipattern"></a><span data-ttu-id="db644-103">琐碎 I/O 对立模式</span><span class="sxs-lookup"><span data-stu-id="db644-103">Chatty I/O antipattern</span></span>

<span data-ttu-id="db644-104">大量 I/O 请求的累积效应可能会对性能和响应能力产生明显的不利影响。</span><span class="sxs-lookup"><span data-stu-id="db644-104">The cumulative effect of a large number of I/O requests can have a significant impact on performance and responsiveness.</span></span>

## <a name="problem-description"></a><span data-ttu-id="db644-105">问题描述</span><span class="sxs-lookup"><span data-stu-id="db644-105">Problem description</span></span>

<span data-ttu-id="db644-106">网络调用和其他 I/O 操作的速度与生俱来就比计算任务要慢。</span><span class="sxs-lookup"><span data-stu-id="db644-106">Network calls and other I/O operations are inherently slow compared to compute tasks.</span></span> <span data-ttu-id="db644-107">每个 I/O 请求通常会产生很高的开销，大量 I/O 操作的累积效应可能会使系统变慢。</span><span class="sxs-lookup"><span data-stu-id="db644-107">Each I/O request typically has significant overhead, and the cumulative effect of numerous I/O operations can slow down the system.</span></span> <span data-ttu-id="db644-108">下面是出现琐碎 I/O 的一些常见原因。</span><span class="sxs-lookup"><span data-stu-id="db644-108">Here are some common causes of chatty I/O.</span></span>

### <a name="reading-and-writing-individual-records-to-a-database-as-distinct-requests"></a><span data-ttu-id="db644-109">以不同请求的形式在数据库中读取和写入单个记录</span><span class="sxs-lookup"><span data-stu-id="db644-109">Reading and writing individual records to a database as distinct requests</span></span>

<span data-ttu-id="db644-110">以下示例从产品数据库中读取数据。</span><span class="sxs-lookup"><span data-stu-id="db644-110">The following example reads from a database of products.</span></span> <span data-ttu-id="db644-111">有三个表：`Product`、`ProductSubcategory` 和 `ProductPriceListHistory`。</span><span class="sxs-lookup"><span data-stu-id="db644-111">There are three tables, `Product`, `ProductSubcategory`, and `ProductPriceListHistory`.</span></span> <span data-ttu-id="db644-112">代码通过执行一系列查询检索子类别中的所有产品以及价格信息：</span><span class="sxs-lookup"><span data-stu-id="db644-112">The code retrieves all of the products in a subcategory, along with the pricing information, by executing a series of queries:</span></span>  

1. <span data-ttu-id="db644-113">查询 `ProductSubcategory` 表中的子类别。</span><span class="sxs-lookup"><span data-stu-id="db644-113">Query the subcategory from the `ProductSubcategory` table.</span></span>
2. <span data-ttu-id="db644-114">通过查询 `Product` 表查找该子类别中的所有产品。</span><span class="sxs-lookup"><span data-stu-id="db644-114">Find all products in that subcategory by querying the `Product` table.</span></span>
3. <span data-ttu-id="db644-115">对于每个产品，查询 `ProductPriceListHistory` 表中的价格数据。</span><span class="sxs-lookup"><span data-stu-id="db644-115">For each product, query the pricing data from the `ProductPriceListHistory` table.</span></span>

<span data-ttu-id="db644-116">应用程序使用[实体框架][ef]查询数据库。</span><span class="sxs-lookup"><span data-stu-id="db644-116">The application uses [Entity Framework][ef] to query the database.</span></span> <span data-ttu-id="db644-117">可在[此处][code-sample]找到完整示例。</span><span class="sxs-lookup"><span data-stu-id="db644-117">You can find the complete sample [here][code-sample].</span></span> 

```csharp
public async Task<IHttpActionResult> GetProductsInSubCategoryAsync(int subcategoryId)
{
    using (var context = GetContext())
    {
        // Get product subcategory.
        var productSubcategory = await context.ProductSubcategories
                .Where(psc => psc.ProductSubcategoryId == subcategoryId)
                .FirstOrDefaultAsync();

        // Find products in that category.
        productSubcategory.Product = await context.Products
            .Where(p => subcategoryId == p.ProductSubcategoryId)
            .ToListAsync();

        // Find price history for each product.
        foreach (var prod in productSubcategory.Product)
        {
            int productId = prod.ProductId;
            var productListPriceHistory = await context.ProductListPriceHistory
                .Where(pl => pl.ProductId == productId)
                .ToListAsync();
            prod.ProductListPriceHistory = productListPriceHistory;
        }
        return Ok(productSubcategory);
    }
}
```

<span data-ttu-id="db644-118">此示例显式显示了问题，但有时，如果 O/RM 逐个地隐式提取子记录，则可能会掩盖问题。</span><span class="sxs-lookup"><span data-stu-id="db644-118">This example shows the problem explicitly, but sometimes an O/RM can mask the problem, if it implicitly fetches child records one at a time.</span></span> <span data-ttu-id="db644-119">这就是所谓的“N+1 问题”。</span><span class="sxs-lookup"><span data-stu-id="db644-119">This is known as the "N+1 problem".</span></span> 

### <a name="implementing-a-single-logical-operation-as-a-series-of-http-requests"></a><span data-ttu-id="db644-120">以一系列 HTTP 请求的形式执行单个逻辑操作</span><span class="sxs-lookup"><span data-stu-id="db644-120">Implementing a single logical operation as a series of HTTP requests</span></span>

<span data-ttu-id="db644-121">当开发人员尝试遵循面向对象的范例，并将远程对象视为内存中的本地对象时，就往往会发生这种情况。</span><span class="sxs-lookup"><span data-stu-id="db644-121">This often happens when developers try to follow an object-oriented paradigm, and treat remote objects as if they were local objects in memory.</span></span> <span data-ttu-id="db644-122">这可能导致过多的网络往返。</span><span class="sxs-lookup"><span data-stu-id="db644-122">This can result in too many network round trips.</span></span> <span data-ttu-id="db644-123">例如，以下 Web API 通过单个 HTTP GET 方法公开 `User` 对象的单个属性。</span><span class="sxs-lookup"><span data-stu-id="db644-123">For example, the following web API exposes the individual properties of `User` objects through individual HTTP GET methods.</span></span> 

```csharp
public class UserController : ApiController
{
    [HttpGet]
    [Route("users/{id:int}/username")]
    public HttpResponseMessage GetUserName(int id)
    {
        ...
    }

    [HttpGet]
    [Route("users/{id:int}/gender")]
    public HttpResponseMessage GetGender(int id)
    {
        ...
    }

    [HttpGet]
    [Route("users/{id:int}/dateofbirth")]
    public HttpResponseMessage GetDateOfBirth(int id)
    {
        ...
    }
}
```

<span data-ttu-id="db644-124">尽管此方法在技术上没有任何问题，但是，大多数客户端可能需要获取每个 `User` 的多个属性，从而导致需要编写如下所示的客户端代码。</span><span class="sxs-lookup"><span data-stu-id="db644-124">While there's nothing technically wrong with this approach, most clients will probably need to get several properties for each `User`, resulting in client code like the following.</span></span> 

```csharp
HttpResponseMessage response = await client.GetAsync("users/1/username");
response.EnsureSuccessStatusCode();
var userName = await response.Content.ReadAsStringAsync();

response = await client.GetAsync("users/1/gender");
response.EnsureSuccessStatusCode();
var gender = await response.Content.ReadAsStringAsync();

response = await client.GetAsync("users/1/dateofbirth");
response.EnsureSuccessStatusCode();
var dob = await response.Content.ReadAsStringAsync();
```

### <a name="reading-and-writing-to-a-file-on-disk"></a><span data-ttu-id="db644-125">读取和写入磁盘中的文件</span><span class="sxs-lookup"><span data-stu-id="db644-125">Reading and writing to a file on disk</span></span>

<span data-ttu-id="db644-126">文件 I/O 涉及到打开某个文件并转到相应的点，然后读取或写入数据。</span><span class="sxs-lookup"><span data-stu-id="db644-126">File I/O involves opening a file and moving to the appropriate point before reading or writing data.</span></span> <span data-ttu-id="db644-127">完成该操作后，文件可能会关闭，以节省操作系统资源。</span><span class="sxs-lookup"><span data-stu-id="db644-127">When the operation is complete, the file might be closed to save operating system resources.</span></span> <span data-ttu-id="db644-128">持续在文件中读取和写入少量信息的应用程序会产生很高的 I/O 开销。</span><span class="sxs-lookup"><span data-stu-id="db644-128">An application that continually reads and writes small amounts of information to a file will generate significant I/O overhead.</span></span> <span data-ttu-id="db644-129">小规模写入请求还可能导致文件碎片，从而进一步减慢后续 I/O 操作的速度。</span><span class="sxs-lookup"><span data-stu-id="db644-129">Small write requests can also lead to file fragmentation, slowing subsequent I/O operations still further.</span></span> 

<span data-ttu-id="db644-130">以下示例使用 `FileStream` 将 `Customer` 对象写入文件。</span><span class="sxs-lookup"><span data-stu-id="db644-130">The following example uses a `FileStream` to write a `Customer` object to a file.</span></span> <span data-ttu-id="db644-131">创建 `FileStream` 会打开该文件，释放它会关闭该文件。</span><span class="sxs-lookup"><span data-stu-id="db644-131">Creating the `FileStream` opens the file, and disposing it closes the file.</span></span> <span data-ttu-id="db644-132">（`using` 语句自动释放 `FileStream` 对象。）如果由于添加新客户，应用程序反复调用此方法，I/O 开销可能会迅速累积。</span><span class="sxs-lookup"><span data-stu-id="db644-132">(The `using` statement automatically disposes the `FileStream` object.) If the application calls this method repeatedly as new customers are added, the I/O overhead can accumulate quickly.</span></span>

```csharp
private async Task SaveCustomerToFileAsync(Customer cust)
{
    using (Stream fileStream = new FileStream(CustomersFileName, FileMode.Append))
    {
        BinaryFormatter formatter = new BinaryFormatter();
        byte [] data = null;
        using (MemoryStream memStream = new MemoryStream())
        {
            formatter.Serialize(memStream, cust);
            data = memStream.ToArray();
        }
        await fileStream.WriteAsync(data, 0, data.Length);
    }
}
```

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="db644-133">如何解决问题</span><span class="sxs-lookup"><span data-stu-id="db644-133">How to fix the problem</span></span>

<span data-ttu-id="db644-134">通过将数据打包成更大但更少的请求来减少 I/O 请求的数量。</span><span class="sxs-lookup"><span data-stu-id="db644-134">Reduce the number of I/O requests by packaging the data into larger, fewer requests.</span></span>

<span data-ttu-id="db644-135">以单个查询而不是多个较小查询的形式从数据库中提取数据。</span><span class="sxs-lookup"><span data-stu-id="db644-135">Fetch data from a database as a single query, instead of several smaller queries.</span></span> <span data-ttu-id="db644-136">下面是检索产品信息的代码的修改版本。</span><span class="sxs-lookup"><span data-stu-id="db644-136">Here's a revised version of the code that retrieves product information.</span></span>

```csharp
public async Task<IHttpActionResult> GetProductCategoryDetailsAsync(int subCategoryId)
{
    using (var context = GetContext())
    {
        var subCategory = await context.ProductSubcategories
                .Where(psc => psc.ProductSubcategoryId == subCategoryId)
                .Include("Product.ProductListPriceHistory")
                .FirstOrDefaultAsync();

        if (subCategory == null)
            return NotFound();

        return Ok(subCategory);
    }
}
```

<span data-ttu-id="db644-137">遵循 Web API 的 REST 设计原则。</span><span class="sxs-lookup"><span data-stu-id="db644-137">Follow REST design principles for web APIs.</span></span> <span data-ttu-id="db644-138">下面是前一示例中 Web API 的修改版本。</span><span class="sxs-lookup"><span data-stu-id="db644-138">Here's a revised version of the web API from the earlier example.</span></span> <span data-ttu-id="db644-139">不要针对每个属性单独使用 GET 方法，而可以使用单个返回 `User` 的 GET 方法。</span><span class="sxs-lookup"><span data-stu-id="db644-139">Instead of separate GET methods for each property, there is a single GET method that returns the `User`.</span></span> <span data-ttu-id="db644-140">这会导致每个请求的响应正文变得更大，但每个客户端可能会发出更少的 API 调用。</span><span class="sxs-lookup"><span data-stu-id="db644-140">This results in a larger response body per request, but each client is likely to make fewer API calls.</span></span>

```csharp
public class UserController : ApiController
{
    [HttpGet]
    [Route("users/{id:int}")]
    public HttpResponseMessage GetUser(int id)
    {
        ...
    }
}

// Client code
HttpResponseMessage response = await client.GetAsync("users/1");
response.EnsureSuccessStatusCode();
var user = await response.Content.ReadAsStringAsync();
```

<span data-ttu-id="db644-141">对于文件 I/O，请考虑在内存中缓冲数据，然后以单个操作的形式将缓冲的数据写入文件。</span><span class="sxs-lookup"><span data-stu-id="db644-141">For file I/O, consider buffering data in memory and then writing the buffered data to a file as a single operation.</span></span> <span data-ttu-id="db644-142">此方法可以减少由于反复打开和关闭文件产生的开销，并有助于减少磁盘中文件的碎片。</span><span class="sxs-lookup"><span data-stu-id="db644-142">This approach reduces the overhead from repeatedly opening and closing the file, and helps to reduce fragmentation of the file on disk.</span></span>

```csharp
// Save a list of customer objects to a file
private async Task SaveCustomerListToFileAsync(List<Customer> customers)
{
    using (Stream fileStream = new FileStream(CustomersFileName, FileMode.Append))
    {
        BinaryFormatter formatter = new BinaryFormatter();
        foreach (var cust in customers)
        {
            byte[] data = null;
            using (MemoryStream memStream = new MemoryStream())
            {
                formatter.Serialize(memStream, cust);
                data = memStream.ToArray();
            }
            await fileStream.WriteAsync(data, 0, data.Length);
        }
    }
}

// In-memory buffer for customers.
List<Customer> customers = new List<Customers>();

// Create a new customer and add it to the buffer
var cust = new Customer(...);
customers.Add(cust);

// Add more customers to the list as they are created
...

// Save the contents of the list, writing all customers in a single operation
await SaveCustomerListToFileAsync(customers);
```

## <a name="considerations"></a><span data-ttu-id="db644-143">注意事项</span><span class="sxs-lookup"><span data-stu-id="db644-143">Considerations</span></span>

- <span data-ttu-id="db644-144">前两个示例发出更少的 I/O 调用，但每个示例检索了更多的信息。</span><span class="sxs-lookup"><span data-stu-id="db644-144">The first two examples make *fewer* I/O calls, but each one retrieves *more* information.</span></span> <span data-ttu-id="db644-145">必须考虑这两种因素的利弊。</span><span class="sxs-lookup"><span data-stu-id="db644-145">You must consider the tradeoff between these two factors.</span></span> <span data-ttu-id="db644-146">正确的答案取决于实际使用模式。</span><span class="sxs-lookup"><span data-stu-id="db644-146">The right answer will depend on the actual usage patterns.</span></span> <span data-ttu-id="db644-147">例如，在 Web API 示例中，客户端可能往往只需检索用户名。</span><span class="sxs-lookup"><span data-stu-id="db644-147">For example, in the web API example, it might turn out that clients often need just the user name.</span></span> <span data-ttu-id="db644-148">在这种情况下，将该操作公开为单独的 API 调用可能有利。</span><span class="sxs-lookup"><span data-stu-id="db644-148">In that case, it might make sense to expose it as a separate API call.</span></span> <span data-ttu-id="db644-149">有关详细信息，请参阅[超量提取][extraneous-fetching]对立模式。</span><span class="sxs-lookup"><span data-stu-id="db644-149">For more information, see the [Extraneous Fetching][extraneous-fetching] antipattern.</span></span>

- <span data-ttu-id="db644-150">读取数据时，请不要发出过大的 I/O 请求。</span><span class="sxs-lookup"><span data-stu-id="db644-150">When reading data, do not make your I/O requests too large.</span></span> <span data-ttu-id="db644-151">应用程序应该只检索它可能要使用的信息。</span><span class="sxs-lookup"><span data-stu-id="db644-151">An application should only retrieve the information that it is likely to use.</span></span> 

- <span data-ttu-id="db644-152">有时，将对象的信息分区成以下两个区块可能会有帮助：经常访问的数据（大多数请求就是针对这些数据发出的），不经常访问的数据（极少使用的数据）。</span><span class="sxs-lookup"><span data-stu-id="db644-152">Sometimes it helps to partition the information for an object into two chunks, *frequently accessed data* that accounts for most requests, and *less frequently accessed data* that is used rarely.</span></span> <span data-ttu-id="db644-153">最常访问的数据往往是对象总体数据中的相对较小一部分，因此，只返回这一部分数据能够大幅节省 I/O 开销。</span><span class="sxs-lookup"><span data-stu-id="db644-153">Often the most frequently accessed data is a relatively small portion of the total data for an object, so returning just that portion can save significant I/O overhead.</span></span>

- <span data-ttu-id="db644-154">写入数据时，请避免将资源锁定超过必要的时间，以减少在执行冗长操作期间发生资源争用的可能性。</span><span class="sxs-lookup"><span data-stu-id="db644-154">When writing data, avoid locking resources for longer than necessary, to reduce the chances of contention during a lengthy operation.</span></span> <span data-ttu-id="db644-155">如果写入操作跨多个数据存储、文件或服务，则采用最终一致性方法。</span><span class="sxs-lookup"><span data-stu-id="db644-155">If a write operation spans multiple data stores, files, or services, then adopt an eventually consistent approach.</span></span> <span data-ttu-id="db644-156">请参阅[数据一致性指南][data-consistency-guidance]。</span><span class="sxs-lookup"><span data-stu-id="db644-156">See [Data Consistency guidance][data-consistency-guidance].</span></span>

- <span data-ttu-id="db644-157">如果在写入数据之前在内存中缓冲数据，则发生进程崩溃时，数据易受攻击。</span><span class="sxs-lookup"><span data-stu-id="db644-157">If you buffer data in memory before writing it, the data is vulnerable if the process crashes.</span></span> <span data-ttu-id="db644-158">如果数据率通常出现喷发或相对稀疏，在[事件中心](http://azure.microsoft.com/services/event-hubs/)等外部持久队列中缓冲数据可能会更安全。</span><span class="sxs-lookup"><span data-stu-id="db644-158">If the data rate typically has bursts or is relatively sparse, it may be safer to buffer the data in an external durable queue such as [Event Hubs](http://azure.microsoft.com/services/event-hubs/).</span></span>

- <span data-ttu-id="db644-159">请考虑缓存从服务或数据库检索的数据。</span><span class="sxs-lookup"><span data-stu-id="db644-159">Consider caching data that you retrieve from a service or a database.</span></span> <span data-ttu-id="db644-160">这可以避免针对相同的数据发出重复请求，从而帮助减少 I/O 数量。</span><span class="sxs-lookup"><span data-stu-id="db644-160">This can help to reduce the volume of I/O by avoiding repeated requests for the same data.</span></span> <span data-ttu-id="db644-161">有关详细信息，请参阅[有关缓存的最佳做法][caching-guidance]。</span><span class="sxs-lookup"><span data-stu-id="db644-161">For more information, see [Caching best practices][caching-guidance].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="db644-162">如何检测问题</span><span class="sxs-lookup"><span data-stu-id="db644-162">How to detect the problem</span></span>

<span data-ttu-id="db644-163">琐碎 I/O 的症状包括高延迟和低吞吐量。</span><span class="sxs-lookup"><span data-stu-id="db644-163">Symptoms of chatty I/O include high latency and low throughput.</span></span> <span data-ttu-id="db644-164">由于 I/O 资源争用加剧，最终用户可能会反映响应时间延长，或服务超时导致失败。</span><span class="sxs-lookup"><span data-stu-id="db644-164">End users are likely to report extended response times or failures caused by services timing out, due to increased contention for I/O resources.</span></span>

<span data-ttu-id="db644-165">可执行以下步骤来帮助确定任何问题的原因：</span><span class="sxs-lookup"><span data-stu-id="db644-165">You can perform the following steps to help identify the causes of any problems:</span></span>

1. <span data-ttu-id="db644-166">对生产系统执行进程监视，识别响应时间不佳的操作。</span><span class="sxs-lookup"><span data-stu-id="db644-166">Perform process monitoring of the production system to identify operations with poor response times.</span></span>
2. <span data-ttu-id="db644-167">对上一步骤中识别到的每个操作执行负载测试。</span><span class="sxs-lookup"><span data-stu-id="db644-167">Perform load testing of each operation identified in the previous step.</span></span>
3. <span data-ttu-id="db644-168">在负载测试期间，收集有关每个操作发出的数据访问请求的遥测数据。</span><span class="sxs-lookup"><span data-stu-id="db644-168">During the load tests, gather telemetry data about the data access requests made by each operation.</span></span>
4. <span data-ttu-id="db644-169">收集已发送到数据存储的每个请求的详细统计信息。</span><span class="sxs-lookup"><span data-stu-id="db644-169">Gather detailed statistics for each request sent to a data store.</span></span>
5. <span data-ttu-id="db644-170">在测试环境中分析应用程序，判定可能出现 I/O 瓶颈的位置。</span><span class="sxs-lookup"><span data-stu-id="db644-170">Profile the application in the test environment to establish where possible I/O bottlenecks might be occurring.</span></span> 

<span data-ttu-id="db644-171">确定是否存在以下任何症状：</span><span class="sxs-lookup"><span data-stu-id="db644-171">Look for any of these symptoms:</span></span>

- <span data-ttu-id="db644-172">向同一个文件发出大量的小型 I/O 请求。</span><span class="sxs-lookup"><span data-stu-id="db644-172">A large number of small I/O requests made to the same file.</span></span>
- <span data-ttu-id="db644-173">某个应用程序实例向同一个服务发出大量的小型网络请求。</span><span class="sxs-lookup"><span data-stu-id="db644-173">A large number of small network requests made by an application instance to the same service.</span></span>
- <span data-ttu-id="db644-174">某个应用程序实例向同一个数据存储发出大量的小型请求。</span><span class="sxs-lookup"><span data-stu-id="db644-174">A large number of small requests made by an application instance to the same data store.</span></span>
- <span data-ttu-id="db644-175">应用程序和服务受 I/O 约束。</span><span class="sxs-lookup"><span data-stu-id="db644-175">Applications and services becoming I/O bound.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="db644-176">示例诊断</span><span class="sxs-lookup"><span data-stu-id="db644-176">Example diagnosis</span></span>

<span data-ttu-id="db644-177">以下部分将这些步骤应用到前面所示的查询数据库的示例。</span><span class="sxs-lookup"><span data-stu-id="db644-177">The following sections apply these steps to the example shown earlier that queries a database.</span></span>

### <a name="load-test-the-application"></a><span data-ttu-id="db644-178">对应用程序进行负载测试</span><span class="sxs-lookup"><span data-stu-id="db644-178">Load test the application</span></span>

<span data-ttu-id="db644-179">此图显示了负载测试的结果。</span><span class="sxs-lookup"><span data-stu-id="db644-179">This graph shows the results of load testing.</span></span> <span data-ttu-id="db644-180">中间响应时间是根据每个请求在数十秒内的表现测得的。</span><span class="sxs-lookup"><span data-stu-id="db644-180">Median response time is measured in 10s of seconds per request.</span></span> <span data-ttu-id="db644-181">该图显示延迟很高。</span><span class="sxs-lookup"><span data-stu-id="db644-181">The graph shows very high latency.</span></span> <span data-ttu-id="db644-182">每加载 1000 个用户，用户就可能需要等待将近一分钟才能看到查询结果。</span><span class="sxs-lookup"><span data-stu-id="db644-182">With a load of 1000 users, a user might have to wait for nearly a minute to see the results of a query.</span></span> 

![琐碎 I/O 示例应用程序的关键指标负载测试结果][key-indicators-chatty-io]

> [!NOTE]
> <span data-ttu-id="db644-184">该应用程序是使用 Azure SQL 数据库作为 Azure 应用服务 Web 应用部署的。</span><span class="sxs-lookup"><span data-stu-id="db644-184">The application was deployed as an Azure App Service web app, using Azure SQL Database.</span></span> <span data-ttu-id="db644-185">负载测试使用了包含多达 1000 个并发用户的模拟步骤工作负荷。</span><span class="sxs-lookup"><span data-stu-id="db644-185">The load test used a simulated step workload of up to 1000 concurrent users.</span></span> <span data-ttu-id="db644-186">数据库中配置了支持最多 1000 个并发连接的连接池，以减少出现连接争用，从而影响结果的可能性。</span><span class="sxs-lookup"><span data-stu-id="db644-186">The database was configured with a connection pool supporting up to 1000 concurrent connections, to reduce the chance that contention for connections would affect the results.</span></span> 

### <a name="monitor-the-application"></a><span data-ttu-id="db644-187">监视应用程序</span><span class="sxs-lookup"><span data-stu-id="db644-187">Monitor the application</span></span>

<span data-ttu-id="db644-188">可以使用应用程序性能监视 (APM) 包来捕获和分析可识别琐碎 I/O 的关键指标。</span><span class="sxs-lookup"><span data-stu-id="db644-188">You can use an application performance monitoring (APM) package to capture and analyze the key metrics that might identify chatty I/O.</span></span> <span data-ttu-id="db644-189">至于哪些指标比较重要，将取决于 I/O 工作负荷。</span><span class="sxs-lookup"><span data-stu-id="db644-189">Which metrics are important will depend on the I/O workload.</span></span> <span data-ttu-id="db644-190">对于此示例，要关注的 I/O 请求是数据库查询。</span><span class="sxs-lookup"><span data-stu-id="db644-190">For this example, the interesting I/O requests were the database queries.</span></span> 

<span data-ttu-id="db644-191">下图显示了使用 [New Relic APM][new-relic] 生成的结果。</span><span class="sxs-lookup"><span data-stu-id="db644-191">The following image shows results generated using [New Relic APM][new-relic].</span></span> <span data-ttu-id="db644-192">在承受最大工作负荷期间，平均数据库响应时间的峰值出现在每个请求的大约 5.6 秒处。</span><span class="sxs-lookup"><span data-stu-id="db644-192">The average database response time peaked at approximately 5.6 seconds per request during the maximum workload.</span></span> <span data-ttu-id="db644-193">在整个测试过程中，系统能够支持每分钟平均 410 个请求。</span><span class="sxs-lookup"><span data-stu-id="db644-193">The system was able to support an average of 410 requests per minute throughout the test.</span></span>

![流量进入 AdventureWorks2012 数据库的概览][databasetraffic]

### <a name="gather-detailed-data-access-information"></a><span data-ttu-id="db644-195">收集详细的数据访问信息</span><span class="sxs-lookup"><span data-stu-id="db644-195">Gather detailed data access information</span></span>

<span data-ttu-id="db644-196">对监视数据进行更深入发掘后发现，应用程序执行了三个不同的 SQL SELECT 语句。</span><span class="sxs-lookup"><span data-stu-id="db644-196">Digging deeper into the monitoring data shows the application executes three different SQL SELECT statements.</span></span> <span data-ttu-id="db644-197">这些语句对应于实体框架从 `ProductListPriceHistory`、`Product` 和 `ProductSubcategory` 表中提取数据时生成的请求。</span><span class="sxs-lookup"><span data-stu-id="db644-197">These correspond to the requests generated by Entity Framework to fetch data from the `ProductListPriceHistory`, `Product`, and `ProductSubcategory` tables.</span></span>
<span data-ttu-id="db644-198">此外，从 `ProductListPriceHistory` 表中检索数据的查询是到目前为止最频繁执行的 SELECT 语句，其执行频率高过其他查询一个数量级。</span><span class="sxs-lookup"><span data-stu-id="db644-198">Furthermore, the query that retrieves data from the `ProductListPriceHistory` table is by far the most frequently executed SELECT statement, by an order of magnitude.</span></span>

![受测试示例应用程序执行的查询][queries]

<span data-ttu-id="db644-200">在测试中发现，前面所示的 `GetProductsInSubCategoryAsync` 方法执行了 45 个 SELECT 查询。</span><span class="sxs-lookup"><span data-stu-id="db644-200">It turns out that the `GetProductsInSubCategoryAsync` method, shown earlier, performs 45 SELECT queries.</span></span> <span data-ttu-id="db644-201">每个查询导致应用程序打开新的 SQL 连接。</span><span class="sxs-lookup"><span data-stu-id="db644-201">Each query causes the application to open a new SQL connection.</span></span>

![受测试示例应用程序的查询统计信息][queries2]

> [!NOTE]
> <span data-ttu-id="db644-203">此图显示了负载测试中 `GetProductsInSubCategoryAsync` 操作的最缓慢实例的跟踪信息。</span><span class="sxs-lookup"><span data-stu-id="db644-203">This image shows trace information for the slowest instance of the `GetProductsInSubCategoryAsync` operation in the load test.</span></span> <span data-ttu-id="db644-204">在生产环境中，有用的做法是检查最缓慢的实例，以确定是否有某个方案提示了问题。</span><span class="sxs-lookup"><span data-stu-id="db644-204">In a production environment, it's useful to examine traces of the slowest instances, to see if there is a pattern that suggests a problem.</span></span> <span data-ttu-id="db644-205">如果只需查看平均值，则可以忽略在承受负载的情况下会急剧恶化的问题。</span><span class="sxs-lookup"><span data-stu-id="db644-205">If you just look at the average values, you might overlook problems that will get dramatically worse under load.</span></span>

<span data-ttu-id="db644-206">下图显示了实际发出的 SQL 语句。</span><span class="sxs-lookup"><span data-stu-id="db644-206">The next image shows the actual SQL statements that were issued.</span></span> <span data-ttu-id="db644-207">提取价格信息的查询是针对产品子类别中的每个产品运行的。</span><span class="sxs-lookup"><span data-stu-id="db644-207">The query that fetches price information is run for each individual product in the product subcategory.</span></span> <span data-ttu-id="db644-208">使用联接可大幅减少数据库调用数。</span><span class="sxs-lookup"><span data-stu-id="db644-208">Using a join would considerably reduce the number of database calls.</span></span>

![受测试示例应用程序的查询详细信息][queries3]

<span data-ttu-id="db644-210">如果使用实体框架等 O/RM，跟踪 SQL 查询可以洞察 O/RM 如何将编程调用转换为 SQL 语句，并指明可在其中优化数据访问的区域。</span><span class="sxs-lookup"><span data-stu-id="db644-210">If you are using an O/RM, such as Entity Framework, tracing the SQL queries can provide insight into how the O/RM translates programmatic calls into SQL statements, and indicate areas where data access might be optimized.</span></span> 

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="db644-211">实施解决方案并验证结果</span><span class="sxs-lookup"><span data-stu-id="db644-211">Implement the solution and verify the result</span></span>

<span data-ttu-id="db644-212">重写对实体框架的调用生成了以下结果。</span><span class="sxs-lookup"><span data-stu-id="db644-212">Rewriting the call to Entity Framework produced the following results.</span></span>

![琐碎 I/O 示例应用程序中块式 API 的关键指标负载测试结果][key-indicators-chunky-io]

<span data-ttu-id="db644-214">此负载测试是使用相同的负载配置文件在相同的部署上执行的。</span><span class="sxs-lookup"><span data-stu-id="db644-214">This load test was performed on the same deployment, using the same load profile.</span></span> <span data-ttu-id="db644-215">这一次，图中显示的延迟要低得多。</span><span class="sxs-lookup"><span data-stu-id="db644-215">This time the graph shows much lower latency.</span></span> <span data-ttu-id="db644-216">在加载 1000 个用户的情况下，平均请求时间下降为 5 至 6 秒，而前面的测试中为将近一分钟。</span><span class="sxs-lookup"><span data-stu-id="db644-216">The average request time at 1000 users is between 5 and 6 seconds, down from nearly a minute.</span></span>

<span data-ttu-id="db644-217">这一次，系统可支持每分钟平均 3,970 个请求，而在前面的测试中为 410 个请求。</span><span class="sxs-lookup"><span data-stu-id="db644-217">This time the system supported an average of 3,970 requests per minute, compared to 410 for the earlier test.</span></span>

![块式 API 的事务概览][databasetraffic2]

<span data-ttu-id="db644-219">跟踪 SQL 语句后发现，所有数据是在单个 SELECT 语句中提取的。</span><span class="sxs-lookup"><span data-stu-id="db644-219">Tracing the SQL statement shows that all the data is fetched in a single SELECT statement.</span></span> <span data-ttu-id="db644-220">尽管此查询要复杂得多，但只需为每个操作执行一次。</span><span class="sxs-lookup"><span data-stu-id="db644-220">Although this query is considerably more complex, it is performed only once per operation.</span></span> <span data-ttu-id="db644-221">此外，尽管复杂的联接可能会产生较高的开销，但关系型数据库系统已针对此类查询进行优化。</span><span class="sxs-lookup"><span data-stu-id="db644-221">And while complex joins can become expensive, relational database systems are optimized for this type of query.</span></span>  

![块式 API 的查询详细信息][queries4]

## <a name="related-resources"></a><span data-ttu-id="db644-223">相关资源</span><span class="sxs-lookup"><span data-stu-id="db644-223">Related resources</span></span>

- <span data-ttu-id="db644-224">[API 设计最佳做法][api-design]</span><span class="sxs-lookup"><span data-stu-id="db644-224">[API Design best practices][api-design]</span></span>
- <span data-ttu-id="db644-225">[有关缓存的最佳做法][caching-guidance]</span><span class="sxs-lookup"><span data-stu-id="db644-225">[Caching best practices][caching-guidance]</span></span>
- <span data-ttu-id="db644-226">[数据一致性入门][data-consistency-guidance]</span><span class="sxs-lookup"><span data-stu-id="db644-226">[Data Consistency Primer][data-consistency-guidance]</span></span>
- <span data-ttu-id="db644-227">[超量提取对立模式][extraneous-fetching]</span><span class="sxs-lookup"><span data-stu-id="db644-227">[Extraneous Fetching antipattern][extraneous-fetching]</span></span>
- <span data-ttu-id="db644-228">[无缓存对立模式][no-cache]</span><span class="sxs-lookup"><span data-stu-id="db644-228">[No Caching antipattern][no-cache]</span></span>

[api-design]: ../../best-practices/api-design.md
[caching-guidance]: ../../best-practices/caching.md
[code-sample]:  https://github.com/mspnp/performance-optimization/tree/master/ChattyIO
[data-consistency-guidance]: http://https://msdn.microsoft.com/library/dn589800.aspx
[ef]: /ef/
[extraneous-fetching]: ../extraneous-fetching/index.md
[new-relic]: https://newrelic.com/application-monitoring
[no-cache]: ../no-caching/index.md

[key-indicators-chatty-io]: _images/ChattyIO.jpg
[key-indicators-chunky-io]: _images/ChunkyIO.jpg
[databasetraffic]: _images/DatabaseTraffic.jpg
[databasetraffic2]: _images/DatabaseTraffic2.jpg
[queries]: _images/DatabaseQueries.jpg
[queries2]: _images/DatabaseQueries2.jpg
[queries3]: _images/DatabaseQueries3.jpg
[queries4]: _images/DatabaseQueries4.jpg

