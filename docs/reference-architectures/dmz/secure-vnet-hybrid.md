---
title: "在 Azure 中实现安全的混合网络体系结构"
description: "如何在 Azure 中实现安全的混合网络体系结构。"
author: telmosampaio
ms.date: 11/23/2016
pnp.series.title: Network DMZ
pnp.series.prev: ./index
pnp.series.next: secure-vnet-dmz
cardTitle: DMZ between Azure and on-premises
ms.openlocfilehash: 778d5ef6967a09b03bb6b5aca67e3e0c170ad016
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="dmz-between-azure-and-your-on-premises-datacenter"></a>Azure 与本地数据中心之间的 DMZ

此参考体系结构显示了一个可将本地网络扩展到 Azure 的安全混合网络。 此体系结构在本地网络与 Azure 虚拟网络 (VNet) 之间实现一个 DMZ（也称为外围网络）。 DMZ 包括防火墙和数据包检查等实现安全功能的网络虚拟设备 (NVA)。 来自 VNet 的所有传出流量都通过本地网络强制隧道传输到 Internet，以便可以进行审核。

[![0]][0] 

*下载此体系结构的 [Visio 文件][visio-download]。*

此体系结构需要使用 [VPN 网关][ra-vpn]或 [ExpressRoute][ra-expressroute] 连接来连接到本地数据中心。 此体系结构的典型用途包括：

* 在本地运行一部分工作负荷，在 Azure 中运行一部分工作负荷的混合应用程序。
* 需要精细控制从本地数据中心进入 Azure VNet 的流量的基础结构。
* 必须审核传出流量的应用程序。 这通常是许多商业系统的监管要求，可以帮助防止公开披露私有信息。

## <a name="architecture"></a>体系结构

该体系结构包括以下组件。

* **本地网络**。 组织中实现的专用局域网。
* **Azure 虚拟网络 (VNet)**。 VNet 承载在 Azure 中运行的应用程序和其他资源。
* **网关**。 网关在本地网络中的路由器与 VNet 之间提供连接。
* **网络虚拟设备 (NVA)**。 NVA 是一种通用术语，用于描述执行诸如允许或拒绝访问（作为防火墙）、优化广域网 (WAN) 操作（包括网络压缩）、自定义路由或其他网络功能这类任务的 VM。
* **Web 层、业务层和数据层子网**。 承载实现在云中运行的示例第 3 层应用程序的 VM 和服务的子网。 有关详细信息，请参阅[在 Azure 上运行 N 层体系结构所用的 Windows VM][ra-n-tier]。
* **用户定义的路由 (UDR)**。 [用户定义的路由][udr-overview]定义 Azure VNet 中的 IP 流量流。

    > [!NOTE]
    > 根据 VPN 连接的要求，可以配置边界网关协议 (BGP) 路由，而不是使用 UDR 实现通过本地网络定向返回流量的转发规则。
    > 
    > 

* **管理子网。** 此子网包含为 VNet 中运行的组件实现管理和监视功能的 VM。

## <a name="recommendations"></a>建议

以下建议适用于大多数方案。 除非有优先于这些建议的特定要求，否则请遵循这些建议。 

### <a name="access-control-recommendations"></a>访问控制建议

使用[基于角色的访问控制][rbac] (RBAC) 在应用程序中管理资源。 考虑创建以下[自定义角色][rbac-custom-roles]：

- 一个 DevOps 角色，具有管理应用程序的基础结构、部署应用程序组件以及监视和重新启动 VM 的权限。  

- 一个集中式 IT 管理员角色，用于管理和监视网络资源。

- 一个安全性 IT 管理员角色，用于管理安全网络资源，如 NVA。 

DevOps 和 IT 管理员角色不应具有 NVA 资源的访问权限。 这应限制为安全性 IT 管理员角色。

### <a name="resource-group-recommendations"></a>有关资源组的建议

可以通过将 Azure 资源（如 VM、VNet 和负载均衡器）分组到资源组中，来轻松地管理它们。 将 RBAC 角色分配给每个资源组以限制访问。

建议创建以下资源组：

* 一个包含 VNet（不包括 VM）、NSG 以及用于连接到本地网络的网关资源的资源组。 将集中式 IT 管理员角色分配给此资源组。
* 一个包含 NVA 的 VM（包括负载均衡器）、jumpbox 和其他管理 VM 以及强制所有流量经过 NVA 的网关子网 UDR 的资源组。 将安全性 IT 管理员角色分配给此资源组。
* 用于每个应用层的单独资源组，包含负载均衡器和 VM。 请注意，此资源组不应包含每个层的子网。 将 DevOps 角色分配给此资源组。

