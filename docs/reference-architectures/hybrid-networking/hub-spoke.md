---
title: "在 Azure 中实现中心辐射型网络拓扑"
description: "如何在 Azure 中实现中心辐射型网络拓扑。"
author: telmosampaio
ms.date: 02/14/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: c03ecd4ba5ddbe50cfb17e56d75c18102b751cfb
ms.sourcegitcommit: 475064f0a3c2fac23e1286ba159aaded287eec86
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/19/2018
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

* **网关子网**。 各个虚拟网络网关都放在同一子网中。

* **共享服务子网**。 中心 VNet 中的子网，用来托管可以在所有辐射之间共享的服务，例如 DNS 或 AD DS。

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

[GitHub][ref-arch-repo] 上提供了此体系结构的部署。 它使用每个 VNet 中的 Ubuntu VM 来测试连接。 没有实际服务托管在 **中心 VNet** 内的**共享服务**中。

### <a name="prerequisites"></a>先决条件

在将参考体系结构部署到自己的订阅之前，必须执行以下步骤。

1. 为 [AzureCAT 参考体系结构][ref-arch-repo] GitHub 存储库克隆、下载 zip 文件或创建其分支。

2. 如果希望使用 Azure CLI，请确保在计算机上安装 Azure CLI 2.0。 若要安装 CLI，请按照[安装 Azure CLI 2.0][azure-cli-2] 中的说明执行操作。

3. 如果希望使用 PowerShell，请确保在计算机上安装适用于 Azure 的最新 PowerShell 模块。 若要安装最新 Azure PowerShell 模块，请按照 [Install PowerShell for Azure][azure-powershell]（安装适用于 Azure 的 PowerShell）中的说明进行操作。

4. 从命令提示符、bash 提示符或 PowerShell 提示符下通过使用以下命令之一，登录到 Azure 帐户，然后按照提示进行操作。

  ```bash
  az login
  ```

  ```powershell
  Login-AzureRmAccount
  ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a>部署模拟的本地数据中心

若要将模拟的本地数据中心部署为 Azure VNet，请执行以下步骤。

1. 导航到已在前面的先决条件步骤中下载的存储库的 `hybrid-networking\hub-spoke\onprem` 文件夹。

2. 打开 `onprem.vm.parameters.json` 文件，在第 11 行和第 12 行中，输入括在引号中的用户名和密码，如下所示，然后保存文件。

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. 运行下面的 bash 或 PowerShell 命令来将模拟的本地环境部署为 Azure 中的 VNet。 将各个值替换为你的订阅、资源组名称和 Azure 区域。

  ```bash
  sh ./onprem.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > 如果决定使用其他资源组名称（而非 `ra-onprem-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。

4. 等待部署完成。 此部署将创建一个虚拟网络、一个运行 Ubuntu 的虚拟机和一个 VPN 网关。 VPN 网关创建可能需要 40 多分钟才能完成。

### <a name="azure-hub-vnet"></a>Azure 中心 VNet

若要部署中心 VNet 并连接到前面创建的模拟的本地 VNet，请执行以下步骤。

1. 导航到已在前面的先决条件步骤中下载的存储库的 `hybrid-networking\hub-spoke\hub` 文件夹。

2. 打开 `hub.vm.parameters.json` 文件，在第 11 行和第 12 行中，输入括在引号中的用户名和密码，如下所示，然后保存文件。

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. 打开 `hub.gateway.parameters.json` 文件，在第 23 行中，输入括在引号中的共享密钥，如下所示，然后保存文件。 请记下此值，稍后在部署中需要使用此值。

  ```bash
  "sharedKey": "",
  ```

4. 运行下面的 bash 或 PowerShell 命令来将模拟的本地环境部署为 Azure 中的 VNet。 将各个值替换为你的订阅、资源组名称和 Azure 区域。

  ```bash
  sh ./hub.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus
  ```

  ```powershell
  ./hub.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus
  ```
  > [!NOTE]
  > 如果决定使用其他资源组名称（而非 `ra-hub-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。

5. 等待部署完成。 此部署将创建一个虚拟网络、一个运行 Ubuntu 的虚拟机、一个 VPN 网关和到上一部分中创建的网关的连接。 VPN 网关创建可能需要 40 多分钟才能完成。

### <a name="connection-from-on-premises-to-the-hub"></a>从本地到中心的连接

若要从模拟的本地数据中心连接到中心 VNet，请执行以下步骤。

1. 导航到已在前面的先决条件步骤中下载的存储库的 `hybrid-networking\hub-spoke\onprem` 文件夹。

