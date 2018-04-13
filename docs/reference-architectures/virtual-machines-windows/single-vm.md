---
title: 在 Azure 上运行 Windows VM
description: 如何在 Azure 上运行 Windows VM，并注意可伸缩性、弹性、可管理性和安全性。
author: telmosampaio
ms.date: 04/03/2018
pnp.series.title: Windows VM workloads
pnp.series.next: multi-vm
pnp.series.prev: ./index
ms.openlocfilehash: 9bc0b4af56b9194fd1bec8a189c86963ad2b0c98
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/06/2018
---
# <a name="run-a-windows-vm-on-azure"></a>在 Azure 上运行 Windows VM

此参考体系结构显示一组已经过验证的、在 Azure 上运行 Windows 虚拟机 (VM) 的做法。 其中包括有关预配 VM 以及网络和存储组件的建议。 此体系结构可用来运行单个 VM 实例，并且是更复杂体系结构（如 N 层应用程序）的基础。 [部署此解决方案。](#deploy-the-solution)

![[0]][0]

下载包含此体系结构关系图的 [Visio 文件][visio-download]。

## <a name="architecture"></a>体系结构

除 VM 本身以外，预配 Azure VM 还需要其他一些组件，包括网络和存储资源。

* **资源组。** [资源组][resource-manager-overview]是保存相关 Azure 资源的逻辑容器。 一般情况下，可根据资源的生存期及其管理者将资源分组。 

* **VM**。 可以通过发布的映像列表或上传到 Azure Blob 存储的自定义托管映像或虚拟硬盘 (VHD) 文件来预配 VM。

* **托管磁盘**。 [Azure 托管磁盘][managed-disks]可代你处理存储，从而简化了磁盘管理。 OS 磁盘是存储在 [Azure 存储][azure-storage]中的 VHD，因此即使主机关闭，OS 磁盘也仍然存在。 我们还建议创建一个或多个[数据磁盘][data-disk]（用于保存应用程序数据的持久性 VHD）。

* **临时磁盘。** 使用临时磁盘（Windows 上的 `D:` 驱动器）创建 VM。 此磁盘存储在主机的物理驱动器上。 它并非保存在 Azure 存储中，并且在重启期间以及发生其他 VM 生命周期事件期间可能会被删除。 只使用此磁盘存储临时数据，如页面文件或交换文件。

* **虚拟网络 (VNet)**。 每个 Azure VM 都会部署到可细分为多个子网的 VNet 中。

* **网络接口 (NIC)**。 NIC 使 VM 能够与虚拟网络进行通信。  

* **公共 IP 地址。** 须使用公共 IP 地址才能与 VM 通信 &mdash; 例如，通过远程桌面 (RDP)。  

* **Azure DNS**。 [Azure DNS][azure-dns] 是 DNS 域的托管服务，它使用 Microsoft Azure 基础结构提供名称解析。 通过在 Azure 中托管域，可以使用与其他 Azure 服务相同的凭据、API、工具和计费来管理 DNS 记录。

* **网络安全组 (NSG)**。 [网络安全组][nsg]用于允许或拒绝向 VM 传送网络流量。 NSG 可与子网或单个 VM 实例相关联。 

* **诊断。** 诊断日志记录对于 VM 管理和故障排除至关重要。

## <a name="recommendations"></a>建议

此体系结构显示有关在 Azure 中运行 Windows VM 的基准建议。 但是，不建议针对任务关键型工作负荷使用单个 VM，因为这会产生单一故障点。 为了提高可用性，请部署两个或更多个负载均衡的 VM。 有关详细信息，请参阅 [Running multiple VMs on Azure][multi-vm]（在 Azure 上运行多个 VM）。

### <a name="vm-recommendations"></a>VM 建议

Azure 提供多种不同的虚拟机大小。 有关详细信息，请参阅 [Azure 中虚拟机的大小][virtual-machine-sizes]。 如果要将现有工作负荷转移到 Azure，开始时请先使用与本地服务器最匹配的 VM 大小。 然后从 CPU、内存和每秒磁盘输入/输出操作次数 (IOPS) 等方面测量实际工作负荷的性能，并根据需要调整大小。 如果 VM 需要多个 NIC，请注意每种 [VM 大小][vm-size-tables]都定义了最大 NIC 数量。

通常选择离内部用户或客户最近的 Azure 区域。 但是，并非所有 VM 大小都可在所有区域中使用。 有关详细信息，请参阅[每个区域的服务][services-by-region]。 要获取特定区域中可用 VM 大小的列表，请从 Azure 命令行接口 (CLI) 运行以下命令：

```
az vm list-sizes --location <location>
```

要了解如何选择发布的 VM 映像，请参阅[查找 Windows VM 映像][select-vm-image]。

启用监视和诊断，包括基本运行状况指标、诊断基础结构日志和[启动诊断][boot-diagnostics]。 如果 VM 陷入不可启动状态，启动诊断有助于诊断启动故障。 有关详细信息，请参阅[启用监视和诊断][enable-monitoring]。  

### <a name="disk-and-storage-recommendations"></a>磁盘和存储建议

为获得最佳磁盘 I/O 性能，建议使用[高级存储][premium-storage]，它在固态硬盘 (SSD) 上存储数据。 成本取决于预配磁盘的容量。 IOPS 和吞吐量（即数据传输速率）也取决于磁盘大小，因此在预配磁盘时，请全面考虑三个因素（容量、IOPS 和吞吐量）。 

我们还建议使用[托管磁盘][managed-disks]。 托管磁盘不需要存储帐户。 只需指定磁盘的大小和类型，就可以将它部署为高度可用的资源。

添加一个或多个数据磁盘。 刚创建的 VHD 尚未格式化， 登录到 VM 对磁盘进行格式化。 如果可能，请将应用程序安装在数据磁盘上，而不是 OS 磁盘上。 某些旧版应用程序可能需要将组件安装在 C: 驱动器上；在这种情况下，可使用 PowerShell [重设 OS 磁盘的大小][resize-os-disk]。

创建用于保存诊断日志的存储帐户。 标准的本地冗余存储 (LRS) 帐户足以存储诊断日志。

> [!NOTE]
> 如果不使用托管磁盘，请为每个 VM 创建单独的 Azure 存储帐户来存放虚拟硬盘 (VHD)，以避免达到存储帐户的 [(IOPS) 限制][vm-disk-limits]。 请注意存储帐户的总 I/O 限制。 有关详细信息，请参阅[虚拟机磁盘限制][vm-disk-limits]。


### <a name="network-recommendations"></a>网络建议

公共 IP 地址可以是动态的或静态的。 默认是动态的。

* 如果需要不会更改的固定 IP 地址 &mdash; 例如，如果需要创建 DNS 'A' 记录或将 IP 地址添加到安全列表，请保留[静态 IP 地址][static-ip]。
* 还可以为 IP 地址创建完全限定域名 (FQDN)。 然后，可在 DNS 中注册指向 FQDN 的 [CNAME 记录][cname-record]。 有关详细信息，请参阅[在 Azure 门户中创建完全限定的域名][fqdn]。

所有 NSG 都包含一组[默认规则][nsg-default-rules]，其中包括阻止所有入站 Internet 流量的规则。 无法删除默认规则，但其他规则可以覆盖它们。 要启用 Internet 流量，请创建允许特定端口的入站流量的规则 &mdash; 例如，将端口 80 用于 HTTP。

若要启用 RDP，请添加允许 TCP 端口 3389 的入站流量的 NSG 规则。

## <a name="scalability-considerations"></a>可伸缩性注意事项

可以通过[更改 VM 大小][vm-resize]来扩展或缩小 VM。 要以水平方式横向扩展，请将两个或更多 VM 放在负载均衡器后面。 有关详细信息，请参阅[运行负载均衡的 VM 以实现可伸缩性和可用性][multi-vm]。

## <a name="availability-considerations"></a>可用性注意事项

为了提高可用性，请在可用性集中部署多个 VM。 这样还可提供更高的[服务级别协议 (SLA)][vm-sla]。

VM 可能会受到[计划内维护][planned-maintenance]或[计划外维护][manage-vm-availability]的影响。 可以使用 [VM 重新启动日志][reboot-logs]来确定 VM 重新启动是否是由计划内维护导致的。

若要防止在正常操作期间意外数据丢失（例如，由于用户错误），则还应使用 [Blob 快照][blob-snapshot]或其他工具实现时间点备份。

## <a name="manageability-considerations"></a>可管理性注意事项

**资源组。** 将共享相同生命周期、密切相关的资源放入同一[资源组][resource-manager-overview]。 资源组可让你以组的形式部署和监视资源，并按资源组跟踪计费成本。 还可以删除作为集的资源，这对于测试部署非常有用。 指定有意义的资源名称，以便简化特定资源的查找并了解其角色。 有关详细信息，请参阅 [Azure 资源的建议命名约定][naming-conventions]。

**停止 VM。** Azure 对“已停止”和“已解除分配”状态做了区分。 当 VM 状态为已停止时（而不是当 VM 已解除分配时）将向你收费。 在 Azure 门户中，“停止”按钮可解除分配 VM。 如果在已登录时通过 OS 关闭，VM 会停止，但不会解除分配，因此仍会产生费用。

**删除 VM。** 如果删除 VM，则不会删除 VHD。 这意味着可以安全地删除 VM，而不会丢失数据。 但是，仍将向你收取存储费用。 若要删除 VHD，请从 [Blob 存储][blob-storage]中删除相应的文件。 要防止意外删除，请使用[资源锁][resource-lock]锁定整个资源组或锁定单个资源（如 VM）。

## <a name="security-considerations"></a>安全注意事项

使用 [Azure 安全中心][security-center]可在一个中心视图中获得 Azure 资源的安全状态。 安全中心监视潜在的安全问题，并全面描述了部署的安全运行状况。 安全中心针对每个 Azure 订阅进行配置。 启用安全数据收集，如 [Azure 安全中心快速入门指南][security-center-get-started]中所述。 启用数据收集后，安全中心会自动扫描该订阅下创建的所有 VM。

**修补程序管理。** 如果启用，安全中心会检查是否缺少任何安全更新和关键更新。 使用 VM 上的[组策略设置][group-policy]可实现自动系统更新。

**反恶意软件。** 如果启用，安全中心会检查是否已安装反恶意软件。 还可以使用安全中心从 Azure 门户中安装反恶意软件。

**操作。** 使用[基于角色的访问控制 (RBAC)][rbac] 来控制对部署的 Azure 资源的访问。 RBAC 允许将授权角色分配给开发运营团队的成员。 例如，“读者”角色可以查看 Azure 资源，但不能创建、管理或删除这些资源。 某些角色特定于特定的 Azure 资源类型。 例如，“虚拟机参与者”角色可以执行重启或解除分配 VM、重置管理员密码、创建新的 VM 等操作。 可能对此体系结构有用的其他[内置 RBAC 角色][rbac-roles]包括[开发测试实验室用户][rbac-devtest]和[网络参与者][rbac-network]。 可将用户分配给多个角色，并且可以创建自定义角色以实现更细化的权限。

> [!NOTE]
> RBAC 不限制已登录到 VM 的用户可以执行的操作。 这些权限由来宾 OS 上的帐户类型决定。   

使用[审核日志][audit-logs]可查看预配操作和其他 VM 事件。

**数据加密。** 如果需要加密 OS 磁盘和数据磁盘，请考虑使用 [Azure 磁盘加密][disk-encryption]。 

## <a name="deploy-the-solution"></a>部署解决方案

[GitHub][github-folder] 上提供了此体系结构的部署。 它将部署以下部分：

  * 一个虚拟网络，其中包含用于托管 VM 的名为 **web** 的单个子网。
  * 一个 NSG，其中包含两个用于允许 RDP 和 HTTP 流量发送到 VM 的传入规则。
  * 一个运行 Windows Server 2016 Datacenter Edition 最新版本的 VM。
  * 一个用于格式化两个数据磁盘的示例自定义脚本扩展，另一个用于部署 Internet Information Services (IIS) 的 PowerShell DSC 脚本。

### <a name="prerequisites"></a>先决条件

1. 克隆、下载[参考体系结构][ref-arch-repo] GitHub 存储库的 zip 文件或创建其分支。

2. 确保在计算机上安装了 Azure CLI 2.0。 有关 CLI 安装说明，请参阅[安装 Azure CLI 2.0][azure-cli-2]。

3. 安装 [Azure 构建基块][azbb] npm 包。

4. 在命令提示符、bash 提示符或 PowerShell 提示符下，输入以下命令登录到 Azure 帐户。

   ```bash
   az login
   ```

### <a name="deploy-the-solution-using-azbb"></a>使用 azbb 部署解决方案

若要部署此参考体系结构，请执行以下步骤：

1. 导航到已在前面的先决条件步骤中下载的存储库的 `virtual-machines\single-vm\parameters\windows` 文件夹。

2. 打开 `single-vm-v2.json` 文件，输入括在引号中的用户名和密码，然后保存文件。

   ```bash
   "adminUsername": "",
   "adminPassword": "",
   ```

3. 运行 `azbb` 以部署示例 VM，如下所示。

   ```bash
   azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
   ```

若要验证部署，请运行以下 Azure CLI 命令，找到 VM 的公共 IP 地址：

```bash
az vm show -n ra-single-windows-vm1 -g <resource-group-name> -d -o table
```

如果在 Web 浏览器中导航到此地址，应会看到默认的 IIS 主页。

有关自定义此部署的信息，请访问我们的 [GitHub 存储库][git]。

## <a name="next-steps"></a>后续步骤

- 了解有关 [Azure 构建基块][azbbv2]的信息。
- 在 Azure 中部署[多个 VM][multi-vm]。

<!-- links -->
[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azbbv2]: https://github.com/mspnp/template-building-blocks
[azure-cli-2]: /cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[azure-storage]: /azure/storage/storage-introduction
[blob-snapshot]: /azure/storage/storage-blob-snapshots
[blob-storage]: /azure/storage/storage-introduction
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: /azure/virtual-machines/virtual-machines-windows-about-disks-vhds
[disk-encryption]: /azure/security/azure-security-disk-encryption
[enable-monitoring]: /azure/monitoring-and-diagnostics/insights-how-to-use-diagnostics
[fqdn]: /azure/virtual-machines/virtual-machines-windows-portal-create-fqdn
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[group-policy]: https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn595129(v=ws.11)
[log-collector]: https://azure.microsoft.com/blog/simplifying-virtual-machine-troubleshooting-using-azure-log-collector/
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[managed-disks]: /azure/storage/storage-managed-disks-overview
[multi-vm]: multi-vm.md
[naming-conventions]: ../../best-practices/naming-conventions.md
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-default-rules]: /azure/virtual-network/virtual-networks-nsg#default-rules
[planned-maintenance]: /azure/virtual-machines/virtual-machines-windows-planned-maintenance
[premium-storage]: /azure/virtual-machines/windows/premium-storage
[premium-storage-supported]: /azure/virtual-machines/windows/premium-storage#supported-vms
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rbac-roles]: /azure/active-directory/role-based-access-built-in-roles
[rbac-devtest]: /azure/active-directory/role-based-access-built-in-roles#devtest-labs-user
[rbac-network]: /azure/active-directory/role-based-access-built-in-roles#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resize-os-disk]: /azure/virtual-machines/virtual-machines-windows-expand-os-disk
[resource-lock]: /azure/resource-group-lock-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[security-center]: /azure/security-center/security-center-intro
[security-center-get-started]: /azure/security-center/security-center-get-started
[select-vm-image]: /azure/virtual-machines/virtual-machines-windows-cli-ps-findimage
[services-by-region]: https://azure.microsoft.com/regions/#services
[static-ip]: /azure/virtual-network/virtual-networks-reserved-public-ip
[virtual-machine-sizes]: /azure/virtual-machines/virtual-machines-windows-sizes
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-resize]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[vm-size-tables]: /azure/virtual-machines/virtual-machines-windows-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[0]: ./images/single-vm-diagram.png "Azure 中的单一 Windows VM 体系结构"
