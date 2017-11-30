---
title: "设计和实施模式"
description: "优秀的设计包含许多要素，例如部件设计和部署中的一致性和连贯性、简化管理和开发的可维护性，以及允许部件和子系统在其他应用程序和方案中使用的可重用性。 在设计和实施阶段做出的决策对云托管应用程序和服务的质量和总拥有成本具有巨大影响。"
keywords: "设计模式"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 3d6e528b8c88c6fcc265d6425dcdae6fae5166fb
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="design-and-implementation-patterns"></a>设计和实施模式

优秀的设计包含许多要素，例如部件设计和部署中的一致性和连贯性、简化管理和开发的可维护性，以及允许部件和子系统在其他应用程序和方案中使用的可重用性。 在设计和实施阶段做出的决策对云托管应用程序和服务的质量和总拥有成本具有巨大影响。

| 模式 | 摘要 |
| ------- | ------- |
| [代表](../ambassador.md) | 创建代表客户服务或应用程序发送网络请求的帮助程序服务。 |
| [防损层](../anti-corruption-layer.md) | 在新式应用程序与旧系统之间实施外层或适配器层。 |
| [用于前端的后端](../backends-for-frontends.md) | 创建单独的后端服务，以供特定的前端应用程序或接口使用。 |
| [CQRS](../cqrs.md) | 使用独立接口将读取数据的操作与更新数据的操作分离。 |
| [计算资源合并](../compute-resource-consolidation.md) | 将多个任务或操作合并到单个计算单元 |
| [外部配置存储](../external-configuration-store.md) | 将配置信息从应用程序部署包移出，移到一个集中的位置。 |
| [网关聚合](../gateway-aggregation.md) | 使用网关可将多个单独请求聚合成一个请求。 |
| [网关卸载](../gateway-offloading.md) | 将共享或专用服务功能卸载到网关代理。 |
| [网关路由](../gateway-routing.md) | 使用单个终结点将请求路由到多个服务。 |
| [领导选择](../leader-election.md) | 通过选拔一个实例作为领导来负责管理其他实例，协调分布式应用程序中协作性任务实例集合所执行的操作。 |
| [管道和筛选器](../pipes-and-filters.md) | 将一个执行复杂处理的任务分解为一系列可重复使用的单个元素。 |
| [Sidecar](../sidecar.md) | 将应用程序的组件部署到单独的进程或容器中，以提供隔离和封装。 |
| [静态内容托管](../static-content-hosting.md) | 将静态内容部署到基于云的存储服务，再由后者将它们直接传送给客户端。 |
| [Strangler](../strangler.md) | 通过将特定的功能片断逐渐取代为新的应用程序和服务，逐步迁移旧系统。 |