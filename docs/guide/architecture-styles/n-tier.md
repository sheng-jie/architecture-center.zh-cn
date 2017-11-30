---
title: "N 层体系结构样式"
description: "介绍 Azure 上 N 层体系结构的优点、挑战和最佳做法"
author: MikeWasson
ms.openlocfilehash: 8333b789e03a9da2b021abe7d7c193cd2af8d6bf
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="n-tier-architecture-style"></a><span data-ttu-id="5dea0-103">N 层体系结构样式</span><span class="sxs-lookup"><span data-stu-id="5dea0-103">N-tier architecture style</span></span>

<span data-ttu-id="5dea0-104">N 层体系结构将应用程序分成逻辑层和物理层。</span><span class="sxs-lookup"><span data-stu-id="5dea0-104">An N-tier architecture divides an application into **logical layers** and **physical tiers**.</span></span> 

![](./images/n-tier-logical.svg)

<span data-ttu-id="5dea0-105">层是分离职责和管理依赖关系的方式。</span><span class="sxs-lookup"><span data-stu-id="5dea0-105">Layers are a way to separate responsibilities and manage dependencies.</span></span> <span data-ttu-id="5dea0-106">每个层都有特定的责任。</span><span class="sxs-lookup"><span data-stu-id="5dea0-106">Each layer has a specific responsibility.</span></span> <span data-ttu-id="5dea0-107">较高层可使用较低层中的服务，反之则不行。</span><span class="sxs-lookup"><span data-stu-id="5dea0-107">A higher layer can use services in a lower layer, but not the other way around.</span></span> 

<span data-ttu-id="5dea0-108">层在物理上是分隔开的，在不同的计算机上运行。</span><span class="sxs-lookup"><span data-stu-id="5dea0-108">Tiers are physically separated, running on separate machines.</span></span> <span data-ttu-id="5dea0-109">一个层可直接调用另一个层，或使用异步消息传递（消息队列）。</span><span class="sxs-lookup"><span data-stu-id="5dea0-109">A tier can call to another tier directly, or use asynchronous messaging (message queue).</span></span> <span data-ttu-id="5dea0-110">虽然每个层可能托管在自己的层中，但这并不是必需的。</span><span class="sxs-lookup"><span data-stu-id="5dea0-110">Although each layer might be hosted in its own tier, that's not required.</span></span> <span data-ttu-id="5dea0-111">多个层可能托管在同一层上。</span><span class="sxs-lookup"><span data-stu-id="5dea0-111">Several layers might be hosted on the same tier.</span></span> <span data-ttu-id="5dea0-112">在物理上分隔层可以提高可伸缩性和复原能力，但因额外的网络通信也增加了延迟。</span><span class="sxs-lookup"><span data-stu-id="5dea0-112">Physically separating the tiers improves scalability and resiliency, but also adds latency from the additional network communication.</span></span> 

<span data-ttu-id="5dea0-113">传统的三层应用程序有表示层、中间层和数据库层。</span><span class="sxs-lookup"><span data-stu-id="5dea0-113">A traditional three-tier application has a presentation tier, a middle tier, and a database tier.</span></span> <span data-ttu-id="5dea0-114">中间层是可选的。</span><span class="sxs-lookup"><span data-stu-id="5dea0-114">The middle tier is optional.</span></span> <span data-ttu-id="5dea0-115">更复杂的应用程序可以多于三层。</span><span class="sxs-lookup"><span data-stu-id="5dea0-115">More complex applications can have more than three tiers.</span></span> <span data-ttu-id="5dea0-116">上图显示具有两个中间层，且封装了不同功能区域的应用程序。</span><span class="sxs-lookup"><span data-stu-id="5dea0-116">The diagram above shows an application with two middle tiers, encapsulating different areas of functionality.</span></span> 

<span data-ttu-id="5dea0-117">N 层应用程序可以有封闭的层体系结构或开放的层体系结构：</span><span class="sxs-lookup"><span data-stu-id="5dea0-117">An N-tier application can have a **closed layer architecture** or an **open layer architecture**:</span></span>

