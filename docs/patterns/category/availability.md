---
title: 可用性模式
description: 可用性定义系统正常工作的时间所占的比例。 它会受系统错误、基础结构问题、恶意攻击和系统负载的影响。 通常通过运行时间百分比衡量。 云应用程序通常向用户提供服务级别协议 (SLA)，这意味着必须以最大程度确保可用性的方式设计和实现应用程序。
keywords: 设计模式
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: eac1d192fe83f8501d11dce02d9bec16e939ba55
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/06/2018
---
# <a name="availability-patterns"></a><span data-ttu-id="c6d94-107">可用性模式</span><span class="sxs-lookup"><span data-stu-id="c6d94-107">Availability patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="c6d94-108">可用性定义系统正常工作的时间所占的比例。</span><span class="sxs-lookup"><span data-stu-id="c6d94-108">Availability defines the proportion of time that the system is functional and working.</span></span> <span data-ttu-id="c6d94-109">它会受系统错误、基础结构问题、恶意攻击和系统负载的影响。</span><span class="sxs-lookup"><span data-stu-id="c6d94-109">It will be affected by system errors, infrastructure problems, malicious attacks, and system load.</span></span> <span data-ttu-id="c6d94-110">通常通过运行时间百分比衡量。</span><span class="sxs-lookup"><span data-stu-id="c6d94-110">It is usually measured as a percentage of uptime.</span></span> <span data-ttu-id="c6d94-111">云应用程序通常向用户提供服务级别协议 (SLA)，这意味着必须以最大程度确保可用性的方式设计和实现应用程序。</span><span class="sxs-lookup"><span data-stu-id="c6d94-111">Cloud applications typically provide users with a service level agreement (SLA), which means that applications must be designed and implemented in a way that maximizes availability.</span></span>


|                            <span data-ttu-id="c6d94-112">模式</span><span class="sxs-lookup"><span data-stu-id="c6d94-112">Pattern</span></span>                             |                                                           <span data-ttu-id="c6d94-113">摘要</span><span class="sxs-lookup"><span data-stu-id="c6d94-113">Summary</span></span>                                                            |
|----------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| [<span data-ttu-id="c6d94-114">运行状况终结点监视</span><span class="sxs-lookup"><span data-stu-id="c6d94-114">Health Endpoint Monitoring</span></span>](../health-endpoint-monitoring.md) | <span data-ttu-id="c6d94-115">在应用程序中实施可让外部工具通过公开终结点定期访问的功能检查。</span><span class="sxs-lookup"><span data-stu-id="c6d94-115">Implement functional checks in an application that external tools can access through exposed endpoints at regular intervals.</span></span> |
|  [<span data-ttu-id="c6d94-116">基于队列的负载调控</span><span class="sxs-lookup"><span data-stu-id="c6d94-116">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md)  | <span data-ttu-id="c6d94-117">使用队列在任务与所调用的服务之间充当缓冲，从而缓解间歇性负载过大现象。</span><span class="sxs-lookup"><span data-stu-id="c6d94-117">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span>  |
|                 [<span data-ttu-id="c6d94-118">限制</span><span class="sxs-lookup"><span data-stu-id="c6d94-118">Throttling</span></span>](../throttling.md)                 |   <span data-ttu-id="c6d94-119">控制应用程序实例、单个租户或整个服务对资源的消耗。</span><span class="sxs-lookup"><span data-stu-id="c6d94-119">Control the consumption of resources used by an instance of an application, an individual tenant, or an entire service.</span></span>    |

