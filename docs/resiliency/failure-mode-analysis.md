---
title: "故障模式分析"
description: "基于 Azure 为云解决方案执行故障模式分析的相关准则。"
author: MikeWasson
ms.date: 03/24/2017
ms.custom: resiliency
pnp.series.title: Design for Resiliency
ms.openlocfilehash: 09d09468eebe5c6fe1c9cdab14e142ff46cf0b25
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="failure-mode-analysis"></a>故障模式分析
[!INCLUDE [header](../_includes/header.md)]

故障模式分析 (FMA) 是一个过程，用于通过标识系统中可能的故障点将复原能力内置到系统中。 FMA 应该是体系结构和设计阶段的一部分，以便从一开始就将故障恢复能力内置到系统中。

下面是执行 FMA 的常规过程：

1. 标识系统中的所有组件。 包括外部依赖项，比如标识提供者、第三方服务等等。   
2. 对于每个组件，标识可能发生的潜在故障。 单个组件可能具有多种故障模式。 例如，应分开考虑读取故障和写入故障，因为其影响和可行的缓解措施有所不同。
3. 根据总体风险对每种故障模式评分。 请考虑以下因素：  

   * 发生故障的几率有多大。 是相对常见？ 还是极其少见？ 无需准确的数字；其目的是帮助进行优先级排名。
   * 对应用程序的可用性、数据丢失、货币成本和业务中断有什么影响？
4. 对于每种故障模式，确定应用程序将如何响应和恢复。 考虑在成本与应用程序复杂程度之间作出折衷。   

作为 FMA 过程的起点，本文包含了潜在故障模式及其缓解措施的目录。 该目录按技术或 Azure 服务分类，另外还有一个针对应用程序级设计的常规目录。 该目录并不详尽，但涵盖许多核心 Azure 服务。

## <a name="app-service"></a>应用服务
### <a name="app-service-app-shuts-down"></a>应用服务应用关闭。
**检测**。 可能的原因：

* 意料中的关闭

  * 操作员关闭该应用程序；例如，使用 Azure 门户。
  * 应用已被卸载，因为它处于闲置状态。 （仅当禁用 `Always On` 设置时。）
* 意料外的关闭

  * 应用崩溃。
  * 应用服务 VM 实例变为不可用。

Application_End 日志记录将捕获应用域关闭（软进程崩溃），它是唯一捕获应用程序域关闭的方法。

**恢复**

* 如果是意料中的关闭，请使用应用程序的关闭事件正常关闭。 例如，在 ASP.NET 中，使用 `Application_End` 方法。
* 如果应用程序在闲置时被卸载，它将在下一次请求时自动重启。 但是，这会产生“冷启动”成本。
* 若要防止应用程序在闲置时被卸载，请启用 Web 应用中的 `Always On` 设置。 请参阅[在 Azure 应用服务中配置 Web 应用][app-service-configure]。
* 若要防止操作员关闭应用，请设置 `ReadOnly` 级别的资源锁。 请参阅[使用 Azure 资源管理器锁定资源][rm-locks]。
* 如果应用崩溃或应用服务 VM 变得不可用，应用服务会自动重启应用。

**诊断**。 应用程序日志和 Web 服务器日志。 请参阅[在 Azure 应用服务中为 Web 应用启用诊断日志记录][app-service-logging]。

### <a name="a-particular-user-repeatedly-makes-bad-requests-or-overloads-the-system"></a>特定用户反复发出错误请求或重载系统。
**检测**。 验证用户身份并将用户 ID 添加到应用程序日志中。

**恢复**

* 使用 [Azure API 管理][api-management]限制该用户的请求。 请参阅[使用 Azure API 管理进行高级请求限制][api-management-throttling]
* 阻止该用户。

**诊断**。 记录所有身份验证请求。

### <a name="a-bad-update-was-deployed"></a>部署了错误的更新。
**检测**。 通过 Azure 门户监视应用程序运行状况（请参阅[监视 Azure Web 应用性能][app-insights-web-apps]）或实施[运行状况终结点监视模式][health-endpoint-monitoring-pattern]。

