---
title: 使用托管服务
description: 如果可能，请使用平台即为服务 (PaaS)，而不是基础结构即服务 (IaaS)
author: MikeWasson
ms.openlocfilehash: 6d3cfb2e97b5a9b25bb1afd72059e981ef45c0d8
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206546"
---
# <a name="use-managed-services"></a>使用托管服务

## <a name="when-possible-use-platform-as-a-service-paas-rather-than-infrastructure-as-a-service-iaas"></a>如果可能，请使用平台即为服务 (PaaS)，而不是基础结构即服务 (IaaS)

IaaS 就像有一盒零件。 你可以构建任何东西，但必须自己组装。 托管服务更易于配置和管理。 无需提供 VM、设置 VNet、管理补丁和更新，以及与在 VM 上运行软件相关的所有其他开销。

例如，假设应用程序需要一个消息队列。 可以使用类似 RabbitMQ 的东西在 VM 上设置自己的消息传递服务。 但 Azure 服务总线已经提供了可靠的消息传递作为服务，而且它更易于设置。 只需创建一个服务总线命名空间（这可以作为部署脚本的一部分来完成），然后使用客户端 SDK 调用服务总线。 

当然，应用程序可能具有某些特定要求，会使 IaaS 方法更合适。 但即使应用程序基于 IaaS，也能找到可以自然合并托管服务的位置。 其中包括缓存、队列和数据存储。

| 取消运行… | 考虑使用… |
|-----------------------|-------------|
| Active Directory | Azure Active Directory 域服务 |
| Elasticsearch | Azure 搜索 |
| Hadoop | HDInsight |
| IIS | 应用服务 |
| MongoDB | Cosmos DB |
| Redis | Azure Redis 缓存 |
| SQL Server | Azure SQL 数据库 |


