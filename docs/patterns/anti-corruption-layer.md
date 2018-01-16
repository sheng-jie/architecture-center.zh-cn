---
title: "防损层模式"
description: "在现代应用程序与旧系统之间实施外观或适配器层。"
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: e41f080abbef772596ee7f8b10ad72bb03a3b829
ms.sourcegitcommit: c93f1b210b3deff17cc969fb66133bc6399cfd10
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/05/2018
---
# <a name="anti-corruption-layer-pattern"></a><span data-ttu-id="9cba8-103">防损层模式</span><span class="sxs-lookup"><span data-stu-id="9cba8-103">Anti-Corruption Layer pattern</span></span>

<span data-ttu-id="9cba8-104">在新式应用程序和它所依赖的旧系统之间实施外层或适配器层。</span><span class="sxs-lookup"><span data-stu-id="9cba8-104">Implement a façade or adapter layer between a modern application and a legacy system that it depends on.</span></span> <span data-ttu-id="9cba8-105">此层将转换新式应用程序与旧系统之间的请求。</span><span class="sxs-lookup"><span data-stu-id="9cba8-105">This layer translates requests between the modern application and the legacy system.</span></span> <span data-ttu-id="9cba8-106">使用此模式以确保应用程序的设计不受旧系统上的依赖项的限制。</span><span class="sxs-lookup"><span data-stu-id="9cba8-106">Use this pattern to ensure that an application's design is not limited by dependencies on legacy systems.</span></span> <span data-ttu-id="9cba8-107">此模式最先由 Eric Evans 在 *Domain-Driven Design*（域驱动的设计）中描述。</span><span class="sxs-lookup"><span data-stu-id="9cba8-107">This pattern was first described by Eric Evans in *Domain-Driven Design*.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="9cba8-108">上下文和问题</span><span class="sxs-lookup"><span data-stu-id="9cba8-108">Context and problem</span></span>

<span data-ttu-id="9cba8-109">大多数应用程序依赖于其他系统的某些数据或功能。</span><span class="sxs-lookup"><span data-stu-id="9cba8-109">Most applications rely on other systems for some data or functionality.</span></span> <span data-ttu-id="9cba8-110">例如，旧版应用程序迁移到新式系统时，可能仍需要现有的旧的资源。</span><span class="sxs-lookup"><span data-stu-id="9cba8-110">For example, when a legacy application is migrated to a modern system, it may still need existing legacy resources.</span></span> <span data-ttu-id="9cba8-111">新功能必须能够调用旧系统。</span><span class="sxs-lookup"><span data-stu-id="9cba8-111">New features must be able to call the legacy system.</span></span> <span data-ttu-id="9cba8-112">逐步迁移尤其如此，随着时间推移，较大型应用程序的不同功能迁移到新式系统中。</span><span class="sxs-lookup"><span data-stu-id="9cba8-112">This is especially true of gradual migrations, where different features of a larger application are moved to a modern system over time.</span></span>

<span data-ttu-id="9cba8-113">这些旧系统通常会出现质量问题，如复杂的数据架构或过时的 API。</span><span class="sxs-lookup"><span data-stu-id="9cba8-113">Often these legacy systems suffer from quality issues such as convoluted data schemas or obsolete APIs.</span></span> <span data-ttu-id="9cba8-114">旧系统使用的功能和技术可能与新式系统中的功能和技术有很大差异。</span><span class="sxs-lookup"><span data-stu-id="9cba8-114">The features and technologies used in legacy systems can vary widely from more modern systems.</span></span> <span data-ttu-id="9cba8-115">若要与旧系统进行互操作，新应用程序可能需要支持过时的基础结构、协议、数据模型、API、或其他不会引入新式应用程序的功能。</span><span class="sxs-lookup"><span data-stu-id="9cba8-115">To interoperate with the legacy system, the new application may need to support outdated infrastructure, protocols, data models, APIs, or other features that you wouldn't otherwise put into a modern application.</span></span>

<span data-ttu-id="9cba8-116">保持新旧系统之间的访问可以强制新系统至少支持某些旧系统的 API 或其他语义。</span><span class="sxs-lookup"><span data-stu-id="9cba8-116">Maintaining access between new and legacy systems can force the new system to adhere to at least some of the legacy system's APIs or other semantics.</span></span> <span data-ttu-id="9cba8-117">这些旧的功能出现质量问题时，支持它们“损坏”可能会是完全设计的新式应用程序。</span><span class="sxs-lookup"><span data-stu-id="9cba8-117">When these legacy features have quality issues, supporting them "corrupts" what might otherwise be a cleanly designed modern application.</span></span> 

## <a name="solution"></a><span data-ttu-id="9cba8-118">解决方案</span><span class="sxs-lookup"><span data-stu-id="9cba8-118">Solution</span></span>

<span data-ttu-id="9cba8-119">通过在旧系统和新式系统之间放置防损层，将它们隔离。</span><span class="sxs-lookup"><span data-stu-id="9cba8-119">Isolate the legacy and modern systems by placing an anti-corruption layer between them.</span></span> <span data-ttu-id="9cba8-120">此层转换两个系统之间的通信，在旧系统保持不变的情况下，新式应用程序可以避免破坏其设计和技术方法。</span><span class="sxs-lookup"><span data-stu-id="9cba8-120">This layer translates communications between the two systems, allowing the legacy system to remain unchanged while the modern application can avoid compromising its design and technological approach.</span></span>

![](./_images/anti-corruption-layer.png) 

