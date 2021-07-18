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
ms.locfileid: "30270432"
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute-with-vpn-failover"></a>使用 ExpressRoute 和 VPN 故障转移将本地网络连接到 Azure

此参考体系结构演示如何使用 ExpressRoute 以及用作故障转移连接的站点到站点虚拟专用网络 (VPN)，将本地网络连接到 Azure 虚拟网络 (VNet)。 本地网络与 Azure VNet 之间的流量通过 ExpressRoute 连接传送。 如果 ExpressRoute 线路的连接断开，则通过 IPSec VPN 隧道路由流量。 [**部署此解决方案**。](#deploy-the-solution)

请注意，如果 ExpressRoute 线路不可用，VPN 路由只会处理专用对等互连。 公共对等互连和 Microsoft 对等互连将通过 Internet 建立。 

![[0]][0]

下载此体系结构的 [Visio 文件][visio-download]。

## <a name="architecture"></a>体系结构 

该体系结构包括以下组件。

* **本地网络**。 组织中运行的专用局域网。

* **VPN 设备**。 用于与本地网络建立外部连接的设备或服务。 该 VPN 设备可以是硬件设备，也可以是软件解决方案，例如 Windows Server 2012 中的路由和远程访问服务 (RRAS)。 有关受支持 VPN 设备的列表和有关为连接到 Azure 而配置所选 VPN 设备的信息，请参阅[关于用于建立站点到站点 VPN 网关连接的 VPN 设备][vpn-appliance]。

* **ExpressRoute 线路**。 连接提供商提供的第 2 层或第 3 层线路，用于通过边缘路由器将本地网络与 Azure 相连接。 该线路使用连接提供商管理的硬件基础结构。

* **ExpressRoute 虚拟网络网关**。 ExpressRoute 虚拟网络网关可将 VNet 连接到用于建立本地网络连接的 ExpressRoute 线路。

* **VPN 虚拟网络网关**。 VPN 虚拟网络网关可让 VNet 连接到本地网络中 VPN 设备。 VPN 虚拟网络网关配置为仅通过 VPN 设备接受来自本地网络的请求。 有关详细信息，请参阅[将本地网络连接到 Microsoft Azure 虚拟网络][connect-to-an-Azure-vnet]。

* **VPN 连接**。 该连接包含一些属性，这些属性指定连接类型 (IPSec)，以及与本地 VPN 设备共享的、用于加密流量的密钥。

* **Azure 虚拟网络 (VNet)**。 每个 VNet 驻留在单个 Azure 区域中，可以托管多个应用层。 可以使用每个 VNet 中的子网将应用层分段。

* **网关子网**。 虚拟网络网关保留在同一子网中。

* **云应用程序**。 Azure 中托管的应用程序。 它可以包含多个层，以及通过 Azure 负载均衡器连接的多个子网。 有关应用程序基础结构的详细信息，请参阅[运行 Windows VM 工作负荷][windows-vm-ra]和[运行 Linux VM 工作负荷][linux-vm-ra]。

## <a name="recommendations"></a>建议

以下建议适用于大多数方案。 除非有优先于这些建议的特定要求，否则请遵循这些建议。

### <a name="vnet-and-gatewaysubnet"></a>VNet 和 GatewaySubnet

在同一个 VNet 中创建 ExpressRoute 虚拟网络网关和 VPN 虚拟网络网关。 这意味着，它们应共享名为 *GatewaySubnet* 的同一个子网。

如果 VNet 已包含名为 *GatewaySubnet* 的子网，请确保它具有 /27 或更大的地址空间。 如果现有子网太小，请使用以下 PowerShell 命令删除该子网： 

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Remove-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet
```

如果 VNet 不包含名为 **GatewaySubnet** 的子网，请创建使用以下 Powershell 命令新建一个子网：

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Add-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.224/27"
$vnet = Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
```

### <a name="vpn-and-expressroute-gateways"></a>VPN 和 ExpressRoute 网关

验证组织是否符合有关连接 Azure 的 [ExpressRoute 先决条件要求][expressroute-prereq]。

如果已在 Azure VNet 中创建 VPN 虚拟网络网关，请使用以下 Powershell 命令将其删除：

```powershell
Remove-AzureRmVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup>
```

遵照[使用 Azure ExpressRoute 实施混合网络体系结构][implementing-expressroute]中的说明建立 ExpressRoute 连接。

遵照[使用 Azure 和本地 VPN 实施混合网络体系结构][implementing-vpn]中的说明建立 VPN 虚拟网络网关连接。

建立虚拟网络网关连接后，请按如下所示测试环境：

1. 确保可从本地网络连接到 Azure VNet。
2. 联系提供商停止 ExpressRoute 连接，以进行测试。
3. 验证是否仍可使用 VPN 虚拟网络网关连接从本地网络连接到 Azure VNet。
4. 联系提供商重新建立 ExpressRoute 连接。

## <a name="considerations"></a>注意事项

有关 ExpressRoute 注意事项，请参阅[使用 Azure ExpressRoute 实施混合网络体系结构][guidance-expressroute]指南。

有关站点到站点 VPN 注意事项，请参阅[使用 Azure 和本地 VPN 实施混合网络体系结构][guidance-vpn]指南。

有关一般性 Azure 安全注意事项，请参阅 [Microsoft 云服务和网络安全][best-practices-security]。

## <a name="deploy-the-solution"></a>部署解决方案

**先决条件。** 必须提供一个已配置适当网络设备的现有本地基础结构。

若要部署该解决方案，请执行以下步骤。

1. 单击下面的按钮：<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. 等待该链接在 Azure 门户中打开，然后执行以下步骤：   
   * 参数文件中已定义**资源组**名称，因此请选择“新建”，并在文本框中输入 `ra-hybrid-vpn-er-rg`。
   * 从“位置”下拉框中选择区域。
   * 不要编辑“模板根 URI”或“参数根 URI”文本框。
   * 查看条款和条件，并单击“我同意上述条款和条件”复选框。
   * 单击“购买”按钮。
3. 等待部署完成。
4. 单击下面的按钮：<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. 等待该链接在 Azure 门户中打开，然后执行以下步骤：
   * 在“资源组”部分中选择“使用现有”，在文本框中输入 `ra-hybrid-vpn-er-rg`。
   * 从“位置”下拉框中选择区域。
   * 不要编辑“模板根 URI”或“参数根 URI”文本框。
   * 查看条款和条件，并单击“我同意上述条款和条件”复选框。
   * 单击“购买”按钮。

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
