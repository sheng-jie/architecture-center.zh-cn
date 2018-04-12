---
title: 可缩放的 Web 应用程序
description: 提高在 Microsoft Azure 中运行的 Web 应用程序的可伸缩性。
author: MikeWasson
pnp.series.title: Azure App Service
pnp.series.prev: basic-web-app
pnp.series.next: multi-region-web-app
ms.date: 11/23/2016
cardTitle: Improve scalability
ms.openlocfilehash: 6459acebfa25491332e2118b9e8fe51d5fc79ff3
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/06/2018
---
# <a name="improve-scalability-in-a-web-application"></a><span data-ttu-id="96d8f-103">提高 Web 应用程序的可伸缩性</span><span class="sxs-lookup"><span data-stu-id="96d8f-103">Improve scalability in a web application</span></span>

<span data-ttu-id="96d8f-104">此参考体系结构显示的经验证做法可以改进 Azure 应用服务 Web 应用程序的可伸缩性和性能。</span><span class="sxs-lookup"><span data-stu-id="96d8f-104">This reference architecture shows proven practices for improving scalability and performance in an Azure App Service web application.</span></span>

<span data-ttu-id="96d8f-105">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="96d8f-105">![[0]][0]</span></span>

<span data-ttu-id="96d8f-106">下载此体系结构的 [Visio 文件][visio-download]。</span><span class="sxs-lookup"><span data-stu-id="96d8f-106">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="96d8f-107">体系结构</span><span class="sxs-lookup"><span data-stu-id="96d8f-107">Architecture</span></span>  

<span data-ttu-id="96d8f-108">此体系结构基于[基本 Web 应用程序][basic-web-app]中显示的体系结构。</span><span class="sxs-lookup"><span data-stu-id="96d8f-108">This architecture builds on the one shown in [Basic web application][basic-web-app].</span></span> <span data-ttu-id="96d8f-109">它包括以下组件：</span><span class="sxs-lookup"><span data-stu-id="96d8f-109">It includes the following components:</span></span>

