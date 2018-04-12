---
title: 将 Active Directory 域服务 (AD DS) 扩展到 Azure
description: >-
  如何在 Azure 中实现采用 Active Directory 授权的安全的混合网络体系结构。

  指南,vpn 网关,ExpressRoute,负载均衡器,虚拟网络,active-directory
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Identity management
pnp.series.prev: azure-ad
pnp.series.next: adds-forest
ms.openlocfilehash: 007d244f29bf11c6e2bd703c7f4f245d22c02f0f
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/30/2018
---
# <a name="extend-active-directory-domain-services-ad-ds-to-azure"></a><span data-ttu-id="6d0a2-104">将 Active Directory 域服务 (AD DS) 扩展到 Azure</span><span class="sxs-lookup"><span data-stu-id="6d0a2-104">Extend Active Directory Domain Services (AD DS) to Azure</span></span>

<span data-ttu-id="6d0a2-105">此参考体系结构展示了如何使用 [Active Directory 域服务 (AD DS)][active-directory-domain-services] 将 Active Directory 环境扩展到 Azure 以提供分布式身份验证服务。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-105">This reference architecture shows how to extend your Active Directory environment to Azure to provide distributed authentication services using [Active Directory Domain Services (AD DS)][active-directory-domain-services].</span></span>  [<span data-ttu-id="6d0a2-106">**部署此解决方案**。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-106">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="6d0a2-107">[![0]][0]</span><span class="sxs-lookup"><span data-stu-id="6d0a2-107">[![0]][0]</span></span> 

<span data-ttu-id="6d0a2-108">下载此体系结构的 [Visio 文件][visio-download]。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-108">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="6d0a2-109">AD DS 用来对安全域中包括的用户、计算机、应用程序或其他标识进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-109">AD DS is used to authenticate user, computer, application, or other identities that are included in a security domain.</span></span> <span data-ttu-id="6d0a2-110">它可以托管在本地，但是如果你的应用程序部分托管在本地部分托管在 Azure 中，则将此功能复制到 Azure 中可能更为高效。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-110">It can be hosted on-premises, but if your application is hosted partly on-premises and partly in Azure, it may be more efficient to replicate this functionality in Azure.</span></span> <span data-ttu-id="6d0a2-111">这可以降低由于将身份验证和本地授权请求从云发送回在本地运行的 AD DS 而导致的延迟。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-111">This can reduce the latency caused by sending authentication and local authorization requests from the cloud back to AD DS running on-premises.</span></span> 

<span data-ttu-id="6d0a2-112">如果本地网络和 Azure 虚拟网络通过 VPN 或 ExpressRoute 连接进行连接，通常会使用此体系结构。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-112">This architecture is commonly used when the on-premises network and the Azure virtual network are connected by a VPN or ExpressRoute connection.</span></span> <span data-ttu-id="6d0a2-113">此体系结构还支持双向复制，这意味着既可以在本地又可以在云中进行更改，并且两个源都将保持一致。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-113">This architecture also supports bidirectional replication, meaning changes can be made either on-premises or in the cloud, and both sources will be kept consistent.</span></span> <span data-ttu-id="6d0a2-114">此体系结构的典型用途包括功能分布在本地和 Azure 中的混合应用程序以及使用 Active Directory 执行身份验证的应用程序和服务。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-114">Typical uses for this architecture include hybrid applications in which functionality is distributed between on-premises and Azure, and applications and services that perform authentication using Active Directory.</span></span>

<span data-ttu-id="6d0a2-115">有关其他注意事项，请参阅[选择用于将本地 Active Directory 与 Azure 相集成的解决方案][considerations]。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-115">For additional considerations, see [Choose a solution for integrating on-premises Active Directory with Azure][considerations].</span></span> 

## <a name="architecture"></a><span data-ttu-id="6d0a2-116">体系结构</span><span class="sxs-lookup"><span data-stu-id="6d0a2-116">Architecture</span></span> 

<span data-ttu-id="6d0a2-117">此体系结构扩展了 [Azure 与 Internet 之间的外围网络][implementing-a-secure-hybrid-network-architecture-with-internet-access]中显示的体系结构。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-117">This architecture extends the architecture shown in [DMZ between Azure and the Internet][implementing-a-secure-hybrid-network-architecture-with-internet-access].</span></span> <span data-ttu-id="6d0a2-118">它包括以下组件。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-118">It has the following components.</span></span>

