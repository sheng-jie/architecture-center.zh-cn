---
title: 实施高可用性混合网络体系结构
description: 如何实施这样一个安全站点到站点网络体系结构：跨 Azure 虚拟网络，以及使用 ExpressRoute 和 VPN 网关故障转移建立连接的本地网络。
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.prev: expressroute
cardTitle: Improving availability
ms.openlocfilehash: 81298215c814cee805eff57fdc28f7c127148b5f
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/30/2018
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute-with-vpn-failover"></a><span data-ttu-id="0a93f-103">使用 ExpressRoute 和 VPN 故障转移将本地网络连接到 Azure</span><span class="sxs-lookup"><span data-stu-id="0a93f-103">Connect an on-premises network to Azure using ExpressRoute with VPN failover</span></span>

<span data-ttu-id="0a93f-104">此参考体系结构演示如何使用 ExpressRoute 以及用作故障转移连接的站点到站点虚拟专用网络 (VPN)，将本地网络连接到 Azure 虚拟网络 (VNet)。</span><span class="sxs-lookup"><span data-stu-id="0a93f-104">This reference architecture shows how to connect an on-premises network to an Azure virtual network (VNet) using ExpressRoute, with a site-to-site virtual private network (VPN) as a failover connection.</span></span> <span data-ttu-id="0a93f-105">本地网络与 Azure VNet 之间的流量通过 ExpressRoute 连接传送。</span><span class="sxs-lookup"><span data-stu-id="0a93f-105">Traffic flows between the on-premises network and the Azure VNet through an ExpressRoute connection.</span></span> <span data-ttu-id="0a93f-106">如果 ExpressRoute 线路的连接断开，则通过 IPSec VPN 隧道路由流量。</span><span class="sxs-lookup"><span data-stu-id="0a93f-106">If there is a loss of connectivity in the ExpressRoute circuit, traffic is routed through an IPSec VPN tunnel.</span></span> [<span data-ttu-id="0a93f-107">**部署此解决方案**。</span><span class="sxs-lookup"><span data-stu-id="0a93f-107">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="0a93f-108">请注意，如果 ExpressRoute 线路不可用，VPN 路由只会处理专用对等互连。</span><span class="sxs-lookup"><span data-stu-id="0a93f-108">Note that if the ExpressRoute circuit is unavailable, the VPN route will only handle private peering connections.</span></span> <span data-ttu-id="0a93f-109">公共对等互连和 Microsoft 对等互连将通过 Internet 建立。</span><span class="sxs-lookup"><span data-stu-id="0a93f-109">Public peering and Microsoft peering connections will pass over the Internet.</span></span> 

<span data-ttu-id="0a93f-110">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="0a93f-110">![[0]][0]</span></span>

<span data-ttu-id="0a93f-111">下载此体系结构的 [Visio 文件][visio-download]。</span><span class="sxs-lookup"><span data-stu-id="0a93f-111">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="0a93f-112">体系结构</span><span class="sxs-lookup"><span data-stu-id="0a93f-112">Architecture</span></span> 

<span data-ttu-id="0a93f-113">该体系结构包括以下组件。</span><span class="sxs-lookup"><span data-stu-id="0a93f-113">The architecture consists of the following components.</span></span>

* <span data-ttu-id="0a93f-114">**本地网络**。</span><span class="sxs-lookup"><span data-stu-id="0a93f-114">**On-premises network**.</span></span> <span data-ttu-id="0a93f-115">组织中运行的专用局域网。</span><span class="sxs-lookup"><span data-stu-id="0a93f-115">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="0a93f-116">**VPN 设备**。</span><span class="sxs-lookup"><span data-stu-id="0a93f-116">**VPN appliance**.</span></span> <span data-ttu-id="0a93f-117">用于与本地网络建立外部连接的设备或服务。</span><span class="sxs-lookup"><span data-stu-id="0a93f-117">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="0a93f-118">该 VPN 设备可以是硬件设备，也可以是软件解决方案，例如 Windows Server 2012 中的路由和远程访问服务 (RRAS)。</span><span class="sxs-lookup"><span data-stu-id="0a93f-118">The VPN appliance may be a hardware device, or it can be a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="0a93f-119">有关受支持 VPN 设备的列表和有关为连接到 Azure 而配置所选 VPN 设备的信息，请参阅[关于用于建立站点到站点 VPN 网关连接的 VPN 设备][vpn-appliance]。</span><span class="sxs-lookup"><span data-stu-id="0a93f-119">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="0a93f-120">**ExpressRoute 线路**。</span><span class="sxs-lookup"><span data-stu-id="0a93f-120">**ExpressRoute circuit**.</span></span> <span data-ttu-id="0a93f-121">连接提供商提供的第 2 层或第 3 层线路，用于通过边缘路由器将本地网络与 Azure 相连接。</span><span class="sxs-lookup"><span data-stu-id="0a93f-121">A layer 2 or layer 3 circuit supplied by the connectivity provider that joins the on-premises network with Azure through the edge routers.</span></span> <span data-ttu-id="0a93f-122">该线路使用连接提供商管理的硬件基础结构。</span><span class="sxs-lookup"><span data-stu-id="0a93f-122">The circuit uses the hardware infrastructure managed by the connectivity provider.</span></span>

* <span data-ttu-id="0a93f-123">**ExpressRoute 虚拟网络网关**。</span><span class="sxs-lookup"><span data-stu-id="0a93f-123">**ExpressRoute virtual network gateway**.</span></span> <span data-ttu-id="0a93f-124">ExpressRoute 虚拟网络网关可将 VNet 连接到用于建立本地网络连接的 ExpressRoute 线路。</span><span class="sxs-lookup"><span data-stu-id="0a93f-124">The ExpressRoute virtual network gateway enables the VNet to connect to the ExpressRoute circuit used for connectivity with your on-premises network.</span></span>

* <span data-ttu-id="0a93f-125">**VPN 虚拟网络网关**。</span><span class="sxs-lookup"><span data-stu-id="0a93f-125">**VPN virtual network gateway**.</span></span> <span data-ttu-id="0a93f-126">VPN 虚拟网络网关可让 VNet 连接到本地网络中 VPN 设备。</span><span class="sxs-lookup"><span data-stu-id="0a93f-126">The VPN virtual network gateway enables the VNet to connect to the VPN appliance in the on-premises network.</span></span> <span data-ttu-id="0a93f-127">VPN 虚拟网络网关配置为仅通过 VPN 设备接受来自本地网络的请求。</span><span class="sxs-lookup"><span data-stu-id="0a93f-127">The VPN virtual network gateway is configured to accept requests from the on-premises network only through the VPN appliance.</span></span> <span data-ttu-id="0a93f-128">有关详细信息，请参阅[将本地网络连接到 Microsoft Azure 虚拟网络][connect-to-an-Azure-vnet]。</span><span class="sxs-lookup"><span data-stu-id="0a93f-128">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

* <span data-ttu-id="0a93f-129">**VPN 连接**。</span><span class="sxs-lookup"><span data-stu-id="0a93f-129">**VPN connection**.</span></span> <span data-ttu-id="0a93f-130">该连接包含一些属性，这些属性指定连接类型 (IPSec)，以及与本地 VPN 设备共享的、用于加密流量的密钥。</span><span class="sxs-lookup"><span data-stu-id="0a93f-130">The connection has properties that specify the connection type (IPSec) and the key shared with the on-premises VPN appliance to encrypt traffic.</span></span>

* <span data-ttu-id="0a93f-131">**Azure 虚拟网络 (VNet)**。</span><span class="sxs-lookup"><span data-stu-id="0a93f-131">**Azure Virtual Network (VNet)**.</span></span> <span data-ttu-id="0a93f-132">每个 VNet 驻留在单个 Azure 区域中，可以托管多个应用层。</span><span class="sxs-lookup"><span data-stu-id="0a93f-132">Each VNet resides in a single Azure region, and can host multiple application tiers.</span></span> <span data-ttu-id="0a93f-133">可以使用每个 VNet 中的子网将应用层分段。</span><span class="sxs-lookup"><span data-stu-id="0a93f-133">Application tiers can be segmented using subnets in each VNet.</span></span>

* <span data-ttu-id="0a93f-134">**网关子网**。</span><span class="sxs-lookup"><span data-stu-id="0a93f-134">**Gateway subnet**.</span></span> <span data-ttu-id="0a93f-135">虚拟网络网关保留在同一子网中。</span><span class="sxs-lookup"><span data-stu-id="0a93f-135">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="0a93f-136">**云应用程序**。</span><span class="sxs-lookup"><span data-stu-id="0a93f-136">**Cloud application**.</span></span> <span data-ttu-id="0a93f-137">Azure 中托管的应用程序。</span><span class="sxs-lookup"><span data-stu-id="0a93f-137">The application hosted in Azure.</span></span> <span data-ttu-id="0a93f-138">它可以包含多个层，以及通过 Azure 负载均衡器连接的多个子网。</span><span class="sxs-lookup"><span data-stu-id="0a93f-138">It might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="0a93f-139">有关应用程序基础结构的详细信息，请参阅[运行 Windows VM 工作负荷][windows-vm-ra]和[运行 Linux VM 工作负荷][linux-vm-ra]。</span><span class="sxs-lookup"><span data-stu-id="0a93f-139">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

## <a name="recommendations"></a><span data-ttu-id="0a93f-140">建议</span><span class="sxs-lookup"><span data-stu-id="0a93f-140">Recommendations</span></span>

<span data-ttu-id="0a93f-141">以下建议适用于大多数方案。</span><span class="sxs-lookup"><span data-stu-id="0a93f-141">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="0a93f-142">除非有优先于这些建议的特定要求，否则请遵循这些建议。</span><span class="sxs-lookup"><span data-stu-id="0a93f-142">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="0a93f-143">VNet 和 GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="0a93f-143">VNet and GatewaySubnet</span></span>

<span data-ttu-id="0a93f-144">在同一个 VNet 中创建 ExpressRoute 虚拟网络网关和 VPN 虚拟网络网关。</span><span class="sxs-lookup"><span data-stu-id="0a93f-144">Create the ExpressRoute virtual network gateway and the VPN virtual network gateway in the same VNet.</span></span> <span data-ttu-id="0a93f-145">这意味着，它们应共享名为 *GatewaySubnet* 的同一个子网。</span><span class="sxs-lookup"><span data-stu-id="0a93f-145">This means that they should share the same subnet named *GatewaySubnet*.</span></span>

<span data-ttu-id="0a93f-146">如果 VNet 已包含名为 *GatewaySubnet* 的子网，请确保它具有 /27 或更大的地址空间。</span><span class="sxs-lookup"><span data-stu-id="0a93f-146">If the VNet already includes a subnet named *GatewaySubnet*, ensure that it has a /27 or larger address space.</span></span> <span data-ttu-id="0a93f-147">如果现有子网太小，请使用以下 PowerShell 命令删除该子网：</span><span class="sxs-lookup"><span data-stu-id="0a93f-147">If the existing subnet is too small, use the following PowerShell command to remove the subnet:</span></span> 

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Remove-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet
```

<span data-ttu-id="0a93f-148">如果 VNet 不包含名为 **GatewaySubnet** 的子网，请创建使用以下 Powershell 命令新建一个子网：</span><span class="sxs-lookup"><span data-stu-id="0a93f-148">If the VNet does not contain a subnet named **GatewaySubnet**, create a new one using the following Powershell command:</span></span>

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Add-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.224/27"
$vnet = Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
```

### <a name="vpn-and-expressroute-gateways"></a><span data-ttu-id="0a93f-149">VPN 和 ExpressRoute 网关</span><span class="sxs-lookup"><span data-stu-id="0a93f-149">VPN and ExpressRoute gateways</span></span>

<span data-ttu-id="0a93f-150">验证组织是否符合有关连接 Azure 的 [ExpressRoute 先决条件要求][expressroute-prereq]。</span><span class="sxs-lookup"><span data-stu-id="0a93f-150">Verify that your organization meets the [ExpressRoute prerequisite requirements][expressroute-prereq] for connecting to Azure.</span></span>

<span data-ttu-id="0a93f-151">如果已在 Azure VNet 中创建 VPN 虚拟网络网关，请使用以下 Powershell 命令将其删除：</span><span class="sxs-lookup"><span data-stu-id="0a93f-151">If you already have a VPN virtual network gateway in your Azure VNet, use the following  Powershell command to remove it:</span></span>

```powershell
Remove-AzureRmVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup>
```

<span data-ttu-id="0a93f-152">遵照[使用 Azure ExpressRoute 实施混合网络体系结构][implementing-expressroute]中的说明建立 ExpressRoute 连接。</span><span class="sxs-lookup"><span data-stu-id="0a93f-152">Follow the instructions in [Implementing a hybrid network architecture with Azure ExpressRoute][implementing-expressroute] to establish your ExpressRoute connection.</span></span>

<span data-ttu-id="0a93f-153">遵照[使用 Azure 和本地 VPN 实施混合网络体系结构][implementing-vpn]中的说明建立 VPN 虚拟网络网关连接。</span><span class="sxs-lookup"><span data-stu-id="0a93f-153">Follow the instructions in [Implementing a hybrid network architecture with Azure and On-premises VPN][implementing-vpn] to establish your VPN virtual network gateway connection.</span></span>

<span data-ttu-id="0a93f-154">建立虚拟网络网关连接后，请按如下所示测试环境：</span><span class="sxs-lookup"><span data-stu-id="0a93f-154">After you have established the virtual network gateway connections, test the environment as follows:</span></span>

1. <span data-ttu-id="0a93f-155">确保可从本地网络连接到 Azure VNet。</span><span class="sxs-lookup"><span data-stu-id="0a93f-155">Make sure you can connect from your on-premises network to your Azure VNet.</span></span>
2. <span data-ttu-id="0a93f-156">联系提供商停止 ExpressRoute 连接，以进行测试。</span><span class="sxs-lookup"><span data-stu-id="0a93f-156">Contact your provider to stop ExpressRoute connectivity for testing.</span></span>
3. <span data-ttu-id="0a93f-157">验证是否仍可使用 VPN 虚拟网络网关连接从本地网络连接到 Azure VNet。</span><span class="sxs-lookup"><span data-stu-id="0a93f-157">Verify that you can still connect from your on-premises network to your Azure VNet using the VPN virtual network gateway connection.</span></span>
4. <span data-ttu-id="0a93f-158">联系提供商重新建立 ExpressRoute 连接。</span><span class="sxs-lookup"><span data-stu-id="0a93f-158">Contact your provider to reestablish ExpressRoute connectivity.</span></span>

## <a name="considerations"></a><span data-ttu-id="0a93f-159">注意事项</span><span class="sxs-lookup"><span data-stu-id="0a93f-159">Considerations</span></span>

<span data-ttu-id="0a93f-160">有关 ExpressRoute 注意事项，请参阅[使用 Azure ExpressRoute 实施混合网络体系结构][guidance-expressroute]指南。</span><span class="sxs-lookup"><span data-stu-id="0a93f-160">For ExpressRoute considerations, see the [Implementing a Hybrid Network Architecture with Azure ExpressRoute][guidance-expressroute] guidance.</span></span>

<span data-ttu-id="0a93f-161">有关站点到站点 VPN 注意事项，请参阅[使用 Azure 和本地 VPN 实施混合网络体系结构][guidance-vpn]指南。</span><span class="sxs-lookup"><span data-stu-id="0a93f-161">For site-to-site VPN considerations, see the [Implementing a Hybrid Network Architecture with Azure and On-premises VPN][guidance-vpn] guidance.</span></span>

<span data-ttu-id="0a93f-162">有关一般性 Azure 安全注意事项，请参阅 [Microsoft 云服务和网络安全][best-practices-security]。</span><span class="sxs-lookup"><span data-stu-id="0a93f-162">For general Azure security considerations, see [Microsoft cloud services and network security][best-practices-security].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="0a93f-163">部署解决方案</span><span class="sxs-lookup"><span data-stu-id="0a93f-163">Deploy the solution</span></span>

<span data-ttu-id="0a93f-164">**先决条件。**</span><span class="sxs-lookup"><span data-stu-id="0a93f-164">**Prequisites.**</span></span> <span data-ttu-id="0a93f-165">必须提供一个已配置适当网络设备的现有本地基础结构。</span><span class="sxs-lookup"><span data-stu-id="0a93f-165">You must have an existing on-premises infrastructure already configured with a suitable network appliance.</span></span>

<span data-ttu-id="0a93f-166">若要部署该解决方案，请执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="0a93f-166">To deploy the solution, perform the following steps.</span></span>

1. <span data-ttu-id="0a93f-167">单击下面的按钮：</span><span class="sxs-lookup"><span data-stu-id="0a93f-167">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="0a93f-168">等待该链接在 Azure 门户中打开，然后执行以下步骤：</span><span class="sxs-lookup"><span data-stu-id="0a93f-168">Wait for the link to open in the Azure portal, then follow these steps:</span></span>   
   * <span data-ttu-id="0a93f-169">参数文件中已定义**资源组**名称，因此请选择“新建”，并在文本框中输入 `ra-hybrid-vpn-er-rg`。</span><span class="sxs-lookup"><span data-stu-id="0a93f-169">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-hybrid-vpn-er-rg` in the text box.</span></span>
   * <span data-ttu-id="0a93f-170">从“位置”下拉框中选择区域。</span><span class="sxs-lookup"><span data-stu-id="0a93f-170">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="0a93f-171">不要编辑“模板根 URI”或“参数根 URI”文本框。</span><span class="sxs-lookup"><span data-stu-id="0a93f-171">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="0a93f-172">查看条款和条件，并单击“我同意上述条款和条件”复选框。</span><span class="sxs-lookup"><span data-stu-id="0a93f-172">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="0a93f-173">单击“购买”按钮。</span><span class="sxs-lookup"><span data-stu-id="0a93f-173">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="0a93f-174">等待部署完成。</span><span class="sxs-lookup"><span data-stu-id="0a93f-174">Wait for the deployment to complete.</span></span>
4. <span data-ttu-id="0a93f-175">单击下面的按钮：</span><span class="sxs-lookup"><span data-stu-id="0a93f-175">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. <span data-ttu-id="0a93f-176">等待该链接在 Azure 门户中打开，然后执行以下步骤：</span><span class="sxs-lookup"><span data-stu-id="0a93f-176">Wait for the link to open in the Azure portal, then enter then follow these steps:</span></span>
   * <span data-ttu-id="0a93f-177">在“资源组”部分中选择“使用现有”，在文本框中输入 `ra-hybrid-vpn-er-rg`。</span><span class="sxs-lookup"><span data-stu-id="0a93f-177">Select **Use existing** in the **Resource group** section and enter `ra-hybrid-vpn-er-rg` in the text box.</span></span>
   * <span data-ttu-id="0a93f-178">从“位置”下拉框中选择区域。</span><span class="sxs-lookup"><span data-stu-id="0a93f-178">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="0a93f-179">不要编辑“模板根 URI”或“参数根 URI”文本框。</span><span class="sxs-lookup"><span data-stu-id="0a93f-179">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="0a93f-180">查看条款和条件，并单击“我同意上述条款和条件”复选框。</span><span class="sxs-lookup"><span data-stu-id="0a93f-180">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="0a93f-181">单击“购买”按钮。</span><span class="sxs-lookup"><span data-stu-id="0a93f-181">Click the **Purchase** button.</span></span>

<!-- links -->

[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md


[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[expressroute-prereq]: /azure/expressroute/expressroute-prerequisites
[implementing-expressroute]: ./expressroute.md
[implementing-vpn]: ./vpn.md
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[best-practices-security]: /azure/best-practices-network-security
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[0]: ./images/expressroute-vpn-failover.png "使用 ExpressRoute 和 VPN 网关的高可用性混合网络体系结构"