* <span data-ttu-id="96d8f-110">资源组。</span><span class="sxs-lookup"><span data-stu-id="96d8f-110">**Resource group**.</span></span> <span data-ttu-id="96d8f-111">[资源组][resource-group]是 Azure 资源的逻辑容器。</span><span class="sxs-lookup"><span data-stu-id="96d8f-111">A [resource group][resource-group] is a logical container for Azure resources.</span></span>
* <span data-ttu-id="96d8f-112">**[Web 应用][app-service-web-app]**和 **[API 应用][app-service-api-app]**。</span><span class="sxs-lookup"><span data-stu-id="96d8f-112">**[Web app][app-service-web-app]** and **[API app][app-service-api-app]**.</span></span> <span data-ttu-id="96d8f-113">典型的现代应用程序可能包括一个网站以及一个或多个 RESTful Web API。</span><span class="sxs-lookup"><span data-stu-id="96d8f-113">A typical modern application might include both a website and one or more RESTful web APIs.</span></span> <span data-ttu-id="96d8f-114">Web API 可供浏览器客户端通过 AJAX 来使用，也可供本机客户端应用程序或服务器端应用程序使用。</span><span class="sxs-lookup"><span data-stu-id="96d8f-114">A web API might be consumed by browser clients through AJAX, by native client applications, or by server-side applications.</span></span> <span data-ttu-id="96d8f-115">有关设计 Web API 的注意事项，请参阅 [API 设计指南][api-guidance]。</span><span class="sxs-lookup"><span data-stu-id="96d8f-115">For considerations on designing web APIs, see [API design guidance][api-guidance].</span></span>    
* <span data-ttu-id="96d8f-116">**WebJob**。</span><span class="sxs-lookup"><span data-stu-id="96d8f-116">**WebJob**.</span></span> <span data-ttu-id="96d8f-117">使用 [Azure WebJobs][webjobs] 在后台运行长时间运行的任务。</span><span class="sxs-lookup"><span data-stu-id="96d8f-117">Use [Azure WebJobs][webjobs] to run long-running tasks in the background.</span></span> <span data-ttu-id="96d8f-118">WebJobs 可以按计划运行、持续运行或者以响应触发器的方式运行，例如将消息放置到队列中。</span><span class="sxs-lookup"><span data-stu-id="96d8f-118">WebJobs can run on a schedule, continously, or in response to a trigger, such as putting a message on a queue.</span></span> <span data-ttu-id="96d8f-119">WebJob 可在应用服务应用上下文中作为后台进程运行。</span><span class="sxs-lookup"><span data-stu-id="96d8f-119">A WebJob runs as a background process in the context of an App Service app.</span></span>
* <span data-ttu-id="96d8f-120">**队列**。</span><span class="sxs-lookup"><span data-stu-id="96d8f-120">**Queue**.</span></span> <span data-ttu-id="96d8f-121">在此处显示的体系结构中，应用程序通过向 [Azure 队列存储][queue-storage]队列放置消息，将后台任务排队。</span><span class="sxs-lookup"><span data-stu-id="96d8f-121">In the architecture shown here, the application queues background tasks by putting a message onto an [Azure Queue storage][queue-storage] queue.</span></span> <span data-ttu-id="96d8f-122">消息触发 WebJob 中的函数。</span><span class="sxs-lookup"><span data-stu-id="96d8f-122">The message triggers a function in the WebJob.</span></span> <span data-ttu-id="96d8f-123">也可使用服务总线队列。</span><span class="sxs-lookup"><span data-stu-id="96d8f-123">Alternatively, you can use Service Bus queues.</span></span> <span data-ttu-id="96d8f-124">如需比较，请参阅 [Azure 队列和服务总线队列 - 比较与对照][queues-compared]。</span><span class="sxs-lookup"><span data-stu-id="96d8f-124">For a comparison, see [Azure Queues and Service Bus queues - compared and contrasted][queues-compared].</span></span>
* <span data-ttu-id="96d8f-125">**缓存**。</span><span class="sxs-lookup"><span data-stu-id="96d8f-125">**Cache**.</span></span> <span data-ttu-id="96d8f-126">在 [Azure Redis 缓存][azure-redis]中存储半静态数据。</span><span class="sxs-lookup"><span data-stu-id="96d8f-126">Store semi-static data in [Azure Redis Cache][azure-redis].</span></span>  
* <span data-ttu-id="96d8f-127"><strong>CDN</strong>。</span><span class="sxs-lookup"><span data-stu-id="96d8f-127"><strong>CDN</strong>.</span></span> <span data-ttu-id="96d8f-128">使用 [Azure 内容交付网络][azure-cdn] (CDN) 缓存公开提供的内容，以便降低延迟并加快内容交付速度。</span><span class="sxs-lookup"><span data-stu-id="96d8f-128">Use [Azure Content Delivery Network][azure-cdn] (CDN) to cache publicly available content for lower latency and faster delivery of content.</span></span>
* <span data-ttu-id="96d8f-129">**数据存储**。</span><span class="sxs-lookup"><span data-stu-id="96d8f-129">**Data storage**.</span></span> <span data-ttu-id="96d8f-130">对关系数据使用 [Azure SQL 数据库][sql-db]。</span><span class="sxs-lookup"><span data-stu-id="96d8f-130">Use [Azure SQL Database][sql-db] for relational data.</span></span> <span data-ttu-id="96d8f-131">对于非关系数据，可考虑使用 NoSQL 存储，例如 [Cosmos DB][cosmosdb]。</span><span class="sxs-lookup"><span data-stu-id="96d8f-131">For non-relational data, consider a NoSQL store, such as [Cosmos DB][cosmosdb].</span></span>
* <span data-ttu-id="96d8f-132">**Azure 搜索**。</span><span class="sxs-lookup"><span data-stu-id="96d8f-132">**Azure Search**.</span></span> <span data-ttu-id="96d8f-133">使用 [Azure 搜索][azure-search]添加搜索功能，例如搜索建议、模糊搜索、特定于语言的搜索。</span><span class="sxs-lookup"><span data-stu-id="96d8f-133">Use [Azure Search][azure-search] to add search functionality such as search suggestions, fuzzy search, and language-specific search.</span></span> <span data-ttu-id="96d8f-134">Azure 搜索通常与其他数据存储结合使用，尤其是在主数据存储对一致性要求严格的情况下。</span><span class="sxs-lookup"><span data-stu-id="96d8f-134">Azure Search is typically used in conjunction with another data store, especially if the primary data store requires strict consistency.</span></span> <span data-ttu-id="96d8f-135">此方法将权威数据存储在其他数据存储中，将搜索索引存储在 Azure 搜索中。</span><span class="sxs-lookup"><span data-stu-id="96d8f-135">In this approach, store authoritative data in the other data store and the search index in Azure Search.</span></span> <span data-ttu-id="96d8f-136">也可使用 Azure 搜索合并来自多个数据存储的单一搜索索引。</span><span class="sxs-lookup"><span data-stu-id="96d8f-136">Azure Search can also be used to consolidate a single search index from multiple data stores.</span></span>  
* <span data-ttu-id="96d8f-137">**电子邮件/短信**。</span><span class="sxs-lookup"><span data-stu-id="96d8f-137">**Email/SMS**.</span></span> <span data-ttu-id="96d8f-138">使用第三方服务（例如 SendGrid 或 Twilio）发送电子邮件或短信，而不是将此功能直接内置到应用程序中。</span><span class="sxs-lookup"><span data-stu-id="96d8f-138">Use a third-party service such as SendGrid or Twilio to send email or SMS messages instead of building this functionality directly into the application.</span></span>
* <span data-ttu-id="96d8f-139">**Azure DNS**。</span><span class="sxs-lookup"><span data-stu-id="96d8f-139">**Azure DNS**.</span></span> <span data-ttu-id="96d8f-140">[Azure DNS][azure-dns] 是 DNS 域的托管服务，它使用 Microsoft Azure 基础结构提供名称解析。</span><span class="sxs-lookup"><span data-stu-id="96d8f-140">[Azure DNS][azure-dns] is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure.</span></span> <span data-ttu-id="96d8f-141">通过在 Azure 中托管域，可以使用与其他 Azure 服务相同的凭据、API、工具和计费来管理 DNS 记录。</span><span class="sxs-lookup"><span data-stu-id="96d8f-141">By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools, and billing as your other Azure services.</span></span>

