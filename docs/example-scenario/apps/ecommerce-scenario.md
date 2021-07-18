---
title: Azure 上的电子商务前端
description: 通过经验证的方案在 Azure 上托管电子商务站点
author: masonch
ms.date: 7/13/18
ms.openlocfilehash: 568821e97c6b90a36429dfa8ec0ef9ed38c7963c
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060968"
---
# <a name="e-commerce-front-end-on-azure"></a>Azure 上的电子商务前端

本示例方案详述了如何使用 Azure 平台即服务 (PaaS) 工具实现一个电子商务前端。 许多电子商务网站在一段时间过后都面临季节性和流量不稳定的问题。 对你的产品或服务的需求增加时（不管是可预见的还是不可预见的），可以通过 PaaS 工具自动应对客户数量和交易数量增加的情况。 另外，本方案只要求你支付所使用的容量，让你充分利用云经济。

本文档介绍各种 Azure PaaS 组件和注意事项，方便你部署示例性的电子商务应用程序 *Relecloud Concerts*，这是一个在线音乐会订票平台。

## <a name="potential-use-cases"></a>可能的用例

以下用例可以考虑本方案：

* 生成一个需要通过弹性缩放来应对不同时间的用户数突增的应用程序。
* 生成一个根据设计可以在全世界的不同 Azure 区域运行并可确保高可用性的应用程序。

## <a name="architecture"></a>体系结构

![适用于电子商务应用程序的示例性方案体系结构][architecture-diagram]

本方案介绍如何通过某个电子商务站点购票。数据流经方案的情形如下所示：

1. Azure 流量管理器会将用户的请求路由到托管在 Azure 应用服务中的电子商务站点。
2. Azure CDN 为用户提供静态图像和内容。
3. 用户通过 Azure Active Directory B2C 租户登录到应用程序。
4. 用户使用 Azure 搜索来搜索音乐会。
5. 网站从 Azure SQL 数据库拉取音乐会详细信息。 
6. 网站引用 Blob 存储中的已购票的图像。
7. 系统将数据库查询结果缓存在 Azure Redis 缓存中，以确保更好的性能。
8. 用户提交订票单和音乐会评价，系统将其放入队列中。
9. Azure Functions 处理订单付款和音乐会评价。
10. 认知服务对音乐会评价进行分析，确定其情绪（正面或负面）。
11. Application Insights 提供用于监视 Web 应用程序运行状况的性能指标。

### <a name="components"></a>组件

* [Azure CDN][docs-cdn] 从靠近用户的位置提供静态缓存内容，以便减轻延迟。
* [Azure 流量管理器][docs-traffic-manager]控制不同 Azure 区域中服务终结点的用户流量分配。
* [应用服务 - Web 应用][docs-webapps]托管的 Web 应用程序可以实现自动缩放和高可用性，不需管理基础结构。
* [Azure Active Directory - B2C][docs-b2c] 是一项标识管理服务，用于自定义和控制客户在应用程序中的注册、登录和管理配置文件的方式。
* [存储队列][docs-storage-queues]存储大量可供应用程序访问的队列消息。
* [Functions][docs-functions] 是无服务器计算选项，可以让应用程序按需运行，不需管理基础结构。
* [认知服务 - 情绪分析][docs-sentiment-analysis]使用机器学习 API，可让开发人员轻松地在其应用程序中添加智能功能 - 例如情绪和视频检测；面部、语音与视觉识别；语音与语言理解。
* [Azure 搜索][docs-search]是一种搜索即服务云解决方案，它可以根据 Web、移动和企业应用程序中的专用异类内容提供丰富的搜索体验。
* [存储 Blob][docs-storage-blobs] 最适合存储大量的非结构化数据，例如文本或二进制数据。
* [Redis 缓存][docs-redis-cache]可以提高严重依赖于后端数据存储的系统的性能和可伸缩性，其方式是将经常访问的数据暂时复制到靠近应用程序的快速存储。
* [SQL 数据库][docs-sql-database]是 Microsoft Azure 中通用的关系数据库托管服务，支持关系数据、JSON、空间和 XML 等结构。
* [Application Insights][docs-application-insights] 旨在持续提高性能与可用性，其方式是通过内置的分析工具了解用户使用某个应用进行的操作，以便自动检测性能异常。