### <a name="virtual-network-gateway-recommendations"></a>有关虚拟网络网关的建议

本地流量会通过虚拟网络网关传递到 VNet。 建议使用 [Azure VPN 网关][guidance-vpn-gateway]或 [Azure ExpressRoute 网关][guidance-expressroute]。

### <a name="nva-recommendations"></a>有关 NVA 的建议

NVA 可提供不同服务来管理和监视网络流量。 [Azure Marketplace][azure-marketplace-nva] 提供多个可以使用的第三方供应商 NVA。 如果这些第三方 NVA 都不满足你的要求，则可以使用 VM 创建自定义 NVA。 

例如，此参考体系结构的解决方案部署在 VM 上实现具有以下功能的 NVA：

* 流量使用 NVA 网络接口 (NIC) 上的 [IP 转发][ip-forwarding]进行路由。
* 仅当合适时才允许流量经过 NVA。 参考体系结构中的每个 NVA VM 都是简单的 Linux 路由器。 入站流量在网络接口 eth0 上到达，出站流量与通过网络接口 eth1 调度的自定义脚本定义的规则匹配。
* NVA 只能从管理子网进行配置。 
* 路由到管理子网的流量不会经过 NVA。 否则，如果 NVA 发生故障，则没有指向管理子网的路由可用于修复它们。  
* NVA 的 VM 放置在负载均衡器后面的[可用性集][availability-set]中。 网关子网中的 UDR 将 NVA 请求定向到负载均衡器。

包含第 7 层 NVA，以在 NVA 级别终止应用程序连接，并保持与后端层的相关性。 这可以保证建立对称连接，使得来自后端层的响应流量可通过 NVA 返回。

可考虑的另一个选项是按顺序连接多个 NVA，其中每个 NVA 都执行专用安全任务。 这样可以按照每个 NVA 来管理每个安全功能。 例如，实现防火墙的 NVA 可以与运行标识服务的 NVA 按顺序放置。 便于管理的代价是会添加额外网络跃点，这可能会增加延迟，因此请确保这不会影响应用程序性能。


### <a name="nsg-recommendations"></a>有关 NSG 的建议

VPN 网关会针对与本地网络的连接公开公共 IP 地址。 建议为入站 NVA 子网创建网络安全组 (NSG)，其规则是阻止不是源自本地网络的所有流量。

还建议对每个子网使用 NSG，以针对入站流量提供第二层保护，并绕过错误配置的或已禁用的 NVA。 例如，参考体系结构中的 Web 层子网实现的 NSG 的一个规则是忽略从本地网络 (192.168.0.0/16) 或 VNet 接收的请求之外的所有请求，另一个规则是忽略不是在端口 80 上进行的所有请求。

### <a name="internet-access-recommendations"></a>有关 Internet 访问的建议

使用站点到站点 VPN 隧道通过本地网络[强制隧道][azure-forced-tunneling]传递所有出站 Internet 流量，并使用网络地址转换 (NAT) 路由到 Internet。 这可以防止任何存储在数据层中的机密信息的意外泄漏，并允许检查和审核所有传出流量。

> [!NOTE]
> 请勿完全阻止来自应用层的 Internet 流量，因为这会阻止这些层使用依赖于公共 IP 地址的 Azure PaaS 服务（如 VM 诊断日志记录、VM 扩展的下澡和其他功能）。 Azure 诊断还要求组件可以读取和写入 Azure 存储帐户。
> 
> 