* <span data-ttu-id="6d0a2-119">**本地网络**。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-119">**On-premises network**.</span></span> <span data-ttu-id="6d0a2-120">本地网络包括可以对位于本地的组件执行身份验证和授权的本地 Active Directory 服务器。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-120">The on-premises network includes local Active Directory servers that can perform authentication and authorization for components located on-premises.</span></span>
* <span data-ttu-id="6d0a2-121">**Active Directory 服务器**。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-121">**Active Directory servers**.</span></span> <span data-ttu-id="6d0a2-122">它们是域控制器，用于实现在云中作为 VM 运行的目录服务 (AD DS)。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-122">These are domain controllers implementing directory services (AD DS) running as VMs in the cloud.</span></span> <span data-ttu-id="6d0a2-123">这些服务器可以为在 Azure 虚拟网络中运行的组件提供身份验证。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-123">These servers can provide authentication of components running in your Azure virtual network.</span></span>
* <span data-ttu-id="6d0a2-124">**Active Directory 子网**。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-124">**Active Directory subnet**.</span></span> <span data-ttu-id="6d0a2-125">AD DS 服务器托管在一个单独的子网中。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-125">The AD DS servers are hosted in a separate subnet.</span></span> <span data-ttu-id="6d0a2-126">网络安全组 (NSG) 规则对 AD DS 服务器进行保护，并提供防火墙以阻止来自意外来源的流量。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-126">Network security group (NSG) rules protect the AD DS servers and provide a firewall against traffic from unexpected sources.</span></span>
* <span data-ttu-id="6d0a2-127">**Azure 网关和 Active Directory 同步**。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-127">**Azure Gateway and Active Directory synchronization**.</span></span> <span data-ttu-id="6d0a2-128">Azure 网关在本地网络与 Azure VNet 之间提供连接。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-128">The Azure gateway provides a connection between the on-premises network and the Azure VNet.</span></span> <span data-ttu-id="6d0a2-129">这可以是 [VPN 连接][azure-vpn-gateway]，也可以是 [Azure ExpressRoute][azure-expressroute]。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-129">This can be a [VPN connection][azure-vpn-gateway] or [Azure ExpressRoute][azure-expressroute].</span></span> <span data-ttu-id="6d0a2-130">云中的 Active Directory 服务器与本地 Active Directory 服务器之间的所有同步请求都通过该网关。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-130">All synchronization requests between the Active Directory servers in the cloud and on-premises pass through the gateway.</span></span> <span data-ttu-id="6d0a2-131">用户定义的路由 (UDR) 处理传递到 Azure 的本地流量的路由。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-131">User-defined routes (UDRs) handle routing for on-premises traffic that passes to Azure.</span></span> <span data-ttu-id="6d0a2-132">传入到 Active Directory 服务器或从中传出的流量不通过此方案中使用的网络虚拟设备 (NVA)。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-132">Traffic to and from the Active Directory servers does not pass through the network virtual appliances (NVAs) used in this scenario.</span></span>

<span data-ttu-id="6d0a2-133">有关配置 UDR 和 NVA 的详细信息，请参阅[在 Azure 中实现安全的混合网络体系结构][implementing-a-secure-hybrid-network-architecture]。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-133">For more information about configuring UDRs and the NVAs, see [Implementing a secure hybrid network architecture in Azure][implementing-a-secure-hybrid-network-architecture].</span></span> 

## <a name="recommendations"></a><span data-ttu-id="6d0a2-134">建议</span><span class="sxs-lookup"><span data-stu-id="6d0a2-134">Recommendations</span></span>

<span data-ttu-id="6d0a2-135">以下建议适用于大多数方案。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-135">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="6d0a2-136">除非有优先于这些建议的特定要求，否则请遵循这些建议。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-136">Follow these recommendations unless you have a specific requirement that overrides them.</span></span> 

### <a name="vm-recommendations"></a><span data-ttu-id="6d0a2-137">VM 建议</span><span class="sxs-lookup"><span data-stu-id="6d0a2-137">VM recommendations</span></span>

