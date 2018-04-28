---
title: 在 Azure 中实现中心辐射型网络拓扑
description: 如何在 Azure 中实现中心辐射型网络拓扑。
author: telmosampaio
ms.date: 04/09/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: f04af90f328a0434d44ca7ea90309f3209a3b69d
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/16/2018
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a>在 Azure 中实现中心辐射型网络拓扑

此参考体系结构展示了如何在 Azure 中实现中心辐射型拓扑。 *中心*是 Azure 中的一个虚拟网络 (VNet)，充当到本地网络的连接的中心点。 *辐射*是与中心对等互连的 VNet，可用于隔离工作负荷。 流量通过 ExpressRoute 或 VPN 网关连接在本地数据中心与中心之间流动。  [**部署此解决方案**](#deploy-the-solution)。

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

* **网关子网**。 虚拟网络网关保留在同一子网中。

* **辐射 VNet**。 用作中心辐射型拓扑中的辐射的一个或多个 Azure VNet。 辐射可以用来隔离其自己的 VNet 中的工作负荷，独立于其他辐射进行管理。 每个工作负荷可以包括多个层，并具有通过 Azure 负载均衡器连接的多个子网。 有关应用程序基础结构的详细信息，请参阅[运行 Windows VM 工作负荷][windows-vm-ra]和[运行 Linux VM 工作负荷][linux-vm-ra]。

* **VNet 对等互连**。 可以使用[对等互连连接][vnet-peering]来连接同一 Azure 区域中的两个 VNet。 对等互连连接是 VNet 之间的不可传递低延迟连接。 进行对等互连后，VNet 可使用 Azure 主干交换流量，不需要使用路由器。 在中心辐射型网络拓扑中，将使用 VNet 对等互连来将中心连接到每个辐射。

> [!NOTE]
> 文本仅涵盖了[资源管理器](/azure/azure-resource-manager/resource-group-overview)部署，但也可以将经典 VNet 连接到同一订阅中的资源管理器 VNet。 这样，辐射将可以托管经典部署，并且仍然可以从中心内共享的各种服务受益。

## <a name="recommendations"></a>建议

以下建议适用于大多数方案。 除非有优先于这些建议的特定要求，否则请遵循这些建议。

### <a name="resource-groups"></a>资源组

中心 VNet 和每个辐射 VNet 可以在不同的资源组中实现，甚至可以在不同的订阅中实现，只要它们属于同一 Azure 区域中的同一 Azure Active Directory (Azure AD) 租户即可。 这样，可以对各个工作负荷进行非集中管理，同时在中心 VNet 内维护共享服务。

### <a name="vnet-and-gatewaysubnet"></a>VNet 和 GatewaySubnet

创建一个名为 *GatewaySubnet* 的子网，使其地址范围为 /27。 此子网是虚拟网络网关所必需的。 向此子网分配 32 个地址将有助于防止将来达到网关大小限制。

有关设置网关的详细信息，请根据你的连接类型参阅以下参考体系结构：

- [使用 ExpressRoute 的混合网络][guidance-expressroute]
- [使用 VPN 网关的混合网络][guidance-vpn]

要实现更高的可用性，可以将 ExpressRoute 外加 VPN 用于故障转移。 请参阅[将本地网络连接到 Azure 并将 ExpressRoute 和 VPN 用于故障转移][hybrid-ha]。

如果不需要与本地网络的连接，还可以在不使用网关的情况下使用中心辐射型拓扑。 

### <a name="vnet-peering"></a>VNet 对等互连

VNet 对等互连是两个 VNet 之间的不可传递关系。 如果需要将各个辐射彼此连接，请考虑在这些辐射之间添加一个单独的对等互连连接。

不过，如果有多个辐射需要彼此连接，则你可能会由于[每个 VNet 的 VNet 对等互连数限制][vnet-peering-limit]而很快耗尽可能的对等互连连接。 在这种情况下，请考虑使用用户定义的路由 (UDR) 强制将目的地为辐射的流量发送到在中心 VNet 中充当路由器的 NVA。 这将允许各个辐射彼此连接。

还可以将辐射配置为使用中心 VNet 网关与远程网络进行通信。 若要允许网关流量从辐射流动到中心，以及允许连接到远程网络，必须：

  - 在中心内配置 VNet 对等互连连接以**允许网关中转**。
  - 在每个辐射中配置 VNet 对等互连连接以**使用远程网关**。
  - 配置所有 VNet 对等互连连接以**允许转发的流量**。

## <a name="considerations"></a>注意事项

### <a name="spoke-connectivity"></a>辐射连接

如果辐射之间需要存在连接，请考虑在中心内实现一个用于路由的 NVA，并在辐射中使用 UDR 将流量转发到中心。

![[2]][2]

在这种情况下，必须配置对等互连连接以**允许转发的流量**。

### <a name="overcoming-vnet-peering-limits"></a>克服 VNet 对等互连限制

请务必考虑 Azure 中[每个 VNet 的 VNet 对等互连数限制][vnet-peering-limit]。 如果你确定所需的辐射多于限制将允许的数量，请考虑创建一个中心-辐射-中心-辐射型拓扑，其中的第一级辐射还充当中心。 下图显示了此方式。

![[3]][3]

另外，请考虑要在中心内共享哪些服务，以确保中心能够针对大量辐射进行缩放。 例如，如果中心提供防火墙服务，则在添加多个辐射时请考虑防火墙解决方案的带宽限制。 你可能希望将这些共享服务中的某一些移动到二级中心内。

## <a name="deploy-the-solution"></a>部署解决方案

[GitHub][ref-arch-repo] 上提供了此体系结构的部署。 该部署使用每个 VNet 中的 VM 来测试连接。 没有实际服务托管在 **中心 VNet** 内的**共享服务**中。

该部署在订阅中创建以下资源组：

- hub-nva-rg
- hub-vnet-rg
- onprem-jb-rg
- onprem-vnet-rg
- spoke1-vnet-rg
- spoke2-vent-rg

模板参数文件将引用这些名称，因此，如果更改了这些名称，请相应地更新参数文件。

### <a name="prerequisites"></a>先决条件

1. 克隆、下载[参考体系结构][ref-arch-repo] GitHub 存储库的 zip 文件或创建其分支。

2. 安装 [Azure CLI 2.0][azure-cli-2]。

3. 安装 [Azure 构建基块][azbb] npm 包。

4. 在命令提示符、bash 提示符或 PowerShell 提示符下使用以下命令登录到 Azure 帐户。

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a>部署模拟的本地数据中心

若要将模拟的本地数据中心部署为 Azure VNet，请执行以下步骤：

1. 导航到参考体系结构存储库的 `hybrid-networking/hub-spoke` 文件夹。

2. 打开 `onprem.json` 文件。 替换 `adminUsername` 和 `adminPassword` 的值。

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

3. （可选）对于 Linux 部署，请将 `osType` 设置为 `Linux`。

4. 运行以下命令：

    ```bash
    azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
    ```

5. 等待部署完成。 此部署创建虚拟网络、虚拟机和 VPN 网关。 创建 VPN 网关可能需要花费大约 40 分钟。

### <a name="deploy-the-hub-vnet"></a>部署中心 VNet

若要部署中心 VNet，请执行以下步骤。

1. 打开 `hub-vnet.json` 文件。 替换 `adminUsername` 和 `adminPassword` 的值。

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. （可选）对于 Linux 部署，请将 `osType` 设置为 `Linux`。

3. 对于 `sharedKey`，请输入 VPN 连接的共享密钥。 

    ```bash
    "sharedKey": "",
    ```

4. 运行以下命令：

    ```bash
    azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
    ```

5. 等待部署完成。 此部署创建虚拟网络、虚拟机、VPN 网关和网关连接。  创建 VPN 网关可能需要花费大约 40 分钟。

### <a name="test-connectivity-with-the-hub"></a>测试与中心的连接

测试从模拟本地环境到中心 VNet 的连接。

**Windows 部署**

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

**Linux 部署**

1. 使用 Azure 门户在 `onprem-jb-rg` 资源组中找到名为 `jb-vm1` 的 VM。

2. 单击 `Connect`，并复制门户中显示的 `ssh` 命令。 

3. 在 Linux 提示符下，运行 `ssh` 连接到模拟本地环境。 使用 `onprem.json` 参数文件中指定的密码。

4. 使用 `ping` 命令测试与中心 VNet 中 Jumpbox VM 连接。

   ```bash
   ping 10.0.0.68
   ```

### <a name="deploy-the-spoke-vnets"></a>部署辐射 VNet

若要部署辐射 VNet，请执行以下步骤。

1. 打开 `spoke1.json` 文件。 替换 `adminUsername` 和 `adminPassword` 的值。

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. （可选）对于 Linux 部署，请将 `osType` 设置为 `Linux`。

3. 运行以下命令：

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```
  
4. 针对 `spoke2.json` 文件重复步骤 1-2。

5. 运行以下命令：

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

6. 运行以下命令：

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
   ```

### <a name="test-connectivity"></a>测试连接

测试从模拟本地环境到辐射 VNet 的连接。

**Windows 部署**

1. 使用 Azure 门户在 `onprem-jb-rg` 资源组中找到名为 `jb-vm1` 的 VM。

2. 单击 `Connect` 来与 VM 建立远程桌面会话。 使用 `onprem.json` 参数文件中指定的密码。

3. 在 VM 中打开 PowerShell 控制台，使用 `Test-NetConnection` cmdlet 验证能否连接到中心 VNet 中的 Jumpbox VM。

   ```powershell
   Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
   Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
   ```

**Linux 部署**

若要使用 Linux VM 测试从模拟的本地环境到辐射 VNet 的连接，请执行以下步骤：

1. 使用 Azure 门户在 `onprem-jb-rg` 资源组中找到名为 `jb-vm1` 的 VM。

2. 单击 `Connect`，并复制门户中显示的 `ssh` 命令。 

3. 在 Linux 提示符下，运行 `ssh` 连接到模拟本地环境。 使用 `onprem.json` 参数文件中指定的密码。

5. 使用 `ping` 命令测试与每个辐射中 Jumpbox VM 的连接。

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

### <a name="add-connectivity-between-spokes"></a>添加辐射之间的连接

此步骤是可选的。 如果需要允许辐射 VNet 互相进行连接，则必须使用网络虚拟设备 (NVA) 作为中心 VNet 中的路由器，并在尝试连接到另一辐射 VNet 时强制流量从辐射 VNet 流向路由器。 若要将基本的示例 NVA 部署为单个 VM 并部署用户定义的路由 (UDR) 以允许这两个辐射 VNet 建立连接，请执行以下步骤：

1. 打开 `hub-nva.json` 文件。 替换 `adminUsername` 和 `adminPassword` 的值。

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. 运行以下命令：

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Azure 中的中心辐射型拓扑"
[1]: ./images/hub-spoke-gateway-routing.svg "Azure 中的具有可传递路由的中心辐射型拓扑"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Azure 中的具有使用 NVA 的可传递路由的中心辐射型拓扑"
[3]: ./images/hub-spokehub-spoke.svg "Azure 中的中心-辐射-中心-辐射型拓扑"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
