---
title: "在 Azure 中实现中心辐射型网络拓扑"
description: "如何在 Azure 中实现中心辐射型网络拓扑。"
author: telmosampaio
ms.date: 05/05/2017
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: e6f07a7962dd5728226b023700268340590d97a3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="3eebf-103">在 Azure 中实现中心辐射型网络拓扑</span><span class="sxs-lookup"><span data-stu-id="3eebf-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="3eebf-104">此参考体系结构展示了如何在 Azure 中实现中心辐射型拓扑。</span><span class="sxs-lookup"><span data-stu-id="3eebf-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="3eebf-105">*中心*是 Azure 中的一个虚拟网络 (VNet)，充当到本地网络的连接的中心点。</span><span class="sxs-lookup"><span data-stu-id="3eebf-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="3eebf-106">*辐射*是与中心对等互连的 VNet，可用于隔离工作负荷。</span><span class="sxs-lookup"><span data-stu-id="3eebf-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="3eebf-107">流量通过 ExpressRoute 或 VPN 网关连接在本地数据中心与中心之间流动。</span><span class="sxs-lookup"><span data-stu-id="3eebf-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="3eebf-108">[**部署此解决方案**](#deploy-the-solution)。</span><span class="sxs-lookup"><span data-stu-id="3eebf-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="3eebf-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="3eebf-109">![[0]][0]</span></span>

<span data-ttu-id="3eebf-110">*下载此体系结构的 [Visio 文件][visio-download]*</span><span class="sxs-lookup"><span data-stu-id="3eebf-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="3eebf-111">此拓扑的好处包括：</span><span class="sxs-lookup"><span data-stu-id="3eebf-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="3eebf-112">**节省成本** - 通过将可以由多个工作负荷（例如网络虚拟设备 (NVAs) 和 DNS 服务器）共享的服务集中放置在单个位置中。</span><span class="sxs-lookup"><span data-stu-id="3eebf-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="3eebf-113">**克服订阅限制** - 通过将不同订阅中的 Vnet 对等互连到中心。</span><span class="sxs-lookup"><span data-stu-id="3eebf-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="3eebf-114">**关注点隔离**（在中心 IT（SecOps、InfraOps）与工作负荷 (DevOps) 之间）。</span><span class="sxs-lookup"><span data-stu-id="3eebf-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="3eebf-115">此体系结构的典型用途包括：</span><span class="sxs-lookup"><span data-stu-id="3eebf-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="3eebf-116">在各种环境（例如开发、测试和生产）中部署的需要使用共享服务（例如 DNS、IDS、NTP 或 AD DS）的工作负荷。</span><span class="sxs-lookup"><span data-stu-id="3eebf-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="3eebf-117">共享服务放置在中心 VNet 中，而每个环境都部署到辐射以保持隔离。</span><span class="sxs-lookup"><span data-stu-id="3eebf-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="3eebf-118">不需要彼此连接但需要访问共享服务的工作负荷。</span><span class="sxs-lookup"><span data-stu-id="3eebf-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="3eebf-119">需要对安全方面进行集中控制（例如作为外围网络的中心内的防火墙），并且需要在每个辐射中对工作负荷进行隔离管理的企业。</span><span class="sxs-lookup"><span data-stu-id="3eebf-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="3eebf-120">体系结构</span><span class="sxs-lookup"><span data-stu-id="3eebf-120">Architecture</span></span>

