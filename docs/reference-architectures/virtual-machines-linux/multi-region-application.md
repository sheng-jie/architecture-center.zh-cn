---
title: "在多个 Azure 区域中运行 Linux VM 以实现高可用性"
description: "如何在 Azure 上的多个区域中部署 VM 以实现高可用性和复原能力。"
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Linux VM workloads
pnp.series.prev: n-tier
ms.openlocfilehash: 3b68f6fc79ba4b29e41ba2b04537b834bb8859b0
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="run-linux-vms-in-multiple-regions-for-high-availability"></a><span data-ttu-id="0c875-103">在多个区域中运行 Linux VM 以实现高可用性</span><span class="sxs-lookup"><span data-stu-id="0c875-103">Run Linux VMs in multiple regions for high availability</span></span>

<span data-ttu-id="0c875-104">此参考体系结构展示了在多个 Azure 区域中运行 N 层应用程序以实现可用性和强健的灾难恢复基础结构的一组经过实践检验的做法。</span><span class="sxs-lookup"><span data-stu-id="0c875-104">This reference architecture shows a set of proven practices for running an N-tier application in multiple Azure regions, in order to achieve availability and a robust disaster recovery infrastructure.</span></span> 

<span data-ttu-id="0c875-105">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="0c875-105">![[0]][0]</span></span>

<span data-ttu-id="0c875-106">下载此体系结构的 [Visio 文件][visio-download]。</span><span class="sxs-lookup"><span data-stu-id="0c875-106">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="0c875-107">体系结构</span><span class="sxs-lookup"><span data-stu-id="0c875-107">Architecture</span></span> 

<span data-ttu-id="0c875-108">此体系结构构建于[运行用于 N 层应用程序的 Linux VM](n-tier.md) 中所示的体系结构基础之上。</span><span class="sxs-lookup"><span data-stu-id="0c875-108">This architecture builds on the one shown in [Run Linux VMs for an N-tier application](n-tier.md).</span></span> 

