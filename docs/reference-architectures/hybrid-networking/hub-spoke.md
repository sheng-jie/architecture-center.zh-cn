---
title: 在 Azure 中实现中心辐射型网络拓扑
description: 如何在 Azure 中实现中心辐射型网络拓扑。
author: telmosampaio
ms.date: 04/09/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: 3b19526a9ed77c1605325a9eec101ffbee7c8401
ms.sourcegitcommit: 3846a0ab2b2b2552202a3c9c21af0097a145ffc6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/29/2018
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="e931b-103">在 Azure 中实现中心辐射型网络拓扑</span><span class="sxs-lookup"><span data-stu-id="e931b-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="e931b-104">此参考体系结构展示了如何在 Azure 中实现中心辐射型拓扑。</span><span class="sxs-lookup"><span data-stu-id="e931b-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="e931b-105">*中心*是 Azure 中的一个虚拟网络 (VNet)，充当到本地网络的连接的中心点。</span><span class="sxs-lookup"><span data-stu-id="e931b-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="e931b-106">*辐射*是与中心对等互连的 VNet，可用于隔离工作负荷。</span><span class="sxs-lookup"><span data-stu-id="e931b-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="e931b-107">流量通过 ExpressRoute 或 VPN 网关连接在本地数据中心与中心之间流动。</span><span class="sxs-lookup"><span data-stu-id="e931b-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="e931b-108">[**部署此解决方案**](#deploy-the-solution)。</span><span class="sxs-lookup"><span data-stu-id="e931b-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="e931b-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="e931b-109">![[0]][0]</span></span>

<span data-ttu-id="e931b-110">*下载此体系结构的 [Visio 文件][visio-download]*</span><span class="sxs-lookup"><span data-stu-id="e931b-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="e931b-111">此拓扑的好处包括：</span><span class="sxs-lookup"><span data-stu-id="e931b-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="e931b-112">**节省成本** - 通过将可以由多个工作负荷（例如网络虚拟设备 (NVAs) 和 DNS 服务器）共享的服务集中放置在单个位置中。</span><span class="sxs-lookup"><span data-stu-id="e931b-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="e931b-113">**克服订阅限制** - 通过将不同订阅中的 Vnet 对等互连到中心。</span><span class="sxs-lookup"><span data-stu-id="e931b-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="e931b-114">**关注点隔离**（在中心 IT（SecOps、InfraOps）与工作负荷 (DevOps) 之间）。</span><span class="sxs-lookup"><span data-stu-id="e931b-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="e931b-115">此体系结构的典型用途包括：</span><span class="sxs-lookup"><span data-stu-id="e931b-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="e931b-116">在各种环境（例如开发、测试和生产）中部署的需要使用共享服务（例如 DNS、IDS、NTP 或 AD DS）的工作负荷。</span><span class="sxs-lookup"><span data-stu-id="e931b-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="e931b-117">共享服务放置在中心 VNet 中，而每个环境都部署到辐射以保持隔离。</span><span class="sxs-lookup"><span data-stu-id="e931b-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="e931b-118">不需要彼此连接但需要访问共享服务的工作负荷。</span><span class="sxs-lookup"><span data-stu-id="e931b-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="e931b-119">需要对安全方面进行集中控制（例如作为外围网络的中心内的防火墙），并且需要在每个辐射中对工作负荷进行隔离管理的企业。</span><span class="sxs-lookup"><span data-stu-id="e931b-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="e931b-120">体系结构</span><span class="sxs-lookup"><span data-stu-id="e931b-120">Architecture</span></span>