2. 打开 `onprem.connection.parameters.json` 文件，在第 9 行中，输入括在引号中的共享密钥，如下所示，然后保存文件。 此共享密钥值必须与之前部署的本地网关中使用的共享密钥值相同。

  ```bash
  "sharedKey": "",
  ```

3. 运行下面的 bash 或 PowerShell 命令来将模拟的本地环境部署为 Azure 中的 VNet。 将各个值替换为你的订阅、资源组名称和 Azure 区域。

  ```bash
  sh ./onprem.connection.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.connection.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > 如果决定使用其他资源组名称（而非 `ra-onprem-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。

4. 等待部署完成。 此部署将在用来模拟本地数据中心的 VNet 与中心 VNet 之间创建一个连接。

### <a name="azure-spoke-vnets"></a>Azure 辐射 VNet

若要部署辐射 VNet 并连接到前面创建的中心 VNet，请执行以下步骤。

1. 切换到已在前面的先决条件步骤中下载的存储库的 `hybrid-networking\hub-spoke\spokes` 文件夹。

2. 打开 `spoke1.web.parameters.json` 文件，在第 53 行和第 54 行中，输入括在引号中的用户名和密码，如下所示，然后保存文件。

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. 针对文件 `spoke2.web.parameters.json` 重复上面的步骤。

4. 运行下面的 bash 或 PowerShell 命令来部署第一个辐射并将其连接到中心。 将各个值替换为你的订阅、资源组名称和 Azure 区域。

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```
  > [!NOTE]
  > 如果决定使用其他资源组名称（而非 `ra-spoke1-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。

5. 等待部署完成。 此部署将创建一个虚拟网络、一个包含运行 Ubuntu 和 Apache 的三个虚拟机的负载均衡器，以及到前面部分中创建的中心 VNet 的 VNet 对等互连连接。 此部署可能需要花费 20 多分钟。

6. 运行下面的 bash 或 PowerShell 命令来部署第一个辐射并将其连接到中心。 将各个值替换为你的订阅、资源组名称和 Azure 区域。

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```
  > [!NOTE]
  > 如果决定使用其他资源组名称（而非 `ra-spoke2-rg`），请确保搜索使用了该名称的所有参数文件并对其进行编辑以使用你自己的资源组名称。

5. 等待部署完成。 此部署将创建一个虚拟网络、一个包含运行 Ubuntu 和 Apache 的三个虚拟机的负载均衡器，以及到前面部分中创建的中心 VNet 的 VNet 对等互连连接。 此部署可能需要花费 20 多分钟。

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a>Azure 中心 VNet 到辐射 VNet 的对等互连

若要部署中心 VNet 的 VNet 对等互连连接，请执行以下步骤。

1. 切换到已在前面的先决条件步骤中下载的存储库的 `hybrid-networking\hub-spoke\hub` 文件夹。

2. 运行下面的 bash 或 PowerShell 命令来部署到第一个辐射的对等互连连接。 将各个值替换为你的订阅、资源组名称和 Azure 区域。

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 1
  ```

2. 运行下面的 bash 或 PowerShell 命令来部署到第二个辐射的对等互连连接。 将各个值替换为你的订阅、资源组名称和 Azure 区域。

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 2
  ```

### <a name="test-connectivity"></a>测试连接

若要验证连接到本地数据中心部署的中心辐射型拓扑是否正常工作，请执行以下步骤。

1. 从 [Azure 门户] [门户] 中，连接到你的订阅，并导航到 `ra-onprem-rg` 资源组中的 `ra-onprem-vm1` 虚拟机。

2. 在 `Overview` 边栏选项卡中，记下 VM 的 `Public IP address`。

3. 使用 SSH 客户端并使用在部署期间指定的用户名和密码登录到在上面记下的 IP 地址。

4. 在连接到的 VM 上，从命令提示符下运行以下命令来测试从本地 VNet 到 Spoke1 VNet 的连接。

  ```bash
  ping 10.1.1.37
  ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azure-powershell]: /powershell/azure/install-azure-ps?view=azuresmps-3.7.0
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

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Azure 中的中心辐射型拓扑"
[1]: ./images/hub-spoke-gateway-routing.svg "Azure 中的具有可传递路由的中心辐射型拓扑"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Azure 中的具有使用 NVA 的可传递路由的中心辐射型拓扑"
[3]: ./images/hub-spokehub-spoke.svg "Azure 中的中心-辐射-中心-辐射型拓扑"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