### <a name="alternatives"></a>备选项

可以通过许多其他技术来生成面向客户的应用程序，侧重于大规模的电子商务。 这涉及到应用程序的前端以及数据层。

其他适用于 Web 层和函数的选项包括：

* [Service Fabric][docs-service-fabric] - 一个平台，侧重于生成分布式组件，而此类组件适合在严格控制的群集上部署和运行。 Service Fabric 也可用于托管容器。
* [Azure Kubernetes 服务][docs-kubernetes-service] - 一个平台，用于生成和部署基于容器的解决方案，这些方案可以用作微服务体系结构的某个实现。 这样就可以确保应用程序的不同组件按照需求灵活地进行独立缩放。
* [Azure 容器实例][docs-container-instances] - 这是一种方法，可以快速地部署和运行生命周期短的容器。 通常情况下，部署此处的容器是为了运行快速处理作业，例如处理一条消息或执行一项计算，然后在它们完成后尽快取消预配。
* [服务总线][service-bus]可以用来替代存储队列。

可用于数据层的其他选项包括：

* [Cosmos DB][docs-cosmosdb] - Microsoft 提供的全局分布式多模型数据库。 这样提供的平台适合运行其他数据模型，例如 Mongo DB、Cassandra、Graph 数据或简单的表存储。

## <a name="considerations"></a>注意事项

### <a name="availability"></a>可用性

* 考虑在生成云应用程序时，利用[针对可用性的典型设计模式][design-patterns-availability]。
* 在适当的[应用服务 Web 应用程序参考体系结构][app-service-reference-architecture]中审核可用性注意事项
* 若要针对可用性进行其他的考量，请参阅体系结构中心的[可用性核对清单][availability]。

### <a name="scalability"></a>可伸缩性

* 生成云应用程序时，请弄清楚[针对可伸缩性的典型设计模式][design-patterns-scalability]。
* 在适当的[应用服务 Web 应用程序参考体系结构][app-service-reference-architecture]中审核可伸缩性注意事项
* 若要了解其他可伸缩性主题，请参阅体系结构中心提供的[可伸缩性核对清单][scalability]。

### <a name="security"></a>“安全”

* 考虑在适当情况下利用[针对安全性的典型设计模式][design-patterns-security]。
* 在适当的[应用服务 Web 应用程序参考体系结构][app-service-reference-architecture]中审核安全性注意事项。
* 考虑跟进某个[安全开发生命周期][secure-development]流程，帮助开发人员生成更安全的软件并解决安全符合性要求问题，同时减少开发成本。
* 查看蓝图体系结构，以确定 [Azure PCI DSS 符合性][pci-dss-blueprint]。

### <a name="resiliency"></a>复原

* 考虑在应用程序的某个部分不可用的情况下，利用[断路器模式][circuit-breaker]进行有效的错误处理。
* 审核[针对复原的典型设计模式][design-patterns-resiliency]，考虑在适当情况下实施这些模式。
* 可以在体系结构中心找到许多[建议用于应用服务的复原做法][resiliency-app-service]。
* 考虑对数据层使用活动的[异地复制][sql-geo-replication]，对图像和队列使用[异地冗余][storage-geo-redudancy]存储。
* 有关[复原][resiliency]的更深入讨论，请查看体系结构中心的相关文章。

## <a name="deploy-the-scenario"></a>部署方案

若要部署本方案，可以按照此[分步教程][end-to-end-walkthrough]的说明操作，手动部署每个组件。 本教程还提供了 .NET 示例应用程序，该程序运行简单的购票应用程序。 此外还有一个 ARM 模板，可以自动部署大多数 Azure 资源。

## <a name="pricing"></a>定价