<span data-ttu-id="3eebf-121">该体系结构包括以下组件。</span><span class="sxs-lookup"><span data-stu-id="3eebf-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="3eebf-122">**本地网络**。</span><span class="sxs-lookup"><span data-stu-id="3eebf-122">**On-premises network**.</span></span> <span data-ttu-id="3eebf-123">在组织内运行的一个专用局域网。</span><span class="sxs-lookup"><span data-stu-id="3eebf-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="3eebf-124">**VPN 设备**。</span><span class="sxs-lookup"><span data-stu-id="3eebf-124">**VPN device**.</span></span> <span data-ttu-id="3eebf-125">提供到本地网络的外部连接的设备或服务。</span><span class="sxs-lookup"><span data-stu-id="3eebf-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="3eebf-126">VPN 设备可以是硬件设备，也可以是软件解决方案，例如 Windows Server 2012 中的路由和远程访问服务 (RRAS)。</span><span class="sxs-lookup"><span data-stu-id="3eebf-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="3eebf-127">有关受支持 VPN 设备的列表和有关为连接到 Azure 而配置所选 VPN 设备的信息，请参阅 [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance]（关于站点到站点 VPN 网关连接的 VPN 设备）。</span><span class="sxs-lookup"><span data-stu-id="3eebf-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="3eebf-128">**VPN 虚拟网络网关或 ExpressRoute 网关**。</span><span class="sxs-lookup"><span data-stu-id="3eebf-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="3eebf-129">虚拟网络网关可以将 VNet 连接到用于本地网络连接的 VPN 设备或 ExpressRoute 线路。</span><span class="sxs-lookup"><span data-stu-id="3eebf-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="3eebf-130">有关详细信息，请参阅[将本地网络连接到 Microsoft Azure 虚拟网络][connect-to-an-Azure-vnet]。</span><span class="sxs-lookup"><span data-stu-id="3eebf-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="3eebf-131">此参考体系结构的部署脚本使用 VPN 网关进行连接，使用 Azure 中的 VNet 来模拟本地网络。</span><span class="sxs-lookup"><span data-stu-id="3eebf-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="3eebf-132">**中心 VNet**。</span><span class="sxs-lookup"><span data-stu-id="3eebf-132">**Hub VNet**.</span></span> <span data-ttu-id="3eebf-133">用作中心辐射型拓扑中的中心的 Azure VNet。</span><span class="sxs-lookup"><span data-stu-id="3eebf-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="3eebf-134">中心是到本地网络的连接的中心点，它还托管着可以由辐射 VNet 中托管的各种工作负荷使用的服务。</span><span class="sxs-lookup"><span data-stu-id="3eebf-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="3eebf-135">**网关子网**。</span><span class="sxs-lookup"><span data-stu-id="3eebf-135">**Gateway subnet**.</span></span> <span data-ttu-id="3eebf-136">各个虚拟网络网关都放在同一子网中。</span><span class="sxs-lookup"><span data-stu-id="3eebf-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="3eebf-137">**共享服务子网**。</span><span class="sxs-lookup"><span data-stu-id="3eebf-137">**Shared services subnet**.</span></span> <span data-ttu-id="3eebf-138">中心 VNet 中的子网，用来托管可以在所有辐射之间共享的服务，例如 DNS 或 AD DS。</span><span class="sxs-lookup"><span data-stu-id="3eebf-138">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

* <span data-ttu-id="3eebf-139">**辐射 VNet**。</span><span class="sxs-lookup"><span data-stu-id="3eebf-139">**Spoke VNets**.</span></span> <span data-ttu-id="3eebf-140">用作中心辐射型拓扑中的辐射的一个或多个 Azure VNet。</span><span class="sxs-lookup"><span data-stu-id="3eebf-140">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="3eebf-141">辐射可以用来隔离其自己的 VNet 中的工作负荷，独立于其他辐射进行管理。</span><span class="sxs-lookup"><span data-stu-id="3eebf-141">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="3eebf-142">每个工作负荷可以包括多个层，并具有通过 Azure 负载均衡器连接的多个子网。</span><span class="sxs-lookup"><span data-stu-id="3eebf-142">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="3eebf-143">有关应用程序基础结构的详细信息，请参阅[运行 Windows VM 工作负荷][windows-vm-ra]和[运行 Linux VM 工作负荷][linux-vm-ra]。</span><span class="sxs-lookup"><span data-stu-id="3eebf-143">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="3eebf-144">**VNet 对等互连**。</span><span class="sxs-lookup"><span data-stu-id="3eebf-144">**VNet peering**.</span></span> <span data-ttu-id="3eebf-145">可以使用[对等互连连接][vnet-peering]来连接同一 Azure 区域中的两个 VNet。</span><span class="sxs-lookup"><span data-stu-id="3eebf-145">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="3eebf-146">对等互连连接是 VNet 之间的不可传递低延迟连接。</span><span class="sxs-lookup"><span data-stu-id="3eebf-146">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="3eebf-147">进行对等互连后，VNet 可使用 Azure 主干交换流量，不需要使用路由器。</span><span class="sxs-lookup"><span data-stu-id="3eebf-147">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="3eebf-148">在中心辐射型网络拓扑中，将使用 VNet 对等互连来将中心连接到每个辐射。</span><span class="sxs-lookup"><span data-stu-id="3eebf-148">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="3eebf-149">文本仅涵盖了[资源管理器](/azure/azure-resource-manager/resource-group-overview)部署，但也可以将经典 VNet 连接到同一订阅中的资源管理器 VNet。</span><span class="sxs-lookup"><span data-stu-id="3eebf-149">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="3eebf-150">这样，辐射将可以托管经典部署，并且仍然可以从中心内共享的各种服务受益。</span><span class="sxs-lookup"><span data-stu-id="3eebf-150">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>


## <a name="recommendations"></a><span data-ttu-id="3eebf-151">建议</span><span class="sxs-lookup"><span data-stu-id="3eebf-151">Recommendations</span></span>

<span data-ttu-id="3eebf-152">以下建议适用于大多数方案。</span><span class="sxs-lookup"><span data-stu-id="3eebf-152">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="3eebf-153">除非有优先于这些建议的特定要求，否则请遵循这些建议。</span><span class="sxs-lookup"><span data-stu-id="3eebf-153">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="3eebf-154">资源组</span><span class="sxs-lookup"><span data-stu-id="3eebf-154">Resource groups</span></span>