- <span data-ttu-id="5dea0-118">在封闭的层体系结构中，层只能调用紧邻的下一层。</span><span class="sxs-lookup"><span data-stu-id="5dea0-118">In a closed layer architecture, a layer can only call the next layer immediately down.</span></span> 
- <span data-ttu-id="5dea0-119">在开放的层体系结构中，层可以调用它下面的任何层。</span><span class="sxs-lookup"><span data-stu-id="5dea0-119">In an open layer architecture, a layer can call any of the layers below it.</span></span> 

<span data-ttu-id="5dea0-120">封闭的层体系结构限制层之间的依赖关系。</span><span class="sxs-lookup"><span data-stu-id="5dea0-120">A closed layer architecture limits the dependencies between layers.</span></span> <span data-ttu-id="5dea0-121">但是，如果一个层仅将请求传递到下一层，可能会产生不必要的流量。</span><span class="sxs-lookup"><span data-stu-id="5dea0-121">However, it might create unnecessary network traffic, if one layer simply passes requests along to the next layer.</span></span> 

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="5dea0-122">此体系结构适用的情况</span><span class="sxs-lookup"><span data-stu-id="5dea0-122">When to use this architecture</span></span>

<span data-ttu-id="5dea0-123">N 层体系结构通常作为服务架构 (IaaS) 应用程序实现，每个层都在独立的 VM 集中运行。</span><span class="sxs-lookup"><span data-stu-id="5dea0-123">N-tier architectures are typically implemented as infrastructure-as-service (IaaS) applications, with each tier running on a separate set of VMs.</span></span> <span data-ttu-id="5dea0-124">然而，N 层应用程序不需要只是 IaaS。</span><span class="sxs-lookup"><span data-stu-id="5dea0-124">However, an N-tier application doesn't need to be pure IaaS.</span></span> <span data-ttu-id="5dea0-125">通常，对体系结构的某些部分使用托管服务是有利的，特别是缓存、消息传递和数据存储。</span><span class="sxs-lookup"><span data-stu-id="5dea0-125">Often, it's advantageous to use managed services for some parts of the architecture, particularly caching, messaging, and data storage.</span></span>

<span data-ttu-id="5dea0-126">请考虑将 N 层体系结构用于：</span><span class="sxs-lookup"><span data-stu-id="5dea0-126">Consider an N-tier architecture for:</span></span>

- <span data-ttu-id="5dea0-127">简单的 Web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="5dea0-127">Simple web applications.</span></span> 
- <span data-ttu-id="5dea0-128">将本地应用程序迁移到 Azure 并进行最小的重构。</span><span class="sxs-lookup"><span data-stu-id="5dea0-128">Migrating an on-premises application to Azure with minimal refactoring.</span></span>
- <span data-ttu-id="5dea0-129">统一开发本地和云应用程序。</span><span class="sxs-lookup"><span data-stu-id="5dea0-129">Unified development of on-premises and cloud applications.</span></span>

<span data-ttu-id="5dea0-130">N 层体系结构在传统的本地应用程序中很常见，因此将现有工作负载迁移到 Azure 是很适合的。</span><span class="sxs-lookup"><span data-stu-id="5dea0-130">N-tier architectures are very common in traditional on-premises applications, so it's a natural fit for migrating existing workloads to Azure.</span></span>

## <a name="benefits"></a><span data-ttu-id="5dea0-131">优点</span><span class="sxs-lookup"><span data-stu-id="5dea0-131">Benefits</span></span>

- <span data-ttu-id="5dea0-132">云与本地之间，云平台之间具有可移植性。</span><span class="sxs-lookup"><span data-stu-id="5dea0-132">Portability between cloud and on-premises, and between cloud platforms.</span></span>
- <span data-ttu-id="5dea0-133">对于大多数开发者来说，学习曲线较少。</span><span class="sxs-lookup"><span data-stu-id="5dea0-133">Less learning curve for most developers.</span></span>
- <span data-ttu-id="5dea0-134">从传统应用程序模型自然演变。</span><span class="sxs-lookup"><span data-stu-id="5dea0-134">Natural evolution from the traditional application model.</span></span>
- <span data-ttu-id="5dea0-135">对异构环境 (Windows/Linux) 开放</span><span class="sxs-lookup"><span data-stu-id="5dea0-135">Open to heterogeneous environment (Windows/Linux)</span></span>

