---
title: 使用 ExpressRoute 将本地网络连接到 Azure
description: 如何实现这样一个安全的站点到站点网络体系结构：跨 Azure 虚拟网络，以及使用 Azure ExpressRoute 建立连接的本地网络。
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute-vpn-failover
pnp.series.prev: vpn
cardTitle: ExpressRoute
ms.openlocfilehash: ada07f399925da6da28b24260f5c73f1e106fd7d
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/30/2018
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute"></a><span data-ttu-id="06d24-103">使用 ExpressRoute 将本地网络连接到 Azure</span><span class="sxs-lookup"><span data-stu-id="06d24-103">Connect an on-premises network to Azure using ExpressRoute</span></span>

<span data-ttu-id="06d24-104">此参考体系结构演示如何使用 [Azure ExpressRoute][expressroute-introduction] 将本地网络连接到 Azure 上的虚拟网络。</span><span class="sxs-lookup"><span data-stu-id="06d24-104">This reference architecture shows how to connect an on-premises network to virtual networks on Azure, using [Azure ExpressRoute][expressroute-introduction].</span></span> <span data-ttu-id="06d24-105">ExpressRoute 连接通过第三方连接提供商使用私有的专用连接。</span><span class="sxs-lookup"><span data-stu-id="06d24-105">ExpressRoute connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="06d24-106">该专用连接将本地网络扩展到 Azure 中。</span><span class="sxs-lookup"><span data-stu-id="06d24-106">The private connection extends your on-premises network into Azure.</span></span> [<span data-ttu-id="06d24-107">**部署此解决方案**。</span><span class="sxs-lookup"><span data-stu-id="06d24-107">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="06d24-108">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="06d24-108">![[0]][0]</span></span>

<span data-ttu-id="06d24-109">下载此体系结构的 [Visio 文件][visio-download]。</span><span class="sxs-lookup"><span data-stu-id="06d24-109">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="06d24-110">体系结构</span><span class="sxs-lookup"><span data-stu-id="06d24-110">Architecture</span></span>

<span data-ttu-id="06d24-111">该体系结构包括以下组件。</span><span class="sxs-lookup"><span data-stu-id="06d24-111">The architecture consists of the following components.</span></span>

* <span data-ttu-id="06d24-112">**本地公司网络**。</span><span class="sxs-lookup"><span data-stu-id="06d24-112">**On-premises corporate network**.</span></span> <span data-ttu-id="06d24-113">组织中运行的专用局域网。</span><span class="sxs-lookup"><span data-stu-id="06d24-113">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="06d24-114">**ExpressRoute 线路**。</span><span class="sxs-lookup"><span data-stu-id="06d24-114">**ExpressRoute circuit**.</span></span> <span data-ttu-id="06d24-115">连接提供商提供的第 2 层或第 3 层线路，用于通过边缘路由器将本地网络与 Azure 相连接。</span><span class="sxs-lookup"><span data-stu-id="06d24-115">A layer 2 or layer 3 circuit supplied by the connectivity provider that joins the on-premises network with Azure through the edge routers.</span></span> <span data-ttu-id="06d24-116">该线路使用连接提供商管理的硬件基础结构。</span><span class="sxs-lookup"><span data-stu-id="06d24-116">The circuit uses the hardware infrastructure managed by the connectivity provider.</span></span>

* <span data-ttu-id="06d24-117">**本地边缘路由器**。</span><span class="sxs-lookup"><span data-stu-id="06d24-117">**Local edge routers**.</span></span> <span data-ttu-id="06d24-118">将本地网络连接到提供商管理的线路的路由器。</span><span class="sxs-lookup"><span data-stu-id="06d24-118">Routers that connect the on-premises network to the circuit managed by the provider.</span></span> <span data-ttu-id="06d24-119">根据连接的预配方式，可能需要提供路由器使用的公共 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="06d24-119">Depending on how your connection is provisioned, you may need to provide the public IP addresses used by the routers.</span></span>
* <span data-ttu-id="06d24-120">**Microsoft Edge 路由器**。</span><span class="sxs-lookup"><span data-stu-id="06d24-120">**Microsoft edge routers**.</span></span> <span data-ttu-id="06d24-121">高度可用的主动-主动配置中的两个路由器。</span><span class="sxs-lookup"><span data-stu-id="06d24-121">Two routers in an active-active highly available configuration.</span></span> <span data-ttu-id="06d24-122">这两个路由器使连接提供商可以将其线路直接连接到其数据中心。</span><span class="sxs-lookup"><span data-stu-id="06d24-122">These routers enable a connectivity provider to connect their circuits directly to their datacenter.</span></span> <span data-ttu-id="06d24-123">根据连接的预配方式，可能需要提供路由器使用的公共 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="06d24-123">Depending on how your connection is provisioned, you may need to provide the public IP addresses used by the routers.</span></span>

* <span data-ttu-id="06d24-124">**Azure 虚拟网络 (VNet)**。</span><span class="sxs-lookup"><span data-stu-id="06d24-124">**Azure virtual networks (VNets)**.</span></span> <span data-ttu-id="06d24-125">每个 VNet 驻留在单个 Azure 区域中，可以托管多个应用层。</span><span class="sxs-lookup"><span data-stu-id="06d24-125">Each VNet resides in a single Azure region, and can host multiple application tiers.</span></span> <span data-ttu-id="06d24-126">可以使用每个 VNet 中的子网将应用层分段。</span><span class="sxs-lookup"><span data-stu-id="06d24-126">Application tiers can be segmented using subnets in each VNet.</span></span>

* <span data-ttu-id="06d24-127">**Azure 公共服务**。</span><span class="sxs-lookup"><span data-stu-id="06d24-127">**Azure public services**.</span></span> <span data-ttu-id="06d24-128">可以在混合应用程序中使用的 Azure 服务。</span><span class="sxs-lookup"><span data-stu-id="06d24-128">Azure services that can be used within a hybrid application.</span></span> <span data-ttu-id="06d24-129">这些服务还可通过 Internet 进行使用，但是使用 ExpressRoute 线路访问它们可提供低延迟和可预测性更高的性能，因为流量不会经过 Internet。</span><span class="sxs-lookup"><span data-stu-id="06d24-129">These services are also available over the Internet, but accessing them using an ExpressRoute circuit provides low latency and more predictable performance, because traffic does not go through the Internet.</span></span> <span data-ttu-id="06d24-130">连接使用[公共对等互连][expressroute-peering]执行，地址由组织拥有或由连接提供商提供。</span><span class="sxs-lookup"><span data-stu-id="06d24-130">Connections are performed using [public peering][expressroute-peering], with addresses that are either owned by your organization or supplied by your connectivity provider.</span></span>

* <span data-ttu-id="06d24-131">**Office 365 服务**。</span><span class="sxs-lookup"><span data-stu-id="06d24-131">**Office 365 services**.</span></span> <span data-ttu-id="06d24-132">Microsoft 提供的公开可用的 Office 365 应用程序和服务。</span><span class="sxs-lookup"><span data-stu-id="06d24-132">The publicly available Office 365 applications and services provided by Microsoft.</span></span> <span data-ttu-id="06d24-133">连接使用 [Microsoft 对等互连][expressroute-peering]执行，地址由组织拥有或由连接提供商提供。</span><span class="sxs-lookup"><span data-stu-id="06d24-133">Connections are performed using [Microsoft peering][expressroute-peering], with addresses that are either owned by your organization or supplied by your connectivity provider.</span></span> <span data-ttu-id="06d24-134">还可以通过 Microsoft 对等互连直接连接到 Microsoft CRM Online。</span><span class="sxs-lookup"><span data-stu-id="06d24-134">You can also connect directly to Microsoft CRM Online through Microsoft peering.</span></span>

* <span data-ttu-id="06d24-135">**连接提供商**（未显示）。</span><span class="sxs-lookup"><span data-stu-id="06d24-135">**Connectivity providers** (not shown).</span></span> <span data-ttu-id="06d24-136">在你的数据中心与 Azure 数据中心之间使用第 2 层或第 3 层连接提供连接的公司。</span><span class="sxs-lookup"><span data-stu-id="06d24-136">Companies that provide a connection either using layer 2 or layer 3 connectivity between your datacenter and an Azure datacenter.</span></span>

## <a name="recommendations"></a><span data-ttu-id="06d24-137">建议</span><span class="sxs-lookup"><span data-stu-id="06d24-137">Recommendations</span></span>

<span data-ttu-id="06d24-138">以下建议适用于大多数方案。</span><span class="sxs-lookup"><span data-stu-id="06d24-138">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="06d24-139">除非有优先于这些建议的特定要求，否则请遵循这些建议。</span><span class="sxs-lookup"><span data-stu-id="06d24-139">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="connectivity-providers"></a><span data-ttu-id="06d24-140">连接提供商</span><span class="sxs-lookup"><span data-stu-id="06d24-140">Connectivity providers</span></span>

<span data-ttu-id="06d24-141">为你所在位置选择合适的 ExpressRoute 连接提供商。</span><span class="sxs-lookup"><span data-stu-id="06d24-141">Select a suitable ExpressRoute connectivity provider for your location.</span></span> <span data-ttu-id="06d24-142">若要获取你所在位置可用的连接提供商的列表，请使用以下 Azure PowerShell 命令：</span><span class="sxs-lookup"><span data-stu-id="06d24-142">To get a list of connectivity providers available at your location, use the following Azure PowerShell command:</span></span>

```powershell
Get-AzureRmExpressRouteServiceProvider
```

<span data-ttu-id="06d24-143">ExpressRoute 连接提供商采用以下方式将你的数据中心连接到 Microsoft：</span><span class="sxs-lookup"><span data-stu-id="06d24-143">ExpressRoute connectivity providers connect your datacenter to Microsoft in the following ways:</span></span>

* <span data-ttu-id="06d24-144">**共置于云交换位置**。</span><span class="sxs-lookup"><span data-stu-id="06d24-144">**Co-located at a cloud exchange**.</span></span> <span data-ttu-id="06d24-145">如果所在的位置提供云交换设施，则可以订购虚拟交叉连接，以通过共同租用提供商的以太网交换连接到 Azure。</span><span class="sxs-lookup"><span data-stu-id="06d24-145">If you're co-located in a facility with a cloud exchange, you can order virtual cross-connections to Azure through the co-location provider’s Ethernet exchange.</span></span> <span data-ttu-id="06d24-146">共同租用提供商可以在共置设施中的基础结构与 Azure 之间提供第 2 层交叉连接或托管的第 3 层交叉连接。</span><span class="sxs-lookup"><span data-stu-id="06d24-146">Co-location providers can offer either layer 2 cross-connections, or managed layer 3 cross-connections between your infrastructure in the co-location facility and Azure.</span></span>
* <span data-ttu-id="06d24-147">**点到点以太网连接**。</span><span class="sxs-lookup"><span data-stu-id="06d24-147">**Point-to-point Ethernet connections**.</span></span> <span data-ttu-id="06d24-148">可以通过点到点以太网链路，将本地数据中心/办公室连接到 Azure。</span><span class="sxs-lookup"><span data-stu-id="06d24-148">You can connect your on-premises datacenters/offices to Azure through point-to-point Ethernet links.</span></span> <span data-ttu-id="06d24-149">点到点以太网提供商可以在站点与 Azure 之间提供第 2 层连接或托管的第 3 层连接。</span><span class="sxs-lookup"><span data-stu-id="06d24-149">Point-to-point Ethernet providers can offer layer 2 connections, or managed layer 3 connections between your site and Azure.</span></span>
* <span data-ttu-id="06d24-150">**任意位置之间的 (IPVPN) 网络**。</span><span class="sxs-lookup"><span data-stu-id="06d24-150">**Any-to-any (IPVPN) networks**.</span></span> <span data-ttu-id="06d24-151">可以将广域网 (WAN) 与 Azure 集成。</span><span class="sxs-lookup"><span data-stu-id="06d24-151">You can integrate your wide area network (WAN) with Azure.</span></span> <span data-ttu-id="06d24-152">Internet 协议虚拟专用网络 (IPVPN) 提供商（通常为多协议标签切换 VPN）可在分支机构与数据中心之间提供任意位置之间的连接。</span><span class="sxs-lookup"><span data-stu-id="06d24-152">Internet protocol virtual private network (IPVPN) providers (typically a multiprotocol label switching VPN) offer any-to-any connectivity between your branch offices and datacenters.</span></span> <span data-ttu-id="06d24-153">Azure 可与 WAN 互连，就如同它是其他任何一个分支机构。</span><span class="sxs-lookup"><span data-stu-id="06d24-153">Azure can be interconnected to your WAN to make it look just like any other branch office.</span></span> <span data-ttu-id="06d24-154">WAN 提供商通常提供托管的第 3 层连接。</span><span class="sxs-lookup"><span data-stu-id="06d24-154">WAN providers typically offer managed layer 3 connectivity.</span></span>

<span data-ttu-id="06d24-155">有关连接提供商的详细信息，请参阅 [ExpressRoute 简介][expressroute-introduction]。</span><span class="sxs-lookup"><span data-stu-id="06d24-155">For more information about connectivity providers, see the [ExpressRoute introduction][expressroute-introduction].</span></span>

### <a name="expressroute-circuit"></a><span data-ttu-id="06d24-156">ExpressRoute 线路</span><span class="sxs-lookup"><span data-stu-id="06d24-156">ExpressRoute circuit</span></span>

<span data-ttu-id="06d24-157">确保组织符合有关连接 Azure 的 [ExpressRoute 先决条件要求][expressroute-prereqs]。</span><span class="sxs-lookup"><span data-stu-id="06d24-157">Ensure that your organization has met the [ExpressRoute prerequisite requirements][expressroute-prereqs] for connecting to Azure.</span></span>

<span data-ttu-id="06d24-158">如果你尚未这样做，请将名为 `GatewaySubnet` 的子网添加到你的 Azure VNet，并使用 Azure VPN 网关服务创建 ExpressRoute 虚拟网络网关。</span><span class="sxs-lookup"><span data-stu-id="06d24-158">If you haven't already done so, add a subnet named `GatewaySubnet` to your Azure VNet and create an ExpressRoute virtual network gateway using the Azure VPN gateway service.</span></span> <span data-ttu-id="06d24-159">有关此过程的详细信息，请参阅 [ExpressRoute 线路预配工作流和线路状态][ExpressRoute-provisioning]。</span><span class="sxs-lookup"><span data-stu-id="06d24-159">For more information about this process, see [ExpressRoute workflows for circuit provisioning and circuit states][ExpressRoute-provisioning].</span></span>

<span data-ttu-id="06d24-160">按如下所示创建 ExpressRoute 线路：</span><span class="sxs-lookup"><span data-stu-id="06d24-160">Create an ExpressRoute circuit as follows:</span></span>

1. <span data-ttu-id="06d24-161">运行以下 PowerShell 命令：</span><span class="sxs-lookup"><span data-stu-id="06d24-161">Run the following PowerShell command:</span></span>
   
    ```powershell
    New-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>> -Location <<location>> -SkuTier <<sku-tier>> -SkuFamily <<sku-family>> -ServiceProviderName <<service-provider-name>> -PeeringLocation <<peering-location>> -BandwidthInMbps <<bandwidth-in-mbps>>
    ```
2. <span data-ttu-id="06d24-162">将新线路的 `ServiceKey` 发送给服务提供商。</span><span class="sxs-lookup"><span data-stu-id="06d24-162">Send the `ServiceKey` for the new circuit to the service provider.</span></span>

3. <span data-ttu-id="06d24-163">等待提供商预配线路。</span><span class="sxs-lookup"><span data-stu-id="06d24-163">Wait for the provider to provision the circuit.</span></span> <span data-ttu-id="06d24-164">若要验证线路的预配状态，请运行以下 PowerShell 命令：</span><span class="sxs-lookup"><span data-stu-id="06d24-164">To verify the provisioning state of a circuit, run the following PowerShell command:</span></span>
   
    ```powershell
    Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    ```

    <span data-ttu-id="06d24-165">当线路准备就绪时，输出的 `Service Provider` 部分中的 `Provisioning state` 字段会从 `NotProvisioned` 更改为 `Provisioned`。</span><span class="sxs-lookup"><span data-stu-id="06d24-165">The `Provisioning state` field in the `Service Provider` section of the output will change from `NotProvisioned` to `Provisioned` when the circuit is ready.</span></span>

    > [!NOTE]
    > <span data-ttu-id="06d24-166">如果你使用第 3 层连接，则提供商应为你配置和管理路由。</span><span class="sxs-lookup"><span data-stu-id="06d24-166">If you're using a layer 3 connection, the provider should configure and manage routing for you.</span></span> <span data-ttu-id="06d24-167">你提供使提供商可以实现相应路由所需的信息。</span><span class="sxs-lookup"><span data-stu-id="06d24-167">You provide the information necessary to enable the provider to implement the appropriate routes.</span></span>
    > 
    > 

4. <span data-ttu-id="06d24-168">如果你使用第 2 层连接：</span><span class="sxs-lookup"><span data-stu-id="06d24-168">If you're using a layer 2 connection:</span></span>

    1. <span data-ttu-id="06d24-169">对于要实现的每种对等互连类型，保留两个由有效公共 IP 地址组成的 /30 子网。</span><span class="sxs-lookup"><span data-stu-id="06d24-169">Reserve two /30 subnets composed of valid public IP addresses for each type of peering you want to implement.</span></span> <span data-ttu-id="06d24-170">这些 /30 子网将用于为用于线路的路由器提供 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="06d24-170">These /30 subnets will be used to provide IP addresses for the routers used for the circuit.</span></span> <span data-ttu-id="06d24-171">如果要实现私有、公共和 Microsoft 对等互连，则需要 6 个包含有效公共 IP 地址的 /30 子网。</span><span class="sxs-lookup"><span data-stu-id="06d24-171">If you are implementing private, public, and Microsoft peering, you'll need 6 /30 subnets with valid public IP addresses.</span></span>     

    2. <span data-ttu-id="06d24-172">配置 ExpressRoute 线路的路由。</span><span class="sxs-lookup"><span data-stu-id="06d24-172">Configure routing for the ExpressRoute circuit.</span></span> <span data-ttu-id="06d24-173">为要配置的每种对等互连类型（私有、公共和 Microsoft）运行以下 PowerShell 命令。</span><span class="sxs-lookup"><span data-stu-id="06d24-173">Run the following PowerShell commands for each type of peering you want to configure (private, public, and Microsoft).</span></span> <span data-ttu-id="06d24-174">有关详细信息，请参阅[创建和修改 ExpressRoute 线路的路由][configure-expressroute-routing]。</span><span class="sxs-lookup"><span data-stu-id="06d24-174">For more information, see [Create and modify routing for an ExpressRoute circuit][configure-expressroute-routing].</span></span>
   
        ```powershell
        Set-AzureRmExpressRouteCircuitPeeringConfig -Name <<peering-name>> -Circuit <<circuit-name>> -PeeringType <<peering-type>> -PeerASN <<peer-asn>> -PrimaryPeerAddressPrefix <<primary-peer-address-prefix>> -SecondaryPeerAddressPrefix <<secondary-peer-address-prefix>> -VlanId <<vlan-id>>

        Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit <<circuit-name>>
        ```

    3. <span data-ttu-id="06d24-175">保留另一个由有效公共 IP 地址组成的池，以用于公共和 Microsoft 对等互连的网络地址转换 (NAT)。</span><span class="sxs-lookup"><span data-stu-id="06d24-175">Reserve another pool of valid public IP addresses to use for network address translation (NAT) for public and Microsoft peering.</span></span> <span data-ttu-id="06d24-176">建议对每个对等互连使用不同池。</span><span class="sxs-lookup"><span data-stu-id="06d24-176">It is recommended to have a different pool for each peering.</span></span> <span data-ttu-id="06d24-177">将该池指定给连接提供商，以便可以为这些范围配置边界网关协议 (BGP) 播发。</span><span class="sxs-lookup"><span data-stu-id="06d24-177">Specify the pool to your connectivity provider, so they can configure border gateway protocol (BGP) advertisements for those ranges.</span></span>

5. <span data-ttu-id="06d24-178">运行以下 PowerShell 命令以将私有 VNet 链接到 ExpressRoute 线路。</span><span class="sxs-lookup"><span data-stu-id="06d24-178">Run the following PowerShell commands to link your private VNet(s) to the ExpressRoute circuit.</span></span> <span data-ttu-id="06d24-179">有关详细信息，请参阅[将虚拟网络链接到 ExpressRoute 线路][link-vnet-to-expressroute]。</span><span class="sxs-lookup"><span data-stu-id="06d24-179">For more information,see [Link a virtual network to an ExpressRoute circuit][link-vnet-to-expressroute].</span></span>

    ```powershell
    $circuit = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $gw = Get-AzureRmVirtualNetworkGateway -Name <<gateway-name>> -ResourceGroupName <<resource-group>>
    New-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> -ResourceGroupName <<resource-group>> -Location <<location> -VirtualNetworkGateway1 $gw -PeerId $circuit.Id -ConnectionType ExpressRoute
    ```

<span data-ttu-id="06d24-180">只要所有 VNet 和 ExpressRoute 线路位于相同地缘政治区域内，便可以将位于不同区域中的多个 VNet 连接到相同 ExpressRoute 线路。</span><span class="sxs-lookup"><span data-stu-id="06d24-180">You can connect multiple VNets located in different regions to the same ExpressRoute circuit, as long as all VNets and the ExpressRoute circuit are located within the same geopolitical region.</span></span>

### <a name="troubleshooting"></a><span data-ttu-id="06d24-181">故障排除</span><span class="sxs-lookup"><span data-stu-id="06d24-181">Troubleshooting</span></span> 

<span data-ttu-id="06d24-182">如果以前正常工作的 ExpressRoute 线路现在未能连接，则在本地或私有 VNet 中不进行任何配置更改的情况下，可能需要与连接提供商联系并与其合作来更正问题。</span><span class="sxs-lookup"><span data-stu-id="06d24-182">If a previously functioning ExpressRoute circuit now fails to connect, in the absence of any configuration changes on-premises or within your private VNet, you may need to contact the connectivity provider and work with them to correct the issue.</span></span> <span data-ttu-id="06d24-183">使用以下 Powershell 命令可验证是否预配了 ExpressRoute 线路：</span><span class="sxs-lookup"><span data-stu-id="06d24-183">Use the following Powershell commands to verify that the ExpressRoute circuit has been provisioned:</span></span>

```powershell
Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

<span data-ttu-id="06d24-184">此命令的输出显示线路的多个属性，包括`ProvisioningState`、`CircuitProvisioningState` 和 `ServiceProviderProvisioningState`，如下所示。</span><span class="sxs-lookup"><span data-stu-id="06d24-184">The output of this command shows several properties for your circuit, including `ProvisioningState`, `CircuitProvisioningState`, and `ServiceProviderProvisioningState` as shown below.</span></span>

```
ProvisioningState                : Succeeded
Sku                              : {
                                     "Name": "Standard_MeteredData",
                                     "Tier": "Standard",
                                     "Family": "MeteredData"
                                   }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : NotProvisioned
```

<span data-ttu-id="06d24-185">如果在尝试创建新线路之后 `ProvisioningState` 未设置为 `Succeeded`，则使用以下命令删除线路并再次尝试创建它。</span><span class="sxs-lookup"><span data-stu-id="06d24-185">If the `ProvisioningState` is not set to `Succeeded` after you tried to create a new circuit, remove the circuit by using the command below and try to create it again.</span></span>

```powershell
Remove-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

<span data-ttu-id="06d24-186">如果提供商已预配了线路，但 `ProvisioningState` 设置为 `Failed`，或 `CircuitProvisioningState` 不是 `Enabled`，请与提供商联系以获得进一步帮助。</span><span class="sxs-lookup"><span data-stu-id="06d24-186">If your provider had already provisioned the circuit, and the `ProvisioningState` is set to `Failed`, or the `CircuitProvisioningState` is not `Enabled`, contact your provider for further assistance.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="06d24-187">可伸缩性注意事项</span><span class="sxs-lookup"><span data-stu-id="06d24-187">Scalability considerations</span></span>

<span data-ttu-id="06d24-188">ExpressRoute 线路在网络之间提供高带宽路径。</span><span class="sxs-lookup"><span data-stu-id="06d24-188">ExpressRoute circuits provide a high bandwidth path between networks.</span></span> <span data-ttu-id="06d24-189">一般情况下，带宽越高，成本便越大。</span><span class="sxs-lookup"><span data-stu-id="06d24-189">Generally, the higher the bandwidth the greater the cost.</span></span> 

<span data-ttu-id="06d24-190">ExpressRoute 向客户提供两个[定价计划][expressroute-pricing]，即按流量计费的计划和无限制数据计划。</span><span class="sxs-lookup"><span data-stu-id="06d24-190">ExpressRoute offers two [pricing plans][expressroute-pricing] to customers, a metered plan and an unlimited data plan.</span></span> <span data-ttu-id="06d24-191">费用因线路带宽而异。</span><span class="sxs-lookup"><span data-stu-id="06d24-191">Charges vary according to circuit bandwidth.</span></span> <span data-ttu-id="06d24-192">可用带宽可能会因提供商而异。</span><span class="sxs-lookup"><span data-stu-id="06d24-192">Available bandwidth will likely vary from provider to provider.</span></span> <span data-ttu-id="06d24-193">使用 `Get-AzureRmExpressRouteServiceProvider` cmdlet 可查看你所在区域中可用的提供商以及它们提供的带宽。</span><span class="sxs-lookup"><span data-stu-id="06d24-193">Use the `Get-AzureRmExpressRouteServiceProvider` cmdlet to see the providers available in your region and the bandwidths that they offer.</span></span>
 
<span data-ttu-id="06d24-194">单个 ExpressRoute 线路可以支持一定数量的对等互连和 VNet 链路。</span><span class="sxs-lookup"><span data-stu-id="06d24-194">A single ExpressRoute circuit can support a certain number of peerings and VNet links.</span></span> <span data-ttu-id="06d24-195">有关详细信息，请参阅 [ExpressRoute 限制](/azure/azure-subscription-service-limits)。</span><span class="sxs-lookup"><span data-stu-id="06d24-195">See [ExpressRoute limits](/azure/azure-subscription-service-limits) for more information.</span></span>

<span data-ttu-id="06d24-196">对于额外费用，ExpressRoute 高级版附加组件提供了一些附加功能：</span><span class="sxs-lookup"><span data-stu-id="06d24-196">For an extra charge, the ExpressRoute Premium add-on provides some additional capability:</span></span>

* <span data-ttu-id="06d24-197">提高公共和私有对等互连的路由限制。</span><span class="sxs-lookup"><span data-stu-id="06d24-197">Increased route limits for public and private peering.</span></span> 
* <span data-ttu-id="06d24-198">增加每个 ExpressRoute 线路的 VNet 链路数。</span><span class="sxs-lookup"><span data-stu-id="06d24-198">Increased number of VNet links per ExpressRoute circuit.</span></span> 
* <span data-ttu-id="06d24-199">服务的全球连接。</span><span class="sxs-lookup"><span data-stu-id="06d24-199">Global connectivity for services.</span></span>

<span data-ttu-id="06d24-200">有关详细信息，请参阅 [ExpressRoute 定价][expressroute-pricing]。</span><span class="sxs-lookup"><span data-stu-id="06d24-200">See [ExpressRoute pricing][expressroute-pricing] for details.</span></span> 

<span data-ttu-id="06d24-201">ExpressRoute 线路的设计允许免费将临时网络速度提升到所购带宽限制的两倍。</span><span class="sxs-lookup"><span data-stu-id="06d24-201">ExpressRoute circuits are designed to allow temporary network bursts up to two times the bandwidth limit that you procured for no additional cost.</span></span> <span data-ttu-id="06d24-202">这使用冗余链路来实现。</span><span class="sxs-lookup"><span data-stu-id="06d24-202">This is achieved by using redundant links.</span></span> <span data-ttu-id="06d24-203">但是，并非所有连接提供商都支持此功能。</span><span class="sxs-lookup"><span data-stu-id="06d24-203">However, not all connectivity providers support this feature.</span></span> <span data-ttu-id="06d24-204">在使用之前请验证连接提供商是否启用此功能。</span><span class="sxs-lookup"><span data-stu-id="06d24-204">Verify that your connectivity provider enables this feature before depending on it.</span></span>

<span data-ttu-id="06d24-205">虽然某些提供商允许你更改带宽，但是请确保选取超出你的需求并且提供上升空间的初始带宽。</span><span class="sxs-lookup"><span data-stu-id="06d24-205">Although some providers allow you to change your bandwidth, make sure you pick an initial bandwidth that surpasses your needs and provides room for growth.</span></span> <span data-ttu-id="06d24-206">如果需要在将来增加带宽，则有两个选择：</span><span class="sxs-lookup"><span data-stu-id="06d24-206">If you need to increase bandwidth in the future, you are left with two options:</span></span>

- <span data-ttu-id="06d24-207">增加带宽。</span><span class="sxs-lookup"><span data-stu-id="06d24-207">Increase the bandwidth.</span></span> <span data-ttu-id="06d24-208">应尽可能避免采用此选择，而且并非所有提供商都允许动态增加带宽。</span><span class="sxs-lookup"><span data-stu-id="06d24-208">You should avoid this option as much as possible, and not all providers allow you to increase bandwidth dynamically.</span></span> <span data-ttu-id="06d24-209">但如果需要增加带宽，请与提供商协商，以验证是否支持通过 Powershell 命令更改 ExpressRoute 带宽属性。</span><span class="sxs-lookup"><span data-stu-id="06d24-209">But if a bandwidth increase is needed, check with your provider to verify they support changing ExpressRoute bandwidth properties via Powershell commands.</span></span> <span data-ttu-id="06d24-210">如果支持，则运行以下命令。</span><span class="sxs-lookup"><span data-stu-id="06d24-210">If they do, run the commands below.</span></span>

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $ckt.ServiceProviderProperties.BandwidthInMbps = <<bandwidth-in-mbps>>
    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    <span data-ttu-id="06d24-211">可以增加带宽而不会丢失连接。</span><span class="sxs-lookup"><span data-stu-id="06d24-211">You can increase the bandwidth without loss of connectivity.</span></span> <span data-ttu-id="06d24-212">使带宽降级会导致连接中断，因为必须删除线路，并使用新配置重新创建它。</span><span class="sxs-lookup"><span data-stu-id="06d24-212">Downgrading the bandwidth will result in disruption in connectivity, because you must delete the circuit and recreate it with the new configuration.</span></span>

- <span data-ttu-id="06d24-213">更改定价计划并且/或者升级到高级版。</span><span class="sxs-lookup"><span data-stu-id="06d24-213">Change your pricing plan and/or upgrade to Premium.</span></span> <span data-ttu-id="06d24-214">为此，请运行以下命令。</span><span class="sxs-lookup"><span data-stu-id="06d24-214">To do so, run the following commands.</span></span> <span data-ttu-id="06d24-215">`Sku.Tier` 属性可以是 `Standard` 或 `Premium`；`Sku.Name` 属性可以是 `MeteredData` 或 `UnlimitedData`。</span><span class="sxs-lookup"><span data-stu-id="06d24-215">The `Sku.Tier` property can be `Standard` or `Premium`; the `Sku.Name` property can be `MeteredData` or `UnlimitedData`.</span></span>

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>

    $ckt.Sku.Tier = "Premium"
    $ckt.Sku.Family = "MeteredData"
    $ckt.Sku.Name = "Premium_MeteredData"

    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    > [!IMPORTANT]
    > <span data-ttu-id="06d24-216">请确保 `Sku.Name` 属性与 `Sku.Tier` 和 `Sku.Family` 匹配。</span><span class="sxs-lookup"><span data-stu-id="06d24-216">Make sure the `Sku.Name` property matches the `Sku.Tier` and `Sku.Family`.</span></span> <span data-ttu-id="06d24-217">如果更改系列和层级，而不是名称，则连接会禁用。</span><span class="sxs-lookup"><span data-stu-id="06d24-217">If you change the family and tier, but not the name, your connection will be disabled.</span></span>
    > 
    > 

    <span data-ttu-id="06d24-218">可以升级 SKU 而不发生中断，但无法从无限制定价计划切换为按流量计费。</span><span class="sxs-lookup"><span data-stu-id="06d24-218">You can upgrade the SKU without disruption, but you cannot switch from the unlimited pricing plan to metered.</span></span> <span data-ttu-id="06d24-219">使 SKU 降级时，带宽消耗必须保持在标准 SKU 的默认限制内。</span><span class="sxs-lookup"><span data-stu-id="06d24-219">When downgrading the SKU, your bandwidth consumption must remain within the default limit of the standard SKU.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="06d24-220">可用性注意事项</span><span class="sxs-lookup"><span data-stu-id="06d24-220">Availability considerations</span></span>

<span data-ttu-id="06d24-221">ExpressRoute 不支持路由器冗余协议（如热备用路由协议 (HSRP) 和虚拟路由器冗余协议 (VRRP)）来实现高可用性。</span><span class="sxs-lookup"><span data-stu-id="06d24-221">ExpressRoute does not support router redundancy protocols such as hot standby routing protocol (HSRP) and virtual router redundancy protocol (VRRP) to implement high availability.</span></span> <span data-ttu-id="06d24-222">而是对每个对等互连使用一对冗余的 BGP 会话。</span><span class="sxs-lookup"><span data-stu-id="06d24-222">Instead, it uses a redundant pair of BGP sessions per peering.</span></span> <span data-ttu-id="06d24-223">为了帮助实现与你的网络之间的高可用性连接，Azure 采用主动-主动配置，在两个路由器（Microsoft Edge 的一部分）上为你预配两个冗余端口。</span><span class="sxs-lookup"><span data-stu-id="06d24-223">To facilitate highly-available connections to your network, Azure provisions you with two redundant ports on two routers (part of the Microsoft edge) in an active-active configuration.</span></span>

<span data-ttu-id="06d24-224">默认情况下，BGP 会话使用 60 秒的空闲超时值。</span><span class="sxs-lookup"><span data-stu-id="06d24-224">By default, BGP sessions use an idle timeout value of 60 seconds.</span></span> <span data-ttu-id="06d24-225">如果会话超时三次（180 秒），则路由器会标记为不可用，所有流量都会重定向到其余路由器。</span><span class="sxs-lookup"><span data-stu-id="06d24-225">If a session times out three times (180 seconds total), the router is marked as unavailable, and all traffic is redirected to the remaining router.</span></span> <span data-ttu-id="06d24-226">此 180 秒超时对于关键应用程序而言可能太长。</span><span class="sxs-lookup"><span data-stu-id="06d24-226">This 180-second timeout might be too long for critical applications.</span></span> <span data-ttu-id="06d24-227">如果是这样，则可以在本地路由器上将 BGP 超时设置为较小值。</span><span class="sxs-lookup"><span data-stu-id="06d24-227">If so, you can change your BGP time-out settings on the on-premises router to a smaller value.</span></span>

<span data-ttu-id="06d24-228">根据使用的提供商类型以及愿意配置的 ExpressRoute 线路和虚拟网络网关连接数量，可以采用不同方式为 Azure 连接配置高可用性。</span><span class="sxs-lookup"><span data-stu-id="06d24-228">You can configure high availability for your Azure connection in different ways, depending on the type of provider you use, and the number of ExpressRoute circuits and virtual network gateway connections you're willing to configure.</span></span> <span data-ttu-id="06d24-229">下面汇总了可用性选项：</span><span class="sxs-lookup"><span data-stu-id="06d24-229">The following summarizes your availability options:</span></span>

* <span data-ttu-id="06d24-230">如果使用第 2 层连接，请采用主动-主动配置在本地网络中部署冗余路由器。</span><span class="sxs-lookup"><span data-stu-id="06d24-230">If you're using a layer 2 connection, deploy redundant routers in your on-premises network in an active-active configuration.</span></span> <span data-ttu-id="06d24-231">将主线路连接到一个路由器，将辅助线路连接到另一个路由器。</span><span class="sxs-lookup"><span data-stu-id="06d24-231">Connect the primary circuit to one router, and the secondary circuit to the other.</span></span> <span data-ttu-id="06d24-232">这可在连接两端同时提供高可用性连接。</span><span class="sxs-lookup"><span data-stu-id="06d24-232">This will give you a highly available connection at both ends of the connection.</span></span> <span data-ttu-id="06d24-233">如果需要 ExpressRoute 服务级别协议 (SLA)，则这是必需的。</span><span class="sxs-lookup"><span data-stu-id="06d24-233">This is necessary if you require the ExpressRoute service level agreement (SLA).</span></span> <span data-ttu-id="06d24-234">有关详细信息，请参阅 [Azure ExpressRoute SLA][sla-for-expressroute]。</span><span class="sxs-lookup"><span data-stu-id="06d24-234">See [SLA for Azure ExpressRoute][sla-for-expressroute] for details.</span></span>

    <span data-ttu-id="06d24-235">下图显示一个将冗余本地路由器连接到主线路和辅助线路的配置。</span><span class="sxs-lookup"><span data-stu-id="06d24-235">The following diagram shows a configuration with redundant on-premises routers connected to the primary and secondary circuits.</span></span> <span data-ttu-id="06d24-236">每个线路都为公共对等互连和私有对等互连处理流量（为每个对等互指定一对 /30 地址空间，如上一节所述）。</span><span class="sxs-lookup"><span data-stu-id="06d24-236">Each circuit handles the traffic for a public peering and a private peering (each peering is designated a pair of /30 address spaces, as described in the previous section).</span></span>

    <span data-ttu-id="06d24-237">![[1]][1]</span><span class="sxs-lookup"><span data-stu-id="06d24-237">![[1]][1]</span></span>

* <span data-ttu-id="06d24-238">如果使用第 3 层连接，请验证它是否提供可为你处理可用性的冗余 BGP 会话。</span><span class="sxs-lookup"><span data-stu-id="06d24-238">If you're using a layer 3 connection, verify that it provides redundant BGP sessions that handle availability for you.</span></span>

* <span data-ttu-id="06d24-239">将 VNet 连接到不同服务提供商提供的多条 ExpressRoute 线路。</span><span class="sxs-lookup"><span data-stu-id="06d24-239">Connect the VNet to multiple ExpressRoute circuits, supplied by different service providers.</span></span> <span data-ttu-id="06d24-240">此策略可提供附加高可用性和灾难恢复功能。</span><span class="sxs-lookup"><span data-stu-id="06d24-240">This strategy provides additional high-availability and disaster recovery capabilities.</span></span>

* <span data-ttu-id="06d24-241">将站点到站点 VPN 配置为 ExpressRoute 的故障转移路径。</span><span class="sxs-lookup"><span data-stu-id="06d24-241">Configure a site-to-site VPN as a failover path for ExpressRoute.</span></span> <span data-ttu-id="06d24-242">有关此选项的详细信息，请参阅[使用 ExpressRoute 和 VPN 故障转移将本地网络连接到 Azure][highly-available-network-architecture]。</span><span class="sxs-lookup"><span data-stu-id="06d24-242">For more about this option, see [Connect an on-premises network to Azure using ExpressRoute with VPN failover][highly-available-network-architecture].</span></span>
 <span data-ttu-id="06d24-243">此选项仅适用于私有对等互连。</span><span class="sxs-lookup"><span data-stu-id="06d24-243">This option only applies to private peering.</span></span> <span data-ttu-id="06d24-244">对于 Azure 和 Office 365 服务，Internet 是唯一的故障转移路径。</span><span class="sxs-lookup"><span data-stu-id="06d24-244">For Azure and Office 365 services, the Internet is the only failover path.</span></span> 

## <a name="manageability-considerations"></a><span data-ttu-id="06d24-245">可管理性注意事项</span><span class="sxs-lookup"><span data-stu-id="06d24-245">Manageability considerations</span></span>

<span data-ttu-id="06d24-246">可以使用 [Azure 连接工具包 (AzureCT)][azurect] 监视本地数据中心与 Azure 之间的连接。</span><span class="sxs-lookup"><span data-stu-id="06d24-246">You can use the [Azure Connectivity Toolkit (AzureCT)][azurect] to monitor connectivity between your on-premises datacenter and Azure.</span></span> 

## <a name="security-considerations"></a><span data-ttu-id="06d24-247">安全注意事项</span><span class="sxs-lookup"><span data-stu-id="06d24-247">Security considerations</span></span>

<span data-ttu-id="06d24-248">根据安全问题和符合性需求，可以采用不同方式为 Azure 连接配置安全选项。</span><span class="sxs-lookup"><span data-stu-id="06d24-248">You can configure security options for your Azure connection in different ways, depending on your security concerns and compliance needs.</span></span> 

<span data-ttu-id="06d24-249">ExpressRoute 在第 3 层运行。</span><span class="sxs-lookup"><span data-stu-id="06d24-249">ExpressRoute operates in layer 3.</span></span> <span data-ttu-id="06d24-250">使用将流量限制到合法资源的网络安全设备可以阻止应用层中的威胁。</span><span class="sxs-lookup"><span data-stu-id="06d24-250">Threats in the application layer can be prevented by using a network security appliance that restricts traffic to legitimate resources.</span></span> <span data-ttu-id="06d24-251">此外，只能从本地发起使用公共对等互连的 ExpressRoute 连接。</span><span class="sxs-lookup"><span data-stu-id="06d24-251">Additionally, ExpressRoute connections using public peering can only be initiated from on-premises.</span></span> <span data-ttu-id="06d24-252">这样可以防止未授权服务从 Internet 访问和破坏本地数据。</span><span class="sxs-lookup"><span data-stu-id="06d24-252">This prevents a rogue service from accessing and compromising on-premises data from the Internet.</span></span>

<span data-ttu-id="06d24-253">若要最大程度提高安全性，请在本地网络与提供商边缘路由器之间添加网络安全设备。</span><span class="sxs-lookup"><span data-stu-id="06d24-253">To maximize security, add network security appliances between the on-premises network and the provider edge routers.</span></span> <span data-ttu-id="06d24-254">这有助于限制来自 VNet 的未经授权的流量流入：</span><span class="sxs-lookup"><span data-stu-id="06d24-254">This will help to restrict the inflow of unauthorized traffic from the VNet:</span></span>

<span data-ttu-id="06d24-255">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="06d24-255">![[2]][2]</span></span>

<span data-ttu-id="06d24-256">出于审核或符合性目的，可能需要禁止在 VNet 中运行的组件直接访问 Internet 并实现[强制隧道][forced-tuneling]。</span><span class="sxs-lookup"><span data-stu-id="06d24-256">For auditing or compliance purposes, it may be necessary to prohibit direct access from components running in the VNet to the Internet and implement [forced tunneling][forced-tuneling].</span></span> <span data-ttu-id="06d24-257">在此情况下，Internet 流量应通过在本地运行的代理（可以进行审核）重定向返回。</span><span class="sxs-lookup"><span data-stu-id="06d24-257">In this situation, Internet traffic should be redirected back through a proxy running  on-premises where it can be audited.</span></span> <span data-ttu-id="06d24-258">代理可以配置为阻止未经授权的流量流出，并筛选潜在的恶意入站流量。</span><span class="sxs-lookup"><span data-stu-id="06d24-258">The proxy can be configured to block unauthorized traffic flowing out, and filter potentially malicious inbound traffic.</span></span>

<span data-ttu-id="06d24-259">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="06d24-259">![[3]][3]</span></span>

<span data-ttu-id="06d24-260">若要最大程度提供安全性，请勿为 VM 启用公共 IP 地址，并使用 NSG 确保无法公开访问这些 VM。</span><span class="sxs-lookup"><span data-stu-id="06d24-260">To maximize security, do not enable a public IP address for your VMs, and use NSGs to ensure that these VMs aren't publicly accessible.</span></span> <span data-ttu-id="06d24-261">只应使用内部 IP 地址提供 VM。</span><span class="sxs-lookup"><span data-stu-id="06d24-261">VMs should only be available using the internal IP address.</span></span> <span data-ttu-id="06d24-262">这些地址可以通过 ExpressRoute 网络进行访问，从而使本地 DevOps 员工可以执行配置或维护。</span><span class="sxs-lookup"><span data-stu-id="06d24-262">These addresses can be made accessible through the ExpressRoute network, enabling on-premises DevOps staff to perform configuration or maintenance.</span></span>

<span data-ttu-id="06d24-263">如果必须向外部网络公开 VM 的管理终结点，请使用 NSG 或访问控制列表将这些端口限制为供 IP 地址或网络允许列表可见。</span><span class="sxs-lookup"><span data-stu-id="06d24-263">If you must expose management endpoints for VMs to an external network, use NSGs or access control lists to restrict the visibility of these ports to a whitelist of IP addresses or networks.</span></span>

> [!NOTE]
> <span data-ttu-id="06d24-264">默认情况下，通过 Azure 门户部署的 Azure VM 包含提供登录访问权限的公共 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="06d24-264">By default, Azure VMs deployed through the Azure portal include a public IP address that provides login access.</span></span>  
> 
> 


## <a name="deploy-the-solution"></a><span data-ttu-id="06d24-265">部署解决方案</span><span class="sxs-lookup"><span data-stu-id="06d24-265">Deploy the solution</span></span>

<span data-ttu-id="06d24-266">**先决条件。**</span><span class="sxs-lookup"><span data-stu-id="06d24-266">**Prequisites.**</span></span> <span data-ttu-id="06d24-267">必须提供一个已配置适当网络设备的现有本地基础结构。</span><span class="sxs-lookup"><span data-stu-id="06d24-267">You must have an existing on-premises infrastructure already configured with a suitable network appliance.</span></span>

<span data-ttu-id="06d24-268">若要部署该解决方案，请执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="06d24-268">To deploy the solution, perform the following steps.</span></span>

1. <span data-ttu-id="06d24-269">单击下面的按钮：</span><span class="sxs-lookup"><span data-stu-id="06d24-269">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="06d24-270">等待该链接在 Azure 门户中打开，然后执行以下步骤：</span><span class="sxs-lookup"><span data-stu-id="06d24-270">Wait for the link to open in the Azure portal, then follow these steps:</span></span>
   * <span data-ttu-id="06d24-271">参数文件中已定义**资源组**名称，因此请选择“新建”，并在文本框中输入 `ra-hybrid-er-rg`。</span><span class="sxs-lookup"><span data-stu-id="06d24-271">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-hybrid-er-rg` in the text box.</span></span>
   * <span data-ttu-id="06d24-272">从“位置”下拉框中选择区域。</span><span class="sxs-lookup"><span data-stu-id="06d24-272">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="06d24-273">不要编辑“模板根 URI”或“参数根 URI”文本框。</span><span class="sxs-lookup"><span data-stu-id="06d24-273">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="06d24-274">查看条款和条件，并单击“我同意上述条款和条件”复选框。</span><span class="sxs-lookup"><span data-stu-id="06d24-274">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="06d24-275">单击“购买”按钮。</span><span class="sxs-lookup"><span data-stu-id="06d24-275">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="06d24-276">等待部署完成。</span><span class="sxs-lookup"><span data-stu-id="06d24-276">Wait for the deployment to complete.</span></span>
4. <span data-ttu-id="06d24-277">单击下面的按钮：</span><span class="sxs-lookup"><span data-stu-id="06d24-277">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. <span data-ttu-id="06d24-278">等待该链接在 Azure 门户中打开，然后执行以下步骤：</span><span class="sxs-lookup"><span data-stu-id="06d24-278">Wait for the link to open in the Azure portal, then follow these steps:</span></span>
   * <span data-ttu-id="06d24-279">在“资源组”部分中选择“使用现有”，在文本框中输入 `ra-hybrid-er-rg`。</span><span class="sxs-lookup"><span data-stu-id="06d24-279">Select **Use existing** in the **Resource group** section and enter `ra-hybrid-er-rg` in the text box.</span></span>
   * <span data-ttu-id="06d24-280">从“位置”下拉框中选择区域。</span><span class="sxs-lookup"><span data-stu-id="06d24-280">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="06d24-281">不要编辑“模板根 URI”或“参数根 URI”文本框。</span><span class="sxs-lookup"><span data-stu-id="06d24-281">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="06d24-282">查看条款和条件，并单击“我同意上述条款和条件”复选框。</span><span class="sxs-lookup"><span data-stu-id="06d24-282">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="06d24-283">单击“购买”按钮。</span><span class="sxs-lookup"><span data-stu-id="06d24-283">Click the **Purchase** button.</span></span>
6. <span data-ttu-id="06d24-284">等待部署完成。</span><span class="sxs-lookup"><span data-stu-id="06d24-284">Wait for the deployment to complete.</span></span>


<!-- links -->
[forced-tuneling]: ../dmz/secure-vnet-hybrid.md
[highly-available-network-architecture]: ./expressroute-vpn-failover.md

[expressroute-technical-overview]: /azure/expressroute/expressroute-introduction
[expressroute-prereqs]: /azure/expressroute/expressroute-prerequisites
[configure-expressroute-routing]: /azure/expressroute/expressroute-howto-routing-arm
[sla-for-expressroute]: https://azure.microsoft.com/support/legal/sla/expressroute/v1_0/
[link-vnet-to-expressroute]: /azure/expressroute/expressroute-howto-linkvnet-arm
[ExpressRoute-provisioning]: /azure/expressroute/expressroute-workflows
[expressroute-introduction]: /azure/expressroute/expressroute-introduction
[expressroute-peering]: /azure/expressroute/expressroute-circuit-peerings
[expressroute-pricing]: https://azure.microsoft.com/pricing/details/expressroute/
[expressroute-limits]: /azure/azure-subscription-service-limits#networking-limits
[azurect]: https://github.com/Azure/NetworkMonitoring/tree/master/AzureCT
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[er-circuit-parameters]: https://github.com/mspnp/reference-architectures/tree/master/hybrid-networking/expressroute/parameters/expressRouteCircuit.parameters.json
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[0]: ./images/expressroute.png "使用 Azure ExpressRoute 的混合网络体系结构"
[1]: ../_images/guidance-hybrid-network-expressroute/figure2.png "将冗余路由器用于 ExpressRoute 主线路和辅助线路"
[2]: ../_images/guidance-hybrid-network-expressroute/figure3.png "将安全设备添加到本地网络"
[3]: ../_images/guidance-hybrid-network-expressroute/figure4.png "使用强制隧道审核 Internet 绑定流量"
[4]: ../_images/guidance-hybrid-network-expressroute/figure5.png "查找 ExpressRoute 线路的 ServiceKey"  