## <a name="recommendations"></a><span data-ttu-id="96d8f-142">建议</span><span class="sxs-lookup"><span data-stu-id="96d8f-142">Recommendations</span></span>

<span data-ttu-id="96d8f-143">你的要求可能不同于此处描述的体系结构。</span><span class="sxs-lookup"><span data-stu-id="96d8f-143">Your requirements might differ from the architecture described here.</span></span> <span data-ttu-id="96d8f-144">一开始可使用此部分的建议。</span><span class="sxs-lookup"><span data-stu-id="96d8f-144">Use the recommendations in this section as a starting point.</span></span>

### <a name="app-service-apps"></a><span data-ttu-id="96d8f-145">应用服务应用</span><span class="sxs-lookup"><span data-stu-id="96d8f-145">App Service apps</span></span>
<span data-ttu-id="96d8f-146">建议以独立应用服务应用的形式创建 Web 应用程序和 Web API。</span><span class="sxs-lookup"><span data-stu-id="96d8f-146">We recommend creating the web application and the web API as separate App Service apps.</span></span> <span data-ttu-id="96d8f-147">此设计允许你按独立的应用服务计划运行它们，以便对它们进行单独缩放。</span><span class="sxs-lookup"><span data-stu-id="96d8f-147">This design lets you run them in separate App Service plans so they can be scaled independently.</span></span> <span data-ttu-id="96d8f-148">如果一开始不需要该级别的可伸缩性，可以先将应用部署到同一计划中，再在以后根据需要将其移至独立的计划中。</span><span class="sxs-lookup"><span data-stu-id="96d8f-148">If you don't need that level of scalability initially, you can deploy the apps into the same plan and move them into separate plans later if necessary.</span></span>

> [!NOTE]
> <span data-ttu-id="96d8f-149">“基本”、“标准”和“高级”计划按计划中的 VM 实例计费，而不是按应用计费。</span><span class="sxs-lookup"><span data-stu-id="96d8f-149">For the Basic, Standard, and Premium plans, you are billed for the VM instances in the plan, not per app.</span></span> <span data-ttu-id="96d8f-150">请参阅[应用服务定价][app-service-pricing]</span><span class="sxs-lookup"><span data-stu-id="96d8f-150">See [App Service Pricing][app-service-pricing]</span></span>
> 
> 

