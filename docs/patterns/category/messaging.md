---
title: 消息传送模式
description: 云应用程序的分布性质要求消息基础结构在理想情况下能以松散耦合的方式连接组件和服务，从而将可伸缩性最大化。 异步消息受到广泛使用并提供了诸多好处，但也带来了许多挑战，如消息排序、有害消息管理和幂等性等。
keywords: 设计模式
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 8bf37903df3a6eb23f1581e0405358a7aee61f79
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/06/2018
---
# <a name="messaging-patterns"></a><span data-ttu-id="e0b30-105">消息传送模式</span><span class="sxs-lookup"><span data-stu-id="e0b30-105">Messaging patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="e0b30-106">云应用程序的分布性质要求消息基础结构在理想情况下能以松散耦合的方式连接组件和服务，从而将可伸缩性最大化。</span><span class="sxs-lookup"><span data-stu-id="e0b30-106">The distributed nature of cloud applications requires a messaging infrastructure that connects the components and services, ideally in a loosely coupled manner in order to maximize scalability.</span></span> <span data-ttu-id="e0b30-107">异步消息受到广泛使用并提供了诸多好处，但也带来了许多挑战，如消息排序、有害消息管理和幂等性等。</span><span class="sxs-lookup"><span data-stu-id="e0b30-107">Asynchronous messaging is widely used, and provides many benefits, but also brings challenges such as the ordering of messages, poison message management, idempotency, and more.</span></span>


|                            <span data-ttu-id="e0b30-108">模式</span><span class="sxs-lookup"><span data-stu-id="e0b30-108">Pattern</span></span>                             |                                                                        <span data-ttu-id="e0b30-109">摘要</span><span class="sxs-lookup"><span data-stu-id="e0b30-109">Summary</span></span>                                                                         |
|----------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
|        [<span data-ttu-id="e0b30-110">竞争性使用者</span><span class="sxs-lookup"><span data-stu-id="e0b30-110">Competing Consumers</span></span>](../competing-consumers.md)        |                            <span data-ttu-id="e0b30-111">使多个并发使用者能够处理同一消息通道上收到的消息。</span><span class="sxs-lookup"><span data-stu-id="e0b30-111">Enable multiple concurrent consumers to process messages received on the same messaging channel.</span></span>                            |
|          [<span data-ttu-id="e0b30-112">管道和筛选器</span><span class="sxs-lookup"><span data-stu-id="e0b30-112">Pipes and Filters</span></span>](../pipes-and-filters.md)          |                       <span data-ttu-id="e0b30-113">将一个执行复杂处理的任务分解为一系列可重复使用的单个元素。</span><span class="sxs-lookup"><span data-stu-id="e0b30-113">Break down a task that performs complex processing into a series of separate elements that can be reused.</span></span>                        |
|             [<span data-ttu-id="e0b30-114">优先级队列</span><span class="sxs-lookup"><span data-stu-id="e0b30-114">Priority Queue</span></span>](../priority-queue.md)             | <span data-ttu-id="e0b30-115">为发送到服务的请求确定优先级，以便高优先级请求能够得到比低优先级请求更快速地接收和处理。</span><span class="sxs-lookup"><span data-stu-id="e0b30-115">Prioritize requests sent to services so that requests with a higher priority are received and processed more quickly than those with a lower priority.</span></span> |
|  [<span data-ttu-id="e0b30-116">基于队列的负载调控</span><span class="sxs-lookup"><span data-stu-id="e0b30-116">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md)  |              <span data-ttu-id="e0b30-117">使用队列在任务与所调用的服务之间充当缓冲，从而缓解间歇性负载过大现象。</span><span class="sxs-lookup"><span data-stu-id="e0b30-117">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span>               |
| [<span data-ttu-id="e0b30-118">计划程序代理监督程序</span><span class="sxs-lookup"><span data-stu-id="e0b30-118">Scheduler Agent Supervisor</span></span>](../scheduler-agent-supervisor.md) |                              <span data-ttu-id="e0b30-119">跨一组分布式服务和其他远程资源协调一组操作。</span><span class="sxs-lookup"><span data-stu-id="e0b30-119">Coordinate a set of actions across a distributed set of services and other remote resources.</span></span>                              |