## <a name="challenges"></a><span data-ttu-id="5dea0-136">挑战</span><span class="sxs-lookup"><span data-stu-id="5dea0-136">Challenges</span></span>

- <span data-ttu-id="5dea0-137">很容易到最后，中间层仅对数据库执行 CRUD 操作，增加额外延迟，但又不完成任何有用的工作。</span><span class="sxs-lookup"><span data-stu-id="5dea0-137">It's easy to end up with a middle tier that just does CRUD operations on the database, adding extra latency without doing any useful work.</span></span> 
- <span data-ttu-id="5dea0-138">单一式设计阻止了独立部署各项功能。</span><span class="sxs-lookup"><span data-stu-id="5dea0-138">Monolithic design prevents independent deployment of features.</span></span>
- <span data-ttu-id="5dea0-139">管理 IaaS 应用程序的工作量要大于管理只使用托管服务的应用程序。</span><span class="sxs-lookup"><span data-stu-id="5dea0-139">Managing an IaaS application is more work than an application that uses only managed services.</span></span> 
- <span data-ttu-id="5dea0-140">管理大型系统中的网络安全比较困难。</span><span class="sxs-lookup"><span data-stu-id="5dea0-140">It can be difficult to manage network security in a large system.</span></span>

## <a name="best-practices"></a><span data-ttu-id="5dea0-141">最佳实践</span><span class="sxs-lookup"><span data-stu-id="5dea0-141">Best practices</span></span>

- <span data-ttu-id="5dea0-142">使用自动缩放处理负载中的更改。</span><span class="sxs-lookup"><span data-stu-id="5dea0-142">Use autoscaling to handle changes in load.</span></span> <span data-ttu-id="5dea0-143">请参阅[自动缩放的最佳做法][autoscaling]。</span><span class="sxs-lookup"><span data-stu-id="5dea0-143">See [Autoscaling best practices][autoscaling].</span></span>
- <span data-ttu-id="5dea0-144">使用异步消息传递来分离层。</span><span class="sxs-lookup"><span data-stu-id="5dea0-144">Use asynchronous messaging to decouple tiers.</span></span>
- <span data-ttu-id="5dea0-145">缓存半静态数据。</span><span class="sxs-lookup"><span data-stu-id="5dea0-145">Cache semi-static data.</span></span> <span data-ttu-id="5dea0-146">请参阅[缓存的最佳做法][caching]。</span><span class="sxs-lookup"><span data-stu-id="5dea0-146">See [Caching best practices][caching].</span></span>
- <span data-ttu-id="5dea0-147">请使用 [SQL Server Always On 可用性组][sql-always-on]等解决方案配置高可用性的数据层。</span><span class="sxs-lookup"><span data-stu-id="5dea0-147">Configure database tier for high availability, using a solution such as [SQL Server Always On Availability Groups][sql-always-on].</span></span>
- <span data-ttu-id="5dea0-148">在前端和 Internet 之间放置 Web 应用程序防火墙 (WAF)。</span><span class="sxs-lookup"><span data-stu-id="5dea0-148">Place a web application firewall (WAF) between the front end and the Internet.</span></span>
- <span data-ttu-id="5dea0-149">将每个层放置在自己的子网中，并将子网用作安全边界。</span><span class="sxs-lookup"><span data-stu-id="5dea0-149">Place each tier in its own subnet, and use subnets as a security boundary.</span></span> 
- <span data-ttu-id="5dea0-150">通过仅允许来自中间层的请求，限制对数据层的访问。</span><span class="sxs-lookup"><span data-stu-id="5dea0-150">Restrict access to the data tier, by allowing requests only from the middle tier(s).</span></span>

