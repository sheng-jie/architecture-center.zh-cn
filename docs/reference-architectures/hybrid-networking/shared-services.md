---
title: "在 Azure 中使用共享服务实现中心辐射型网络拓扑"
description: "如何在 Azure 中使用共享服务实现中心辐射型网络拓扑。"
author: telmosampaio
ms.date: 02/25/2018
pnp.series.title: Implement a hub-spoke network topology with shared services in Azure
pnp.series.prev: hub-spoke
ms.openlocfilehash: c0fb1d1ddd7c70ed914d58e7c73b10475b91aedf
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/08/2018
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a><span data-ttu-id="2d773-103">在 Azure 中使用共享服务实现中心辐射型网络拓扑</span><span class="sxs-lookup"><span data-stu-id="2d773-103">Implement a hub-spoke network topology with shared services in Azure</span></span>

<span data-ttu-id="2d773-104">此参考体系结构在[中心辐射型][guidance-hub-spoke]参考体系结构的基础上生成，在中心包括了可供所有辐射 VNet 使用的共享服务。</span><span class="sxs-lookup"><span data-stu-id="2d773-104">This reference architecture builds on the [hub-spoke][guidance-hub-spoke] reference architecture to include shared services in the hub that can be consumed by all spokes.</span></span> <span data-ttu-id="2d773-105">首先需要共享第一批服务，即标识和安全性，然后才能将数据中心迁移到云并生成[虚拟数据中心]。</span><span class="sxs-lookup"><span data-stu-id="2d773-105">As a first step toward migrating a datacenter to the cloud, and building a [virtual datacenter], the first services you need to share are identity and security.</span></span> <span data-ttu-id="2d773-106">此参考体系结构显示了如何将 Active Directory 服务从本地数据中心扩展到 Azure，以及如何在中心辐射型拓扑中添加可以充当防火墙的网络虚拟设备 (NVA)。</span><span class="sxs-lookup"><span data-stu-id="2d773-106">This reference archiecture shows you how to extend your Active Directory services from your on-premises datacenter to Azure, and how to add a network virtual appliance (NVA) that can act as a firewall, in a hub-spoke topology.</span></span>  <span data-ttu-id="2d773-107">[**部署此解决方案**](#deploy-the-solution)。</span><span class="sxs-lookup"><span data-stu-id="2d773-107">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="2d773-108">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="2d773-108">![[0]][0]</span></span>

<span data-ttu-id="2d773-109">*下载此体系结构的 [Visio 文件][visio-download]*</span><span class="sxs-lookup"><span data-stu-id="2d773-109">*Download a [Visio file][visio-download] of this architecture*</span></span>

<span data-ttu-id="2d773-110">此拓扑的好处包括：</span><span class="sxs-lookup"><span data-stu-id="2d773-110">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="2d773-111">**节省成本** - 通过将可以由多个工作负荷（例如网络虚拟设备 (NVAs) 和 DNS 服务器）共享的服务集中放置在单个位置中。</span><span class="sxs-lookup"><span data-stu-id="2d773-111">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="2d773-112">**克服订阅限制** - 通过将不同订阅中的 Vnet 对等互连到中心。</span><span class="sxs-lookup"><span data-stu-id="2d773-112">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="2d773-113">**关注点隔离**（在中心 IT（SecOps、InfraOps）与工作负荷 (DevOps) 之间）。</span><span class="sxs-lookup"><span data-stu-id="2d773-113">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="2d773-114">此体系结构的典型用途包括：</span><span class="sxs-lookup"><span data-stu-id="2d773-114">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="2d773-115">在各种环境（例如开发、测试和生产）中部署的需要使用共享服务（例如 DNS、IDS、NTP 或 AD DS）的工作负荷。</span><span class="sxs-lookup"><span data-stu-id="2d773-115">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="2d773-116">共享服务放置在中心 VNet 中，而每个环境都部署到辐射以保持隔离。</span><span class="sxs-lookup"><span data-stu-id="2d773-116">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="2d773-117">不需要彼此连接但需要访问共享服务的工作负荷。</span><span class="sxs-lookup"><span data-stu-id="2d773-117">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="2d773-118">需要对安全方面进行集中控制（例如作为外围网络的中心内的防火墙），并且需要在每个辐射中对工作负荷进行隔离管理的企业。</span><span class="sxs-lookup"><span data-stu-id="2d773-118">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="2d773-119">体系结构</span><span class="sxs-lookup"><span data-stu-id="2d773-119">Architecture</span></span>

<span data-ttu-id="2d773-120">该体系结构包括以下组件。</span><span class="sxs-lookup"><span data-stu-id="2d773-120">The architecture consists of the following components.</span></span>

* <span data-ttu-id="2d773-121">**本地网络**。</span><span class="sxs-lookup"><span data-stu-id="2d773-121">**On-premises network**.</span></span> <span data-ttu-id="2d773-122">在组织内运行的一个专用局域网。</span><span class="sxs-lookup"><span data-stu-id="2d773-122">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="2d773-123">**VPN 设备**。</span><span class="sxs-lookup"><span data-stu-id="2d773-123">**VPN device**.</span></span> <span data-ttu-id="2d773-124">提供到本地网络的外部连接的设备或服务。</span><span class="sxs-lookup"><span data-stu-id="2d773-124">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="2d773-125">VPN 设备可以是硬件设备，也可以是软件解决方案，例如 Windows Server 2012 中的路由和远程访问服务 (RRAS)。</span><span class="sxs-lookup"><span data-stu-id="2d773-125">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="2d773-126">有关受支持 VPN 设备的列表和有关为连接到 Azure 而配置所选 VPN 设备的信息，请参阅 [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance]（关于站点到站点 VPN 网关连接的 VPN 设备）。</span><span class="sxs-lookup"><span data-stu-id="2d773-126">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="2d773-127">**VPN 虚拟网络网关或 ExpressRoute 网关**。</span><span class="sxs-lookup"><span data-stu-id="2d773-127">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="2d773-128">虚拟网络网关可以将 VNet 连接到用于本地网络连接的 VPN 设备或 ExpressRoute 线路。</span><span class="sxs-lookup"><span data-stu-id="2d773-128">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="2d773-129">有关详细信息，请参阅[将本地网络连接到 Microsoft Azure 虚拟网络][connect-to-an-Azure-vnet]。</span><span class="sxs-lookup"><span data-stu-id="2d773-129">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="2d773-130">此参考体系结构的部署脚本使用 VPN 网关进行连接，使用 Azure 中的 VNet 来模拟本地网络。</span><span class="sxs-lookup"><span data-stu-id="2d773-130">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="2d773-131">**中心 VNet**。</span><span class="sxs-lookup"><span data-stu-id="2d773-131">**Hub VNet**.</span></span> <span data-ttu-id="2d773-132">用作中心辐射型拓扑中的中心的 Azure VNet。</span><span class="sxs-lookup"><span data-stu-id="2d773-132">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="2d773-133">中心是到本地网络的连接的中心点，它还托管着可以由辐射 VNet 中托管的各种工作负荷使用的服务。</span><span class="sxs-lookup"><span data-stu-id="2d773-133">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="2d773-134">**网关子网**。</span><span class="sxs-lookup"><span data-stu-id="2d773-134">**Gateway subnet**.</span></span> <span data-ttu-id="2d773-135">各个虚拟网络网关都放在同一子网中。</span><span class="sxs-lookup"><span data-stu-id="2d773-135">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="2d773-136">**共享服务子网**。</span><span class="sxs-lookup"><span data-stu-id="2d773-136">**Shared services subnet**.</span></span> <span data-ttu-id="2d773-137">中心 VNet 中的子网，用来托管可以在所有辐射之间共享的服务，例如 DNS 或 AD DS。</span><span class="sxs-lookup"><span data-stu-id="2d773-137">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

* <span data-ttu-id="2d773-138">**外围网络子网**。</span><span class="sxs-lookup"><span data-stu-id="2d773-138">**DMZ subnet**.</span></span> <span data-ttu-id="2d773-139">中心 VNet 中的子网，用于托管的 NVA 可以充当安全设备，例如防火墙。</span><span class="sxs-lookup"><span data-stu-id="2d773-139">A subnet in the hub VNet used to host NVAs that can act as security appliances, such as firewalls.</span></span>

* <span data-ttu-id="2d773-140">**辐射 VNet**。</span><span class="sxs-lookup"><span data-stu-id="2d773-140">**Spoke VNets**.</span></span> <span data-ttu-id="2d773-141">用作中心辐射型拓扑中的辐射的一个或多个 Azure VNet。</span><span class="sxs-lookup"><span data-stu-id="2d773-141">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="2d773-142">辐射可以用来隔离其自己的 VNet 中的工作负荷，独立于其他辐射进行管理。</span><span class="sxs-lookup"><span data-stu-id="2d773-142">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="2d773-143">每个工作负荷可以包括多个层，并具有通过 Azure 负载均衡器连接的多个子网。</span><span class="sxs-lookup"><span data-stu-id="2d773-143">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="2d773-144">有关应用程序基础结构的详细信息，请参阅[运行 Windows VM 工作负荷][windows-vm-ra]和[运行 Linux VM 工作负荷][linux-vm-ra]。</span><span class="sxs-lookup"><span data-stu-id="2d773-144">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="2d773-145">**VNet 对等互连**。</span><span class="sxs-lookup"><span data-stu-id="2d773-145">**VNet peering**.</span></span> <span data-ttu-id="2d773-146">可以使用[对等互连连接][vnet-peering]来连接同一 Azure 区域中的两个 VNet。</span><span class="sxs-lookup"><span data-stu-id="2d773-146">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="2d773-147">对等互连连接是 VNet 之间的不可传递低延迟连接。</span><span class="sxs-lookup"><span data-stu-id="2d773-147">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="2d773-148">进行对等互连后，VNet 可使用 Azure 主干交换流量，不需要使用路由器。</span><span class="sxs-lookup"><span data-stu-id="2d773-148">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="2d773-149">在中心辐射型网络拓扑中，将使用 VNet 对等互连来将中心连接到每个辐射。</span><span class="sxs-lookup"><span data-stu-id="2d773-149">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="2d773-150">文本仅涵盖了[资源管理器](/azure/azure-resource-manager/resource-group-overview)部署，但也可以将经典 VNet 连接到同一订阅中的资源管理器 VNet。</span><span class="sxs-lookup"><span data-stu-id="2d773-150">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="2d773-151">这样，辐射将可以托管经典部署，并且仍然可以从中心内共享的各种服务受益。</span><span class="sxs-lookup"><span data-stu-id="2d773-151">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="2d773-152">建议</span><span class="sxs-lookup"><span data-stu-id="2d773-152">Recommendations</span></span>

<span data-ttu-id="2d773-153">针对[中心辐射型][guidance-hub-spoke]参考体系结构的所有建议也适用于共享服务参考体系结构。</span><span class="sxs-lookup"><span data-stu-id="2d773-153">All the recommendations for the [hub-spoke][guidance-hub-spoke] reference architecture also apply to the shared services reference architecture.</span></span> 

<span data-ttu-id="2d773-154">另外，以下建议适用于共享服务中的大多数方案。</span><span class="sxs-lookup"><span data-stu-id="2d773-154">ALso, the following recommendations apply for most scenarios under shared services.</span></span> <span data-ttu-id="2d773-155">除非有优先于这些建议的特定要求，否则请遵循这些建议。</span><span class="sxs-lookup"><span data-stu-id="2d773-155">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="identity"></a><span data-ttu-id="2d773-156">标识</span><span class="sxs-lookup"><span data-stu-id="2d773-156">Identity</span></span>

<span data-ttu-id="2d773-157">大多数企业组织在其本地数据中心都有 Active Directory 目录服务 (ADDS) 环境。</span><span class="sxs-lookup"><span data-stu-id="2d773-157">Most enterprise organizations have an Active Directory Directory Services (ADDS) environment in their on-premises datacenter.</span></span> <span data-ttu-id="2d773-158">为了便于管理从本地网络移到 Azure 且依赖于 ADDS 的资产，建议将 ADDS 域控制器托管在 Azure 中。</span><span class="sxs-lookup"><span data-stu-id="2d773-158">To facilitate management of assets moved to Azure from your on-premises network that depend on ADDS, it is recommended to host ADDS domain controllers in Azure.</span></span>

<span data-ttu-id="2d773-159">如果使用需要针对 Azure 和本地环境分开进行控制的组策略对象，请针对每个 Azure 区域使用不同的 AD 站点。</span><span class="sxs-lookup"><span data-stu-id="2d773-159">If you make use of Group Policy Objects, that you want to control separately for Azure and your on-premises environment, use a different AD site for each Azure region.</span></span> <span data-ttu-id="2d773-160">将域控制器置于可供依赖性工作负荷访问的中心 VNet（简称“中心”）。</span><span class="sxs-lookup"><span data-stu-id="2d773-160">Place your domain controllers in a central VNet (hub) that dependent workloads can access.</span></span>

### <a name="security"></a><span data-ttu-id="2d773-161">“安全”</span><span class="sxs-lookup"><span data-stu-id="2d773-161">Security</span></span>

<span data-ttu-id="2d773-162">将工作负荷从本地环境移到 Azure 时，其中的某些工作负荷将需要托管在 VM 中。</span><span class="sxs-lookup"><span data-stu-id="2d773-162">As you move workloads from your on-premises environment to Azure, some of these workloads will require to be hosted in VMs.</span></span> <span data-ttu-id="2d773-163">出于符合性考虑，可能需要对流经这些工作负荷的流量强制实施限制。</span><span class="sxs-lookup"><span data-stu-id="2d773-163">For compliance reasons, you may need to enforce restrictions on traffic traversing those workloads.</span></span> 

<span data-ttu-id="2d773-164">可以在 Azure 中使用网络虚拟设备 (NVA) 来托管各类安全和性能服务。</span><span class="sxs-lookup"><span data-stu-id="2d773-164">You can use network virtual appliances (NVAs) in Azure to host different types of security and performance services.</span></span> <span data-ttu-id="2d773-165">如果熟悉当前在本地使用的特定设备组，建议在 Azure 中也使用相同的虚拟化设备，如果适用的话。</span><span class="sxs-lookup"><span data-stu-id="2d773-165">If you are familiar with a given set of appliances on-premises today, it is recommended to use the same virtualized appliances in Azure, where applicable.</span></span>

> [!NOTE]
> <span data-ttu-id="2d773-166">针对此参考体系结构的部署脚本使用已启用 IP 转发功能的 Ubuntu VM，以便模拟网络虚拟设备。</span><span class="sxs-lookup"><span data-stu-id="2d773-166">The deployment scripts for this reference architecture use an Ubuntu VM with IP forwarding enbaled to mimic a network virtual appliance.</span></span>

## <a name="considerations"></a><span data-ttu-id="2d773-167">注意事项</span><span class="sxs-lookup"><span data-stu-id="2d773-167">Considerations</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="2d773-168">克服 VNet 对等互连限制</span><span class="sxs-lookup"><span data-stu-id="2d773-168">Overcoming VNet peering limits</span></span>

<span data-ttu-id="2d773-169">请务必考虑 Azure 中[每个 VNet 的 VNet 对等互连数限制][vnet-peering-limit]。</span><span class="sxs-lookup"><span data-stu-id="2d773-169">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="2d773-170">如果你确定所需的辐射多于限制将允许的数量，请考虑创建一个中心-辐射-中心-辐射型拓扑，其中的第一级辐射还充当中心。</span><span class="sxs-lookup"><span data-stu-id="2d773-170">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="2d773-171">下图显示了此方式。</span><span class="sxs-lookup"><span data-stu-id="2d773-171">The following diagram shows this approach.</span></span>

<span data-ttu-id="2d773-172">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="2d773-172">![[3]][3]</span></span>

<span data-ttu-id="2d773-173">另外，请考虑要在中心内共享哪些服务，以确保中心能够针对大量辐射进行缩放。</span><span class="sxs-lookup"><span data-stu-id="2d773-173">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="2d773-174">例如，如果中心提供防火墙服务，则在添加多个辐射时请考虑防火墙解决方案的带宽限制。</span><span class="sxs-lookup"><span data-stu-id="2d773-174">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="2d773-175">你可能希望将这些共享服务中的某一些移动到二级中心内。</span><span class="sxs-lookup"><span data-stu-id="2d773-175">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="2d773-176">部署解决方案</span><span class="sxs-lookup"><span data-stu-id="2d773-176">Deploy the solution</span></span>

<span data-ttu-id="2d773-177">[GitHub][ref-arch-repo] 上提供了此体系结构的部署。</span><span class="sxs-lookup"><span data-stu-id="2d773-177">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="2d773-178">它使用每个 VNet 中的 Ubuntu VM 来测试连接。</span><span class="sxs-lookup"><span data-stu-id="2d773-178">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="2d773-179">没有实际服务托管在 **中心 VNet** 内的**共享服务**中。</span><span class="sxs-lookup"><span data-stu-id="2d773-179">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="2d773-180">先决条件</span><span class="sxs-lookup"><span data-stu-id="2d773-180">Prerequisites</span></span>

<span data-ttu-id="2d773-181">在将参考体系结构部署到自己的订阅之前，必须执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="2d773-181">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="2d773-182">为 [AzureCAT 参考体系结构][ref-arch-repo] GitHub 存储库克隆、下载 zip 文件或创建其分支。</span><span class="sxs-lookup"><span data-stu-id="2d773-182">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="2d773-183">确保在计算机上安装了 Azure CLI 2.0。</span><span class="sxs-lookup"><span data-stu-id="2d773-183">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="2d773-184">有关 CLI 安装说明，请参阅[安装 Azure CLI 2.0][azure-cli-2]。</span><span class="sxs-lookup"><span data-stu-id="2d773-184">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="2d773-185">安装 [Azure 构建基块][azbb] npm 包。</span><span class="sxs-lookup"><span data-stu-id="2d773-185">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="2d773-186">在命令提示符、bash 提示符或 PowerShell 提示符处使用以下命令登录到 Azure 帐户，然后按提示操作。</span><span class="sxs-lookup"><span data-stu-id="2d773-186">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using the command below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="2d773-187">使用 azbb 部署模拟的本地数据中心</span><span class="sxs-lookup"><span data-stu-id="2d773-187">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="2d773-188">若要将模拟的本地数据中心部署为 Azure VNet，请执行以下步骤：</span><span class="sxs-lookup"><span data-stu-id="2d773-188">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="2d773-189">导航到已在前面的先决条件步骤中下载的存储库的 `hybrid-networking\shared-services-stack\` 文件夹。</span><span class="sxs-lookup"><span data-stu-id="2d773-189">Navigate to the `hybrid-networking\shared-services-stack\` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="2d773-190">打开 `onprem.json` 文件，在第 45 行和第 46 行中输入括在引号中的用户名和密码，如下所示，然后保存文件。</span><span class="sxs-lookup"><span data-stu-id="2d773-190">Open the `onprem.json` file and enter a username and password between the quotes in line 45 and 46, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="2d773-191">运行 `azbb` 以部署模拟的本地环境，如下所示。</span><span class="sxs-lookup"><span data-stu-id="2d773-191">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="2d773-192">如果决定使用其他资源组名称（而非 `onprem-vnet-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。</span><span class="sxs-lookup"><span data-stu-id="2d773-192">If you decide to use a different resource group name (other than `onprem-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="2d773-193">等待部署完成。</span><span class="sxs-lookup"><span data-stu-id="2d773-193">Wait for the deployment to finish.</span></span> <span data-ttu-id="2d773-194">此部署创建虚拟网络、运行 Windows 的虚拟机和 VPN 网关。</span><span class="sxs-lookup"><span data-stu-id="2d773-194">This deployment creates a virtual network, a virtual machine running Windows, and a VPN gateway.</span></span> <span data-ttu-id="2d773-195">VPN 网关创建可能需要 40 多分钟才能完成。</span><span class="sxs-lookup"><span data-stu-id="2d773-195">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="2d773-196">Azure 中心 VNet</span><span class="sxs-lookup"><span data-stu-id="2d773-196">Azure hub VNet</span></span>

<span data-ttu-id="2d773-197">若要部署中心 VNet 并连接到前面创建的模拟的本地 VNet，请执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="2d773-197">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="2d773-198">打开 `hub-vnet.json` 文件，在第 50 行和第 51 行中输入括在引号中的用户名和密码，如下所示。</span><span class="sxs-lookup"><span data-stu-id="2d773-198">Open the `hub-vnet.json` file and enter a username and password between the quotes in line 50 and 51, as shown below.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. <span data-ttu-id="2d773-199">在第 52 行中键入 `Windows` 或 `Linux` 作为 `osType`，以便安装 Windows Server 2016 Datacenter 或 Ubuntu 16.04 作为 Jumpbox 的操作系统。</span><span class="sxs-lookup"><span data-stu-id="2d773-199">On line 52, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="2d773-200">在第 83 行中输入括在引号中的共享密钥，如下所示，然后保存文件。</span><span class="sxs-lookup"><span data-stu-id="2d773-200">Enter a shared key between the quotes in line 83, as shown below, then save the file.</span></span>

  ```bash
  "sharedKey": "",
  ```

4. <span data-ttu-id="2d773-201">运行 `azbb` 以部署模拟的本地环境，如下所示。</span><span class="sxs-lookup"><span data-stu-id="2d773-201">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="2d773-202">如果决定使用其他资源组名称（而非 `hub-vnet-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。</span><span class="sxs-lookup"><span data-stu-id="2d773-202">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="2d773-203">等待部署完成。</span><span class="sxs-lookup"><span data-stu-id="2d773-203">Wait for the deployment to finish.</span></span> <span data-ttu-id="2d773-204">此部署创建虚拟网络、虚拟机、VPN 网关和到上一部分创建的网关的连接。</span><span class="sxs-lookup"><span data-stu-id="2d773-204">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="2d773-205">VPN 网关创建可能需要 40 多分钟才能完成。</span><span class="sxs-lookup"><span data-stu-id="2d773-205">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="adds-in-azure"></a><span data-ttu-id="2d773-206">Azure 中的 ADDS</span><span class="sxs-lookup"><span data-stu-id="2d773-206">ADDS in Azure</span></span>

<span data-ttu-id="2d773-207">若要在 Azure 中部署 ADDS 域控制器，请执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="2d773-207">To deploy the ADDS domain controllers in Azure, perform the following steps.</span></span>

1. <span data-ttu-id="2d773-208">打开 `hub-adds.json` 文件，在第 14 行和第 15 行中输入括在引号中的用户名和密码，如下所示，然后保存文件。</span><span class="sxs-lookup"><span data-stu-id="2d773-208">Open the `hub-adds.json` file and enter a username and password between the quotes in lines 14 and 15, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. <span data-ttu-id="2d773-209">运行 `azbb`，以便部署 ADDS 域控制器，如下所示。</span><span class="sxs-lookup"><span data-stu-id="2d773-209">Run `azbb` to deploy the ADDS domain controllers as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-adds-rg - l <location> -p hub-adds.json --deploy
  ```
  
  > [!NOTE]
  > <span data-ttu-id="2d773-210">如果决定使用其他资源组名称（而非 `hub-adds-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。</span><span class="sxs-lookup"><span data-stu-id="2d773-210">If you decide to use a different resource group name (other than `hub-adds-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

  > [!NOTE]
  > <span data-ttu-id="2d773-211">这部分的部署可能需要数分钟的时间，因为需要将两个 VM 加入到域中，而该域托管在模拟的本地数据中心，然后需要在其上安装 AD DS。</span><span class="sxs-lookup"><span data-stu-id="2d773-211">This part of the deployment may take several minutes, since it requires joining the two VMs to the domain hosted int he simulated on-premises datacenter, then installing AD DS on them.</span></span>

### <a name="nva"></a><span data-ttu-id="2d773-212">NVA</span><span class="sxs-lookup"><span data-stu-id="2d773-212">NVA</span></span>

<span data-ttu-id="2d773-213">若要在 `dmz` 子网中部署 NVA，请执行以下步骤：</span><span class="sxs-lookup"><span data-stu-id="2d773-213">To deploy an NVA in the `dmz` subnet, perform the following steps:</span></span>

1. <span data-ttu-id="2d773-214">打开 `hub-nva.json` 文件，在第 13 行和第 14 行中输入括在引号中的用户名和密码，如下所示，然后保存文件。</span><span class="sxs-lookup"><span data-stu-id="2d773-214">Open the `hub-nva.json` file and enter a username and password between the quotes in lines 13 and 14, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```
2. <span data-ttu-id="2d773-215">运行 `azbb`，以便部署 NVA VM 和用户定义的路由。</span><span class="sxs-lookup"><span data-stu-id="2d773-215">Run `azbb` to deploy the NVA VM and user defined routes.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-nva-rg - l <location> -p hub-nva.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="2d773-216">如果决定使用其他资源组名称（而非 `hub-nva-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。</span><span class="sxs-lookup"><span data-stu-id="2d773-216">If you decide to use a different resource group name (other than `hub-nva-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-spoke-vnets"></a><span data-ttu-id="2d773-217">Azure 辐射 VNet</span><span class="sxs-lookup"><span data-stu-id="2d773-217">Azure spoke VNets</span></span>

<span data-ttu-id="2d773-218">若要部署辐射 VNet，请执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="2d773-218">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="2d773-219">打开 `spoke1.json` 文件，在第 52 行和第 53 行中输入括在引号中的用户名和密码，如下所示，然后保存文件。</span><span class="sxs-lookup"><span data-stu-id="2d773-219">Open the `spoke1.json` file and enter a username and password between the quotes in lines 52 and 53, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. <span data-ttu-id="2d773-220">在第 54 行中键入 `Windows` 或 `Linux` 作为 `osType`，以便安装 Windows Server 2016 Datacenter 或 Ubuntu 16.04 作为 Jumpbox 的操作系统。</span><span class="sxs-lookup"><span data-stu-id="2d773-220">On line 54, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="2d773-221">运行 `azbb` 以部署第一个辐射 VNet 环境，如下所示。</span><span class="sxs-lookup"><span data-stu-id="2d773-221">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g spoke1-vnet-rg - l <location> -p spoke1.json --deploy
  ```
  
  > [!NOTE]
  > <span data-ttu-id="2d773-222">如果决定使用其他资源组名称（而非 `spoke1-vnet-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。</span><span class="sxs-lookup"><span data-stu-id="2d773-222">If you decide to use a different resource group name (other than `spoke1-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

3. <span data-ttu-id="2d773-223">对文件 `spoke2.json` 重复上面的步骤 1。</span><span class="sxs-lookup"><span data-stu-id="2d773-223">Repeat step 1 above for file `spoke2.json`.</span></span>

4. <span data-ttu-id="2d773-224">运行 `azbb` 以部署第二个辐射 VNet 环境，如下所示。</span><span class="sxs-lookup"><span data-stu-id="2d773-224">Run `azbb` to deploy the second spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g spoke2-vnet-rg - l <location> -p spoke2.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="2d773-225">如果决定使用其他资源组名称（而非 `spoke2-vnet-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。</span><span class="sxs-lookup"><span data-stu-id="2d773-225">If you decide to use a different resource group name (other than `spoke2-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="2d773-226">Azure 中心 VNet 到辐射 VNet 的对等互连</span><span class="sxs-lookup"><span data-stu-id="2d773-226">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="2d773-227">若要创建从中心 VNet 到辐射 VNet 的对等互连连接，请执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="2d773-227">To create a peering connection from the hub VNet to the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="2d773-228">打开 `hub-vnet-peering.json` 文件，验证资源组名称以及在第 29 行中开始的每个虚拟网络对等互连的虚拟网络名称是否正确。</span><span class="sxs-lookup"><span data-stu-id="2d773-228">Open the `hub-vnet-peering.json` file and verify that the resource group name, and virtual network name for each of the virtual network peerings starting in line 29 are correct.</span></span>

2. <span data-ttu-id="2d773-229">运行 `azbb` 以部署第一个辐射 VNet 环境，如下所示。</span><span class="sxs-lookup"><span data-stu-id="2d773-229">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet-peering.json --deploy
  ```

  > [!NOTE]
  > <span data-ttu-id="2d773-230">如果决定使用其他资源组名称（而非 `hub-vnet-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。</span><span class="sxs-lookup"><span data-stu-id="2d773-230">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[guidance-hub-spoke]: ./hub-spoke.md
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[虚拟数据中心]: https://aka.ms/vdc
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/shared-services.png "Azure 中的共享服务拓扑"
[3]: ./images/hub-spokehub-spoke.svg "Azure 中的中心-辐射-中心-辐射型拓扑"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