验证出站 Internet 流量是否正确地强制隧道传递。 如果在本地服务器上使用 VPN 连接与[路由和远程访问服务][routing-and-remote-access-service]，请使用诸如 [WireShark][wireshark] 或 [Microsoft 消息分析器](https://www.microsoft.com/download/details.aspx?id=44226)这类工具。

### <a name="management-subnet-recommendations"></a>有关管理子网的建议

管理子网包含执行管理和监视功能的 jumpbox。 将所有安全管理任务的执行限制到 jumpbox。
 
请勿为 jumpbox 创建公共 IP 地址。 而是创建一个路由以通过传入网关访问 jumpbox。 创建 NSG 规则，以便管理子网仅响应来自允许的路由的请求。

## <a name="scalability-considerations"></a>可伸缩性注意事项

参考体系结构使用负载均衡器将本地网络流量定向到 NVA 设备池，这些设备会对流量进行路由。 NVA 放置在[可用性集][availability-set]中。 此设计使你可以随着时间推移监视 NVA 的吞吐量，以及添加 NVA 设备以响应负载增加。

标准 SKU VPN 网关支持最多 100 Mbps 的持续吞吐量。 高性能 SKU 提供最多 200 Mbps。 对于更高带宽，请考虑升级到 ExpressRoute 网关。 ExpressRoute 提供最多 10 Gbps 的带宽，且延迟低于 VPN 连接。

有关 Azure 网关的可伸缩性的详细信息，请参阅[使用 Azure 和本地 VPN 实现混合网络体系结构][guidance-vpn-gateway-scalability]和[使用 Azure ExpressRoute 实现混合网络体系结构][guidance-expressroute-scalability]中的可伸缩性注意事项部分。

## <a name="availability-considerations"></a>可用性注意事项

如上所述，参考体系结构使用位于负载均衡器后面的 NVA 设备池。 负载均衡器使用运行状况探测监视每个 NVA，并且会从池中删除任何无法响应的 NVA。

如果要使用 Azure ExpressRoute 在 VNet 与本地网络之间提供连接，请[配置 VPN 网关以提供故障转移][ra-vpn-failover]（在 ExpressRoute 连接成为不可用状态时）。

有关为 VPN 和 ExpressRoute 连接保持可用性的特定信息，请参阅[使用 Azure 和本地 VPN 实现混合网络体系结构][guidance-vpn-gateway-availability]和[使用 Azure ExpressRoute 实现混合网络体系结构][guidance-expressroute-availability]中的可用性注意事项。 

## <a name="manageability-considerations"></a>可管理性注意事项

所有应用程序和资源监视都应由管理子网中的 jumpbox 执行。 根据应用程序要求，可能需要管理子网中的其他监视资源。 如果是这样，则应通过 jumpbox 访问这些资源。

如果从本地网络到 Azure 的网关连接断开，仍可以通过部署公共 IP 地址，将该地址添加到 Jumpbox，然后从 Internet 远程访问，来连接 Jumpbox。

参考体系结构中每个层的子网都受 NSG 规则保护。 可能需要创建一个规则以打开用于在 Windows VM 上进行远程桌面协议 (RDP) 访问的端口 3389，或是用于在 Linux VM 上进行安全外壳 (SSH) 访问的端口 22。 其他管理和监视工具可能需要规则打开其他端口。

如果要使用 ExpressRoute 在本地数据中心与 Azure 之间提供连接，请使用 [Azure 连接 工具包 (AzureCT)][azurect] 监视和解决连接问题.

可以在文章[使用 Azure 和本地 VPN 实现混合网络体系结构][guidance-vpn-gateway-manageability]和[使用 Azure ExpressRoute 实现混合网络体系结构][guidance-expressroute-manageability]中找到专门针对监视和管理 VPN 和 ExpressRoute 连接的其他信息。

## <a name="security-considerations"></a>安全注意事项

此参考体系结构实现多个安全级别。

### <a name="routing-all-on-premises-user-requests-through-the-nva"></a>通过 NVA 路由所有本地用户请求
网关子网中的 UDR 阻止从本地接收的请求之外的所有用户请求。 UDR 将允许的请求传递到专用 DMZ 子网中的 NVA，如果 NVA 规则允许，则这些请求会传递到应用程序。 可以将其他路由添加到 UDR，但请确保它们不会无意中绕过 NVA 或阻止以管理子网为目标的管理流量。

NVA 前面的负载均衡器还通过忽略在负载均衡规则中未打开的端口上的流量来充当安全设备。 参考体系结构中的负载均衡器仅侦听端口 80 上的 HTTP 请求和端口 443 上的 HTTPS 请求。 记录添加到负载均衡器的任何其他规则，并监视流量以确保不存在安全问题。

### <a name="using-nsgs-to-blockpass-traffic-between-application-tiers"></a>使用 NSG 阻止/传递应用层之间的流量
各层之间的流量使用 NSG 进行限制。 业务层阻止不是源自 Web 层的所有流量，而数据层阻止不是源自业务层的所有流量。 如果要求扩展 NSG 规则以允许对这些层进行更广泛的访问，请针对安全风险权衡这些要求。 每个新入站通道都表示可能会发生意外或有意的数据泄露或应用程序损坏。

### <a name="devops-access"></a>DevOps 访问权限
使用 [RBAC][rbac] 可限制 DevOps 可以在每个层执行的操作。 授予权限时，请使用[最低特权原则][security-principle-of-least-privilege]。 记录所有管理操作并执行定期审核，确保所有配置更改按计划进行。

## <a name="solution-deployment"></a>解决方案部署

[GitHub][github-folder] 上提供了可实施这些建议的参考体系结构部署。 可以遵照以下指导，部署该参考体系结构：

1. 单击下面的按钮：<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-hybrid%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. 在 Azure 门户中打开该链接后，必须输入某些设置的值：   
   * 参数文件中已定义**资源组**名称，因此请选择“新建”，并在文本框中输入 `ra-private-dmz-rg`。
   * 从“位置”下拉框中选择区域。
   * 不要编辑“模板根 URI”或“参数根 URI”文本框。
   * 查看条款和条件，并单击“我同意上述条款和条件”复选框。
   * 单击“购买”按钮。
3. 等待部署完成。
4. 参数文件包括所有 VM 的硬编码管理员用户名和密码，我们强烈建议立即更改。 针对部署中的每个 VM，请在 Azure 门户选择该 VM，并在“支持 + 故障排除”边栏选项卡中单击“重置密码”。 在“模式”下拉框中选择“重置密码”，并选择新**用户名**和**密码**。 单击“更新”按钮保存设置。

## <a name="next-steps"></a>后续步骤

* 了解如何[在 Azure 与 Internet 之间实现 DMZ](secure-vnet-dmz.md)。
* 了解如何实现[高可用性混合网络体系结构][ra-vpn-failover]。
* 有关使用 Azure 管理网络安全的详细信息，请参阅 [Microsoft 云服务和网络安全][cloud-services-network-security]。
* 有关在 Azure 中保护资源的详细信息，请参阅 [Microsoft Azure 安全入门][getting-started-with-azure-security]。 
* 有关解决 Azure 网关连接上的安全问题的其他详细信息，请参阅[使用 Azure 和本地 VPN 实现混合网络体系结构][guidance-vpn-gateway-security]和[使用 Azure ExpressRoute 实现混合网络体系结构][guidance-expressroute-security]。
> 

<!-- links -->

[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azurect]: https://github.com/Azure/NetworkMonitoring/tree/master/AzureCT
[azure-forced-tunneling]: https://azure.microsoft.com/en-gb/documentation/articles/vpn-gateway-forced-tunneling-rm/
[azure-marketplace-nva]: https://azuremarketplace.microsoft.com/marketplace/apps/category/networking
[cloud-services-network-security]: https://azure.microsoft.com/documentation/articles/best-practices-network-security/
[getting-started-with-azure-security]: /azure/security/azure-security-getting-started
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-hybrid
[guidance-expressroute]: ../hybrid-networking/expressroute.md
[guidance-expressroute-availability]: ../hybrid-networking/expressroute.md#availability-considerations
[guidance-expressroute-manageability]: ../hybrid-networking/expressroute.md#manageability-considerations
[guidance-expressroute-security]: ../hybrid-networking/expressroute.md#security-considerations
[guidance-expressroute-scalability]: ../hybrid-networking/expressroute.md#scalability-considerations
[guidance-vpn-gateway]: ../hybrid-networking/vpn.md
[guidance-vpn-gateway-availability]: ../hybrid-networking/vpn.md#availability-considerations
[guidance-vpn-gateway-manageability]: ../hybrid-networking/vpn.md#manageability-considerations
[guidance-vpn-gateway-scalability]: ../hybrid-networking/vpn.md#scalability-considerations
[guidance-vpn-gateway-security]: ../hybrid-networking/vpn.md#security-considerations
[ip-forwarding]: /azure/virtual-network/virtual-networks-udr-overview#ip-forwarding
[ra-expressroute]: ../hybrid-networking/expressroute.md
[ra-n-tier]: ../virtual-machines-windows/n-tier.md
[ra-vpn]: ../hybrid-networking/vpn.md
[ra-vpn-failover]: ../hybrid-networking/expressroute-vpn-failover.md
[rbac]: /azure/active-directory/role-based-access-control-configure
[rbac-custom-roles]: /azure/active-directory/role-based-access-control-custom-roles
[routing-and-remote-access-service]: https://technet.microsoft.com/library/dd469790(v=ws.11).aspx
[security-principle-of-least-privilege]: https://msdn.microsoft.com/library/hdb58b2f(v=vs.110).aspx#Anchor_1
[udr-overview]: /azure/virtual-network/virtual-networks-udr-overview
[visio-download]: https://archcenter.azureedge.net/cdn/dmz-reference-architectures.vsdx
[wireshark]: https://www.wireshark.org/
[0]: ./images/dmz-private.png "保护混合网络体系结构"
