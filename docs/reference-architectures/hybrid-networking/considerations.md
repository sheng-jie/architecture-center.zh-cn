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
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a>选择用于将本地网络连接到 Azure 的解决方案

本文比较了用于将本地网络连接到 Azure 虚拟网络 (VNet) 的选项。 我们为每个选项提供了参考体系结构和可部署的解决方案。

## <a name="vpn-connection"></a>VPN 连接

使用虚拟专用网络 (VPN) 可通过 IPSec VPN 隧道将本地网络与 Azure VNet 进行连接。

此体系结构适合满足以下条件的混合应用程序：本地硬件与云之间的流量可能较小，或者用户愿意用略微延长的延迟换得云的灵活性和处理能力。

**优点**

- 易于配置。

**挑战**

- 需要本地 VPN 设备。
- 尽管 Microsoft 保证每个 VPN 网关 99.9% 的可用性，但此 SLA 仅涵盖 VPN 网关，并不涉及与网关之间的网络连接。
- 通过 Azure VPN 网关的 VPN 连接当前最多支持 200 Mbps 的带宽。 如果预期会超过此吞吐量，可能需要跨多个 VPN 连接对 Azure 虚拟网络进行分区。

**[了解详细信息...][vpn]**

## <a name="azure-expressroute-connection"></a>Azure ExpressRoute 连接

ExpressRoute 连接通过第三方连接提供商使用私有的专用连接。 该专用连接将本地网络扩展到 Azure 中。 

此体系结构适合满足以下条件的混合应用程序：运行需要较高程度可伸缩性的大规模、任务关键型工作负荷。 

**优点**

- 有很高的带宽可用；最高可达 10 Gbps，具体取决于连接提供商。
- 支持动态缩放带宽以帮助在需求较低的时段降低成本。 但是，并非所有连接提供商都提供此选项。
- 可能会允许组织直接访问国家/地区云，具体取决于连接提供商。
- 跨整个连接的 99.9% 可用性 SLA。

**挑战**

- 设置可能复杂。 创建 ExpressRoute 连接需要使用第三方连接提供商。 该提供商负责预配网络连接。
- 需要本地高带宽路由器。

**[了解详细信息...][expressroute]**

## <a name="expressroute-with-vpn-failover"></a>使用 ExpressRoute 和 VPN 实现故障转移

此选项合并了前面两个选项，在正常情况下使用 ExpressRoute，但在 ExpressRoute 线路中发生连接丢失时故障转移到 VPN 连接。

此体系结构适合需要 ExpressRoute 的较高带宽并且还需要高度可用的网络连接的混合应用程序。 

**优点**

- 在 ExpressRoute 线路出现故障时具有高可用性，虽然回退连接位于带宽较低的网络上。

**挑战**

- 配置复杂。 需要设置 VPN 连接和 ExpressRoute 线路。
- 需要冗余硬件（VPN 设备），以及你需要为其付费的冗余 Azure VPN 网关连接。

**[了解详细信息...][expressroute-vpn-failover]**

<!-- links -->
[expressroute]: ./expressroute.md
[expressroute-vpn-failover]: ./expressroute-vpn-failover.md
[vpn]: ./vpn.md