<span data-ttu-id="6d0a2-138">根据预计的身份验证请求量决定 [VM 大小][vm-windows-sizes]。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-138">Determine your [VM size][vm-windows-sizes] requirements based on the expected volume of authentication requests.</span></span> <span data-ttu-id="6d0a2-139">使用在本地托管着 AD DS 的计算机的规范作为起点，并使其与 Azure VM 大小相匹配。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-139">Use the specifications of the machines hosting AD DS on premises as a starting point, and match them with the Azure VM sizes.</span></span> <span data-ttu-id="6d0a2-140">在部署后，监视利用率并根据 VM 上的实际负载纵向扩展或收缩。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-140">Once deployed, monitor utilization and scale up or down based on the actual load on the VMs.</span></span> <span data-ttu-id="6d0a2-141">有关确定 AD DS 域控制器大小的详细信息，请参阅 [Capacity Planning for Active Directory Domain Services][capacity-planning-for-adds]（Active Directory 域服务的容量规划）。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-141">For more information about sizing AD DS domain controllers, see [Capacity Planning for Active Directory Domain Services][capacity-planning-for-adds].</span></span>

<span data-ttu-id="6d0a2-142">创建一个单独的虚拟数据磁盘，用以存储 Active Directory 的数据库、日志和 SYSVOL。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-142">Create a separate virtual data disk for storing the database, logs, and SYSVOL for Active Directory.</span></span> <span data-ttu-id="6d0a2-143">不要将这些项与操作系统存储在同一磁盘上。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-143">Do not store these items on the same disk as the operating system.</span></span> <span data-ttu-id="6d0a2-144">注意，默认情况下，附加到 VM 的数据磁盘使用直写式缓存。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-144">Note that by default, data disks that are attached to a VM use write-through caching.</span></span> <span data-ttu-id="6d0a2-145">但是，这种形式的缓存可能会与 AD DS 的要求发生冲突。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-145">However, this form of caching can conflict with the requirements of AD DS.</span></span> <span data-ttu-id="6d0a2-146">因此，请将数据磁盘上的“主机缓存首选项”设置设为“无”。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-146">For this reason, set the *Host Cache Preference* setting on the data disk to *None*.</span></span> <span data-ttu-id="6d0a2-147">有关详细信息，请参阅[放置 Windows Server AD DS 数据库和 SYSVOL][adds-data-disks]。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-147">For more information, see [Placement of the Windows Server AD DS database and SYSVOL][adds-data-disks].</span></span>

<span data-ttu-id="6d0a2-148">部署至少两个运行 AD DS 作为域控制器的 VM，并将它们添加到[可用性集][availability-set]中。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-148">Deploy at least two VMs running AD DS as domain controllers and add them to an [availability set][availability-set].</span></span>

### <a name="networking-recommendations"></a><span data-ttu-id="6d0a2-149">网络建议</span><span class="sxs-lookup"><span data-stu-id="6d0a2-149">Networking recommendations</span></span>

<span data-ttu-id="6d0a2-150">为每个 AD DS 服务器的 VM 网络接口 (NIC) 配置一个静态专用 IP 地址，以提供完整的域名服务 (DNS) 支持。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-150">Configure the VM network interface (NIC) for each AD DS server with a static private IP address for full domain name service (DNS) support.</span></span> <span data-ttu-id="6d0a2-151">有关详细信息，请参阅[如何在 Azure 门户中设置静态专用 IP 地址][set-a-static-ip-address]。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-151">For more information, see [How to set a static private IP address in the Azure portal][set-a-static-ip-address].</span></span>

> [!NOTE]
> <span data-ttu-id="6d0a2-152">不要为任何 AD DS 的 VM NIC 配置公共 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-152">Do not configure the VM NIC for any AD DS with a public IP address.</span></span> <span data-ttu-id="6d0a2-153">有关更多详细信息，请参阅[安全注意事项][security-considerations]。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-153">See [Security considerations][security-considerations] for more details.</span></span>
> 
> 

