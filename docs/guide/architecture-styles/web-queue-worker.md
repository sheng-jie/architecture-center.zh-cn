---
title: Web 队列辅助角色体系结构样式
description: 介绍 Azure 上的 Web 队列辅助角色的体系结构的优点、难题和最佳做法
author: MikeWasson
ms.openlocfilehash: 545472e71ffcd43717ad24af0dc9218a221ca910
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="web-queue-worker-architecture-style"></a><span data-ttu-id="da0eb-103">Web 队列辅助角色体系结构样式</span><span class="sxs-lookup"><span data-stu-id="da0eb-103">Web-Queue-Worker architecture style</span></span>

<span data-ttu-id="da0eb-104">该体系结构的核心组件是处理客户端请求的 Web 前端和执行资源密集型任务、长时间运行的工作流或批处理作业的辅助角色。</span><span class="sxs-lookup"><span data-stu-id="da0eb-104">The core components of this architecture are a **web front end** that serves client requests, and a **worker** that performs resource-intensive tasks, long-running workflows, or batch jobs.</span></span>  <span data-ttu-id="da0eb-105">Web 前端通过消息队列与辅助角色进行通信。</span><span class="sxs-lookup"><span data-stu-id="da0eb-105">The web front end communicates with the worker through a **message queue**.</span></span>  

![](./images/web-queue-worker-logical.svg)

<span data-ttu-id="da0eb-106">通常合并到该体系结构中的其他组件包括：</span><span class="sxs-lookup"><span data-stu-id="da0eb-106">Other components that are commonly incorporated into this architecture include:</span></span>

- <span data-ttu-id="da0eb-107">一个或多个数据库。</span><span class="sxs-lookup"><span data-stu-id="da0eb-107">One or more databases.</span></span> 
- <span data-ttu-id="da0eb-108">存储来自数据库的值以便快速读取的缓存。</span><span class="sxs-lookup"><span data-stu-id="da0eb-108">A cache to store values from the database for quick reads.</span></span>
- <span data-ttu-id="da0eb-109">提供静态内容的 CDN</span><span class="sxs-lookup"><span data-stu-id="da0eb-109">CDN to serve static content</span></span>
- <span data-ttu-id="da0eb-110">远程服务，如电子邮件或 SMS 服务。</span><span class="sxs-lookup"><span data-stu-id="da0eb-110">Remote services, such as email or SMS service.</span></span> <span data-ttu-id="da0eb-111">这些通常由第三方提供。</span><span class="sxs-lookup"><span data-stu-id="da0eb-111">Often these are provided by third parties.</span></span>
- <span data-ttu-id="da0eb-112">用于身份验证的标识提供程序。</span><span class="sxs-lookup"><span data-stu-id="da0eb-112">Identity provider for authentication.</span></span>

<span data-ttu-id="da0eb-113">Web 和辅助角色都无状态。</span><span class="sxs-lookup"><span data-stu-id="da0eb-113">The web and worker are both stateless.</span></span> <span data-ttu-id="da0eb-114">会话状态可以存储在分布式缓存中。</span><span class="sxs-lookup"><span data-stu-id="da0eb-114">Session state can be stored in a distributed cache.</span></span> <span data-ttu-id="da0eb-115">任何长时间运行的工作都由辅助角色异步完成。</span><span class="sxs-lookup"><span data-stu-id="da0eb-115">Any long-running work is done asynchronously by the worker.</span></span> <span data-ttu-id="da0eb-116">辅助角色可以由队列上的消息触发，或者在用于批处理的计划上运行。</span><span class="sxs-lookup"><span data-stu-id="da0eb-116">The worker can be triggered by messages on the queue, or run on a schedule for batch processing.</span></span> <span data-ttu-id="da0eb-117">辅助角色是一个可选组件。</span><span class="sxs-lookup"><span data-stu-id="da0eb-117">The worker is an optional component.</span></span> <span data-ttu-id="da0eb-118">如果没有任何长时间运行的操作，则可以省略辅助角色。</span><span class="sxs-lookup"><span data-stu-id="da0eb-118">If there are no long-running operations, the worker can be omitted.</span></span>  

<span data-ttu-id="da0eb-119">前端可能包括 Web API。</span><span class="sxs-lookup"><span data-stu-id="da0eb-119">The front end might consist of a web API.</span></span> <span data-ttu-id="da0eb-120">在客户端上，Web API 可以由进行 AJAX 调用的单页应用程序或者本机客户端应用程序使用。</span><span class="sxs-lookup"><span data-stu-id="da0eb-120">On the client side, the web API can be consumed by a single-page application that makes AJAX calls, or by a native client application.</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="da0eb-121">此体系结构适用的情况</span><span class="sxs-lookup"><span data-stu-id="da0eb-121">When to use this architecture</span></span>

