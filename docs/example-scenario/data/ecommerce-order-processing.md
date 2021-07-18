---
title: Azure 上的可缩放订单处理
description: 通过示例方案介绍如何使用 Azure Cosmos DB 生成高度可缩放的订单处理管道。
author: alexbuckgit
ms.date: 07/10/2018
ms.openlocfilehash: 541b5e9f523c64bc55526e4e2dffc57a5212e67f
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060978"
---
# <a name="scalable-order-processing-on-azure"></a>Azure 上的可缩放订单处理

本示例方案针对那些需要高度可缩放且复原性好的体系结构进行联机订单处理的组织。 可能的应用包括：电子商务和零售点、订单履行、库存预留和跟踪。 

本方案采用事件溯源方法，使用通过微服务实现的函数编程模型。 可以将每个微服务视为一个流处理器，所有业务逻辑都通过微服务实现。 此方法可以实现高可用性和复原性、异地复制和快速执行。

使用托管式 Azure 服务（例如 Cosmos DB 和 HDInsight）可以充分利用 Microsoft 在全局分布式云规模数据存储和检索方面的专业技术，因此可以降低成本。 本方案专门针对电子商务或零售场景；若有其他的数据服务需求，则应查看 [Azure 中提供的完全托管式智能数据库服务][product-category]的列表。

## <a name="related-use-cases"></a>相关的用例

以下用例可以考虑本方案：

* 电子商务或零售点后端系统。
* 库存管理系统。
* 订单履行系统。
* 其他与订单处理管道相关的集成方案。

## <a name="architecture"></a>体系结构

![可缩放订单处理管道的示例体系结构][architecture-diagram]

此体系结构详细说明了订单处理管道的重要组件。 数据流经方案的情形如下所示：

1. 事件消息进入系统时，可以通过面向客户的应用程序进入（通过 HTTP 同步进入），也可以通过各种后端系统进入（通过 Apache Kafka 异步进入）。 这些消息会传递到一个命令处理管道中。
2. 每个事件消息都通过一个命令处理器微服务引入并映射到定义的一组命令中的一个。 命令处理器从事件流快照数据库中检索与执行该命令相关的任何最新状态。 然后会执行该命令，并将命令的输出以新事件的形式发出。
3. 以命令输出的形式发出的每个事件会通过 Cosmos DB 提交到一个事件流数据库。
4. 每次将数据库插入或更新提交到事件流数据库时，Cosmos DB 更改源都会引发一个事件。 下游系统可以订阅与该系统相关的任何事件主题。
5. 来自 Cosmos DB 更改源的所有事件也会发送到快照事件流微服务，后者会计算已发生事件导致的任何状态更改。 然后，新状态会提交到存储在 Cosmos DB 中的事件流快照数据库。  快照数据库为所有数据元素的当前状态提供一个全局分布式的低延迟数据源。 事件流数据库提供已通过体系结构传递的所有事件消息的完整记录，因此可以进行可靠的测试、故障排除和灾难恢复。  

### <a name="components"></a>组件

* [Cosmos DB][docs-cosmos-db] 是 Microsoft 推出的全局分布式多模型数据库，可以让解决方案跨任意数量的地理区域灵活且独立地缩放吞吐量与存储。 它通过综合服务级别协议 (SLA) 提供吞吐量、延迟、可用性和一致性保证。 本方案使用 Cosmos DB 进行事件流存储和快照存储，并利用 Cosmos DB 的更改源功能来确保数据一致性和故障恢复。 
* [Apache Kafka on HDInsight][docs-kafka] 是以托管服务方式实现的 Apache Kafka，是一种开源分布式流式处理平台，用于生成实时流数据管道和应用程序。 Kafka 还提供了类似于消息队列的消息中转站功能，用于发布和订阅命名数据流。 本方案使用 Kafka 在订单处理管道中处理出入事件和下游事件。 

