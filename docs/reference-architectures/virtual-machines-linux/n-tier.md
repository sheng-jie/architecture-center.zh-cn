---
title: "在 Azure 上运行用于 N 层应用程序的 Linux VM"
description: "如何在 Microsoft Azure 中运行用于 N 层体系结构的 Linux VM。"
author: MikeWasson
ms.date: 11/22/2017
pnp.series.title: Linux VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: e875a58aa83339560fd1de5b03a960f071883927
ms.sourcegitcommit: c9e6d8edb069b8c513de748ce8114c879bad5f49
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/08/2018
---
# <a name="run-linux-vms-for-an-n-tier-application"></a><span data-ttu-id="c4709-103">运行用于 N 层应用程序的 Linux VM</span><span class="sxs-lookup"><span data-stu-id="c4709-103">Run Linux VMs for an N-tier application</span></span>

<span data-ttu-id="c4709-104">此参考体系结构显示了一组经过实践检验的、运行用于 N 层应用程序的 Linux 虚拟机 (VM) 的做法。</span><span class="sxs-lookup"><span data-stu-id="c4709-104">This reference architecture shows a set of proven practices for running Linux virtual machines (VMs) for an N-tier application.</span></span> [<span data-ttu-id="c4709-105">**部署此解决方案**。</span><span class="sxs-lookup"><span data-stu-id="c4709-105">**Deploy this solution**.</span></span>](#deploy-the-solution)  

<span data-ttu-id="c4709-106">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="c4709-106">![[0]][0]</span></span>

<span data-ttu-id="c4709-107">*下载此体系结构的 [Visio 文件][visio-download]。*</span><span class="sxs-lookup"><span data-stu-id="c4709-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="c4709-108">体系结构</span><span class="sxs-lookup"><span data-stu-id="c4709-108">Architecture</span></span>

<span data-ttu-id="c4709-109">有许多方法可用来实现 N 层体系结构。</span><span class="sxs-lookup"><span data-stu-id="c4709-109">There are many ways to implement an N-tier architecture.</span></span> <span data-ttu-id="c4709-110">关系图中显示了一个典型的 3 层 Web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="c4709-110">The diagram shows a typical 3-tier web application.</span></span> <span data-ttu-id="c4709-111">此体系结构基于[运行负载均衡的 VM 以提高可伸缩性和可用性][multi-vm]中所述的内容。</span><span class="sxs-lookup"><span data-stu-id="c4709-111">This architecture builds on [Run load-balanced VMs for scalability and availability][multi-vm].</span></span> <span data-ttu-id="c4709-112">Web 层和业务层都使用负载均衡的 VM。</span><span class="sxs-lookup"><span data-stu-id="c4709-112">The web and business tiers use load-balanced VMs.</span></span>

* <span data-ttu-id="c4709-113">**可用性集。**</span><span class="sxs-lookup"><span data-stu-id="c4709-113">**Availability sets.**</span></span> <span data-ttu-id="c4709-114">为每个层创建一个[可用性集][azure-availability-sets]，并且在每个层中至少预配两个 VM。</span><span class="sxs-lookup"><span data-stu-id="c4709-114">Create an [availability set][azure-availability-sets] for each tier, and provision at least two VMs in each tier.</span></span>  <span data-ttu-id="c4709-115">这样，VM 便可以满足 VM 的更高[服务级别协议 (SLA)][vm-sla]。</span><span class="sxs-lookup"><span data-stu-id="c4709-115">This makes the VMs eligible for a higher [service level agreement (SLA)][vm-sla] for VMs.</span></span> <span data-ttu-id="c4709-116">可在可用性集中部署单个 VM，但单个 VM 不具备 SLA 保证的资格，除非单个 VM 针对所有 OS 和数据磁盘使用 Azure 高级存储。</span><span class="sxs-lookup"><span data-stu-id="c4709-116">You can deploy a single VM in an availability set, but the single VM will not qualify for an SLA guarantee unless the single VM is using Azure Premium Storage for all OS and data disks.</span></span>  
* <span data-ttu-id="c4709-117">**子网。**</span><span class="sxs-lookup"><span data-stu-id="c4709-117">**Subnets.**</span></span> <span data-ttu-id="c4709-118">为每个层创建一个单独的子网。</span><span class="sxs-lookup"><span data-stu-id="c4709-118">Create a separate subnet for each tier.</span></span> <span data-ttu-id="c4709-119">使用 [CIDR] 表示法指定地址范围和子网掩码。</span><span class="sxs-lookup"><span data-stu-id="c4709-119">Specify the address range and subnet mask using [CIDR] notation.</span></span> 
* <span data-ttu-id="c4709-120">**负载均衡器。**</span><span class="sxs-lookup"><span data-stu-id="c4709-120">**Load balancers.**</span></span> <span data-ttu-id="c4709-121">使用[面向 Internet 的负载均衡器][load-balancer-external]将传入的 Internet 流量分布到 Web 层，使用[内部负载均衡器][load-balancer-internal]将来自 Web 层的网络流量分布到业务层。</span><span class="sxs-lookup"><span data-stu-id="c4709-121">Use an [Internet-facing load balancer][load-balancer-external] to distribute incoming Internet traffic to the web tier, and an [internal load balancer][load-balancer-internal] to distribute network traffic from the web tier to the business tier.</span></span>
* <span data-ttu-id="c4709-122">**Azure DNS**。</span><span class="sxs-lookup"><span data-stu-id="c4709-122">**Azure DNS**.</span></span> <span data-ttu-id="c4709-123">[Azure DNS][azure-dns] 是 DNS 域的托管服务，它使用 Microsoft Azure 基础结构提供名称解析。</span><span class="sxs-lookup"><span data-stu-id="c4709-123">[Azure DNS][azure-dns] is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure.</span></span> <span data-ttu-id="c4709-124">通过在 Azure 中托管域，可以使用与其他 Azure 服务相同的凭据、API、工具和计费来管理 DNS 记录。</span><span class="sxs-lookup"><span data-stu-id="c4709-124">By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools, and billing as your other Azure services.</span></span>
* <span data-ttu-id="c4709-125">**Jumpbox。**</span><span class="sxs-lookup"><span data-stu-id="c4709-125">**Jumpbox.**</span></span> <span data-ttu-id="c4709-126">也称为[守护主机]。</span><span class="sxs-lookup"><span data-stu-id="c4709-126">Also called a [bastion host].</span></span> <span data-ttu-id="c4709-127">网络上的一个安全 VM，管理员使用它来连接到其他 VM。</span><span class="sxs-lookup"><span data-stu-id="c4709-127">A secure VM on the network that administrators use to connect to the other VMs.</span></span> <span data-ttu-id="c4709-128">Jumpbox 中的某个 NSG 只允许来自安全列表中的公共 IP 地址的远程流量。</span><span class="sxs-lookup"><span data-stu-id="c4709-128">The jumpbox has an NSG that allows remote traffic only from public IP addresses on a safe list.</span></span> <span data-ttu-id="c4709-129">NSG 应允许安全外壳 (SSH) 流量。</span><span class="sxs-lookup"><span data-stu-id="c4709-129">The NSG should permit secure shell (SSH) traffic.</span></span>
* <span data-ttu-id="c4709-130">**监视。**</span><span class="sxs-lookup"><span data-stu-id="c4709-130">**Monitoring.**</span></span> <span data-ttu-id="c4709-131">可以使用 [Nagios]、[Zabbix] 或 [Icinga] 等监视软件深入了解响应时间、VM 运行时间和系统的整体运行状况。</span><span class="sxs-lookup"><span data-stu-id="c4709-131">Monitoring software such as [Nagios], [Zabbix], or [Icinga] can give you insight into response time, VM uptime, and the overall health of your system.</span></span> <span data-ttu-id="c4709-132">在置于单独的管理子网中的 VM 上安装监视软件。</span><span class="sxs-lookup"><span data-stu-id="c4709-132">Install the monitoring software on a VM that's placed in a separate management subnet.</span></span>
* <span data-ttu-id="c4709-133">**NSG。**</span><span class="sxs-lookup"><span data-stu-id="c4709-133">**NSGs.**</span></span> <span data-ttu-id="c4709-134">使用[网络安全组][nsg] (NSG) 来限制 VNet 中的网络流量。</span><span class="sxs-lookup"><span data-stu-id="c4709-134">Use [network security groups][nsg] (NSGs) to restrict network traffic within the VNet.</span></span> <span data-ttu-id="c4709-135">例如，在此处显示的 3 层体系结构中，数据库层不接受来自 Web 前端的流量，仅接受来自业务层和管理子网的流量。</span><span class="sxs-lookup"><span data-stu-id="c4709-135">For example, in the 3-tier architecture shown here, the database tier does not accept traffic from the web front end, only from the business tier and the management subnet.</span></span>
* <span data-ttu-id="c4709-136">**Apache Cassandra 数据库**。</span><span class="sxs-lookup"><span data-stu-id="c4709-136">**Apache Cassandra database**.</span></span> <span data-ttu-id="c4709-137">通过启用复制和故障转移，在数据层提供高可用性。</span><span class="sxs-lookup"><span data-stu-id="c4709-137">Provides high availability at the data tier, by enabling replication and failover.</span></span>

## <a name="recommendations"></a><span data-ttu-id="c4709-138">建议</span><span class="sxs-lookup"><span data-stu-id="c4709-138">Recommendations</span></span>

<span data-ttu-id="c4709-139">你的要求可能不同于此处描述的体系结构。</span><span class="sxs-lookup"><span data-stu-id="c4709-139">Your requirements might differ from the architecture described here.</span></span> <span data-ttu-id="c4709-140">请使用以下建议作为入手点。</span><span class="sxs-lookup"><span data-stu-id="c4709-140">Use these recommendations as a starting point.</span></span> 

### <a name="vnet--subnets"></a><span data-ttu-id="c4709-141">VNet/子网</span><span class="sxs-lookup"><span data-stu-id="c4709-141">VNet / Subnets</span></span>

<span data-ttu-id="c4709-142">在创建 VNet 时，确定每个子网中的资源需要多少 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="c4709-142">When you create the VNet, determine how many IP addresses your resources in each subnet require.</span></span> <span data-ttu-id="c4709-143">使用 [CIDR] 表示法为所需的 IP 地址指定子网掩码和足够大的 VNet 地址范围。</span><span class="sxs-lookup"><span data-stu-id="c4709-143">Specify a subnet mask and a VNet address range large enough for the required IP addresses using [CIDR] notation.</span></span> <span data-ttu-id="c4709-144">使用标准[专用 IP 地址块][private-ip-space]内的一个地址空间，这些地址块为 10.0.0.0/8、172.16.0.0/12 和 192.168.0.0/16。</span><span class="sxs-lookup"><span data-stu-id="c4709-144">Use an address space that falls within the standard [private IP address blocks][private-ip-space], which are 10.0.0.0/8, 172.16.0.0/12, and 192.168.0.0/16.</span></span>

<span data-ttu-id="c4709-145">如果以后需要在 VNet 与本地网络之间设置一个网关，请选择一个不与本地网络重叠的地址范围。</span><span class="sxs-lookup"><span data-stu-id="c4709-145">Choose an address range that does not overlap with your on-premises network, in case you need to set up a gateway between the VNet and your on-premises network later.</span></span> <span data-ttu-id="c4709-146">在创建 VNet 后，将无法更改地址范围。</span><span class="sxs-lookup"><span data-stu-id="c4709-146">Once you create the VNet, you can't change the address range.</span></span>

<span data-ttu-id="c4709-147">在设计子网时一定要牢记功能和安全要求。</span><span class="sxs-lookup"><span data-stu-id="c4709-147">Design subnets with functionality and security requirements in mind.</span></span> <span data-ttu-id="c4709-148">同一层或同一角色中的所有 VM 应当置于同一子网，这可能是一个安全边界。</span><span class="sxs-lookup"><span data-stu-id="c4709-148">All VMs within the same tier or role should go into the same subnet, which can be a security boundary.</span></span> <span data-ttu-id="c4709-149">有关设计 VNet 和子网的详细信息，请参阅[规划和设计 Azure 虚拟网络][plan-network]。</span><span class="sxs-lookup"><span data-stu-id="c4709-149">For more information about designing VNets and subnets, see [Plan and design Azure Virtual Networks][plan-network].</span></span>

<span data-ttu-id="c4709-150">针对每个子网，采用 CIDR 表示法指定子网的地址空间。</span><span class="sxs-lookup"><span data-stu-id="c4709-150">For each subnet, specify the address space for the subnet in CIDR notation.</span></span> <span data-ttu-id="c4709-151">例如，“10.0.0.0/24”将创建包含 256 个 IP 地址的范围。</span><span class="sxs-lookup"><span data-stu-id="c4709-151">For example, '10.0.0.0/24' creates a range of 256 IP addresses.</span></span> <span data-ttu-id="c4709-152">VM 可以使用其中的 251 个；5 个会被保留。</span><span class="sxs-lookup"><span data-stu-id="c4709-152">VMs can use 251 of these; five are reserved.</span></span> <span data-ttu-id="c4709-153">请确保地址范围在子网之间不重叠。</span><span class="sxs-lookup"><span data-stu-id="c4709-153">Make sure the address ranges don't overlap across subnets.</span></span> <span data-ttu-id="c4709-154">请参阅[虚拟网络常见问题解答][vnet faq]。</span><span class="sxs-lookup"><span data-stu-id="c4709-154">See the [Virtual Network FAQ][vnet faq].</span></span>

### <a name="network-security-groups"></a><span data-ttu-id="c4709-155">网络安全组</span><span class="sxs-lookup"><span data-stu-id="c4709-155">Network security groups</span></span>

<span data-ttu-id="c4709-156">使用 NSG 规则限制各个层之间的流量。</span><span class="sxs-lookup"><span data-stu-id="c4709-156">Use NSG rules to restrict traffic between tiers.</span></span> <span data-ttu-id="c4709-157">例如，在上面显示的 3 层体系结构中，Web 层不直接与数据库层进行通信。</span><span class="sxs-lookup"><span data-stu-id="c4709-157">For example, in the 3-tier architecture shown above, the web tier does not communicate directly with the database tier.</span></span> <span data-ttu-id="c4709-158">为强制实现此目的，数据库层应当阻止来自 Web 层子网的传入流量。</span><span class="sxs-lookup"><span data-stu-id="c4709-158">To enforce this, the database tier should block incoming traffic from the web tier subnet.</span></span>  

1. <span data-ttu-id="c4709-159">创建一个 NSG 并将其关联到数据库层子网。</span><span class="sxs-lookup"><span data-stu-id="c4709-159">Create an NSG and associate it to the database tier subnet.</span></span>
2. <span data-ttu-id="c4709-160">添加一个拒绝来自 VNet 的所有入站流量的规则。</span><span class="sxs-lookup"><span data-stu-id="c4709-160">Add a rule that denies all inbound traffic from the VNet.</span></span> <span data-ttu-id="c4709-161">（在规则中使用 `VIRTUAL_NETWORK` 标记。）</span><span class="sxs-lookup"><span data-stu-id="c4709-161">(Use the `VIRTUAL_NETWORK` tag in the rule.)</span></span> 
3. <span data-ttu-id="c4709-162">添加一个具有更高优先级的规则，用以允许来自业务层子网的入站流量。</span><span class="sxs-lookup"><span data-stu-id="c4709-162">Add a rule with a higher priority that allows inbound traffic from the business tier subnet.</span></span> <span data-ttu-id="c4709-163">此规则将替代前面的规则，并允许业务层与数据库层进行通信。</span><span class="sxs-lookup"><span data-stu-id="c4709-163">This rule overrides the previous rule, and allows the business tier to talk to the database tier.</span></span>
4. <span data-ttu-id="c4709-164">添加一个允许来自数据库层子网自身的入站流量的规则。</span><span class="sxs-lookup"><span data-stu-id="c4709-164">Add a rule that allows inbound traffic from within the database tier subnet itself.</span></span> <span data-ttu-id="c4709-165">此规则允许在数据库层中的各个 VM 之间进行通信，这是进行数据库复制和故障转移所必需的。</span><span class="sxs-lookup"><span data-stu-id="c4709-165">This rule allows communication between VMs in the database tier, which is needed for database replication and failover.</span></span>
5. <span data-ttu-id="c4709-166">添加一个允许来自 jumpbox 子网的 SSH 流量的规则。</span><span class="sxs-lookup"><span data-stu-id="c4709-166">Add a rule that allows SSH traffic from the jumpbox subnet.</span></span> <span data-ttu-id="c4709-167">此规则允许管理员从 jumpbox 连接到数据库层。</span><span class="sxs-lookup"><span data-stu-id="c4709-167">This rule lets administrators connect to the database tier from the jumpbox.</span></span>
   
   > [!NOTE]
   > <span data-ttu-id="c4709-168">NSG 包含允许来自 VNet 的任何入站流量的[默认规则][nsg-rules]。</span><span class="sxs-lookup"><span data-stu-id="c4709-168">An NSG has [default rules][nsg-rules] that allow any inbound traffic from within the VNet.</span></span> <span data-ttu-id="c4709-169">这些规则无法删除，但可以通过创建优先级更高的规则来替代它们。</span><span class="sxs-lookup"><span data-stu-id="c4709-169">These rules can't be deleted, but you can override them by creating higher-priority rules.</span></span>
   > 
   > 

### <a name="load-balancers"></a><span data-ttu-id="c4709-170">负载均衡器</span><span class="sxs-lookup"><span data-stu-id="c4709-170">Load balancers</span></span>

<span data-ttu-id="c4709-171">外部负载均衡器将 Internet 流量分布到 Web 层。</span><span class="sxs-lookup"><span data-stu-id="c4709-171">The external load balancer distributes Internet traffic to the web tier.</span></span> <span data-ttu-id="c4709-172">为此负载均衡器创建一个公共 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="c4709-172">Create a public IP address for this load balancer.</span></span> <span data-ttu-id="c4709-173">请参阅[创建面向 Internet 的负载均衡器][lb-external-create]。</span><span class="sxs-lookup"><span data-stu-id="c4709-173">See [Creating an Internet-facing load balancer][lb-external-create].</span></span>

<span data-ttu-id="c4709-174">内部负载均衡器将网络流量从 Web 层分布到业务层。</span><span class="sxs-lookup"><span data-stu-id="c4709-174">The internal load balancer distributes network traffic from the web tier to the business tier.</span></span> <span data-ttu-id="c4709-175">若要为此负载均衡器提供专用 IP 地址，请创建一个前端 IP 配置，并将其与业务层的子网相关联。</span><span class="sxs-lookup"><span data-stu-id="c4709-175">To give this load balancer a private IP address, create a frontend IP configuration and associate it with the subnet for the business tier.</span></span> <span data-ttu-id="c4709-176">请参阅[开始创建内部负载均衡器][lb-internal-create]。</span><span class="sxs-lookup"><span data-stu-id="c4709-176">See [Get started creating an Internal load balancer][lb-internal-create].</span></span>

### <a name="cassandra"></a><span data-ttu-id="c4709-177">Cassandra</span><span class="sxs-lookup"><span data-stu-id="c4709-177">Cassandra</span></span>

<span data-ttu-id="c4709-178">我们建议将 [DataStax Enterprise][datastax] 用于生产用途，但这些建议适用于任何 Cassandra 版本。</span><span class="sxs-lookup"><span data-stu-id="c4709-178">We recommend [DataStax Enterprise][datastax] for production use, but these recommendations apply to any Cassandra edition.</span></span> <span data-ttu-id="c4709-179">有关在 Azure 中运行 DataStax 的详细信息，请参阅[适用于 Azure 的 DataStax Enterprise 部署指南][cassandra-in-azure]。</span><span class="sxs-lookup"><span data-stu-id="c4709-179">For more information on running DataStax in Azure, see [DataStax Enterprise Deployment Guide for Azure][cassandra-in-azure].</span></span> 

<span data-ttu-id="c4709-180">将 Cassandra 群集的 VM 放置在一个可用性集中，以确保 Cassandra 副本分布在多个容错域和升级域中。</span><span class="sxs-lookup"><span data-stu-id="c4709-180">Put the VMs for a Cassandra cluster in an availability set to ensure that the Cassandra replicas are distributed across multiple fault domains and upgrade domains.</span></span> <span data-ttu-id="c4709-181">有关容错域和升级域的详细信息，请参阅[管理虚拟机的可用性][azure-availability-sets]。</span><span class="sxs-lookup"><span data-stu-id="c4709-181">For more information about fault domains and upgrade domains, see [Manage the availability of virtual machines][azure-availability-sets].</span></span> 

<span data-ttu-id="c4709-182">为每个可用性集配置三个容错域（最大数目），并且为每个可用性集配置 18 个升级域。</span><span class="sxs-lookup"><span data-stu-id="c4709-182">Configure three fault domains (the maximum) per availability set and 18 upgrade domains per availability set.</span></span> <span data-ttu-id="c4709-183">这提供了仍可均匀分布在容错域中的升级域的最大数量。</span><span class="sxs-lookup"><span data-stu-id="c4709-183">This provides the maximum number of upgrade domains that can still be distributed evenly across the fault domains.</span></span>   

<span data-ttu-id="c4709-184">以机架感知模式配置节点。</span><span class="sxs-lookup"><span data-stu-id="c4709-184">Configure nodes in rack-aware mode.</span></span> <span data-ttu-id="c4709-185">将故障域映射到 `cassandra-rackdc.properties` 文件中的机架。</span><span class="sxs-lookup"><span data-stu-id="c4709-185">Map fault domains to racks in the `cassandra-rackdc.properties` file.</span></span>

<span data-ttu-id="c4709-186">群集前面不需要负载均衡器。</span><span class="sxs-lookup"><span data-stu-id="c4709-186">You don't need a load balancer in front of the cluster.</span></span> <span data-ttu-id="c4709-187">客户端会直接连接到群集中的节点。</span><span class="sxs-lookup"><span data-stu-id="c4709-187">The client connects directly to a node in the cluster.</span></span>

### <a name="jumpbox"></a><span data-ttu-id="c4709-188">Jumpbox</span><span class="sxs-lookup"><span data-stu-id="c4709-188">Jumpbox</span></span>

<span data-ttu-id="c4709-189">Jumpbox 的性能要求非常低，因此请为 jumpbox 选择一个较小的 VM 大小，例如标准 A1。</span><span class="sxs-lookup"><span data-stu-id="c4709-189">The jumpbox will have minimal performance requirements, so select a small VM size for the jumpbox such as Standard A1.</span></span> 

<span data-ttu-id="c4709-190">为 jumpbox 创建一个[公共 IP 地址]。</span><span class="sxs-lookup"><span data-stu-id="c4709-190">Create a [public IP address] for the jumpbox.</span></span> <span data-ttu-id="c4709-191">将 jumpbox 放置在与其他 VM 相同的 VNet 中，但将其置于一个单独的管理子网中。</span><span class="sxs-lookup"><span data-stu-id="c4709-191">Place the jumpbox in the same VNet as the other VMs, but in a separate management subnet.</span></span>

<span data-ttu-id="c4709-192">不要允许从公共 Internet 对运行应用程序工作负荷的 VM 进行 SSH 访问。</span><span class="sxs-lookup"><span data-stu-id="c4709-192">Do not allow SSH access from the public Internet to the VMs that run the application workload.</span></span> <span data-ttu-id="c4709-193">相反，对这些 VM 的所有 SSH 访问都必须通过 jumpbox 进行。</span><span class="sxs-lookup"><span data-stu-id="c4709-193">Instead, all SSH access to these VMs must come through the jumpbox.</span></span> <span data-ttu-id="c4709-194">管理员登录到 jumpbox，然后从 jumpbox 登录到其他 VM。</span><span class="sxs-lookup"><span data-stu-id="c4709-194">An administrator logs into the jumpbox, and then logs into the other VM from the jumpbox.</span></span> <span data-ttu-id="c4709-195">Jumpbox 允许来自 Internet 的 SSH 流量，但仅允许来自已知的安全 IP 地址的流量。</span><span class="sxs-lookup"><span data-stu-id="c4709-195">The jumpbox allows SSH traffic from the Internet, but only from known, safe IP addresses.</span></span>

<span data-ttu-id="c4709-196">若要保护 jumpbox，请创建一个 NSG 并将其应用于 jumpbox 子网。</span><span class="sxs-lookup"><span data-stu-id="c4709-196">To secure the jumpbox, create an NSG and apply it to the jumpbox subnet.</span></span> <span data-ttu-id="c4709-197">添加一个 NSG 规则，使其仅允许来自一组安全的公共 IP 地址的 SSH 连接。</span><span class="sxs-lookup"><span data-stu-id="c4709-197">Add an NSG rule that allows SSH connections only from a safe set of public IP addresses.</span></span> <span data-ttu-id="c4709-198">可以将 NSG 附加到子网或 jumpbox NIC。</span><span class="sxs-lookup"><span data-stu-id="c4709-198">The NSG can be attached either to the subnet or to the jumpbox NIC.</span></span> <span data-ttu-id="c4709-199">在本例中，我们建议将其附加到 NIC，以便仅允许将 SSH 流量发送到 jumpbox，即使将其他 VM 添加到同一子网也是如此。</span><span class="sxs-lookup"><span data-stu-id="c4709-199">In this case, we recommend attaching it to the NIC, so SSH traffic is permitted only to the jumpbox, even if you add other VMs to the same subnet.</span></span>

<span data-ttu-id="c4709-200">为其他子网配置 NSG 以允许来自管理子网的 SSH 流量。</span><span class="sxs-lookup"><span data-stu-id="c4709-200">Configure the NSGs for the other subnets to allow SSH traffic from the management subnet.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="c4709-201">可用性注意事项</span><span class="sxs-lookup"><span data-stu-id="c4709-201">Availability considerations</span></span>

<span data-ttu-id="c4709-202">将每个层或 VM 角色放入单独的可用性集中。</span><span class="sxs-lookup"><span data-stu-id="c4709-202">Put each tier or VM role into a separate availability set.</span></span> 

<span data-ttu-id="c4709-203">在数据库层，具有多个 VM 不会自动转换为高度可用的数据库。</span><span class="sxs-lookup"><span data-stu-id="c4709-203">At the database tier, having multiple VMs does not automatically translate into a highly available database.</span></span> <span data-ttu-id="c4709-204">对于关系数据库，通常需要使用复制和故障转移来实现高可用性。</span><span class="sxs-lookup"><span data-stu-id="c4709-204">For a relational database, you will typically need to use replication and failover to achieve high availability.</span></span>  

<span data-ttu-id="c4709-205">如果需要的可用性高于 [VM 的 Azure SLA][vm-sla] 提供的可用性，请跨两个区域复制应用程序，并使用 Azure 流量管理器进行故障转移。</span><span class="sxs-lookup"><span data-stu-id="c4709-205">If you need higher availability than the [Azure SLA for VMs][vm-sla] provides, replicate the application across two regions and use Azure Traffic Manager for failover.</span></span> <span data-ttu-id="c4709-206">有关详细信息，请参阅[在多个区域中运行 Linux VM 以实现高可用性][multi-dc]。</span><span class="sxs-lookup"><span data-stu-id="c4709-206">For more information, see [Run Linux VMs in multiple regions for high availability][multi-dc].</span></span>  

## <a name="security-considerations"></a><span data-ttu-id="c4709-207">安全注意事项</span><span class="sxs-lookup"><span data-stu-id="c4709-207">Security considerations</span></span>

<span data-ttu-id="c4709-208">请考虑添加一个网络虚拟设备 (NVA) 以在公共 Internet 与 Azure 虚拟网络之间创建一个外围网络。</span><span class="sxs-lookup"><span data-stu-id="c4709-208">Consider adding a network virtual appliance (NVA) to create a DMZ between the public Internet and the Azure virtual network.</span></span> <span data-ttu-id="c4709-209">NVA 是虚拟设备的一个通用术语，该虚拟设备可以执行与网络相关的任务，例如防火墙、包检查、审核和自定义路由。</span><span class="sxs-lookup"><span data-stu-id="c4709-209">NVA is a generic term for a virtual appliance that can perform network-related tasks such as firewall, packet inspection, auditing, and custom routing.</span></span> <span data-ttu-id="c4709-210">有关详细信息，请参阅[在 Azure 与 Internet 之间实现外围网络][dmz]。</span><span class="sxs-lookup"><span data-stu-id="c4709-210">For more information, see [Implementing a DMZ between Azure and the Internet][dmz].</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="c4709-211">可伸缩性注意事项</span><span class="sxs-lookup"><span data-stu-id="c4709-211">Scalability considerations</span></span>

<span data-ttu-id="c4709-212">负载均衡器将网络流量分布到 Web 层和业务层。</span><span class="sxs-lookup"><span data-stu-id="c4709-212">The load balancers distribute network traffic to the web and business tiers.</span></span> <span data-ttu-id="c4709-213">可以通过添加新的 VM 实例进行水平缩放。</span><span class="sxs-lookup"><span data-stu-id="c4709-213">Scale horizontally by adding new VM instances.</span></span> <span data-ttu-id="c4709-214">请注意，可以根据负载分别对 Web 层和业务层进行缩放。</span><span class="sxs-lookup"><span data-stu-id="c4709-214">Note that you can scale the web and business tiers independently, based on load.</span></span> <span data-ttu-id="c4709-215">为减少由于需要维护客户端相关性而可能导致的复杂情况，Web 层中的 VM 应当是无状态的。</span><span class="sxs-lookup"><span data-stu-id="c4709-215">To reduce possible complications caused by the need to maintain client affinity, the VMs in the web tier should be stateless.</span></span> <span data-ttu-id="c4709-216">托管着业务逻辑的 VM 也应当是无状态的。</span><span class="sxs-lookup"><span data-stu-id="c4709-216">The VMs hosting the business logic should also be stateless.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="c4709-217">可管理性注意事项</span><span class="sxs-lookup"><span data-stu-id="c4709-217">Manageability considerations</span></span>

<span data-ttu-id="c4709-218">可以通过使用 [Azure 自动化][azure-administration]、[Microsoft Operations Management Suite][operations-management-suite]、[Chef][chef] 或 [Puppet][puppet] 等集中式管理工具来简化整个系统的管理。</span><span class="sxs-lookup"><span data-stu-id="c4709-218">Simplify management of the entire system by using centralized administration tools such as [Azure Automation][azure-administration], [Microsoft Operations Management Suite][operations-management-suite], [Chef][chef], or [Puppet][puppet].</span></span> <span data-ttu-id="c4709-219">这些工具可以整合从多个 VM 捕获的诊断和运行状况信息，使用户可以全面了解该系统。</span><span class="sxs-lookup"><span data-stu-id="c4709-219">These tools can consolidate diagnostic and health information captured from multiple VMs to provide an overall view of the system.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="c4709-220">部署解决方案</span><span class="sxs-lookup"><span data-stu-id="c4709-220">Deploy the solution</span></span>

<span data-ttu-id="c4709-221">[GitHub][github-folder] 中提供了此参考体系结构的部署。</span><span class="sxs-lookup"><span data-stu-id="c4709-221">A deployment for this reference architecture is available on [GitHub][github-folder].</span></span> 

### <a name="prerequisites"></a><span data-ttu-id="c4709-222">系统必备</span><span class="sxs-lookup"><span data-stu-id="c4709-222">Prerequisites</span></span>

<span data-ttu-id="c4709-223">在将参考体系结构部署到自己的订阅之前，必须执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="c4709-223">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="c4709-224">为 [AzureCAT 参考体系结构][ref-arch-repo] GitHub 存储库克隆、下载 zip 文件或创建其分支。</span><span class="sxs-lookup"><span data-stu-id="c4709-224">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="c4709-225">确保在计算机上安装了 Azure CLI 2.0。</span><span class="sxs-lookup"><span data-stu-id="c4709-225">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="c4709-226">若要安装 CLI，请按照[安装 Azure CLI 2.0][azure-cli-2] 中的说明执行操作。</span><span class="sxs-lookup"><span data-stu-id="c4709-226">To install the CLI, follow the instructions in [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="c4709-227">安装 [Azure 构建基块][azbb] npm 包。</span><span class="sxs-lookup"><span data-stu-id="c4709-227">Install the [Azure building blocks][azbb] npm package.</span></span>

  ```bash
  npm install -g @mspnp/azure-building-blocks
  ```

4. <span data-ttu-id="c4709-228">从命令提示符、bash 提示符或 PowerShell 提示符下通过使用以下命令之一，登录到 Azure 帐户，然后按照提示进行操作。</span><span class="sxs-lookup"><span data-stu-id="c4709-228">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using one of the commands below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a><span data-ttu-id="c4709-229">使用 azbb 部署解决方案</span><span class="sxs-lookup"><span data-stu-id="c4709-229">Deploy the solution using azbb</span></span>

<span data-ttu-id="c4709-230">若要为 N 层应用程序参考体系结构部署 Linux VM，请执行以下步骤：</span><span class="sxs-lookup"><span data-stu-id="c4709-230">To deploy the Linux VMs for an N-tier application reference architecture, follow these steps:</span></span>

1. <span data-ttu-id="c4709-231">导航到在以上前决条件的第 1 步中克隆的存储库的 `virtual-machines\n-tier-linux` 文件夹。</span><span class="sxs-lookup"><span data-stu-id="c4709-231">Navigate to the `virtual-machines\n-tier-linux` folder for the repository you cloned in step 1 of the pre-requisites above.</span></span>

2. <span data-ttu-id="c4709-232">参数文件为部署中的每个 VM 指定默认管理员用户名称和密码。</span><span class="sxs-lookup"><span data-stu-id="c4709-232">The parameter file specifies a default adminstrator user name and password for each VM in the deployment.</span></span> <span data-ttu-id="c4709-233">必须在部署参考体系结构前，对其进行更改。</span><span class="sxs-lookup"><span data-stu-id="c4709-233">You must change these before you deploy the reference architecture.</span></span> <span data-ttu-id="c4709-234">打开 `n-tier-linux.json` 文件，然后将每个“adminUsername”和“adminPassword”字段替换为新设置。</span><span class="sxs-lookup"><span data-stu-id="c4709-234">Open the `n-tier-linux.json` file and replace each **adminUsername** and **adminPassword** field with your new settings.</span></span>   <span data-ttu-id="c4709-235">保存文件。</span><span class="sxs-lookup"><span data-stu-id="c4709-235">Save the file.</span></span>

3. <span data-ttu-id="c4709-236">使用 azbb 命令行工具部署参考体系结构，如下所示。</span><span class="sxs-lookup"><span data-stu-id="c4709-236">Deploy the reference architecture using the **azbb** command line tool as shown below.</span></span>

  ```bash
  azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-linux.json --deploy
  ```

<span data-ttu-id="c4709-237">若要详细了解如何使用 Azure 构建基块部署此示例参考体系结构，请访问 [GitHub 存储库][git]。</span><span class="sxs-lookup"><span data-stu-id="c4709-237">For more information on deploying this sample reference architecture using Azure Building Blocks, visit the [GitHub repository][git].</span></span>

<!-- links -->
[multi-dc]: multi-region-application.md
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-vm]: ./multi-vm.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[守护主机]: https://en.wikipedia.org/wiki/Bastion_host
[cassandra-in-azure]: https://docs.datastax.com/en/datastax_enterprise/4.5/datastax_enterprise/install/installAzure.html
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[datastax]: http://www.datastax.com/products/datastax-enterprise
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-linux
[lb-external-create]: /azure/load-balancer/load-balancer-get-started-internet-portal
[lb-internal-create]: /azure/load-balancer/load-balancer-get-started-ilb-arm-portal
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-rules]: /azure/azure-resource-manager/best-practices-resource-manager-security#network-security-groups
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[公共 IP 地址]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[Nagios]: https://www.nagios.org/
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[0]: ./images/n-tier-diagram.png "使用 Microsoft Azure 的 N 层体系结构"