<span data-ttu-id="da0eb-122">通常使用托管的计算服务（Azure App Service 或 Azure 云服务）实施 Web 队列辅助角色体系结构。</span><span class="sxs-lookup"><span data-stu-id="da0eb-122">The Web-Queue-Worker architecture is typically implemented using managed compute services, either Azure App Service or Azure Cloud Services.</span></span> 

<span data-ttu-id="da0eb-123">请对以下情况考虑使用此体系结构样式：</span><span class="sxs-lookup"><span data-stu-id="da0eb-123">Consider this architecture style for:</span></span>

- <span data-ttu-id="da0eb-124">具有相对简单的域的应用程序。</span><span class="sxs-lookup"><span data-stu-id="da0eb-124">Applications with a relatively simple domain.</span></span>
- <span data-ttu-id="da0eb-125">具有某些长时间的工作流或批处理操作的应用程序。</span><span class="sxs-lookup"><span data-stu-id="da0eb-125">Applications with some long-running workflows or batch operations.</span></span>
- <span data-ttu-id="da0eb-126">想要使用托管的服务而不是服务架构 (IaaS) 时。</span><span class="sxs-lookup"><span data-stu-id="da0eb-126">When you want to use managed services, rather than infrastructure as a service (IaaS).</span></span>

## <a name="benefits"></a><span data-ttu-id="da0eb-127">优点</span><span class="sxs-lookup"><span data-stu-id="da0eb-127">Benefits</span></span>

- <span data-ttu-id="da0eb-128">体系结构相对简单，易于理解。</span><span class="sxs-lookup"><span data-stu-id="da0eb-128">Relatively simple architecture that is easy to understand.</span></span>
- <span data-ttu-id="da0eb-129">易于部署和管理。</span><span class="sxs-lookup"><span data-stu-id="da0eb-129">Easy to deploy and manage.</span></span>
- <span data-ttu-id="da0eb-130">清晰的关注点分离。</span><span class="sxs-lookup"><span data-stu-id="da0eb-130">Clear separation of concerns.</span></span>
- <span data-ttu-id="da0eb-131">使用异步消息将前端从辅助角色中分离。</span><span class="sxs-lookup"><span data-stu-id="da0eb-131">The front end is decoupled from the worker using asynchronous messaging.</span></span>
- <span data-ttu-id="da0eb-132">前端和辅助角色可以独立缩放。</span><span class="sxs-lookup"><span data-stu-id="da0eb-132">The front end and the worker can be scaled independently.</span></span>

## <a name="challenges"></a><span data-ttu-id="da0eb-133">挑战</span><span class="sxs-lookup"><span data-stu-id="da0eb-133">Challenges</span></span>

- <span data-ttu-id="da0eb-134">如果没有经过仔细设计，前端和辅助角色可能变成难以维护和更新的庞大组件。</span><span class="sxs-lookup"><span data-stu-id="da0eb-134">Without careful design, the front end and the worker can become large, monolithic components that are difficult to maintain and update.</span></span>
- <span data-ttu-id="da0eb-135">如果前端和辅助角色共享数据架构或代码模块，可能会有隐藏依赖项。</span><span class="sxs-lookup"><span data-stu-id="da0eb-135">There may be hidden dependencies, if the front end and worker share data schemas or code modules.</span></span> 

## <a name="best-practices"></a><span data-ttu-id="da0eb-136">最佳实践</span><span class="sxs-lookup"><span data-stu-id="da0eb-136">Best practices</span></span>