<span data-ttu-id="96d8f-151">若要使用应用服务移动应用的“简易表”或“简易 API”功能，请创建适合此用途的独立应用服务应用。</span><span class="sxs-lookup"><span data-stu-id="96d8f-151">If you intend to use the *Easy Tables* or *Easy APIs* features of App Service Mobile Apps, create a separate App Service app for this purpose.</span></span>  <span data-ttu-id="96d8f-152">这些功能需要通过特定的应用程序框架来启用。</span><span class="sxs-lookup"><span data-stu-id="96d8f-152">These features rely on a specific application framework to enable them.</span></span>

### <a name="webjobs"></a><span data-ttu-id="96d8f-153">Web 作业</span><span class="sxs-lookup"><span data-stu-id="96d8f-153">WebJobs</span></span>
<span data-ttu-id="96d8f-154">考虑在独立的应用服务计划中将资源密集型 WebJobs 部署到空的应用服务应用。</span><span class="sxs-lookup"><span data-stu-id="96d8f-154">Consider deploying resource intensive WebJobs to an empty App Service app within a separate App Service plan.</span></span> <span data-ttu-id="96d8f-155">这样可以为 WebJob 提供专用实例。</span><span class="sxs-lookup"><span data-stu-id="96d8f-155">This provides dedicated instances for the WebJob.</span></span> <span data-ttu-id="96d8f-156">请参阅[后台作业指南][webjobs-guidance]。</span><span class="sxs-lookup"><span data-stu-id="96d8f-156">See [Background jobs guidance][webjobs-guidance].</span></span>  

### <a name="cache"></a><span data-ttu-id="96d8f-157">缓存</span><span class="sxs-lookup"><span data-stu-id="96d8f-157">Cache</span></span>
<span data-ttu-id="96d8f-158">可以使用 [Azure Redis 缓存][azure-redis]来缓存一些数据，从而改进性能和可伸缩性。</span><span class="sxs-lookup"><span data-stu-id="96d8f-158">You can improve performance and scalability by using [Azure Redis Cache][azure-redis] to cache some data.</span></span> <span data-ttu-id="96d8f-159">考虑将 Redis 缓存用于：</span><span class="sxs-lookup"><span data-stu-id="96d8f-159">Consider using Redis Cache for:</span></span>

* <span data-ttu-id="96d8f-160">半静态事务数据。</span><span class="sxs-lookup"><span data-stu-id="96d8f-160">Semi-static transaction data.</span></span>
* <span data-ttu-id="96d8f-161">会话状态。</span><span class="sxs-lookup"><span data-stu-id="96d8f-161">Session state.</span></span>
* <span data-ttu-id="96d8f-162">HTML 输出。</span><span class="sxs-lookup"><span data-stu-id="96d8f-162">HTML output.</span></span> <span data-ttu-id="96d8f-163">这适用于可呈现复杂 HTML 输出的应用程序。</span><span class="sxs-lookup"><span data-stu-id="96d8f-163">This can be useful in applications that render complex HTML output.</span></span>

<span data-ttu-id="96d8f-164">有关缓存策略设计的更详细指南，请参阅[缓存指南][caching-guidance]。</span><span class="sxs-lookup"><span data-stu-id="96d8f-164">For more detailed guidance on designing a caching strategy, see [Caching guidance][caching-guidance].</span></span>

### <a name="cdn"></a><span data-ttu-id="96d8f-165">CDN</span><span class="sxs-lookup"><span data-stu-id="96d8f-165">CDN</span></span>
<span data-ttu-id="96d8f-166">使用 [Azure CDN][azure-cdn] 来缓存静态内容。</span><span class="sxs-lookup"><span data-stu-id="96d8f-166">Use [Azure CDN][azure-cdn] to cache static content.</span></span> <span data-ttu-id="96d8f-167">CDN 的主要优势是降低用户的延迟，因为内容缓存在靠近用户的边缘服务器上。</span><span class="sxs-lookup"><span data-stu-id="96d8f-167">The main benefit of a CDN is to reduce latency for users, because content is cached at an edge server that is geographically close to the user.</span></span> <span data-ttu-id="96d8f-168">CDN 还可以减轻应用程序的负载，因为相应的流量不是由应用程序处理。</span><span class="sxs-lookup"><span data-stu-id="96d8f-168">CDN can also reduce load on the application, because that traffic is not being handled by the application.</span></span>

