---
title: 将 Azure 云服务应用程序迁移到 Azure Service Fabric
description: 如何将 Azure 云服务中的应用程序迁移到 Azure Service Fabric。
author: MikeWasson
ms.date: 04/27/2017
ms.openlocfilehash: ce9c138a6b093fb7f0329c619c75bd4f4aacc2e7
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/23/2018
---
# <a name="migrate-an-azure-cloud-services-application-to-azure-service-fabric"></a><span data-ttu-id="66f61-103">将 Azure 云服务应用程序迁移到 Azure Service Fabric</span><span class="sxs-lookup"><span data-stu-id="66f61-103">Migrate an Azure Cloud Services application to Azure Service Fabric</span></span> 

<span data-ttu-id="66f61-104">[![GitHub](../_images/github.png) 示例代码][sample-code]</span><span class="sxs-lookup"><span data-stu-id="66f61-104">[![GitHub](../_images/github.png) Sample code][sample-code]</span></span>

<span data-ttu-id="66f61-105">本文说明如何将 Azure 云服务中的应用程序迁移到 Azure Service Fabric。</span><span class="sxs-lookup"><span data-stu-id="66f61-105">This article describes migrating an application from Azure Cloud Services to Azure Service Fabric.</span></span> <span data-ttu-id="66f61-106">它重点介绍了一些体系结构决策和建议的做法。</span><span class="sxs-lookup"><span data-stu-id="66f61-106">It focuses on architectural decisions and recommended practices.</span></span> 

<span data-ttu-id="66f61-107">对于本项目，我们从一款名为 Surveys 的云服务应用程序开始，并将其移植到 Service Fabric 中。</span><span class="sxs-lookup"><span data-stu-id="66f61-107">For this project, we started with a Cloud Services application called Surveys and ported it to Service Fabric.</span></span> <span data-ttu-id="66f61-108">目标是在迁移该应用程序时尽可能少做更改。</span><span class="sxs-lookup"><span data-stu-id="66f61-108">The goal was to migrate the application with as few changes as possible.</span></span> <span data-ttu-id="66f61-109">在后续文章中，我们将采用一种微服务体系结构，针对 Service Fabric 优化该应用程序。</span><span class="sxs-lookup"><span data-stu-id="66f61-109">In a later article, we will optimize the application for Service Fabric by adopting a microservices architecture.</span></span>

<span data-ttu-id="66f61-110">阅读本文之前，了解一下 Service Fabric 和微服务体系结构的基础知识通常大有裨益。</span><span class="sxs-lookup"><span data-stu-id="66f61-110">Before reading this article, it will be useful to understand the basics of Service Fabric and microservices architectures in general.</span></span> <span data-ttu-id="66f61-111">请参阅以下文章：</span><span class="sxs-lookup"><span data-stu-id="66f61-111">See the following articles:</span></span>

- <span data-ttu-id="66f61-112">[Azure Service Fabric 概述][sf-overview]</span><span class="sxs-lookup"><span data-stu-id="66f61-112">[Overview of Azure Service Fabric][sf-overview]</span></span>
- <span data-ttu-id="66f61-113">[为什么通过微服务的方法构建应用程序？][sf-why-microservices]</span><span class="sxs-lookup"><span data-stu-id="66f61-113">[Why a microservices approach to building applications?][sf-why-microservices]</span></span>


## <a name="about-the-surveys-application"></a><span data-ttu-id="66f61-114">关于 Surveys 应用程序</span><span class="sxs-lookup"><span data-stu-id="66f61-114">About the Surveys application</span></span>

<span data-ttu-id="66f61-115">2012 年，模式和实践小组为[为云开发多租户应用程序][tailspin-book]一书创建了一个名为 Surveys 的应用程序。</span><span class="sxs-lookup"><span data-stu-id="66f61-115">In 2012, the patterns & practices group created an application called Surveys, for a book called [Developing Multi-tenant Applications for the Cloud][tailspin-book].</span></span> <span data-ttu-id="66f61-116">书中描述了一个设计并实现 Surveys 应用程序的虚构公司 Tailspin。</span><span class="sxs-lookup"><span data-stu-id="66f61-116">The book describes a fictitious company named Tailspin that designs and implements the Surveys application.</span></span>

<span data-ttu-id="66f61-117">Surveys 是一款多租户应用程序，允许客户创建问卷调查。</span><span class="sxs-lookup"><span data-stu-id="66f61-117">Surveys is a multitenant application that allows customers to create surveys.</span></span> <span data-ttu-id="66f61-118">客户注册该应用程序后，客户组织中的成员可以创建和发布问卷调查，并收集结果进行分析。</span><span class="sxs-lookup"><span data-stu-id="66f61-118">After a customer signs up for the application,  members of the customer's organization can create and publish surveys, and collect the results for analysis.</span></span> <span data-ttu-id="66f61-119">该应用程序包括一个公共网站，人们可在网站上参与问卷调查。</span><span class="sxs-lookup"><span data-stu-id="66f61-119">The application includes a public website where people can take a survey.</span></span> <span data-ttu-id="66f61-120">请从[此处][tailspin-scenario]阅读有关原始 Tailspin 方案的详细信息。</span><span class="sxs-lookup"><span data-stu-id="66f61-120">Read more about the original Tailspin scenario [here][tailspin-scenario].</span></span>

<span data-ttu-id="66f61-121">现在，Tailspin 想使用 Azure 上运行的 Service Fabric，将 Surveys 应用程序迁移到微服务体系结构中。</span><span class="sxs-lookup"><span data-stu-id="66f61-121">Now Tailspin wants to move the Surveys application to a microservices architecture, using Service Fabric running on Azure.</span></span> <span data-ttu-id="66f61-122">由于该应用程序已部署为云服务应用程序，因此 Tailspin 采用一种多阶段方法：</span><span class="sxs-lookup"><span data-stu-id="66f61-122">Because the application is already deployed as a Cloud Services application, Tailspin adopts a multi-phase approach:</span></span>

1.  <span data-ttu-id="66f61-123">将云服务移植到 Service Fabric，同时尽量减少对应用程序的更改。</span><span class="sxs-lookup"><span data-stu-id="66f61-123">Port the cloud services to Service Fabric, while minimizing changes to the application.</span></span>
2.  <span data-ttu-id="66f61-124">通过迁移到微服务体系结构，针对 Service Fabric 优化应用程序。</span><span class="sxs-lookup"><span data-stu-id="66f61-124">Optimize the application for Service Fabric, by moving to a microservices architecture.</span></span>

<span data-ttu-id="66f61-125">本文介绍第一个阶段。</span><span class="sxs-lookup"><span data-stu-id="66f61-125">This article describes the first phase.</span></span> <span data-ttu-id="66f61-126">后续文章将介绍第二个阶段。</span><span class="sxs-lookup"><span data-stu-id="66f61-126">A later article will describe the second phase.</span></span> <span data-ttu-id="66f61-127">在实际项目中，这两个阶段有可能重叠。</span><span class="sxs-lookup"><span data-stu-id="66f61-127">In a real-world project, it's likely that both stages would overlap.</span></span> <span data-ttu-id="66f61-128">移植到 Service Fabric 的同时，你也会开始将应用程序重新构建到微服务中。</span><span class="sxs-lookup"><span data-stu-id="66f61-128">While porting to Service Fabric, you would also start to re-architect the application into micro-services.</span></span> <span data-ttu-id="66f61-129">之后，可能会进一步优化体系结构，可能将粗粒度的服务划分为粒度较小的服务。</span><span class="sxs-lookup"><span data-stu-id="66f61-129">Later you might refine the architecture further, perhaps dividing coarse-grained services into smaller services.</span></span>  

<span data-ttu-id="66f61-130">[GitHub][sample-code] 上提供了应用程序代码。</span><span class="sxs-lookup"><span data-stu-id="66f61-130">The application code is available on [GitHub][sample-code].</span></span> <span data-ttu-id="66f61-131">此存储库包含云服务应用程序和 Service Fabric 版本。</span><span class="sxs-lookup"><span data-stu-id="66f61-131">This repo contains both the Cloud Services application and the Service Fabric version.</span></span> 

> <span data-ttu-id="66f61-132">该云服务是*开发多租户应用程序*一书中原始应用程序的更新版本。</span><span class="sxs-lookup"><span data-stu-id="66f61-132">The cloud service is an updated version of the original application from the *Developing Multi-tenant Applications* book.</span></span>

## <a name="why-microservices"></a><span data-ttu-id="66f61-133">为何使用微服务？</span><span class="sxs-lookup"><span data-stu-id="66f61-133">Why Microservices?</span></span>