- <span data-ttu-id="da0eb-137">向客户端公开经过良好设计的 API。</span><span class="sxs-lookup"><span data-stu-id="da0eb-137">Expose a well-designed API to the client.</span></span> <span data-ttu-id="da0eb-138">请参阅 [API 设计最佳做法][api-design]。</span><span class="sxs-lookup"><span data-stu-id="da0eb-138">See [API design best practices][api-design].</span></span>
- <span data-ttu-id="da0eb-139">自动缩放以处理负载的更改。</span><span class="sxs-lookup"><span data-stu-id="da0eb-139">Autoscale to handle changes in load.</span></span> <span data-ttu-id="da0eb-140">请参阅[自动缩放的最佳做法][autoscaling]。</span><span class="sxs-lookup"><span data-stu-id="da0eb-140">See [Autoscaling best practices][autoscaling].</span></span>
- <span data-ttu-id="da0eb-141">缓存半静态数据。</span><span class="sxs-lookup"><span data-stu-id="da0eb-141">Cache semi-static data.</span></span> <span data-ttu-id="da0eb-142">请参阅[缓存的最佳做法][caching]。</span><span class="sxs-lookup"><span data-stu-id="da0eb-142">See [Caching best practices][caching].</span></span>
- <span data-ttu-id="da0eb-143">使用 CDN 来托管静态内容。</span><span class="sxs-lookup"><span data-stu-id="da0eb-143">Use a CDN to host static content.</span></span> <span data-ttu-id="da0eb-144">请参阅 [CDN 最佳做法][cdn]。</span><span class="sxs-lookup"><span data-stu-id="da0eb-144">See [CDN best practices][cdn].</span></span>
- <span data-ttu-id="da0eb-145">适当时使用混合持久性。</span><span class="sxs-lookup"><span data-stu-id="da0eb-145">Use polyglot persistence when appropriate.</span></span> <span data-ttu-id="da0eb-146">请参阅[为作业使用最佳数据存储][polyglot]。</span><span class="sxs-lookup"><span data-stu-id="da0eb-146">See [Use the best data store for the job][polyglot].</span></span>
- <span data-ttu-id="da0eb-147">分区数据，可提高可伸缩性、减少争用以及优化性能。</span><span class="sxs-lookup"><span data-stu-id="da0eb-147">Partition data to improve scalability, reduce contention, and optimize performance.</span></span> <span data-ttu-id="da0eb-148">请参阅[数据分区最佳做法][data-partition]。</span><span class="sxs-lookup"><span data-stu-id="da0eb-148">See [Data partitioning best practices][data-partition].</span></span>


## <a name="web-queue-worker-on-azure-app-service"></a><span data-ttu-id="da0eb-149">Azure App Service 上的 Web 队列辅助角色</span><span class="sxs-lookup"><span data-stu-id="da0eb-149">Web-Queue-Worker on Azure App Service</span></span>

<span data-ttu-id="da0eb-150">本部分介绍使用 Azure App Service 的建议的 Web 队列辅助角色体系结构。</span><span class="sxs-lookup"><span data-stu-id="da0eb-150">This section describes a recommended Web-Queue-Worker architecture that uses Azure App Service.</span></span> 

![](./images/web-queue-worker-physical.png)

<span data-ttu-id="da0eb-151">Azure App Service Web 应用作为前端实现，辅助角色作为 Web 作业实现。</span><span class="sxs-lookup"><span data-stu-id="da0eb-151">The front end is implemented as an Azure App Service web app, and the worker is implemented as a WebJob.</span></span> <span data-ttu-id="da0eb-152">Web 应用和 Web 作业都与提供 VM 实例的应用服务计划相关联。</span><span class="sxs-lookup"><span data-stu-id="da0eb-152">The web app and the WebJob are both associated with an App Service plan that provides the VM instances.</span></span> 

<span data-ttu-id="da0eb-153">可以为消息队列使用 Azure 服务总线或 Azure 存储队列。</span><span class="sxs-lookup"><span data-stu-id="da0eb-153">You can use either Azure Service Bus or Azure Storage queues for the message queue.</span></span> <span data-ttu-id="da0eb-154">（该图显示了 Azure 存储队列。）</span><span class="sxs-lookup"><span data-stu-id="da0eb-154">(The diagram shows an Azure Storage queue.)</span></span>

<span data-ttu-id="da0eb-155">Azure Redis 缓存存储会话状态和需要低延迟访问的其他数据。</span><span class="sxs-lookup"><span data-stu-id="da0eb-155">Azure Redis Cache stores session state and other data that needs low latency access.</span></span>

<span data-ttu-id="da0eb-156">Azure CDN 用于缓存静态内容，如映像、CSS 或 HTML。</span><span class="sxs-lookup"><span data-stu-id="da0eb-156">Azure CDN is used to cache static content such as images, CSS, or HTML.</span></span>

<span data-ttu-id="da0eb-157">对于存储，请选择最符合应用程序需要的存储技术。</span><span class="sxs-lookup"><span data-stu-id="da0eb-157">For storage, choose the storage technologies that best fit the needs of the application.</span></span> <span data-ttu-id="da0eb-158">可能使用多个存储技术（混合持久性）。</span><span class="sxs-lookup"><span data-stu-id="da0eb-158">You might use multiple storage technologies (polyglot persistence).</span></span> <span data-ttu-id="da0eb-159">为了说明这一点，该图显示了 Azure SQL 数据库和 Azure Cosmos DB。</span><span class="sxs-lookup"><span data-stu-id="da0eb-159">To illustrate this idea, the diagram shows Azure SQL Database and Azure Cosmos DB.</span></span>  

