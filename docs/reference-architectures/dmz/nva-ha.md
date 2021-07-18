---
title: 部署高可用性网络虚拟设备
description: 如何部署具有高可用性的网络虚拟设备。
author: telmosampaio
ms.date: 12/06/2016
pnp.series.title: Network DMZ
pnp.series.prev: secure-vnet-dmz
cardTitle: Deploy highly available network virtual appliances
ms.openlocfilehash: fe279eea3f9cb024d6c6c14943013b9b9a87bc9c
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/06/2018
ms.locfileid: "30847741"
---
# <a name="deploy-highly-available-network-virtual-appliances"></a>部署具有高可用性的网络虚拟设备。

本文展示了如何在 Azure 中部署一组网络虚拟设备 (NVA) 以实现高可用性。 NVA 通常用来控制从外围网络（也称为 DMZ）到其他网络或子网的网络流量流。 若要了解如何在 Azure 中实现外围网络，请参阅 [Microsoft cloud services and network security][cloud-security]（Microsoft 云服务和网络安全）。 本文包括了仅用于入口、仅用于出口和同时用于入口和出口的示例体系结构。 

<strong>先决条件：</strong>本文假定读者对 Azure 网络、[Azure 负载均衡器][lb-overview]和[用户定义的路由][udr-overview] (UDR) 有一个基本了解。 


## <a name="architecture-diagrams"></a>体系结构示意图

NVA 可以采用许多不同的体系结构部署到外围网络中。 例如，下图展示了用于入口的[单个 NVA][nva-scenario] 的使用。 

![[0]][0]

在此体系结构中，NVA 会检查所有入站和出站网络流量并且仅会放行符合网络安全规则的流量，从而提供一个安全的网络边界。 不过，因为所有网络流量都必须通过 NVA，这意味着 NVA 是网络中的单一故障点。 如果 NVA 发生故障，则网络流量没有其他路径可用，并且所有后端子网都不可用。

若要使 NVA 高度可用，请将多个 NVA 部署到可用性集中。    

下面的体系结构描述了实现高度可用的 NVA 所需的资源和配置：

| 解决方案 | 优点 | 注意事项 |
| --- | --- | --- |
| [具有第 7 层 NVA 的入口][ingress-with-layer-7] |所有 NVA 节点都是主动的 |需要一个可以终止连接的 NVA 并使用 SNAT</br> 对于来自 Internet 和来自 Azure 的流量，需要单独的一组 NVA </br> 只能用于在 Azure 外部产生的流量 |
| [具有第 7 层 NVA 的出口][egress-with-layer-7] |所有 NVA 节点都是主动的 | 需要一个可以终止连接的 NVA 并实现源网络地址转换 (SNAT)
| [具有第 7 层 NVA 的入口-出口][ingress-egress-with-layer-7] |所有节点都是主动的<br/>能够处理在 Azure 中产生的流量 |需要一个可以终止连接的 NVA 并使用 SNAT<br/>对于来自 Internet 和来自 Azure 的流量，需要单独的一组 NVA |
| [PIP-UDR 切换][pip-udr-switch] |用于所有流量的单组 NVA<br/>可以处理所有流量（没有端口限制规则） |主动-被动<br/>需要故障转移流程 |

## <a name="ingress-with-layer-7-nvas"></a>具有第 7 层 NVA 的入口

下图显示了一个高可用性体系结构，它在面向 Internet 的负载均衡器后实现了一个入口外围网络。 此体系结构设计用于提供到 Azure 工作负荷的连接以用于第 7 层流量，例如 HTTP 或 HTTPS：

![[1]][1]

此体系结构的好处是所有 NVA 都是主动的，并且如果其中一个发生故障，则负载均衡器会将网络流量定向到另一个 NVA。 两个 NVA 都将流量路由到内部负载均衡器，因此，只要有一个 NVA 是主动的，流量便可继续流动。 这些 NVA 是终止用于 Web 层 VM 的 SSL 流量所必需的。 不能扩展这些 NVA 来处理内部流量，因为内部流量需要另一组具有其自己的网络路由的专用 NVA。

> [!NOTE]
> 此体系结构用于 [Azure 与本地数据中心之间的外围网络][dmz-on-prem]参考体系结构和 [Azure 与 Internet 之间的外围网络][dmz-internet]参考体系结构中。 这些参考体系结构每个都包括你可以使用的部署解决方案。 有关详细信息，请参阅以下链接。

## <a name="egress-with-layer-7-nvas"></a>具有第 7 层 NVA 的出口

可以扩展上面的体系结构来为在 Azure 工作负荷中产生的请求提供一个出口外围网络。 下面的体系结构设计用于在外围网络中提供具有高可用性的 NVA 以用于第 7 层流量，例如 HTTP 或 HTTPS：

![[2]][2]

在此体系结构中，在 Azure 中产生的所有流量都被路由到一个外部负载均衡器。 该负载均衡器将传出请求分布到一组 NVA 中。 这些 NVA 使用其各自的公共 IP 地址将流量定向到 Internet。

> [!NOTE]
> 此体系结构用于 [Azure 与本地数据中心之间的外围网络][dmz-on-prem]参考体系结构和 [Azure 与 Internet 之间的外围网络][dmz-internet]参考体系结构中。 这些参考体系结构每个都包括你可以使用的部署解决方案。 有关详细信息，请参阅以下链接。