<span data-ttu-id="66f61-134">对微服务的深入讨论不在本文范围内，但下面介绍了 Tailspin 希望通过迁移到微服务体系结构获得的一些优势：</span><span class="sxs-lookup"><span data-stu-id="66f61-134">An in-depth discussion of microservices is beyond scope of this article, but here are some of the benefits that Tailspin hopes to get by moving to a microservices architecture:</span></span>

- <span data-ttu-id="66f61-135">**应用程序升级**。</span><span class="sxs-lookup"><span data-stu-id="66f61-135">**Application upgrades**.</span></span> <span data-ttu-id="66f61-136">服务可独立部署，因此，你可以使用递增的方法对应用程序升级。</span><span class="sxs-lookup"><span data-stu-id="66f61-136">Services can be deployed independently, so you can take an incremental approach to upgrading an application.</span></span>
- <span data-ttu-id="66f61-137">**复原能力和故障隔离**。</span><span class="sxs-lookup"><span data-stu-id="66f61-137">**Resiliency and fault isolation**.</span></span> <span data-ttu-id="66f61-138">如果某个服务出现故障，其他服务将继续运行。</span><span class="sxs-lookup"><span data-stu-id="66f61-138">If a service fails, other services continue to run.</span></span>
- <span data-ttu-id="66f61-139">**可伸缩性**。</span><span class="sxs-lookup"><span data-stu-id="66f61-139">**Scalability**.</span></span> <span data-ttu-id="66f61-140">服务可独立缩放。</span><span class="sxs-lookup"><span data-stu-id="66f61-140">Services can be scaled independently.</span></span>
- <span data-ttu-id="66f61-141">**灵活性**。</span><span class="sxs-lookup"><span data-stu-id="66f61-141">**Flexibility**.</span></span> <span data-ttu-id="66f61-142">服务根据业务方案而非技术堆栈设计而成，因此可以更轻松地将服务迁移到新的技术、框架或数据存储。</span><span class="sxs-lookup"><span data-stu-id="66f61-142">Services are designed around business scenarios, not technology stacks, making it easier to migrate services to new technologies, frameworks, or data stores.</span></span>
- <span data-ttu-id="66f61-143">**敏捷开发**。</span><span class="sxs-lookup"><span data-stu-id="66f61-143">**Agile development**.</span></span> <span data-ttu-id="66f61-144">相比单一式应用程序，单个服务的代码更少，从而使基本代码更易理解、推理和测试。</span><span class="sxs-lookup"><span data-stu-id="66f61-144">Individual services have less code than a monolithic application, making the code base easier to understand, reason about, and test.</span></span>
- <span data-ttu-id="66f61-145">**小型专属团队**。</span><span class="sxs-lookup"><span data-stu-id="66f61-145">**Small, focused teams**.</span></span> <span data-ttu-id="66f61-146">由于应用程序被分解为多个小型服务，因此每个服务可由一个小型专属团队构建。</span><span class="sxs-lookup"><span data-stu-id="66f61-146">Because the application is broken down into many small services, each service can be built by a small focused team.</span></span>

## <a name="why-service-fabric"></a><span data-ttu-id="66f61-147">为何使用 Service Fabric？</span><span class="sxs-lookup"><span data-stu-id="66f61-147">Why Service Fabric?</span></span>
      
<span data-ttu-id="66f61-148">Service Fabric 非常适合微服务体系结构，因为 Service Fabric 内置了分布式系统所需的大部分功能，包括：</span><span class="sxs-lookup"><span data-stu-id="66f61-148">Service Fabric is a good fit for a microservices architecture, because most of the features needed in a distributed system are built into Service Fabric, including:</span></span>

- <span data-ttu-id="66f61-149">**群集管理**。</span><span class="sxs-lookup"><span data-stu-id="66f61-149">**Cluster management**.</span></span> <span data-ttu-id="66f61-150">Service Fabric 自动处理节点故障转移、运行状况监视和其他群集管理功能。</span><span class="sxs-lookup"><span data-stu-id="66f61-150">Service Fabric automatically handles node failover, health monitoring, and other cluster management functions.</span></span>
- <span data-ttu-id="66f61-151">**水平扩展**。</span><span class="sxs-lookup"><span data-stu-id="66f61-151">**Horizontal scaling**.</span></span> <span data-ttu-id="66f61-152">将节点添加到 Service Fabric 群集时，应用程序会自动缩放，因为服务分布在各个新节点中。</span><span class="sxs-lookup"><span data-stu-id="66f61-152">When you add nodes to a Service Fabric cluster, the application automatically scales, as services are distributed across the new nodes.</span></span>
- <span data-ttu-id="66f61-153">**服务发现**。</span><span class="sxs-lookup"><span data-stu-id="66f61-153">**Service discovery**.</span></span> <span data-ttu-id="66f61-154">Service Fabric 提供发现服务，可用于解析指定服务的终结点。</span><span class="sxs-lookup"><span data-stu-id="66f61-154">Service Fabric provides a discovery service that can resolve the endpoint for a named service.</span></span>
- <span data-ttu-id="66f61-155">**无状态和有状态服务**。</span><span class="sxs-lookup"><span data-stu-id="66f61-155">**Stateless and stateful services**.</span></span> <span data-ttu-id="66f61-156">有状态服务使用[可靠集合][sf-reliable-collections]，可取代缓存或队列并且可分区。</span><span class="sxs-lookup"><span data-stu-id="66f61-156">Stateful services use [reliable collections][sf-reliable-collections], which can take the place of a cache or queue, and can be partitioned.</span></span>
- <span data-ttu-id="66f61-157">**应用程序生命周期管理**。</span><span class="sxs-lookup"><span data-stu-id="66f61-157">**Application lifecycle management**.</span></span> <span data-ttu-id="66f61-158">服务可独立升级，无需关闭应用程序。</span><span class="sxs-lookup"><span data-stu-id="66f61-158">Services can be upgraded independently and without application downtime.</span></span>
- <span data-ttu-id="66f61-159">跨一组计算机的**服务业务流程**。</span><span class="sxs-lookup"><span data-stu-id="66f61-159">**Service orchestration** across a cluster of machines.</span></span>
- <span data-ttu-id="66f61-160">**更高的密度**，优化了资源消耗。</span><span class="sxs-lookup"><span data-stu-id="66f61-160">**Higher density** for optimizing resource consumption.</span></span> <span data-ttu-id="66f61-161">一个节点可以托管多个服务。</span><span class="sxs-lookup"><span data-stu-id="66f61-161">A single node can host multiple services.</span></span>

<span data-ttu-id="66f61-162">Service Fabric 可供各种 Microsoft 服务使用，包括 Azure SQL 数据库、Cosmos DB、Azure 事件中心等等，是一款用于构建分布式云应用程序的广受赞誉的平台。</span><span class="sxs-lookup"><span data-stu-id="66f61-162">Service Fabric is used by various Microsoft services, including Azure SQL Database, Cosmos DB, Azure Event Hubs, and others, making it a proven platform for building distributed cloud applications.</span></span> 

## <a name="comparing-cloud-services-with-service-fabric"></a><span data-ttu-id="66f61-163">比较云服务和 Service Fabric</span><span class="sxs-lookup"><span data-stu-id="66f61-163">Comparing Cloud Services with Service Fabric</span></span>

<span data-ttu-id="66f61-164">下表总结了云服务应用程序与 Service Fabric 应用程序之间的一些重要差异。</span><span class="sxs-lookup"><span data-stu-id="66f61-164">The following table summarizes some of the important differences between Cloud Services and Service Fabric applications.</span></span> <span data-ttu-id="66f61-165">有关更深入的讨论，请参阅[迁移应用程序之前了解云服务与 Service Fabric 之间的差异][sf-compare-cloud-services]。</span><span class="sxs-lookup"><span data-stu-id="66f61-165">For a more in-depth discussion, see [Learn about the differences between Cloud Services and Service Fabric before migrating applications][sf-compare-cloud-services].</span></span>