**恢复**。 使用多个[部署槽][app-service-slots]并回滚到上次已知正常的部署。 有关详细信息，请参阅[基本 Web 应用程序][ra-web-apps-basic]。

## <a name="azure-active-directory"></a>Azure Active Directory
### <a name="openid-connect-oidc-authentication-fails"></a>OpenID Connect (OIDC) 身份验证失败。
**检测**。 可能的故障模式包括：

1. Azure AD 不可用，或由于网络问题而不可访问。 重定向到身份验证终结点失败，OIDC 中间件引发异常。
2. Azure AD 租户不存在。 重定向到身份验证终结点返回 HTTP 错误代码，OIDC 中间件引发异常。
3. 用户无法进行身份验证。 无需实施任何检测策略；Azure AD 会处理登录失败。

**恢复**

1. 捕获中间件引发的未经处理的异常。
2. 处理 `AuthenticationFailed` 事件。
3. 将用户重定向到错误页。
4. 用户重试。

## <a name="azure-search"></a>Azure 搜索
### <a name="writing-data-to-azure-search-fails"></a>将数据写入 Azure 搜索失败。
**检测**。 捕获 `Microsoft.Rest.Azure.CloudException` 错误。

**恢复**

[搜索 .NET SDK][search-sdk] 在发生暂时性故障后会自动重试。 由客户端 SDK 引发的任何异常均应视为非暂时性错误。

默认重试策略使用指数回退。 若要使用其他重试策略，请调用 `SearchIndexClient` 或 `SearchServiceClient` 类上的 `SetRetryPolicy`。 有关详细信息，请参阅[自动重试][auto-rest-client-retry]。

**诊断**。 使用[搜索流量分析][search-analytics]。

### <a name="reading-data-from-azure-search-fails"></a>从 Azure 搜索中读取数据失败。
**检测**。 捕获 `Microsoft.Rest.Azure.CloudException` 错误。

**恢复**

[搜索 .NET SDK][search-sdk] 在发生暂时性故障后会自动重试。 由客户端 SDK 引发的任何异常均应视为非暂时性错误。

默认重试策略使用指数回退。 若要使用其他重试策略，请调用 `SearchIndexClient` 或 `SearchServiceClient` 类上的 `SetRetryPolicy`。 有关详细信息，请参阅[自动重试][auto-rest-client-retry]。

**诊断**。 使用[搜索流量分析][search-analytics]。