* <span data-ttu-id="0c875-109">**主要和次要区域**。</span><span class="sxs-lookup"><span data-stu-id="0c875-109">**Primary and secondary regions**.</span></span> <span data-ttu-id="0c875-110">使用两个区域来实现更高的可用性。</span><span class="sxs-lookup"><span data-stu-id="0c875-110">Use two regions to achieve higher availability.</span></span> <span data-ttu-id="0c875-111">一个是主区域。另一个区域用于故障转移。</span><span class="sxs-lookup"><span data-stu-id="0c875-111">One is the primary region.The other region is for failover.</span></span>
* <span data-ttu-id="0c875-112">**Azure 流量管理器**。</span><span class="sxs-lookup"><span data-stu-id="0c875-112">**Azure Traffic Manager**.</span></span> <span data-ttu-id="0c875-113">[流量管理器][traffic-manager]将传入请求路由到其中一个区域。</span><span class="sxs-lookup"><span data-stu-id="0c875-113">[Traffic Manager][traffic-manager] routes incoming requests to one of the regions.</span></span> <span data-ttu-id="0c875-114">在正常运行期间，它将请求路由到主要区域。</span><span class="sxs-lookup"><span data-stu-id="0c875-114">During normal operations, it routes requests to the primary region.</span></span> <span data-ttu-id="0c875-115">如果该区域变得不可用，则流量管理器将故障转移到次要区域。</span><span class="sxs-lookup"><span data-stu-id="0c875-115">If that region becomes unavailable, Traffic Manager fails over to the secondary region.</span></span> <span data-ttu-id="0c875-116">有关详细信息，请参阅[流量管理器配置](#traffic-manager-configuration)部分。</span><span class="sxs-lookup"><span data-stu-id="0c875-116">For more information, see the section [Traffic Manager configuration](#traffic-manager-configuration).</span></span>
* <span data-ttu-id="0c875-117">**资源组**。</span><span class="sxs-lookup"><span data-stu-id="0c875-117">**Resource groups**.</span></span> <span data-ttu-id="0c875-118">为主要区域、次要区域和流量管理器创建单独的[资源组][resource groups]。</span><span class="sxs-lookup"><span data-stu-id="0c875-118">Create separate [resource groups][resource groups] for the primary region, the secondary region, and for Traffic Manager.</span></span> <span data-ttu-id="0c875-119">这允许你将每个区域作为单个资源集合灵活进行管理。</span><span class="sxs-lookup"><span data-stu-id="0c875-119">This gives you the flexibility to manage each region as a single collection of resources.</span></span> <span data-ttu-id="0c875-120">例如，可以重新部署一个区域而无需关闭另一个区域。</span><span class="sxs-lookup"><span data-stu-id="0c875-120">For example, you could redeploy one region, without taking down the other one.</span></span> <span data-ttu-id="0c875-121">[链接资源组][resource-group-links]，以便可以运行查询来列出应用程序的所有资源。</span><span class="sxs-lookup"><span data-stu-id="0c875-121">[Link the resource groups][resource-group-links], so that you can run a query to list all the resources for the application.</span></span>
* <span data-ttu-id="0c875-122">**Vnet**。</span><span class="sxs-lookup"><span data-stu-id="0c875-122">**VNets**.</span></span> <span data-ttu-id="0c875-123">为每个区域创建一个单独的 VNet。</span><span class="sxs-lookup"><span data-stu-id="0c875-123">Create a separate VNet for each region.</span></span> <span data-ttu-id="0c875-124">请确保地址空间不重叠。</span><span class="sxs-lookup"><span data-stu-id="0c875-124">Make sure the address spaces do not overlap.</span></span>
* <span data-ttu-id="0c875-125">**Apache Cassandra**。</span><span class="sxs-lookup"><span data-stu-id="0c875-125">**Apache Cassandra**.</span></span> <span data-ttu-id="0c875-126">在跨 Azure 区域的数据中心部署 Cassandra 以实现高可用性。</span><span class="sxs-lookup"><span data-stu-id="0c875-126">Deploy Cassandra in data centers across Azure regions for high availability.</span></span> <span data-ttu-id="0c875-127">每个区域中，节点以带故障和升级域的机架感知模式配置，以实现区域内的复原能力。</span><span class="sxs-lookup"><span data-stu-id="0c875-127">Within each region, nodes are configured in rack-aware mode with fault and upgrade domains, for resiliency inside the region.</span></span>

## <a name="recommendations"></a><span data-ttu-id="0c875-128">建议</span><span class="sxs-lookup"><span data-stu-id="0c875-128">Recommendations</span></span>

<span data-ttu-id="0c875-129">与部署到单个区域相比，多区域体系结构可以提供更高的可用性。</span><span class="sxs-lookup"><span data-stu-id="0c875-129">A multi-region architecture can provide higher availability than deploying to a single region.</span></span> <span data-ttu-id="0c875-130">如果区域性故障影响了主要区域，则可以使用[流量管理器][traffic-manager]故障转移到次要区域。</span><span class="sxs-lookup"><span data-stu-id="0c875-130">If a regional outage affects the primary region, you can use [Traffic Manager][traffic-manager] to fail over to the secondary region.</span></span> <span data-ttu-id="0c875-131">当应用程序的单个子系统出现故障时，此体系结构可能也比较有用。</span><span class="sxs-lookup"><span data-stu-id="0c875-131">This architecture can also help if an individual subsystem of the application fails.</span></span>

<span data-ttu-id="0c875-132">有多种常规方法可跨区域实现高可用性：</span><span class="sxs-lookup"><span data-stu-id="0c875-132">There are several general approaches to achieving high availability across regions:</span></span>   

* <span data-ttu-id="0c875-133">主动/被动（采用热备用模式）。</span><span class="sxs-lookup"><span data-stu-id="0c875-133">Active/passive with hot standby.</span></span> <span data-ttu-id="0c875-134">流量将前往一个区域，而另一个区域将以热备用模式等待。</span><span class="sxs-lookup"><span data-stu-id="0c875-134">Traffic goes to one region, while the other waits on hot standby.</span></span> <span data-ttu-id="0c875-135">“热备用模式”意味着次要区域中的 VM 已被分配并总是处于运行状态。</span><span class="sxs-lookup"><span data-stu-id="0c875-135">Hot standby means the VMs in the secondary region are allocated and running at all times.</span></span>
* <span data-ttu-id="0c875-136">主动/被动（采用冷备用模式）。</span><span class="sxs-lookup"><span data-stu-id="0c875-136">Active/passive with cold standby.</span></span> <span data-ttu-id="0c875-137">流量将前往一个区域，而另一个区域将以冷备用模式等待。</span><span class="sxs-lookup"><span data-stu-id="0c875-137">Traffic goes to one region, while the other waits on cold standby.</span></span> <span data-ttu-id="0c875-138">“冷备用模式”意味着次要区域中的 VM 不会被分配，直到故障转移需要它们。</span><span class="sxs-lookup"><span data-stu-id="0c875-138">Cold standby means the VMs in the secondary region are not allocated until needed for failover.</span></span> <span data-ttu-id="0c875-139">此方法的运行成本较低，但是当发生故障时通常需要花费更长时间才能联机。</span><span class="sxs-lookup"><span data-stu-id="0c875-139">This approach costs less to run, but will generally take longer to come online during a failure.</span></span>
* <span data-ttu-id="0c875-140">主动/主动。</span><span class="sxs-lookup"><span data-stu-id="0c875-140">Active/active.</span></span> <span data-ttu-id="0c875-141">两个区域都处于活动状态，并且会在它们之间对请求进行负载均衡。</span><span class="sxs-lookup"><span data-stu-id="0c875-141">Both regions are active, and requests are load balanced between them.</span></span> <span data-ttu-id="0c875-142">如果一个区域变得不可用，则不再使其参与轮换。</span><span class="sxs-lookup"><span data-stu-id="0c875-142">If one region becomes unavailable, it is taken out of rotation.</span></span> 

<span data-ttu-id="0c875-143">此体系结构侧重于“主动/被动（采用热备用模式）”，使用流量管理器进行故障转移。</span><span class="sxs-lookup"><span data-stu-id="0c875-143">This architecture focuses on active/passive with hot standby, using Traffic Manager for failover.</span></span> <span data-ttu-id="0c875-144">注意，可以为热备用模式部署少量 VM，然后根据需要横向扩展。</span><span class="sxs-lookup"><span data-stu-id="0c875-144">Note that you could deploy a small number of VMs for hot standby and then scale out as needed.</span></span>


### <a name="regional-pairing"></a><span data-ttu-id="0c875-145">区域配对</span><span class="sxs-lookup"><span data-stu-id="0c875-145">Regional pairing</span></span>

<span data-ttu-id="0c875-146">每个 Azure 区域都与同一地域内的另一个区域配对。</span><span class="sxs-lookup"><span data-stu-id="0c875-146">Each Azure region is paired with another region within the same geography.</span></span> <span data-ttu-id="0c875-147">通常，请选择同一区域对中的区域（例如“美国东部 2”和“美国中部”）。</span><span class="sxs-lookup"><span data-stu-id="0c875-147">In general, choose regions from the same regional pair (for example, East US 2 and US Central).</span></span> <span data-ttu-id="0c875-148">这样做的好处包括：</span><span class="sxs-lookup"><span data-stu-id="0c875-148">Benefits of doing so include:</span></span>

* <span data-ttu-id="0c875-149">如果发生大范围的故障，会优先恢复每个区域对中的至少一个区域。</span><span class="sxs-lookup"><span data-stu-id="0c875-149">If there is a broad outage, recovery of at least one region out of every pair is prioritized.</span></span>
* <span data-ttu-id="0c875-150">计划内 Azure 系统更新会按顺序提供给配对的区域，以尽可能减少停机时间。</span><span class="sxs-lookup"><span data-stu-id="0c875-150">Planned Azure system updates are rolled out to paired regions sequentially, to minimize possible downtime.</span></span>
* <span data-ttu-id="0c875-151">区域对位于同一地域内，以满足数据驻留要求。</span><span class="sxs-lookup"><span data-stu-id="0c875-151">Pairs reside within the same geography, to meet data residency requirements.</span></span>

<span data-ttu-id="0c875-152">但请确保两个区域都支持应用程序所需的所有 Azure 服务（请参阅[每个区域的服务][services-by-region]）。</span><span class="sxs-lookup"><span data-stu-id="0c875-152">However, make sure that both regions support all of the Azure services needed for your application (see [Services by region][services-by-region]).</span></span> <span data-ttu-id="0c875-153">有关区域对的详细信息，请参阅[业务连续性和灾难恢复 (BCDR)：Azure 配对区域][regional-pairs]。</span><span class="sxs-lookup"><span data-stu-id="0c875-153">For more information about regional pairs, see [Business continuity and disaster recovery (BCDR): Azure Paired Regions][regional-pairs].</span></span>

### <a name="traffic-manager-configuration"></a><span data-ttu-id="0c875-154">流量管理器配置</span><span class="sxs-lookup"><span data-stu-id="0c875-154">Traffic Manager configuration</span></span>

<span data-ttu-id="0c875-155">配置流量管理器时，请考虑以下几点：</span><span class="sxs-lookup"><span data-stu-id="0c875-155">Consider the following points when configuring Traffic Manager:</span></span>

* <span data-ttu-id="0c875-156">**路由**。</span><span class="sxs-lookup"><span data-stu-id="0c875-156">**Routing**.</span></span> <span data-ttu-id="0c875-157">流量管理器支持多个[路由算法][tm-routing]。</span><span class="sxs-lookup"><span data-stu-id="0c875-157">Traffic Manager supports several [routing algorithms][tm-routing].</span></span> <span data-ttu-id="0c875-158">对于本文中所述的情况，请使用“优先级”路由（以前称为“故障转移”路由）。</span><span class="sxs-lookup"><span data-stu-id="0c875-158">For the scenario described in this article, use *priority* routing (formerly called *failover* routing).</span></span> <span data-ttu-id="0c875-159">使用此设置时，流量管理器将所有请求都发送到主要区域，除非主要区域变得无法访问。</span><span class="sxs-lookup"><span data-stu-id="0c875-159">With this setting, Traffic Manager sends all requests to the primary region, unless the primary region becomes unreachable.</span></span> <span data-ttu-id="0c875-160">那时，它将自动故障转移到次要区域。</span><span class="sxs-lookup"><span data-stu-id="0c875-160">At that point, it automatically fails over to the secondary region.</span></span> <span data-ttu-id="0c875-161">请参阅[配置故障转移路由方法][tm-configure-failover]。</span><span class="sxs-lookup"><span data-stu-id="0c875-161">See [Configure Failover routing method][tm-configure-failover].</span></span>
* <span data-ttu-id="0c875-162">**运行状况探测**。</span><span class="sxs-lookup"><span data-stu-id="0c875-162">**Health probe**.</span></span> <span data-ttu-id="0c875-163">流量管理器使用 HTTP（或 HTTPS）[探测][tm-monitoring]来监视每个区域的可用性。</span><span class="sxs-lookup"><span data-stu-id="0c875-163">Traffic Manager uses an HTTP (or HTTPS) [probe][tm-monitoring] to monitor the availability of each region.</span></span> <span data-ttu-id="0c875-164">探测检查指定 URL 路径的 HTTP 200 响应。</span><span class="sxs-lookup"><span data-stu-id="0c875-164">The probe checks for an HTTP 200 response for a specified URL path.</span></span> <span data-ttu-id="0c875-165">作为最佳做法，请创建一个用于报告应用程序整体运行状况的终结点，并使用此终结点进行运行状况探测。</span><span class="sxs-lookup"><span data-stu-id="0c875-165">As a best practice, create an endpoint that reports the overall health of the application, and use this endpoint for the health probe.</span></span> <span data-ttu-id="0c875-166">否则，探测可能会在应用程序的关键部分实际上已出现故障时报告终结点运行状况正常。</span><span class="sxs-lookup"><span data-stu-id="0c875-166">Otherwise, the probe might report a healthy endpoint when critical parts of the application are actually failing.</span></span> <span data-ttu-id="0c875-167">有关详细信息，请参阅[运行状况终结点监视模式][health-endpoint-monitoring-pattern]。</span><span class="sxs-lookup"><span data-stu-id="0c875-167">For more information, see [Health Endpoint Monitoring Pattern][health-endpoint-monitoring-pattern].</span></span>

<span data-ttu-id="0c875-168">当流量管理器进行故障转移时，一段时间内客户端将无法访问应用程序。</span><span class="sxs-lookup"><span data-stu-id="0c875-168">When Traffic Manager fails over there is a period of time when clients cannot reach the application.</span></span> <span data-ttu-id="0c875-169">持续时间受以下因素影响：</span><span class="sxs-lookup"><span data-stu-id="0c875-169">The duration is affected by the following factors:</span></span>

* <span data-ttu-id="0c875-170">运行状况探测必须检测主要区域是否变得无法访问。</span><span class="sxs-lookup"><span data-stu-id="0c875-170">The health probe must detect that the primary region has become unreachable.</span></span>
* <span data-ttu-id="0c875-171">DNS 服务器必须更新 IP 地址的已缓存 DNS 记录，这取决于 DNS 生存时间 (TTL)。</span><span class="sxs-lookup"><span data-stu-id="0c875-171">DNS servers must update the cached DNS records for the IP address, which depends on the DNS time-to-live (TTL).</span></span> <span data-ttu-id="0c875-172">默认 TTL 为 300 秒（5 分钟），但可以在创建流量管理器配置文件时配置此值。</span><span class="sxs-lookup"><span data-stu-id="0c875-172">The default TTL is 300 seconds (5 minutes), but you can configure this value when you create the Traffic Manager profile.</span></span>

<span data-ttu-id="0c875-173">相关详细信息，请参阅[关于流量管理器监视][tm-monitoring]。</span><span class="sxs-lookup"><span data-stu-id="0c875-173">For details, see [About Traffic Manager Monitoring][tm-monitoring].</span></span>

<span data-ttu-id="0c875-174">如果流量管理器进行故障转移，我们建议执行手动故障回复，而不是实施自动故障回复。</span><span class="sxs-lookup"><span data-stu-id="0c875-174">If Traffic Manager fails over, we recommend performing a manual failback rather than implementing an automatic failback.</span></span> <span data-ttu-id="0c875-175">否则，可能会造成应用程序在区域之间来回转移的情况。</span><span class="sxs-lookup"><span data-stu-id="0c875-175">Otherwise, you can create a situation where the application flips back and forth between regions.</span></span> <span data-ttu-id="0c875-176">在进行故障回复之前，请验证是否所有应用程序子系统的运行状况都正常。</span><span class="sxs-lookup"><span data-stu-id="0c875-176">Verify that all application subsystems are healthy before failing back.</span></span>

<span data-ttu-id="0c875-177">请注意，默认情况下，流量管理器会自动进行故障回复。</span><span class="sxs-lookup"><span data-stu-id="0c875-177">Note that Traffic Manager automatically fails back by default.</span></span> <span data-ttu-id="0c875-178">若要禁止此操作，请在发生故障转移事件后手动降低主要区域的优先级。</span><span class="sxs-lookup"><span data-stu-id="0c875-178">To prevent this, manually lower the priority of the primary region after a failover event.</span></span> <span data-ttu-id="0c875-179">例如，假设主要区域的优先级为 1，次要区域的优先级为 2。</span><span class="sxs-lookup"><span data-stu-id="0c875-179">For example, suppose the primary region is priority 1 and the secondary is priority 2.</span></span> <span data-ttu-id="0c875-180">在故障转移后，请将主要区域的优先级设置为 3，以禁止自动故障回复。</span><span class="sxs-lookup"><span data-stu-id="0c875-180">After a failover, set the primary region to priority 3, to prevent automatic failback.</span></span> <span data-ttu-id="0c875-181">当准备好切换回来时，请将优先级更新为 1。</span><span class="sxs-lookup"><span data-stu-id="0c875-181">When you are ready to switch back, update the priority to 1.</span></span>

<span data-ttu-id="0c875-182">以下 [Azure CLI][install-azure-cli] 命令更新优先级：</span><span class="sxs-lookup"><span data-stu-id="0c875-182">The following [Azure CLI][install-azure-cli] command updates the priority:</span></span>

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --priority 3
```    

<span data-ttu-id="0c875-183">另一种方法是暂时禁用终结点，直到你准备好进行故障回复：</span><span class="sxs-lookup"><span data-stu-id="0c875-183">Another approach is to temporarily disable the endpoint until you are ready to fail back:</span></span>

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --status Disabled
```    

<span data-ttu-id="0c875-184">可能需要重新部署某个区域中的资源，具体取决于故障转移原因。</span><span class="sxs-lookup"><span data-stu-id="0c875-184">Depending on the cause of a failover, you might need to redeploy the resources within a region.</span></span> <span data-ttu-id="0c875-185">在进行故障回复之前，请执行操作准备情况测试。</span><span class="sxs-lookup"><span data-stu-id="0c875-185">Before failing back, perform an operational readiness test.</span></span> <span data-ttu-id="0c875-186">测试应当验证的事项如下所示：</span><span class="sxs-lookup"><span data-stu-id="0c875-186">The test should verify things like:</span></span>

* <span data-ttu-id="0c875-187">VM 是否已正确配置。</span><span class="sxs-lookup"><span data-stu-id="0c875-187">VMs are configured correctly.</span></span> <span data-ttu-id="0c875-188">（所有必需的软件是否已安装、IIS 是否正在运行，等等。）</span><span class="sxs-lookup"><span data-stu-id="0c875-188">(All required software is installed, IIS is running, and so on.)</span></span>
* <span data-ttu-id="0c875-189">应用程序子系统运行状况是否正常。</span><span class="sxs-lookup"><span data-stu-id="0c875-189">Application subsystems are healthy.</span></span>
* <span data-ttu-id="0c875-190">功能测试。</span><span class="sxs-lookup"><span data-stu-id="0c875-190">Functional testing.</span></span> <span data-ttu-id="0c875-191">（例如，是否可以从 Web 层访问数据库层。）</span><span class="sxs-lookup"><span data-stu-id="0c875-191">(For example, the database tier is reachable from the web tier.)</span></span>

### <a name="cassandra-deployment-across-multiple-regions"></a><span data-ttu-id="0c875-192">跨多个区域的 Cassandra 部署</span><span class="sxs-lookup"><span data-stu-id="0c875-192">Cassandra deployment across multiple regions</span></span>

<span data-ttu-id="0c875-193">Cassandra 数据中心是一组相关的数据节点，这些节点一起配置在群集中，以实现复制和工作负荷分离。</span><span class="sxs-lookup"><span data-stu-id="0c875-193">Cassandra data centers are a group of related data nodes that are configured together within a cluster for replication and workload segregation.</span></span>

<span data-ttu-id="0c875-194">建议将 [DataStax Enterprise][datastax] 用于生产用途。</span><span class="sxs-lookup"><span data-stu-id="0c875-194">We recommend [DataStax Enterprise][datastax] for production use.</span></span> <span data-ttu-id="0c875-195">有关在 Azure 中运行 DataStax 的详细信息，请参阅[适用于 Azure 的 DataStax Enterprise 部署指南][cassandra-in-azure]。</span><span class="sxs-lookup"><span data-stu-id="0c875-195">For more information on running DataStax in Azure, see [DataStax Enterprise Deployment Guide for Azure][cassandra-in-azure].</span></span> <span data-ttu-id="0c875-196">以下常规建议也适用于任何 Cassandra 版本：</span><span class="sxs-lookup"><span data-stu-id="0c875-196">The following general recommendations apply to any Cassandra edition:</span></span> 

* <span data-ttu-id="0c875-197">将公共 IP 地址分配给每个节点。</span><span class="sxs-lookup"><span data-stu-id="0c875-197">Assign a public IP address to each node.</span></span> <span data-ttu-id="0c875-198">这使群集可以使用 Azure 主干基础结构跨区域进行通信，以较低的成本提供高吞吐量。</span><span class="sxs-lookup"><span data-stu-id="0c875-198">This enables the clusters to communicate across regions using the Azure backbone infrastructure, providing high throughput at low cost.</span></span>
* <span data-ttu-id="0c875-199">使用相应的防火墙和网络安全组 (NSG) 配置保护节点，使流量只能在已知主机（包括客户端和其他群集节点）之间传输。</span><span class="sxs-lookup"><span data-stu-id="0c875-199">Secure nodes using the appropriate firewall and network security group (NSG) configurations, allowing traffic only to and from known hosts, including clients and other cluster nodes.</span></span> <span data-ttu-id="0c875-200">请注意，Cassandra 将不同的端口用于通信、OpsCenter、Spark 等等。</span><span class="sxs-lookup"><span data-stu-id="0c875-200">Note that Cassandra uses different ports for communication, OpsCenter, Spark, and so forth.</span></span> <span data-ttu-id="0c875-201">有关 Cassandra 中的端口用法，请参阅[配置防火墙端口访问权限][cassandra-ports]。</span><span class="sxs-lookup"><span data-stu-id="0c875-201">For port usage in Cassandra, see [Configuring firewall port access][cassandra-ports].</span></span>
* <span data-ttu-id="0c875-202">将 SSL 加密用于所有[客户端到节点][ssl-client-node]和[节点到节点][ssl-node-node]通信。</span><span class="sxs-lookup"><span data-stu-id="0c875-202">Use SSL encryption for all [client-to-node][ssl-client-node] and [node-to-node][ssl-node-node] communications.</span></span>
* <span data-ttu-id="0c875-203">在区域中，请遵循 [Cassandra 建议](n-tier.md#cassandra)中的指导。</span><span class="sxs-lookup"><span data-stu-id="0c875-203">Within a region, follow the guidelines in [Cassandra recommendations](n-tier.md#cassandra).</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="0c875-204">可用性注意事项</span><span class="sxs-lookup"><span data-stu-id="0c875-204">Availability considerations</span></span>

<span data-ttu-id="0c875-205">使用复杂的 N 层应用时，可能不需要在次要区域中复制整个应用程序。</span><span class="sxs-lookup"><span data-stu-id="0c875-205">With a complex N-tier app, you may not need to replicate the entire application in the secondary region.</span></span> <span data-ttu-id="0c875-206">相反，可能只需复制支持业务连续性所需的关键子系统。</span><span class="sxs-lookup"><span data-stu-id="0c875-206">Instead, you might just replicate a critical subsystem that is needed to support business continuity.</span></span>

<span data-ttu-id="0c875-207">流量管理器是系统中的一个潜在故障点。</span><span class="sxs-lookup"><span data-stu-id="0c875-207">Traffic Manager is a possible failure point in the system.</span></span> <span data-ttu-id="0c875-208">如果流量管理器服务出现故障，则客户端在停机期间无法访问应用程序。</span><span class="sxs-lookup"><span data-stu-id="0c875-208">If the Traffic Manager service fails, clients cannot access your application during the downtime.</span></span> <span data-ttu-id="0c875-209">查看[流量管理器 SLA][tm-sla]，然后确定仅使用流量管理器是否能满足业务对高可用性的需求。</span><span class="sxs-lookup"><span data-stu-id="0c875-209">Review the [Traffic Manager SLA][tm-sla], and determine whether using Traffic Manager alone meets your business requirements for high availability.</span></span> <span data-ttu-id="0c875-210">如果不能，请考虑添加另一个流量管理解决方案作为故障回复机制。</span><span class="sxs-lookup"><span data-stu-id="0c875-210">If not, consider adding another traffic management solution as a failback.</span></span> <span data-ttu-id="0c875-211">如果 Azure 流量管理器服务出现故障，请将 DNS 中的 CNAME 记录更改为指向其他流量管理服务。</span><span class="sxs-lookup"><span data-stu-id="0c875-211">If the Azure Traffic Manager service fails, change your CNAME records in DNS to point to the other traffic management service.</span></span> <span data-ttu-id="0c875-212">（此步骤必须手动执行，并且在 DNS 更改被传播之前，应用程序将不可用。）</span><span class="sxs-lookup"><span data-stu-id="0c875-212">(This step must be performed manually, and your application will be unavailable until the DNS changes are propagated.)</span></span>

<span data-ttu-id="0c875-213">对于 Cassandra 群集，要考虑的故障转移方案取决于应用程序使用的一致性级别，以及使用的副本数目。</span><span class="sxs-lookup"><span data-stu-id="0c875-213">For the Cassandra cluster, the failover scenarios to consider depend on the consistency levels used by the application, as well as the number of replicas used.</span></span> <span data-ttu-id="0c875-214">有关 Cassandra 中的一致性级别和用法，请参阅[配置数据一致性][cassandra-consistency]和 [Cassandra：使用仲裁时要与多少个节点通信？][cassandra-consistency-usage]</span><span class="sxs-lookup"><span data-stu-id="0c875-214">For consistency levels and usage in Cassandra, see [Configuring data consistency][cassandra-consistency] and [Cassandra: How many nodes are talked to with Quorum?][cassandra-consistency-usage]</span></span> <span data-ttu-id="0c875-215">Cassandra 中的数据可用性取决于应用程序使用的一致性级别和复制机制。</span><span class="sxs-lookup"><span data-stu-id="0c875-215">Data availability in Cassandra is determined by the consistency level used by the application and the replication mechanism.</span></span> <span data-ttu-id="0c875-216">有关 Cassandra 中的复制，请参阅 [NoSQL 数据库的数据复制说明][cassandra-replication]。</span><span class="sxs-lookup"><span data-stu-id="0c875-216">For replication in Cassandra, see [Data Replication in NoSQL Databases Explained][cassandra-replication].</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="0c875-217">可管理性注意事项</span><span class="sxs-lookup"><span data-stu-id="0c875-217">Manageability considerations</span></span>

<span data-ttu-id="0c875-218">更新部署时，请一次更新一个区域，以减少由于错误配置或应用程序中的错误而导致全局故障的可能性。</span><span class="sxs-lookup"><span data-stu-id="0c875-218">When you update your deployment, update one region at a time to reduce the chance of a global failure from an incorrect configuration or an error in the application.</span></span>

<span data-ttu-id="0c875-219">测试系统在发生故障时的复原能力。</span><span class="sxs-lookup"><span data-stu-id="0c875-219">Test the resiliency of the system to failures.</span></span> <span data-ttu-id="0c875-220">下面是要测试的一些常见故障方案：</span><span class="sxs-lookup"><span data-stu-id="0c875-220">Here are some common failure scenarios to test:</span></span>

* <span data-ttu-id="0c875-221">关闭 VM 实例。</span><span class="sxs-lookup"><span data-stu-id="0c875-221">Shut down VM instances.</span></span>
* <span data-ttu-id="0c875-222">对 CPU 和内存等资源进行压力测试。</span><span class="sxs-lookup"><span data-stu-id="0c875-222">Pressure resources such as CPU and memory.</span></span>
* <span data-ttu-id="0c875-223">断开网络连接/使网络传输出现延迟。</span><span class="sxs-lookup"><span data-stu-id="0c875-223">Disconnect/delay network.</span></span>
* <span data-ttu-id="0c875-224">使进程崩溃。</span><span class="sxs-lookup"><span data-stu-id="0c875-224">Crash processes.</span></span>
* <span data-ttu-id="0c875-225">使证书过期。</span><span class="sxs-lookup"><span data-stu-id="0c875-225">Expire certificates.</span></span>
* <span data-ttu-id="0c875-226">模拟硬件错误。</span><span class="sxs-lookup"><span data-stu-id="0c875-226">Simulate hardware faults.</span></span>
* <span data-ttu-id="0c875-227">关闭域控制器上的 DNS 服务。</span><span class="sxs-lookup"><span data-stu-id="0c875-227">Shut down the DNS service on the domain controllers.</span></span>

<span data-ttu-id="0c875-228">测量恢复时间，并验证它们是否满足你的业务要求。</span><span class="sxs-lookup"><span data-stu-id="0c875-228">Measure the recovery times and verify they meet your business requirements.</span></span> <span data-ttu-id="0c875-229">另外还测试故障模式的组合。</span><span class="sxs-lookup"><span data-stu-id="0c875-229">Test combinations of failure modes, as well.</span></span>


<!-- Links -->
[hybrid-vpn]: ../hybrid-networking/vpn.md

[cassandra-in-azure]: https://academy.datastax.com/resources/deployment-guide-azure
[cassandra-consistency]: http://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html
[cassandra-replication]: http://www.planetcassandra.org/data-replication-in-nosql-databases-explained/
[cassandra-consistency-usage]: https://medium.com/@foundev/cassandra-how-many-nodes-are-talked-to-with-quorum-also-should-i-use-it-98074e75d7d5#.b4pb4alb2
[cassandra-ports]: https://docs.datastax.com/en/datastax_enterprise/5.0/datastax_enterprise/sec/configFirewallPorts.html
[datastax]: https://www.datastax.com/products/datastax-enterprise
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[install-azure-cli]: /azure/xplat-cli-install
[regional-pairs]: /azure/best-practices-availability-paired-regions
[resource groups]: /azure/azure-resource-manager/resource-group-overview
[resource-group-links]: /azure/resource-group-link-resources
[services-by-region]: https://azure.microsoft.com/regions/#services
[ssl-client-node]: http://docs.datastax.com/en/cassandra/2.0/cassandra/security/secureSSLClientToNode_t.html
[ssl-node-node]: http://docs.datastax.com/en/cassandra/2.0/cassandra/security/secureSSLNodeToNode_t.html
[tablediff]: https://msdn.microsoft.com/library/ms162843.aspx
[tm-configure-failover]: /azure/traffic-manager/traffic-manager-configure-failover-routing-method
[tm-monitoring]: /azure/traffic-manager/traffic-manager-monitoring
[tm-routing]: /azure/traffic-manager/traffic-manager-routing-methods
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager/v1_0/
[traffic-manager]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[wsfc]: https://msdn.microsoft.com/library/hh270278.aspx
[0]: ./images/multi-region-application-diagram.png "Azure N 层应用程序的高可用性网络体系结构"