<span data-ttu-id="da0eb-160">有关详细信息，请参阅[应用服务 Web 应用程序参考结构][scalable-web-app]。</span><span class="sxs-lookup"><span data-stu-id="da0eb-160">For more details, see [App Service web application reference architecture][scalable-web-app].</span></span>

### <a name="additional-considerations"></a><span data-ttu-id="da0eb-161">其他注意事项</span><span class="sxs-lookup"><span data-stu-id="da0eb-161">Additional considerations</span></span>

- <span data-ttu-id="da0eb-162">并不是每个事务都必须经过队列和辅助角色才能存储。</span><span class="sxs-lookup"><span data-stu-id="da0eb-162">Not every transaction has to go through the queue and worker to storage.</span></span> <span data-ttu-id="da0eb-163">Web 前端可以直接执行简单的读取/写入操作。</span><span class="sxs-lookup"><span data-stu-id="da0eb-163">The web front end can perform simple read/write operations directly.</span></span> <span data-ttu-id="da0eb-164">辅助角色设计用于资源密集型任务或长时间运行的工作流。</span><span class="sxs-lookup"><span data-stu-id="da0eb-164">Workers are designed for resource-intensive tasks or long-running workflows.</span></span> <span data-ttu-id="da0eb-165">在某些情况下，可能根本不需要辅助角色。</span><span class="sxs-lookup"><span data-stu-id="da0eb-165">In some cases, you might not need a worker at all.</span></span>

- <span data-ttu-id="da0eb-166">使用应用服务的内置自动缩放功能来扩大 VM 实例数。</span><span class="sxs-lookup"><span data-stu-id="da0eb-166">Use the built-in autoscale feature of App Service to scale out the number of VM instances.</span></span> <span data-ttu-id="da0eb-167">如果应用程序上的负载遵循可预测的模式，请使用基于计划的自动缩放。</span><span class="sxs-lookup"><span data-stu-id="da0eb-167">If the load on the application follows predictable patterns, use schedule-based autoscale.</span></span> <span data-ttu-id="da0eb-168">如果负载是不可预测的，请使用基于指标的自动缩放规则。</span><span class="sxs-lookup"><span data-stu-id="da0eb-168">If the load is unpredictable, use metrics-based autoscaling rules.</span></span>      

- <span data-ttu-id="da0eb-169">请考虑将 Web 应用和 Web 作业放入单独的应用服务计划中。</span><span class="sxs-lookup"><span data-stu-id="da0eb-169">Consider putting the web app and the WebJob into separate App Service plans.</span></span> <span data-ttu-id="da0eb-170">这样，它们就托管在单独的 VM 实例上，并可以独立缩放。</span><span class="sxs-lookup"><span data-stu-id="da0eb-170">That way, they are hosted on separate VM instances and can be scaled independently.</span></span> 

- <span data-ttu-id="da0eb-171">使用单独的应用服务计划进行生产和测试。</span><span class="sxs-lookup"><span data-stu-id="da0eb-171">Use separate App Service plans for production and testing.</span></span> <span data-ttu-id="da0eb-172">否则，如果使用同一计划进行生产和测试，则说明测试正在生产 VM 上运行。</span><span class="sxs-lookup"><span data-stu-id="da0eb-172">Otherwise, if you use the same plan for production and testing, it means your tests are running on your production VMs.</span></span>

- <span data-ttu-id="da0eb-173">使用部署槽位以管理部署。</span><span class="sxs-lookup"><span data-stu-id="da0eb-173">Use deployment slots to manage deployments.</span></span> <span data-ttu-id="da0eb-174">这使你可以将更新的版本部署到过渡槽，然后切换到新版本。</span><span class="sxs-lookup"><span data-stu-id="da0eb-174">This lets you to deploy an updated version to a staging slot, then swap over to the new version.</span></span> <span data-ttu-id="da0eb-175">如果更新时出现问题，这也使你可以切换回上一版本。</span><span class="sxs-lookup"><span data-stu-id="da0eb-175">It also lets you swap back to the previous version, if there was a problem with the update.</span></span>

<!-- links -->

[api-design]: ../../best-practices/api-design.md
[autoscaling]: ../../best-practices/auto-scaling.md
[caching]: ../../best-practices/caching.md
[cdn]: ../../best-practices/cdn.md
[data-partition]: ../../best-practices/data-partitioning.md
[polyglot]: ../design-principles/use-the-best-data-store.md
[scalable-web-app]: ../../reference-architectures/app-service-web-app/scalable-web-app.md