|        | <span data-ttu-id="66f61-166">云服务</span><span class="sxs-lookup"><span data-stu-id="66f61-166">Cloud Services</span></span> | <span data-ttu-id="66f61-167">Service Fabric</span><span class="sxs-lookup"><span data-stu-id="66f61-167">Service Fabric</span></span> |
|--------|---------------|----------------|
| <span data-ttu-id="66f61-168">应用程序组合</span><span class="sxs-lookup"><span data-stu-id="66f61-168">Application composition</span></span> | <span data-ttu-id="66f61-169">角色</span><span class="sxs-lookup"><span data-stu-id="66f61-169">Roles</span></span>| <span data-ttu-id="66f61-170">服务</span><span class="sxs-lookup"><span data-stu-id="66f61-170">Services</span></span> |
| <span data-ttu-id="66f61-171">密度</span><span class="sxs-lookup"><span data-stu-id="66f61-171">Density</span></span> |<span data-ttu-id="66f61-172">每个 VM 一个角色实例</span><span class="sxs-lookup"><span data-stu-id="66f61-172">One role instance per VM</span></span> | <span data-ttu-id="66f61-173">单个节点上多个服务</span><span class="sxs-lookup"><span data-stu-id="66f61-173">Multiple services in a single node</span></span> |
| <span data-ttu-id="66f61-174">最小节点数</span><span class="sxs-lookup"><span data-stu-id="66f61-174">Minimum number of nodes</span></span> | <span data-ttu-id="66f61-175">每个角色 2 个</span><span class="sxs-lookup"><span data-stu-id="66f61-175">2 per role</span></span> | <span data-ttu-id="66f61-176">对于生产部署，每个群集 5 个</span><span class="sxs-lookup"><span data-stu-id="66f61-176">5 per cluster, for production deployments</span></span> |
| <span data-ttu-id="66f61-177">状态管理</span><span class="sxs-lookup"><span data-stu-id="66f61-177">State management</span></span> | <span data-ttu-id="66f61-178">无状态</span><span class="sxs-lookup"><span data-stu-id="66f61-178">Stateless</span></span> | <span data-ttu-id="66f61-179">无状态或有状态\*</span><span class="sxs-lookup"><span data-stu-id="66f61-179">Stateless or stateful\*</span></span> |
| <span data-ttu-id="66f61-180">Hosting</span><span class="sxs-lookup"><span data-stu-id="66f61-180">Hosting</span></span> | <span data-ttu-id="66f61-181">Azure</span><span class="sxs-lookup"><span data-stu-id="66f61-181">Azure</span></span> | <span data-ttu-id="66f61-182">云或本地</span><span class="sxs-lookup"><span data-stu-id="66f61-182">Cloud or on-premises</span></span> |
| <span data-ttu-id="66f61-183">Web 托管</span><span class="sxs-lookup"><span data-stu-id="66f61-183">Web hosting</span></span> | <span data-ttu-id="66f61-184">IIS**</span><span class="sxs-lookup"><span data-stu-id="66f61-184">IIS**</span></span> | <span data-ttu-id="66f61-185">自托管</span><span class="sxs-lookup"><span data-stu-id="66f61-185">Self-hosting</span></span> |
| <span data-ttu-id="66f61-186">部署模型</span><span class="sxs-lookup"><span data-stu-id="66f61-186">Deployment model</span></span> | <span data-ttu-id="66f61-187">[经典部署模型][azure-deployment-models]</span><span class="sxs-lookup"><span data-stu-id="66f61-187">[Classic deployment model][azure-deployment-models]</span></span> | <span data-ttu-id="66f61-188">[资源管理器][azure-deployment-models]</span><span class="sxs-lookup"><span data-stu-id="66f61-188">[Resource Manager][azure-deployment-models]</span></span>  |
| <span data-ttu-id="66f61-189">打包</span><span class="sxs-lookup"><span data-stu-id="66f61-189">Packaging</span></span> | <span data-ttu-id="66f61-190">云服务包文件 (.cspkg)</span><span class="sxs-lookup"><span data-stu-id="66f61-190">Cloud service package files (.cspkg)</span></span> | <span data-ttu-id="66f61-191">应用程序和服务包</span><span class="sxs-lookup"><span data-stu-id="66f61-191">Application and service packages</span></span> |
| <span data-ttu-id="66f61-192">应用程序更新</span><span class="sxs-lookup"><span data-stu-id="66f61-192">Application update</span></span> | <span data-ttu-id="66f61-193">VIP 交换或滚动更新</span><span class="sxs-lookup"><span data-stu-id="66f61-193">VIP swap or rolling update</span></span> | <span data-ttu-id="66f61-194">滚动更新</span><span class="sxs-lookup"><span data-stu-id="66f61-194">Rolling update</span></span> |
| <span data-ttu-id="66f61-195">自动缩放</span><span class="sxs-lookup"><span data-stu-id="66f61-195">Auto-scaling</span></span> | <span data-ttu-id="66f61-196">[内置服务][cloud-service-autoscale]</span><span class="sxs-lookup"><span data-stu-id="66f61-196">[Built-in service][cloud-service-autoscale]</span></span> | <span data-ttu-id="66f61-197">使用 VM 规模集进行自动缩放</span><span class="sxs-lookup"><span data-stu-id="66f61-197">VM Scale Sets for auto scale out</span></span> |
| <span data-ttu-id="66f61-198">调试</span><span class="sxs-lookup"><span data-stu-id="66f61-198">Debugging</span></span> | <span data-ttu-id="66f61-199">本地仿真器</span><span class="sxs-lookup"><span data-stu-id="66f61-199">Local emulator</span></span> | <span data-ttu-id="66f61-200">本地群集</span><span class="sxs-lookup"><span data-stu-id="66f61-200">Local cluster</span></span> |


<span data-ttu-id="66f61-201">\* 有状态服务使用[可靠集合][sf-reliable-collections]将状态存储在各个副本中，以便所有读取操作都在群集节点本地进行。</span><span class="sxs-lookup"><span data-stu-id="66f61-201">\* Stateful services use [reliable collections][sf-reliable-collections] to store state across replicas, so that all reads are local to the nodes in the cluster.</span></span> <span data-ttu-id="66f61-202">写入操作跨节点进行复制，以保证可靠性。</span><span class="sxs-lookup"><span data-stu-id="66f61-202">Writes are replicated across nodes for reliability.</span></span> <span data-ttu-id="66f61-203">无状态服务通过使用数据库或其他外部存储，可以有外部状态。</span><span class="sxs-lookup"><span data-stu-id="66f61-203">Stateless services can have external state, using a database or other external storage.</span></span>

<span data-ttu-id="66f61-204">** 辅助角色也可以使用 OWIN 自托管 ASP.NET Web API。</span><span class="sxs-lookup"><span data-stu-id="66f61-204">** Worker roles can also self-host ASP.NET Web API using OWIN.</span></span>

## <a name="the-surveys-application-on-cloud-services"></a><span data-ttu-id="66f61-205">云服务上的 Surveys 应用程序</span><span class="sxs-lookup"><span data-stu-id="66f61-205">The Surveys application on Cloud Services</span></span>

<span data-ttu-id="66f61-206">下图展示了云服务上运行的 Surveys 应用程序的体系结构。</span><span class="sxs-lookup"><span data-stu-id="66f61-206">The following diagram shows the architecture of the Surveys application running on Cloud Services.</span></span> 

![](./images/tailspin01.png)

<span data-ttu-id="66f61-207">该应用程序由两个 Web 角色和一个辅助角色组成。</span><span class="sxs-lookup"><span data-stu-id="66f61-207">The application consists of two web roles and a worker role.</span></span>

- <span data-ttu-id="66f61-208">**Tailspin.Web** Web 角色托管一个 ASP.NET 网站，Tailspin 客户可用于创建和管理问卷调查。</span><span class="sxs-lookup"><span data-stu-id="66f61-208">The **Tailspin.Web** web role hosts an ASP.NET website that Tailspin customers use to create and manage surveys.</span></span> <span data-ttu-id="66f61-209">客户还使用此网站注册应用程序和管理其订阅。</span><span class="sxs-lookup"><span data-stu-id="66f61-209">Customers also use this website to sign up for the application and manage their subscriptions.</span></span> <span data-ttu-id="66f61-210">最后，Tailspin 管理员可通过它查看租户列表和管理租户数据。</span><span class="sxs-lookup"><span data-stu-id="66f61-210">Finally, Tailspin administrators can use it to see the list of tenants and manage tenant data.</span></span> 

- <span data-ttu-id="66f61-211">**Tailspin.Web.Survey.Public** Web 角色托管一个 ASP.NET 网站，人们可在网站上参与 Tailspin 客户发布的问卷调查。</span><span class="sxs-lookup"><span data-stu-id="66f61-211">The **Tailspin.Web.Survey.Public** web role hosts an ASP.NET website where people can take the surveys that Tailspin customers publish.</span></span> 

- <span data-ttu-id="66f61-212">**Tailspin.Workers.Survey** 辅助角色执行后台处理。</span><span class="sxs-lookup"><span data-stu-id="66f61-212">The **Tailspin.Workers.Survey** worker role does background processing.</span></span> <span data-ttu-id="66f61-213">Web 角色将工作项放入队列，辅助角色则处理这些项。</span><span class="sxs-lookup"><span data-stu-id="66f61-213">The web roles put work items onto a queue, and the worker role processes the items.</span></span> <span data-ttu-id="66f61-214">定义了两个后台任务：将问卷调查答案导出到 Azure SQL 数据库，以及计算问卷调查答案的统计数据。</span><span class="sxs-lookup"><span data-stu-id="66f61-214">Two background tasks are defined: Exporting survey answers to Azure SQL Database, and calculating statistics for survey answers.</span></span>