<span data-ttu-id="96d8f-169">如果应用包含的内容大部分为静态页面，可考虑使用 [CDN 来缓存整个应用][cdn-app-service]。</span><span class="sxs-lookup"><span data-stu-id="96d8f-169">If your app consists mostly of static pages, consider using [CDN to cache the entire app][cdn-app-service].</span></span> <span data-ttu-id="96d8f-170">否则，请将静态内容（例如图像、CSS 和 HTML 文件）置于 [Azure 存储中，并使用 CDN 缓存这些文件][cdn-storage-account]。</span><span class="sxs-lookup"><span data-stu-id="96d8f-170">Otherwise, put static content such as images, CSS, and HTML files, into [Azure Storage and use CDN to cache those files][cdn-storage-account].</span></span>

> [!NOTE]
> <span data-ttu-id="96d8f-171">Azure CDN 不能提供需要身份验证的内容。</span><span class="sxs-lookup"><span data-stu-id="96d8f-171">Azure CDN cannot serve content that requires authentication.</span></span>
> 
> 

<span data-ttu-id="96d8f-172">如需更详细的指南，请参阅[内容交付网络 (CDN) 指南][cdn-guidance]。</span><span class="sxs-lookup"><span data-stu-id="96d8f-172">For more detailed guidance, see [Content Delivery Network (CDN) guidance][cdn-guidance].</span></span>

### <a name="storage"></a><span data-ttu-id="96d8f-173">存储</span><span class="sxs-lookup"><span data-stu-id="96d8f-173">Storage</span></span>
<span data-ttu-id="96d8f-174">现代应用程序通常处理大量的数据。</span><span class="sxs-lookup"><span data-stu-id="96d8f-174">Modern applications often process large amounts of data.</span></span> <span data-ttu-id="96d8f-175">若要进行适合云的缩放，请务必选择适当的存储类型。</span><span class="sxs-lookup"><span data-stu-id="96d8f-175">In order to scale for the cloud, it's important to choose the right storage type.</span></span> <span data-ttu-id="96d8f-176">以下是一些基线建议。</span><span class="sxs-lookup"><span data-stu-id="96d8f-176">Here are some baseline recommendations.</span></span> 

