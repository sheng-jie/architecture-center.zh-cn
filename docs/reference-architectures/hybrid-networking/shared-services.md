---
title: 在 Azure 中使用共享服务实现中心辐射型网络拓扑
description: 如何在 Azure 中使用共享服务实现中心辐射型网络拓扑。
author: telmosampaio
ms.date: 06/19/2018
pnp.series.title: Implement a hub-spoke network topology with shared services in Azure
pnp.series.prev: hub-spoke
ms.openlocfilehash: 283251d5b11f76985405410c5c237e5a64ee98fe
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060789"
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a>在 Azure 中使用共享服务实现中心辐射型网络拓扑

此参考体系结构在[中心辐射型][guidance-hub-spoke]参考体系结构的基础上生成，在中心包括了可供所有辐射 VNet 使用的共享服务。 首先需要共享第一批服务，即标识和安全性，然后才能将数据中心迁移到云并生成[虚拟数据中心]。 此参考体系结构显示了如何将 Active Directory 服务从本地数据中心扩展到 Azure，以及如何在中心辐射型拓扑中添加可以充当防火墙的网络虚拟设备 (NVA)。  [**部署此解决方案**](#deploy-the-solution)。

![[0]][0]

*下载此体系结构的 [Visio 文件][visio-download]*

此拓扑的好处包括：

* **节省成本** - 通过将可以由多个工作负荷（例如网络虚拟设备 (NVAs) 和 DNS 服务器）共享的服务集中放置在单个位置中。
* **克服订阅限制** - 通过将不同订阅中的 Vnet 对等互连到中心。
* **关注点隔离**（在中心 IT（SecOps、InfraOps）与工作负荷 (DevOps) 之间）。

此体系结构的典型用途包括：

* 在各种环境（例如开发、测试和生产）中部署的需要使用共享服务（例如 DNS、IDS、NTP 或 AD DS）的工作负荷。 共享服务放置在中心 VNet 中，而每个环境都部署到辐射以保持隔离。
* 不需要彼此连接但需要访问共享服务的工作负荷。
* 需要对安全方面进行集中控制（例如作为外围网络的中心内的防火墙），并且需要在每个辐射中对工作负荷进行隔离管理的企业。

## <a name="architecture"></a>体系结构

该体系结构包括以下组件。

* **本地网络**。 在组织内运行的一个专用局域网。

* **VPN 设备**。 提供到本地网络的外部连接的设备或服务。 VPN 设备可以是硬件设备，也可以是软件解决方案，例如 Windows Server 2012 中的路由和远程访问服务 (RRAS)。 有关受支持 VPN 设备的列表和有关为连接到 Azure 而配置所选 VPN 设备的信息，请参阅 [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance]（关于站点到站点 VPN 网关连接的 VPN 设备）。

* **VPN 虚拟网络网关或 ExpressRoute 网关**。 虚拟网络网关可以将 VNet 连接到用于本地网络连接的 VPN 设备或 ExpressRoute 线路。 有关详细信息，请参阅[将本地网络连接到 Microsoft Azure 虚拟网络][connect-to-an-Azure-vnet]。

> [!NOTE]
> 此参考体系结构的部署脚本使用 VPN 网关进行连接，使用 Azure 中的 VNet 来模拟本地网络。

* **中心 VNet**。 用作中心辐射型拓扑中的中心的 Azure VNet。 中心是到本地网络的连接的中心点，它还托管着可以由辐射 VNet 中托管的各种工作负荷使用的服务。

* **网关子网**。 各个虚拟网络网关都放在同一子网中。

* **共享服务子网**。 中心 VNet 中的子网，用来托管可以在所有辐射之间共享的服务，例如 DNS 或 AD DS。

* **外围网络子网**。 中心 VNet 中的子网，用于托管的 NVA 可以充当安全设备，例如防火墙。

* **辐射 VNet**。 用作中心辐射型拓扑中的辐射的一个或多个 Azure VNet。 辐射可以用来隔离其自己的 VNet 中的工作负荷，独立于其他辐射进行管理。 每个工作负荷可以包括多个层，并具有通过 Azure 负载均衡器连接的多个子网。 有关应用程序基础结构的详细信息，请参阅[运行 Windows VM 工作负荷][windows-vm-ra]和[运行 Linux VM 工作负荷][linux-vm-ra]。

* **VNet 对等互连**。 可以使用[对等互连连接][vnet-peering]来连接同一 Azure 区域中的两个 VNet。 对等互连连接是 VNet 之间的不可传递低延迟连接。 进行对等互连后，VNet 可使用 Azure 主干交换流量，不需要使用路由器。 在中心辐射型网络拓扑中，将使用 VNet 对等互连来将中心连接到每个辐射。

> [!NOTE]
> 文本仅涵盖了[资源管理器](/azure/azure-resource-manager/resource-group-overview)部署，但也可以将经典 VNet 连接到同一订阅中的资源管理器 VNet。 这样，辐射将可以托管经典部署，并且仍然可以从中心内共享的各种服务受益。

## <a name="recommendations"></a>建议

针对[中心辐射型][guidance-hub-spoke]参考体系结构的所有建议也适用于共享服务参考体系结构。 

另外，以下建议适用于共享服务中的大多数方案。 除非有优先于这些建议的特定要求，否则请遵循这些建议。

### <a name="identity"></a>标识

大多数企业组织在其本地数据中心都有 Active Directory 目录服务 (ADDS) 环境。 为了便于管理从本地网络移到 Azure 且依赖于 ADDS 的资产，建议将 ADDS 域控制器托管在 Azure 中。

如果使用需要针对 Azure 和本地环境分开进行控制的组策略对象，请针对每个 Azure 区域使用不同的 AD 站点。 将域控制器置于可供依赖性工作负荷访问的中心 VNet（简称“中心”）。

### <a name="security"></a>“安全”

将工作负荷从本地环境移到 Azure 时，其中的某些工作负荷将需要托管在 VM 中。 出于符合性考虑，可能需要对流经这些工作负荷的流量强制实施限制。 

可以在 Azure 中使用网络虚拟设备 (NVA) 来托管各类安全和性能服务。 如果熟悉当前在本地使用的特定设备组，建议在 Azure 中也使用相同的虚拟化设备，如果适用的话。

> [!NOTE]
> 针对此参考体系结构的部署脚本使用已启用 IP 转发功能的 Ubuntu VM，以便模拟网络虚拟设备。

## <a name="considerations"></a>注意事项

### <a name="overcoming-vnet-peering-limits"></a>克服 VNet 对等互连限制

请务必考虑 Azure 中[每个 VNet 的 VNet 对等互连数限制][vnet-peering-limit]。 如果你确定所需的辐射多于限制将允许的数量，请考虑创建一个中心-辐射-中心-辐射型拓扑，其中的第一级辐射还充当中心。 下图显示了此方式。

![[3]][3]

另外，请考虑要在中心内共享哪些服务，以确保中心能够针对大量辐射进行缩放。 例如，如果中心提供防火墙服务，则在添加多个辐射时请考虑防火墙解决方案的带宽限制。 你可能希望将这些共享服务中的某一些移动到二级中心内。

## <a name="deploy-the-solution"></a>部署解决方案

[GitHub][ref-arch-repo] 上提供了此体系结构的部署。 该部署在订阅中创建以下资源组：

- hub-adds-rg
- hub-nva-rg
- hub-vnet-rg
- onprem-vnet-rg
- spoke1-vnet-rg
- spoke2-vent-rg

模板参数文件将引用这些名称，因此，如果更改了这些名称，请相应地更新参数文件。

### <a name="prerequisites"></a>先决条件

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a>使用 azbb 部署模拟的本地数据中心

此步骤将模拟的本地数据中心部署为 Azure VNet。

1. 导航到 GitHub 存储库的 `hybrid-networking\shared-services-stack\` 文件夹。

2. 打开 `onprem.json` 文件。 

3. 搜索 `Password` 和 `adminPassword` 的所有实例。 在参数中输入用户名和密码的值，然后保存文件。 

4. 运行以下命令：

   ```bash
   azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
   ```
5. 等待部署完成。 此部署创建虚拟网络、运行 Windows 的虚拟机和 VPN 网关。 VPN 网关创建可能需要 40 多分钟才能完成。

### <a name="deploy-the-hub-vnet"></a>部署中心 VNet

此步骤部署中心 VNet 并将其连接到模拟的本地 VNet。

1. 打开 `hub-vnet.json` 文件。 

2. 搜索 `adminPassword`，然后在参数中输入用户名和密码。 

3. 搜索 `sharedKey` 的所有实例，然后输入某个共享密钥的值。 保存文件。

   ```bash
   "sharedKey": "abc123",
   ```

4. 运行以下命令：

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
   ```

5. 等待部署完成。 此部署创建虚拟网络、虚拟机、VPN 网关和到上一部分创建的网关的连接。 VPN 网关可能需要 40 多分钟才能完成。

### <a name="deploy-ad-ds-in-azure"></a>在 Azure 中部署 AD DS

此步骤在 Azure 中部署 AD DS 域控制器。

1. 打开 `hub-adds.json` 文件。

2. 搜索 `Password` 和 `adminPassword` 的所有实例。 在参数中输入用户名和密码的值，然后保存文件。 

3. 运行以下命令：

   ```bash
   azbb -s <subscription_id> -g hub-adds-rg -l <location> -p hub-adds.json --deploy
   ```
  
此部署步骤可能需要数分钟的时间，因为需要将两个 VM 加入到域中，而该域托管在模拟的本地数据中心，然后需要在其上安装 AD DS。

### <a name="deploy-the-spoke-vnets"></a>部署辐射 VNet

此步骤部署辐射 VNet。

1. 打开 `spoke1.json` 文件。

2. 搜索 `adminPassword`，然后在参数中输入用户名和密码。 

3. 运行以下命令：

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```
  
4. 针对 `spoke2.json` 文件重复步骤 1 和 2。

5. 运行以下命令：

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

### <a name="peer-the-hub-vnet-to-the-spoke-vnets"></a>将中心 VNet 与辐射 VNet 对等互连

若要创建从中心 VNet 到辐射 VNet 的对等互连连接，请运行以下命令：

```bash
azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
```

### <a name="deploy-the-nva"></a>部署 NVA

此步骤在 `dmz` 子网中部署 NVA。

1. 打开 `hub-nva.json` 文件。

2. 搜索 `adminPassword`，然后在参数中输入用户名和密码。 

3. 运行以下命令：

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

### <a name="test-connectivity"></a>测试连接 

测试从模拟本地环境到中心 VNet 的连接。

1. 使用 Azure 门户在 `onprem-jb-rg` 资源组中找到名为 `jb-vm1` 的 VM。

2. 单击 `Connect` 来与 VM 建立远程桌面会话。 使用 `onprem.json` 参数文件中指定的密码。

3. 在 VM 中打开 PowerShell 控制台，使用 `Test-NetConnection` cmdlet 验证能否连接到中心 VNet 中的 Jumpbox VM。

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```
输出应如下所示：

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> 默认情况下，Windows Server VM 不允许 Azure 中的 ICMP 响应。 若要使用 `ping` 来测试连接，需在“Windows 高级防火墙”中为每个 VM 启用 ICMP 流量。

重复相同的步骤，测试到辐射 VNet 的连接：

```powershell
Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
```


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

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/shared-services.png "Azure 中的共享服务拓扑"
[3]: ./images/hub-spokehub-spoke.svg "Azure 中的中心-辐射-中心-辐射型拓扑"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
