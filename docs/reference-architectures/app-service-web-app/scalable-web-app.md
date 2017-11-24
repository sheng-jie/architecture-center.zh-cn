---
title: "可缩放的 Web 应用程序"
description: "提高在 Microsoft Azure 中运行的 Web 应用程序的可伸缩性。"
author: MikeWasson
pnp.series.title: Azure App Service
pnp.series.prev: basic-web-app
pnp.series.next: multi-region-web-app
ms.date: 11/23/2016
cardTitle: Improve scalability
ms.openlocfilehash: b875b89b87edd5636d90da8b7f8211f965b39937
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="improve-scalability-in-a-web-application"></a>提高 Web 应用程序的可伸缩性

此参考体系结构显示的经验证做法可以改进 Azure 应用服务 Web 应用程序的可伸缩性和性能。

![[0]][0]

下载此体系结构的 [Visio 文件][visio-download]。

## <a name="architecture"></a>体系结构  

此体系结构基于[基本 Web 应用程序][basic-web-app]中显示的体系结构。 它包括以下组件：

* 资源组。 [资源组][resource-group]是 Azure 资源的逻辑容器。
* **[Web 应用][app-service-web-app]**和 **[API 应用][app-service-api-app]**。 典型的现代应用程序可能包括一个网站以及一个或多个 RESTful Web API。 Web API 可供浏览器客户端通过 AJAX 来使用，也可供本机客户端应用程序或服务器端应用程序使用。 有关设计 Web API 的注意事项，请参阅 [API 设计指南][api-guidance]。    
* **WebJob**。 使用 [Azure WebJobs][webjobs] 在后台运行长时间运行的任务。 WebJobs 可以按计划运行、持续运行或者以响应触发器的方式运行，例如将消息放置到队列中。 WebJob 可在应用服务应用上下文中作为后台进程运行。
* **队列**。 在此处显示的体系结构中，应用程序通过向 [Azure 队列存储][queue-storage]队列放置消息，将后台任务排队。 消息触发 WebJob 中的函数。 也可使用服务总线队列。 如需比较，请参阅 [Azure 队列和服务总线队列 - 比较与对照][queues-compared]。
* **缓存**。 在 [Azure Redis 缓存][azure-redis]中存储半静态数据。  
* **CDN**。 使用 [Azure 内容交付网络][azure-cdn] (CDN) 缓存公开提供的内容，以便降低延迟并加快内容交付速度。
* **数据存储**。 对关系数据使用 [Azure SQL 数据库][sql-db]。 对于非关系数据，可考虑使用 NoSQL 存储，例如 [Cosmos DB][documentdb]。
* **Azure 搜索**。 使用 [Azure 搜索][azure-search]添加搜索功能，例如搜索建议、模糊搜索、特定于语言的搜索。 Azure 搜索通常与其他数据存储结合使用，尤其是在主数据存储对一致性要求严格的情况下。 此方法将权威数据存储在其他数据存储中，将搜索索引存储在 Azure 搜索中。 也可使用 Azure 搜索合并来自多个数据存储的单一搜索索引。  
* **电子邮件/短信**。 使用第三方服务（例如 SendGrid 或 Twilio）发送电子邮件或短信，而不是将此功能直接内置到应用程序中。

## <a name="recommendations"></a>建议

你的要求可能不同于此处描述的体系结构。 一开始可使用此部分的建议。

### <a name="app-service-apps"></a>应用服务应用
建议以独立应用服务应用的形式创建 Web 应用程序和 Web API。 此设计允许你按独立的应用服务计划运行它们，以便对它们进行单独缩放。 如果一开始不需要该级别的可伸缩性，可以先将应用部署到同一计划中，再在以后根据需要将其移至独立的计划中。

> [!NOTE]
> “基本”、“标准”和“高级”计划按计划中的 VM 实例计费，而不是按应用计费。 请参阅[应用服务定价][app-service-pricing]
> 
> 

若要使用应用服务移动应用的“简易表”或“简易 API”功能，请创建适合此用途的独立应用服务应用。  这些功能需要通过特定的应用程序框架来启用。

### <a name="webjobs"></a>Web 作业
考虑在独立的应用服务计划中将资源密集型 WebJobs 部署到空的应用服务应用。 这样可以为 WebJob 提供专用实例。 请参阅[后台作业指南][webjobs-guidance]。  

### <a name="cache"></a>缓存
可以使用 [Azure Redis 缓存][azure-redis]来缓存一些数据，从而改进性能和可伸缩性。 考虑将 Redis 缓存用于：

* 半静态事务数据。
* 会话状态。
* HTML 输出。 这适用于可呈现复杂 HTML 输出的应用程序。

有关缓存策略设计的更详细指南，请参阅[缓存指南][caching-guidance]。

### <a name="cdn"></a>CDN
使用 [Azure CDN][azure-cdn] 来缓存静态内容。 CDN 的主要优势是降低用户的延迟，因为内容缓存在靠近用户的边缘服务器上。 CDN 还可以减轻应用程序的负载，因为相应的流量不是由应用程序处理。

如果应用包含的内容大部分为静态页面，可考虑使用 [CDN 来缓存整个应用][cdn-app-service]。 否则，请将静态内容（例如图像、CSS 和 HTML 文件）置于 [Azure 存储中，并使用 CDN 缓存这些文件][cdn-storage-account]。

> [!NOTE]
> Azure CDN 不能提供需要身份验证的内容。
> 
> 

如需更详细的指南，请参阅[内容交付网络 (CDN) 指南][cdn-guidance]。