## <a name="considerations"></a>注意事项

进行实时消息引入、数据存储、流处理、分析数据存储以及分析和报告时，有许多技术选项。 有关这些选项及其功能和主要选择标准的概述，请参阅 [Azure 数据体系结构指南](/azure/architecture/data-guide/)中的[大数据体系结构：实时处理](/azure/architecture/data-guide/technology-choices/real-time-ingestion)。

微服务已成为一种流行的体系结构类型，可用于构建可复原、高度可缩放、可独立部署且能快速演变的云应用程序。 微服务需要利用不同的方法来设计和生成应用程序。 可以使用 Docker、Kubernetes、Azure Service Fabric 和 Nomad 之类的工具来开发基于微服务的体系结构。 有关如何生成并运行基于微服务的体系结构的指南，请参阅 Azure 体系结构中心的[在 Azure 上设计微服务](/azure/architecture/microservices/)。

### <a name="availability"></a>可用性

本方案的事件溯源方法允许将系统组件松散地耦合在一起，在部署时这些组件可以彼此独立。 Cosmos DB 提供[高可用性][docs-cosmos-db-regional-failover]，可以帮助组织管理与一致性、可用性和性能相关的权衡因素，这些都有[相应的保证][docs-cosmos-db-guarantees]。 Apache Kafka on HDInsight 在设计时也考虑到了[高可用性][docs-kafka-high-availability]。

Azure Monitor 提供了统一的用户界面，可用于监视各种 Azure 服务。 有关详细信息，请参阅[在 Microsoft Azure 中进行监视](/azure/monitoring-and-diagnostics/monitoring-overview)。 事件中心和流分析均与 Azure Monitor 集成在一起。 

有关其他可用性注意事项，请参阅[可用性核对清单][availability]。

### <a name="scalability"></a>可伸缩性

Kafka on HDInsight 可用于为 Kafka 群集[配置存储和可伸缩性](/azure/hdinsight/kafka/apache-kafka-scalability)。 Cosmos DB 提供快速的可预测性能，并且可以随着应用程序规模的增长进行[无缝缩放](/azure/cosmos-db/partition-data)。
使用本方案的基于事件溯源微服务的体系结构，还可以更轻松地缩放系统并扩展其功能。

若要了解其他可伸缩性注意事项，请参阅 Azure 体系结构中心提供的[可伸缩性核对清单][scalability]。

### <a name="security"></a>“安全”

[Cosmos DB 安全模型](/azure/cosmos-db/secure-access-to-data)可以用来验证用户身份并访问其数据和资源。 有关详细信息，请参阅 [Cosmos DB 数据库安全性](/en-us/azure/cosmos-db/database-security)。

若需安全解决方案的通用设计指南，请参阅 [Azure 安全性文档][security]。

### <a name="resiliency"></a>复原

本示例方案中的事件溯源体系结构和关联的技术使得本方案在出现故障时很容易进行复原。 若需可复原解决方案的通用设计指南，请参阅[设计适用于 Azure 的可复原应用程序][resiliency]。

## <a name="pricing"></a>定价

为了方便用户查看运行本方案的成本，我们已在成本计算器中预配置了所有服务。  若要了解自己的特定方案的定价变化情况，请按预期的数据量更改相应的变量。 就本方案来说，示例定价仅包括 Cosmos DB 和一个 Kafka 群集，用于处理 Cosmos DB 更改源引发的事件。 用于始发系统和其他下游系统的事件处理器和微服务未包括在内，其成本主要取决于这些服务的数量和规模，以及为实现它们而选择的技术。