<span data-ttu-id="3eebf-155">中心 VNet 和每个辐射 VNet 可以在不同的资源组中实现，甚至可以在不同的订阅中实现，只要它们属于同一 Azure 区域中的同一 Azure Active Directory (Azure AD) 租户即可。</span><span class="sxs-lookup"><span data-stu-id="3eebf-155">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions, as long as they belong to the same Azure Active Directory (Azure AD) tenant in the same Azure region.</span></span> <span data-ttu-id="3eebf-156">这样，可以对各个工作负荷进行非集中管理，同时在中心 VNet 内维护共享服务。</span><span class="sxs-lookup"><span data-stu-id="3eebf-156">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="3eebf-157">VNet 和 GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="3eebf-157">VNet and GatewaySubnet</span></span>

<span data-ttu-id="3eebf-158">创建一个名为 *GatewaySubnet* 的子网，使其地址范围为 /27。</span><span class="sxs-lookup"><span data-stu-id="3eebf-158">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="3eebf-159">此子网是虚拟网络网关所必需的。</span><span class="sxs-lookup"><span data-stu-id="3eebf-159">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="3eebf-160">向此子网分配 32 个地址将有助于防止将来达到网关大小限制。</span><span class="sxs-lookup"><span data-stu-id="3eebf-160">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="3eebf-161">有关设置网关的详细信息，请根据你的连接类型参阅以下参考体系结构：</span><span class="sxs-lookup"><span data-stu-id="3eebf-161">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="3eebf-162">[使用 ExpressRoute 的混合网络][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="3eebf-162">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="3eebf-163">[使用 VPN 网关的混合网络][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="3eebf-163">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="3eebf-164">要实现更高的可用性，可以将 ExpressRoute 外加 VPN 用于故障转移。</span><span class="sxs-lookup"><span data-stu-id="3eebf-164">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="3eebf-165">请参阅[将本地网络连接到 Azure 并将 ExpressRoute 和 VPN 用于故障转移][hybrid-ha]。</span><span class="sxs-lookup"><span data-stu-id="3eebf-165">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="3eebf-166">如果不需要与本地网络的连接，还可以在不使用网关的情况下使用中心辐射型拓扑。</span><span class="sxs-lookup"><span data-stu-id="3eebf-166">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="3eebf-167">VNet 对等互连</span><span class="sxs-lookup"><span data-stu-id="3eebf-167">VNet peering</span></span>

<span data-ttu-id="3eebf-168">VNet 对等互连是两个 VNet 之间的不可传递关系。</span><span class="sxs-lookup"><span data-stu-id="3eebf-168">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="3eebf-169">如果需要将各个辐射彼此连接，请考虑在这些辐射之间添加一个单独的对等互连连接。</span><span class="sxs-lookup"><span data-stu-id="3eebf-169">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="3eebf-170">不过，如果有多个辐射需要彼此连接，则你可能会由于[每个 VNet 的 VNet 对等互连数限制][vnet-peering-limit]而很快耗尽可能的对等互连连接。</span><span class="sxs-lookup"><span data-stu-id="3eebf-170">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="3eebf-171">在这种情况下，请考虑使用用户定义的路由 (UDR) 强制将目的地为辐射的流量发送到在中心 VNet 中充当路由器的 NVA。</span><span class="sxs-lookup"><span data-stu-id="3eebf-171">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="3eebf-172">这将允许各个辐射彼此连接。</span><span class="sxs-lookup"><span data-stu-id="3eebf-172">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="3eebf-173">还可以将辐射配置为使用中心 VNet 网关与远程网络进行通信。</span><span class="sxs-lookup"><span data-stu-id="3eebf-173">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="3eebf-174">若要允许网关流量从辐射流动到中心，以及允许连接到远程网络，必须：</span><span class="sxs-lookup"><span data-stu-id="3eebf-174">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="3eebf-175">在中心内配置 VNet 对等互连连接以**允许网关中转**。</span><span class="sxs-lookup"><span data-stu-id="3eebf-175">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="3eebf-176">在每个辐射中配置 VNet 对等互连连接以**使用远程网关**。</span><span class="sxs-lookup"><span data-stu-id="3eebf-176">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="3eebf-177">配置所有 VNet 对等互连连接以**允许转发的流量**。</span><span class="sxs-lookup"><span data-stu-id="3eebf-177">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="3eebf-178">注意事项</span><span class="sxs-lookup"><span data-stu-id="3eebf-178">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="3eebf-179">辐射连接</span><span class="sxs-lookup"><span data-stu-id="3eebf-179">Spoke connectivity</span></span>

<span data-ttu-id="3eebf-180">如果辐射之间需要存在连接，请考虑在中心内实现一个用于路由的 NVA，并在辐射中使用 UDR 将流量转发到中心。</span><span class="sxs-lookup"><span data-stu-id="3eebf-180">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="3eebf-181">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="3eebf-181">![[2]][2]</span></span>

<span data-ttu-id="3eebf-182">在这种情况下，必须配置对等互连连接以**允许转发的流量**。</span><span class="sxs-lookup"><span data-stu-id="3eebf-182">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="3eebf-183">克服 VNet 对等互连限制</span><span class="sxs-lookup"><span data-stu-id="3eebf-183">Overcoming VNet peering limits</span></span>

<span data-ttu-id="3eebf-184">请务必考虑 Azure 中[每个 VNet 的 VNet 对等互连数限制][vnet-peering-limit]。</span><span class="sxs-lookup"><span data-stu-id="3eebf-184">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="3eebf-185">如果你确定所需的辐射多于限制将允许的数量，请考虑创建一个中心-辐射-中心-辐射型拓扑，其中的第一级辐射还充当中心。</span><span class="sxs-lookup"><span data-stu-id="3eebf-185">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="3eebf-186">下图显示了此方式。</span><span class="sxs-lookup"><span data-stu-id="3eebf-186">The following diagram shows this approach.</span></span>

<span data-ttu-id="3eebf-187">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="3eebf-187">![[3]][3]</span></span>

<span data-ttu-id="3eebf-188">另外，请考虑要在中心内共享哪些服务，以确保中心能够针对大量辐射进行缩放。</span><span class="sxs-lookup"><span data-stu-id="3eebf-188">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="3eebf-189">例如，如果中心提供防火墙服务，则在添加多个辐射时请考虑防火墙解决方案的带宽限制。</span><span class="sxs-lookup"><span data-stu-id="3eebf-189">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="3eebf-190">你可能希望将这些共享服务中的某一些移动到二级中心内。</span><span class="sxs-lookup"><span data-stu-id="3eebf-190">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="3eebf-191">部署解决方案</span><span class="sxs-lookup"><span data-stu-id="3eebf-191">Deploy the solution</span></span>

<span data-ttu-id="3eebf-192">[GitHub][ref-arch-repo] 上提供了此体系结构的部署。</span><span class="sxs-lookup"><span data-stu-id="3eebf-192">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="3eebf-193">它使用每个 VNet 中的 Ubuntu VM 来测试连接。</span><span class="sxs-lookup"><span data-stu-id="3eebf-193">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="3eebf-194">没有实际服务托管在 **中心 VNet** 内的**共享服务**中。</span><span class="sxs-lookup"><span data-stu-id="3eebf-194">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="3eebf-195">先决条件</span><span class="sxs-lookup"><span data-stu-id="3eebf-195">Prerequisites</span></span>

<span data-ttu-id="3eebf-196">在将参考体系结构部署到自己的订阅之前，必须执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="3eebf-196">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="3eebf-197">为 [AzureCAT 参考体系结构][ref-arch-repo] GitHub 存储库克隆、下载 zip 文件或创建其分支。</span><span class="sxs-lookup"><span data-stu-id="3eebf-197">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="3eebf-198">如果希望使用 Azure CLI，请确保在计算机上安装 Azure CLI 2.0。</span><span class="sxs-lookup"><span data-stu-id="3eebf-198">If you prefer to use the Azure CLI, make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="3eebf-199">若要安装 CLI，请按照[安装 Azure CLI 2.0][azure-cli-2] 中的说明执行操作。</span><span class="sxs-lookup"><span data-stu-id="3eebf-199">To install the CLI, follow the instructions in [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="3eebf-200">如果希望使用 PowerShell，请确保在计算机上安装适用于 Azure 的最新 PowerShell 模块。</span><span class="sxs-lookup"><span data-stu-id="3eebf-200">If you prefer to use PowerShell, make sure you have the latest PowerShell module for Azure installed on you computer.</span></span> <span data-ttu-id="3eebf-201">若要安装最新 Azure PowerShell 模块，请按照 [Install PowerShell for Azure][azure-powershell]（安装适用于 Azure 的 PowerShell）中的说明进行操作。</span><span class="sxs-lookup"><span data-stu-id="3eebf-201">To install the latest Azure PowerShell module, follow the instructions in [Install PowerShell for Azure][azure-powershell].</span></span>

4. <span data-ttu-id="3eebf-202">从命令提示符、bash 提示符或 PowerShell 提示符下通过使用以下命令之一，登录到 Azure 帐户，然后按照提示进行操作。</span><span class="sxs-lookup"><span data-stu-id="3eebf-202">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using one of the commands below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

  ```powershell
  Login-AzureRmAccount
  ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="3eebf-203">部署模拟的本地数据中心</span><span class="sxs-lookup"><span data-stu-id="3eebf-203">Deploy the simulated on-premises datacenter</span></span>

<span data-ttu-id="3eebf-204">若要将模拟的本地数据中心部署为 Azure VNet，请执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="3eebf-204">To deploy the simulated on-premises datacenter as an Azure VNet, perform the following steps.</span></span>

1. <span data-ttu-id="3eebf-205">导航到已在前面的先决条件步骤中下载的存储库的 `hybrid-networking\hub-spoke\onprem` 文件夹。</span><span class="sxs-lookup"><span data-stu-id="3eebf-205">Navigate to the `hybrid-networking\hub-spoke\onprem` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="3eebf-206">打开 `onprem.vm.parameters.json` 文件，在第 11 行和第 12 行中，输入括在引号中的用户名和密码，如下所示，然后保存文件。</span><span class="sxs-lookup"><span data-stu-id="3eebf-206">Open the `onprem.vm.parameters.json` file and enter a username and password between the quotes in line 11 and 12, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="3eebf-207">运行下面的 bash 或 PowerShell 命令来将模拟的本地环境部署为 Azure 中的 VNet。</span><span class="sxs-lookup"><span data-stu-id="3eebf-207">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="3eebf-208">将各个值替换为你的订阅、资源组名称和 Azure 区域。</span><span class="sxs-lookup"><span data-stu-id="3eebf-208">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./onprem.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > <span data-ttu-id="3eebf-209">如果决定使用其他资源组名称（而非 `ra-onprem-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。</span><span class="sxs-lookup"><span data-stu-id="3eebf-209">If you decide to use a different resource group name (other than `ra-onprem-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="3eebf-210">等待部署完成。</span><span class="sxs-lookup"><span data-stu-id="3eebf-210">Wait for the deployment to finish.</span></span> <span data-ttu-id="3eebf-211">此部署将创建一个虚拟网络、一个运行 Ubuntu 的虚拟机和一个 VPN 网关。</span><span class="sxs-lookup"><span data-stu-id="3eebf-211">This deployment creates a virtual network, a virtual machine running Ubuntu, and a VPN gateway.</span></span> <span data-ttu-id="3eebf-212">VPN 网关创建可能需要 40 多分钟才能完成。</span><span class="sxs-lookup"><span data-stu-id="3eebf-212">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="3eebf-213">Azure 中心 VNet</span><span class="sxs-lookup"><span data-stu-id="3eebf-213">Azure hub VNet</span></span>

<span data-ttu-id="3eebf-214">若要部署中心 VNet 并连接到前面创建的模拟的本地 VNet，请执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="3eebf-214">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="3eebf-215">导航到已在前面的先决条件步骤中下载的存储库的 `hybrid-networking\hub-spoke\hub` 文件夹。</span><span class="sxs-lookup"><span data-stu-id="3eebf-215">Navigate to the `hybrid-networking\hub-spoke\hub` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="3eebf-216">打开 `hub.vm.parameters.json` 文件，在第 11 行和第 12 行中，输入括在引号中的用户名和密码，如下所示，然后保存文件。</span><span class="sxs-lookup"><span data-stu-id="3eebf-216">Open the `hub.vm.parameters.json` file and enter a username and password between the quotes in line 11 and 12, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="3eebf-217">打开 `hub.gateway.parameters.json` 文件，在第 23 行中，输入括在引号中的共享密钥，如下所示，然后保存文件。</span><span class="sxs-lookup"><span data-stu-id="3eebf-217">Open the `hub.gateway.parameters.json` file and enter a shared key between the quotes in line 23, as shown below, then save the file.</span></span> <span data-ttu-id="3eebf-218">请记下此值，稍后在部署中需要使用此值。</span><span class="sxs-lookup"><span data-stu-id="3eebf-218">Keep a note of this value, you will need to use it later in the deployment.</span></span>

  ```bash
  "sharedKey": "",
  ```

4. <span data-ttu-id="3eebf-219">运行下面的 bash 或 PowerShell 命令来将模拟的本地环境部署为 Azure 中的 VNet。</span><span class="sxs-lookup"><span data-stu-id="3eebf-219">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="3eebf-220">将各个值替换为你的订阅、资源组名称和 Azure 区域。</span><span class="sxs-lookup"><span data-stu-id="3eebf-220">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./hub.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus
  ```

  ```powershell
  ./hub.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus
  ```
  > [!NOTE]
  > <span data-ttu-id="3eebf-221">如果决定使用其他资源组名称（而非 `ra-hub-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。</span><span class="sxs-lookup"><span data-stu-id="3eebf-221">If you decide to use a different resource group name (other than `ra-hub-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="3eebf-222">等待部署完成。</span><span class="sxs-lookup"><span data-stu-id="3eebf-222">Wait for the deployment to finish.</span></span> <span data-ttu-id="3eebf-223">此部署将创建一个虚拟网络、一个运行 Ubuntu 的虚拟机、一个 VPN 网关和到上一部分中创建的网关的连接。</span><span class="sxs-lookup"><span data-stu-id="3eebf-223">This deployment creates a virtual network, a virtual machine running Ubuntu, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="3eebf-224">VPN 网关创建可能需要 40 多分钟才能完成。</span><span class="sxs-lookup"><span data-stu-id="3eebf-224">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="connection-from-on-premises-to-the-hub"></a><span data-ttu-id="3eebf-225">从本地到中心的连接</span><span class="sxs-lookup"><span data-stu-id="3eebf-225">Connection from on-premises to the hub</span></span>

<span data-ttu-id="3eebf-226">若要从模拟的本地数据中心连接到中心 VNet，请执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="3eebf-226">To connect from the simulated on-premises datacenter to the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="3eebf-227">导航到已在前面的先决条件步骤中下载的存储库的 `hybrid-networking\hub-spoke\onprem` 文件夹。</span><span class="sxs-lookup"><span data-stu-id="3eebf-227">Navigate to the `hybrid-networking\hub-spoke\onprem` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="3eebf-228">打开 `onprem.connection.parameters.json` 文件，在第 9 行中，输入括在引号中的共享密钥，如下所示，然后保存文件。</span><span class="sxs-lookup"><span data-stu-id="3eebf-228">Open the `onprem.connection.parameters.json` file and enter a shared key between the quotes in line 9, as shown below, then save the file.</span></span> <span data-ttu-id="3eebf-229">此共享密钥值必须与之前部署的本地网关中使用的共享密钥值相同。</span><span class="sxs-lookup"><span data-stu-id="3eebf-229">This shared key value must be the same used in the on-premises gateway you deployed previously.</span></span>

  ```bash
  "sharedKey": "",
  ```

3. <span data-ttu-id="3eebf-230">运行下面的 bash 或 PowerShell 命令来将模拟的本地环境部署为 Azure 中的 VNet。</span><span class="sxs-lookup"><span data-stu-id="3eebf-230">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="3eebf-231">将各个值替换为你的订阅、资源组名称和 Azure 区域。</span><span class="sxs-lookup"><span data-stu-id="3eebf-231">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./onprem.connection.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.connection.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > <span data-ttu-id="3eebf-232">如果决定使用其他资源组名称（而非 `ra-onprem-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。</span><span class="sxs-lookup"><span data-stu-id="3eebf-232">If you decide to use a different resource group name (other than `ra-onprem-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="3eebf-233">等待部署完成。</span><span class="sxs-lookup"><span data-stu-id="3eebf-233">Wait for the deployment to finish.</span></span> <span data-ttu-id="3eebf-234">此部署将在用来模拟本地数据中心的 VNet 与中心 VNet 之间创建一个连接。</span><span class="sxs-lookup"><span data-stu-id="3eebf-234">This deployment creates a connection between the VNet used to simulate an on-premises datacenter, and the hub VNet.</span></span>

### <a name="azure-spoke-vnets"></a><span data-ttu-id="3eebf-235">Azure 辐射 VNet</span><span class="sxs-lookup"><span data-stu-id="3eebf-235">Azure spoke VNets</span></span>

<span data-ttu-id="3eebf-236">若要部署辐射 VNet 并连接到前面创建的中心 VNet，请执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="3eebf-236">To deploy the spoke VNets, and connect to the hub VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="3eebf-237">切换到已在前面的先决条件步骤中下载的存储库的 `hybrid-networking\hub-spoke\spokes` 文件夹。</span><span class="sxs-lookup"><span data-stu-id="3eebf-237">Switch to the `hybrid-networking\hub-spoke\spokes` folder for the repository you download in the pre-requisites step above.</span></span>

2. <span data-ttu-id="3eebf-238">打开 `spoke1.web.parameters.json` 文件，在第 53 行和第 54 行中，输入括在引号中的用户名和密码，如下所示，然后保存文件。</span><span class="sxs-lookup"><span data-stu-id="3eebf-238">Open the `spoke1.web.parameters.json` file and enter a username and password between the quotes in line 53 and 54, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="3eebf-239">针对文件 `spoke2.web.parameters.json` 重复上面的步骤。</span><span class="sxs-lookup"><span data-stu-id="3eebf-239">Repeat the previous step for file `spoke2.web.parameters.json`.</span></span>

4. <span data-ttu-id="3eebf-240">运行下面的 bash 或 PowerShell 命令来部署第一个辐射并将其连接到中心。</span><span class="sxs-lookup"><span data-stu-id="3eebf-240">Run the bash or PowerShell command below to deploy the first spoke and connect it to the hub.</span></span> <span data-ttu-id="3eebf-241">将各个值替换为你的订阅、资源组名称和 Azure 区域。</span><span class="sxs-lookup"><span data-stu-id="3eebf-241">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```
  > [!NOTE]
  > <span data-ttu-id="3eebf-242">如果决定使用其他资源组名称（而非 `ra-spoke1-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。</span><span class="sxs-lookup"><span data-stu-id="3eebf-242">If you decide to use a different resource group name (other than `ra-spoke1-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="3eebf-243">等待部署完成。</span><span class="sxs-lookup"><span data-stu-id="3eebf-243">Wait for the deployment to finish.</span></span> <span data-ttu-id="3eebf-244">此部署将创建一个虚拟网络、一个包含运行 Ubuntu 和 Apache 的三个虚拟机的负载均衡器，以及到前面部分中创建的中心 VNet 的 VNet 对等互连连接。</span><span class="sxs-lookup"><span data-stu-id="3eebf-244">This deployment creates a virtual network, a load balancer with three virtual machine running Ubuntu and Apache, and a VNet peering connection to the hub VNet created in the previous section.</span></span> <span data-ttu-id="3eebf-245">此部署可能需要花费 20 多分钟。</span><span class="sxs-lookup"><span data-stu-id="3eebf-245">This deployment may take over 20 minutes.</span></span>

6. <span data-ttu-id="3eebf-246">运行下面的 bash 或 PowerShell 命令来部署第一个辐射并将其连接到中心。</span><span class="sxs-lookup"><span data-stu-id="3eebf-246">Run the bash or PowerShell command below to deploy the first spoke and connect it to the hub.</span></span> <span data-ttu-id="3eebf-247">将各个值替换为你的订阅、资源组名称和 Azure 区域。</span><span class="sxs-lookup"><span data-stu-id="3eebf-247">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```
  > [!NOTE]
  > <span data-ttu-id="3eebf-248">如果决定使用其他资源组名称（而非 `ra-spoke2-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。</span><span class="sxs-lookup"><span data-stu-id="3eebf-248">If you decide to use a different resource group name (other than `ra-spoke2-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="3eebf-249">等待部署完成。</span><span class="sxs-lookup"><span data-stu-id="3eebf-249">Wait for the deployment to finish.</span></span> <span data-ttu-id="3eebf-250">此部署将创建一个虚拟网络、一个包含运行 Ubuntu 和 Apache 的三个虚拟机的负载均衡器，以及到前面部分中创建的中心 VNet 的 VNet 对等互连连接。</span><span class="sxs-lookup"><span data-stu-id="3eebf-250">This deployment creates a virtual network, a load balancer with three virtual machine running Ubuntu and Apache, and a VNet peering connection to the hub VNet created in the previous section.</span></span> <span data-ttu-id="3eebf-251">此部署可能需要花费 20 多分钟。</span><span class="sxs-lookup"><span data-stu-id="3eebf-251">This deployment may take over 20 minutes.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="3eebf-252">Azure 中心 VNet 到辐射 VNet 的对等互连</span><span class="sxs-lookup"><span data-stu-id="3eebf-252">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="3eebf-253">若要部署中心 VNet 的 VNet 对等互连连接，请执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="3eebf-253">To deploy the VNet peering connections for the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="3eebf-254">切换到已在前面的先决条件步骤中下载的存储库的 `hybrid-networking\hub-spoke\hub` 文件夹。</span><span class="sxs-lookup"><span data-stu-id="3eebf-254">Switch to the `hybrid-networking\hub-spoke\hub` folder for the repository you download in the pre-requisites step above.</span></span>

2. <span data-ttu-id="3eebf-255">运行下面的 bash 或 PowerShell 命令来部署到第一个辐射的对等互连连接。</span><span class="sxs-lookup"><span data-stu-id="3eebf-255">Run the bash or PowerShell command below to deploy the peering connection to the first spoke.</span></span> <span data-ttu-id="3eebf-256">将各个值替换为你的订阅、资源组名称和 Azure 区域。</span><span class="sxs-lookup"><span data-stu-id="3eebf-256">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 1
  ```

2. <span data-ttu-id="3eebf-257">运行下面的 bash 或 PowerShell 命令来部署到第二个辐射的对等互连连接。</span><span class="sxs-lookup"><span data-stu-id="3eebf-257">Run the bash or PowerShell command below to deploy the peering connection to the second spoke.</span></span> <span data-ttu-id="3eebf-258">将各个值替换为你的订阅、资源组名称和 Azure 区域。</span><span class="sxs-lookup"><span data-stu-id="3eebf-258">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 2
  ```

### <a name="test-connectivity"></a><span data-ttu-id="3eebf-259">测试连接</span><span class="sxs-lookup"><span data-stu-id="3eebf-259">Test connectivity</span></span>

<span data-ttu-id="3eebf-260">若要验证连接到本地数据中心部署的中心辐射型拓扑是否正常工作，请执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="3eebf-260">To verify that the hub-spoke topology connected to an on-premises datacenter deployment worked, follow these steps.</span></span>

1. <span data-ttu-id="3eebf-261">从 [Azure 门户] [门户] 中，连接到你的订阅，并导航到 `ra-onprem-rg` 资源组中的 `ra-onprem-vm1` 虚拟机。</span><span class="sxs-lookup"><span data-stu-id="3eebf-261">From the [Azure portal][portal], connect to your subscription, and navigate to the `ra-onprem-vm1` virtual machine in the `ra-onprem-rg` resource group.</span></span>

2. <span data-ttu-id="3eebf-262">在 `Overview` 边栏选项卡中，记下 VM 的 `Public IP address`。</span><span class="sxs-lookup"><span data-stu-id="3eebf-262">In the `Overview` blade, note the `Public IP address` for the VM.</span></span>

3. <span data-ttu-id="3eebf-263">使用 SSH 客户端并使用在部署期间指定的用户名和密码登录到在上面记下的 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="3eebf-263">Use an SSH client to connect to the IP address you noted above using the user name and password you specified during deployment.</span></span>

4. <span data-ttu-id="3eebf-264">在连接到的 VM 上，从命令提示符下运行以下命令来测试从本地 VNet 到 Spoke1 VNet 的连接。</span><span class="sxs-lookup"><span data-stu-id="3eebf-264">From the command prompt on the VM you connected to, run the command below to test connectivity from the on-premises VNet to the Spoke1 VNet.</span></span>

  ```bash
  ping 10.1.1.37
  ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="3eebf-265">添加辐射之间的连接</span><span class="sxs-lookup"><span data-stu-id="3eebf-265">Add connectivity between spokes</span></span>

<span data-ttu-id="3eebf-266">如果要允许各个辐射连接到彼此，必须为每个辐射部署 UDR，用以将以其他辐射为目的地的流量转发到中心 VNet 中的网关。</span><span class="sxs-lookup"><span data-stu-id="3eebf-266">If you want to allow spokes to connect to each other, you must deploy UDRs to each spoke that forward traffic destined to other spokes to the gateway in the hub VNet.</span></span> <span data-ttu-id="3eebf-267">执行以下步骤来验证当前是否能够从一个辐射连接到另一个辐射，然后部署 UDR 并再次测试连接。</span><span class="sxs-lookup"><span data-stu-id="3eebf-267">Perform the following steps to verify that currently you are not able to connect from a spoke to another, then deploy the UDRs and test connectivity again.</span></span>

1. <span data-ttu-id="3eebf-268">如果不再连接到 jumpbox VM，请重复上面的步骤 1 到 4。</span><span class="sxs-lookup"><span data-stu-id="3eebf-268">Repeat steps 1 to 4 above, if you are not connected to the jumpbox VM any longer.</span></span>

2. <span data-ttu-id="3eebf-269">连接到 spoke 1 中的 Web 服务器之一。</span><span class="sxs-lookup"><span data-stu-id="3eebf-269">Connect to one of the web servers in spoke 1.</span></span>

  ```bash
  ssh 10.1.1.37
  ```

3. <span data-ttu-id="3eebf-270">测试 spoke 1 与 spoke 2 之间的连接。</span><span class="sxs-lookup"><span data-stu-id="3eebf-270">Test the connectivity between spoke 1 and spoke 2.</span></span> <span data-ttu-id="3eebf-271">它应当会失败。</span><span class="sxs-lookup"><span data-stu-id="3eebf-271">It should fail.</span></span>

  ```bash
  ping 10.1.2.37
  ```

4. <span data-ttu-id="3eebf-272">切换回你的计算机的命令提示符。</span><span class="sxs-lookup"><span data-stu-id="3eebf-272">Switch back to your computer's command prompt.</span></span>

5. <span data-ttu-id="3eebf-273">切换到已在前面的先决条件步骤中下载的存储库的 `hybrid-networking\hub-spoke\spokes` 文件夹。</span><span class="sxs-lookup"><span data-stu-id="3eebf-273">Switch to the `hybrid-networking\hub-spoke\spokes` folder for the repository you downloaded in the pre-requisites step above.</span></span>

6. <span data-ttu-id="3eebf-274">运行下面的 bash 或 PowerShell 命令来为第一个辐射部署 UDR。</span><span class="sxs-lookup"><span data-stu-id="3eebf-274">Run the bash or PowerShell command below to deploy an UDR to the first spoke.</span></span> <span data-ttu-id="3eebf-275">将各个值替换为你的订阅、资源组名称和 Azure 区域。</span><span class="sxs-lookup"><span data-stu-id="3eebf-275">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.udr.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.udr.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```

7. <span data-ttu-id="3eebf-276">运行下面的 bash 或 PowerShell 命令来为第二个辐射部署 UDR。</span><span class="sxs-lookup"><span data-stu-id="3eebf-276">Run the bash or PowerShell command below to deploy an UDR to the second spoke.</span></span> <span data-ttu-id="3eebf-277">将各个值替换为你的订阅、资源组名称和 Azure 区域。</span><span class="sxs-lookup"><span data-stu-id="3eebf-277">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.udr.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.udr.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```

8. <span data-ttu-id="3eebf-278">切换回 ssh 终端。</span><span class="sxs-lookup"><span data-stu-id="3eebf-278">Switch back to the ssh terminal.</span></span>

9. <span data-ttu-id="3eebf-279">测试 spoke 1 与 spoke 2 之间的连接。</span><span class="sxs-lookup"><span data-stu-id="3eebf-279">Test the connectivity between spoke 1 and spoke 2.</span></span> <span data-ttu-id="3eebf-280">它应当会成功。</span><span class="sxs-lookup"><span data-stu-id="3eebf-280">It should succeed.</span></span>

  ```bash
  ping 10.1.2.37
  ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azure-powershell]: /powershell/azure/install-azure-ps?view=azuresmps-3.7.0
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

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Azure 中的中心辐射型拓扑"
[1]: ./images/hub-spoke-gateway-routing.svg "Azure 中的具有可传递路由的中心辐射型拓扑"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Azure 中的具有使用 NVA 的可传递路由的中心辐射型拓扑"
[3]: ./images/hub-spokehub-spoke.svg "Azure 中的中心-辐射-中心-辐射型拓扑"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