### <a name="storage"></a>存储
现代应用程序通常处理大量的数据。 若要进行适合云的缩放，请务必选择适当的存储类型。 以下是一些基线建议。 

| 要存储的内容 | 示例 | 建议的存储 |
| --- | --- | --- |
| 文件 |图像、文档、PDF |Azure Blob 存储 |
| 键值对 |按用户 ID 查找的用户配置文件数据 |Azure 表存储 |
| 旨在触发进一步处理的短消息 |订单请求 |Azure 队列存储、服务总线队列或服务总线主题 |
| 架构灵活但只需要进行基本查询的非关系数据 |产品目录 |文档数据库，例如 Azure Cosmos DB、MongoDB 或 Apache CouchDB |
| 需要更丰富的查询支持、严格的架构和/或高一致性的关系数据 |产品清单 |Azure SQL 数据库 |

## <a name="scalability-considerations"></a>可伸缩性注意事项

Azure 应用服务的主要优势是能够根据负载缩放应用程序。 下面是在计划缩放应用程序时需要考虑的一些注意事项。

### <a name="app-service-app"></a>应用服务应用
如果解决方案包括多个应用服务应用，可考虑将其部署到不同的应用服务计划。 这种方法允许独立缩放应用，因为应用在不同的实例上运行。 

同样，可以考虑将 WebJob 置于其自己的计划中，使后台任务不在处理 HTTP 请求的实例上运行。  

### <a name="sql-database"></a>SQL 数据库
通过数据库分片，增加 SQL 数据库的可伸缩性。 分片是指将数据库水平分区。 可以通过分片，使用[弹性数据库工具][sql-elastic]横向扩展数据库。 分片的潜在好处包括：

- 提高事务吞吐量。
- 对数据子集运行查询可以提高速度。

### <a name="azure-search"></a>Azure 搜索
Azure 搜索没有在主数据存储中执行复杂的数据搜索所需的开销，并可通过缩放来处理负载。 请参阅[在 Azure 搜索中缩放用于查询和索引工作负荷的资源级别][azure-search-scaling]。

## <a name="security-considerations"></a>安全注意事项
此部分列出的安全注意事项特定于本文中描述的 Azure 服务。 它不是安全性最佳做法的完整列表。 至于其他安全注意事项，请参阅[在 Azure 应用服务中保护应用安全][app-service-security]。

### <a name="cross-origin-resource-sharing-cors"></a>跨源资源共享 (CORS)
如果将网站和 Web API 作为独立应用创建，则网站不能向 API 进行客户端 AJAX 调用，除非启用 CORS。

> [!NOTE]
> 浏览器安全性将阻止网页向另一个域发出 AJAX 请求。 这种限制称为同域策略，可阻止恶意站点读取另一个站点中的敏感数据。 CORS 是一项 W3C 标准，可让服务器放宽同域策略，在拒绝某些跨域请求的同时，允许另一些跨域请求。
> 
> 

应用服务内置了对 CORS 的支持，不需编写任何应用程序代码。 请参阅[借助 CORS 从 JavaScript 使用 API 应用][cors]。 请将网站添加到 API 允许域的列表。

### <a name="sql-database-encryption"></a>SQL 数据库加密
如需加密数据库中的静态数据，请使用[透明数据加密][sql-encryption]。 此功能对整个数据库（包括备份和事务日志文件）执行实时加密和解密，不需对应用程序进行更改。 加密会增加一些延迟，因此最好将必须保护的数据单独放置在自己的数据库中，仅对该数据库启用加密。  
  

<!-- links -->

[api-guidance]: ../../best-practices/api-design.md
[app-service-security]: /azure/app-service-web/web-sites-security
[app-service-web-app]: /azure/app-service-web/app-service-web-overview
[app-service-api-app]: /azure/app-service-api/app-service-api-apps-why-best-platform
[app-service-pricing]: https://azure.microsoft.com/pricing/details/app-service/
[azure-cdn]: https://azure.microsoft.com/services/cdn/
[azure-redis]: https://azure.microsoft.com/services/cache/
[azure-search]: https://azure.microsoft.com/documentation/services/search/
[azure-search-scaling]: /azure/search/search-capacity-planning
[background-jobs]: ../../best-practices/background-jobs.md
[basic-web-app]: basic-web-app.md
[basic-web-app-scalability]: basic-web-app.md#scalability-considerations
[caching-guidance]: ../../best-practices/caching.md
[cdn-app-service]: /azure/app-service-web/cdn-websites-with-cdn
[cdn-storage-account]: /azure/cdn/cdn-create-a-storage-account-with-cdn
[cdn-guidance]: ../../best-practices/cdn.md
[cors]: /azure/app-service-api/app-service-api-cors-consume-javascript
[documentdb]: https://azure.microsoft.com/documentation/services/documentdb/
[queue-storage]: /azure/storage/storage-dotnet-how-to-use-queues
[queues-compared]: /azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted
[resource-group]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[sql-elastic]: /azure/sql-database/sql-database-elastic-scale-introduction
[sql-encryption]: https://msdn.microsoft.com/library/dn948096.aspx
[tm]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.azureedge.net/cdn/app-service-reference-architectures.vsdx
[web-app-multi-region]: ./multi-region.md
[webjobs-guidance]: ../../best-practices/background-jobs.md
[webjobs]: /azure/app-service/app-service-webjobs-readme
[0]: ./images/scalable-web-app.png "Azure 中改进了可伸缩性的 Web 应用程序"