## <a name="n-tier-architecture-on-virtual-machines"></a><span data-ttu-id="5dea0-151">虚拟机上的 N 层体系结构</span><span class="sxs-lookup"><span data-stu-id="5dea0-151">N-tier architecture on virtual machines</span></span>

<span data-ttu-id="5dea0-152">本部分介绍在 VM 上运行的建议的 N 层体系结构。</span><span class="sxs-lookup"><span data-stu-id="5dea0-152">This section describes a recommended N-tier architecture running on VMs.</span></span> 

![](./images/n-tier-physical.png)

<span data-ttu-id="5dea0-153">每个层包含两个或多个 VM，它们放置在可用性集或 VM 规模集中。</span><span class="sxs-lookup"><span data-stu-id="5dea0-153">Each tier consists of two or more VMs, placed in an availability set or VM scale set.</span></span> <span data-ttu-id="5dea0-154">如果一个 VM 失败，多个 VM 可以提供复原能力。</span><span class="sxs-lookup"><span data-stu-id="5dea0-154">Multiple VMs provide resiliency in case one VM fails.</span></span> <span data-ttu-id="5dea0-155">负载均衡器用于将请求分布到一个层中的 VM 上。</span><span class="sxs-lookup"><span data-stu-id="5dea0-155">Load balancers are used to distribute requests across the VMs in a tier.</span></span> <span data-ttu-id="5dea0-156">通过向池添加更多 VM 可以水平缩放层。</span><span class="sxs-lookup"><span data-stu-id="5dea0-156">A tier can be scaled horizontally by adding more VMs to the pool.</span></span> 

<span data-ttu-id="5dea0-157">每个层也放置在自己的子网中，这意味着它们的内部 IP 地址在同一个地址范围内。</span><span class="sxs-lookup"><span data-stu-id="5dea0-157">Each tier is also placed inside its own subnet, meaning their internal IP addresses fall within the same address range.</span></span> <span data-ttu-id="5dea0-158">这样可以容易地应用网络安全组 (NSG) 规则，并将表路由到各个层。</span><span class="sxs-lookup"><span data-stu-id="5dea0-158">That makes it easy to apply network security group (NSG) rules and route tables to individual tiers.</span></span>

<span data-ttu-id="5dea0-159">Web 和业务层是无状态的。</span><span class="sxs-lookup"><span data-stu-id="5dea0-159">The web and business tiers are stateless.</span></span> <span data-ttu-id="5dea0-160">任何 VM 都可以处理该层的任何请求。</span><span class="sxs-lookup"><span data-stu-id="5dea0-160">Any VM can handle any request for that tier.</span></span> <span data-ttu-id="5dea0-161">数据层应该包含复制的数据库。</span><span class="sxs-lookup"><span data-stu-id="5dea0-161">The data tier should consist of a replicated database.</span></span> <span data-ttu-id="5dea0-162">对于 Windows，我们推荐 SQL Server，使用 Always On 可用性组实现高可用性。</span><span class="sxs-lookup"><span data-stu-id="5dea0-162">For Windows, we recommend SQL Server, using Always On Availability Groups for high availability.</span></span> <span data-ttu-id="5dea0-163">对于 Linux，请选择支持复制的数据库，例如 Apache Cassandra。</span><span class="sxs-lookup"><span data-stu-id="5dea0-163">For Linux, choose a database that supports replication, such as Apache Cassandra.</span></span> 

<span data-ttu-id="5dea0-164">网络安全组 (NSG) 限制对每个层的访问。</span><span class="sxs-lookup"><span data-stu-id="5dea0-164">Network Security Groups (NSGs) restrict access to each tier.</span></span> <span data-ttu-id="5dea0-165">例如，数据库层仅允许来自业务层的访问。</span><span class="sxs-lookup"><span data-stu-id="5dea0-165">For example, the database tier only allows access from the business tier.</span></span>

<span data-ttu-id="5dea0-166">有关更多详细信息和可部署的资源管理器模板，请参阅以下参考体系结构：</span><span class="sxs-lookup"><span data-stu-id="5dea0-166">For more details and a deployable Resource Manager template, see the following reference architectures:</span></span>