为了方便用户了解运行本方案的成本，我们已在成本计算器中预配置了所有服务。 若要了解自己的特定用例的定价变化情况，请按预期的流量更改相应的变量。

我们已根据你预期接收的流量提供了三个示例性的成本配置文件：

* [小][small-pricing]：表示生成最低生产级别实例所需的组件。 在这里，我们假定用户数量很少，每月只有数千。 此应用使用某个标准 Web 应用的单个实例，这对于启动自动缩放来说已足够。 其他组件都缩放成一个基本层，可以尽量减少成本，但仍可确保有 SLA 支持和足够的容量，可以处理生产级别的工作负荷。
* [中][medium-pricing]：表示组件情况符合中等大小的部署。 在这里，我们估计一个月有大约 100,000 个用户使用此系统。 预计的流量在处于中等标准层的单个应用服务实例中处理。 另外，认知和搜索服务的中型层也会添加到计算器中。
* [大][large-pricing]：表示应用程序需进行大规模的处理，每月的用户数在百万级别，移动的数据量为数 TB。 对于这种级别的使用量，必须将高级层 Web 应用部署在多个区域，并且需前置一个流量管理器。 存储数据的存储空间、数据库和 CDN 均按 TB 级数据量进行配置。

## <a name="related-resources"></a>相关资源

* [多区域 Web 应用程序的参考体系结构][multi-region-web-app]
* [容器中的网上商店参考示例][microservices-ecommerce]

<!-- links -->
[small-pricing]: https://azure.com/e/90fbb6a661a04888a57322985f9b34ac
[medium-pricing]: https://azure.com/e/38d5d387e3234537b6859660db1c9973
[large-pricing]: https://azure.com/e/f07f99b6c3134803a14c9b43fcba3e2f
[app-service-reference-architecture]: /azure/architecture/reference-architectures/app-service-web-app/
[architecture-diagram]: ./media/architecture-diagram-ecommerce-solution.png
[availability]: /azure/architecture/checklist/availability
[circuit-breaker]: /azure/architecture/patterns/circuit-breaker
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[docs-application-insights]: /azure/application-insights/app-insights-overview
[docs-b2c]: /azure/active-directory-b2c/active-directory-b2c-overview
[docs-cdn]: /azure/cdn/cdn-overview
[docs-container-instances]: /azure/container-instances/
[docs-kubernetes-service]: /azure/aks/
[docs-cosmosdb]: /azure/cosmos-db/
[docs-functions]: /azure/azure-functions/functions-overview
[docs-redis-cache]: /azure/redis-cache/cache-overview
[docs-search]: /azure/search/search-what-is-azure-search
[docs-service-fabric]: /azure/service-fabric/
[docs-sentiment-analysis]: /azure/cognitive-services/welcome
[docs-sql-database]: /azure/sql-database/sql-database-technical-overview
[docs-storage-blobs]: /azure/storage/blobs/storage-blobs-introduction
[docs-storage-queues]: /azure/storage/queues/storage-queues-introduction
[docs-traffic-manager]: /azure/traffic-manager/traffic-manager-overview
[docs-webapps]: /azure/app-service/app-service-web-overview
[end-to-end-walkthrough]: https://github.com/Azure/fta-customerfacingapps/tree/master/ecommerce/articles
[microservices-ecommerce]: https://github.com/dotnet-architecture/eShopOnContainers
[multi-region-web-app]: /azure/architecture/reference-architectures/app-service-web-app/multi-region
[pci-dss-blueprint]: /azure/security/blueprints/payment-processing-blueprint
[resiliency-app-service]: /azure/architecture/checklist/resiliency-per-service#app-service
[resiliency]: /azure/architecture/checklist/resiliency
[scalability]: /azure/architecture/checklist/scalability
[secure-development]: https://www.microsoft.com/en-us/SDL/process/design.aspx
[sql-geo-replication]: /azure/sql-database/sql-database-geo-replication-overview
[storage-geo-redudancy]: /azure/storage/common/storage-redundancy-grs