## <a name="ingress-egress-with-layer-7-nvas"></a>具有第 7 层 NVA 的入口-出口

在上面的两个体系结构中，都有一个单独的用于入口和出口的外围网络。 下面的体系结构演示了如何创建可以同时用于入口和出口的外围网络以用于第 7 层流量，例如 HTTP 或 HTTPS： 

![[4]][4]

在此体系结构中，NVA 处理来自应用程序网关的传入请求。 NVA 还处理负载均衡器的后端池中的工作负荷 VM 发出的传出请求。 因为传入流量是通过应用程序网关进行路由的，并且传出流量是通过负载均衡器进行路由的，所以，NVA 负责维护会话相关性。 也就是说，应用程序网关维护入站和出站请求的映射，因此，它可以将正确的响应转发到原始请求者。 但是，内部负载均衡器无权访问应用程序网关映射，它使用其自己的逻辑将响应发送到 NVA。 负载均衡器可能会将响应发送到起初没有从应用程序网关收到请求的 NVA。 在这种情况下，各个 NVA 必须进行通信并在它们之间传输响应，以便正确的 NVA 可以将响应转发到应用程序网关。

> [!NOTE]
> 你还可以通过确保 NVA 执行入站源网络地址转换 (SNAT) 来解决非对称路由问题。 这会将请求者的原始源 IP 替换为入站流上使用的 NVA 的 IP 地址之一。 这确保可以一次使用多个 NVA，同时保持路由对称性。

## <a name="pip-udr-switch-with-layer-4-nvas"></a>采用第 4 层 NVA 的 PIP-UDR 切换

下面的体系结构展示了具有一个主动和一个被动 NVA 的体系结构。 此体系结构同时处理第 4 层流量的入口和出口： 

![[3]][3]

此体系结构类似于本文中讨论的第一个体系结构。 该体系结构包括了用于接受和筛选传入的第 4 层请求的单个 NVA。 此体系结构添加了另一个被动 NVA 来提供高可用性。 如果主动 NVA 发生故障，则被动 NVA 将成为主动的，并且 UDR 和 PIP 将更改为指向目前的主动 NVA 上的 NIC。 对 UDR 和 PIP 的这些更改可以手动完成，也可以使用自动化流程来完成。 自动化流程通常是在 Azure 中运行的守护程序或其他监视服务。 它查询主动 NVA 上的运行状态探测，并且在检测到 NVA 故障时执行 UDR 和 PIP 切换。 

上图显示了一个示例 [ZooKeeper][zookeeper] 群集，它提供了一个高可用性守护程序。 在 ZooKeeper 群集内，节点的仲裁选拨一个领导。 如果领导发生故障，则剩余节点将举行选举来选拨新领导。 对于此体系结构，领导节点将执行对 NVA 上的运行状况终结点进行查询的守护程序。 如果 NVA 无法响应运行状况探测，则守护程序将激活被动 NVA。 然后，守护程序将调用 Azure REST API 来从发生故障的 NVA 中删除 PIP 并将其附加到新激活的 NVA。 然后，守护程序将修改 UDR 来指向新激活的 NVA 的内部 IP 地址。

> [!NOTE]
> 不要将 ZooKeeper 节点包括在只能使用包括 NVA 的路由进行访问的子网中。 否则，如果 NVA 发生故障，则 ZooKeeper 节点将无法访问。 如果守护程序因任何原因而发生故障，则你将无法访问任何 ZooKeeper 节点来诊断问题。 

<!--### Solution Deployment-->

<!-- instructions for deploying this solution here --> 

## <a name="next-steps"></a>后续步骤
* 了解如何使用第 7 层 NVA [在 Azure 与本地数据中心之间实现外围网络][dmz-on-prem]。
* 了解如何使用第 7 层 NVA [在 Azure 与 Internet 之间实现外围网络][dmz-internet]。

<!-- links -->
[cloud-security]: /azure/best-practices-network-security
[dmz-on-prem]: ./secure-vnet-hybrid.md
[dmz-internet]: ./secure-vnet-dmz.md
[egress-with-layer-7]: #egress-with-layer-7-nvas
[ingress-with-layer-7]: #ingress-with-layer-7-nvas
[ingress-egress-with-layer-7]: #ingress-egress-with-layer-7-nvas
[lb-overview]: /azure/load-balancer/load-balancer-overview/
[nva-scenario]: /azure/virtual-network/virtual-network-scenario-udr-gw-nva/
[pip-udr-switch]: #pip-udr-switch-with-layer-4-nvas
[udr-overview]: /azure/virtual-network/virtual-networks-udr-overview/
[zookeeper]: https://zookeeper.apache.org/

<!-- images -->
[0]: ./images/nva-ha/single-nva.png "单 NVA 体系结构"
[1]: ./images/nva-ha/l7-ingress.png "第 7 层入口"
[2]: ./images/nva-ha/l7-ingress-egress.png "第 7 层出口"
[3]: ./images/nva-ha/active-passive.png "主动-被动群集"
[4]: ./images/nva-ha/l7-ingress-egress-ag.png
