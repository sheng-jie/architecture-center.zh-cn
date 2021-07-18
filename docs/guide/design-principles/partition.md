---
title: 通过分区解决限制
description: 使用分区来解决数据库、网络和计算限制
author: MikeWasson
ms.openlocfilehash: 86306c6c33ea7a93c4c1f868d820cc522095a8b7
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206528"
---
# <a name="partition-around-limits"></a>通过分区解决限制

## <a name="use-partitioning-to-work-around-database-network-and-compute-limits"></a>使用分区来解决数据库、网络和计算限制

在云中，所有服务都有纵向扩展的限制。 [Azure 订阅和服务限制、配额和约束][azure-limits]中介绍了 Azure 服务限制。 限制包括内核数、数据库大小、查询吞吐量和网络吞吐量。 如果系统增长到足够大，可能会命中一个或多个限制。 使用分区来解决这些限制。

有很多方法可用于分区系统，例如：

- 分区数据库，以避免对数据库大小、数据 I/O 或并发会话数的限制。

- 分区队列或消息总线，以避免对请求数或并发连接数的限制。

- 分区应用服务 Web 应用，以避免对每个应用服务计划的实例数的限制。 

可以水平、垂直、或按功能分区数据库。

- 在水平分区（也称为分片）中，每个分区保存总数据集的子集的数据。 这些分区共享相同的数据架构。 例如，名称以 A&ndash;M 开头的客户进入一个分区，以 N&ndash;Z 开头的进入另一个分区。

- 在垂直分区中，每个分区保存数据存储中项字段的子集。 例如，将经常访问的字段放在一个分区中，将较不经常访问的字段放在另一个分区。

- 在功能分区中，根据系统中每个界限上下文使用数据的方式对数据进行分区。 例如，在一个分区中存储发票数据，在另一个分区中存储产品库存数据。 架构是独立的。

有关更多详细指南，请参阅[数据分区][data-partitioning-guidance]。

## <a name="recommendations"></a>建议

**分区应用程序的不同部分**。 数据库显然很适合用于分区，但也需考虑存储、缓存、队列和计算实例。

**设计分区键以避免产生热点**。 如果分区数据库，但一个分片仍然获取大多数请求，那么问题还未解决。 理想情况下，负载会在所有分区中均匀分布。 例如，按客户 ID 而不是客户名称的首字母进行哈希分区，因为某些首字母会更集中。 分区消息队列时，该原则也同样适用。 选择一个可以在队列集中平均分布消息的分区键。 有关更多信息，请参阅[分片][sharding]。

**通过分区解决 Azure 订阅和服务限制**。 单个组件和服务有限制，但订阅和资源组也有限制。 对于非常大的应用程序，可能需要进行分区来解决这些限制。  

**在不同级别分区**。 请考虑部署在 VM 上的数据库服务器。 VM 有一个由 Azure 存储支持的 VHD。 存储帐户属于 Azure 订阅。 请注意，层次结构中的每个步骤都有限制。 数据库服务器可能有连接池限制。 VM 有 CPU 和网络限制。 存储有 IOPS 限制。 订阅有 VM 内核数的限制。 一般来说，在较低的层次结构更容易分区。 仅大型应用程序需要在订阅级别进行分区。 

<!-- links -->

[azure-limits]: /azure/azure-subscription-service-limits
[data-partitioning-guidance]: ../../best-practices/data-partitioning.md
[sharding]: ../../patterns/sharding.md

 