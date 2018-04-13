---
title: 在 Azure 中实现中心辐射型网络拓扑
description: 如何在 Azure 中实现中心辐射型网络拓扑。
author: telmosampaio
ms.date: 02/23/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: 243ad026c7c9703d9659cbef6815131fcdaa8a11
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/06/2018
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="8b52c-103">在 Azure 中实现中心辐射型网络拓扑</span><span class="sxs-lookup"><span data-stu-id="8b52c-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="8b52c-104">此参考体系结构展示了如何在 Azure 中实现中心辐射型拓扑。</span><span class="sxs-lookup"><span data-stu-id="8b52c-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="8b52c-105">*中心*是 Azure 中的一个虚拟网络 (VNet)，充当到本地网络的连接的中心点。</span><span class="sxs-lookup"><span data-stu-id="8b52c-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="8b52c-106">*辐射*是与中心对等互连的 VNet，可用于隔离工作负荷。</span><span class="sxs-lookup"><span data-stu-id="8b52c-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="8b52c-107">流量通过 ExpressRoute 或 VPN 网关连接在本地数据中心与中心之间流动。</span><span class="sxs-lookup"><span data-stu-id="8b52c-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="8b52c-108">[**部署此解决方案**](#deploy-the-solution)。</span><span class="sxs-lookup"><span data-stu-id="8b52c-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="8b52c-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="8b52c-109">![[0]][0]</span></span>

<span data-ttu-id="8b52c-110">*下载此体系结构的 [Visio 文件][visio-download]*</span><span class="sxs-lookup"><span data-stu-id="8b52c-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="8b52c-111">此拓扑的好处包括：</span><span class="sxs-lookup"><span data-stu-id="8b52c-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="8b52c-112">**节省成本** - 通过将可以由多个工作负荷（例如网络虚拟设备 (NVAs) 和 DNS 服务器）共享的服务集中放置在单个位置中。</span><span class="sxs-lookup"><span data-stu-id="8b52c-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="8b52c-113">**克服订阅限制** - 通过将不同订阅中的 Vnet 对等互连到中心。</span><span class="sxs-lookup"><span data-stu-id="8b52c-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="8b52c-114">**关注点隔离**（在中心 IT（SecOps、InfraOps）与工作负荷 (DevOps) 之间）。</span><span class="sxs-lookup"><span data-stu-id="8b52c-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="8b52c-115">此体系结构的典型用途包括：</span><span class="sxs-lookup"><span data-stu-id="8b52c-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="8b52c-116">在各种环境（例如开发、测试和生产）中部署的需要使用共享服务（例如 DNS、IDS、NTP 或 AD DS）的工作负荷。</span><span class="sxs-lookup"><span data-stu-id="8b52c-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="8b52c-117">共享服务放置在中心 VNet 中，而每个环境都部署到辐射以保持隔离。</span><span class="sxs-lookup"><span data-stu-id="8b52c-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="8b52c-118">不需要彼此连接但需要访问共享服务的工作负荷。</span><span class="sxs-lookup"><span data-stu-id="8b52c-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="8b52c-119">需要对安全方面进行集中控制（例如作为外围网络的中心内的防火墙），并且需要在每个辐射中对工作负荷进行隔离管理的企业。</span><span class="sxs-lookup"><span data-stu-id="8b52c-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="8b52c-120">体系结构</span><span class="sxs-lookup"><span data-stu-id="8b52c-120">Architecture</span></span>