| <span data-ttu-id="96d8f-177">要存储的内容</span><span class="sxs-lookup"><span data-stu-id="96d8f-177">What you want to store</span></span> | <span data-ttu-id="96d8f-178">示例</span><span class="sxs-lookup"><span data-stu-id="96d8f-178">Example</span></span> | <span data-ttu-id="96d8f-179">建议的存储</span><span class="sxs-lookup"><span data-stu-id="96d8f-179">Recommended storage</span></span> |
| --- | --- | --- |
| <span data-ttu-id="96d8f-180">文件</span><span class="sxs-lookup"><span data-stu-id="96d8f-180">Files</span></span> |<span data-ttu-id="96d8f-181">图像、文档、PDF</span><span class="sxs-lookup"><span data-stu-id="96d8f-181">Images, documents, PDFs</span></span> |<span data-ttu-id="96d8f-182">Azure Blob 存储</span><span class="sxs-lookup"><span data-stu-id="96d8f-182">Azure Blob Storage</span></span> |
| <span data-ttu-id="96d8f-183">键值对</span><span class="sxs-lookup"><span data-stu-id="96d8f-183">Key/Value pairs</span></span> |<span data-ttu-id="96d8f-184">按用户 ID 查找的用户配置文件数据</span><span class="sxs-lookup"><span data-stu-id="96d8f-184">User profile data looked up by user ID</span></span> |<span data-ttu-id="96d8f-185">Azure 表存储</span><span class="sxs-lookup"><span data-stu-id="96d8f-185">Azure Table storage</span></span> |
| <span data-ttu-id="96d8f-186">旨在触发进一步处理的短消息</span><span class="sxs-lookup"><span data-stu-id="96d8f-186">Short messages intended to trigger further processing</span></span> |<span data-ttu-id="96d8f-187">订单请求</span><span class="sxs-lookup"><span data-stu-id="96d8f-187">Order requests</span></span> |<span data-ttu-id="96d8f-188">Azure 队列存储、服务总线队列或服务总线主题</span><span class="sxs-lookup"><span data-stu-id="96d8f-188">Azure Queue storage, Service Bus queue, or Service Bus topic</span></span> |
| <span data-ttu-id="96d8f-189">架构灵活但只需要进行基本查询的非关系数据</span><span class="sxs-lookup"><span data-stu-id="96d8f-189">Non-relational data with a flexible schema requiring basic querying</span></span> |<span data-ttu-id="96d8f-190">产品目录</span><span class="sxs-lookup"><span data-stu-id="96d8f-190">Product catalog</span></span> |<span data-ttu-id="96d8f-191">文档数据库，例如 Azure Cosmos DB、MongoDB 或 Apache CouchDB</span><span class="sxs-lookup"><span data-stu-id="96d8f-191">Document database, such as Azure Cosmos DB, MongoDB, or Apache CouchDB</span></span> |
| <span data-ttu-id="96d8f-192">需要更丰富的查询支持、严格的架构和/或高一致性的关系数据</span><span class="sxs-lookup"><span data-stu-id="96d8f-192">Relational data requiring richer query support, strict schema, and/or strong consistency</span></span> |<span data-ttu-id="96d8f-193">产品清单</span><span class="sxs-lookup"><span data-stu-id="96d8f-193">Product inventory</span></span> |<span data-ttu-id="96d8f-194">Azure SQL 数据库</span><span class="sxs-lookup"><span data-stu-id="96d8f-194">Azure SQL Database</span></span> |

## <a name="scalability-considerations"></a><span data-ttu-id="96d8f-195">可伸缩性注意事项</span><span class="sxs-lookup"><span data-stu-id="96d8f-195">Scalability considerations</span></span>

<span data-ttu-id="96d8f-196">Azure 应用服务的主要优势是能够根据负载缩放应用程序。</span><span class="sxs-lookup"><span data-stu-id="96d8f-196">A major benefit of Azure App Service is the ability to scale your application based on load.</span></span> <span data-ttu-id="96d8f-197">下面是在计划缩放应用程序时需要考虑的一些注意事项。</span><span class="sxs-lookup"><span data-stu-id="96d8f-197">Here are some considerations to keep in mind when planning to scale your application.</span></span>

### <a name="app-service-app"></a><span data-ttu-id="96d8f-198">应用服务应用</span><span class="sxs-lookup"><span data-stu-id="96d8f-198">App Service app</span></span>
<span data-ttu-id="96d8f-199">如果解决方案包括多个应用服务应用，可考虑将其部署到不同的应用服务计划。</span><span class="sxs-lookup"><span data-stu-id="96d8f-199">If your solution includes several App Service apps, consider deploying them to separate App Service plans.</span></span> <span data-ttu-id="96d8f-200">这种方法允许独立缩放应用，因为应用在不同的实例上运行。</span><span class="sxs-lookup"><span data-stu-id="96d8f-200">This approach enables you to scale them independently because they run on separate instances.</span></span> 

<span data-ttu-id="96d8f-201">同样，可以考虑将 WebJob 置于其自己的计划中，使后台任务不在处理 HTTP 请求的实例上运行。</span><span class="sxs-lookup"><span data-stu-id="96d8f-201">Similarly, consider putting a WebJob into its own plan so that background tasks don't run on the same instances that handle HTTP requests.</span></span>  

