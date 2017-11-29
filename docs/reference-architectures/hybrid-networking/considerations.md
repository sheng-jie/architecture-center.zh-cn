---
title: "选择用于将本地网络连接到 Azure 的解决方案"
description: "比较用于将本地网络连接到 Azure 的参考体系结构。"
author: telmosampaio
ms.date: 04/06/2017
ms.openlocfilehash: 274b9df1817632a7f3eaafa8bf02e965fdc3feea
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a><span data-ttu-id="9e81b-103">选择用于将本地网络连接到 Azure 的解决方案</span><span class="sxs-lookup"><span data-stu-id="9e81b-103">Choose a solution for connecting an on-premises network to Azure</span></span>

<span data-ttu-id="9e81b-104">本文比较了用于将本地网络连接到 Azure 虚拟网络 (VNet) 的选项。</span><span class="sxs-lookup"><span data-stu-id="9e81b-104">This article compares options for connecting an on-premises network to an Azure Virtual Network (VNet).</span></span> <span data-ttu-id="9e81b-105">我们为每个选项提供了参考体系结构和可部署的解决方案。</span><span class="sxs-lookup"><span data-stu-id="9e81b-105">We provide a reference architecture and a deployable solution for each option.</span></span>

## <a name="vpn-connection"></a><span data-ttu-id="9e81b-106">VPN 连接</span><span class="sxs-lookup"><span data-stu-id="9e81b-106">VPN connection</span></span>

<span data-ttu-id="9e81b-107">使用虚拟专用网络 (VPN) 可通过 IPSec VPN 隧道将本地网络与 Azure VNet 进行连接。</span><span class="sxs-lookup"><span data-stu-id="9e81b-107">Use a virtual private network (VPN) to connect your on-premises network with an Azure VNet through an IPSec VPN tunnel.</span></span>

<span data-ttu-id="9e81b-108">此体系结构适合满足以下条件的混合应用程序：本地硬件与云之间的流量可能较小，或者用户愿意用略微延长的延迟换得云的灵活性和处理能力。</span><span class="sxs-lookup"><span data-stu-id="9e81b-108">This architecture is suitable for hybrid applications where the traffic between on-premises hardware and the cloud is likely to be light, or you are willing to trade slightly extended latency for the flexibility and processing power of the cloud.</span></span>

<span data-ttu-id="9e81b-109">**优点**</span><span class="sxs-lookup"><span data-stu-id="9e81b-109">**Benefits**</span></span>

- <span data-ttu-id="9e81b-110">易于配置。</span><span class="sxs-lookup"><span data-stu-id="9e81b-110">Simple to configure.</span></span>

<span data-ttu-id="9e81b-111">**挑战**</span><span class="sxs-lookup"><span data-stu-id="9e81b-111">**Challenges**</span></span>

- <span data-ttu-id="9e81b-112">需要本地 VPN 设备。</span><span class="sxs-lookup"><span data-stu-id="9e81b-112">Requires an on-premises VPN device.</span></span>
- <span data-ttu-id="9e81b-113">尽管 Microsoft 保证每个 VPN 网关 99.9% 的可用性，但此 SLA 仅涵盖 VPN 网关，并不涉及与网关之间的网络连接。</span><span class="sxs-lookup"><span data-stu-id="9e81b-113">Although Microsoft guarantees 99.9% availability for each VPN Gateway, this SLA only covers the VPN gateway, and not your network connection to the gateway.</span></span>
- <span data-ttu-id="9e81b-114">通过 Azure VPN 网关的 VPN 连接当前最多支持 200 Mbps 的带宽。</span><span class="sxs-lookup"><span data-stu-id="9e81b-114">A VPN connection over Azure VPN Gateway currently supports a maximum of 200 Mbps bandwidth.</span></span> <span data-ttu-id="9e81b-115">如果预期会超过此吞吐量，可能需要跨多个 VPN 连接对 Azure 虚拟网络进行分区。</span><span class="sxs-lookup"><span data-stu-id="9e81b-115">You may need to partition your Azure virtual network across multiple VPN connections if you expect to exceed this throughput.</span></span>

<span data-ttu-id="9e81b-116">**[了解详细信息...][vpn]**</span><span class="sxs-lookup"><span data-stu-id="9e81b-116">**[Read more...][vpn]**</span></span>

## <a name="azure-expressroute-connection"></a><span data-ttu-id="9e81b-117">Azure ExpressRoute 连接</span><span class="sxs-lookup"><span data-stu-id="9e81b-117">Azure ExpressRoute connection</span></span>

<span data-ttu-id="9e81b-118">ExpressRoute 连接通过第三方连接提供商使用私有的专用连接。</span><span class="sxs-lookup"><span data-stu-id="9e81b-118">ExpressRoute connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="9e81b-119">该专用连接将本地网络扩展到 Azure 中。</span><span class="sxs-lookup"><span data-stu-id="9e81b-119">The private connection extends your on-premises network into Azure.</span></span> 

<span data-ttu-id="9e81b-120">此体系结构适合满足以下条件的混合应用程序：运行需要较高程度可伸缩性的大规模、任务关键型工作负荷。</span><span class="sxs-lookup"><span data-stu-id="9e81b-120">This architecture is suitable for hybrid applications running large-scale, mission-critical workloads that require a high degree of scalability.</span></span> 

<span data-ttu-id="9e81b-121">**优点**</span><span class="sxs-lookup"><span data-stu-id="9e81b-121">**Benefits**</span></span>

