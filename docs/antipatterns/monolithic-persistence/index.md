---
title: 整体持久性反模式
description: 将应用程序的所有数据放入单个数据存储可能会损害性能。
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 7f04b9f0805c281068b6b2edaf040683773e6f6e
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="monolithic-persistence-antipattern"></a>整体持久性反模式

将应用程序的所有数据放入单个数据存储可能会降低性能，原因是这会导致资源争用，或者数据存储不很适合某些数据。

## <a name="problem-description"></a>问题描述

一直以来，不管应用程序需要存储哪些不同类型的数据，往往都只使用单个数据存储。 这样做的原因通常是为了简化应用程序设计，或者受限于开发团队的现有技能组合。 

现代基于云的系统往往附带其他功能性和非功能性要求，需要存储许多异构类型的数据，例如文档、图像、缓存数据、排队消息、应用程序日志和遥测数据。 遵循传统方法将所有这些信息放入同一个数据存储可能会损害性能，主要原因有两个：

- 在同一个数据存储中存储和检索大量不相关的数据可能导致资源争用，从而导致响应时间增加和连接失败。
- 不管选择哪个数据存储，它都不一定最适合所有不同类型的数据，或者未针对应用程序执行的操作进行优化。 

以下示例演示了一个向数据库添加新记录，并将结果记录到日志的 ASP.NET Web API 控制器。 日志保存在业务数据所在的同一个数据库中。 可在[此处][sample-app]找到完整示例。

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

日志记录的生成速率可能会影响业务操作的性能。 如果另一个组件（例如应用程序进程监视器）定期读取和处理日志数据，则它们也可能影响业务操作。

## <a name="how-to-fix-the-problem"></a>如何解决问题

根据数据的用法隔离数据。 对于每个数据集，选择最符合数据集用法的数据存储。 在前面的示例中，应用程序应将日志记录到与保存业务数据的数据库不同的存储： 

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

## <a name="considerations"></a>注意事项

- 根据数据的使用方式及其访问方式隔离数据。 例如，不要将日志信息和业务数据存储在同一个数据存储中。 这些数据类型的要求和访问模式明显不同。 日志记录原生是连续的，而业务数据更有可能需要随机访问，因此通常是关系型的。

- 考虑每种数据类型的数据访问模式。 例如，将带有格式的报告和文档存储在 [Cosmos DB][CosmosDB] 等文档数据库中，但使用 [Azure Redis 缓存][Azure-cache]来缓存临时数据。

- 如果遵循了此指导原则，但仍达到了数据库限制，可能需要纵向扩展数据库。 此外，请考虑横向扩展，并将负载分区到不同的数据库服务器。 但是，分区可能需要重新设计应用程序。 有关详细信息，请参阅[数据分区][DataPartitioningGuidance]。

## <a name="how-to-detect-the-problem"></a>如何检测问题

随着系统耗尽数据库连接等资源，系统的运行速度可能明显下降并最终发生故障。

可执行以下步骤来帮助确定原因。

1. 检测系统以记录关键性能统计信息。 捕获每个操作的计时信息，以及应用程序读取和写入数据的位置。
1. 如果可能，请在生产环境中监视运行了几天的系统，以获得有关系统使用方式的真实视图。 如果无法进行这种监视，请配合实际数量的虚拟用户（这些用户执行一系列典型操作）运行脚本化负载测试。
2. 使用遥测数据来识别性能不佳的时段。
3. 识别在这些时段访问了哪些数据存储。
4. 识别可能发生争用的数据存储资源。

## <a name="example-diagnosis"></a>示例诊断

以下部分将这些步骤应用到前面所述的示例应用程序。

### <a name="instrument-and-monitor-the-system"></a>检测和监视系统

下图显示了对上述示例应用程序执行负载测试的结果。 该测试使用了包含多达 1000 个并发用户的阶跃负载。

![对基于 SQL 的控制器执行的负载测试性能结果][MonolithicScenarioLoadTest]

随着负载提高到 700 个用户，吞吐量随之上升。 但此时，吞吐量保持稳定，系统似乎以最大的容量运行。 平均响应时间随着用户负载的提高而逐渐增加，表明系统无法跟上需求。

### <a name="identify-periods-of-poor-performance"></a>识别性能不佳的时段

监视生产系统时，可能会看到一些模式。 例如，响应时间可能在每天的相同时间明显下降。 这可能是定期工作负荷或计划的批处理作业造成的，或只是因为系统在某些时候包含更多的用户。 应该重点关注这些事件的遥测数据。

找出响应时间增加与数据库活动增加或者对共享资源发出的 I/O 请求增加之间的关联。 如果存在关联，则意味着数据库可能是瓶颈。

### <a name="identify-which-data-stores-are-accessed-during-those-periods"></a>识别在这些时段访问了哪些数据存储

下图显示了负载测试期间数据库吞吐量单位 (DTU) 的利用率。 （DTU 用于度量可用容量，是 CPU 利用率、内存分配和 I/O 速率的组合。）DTU 利用率很快达到 100%。 这大概就是上图中出现吞吐量峰值的位置。 在测试完成之前，数据库利用率一直很高。 在测试快要结束时略微下降，原因可能是实施了限制、数据库连接争用或其他因素。

![Azure 经典门户中的数据库监视器显示数据库的资源利用率][MonolithicDatabaseUtilization]

### <a name="examine-the-telemetry-for-the-data-stores"></a>检查数据存储的遥测数据

检测数据存储以捕获活动的低级详细信息。 在示例应用程序中，数据访问统计信息显示针对 `PurchaseOrderHeader` 表和 `MonoLog` 表执行了大量插入操作。 

![示例应用程序的数据访问统计信息][MonolithicDataAccessStats]

### <a name="identify-resource-contention"></a>识别资源竞争

此时，可以检查源代码，并重点检查应用程序在哪些位置访问了争用的资源。 找到如下所述的情况：

- 将逻辑隔离的数据写入相同的存储。 日志、报告和排队消息等数据不应保存在业务信息所在的同一个数据库中。
- 所选数据存储与数据类型（例如关系型数据库中的大型 Blob 或 XML 文档）之间不匹配。
- 具有明显不同使用模式的数据共享同一个存储，例如将 high-write/low-read 数据与 low-write/high-read 数据存储在一起。

### <a name="implement-the-solution-and-verify-the-result"></a>实施解决方案并验证结果

应用程序已更改为将日志写入独立的数据存储。 下面是负载测试结果：

![使用 Polyglot 控制器的负载测试性能结果][PolyglotScenarioLoadTest]

吞吐量模式类似于前面的图形，但性能峰值位置出现在每秒大约发出 500 个或更多的请求时。 平均响应时间略微下降。 但是，这些统计信息不能反映整体形式。 业务数据库的遥测数据显示，DTU 利用率峰值大约为 75% 而不是 100%。

![Azure 经典门户中的数据库监视器显示 polyglot 方案中数据库的资源利用率][PolyglotDatabaseUtilization]

同样，日志数据库的最大 DTU 利用率只达到了大约 70%。 数据库不再是系统性能的制约因素。

![Azure 经典门户中的数据库监视器显示 polyglot 方案中日志数据库的资源利用率][LogDatabaseUtilization]


## <a name="related-resources"></a>相关资源

- [选择适当的数据存储][data-store-overview]
- [有关选择数据存储的准则][data-store-comparison]
- [高度可缩放解决方案的数据访问：使用 SQL、NoSQL 和 Polyglot 持久性][Data-Access-Guide]
- [数据分区][DataPartitioningGuidance]

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