<span data-ttu-id="66f61-215">除云服务外，Surveys 应用程序还使用其他 Azure 服务：</span><span class="sxs-lookup"><span data-stu-id="66f61-215">In addition to Cloud Services, the Surveys application uses some other Azure services:</span></span>

- <span data-ttu-id="66f61-216">**Azure 存储**，用于存储问卷调查、问卷调查答案和租户信息。</span><span class="sxs-lookup"><span data-stu-id="66f61-216">**Azure Storage** to store surveys, surveys answers, and tenant information.</span></span>

- <span data-ttu-id="66f61-217">**Azure Redis 缓存**，用于缓存 Azure 存储中存储的一些数据，以便加快读取访问速度。</span><span class="sxs-lookup"><span data-stu-id="66f61-217">**Azure Redis Cache** to cache some of the data that is stored in Azure Storage, for faster read access.</span></span> 

- <span data-ttu-id="66f61-218">**Azure Active Directory** (Azure AD)，用于验证客户和 Tailspin 管理员的身份。</span><span class="sxs-lookup"><span data-stu-id="66f61-218">**Azure Active Directory** (Azure AD) to authenticate customers and Tailspin administrators.</span></span>

- <span data-ttu-id="66f61-219">**Azure SQL 数据库**，用于存储问卷调查答案进行分析。</span><span class="sxs-lookup"><span data-stu-id="66f61-219">**Azure SQL Database** to store the survey answers for analysis.</span></span> 

## <a name="moving-to-service-fabric"></a><span data-ttu-id="66f61-220">迁移到 Service Fabric</span><span class="sxs-lookup"><span data-stu-id="66f61-220">Moving to Service Fabric</span></span>

<span data-ttu-id="66f61-221">如前文所述，此阶段的目标是在迁移到 Service Fabric 时尽量减少所需的更改。</span><span class="sxs-lookup"><span data-stu-id="66f61-221">As mentioned, the goal of this phase was migrating to Service Fabric with the minimum necessary changes.</span></span> <span data-ttu-id="66f61-222">为此，我们创建了与原始应用程序中每个云服务角色相对应的无状态服务：</span><span class="sxs-lookup"><span data-stu-id="66f61-222">To that end, we created stateless services corresponding to each cloud service role in the original application:</span></span>

![](./images/tailspin02.png)

<span data-ttu-id="66f61-223">此体系结构有意设计成与原始应用程序非常类似。</span><span class="sxs-lookup"><span data-stu-id="66f61-223">Intentionally, this architecture is very similar to the original application.</span></span> <span data-ttu-id="66f61-224">不过，该图隐藏了一些重要差异。</span><span class="sxs-lookup"><span data-stu-id="66f61-224">However, the diagram hides some important differences.</span></span> <span data-ttu-id="66f61-225">在本文的其余部分，我们将探讨这些差异。</span><span class="sxs-lookup"><span data-stu-id="66f61-225">In the rest of this article, we'll explore those differences.</span></span> 


## <a name="converting-the-cloud-service-roles-to-services"></a><span data-ttu-id="66f61-226">将云服务角色转换为服务</span><span class="sxs-lookup"><span data-stu-id="66f61-226">Converting the cloud service roles to services</span></span>

<span data-ttu-id="66f61-227">如前文所述，我们已将各云服务角色迁移到 Service Fabric 服务。</span><span class="sxs-lookup"><span data-stu-id="66f61-227">As mentioned, we migrated each cloud service role to a Service Fabric service.</span></span> <span data-ttu-id="66f61-228">由于云服务角色是无状态的，因此，在此阶段，在 Service Fabric 中创建无状态服务较为合理。</span><span class="sxs-lookup"><span data-stu-id="66f61-228">Because cloud service roles are stateless, for this phase it made sense to create stateless services in Service Fabric.</span></span> 

<span data-ttu-id="66f61-229">迁移时，我们遵循了[将 Web 角色和辅助角色转换成 Service Fabric 无状态服务的指南][sf-migration]中概述的步骤。</span><span class="sxs-lookup"><span data-stu-id="66f61-229">For the migration, we followed the steps outlined in [Guide to converting Web and Worker Roles to Service Fabric stateless services][sf-migration].</span></span> 

### <a name="creating-the-web-front-end-services"></a><span data-ttu-id="66f61-230">创建 Web 前端服务</span><span class="sxs-lookup"><span data-stu-id="66f61-230">Creating the web front-end services</span></span>

<span data-ttu-id="66f61-231">在 Service Fabric 中，服务在由 Service Fabric 运行时创建的进程内运行。</span><span class="sxs-lookup"><span data-stu-id="66f61-231">In Service Fabric, a service runs inside a process created by the Service Fabric runtime.</span></span> <span data-ttu-id="66f61-232">对于 Web 前端，这意味着该服务不在 IIS 内运行，</span><span class="sxs-lookup"><span data-stu-id="66f61-232">For a web front end, that means the service is not running inside IIS.</span></span> <span data-ttu-id="66f61-233">而必须托管 Web 服务器。</span><span class="sxs-lookup"><span data-stu-id="66f61-233">Instead, the service must host a web server.</span></span> <span data-ttu-id="66f61-234">此方法称为*自托管*，因为进程内运行的代码充当 Web 服务器主机。</span><span class="sxs-lookup"><span data-stu-id="66f61-234">This approach is called *self-hosting*, because the code that runs inside the process acts as the web server host.</span></span> 

<span data-ttu-id="66f61-235">自托管要求意味着 Service Fabric 服务不能使用 ASP.NET MVC 或 ASP.NET Web 窗体，因为这些框架需要 IIS，并且不支持自托管。</span><span class="sxs-lookup"><span data-stu-id="66f61-235">The requirement to self-host means that a Service Fabric service can't use ASP.NET MVC or ASP.NET Web Forms, because those frameworks require IIS and do not support self-hosting.</span></span> <span data-ttu-id="66f61-236">自托管选项包括：</span><span class="sxs-lookup"><span data-stu-id="66f61-236">Options for self-hosting include:</span></span>

