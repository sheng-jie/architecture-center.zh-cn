---
title: "数据管理模式"
description: "数据管理是云应用程序的关键元素，它影响着大部分质量属性。 出于性能、可伸缩性或可用性等方面的原因，数据通常托管在不同的位置并跨多个服务器，这可能会带来一系列的挑战。 例如，必须维护数据一致性，通常需要跨不同的位置同步数据。"
keywords: "设计模式"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: a009a06268f114ab7be4544dd81710612dabd8f4
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="data-management-patterns"></a>数据管理模式

[!INCLUDE [header](../../_includes/header.md)]

数据管理是云应用程序的关键元素，它影响着大部分质量属性。 出于性能、可伸缩性或可用性等方面的原因，数据通常托管在不同的位置并跨多个服务器，这可能会带来一系列的挑战。 例如，必须维护数据一致性，通常需要跨不同的位置同步数据。

| 模式 | 摘要 |
| ------- | ------- |
| [缓存端](../cache-aside.md) | 将数据按需从数据存储加载到缓存中 |
| [CQRS](../cqrs.md) | 使用独立接口将读取数据的操作与更新数据的操作分离。 |
| [事件溯源](../event-sourcing.md) | 使用只追加存储来记录描述域中数据采取的操作的完整系列事件。 |
| [索引表](../index-table.md) | 在数据存储中对查询经常引用的字段创建索引。 |
| [具体化视图](../materialized-view.md) | 当未针对所需的查询操作完美设置数据的格式时，在一个或多个数据存储中基于数据生成预填充的视图。 |
| [分片](../sharding.md) | 将数据存储划分为一组水平分区或分片。 |
| [静态内容托管](../static-content-hosting.md) | 将静态内容部署到基于云的存储服务，再由后者将它们直接传送给客户端。 |
| [附属密钥](../valet-key.md) | 使用令牌或密钥，向客户端授予对特定资源或服务的受限直接访问权限。 |