<span data-ttu-id="9cba8-121">新式应用程序和防损层之间的通信始终使用应用程序的数据模型和体系结构。</span><span class="sxs-lookup"><span data-stu-id="9cba8-121">Communication between the modern application and the anti-corruption layer always uses the application's data model and architecture.</span></span> <span data-ttu-id="9cba8-122">从防损层到旧系统的调用符合该系统的数据模型或方法。</span><span class="sxs-lookup"><span data-stu-id="9cba8-122">Calls from the anti-corruption layer to the legacy system conform to that system's data model or methods.</span></span> <span data-ttu-id="9cba8-123">防损层包含在两个系统之间转换所必需的所有逻辑。</span><span class="sxs-lookup"><span data-stu-id="9cba8-123">The anti-corruption layer contains all of the logic necessary to translate between the two systems.</span></span> <span data-ttu-id="9cba8-124">该层可作为应用程序内的组件或作为独立服务实现。</span><span class="sxs-lookup"><span data-stu-id="9cba8-124">The layer can be implemented as a component within the application or as an independent service.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="9cba8-125">问题和注意事项</span><span class="sxs-lookup"><span data-stu-id="9cba8-125">Issues and considerations</span></span>

- <span data-ttu-id="9cba8-126">防损层可能将延迟添加到两个系统之间的调用。</span><span class="sxs-lookup"><span data-stu-id="9cba8-126">The anti-corruption layer may add latency to calls made between the two systems.</span></span>
- <span data-ttu-id="9cba8-127">防损层将添加一项必须管理和维护的其他服务。</span><span class="sxs-lookup"><span data-stu-id="9cba8-127">The anti-corruption layer adds an additional service that must be managed and maintained.</span></span>
- <span data-ttu-id="9cba8-128">请考虑防损层的缩放方式。</span><span class="sxs-lookup"><span data-stu-id="9cba8-128">Consider how your anti-corruption layer will scale.</span></span>
- <span data-ttu-id="9cba8-129">请考虑是否需要多个防损层。</span><span class="sxs-lookup"><span data-stu-id="9cba8-129">Consider whether you need more than one anti-corruption layer.</span></span> <span data-ttu-id="9cba8-130">可能需要使用不同的技术或语言将功能分解为多个服务，或者可能因其他原因对防损层进行分区。</span><span class="sxs-lookup"><span data-stu-id="9cba8-130">You may want to decompose functionality into multiple services using different technologies or languages, or there may be other reasons to partition the anti-corruption layer.</span></span>
- <span data-ttu-id="9cba8-131">请考虑如何管理与其他应用程序或服务相关的防损层。</span><span class="sxs-lookup"><span data-stu-id="9cba8-131">Consider how the anti-corruption layer will be managed in relation with your other applications or services.</span></span> <span data-ttu-id="9cba8-132">如何将其集成到监视、发布和配置进程中？</span><span class="sxs-lookup"><span data-stu-id="9cba8-132">How will it be integrated into your monitoring, release, and configuration processes?</span></span>
- <span data-ttu-id="9cba8-133">确保维护并可以监视事务和数据一致性。</span><span class="sxs-lookup"><span data-stu-id="9cba8-133">Make sure transaction and data consistency are maintained and can be monitored.</span></span>
- <span data-ttu-id="9cba8-134">请考虑防损层需要处理旧系统和新式系统之间的所有通信，还是仅需要处理部分功能。</span><span class="sxs-lookup"><span data-stu-id="9cba8-134">Consider whether the anti-corruption layer needs to handle all communication between legacy and modern systems, or just a subset of features.</span></span> 
- <span data-ttu-id="9cba8-135">请考虑防损层是永久性，还是在所有旧功能迁移后即停用。</span><span class="sxs-lookup"><span data-stu-id="9cba8-135">Consider whether the anti-corruption layer is meant to be permanent, or eventually retired once all legacy functionality has been migrated.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="9cba8-136">何时使用此模式</span><span class="sxs-lookup"><span data-stu-id="9cba8-136">When to use this pattern</span></span>

<span data-ttu-id="9cba8-137">在以下情况下使用此模式：</span><span class="sxs-lookup"><span data-stu-id="9cba8-137">Use this pattern when:</span></span>

- <span data-ttu-id="9cba8-138">迁移计划为发生在多个阶段，但是新旧系统之间的集成需要维护。</span><span class="sxs-lookup"><span data-stu-id="9cba8-138">A migration is planned to happen over multiple stages, but integration between new and legacy systems needs to be maintained.</span></span>
- <span data-ttu-id="9cba8-139">新旧系统具有不同的语义，但是仍需要进行通信。</span><span class="sxs-lookup"><span data-stu-id="9cba8-139">New and legacy system have different semantics, but still need to communicate.</span></span>

<span data-ttu-id="9cba8-140">如果新旧系统之间没有重要的语义差异，则此模式可能不适合。</span><span class="sxs-lookup"><span data-stu-id="9cba8-140">This pattern may not be suitable if there are no significant semantic differences between new and legacy systems.</span></span> 

## <a name="related-guidance"></a><span data-ttu-id="9cba8-141">相关指南</span><span class="sxs-lookup"><span data-stu-id="9cba8-141">Related guidance</span></span>

- <span data-ttu-id="9cba8-142">[Strangler 模式][strangler]</span><span class="sxs-lookup"><span data-stu-id="9cba8-142">[Strangler pattern][strangler]</span></span>

[strangler]: ./strangler.md
