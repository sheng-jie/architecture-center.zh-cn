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
# <a name="chatty-io-antipattern"></a>琐碎 I/O 对立模式

大量 I/O 请求的累积效应可能会对性能和响应能力产生明显的不利影响。

## <a name="problem-description"></a>问题描述

网络调用和其他 I/O 操作的速度与生俱来就比计算任务要慢。 每个 I/O 请求通常会产生很高的开销，大量 I/O 操作的累积效应可能会使系统变慢。 下面是出现琐碎 I/O 的一些常见原因。

### <a name="reading-and-writing-individual-records-to-a-database-as-distinct-requests"></a>以不同请求的形式在数据库中读取和写入单个记录

以下示例从产品数据库中读取数据。 有三个表：`Product`、`ProductSubcategory` 和 `ProductPriceListHistory`。 代码通过执行一系列查询检索子类别中的所有产品以及价格信息：  

1. 查询 `ProductSubcategory` 表中的子类别。
2. 通过查询 `Product` 表查找该子类别中的所有产品。
3. 对于每个产品，查询 `ProductPriceListHistory` 表中的价格数据。

应用程序使用[实体框架][ef]查询数据库。 可在[此处][code-sample]找到完整示例。 

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

此示例显式显示了问题，但有时，如果 O/RM 逐个地隐式提取子记录，则可能会掩盖问题。 这就是所谓的“N+1 问题”。 

### <a name="implementing-a-single-logical-operation-as-a-series-of-http-requests"></a>以一系列 HTTP 请求的形式执行单个逻辑操作

当开发人员尝试遵循面向对象的范例，并将远程对象视为内存中的本地对象时，就往往会发生这种情况。 这可能导致过多的网络往返。 例如，以下 Web API 通过单个 HTTP GET 方法公开 `User` 对象的单个属性。 

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

尽管此方法在技术上没有任何问题，但是，大多数客户端可能需要获取每个 `User` 的多个属性，从而导致需要编写如下所示的客户端代码。 

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

### <a name="reading-and-writing-to-a-file-on-disk"></a>读取和写入磁盘中的文件

文件 I/O 涉及到打开某个文件并转到相应的点，然后读取或写入数据。 完成该操作后，文件可能会关闭，以节省操作系统资源。 持续在文件中读取和写入少量信息的应用程序会产生很高的 I/O 开销。 小规模写入请求还可能导致文件碎片，从而进一步减慢后续 I/O 操作的速度。 

以下示例使用 `FileStream` 将 `Customer` 对象写入文件。 创建 `FileStream` 会打开该文件，释放它会关闭该文件。 （`using` 语句自动释放 `FileStream` 对象。）如果由于添加新客户，应用程序反复调用此方法，I/O 开销可能会迅速累积。

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

## <a name="how-to-fix-the-problem"></a>如何解决问题

通过将数据打包成更大但更少的请求来减少 I/O 请求的数量。

以单个查询而不是多个较小查询的形式从数据库中提取数据。 下面是检索产品信息的代码的修改版本。

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

遵循 Web API 的 REST 设计原则。 下面是前一示例中 Web API 的修改版本。 不要针对每个属性单独使用 GET 方法，而可以使用单个返回 `User` 的 GET 方法。 这会导致每个请求的响应正文变得更大，但每个客户端可能会发出更少的 API 调用。

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

对于文件 I/O，请考虑在内存中缓冲数据，然后以单个操作的形式将缓冲的数据写入文件。 此方法可以减少由于反复打开和关闭文件产生的开销，并有助于减少磁盘中文件的碎片。

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

## <a name="considerations"></a>注意事项

- 前两个示例发出更少的 I/O 调用，但每个示例检索了更多的信息。 必须考虑这两种因素的利弊。 正确的答案取决于实际使用模式。 例如，在 Web API 示例中，客户端可能往往只需检索用户名。 在这种情况下，将该操作公开为单独的 API 调用可能有利。 有关详细信息，请参阅[超量提取][extraneous-fetching]对立模式。

- 读取数据时，请不要发出过大的 I/O 请求。 应用程序应该只检索它可能要使用的信息。 

- 有时，将对象的信息分区成以下两个区块可能会有帮助：经常访问的数据（大多数请求就是针对这些数据发出的），不经常访问的数据（极少使用的数据）。 最常访问的数据往往是对象总体数据中的相对较小一部分，因此，只返回这一部分数据能够大幅节省 I/O 开销。

- 写入数据时，请避免将资源锁定超过必要的时间，以减少在执行冗长操作期间发生资源争用的可能性。 如果写入操作跨多个数据存储、文件或服务，则采用最终一致性方法。 请参阅[数据一致性指南][data-consistency-guidance]。