<span data-ttu-id="6d0a2-154">Active Directory 子网 NSG 要求规则允许来自本地的传入流量。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-154">The Active Directory subnet NSG requires rules to permit incoming traffic from on-premises.</span></span> <span data-ttu-id="6d0a2-155">有关 AD DS 使用的端口的详细信息，请参阅 [Active Directory and Active Directory Domain Services Port Requirements][ad-ds-ports]（Active Directory 和 Active Directory 域服务端口要求）。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-155">For detailed information on the ports used by AD DS, see [Active Directory and Active Directory Domain Services Port Requirements][ad-ds-ports].</span></span> <span data-ttu-id="6d0a2-156">此外，请确保 UDR 表不通过此体系结构中使用的 NVA 来路由 AD DS 流量。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-156">Also, ensure the UDR tables do not route AD DS traffic through the NVAs used in this architecture.</span></span> 

### <a name="active-directory-site"></a><span data-ttu-id="6d0a2-157">Active Directory 站点</span><span class="sxs-lookup"><span data-stu-id="6d0a2-157">Active Directory site</span></span>

<span data-ttu-id="6d0a2-158">在 AD DS 中，站点表示一个物理位置、网络或表示设备集合。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-158">In AD DS, a site represents a physical location, network, or collection of devices.</span></span> <span data-ttu-id="6d0a2-159">AD DS 站点用来管理 AD DS 数据库复制，它们将位置彼此靠近且由高速网络连接的 AD DS 对象分组到一起。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-159">AD DS sites are used to manage AD DS database replication by grouping together AD DS objects that are located close to one another and are connected by a high speed network.</span></span> <span data-ttu-id="6d0a2-160">AD DS 包括了相应的逻辑来选择用于在站点之间复制 AD DS 数据库的最佳策略。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-160">AD DS includes logic to select the best strategy for replacating the AD DS database between sites.</span></span>

<span data-ttu-id="6d0a2-161">建议你创建一个 AD DS 站点，并使其包括在 Azure 中为你的应用程序定义的子网。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-161">We recommend that you create an AD DS site including the subnets defined for your application in Azure.</span></span> <span data-ttu-id="6d0a2-162">然后，配置本地 AD DS 站点之间的站点链接，AD DS 将自动执行最高效的数据库复制。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-162">Then, configure a site link between your on-premises AD DS sites, and AD DS will automatically perform the most efficient database replication possible.</span></span> <span data-ttu-id="6d0a2-163">注意，除了初始配置之外，此数据库复制还要求进行少量其他配置。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-163">Note that this database replication requires little beyond the initial configuration.</span></span>

### <a name="active-directory-operations-masters"></a><span data-ttu-id="6d0a2-164">Active Directory 操作主机</span><span class="sxs-lookup"><span data-stu-id="6d0a2-164">Active Directory operations masters</span></span>

<span data-ttu-id="6d0a2-165">可以向 AD DS 域控制器分配操作主机角色以支持在复制的 AD DS 数据库实例之间执行一致性检查。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-165">The operations masters role can be assigned to AD DS domain controllers to support consistency checking between instances of replicated AD DS databases.</span></span> <span data-ttu-id="6d0a2-166">有五个操作主机角色：架构主机、域命名主机、相对标识符主机、主域控制器主机模拟器和基础结构主机。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-166">There are five operations master roles: schema master, domain naming master, relative identifier master, primary domain controller master emulator, and infrastructure master.</span></span> <span data-ttu-id="6d0a2-167">有关这些角色的详细信息，请参阅 [What are Operations Masters?][ad-ds-operations-masters]（什么是操作主机？）。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-167">For more information about these roles, see [What are Operations Masters?][ad-ds-operations-masters].</span></span>

<span data-ttu-id="6d0a2-168">建议不要向在 Azure 中部署的域控制器分配操作主机角色。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-168">We recommend you do not assign operations masters roles to the domain controllers deployed in Azure.</span></span>

### <a name="monitoring"></a><span data-ttu-id="6d0a2-169">监视</span><span class="sxs-lookup"><span data-stu-id="6d0a2-169">Monitoring</span></span>

<span data-ttu-id="6d0a2-170">监视域控制器 VM 以及 AD DS 服务的资源并创建一个计划来快速更正任何问题。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-170">Monitor the resources of the domain controller VMs as well as the AD DS Services and create a plan to quickly correct any problems.</span></span> <span data-ttu-id="6d0a2-171">有关详细信息，请参阅 [Monitoring Active Directory][monitoring_ad]（监视 Active Directory）。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-171">For more information, see [Monitoring Active Directory][monitoring_ad].</span></span> <span data-ttu-id="6d0a2-172">可以在监视服务器上（请参阅体系结构图）安装 [Microsoft Systems Center][microsoft_systems_center] 之类的工具来帮助执行这些任务。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-172">You can also install tools such as [Microsoft Systems Center][microsoft_systems_center] on the monitoring server (see the architecture diagram) to help perform these tasks.</span></span>  