- <span data-ttu-id="5dea0-167">[运行用于 N 层应用程序的 Windows VM][n-tier-windows]</span><span class="sxs-lookup"><span data-stu-id="5dea0-167">[Run Windows VMs for an N-tier application][n-tier-windows]</span></span>
- <span data-ttu-id="5dea0-168">[运行用于 N 层应用程序的 Linux VM][n-tier-linux]</span><span class="sxs-lookup"><span data-stu-id="5dea0-168">[Run Linux VMs for an N-tier application][n-tier-linux]</span></span>

### <a name="additional-considerations"></a><span data-ttu-id="5dea0-169">其他注意事项</span><span class="sxs-lookup"><span data-stu-id="5dea0-169">Additional considerations</span></span>

- <span data-ttu-id="5dea0-170">N 层体系结构不限于三层。</span><span class="sxs-lookup"><span data-stu-id="5dea0-170">N-tier architectures are not restricted to three tiers.</span></span> <span data-ttu-id="5dea0-171">对于更复杂的应用程序，通常会有更多层。</span><span class="sxs-lookup"><span data-stu-id="5dea0-171">For more complex applications, it is common to have more tiers.</span></span> <span data-ttu-id="5dea0-172">在这种情况下，请考虑使用第 7 层路由将请求路由到特定的层。</span><span class="sxs-lookup"><span data-stu-id="5dea0-172">In that case, consider using layer-7 routing to route requests to a particular tier.</span></span>

- <span data-ttu-id="5dea0-173">层是可伸缩性、可靠性和安全性的边界。</span><span class="sxs-lookup"><span data-stu-id="5dea0-173">Tiers are the boundary of scalability, reliability, and security.</span></span> <span data-ttu-id="5dea0-174">请考虑为这些区域中有不同需求的服务提供单独的层。</span><span class="sxs-lookup"><span data-stu-id="5dea0-174">Consider having separate tiers for services with different requirements in those areas.</span></span>

- <span data-ttu-id="5dea0-175">使用 VM 规模集进行自动缩放。</span><span class="sxs-lookup"><span data-stu-id="5dea0-175">Use VM Scale Sets for autoscaling.</span></span>

- <span data-ttu-id="5dea0-176">在体系结构中寻找可以使用托管服务而无需进行大量重构的位置。</span><span class="sxs-lookup"><span data-stu-id="5dea0-176">Look for places in the architecture where you can use a managed service without significant refactoring.</span></span> <span data-ttu-id="5dea0-177">具体来说，就是缓存、消息传递、存储和数据库。</span><span class="sxs-lookup"><span data-stu-id="5dea0-177">In particular, look at caching, messaging, storage, and databases.</span></span> 

- <span data-ttu-id="5dea0-178">为了提高安全性，请在应用程序前放置网络 DMZ。</span><span class="sxs-lookup"><span data-stu-id="5dea0-178">For higher security, place a network DMZ in front of the application.</span></span> <span data-ttu-id="5dea0-179">DMZ 包括防火墙和数据包检查等实现安全功能的网络虚拟设备 (NVA)。</span><span class="sxs-lookup"><span data-stu-id="5dea0-179">The DMZ includes network virtual appliances (NVAs) that implement security functionality such as firewalls and packet inspection.</span></span> <span data-ttu-id="5dea0-180">有关详细信息，请参阅[网络 DMZ 参考体系结构][dmz]。</span><span class="sxs-lookup"><span data-stu-id="5dea0-180">For more information, see [Network DMZ reference architecture][dmz].</span></span>

- <span data-ttu-id="5dea0-181">为实现高可用性，请在可用性集中放置两个或多个 NVA，并使用外部负载均衡器在实例间分布 Internet 请求。</span><span class="sxs-lookup"><span data-stu-id="5dea0-181">For high availability, place two or more NVAs in an availability set, with an external load balancer to distribute Internet requests across the instances.</span></span> <span data-ttu-id="5dea0-182">有关详细信息，请参阅[部署高可用性网络虚拟设备][ha-nva]。</span><span class="sxs-lookup"><span data-stu-id="5dea0-182">For more information, see [Deploy highly available network virtual appliances][ha-nva].</span></span>