- 如果在写入数据之前在内存中缓冲数据，则发生进程崩溃时，数据易受攻击。 如果数据率通常出现喷发或相对稀疏，在[事件中心](http://azure.microsoft.com/services/event-hubs/)等外部持久队列中缓冲数据可能会更安全。

- 请考虑缓存从服务或数据库检索的数据。 这可以避免针对相同的数据发出重复请求，从而帮助减少 I/O 数量。 有关详细信息，请参阅[有关缓存的最佳做法][caching-guidance]。

## <a name="how-to-detect-the-problem"></a>如何检测问题

琐碎 I/O 的症状包括高延迟和低吞吐量。 由于 I/O 资源争用加剧，最终用户可能会反映响应时间延长，或服务超时导致失败。

可执行以下步骤来帮助确定任何问题的原因：

1. 对生产系统执行进程监视，识别响应时间不佳的操作。
2. 对上一步骤中识别到的每个操作执行负载测试。
3. 在负载测试期间，收集有关每个操作发出的数据访问请求的遥测数据。
4. 收集已发送到数据存储的每个请求的详细统计信息。
5. 在测试环境中分析应用程序，判定可能出现 I/O 瓶颈的位置。 

确定是否存在以下任何症状：

- 向同一个文件发出大量的小型 I/O 请求。
- 某个应用程序实例向同一个服务发出大量的小型网络请求。
- 某个应用程序实例向同一个数据存储发出大量的小型请求。
- 应用程序和服务受 I/O 约束。

## <a name="example-diagnosis"></a>示例诊断

以下部分将这些步骤应用到前面所示的查询数据库的示例。

### <a name="load-test-the-application"></a>对应用程序进行负载测试

此图显示了负载测试的结果。 中间响应时间是根据每个请求在数十秒内的表现测得的。 该图显示延迟很高。 每加载 1000 个用户，用户就可能需要等待将近一分钟才能看到查询结果。 

![琐碎 I/O 示例应用程序的关键指标负载测试结果][key-indicators-chatty-io]

> [!NOTE]
> 该应用程序是使用 Azure SQL 数据库作为 Azure 应用服务 Web 应用部署的。 负载测试使用了包含多达 1000 个并发用户的模拟步骤工作负荷。 数据库中配置了支持最多 1000 个并发连接的连接池，以减少出现连接争用，从而影响结果的可能性。 

### <a name="monitor-the-application"></a>监视应用程序

可以使用应用程序性能监视 (APM) 包来捕获和分析可识别琐碎 I/O 的关键指标。 至于哪些指标比较重要，将取决于 I/O 工作负荷。 对于此示例，要关注的 I/O 请求是数据库查询。 

下图显示了使用 [New Relic APM][new-relic] 生成的结果。 在承受最大工作负荷期间，平均数据库响应时间的峰值出现在每个请求的大约 5.6 秒处。 在整个测试过程中，系统能够支持每分钟平均 410 个请求。

![流量进入 AdventureWorks2012 数据库的概览][databasetraffic]

### <a name="gather-detailed-data-access-information"></a>收集详细的数据访问信息

对监视数据进行更深入发掘后发现，应用程序执行了三个不同的 SQL SELECT 语句。 这些语句对应于实体框架从 `ProductListPriceHistory`、`Product` 和 `ProductSubcategory` 表中提取数据时生成的请求。
此外，从 `ProductListPriceHistory` 表中检索数据的查询是到目前为止最频繁执行的 SELECT 语句，其执行频率高过其他查询一个数量级。

![受测试示例应用程序执行的查询][queries]

在测试中发现，前面所示的 `GetProductsInSubCategoryAsync` 方法执行了 45 个 SELECT 查询。 每个查询导致应用程序打开新的 SQL 连接。

![受测试示例应用程序的查询统计信息][queries2]

> [!NOTE]
> 此图显示了负载测试中 `GetProductsInSubCategoryAsync` 操作的最缓慢实例的跟踪信息。 在生产环境中，有用的做法是检查最缓慢的实例，以确定是否有某个方案提示了问题。 如果只需查看平均值，则可以忽略在承受负载的情况下会急剧恶化的问题。

下图显示了实际发出的 SQL 语句。 提取价格信息的查询是针对产品子类别中的每个产品运行的。 使用联接可大幅减少数据库调用数。

![受测试示例应用程序的查询详细信息][queries3]

如果使用实体框架等 O/RM，跟踪 SQL 查询可以洞察 O/RM 如何将编程调用转换为 SQL 语句，并指明可在其中优化数据访问的区域。 

### <a name="implement-the-solution-and-verify-the-result"></a>实施解决方案并验证结果

重写对实体框架的调用生成了以下结果。

![琐碎 I/O 示例应用程序中块式 API 的关键指标负载测试结果][key-indicators-chunky-io]

此负载测试是使用相同的负载配置文件在相同的部署上执行的。 这一次，图中显示的延迟要低得多。 在加载 1000 个用户的情况下，平均请求时间下降为 5 至 6 秒，而前面的测试中为将近一分钟。

这一次，系统可支持每分钟平均 3,970 个请求，而在前面的测试中为 410 个请求。

![块式 API 的事务概览][databasetraffic2]

跟踪 SQL 语句后发现，所有数据是在单个 SELECT 语句中提取的。 尽管此查询要复杂得多，但只需为每个操作执行一次。 此外，尽管复杂的联接可能会产生较高的开销，但关系型数据库系统已针对此类查询进行优化。  

![块式 API 的查询详细信息][queries4]

## <a name="related-resources"></a>相关资源

- [API 设计最佳做法][api-design]
- [有关缓存的最佳做法][caching-guidance]
- [数据一致性入门][data-consistency-guidance]
- [超量提取对立模式][extraneous-fetching]
- [无缓存对立模式][no-cache]

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