<span data-ttu-id="e931b-121">该体系结构包括以下组件。</span><span class="sxs-lookup"><span data-stu-id="e931b-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="e931b-122">**本地网络**。</span><span class="sxs-lookup"><span data-stu-id="e931b-122">**On-premises network**.</span></span> <span data-ttu-id="e931b-123">在组织内运行的一个专用局域网。</span><span class="sxs-lookup"><span data-stu-id="e931b-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="e931b-124">**VPN 设备**。</span><span class="sxs-lookup"><span data-stu-id="e931b-124">**VPN device**.</span></span> <span data-ttu-id="e931b-125">提供到本地网络的外部连接的设备或服务。</span><span class="sxs-lookup"><span data-stu-id="e931b-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="e931b-126">VPN 设备可以是硬件设备，也可以是软件解决方案，例如 Windows Server 2012 中的路由和远程访问服务 (RRAS)。</span><span class="sxs-lookup"><span data-stu-id="e931b-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="e931b-127">有关受支持 VPN 设备的列表和有关为连接到 Azure 而配置所选 VPN 设备的信息，请参阅 [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance]（关于站点到站点 VPN 网关连接的 VPN 设备）。</span><span class="sxs-lookup"><span data-stu-id="e931b-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="e931b-128">**VPN 虚拟网络网关或 ExpressRoute 网关**。</span><span class="sxs-lookup"><span data-stu-id="e931b-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="e931b-129">虚拟网络网关可以将 VNet 连接到用于本地网络连接的 VPN 设备或 ExpressRoute 线路。</span><span class="sxs-lookup"><span data-stu-id="e931b-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="e931b-130">有关详细信息，请参阅[将本地网络连接到 Microsoft Azure 虚拟网络][connect-to-an-Azure-vnet]。</span><span class="sxs-lookup"><span data-stu-id="e931b-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="e931b-131">此参考体系结构的部署脚本使用 VPN 网关进行连接，使用 Azure 中的 VNet 来模拟本地网络。</span><span class="sxs-lookup"><span data-stu-id="e931b-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="e931b-132">**中心 VNet**。</span><span class="sxs-lookup"><span data-stu-id="e931b-132">**Hub VNet**.</span></span> <span data-ttu-id="e931b-133">用作中心辐射型拓扑中的中心的 Azure VNet。</span><span class="sxs-lookup"><span data-stu-id="e931b-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="e931b-134">中心是到本地网络的连接的中心点，它还托管着可以由辐射 VNet 中托管的各种工作负荷使用的服务。</span><span class="sxs-lookup"><span data-stu-id="e931b-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="e931b-135">**网关子网**。</span><span class="sxs-lookup"><span data-stu-id="e931b-135">**Gateway subnet**.</span></span> <span data-ttu-id="e931b-136">虚拟网络网关保留在同一子网中。</span><span class="sxs-lookup"><span data-stu-id="e931b-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="e931b-137">**辐射 VNet**。</span><span class="sxs-lookup"><span data-stu-id="e931b-137">**Spoke VNets**.</span></span> <span data-ttu-id="e931b-138">用作中心辐射型拓扑中的辐射的一个或多个 Azure VNet。</span><span class="sxs-lookup"><span data-stu-id="e931b-138">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="e931b-139">辐射可以用来隔离其自己的 VNet 中的工作负荷，独立于其他辐射进行管理。</span><span class="sxs-lookup"><span data-stu-id="e931b-139">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="e931b-140">每个工作负荷可以包括多个层，并具有通过 Azure 负载均衡器连接的多个子网。</span><span class="sxs-lookup"><span data-stu-id="e931b-140">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="e931b-141">有关应用程序基础结构的详细信息，请参阅[运行 Windows VM 工作负荷][windows-vm-ra]和[运行 Linux VM 工作负荷][linux-vm-ra]。</span><span class="sxs-lookup"><span data-stu-id="e931b-141">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="e931b-142">**VNet 对等互连**。</span><span class="sxs-lookup"><span data-stu-id="e931b-142">**VNet peering**.</span></span> <span data-ttu-id="e931b-143">可以使用[对等互连连接][vnet-peering]来连接同一 Azure 区域中的两个 VNet。</span><span class="sxs-lookup"><span data-stu-id="e931b-143">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="e931b-144">对等互连连接是 VNet 之间的不可传递低延迟连接。</span><span class="sxs-lookup"><span data-stu-id="e931b-144">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="e931b-145">进行对等互连后，VNet 可使用 Azure 主干交换流量，不需要使用路由器。</span><span class="sxs-lookup"><span data-stu-id="e931b-145">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="e931b-146">在中心辐射型网络拓扑中，将使用 VNet 对等互连来将中心连接到每个辐射。</span><span class="sxs-lookup"><span data-stu-id="e931b-146">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="e931b-147">文本仅涵盖了[资源管理器](/azure/azure-resource-manager/resource-group-overview)部署，但也可以将经典 VNet 连接到同一订阅中的资源管理器 VNet。</span><span class="sxs-lookup"><span data-stu-id="e931b-147">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="e931b-148">这样，辐射将可以托管经典部署，并且仍然可以从中心内共享的各种服务受益。</span><span class="sxs-lookup"><span data-stu-id="e931b-148">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="e931b-149">建议</span><span class="sxs-lookup"><span data-stu-id="e931b-149">Recommendations</span></span>