<span data-ttu-id="8b52c-121">该体系结构包括以下组件。</span><span class="sxs-lookup"><span data-stu-id="8b52c-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="8b52c-122">**本地网络**。</span><span class="sxs-lookup"><span data-stu-id="8b52c-122">**On-premises network**.</span></span> <span data-ttu-id="8b52c-123">在组织内运行的一个专用局域网。</span><span class="sxs-lookup"><span data-stu-id="8b52c-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="8b52c-124">**VPN 设备**。</span><span class="sxs-lookup"><span data-stu-id="8b52c-124">**VPN device**.</span></span> <span data-ttu-id="8b52c-125">提供到本地网络的外部连接的设备或服务。</span><span class="sxs-lookup"><span data-stu-id="8b52c-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="8b52c-126">VPN 设备可以是硬件设备，也可以是软件解决方案，例如 Windows Server 2012 中的路由和远程访问服务 (RRAS)。</span><span class="sxs-lookup"><span data-stu-id="8b52c-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="8b52c-127">有关受支持 VPN 设备的列表和有关为连接到 Azure 而配置所选 VPN 设备的信息，请参阅 [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance]（关于站点到站点 VPN 网关连接的 VPN 设备）。</span><span class="sxs-lookup"><span data-stu-id="8b52c-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="8b52c-128">**VPN 虚拟网络网关或 ExpressRoute 网关**。</span><span class="sxs-lookup"><span data-stu-id="8b52c-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="8b52c-129">虚拟网络网关可以将 VNet 连接到用于本地网络连接的 VPN 设备或 ExpressRoute 线路。</span><span class="sxs-lookup"><span data-stu-id="8b52c-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="8b52c-130">有关详细信息，请参阅[将本地网络连接到 Microsoft Azure 虚拟网络][connect-to-an-Azure-vnet]。</span><span class="sxs-lookup"><span data-stu-id="8b52c-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="8b52c-131">此参考体系结构的部署脚本使用 VPN 网关进行连接，使用 Azure 中的 VNet 来模拟本地网络。</span><span class="sxs-lookup"><span data-stu-id="8b52c-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="8b52c-132">**中心 VNet**。</span><span class="sxs-lookup"><span data-stu-id="8b52c-132">**Hub VNet**.</span></span> <span data-ttu-id="8b52c-133">用作中心辐射型拓扑中的中心的 Azure VNet。</span><span class="sxs-lookup"><span data-stu-id="8b52c-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="8b52c-134">中心是到本地网络的连接的中心点，它还托管着可以由辐射 VNet 中托管的各种工作负荷使用的服务。</span><span class="sxs-lookup"><span data-stu-id="8b52c-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="8b52c-135">**网关子网**。</span><span class="sxs-lookup"><span data-stu-id="8b52c-135">**Gateway subnet**.</span></span> <span data-ttu-id="8b52c-136">虚拟网络网关保留在同一子网中。</span><span class="sxs-lookup"><span data-stu-id="8b52c-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="8b52c-137">**辐射 VNet**。</span><span class="sxs-lookup"><span data-stu-id="8b52c-137">**Spoke VNets**.</span></span> <span data-ttu-id="8b52c-138">用作中心辐射型拓扑中的辐射的一个或多个 Azure VNet。</span><span class="sxs-lookup"><span data-stu-id="8b52c-138">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="8b52c-139">辐射可以用来隔离其自己的 VNet 中的工作负荷，独立于其他辐射进行管理。</span><span class="sxs-lookup"><span data-stu-id="8b52c-139">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="8b52c-140">每个工作负荷可以包括多个层，并具有通过 Azure 负载均衡器连接的多个子网。</span><span class="sxs-lookup"><span data-stu-id="8b52c-140">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="8b52c-141">有关应用程序基础结构的详细信息，请参阅[运行 Windows VM 工作负荷][windows-vm-ra]和[运行 Linux VM 工作负荷][linux-vm-ra]。</span><span class="sxs-lookup"><span data-stu-id="8b52c-141">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="8b52c-142">**VNet 对等互连**。</span><span class="sxs-lookup"><span data-stu-id="8b52c-142">**VNet peering**.</span></span> <span data-ttu-id="8b52c-143">可以使用[对等互连连接][vnet-peering]来连接同一 Azure 区域中的两个 VNet。</span><span class="sxs-lookup"><span data-stu-id="8b52c-143">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="8b52c-144">对等互连连接是 VNet 之间的不可传递低延迟连接。</span><span class="sxs-lookup"><span data-stu-id="8b52c-144">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="8b52c-145">进行对等互连后，VNet 可使用 Azure 主干交换流量，不需要使用路由器。</span><span class="sxs-lookup"><span data-stu-id="8b52c-145">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="8b52c-146">在中心辐射型网络拓扑中，将使用 VNet 对等互连来将中心连接到每个辐射。</span><span class="sxs-lookup"><span data-stu-id="8b52c-146">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="8b52c-147">文本仅涵盖了[资源管理器](/azure/azure-resource-manager/resource-group-overview)部署，但也可以将经典 VNet 连接到同一订阅中的资源管理器 VNet。</span><span class="sxs-lookup"><span data-stu-id="8b52c-147">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="8b52c-148">这样，辐射将可以托管经典部署，并且仍然可以从中心内共享的各种服务受益。</span><span class="sxs-lookup"><span data-stu-id="8b52c-148">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="8b52c-149">建议</span><span class="sxs-lookup"><span data-stu-id="8b52c-149">Recommendations</span></span>