### <a name="sql-database"></a><span data-ttu-id="96d8f-202">SQL 数据库</span><span class="sxs-lookup"><span data-stu-id="96d8f-202">SQL Database</span></span>
<span data-ttu-id="96d8f-203">通过数据库分片，增加 SQL 数据库的可伸缩性。</span><span class="sxs-lookup"><span data-stu-id="96d8f-203">Increase scalability of a SQL database by *sharding* the database.</span></span> <span data-ttu-id="96d8f-204">分片是指将数据库水平分区。</span><span class="sxs-lookup"><span data-stu-id="96d8f-204">Sharding refers to partitioning the database horizontally.</span></span> <span data-ttu-id="96d8f-205">可以通过分片，使用[弹性数据库工具][sql-elastic]横向扩展数据库。</span><span class="sxs-lookup"><span data-stu-id="96d8f-205">Sharding allows you to scale out the database horizontally using [Elastic Database tools][sql-elastic].</span></span> <span data-ttu-id="96d8f-206">分片的潜在好处包括：</span><span class="sxs-lookup"><span data-stu-id="96d8f-206">Potential benefits of sharding include:</span></span>

- <span data-ttu-id="96d8f-207">提高事务吞吐量。</span><span class="sxs-lookup"><span data-stu-id="96d8f-207">Better transaction throughput.</span></span>
- <span data-ttu-id="96d8f-208">对数据子集运行查询可以提高速度。</span><span class="sxs-lookup"><span data-stu-id="96d8f-208">Queries can run faster over a subset of the data.</span></span>

### <a name="azure-search"></a><span data-ttu-id="96d8f-209">Azure 搜索</span><span class="sxs-lookup"><span data-stu-id="96d8f-209">Azure Search</span></span>
<span data-ttu-id="96d8f-210">Azure 搜索没有在主数据存储中执行复杂的数据搜索所需的开销，并可通过缩放来处理负载。</span><span class="sxs-lookup"><span data-stu-id="96d8f-210">Azure Search removes the overhead of performing complex data searches from the primary data store, and it can scale to handle load.</span></span> <span data-ttu-id="96d8f-211">请参阅[在 Azure 搜索中缩放用于查询和索引工作负荷的资源级别][azure-search-scaling]。</span><span class="sxs-lookup"><span data-stu-id="96d8f-211">See [Scale resource levels for query and indexing workloads in Azure Search][azure-search-scaling].</span></span>

## <a name="security-considerations"></a><span data-ttu-id="96d8f-212">安全注意事项</span><span class="sxs-lookup"><span data-stu-id="96d8f-212">Security considerations</span></span>
<span data-ttu-id="96d8f-213">本部分列出专门与本文中所述 Azure 服务相关的安全注意事项，</span><span class="sxs-lookup"><span data-stu-id="96d8f-213">This section lists security considerations that are specific to the Azure services described in this article.</span></span> <span data-ttu-id="96d8f-214">内容并非安全最佳做法的完整列表。</span><span class="sxs-lookup"><span data-stu-id="96d8f-214">It's not a complete list of security best practices.</span></span> <span data-ttu-id="96d8f-215">有关其他一些安全注意事项，请参阅[保护 Azure 应用服务中的应用][app-service-security]。</span><span class="sxs-lookup"><span data-stu-id="96d8f-215">For some additional security considerations, see [Secure an app in Azure App Service][app-service-security].</span></span>

### <a name="cross-origin-resource-sharing-cors"></a><span data-ttu-id="96d8f-216">跨源资源共享 (CORS)</span><span class="sxs-lookup"><span data-stu-id="96d8f-216">Cross-Origin Resource Sharing (CORS)</span></span>
<span data-ttu-id="96d8f-217">如果将网站和 Web API 作为独立应用创建，则网站不能向 API 进行客户端 AJAX 调用，除非启用 CORS。</span><span class="sxs-lookup"><span data-stu-id="96d8f-217">If you create a website and web API as separate apps, the website cannot make client-side AJAX calls to the API unless you enable CORS.</span></span>

> [!NOTE]
> <span data-ttu-id="96d8f-218">浏览器安全性将阻止网页向另一个域发出 AJAX 请求。</span><span class="sxs-lookup"><span data-stu-id="96d8f-218">Browser security prevents a web page from making AJAX requests to another domain.</span></span> <span data-ttu-id="96d8f-219">这种限制称为同域策略，可阻止恶意站点读取另一个站点中的敏感数据。</span><span class="sxs-lookup"><span data-stu-id="96d8f-219">This restriction is called the same-origin policy, and prevents a malicious site from reading sentitive data from another site.</span></span> <span data-ttu-id="96d8f-220">CORS 是一项 W3C 标准，可让服务器放宽同域策略，在拒绝某些跨域请求的同时，允许另一些跨域请求。</span><span class="sxs-lookup"><span data-stu-id="96d8f-220">CORS is a W3C standard that allows a server to relax the same-origin policy and allow some cross-origin requests while rejecting others.</span></span>
> 
> 