- <span data-ttu-id="66f61-237">[ASP.NET Core][aspnet-core]，使用 [Kestrel][kestrel] Web 服务器自托管。</span><span class="sxs-lookup"><span data-stu-id="66f61-237">[ASP.NET Core][aspnet-core], self-hosted using the [Kestrel][kestrel] web server.</span></span> 
- <span data-ttu-id="66f61-238">[ASP.NET Web API][aspnet-webapi]，使用 [OWIN][owin] 自托管。</span><span class="sxs-lookup"><span data-stu-id="66f61-238">[ASP.NET Web API][aspnet-webapi], self-hosted using [OWIN][owin].</span></span>
- <span data-ttu-id="66f61-239">[Nancy](http://nancyfx.org/) 等第三方框架。</span><span class="sxs-lookup"><span data-stu-id="66f61-239">Third-party frameworks such as [Nancy](http://nancyfx.org/).</span></span>

<span data-ttu-id="66f61-240">原始 Surveys 应用程序使用 ASP.NET MVC。</span><span class="sxs-lookup"><span data-stu-id="66f61-240">The original Surveys application uses ASP.NET MVC.</span></span> <span data-ttu-id="66f61-241">由于 ASP.NET MVC 不能在 Service Fabric 中自托管，因此，我们曾考虑使用以下迁移选项：</span><span class="sxs-lookup"><span data-stu-id="66f61-241">Because ASP.NET MVC cannot be self-hosted in Service Fabric, we considered the following migration options:</span></span>

- <span data-ttu-id="66f61-242">将 Web 角色移植到可自托管的 ASP.NET Core。</span><span class="sxs-lookup"><span data-stu-id="66f61-242">Port the web roles to ASP.NET Core, which can be self-hosted.</span></span>
- <span data-ttu-id="66f61-243">将网站转换成单页应用程序 (SPA)，以调用使用 ASP.NET Web API 实现的 Web API。</span><span class="sxs-lookup"><span data-stu-id="66f61-243">Convert the web site into a single-page application (SPA) that calls a web API implemented using ASP.NET Web API.</span></span> <span data-ttu-id="66f61-244">这需要重新设计 Web 前端。</span><span class="sxs-lookup"><span data-stu-id="66f61-244">This would have required a complete redesign of the web front end.</span></span>
- <span data-ttu-id="66f61-245">保留现有的 ASP.NET MVC 代码，并将 Windows Server 容器中的 IIS 部署到 Service Fabric。</span><span class="sxs-lookup"><span data-stu-id="66f61-245">Keep the existing ASP.NET MVC code and deploy IIS in a Windows Server container to Service Fabric.</span></span> <span data-ttu-id="66f61-246">此方法只需对代码稍作改动，甚至无需改动。</span><span class="sxs-lookup"><span data-stu-id="66f61-246">This approach would require little or no code change.</span></span> <span data-ttu-id="66f61-247">但是，Service Fabric 中的[容器支持][sf-containers]目前仍处于预览阶段。</span><span class="sxs-lookup"><span data-stu-id="66f61-247">However, [container support][sf-containers] in Service Fabric is currently still in preview.</span></span>

<span data-ttu-id="66f61-248">基于以上考虑，我们选择了第一个选项：移植到 ASP.NET Core。</span><span class="sxs-lookup"><span data-stu-id="66f61-248">Based on these considerations, we selected the first option, porting to ASP.NET Core.</span></span> <span data-ttu-id="66f61-249">为此，我们遵循了[从 ASP.NET MVC 迁移到 ASP.NET Core MVC][aspnet-migration] 中所述的步骤。</span><span class="sxs-lookup"><span data-stu-id="66f61-249">To do so, we followed the steps described in [Migrating From ASP.NET MVC to ASP.NET Core MVC][aspnet-migration].</span></span> 

> [!NOTE]
> <span data-ttu-id="66f61-250">在 ASP.NET Core 中使用 Kestrel 时，出于安全考虑，应在 Kestrel 前面放置一个反向代理，以处理来自 Internet 的流量。</span><span class="sxs-lookup"><span data-stu-id="66f61-250">When using ASP.NET Core with Kestrel, you should place a reverse proxy in front of Kestrel to handle traffic from the Internet, for security reasons.</span></span> <span data-ttu-id="66f61-251">有关详细信息，请参阅 [ASP.NET Core 中的 Kestrel Web 服务器实现][kestrel]。</span><span class="sxs-lookup"><span data-stu-id="66f61-251">For more information, see [Kestrel web server implementation in ASP.NET Core][kestrel].</span></span> <span data-ttu-id="66f61-252">[部署应用程序](#deploying-the-application)部分介绍了建议的 Azure 部署。</span><span class="sxs-lookup"><span data-stu-id="66f61-252">The section [Deploying the application](#deploying-the-application) describes a recommended Azure deployment.</span></span>

### <a name="http-listeners"></a><span data-ttu-id="66f61-253">HTTP 侦听器</span><span class="sxs-lookup"><span data-stu-id="66f61-253">HTTP listeners</span></span>

<span data-ttu-id="66f61-254">在云服务中，Web 角色或辅助角色通过在[服务定义文件][cloud-service-endpoints]中声明 HTTP 终结点来公开该终结点。</span><span class="sxs-lookup"><span data-stu-id="66f61-254">In Cloud Services, a web or worker role exposes an HTTP endpoint by declaring it in the [service definition file][cloud-service-endpoints].</span></span> <span data-ttu-id="66f61-255">Web 角色必须具有至少一个终结点。</span><span class="sxs-lookup"><span data-stu-id="66f61-255">A web role must have at least one endpoint.</span></span>

```xml
<!-- Cloud service endpoint -->
<Endpoints>
    <InputEndpoint name="HttpIn" protocol="http" port="80" />
</Endpoints>
```

<span data-ttu-id="66f61-256">同样，Service Fabric 终结点在服务清单中声明：</span><span class="sxs-lookup"><span data-stu-id="66f61-256">Similarly, Service Fabric endpoints are declared in a service manifest:</span></span> 

```xml
<!-- Service Fabric endpoint -->
<Endpoints>
    <Endpoint Protocol="http" Name="ServiceEndpoint" Type="Input" Port="8002" />
</Endpoints>
```

<span data-ttu-id="66f61-257">但与云服务角色不同的是，Service Fabric 服务可共存于同一节点内。</span><span class="sxs-lookup"><span data-stu-id="66f61-257">Unlike a cloud service role, however, Service Fabric services can be co-located within the same node.</span></span> <span data-ttu-id="66f61-258">因此，每个服务必须侦听不同的端口。</span><span class="sxs-lookup"><span data-stu-id="66f61-258">Therefore, every service must listen on a distinct port.</span></span> <span data-ttu-id="66f61-259">本文后面会讨论如何将端口 80 或端口 443 上的客户端请求路由到服务的正确端口。</span><span class="sxs-lookup"><span data-stu-id="66f61-259">Later in this article, we'll discuss how client requests on port 80 or port 443 get routed to the correct port for the service.</span></span>

<span data-ttu-id="66f61-260">服务必须为每个终结点显式创建侦听器。</span><span class="sxs-lookup"><span data-stu-id="66f61-260">A service must explicitly create listeners for each endpoint.</span></span> <span data-ttu-id="66f61-261">原因是 Service Fabric 对通信堆栈不可知。</span><span class="sxs-lookup"><span data-stu-id="66f61-261">The reason is that Service Fabric is agnostic about communication stacks.</span></span> <span data-ttu-id="66f61-262">有关详细信息，请参阅[使用 ASP.NET Core 生成应用程序的 Web 服务前端][sf-aspnet-core]。</span><span class="sxs-lookup"><span data-stu-id="66f61-262">For more information, see [Build a web service front end for your application using ASP.NET Core][sf-aspnet-core].</span></span>

## <a name="packaging-and-configuration"></a><span data-ttu-id="66f61-263">打包和配置</span><span class="sxs-lookup"><span data-stu-id="66f61-263">Packaging and configuration</span></span>

 <span data-ttu-id="66f61-264">云服务包含以下配置和包文件：</span><span class="sxs-lookup"><span data-stu-id="66f61-264">A cloud service contains the following configuration and package files:</span></span>

| <span data-ttu-id="66f61-265">文件</span><span class="sxs-lookup"><span data-stu-id="66f61-265">File</span></span> | <span data-ttu-id="66f61-266">说明</span><span class="sxs-lookup"><span data-stu-id="66f61-266">Description</span></span> |
|------|-------------|
| <span data-ttu-id="66f61-267">服务定义 (.csdef)</span><span class="sxs-lookup"><span data-stu-id="66f61-267">Service definition (.csdef)</span></span> | <span data-ttu-id="66f61-268">Azure 用于配置云服务的设置。</span><span class="sxs-lookup"><span data-stu-id="66f61-268">Settings used by Azure to configure the cloud service.</span></span> <span data-ttu-id="66f61-269">定义角色、终结点、启动任务和配置设置名称。</span><span class="sxs-lookup"><span data-stu-id="66f61-269">Defines the roles, endpoints, startup tasks, and the names of configuration settings.</span></span> |
| <span data-ttu-id="66f61-270">服务配置 (.cscfg)</span><span class="sxs-lookup"><span data-stu-id="66f61-270">Service configuration (.cscfg)</span></span> | <span data-ttu-id="66f61-271">针对每个部署的设置，包括角色实例数、终结点端口号和配置设置的值。</span><span class="sxs-lookup"><span data-stu-id="66f61-271">Per-deployment settings, including the number of role instances, endpoint port numbers, and the values of configuration settings.</span></span> 
| <span data-ttu-id="66f61-272">服务包 (.cspkg)</span><span class="sxs-lookup"><span data-stu-id="66f61-272">Service package (.cspkg)</span></span> | <span data-ttu-id="66f61-273">包含应用程序代码和配置以及服务定义文件。</span><span class="sxs-lookup"><span data-stu-id="66f61-273">Contains the application code and configurations, and the service definition file.</span></span>  |

<span data-ttu-id="66f61-274">整个应用程序只有一个 .csdef 文件。</span><span class="sxs-lookup"><span data-stu-id="66f61-274">There is one .csdef file for the entire application.</span></span> <span data-ttu-id="66f61-275">针对不同的环境可以有多个 .cscfg 文件，比如本地环境、测试环境或生产环境。</span><span class="sxs-lookup"><span data-stu-id="66f61-275">You can have multiple .cscfg files for different environments, such as local, test, or production.</span></span> <span data-ttu-id="66f61-276">当服务运行时，可以只更新 .cscfg，而不更新 .csdef。</span><span class="sxs-lookup"><span data-stu-id="66f61-276">When the service is running, you can update the .cscfg but not the .csdef.</span></span> <span data-ttu-id="66f61-277">有关详细信息，请参阅[什么是云服务模型以及如何将其打包？][cloud-service-config]</span><span class="sxs-lookup"><span data-stu-id="66f61-277">For more information, see [What is the Cloud Service model and how do I package it?][cloud-service-config]</span></span>

<span data-ttu-id="66f61-278">Service Fabric 对服务*定义*和服务*设置*也有类似的区分，但结构更精细。</span><span class="sxs-lookup"><span data-stu-id="66f61-278">Service Fabric has a similar division between a service *definition* and service *settings*, but the structure is more granular.</span></span> <span data-ttu-id="66f61-279">若要了解 Service Fabric 的配置模型，可先了解如何打包 Service Fabric 应用程序。</span><span class="sxs-lookup"><span data-stu-id="66f61-279">To understand Service Fabric's configuration model, it helps to understand how a Service Fabric application is packaged.</span></span> <span data-ttu-id="66f61-280">其结构如下：</span><span class="sxs-lookup"><span data-stu-id="66f61-280">Here is the structure:</span></span>

```
Application package
  - Service packages
    - Code package
    - Configuration package
    - Data package (optional)
```

<span data-ttu-id="66f61-281">应用程序包是你部署的包。</span><span class="sxs-lookup"><span data-stu-id="66f61-281">The application package is what you deploy.</span></span> <span data-ttu-id="66f61-282">它包含一个或多个服务包。</span><span class="sxs-lookup"><span data-stu-id="66f61-282">It contains one or more service packages.</span></span> <span data-ttu-id="66f61-283">服务包包含代码包、配置包和数据包。</span><span class="sxs-lookup"><span data-stu-id="66f61-283">A service package contains code, configuration, and data packages.</span></span> <span data-ttu-id="66f61-284">代码包包含服务的二进制文件，配置包包含配置设置。</span><span class="sxs-lookup"><span data-stu-id="66f61-284">The code package contains the binaries for the services, and the configuration package contains configuration settings.</span></span> <span data-ttu-id="66f61-285">在此模型中，升级各个服务时无需重新部署整个应用程序。</span><span class="sxs-lookup"><span data-stu-id="66f61-285">This model allows you to upgrade individual services without redeploying the entire application.</span></span> <span data-ttu-id="66f61-286">还可以只更新配置设置，而无需重新部署代码或重新启动服务。</span><span class="sxs-lookup"><span data-stu-id="66f61-286">It also lets you update just the configuration settings, without redeploying the code or restarting the service.</span></span>

<span data-ttu-id="66f61-287">Service Fabric 应用程序包含以下配置文件：</span><span class="sxs-lookup"><span data-stu-id="66f61-287">A Service Fabric application contains the following configuration files:</span></span>

| <span data-ttu-id="66f61-288">文件</span><span class="sxs-lookup"><span data-stu-id="66f61-288">File</span></span> | <span data-ttu-id="66f61-289">Location</span><span class="sxs-lookup"><span data-stu-id="66f61-289">Location</span></span> | <span data-ttu-id="66f61-290">说明</span><span class="sxs-lookup"><span data-stu-id="66f61-290">Description</span></span> |
|------|----------|-------------|
| <span data-ttu-id="66f61-291">ApplicationManifest.xml</span><span class="sxs-lookup"><span data-stu-id="66f61-291">ApplicationManifest.xml</span></span> | <span data-ttu-id="66f61-292">应用程序包</span><span class="sxs-lookup"><span data-stu-id="66f61-292">Application package</span></span> | <span data-ttu-id="66f61-293">定义构成应用程序的服务。</span><span class="sxs-lookup"><span data-stu-id="66f61-293">Defines the services that compose the application.</span></span> |
| <span data-ttu-id="66f61-294">ServiceManifest.xml</span><span class="sxs-lookup"><span data-stu-id="66f61-294">ServiceManifest.xml</span></span> | <span data-ttu-id="66f61-295">服务包</span><span class="sxs-lookup"><span data-stu-id="66f61-295">Service package</span></span>| <span data-ttu-id="66f61-296">描述一个或多个服务。</span><span class="sxs-lookup"><span data-stu-id="66f61-296">Describes one or more services.</span></span> |
| <span data-ttu-id="66f61-297">Settings.xml</span><span class="sxs-lookup"><span data-stu-id="66f61-297">Settings.xml</span></span> | <span data-ttu-id="66f61-298">配置包</span><span class="sxs-lookup"><span data-stu-id="66f61-298">Configuration package</span></span> | <span data-ttu-id="66f61-299">包含服务包中定义的服务的配置设置。</span><span class="sxs-lookup"><span data-stu-id="66f61-299">Contains configuration settings for the services defined in the service package.</span></span> |

<span data-ttu-id="66f61-300">有关详细信息，请参阅[在 Service Fabric 中对应用程序建模][sf-application-model]。</span><span class="sxs-lookup"><span data-stu-id="66f61-300">For more information, see [Model an application in Service Fabric][sf-application-model].</span></span>

<span data-ttu-id="66f61-301">若要支持多个环境的不同配置设置，请使用[管理多个环境的应用程序参数][sf-multiple-environments]中所述的以下方法：</span><span class="sxs-lookup"><span data-stu-id="66f61-301">To support different configuration settings for multiple environments, use the following approach, described in [Manage application parameters for multiple environments][sf-multiple-environments]:</span></span>

1. <span data-ttu-id="66f61-302">在服务的 Setting.xml 文件中定义设置。</span><span class="sxs-lookup"><span data-stu-id="66f61-302">Define the setting in the Setting.xml file for the service.</span></span>
2. <span data-ttu-id="66f61-303">在应用程序清单中，定义设置的替代项。</span><span class="sxs-lookup"><span data-stu-id="66f61-303">In the application manifest, define an override for the setting.</span></span>
3. <span data-ttu-id="66f61-304">将特定于环境的设置放入应用程序参数文件。</span><span class="sxs-lookup"><span data-stu-id="66f61-304">Put environment-specific settings into application parameter files.</span></span>


## <a name="deploying-the-application"></a><span data-ttu-id="66f61-305">部署应用程序</span><span class="sxs-lookup"><span data-stu-id="66f61-305">Deploying the application</span></span>

<span data-ttu-id="66f61-306">Azure 云服务是托管服务，而 Service Fabric 是运行时。</span><span class="sxs-lookup"><span data-stu-id="66f61-306">Whereas Azure Cloud Services is a managed service, Service Fabric is a runtime.</span></span> <span data-ttu-id="66f61-307">可以在许多环境中创建 Service Fabric 群集，包括 Azure 和本地。</span><span class="sxs-lookup"><span data-stu-id="66f61-307">You can create Service Fabric clusters in many environments, including Azure and on premises.</span></span> <span data-ttu-id="66f61-308">本文重点介绍如何部署到 Azure。</span><span class="sxs-lookup"><span data-stu-id="66f61-308">In this article, we focus on deploying to Azure.</span></span> 

<span data-ttu-id="66f61-309">下图展示了建议的部署：</span><span class="sxs-lookup"><span data-stu-id="66f61-309">The following diagram shows a recommended deployment:</span></span>

![](./images/tailspin-cluster.png)

<span data-ttu-id="66f61-310">Service Fabric 群集部署到 [VM 规模集][vm-scale-sets]。</span><span class="sxs-lookup"><span data-stu-id="66f61-310">The Service Fabric cluster is deployed to a [VM scale set][vm-scale-sets].</span></span> <span data-ttu-id="66f61-311">规模集是一种 Azure 计算资源，可用于部署和管理一组相同的 VM。</span><span class="sxs-lookup"><span data-stu-id="66f61-311">Scale sets are an Azure Compute resource that can be used to deploy and manage a set of identical VMs.</span></span> 

<span data-ttu-id="66f61-312">如前文所述，Kestrel Web 服务器出于安全原因需要一个反向代理。</span><span class="sxs-lookup"><span data-stu-id="66f61-312">As mentioned, the Kestrel web server requires a reverse proxy for security reasons.</span></span> <span data-ttu-id="66f61-313">此图展示了 [Azure 应用程序网关][application-gateway]，它是一种可提供各种第 7 层负载均衡功能的 Azure 服务。</span><span class="sxs-lookup"><span data-stu-id="66f61-313">This diagram shows [Azure Application Gateway][application-gateway], which is an Azure service that offers various layer 7 load balancing capabilities.</span></span> <span data-ttu-id="66f61-314">它充当反向代理服务，终止客户端连接，并将请求转发到后端终结点。</span><span class="sxs-lookup"><span data-stu-id="66f61-314">It acts as a reverse-proxy service, terminating the client connection and forwarding requests to back-end endpoints.</span></span> <span data-ttu-id="66f61-315">你可能会使用其他反向代理解决方案，比如 nginx。</span><span class="sxs-lookup"><span data-stu-id="66f61-315">You might use a different reverse proxy solution, such as nginx.</span></span>  

### <a name="layer-7-routing"></a><span data-ttu-id="66f61-316">第 7 层路由</span><span class="sxs-lookup"><span data-stu-id="66f61-316">Layer 7 routing</span></span>

<span data-ttu-id="66f61-317">在[原始 Surveys 应用程序](https://msdn.microsoft.com/library/hh534477.aspx#sec21)中，一个 Web 角色侦听端口 80，另一个 Web 角色侦听端口 443。</span><span class="sxs-lookup"><span data-stu-id="66f61-317">In the [original Surveys application](https://msdn.microsoft.com/library/hh534477.aspx#sec21), one web role listened on port 80, and the other web role listened on port 443.</span></span> 

| <span data-ttu-id="66f61-318">公共站点</span><span class="sxs-lookup"><span data-stu-id="66f61-318">Public site</span></span> | <span data-ttu-id="66f61-319">Survey 管理站点</span><span class="sxs-lookup"><span data-stu-id="66f61-319">Survey management site</span></span> |
|-------------|------------------------|
| `http://tailspin.cloudapp.net` | `https://tailspin.cloudapp.net` |

<span data-ttu-id="66f61-320">另一种做法是使用第 7 层路由。</span><span class="sxs-lookup"><span data-stu-id="66f61-320">Another option is to use layer 7 routing.</span></span> <span data-ttu-id="66f61-321">在此方法中，不同的 URL 路径会路由到后端的不同端口号。</span><span class="sxs-lookup"><span data-stu-id="66f61-321">In this approach, different URL paths get routed to different port numbers on the back end.</span></span> <span data-ttu-id="66f61-322">例如，公共站点可能使用以 `/public/` 开头的 URL 路径。</span><span class="sxs-lookup"><span data-stu-id="66f61-322">For example, the public site might use URL paths starting with `/public/`.</span></span> 

<span data-ttu-id="66f61-323">第 7 层路由选项包括：</span><span class="sxs-lookup"><span data-stu-id="66f61-323">Options for layer 7 routing include:</span></span>

- <span data-ttu-id="66f61-324">使用应用程序网关。</span><span class="sxs-lookup"><span data-stu-id="66f61-324">Use Application Gateway.</span></span> 

- <span data-ttu-id="66f61-325">使用网络虚拟设备 (NVA)，比如 nginx。</span><span class="sxs-lookup"><span data-stu-id="66f61-325">Use a network virtual appliance (NVA), such as nginx.</span></span>

- <span data-ttu-id="66f61-326">编写一个自定义网关作为无状态服务。</span><span class="sxs-lookup"><span data-stu-id="66f61-326">Write a custom gateway as a stateless service.</span></span>

<span data-ttu-id="66f61-327">如果有两个或更多个具有公共 HTTP 终结点的服务，但希望它们显示为一个具有单一域名的站点，可考虑使用此方法。</span><span class="sxs-lookup"><span data-stu-id="66f61-327">Consider this approach if you have two or more services with public HTTP endpoints, but want them to appear as one site with a single domain name.</span></span>

> <span data-ttu-id="66f61-328">还有一个方法是允许外部客户端通过 Service Fabric [反向代理][sf-reverse-proxy]发送请求，但*不*建议采用此方法。</span><span class="sxs-lookup"><span data-stu-id="66f61-328">One approach that we *don't* recommend is allowing external clients to send requests through the Service Fabric [reverse proxy][sf-reverse-proxy].</span></span> <span data-ttu-id="66f61-329">尽管这是可行的，但反向代理适用于服务到服务通信。</span><span class="sxs-lookup"><span data-stu-id="66f61-329">Although this is possible, the reverse proxy is intended for service-to-service communication.</span></span> <span data-ttu-id="66f61-330">向外部客户端打开反向代理会公开群集中运行的*所有*具有 HTTP 终结点的服务。</span><span class="sxs-lookup"><span data-stu-id="66f61-330">Opening it to external clients exposes *any* service running in the cluster that has an HTTP endpoint.</span></span>

### <a name="node-types-and-placement-constraints"></a><span data-ttu-id="66f61-331">节点类型和放置约束</span><span class="sxs-lookup"><span data-stu-id="66f61-331">Node types and placement constraints</span></span>

<span data-ttu-id="66f61-332">在上面所示的部署中，所有服务在所有节点上运行。</span><span class="sxs-lookup"><span data-stu-id="66f61-332">In the deployment shown above, all the services run on all the nodes.</span></span> <span data-ttu-id="66f61-333">但是，也可以对服务分组，以便特定服务仅在群集内的特定节点上运行。</span><span class="sxs-lookup"><span data-stu-id="66f61-333">However, you can also group services, so that certain services run only on particular nodes within the cluster.</span></span> <span data-ttu-id="66f61-334">使用此方法的原因包括：</span><span class="sxs-lookup"><span data-stu-id="66f61-334">Reasons to use this approach include:</span></span>

- <span data-ttu-id="66f61-335">在不同的 VM 类型上运行某些服务。</span><span class="sxs-lookup"><span data-stu-id="66f61-335">Run some services on different VM types.</span></span> <span data-ttu-id="66f61-336">例如，某些服务可能需要大量计算资源或需要 GPU。</span><span class="sxs-lookup"><span data-stu-id="66f61-336">For example, some services might be compute-intensive or require GPUs.</span></span> <span data-ttu-id="66f61-337">Service Fabric 群集中可以有各种 VM 类型。</span><span class="sxs-lookup"><span data-stu-id="66f61-337">You can have a mix of VM types in your Service Fabric cluster.</span></span>
- <span data-ttu-id="66f61-338">出于安全考虑，隔离前端服务与后端服务。</span><span class="sxs-lookup"><span data-stu-id="66f61-338">Isolate front-end services from back-end services, for security reasons.</span></span> <span data-ttu-id="66f61-339">所有前端服务在一组节点上运行，后端服务则在同一群集中的其他节点上运行。</span><span class="sxs-lookup"><span data-stu-id="66f61-339">All the front-end services will run on one set of nodes, and the back-end services will run on different nodes in the same cluster.</span></span>
- <span data-ttu-id="66f61-340">不同的缩放要求。</span><span class="sxs-lookup"><span data-stu-id="66f61-340">Different scale requirements.</span></span> <span data-ttu-id="66f61-341">某些服务相比其他服务，可能需要在更多节点上运行。</span><span class="sxs-lookup"><span data-stu-id="66f61-341">Some services might need to run on more nodes than other services.</span></span> <span data-ttu-id="66f61-342">例如，如果定义前端节点和后端节点，则每组节点都可以独立缩放。</span><span class="sxs-lookup"><span data-stu-id="66f61-342">For example, if you define front-end nodes and back-end nodes, each set can be scaled independently.</span></span>

<span data-ttu-id="66f61-343">下图展示隔离了前端服务与后端服务的群集：</span><span class="sxs-lookup"><span data-stu-id="66f61-343">The following diagram shows a cluster that separates front-end and back-end services:</span></span>

![](././images/node-placement.png)

<span data-ttu-id="66f61-344">若要实现此方法，请执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="66f61-344">To implement this approach:</span></span>

1.  <span data-ttu-id="66f61-345">创建群集时，定义两种或更多节点类型。</span><span class="sxs-lookup"><span data-stu-id="66f61-345">When you create the cluster, define two or more node types.</span></span> 
2.  <span data-ttu-id="66f61-346">对于每个服务，均使用[放置约束][sf-placement-constraints]将该服务分配到一种节点类型。</span><span class="sxs-lookup"><span data-stu-id="66f61-346">For each service, use [placement constraints][sf-placement-constraints] to assign the service to a node type.</span></span>

<span data-ttu-id="66f61-347">在部署到 Azure 时，每种节点类型都会部署到不同的 VM 规模集。</span><span class="sxs-lookup"><span data-stu-id="66f61-347">When you deploy to Azure, each node type is deployed to a separate VM scale set.</span></span> <span data-ttu-id="66f61-348">Service Fabric 群集跨所有节点类型。</span><span class="sxs-lookup"><span data-stu-id="66f61-348">The Service Fabric cluster spans all node types.</span></span> <span data-ttu-id="66f61-349">有关详细信息，请参阅 [Service Fabric 节点类型与虚拟机规模集之间的关系][sf-node-types]。</span><span class="sxs-lookup"><span data-stu-id="66f61-349">For more information, see [The relationship between Service Fabric node types and Virtual Machine Scale Sets][sf-node-types].</span></span>

> <span data-ttu-id="66f61-350">如果某个群集具有多种节点类型，则将其中一种节点类型指定为*主*节点类型。</span><span class="sxs-lookup"><span data-stu-id="66f61-350">If a cluster has multiple node types, one node type is designated as the *primary* node type.</span></span> <span data-ttu-id="66f61-351">Service Fabric 运行时服务（如群集管理服务）在主节点类型上运行。</span><span class="sxs-lookup"><span data-stu-id="66f61-351">Service Fabric runtime services, such as the Cluster Management Service, run on the primary node type.</span></span> <span data-ttu-id="66f61-352">在生产环境中，需为主节点类型预配至少 5 个节点。</span><span class="sxs-lookup"><span data-stu-id="66f61-352">Provision at least 5 nodes for the primary node type in a production environment.</span></span> <span data-ttu-id="66f61-353">其他节点类型应具有至少 2 个节点。</span><span class="sxs-lookup"><span data-stu-id="66f61-353">The other node type should have at least 2 nodes.</span></span>

## <a name="configuring-and-managing-the-cluster"></a><span data-ttu-id="66f61-354">配置和管理群集</span><span class="sxs-lookup"><span data-stu-id="66f61-354">Configuring and managing the cluster</span></span>

<span data-ttu-id="66f61-355">必须保护群集，防止未经授权的用户与其连接。</span><span class="sxs-lookup"><span data-stu-id="66f61-355">Clusters must be secured to prevent unauthorized users from connecting to your cluster.</span></span> <span data-ttu-id="66f61-356">建议使用 Azure AD 对客户端进行身份验证，并使用 X.509 证书确保节点到节点的安全性。</span><span class="sxs-lookup"><span data-stu-id="66f61-356">It is recommended to use Azure AD to authenticate clients, and X.509 certificates for node-to-node security.</span></span> <span data-ttu-id="66f61-357">有关详细信息，请参阅 [Service Fabric 群集安全方案][sf-security]。</span><span class="sxs-lookup"><span data-stu-id="66f61-357">For more information, see [Service Fabric cluster security scenarios][sf-security].</span></span>

<span data-ttu-id="66f61-358">若要配置公共 HTTPS 终结点，请参阅[在服务清单中指定资源][sf-manifest-resources]。</span><span class="sxs-lookup"><span data-stu-id="66f61-358">To configure a public HTTPS endpoint, see [Specify resources in a service manifest][sf-manifest-resources].</span></span>

<span data-ttu-id="66f61-359">通过向群集添加 VM，可以横向扩展应用程序。</span><span class="sxs-lookup"><span data-stu-id="66f61-359">You can scale out the application by adding VMs to the cluster.</span></span> <span data-ttu-id="66f61-360">VM 规模集支持使用自动缩放规则，基于性能计数器进行自动缩放。</span><span class="sxs-lookup"><span data-stu-id="66f61-360">VM scale sets support auto-scaling using auto-scale rules based on performance counters.</span></span> <span data-ttu-id="66f61-361">有关详细信息，请参阅[使用自动缩放规则扩大或缩小 Service Fabric 群集][sf-auto-scale]。</span><span class="sxs-lookup"><span data-stu-id="66f61-361">For more information, see [Scale a Service Fabric cluster in or out using auto-scale rules][sf-auto-scale].</span></span>

<span data-ttu-id="66f61-362">运行群集时，应在某个中心位置从所有节点收集日志。</span><span class="sxs-lookup"><span data-stu-id="66f61-362">While the cluster is running, you should collect logs from all the nodes in a central location.</span></span> <span data-ttu-id="66f61-363">有关详细信息，请参阅[使用 Azure 诊断收集日志][sf-logs]。</span><span class="sxs-lookup"><span data-stu-id="66f61-363">For more information, see [Collect logs by using Azure Diagnostics][sf-logs].</span></span>   


## <a name="conclusion"></a><span data-ttu-id="66f61-364">结束语</span><span class="sxs-lookup"><span data-stu-id="66f61-364">Conclusion</span></span>

<span data-ttu-id="66f61-365">将 Surveys 应用程序移植到 Service Fabric 的过程非常简单。</span><span class="sxs-lookup"><span data-stu-id="66f61-365">Porting the Surveys application to Service Fabric was fairly straightforward.</span></span> <span data-ttu-id="66f61-366">概括来说，我们执行了以下操作：</span><span class="sxs-lookup"><span data-stu-id="66f61-366">To summarize, we did the following:</span></span>

- <span data-ttu-id="66f61-367">将角色转换成无状态服务。</span><span class="sxs-lookup"><span data-stu-id="66f61-367">Converted the roles to stateless services.</span></span>
- <span data-ttu-id="66f61-368">将 Web 前端转换成 ASP.NET Core。</span><span class="sxs-lookup"><span data-stu-id="66f61-368">Converted the web front ends to ASP.NET Core.</span></span>
- <span data-ttu-id="66f61-369">将打包和配置文件更改为 Service Fabric 模型。</span><span class="sxs-lookup"><span data-stu-id="66f61-369">Changed the packaging and configuration files to the Service Fabric model.</span></span>

<span data-ttu-id="66f61-370">另外，将部署从云服务更改为在 VM 规模集中运行的 Service Fabric 群集。</span><span class="sxs-lookup"><span data-stu-id="66f61-370">In addition, the deployment changed from Cloud Services to a Service Fabric cluster running in a VM Scale Set.</span></span>

## <a name="next-steps"></a><span data-ttu-id="66f61-371">后续步骤</span><span class="sxs-lookup"><span data-stu-id="66f61-371">Next steps</span></span>

<span data-ttu-id="66f61-372">成功移植 Surveys 应用程序后，Tailspin 可以利用独立服务部署和版本控制等 Service Fabric 功能。</span><span class="sxs-lookup"><span data-stu-id="66f61-372">Now that the Surveys application has been successfully ported, Tailspin wants to take advantage of Service Fabric features such as independent service deployment and versioning.</span></span> <span data-ttu-id="66f61-373">[重构从 Azure 云服务迁移的 Azure Service Fabric 应用程序][refactor-surveys]中介绍了 Tailspin 如何将这些服务分解成更精细的体系结构，以利用这些 Service Fabric 功能</span><span class="sxs-lookup"><span data-stu-id="66f61-373">Learn how Tailspin decomposed these services to a more granular architecture to take advantage of these Service Fabric features in [Refactor an Azure Service Fabric Application migrated from Azure Cloud Services][refactor-surveys]</span></span>

<!-- links -->

[application-gateway]: /azure/application-gateway/
[aspnet-core]: /aspnet/core/
[aspnet-webapi]: https://www.asp.net/web-api
[aspnet-migration]: /aspnet/core/migration/mvc
[aspnet-hosting]: /aspnet/core/fundamentals/hosting
[aspnet-webapi]: https://www.asp.net/web-api
[azure-deployment-models]: /azure/azure-resource-manager/resource-manager-deployment-model
[cloud-service-autoscale]: /azure/cloud-services/cloud-services-how-to-scale-portal
[cloud-service-config]: /azure/cloud-services/cloud-services-model-and-package
[cloud-service-endpoints]: /azure/cloud-services/cloud-services-enable-communication-role-instances#worker-roles-vs-web-roles
[kestrel]: https://docs.microsoft.com/aspnet/core/fundamentals/servers/kestrel
[lb-probes]: /azure/load-balancer/load-balancer-custom-probe-overview
[owin]: https://www.asp.net/aspnet/overview/owin-and-katana
[refactor-surveys]: refactor-migrated-app.md
[sample-code]: https://github.com/mspnp/cloud-services-to-service-fabric
[sf-application-model]: /azure/service-fabric/service-fabric-application-model
[sf-aspnet-core]: /azure/service-fabric/service-fabric-add-a-web-frontend
[sf-auto-scale]: /azure/service-fabric/service-fabric-cluster-scale-up-down
[sf-compare-cloud-services]: /azure/service-fabric/service-fabric-cloud-services-migration-differences
[sf-connect-and-communicate]: /azure/service-fabric/service-fabric-connect-and-communicate-with-services
[sf-containers]: /azure/service-fabric/service-fabric-containers-overview
[sf-logs]: /azure/service-fabric/service-fabric-diagnostics-how-to-setup-wad
[sf-manifest-resources]: /azure/service-fabric/service-fabric-service-manifest-resources
[sf-migration]: /azure/service-fabric/service-fabric-cloud-services-migration-worker-role-stateless-service
[sf-multiple-environments]: /azure/service-fabric/service-fabric-manage-multiple-environment-app-configuration
[sf-node-types]: /azure/service-fabric/service-fabric-cluster-nodetypes
[sf-overview]: /azure/service-fabric/service-fabric-overview
[sf-placement-constraints]: /azure/service-fabric/service-fabric-cluster-resource-manager-cluster-description
[sf-reliable-collections]: /azure/service-fabric/service-fabric-reliable-services-reliable-collections
[sf-reliable-services]: /azure/service-fabric/service-fabric-reliable-services-introduction
[sf-reverse-proxy]: /azure/service-fabric/service-fabric-reverseproxy
[sf-security]: /azure/service-fabric/service-fabric-cluster-security
[sf-why-microservices]: /azure/service-fabric/service-fabric-overview-microservices
[tailspin-book]: https://msdn.microsoft.com/library/ff966499.aspx
[tailspin-scenario]: https://msdn.microsoft.com/library/hh534482.aspx
[unity]: https://msdn.microsoft.com/library/ff647202.aspx
[vm-scale-sets]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