<span data-ttu-id="8b52c-150">以下建议适用于大多数方案。</span><span class="sxs-lookup"><span data-stu-id="8b52c-150">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="8b52c-151">除非有优先于这些建议的特定要求，否则请遵循这些建议。</span><span class="sxs-lookup"><span data-stu-id="8b52c-151">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="8b52c-152">资源组</span><span class="sxs-lookup"><span data-stu-id="8b52c-152">Resource groups</span></span>

<span data-ttu-id="8b52c-153">中心 VNet 和每个辐射 VNet 可以在不同的资源组中实现，甚至可以在不同的订阅中实现，只要它们属于同一 Azure 区域中的同一 Azure Active Directory (Azure AD) 租户即可。</span><span class="sxs-lookup"><span data-stu-id="8b52c-153">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions, as long as they belong to the same Azure Active Directory (Azure AD) tenant in the same Azure region.</span></span> <span data-ttu-id="8b52c-154">这样，可以对各个工作负荷进行非集中管理，同时在中心 VNet 内维护共享服务。</span><span class="sxs-lookup"><span data-stu-id="8b52c-154">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="8b52c-155">VNet 和 GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="8b52c-155">VNet and GatewaySubnet</span></span>

<span data-ttu-id="8b52c-156">创建一个名为 *GatewaySubnet* 的子网，使其地址范围为 /27。</span><span class="sxs-lookup"><span data-stu-id="8b52c-156">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="8b52c-157">此子网是虚拟网络网关所必需的。</span><span class="sxs-lookup"><span data-stu-id="8b52c-157">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="8b52c-158">向此子网分配 32 个地址将有助于防止将来达到网关大小限制。</span><span class="sxs-lookup"><span data-stu-id="8b52c-158">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="8b52c-159">有关设置网关的详细信息，请根据你的连接类型参阅以下参考体系结构：</span><span class="sxs-lookup"><span data-stu-id="8b52c-159">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="8b52c-160">[使用 ExpressRoute 的混合网络][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="8b52c-160">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="8b52c-161">[使用 VPN 网关的混合网络][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="8b52c-161">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="8b52c-162">要实现更高的可用性，可以将 ExpressRoute 外加 VPN 用于故障转移。</span><span class="sxs-lookup"><span data-stu-id="8b52c-162">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="8b52c-163">请参阅[将本地网络连接到 Azure 并将 ExpressRoute 和 VPN 用于故障转移][hybrid-ha]。</span><span class="sxs-lookup"><span data-stu-id="8b52c-163">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="8b52c-164">如果不需要与本地网络的连接，还可以在不使用网关的情况下使用中心辐射型拓扑。</span><span class="sxs-lookup"><span data-stu-id="8b52c-164">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="8b52c-165">VNet 对等互连</span><span class="sxs-lookup"><span data-stu-id="8b52c-165">VNet peering</span></span>

<span data-ttu-id="8b52c-166">VNet 对等互连是两个 VNet 之间的不可传递关系。</span><span class="sxs-lookup"><span data-stu-id="8b52c-166">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="8b52c-167">如果需要将各个辐射彼此连接，请考虑在这些辐射之间添加一个单独的对等互连连接。</span><span class="sxs-lookup"><span data-stu-id="8b52c-167">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="8b52c-168">不过，如果有多个辐射需要彼此连接，则你可能会由于[每个 VNet 的 VNet 对等互连数限制][vnet-peering-limit]而很快耗尽可能的对等互连连接。</span><span class="sxs-lookup"><span data-stu-id="8b52c-168">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="8b52c-169">在这种情况下，请考虑使用用户定义的路由 (UDR) 强制将目的地为辐射的流量发送到在中心 VNet 中充当路由器的 NVA。</span><span class="sxs-lookup"><span data-stu-id="8b52c-169">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="8b52c-170">这将允许各个辐射彼此连接。</span><span class="sxs-lookup"><span data-stu-id="8b52c-170">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="8b52c-171">还可以将辐射配置为使用中心 VNet 网关与远程网络进行通信。</span><span class="sxs-lookup"><span data-stu-id="8b52c-171">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="8b52c-172">若要允许网关流量从辐射流动到中心，以及允许连接到远程网络，必须：</span><span class="sxs-lookup"><span data-stu-id="8b52c-172">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="8b52c-173">在中心内配置 VNet 对等互连连接以**允许网关中转**。</span><span class="sxs-lookup"><span data-stu-id="8b52c-173">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="8b52c-174">在每个辐射中配置 VNet 对等互连连接以**使用远程网关**。</span><span class="sxs-lookup"><span data-stu-id="8b52c-174">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="8b52c-175">配置所有 VNet 对等互连连接以**允许转发的流量**。</span><span class="sxs-lookup"><span data-stu-id="8b52c-175">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="8b52c-176">注意事项</span><span class="sxs-lookup"><span data-stu-id="8b52c-176">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="8b52c-177">辐射连接</span><span class="sxs-lookup"><span data-stu-id="8b52c-177">Spoke connectivity</span></span>

<span data-ttu-id="8b52c-178">如果辐射之间需要存在连接，请考虑在中心内实现一个用于路由的 NVA，并在辐射中使用 UDR 将流量转发到中心。</span><span class="sxs-lookup"><span data-stu-id="8b52c-178">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="8b52c-179">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="8b52c-179">![[2]][2]</span></span>

<span data-ttu-id="8b52c-180">在这种情况下，必须配置对等互连连接以**允许转发的流量**。</span><span class="sxs-lookup"><span data-stu-id="8b52c-180">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="8b52c-181">克服 VNet 对等互连限制</span><span class="sxs-lookup"><span data-stu-id="8b52c-181">Overcoming VNet peering limits</span></span>

<span data-ttu-id="8b52c-182">请务必考虑 Azure 中[每个 VNet 的 VNet 对等互连数限制][vnet-peering-limit]。</span><span class="sxs-lookup"><span data-stu-id="8b52c-182">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="8b52c-183">如果你确定所需的辐射多于限制将允许的数量，请考虑创建一个中心-辐射-中心-辐射型拓扑，其中的第一级辐射还充当中心。</span><span class="sxs-lookup"><span data-stu-id="8b52c-183">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="8b52c-184">下图显示了此方式。</span><span class="sxs-lookup"><span data-stu-id="8b52c-184">The following diagram shows this approach.</span></span>

<span data-ttu-id="8b52c-185">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="8b52c-185">![[3]][3]</span></span>

<span data-ttu-id="8b52c-186">另外，请考虑要在中心内共享哪些服务，以确保中心能够针对大量辐射进行缩放。</span><span class="sxs-lookup"><span data-stu-id="8b52c-186">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="8b52c-187">例如，如果中心提供防火墙服务，则在添加多个辐射时请考虑防火墙解决方案的带宽限制。</span><span class="sxs-lookup"><span data-stu-id="8b52c-187">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="8b52c-188">你可能希望将这些共享服务中的某一些移动到二级中心内。</span><span class="sxs-lookup"><span data-stu-id="8b52c-188">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="8b52c-189">部署解决方案</span><span class="sxs-lookup"><span data-stu-id="8b52c-189">Deploy the solution</span></span>

<span data-ttu-id="8b52c-190">[GitHub][ref-arch-repo] 上提供了此体系结构的部署。</span><span class="sxs-lookup"><span data-stu-id="8b52c-190">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="8b52c-191">它使用每个 VNet 中的 Ubuntu VM 来测试连接。</span><span class="sxs-lookup"><span data-stu-id="8b52c-191">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="8b52c-192">没有实际服务托管在 **中心 VNet** 内的**共享服务**中。</span><span class="sxs-lookup"><span data-stu-id="8b52c-192">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="8b52c-193">先决条件</span><span class="sxs-lookup"><span data-stu-id="8b52c-193">Prerequisites</span></span>

<span data-ttu-id="8b52c-194">在将参考体系结构部署到自己的订阅之前，必须执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="8b52c-194">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="8b52c-195">克隆、下载[参考体系结构][ref-arch-repo] GitHub 存储库的 zip 文件或创建其分支。</span><span class="sxs-lookup"><span data-stu-id="8b52c-195">Clone, fork, or download the zip file for the [reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="8b52c-196">确保在计算机上安装了 Azure CLI 2.0。</span><span class="sxs-lookup"><span data-stu-id="8b52c-196">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="8b52c-197">有关 CLI 安装说明，请参阅[安装 Azure CLI 2.0][azure-cli-2]。</span><span class="sxs-lookup"><span data-stu-id="8b52c-197">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="8b52c-198">安装 [Azure 构建基块][azbb] npm 包。</span><span class="sxs-lookup"><span data-stu-id="8b52c-198">Install the [Azure buulding blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="8b52c-199">在命令提示符、bash 提示符或 PowerShell 提示符处使用以下命令登录到 Azure 帐户，然后按提示操作。</span><span class="sxs-lookup"><span data-stu-id="8b52c-199">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using the command below, and follow the prompts.</span></span>

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="8b52c-200">使用 azbb 部署模拟的本地数据中心</span><span class="sxs-lookup"><span data-stu-id="8b52c-200">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="8b52c-201">若要将模拟的本地数据中心部署为 Azure VNet，请执行以下步骤：</span><span class="sxs-lookup"><span data-stu-id="8b52c-201">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="8b52c-202">导航到已在前面的先决条件步骤中下载的存储库的 `hybrid-networking\hub-spoke\` 文件夹。</span><span class="sxs-lookup"><span data-stu-id="8b52c-202">Navigate to the `hybrid-networking\hub-spoke\` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="8b52c-203">打开 `onprem.json` 文件，在第 36 行和第 37 行中输入括在引号中的用户名和密码，如下所示，然后保存文件。</span><span class="sxs-lookup"><span data-stu-id="8b52c-203">Open the `onprem.json` file and enter a username and password between the quotes in line 36 and 37, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

3. <span data-ttu-id="8b52c-204">在第 38 行中键入 `Windows` 或 `Linux` 作为 `osType`，以便安装 Windows Server 2016 Datacenter 或 Ubuntu 16.04 作为 Jumpbox 的操作系统。</span><span class="sxs-lookup"><span data-stu-id="8b52c-204">On line 38, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

4. <span data-ttu-id="8b52c-205">运行 `azbb` 以部署模拟的本地环境，如下所示。</span><span class="sxs-lookup"><span data-stu-id="8b52c-205">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="8b52c-206">如果决定使用其他资源组名称（而非 `onprem-vnet-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。</span><span class="sxs-lookup"><span data-stu-id="8b52c-206">If you decide to use a different resource group name (other than `onprem-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="8b52c-207">等待部署完成。</span><span class="sxs-lookup"><span data-stu-id="8b52c-207">Wait for the deployment to finish.</span></span> <span data-ttu-id="8b52c-208">此部署创建虚拟网络、虚拟机和 VPN 网关。</span><span class="sxs-lookup"><span data-stu-id="8b52c-208">This deployment creates a virtual network, a virtual machine, and a VPN gateway.</span></span> <span data-ttu-id="8b52c-209">VPN 网关创建可能需要 40 多分钟才能完成。</span><span class="sxs-lookup"><span data-stu-id="8b52c-209">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="8b52c-210">Azure 中心 VNet</span><span class="sxs-lookup"><span data-stu-id="8b52c-210">Azure hub VNet</span></span>

<span data-ttu-id="8b52c-211">若要部署中心 VNet 并连接到前面创建的模拟的本地 VNet，请执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="8b52c-211">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="8b52c-212">打开 `hub-vnet.json` 文件，在第 39 行和第 40 行中输入括在引号中的用户名和密码，如下所示。</span><span class="sxs-lookup"><span data-stu-id="8b52c-212">Open the `hub-vnet.json` file and enter a username and password between the quotes in line 39 and 40, as shown below.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. <span data-ttu-id="8b52c-213">在第 41 行中键入 `Windows` 或 `Linux` 作为 `osType`，以便安装 Windows Server 2016 Datacenter 或 Ubuntu 16.04 作为 Jumpbox 的操作系统。</span><span class="sxs-lookup"><span data-stu-id="8b52c-213">On line 41, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="8b52c-214">在第 72 行中输入括在引号中的共享密钥，如下所示，然后保存文件。</span><span class="sxs-lookup"><span data-stu-id="8b52c-214">Enter a shared key between the quotes in line 72, as shown below, then save the file.</span></span>

   ```bash
   "sharedKey": "",
   ```

4. <span data-ttu-id="8b52c-215">运行 `azbb` 以部署模拟的本地环境，如下所示。</span><span class="sxs-lookup"><span data-stu-id="8b52c-215">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="8b52c-216">如果决定使用其他资源组名称（而非 `hub-vnet-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。</span><span class="sxs-lookup"><span data-stu-id="8b52c-216">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="8b52c-217">等待部署完成。</span><span class="sxs-lookup"><span data-stu-id="8b52c-217">Wait for the deployment to finish.</span></span> <span data-ttu-id="8b52c-218">此部署创建虚拟网络、虚拟机、VPN 网关和到上一部分创建的网关的连接。</span><span class="sxs-lookup"><span data-stu-id="8b52c-218">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="8b52c-219">VPN 网关创建可能需要 40 多分钟才能完成。</span><span class="sxs-lookup"><span data-stu-id="8b52c-219">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="optional-test-connectivity-from-onprem-to-hub"></a><span data-ttu-id="8b52c-220">（可选）测试从本地到中心的连接</span><span class="sxs-lookup"><span data-stu-id="8b52c-220">(Optional) Test connectivity from onprem to hub</span></span>

<span data-ttu-id="8b52c-221">若要使用 Windows VM 测试从模拟的本地环境到中心 VNet 的连接，请执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="8b52c-221">To test conectivity from the simulated on-premises environment to the hub VNet using Windows VMs, perform the following steps.</span></span>

1. <span data-ttu-id="8b52c-222">从 Azure 门户导航到 `onprem-jb-rg` 资源组，然后单击 `jb-vm1` 虚拟机资源。</span><span class="sxs-lookup"><span data-stu-id="8b52c-222">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2. <span data-ttu-id="8b52c-223">在门户中的 VM 边栏选项卡的左上角单击 `Connect`，然后按提示使用远程桌面连接到 VM。</span><span class="sxs-lookup"><span data-stu-id="8b52c-223">On the top left hand corner of your VM blade in the portal, click `Connect`, and follow the prompts to use remote desktop to connect to the VM.</span></span> <span data-ttu-id="8b52c-224">确保使用在 `onprem.json` 文件的第 36 行和 37 行中指定的用户名和密码。</span><span class="sxs-lookup"><span data-stu-id="8b52c-224">Make sure to use the username and password you specified in lines 36 and 37 in the `onprem.json` file.</span></span>

3. <span data-ttu-id="8b52c-225">在 VM 中打开 PowerShell 控制台，使用 `Test-NetConnection` cmdlet 验证能否连接到中心 Jumpbox VM，如下所示。</span><span class="sxs-lookup"><span data-stu-id="8b52c-225">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the hub jumpbox VM as shown below.</span></span>

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```
   > [!NOTE]
   > <span data-ttu-id="8b52c-226">默认情况下，Windows Server VM 不允许 Azure 中的 ICMP 响应。</span><span class="sxs-lookup"><span data-stu-id="8b52c-226">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="8b52c-227">若要使用 `ping` 来测试连接，需在“Windows 高级防火墙”中为每个 VM 启用 ICMP 流量。</span><span class="sxs-lookup"><span data-stu-id="8b52c-227">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="8b52c-228">若要使用 Linux VM 测试从模拟的本地环境到中心 VNet 的连接，请执行以下步骤：</span><span class="sxs-lookup"><span data-stu-id="8b52c-228">To test conectivity from the simulated on-premises environment to the hub VNet using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="8b52c-229">从 Azure 门户导航到 `onprem-jb-rg` 资源组，然后单击 `jb-vm1` 虚拟机资源。</span><span class="sxs-lookup"><span data-stu-id="8b52c-229">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2. <span data-ttu-id="8b52c-230">在门户的 VM 边栏选项卡的左上角单击`Connect`，然后复制门户中显示的 `ssh` 命令。</span><span class="sxs-lookup"><span data-stu-id="8b52c-230">On the top left hand corner of your VM blade in the portal, click `Connect`, and then copy the `ssh` command shown on the portal.</span></span> 

3. <span data-ttu-id="8b52c-231">在 Linux 提示符处运行 `ssh`，以便使用在上面的步骤 2 中复制的信息连接到模拟的本地环境 Jumpbox，如下所示。</span><span class="sxs-lookup"><span data-stu-id="8b52c-231">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment jumpbox witht the information you copied in step 2 above, as shown below.</span></span>

   ```bash
   ssh <your_user>@<public_ip_address>
   ```

4. <span data-ttu-id="8b52c-232">使用在 `onprem.json` 文件的第 37 行中指定的密码连接到 VM。</span><span class="sxs-lookup"><span data-stu-id="8b52c-232">Use the password you specified in line 37 in the `onprem.json` file to the connect to the VM.</span></span>

5. <span data-ttu-id="8b52c-233">使用 `ping` 命令测试到中心 Jumpbox 的连接，如下所示。</span><span class="sxs-lookup"><span data-stu-id="8b52c-233">Use the `ping` command to test connectivity to the hub jumpbox, as shown below.</span></span>

   ```bash
   ping 10.0.0.68
   ```

### <a name="azure-spoke-vnets"></a><span data-ttu-id="8b52c-234">Azure 辐射 VNet</span><span class="sxs-lookup"><span data-stu-id="8b52c-234">Azure spoke VNets</span></span>

<span data-ttu-id="8b52c-235">若要部署辐射 VNet，请执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="8b52c-235">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="8b52c-236">打开 `spoke1.json` 文件，在第 47 行和第 48 行中输入括在引号中的用户名和密码，如下所示，然后保存文件。</span><span class="sxs-lookup"><span data-stu-id="8b52c-236">Open the `spoke1.json` file and enter a username and password between the quotes in lines 47 and 48, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. <span data-ttu-id="8b52c-237">在第 49 行中键入 `Windows` 或 `Linux` 作为 `osType`，以便安装 Windows Server 2016 Datacenter 或 Ubuntu 16.04 作为 Jumpbox 的操作系统。</span><span class="sxs-lookup"><span data-stu-id="8b52c-237">On line 49, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="8b52c-238">运行 `azbb` 以部署第一个辐射 VNet 环境，如下所示。</span><span class="sxs-lookup"><span data-stu-id="8b52c-238">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg - l <location> -p spoke1.json --deploy
   ```
  
   > [!NOTE]
   > <span data-ttu-id="8b52c-239">如果决定使用其他资源组名称（而非 `spoke1-vnet-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。</span><span class="sxs-lookup"><span data-stu-id="8b52c-239">If you decide to use a different resource group name (other than `spoke1-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="8b52c-240">对文件 `spoke2.json` 重复上面的步骤 1。</span><span class="sxs-lookup"><span data-stu-id="8b52c-240">Repeat step 1 above for file `spoke2.json`.</span></span>

5. <span data-ttu-id="8b52c-241">运行 `azbb` 以部署第二个辐射 VNet 环境，如下所示。</span><span class="sxs-lookup"><span data-stu-id="8b52c-241">Run `azbb` to deploy the second spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg - l <location> -p spoke2.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="8b52c-242">如果决定使用其他资源组名称（而非 `spoke2-vnet-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。</span><span class="sxs-lookup"><span data-stu-id="8b52c-242">If you decide to use a different resource group name (other than `spoke2-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="8b52c-243">Azure 中心 VNet 到辐射 VNet 的对等互连</span><span class="sxs-lookup"><span data-stu-id="8b52c-243">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="8b52c-244">若要创建从中心 VNet 到辐射 VNet 的对等互连连接，请执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="8b52c-244">To create a peering connection from the hub VNet to the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="8b52c-245">打开 `hub-vnet-peering.json` 文件，验证资源组名称以及在第 29 行中开始的每个虚拟网络对等互连的虚拟网络名称是否正确。</span><span class="sxs-lookup"><span data-stu-id="8b52c-245">Open the `hub-vnet-peering.json` file and verify that the resource group name, and virtual network name for each of the virtual network peerings starting in line 29 are correct.</span></span>

2. <span data-ttu-id="8b52c-246">运行 `azbb` 以部署第一个辐射 VNet 环境，如下所示。</span><span class="sxs-lookup"><span data-stu-id="8b52c-246">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet-peering.json --deploy
   ```

   > [!NOTE]
   > <span data-ttu-id="8b52c-247">如果决定使用其他资源组名称（而非 `hub-vnet-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。</span><span class="sxs-lookup"><span data-stu-id="8b52c-247">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="test-connectivity"></a><span data-ttu-id="8b52c-248">测试连接</span><span class="sxs-lookup"><span data-stu-id="8b52c-248">Test connectivity</span></span>

<span data-ttu-id="8b52c-249">若要使用 Windows VM 测试从模拟的本地环境到辐射 VNet 的连接，请执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="8b52c-249">To test conectivity from the simulated on-premises environment to the spoke VNets using Windows VMs, perform the following steps.</span></span>

1. <span data-ttu-id="8b52c-250">从 Azure 门户导航到 `onprem-jb-rg` 资源组，然后单击 `jb-vm1` 虚拟机资源。</span><span class="sxs-lookup"><span data-stu-id="8b52c-250">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2. <span data-ttu-id="8b52c-251">在门户中的 VM 边栏选项卡的左上角单击 `Connect`，然后按提示使用远程桌面连接到 VM。</span><span class="sxs-lookup"><span data-stu-id="8b52c-251">On the top left hand corner of your VM blade in the portal, click `Connect`, and follow the prompts to use remote desktop to connect to the VM.</span></span> <span data-ttu-id="8b52c-252">确保使用在 `onprem.json` 文件的第 36 行和 37 行中指定的用户名和密码。</span><span class="sxs-lookup"><span data-stu-id="8b52c-252">Make sure to use the username and password you specified in lines 36 and 37 in the `onprem.json` file.</span></span>

3. <span data-ttu-id="8b52c-253">在 VM 中打开 PowerShell 控制台，使用 `Test-NetConnection` cmdlet 验证能否连接到中心 Jumpbox VM，如下所示。</span><span class="sxs-lookup"><span data-stu-id="8b52c-253">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the hub jumpbox VM as shown below.</span></span>

   ```powershell
   Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
   Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
   ```

<span data-ttu-id="8b52c-254">若要使用 Linux VM 测试从模拟的本地环境到辐射 VNet 的连接，请执行以下步骤：</span><span class="sxs-lookup"><span data-stu-id="8b52c-254">To test conectivity from the simulated on-premises environment to the spoke VNets using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="8b52c-255">从 Azure 门户导航到 `onprem-jb-rg` 资源组，然后单击 `jb-vm1` 虚拟机资源。</span><span class="sxs-lookup"><span data-stu-id="8b52c-255">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2. <span data-ttu-id="8b52c-256">在门户的 VM 边栏选项卡的左上角单击`Connect`，然后复制门户中显示的 `ssh` 命令。</span><span class="sxs-lookup"><span data-stu-id="8b52c-256">On the top left hand corner of your VM blade in the portal, click `Connect`, and then copy the `ssh` command shown on the portal.</span></span> 

3. <span data-ttu-id="8b52c-257">在 Linux 提示符处运行 `ssh`，以便使用在上面的步骤 2 中复制的信息连接到模拟的本地环境 Jumpbox，如下所示。</span><span class="sxs-lookup"><span data-stu-id="8b52c-257">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment jumpbox witht the information you copied in step 2 above, as shown below.</span></span>

   ```bash
   ssh <your_user>@<public_ip_address>
   ```

4. <span data-ttu-id="8b52c-258">使用在 `onprem.json` 文件的第 37 行中指定的密码连接到 VM。</span><span class="sxs-lookup"><span data-stu-id="8b52c-258">Use the password you specified in line 37 in the `onprem.json` file to the connect to the VM.</span></span>

5. <span data-ttu-id="8b52c-259">使用 `ping` 命令测试到每个辐射 VNet 中的 Jumpbox VM 的连接，如下所示。</span><span class="sxs-lookup"><span data-stu-id="8b52c-259">Use the `ping` command to test connectivity to the jumpbox VMs in each spoke, as shown below.</span></span>

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="8b52c-260">添加辐射之间的连接</span><span class="sxs-lookup"><span data-stu-id="8b52c-260">Add connectivity between spokes</span></span>

<span data-ttu-id="8b52c-261">如果需要允许辐射 VNet 互相进行连接，则需使用网络虚拟设备 (NVA) 作为中心虚拟网络中的路由器，并在尝试连接到另一辐射 VNet 时强制流量从辐射 VNet 流向路由器。</span><span class="sxs-lookup"><span data-stu-id="8b52c-261">If you want to allow spokes to connect to each other, you need to use a newtwork virtual appliance (NVA) as a router in the hub virtual netowrk, and force traffic from spokes to the router when trying to connect to another spoke.</span></span> <span data-ttu-id="8b52c-262">若要将基本的示例 NVA 部署为单个 VM 并部署必需的用户定义的路由，以便让这两个辐射 VNet 进行连接，请执行以下步骤：</span><span class="sxs-lookup"><span data-stu-id="8b52c-262">To deploy a basic sample NVA as a single VM, and the necessary uder defined routes to allow the two spoke VNets to connect, perform the following steps:</span></span>

1. <span data-ttu-id="8b52c-263">打开 `hub-nva.json` 文件，在第 13 行和第 14 行中输入括在引号中的用户名和密码，如下所示，然后保存文件。</span><span class="sxs-lookup"><span data-stu-id="8b52c-263">Open the `hub-nva.json` file and enter a username and password between the quotes in lines 13 and 14, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```
2. <span data-ttu-id="8b52c-264">运行 `azbb`，以便部署 NVA VM 和用户定义的路由。</span><span class="sxs-lookup"><span data-stu-id="8b52c-264">Run `azbb` to deploy the NVA VM and user defined routes.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg - l <location> -p hub-nva.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="8b52c-265">如果决定使用其他资源组名称（而非 `hub-nva-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。</span><span class="sxs-lookup"><span data-stu-id="8b52c-265">If you decide to use a different resource group name (other than `hub-nva-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

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