- <span data-ttu-id="9e81b-122">有很高的带宽可用；最高可达 10 Gbps，具体取决于连接提供商。</span><span class="sxs-lookup"><span data-stu-id="9e81b-122">Much higher bandwidth available; up to 10 Gbps depending on the connectivity provider.</span></span>
- <span data-ttu-id="9e81b-123">支持动态缩放带宽以帮助在需求较低的时段降低成本。</span><span class="sxs-lookup"><span data-stu-id="9e81b-123">Supports dynamic scaling of bandwidth to help reduce costs during periods of lower demand.</span></span> <span data-ttu-id="9e81b-124">但是，并非所有连接提供商都提供此选项。</span><span class="sxs-lookup"><span data-stu-id="9e81b-124">However, not all connectivity providers have this option.</span></span>
- <span data-ttu-id="9e81b-125">可能会允许组织直接访问国家/地区云，具体取决于连接提供商。</span><span class="sxs-lookup"><span data-stu-id="9e81b-125">May allow your organization direct access to national clouds, depending on the connectivity provider.</span></span>
- <span data-ttu-id="9e81b-126">跨整个连接的 99.9% 可用性 SLA。</span><span class="sxs-lookup"><span data-stu-id="9e81b-126">99.9% availability SLA across the entire connection.</span></span>

<span data-ttu-id="9e81b-127">**挑战**</span><span class="sxs-lookup"><span data-stu-id="9e81b-127">**Challenges**</span></span>

- <span data-ttu-id="9e81b-128">设置可能复杂。</span><span class="sxs-lookup"><span data-stu-id="9e81b-128">Can be complex to set up.</span></span> <span data-ttu-id="9e81b-129">创建 ExpressRoute 连接需要使用第三方连接提供商。</span><span class="sxs-lookup"><span data-stu-id="9e81b-129">Creating an ExpressRoute connection requires working with a third-party connectivity provider.</span></span> <span data-ttu-id="9e81b-130">该提供商负责预配网络连接。</span><span class="sxs-lookup"><span data-stu-id="9e81b-130">The provider is responsible for provisioning the network connection.</span></span>
- <span data-ttu-id="9e81b-131">需要本地高带宽路由器。</span><span class="sxs-lookup"><span data-stu-id="9e81b-131">Requires high-bandwidth routers on-premises.</span></span>

<span data-ttu-id="9e81b-132">**[了解详细信息...][expressroute]**</span><span class="sxs-lookup"><span data-stu-id="9e81b-132">**[Read more...][expressroute]**</span></span>

## <a name="expressroute-with-vpn-failover"></a><span data-ttu-id="9e81b-133">使用 ExpressRoute 和 VPN 实现故障转移</span><span class="sxs-lookup"><span data-stu-id="9e81b-133">ExpressRoute with VPN failover</span></span>

<span data-ttu-id="9e81b-134">此选项合并了前面两个选项，在正常情况下使用 ExpressRoute，但在 ExpressRoute 线路中发生连接丢失时故障转移到 VPN 连接。</span><span class="sxs-lookup"><span data-stu-id="9e81b-134">This options combines the previous two, using ExpressRoute in normal conditions, but failing over to a VPN connection if there is a loss of connectivity in the ExpressRoute circuit.</span></span>

<span data-ttu-id="9e81b-135">此体系结构适合需要 ExpressRoute 的较高带宽并且还需要高度可用的网络连接的混合应用程序。</span><span class="sxs-lookup"><span data-stu-id="9e81b-135">This architecture is suitable for hybrid applications that need the higher bandwidth of ExpressRoute, and also require highly available network connectivity.</span></span> 

<span data-ttu-id="9e81b-136">**优点**</span><span class="sxs-lookup"><span data-stu-id="9e81b-136">**Benefits**</span></span>

- <span data-ttu-id="9e81b-137">在 ExpressRoute 线路出现故障时具有高可用性，虽然回退连接位于带宽较低的网络上。</span><span class="sxs-lookup"><span data-stu-id="9e81b-137">High availability if the ExpressRoute circuit fails, although the fallback connection is on a lower bandwidth network.</span></span>

<span data-ttu-id="9e81b-138">**挑战**</span><span class="sxs-lookup"><span data-stu-id="9e81b-138">**Challenges**</span></span>

- <span data-ttu-id="9e81b-139">配置复杂。</span><span class="sxs-lookup"><span data-stu-id="9e81b-139">Complex to configure.</span></span> <span data-ttu-id="9e81b-140">需要设置 VPN 连接和 ExpressRoute 线路。</span><span class="sxs-lookup"><span data-stu-id="9e81b-140">You need to set up both a VPN connection and an ExpressRoute circuit.</span></span>
- <span data-ttu-id="9e81b-141">需要冗余硬件（VPN 设备），以及你需要为其付费的冗余 Azure VPN 网关连接。</span><span class="sxs-lookup"><span data-stu-id="9e81b-141">Requires redundant hardware (VPN appliances), and a redundant Azure VPN Gateway connection for which you pay charges.</span></span>

<span data-ttu-id="9e81b-142">**[了解详细信息...][expressroute-vpn-failover]**</span><span class="sxs-lookup"><span data-stu-id="9e81b-142">**[Read more...][expressroute-vpn-failover]**</span></span>

<!-- links -->
[expressroute]: ./expressroute.md
[expressroute-vpn-failover]: ./expressroute-vpn-failover.md
[vpn]: ./vpn.md