## <a name="scalability-considerations"></a><span data-ttu-id="6d0a2-173">可伸缩性注意事项</span><span class="sxs-lookup"><span data-stu-id="6d0a2-173">Scalability considerations</span></span>

<span data-ttu-id="6d0a2-174">AD DS 是为实现可伸缩性而设计的。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-174">AD DS is designed for scalability.</span></span> <span data-ttu-id="6d0a2-175">你不需要配置负载均衡器或流量控制器来将请求定向到 AD DS 域控制器。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-175">You don't need to configure a load balancer or traffic controller to direct requests to AD DS domain controllers.</span></span> <span data-ttu-id="6d0a2-176">唯一的可伸缩性注意事项是为运行 AD DS 的 VM 配置满足你的网络负载要求的合适大小，监视 VM 上的负载并根据需要纵向扩展或收缩。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-176">The only scalability consideration is to configure the VMs running AD DS with the correct size for your network load requirements, monitor the load on the VMs, and scale up or down as necessary.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="6d0a2-177">可用性注意事项</span><span class="sxs-lookup"><span data-stu-id="6d0a2-177">Availability considerations</span></span>

<span data-ttu-id="6d0a2-178">将运行 AD DS 的 VM 部署到一个[可用性集][availability-set]中。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-178">Deploy the VMs running AD DS into an [availability set][availability-set].</span></span> <span data-ttu-id="6d0a2-179">另外，请考虑根据需要向至少一台（可能更多）服务器分配[备用操作主机][standby-operations-masters]角色。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-179">Also, consider assigning the role of [standby operations master][standby-operations-masters] to at least one server, and possibly more depending on your requirements.</span></span> <span data-ttu-id="6d0a2-180">备用操作主机是操作主机的主动副本，在故障转移期间可以用来替代主操作主机服务器。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-180">A standby operations master is an active copy of the operations master that can be used in place of the primary operations masters server during fail over.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="6d0a2-181">可管理性注意事项</span><span class="sxs-lookup"><span data-stu-id="6d0a2-181">Manageability considerations</span></span>

<span data-ttu-id="6d0a2-182">执行定期 AD DS 备份。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-182">Perform regular AD DS backups.</span></span> <span data-ttu-id="6d0a2-183">不要只是简单地复制域控制器的 VHD 文件，而是要执行定期备份，因为在复制 VHD 上的 AD DS 数据库文件时，该文件可能会未处于一致状态，导致无法重新启动数据库。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-183">Don't simply copy the VHD files of domain controllers instead of performing regular backups, because the AD DS database file on the VHD may not be in a consistent state when it's copied, making it impossible to restart the database.</span></span>

<span data-ttu-id="6d0a2-184">不要使用 Azure 门户关闭域控制器 VM。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-184">Do not shut down a domain controller VM using Azure portal.</span></span> <span data-ttu-id="6d0a2-185">相反，请从来宾操作系统进行关闭和重新启动。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-185">Instead, shut down and restart from the guest operating system.</span></span> <span data-ttu-id="6d0a2-186">通过门户进行关闭会导致 VM 被解除分配，这会重置 Active Directory 存储库的 `VM-GenerationID` 和 `invocationID`。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-186">Shutting down through the portal causes the VM to be deallocated, which resets both the `VM-GenerationID` and the `invocationID` of the Active Directory repository.</span></span> <span data-ttu-id="6d0a2-187">这将放弃 AD DS 相对标识符 (RID) 池并将 SYSVOL 标记为非权威的，并且可能需要配置域控制器。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-187">This discards the AD DS relative identifier (RID) pool and marks SYSVOL as nonauthoritative, and may require reconfiguration of the domain controller.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="6d0a2-188">安全注意事项</span><span class="sxs-lookup"><span data-stu-id="6d0a2-188">Security considerations</span></span>

<span data-ttu-id="6d0a2-189">AD DS 服务器提供身份验证服务并且是引入注目的攻击目标。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-189">AD DS servers provide authentication services and are an attractive target for attacks.</span></span> <span data-ttu-id="6d0a2-190">若要保护它们，请通过以下方法阻止直接 Internet 连接：将 AD DS 服务器放置在一个单独的子网中并使用 NSG 作为防火墙。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-190">To secure them, prevent direct Internet connectivity by placing the AD DS servers in a separate subnet with an NSG acting as a firewall.</span></span> <span data-ttu-id="6d0a2-191">关闭 AD DS 服务器上的所有端口，但身份验证、授权和服务器同步所需的那些端口除外。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-191">Close all ports on the AD DS servers except those necessary for authentication, authorization, and server synchronization.</span></span> <span data-ttu-id="6d0a2-192">有关详细信息，请参阅 [Active Directory and Active Directory Domain Services Port Requirements][ad-ds-ports]（Active Directory 和 Active Directory 域服务端口要求）。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-192">For more information, see [Active Directory and Active Directory Domain Services Port Requirements][ad-ds-ports].</span></span>

<span data-ttu-id="6d0a2-193">考虑通过一对子网和 NVA 在服务器周围实现一个额外的安全外围，如[在 Azure 中实现具有 Internet 访问的安全的混合网络体系结构][implementing-a-secure-hybrid-network-architecture-with-internet-access]中所述。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-193">Consider implementing an additional security perimeter around servers with a pair of subnets and NVAs, as described in [Implementing a secure hybrid network architecture with Internet access in Azure][implementing-a-secure-hybrid-network-architecture-with-internet-access].</span></span>

<span data-ttu-id="6d0a2-194">使用 BitLocker 或 Azure 磁盘加密对托管着 AD DS 数据库的磁盘进行加密。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-194">Use either BitLocker or Azure disk encryption to encrypt the disk hosting the AD DS database.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="6d0a2-195">部署解决方案</span><span class="sxs-lookup"><span data-stu-id="6d0a2-195">Deploy the solution</span></span>

<span data-ttu-id="6d0a2-196">[GitHub][github] 上提供了一个用于部署此参考体系结构的解决方案。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-196">A solution is available on [GitHub][github] to deploy this reference architecture.</span></span> <span data-ttu-id="6d0a2-197">若要运行部署此解决方案的 Powershell 脚本，需要具有 [Azure CLI][azure-powershell] 的最新版本。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-197">You will need the latest version of the [Azure CLI][azure-powershell] to run the Powershell script that deploys the solution.</span></span> <span data-ttu-id="6d0a2-198">若要部署此参考体系结构，请执行以下步骤：</span><span class="sxs-lookup"><span data-stu-id="6d0a2-198">To deploy the reference architecture, follow these steps:</span></span>

1. <span data-ttu-id="6d0a2-199">将解决方案文件夹从 [GitHub][github] 克隆到本地计算机。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-199">Download or clone the solution folder from [GitHub][github] to your local machine.</span></span>

2. <span data-ttu-id="6d0a2-200">打开 Azure CLI 并导航到本地解决方案文件夹。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-200">Open the Azure CLI and navigate to the local solution folder.</span></span>

3. <span data-ttu-id="6d0a2-201">运行以下命令：</span><span class="sxs-lookup"><span data-stu-id="6d0a2-201">Run the following command:</span></span>
    ```Powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```
    <span data-ttu-id="6d0a2-202">将 `<subscription id>` 替换为你的 Azure 订阅 ID。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-202">Replace `<subscription id>` with your Azure subscription ID.</span></span>
    <span data-ttu-id="6d0a2-203">对于 `<location>`，请指定一个 Azure 区域，例如 `eastus` 或 `westus`。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-203">For `<location>`, specify an Azure region, such as `eastus` or `westus`.</span></span>
    <span data-ttu-id="6d0a2-204">`<mode>` 参数控制部署粒度，可以是下列值之一：</span><span class="sxs-lookup"><span data-stu-id="6d0a2-204">The `<mode>` parameter controls the granularity of the deployment, and can be one of the following values:</span></span>
    * <span data-ttu-id="6d0a2-205">`Onpremise`：部署模拟的本地环境。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-205">`Onpremise`: deploys the simulated on-premises environment.</span></span>
    * <span data-ttu-id="6d0a2-206">`Infrastructure`：在 Azure 中部署 VNet 基础结构和 jumpbox。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-206">`Infrastructure`: deploys the VNet infrastructure and jump box in Azure.</span></span>
    * <span data-ttu-id="6d0a2-207">`CreateVpn`：部署 Azure 虚拟网络网关并将其连接到模拟的本地网络。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-207">`CreateVpn`: deploys the Azure virtual network gateway and connects it to the simulated on-premises network.</span></span>
    * <span data-ttu-id="6d0a2-208">`AzureADDS`：部署充当 AD DS 服务器的 VM，将 Active Directory 部署到这些 VM，并在 Azure 中部署域。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-208">`AzureADDS`: deploys the VMs acting as AD DS servers, deploys Active Directory to these VMs, and deploys the domain in Azure.</span></span>
    * <span data-ttu-id="6d0a2-209">`Workload`：部署公共和专用外围网络以及工作负荷层。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-209">`Workload`: deploys the public and private DMZs and the workload tier.</span></span>
    * <span data-ttu-id="6d0a2-210">`All`：部署上述所有部署。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-210">`All`: deploys all of the preceding deployments.</span></span> <span data-ttu-id="6d0a2-211">**如果没有现有的本地网络，但是希望如上所述部署完整的参考体系结构以用于测试或评估，则这是建议使用的选项。**</span><span class="sxs-lookup"><span data-stu-id="6d0a2-211">**This is the recommended option if If you do not have an existing on-premises network but you want to deploy the complete reference architecture described above for testing or evaluation.**</span></span>

4. <span data-ttu-id="6d0a2-212">等待部署完成。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-212">Wait for the deployment to complete.</span></span> <span data-ttu-id="6d0a2-213">如果要部署 `All` 部署，则将需要花费几个小时。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-213">If you are deploying the `All` deployment, it will take several hours.</span></span>

## <a name="next-steps"></a><span data-ttu-id="6d0a2-214">后续步骤</span><span class="sxs-lookup"><span data-stu-id="6d0a2-214">Next steps</span></span>

* <span data-ttu-id="6d0a2-215">了解在 Azure 中[创建 AD DS 资源林][adds-resource-forest]的最佳做法。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-215">Learn the best practices for [creating an AD DS resource forest][adds-resource-forest] in Azure.</span></span>
* <span data-ttu-id="6d0a2-216">了解在 Azure 中[创建 Active Directory 联合身份验证服务 (AD FS) 基础结构][adfs]的最佳做法。</span><span class="sxs-lookup"><span data-stu-id="6d0a2-216">Learn the best practices for [creating an Active Directory Federation Services (AD FS) infrastructure][adfs] in Azure.</span></span>

<!-- links -->
[adds-resource-forest]: adds-forest.md
[adfs]: adfs.md

[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[active-directory-domain-services]: https://technet.microsoft.com/library/dd448614.aspx
[adds-data-disks]: https://msdn.microsoft.com/library/azure/jj156090.aspx#BKMK_PlaceDB
[ad-ds-operations-masters]: https://technet.microsoft.com/library/cc779716(v=ws.10).aspx
[ad-ds-ports]: https://technet.microsoft.com/library/dd772723(v=ws.11).aspx
[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azure-expressroute]: https://azure.microsoft.com/documentation/articles/expressroute-introduction/
[azure-powershell]: /powershell/azureps-cmdlets-docs
[azure-vpn-gateway]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-vpngateways/
[capacity-planning-for-adds]: http://social.technet.microsoft.com/wiki/contents/articles/14355.capacity-planning-for-active-directory-domain-services.aspx
[considerations]: ./considerations.md
[GitHub]: https://github.com/mspnp/reference-architectures/tree/master/identity/adds-extend-domain
[microsoft_systems_center]: https://www.microsoft.com/server-cloud/products/system-center-2016/
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[security-considerations]: #security-considerations
[set-a-static-ip-address]: https://azure.microsoft.com/documentation/articles/virtual-networks-static-private-ip-arm-pportal/
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
[vm-windows-sizes]: /azure/virtual-machines/virtual-machines-windows-sizes

[0]: ./images/adds-extend-domain.png "使用 Active Directory 保护混合网络体系结构的安全"