## <a name="cassandra"></a>Cassandra
### <a name="reading-or-writing-to-a-node-fails"></a>读取或写入节点失败。
**检测**。 捕获异常。 对于 .NET 客户端，通常为 `System.Web.HttpException`。 其他客户端可能有其他异常类型。  有关详细信息，请参阅 [Cassandra error handling done right](http://www.datastax.com/dev/blog/cassandra-error-handling-done-right)（正确的 Cassandra 错误处理）。

**恢复**

* 每个 [Cassandra 客户端](https://wiki.apache.org/cassandra/ClientOptions)都有自己的重试策略和功能。 有关详细信息，请参阅 [Cassandra error handling done right][cassandra-error-handling]（正确的 Cassandra 错误处理）。
* 使用机架感知部署，将数据节点分布在各个容错域中。
* 部署到具有本地仲裁一致性的多个区域。 如果发生非暂时性故障，则故障转移到另一个区域。

**诊断**。 应用程序日志

## <a name="cloud-service"></a>云服务
### <a name="web-or-worker-roles-are-unexpectedlybeing-shut-down"></a>Web 角色或辅助角色意外关闭。
**检测**。 触发了 [RoleEnvironment.Stopping][RoleEnvironment.Stopping] 事件。

**恢复**。 重写 [RoleEntryPoint.OnStop][RoleEntryPoint.OnStop] 方法以便正常清理。 有关详细信息，请参阅[处理 Azure OnStop 事件的正确方式][onstop-events]（博客）。

## <a name="cosmos-db"></a>Cosmos DB 
### <a name="reading-data-fails"></a>读取数据失败。
**检测**。 捕获 `System.Net.Http.HttpRequestException` 或 `Microsoft.Azure.Documents.DocumentClientException`。

**恢复**

* SDK 自动重试失败的尝试。 若要设置重试次数和最长等待时间，请配置 `ConnectionPolicy.RetryOptions`。 客户端引发的异常会超出重试策略，或不是暂时性错误。
* 如果 Cosmos DB 限制客户端，它会返回 HTTP 429 错误。 请检查 `DocumentClientException` 中的状态代码。 如果一直收到错误 429，请考虑增加集合的吞吐量值。
    * 如果使用的是 MongoDB API，该服务会在进行限制时返回错误代码 16500。
* 跨两个或更多区域复制 Cosmos DB 数据库。 所有副本都是可读的。 使用客户端 SDK 指定 `PreferredLocations` 参数。 这是一个已排序的 Azure 区域列表。 所有读取请求将发送到列表中的第一个可用区域。 如果请求失败，客户端会按顺序尝试发送到列表中的其他区域。 有关详细信息，请参阅[如何使用 DocumentDB API 设置 Azure Cosmos DB 全局分发][docdb-multi-region]。

**诊断**。 记录客户端上的所有错误。

### <a name="writing-data-fails"></a>写入数据失败。
**检测**。 捕获 `System.Net.Http.HttpRequestException` 或 `Microsoft.Azure.Documents.DocumentClientException`。

**恢复**

* SDK 自动重试失败的尝试。 若要设置重试次数和最长等待时间，请配置 `ConnectionPolicy.RetryOptions`。 客户端引发的异常会超出重试策略，或不是暂时性错误。
* 如果 Cosmos DB 限制客户端，它会返回 HTTP 429 错误。 请检查 `DocumentClientException` 中的状态代码。 如果一直收到错误 429，请考虑增加集合的吞吐量值。
* 跨两个或更多区域复制 Cosmos DB 数据库。 如果主要区域发生故障，将提升另一个区域用于写入。 也可以手动触发故障转移。 SDK 执行自动发现和路由，以便应用程序代码在故障转移后继续工作。 在故障转移期间（通常按分钟计），写入操作将有更高的延迟，因为 SDK 会查找新的写入区域。
  有关详细信息，请参阅[如何使用 DocumentDB API 设置 Azure Cosmos DB 全局分发][docdb-multi-region]。
* 作为回退策略，可将文档暂留到备份队列中，稍后再处理该队列。

**诊断**。 记录客户端上的所有错误。

## <a name="elasticsearch"></a>Elasticsearch
### <a name="reading-data-from-elasticsearch-fails"></a>从 Elasticsearch 读取数据失败。
**检测**。 捕获正在使用的特定 [Elasticsearch 客户端][elasticsearch-client]的相应异常。

**恢复**

* 使用重试机制。 每个客户端都有自己的重试策略。
* 部署多个 Elasticsearch 节点并使用复制来实现高可用性。

有关详细信息，请参阅[在 Azure 上运行 Elasticsearch][elasticsearch-azure]。

**诊断**。 可以对 Elasticsearch 使用监视工具，或记录具有有效负载的客户端上的所有错误。 请参阅[在 Azure 上运行 Elasticsearch][elasticsearch-azure] 中的“监视”部分。

### <a name="writing-data-to-elasticsearch-fails"></a>将数据写入 Elasticsearch 失败。
**检测**。 捕获正在使用的特定 [Elasticsearch 客户端][elasticsearch-client]的相应异常。  

**恢复**

* 使用重试机制。 每个客户端都有自己的重试策略。
* 如果应用程序可承受降低的一致性级别，则考虑在写入时将 `write_consistency` 设置为 `quorum`。

有关详细信息，请参阅[在 Azure 上运行 Elasticsearch][elasticsearch-azure]。

**诊断**。 可以对 Elasticsearch 使用监视工具，或记录具有有效负载的客户端上的所有错误。 请参阅[在 Azure 上运行 Elasticsearch][elasticsearch-azure] 中的“监视”部分。

## <a name="queue-storage"></a>队列存储
### <a name="writing-a-message-to-azure-queue-storage-fails-consistently"></a>将消息写入 Azure 队列存储一直失败。
**检测**。 在尝试 *N* 次重试后，写入操作仍然失败。

**恢复**

* 将数据存储在本地缓存中，等服务变为可用后，再将写入数据转发到存储。
* 如果主队列不可用，则创建辅助队列并写入该队列。

**诊断**。 使用[存储度量值][storage-metrics]。

### <a name="the-application-cannot-process-a-particular-message-from-the-queue"></a>应用程序无法处理来自队列的某条特定消息。
**检测**。 特定于应用程序。 例如，该消息包含无效数据，或业务逻辑因某些原因而失败。

**恢复**

将该消息移到单独的队列中。 运行单独的进程来检查该队列中的消息。

考虑使用 Azure 服务总线消息传送队列，它会为此提供[死信队列][sb-dead-letter-queue]功能。

> [!NOTE]
> 如果通过 WebJobs 使用存储队列，WebJobs SDK 会提供内置的有害消息处理功能。 请参阅[如何通过 WebJobs SDK 使用 Azure 队列存储][sb-poison-message]。

**诊断**。 使用应用程序日志记录。

## <a name="redis-cache"></a>Redis 缓存
### <a name="reading-from-the-cache-fails"></a>从缓存读取失败。
**检测**。 捕获 `StackExchange.Redis.RedisConnectionException`。

**恢复**

1. 对暂时性故障进行重试。 Azure Redis 缓存支持内置的重试，请参阅 [Redis 缓存重试准则][redis-retry]。
2. 将非暂时性故障视为缓存失误，并回退到原始数据源。

**诊断**。 使用 [Redis 缓存诊断][redis-monitor]。

### <a name="writing-to-the-cache-fails"></a>写入缓存失败。
**检测**。 捕获 `StackExchange.Redis.RedisConnectionException`。

**恢复**

1. 对暂时性故障进行重试。 Azure Redis 缓存支持内置的重试，请参阅 [Redis 缓存重试准则][redis-retry]。
2. 如果是非暂时性错误，则忽略它，稍后让其他事务写入缓存。

**诊断**。 使用 [Redis 缓存诊断][redis-monitor]。

## <a name="sql-database"></a>SQL 数据库
### <a name="cannot-connect-to-the-database-in-the-primary-region"></a>无法连接到主要区域中的数据库。
**检测**。 连接失败。

**恢复**

先决条件：必须为数据库配置活动异地复制。 请参阅 [SQL 数据库活动异地复制][sql-db-replication]。

* 对于查询，请从次要副本中读取。
* 对于插入和更新，请手动故障转移到次要副本。 请参阅[为 Azure SQL 数据库启动计划内或计划外故障转移][sql-db-failover]。

该副本使用不同的连接字符串，因此需要更新应用程序中的连接字符串。

### <a name="client-runs-out-of-connections-in-the-connection-pool"></a>客户端用尽了连接池中的连接。
**检测**。 捕获 `System.InvalidOperationException` 错误。

**恢复**

* 请重试操作即可。
* 作为缓解计划，可针对每个用例将连接池隔离开来，使一个用例无法支配所有连接。
* 增加连接池数量上限。

**诊断**。 应用程序日志。

### <a name="database-connection-limit-is-reached"></a>达到数据库连接限制。
**检测**。 Azure SQL 数据库限制并发辅助角色、登录名和会话的数目。 具体限制视服务层而定。 有关详细信息，请参阅 [Azure SQL 数据库资源限制][sql-db-limits]。

若要检测这些错误，请捕获 `System.Data.SqlClient.SqlException` 并检查 SQL 错误代码中 `SqlException.Number` 的值。 有关相关错误代码的列表，请参阅 [SQL 数据库客户端应用程序的 SQL 错误代码：数据库连接错误和其他问题][sql-db-errors]。

**恢复**。 这些错误被视为暂时性错误，因此重试可能解决此问题。 如果持续遇到这些错误，请考虑缩放数据库。

**诊断**。 - [sys.event_log][sys.event_log] 查询返回成功的数据库连接、失败的连接和死锁。

* 为失败的连接创建[警报规则][azure-alerts]。
* 启用 [SQL 数据库审核][sql-db-audit]并检查失败的登录。

## <a name="service-bus-messaging"></a>服务总线消息传送
### <a name="reading-a-message-from-a-service-bus-queue-fails"></a>从服务总线队列读取消息失败。
**检测**。 捕获客户端 SDK 中的异常。 服务总线异常的基类是 [MessagingException][sb-messagingexception-class]。 如果是暂时性错误，则 `IsTransient` 属性为 true。

有关详细信息，请参阅[服务总线消息传送异常][sb-messaging-exceptions]。

**恢复**

1. 对暂时性故障进行重试。 请参阅[服务总线重试准则][sb-retry]。
2. 无法传递给任何接收方的消息位于*死信队列*中。 可使用此队列查看无法接收哪些消息。 死信队列不自动执行清理。 在显式检索这些消息前，它们会一直留在其中。 请参阅[服务总线死信队列概述][sb-dead-letter-queue]。

### <a name="writing-a-message-to-a-service-bus-queue-fails"></a>向服务总线队列写入消息失败。
**检测**。 捕获客户端 SDK 中的异常。 服务总线异常的基类是 [MessagingException][sb-messagingexception-class]。 如果是暂时性错误，则 `IsTransient` 属性为 true。

有关详细信息，请参阅[服务总线消息传送异常][sb-messaging-exceptions]。

**恢复**

1. 服务总线客户端在发生暂时性错误后会自动重试。 默认情况下，它使用指数回退。 在达到最大重试计数或最长超时期限后，客户端会引发异常。 有关详细信息，请参阅[服务总线重试准则][sb-retry]。
2. 如果超过队列配额，客户端会引发 [QuotaExceededException][QuotaExceededException]。 该异常消息会提供更多详细信息。 重试前清空队列中的某些消息，并考虑使用断路器模式，以免在超过配额时继续重试。 另外，确保 [BrokeredMessage.TimeToLive] 属性不要设置得过高。
3. 在一个区域内，可以使用[分区的队列和主题][sb-partition]提高复原能力。 将未分区的队列或主题分配到一个消息存储空间。 如果此消息存储空间不可用，则针对该队列或主题的所有操作将都失败。 将分区的队列或主题跨多个消息传送存储进行分区。
4. 若要提高复原能力，可在不同的区域中创建两个服务总线命名空间，并复制消息。 可使用主动复制或被动复制。

   * 主动复制：客户端将每条消息同时发送到两个队列。 接收方对两个队列都进行侦听。 使用唯一标识符标记消息，以便客户端放弃重复的消息。
   * 被动复制：客户端将消息发送到一个队列。 如果出错，客户端回退到另一个队列。 接收方对两个队列都进行侦听。 此方法可以减少发送的重复消息数。 但是，接收方仍须处理重复消息。

     有关详细信息，请参阅[异地复制示例][sb-georeplication-sample]和[使应用程序免受服务总线中断和灾难影响的最佳实践](/azure/service-bus-messaging/service-bus-outages-disasters/)。

### <a name="duplicate-message"></a>消息重复。
**检测**。 检查消息的 `MessageId` 和 `DeliveryCount` 属性。

**恢复**

* 如果可能，将消息处理操作设计为幂等。 否则，存储已处理消息的消息 ID，并在处理消息之前检查 ID。
* 通过创建 `RequiresDuplicateDetection` 设置为 true 的队列，启用重复检测。 使用此设置时，服务总线会自动删除发送的与以前消息 `MessageId` 相同的所有消息。  注意以下事项：

  * 此设置可防止将重复的消息放入队列中， 但不能防止接收方多次处理同一消息。
  * 重复检测具有时间范围。 如果超过此范围发送重复项，则检测不到该重复项。

**诊断**。 记录重复消息。

### <a name="the-application-cannot-process-a-particular-message-from-the-queue"></a>应用程序无法处理来自队列的某条特定消息。
**检测**。 特定于应用程序。 例如，该消息包含无效数据，或业务逻辑因某些原因而失败。

**恢复**

考虑以下两种故障模式。

* 接收方检测到故障。 在此情况下，将该消息移至死信队列。 然后，运行单独的进程来检查死信队列中的消息。
* 接收方在处理该消息时失败 &mdash; 例如，由于未经处理的异常。 若要处理这种情况，可使用 `PeekLock` 模式。 在此模式下，如果锁定到期，该消息会变为可供其他接收方处理。 如果该消息超过最大传递计数或生存时间，则会自动移至死信队列。

有关详细信息，请参阅[服务总线死信队列概述][sb-dead-letter-queue]。

**诊断**。 应用程序每次将消息移至死信队列时，都会向应用程序日志写入一个事件。

## <a name="service-fabric"></a>Service Fabric
### <a name="a-request-to-a-service-fails"></a>服务请求失败。
**检测**。 服务返回一个错误。

**恢复**

* 重新找一个代理（`ServiceProxy` 或 `ActorProxy`），并重新调用该服务/参与者方法。
* **有状态服务**。 将针对可靠集合的操作包装在一个事务中。 如果出错，将回滚该事务。 将重新处理该请求（如果已从队列中拉取）。
* **无状态服务**。 如果服务将数据暂留在外部存储中，则所有操作必须幂等。

**诊断**。 应用程序日志

### <a name="service-fabric-node-is-shut-down"></a>Service Fabric 节点关闭。
**检测**。 向服务的 `RunAsync` 方法传递一个取消标记。 Service Fabric 会在关闭节点前取消任务。

**恢复**。 使用该取消标记来检测关闭。 当 Service Fabric 请求取消时，请完成所有工作并尽快退出  `RunAsync`。

**诊断**。 应用程序日志

## <a name="storage"></a>存储
### <a name="writing-data-to-azure-storage-fails"></a>向 Azure 存储写入数据失败
**检测**。 写入时，客户端收到错误。

**恢复**

1. 重试该操作，以从暂时性故障中恢复。 客户端 SDK 中的[重试策略][Storage.RetryPolicies]会自动处理此任务。
2. 实施断路器模式，以免存储瘫痪。
3. 如果尝试 N 次重试均失败，则执行正常回退。 例如：

   * 将数据存储在本地缓存中，等服务变为可用后，再将写入数据转发到存储。
   * 如果写入操作在事务范围内，则补偿事务。

**诊断**。 使用[存储度量值][storage-metrics]。

### <a name="reading-data-from-azure-storage-fails"></a>从 Azure 存储读取数据失败。
**检测**。 读取时，客户端收到错误。

**恢复**

1. 重试该操作，以从暂时性故障中恢复。 客户端 SDK 中的[重试策略][Storage.RetryPolicies]会自动处理此任务。
2. 对于 RA-GRS 存储，如果从主终结点读取失败，则尝试从辅助终结点读取。 客户端 SDK 可以自动处理此任务。 请参阅 [Azure 存储复制][storage-replication]。
3. 如果尝试 *N* 次重试均失败，则执行回退操作，以便正常降级。 例如，如果从存储中检索不到产品图像，则显示一般的占位符图像。

**诊断**。 使用[存储度量值][storage-metrics]。

## <a name="virtual-machine"></a>虚拟机
### <a name="connection-to-a-backend-vm-fails"></a>连接到后端 VM 失败。
**检测**。 网络连接错误。

**恢复**

* 将至少两个后端 VM 部署在可用性集中，并放在负载均衡器后面。
* 如果连接错误是暂时性的，TCP 有时会成功重试发送消息。
* 在应用程序中实施重试策略。
* 对于持久性或非暂时性错误，实施[断路器][circuit-breaker]模式。
* 如果调用 VM 超过其网络出口限制，出站队列将填满。 如果出站队列一直很满，请考虑横向扩展。

**诊断**。 记录服务边界的事件。

### <a name="vm-instance-becomes-unavailable-or-unhealthy"></a>VM 实例变得不可用或不正常。
**检测**。 配置负载均衡器[运行状况探测][lb-probe]，以便发信号指示 VM 实例是否正常。 探测应检查关键功能能否正确响应。

**恢复**。 对于每个应用程序层，将多个 VM 实例放入同一可用性集中，并在 VM 前面放一个负载均衡器。 如果运行状况探测失败，负载均衡器会停止向不正常的实例发送新连接。

**诊断**。 - 使用负载均衡器 [Log Analytics][lb-monitor]。

* 配置监视系统，以监视所有运行状况监视终结点。

### <a name="operator-accidentally-shuts-down-a-vm"></a>操作员意外关闭 VM。
**检测**。 不适用

**恢复**。 设置 `ReadOnly` 级别的资源锁。 请参阅[使用 Azure 资源管理器锁定资源][rm-locks]。

**诊断**。 使用 [Azure 活动日志][azure-activity-logs]。

## <a name="webjobs"></a>Web 作业
### <a name="continuous-job-stops-running-when-the-scm-host-is-idle"></a>SCM 主机闲置时，连续作业停止运行。
**检测**。 向 WebJob 函数传递一个取消标记。 有关详细信息，请参阅[正常关闭][web-jobs-shutdown]。

**恢复**。 在 Web 应用中启用 `Always On` 设置。 有关详细信息，请参阅[使用 WebJobs 运行后台任务][web-jobs]。

## <a name="application-design"></a>应用程序设计
### <a name="application-cant-handle-a-spike-in-incoming-requests"></a>应用程序无法处理陡增的传入请求。
**检测**。 取决于应用程序。 典型症状：

* 网站开始返回 HTTP 5xx 错误代码。
* 从属服务（比如数据库或存储）开始限制请求。 请查找 HTTP 错误，比如 HTTP 429（请求过多），具体视服务而定。
* HTTP 队列长度变长。

**恢复**

* 横向扩展以处理增加的负载。
* 缓解故障影响，避免让级联故障中断整个应用程序。 缓解策略包括：

  * 实施[限制模式][throttling-pattern]，以免后端系统瘫痪。
  * 使用[基于队列的负载调节][queue-based-load-leveling]缓冲请求，并以适当的节奏处理它们。
  * 优先某些客户端。 例如，如果应用程序具有免费层和付费层，则限制免费层的客户，而不是付费客户。 请参阅[优先级队列模式][priority-queue-pattern]。

**诊断**。 使用[应用服务诊断日志记录][app-service-logging]。 使用 [Azure Log Analytics][azure-log-analytics]、[Application Insights][app-insights] 或 [New Relic][new-relic] 等服务帮助了解诊断日志。

### <a name="one-of-the-operations-in-a-workflow-or-distributed-transaction-fails"></a>工作流或分布式事务中的某个操作失败。
**检测**。 尝试 *N* 次重试后仍然失败。

**恢复**

* 作为缓解计划，可实施[计划程序代理监督程序][scheduler-agent-supervisor]模式来管理整个工作流。
* 对于超时错误，不要重试。 此错误的成功率较低。
* 将工作排队，以便稍后重试。

**诊断**。 记录所有操作（成功的和失败的），包括补偿操作。 使用关联 ID，以便跟踪同一事务内的所有操作。

### <a name="a-call-to-a-remote-service-fails"></a>对远程服务的调用失败。
**检测**。 HTTP 错误代码。

**恢复**

1. 对暂时性故障进行重试。
2. 如果尝试 *N* 次后，调用仍失败，则执行回退操作。 （特定于应用程序。）
3. 实施[断路器模式][circuit-breaker]以免出现级联故障。

**诊断**。 记录所有远程调用失败。

## <a name="next-steps"></a>后续步骤
有关 FMA 过程的详细信息，请参阅[专为云服务设计的复原能力][resilience-by-design-pdf]（PDF 下载）。

<!-- links -->

[api-management]: https://azure.microsoft.com/documentation/services/api-management/
[api-management-throttling]: /azure/api-management/api-management-sample-flexible-throttling/
[app-insights]: /azure/application-insights/app-insights-overview/
[app-insights-web-apps]: /azure/application-insights/app-insights-azure-web-apps/
[app-service-configure]: /azure/app-service-web/web-sites-configure/
[app-service-logging]: /azure/app-service-web/web-sites-enable-diagnostic-log/
[app-service-slots]: /azure/app-service-web/web-sites-staged-publishing/
[auto-rest-client-retry]: https://github.com/Azure/autorest/tree/master/docs
[azure-activity-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-activity-logs/
[azure-alerts]: /azure/monitoring-and-diagnostics/insights-alerts-portal/
[azure-log-analytics]: /azure/log-analytics/log-analytics-overview/
[BrokeredMessage.TimeToLive]: https://msdn.microsoft.com/library/microsoft.servicebus.messaging.brokeredmessage.timetolive.aspx
[cassandra-error-handling]: http://www.datastax.com/dev/blog/cassandra-error-handling-done-right
[circuit-breaker]: https://msdn.microsoft.com/library/dn589784.aspx
[docdb-multi-region]: /azure/documentdb/documentdb-developing-with-multiple-regions/
[elasticsearch-azure]: ../elasticsearch/index.md
[elasticsearch-client]: https://www.elastic.co/guide/en/elasticsearch/client/index.html
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[onstop-events]: https://azure.microsoft.com/blog/the-right-way-to-handle-azure-onstop-events/
[lb-monitor]: /azure/load-balancer/load-balancer-monitor-log/
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview/#learn-about-the-types-of-probes
[new-relic]: https://newrelic.com/
[priority-queue-pattern]: https://msdn.microsoft.com/library/dn589794.aspx
[queue-based-load-leveling]: https://msdn.microsoft.com/library/dn589783.aspx
[QuotaExceededException]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.quotaexceededexception.aspx
[ra-web-apps-basic]: ../reference-architectures/app-service-web-app/basic-web-app.md
[redis-monitor]: /azure/redis-cache/cache-how-to-monitor/
[redis-retry]: ../best-practices/retry-service-specific.md#azure-redis-cache-retry-guidelines
[resilience-by-design-pdf]: http://download.microsoft.com/download/D/8/C/D8C599A4-4E8A-49BF-80EE-FE35F49B914D/Resilience_by_Design_for_Cloud_Services_White_Paper.pdf
[RoleEntryPoint.OnStop]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.onstop.aspx
[RoleEnvironment.Stopping]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.stopping.aspx
[rm-locks]: /azure/azure-resource-manager/resource-group-lock-resources/
[sb-dead-letter-queue]: /azure/service-bus-messaging/service-bus-dead-letter-queues/
[sb-georeplication-sample]: https://github.com/Azure-Samples/azure-servicebus-messaging-samples/tree/master/GeoReplication
[sb-messagingexception-class]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.messagingexception.aspx
[sb-messaging-exceptions]: /azure/service-bus-messaging/service-bus-messaging-exceptions/
[sb-outages]: /azure/service-bus-messaging/service-bus-outages-disasters/#protecting-queues-and-topics-against-datacenter-outages-or-disasters
[sb-partition]: /azure/service-bus-messaging/service-bus-partitioning/
[sb-poison-message]: /azure/app-service-web/websites-dotnet-webjobs-sdk-storage-queues-how-to/#poison
[sb-retry]: ../best-practices/retry-service-specific.md#service-bus-retry-guidelines
[search-sdk]: https://msdn.microsoft.com/library/dn951165.aspx
[scheduler-agent-supervisor]: https://msdn.microsoft.com/library/dn589780.aspx
[search-analytics]: /azure/search/search-traffic-analytics/
[sql-db-audit]: /azure/sql-database/sql-database-auditing-get-started/
[sql-db-errors]: /azure/sql-database/sql-database-develop-error-messages/#resource-governance-errors
[sql-db-failover]: /azure/sql-database/sql-database-geo-replication-failover-portal/
[sql-db-limits]: /azure/sql-database/sql-database-resource-limits/
[sql-db-replication]: /azure/sql-database/sql-database-geo-replication-overview/
[storage-metrics]: https://msdn.microsoft.com/library/dn782843.aspx
[storage-replication]: /azure/storage/storage-redundancy/
[Storage.RetryPolicies]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.aspx
[sys.event_log]: https://msdn.microsoft.com/library/dn270018.aspx
[throttling-pattern]: https://msdn.microsoft.com/library/dn589798.aspx
[web-jobs]: /azure/app-service-web/web-sites-create-web-jobs/
[web-jobs-shutdown]: /azure/app-service-web/websites-dotnet-webjobs-sdk-storage-queues-how-to/#graceful