<span data-ttu-id="e931b-150">以下建议适用于大多数方案。</span><span class="sxs-lookup"><span data-stu-id="e931b-150">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="e931b-151">除非有优先于这些建议的特定要求，否则请遵循这些建议。</span><span class="sxs-lookup"><span data-stu-id="e931b-151">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="e931b-152">资源组</span><span class="sxs-lookup"><span data-stu-id="e931b-152">Resource groups</span></span>

<span data-ttu-id="e931b-153">中心 VNet 和每个辐射 VNet 可以在不同的资源组中实现，甚至可以在不同的订阅中实现，只要它们属于同一 Azure 区域中的同一 Azure Active Directory (Azure AD) 租户即可。</span><span class="sxs-lookup"><span data-stu-id="e931b-153">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions, as long as they belong to the same Azure Active Directory (Azure AD) tenant in the same Azure region.</span></span> <span data-ttu-id="e931b-154">这样，可以对各个工作负荷进行非集中管理，同时在中心 VNet 内维护共享服务。</span><span class="sxs-lookup"><span data-stu-id="e931b-154">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="e931b-155">VNet 和 GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="e931b-155">VNet and GatewaySubnet</span></span>

<span data-ttu-id="e931b-156">创建一个名为 *GatewaySubnet* 的子网，使其地址范围为 /27。</span><span class="sxs-lookup"><span data-stu-id="e931b-156">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="e931b-157">此子网是虚拟网络网关所必需的。</span><span class="sxs-lookup"><span data-stu-id="e931b-157">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="e931b-158">向此子网分配 32 个地址将有助于防止将来达到网关大小限制。</span><span class="sxs-lookup"><span data-stu-id="e931b-158">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="e931b-159">有关设置网关的详细信息，请根据你的连接类型参阅以下参考体系结构：</span><span class="sxs-lookup"><span data-stu-id="e931b-159">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="e931b-160">[使用 ExpressRoute 的混合网络][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="e931b-160">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="e931b-161">[使用 VPN 网关的混合网络][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="e931b-161">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="e931b-162">要实现更高的可用性，可以将 ExpressRoute 外加 VPN 用于故障转移。</span><span class="sxs-lookup"><span data-stu-id="e931b-162">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="e931b-163">请参阅[将本地网络连接到 Azure 并将 ExpressRoute 和 VPN 用于故障转移][hybrid-ha]。</span><span class="sxs-lookup"><span data-stu-id="e931b-163">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="e931b-164">如果不需要与本地网络的连接，还可以在不使用网关的情况下使用中心辐射型拓扑。</span><span class="sxs-lookup"><span data-stu-id="e931b-164">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="e931b-165">VNet 对等互连</span><span class="sxs-lookup"><span data-stu-id="e931b-165">VNet peering</span></span>

<span data-ttu-id="e931b-166">VNet 对等互连是两个 VNet 之间的不可传递关系。</span><span class="sxs-lookup"><span data-stu-id="e931b-166">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="e931b-167">如果需要将各个辐射彼此连接，请考虑在这些辐射之间添加一个单独的对等互连连接。</span><span class="sxs-lookup"><span data-stu-id="e931b-167">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="e931b-168">不过，如果有多个辐射需要彼此连接，则你可能会由于[每个 VNet 的 VNet 对等互连数限制][vnet-peering-limit]而很快耗尽可能的对等互连连接。</span><span class="sxs-lookup"><span data-stu-id="e931b-168">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="e931b-169">在这种情况下，请考虑使用用户定义的路由 (UDR) 强制将目的地为辐射的流量发送到在中心 VNet 中充当路由器的 NVA。</span><span class="sxs-lookup"><span data-stu-id="e931b-169">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="e931b-170">这将允许各个辐射彼此连接。</span><span class="sxs-lookup"><span data-stu-id="e931b-170">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="e931b-171">还可以将辐射配置为使用中心 VNet 网关与远程网络进行通信。</span><span class="sxs-lookup"><span data-stu-id="e931b-171">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="e931b-172">若要允许网关流量从辐射流动到中心，以及允许连接到远程网络，必须：</span><span class="sxs-lookup"><span data-stu-id="e931b-172">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="e931b-173">在中心内配置 VNet 对等互连连接以**允许网关中转**。</span><span class="sxs-lookup"><span data-stu-id="e931b-173">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="e931b-174">在每个辐射中配置 VNet 对等互连连接以**使用远程网关**。</span><span class="sxs-lookup"><span data-stu-id="e931b-174">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="e931b-175">配置所有 VNet 对等互连连接以**允许转发的流量**。</span><span class="sxs-lookup"><span data-stu-id="e931b-175">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="e931b-176">注意事项</span><span class="sxs-lookup"><span data-stu-id="e931b-176">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="e931b-177">辐射连接</span><span class="sxs-lookup"><span data-stu-id="e931b-177">Spoke connectivity</span></span>

<span data-ttu-id="e931b-178">如果辐射之间需要存在连接，请考虑在中心内实现一个用于路由的 NVA，并在辐射中使用 UDR 将流量转发到中心。</span><span class="sxs-lookup"><span data-stu-id="e931b-178">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="e931b-179">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="e931b-179">![[2]][2]</span></span>

<span data-ttu-id="e931b-180">在这种情况下，必须配置对等互连连接以**允许转发的流量**。</span><span class="sxs-lookup"><span data-stu-id="e931b-180">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="e931b-181">克服 VNet 对等互连限制</span><span class="sxs-lookup"><span data-stu-id="e931b-181">Overcoming VNet peering limits</span></span>

<span data-ttu-id="e931b-182">请务必考虑 Azure 中[每个 VNet 的 VNet 对等互连数限制][vnet-peering-limit]。</span><span class="sxs-lookup"><span data-stu-id="e931b-182">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="e931b-183">如果你确定所需的辐射多于限制将允许的数量，请考虑创建一个中心-辐射-中心-辐射型拓扑，其中的第一级辐射还充当中心。</span><span class="sxs-lookup"><span data-stu-id="e931b-183">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="e931b-184">下图显示了此方式。</span><span class="sxs-lookup"><span data-stu-id="e931b-184">The following diagram shows this approach.</span></span>

<span data-ttu-id="e931b-185">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="e931b-185">![[3]][3]</span></span>

<span data-ttu-id="e931b-186">另外，请考虑要在中心内共享哪些服务，以确保中心能够针对大量辐射进行缩放。</span><span class="sxs-lookup"><span data-stu-id="e931b-186">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="e931b-187">例如，如果中心提供防火墙服务，则在添加多个辐射时请考虑防火墙解决方案的带宽限制。</span><span class="sxs-lookup"><span data-stu-id="e931b-187">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="e931b-188">你可能希望将这些共享服务中的某一些移动到二级中心内。</span><span class="sxs-lookup"><span data-stu-id="e931b-188">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="e931b-189">部署解决方案</span><span class="sxs-lookup"><span data-stu-id="e931b-189">Deploy the solution</span></span>

<span data-ttu-id="e931b-190">[GitHub][ref-arch-repo] 上提供了此体系结构的部署。</span><span class="sxs-lookup"><span data-stu-id="e931b-190">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="e931b-191">该部署使用每个 VNet 中的 VM 来测试连接。</span><span class="sxs-lookup"><span data-stu-id="e931b-191">It uses VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="e931b-192">没有实际服务托管在 **中心 VNet** 内的**共享服务**中。</span><span class="sxs-lookup"><span data-stu-id="e931b-192">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

<span data-ttu-id="e931b-193">该部署在订阅中创建以下资源组：</span><span class="sxs-lookup"><span data-stu-id="e931b-193">The deployment creates the following resource groups in your subscription:</span></span>

- <span data-ttu-id="e931b-194">hub-nva-rg</span><span class="sxs-lookup"><span data-stu-id="e931b-194">hub-nva-rg</span></span>
- <span data-ttu-id="e931b-195">hub-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="e931b-195">hub-vnet-rg</span></span>
- <span data-ttu-id="e931b-196">onprem-jb-rg</span><span class="sxs-lookup"><span data-stu-id="e931b-196">onprem-jb-rg</span></span>
- <span data-ttu-id="e931b-197">onprem-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="e931b-197">onprem-vnet-rg</span></span>
- <span data-ttu-id="e931b-198">spoke1-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="e931b-198">spoke1-vnet-rg</span></span>
- <span data-ttu-id="e931b-199">spoke2-vent-rg</span><span class="sxs-lookup"><span data-stu-id="e931b-199">spoke2-vent-rg</span></span>

<span data-ttu-id="e931b-200">模板参数文件将引用这些名称，因此，如果更改了这些名称，请相应地更新参数文件。</span><span class="sxs-lookup"><span data-stu-id="e931b-200">The template parameter files refer to these names, so if you change them, update the parameter files to match.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="e931b-201">先决条件</span><span class="sxs-lookup"><span data-stu-id="e931b-201">Prerequisites</span></span>

1. <span data-ttu-id="e931b-202">克隆、下载[参考体系结构][ref-arch-repo] GitHub 存储库的 zip 文件或创建其分支。</span><span class="sxs-lookup"><span data-stu-id="e931b-202">Clone, fork, or download the zip file for the [reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="e931b-203">安装 [Azure CLI 2.0][azure-cli-2]。</span><span class="sxs-lookup"><span data-stu-id="e931b-203">Install [Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="e931b-204">安装 [Azure 构建基块][azbb] npm 包。</span><span class="sxs-lookup"><span data-stu-id="e931b-204">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="e931b-205">在命令提示符、bash 提示符或 PowerShell 提示符下使用以下命令登录到 Azure 帐户。</span><span class="sxs-lookup"><span data-stu-id="e931b-205">From a command prompt, bash prompt, or PowerShell prompt, log into your Azure account by using the command below.</span></span>

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="e931b-206">部署模拟的本地数据中心</span><span class="sxs-lookup"><span data-stu-id="e931b-206">Deploy the simulated on-premises datacenter</span></span>

<span data-ttu-id="e931b-207">若要将模拟的本地数据中心部署为 Azure VNet，请执行以下步骤：</span><span class="sxs-lookup"><span data-stu-id="e931b-207">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="e931b-208">导航到参考体系结构存储库的 `hybrid-networking/hub-spoke` 文件夹。</span><span class="sxs-lookup"><span data-stu-id="e931b-208">Navigate to the `hybrid-networking/hub-spoke` folder of the reference architectures repository.</span></span>

2. <span data-ttu-id="e931b-209">打开 `onprem.json` 文件。</span><span class="sxs-lookup"><span data-stu-id="e931b-209">Open the `onprem.json` file.</span></span> <span data-ttu-id="e931b-210">替换 `adminUsername` 和 `adminPassword` 的值。</span><span class="sxs-lookup"><span data-stu-id="e931b-210">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

3. <span data-ttu-id="e931b-211">（可选）对于 Linux 部署，请将 `osType` 设置为 `Linux`。</span><span class="sxs-lookup"><span data-stu-id="e931b-211">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

4. <span data-ttu-id="e931b-212">运行以下命令：</span><span class="sxs-lookup"><span data-stu-id="e931b-212">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
    ```

5. <span data-ttu-id="e931b-213">等待部署完成。</span><span class="sxs-lookup"><span data-stu-id="e931b-213">Wait for the deployment to finish.</span></span> <span data-ttu-id="e931b-214">此部署创建虚拟网络、虚拟机和 VPN 网关。</span><span class="sxs-lookup"><span data-stu-id="e931b-214">This deployment creates a virtual network, a virtual machine, and a VPN gateway.</span></span> <span data-ttu-id="e931b-215">创建 VPN 网关可能需要花费大约 40 分钟。</span><span class="sxs-lookup"><span data-stu-id="e931b-215">It can take about 40 minutes to create the VPN gateway.</span></span>

### <a name="deploy-the-hub-vnet"></a><span data-ttu-id="e931b-216">部署中心 VNet</span><span class="sxs-lookup"><span data-stu-id="e931b-216">Deploy the hub VNet</span></span>

<span data-ttu-id="e931b-217">若要部署中心 VNet，请执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="e931b-217">To deploy the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="e931b-218">打开 `hub-vnet.json` 文件。</span><span class="sxs-lookup"><span data-stu-id="e931b-218">Open the `hub-vnet.json` file.</span></span> <span data-ttu-id="e931b-219">替换 `adminUsername` 和 `adminPassword` 的值。</span><span class="sxs-lookup"><span data-stu-id="e931b-219">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="e931b-220">（可选）对于 Linux 部署，请将 `osType` 设置为 `Linux`。</span><span class="sxs-lookup"><span data-stu-id="e931b-220">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

3. <span data-ttu-id="e931b-221">对于 `sharedKey`，请输入 VPN 连接的共享密钥。</span><span class="sxs-lookup"><span data-stu-id="e931b-221">For `sharedKey`, enter a shared key for the VPN connection.</span></span> 

    ```bash
    "sharedKey": "",
    ```

4. <span data-ttu-id="e931b-222">运行以下命令：</span><span class="sxs-lookup"><span data-stu-id="e931b-222">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
    ```

5. <span data-ttu-id="e931b-223">等待部署完成。</span><span class="sxs-lookup"><span data-stu-id="e931b-223">Wait for the deployment to finish.</span></span> <span data-ttu-id="e931b-224">此部署创建虚拟网络、虚拟机、VPN 网关和网关连接。</span><span class="sxs-lookup"><span data-stu-id="e931b-224">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway.</span></span>  <span data-ttu-id="e931b-225">创建 VPN 网关可能需要花费大约 40 分钟。</span><span class="sxs-lookup"><span data-stu-id="e931b-225">It can take about 40 minutes to create the VPN gateway.</span></span>

### <a name="test-connectivity-with-the-hub"></a><span data-ttu-id="e931b-226">测试与中心的连接</span><span class="sxs-lookup"><span data-stu-id="e931b-226">Test connectivity with the hub</span></span>

<span data-ttu-id="e931b-227">测试从模拟本地环境到中心 VNet 的连接。</span><span class="sxs-lookup"><span data-stu-id="e931b-227">Test conectivity from the simulated on-premises environment to the hub VNet.</span></span>

<span data-ttu-id="e931b-228">**Windows 部署**</span><span class="sxs-lookup"><span data-stu-id="e931b-228">**Windows deployment**</span></span>

1. <span data-ttu-id="e931b-229">使用 Azure 门户在 `onprem-jb-rg` 资源组中找到名为 `jb-vm1` 的 VM。</span><span class="sxs-lookup"><span data-stu-id="e931b-229">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="e931b-230">单击 `Connect` 来与 VM 建立远程桌面会话。</span><span class="sxs-lookup"><span data-stu-id="e931b-230">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="e931b-231">使用 `onprem.json` 参数文件中指定的密码。</span><span class="sxs-lookup"><span data-stu-id="e931b-231">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="e931b-232">在 VM 中打开 PowerShell 控制台，使用 `Test-NetConnection` cmdlet 验证能否连接到中心 VNet 中的 Jumpbox VM。</span><span class="sxs-lookup"><span data-stu-id="e931b-232">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VM in the hub VNet.</span></span>

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```
<span data-ttu-id="e931b-233">输出应如下所示：</span><span class="sxs-lookup"><span data-stu-id="e931b-233">The output should look similar to the following:</span></span>

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> <span data-ttu-id="e931b-234">默认情况下，Windows Server VM 不允许 Azure 中的 ICMP 响应。</span><span class="sxs-lookup"><span data-stu-id="e931b-234">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="e931b-235">若要使用 `ping` 来测试连接，需在“Windows 高级防火墙”中为每个 VM 启用 ICMP 流量。</span><span class="sxs-lookup"><span data-stu-id="e931b-235">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="e931b-236">**Linux 部署**</span><span class="sxs-lookup"><span data-stu-id="e931b-236">**Linux deployment**</span></span>

1. <span data-ttu-id="e931b-237">使用 Azure 门户在 `onprem-jb-rg` 资源组中找到名为 `jb-vm1` 的 VM。</span><span class="sxs-lookup"><span data-stu-id="e931b-237">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="e931b-238">单击 `Connect`，并复制门户中显示的 `ssh` 命令。</span><span class="sxs-lookup"><span data-stu-id="e931b-238">Click `Connect` and copy the `ssh` command shown in the portal.</span></span> 

3. <span data-ttu-id="e931b-239">在 Linux 提示符下，运行 `ssh` 连接到模拟本地环境。</span><span class="sxs-lookup"><span data-stu-id="e931b-239">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment.</span></span> <span data-ttu-id="e931b-240">使用 `onprem.json` 参数文件中指定的密码。</span><span class="sxs-lookup"><span data-stu-id="e931b-240">Use the password that you specified in the `onprem.json` parameter file.</span></span>

4. <span data-ttu-id="e931b-241">使用 `ping` 命令测试与中心 VNet 中 Jumpbox VM 连接。</span><span class="sxs-lookup"><span data-stu-id="e931b-241">Use the `ping` command to test connectivity to the jumpbox VM in the hub VNet:</span></span>

   ```bash
   ping 10.0.0.68
   ```

### <a name="deploy-the-spoke-vnets"></a><span data-ttu-id="e931b-242">部署辐射 VNet</span><span class="sxs-lookup"><span data-stu-id="e931b-242">Deploy the spoke VNets</span></span>

<span data-ttu-id="e931b-243">若要部署辐射 VNet，请执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="e931b-243">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="e931b-244">打开 `spoke1.json` 文件。</span><span class="sxs-lookup"><span data-stu-id="e931b-244">Open the `spoke1.json` file.</span></span> <span data-ttu-id="e931b-245">替换 `adminUsername` 和 `adminPassword` 的值。</span><span class="sxs-lookup"><span data-stu-id="e931b-245">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="e931b-246">（可选）对于 Linux 部署，请将 `osType` 设置为 `Linux`。</span><span class="sxs-lookup"><span data-stu-id="e931b-246">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

3. <span data-ttu-id="e931b-247">运行以下命令：</span><span class="sxs-lookup"><span data-stu-id="e931b-247">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```
  
4. <span data-ttu-id="e931b-248">针对 `spoke2.json` 文件重复步骤 1-2。</span><span class="sxs-lookup"><span data-stu-id="e931b-248">Repeat steps 1-2 for the `spoke2.json` file.</span></span>

5. <span data-ttu-id="e931b-249">运行以下命令：</span><span class="sxs-lookup"><span data-stu-id="e931b-249">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

6. <span data-ttu-id="e931b-250">运行以下命令：</span><span class="sxs-lookup"><span data-stu-id="e931b-250">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
   ```

### <a name="test-connectivity"></a><span data-ttu-id="e931b-251">测试连接</span><span class="sxs-lookup"><span data-stu-id="e931b-251">Test connectivity</span></span>

<span data-ttu-id="e931b-252">测试从模拟本地环境到辐射 VNet 的连接。</span><span class="sxs-lookup"><span data-stu-id="e931b-252">Test conectivity from the simulated on-premises environment to the spoke VNets.</span></span>

<span data-ttu-id="e931b-253">**Windows 部署**</span><span class="sxs-lookup"><span data-stu-id="e931b-253">**Windows deployment**</span></span>

1. <span data-ttu-id="e931b-254">使用 Azure 门户在 `onprem-jb-rg` 资源组中找到名为 `jb-vm1` 的 VM。</span><span class="sxs-lookup"><span data-stu-id="e931b-254">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="e931b-255">单击 `Connect` 来与 VM 建立远程桌面会话。</span><span class="sxs-lookup"><span data-stu-id="e931b-255">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="e931b-256">使用 `onprem.json` 参数文件中指定的密码。</span><span class="sxs-lookup"><span data-stu-id="e931b-256">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="e931b-257">在 VM 中打开 PowerShell 控制台，使用 `Test-NetConnection` cmdlet 验证能否连接到中心 VNet 中的 Jumpbox VM。</span><span class="sxs-lookup"><span data-stu-id="e931b-257">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VM in the hub VNet.</span></span>

   ```powershell
   Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
   Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
   ```

<span data-ttu-id="e931b-258">**Linux 部署**</span><span class="sxs-lookup"><span data-stu-id="e931b-258">**Linux deployment**</span></span>

<span data-ttu-id="e931b-259">若要使用 Linux VM 测试从模拟的本地环境到辐射 VNet 的连接，请执行以下步骤：</span><span class="sxs-lookup"><span data-stu-id="e931b-259">To test conectivity from the simulated on-premises environment to the spoke VNets using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="e931b-260">使用 Azure 门户在 `onprem-jb-rg` 资源组中找到名为 `jb-vm1` 的 VM。</span><span class="sxs-lookup"><span data-stu-id="e931b-260">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="e931b-261">单击 `Connect`，并复制门户中显示的 `ssh` 命令。</span><span class="sxs-lookup"><span data-stu-id="e931b-261">Click `Connect` and copy the `ssh` command shown in the portal.</span></span> 

3. <span data-ttu-id="e931b-262">在 Linux 提示符下，运行 `ssh` 连接到模拟本地环境。</span><span class="sxs-lookup"><span data-stu-id="e931b-262">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment.</span></span> <span data-ttu-id="e931b-263">使用 `onprem.json` 参数文件中指定的密码。</span><span class="sxs-lookup"><span data-stu-id="e931b-263">Use the password that you specified in the `onprem.json` parameter file.</span></span>

5. <span data-ttu-id="e931b-264">使用 `ping` 命令测试与每个辐射中 Jumpbox VM 的连接。</span><span class="sxs-lookup"><span data-stu-id="e931b-264">Use the `ping` command to test connectivity to the jumpbox VMs in each spoke:</span></span>

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="e931b-265">添加辐射之间的连接</span><span class="sxs-lookup"><span data-stu-id="e931b-265">Add connectivity between spokes</span></span>

<span data-ttu-id="e931b-266">此步骤是可选的。</span><span class="sxs-lookup"><span data-stu-id="e931b-266">This step is optional.</span></span> <span data-ttu-id="e931b-267">如果需要允许辐射 VNet 互相进行连接，则必须使用网络虚拟设备 (NVA) 作为中心 VNet 中的路由器，并在尝试连接到另一辐射 VNet 时强制流量从辐射 VNet 流向路由器。</span><span class="sxs-lookup"><span data-stu-id="e931b-267">If you want to allow spokes to connect to each other, you must use a newtwork virtual appliance (NVA) as a router in the hub VNet, and force traffic from spokes to the router when trying to connect to another spoke.</span></span> <span data-ttu-id="e931b-268">若要将基本的示例 NVA 部署为单个 VM 并部署用户定义的路由 (UDR) 以允许这两个辐射 VNet 建立连接，请执行以下步骤：</span><span class="sxs-lookup"><span data-stu-id="e931b-268">To deploy a basic sample NVA as a single VM, along with user-defined routes (UDRs) to allow the two spoke VNets to connect, perform the following steps:</span></span>

1. <span data-ttu-id="e931b-269">打开 `hub-nva.json` 文件。</span><span class="sxs-lookup"><span data-stu-id="e931b-269">Open the `hub-nva.json` file.</span></span> <span data-ttu-id="e931b-270">替换 `adminUsername` 和 `adminPassword` 的值。</span><span class="sxs-lookup"><span data-stu-id="e931b-270">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="e931b-271">运行以下命令：</span><span class="sxs-lookup"><span data-stu-id="e931b-271">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Azure 中的中心辐射型拓扑"
[1]: ./images/hub-spoke-gateway-routing.svg "Azure 中的具有可传递路由的中心辐射型拓扑"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Azure 中的具有使用 NVA 的可传递路由的中心辐射型拓扑"
[3]: ./images/hub-spokehub-spoke.svg "Azure 中的中心-辐射-中心-辐射型拓扑"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
