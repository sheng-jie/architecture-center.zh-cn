---
title: "在 Azure 上运行负载均衡的 VM 以提高可伸缩性和可用性"
description: "如何在 Azure 上运行多个 Linux VM 以提高可伸缩性和可用性。"
author: telmosampaio
ms.date: 09/07/2017
pnp.series.title: Linux VM workloads
pnp.series.next: n-tier
pnp.series.prev: single-vm
ms.openlocfilehash: 0f772cdc596ea96073da3b9b2a2709ce3041889d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="run-load-balanced-vms-for-scalability-and-availability"></a>运行负载均衡的 VM 以提高可伸缩性和可用性

对于在负载均衡器后的规模集中运行多个 Linux 虚拟机 (VM) 以提高可用性和可伸缩性，此参考体系结构显示了一组已经过验证的做法。 此体系结构可用于任何无状态工作负荷（如 Web 服务器），并且是用于部署 n 层应用程序的构建基块。 [**部署此解决方案**。](#deploy-the-solution)

![[0]][0]

下载此体系结构的 [Visio 文件][visio-download]。

## <a name="architecture"></a>体系结构

此体系结构基于[在 Azure 上运行 Linux VM][single vm] 中显示的体系结构构建。 那里的建议也适用于此体系结构。

在此体系结构中，工作负荷分布于多个 VM 实例上。 有单个公共 IP 地址，并且 Internet 流量通过负载均衡器分布到这些 VM。 此体系结构可用于单层应用程序，如无状态 Web 应用程序。

该体系结构具有以下组件：

* **资源组。** [*资源组*][resource-manager-overview]用于对资源进行分组，以便它们可以按生存期、所有者和其他条件进行管理。
* **虚拟网络 (VNet) 和子网。** Azure 中的每个 VM 都部署在 VNet 中，后者进一步划分为多个子网。
* **Azure 负载均衡器**。 [负载均衡器]将传入 Internet 将请求分布到这些 VM 实例。 
* **公共 IP 地址**。 负载均衡器需要使用一个公共 IP 地址来接收 Internet 流量。
* **VM 规模集**。 [VM 规模集][vm-scaleset]是一组用于托管工作负荷的相同 VM。 规模集允许以手动方式或基于预定义规则增加或减少 VM 的数量。
* **可用性集**。 [可用性集][availability set]包含 VM，使 VM 符合更高[服务级别协议 (SLA)][vm-sla] 的要求。 要应用更高级别的 SLA，可用性集必须包含至少两个 VM。 可用性集隐含于规模集中。 如果在规模集外创建 VM，需要独立创建可用性集。
* **托管磁盘**。 Azure 托管磁盘管理 VM 磁盘的虚拟硬盘 (VHD) 文件。 
* **存储**。 创建 Azure 存储帐户以保存 VM 的诊断日志。

## <a name="recommendations"></a>建议

你的要求可能不同于此处描述的体系结构。 请使用以下建议作为入手点。 

### <a name="availability-and-scalability-recommendations"></a>可用性和可伸缩性建议

实现可用性和可伸缩性的一个选项是使用[虚拟机规模集][vmss]。 VM 规模集可帮助部署和管理一组相同的 VM。 规模集支持基于性能指标自动缩放。 VM 上的负载增加时，会自动向负载均衡器添加更多 VM。 如果需要快速横向扩展 VM，或者需要进行自动缩放，请考虑规模集。

默认情况下，规模集使用“过度预配”，这意味着规模集最初会预配多于你要求的数量的 VM，然后删除额外的 VM。 这将在预配 VM 时提高整体成功率。 如果不使用[托管磁盘](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-managed-disks)，则建议在启用过度预配的情况下每个存储帐户不超过 20 个 VM，或者在禁用过度预配的情况下每个存储帐户不超过 40 个 VM。

有两种基本方法可用来配置规模集中部署的 VM：

- 在预配 VM 后使用扩展对其进行配置。 使用此方法时，启动新 VM 实例的所需时间可能会长于启动不带扩展的 VM 的所需时间。

- 使用自定义磁盘映像部署[托管磁盘](/azure/storage/storage-managed-disks-overview)。 此选项的部署速度可能更快。 但是，它要求将映像保持最新。

有关其他注意事项，请参阅[规模集的设计注意事项][vmss-design]。

> [!TIP]
> 在使用任何自动缩放解决方案时，请早早提前使用生产级工作负荷测试它。

如果不使用规模集，请考虑至少使用可用性集。 请在可用性集中至少创建两个 VM 以支持 [Azure VM 的可用性 SLA][vm-sla]。 Azure 负载均衡器还要求负载均衡的 VM 属于同一可用性集。

每个 Azure 订阅都有适用的默认限制，包括每个区域的最大 VM 数量。 可以通过提出支持请求提高上限。 有关详细信息，请参阅 [Azure 订阅和服务限制、配额与约束][subscription-limits]。

### <a name="network-recommendations"></a>网络建议

将 VM 置于同一子网内。 不要将 VM 直接向 Internet 公开，而是改为给每个 VM 提供专用 IP 地址。 客户端使用负载均衡器的公共 IP 地址进行连接。

如果需要登录到负载均衡器后面的 VM，请考虑将单个 VM 添加为具有可以登录到的公共 IP 地址的守护主机/jumpbox。 然后从 jumpbox 登录到负载均衡器后面的 VM。 或者，配置负载均衡器中的入站 NAT 规则以实现同一目的。 但是，托管 n 层工作负荷或多个工作负荷时，使用 jumpbox 是更好的解决方案。

### <a name="load-balancer-recommendations"></a>负载均衡器建议

将可用性集中的所有 VM 都添加到负载均衡器的后端地址池。

定义用于将网络流量定向到 VM 的负载均衡器规则。 例如，若要启用 HTTP 流量，请创建将前端配置中的端口 80 映射到后端地址池上的端口 80 的规则。 当客户端将 HTTP 请求发送到端口 80 时，负载均衡器会通过使用包括源 IP 地址的[哈希算法][load balancer hashing]选择后端 IP 地址。 这样，客户端请求就会分布在所有 VM 上。

若要将流量路由到特定 VM，请使用 NAT 规则。 例如，若要为 VM 启用 RDP，请为每个 VM 创建单独的 NAT 规则。 每个规则应将不重复的端口号映射到端口 3389（RDP 的默认端口）。 例如，将端口 50001 用于“VM1”，将端口 50002 用于“VM2”，以此类推。 将 NAT 规则分配给 VM 上的 NIC。

### <a name="storage-account-recommendations"></a>存储帐户建议

为避免达到存储帐户的每秒输入输出/操作数 [(IOPS) 限制][vm-disk-limits]，请为每个 VM 创建单独的 Azure 存储帐户来存放虚拟硬盘 (VHD)。

建议将[托管磁盘](/azure/storage/storage-managed-disks-overview)与[高级存储][premium]配合使用。 托管磁盘不需要存储帐户。 只需指定磁盘的大小和类型，它就会以高度可用的方式部署。

为诊断日志创建一个存储帐户。 此存储帐户可由所有 VM 共享。 这可以是使用标准磁盘的非托管存储帐户。

## <a name="availability-considerations"></a>可用性注意事项

可用性集使得应用程序对计划内和计划外维护事件都更具弹性。

* *计划内维护*在 Microsoft 更新基础平台时发生，有时会导致 VM 重新启动。 Azure 确保可用性集中的 VM 并非都同时重新启动。 至少有一个 VM 在其他 VM 重新启动时保持运行。
* *计划外维护*在有硬件故障时发生。 Azure 确保可用性集中的 VM 跨多个服务器机架预配。 这会帮助减少硬件故障、网络中断、电力中断等事件的影响。

有关详细信息，请参阅[管理虚拟机的可用性][availability set]。 下面的视频还包含一个不错的可用性集概述：[如何配置可用性集以缩放 VM][availability set ch9]。

> [!WARNING]
> 请确保在预配 VM 时配置可用性集。 目前，没有任何方法可以在预配 VM 后将资源管理器 VM 添加到可用性集。

负载均衡器使用[运行状况探测]监视 VM 实例的可用性。 如果探测在超时期限内无法到达实例，负载均衡器会停止向该 VM 发送流量。 但是，负载均衡器将继续探测，并且如果 VM 再次变为可用，负载均衡器会继续向该 VM 发送流量。

以下是有关负载均衡器运行状况探测的一些建议：

* 探测可以测试 HTTP 或 TCP。 如果 VM 运行 HTTP 服务器，请创建 HTTP 探测。 否则请创建 TCP 探测。
* 对于 HTTP 探测，请指定指向 HTTP 终结点的路径。 探测将检查是否有来自此路径的 HTTP 200 响应。 这可以是根路径 ("/")，也可以是一个运行状况监视终结点，该终结点实现某些自定义逻辑来检查应用程序运行状况。 终结点必须允许匿名 HTTP 请求。
* 探测是从[已知][health-probe-ip] IP 地址 168.63.129.16 发送的。 请确保在任何防火墙策略或网络安全组 (NSG) 规则中未阻止与此 IP 之间的流量。
* 使用[运行状况探测日志][health probe log]查看运行状况探测的状态。 请在 Azure 门户中为每个负载均衡器启用日志记录。 日志将写入到 Azure Blob 存储。 这些日志显示后端有多少个 VM 由于失败的探测响应而未收到网络流量。

## <a name="manageability-considerations"></a>可管理性注意事项

如果使用多个 VM ，那么自动执行过程会非常重要，这样过程将非常可靠且可重复。 可以使用 [Azure 自动化][azure-automation]自动执行部署、OS 修补和其他任务。

## <a name="security-considerations"></a>安全注意事项

虚拟网络是 Azure 中的流量隔离边界。 一个 VNet 中的 VM 无法直接与其他 VNet 中的 VM 通信。 同一个 VNet 中的 VM 可以通信，除非你创建[网络安全组][nsg] (NSG) 来限制流量。 有关详细信息，请参阅 [Microsoft 云服务和网络安全性][network-security]。

对于传入 Internet 流量，负载均衡器规则定义哪些流量可以到达后端。 但是，负载均衡器规则不支持 IP 安全列表，因此如果要将某些公共 IP 地址添加到安全列表，请将 NSG 添加到子网。

## <a name="deploy-the-solution"></a>部署解决方案

[GitHub][github-folder] 上提供了此体系结构的部署。 它包括 VNet、NSG 和负载均衡器后面的规模集中的三个 VM，如下所述：

  * 一个虚拟网络，其中包含用于托管 VM 的名为 **web** 的单个子网。
  * 一个 VM 规模集，其中包含运行 Ubuntu 16.04.3 最新版本的 VM。 自动缩放已启用。
  * 一个位于 VM 规模集前面的负载均衡器。
  * 一个 NSG，其中包含用于允许 HTTP 流量发送到 VM 规模集的传入规则。

### <a name="prerequisites"></a>先决条件

在将参考体系结构部署到自己的订阅之前，必须执行以下步骤。

1. 为 [AzureCAT 参考体系结构][ref-arch-repo] GitHub 存储库克隆、下载 zip 文件或创建其分支。

2. 确保在计算机上安装了 Azure CLI 2.0。 若要安装 CLI，请按照[安装 Azure CLI 2.0][azure-cli-2] 中的说明执行操作。

3. 安装 [Azure 构建基块][azbb] npm 包。

4. 从命令提示符、bash 提示符或 PowerShell 提示符下通过使用以下命令之一，登录到 Azure 帐户，然后按照提示进行操作。

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a>使用 azbb 部署解决方案

若要部署示例单一 VM 工作负荷，请执行下列步骤：

1. 导航到已在前面的先决条件步骤中下载的存储库的 `virtual-machines\multi-vm\parameters\linux` 文件夹。

2. 打开 `multi-vm-v2.json` 文件并在引号之间输入用户名和 SSH 密钥，如下所示，然后保存文件。

  ```bash
  "adminUsername": "",
  "sshPublicKey": "",
  ```

3. 运行 `azbb` 以部署 VM，如下所示。

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p multi-vm-v2.json --deploy
  ```

有关部署此示例参考体系结构的详细信息，请访问 [GitHub 存储库][git]。

<!-- Links -->
[github-folder]: http://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[n-tier-linux]: ../virtual-machines-linux/n-tier.md
[n-tier-windows]: n-tier.md
[single vm]: single-vm.md
[premium]: /azure/storage/common/storage-premium-storage
[naming conventions]: /azure/guidance/guidance-naming-conventions
[vm-scaleset]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[availability set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[availability set ch9]: https://channel9.msdn.com/Series/Microsoft-Azure-Fundamentals-Virtual-Machines/08
[azure-automation]: https://azure.microsoft.com/documentation/services/automation/
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-automation]: /azure/automation/automation-intro
[bastion host]: https://en.wikipedia.org/wiki/Bastion_host
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[health probe log]: /azure/load-balancer/load-balancer-monitor-log
[运行状况探测]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[负载均衡器]: /azure/load-balancer/load-balancer-get-started-internet-arm-cli
[load balancer hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[network-security]: /azure/best-practices-network-security
[nsg]: /azure/virtual-network/virtual-networks-nsg
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview 
[Runbook Gallery]: /azure/automation/automation-runbook-gallery#runbooks-in-runbook-gallery
[subscription-limits]: /azure/azure-subscription-service-limits
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_2/
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[vmss-quickstart]: https://azure.microsoft.com/documentation/templates/?term=scale+set
[VM-sizes]: https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/
[0]: ./images/multi-vm-diagram.png "Azure 上的多 VM 解决方案的体系结构，其中包含带有两个 VM 和一个负载均衡器的可用性集"
[azure-cli-2]: /azure/install-azure-cli?view=azure-cli-latest
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Template-Building-Blocks-Version-2-(Linux)
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm