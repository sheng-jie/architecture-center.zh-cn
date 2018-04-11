---
title: 微服务体系结构样式
description: 介绍 Azure 上微服务体系结构的好处、挑战和最佳做法
author: MikeWasson
ms.openlocfilehash: 08fd39b6cf0b3c88af654b27e21b2d7dd9fb19b1
ms.sourcegitcommit: 7764a804f000180c37a4f8dbab946b525f784f58
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/09/2018
---
# <a name="microservices-architecture-style"></a><span data-ttu-id="19b1f-103">微服务体系结构样式</span><span class="sxs-lookup"><span data-stu-id="19b1f-103">Microservices architecture style</span></span>

<span data-ttu-id="19b1f-104">微服务体系结构由一系列小型的自治服务组成。</span><span class="sxs-lookup"><span data-stu-id="19b1f-104">A microservices architecture consists of a collection of small, autonomous services.</span></span> <span data-ttu-id="19b1f-105">每个服务都是自包含服务，并且应实现单个业务功能。</span><span class="sxs-lookup"><span data-stu-id="19b1f-105">Each service is self-contained and should implement a single business capability.</span></span> <span data-ttu-id="19b1f-106">有关在 Azure 上构建微服务体系结构的详细指导，请参阅[在 Azure 中设计、构建和操作微服务](../../microservices/index.md)。</span><span class="sxs-lookup"><span data-stu-id="19b1f-106">For detailed guidance about building a microservices architecture on Azure, see [Designing, building, and operating microservices on Azure](../../microservices/index.md).</span></span>

![](./images/microservices-logical.svg)
 
<span data-ttu-id="19b1f-107">在某些方面，微服务是面向服务的体系结构 (SOA) 的自然演变，但微服务与 SOA 之间也存在一些差异。</span><span class="sxs-lookup"><span data-stu-id="19b1f-107">In some ways, microservices are the natural evolution of service oriented architectures (SOA), but there are differences between microservices and SOA.</span></span> <span data-ttu-id="19b1f-108">下面是微服务的一些典型特征：</span><span class="sxs-lookup"><span data-stu-id="19b1f-108">Here are some defining characteristics of a microservice:</span></span>

- <span data-ttu-id="19b1f-109">在微服务体系结构中，服务具有规模小、独立和松散耦合的特点。</span><span class="sxs-lookup"><span data-stu-id="19b1f-109">In a microservices architecture, services are small, independent, and loosely coupled.</span></span>

- <span data-ttu-id="19b1f-110">每个服务都是一个单独的基本代码，可由小型开发团队管理。</span><span class="sxs-lookup"><span data-stu-id="19b1f-110">Each service is a separate codebase, which can be managed by a small development team.</span></span>

- <span data-ttu-id="19b1f-111">服务可独立部署。</span><span class="sxs-lookup"><span data-stu-id="19b1f-111">Services can be deployed independently.</span></span> <span data-ttu-id="19b1f-112">团队可以更新现有服务，而无需重新生成和重新部署整个应用程序。</span><span class="sxs-lookup"><span data-stu-id="19b1f-112">A team can update an existing service without rebuilding and redeploying the entire application.</span></span>

- <span data-ttu-id="19b1f-113">服务负责暂留自己的数据或外部状态。</span><span class="sxs-lookup"><span data-stu-id="19b1f-113">Services are responsible for persisting their own data or external state.</span></span> <span data-ttu-id="19b1f-114">这一点与传统模型不同，后者由单独的数据层处理数据暂留。</span><span class="sxs-lookup"><span data-stu-id="19b1f-114">This differs from the traditional model, where a separate data layer handles data persistence.</span></span>

- <span data-ttu-id="19b1f-115">服务通过定义完善的 API 相互通信。</span><span class="sxs-lookup"><span data-stu-id="19b1f-115">Services communicate with each other by using well-defined APIs.</span></span> <span data-ttu-id="19b1f-116">每个服务的内部实现细节均对其他服务隐藏。</span><span class="sxs-lookup"><span data-stu-id="19b1f-116">Internal implementation details of each service are hidden from other services.</span></span>

- <span data-ttu-id="19b1f-117">服务无需共享相同的技术堆栈、库或框架。</span><span class="sxs-lookup"><span data-stu-id="19b1f-117">Services don't need to share the same technology stack, libraries, or frameworks.</span></span>

<span data-ttu-id="19b1f-118">除了服务本身，典型微服务体系结构中还会出现其他组件：</span><span class="sxs-lookup"><span data-stu-id="19b1f-118">Besides for the services themselves, some other components appear in a typical microservices architecture:</span></span>

<span data-ttu-id="19b1f-119">**管理**。</span><span class="sxs-lookup"><span data-stu-id="19b1f-119">**Management**.</span></span> <span data-ttu-id="19b1f-120">管理组件负责将服务放置在节点上、标识故障、跨节点重新平衡服务等等。</span><span class="sxs-lookup"><span data-stu-id="19b1f-120">The management component is responsible for placing services on nodes, identifying failures, rebalancing services across nodes, and so forth.</span></span>  

<span data-ttu-id="19b1f-121">**服务发现**。</span><span class="sxs-lookup"><span data-stu-id="19b1f-121">**Service Discovery**.</span></span>  <span data-ttu-id="19b1f-122">维护一个包含服务及其所在节点的列表。</span><span class="sxs-lookup"><span data-stu-id="19b1f-122">Maintains a list of services and which nodes they are located on.</span></span> <span data-ttu-id="19b1f-123">支持使用服务查找功能查找服务的终结点。</span><span class="sxs-lookup"><span data-stu-id="19b1f-123">Enables service lookup to find the endpoint for a service.</span></span> 

<span data-ttu-id="19b1f-124">**API 网关**。</span><span class="sxs-lookup"><span data-stu-id="19b1f-124">**API Gateway**.</span></span> <span data-ttu-id="19b1f-125">API 网关是客户端的入口点。</span><span class="sxs-lookup"><span data-stu-id="19b1f-125">The API gateway is the entry point for clients.</span></span> <span data-ttu-id="19b1f-126">客户端不直接调用服务，</span><span class="sxs-lookup"><span data-stu-id="19b1f-126">Clients don't call services directly.</span></span> <span data-ttu-id="19b1f-127">而是调用 API 网关，网关再将调用转发到后端上的相应服务。</span><span class="sxs-lookup"><span data-stu-id="19b1f-127">Instead, they call the API gateway, which forwards the call to the appropriate services on the back end.</span></span> <span data-ttu-id="19b1f-128">API 网关可以聚合来自多个服务的响应，并返回聚合的响应。</span><span class="sxs-lookup"><span data-stu-id="19b1f-128">The API gateway might aggregate the responses from several services and return the aggregated response.</span></span> 

<span data-ttu-id="19b1f-129">使用 API 网关的优点如下：</span><span class="sxs-lookup"><span data-stu-id="19b1f-129">The advantages of using an API gateway include:</span></span>

- <span data-ttu-id="19b1f-130">它分离了客户端与服务。</span><span class="sxs-lookup"><span data-stu-id="19b1f-130">It decouples clients from services.</span></span> <span data-ttu-id="19b1f-131">无需更新所有客户端，便可对服务进行版本控制或重构。</span><span class="sxs-lookup"><span data-stu-id="19b1f-131">Services can be versioned or refactored without needing to update all of the clients.</span></span>

-  <span data-ttu-id="19b1f-132">服务可以使用对 Web 不友好的消息传递协议，比如 AMQP。</span><span class="sxs-lookup"><span data-stu-id="19b1f-132">Services can use messaging protocols that are not web friendly, such as AMQP.</span></span>

- <span data-ttu-id="19b1f-133">API 网关可执行身份验证、日志记录、SSL 终止和负载均衡等其他跨领域功能。</span><span class="sxs-lookup"><span data-stu-id="19b1f-133">The API Gateway can perform other cross-cutting functions such as authentication, logging, SSL termination, and load balancing.</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="19b1f-134">何时使用此体系结构</span><span class="sxs-lookup"><span data-stu-id="19b1f-134">When to use this architecture</span></span>

<span data-ttu-id="19b1f-135">请对以下情况考虑使用此体系结构样式：</span><span class="sxs-lookup"><span data-stu-id="19b1f-135">Consider this architecture style for:</span></span>

- <span data-ttu-id="19b1f-136">需要较高发布速度的大型应用程序。</span><span class="sxs-lookup"><span data-stu-id="19b1f-136">Large applications that require a high release velocity.</span></span>

- <span data-ttu-id="19b1f-137">需要高度可缩放的复杂应用程序。</span><span class="sxs-lookup"><span data-stu-id="19b1f-137">Complex applications that need to be highly scalable.</span></span>

- <span data-ttu-id="19b1f-138">具有大量域或多个子域的应用程序。</span><span class="sxs-lookup"><span data-stu-id="19b1f-138">Applications with rich domains or many subdomains.</span></span>

- <span data-ttu-id="19b1f-139">由小型开发团队组成的组织。</span><span class="sxs-lookup"><span data-stu-id="19b1f-139">An organization that consists of small development teams.</span></span>


## <a name="benefits"></a><span data-ttu-id="19b1f-140">优点</span><span class="sxs-lookup"><span data-stu-id="19b1f-140">Benefits</span></span> 

- <span data-ttu-id="19b1f-141">**独立部署**。</span><span class="sxs-lookup"><span data-stu-id="19b1f-141">**Independent deployments**.</span></span> <span data-ttu-id="19b1f-142">无需重新部署整个应用程序便可更新服务，出现问题时可回滚或前滚更新。</span><span class="sxs-lookup"><span data-stu-id="19b1f-142">You can update a service without redeploying the entire application, and roll back or roll forward an update if something goes wrong.</span></span> <span data-ttu-id="19b1f-143">Bug 修复和功能发布更易管理，风险更低。</span><span class="sxs-lookup"><span data-stu-id="19b1f-143">Bug fixes and feature releases are more manageable and less risky.</span></span>

- <span data-ttu-id="19b1f-144">**独立开发**。</span><span class="sxs-lookup"><span data-stu-id="19b1f-144">**Independent development**.</span></span> <span data-ttu-id="19b1f-145">单个开发团队便可生成、测试和部署服务，</span><span class="sxs-lookup"><span data-stu-id="19b1f-145">A single development team can build, test, and deploy a service.</span></span> <span data-ttu-id="19b1f-146">从而推动持续创新，加快发布节奏。</span><span class="sxs-lookup"><span data-stu-id="19b1f-146">The result is continuous innovation and a faster release cadence.</span></span> 

- <span data-ttu-id="19b1f-147">**小型专属团队**。</span><span class="sxs-lookup"><span data-stu-id="19b1f-147">**Small, focused teams**.</span></span> <span data-ttu-id="19b1f-148">团队可专注于一个服务。</span><span class="sxs-lookup"><span data-stu-id="19b1f-148">Teams can focus on one service.</span></span> <span data-ttu-id="19b1f-149">缩小每个服务的范围后，基本代码变得更好理解，新的团队成员也能更快上手。</span><span class="sxs-lookup"><span data-stu-id="19b1f-149">The smaller scope of each service makes the code base easier to understand, and it's easier for new team members to ramp up.</span></span>

- <span data-ttu-id="19b1f-150">**错误隔离**。</span><span class="sxs-lookup"><span data-stu-id="19b1f-150">**Fault isolation**.</span></span> <span data-ttu-id="19b1f-151">某个服务中断不会影响整个应用程序。</span><span class="sxs-lookup"><span data-stu-id="19b1f-151">If a service goes down, it won't take out the entire application.</span></span> <span data-ttu-id="19b1f-152">但是，这并不意味着用户可以无偿复原。</span><span class="sxs-lookup"><span data-stu-id="19b1f-152">However, that doesn't mean you get resiliency for free.</span></span> <span data-ttu-id="19b1f-153">用户仍需遵循复原最佳做法和设计模式。</span><span class="sxs-lookup"><span data-stu-id="19b1f-153">You still need to follow resiliency best practices and design patterns.</span></span> <span data-ttu-id="19b1f-154">请参阅[设计适用于 Azure 的可复原应用程序][resiliency-overview]。</span><span class="sxs-lookup"><span data-stu-id="19b1f-154">See [Designing resilient applications for Azure][resiliency-overview].</span></span>

- <span data-ttu-id="19b1f-155">**混合技术堆栈**。</span><span class="sxs-lookup"><span data-stu-id="19b1f-155">**Mixed technology stacks**.</span></span> <span data-ttu-id="19b1f-156">团队可选取最适合其服务的技术。</span><span class="sxs-lookup"><span data-stu-id="19b1f-156">Teams can pick the technology that best fits their service.</span></span> 

- <span data-ttu-id="19b1f-157">**精细缩放**。</span><span class="sxs-lookup"><span data-stu-id="19b1f-157">**Granular scaling**.</span></span> <span data-ttu-id="19b1f-158">服务可独立缩放。</span><span class="sxs-lookup"><span data-stu-id="19b1f-158">Services can be scaled independently.</span></span> <span data-ttu-id="19b1f-159">与此同时，每个 VM 的较高服务密度也意味着 VM 资源得到充分利用。</span><span class="sxs-lookup"><span data-stu-id="19b1f-159">At the same time, the higher density of services per VM means that VM resources are fully utilized.</span></span> <span data-ttu-id="19b1f-160">使用放置约束，可将服务与 VM 配置文件（高 CPU、高内存等等）匹配。</span><span class="sxs-lookup"><span data-stu-id="19b1f-160">Using placement constraints, a services can be matched to a VM profile (high CPU, high memory, and so on).</span></span>

## <a name="challenges"></a><span data-ttu-id="19b1f-161">挑战</span><span class="sxs-lookup"><span data-stu-id="19b1f-161">Challenges</span></span>

- <span data-ttu-id="19b1f-162">**复杂性**。</span><span class="sxs-lookup"><span data-stu-id="19b1f-162">**Complexity**.</span></span> <span data-ttu-id="19b1f-163">与同等的单一式应用程序相比，微服务应用程序具有更多移动部件。</span><span class="sxs-lookup"><span data-stu-id="19b1f-163">A microservices application has more moving parts than the equivalent monolithic application.</span></span> <span data-ttu-id="19b1f-164">每个服务更简单，但整个系统作为整体来说更复杂。</span><span class="sxs-lookup"><span data-stu-id="19b1f-164">Each service is simpler, but the entire system as a whole is more complex.</span></span>

- <span data-ttu-id="19b1f-165">**开发和测试**。</span><span class="sxs-lookup"><span data-stu-id="19b1f-165">**Development and test**.</span></span> <span data-ttu-id="19b1f-166">针对服务依赖关系的开发需要采用不同的方法。</span><span class="sxs-lookup"><span data-stu-id="19b1f-166">Developing against service dependencies requires a different approach.</span></span> <span data-ttu-id="19b1f-167">现有工具不一定能处理这些服务依赖关系。</span><span class="sxs-lookup"><span data-stu-id="19b1f-167">Existing tools are not necessarily designed to work with service dependencies.</span></span> <span data-ttu-id="19b1f-168">跨服务边界进行重构可能很困难。</span><span class="sxs-lookup"><span data-stu-id="19b1f-168">Refactoring across service boundaries can be difficult.</span></span> <span data-ttu-id="19b1f-169">测试服务依赖关系也有一定难度，尤其是在应用程序快速发展之时。</span><span class="sxs-lookup"><span data-stu-id="19b1f-169">It is also challenging to test service dependencies, especially when the application is evolving quickly.</span></span>

- <span data-ttu-id="19b1f-170">**缺乏监管**。</span><span class="sxs-lookup"><span data-stu-id="19b1f-170">**Lack of governance**.</span></span> <span data-ttu-id="19b1f-171">用于生成微服务的分散式方法具有一定优势，但也可能导致许多问题。</span><span class="sxs-lookup"><span data-stu-id="19b1f-171">The decentralized approach to building microservices has advantages, but it can also lead to problems.</span></span> <span data-ttu-id="19b1f-172">用户在生成过程中可能采用了许多不同的语言和框架，从而使应用程序变得难以维护。</span><span class="sxs-lookup"><span data-stu-id="19b1f-172">You may end up with so many different languages and frameworks that the application becomes hard to maintain.</span></span> <span data-ttu-id="19b1f-173">这种情况下可以实施一些项目范围内的标准，不过分限制团队的灵活性。</span><span class="sxs-lookup"><span data-stu-id="19b1f-173">It may be useful to put some project-wide standards in place, without overly restricting teams' flexibility.</span></span> <span data-ttu-id="19b1f-174">这尤其适用于日志记录等跨领域功能。</span><span class="sxs-lookup"><span data-stu-id="19b1f-174">This especially applies to cross-cutting functionality such as logging.</span></span>

- <span data-ttu-id="19b1f-175">**网络拥塞和延迟**。</span><span class="sxs-lookup"><span data-stu-id="19b1f-175">**Network congestion and latency**.</span></span> <span data-ttu-id="19b1f-176">使用大量小型的精细服务可能会增加服务间的通信量。</span><span class="sxs-lookup"><span data-stu-id="19b1f-176">The use of many small, granular services can result in more interservice communication.</span></span> <span data-ttu-id="19b1f-177">此外，如果服务依赖关系链变得太长（服务 A 调用 B，B 调用 C...），额外延迟可能会成为一个问题。</span><span class="sxs-lookup"><span data-stu-id="19b1f-177">Also, if the chain of service dependencies gets too long (service A calls B, which calls C...), the additional latency can become a problem.</span></span> <span data-ttu-id="19b1f-178">用户需要精心设计 API。</span><span class="sxs-lookup"><span data-stu-id="19b1f-178">You will need to design APIs carefully.</span></span> <span data-ttu-id="19b1f-179">应避免过于繁琐的 API，考虑使用序列化格式，并找到可以使用异步通信模式的地方。</span><span class="sxs-lookup"><span data-stu-id="19b1f-179">Avoid overly chatty APIs, think about serialization formats, and look for places to use asynchronous communication patterns.</span></span>

- <span data-ttu-id="19b1f-180">**数据完整性**。</span><span class="sxs-lookup"><span data-stu-id="19b1f-180">**Data integrity**.</span></span> <span data-ttu-id="19b1f-181">每个微服务负责自己的数据暂留。</span><span class="sxs-lookup"><span data-stu-id="19b1f-181">With each microservice responsible for its own data persistence.</span></span> <span data-ttu-id="19b1f-182">因此，数据一致性可能是个挑战。</span><span class="sxs-lookup"><span data-stu-id="19b1f-182">As a result, data consistency can be a challenge.</span></span> <span data-ttu-id="19b1f-183">如果可能，请采用最终一致性。</span><span class="sxs-lookup"><span data-stu-id="19b1f-183">Embrace eventual consistency where possible.</span></span>

- <span data-ttu-id="19b1f-184">**管理**。</span><span class="sxs-lookup"><span data-stu-id="19b1f-184">**Management**.</span></span> <span data-ttu-id="19b1f-185">成功使用微服务需要有成熟的 DevOps 区域性。</span><span class="sxs-lookup"><span data-stu-id="19b1f-185">To be successful with microservices requires a mature DevOps culture.</span></span> <span data-ttu-id="19b1f-186">跨服务的关联日志记录可能很难。</span><span class="sxs-lookup"><span data-stu-id="19b1f-186">Correlated logging across services can be challenging.</span></span> <span data-ttu-id="19b1f-187">通常情况下，日志记录必须为单个用户操作关联多个服务调用。</span><span class="sxs-lookup"><span data-stu-id="19b1f-187">Typically, logging must correlate multiple service calls for a single user operation.</span></span>

- <span data-ttu-id="19b1f-188">**版本控制**。</span><span class="sxs-lookup"><span data-stu-id="19b1f-188">**Versioning**.</span></span> <span data-ttu-id="19b1f-189">对某个服务的更新不应中断依赖于它的其他服务。</span><span class="sxs-lookup"><span data-stu-id="19b1f-189">Updates to a service must not break services that depend on it.</span></span> <span data-ttu-id="19b1f-190">多个服务可在任意给定时间更新，因此，若不精心设计，可能会遇到向后或向前兼容性问题。</span><span class="sxs-lookup"><span data-stu-id="19b1f-190">Multiple services could be updated at any given time, so without careful design, you might have problems with backward or forward compatibility.</span></span>

- <span data-ttu-id="19b1f-191">**技能组合**。</span><span class="sxs-lookup"><span data-stu-id="19b1f-191">**Skillset**.</span></span> <span data-ttu-id="19b1f-192">微服务是一种高度分布式系统。</span><span class="sxs-lookup"><span data-stu-id="19b1f-192">Microservices are highly distributed systems.</span></span> <span data-ttu-id="19b1f-193">请仔细评估团队是否具有成功使用微服务所需的技能和经验。</span><span class="sxs-lookup"><span data-stu-id="19b1f-193">Carefully evaluate whether the team has the skills and experience to be successful.</span></span>

## <a name="best-practices"></a><span data-ttu-id="19b1f-194">最佳实践</span><span class="sxs-lookup"><span data-stu-id="19b1f-194">Best practices</span></span>

- <span data-ttu-id="19b1f-195">围绕业务域对服务建模。</span><span class="sxs-lookup"><span data-stu-id="19b1f-195">Model services around the business domain.</span></span> 

- <span data-ttu-id="19b1f-196">分散所有资源。</span><span class="sxs-lookup"><span data-stu-id="19b1f-196">Decentralize everything.</span></span> <span data-ttu-id="19b1f-197">单个团队负责设计和生成服务。</span><span class="sxs-lookup"><span data-stu-id="19b1f-197">Individual teams are responsible for designing and building services.</span></span> <span data-ttu-id="19b1f-198">避免共享代码或数据架构。</span><span class="sxs-lookup"><span data-stu-id="19b1f-198">Avoid sharing code or data schemas.</span></span> 

- <span data-ttu-id="19b1f-199">拥有数据的服务应当有专用的数据存储。</span><span class="sxs-lookup"><span data-stu-id="19b1f-199">Data storage should be private to the service that owns the data.</span></span> <span data-ttu-id="19b1f-200">为每个服务和数据类型使用最合适的存储。</span><span class="sxs-lookup"><span data-stu-id="19b1f-200">Use the best storage for each service and data type.</span></span> 

- <span data-ttu-id="19b1f-201">服务通过设计完善的 API 进行通信。</span><span class="sxs-lookup"><span data-stu-id="19b1f-201">Services communicate through well-designed APIs.</span></span> <span data-ttu-id="19b1f-202">避免泄露实现细节。</span><span class="sxs-lookup"><span data-stu-id="19b1f-202">Avoid leaking implementation details.</span></span> <span data-ttu-id="19b1f-203">API 应对域建模，而不是对服务的内部实现建模。</span><span class="sxs-lookup"><span data-stu-id="19b1f-203">APIs should model the domain, not the internal implementation of the service.</span></span>

- <span data-ttu-id="19b1f-204">避免服务之间耦合。</span><span class="sxs-lookup"><span data-stu-id="19b1f-204">Avoid coupling between services.</span></span> <span data-ttu-id="19b1f-205">耦合的原因包括共享的数据库架构和严格的通信协议。</span><span class="sxs-lookup"><span data-stu-id="19b1f-205">Causes of coupling include shared database schemas and rigid communication protocols.</span></span>

- <span data-ttu-id="19b1f-206">将身份验证和 SSL 终止等跨领域操作分流到网关。</span><span class="sxs-lookup"><span data-stu-id="19b1f-206">Offload cross-cutting concerns, such as authentication and SSL termination, to the gateway.</span></span>

- <span data-ttu-id="19b1f-207">让网关不必了解域。</span><span class="sxs-lookup"><span data-stu-id="19b1f-207">Keep domain knowledge out of the gateway.</span></span> <span data-ttu-id="19b1f-208">网关应处理和路由客户端请求，而无需了解业务规则或域逻辑。</span><span class="sxs-lookup"><span data-stu-id="19b1f-208">The gateway should handle and route client requests without any knowledge of the business rules or domain logic.</span></span> <span data-ttu-id="19b1f-209">否则，网关会变成一个从属物，从而导致服务之间耦合。</span><span class="sxs-lookup"><span data-stu-id="19b1f-209">Otherwise, the gateway becomes a dependency and can cause coupling between services.</span></span>

- <span data-ttu-id="19b1f-210">服务应具有松散耦合和高功能内聚的特点。</span><span class="sxs-lookup"><span data-stu-id="19b1f-210">Services should have loose coupling and high functional cohesion.</span></span> <span data-ttu-id="19b1f-211">应当将可能会一起更改的函数打包并部署在一起。</span><span class="sxs-lookup"><span data-stu-id="19b1f-211">Functions that are likely to change together should be packaged and deployed together.</span></span> <span data-ttu-id="19b1f-212">如果它们驻留在不同的服务中，这些服务最终会紧密耦合，因为一个服务中的更改将需要更新其他服务。</span><span class="sxs-lookup"><span data-stu-id="19b1f-212">If they reside in separate services, those services end up being tightly coupled, because a change in one service will require updating the other service.</span></span> <span data-ttu-id="19b1f-213">两个服务之间的通信过于频繁可能是紧密耦合和低内聚的征兆。</span><span class="sxs-lookup"><span data-stu-id="19b1f-213">Overly chatty communication between two services may be a symptom of tight coupling and low cohesion.</span></span> 

- <span data-ttu-id="19b1f-214">隔离故障。</span><span class="sxs-lookup"><span data-stu-id="19b1f-214">Isolate failures.</span></span> <span data-ttu-id="19b1f-215">使用复原策略可防止某个服务中的故障级联。</span><span class="sxs-lookup"><span data-stu-id="19b1f-215">Use resiliency strategies to prevent failures within a service from cascading.</span></span> <span data-ttu-id="19b1f-216">请参阅[复原模式][resiliency-patterns]和[设计可复原应用程序][resiliency-overview]。</span><span class="sxs-lookup"><span data-stu-id="19b1f-216">See [Resiliency patterns][resiliency-patterns] and [Designing resilient applications][resiliency-overview].</span></span>

## <a name="microservices-using-azure-container-service"></a><span data-ttu-id="19b1f-217">使用 Azure 容器服务的微服务</span><span class="sxs-lookup"><span data-stu-id="19b1f-217">Microservices using Azure Container Service</span></span> 

<span data-ttu-id="19b1f-218">可以使用 [Azure 容器服务](/azure/container-service/)来配置和预配 Docker 群集。</span><span class="sxs-lookup"><span data-stu-id="19b1f-218">You can use [Azure Container Service](/azure/container-service/) to configure and provision a Docker cluster.</span></span> <span data-ttu-id="19b1f-219">Azure 容器服务支持多种常用容器业务流程协调程序，包括 Kubernetes、DC/OS 和 Docker Swarm。</span><span class="sxs-lookup"><span data-stu-id="19b1f-219">Azure Container Services supports several popular container orchestrators, including Kubernetes, DC/OS, and Docker Swarm.</span></span>

![](./images/microservices-acs.png)
 
<span data-ttu-id="19b1f-220">**公共节点**。</span><span class="sxs-lookup"><span data-stu-id="19b1f-220">**Public nodes**.</span></span> <span data-ttu-id="19b1f-221">这些节点可通过面向公众的负载均衡器访问。</span><span class="sxs-lookup"><span data-stu-id="19b1f-221">These nodes are reachable through a public-facing load balancer.</span></span> <span data-ttu-id="19b1f-222">API 网关就托管在这些节点上。</span><span class="sxs-lookup"><span data-stu-id="19b1f-222">The API gateway is hosted on these nodes.</span></span>

<span data-ttu-id="19b1f-223">**后端节点**。</span><span class="sxs-lookup"><span data-stu-id="19b1f-223">**Backend nodes**.</span></span> <span data-ttu-id="19b1f-224">这些节点运行客户端通过 API 网关访问的服务。</span><span class="sxs-lookup"><span data-stu-id="19b1f-224">These nodes run services that clients reach via the API gateway.</span></span> <span data-ttu-id="19b1f-225">这些节点不直接接收 Internet 流量。</span><span class="sxs-lookup"><span data-stu-id="19b1f-225">These nodes don't receive Internet traffic directly.</span></span> <span data-ttu-id="19b1f-226">后端节点可包含多个 VM 池，每个池都有一个不同的硬件配置文件。</span><span class="sxs-lookup"><span data-stu-id="19b1f-226">The backend nodes might include more than one pool of VMs, each with a different hardware profile.</span></span> <span data-ttu-id="19b1f-227">例如，可为常规计算工作负载、高 CPU 工作负载和高内存工作负载分别创建不同的池。</span><span class="sxs-lookup"><span data-stu-id="19b1f-227">For example, you could create separate pools for general compute workloads, high CPU workloads, and high memory workloads.</span></span> 

<span data-ttu-id="19b1f-228">**管理 VM**。</span><span class="sxs-lookup"><span data-stu-id="19b1f-228">**Management VMs**.</span></span> <span data-ttu-id="19b1f-229">这些 VM 运行容器业务流程协调程序的主节点。</span><span class="sxs-lookup"><span data-stu-id="19b1f-229">These VMs run the master nodes for the container orchestrator.</span></span> 

<span data-ttu-id="19b1f-230">**网络**。</span><span class="sxs-lookup"><span data-stu-id="19b1f-230">**Networking**.</span></span> <span data-ttu-id="19b1f-231">公共节点、后端节点和管理 VM 放置在同一虚拟网络 (VNet) 内的不同子网中。</span><span class="sxs-lookup"><span data-stu-id="19b1f-231">The public nodes, backend nodes, and management VMs are placed in separate subnets within the same virtual network (VNet).</span></span> 

<span data-ttu-id="19b1f-232">**负载均衡器**。</span><span class="sxs-lookup"><span data-stu-id="19b1f-232">**Load balancers**.</span></span>  <span data-ttu-id="19b1f-233">一个面向外部的负载均衡器位于公共节点前面。</span><span class="sxs-lookup"><span data-stu-id="19b1f-233">An externally facing load balancer sits in front of the public nodes.</span></span> <span data-ttu-id="19b1f-234">它将 Internet 请求分布到公共节点。</span><span class="sxs-lookup"><span data-stu-id="19b1f-234">It distributes internet requests to the public nodes.</span></span> <span data-ttu-id="19b1f-235">另一个负载均衡器放在管理 VM 前面，以允许使用 NAT 规则将安全外壳 (ssh) 流量发送到管理 VM。</span><span class="sxs-lookup"><span data-stu-id="19b1f-235">Another load balancer is placed in front of the management VMs, to allow secure shell (ssh) traffic to the management VMs, using NAT rules.</span></span>

<span data-ttu-id="19b1f-236">为了实现可靠性和可伸缩性，会跨多个 VM 复制每个服务。</span><span class="sxs-lookup"><span data-stu-id="19b1f-236">For reliability and scalability, each service is replicated across multiple VMs.</span></span> <span data-ttu-id="19b1f-237">但是，由于服务也相对轻量（相比单一式应用程序），因此，通常会将多个服务打包到一个 VM 中。</span><span class="sxs-lookup"><span data-stu-id="19b1f-237">However, because services are also relatively lightweight (compared with a monolithic application), multiple services are usually packed into a single VM.</span></span> <span data-ttu-id="19b1f-238">密度越高意味着资源利用率越高。</span><span class="sxs-lookup"><span data-stu-id="19b1f-238">Higher density allows better resource utilization.</span></span> <span data-ttu-id="19b1f-239">如果某个服务不使用大量资源，则无需专门使用整个 VM 来运行该服务。</span><span class="sxs-lookup"><span data-stu-id="19b1f-239">If a particular service doesn't use a lot of resources, you don't need to dedicate an entire VM to running that service.</span></span>

<span data-ttu-id="19b1f-240">下图展示运行四个不同服务（由不同的形状表示）的三个节点。</span><span class="sxs-lookup"><span data-stu-id="19b1f-240">The following diagram shows three nodes running four different services (indicated by different shapes).</span></span> <span data-ttu-id="19b1f-241">请注意，每个服务至少有两个实例。</span><span class="sxs-lookup"><span data-stu-id="19b1f-241">Notice that each service has at least two instances.</span></span> 
 
![](./images/microservices-node-density.png)

## <a name="microservices-using-azure-service-fabric"></a><span data-ttu-id="19b1f-242">使用 Azure Service Fabric 的微服务</span><span class="sxs-lookup"><span data-stu-id="19b1f-242">Microservices using Azure Service Fabric</span></span>

<span data-ttu-id="19b1f-243">下图展示使用 [Azure Service Fabric](/azure/service-fabric/) 的微服务体系结构。</span><span class="sxs-lookup"><span data-stu-id="19b1f-243">The following diagram shows a microservices architecture using [Azure Service Fabric](/azure/service-fabric/).</span></span>

![](./images/service-fabric.png)

<span data-ttu-id="19b1f-244">Service Fabric 群集部署到一个或多个 VM 规模集。</span><span class="sxs-lookup"><span data-stu-id="19b1f-244">The Service Fabric cluster is deployed to one or more VM scale sets.</span></span> <span data-ttu-id="19b1f-245">你的群集中可能有多个 VM 规模集，以便包含各种 VM 类型。</span><span class="sxs-lookup"><span data-stu-id="19b1f-245">You might have more than one VM scale set in the cluster, in order to have a mix of VM types.</span></span> <span data-ttu-id="19b1f-246">API 网关放在 Service Fabric 群集前面，由一个外部负载均衡器接收客户端请求。</span><span class="sxs-lookup"><span data-stu-id="19b1f-246">An API Gateway is placed in front of the Service Fabric cluster, with an external load balancer to receive client requests.</span></span>

<span data-ttu-id="19b1f-247">Service Fabric 运行时执行群集管理，包括服务放置、节点故障转移和运行状况监视。</span><span class="sxs-lookup"><span data-stu-id="19b1f-247">The Service Fabric runtime performs cluster management, including service placement, node failover, and health monitoring.</span></span> <span data-ttu-id="19b1f-248">该运行时部署于群集节点自身。</span><span class="sxs-lookup"><span data-stu-id="19b1f-248">The runtime is deployed on the cluster nodes themselves.</span></span> <span data-ttu-id="19b1f-249">这里没有一组单独的群集管理 VM。</span><span class="sxs-lookup"><span data-stu-id="19b1f-249">There isn't a separate set of cluster management VMs.</span></span>

<span data-ttu-id="19b1f-250">服务使用 Service Fabric 内置的反向代理相互通信。</span><span class="sxs-lookup"><span data-stu-id="19b1f-250">Services communicate with each other using the reverse proxy that is built into Service Fabric.</span></span> <span data-ttu-id="19b1f-251">Service Fabric 提供发现服务，可用于解析指定服务的终结点。</span><span class="sxs-lookup"><span data-stu-id="19b1f-251">Service Fabric provides a discovery service that can resolve the endpoint for a named service.</span></span>


<!-- links -->

[resiliency-overview]: ../../resiliency/index.md
[resiliency-patterns]: ../../patterns/category/resiliency.md