<span data-ttu-id="96d8f-221">应用服务内置了对 CORS 的支持，不需编写任何应用程序代码。</span><span class="sxs-lookup"><span data-stu-id="96d8f-221">App Services has built-in support for CORS, without needing to write any application code.</span></span> <span data-ttu-id="96d8f-222">请参阅[借助 CORS 从 JavaScript 使用 API 应用][cors]。</span><span class="sxs-lookup"><span data-stu-id="96d8f-222">See [Consume an API app from JavaScript using CORS][cors].</span></span> <span data-ttu-id="96d8f-223">请将网站添加到 API 允许域的列表。</span><span class="sxs-lookup"><span data-stu-id="96d8f-223">Add the website to the list of allowed origins for the API.</span></span>

### <a name="sql-database-encryption"></a><span data-ttu-id="96d8f-224">SQL 数据库加密</span><span class="sxs-lookup"><span data-stu-id="96d8f-224">SQL Database encryption</span></span>
<span data-ttu-id="96d8f-225">如需加密数据库中的静态数据，请使用[透明数据加密][sql-encryption]。</span><span class="sxs-lookup"><span data-stu-id="96d8f-225">Use [Transparent Data Encryption][sql-encryption] if you need to encrypt data at rest in the database.</span></span> <span data-ttu-id="96d8f-226">此功能对整个数据库（包括备份和事务日志文件）执行实时加密和解密，不需对应用程序进行更改。</span><span class="sxs-lookup"><span data-stu-id="96d8f-226">This feature performs real-time encryption and decryption of an entire database (including backups and transaction log files) and requires no changes to the application.</span></span> <span data-ttu-id="96d8f-227">加密会增加一些延迟，因此最好将必须保护的数据单独放置在自己的数据库中，仅对该数据库启用加密。</span><span class="sxs-lookup"><span data-stu-id="96d8f-227">Encryption does add some latency, so it's a good practice to separate the data that must be secure into its own database and enable encryption only for that database.</span></span>  
  

<!-- links -->

[api-guidance]: ../../best-practices/api-design.md
[app-service-security]: /azure/app-service-web/web-sites-security
[app-service-web-app]: /azure/app-service-web/app-service-web-overview
[app-service-api-app]: /azure/app-service-api/app-service-api-apps-why-best-platform
[app-service-pricing]: https://azure.microsoft.com/pricing/details/app-service/
[azure-cdn]: https://azure.microsoft.com/services/cdn/
[azure-dns]: /azure/dns/dns-overview
[azure-redis]: https://azure.microsoft.com/services/cache/
[azure-search]: https://azure.microsoft.com/documentation/services/search/
[azure-search-scaling]: /azure/search/search-capacity-planning
[background-jobs]: ../../best-practices/background-jobs.md
[basic-web-app]: basic-web-app.md
[basic-web-app-scalability]: basic-web-app.md#scalability-considerations
[caching-guidance]: ../../best-practices/caching.md
[cdn-app-service]: /azure/app-service-web/cdn-websites-with-cdn
[cdn-storage-account]: /azure/cdn/cdn-create-a-storage-account-with-cdn
[cdn-guidance]: ../../best-practices/cdn.md
[cors]: /azure/app-service-api/app-service-api-cors-consume-javascript
[cosmosdb]: /azure/cosmos-db/
[queue-storage]: /azure/storage/storage-dotnet-how-to-use-queues
[queues-compared]: /azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted
[resource-group]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[sql-elastic]: /azure/sql-database/sql-database-elastic-scale-introduction
[sql-encryption]: https://msdn.microsoft.com/library/dn948096.aspx
[tm]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/app-service-reference-architectures.vsdx
[web-app-multi-region]: ./multi-region.md
[webjobs-guidance]: ../../best-practices/background-jobs.md
[webjobs]: /azure/app-service/app-service-webjobs-readme
[0]: ./images/scalable-web-app.png "Azure 中改进了可伸缩性的 Web 应用程序"
