---
title: "消息模式"
description: "云应用程序的分布性质要求消息基础结构在理想情况下能以松散耦合的方式连接组件和服务，从而将可伸缩性最大化。 异步消息受到广泛使用并提供了诸多好处，但也带来了许多挑战，如消息排序、有害消息管理和幂等性等。"
keywords: "设计模式"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 6151f7f76fc7b3a953988122db75bdc25b49811f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="messaging-patterns"></a>消息模式

[!INCLUDE [header](../../_includes/header.md)]

云应用程序的分布性质要求消息基础结构在理想情况下能以松散耦合的方式连接组件和服务，从而将可伸缩性最大化。 异步消息受到广泛使用并提供了诸多好处，但也带来了许多挑战，如消息排序、有害消息管理和幂等性等。

| 模式 | 摘要 |
| ------- | ------- |
| [使用者竞争](../competing-consumers.md) | 使多个并发使用者能够处理同一消息通道上收到的消息。 |
| [管道和筛选器](../pipes-and-filters.md) | 将一个执行复杂处理的任务分解为一系列可重复使用的单个元素。 |
| [优先级队列](../priority-queue.md) | 为发送到服务的请求确定优先级，以便高优先级请求能够得到比低优先级请求更快速地接收和处理。 |
| [基于队列的负载调节](../queue-based-load-leveling.md) | 使用队列在任务与所调用的服务之间充当缓冲，从而缓解间歇性负载过大现象。 |
| [计划程序代理监督程序](../scheduler-agent-supervisor.md) | 跨一组分布式服务和其他远程资源协调一组操作。 |