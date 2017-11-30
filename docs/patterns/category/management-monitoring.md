---
title: "监视和管理模式"
description: "云应用程序在远程数据中心内运行，在此中心内，无法完全控制基础架构，或者在某些情况下无法控制操作系统。 与本地部署相比，管理和监视难度更大。 应用程序必须公开运行信息，以便管理员和运算符可用于管理和监视此系统，支持不断变化的业务要求和自定义项，而无需停止或重新部署应用程序。"
keywords: "设计模式"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 6281f7e5c62593cc60727c994d5dba2c52976b75
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="management-and-monitoring-patterns"></a>监视和管理模式

云应用程序在远程数据中心内运行，在此中心内，无法完全控制基础架构，或者在某些情况下无法控制操作系统。 与本地部署相比，管理和监视难度更大。 应用程序必须公开运行信息，以便管理员和运算符可用于管理和监视此系统，支持不断变化的业务要求和自定义项，而无需停止或重新部署应用程序。

| 模式 | 摘要 |
| ------- | ------- |
| [代表](../ambassador.md) | 创建代表客户服务或应用程序发送网络请求的帮助程序服务。 |
| [防损层](../anti-corruption-layer.md) | 在现代应用程序与旧系统之间实施外观或适配器层。 |
| [外部配置存储](../external-configuration-store.md) | 将配置信息从应用程序部署包移出，移到一个集中的位置。 |
| [网关聚合](../gateway-aggregation.md) | 使用网关可将多个单独请求聚合成一个请求。 |
| [网关卸载](../gateway-offloading.md) | 将共享或专用服务功能卸载到网关代理。 |
| [网关路由](../gateway-routing.md) | 使用单个终结点将请求路由到多个服务。 |
| [运行状况终结点监视](../health-endpoint-monitoring.md) | 在应用程序中实施可让外部工具通过公开终结点定期访问的功能检查。 |
| [sidecar](../sidecar.md) | 将应用程序的组件部署到单独的进程或容器中，以提供隔离和封装。 |
| [Strangler](../strangler.md) | 通过将特定的功能片断逐渐取代为新的应用程序和服务，逐步迁移旧系统。 |