- <span data-ttu-id="5dea0-183">不允许将 RDP 或 SSH 访问定向到正在运行应用程序代码的 VM。</span><span class="sxs-lookup"><span data-stu-id="5dea0-183">Do not allow direct RDP or SSH access to VMs that are running application code.</span></span> <span data-ttu-id="5dea0-184">相反，运算符应登录到 jumpbox，也称为壁垒主机。</span><span class="sxs-lookup"><span data-stu-id="5dea0-184">Instead, operators should log into a jumpbox, also called a bastion host.</span></span> <span data-ttu-id="5dea0-185">这是管理员在网络上用来连接其他 VM 的 VM。</span><span class="sxs-lookup"><span data-stu-id="5dea0-185">This is a  VM on the network that administrators use to connect to the other VMs.</span></span> <span data-ttu-id="5dea0-186">jumpbox 中有一个 NSG，它仅允许来自已批准的公共 IP 地址的 RDP 或 SSH。</span><span class="sxs-lookup"><span data-stu-id="5dea0-186">The jumpbox has an NSG that allows RDP or SSH only from approved public IP addresses.</span></span>

- <span data-ttu-id="5dea0-187">可使用站点到站点虚拟专用网络 (VPN) 或 Azure ExpressRoute，将 Azure 虚拟网络扩展到本地网络。</span><span class="sxs-lookup"><span data-stu-id="5dea0-187">You can extend the Azure virtual network to your on-premises network using a site-to-site virtual private network (VPN) or Azure ExpressRoute.</span></span> <span data-ttu-id="5dea0-188">有关详细信息，请参阅[混合网络参考体系结构][hybrid-network]。</span><span class="sxs-lookup"><span data-stu-id="5dea0-188">For more information, see [Hybrid network reference architecture][hybrid-network].</span></span>

- <span data-ttu-id="5dea0-189">如果组织使用 Active Directory 管理标识，建议将 Active Directory 环境扩展到 Azure VNet。</span><span class="sxs-lookup"><span data-stu-id="5dea0-189">If your organization uses Active Directory to manage identity, you may want to extend your Active Directory environment to the Azure VNet.</span></span> <span data-ttu-id="5dea0-190">有关详细信息，请参阅[标识管理参考体系结构][identity]。</span><span class="sxs-lookup"><span data-stu-id="5dea0-190">For more information, see [Identity management reference architecture][identity].</span></span>

- <span data-ttu-id="5dea0-191">如果需要比 VM 提供的 Azure SLA 更高的可用性，可以跨两个区域复制应用程序，并使用 Azure 流量管理器进行故障转移。</span><span class="sxs-lookup"><span data-stu-id="5dea0-191">If you need higher availability than the Azure SLA for VMs provides, replicate the application across two regions and use Azure Traffic Manager for failover.</span></span> <span data-ttu-id="5dea0-192">有关详细信息，请参阅[在多个区域中运行 Windows VM][multiregion-windows] 或[在多个区域中运行 Linux VM][multiregion-linux]。</span><span class="sxs-lookup"><span data-stu-id="5dea0-192">For more information, see [Run Windows VMs in multiple regions][multiregion-windows] or [Run Linux VMs in multiple regions][multiregion-linux].</span></span>

[autoscaling]: ../../best-practices/auto-scaling.md
[caching]: ../../best-practices/caching.md
[dmz]: ../../reference-architectures/dmz/index.md
[ha-nva]: ../../reference-architectures/dmz/nva-ha.md
[hybrid-network]: ../../reference-architectures/hybrid-networking/index.md
[identity]: ../../reference-architectures/identity/index.md
[multiregion-linux]: ../../reference-architectures/virtual-machines-linux/multi-region-application.md
[multiregion-windows]: ../../reference-architectures/virtual-machines-windows/multi-region-application.md
[n-tier-linux]: ../../reference-architectures/virtual-machines-linux/n-tier.md
[n-tier-windows]: ../../reference-architectures/virtual-machines-windows/n-tier.md
[sql-always-on]: /sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server