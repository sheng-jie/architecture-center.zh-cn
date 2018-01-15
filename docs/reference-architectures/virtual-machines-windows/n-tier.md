---
title: "运行用于 N 层体系结构的 Windows VM"
description: "如何在 Azure 上实现多层体系结构，并且特别注意可用性、安全性、可伸缩性和可管理性安全。"
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Windows VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: 0654239a5bbd966a2aa776415b7f15ae723ffd63
ms.sourcegitcommit: c9e6d8edb069b8c513de748ce8114c879bad5f49
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/08/2018
---
# <a name="run-windows-vms-for-an-n-tier-application"></a><span data-ttu-id="0bc8f-103">运行用于 N 层应用程序的 Windows VM</span><span class="sxs-lookup"><span data-stu-id="0bc8f-103">Run Windows VMs for an N-tier application</span></span>

<span data-ttu-id="0bc8f-104">此参考体系结构显示了一组经过实践检验的、运行用于 N 层应用程序的 Windows 虚拟机 (VM) 的做法。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-104">This reference architecture shows a set of proven practices for running Windows virtual machines (VMs) for an N-tier application.</span></span> [<span data-ttu-id="0bc8f-105">**部署此解决方案**。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-105">**Deploy this solution**.</span></span>](#deploy-the-solution) 

<span data-ttu-id="0bc8f-106">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="0bc8f-106">![[0]][0]</span></span>

<span data-ttu-id="0bc8f-107">*下载此体系结构的 [Visio 文件][visio-download]。*</span><span class="sxs-lookup"><span data-stu-id="0bc8f-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="0bc8f-108">体系结构</span><span class="sxs-lookup"><span data-stu-id="0bc8f-108">Architecture</span></span> 

<span data-ttu-id="0bc8f-109">有许多方法可用来实现 N 层体系结构。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-109">There are many ways to implement an N-tier architecture.</span></span> <span data-ttu-id="0bc8f-110">关系图中显示了一个典型的 3 层 Web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-110">The diagram shows a typical 3-tier web application.</span></span> <span data-ttu-id="0bc8f-111">此体系结构基于[运行负载均衡的 VM 以提高可伸缩性和可用性][multi-vm]中所述的内容。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-111">This architecture builds on [Run load-balanced VMs for scalability and availability][multi-vm].</span></span> <span data-ttu-id="0bc8f-112">Web 层和业务层都使用负载均衡的 VM。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-112">The web and business tiers use load-balanced VMs.</span></span>

* <span data-ttu-id="0bc8f-113">**可用性集。**</span><span class="sxs-lookup"><span data-stu-id="0bc8f-113">**Availability sets.**</span></span> <span data-ttu-id="0bc8f-114">为每个层创建一个[可用性集][azure-availability-sets]，并且在每个层中至少预配两个 VM。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-114">Create an [availability set][azure-availability-sets] for each tier, and provision at least two VMs in each tier.</span></span> <span data-ttu-id="0bc8f-115">这样，VM 便可以满足 VM 的更高[服务级别协议 (SLA)][vm-sla]。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-115">This makes the VMs eligible for a higher [service level agreement (SLA)][vm-sla] for VMs.</span></span> <span data-ttu-id="0bc8f-116">可在可用性集中部署单个 VM，但单个 VM 不具备 SLA 保证的资格，除非单个 VM 针对所有 OS 和数据磁盘使用 Azure 高级存储。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-116">You can deploy a single VM in an availability set, but the single VM will not qualify for an SLA guarantee unless the single VM is using Azure Premium Storage for all OS and data disks.</span></span>  
* <span data-ttu-id="0bc8f-117">**子网。**</span><span class="sxs-lookup"><span data-stu-id="0bc8f-117">**Subnets.**</span></span> <span data-ttu-id="0bc8f-118">为每个层创建一个单独的子网。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-118">Create a separate subnet for each tier.</span></span> <span data-ttu-id="0bc8f-119">使用 [CIDR] 表示法指定地址范围和子网掩码。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-119">Specify the address range and subnet mask using [CIDR] notation.</span></span> 
* <span data-ttu-id="0bc8f-120">**负载均衡器。**</span><span class="sxs-lookup"><span data-stu-id="0bc8f-120">**Load balancers.**</span></span> <span data-ttu-id="0bc8f-121">使用[面向 Internet 的负载均衡器][load-balancer-external]将传入的 Internet 流量分布到 Web 层，使用[内部负载均衡器][load-balancer-internal]将来自 Web 层的网络流量分布到业务层。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-121">Use an [Internet-facing load balancer][load-balancer-external] to distribute incoming Internet traffic to the web tier, and an [internal load balancer][load-balancer-internal] to distribute network traffic from the web tier to the business tier.</span></span>
* <span data-ttu-id="0bc8f-122">**Jumpbox。**</span><span class="sxs-lookup"><span data-stu-id="0bc8f-122">**Jumpbox.**</span></span> <span data-ttu-id="0bc8f-123">也称为[守护主机]。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-123">Also called a [bastion host].</span></span> <span data-ttu-id="0bc8f-124">网络上的一个安全 VM，管理员使用它来连接到其他 VM。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-124">A secure VM on the network that administrators use to connect to the other VMs.</span></span> <span data-ttu-id="0bc8f-125">Jumpbox 中的某个 NSG 只允许来自安全列表中的公共 IP 地址的远程流量。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-125">The jumpbox has an NSG that allows remote traffic only from public IP addresses on a safe list.</span></span> <span data-ttu-id="0bc8f-126">该 NSG 应允许远程桌面 (RDP) 流量。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-126">The NSG should permit remote desktop (RDP) traffic.</span></span>
* <span data-ttu-id="0bc8f-127">**监视。**</span><span class="sxs-lookup"><span data-stu-id="0bc8f-127">**Monitoring.**</span></span> <span data-ttu-id="0bc8f-128">可以使用 [Nagios]、[Zabbix] 或 [Icinga] 等监视软件深入了解响应时间、VM 运行时间和系统的整体运行状况。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-128">Monitoring software such as [Nagios], [Zabbix], or [Icinga] can give you insight into response time, VM uptime, and the overall health of your system.</span></span> <span data-ttu-id="0bc8f-129">在置于单独的管理子网中的 VM 上安装监视软件。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-129">Install the monitoring software on a VM that's placed in a separate management subnet.</span></span>
* <span data-ttu-id="0bc8f-130">**NSG。**</span><span class="sxs-lookup"><span data-stu-id="0bc8f-130">**NSGs.**</span></span> <span data-ttu-id="0bc8f-131">使用[网络安全组][nsg] (NSG) 来限制 VNet 中的网络流量。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-131">Use [network security groups][nsg] (NSGs) to restrict network traffic within the VNet.</span></span> <span data-ttu-id="0bc8f-132">例如，在此处显示的 3 层体系结构中，数据库层不接受来自 Web 前端的流量，仅接受来自业务层和管理子网的流量。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-132">For example, in the 3-tier architecture shown here, the database tier does not accept traffic from the web front end, only from the business tier and the management subnet.</span></span>
* <span data-ttu-id="0bc8f-133">**SQL Server Always On 可用性组。**</span><span class="sxs-lookup"><span data-stu-id="0bc8f-133">**SQL Server Always On Availability Group.**</span></span> <span data-ttu-id="0bc8f-134">通过启用复制和故障转移，在数据层提供高可用性。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-134">Provides high availability at the data tier, by enabling replication and failover.</span></span>
* <span data-ttu-id="0bc8f-135">**Active Directory 域服务 (AD DS) 服务器**。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-135">**Active Directory Domain Services (AD DS) Servers**.</span></span> <span data-ttu-id="0bc8f-136">在 Windows Server 2016 之前，SQL Server Always On 可用性组必须加入到域中。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-136">Prior to Windows Server 2016, SQL Server Always On Availability Groups must be joined to a domain.</span></span> <span data-ttu-id="0bc8f-137">这是因为可用性组依赖于 Windows Server 故障转移群集 (WSFC) 技术。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-137">This is because Availability Groups depend on Windows Server Failover Cluster (WSFC) technology.</span></span> <span data-ttu-id="0bc8f-138">Windows Server 2016 引入了在没有 Active Directory 的情况下创建故障转移群集的能力，在这种情况下，AD DS 服务器不是此体系结构所必需的。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-138">Windows Server 2016 introduces the ability to create a Failover Cluster without Active Directory, in which case the AD DS servers are not required for this architecture.</span></span> <span data-ttu-id="0bc8f-139">有关详细信息，请参阅 [What's new in Failover Clustering in Windows Server 2016][wsfc-whats-new]（Windows Server 2016 中的故障转移群集的新增功能）。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-139">For more information, see [What's new in Failover Clustering in Windows Server 2016][wsfc-whats-new].</span></span>
* <span data-ttu-id="0bc8f-140">**Azure DNS**。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-140">**Azure DNS**.</span></span> <span data-ttu-id="0bc8f-141">[Azure DNS][azure-dns] 是 DNS 域的托管服务，它使用 Microsoft Azure 基础结构提供名称解析。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-141">[Azure DNS][azure-dns] is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure.</span></span> <span data-ttu-id="0bc8f-142">通过在 Azure 中托管域，可以使用与其他 Azure 服务相同的凭据、API、工具和计费来管理 DNS 记录。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-142">By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools, and billing as your other Azure services.</span></span>

## <a name="recommendations"></a><span data-ttu-id="0bc8f-143">建议</span><span class="sxs-lookup"><span data-stu-id="0bc8f-143">Recommendations</span></span>

<span data-ttu-id="0bc8f-144">你的要求可能不同于此处描述的体系结构。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-144">Your requirements might differ from the architecture described here.</span></span> <span data-ttu-id="0bc8f-145">请使用以下建议作为入手点。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-145">Use these recommendations as a starting point.</span></span> 

### <a name="vnet--subnets"></a><span data-ttu-id="0bc8f-146">VNet/子网</span><span class="sxs-lookup"><span data-stu-id="0bc8f-146">VNet / Subnets</span></span>

<span data-ttu-id="0bc8f-147">在创建 VNet 时，确定每个子网中的资源需要多少 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-147">When you create the VNet, determine how many IP addresses your resources in each subnet require.</span></span> <span data-ttu-id="0bc8f-148">使用 [CIDR] 表示法为所需的 IP 地址指定子网掩码和足够大的 VNet 地址范围。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-148">Specify a subnet mask and a VNet address range large enough for the required IP addresses, using [CIDR] notation.</span></span> <span data-ttu-id="0bc8f-149">使用标准[专用 IP 地址块][private-ip-space]内的一个地址空间，这些地址块为 10.0.0.0/8、172.16.0.0/12 和 192.168.0.0/16。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-149">Use an address space that falls within the standard [private IP address blocks][private-ip-space], which are 10.0.0.0/8, 172.16.0.0/12, and 192.168.0.0/16.</span></span>

<span data-ttu-id="0bc8f-150">如果以后需要在 VNet 与本地网络之间设置一个网关，请选择一个不与你的本地网络重叠的地址范围。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-150">Choose an address range that does not overlap with your on-premises network, in case you need to set up a gateway between the VNet and your on-premise network later.</span></span> <span data-ttu-id="0bc8f-151">在创建 VNet 后，将无法更改地址范围。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-151">Once you create the VNet, you can't change the address range.</span></span>

<span data-ttu-id="0bc8f-152">在设计子网时一定要牢记功能和安全要求。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-152">Design subnets with functionality and security requirements in mind.</span></span> <span data-ttu-id="0bc8f-153">同一层或同一角色中的所有 VM 应当置于同一子网，这可能是一个安全边界。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-153">All VMs within the same tier or role should go into the same subnet, which can be a security boundary.</span></span> <span data-ttu-id="0bc8f-154">有关设计 VNet 和子网的详细信息，请参阅[规划和设计 Azure 虚拟网络][plan-network]。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-154">For more information about designing VNets and subnets, see [Plan and design Azure Virtual Networks][plan-network].</span></span>

<span data-ttu-id="0bc8f-155">针对每个子网，采用 CIDR 表示法指定子网的地址空间。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-155">For each subnet, specify the address space for the subnet in CIDR notation.</span></span> <span data-ttu-id="0bc8f-156">例如，“10.0.0.0/24”将创建包含 256 个 IP 地址的范围。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-156">For example, '10.0.0.0/24' creates a range of 256 IP addresses.</span></span> <span data-ttu-id="0bc8f-157">VM 可以使用其中的 251 个；5 个会被保留。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-157">VMs can use 251 of these; five are reserved.</span></span> <span data-ttu-id="0bc8f-158">请确保地址范围在子网之间不重叠。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-158">Make sure the address ranges don't overlap across subnets.</span></span> <span data-ttu-id="0bc8f-159">请参阅[虚拟网络常见问题解答][vnet faq]。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-159">See the [Virtual Network FAQ][vnet faq].</span></span>

### <a name="network-security-groups"></a><span data-ttu-id="0bc8f-160">网络安全组</span><span class="sxs-lookup"><span data-stu-id="0bc8f-160">Network security groups</span></span>

<span data-ttu-id="0bc8f-161">使用 NSG 规则限制各个层之间的流量。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-161">Use NSG rules to restrict traffic between tiers.</span></span> <span data-ttu-id="0bc8f-162">例如，在上面显示的 3 层体系结构中，Web 层不直接与数据库层进行通信。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-162">For example, in the 3-tier architecture shown above, the web tier does not communicate directly with the database tier.</span></span> <span data-ttu-id="0bc8f-163">为强制实现此目的，数据库层应当阻止来自 Web 层子网的传入流量。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-163">To enforce this, the database tier should block incoming traffic from the web tier subnet.</span></span>  

1. <span data-ttu-id="0bc8f-164">创建一个 NSG 并将其关联到数据库层子网。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-164">Create an NSG and associate it to the database tier subnet.</span></span>
2. <span data-ttu-id="0bc8f-165">添加一个拒绝来自 VNet 的所有入站流量的规则。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-165">Add a rule that denies all inbound traffic from the VNet.</span></span> <span data-ttu-id="0bc8f-166">（在规则中使用 `VIRTUAL_NETWORK` 标记。）</span><span class="sxs-lookup"><span data-stu-id="0bc8f-166">(Use the `VIRTUAL_NETWORK` tag in the rule.)</span></span> 
3. <span data-ttu-id="0bc8f-167">添加一个具有更高优先级的规则，用以允许来自业务层子网的入站流量。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-167">Add a rule with a higher priority that allows inbound traffic from the business tier subnet.</span></span> <span data-ttu-id="0bc8f-168">此规则将替代前面的规则，并允许业务层与数据库层进行通信。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-168">This rule overrides the previous rule, and allows the business tier to talk to the database tier.</span></span>
4. <span data-ttu-id="0bc8f-169">添加一个允许来自数据库层子网自身的入站流量的规则。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-169">Add a rule that allows inbound traffic from within the database tier subnet itself.</span></span> <span data-ttu-id="0bc8f-170">此规则允许在数据库层中的各个 VM 之间进行通信，这是进行数据库复制和故障转移所必需的。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-170">This rule allows communication between VMs in the database tier, which is needed for database replication and failover.</span></span>
5. <span data-ttu-id="0bc8f-171">添加一个允许来自 jumpbox 子网的 RDP 流量的规则。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-171">Add a rule that allows RDP traffic from the jumpbox subnet.</span></span> <span data-ttu-id="0bc8f-172">此规则允许管理员从 jumpbox 连接到数据库层。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-172">This rule lets administrators connect to the database tier from the jumpbox.</span></span>
   
   > [!NOTE]
   > <span data-ttu-id="0bc8f-173">NSG 包含允许来自 VNet 中的任何入站流量的默认规则。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-173">An NSG has default rules that allow any inbound traffic from within the VNet.</span></span> <span data-ttu-id="0bc8f-174">这些规则无法删除，但可以通过创建优先级更高的规则来替代它们。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-174">These rules can't be deleted, but you can override them by creating higher priority rules.</span></span>
   > 
   > 

### <a name="load-balancers"></a><span data-ttu-id="0bc8f-175">负载均衡器</span><span class="sxs-lookup"><span data-stu-id="0bc8f-175">Load balancers</span></span>

<span data-ttu-id="0bc8f-176">外部负载均衡器将 Internet 流量分布到 Web 层。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-176">The external load balancer distributes Internet traffic to the web tier.</span></span> <span data-ttu-id="0bc8f-177">为此负载均衡器创建一个公共 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-177">Create a public IP address for this load balancer.</span></span> <span data-ttu-id="0bc8f-178">请参阅[创建面向 Internet 的负载均衡器][lb-external-create]。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-178">See [Creating an Internet-facing load balancer][lb-external-create].</span></span>

<span data-ttu-id="0bc8f-179">内部负载均衡器将网络流量从 Web 层分布到业务层。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-179">The internal load balancer distributes network traffic from the web tier to the business tier.</span></span> <span data-ttu-id="0bc8f-180">若要为此负载均衡器提供专用 IP 地址，请创建一个前端 IP 配置，并将其与业务层的子网相关联。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-180">To give this load balancer a private IP address, create a frontend IP configuration and associate it with the subnet for the business tier.</span></span> <span data-ttu-id="0bc8f-181">请参阅[开始创建内部负载均衡器][lb-internal-create]。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-181">See [Get started creating an Internal load balancer][lb-internal-create].</span></span>

### <a name="sql-server-always-on-availability-groups"></a><span data-ttu-id="0bc8f-182">SQL Server Always On 可用性组</span><span class="sxs-lookup"><span data-stu-id="0bc8f-182">SQL Server Always On Availability Groups</span></span>

<span data-ttu-id="0bc8f-183">建议使用 [Always On 可用性组][sql-alwayson]以实现高可用性。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-183">We recommend [Always On Availability Groups][sql-alwayson] for SQL Server high availability.</span></span> <span data-ttu-id="0bc8f-184">在 Windows Server 2016 之前，Always On 可用性组需要一个域控制器，并且可用性组中的所有节点必须在同一 AD 域中。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-184">Prior to Windows Server 2016, Always On Availability Groups require a domain controller, and all nodes in the availability group must be in the same AD domain.</span></span>

<span data-ttu-id="0bc8f-185">其他层通过[可用性组侦听程序][sql-alwayson-listeners]连接到数据库。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-185">Other tiers connect to the database through an [availability group listener][sql-alwayson-listeners].</span></span> <span data-ttu-id="0bc8f-186">该侦听程序使得 SQL 客户端能够在不知道 SQL Server 物理实例名称的情况下进行连接。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-186">The listener enables a SQL client to connect without knowing the name of the physical instance of SQL Server.</span></span> <span data-ttu-id="0bc8f-187">访问数据库的 VM 必须加入域。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-187">VMs that access the database must be joined to the domain.</span></span> <span data-ttu-id="0bc8f-188">客户端（在本例中为另一个层）使用 DNS 将该侦听程序的虚拟网络名称解析为 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-188">The client (in this case, another tier) uses DNS to resolve the listener's virtual network name into IP addresses.</span></span>

<span data-ttu-id="0bc8f-189">如下所述配置 SQL Server Always On 可用性组：</span><span class="sxs-lookup"><span data-stu-id="0bc8f-189">Configure the SQL Server Always On Availability Group as follows:</span></span>

1. <span data-ttu-id="0bc8f-190">创建一个 Windows Server 故障转移群集 (WSFC) 群集、一个 SQL Server Always On 可用性组和一个主要副本。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-190">Create a Windows Server Failover Clustering (WSFC) cluster, a SQL Server Always On Availability Group, and a primary replica.</span></span> <span data-ttu-id="0bc8f-191">有关详细信息，请参阅 [Always On 可用性组入门 (SQL Server)][sql-alwayson-getting-started]。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-191">For more information, see [Getting Started with Always On Availability Groups][sql-alwayson-getting-started].</span></span> 
2. <span data-ttu-id="0bc8f-192">创建一个具有静态专用 IP 地址的内部负载均衡器。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-192">Create an internal load balancer with a static private IP address.</span></span>
3. <span data-ttu-id="0bc8f-193">创建一个可用性组侦听程序，并将该侦听程序的 DNS 名称映射到一个内部负载均衡器的 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-193">Create an availability group listener, and map the listener's DNS name to the IP address of an internal load balancer.</span></span> 
4. <span data-ttu-id="0bc8f-194">为 SQL Server 侦听端口（默认情况下为 TCP 端口 1433）创建一个负载均衡器规则。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-194">Create a load balancer rule for the SQL Server listening port (TCP port 1433 by default).</span></span> <span data-ttu-id="0bc8f-195">该负载均衡器规则必须启用*浮动 IP*，也称为“直接服务器返回”。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-195">The load balancer rule must enable *floating IP*, also called Direct Server Return.</span></span> <span data-ttu-id="0bc8f-196">这将导致 VM 直接回复客户端，从而实现到主要副本的直接连接。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-196">This causes the VM to reply directly to the client, which enables a direct connection to the primary replica.</span></span>
  
  > [!NOTE]
  > <span data-ttu-id="0bc8f-197">当启用了浮动 IP 时，前端端口号必须与负载均衡器规则中的后端端口号相同。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-197">When floating IP is enabled, the front-end port number must be the same as the back-end port number in the load balancer rule.</span></span>
  > 
  > 

<span data-ttu-id="0bc8f-198">当 SQL 客户端尝试连接时，负载均衡器会将连接请求路由到主要副本。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-198">When a SQL client tries to connect, the load balancer routes the connection request to the primary replica.</span></span> <span data-ttu-id="0bc8f-199">如果发生到其他副本的故障转移，则负载均衡器会自动将后续请求路由到新的主要副本。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-199">If there is a failover to another replica, the load balancer automatically routes subsequent requests to a new primary replica.</span></span> <span data-ttu-id="0bc8f-200">有关详细信息，请参阅[Configure an ILB listener for SQL Server Always On Availability Groups][sql-alwayson-ilb]（为 SQL Server Always On 可用性组配置 ILB 侦听程序）。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-200">For more information, see [Configure an ILB listener for SQL Server Always On Availability Groups][sql-alwayson-ilb].</span></span>

<span data-ttu-id="0bc8f-201">在故障转移期间，现有的客户端连接将关闭。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-201">During a failover, existing client connections are closed.</span></span> <span data-ttu-id="0bc8f-202">在故障转移完成后，新连接将被路由到新的主要副本。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-202">After the failover completes, new connections will be routed to the new primary replica.</span></span>

<span data-ttu-id="0bc8f-203">如果应用程序执行的读取操作显著多于写入操作，则可以将一些只读查询转移到次要副本。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-203">If your application makes significantly more reads than writes, you can offload some of the read-only queries to a secondary replica.</span></span> <span data-ttu-id="0bc8f-204">请参阅[Using a Listener to Connect to a Read-Only Secondary Replica (Read-Only Routing)][sql-alwayson-read-only-routing]（使用侦听程序连接到只读次要副本（只读路由））。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-204">See [Using a Listener to Connect to a Read-Only Secondary Replica (Read-Only Routing)][sql-alwayson-read-only-routing].</span></span>

<span data-ttu-id="0bc8f-205">通过执行可用性组的[强制手动故障转移][sql-alwayson-force-failover]来测试部署。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-205">Test your deployment by [forcing a manual failover][sql-alwayson-force-failover] of the availability group.</span></span>

### <a name="jumpbox"></a><span data-ttu-id="0bc8f-206">Jumpbox</span><span class="sxs-lookup"><span data-stu-id="0bc8f-206">Jumpbox</span></span>

<span data-ttu-id="0bc8f-207">Jumpbox 的性能要求非常低，因此请为 jumpbox 选择一个较小的 VM 大小，例如标准 A1。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-207">The jumpbox will have minimal performance requirements, so select a small VM size for the jumpbox such as Standard A1.</span></span> 

<span data-ttu-id="0bc8f-208">为 jumpbox 创建一个[公共 IP 地址]。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-208">Create a [public IP address] for the jumpbox.</span></span> <span data-ttu-id="0bc8f-209">将 jumpbox 放置在与其他 VM 相同的 VNet 中，但将其置于一个单独的管理子网中。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-209">Place the jumpbox in the same VNet as the other VMs, but in a separate management subnet.</span></span>

<span data-ttu-id="0bc8f-210">不要允许通过公共 Internet 对运行应用程序工作负荷的 VM 进行 RDP 访问。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-210">Do not allow RDP access from the public Internet to the VMs that run the application workload.</span></span> <span data-ttu-id="0bc8f-211">相反，对这些 VM 的所有 RDP 访问都必须通过 jumpbox 进行。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-211">Instead, all RDP access to these VMs must come through the jumpbox.</span></span> <span data-ttu-id="0bc8f-212">管理员登录到 jumpbox，然后从 jumpbox 登录到其他 VM。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-212">An administrator logs into the jumpbox, and then logs into the other VM from the jumpbox.</span></span> <span data-ttu-id="0bc8f-213">Jumpbox 允许来自 Internet 的 RDP 流量，但仅允许来自已知的安全 IP 地址的流量。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-213">The jumpbox allows RDP traffic from the Internet, but only from known, safe IP addresses.</span></span>

<span data-ttu-id="0bc8f-214">若要保护 jumpbox，请创建一个 NSG 并将其应用于 jumpbox 子网。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-214">To secure the jumpbox, create an NSG and apply it to the jumpbox subnet.</span></span> <span data-ttu-id="0bc8f-215">添加一个 NSG 规则，使其仅允许来自一组安全的公共 IP 地址的 RDP 连接。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-215">Add an NSG rule that allows RDP connections only from a safe set of public IP addresses.</span></span> <span data-ttu-id="0bc8f-216">可以将 NSG 附加到子网或 jumpbox NIC。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-216">The NSG can be attached either to the subnet or to the jumpbox NIC.</span></span> <span data-ttu-id="0bc8f-217">在本例中，我们建议将其附加到 NIC，以便仅允许将 RDP 流量发送到 jumpbox，即使你将其他 VM 添加到了同一子网。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-217">In this case, we recommend attaching it to the NIC, so RDP traffic is permitted only to the jumpbox, even if you add other VMs to the same subnet.</span></span>

<span data-ttu-id="0bc8f-218">为其他子网配置 NSG 以允许来自管理子网的 RDP 流量。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-218">Configure the NSGs for the other subnets to allow RDP traffic from the management subnet.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="0bc8f-219">可用性注意事项</span><span class="sxs-lookup"><span data-stu-id="0bc8f-219">Availability considerations</span></span>

<span data-ttu-id="0bc8f-220">在数据库层，具有多个 VM 不会自动转换为高度可用的数据库。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-220">At the database tier, having multiple VMs does not automatically translate into a highly available database.</span></span> <span data-ttu-id="0bc8f-221">对于关系数据库，通常需要使用复制和故障转移来实现高可用性。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-221">For a relational database, you will typically need to use replication and failover to achieve high availability.</span></span> <span data-ttu-id="0bc8f-222">对于 SQL Server，建议使用 [Always On 可用性组][sql-alwayson]。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-222">For SQL Server, we recommend using [Always On Availability Groups][sql-alwayson].</span></span> 

<span data-ttu-id="0bc8f-223">如果需要的可用性高于 [VM 的 Azure SLA][vm-sla] 提供的可用性，请跨两个区域复制应用程序，并使用 Azure 流量管理器进行故障转移。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-223">If you need higher availability than the [Azure SLA for VMs][vm-sla] provides, replicate the application across two regions and use Azure Traffic Manager for failover.</span></span> <span data-ttu-id="0bc8f-224">有关详细信息，请参阅[在多个区域中运行 Windows VM 以实现高可用性][multi-dc]。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-224">For more information, see [Run Windows VMs in multiple regions for high availability][multi-dc].</span></span>   

## <a name="security-considerations"></a><span data-ttu-id="0bc8f-225">安全注意事项</span><span class="sxs-lookup"><span data-stu-id="0bc8f-225">Security considerations</span></span>

<span data-ttu-id="0bc8f-226">加密静态的敏感数据并使用 [Azure Key Vault][azure-key-vault] 管理数据库加密密钥。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-226">Encrypt sensitive data at rest and use [Azure Key Vault][azure-key-vault] to manage the database encryption keys.</span></span> <span data-ttu-id="0bc8f-227">Key Vault 可以将加密密钥存储在硬件安全模块 (HSM) 中。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-227">Key Vault can store encryption keys in hardware security modules (HSMs).</span></span> <span data-ttu-id="0bc8f-228">有关详细信息，请参阅 [Configure Azure Key Vault Integration for SQL Server on Azure VMs][sql-keyvault]（为 Azure VM 上的 SQL Server 配置 Azure Key Vault 集成）。另外，建议将应用程序机密（例如数据库连接字符串）也存储在 Key Vault 中。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-228">For more information, see [Configure Azure Key Vault Integration for SQL Server on Azure VMs][sql-keyvault] It's also recommended to store application secrets, such as database connection strings, in Key Vault.</span></span>

<span data-ttu-id="0bc8f-229">请考虑添加一个网络虚拟设备 (NVA) 以在 Internet 与 Azure 虚拟网络之间创建一个外围网络。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-229">Consider adding a network virtual appliance (NVA) to create a DMZ between the Internet and the Azure virtual network.</span></span> <span data-ttu-id="0bc8f-230">NVA 是虚拟设备的一个通用术语，可以执行与网络相关的任务，例如防火墙、包检查、审核和自定义路由。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-230">NVA is a generic term for a virtual appliance that can perform network-related tasks, such as firewall, packet inspection, auditing, and custom routing.</span></span> <span data-ttu-id="0bc8f-231">有关详细信息，请参阅[在 Azure 与 Internet 之间实现外围网络][dmz]。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-231">For more information, see [Implementing a DMZ between Azure and the Internet][dmz].</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="0bc8f-232">可伸缩性注意事项</span><span class="sxs-lookup"><span data-stu-id="0bc8f-232">Scalability considerations</span></span>

<span data-ttu-id="0bc8f-233">负载均衡器将网络流量分布到 Web 层和业务层。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-233">The load balancers distribute network traffic to the web and business tiers.</span></span> <span data-ttu-id="0bc8f-234">可以通过添加新的 VM 实例进行水平缩放。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-234">Scale horizontally by adding new VM instances.</span></span> <span data-ttu-id="0bc8f-235">请注意，可以根据负载分别对 Web 层和业务层进行缩放。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-235">Note that you can scale the web and business tiers independently, based on load.</span></span> <span data-ttu-id="0bc8f-236">为减少由于需要维护客户端相关性而可能导致的复杂情况，Web 层中的 VM 应当是无状态的。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-236">To reduce possible complications caused by the need to maintain client affinity, the VMs in the web tier should be stateless.</span></span> <span data-ttu-id="0bc8f-237">托管着业务逻辑的 VM 也应当是无状态的。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-237">The VMs hosting the business logic should also be stateless.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="0bc8f-238">可管理性注意事项</span><span class="sxs-lookup"><span data-stu-id="0bc8f-238">Manageability considerations</span></span>

<span data-ttu-id="0bc8f-239">可以通过使用 [Azure 自动化][azure-administration]、[Microsoft Operations Management Suite][operations-management-suite]、[Chef][chef] 或 [Puppet][puppet] 等集中式管理工具来简化整个系统的管理。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-239">Simplify management of the entire system by using centralized administration tools such as [Azure Automation][azure-administration], [Microsoft Operations Management Suite][operations-management-suite], [Chef][chef], or [Puppet][puppet].</span></span> <span data-ttu-id="0bc8f-240">这些工具可以整合从多个 VM 捕获的诊断和运行状况信息，使用户可以全面了解该系统。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-240">These tools can consolidate diagnostic and health information captured from multiple VMs to provide an overall view of the system.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="0bc8f-241">部署解决方案</span><span class="sxs-lookup"><span data-stu-id="0bc8f-241">Deploy the solution</span></span>

<span data-ttu-id="0bc8f-242">[GitHub][github-folder] 中提供了此参考体系结构的部署。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-242">A deployment for this reference architecture is available on [GitHub][github-folder].</span></span> 

### <a name="prerequisites"></a><span data-ttu-id="0bc8f-243">系统必备</span><span class="sxs-lookup"><span data-stu-id="0bc8f-243">Prerequisites</span></span>

<span data-ttu-id="0bc8f-244">在将参考体系结构部署到自己的订阅之前，必须执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-244">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="0bc8f-245">为 [AzureCAT 参考体系结构][ref-arch-repo] GitHub 存储库克隆、下载 zip 文件或创建其分支。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-245">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="0bc8f-246">确保在计算机上安装了 Azure CLI 2.0。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-246">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="0bc8f-247">若要安装 CLI，请按照[安装 Azure CLI 2.0][azure-cli-2] 中的说明执行操作。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-247">To install the CLI, follow the instructions in [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="0bc8f-248">安装 [Azure 构建基块][azbb] npm 包。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-248">Install the [Azure building blocks][azbb] npm package.</span></span>

  ```bash
  npm install -g @mspnp/azure-building-blocks
  ```

4. <span data-ttu-id="0bc8f-249">从命令提示符、bash 提示符或 PowerShell 提示符下通过使用以下命令之一，登录到 Azure 帐户，然后按照提示进行操作。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-249">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using one of the commands below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a><span data-ttu-id="0bc8f-250">使用 azbb 部署解决方案</span><span class="sxs-lookup"><span data-stu-id="0bc8f-250">Deploy the solution using azbb</span></span>

<span data-ttu-id="0bc8f-251">若要为 N 层应用程序参考体系结构部署 Windows VM，请按照以下步骤操作：</span><span class="sxs-lookup"><span data-stu-id="0bc8f-251">To deploy the Windows VMs for an N-tier application reference architecture, follow these steps:</span></span>

1. <span data-ttu-id="0bc8f-252">导航到在以上前决条件的第 1 步中克隆的存储库的 `virtual-machines\n-tier-windows` 文件夹。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-252">Navigate to the `virtual-machines\n-tier-windows` folder for the repository you cloned in step 1 of the pre-requisites above.</span></span>

2. <span data-ttu-id="0bc8f-253">参数文件为部署中的每个 VM 指定默认管理员用户名称和密码。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-253">The parameter file specifies a default adminstrator user name and password for each VM in the deployment.</span></span> <span data-ttu-id="0bc8f-254">必须在部署参考体系结构前，对其进行更改。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-254">You must change these before you deploy the reference architecture.</span></span> <span data-ttu-id="0bc8f-255">打开 `n-tier-windows.json` 文件，然后将每个“adminUsername”和“adminPassword”字段替换为新设置。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-255">Open the `n-tier-windows.json` file and replace each **adminUsername** and **adminPassword** field with your new settings.</span></span>
  
  > [!NOTE]
  > <span data-ttu-id="0bc8f-256">此部署期间，多个脚本在一些 VirtualMachine 对象的 VirtualMachineExtension 对象和 extensions 设置中运行。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-256">There are multiple scripts that run during this deployment both in the  **VirtualMachineExtension** objects and in the **extensions** settings for some of the **VirtualMachine** objects.</span></span> <span data-ttu-id="0bc8f-257">这些脚本中的一些需要你刚更改的管理员用户名和密码。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-257">Some of these scripts require the administrator user name and password that you have just changed.</span></span> <span data-ttu-id="0bc8f-258">建议查看这些脚本，确保指定正确的凭据。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-258">It's recommended that you review these scripts to ensure that you specified the correct credentials.</span></span> <span data-ttu-id="0bc8f-259">如果未指定正确的凭据，部署可能失败。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-259">The deployment may fail if you have not specified the correct credentials.</span></span>
  > 
  > 

<span data-ttu-id="0bc8f-260">保存文件。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-260">Save the file.</span></span>

3. <span data-ttu-id="0bc8f-261">使用 azbb 命令行工具部署参考体系结构，如下所示。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-261">Deploy the reference architecture using the **azbb** command line tool as shown below.</span></span>

  ```bash
  azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-windows.json --deploy
  ```

<span data-ttu-id="0bc8f-262">若要详细了解如何使用 Azure 构建基块部署此示例参考体系结构，请访问 [GitHub 存储库][git]。</span><span class="sxs-lookup"><span data-stu-id="0bc8f-262">For more information on deploying this sample reference architecture using Azure Building Blocks, visit the [GitHub repository][git].</span></span>


<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-dc]: multi-region-application.md
[multi-vm]: multi-vm.md
[n-tier]: n-tier.md
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-windows-manage-availability#configure-each-application-tier-into-separate-availability-sets
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault
[守护主机]: https://en.wikipedia.org/wiki/Bastion_host
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-windows
[lb-external-create]: /azure/load-balancer/load-balancer-get-started-internet-portal
[lb-internal-create]: /azure/load-balancer/load-balancer-get-started-ilb-arm-portal
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[公共 IP 地址]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[sql-alwayson]: https://msdn.microsoft.com/library/hh510230.aspx
[sql-alwayson-force-failover]: https://msdn.microsoft.com/library/ff877957.aspx
[sql-alwayson-getting-started]: https://msdn.microsoft.com/library/gg509118.aspx
[sql-alwayson-ilb]: /azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-alwayson-int-listener
[sql-alwayson-listeners]: https://msdn.microsoft.com/library/hh213417.aspx
[sql-alwayson-read-only-routing]: https://technet.microsoft.com/library/hh213417.aspx#ConnectToSecondary
[sql-keyvault]: /azure/virtual-machines/virtual-machines-windows-ps-sql-keyvault
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[wsfc-whats-new]: https://technet.microsoft.com/windows-server-docs/failover-clustering/whats-new-in-failover-clustering
[Nagios]: https://www.nagios.org/
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[0]: ./images/n-tier-diagram.png "使用 Microsoft Azure 的 N 层体系结构"