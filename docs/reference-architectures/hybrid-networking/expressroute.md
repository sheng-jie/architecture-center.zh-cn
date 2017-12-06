---
title: "使用 ExpressRoute 将本地网络连接到 Azure"
description: "如何实现这样一个安全的站点到站点网络体系结构：跨 Azure 虚拟网络，以及使用 Azure ExpressRoute 建立连接的本地网络。"
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute-vpn-failover
pnp.series.prev: vpn
cardTitle: ExpressRoute
ms.openlocfilehash: 671be5118faaefab5ba5348de81642d8a8124b59
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute"></a>使用 ExpressRoute 将本地网络连接到 Azure

此参考体系结构演示如何使用 [Azure ExpressRoute][expressroute-introduction] 将本地网络连接到 Azure 上的虚拟网络。 ExpressRoute 连接通过第三方连接提供商使用私有的专用连接。 该专用连接将本地网络扩展到 Azure 中。 [**部署此解决方案**。](#deploy-the-solution)

![[0]][0]

*下载此体系结构的 [Visio 文件][visio-download]。*

## <a name="architecture"></a>体系结构

该体系结构包括以下组件。

* **本地公司网络**。 组织中运行的专用局域网。

* **ExpressRoute 线路**。 连接提供商提供的第 2 层或第 3 层线路，用于通过边缘路由器将本地网络与 Azure 相连接。 该线路使用连接提供商管理的硬件基础结构。

* **本地边缘路由器**。 将本地网络连接到提供商管理的线路的路由器。 根据连接的预配方式，可能需要提供路由器使用的公共 IP 地址。
* **Microsoft Edge 路由器**。 高度可用的主动-主动配置中的两个路由器。 这两个路由器使连接提供商可以将其线路直接连接到其数据中心。 根据连接的预配方式，可能需要提供路由器使用的公共 IP 地址。

* **Azure 虚拟网络 (VNet)**。 每个 VNet 驻留在单个 Azure 区域中，可以托管多个应用层。 可以使用每个 VNet 中的子网将应用层分段。

* **Azure 公共服务**。 可以在混合应用程序中使用的 Azure 服务。 这些服务还可通过 Internet 进行使用，但是使用 ExpressRoute 线路访问它们可提供低延迟和可预测性更高的性能，因为流量不会经过 Internet。 连接使用[公共对等互连][expressroute-peering]执行，地址由组织拥有或由连接提供商提供。

* **Office 365 服务**。 Microsoft 提供的公开可用的 Office 365 应用程序和服务。 连接使用 [Microsoft 对等互连][expressroute-peering]执行，地址由组织拥有或由连接提供商提供。 还可以通过 Microsoft 对等互连直接连接到 Microsoft CRM Online。

* **连接提供商**（未显示）。 在你的数据中心与 Azure 数据中心之间使用第 2 层或第 3 层连接提供连接的公司。

## <a name="recommendations"></a>建议

以下建议适用于大多数方案。 除非有优先于这些建议的特定要求，否则请遵循这些建议。

### <a name="connectivity-providers"></a>连接提供商

为你所在位置选择合适的 ExpressRoute 连接提供商。 若要获取你所在位置可用的连接提供商的列表，请使用以下 Azure PowerShell 命令：

```powershell
Get-AzureRmExpressRouteServiceProvider
```

ExpressRoute 连接提供商采用以下方式将你的数据中心连接到 Microsoft：

* **共置于云交换位置**。 如果所在的位置提供云交换设施，则可以订购虚拟交叉连接，以通过共同租用提供商的以太网交换连接到 Azure。 共同租用提供商可以在共置设施中的基础结构与 Azure 之间提供第 2 层交叉连接或托管的第 3 层交叉连接。
* **点到点以太网连接**。 可以通过点到点以太网链路，将本地数据中心/办公室连接到 Azure。 点到点以太网提供商可以在站点与 Azure 之间提供第 2 层连接或托管的第 3 层连接。
* **任意位置之间的 (IPVPN) 网络**。 可以将广域网 (WAN) 与 Azure 集成。 Internet 协议虚拟专用网络 (IPVPN) 提供商（通常为多协议标签切换 VPN）可在分支机构与数据中心之间提供任意位置之间的连接。 Azure 可与 WAN 互连，就如同它是其他任何一个分支机构。 WAN 提供商通常提供托管的第 3 层连接。

有关连接提供商的详细信息，请参阅 [ExpressRoute 简介][expressroute-introduction]。

### <a name="expressroute-circuit"></a>ExpressRoute 线路

确保组织符合有关连接 Azure 的 [ExpressRoute 先决条件要求][expressroute-prereqs]。

如果你尚未这样做，请将名为 `GatewaySubnet` 的子网添加到你的 Azure VNet，并使用 Azure VPN 网关服务创建 ExpressRoute 虚拟网络网关。 有关此过程的详细信息，请参阅 [ExpressRoute 线路预配工作流和线路状态][ExpressRoute-provisioning]。

按如下所示创建 ExpressRoute 线路：

1. 运行以下 PowerShell 命令：
   
    ```powershell
    New-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>> -Location <<location>> -SkuTier <<sku-tier>> -SkuFamily <<sku-family>> -ServiceProviderName <<service-provider-name>> -PeeringLocation <<peering-location>> -BandwidthInMbps <<bandwidth-in-mbps>>
    ```
2. 将新线路的 `ServiceKey` 发送给服务提供商。

3. 等待提供商预配线路。 若要验证线路的预配状态，请运行以下 PowerShell 命令：
   
    ```powershell
    Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    ```

    当线路准备就绪时，输出的 `Service Provider` 部分中的 `Provisioning state` 字段会从 `NotProvisioned` 更改为 `Provisioned`。

    > [!NOTE]
    > 如果你使用第 3 层连接，则提供商应为你配置和管理路由。 你提供使提供商可以实现相应路由所需的信息。
    > 
    > 

4. 如果你使用第 2 层连接：

    1. 对于要实现的每种对等互连类型，保留两个由有效公共 IP 地址组成的 /30 子网。 这些 /30 子网将用于为用于线路的路由器提供 IP 地址。 如果要实现私有、公共和 Microsoft 对等互连，则需要 6 个包含有效公共 IP 地址的 /30 子网。     

    2. 配置 ExpressRoute 线路的路由。 为要配置的每种对等互连类型（私有、公共和 Microsoft）运行以下 PowerShell 命令。 有关详细信息，请参阅[创建和修改 ExpressRoute 线路的路由][configure-expressroute-routing]。
   
        ```powershell
        Set-AzureRmExpressRouteCircuitPeeringConfig -Name <<peering-name>> -Circuit <<circuit-name>> -PeeringType <<peering-type>> -PeerASN <<peer-asn>> -PrimaryPeerAddressPrefix <<primary-peer-address-prefix>> -SecondaryPeerAddressPrefix <<secondary-peer-address-prefix>> -VlanId <<vlan-id>>

        Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit <<circuit-name>>
        ```

    3. 保留另一个由有效公共 IP 地址组成的池，以用于公共和 Microsoft 对等互连的网络地址转换 (NAT)。 建议对每个对等互连使用不同池。 将该池指定给连接提供商，以便可以为这些范围配置边界网关协议 (BGP) 播发。

5. 运行以下 PowerShell 命令以将私有 VNet 链接到 ExpressRoute 线路。 有关详细信息，请参阅[将虚拟网络链接到 ExpressRoute 线路][link-vnet-to-expressroute]。

    ```powershell
    $circuit = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $gw = Get-AzureRmVirtualNetworkGateway -Name <<gateway-name>> -ResourceGroupName <<resource-group>>
    New-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> -ResourceGroupName <<resource-group>> -Location <<location> -VirtualNetworkGateway1 $gw -PeerId $circuit.Id -ConnectionType ExpressRoute
    ```

只要所有 VNet 和 ExpressRoute 线路位于相同地缘政治区域内，便可以将位于不同区域中的多个 VNet 连接到相同 ExpressRoute 线路。

### <a name="troubleshooting"></a>故障排除 

如果以前正常工作的 ExpressRoute 线路现在未能连接，则在本地或私有 VNet 中不进行任何配置更改的情况下，可能需要与连接提供商联系并与其合作来更正问题。 使用以下 Powershell 命令可验证是否预配了 ExpressRoute 线路：

```powershell
Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

此命令的输出显示线路的多个属性，包括`ProvisioningState`、`CircuitProvisioningState` 和 `ServiceProviderProvisioningState`，如下所示。

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

如果在尝试创建新线路之后 `ProvisioningState` 未设置为 `Succeeded`，则使用以下命令删除线路并再次尝试创建它。

```powershell
Remove-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

如果提供商已预配了线路，但 `ProvisioningState` 设置为 `Failed`，或 `CircuitProvisioningState` 不是 `Enabled`，请与提供商联系以获得进一步帮助。

## <a name="scalability-considerations"></a>可伸缩性注意事项

ExpressRoute 线路在网络之间提供高带宽路径。 一般情况下，带宽越高，成本便越大。 

ExpressRoute 向客户提供两个[定价计划][expressroute-pricing]，即按流量计费的计划和无限制数据计划。 费用因线路带宽而异。 可用带宽可能会因提供商而异。 使用 `Get-AzureRmExpressRouteServiceProvider` cmdlet 可查看你所在区域中可用的提供商以及它们提供的带宽。
 
单个 ExpressRoute 线路可以支持一定数量的对等互连和 VNet 链路。 有关详细信息，请参阅 [ExpressRoute 限制](/azure/azure-subscription-service-limits)。

对于额外费用，ExpressRoute 高级版附加组件提供了一些附加功能：

* 提高公共和私有对等互连的路由限制。 
* 增加每个 ExpressRoute 线路的 VNet 链路数。 
* 服务的全球连接。

有关详细信息，请参阅 [ExpressRoute 定价][expressroute-pricing]。 

ExpressRoute 线路的设计允许免费将临时网络速度提升到所购带宽限制的两倍。 这使用冗余链路来实现。 但是，并非所有连接提供商都支持此功能。 在使用之前请验证连接提供商是否启用此功能。

虽然某些提供商允许你更改带宽，但是请确保选取超出你的需求并且提供上升空间的初始带宽。 如果需要在将来增加带宽，则有两个选择：

- 增加带宽。 应尽可能避免采用此选择，而且并非所有提供商都允许动态增加带宽。 但如果需要增加带宽，请与提供商协商，以验证是否支持通过 Powershell 命令更改 ExpressRoute 带宽属性。 如果支持，则运行以下命令。

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $ckt.ServiceProviderProperties.BandwidthInMbps = <<bandwidth-in-mbps>>
    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    可以增加带宽而不会丢失连接。 使带宽降级会导致连接中断，因为必须删除线路，并使用新配置重新创建它。

- 更改定价计划并且/或者升级到高级版。 为此，请运行以下命令。 `Sku.Tier` 属性可以是 `Standard` 或 `Premium`；`Sku.Name` 属性可以是 `MeteredData` 或 `UnlimitedData`。

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>

    $ckt.Sku.Tier = "Premium"
    $ckt.Sku.Family = "MeteredData"
    $ckt.Sku.Name = "Premium_MeteredData"

    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    > [!IMPORTANT]
    > 请确保 `Sku.Name` 属性与 `Sku.Tier` 和 `Sku.Family` 匹配。 如果更改系列和层级，而不是名称，则连接会禁用。
    > 
    > 

    可以升级 SKU 而不发生中断，但无法从无限制定价计划切换为按流量计费。 使 SKU 降级时，带宽消耗必须保持在标准 SKU 的默认限制内。

## <a name="availability-considerations"></a>可用性注意事项

ExpressRoute 不支持路由器冗余协议（如热备用路由协议 (HSRP) 和虚拟路由器冗余协议 (VRRP)）来实现高可用性。 而是对每个对等互连使用一对冗余的 BGP 会话。 为了帮助实现与你的网络之间的高可用性连接，Azure 采用主动-主动配置，在两个路由器（Microsoft Edge 的一部分）上为你预配两个冗余端口。

默认情况下，BGP 会话使用 60 秒的空闲超时值。 如果会话超时三次（180 秒），则路由器会标记为不可用，所有流量都会重定向到其余路由器。 此 180 秒超时对于关键应用程序而言可能太长。 如果是这样，则可以在本地路由器上将 BGP 超时设置为较小值。

根据使用的提供商类型以及愿意配置的 ExpressRoute 线路和虚拟网络网关连接数量，可以采用不同方式为 Azure 连接配置高可用性。 下面汇总了可用性选项：

* 如果使用第 2 层连接，请采用主动-主动配置在本地网络中部署冗余路由器。 将主线路连接到一个路由器，将辅助线路连接到另一个路由器。 这可在连接两端同时提供高可用性连接。 如果需要 ExpressRoute 服务级别协议 (SLA)，则这是必需的。 有关详细信息，请参阅 [Azure ExpressRoute SLA][sla-for-expressroute]。

    下图显示一个将冗余本地路由器连接到主线路和辅助线路的配置。 每个线路都为公共对等互连和私有对等互连处理流量（为每个对等互指定一对 /30 地址空间，如上一节所述）。

    ![[1]][1]

* 如果使用第 3 层连接，请验证它是否提供可为你处理可用性的冗余 BGP 会话。

* 将 VNet 连接到不同服务提供商提供的多条 ExpressRoute 线路。 此策略可提供附加高可用性和灾难恢复功能。

* 将站点到站点 VPN 配置为 ExpressRoute 的故障转移路径。 有关此选项的详细信息，请参阅[使用 ExpressRoute 和 VPN 故障转移将本地网络连接到 Azure][highly-available-network-architecture]。
 此选项仅适用于私有对等互连。 对于 Azure 和 Office 365 服务，Internet 是唯一的故障转移路径。 

## <a name="manageability-considerations"></a>可管理性注意事项

可以使用 [Azure 连接工具包 (AzureCT)][azurect] 监视本地数据中心与 Azure 之间的连接。 

## <a name="security-considerations"></a>安全注意事项

根据安全问题和符合性需求，可以采用不同方式为 Azure 连接配置安全选项。 

ExpressRoute 在第 3 层运行。 使用将流量限制到合法资源的网络安全设备可以阻止应用层中的威胁。 此外，只能从本地发起使用公共对等互连的 ExpressRoute 连接。 这样可以防止未授权服务从 Internet 访问和破坏本地数据。

若要最大程度提高安全性，请在本地网络与提供商边缘路由器之间添加网络安全设备。 这有助于限制来自 VNet 的未经授权的流量流入：

![[2]][2]

出于审核或符合性目的，可能需要禁止在 VNet 中运行的组件直接访问 Internet 并实现[强制隧道][forced-tuneling]。 在此情况下，Internet 流量应通过在本地运行的代理（可以进行审核）重定向返回。 代理可以配置为阻止未经授权的流量流出，并筛选潜在的恶意入站流量。

![[3]][3]

若要最大程度提供安全性，请勿为 VM 启用公共 IP 地址，并使用 NSG 确保无法公开访问这些 VM。 只应使用内部 IP 地址提供 VM。 这些地址可以通过 ExpressRoute 网络进行访问，从而使本地 DevOps 员工可以执行配置或维护。

如果必须向外部网络公开 VM 的管理终结点，请使用 NSG 或访问控制列表将这些端口限制为供 IP 地址或网络白名单可见。

> [!NOTE]
> 默认情况下，通过 Azure 门户部署的 Azure VM 包含提供登录访问权限的公共 IP 地址。  
> 
> 


## <a name="deploy-the-solution"></a>部署解决方案

**先决条件。** 必须提供一个已配置适当网络设备的现有本地基础结构。

若要部署该解决方案，请执行以下步骤。

1. 单击下面的按钮：<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. 等待该链接在 Azure 门户中打开，然后执行以下步骤：
   * 参数文件中已定义**资源组**名称，因此请选择“新建”，并在文本框中输入 `ra-hybrid-er-rg`。
   * 从“位置”下拉框中选择区域。
   * 不要编辑“模板根 URI”或“参数根 URI”文本框。
   * 查看条款和条件，并单击“我同意上述条款和条件”复选框。
   * 单击“购买”按钮。
3. 等待部署完成。
4. 单击下面的按钮：<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. 等待该链接在 Azure 门户中打开，然后执行以下步骤：
   * 在“资源组”部分中选择“使用现有”，在文本框中输入 `ra-hybrid-er-rg`。
   * 从“位置”下拉框中选择区域。
   * 不要编辑“模板根 URI”或“参数根 URI”文本框。
   * 查看条款和条件，并单击“我同意上述条款和条件”复选框。
   * 单击“购买”按钮。
6. 等待部署完成。


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
[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-architectures.vsdx
[er-circuit-parameters]: https://github.com/mspnp/reference-architectures/tree/master/hybrid-networking/expressroute/parameters/expressRouteCircuit.parameters.json
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[0]: ./images/expressroute.png "使用 Azure ExpressRoute 的混合网络体系结构"
[1]: ../_images/guidance-hybrid-network-expressroute/figure2.png "将冗余路由器用于 ExpressRoute 主线路和辅助线路"
[2]: ../_images/guidance-hybrid-network-expressroute/figure3.png "将安全设备添加到本地网络"
[3]: ../_images/guidance-hybrid-network-expressroute/figure4.png "使用强制隧道审核 Internet 绑定流量"
[4]: ../_images/guidance-hybrid-network-expressroute/figure5.png "查找 ExpressRoute 线路的 ServiceKey"  