Azure Cosmos DB 的货币是请求单位 (RU)。 借助请求单位，无需保留读取/写入容量或预配 CPU、内存和 IOPS。 Azure Cosmos DB 支持不同操作（范围从简单读取、写入到复杂图形查询等）的许多 API。 并非所有请求都是相同的，因此系统会根据请求所需的计算量为它们分配规范化数量的请求单位。 解决方案需要的请求单位数取决于数据元素大小以及每秒的数据库读写操作数。 有关详细信息，请参阅 [Azure Cosmos DB 中的请求单位](/azure/cosmos-db/request-units)。 这些估价基于在两个 Azure 区域中运行的 Cosmos DB。

我们已根据你预期的活动量提供了三个示例性的成本配置文件：

* [小][small-pricing]：对应于 5 个保留的 RU 和 Cosmos DB 中的 1TB 数据存储，以及一个小型 (D3 v2) Kafka 群集。
* [中][medium-pricing]：对应于 50 个保留的 RU 和 Cosmos DB 中的 10TB 数据存储，以及一个中型 (D4 v2) Kafka 群集。
* [大][large-pricing]：对应于 500 个保留的 RU 和 Cosmos DB 中的 30TB 数据存储，以及一个大型 (D5 v2) Kafka 群集。

## <a name="related-resources"></a>相关资源

本示例方案基于此体系结构的一个更广泛的版本，该版本由 [Jet.com](https://jet.com) 针对其端到端订单处理管道而构建。 有关详细信息，请参阅 [jet.com 技术方面的客户配置文件][source-document]和 [jet.com 在 Build 2018 的演示文稿][source-presentation]。 

其他相关资源包括：
* _[Designing Data-Intensive Applications](https://dataintensive.net/)_（设计数据密集型应用程序），作者：Martin Kleppmann（O'Reilly Media，2017）。
* _[ Modeling Made Functional: Tackle Software Complexity with Domain-Driven Design and F#](https://pragprog.com/book/swdddf/domain-modeling-made-functional)_（域建模生效：使用域驱动型设计和 F# 减轻软件复杂性），作者：Scott Wlaschin（Pragmatic Programmers LLC，2018）。
* 其他 [Cosmos DB 用例][docs-cosmos-db-use-cases]
* [Azure 数据体系结构指南](/azure/architecture/data-guide/)中的[实时处理体系结构](/azure/architecture/data-guide/big-data/real-time-processing)

<!-- links -->
[product-category]: https://azure.microsoft.com/product-categories/databases/
[source-document]: https://customers.microsoft.com/en-us/story/jet-com-powers-innovative-e-commerce-engine-on-azure-in-less-than-12-months
[source-presentation]: https://channel9.msdn.com/events/Build/2018/BRK3602
[small-pricing]: https://azure.com/e/3d43949ffbb945a88cc0a126dc3a0e6e
[medium-pricing]: https://azure.com/e/1f1e7bf2a6ad4f7799581211f4369b9b
[large-pricing]: https://azure.com/e/75207172ece94cf6b5fb354a2252b333
[architecture-diagram]: ./images/architecture-diagram-cosmos-db.png
[docs-cosmos-db]: /azure/cosmos-db
[docs-cosmos-db-change-feed]: /azure/cosmos-db/change-feed
[docs-cosmos-db-regional-failover]: /azure/cosmos-db/regional-failover
[docs-cosmos-db-guarantees]: /azure/cosmos-db/distribute-data-globally#AvailabilityGuarantees
[docs-cosmos-db-use-cases]: /azure/cosmos-db/use-cases
[docs-kafka]: /azure/hdinsight/kafka/apache-kafka-introduction
[docs-kafka-high-availability]: /azure/hdinsight/kafka/apache-kafka-high-availability
[docs-event-hubs]: /azure/event-hubs/event-hubs-what-is-event-hubs
[docs-stream-analytics]: /azure/stream-analytics/stream-analytics-introduction
[docs-blob-storage]: /azure/storage/blobs/storage-blobs-introduction
[availability]: /azure/architecture/checklist/availability
[scalability]: /azure/architecture/checklist/scalability
[resiliency]: /azure/architecture/patterns/category/resiliency/
[security]: /azure/security/
