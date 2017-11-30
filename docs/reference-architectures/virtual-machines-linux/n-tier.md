---
title: "在 Azure 上运行用于 N 层应用程序的 Linux VM"
description: "如何在 Microsoft Azure 中运行用于 N 层体系结构的 Linux VM。"
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Linux VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: 673e090e306ffc421603371658c8273b10b980d4
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="run-linux-vms-for-an-n-tier-application"></a><span data-ttu-id="0ce03-103">运行用于 N 层应用程序的 Linux VM</span><span class="sxs-lookup"><span data-stu-id="0ce03-103">Run Linux VMs for an N-tier application</span></span>

<span data-ttu-id="0ce03-104">此参考体系结构显示了一组经过实践检验的、运行用于 N 层应用程序的 Linux 虚拟机 (VM) 的做法。</span><span class="sxs-lookup"><span data-stu-id="0ce03-104">This reference architecture shows a set of proven practices for running Linux virtual machines (VMs) for an N-tier application.</span></span> [<span data-ttu-id="0ce03-105">**部署此解决方案**。</span><span class="sxs-lookup"><span data-stu-id="0ce03-105">**Deploy this solution**.</span></span>](#deploy-the-solution)  

<span data-ttu-id="0ce03-106">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="0ce03-106">![[0]][0]</span></span>

<span data-ttu-id="0ce03-107">下载此体系结构的 [Visio 文件][visio-download]。</span><span class="sxs-lookup"><span data-stu-id="0ce03-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="0ce03-108">体系结构</span><span class="sxs-lookup"><span data-stu-id="0ce03-108">Architecture</span></span>

<span data-ttu-id="0ce03-109">有许多方法可用来实现 N 层体系结构。</span><span class="sxs-lookup"><span data-stu-id="0ce03-109">There are many ways to implement an N-tier architecture.</span></span> <span data-ttu-id="0ce03-110">关系图中显示了一个典型的 3 层 Web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="0ce03-110">The diagram shows a typical 3-tier web application.</span></span> <span data-ttu-id="0ce03-111">此体系结构基于[运行负载均衡的 VM 以提高可伸缩性和可用性][multi-vm]中所述的内容。</span><span class="sxs-lookup"><span data-stu-id="0ce03-111">This architecture builds on [Run load-balanced VMs for scalability and availability][multi-vm].</span></span> <span data-ttu-id="0ce03-112">Web 层和业务层都使用负载均衡的 VM。</span><span class="sxs-lookup"><span data-stu-id="0ce03-112">The web and business tiers use load-balanced VMs.</span></span>

* <span data-ttu-id="0ce03-113">**可用性集。**</span><span class="sxs-lookup"><span data-stu-id="0ce03-113">**Availability sets.**</span></span> <span data-ttu-id="0ce03-114">为每个层创建一个[可用性集][azure-availability-sets]，并且在每个层中至少预配两个 VM。</span><span class="sxs-lookup"><span data-stu-id="0ce03-114">Create an [availability set][azure-availability-sets] for each tier, and provision at least two VMs in each tier.</span></span>  <span data-ttu-id="0ce03-115">这样，VM 便可以满足 VM 的更高[服务级别协议 (SLA)][vm-sla]。</span><span class="sxs-lookup"><span data-stu-id="0ce03-115">This makes the VMs eligible for a higher [service level agreement (SLA)][vm-sla] for VMs.</span></span>
* <span data-ttu-id="0ce03-116">**子网。**</span><span class="sxs-lookup"><span data-stu-id="0ce03-116">**Subnets.**</span></span> <span data-ttu-id="0ce03-117">为每个层创建一个单独的子网。</span><span class="sxs-lookup"><span data-stu-id="0ce03-117">Create a separate subnet for each tier.</span></span> <span data-ttu-id="0ce03-118">使用 [CIDR] 表示法指定地址范围和子网掩码。</span><span class="sxs-lookup"><span data-stu-id="0ce03-118">Specify the address range and subnet mask using [CIDR] notation.</span></span> 
* <span data-ttu-id="0ce03-119">**负载均衡器。**</span><span class="sxs-lookup"><span data-stu-id="0ce03-119">**Load balancers.**</span></span> <span data-ttu-id="0ce03-120">使用[面向 Internet 的负载均衡器][load-balancer-external]将传入的 Internet 流量分布到 Web 层，使用[内部负载均衡器][load-balancer-internal]将来自 Web 层的网络流量分布到业务层。</span><span class="sxs-lookup"><span data-stu-id="0ce03-120">Use an [Internet-facing load balancer][load-balancer-external] to distribute incoming Internet traffic to the web tier, and an [internal load balancer][load-balancer-internal] to distribute network traffic from the web tier to the business tier.</span></span>
* <span data-ttu-id="0ce03-121">**Jumpbox。**</span><span class="sxs-lookup"><span data-stu-id="0ce03-121">**Jumpbox.**</span></span> <span data-ttu-id="0ce03-122">也称为[守护主机]。</span><span class="sxs-lookup"><span data-stu-id="0ce03-122">Also called a [bastion host].</span></span> <span data-ttu-id="0ce03-123">网络上的一个安全 VM，管理员使用它来连接到其他 VM。</span><span class="sxs-lookup"><span data-stu-id="0ce03-123">A secure VM on the network that administrators use to connect to the other VMs.</span></span> <span data-ttu-id="0ce03-124">Jumpbox 中的某个 NSG 只允许来自安全列表中的公共 IP 地址的远程流量。</span><span class="sxs-lookup"><span data-stu-id="0ce03-124">The jumpbox has an NSG that allows remote traffic only from public IP addresses on a safe list.</span></span> <span data-ttu-id="0ce03-125">NSG 应允许安全外壳 (SSH) 流量。</span><span class="sxs-lookup"><span data-stu-id="0ce03-125">The NSG should permit secure shell (SSH) traffic.</span></span>
* <span data-ttu-id="0ce03-126">**监视。**</span><span class="sxs-lookup"><span data-stu-id="0ce03-126">**Monitoring.**</span></span> <span data-ttu-id="0ce03-127">可以使用 [Nagios]、[Zabbix] 或 [Icinga] 等监视软件深入了解响应时间、VM 运行时间和系统的整体运行状况。</span><span class="sxs-lookup"><span data-stu-id="0ce03-127">Monitoring software such as [Nagios], [Zabbix], or [Icinga] can give you insight into response time, VM uptime, and the overall health of your system.</span></span> <span data-ttu-id="0ce03-128">在置于单独的管理子网中的 VM 上安装监视软件。</span><span class="sxs-lookup"><span data-stu-id="0ce03-128">Install the monitoring software on a VM that's placed in a separate management subnet.</span></span>
* <span data-ttu-id="0ce03-129">**NSG。**</span><span class="sxs-lookup"><span data-stu-id="0ce03-129">**NSGs.**</span></span> <span data-ttu-id="0ce03-130">使用[网络安全组][nsg] (NSG) 来限制 VNet 中的网络流量。</span><span class="sxs-lookup"><span data-stu-id="0ce03-130">Use [network security groups][nsg] (NSGs) to restrict network traffic within the VNet.</span></span> <span data-ttu-id="0ce03-131">例如，在此处显示的 3 层体系结构中，数据库层不接受来自 Web 前端的流量，仅接受来自业务层和管理子网的流量。</span><span class="sxs-lookup"><span data-stu-id="0ce03-131">For example, in the 3-tier architecture shown here, the database tier does not accept traffic from the web front end, only from the business tier and the management subnet.</span></span>
* <span data-ttu-id="0ce03-132">**Apache Cassandra 数据库**。</span><span class="sxs-lookup"><span data-stu-id="0ce03-132">**Apache Cassandra database**.</span></span> <span data-ttu-id="0ce03-133">通过启用复制和故障转移，在数据层提供高可用性。</span><span class="sxs-lookup"><span data-stu-id="0ce03-133">Provides high availability at the data tier, by enabling replication and failover.</span></span>

## <a name="recommendations"></a><span data-ttu-id="0ce03-134">建议</span><span class="sxs-lookup"><span data-stu-id="0ce03-134">Recommendations</span></span>

<span data-ttu-id="0ce03-135">你的要求可能不同于此处描述的体系结构。</span><span class="sxs-lookup"><span data-stu-id="0ce03-135">Your requirements might differ from the architecture described here.</span></span> <span data-ttu-id="0ce03-136">请使用以下建议作为入手点。</span><span class="sxs-lookup"><span data-stu-id="0ce03-136">Use these recommendations as a starting point.</span></span> 

### <a name="vnet--subnets"></a><span data-ttu-id="0ce03-137">VNet/子网</span><span class="sxs-lookup"><span data-stu-id="0ce03-137">VNet / Subnets</span></span>

<span data-ttu-id="0ce03-138">在创建 VNet 时，确定每个子网中的资源需要多少 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="0ce03-138">When you create the VNet, determine how many IP addresses your resources in each subnet require.</span></span> <span data-ttu-id="0ce03-139">使用 [CIDR] 表示法为所需的 IP 地址指定子网掩码和足够大的 VNet 地址范围。</span><span class="sxs-lookup"><span data-stu-id="0ce03-139">Specify a subnet mask and a VNet address range large enough for the required IP addresses using [CIDR] notation.</span></span> <span data-ttu-id="0ce03-140">使用标准[专用 IP 地址块][private-ip-space]内的一个地址空间，这些地址块为 10.0.0.0/8、172.16.0.0/12 和 192.168.0.0/16。</span><span class="sxs-lookup"><span data-stu-id="0ce03-140">Use an address space that falls within the standard [private IP address blocks][private-ip-space], which are 10.0.0.0/8, 172.16.0.0/12, and 192.168.0.0/16.</span></span>

<span data-ttu-id="0ce03-141">如果以后需要在 VNet 与本地网络之间设置一个网关，请选择一个不与本地网络重叠的地址范围。</span><span class="sxs-lookup"><span data-stu-id="0ce03-141">Choose an address range that does not overlap with your on-premises network, in case you need to set up a gateway between the VNet and your on-premises network later.</span></span> <span data-ttu-id="0ce03-142">在创建 VNet 后，将无法更改地址范围。</span><span class="sxs-lookup"><span data-stu-id="0ce03-142">Once you create the VNet, you can't change the address range.</span></span>

<span data-ttu-id="0ce03-143">在设计子网时一定要牢记功能和安全要求。</span><span class="sxs-lookup"><span data-stu-id="0ce03-143">Design subnets with functionality and security requirements in mind.</span></span> <span data-ttu-id="0ce03-144">同一层或同一角色中的所有 VM 应当置于同一子网，这可能是一个安全边界。</span><span class="sxs-lookup"><span data-stu-id="0ce03-144">All VMs within the same tier or role should go into the same subnet, which can be a security boundary.</span></span> <span data-ttu-id="0ce03-145">有关设计 VNet 和子网的详细信息，请参阅[规划和设计 Azure 虚拟网络][plan-network]。</span><span class="sxs-lookup"><span data-stu-id="0ce03-145">For more information about designing VNets and subnets, see [Plan and design Azure Virtual Networks][plan-network].</span></span>

<span data-ttu-id="0ce03-146">针对每个子网，采用 CIDR 表示法指定子网的地址空间。</span><span class="sxs-lookup"><span data-stu-id="0ce03-146">For each subnet, specify the address space for the subnet in CIDR notation.</span></span> <span data-ttu-id="0ce03-147">例如，“10.0.0.0/24”将创建包含 256 个 IP 地址的范围。</span><span class="sxs-lookup"><span data-stu-id="0ce03-147">For example, '10.0.0.0/24' creates a range of 256 IP addresses.</span></span> <span data-ttu-id="0ce03-148">VM 可以使用其中的 251 个；5 个会被保留。</span><span class="sxs-lookup"><span data-stu-id="0ce03-148">VMs can use 251 of these; five are reserved.</span></span> <span data-ttu-id="0ce03-149">请确保地址范围在子网之间不重叠。</span><span class="sxs-lookup"><span data-stu-id="0ce03-149">Make sure the address ranges don't overlap across subnets.</span></span> <span data-ttu-id="0ce03-150">请参阅[虚拟网络常见问题解答][vnet faq]。</span><span class="sxs-lookup"><span data-stu-id="0ce03-150">See the [Virtual Network FAQ][vnet faq].</span></span>

### <a name="network-security-groups"></a><span data-ttu-id="0ce03-151">网络安全组</span><span class="sxs-lookup"><span data-stu-id="0ce03-151">Network security groups</span></span>

<span data-ttu-id="0ce03-152">使用 NSG 规则限制各个层之间的流量。</span><span class="sxs-lookup"><span data-stu-id="0ce03-152">Use NSG rules to restrict traffic between tiers.</span></span> <span data-ttu-id="0ce03-153">例如，在上面显示的 3 层体系结构中，Web 层不直接与数据库层进行通信。</span><span class="sxs-lookup"><span data-stu-id="0ce03-153">For example, in the 3-tier architecture shown above, the web tier does not communicate directly with the database tier.</span></span> <span data-ttu-id="0ce03-154">为强制实现此目的，数据库层应当阻止来自 Web 层子网的传入流量。</span><span class="sxs-lookup"><span data-stu-id="0ce03-154">To enforce this, the database tier should block incoming traffic from the web tier subnet.</span></span>  

1. <span data-ttu-id="0ce03-155">创建一个 NSG 并将其关联到数据库层子网。</span><span class="sxs-lookup"><span data-stu-id="0ce03-155">Create an NSG and associate it to the database tier subnet.</span></span>
2. <span data-ttu-id="0ce03-156">添加一个拒绝来自 VNet 的所有入站流量的规则。</span><span class="sxs-lookup"><span data-stu-id="0ce03-156">Add a rule that denies all inbound traffic from the VNet.</span></span> <span data-ttu-id="0ce03-157">（在规则中使用 `VIRTUAL_NETWORK` 标记。）</span><span class="sxs-lookup"><span data-stu-id="0ce03-157">(Use the `VIRTUAL_NETWORK` tag in the rule.)</span></span> 
3. <span data-ttu-id="0ce03-158">添加一个具有更高优先级的规则，用以允许来自业务层子网的入站流量。</span><span class="sxs-lookup"><span data-stu-id="0ce03-158">Add a rule with a higher priority that allows inbound traffic from the business tier subnet.</span></span> <span data-ttu-id="0ce03-159">此规则将替代前面的规则，并允许业务层与数据库层进行通信。</span><span class="sxs-lookup"><span data-stu-id="0ce03-159">This rule overrides the previous rule, and allows the business tier to talk to the database tier.</span></span>
4. <span data-ttu-id="0ce03-160">添加一个允许来自数据库层子网自身的入站流量的规则。</span><span class="sxs-lookup"><span data-stu-id="0ce03-160">Add a rule that allows inbound traffic from within the database tier subnet itself.</span></span> <span data-ttu-id="0ce03-161">此规则允许在数据库层中的各个 VM 之间进行通信，这是进行数据库复制和故障转移所必需的。</span><span class="sxs-lookup"><span data-stu-id="0ce03-161">This rule allows communication between VMs in the database tier, which is needed for database replication and failover.</span></span>
5. <span data-ttu-id="0ce03-162">添加一个允许来自 jumpbox 子网的 SSH 流量的规则。</span><span class="sxs-lookup"><span data-stu-id="0ce03-162">Add a rule that allows SSH traffic from the jumpbox subnet.</span></span> <span data-ttu-id="0ce03-163">此规则允许管理员从 jumpbox 连接到数据库层。</span><span class="sxs-lookup"><span data-stu-id="0ce03-163">This rule lets administrators connect to the database tier from the jumpbox.</span></span>
   
   > [!NOTE]
   > <span data-ttu-id="0ce03-164">NSG 包含允许来自 VNet 的任何入站流量的[默认规则][nsg-rules]。</span><span class="sxs-lookup"><span data-stu-id="0ce03-164">An NSG has [default rules][nsg-rules] that allow any inbound traffic from within the VNet.</span></span> <span data-ttu-id="0ce03-165">这些规则无法删除，但可以通过创建优先级更高的规则来替代它们。</span><span class="sxs-lookup"><span data-stu-id="0ce03-165">These rules can't be deleted, but you can override them by creating higher-priority rules.</span></span>
   > 
   > 

### <a name="load-balancers"></a><span data-ttu-id="0ce03-166">负载均衡器</span><span class="sxs-lookup"><span data-stu-id="0ce03-166">Load balancers</span></span>

<span data-ttu-id="0ce03-167">外部负载均衡器将 Internet 流量分布到 Web 层。</span><span class="sxs-lookup"><span data-stu-id="0ce03-167">The external load balancer distributes Internet traffic to the web tier.</span></span> <span data-ttu-id="0ce03-168">为此负载均衡器创建一个公共 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="0ce03-168">Create a public IP address for this load balancer.</span></span> <span data-ttu-id="0ce03-169">请参阅[创建面向 Internet 的负载均衡器][lb-external-create]。</span><span class="sxs-lookup"><span data-stu-id="0ce03-169">See [Creating an Internet-facing load balancer][lb-external-create].</span></span>

<span data-ttu-id="0ce03-170">内部负载均衡器将网络流量从 Web 层分布到业务层。</span><span class="sxs-lookup"><span data-stu-id="0ce03-170">The internal load balancer distributes network traffic from the web tier to the business tier.</span></span> <span data-ttu-id="0ce03-171">若要为此负载均衡器提供专用 IP 地址，请创建一个前端 IP 配置，并将其与业务层的子网相关联。</span><span class="sxs-lookup"><span data-stu-id="0ce03-171">To give this load balancer a private IP address, create a frontend IP configuration and associate it with the subnet for the business tier.</span></span> <span data-ttu-id="0ce03-172">请参阅[开始创建内部负载均衡器][lb-internal-create]。</span><span class="sxs-lookup"><span data-stu-id="0ce03-172">See [Get started creating an Internal load balancer][lb-internal-create].</span></span>

### <a name="cassandra"></a><span data-ttu-id="0ce03-173">Cassandra</span><span class="sxs-lookup"><span data-stu-id="0ce03-173">Cassandra</span></span>

<span data-ttu-id="0ce03-174">我们建议将 [DataStax Enterprise][datastax] 用于生产用途，但这些建议适用于任何 Cassandra 版本。</span><span class="sxs-lookup"><span data-stu-id="0ce03-174">We recommend [DataStax Enterprise][datastax] for production use, but these recommendations apply to any Cassandra edition.</span></span> <span data-ttu-id="0ce03-175">有关在 Azure 中运行 DataStax 的详细信息，请参阅[适用于 Azure 的 DataStax Enterprise 部署指南][cassandra-in-azure]。</span><span class="sxs-lookup"><span data-stu-id="0ce03-175">For more information on running DataStax in Azure, see [DataStax Enterprise Deployment Guide for Azure][cassandra-in-azure].</span></span> 

<span data-ttu-id="0ce03-176">将 Cassandra 群集的 VM 放置在一个可用性集中，以确保 Cassandra 副本分布在多个容错域和升级域中。</span><span class="sxs-lookup"><span data-stu-id="0ce03-176">Put the VMs for a Cassandra cluster in an availability set to ensure that the Cassandra replicas are distributed across multiple fault domains and upgrade domains.</span></span> <span data-ttu-id="0ce03-177">有关容错域和升级域的详细信息，请参阅[管理虚拟机的可用性][azure-availability-sets]。</span><span class="sxs-lookup"><span data-stu-id="0ce03-177">For more information about fault domains and upgrade domains, see [Manage the availability of virtual machines][azure-availability-sets].</span></span> 

<span data-ttu-id="0ce03-178">为每个可用性集配置三个容错域（最大数目），并且为每个可用性集配置 18 个升级域。</span><span class="sxs-lookup"><span data-stu-id="0ce03-178">Configure three fault domains (the maximum) per availability set and 18 upgrade domains per availability set.</span></span> <span data-ttu-id="0ce03-179">这提供了仍可均匀分布在容错域中的升级域的最大数量。</span><span class="sxs-lookup"><span data-stu-id="0ce03-179">This provides the maximum number of upgrade domains that can still be distributed evenly across the fault domains.</span></span>   

<span data-ttu-id="0ce03-180">以机架感知模式配置节点。</span><span class="sxs-lookup"><span data-stu-id="0ce03-180">Configure nodes in rack-aware mode.</span></span> <span data-ttu-id="0ce03-181">将故障域映射到 `cassandra-rackdc.properties` 文件中的机架。</span><span class="sxs-lookup"><span data-stu-id="0ce03-181">Map fault domains to racks in the `cassandra-rackdc.properties` file.</span></span>

<span data-ttu-id="0ce03-182">群集前面不需要负载均衡器。</span><span class="sxs-lookup"><span data-stu-id="0ce03-182">You don't need a load balancer in front of the cluster.</span></span> <span data-ttu-id="0ce03-183">客户端会直接连接到群集中的节点。</span><span class="sxs-lookup"><span data-stu-id="0ce03-183">The client connects directly to a node in the cluster.</span></span>

### <a name="jumpbox"></a><span data-ttu-id="0ce03-184">Jumpbox</span><span class="sxs-lookup"><span data-stu-id="0ce03-184">Jumpbox</span></span>

<span data-ttu-id="0ce03-185">Jumpbox 的性能要求非常低，因此请为 jumpbox 选择一个较小的 VM 大小，例如标准 A1。</span><span class="sxs-lookup"><span data-stu-id="0ce03-185">The jumpbox will have minimal performance requirements, so select a small VM size for the jumpbox such as Standard A1.</span></span> 

<span data-ttu-id="0ce03-186">为 jumpbox 创建一个[公共 IP 地址]。</span><span class="sxs-lookup"><span data-stu-id="0ce03-186">Create a [public IP address] for the jumpbox.</span></span> <span data-ttu-id="0ce03-187">将 jumpbox 放置在与其他 VM 相同的 VNet 中，但将其置于一个单独的管理子网中。</span><span class="sxs-lookup"><span data-stu-id="0ce03-187">Place the jumpbox in the same VNet as the other VMs, but in a separate management subnet.</span></span>

<span data-ttu-id="0ce03-188">不要允许从公共 Internet 对运行应用程序工作负荷的 VM 进行 SSH 访问。</span><span class="sxs-lookup"><span data-stu-id="0ce03-188">Do not allow SSH access from the public Internet to the VMs that run the application workload.</span></span> <span data-ttu-id="0ce03-189">相反，对这些 VM 的所有 SSH 访问都必须通过 jumpbox 进行。</span><span class="sxs-lookup"><span data-stu-id="0ce03-189">Instead, all SSH access to these VMs must come through the jumpbox.</span></span> <span data-ttu-id="0ce03-190">管理员登录到 jumpbox，然后从 jumpbox 登录到其他 VM。</span><span class="sxs-lookup"><span data-stu-id="0ce03-190">An administrator logs into the jumpbox, and then logs into the other VM from the jumpbox.</span></span> <span data-ttu-id="0ce03-191">Jumpbox 允许来自 Internet 的 SSH 流量，但仅允许来自已知的安全 IP 地址的流量。</span><span class="sxs-lookup"><span data-stu-id="0ce03-191">The jumpbox allows SSH traffic from the Internet, but only from known, safe IP addresses.</span></span>

<span data-ttu-id="0ce03-192">若要保护 jumpbox，请创建一个 NSG 并将其应用于 jumpbox 子网。</span><span class="sxs-lookup"><span data-stu-id="0ce03-192">To secure the jumpbox, create an NSG and apply it to the jumpbox subnet.</span></span> <span data-ttu-id="0ce03-193">添加一个 NSG 规则，使其仅允许来自一组安全的公共 IP 地址的 SSH 连接。</span><span class="sxs-lookup"><span data-stu-id="0ce03-193">Add an NSG rule that allows SSH connections only from a safe set of public IP addresses.</span></span> <span data-ttu-id="0ce03-194">可以将 NSG 附加到子网或 jumpbox NIC。</span><span class="sxs-lookup"><span data-stu-id="0ce03-194">The NSG can be attached either to the subnet or to the jumpbox NIC.</span></span> <span data-ttu-id="0ce03-195">在本例中，我们建议将其附加到 NIC，以便仅允许将 SSH 流量发送到 jumpbox，即使将其他 VM 添加到同一子网也是如此。</span><span class="sxs-lookup"><span data-stu-id="0ce03-195">In this case, we recommend attaching it to the NIC, so SSH traffic is permitted only to the jumpbox, even if you add other VMs to the same subnet.</span></span>

<span data-ttu-id="0ce03-196">为其他子网配置 NSG 以允许来自管理子网的 SSH 流量。</span><span class="sxs-lookup"><span data-stu-id="0ce03-196">Configure the NSGs for the other subnets to allow SSH traffic from the management subnet.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="0ce03-197">可用性注意事项</span><span class="sxs-lookup"><span data-stu-id="0ce03-197">Availability considerations</span></span>

<span data-ttu-id="0ce03-198">将每个层或 VM 角色放入单独的可用性集中。</span><span class="sxs-lookup"><span data-stu-id="0ce03-198">Put each tier or VM role into a separate availability set.</span></span> 

<span data-ttu-id="0ce03-199">在数据库层，具有多个 VM 不会自动转换为高度可用的数据库。</span><span class="sxs-lookup"><span data-stu-id="0ce03-199">At the database tier, having multiple VMs does not automatically translate into a highly available database.</span></span> <span data-ttu-id="0ce03-200">对于关系数据库，通常需要使用复制和故障转移来实现高可用性。</span><span class="sxs-lookup"><span data-stu-id="0ce03-200">For a relational database, you will typically need to use replication and failover to achieve high availability.</span></span>  

<span data-ttu-id="0ce03-201">如果需要的可用性高于 [VM 的 Azure SLA][vm-sla] 提供的可用性，请跨两个区域复制应用程序，并使用 Azure 流量管理器进行故障转移。</span><span class="sxs-lookup"><span data-stu-id="0ce03-201">If you need higher availability than the [Azure SLA for VMs][vm-sla] provides, replicate the application across two regions and use Azure Traffic Manager for failover.</span></span> <span data-ttu-id="0ce03-202">有关详细信息，请参阅[在多个区域中运行 Linux VM 以实现高可用性][multi-dc]。</span><span class="sxs-lookup"><span data-stu-id="0ce03-202">For more information, see [Run Linux VMs in multiple regions for high availability][multi-dc].</span></span>  

## <a name="security-considerations"></a><span data-ttu-id="0ce03-203">安全注意事项</span><span class="sxs-lookup"><span data-stu-id="0ce03-203">Security considerations</span></span>

<span data-ttu-id="0ce03-204">请考虑添加一个网络虚拟设备 (NVA) 以在公共 Internet 与 Azure 虚拟网络之间创建一个外围网络。</span><span class="sxs-lookup"><span data-stu-id="0ce03-204">Consider adding a network virtual appliance (NVA) to create a DMZ between the public Internet and the Azure virtual network.</span></span> <span data-ttu-id="0ce03-205">NVA 是虚拟设备的一个通用术语，该虚拟设备可以执行与网络相关的任务，例如防火墙、包检查、审核和自定义路由。</span><span class="sxs-lookup"><span data-stu-id="0ce03-205">NVA is a generic term for a virtual appliance that can perform network-related tasks such as firewall, packet inspection, auditing, and custom routing.</span></span> <span data-ttu-id="0ce03-206">有关详细信息，请参阅[在 Azure 与 Internet 之间实现外围网络][dmz]。</span><span class="sxs-lookup"><span data-stu-id="0ce03-206">For more information, see [Implementing a DMZ between Azure and the Internet][dmz].</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="0ce03-207">可伸缩性注意事项</span><span class="sxs-lookup"><span data-stu-id="0ce03-207">Scalability considerations</span></span>

<span data-ttu-id="0ce03-208">负载均衡器将网络流量分布到 Web 层和业务层。</span><span class="sxs-lookup"><span data-stu-id="0ce03-208">The load balancers distribute network traffic to the web and business tiers.</span></span> <span data-ttu-id="0ce03-209">可以通过添加新的 VM 实例进行水平缩放。</span><span class="sxs-lookup"><span data-stu-id="0ce03-209">Scale horizontally by adding new VM instances.</span></span> <span data-ttu-id="0ce03-210">请注意，可以根据负载分别对 Web 层和业务层进行缩放。</span><span class="sxs-lookup"><span data-stu-id="0ce03-210">Note that you can scale the web and business tiers independently, based on load.</span></span> <span data-ttu-id="0ce03-211">为减少由于需要维护客户端相关性而可能导致的复杂情况，Web 层中的 VM 应当是无状态的。</span><span class="sxs-lookup"><span data-stu-id="0ce03-211">To reduce possible complications caused by the need to maintain client affinity, the VMs in the web tier should be stateless.</span></span> <span data-ttu-id="0ce03-212">托管着业务逻辑的 VM 也应当是无状态的。</span><span class="sxs-lookup"><span data-stu-id="0ce03-212">The VMs hosting the business logic should also be stateless.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="0ce03-213">可管理性注意事项</span><span class="sxs-lookup"><span data-stu-id="0ce03-213">Manageability considerations</span></span>

<span data-ttu-id="0ce03-214">可以通过使用 [Azure 自动化][azure-administration]、[Microsoft Operations Management Suite][operations-management-suite]、[Chef][chef] 或 [Puppet][puppet] 等集中式管理工具来简化整个系统的管理。</span><span class="sxs-lookup"><span data-stu-id="0ce03-214">Simplify management of the entire system by using centralized administration tools such as [Azure Automation][azure-administration], [Microsoft Operations Management Suite][operations-management-suite], [Chef][chef], or [Puppet][puppet].</span></span> <span data-ttu-id="0ce03-215">这些工具可以整合从多个 VM 捕获的诊断和运行状况信息，使用户可以全面了解该系统。</span><span class="sxs-lookup"><span data-stu-id="0ce03-215">These tools can consolidate diagnostic and health information captured from multiple VMs to provide an overall view of the system.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="0ce03-216">部署解决方案</span><span class="sxs-lookup"><span data-stu-id="0ce03-216">Deploy the solution</span></span>

<span data-ttu-id="0ce03-217">[GitHub][github-folder] 上提供了此体系结构的部署。</span><span class="sxs-lookup"><span data-stu-id="0ce03-217">A deployment for this architecture is available on [GitHub][github-folder].</span></span> <span data-ttu-id="0ce03-218">此体系结构是分三个阶段部署的。</span><span class="sxs-lookup"><span data-stu-id="0ce03-218">The architecture is deployed in three stages.</span></span> <span data-ttu-id="0ce03-219">若要部署该体系结构，请执行以下步骤：</span><span class="sxs-lookup"><span data-stu-id="0ce03-219">To deploy the architecture, follow these steps:</span></span> 

1. <span data-ttu-id="0ce03-220">单击下面的按钮：</span><span class="sxs-lookup"><span data-stu-id="0ce03-220">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fvirtual-machines%2Fn-tier-linux%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="0ce03-221">当链接在 Azure 门户中打开后，输入以下值：</span><span class="sxs-lookup"><span data-stu-id="0ce03-221">Once the link has opened in the Azure portal, enter the follow values:</span></span> 
   * <span data-ttu-id="0ce03-222">参数文件中已定义**资源组**名称，因此请选择“新建”，并在文本框中输入 `ra-ntier-cassandra-rg`。</span><span class="sxs-lookup"><span data-stu-id="0ce03-222">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-ntier-cassandra-rg` in the text box.</span></span>
   * <span data-ttu-id="0ce03-223">从“位置”下拉框中选择区域。</span><span class="sxs-lookup"><span data-stu-id="0ce03-223">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="0ce03-224">不要编辑“模板根 URI”或“参数根 URI”文本框。</span><span class="sxs-lookup"><span data-stu-id="0ce03-224">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="0ce03-225">查看条款和条件，并单击“我同意上述条款和条件”复选框。</span><span class="sxs-lookup"><span data-stu-id="0ce03-225">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="0ce03-226">单击“购买”按钮。</span><span class="sxs-lookup"><span data-stu-id="0ce03-226">Click on the **Purchase** button.</span></span>
3. <span data-ttu-id="0ce03-227">检查 Azure 门户通知，以查找指明部署已完成的消息。</span><span class="sxs-lookup"><span data-stu-id="0ce03-227">Check Azure portal notification for a message the deployment is complete.</span></span>
4. <span data-ttu-id="0ce03-228">参数文件包括硬编码的管理员用户名和密码，强烈建议立即在所有 VM 上更改它们。</span><span class="sxs-lookup"><span data-stu-id="0ce03-228">The parameter files include a hard-coded administrator user names and passwords, and it is strongly recommended that you immediately change both on all the VMs.</span></span> <span data-ttu-id="0ce03-229">在 Azure 门户中单击每个 VM，然后，在“支持 + 故障排除”边栏选项卡中单击“重置密码”。</span><span class="sxs-lookup"><span data-stu-id="0ce03-229">Click on each VM in the Azure portal then click on **Reset password** in the **Support + troubleshooting** blade.</span></span> <span data-ttu-id="0ce03-230">在“模式”下拉框中选择“重置密码”，并选择新**用户名**和**密码**。</span><span class="sxs-lookup"><span data-stu-id="0ce03-230">Select **Reset password** in the **Mode** dropdown box, then select a new **User name** and **Password**.</span></span> <span data-ttu-id="0ce03-231">单击“更新”按钮保存新用户名和密码。</span><span class="sxs-lookup"><span data-stu-id="0ce03-231">Click the **Update** button to persist the new user name and password.</span></span>

<!-- links -->
[multi-dc]: multi-region-application.md
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-vm]: ./multi-vm.md
[naming conventions]: /azure/guidance/guidance-naming-conventions

[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[守护主机]: https://en.wikipedia.org/wiki/Bastion_host
[cassandra-in-azure]: https://docs.datastax.com/en/datastax_enterprise/4.5/datastax_enterprise/install/installAzure.html
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[datastax]: http://www.datastax.com/products/datastax-enterprise
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
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[Nagios]: https://www.nagios.org/
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[0]: ./images/n-tier-diagram.png "使用 Microsoft Azure 的 N 层体系结构"

