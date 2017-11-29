---
title: "在 Azure 上运行 Linux VM"
description: "如何在 Azure 上运行 Linux VM，并注意可伸缩性、弹性、可管理性和安全性。"
author: telmosampaio
ms.date: 09/06/2017
pnp.series.title: Linux VM workloads
pnp.series.next: multi-vm
pnp.series.prev: ./index
ms.openlocfilehash: f38c53db5df1681a1abc926df9f0b7f929ceb76b
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="run-a-linux-vm-on-azure"></a>在 Azure 上运行 Linux VM

此参考体系结构显示一组已经过验证的、在 Azure 上运行 Linux 虚拟机 (VM) 的做法。 其中包括有关预配 VM 以及网络和存储组件的建议。 此体系结构可用来运行单个实例，并且是更复杂的体系结构（如 N 层应用程序）的基础。 [**部署此解决方案**。](#deploy-the-solution)

![[0]][0]

下载此体系结构的 [Visio 文件][visio-download]。

## <a name="architecture"></a>体系结构

在 Azure 中预配 VM 涉及更多移动部件，而不只是 VM 本身。 需要考虑的因素有计算、网络和存储。

* **资源组。** [resource-manager-overview]*资源组*][是保存相关资源的容器。 通常需基于解决方案中不同资源的生存期以及要管理资源的人员，为这些资源创建资源组。 对于单一 VM 工作负荷，可以为所有资源创建单个资源组。
* **VM**。 Azure 支持运行大量常用的 Linux 分发，包括 CentOS、Debian、Red Hat Enterprise、Ubuntu 和 FreeBSD。 有关详细信息，请参阅 [Azure 和 Linux][azure-linux]。 可以基于已发布的映像列表或上传到 Azure Blob 存储的虚拟硬盘 (VHD) 文件预配 VM。
* **OS 磁盘。** OS 磁盘是存储在 [Azure 存储][azure-storage]中的一个 VHD。 这意味着它一直存在，即使主机出现故障，也是如此。 OS 磁盘是 `/dev/sda1`。
* **临时磁盘。** 使用临时磁盘创建 VM。 此磁盘存储在主机的物理驱动器上。 它不保存在 Azure 存储中，并且在重新启动期间以及发生其他 VM 生命周期事件期间可能会被删除。 只使用此磁盘存储临时数据，如页面文件或交换文件。 临时磁盘是 `/dev/sdb1`，并在 `/mnt/resource` 或 `/mnt` 中装入。
* **数据磁盘。** [数据磁盘][data-disk]是用于保存应用程序数据的持久性 VHD。 数据磁盘像 OS 磁盘一样，存储在 Azure 存储中。
* **虚拟网络 (VNet) 和子网。** Azure 中的每个 VM 都部署在 VNet 中，后者进一步划分为多个子网。
* **公共 IP 地址。** 需要使用公共 IP 地址与 VM 通信 &mdash; 例如，通过 SSH。
* **网络接口 (NIC)**。 NIC 使 VM 能够与虚拟网络进行通信。
* **网络安全组 (NSG)**。 [NSG][nsg] 用于允许/拒绝到子网的网络流量。 可以将 NSG 与单个 NIC 或与子网相关联。 如果将 NSG 与一个子网相关联，则 NSG 规则适用于该子网中的所有 VM。
* **诊断。** 诊断日志记录对于 VM 管理和故障排除至关重要。

## <a name="recommendations"></a>建议

此体系结构显示有关在 Azure 中运行 Linux VM 的基准建议。 但是，不建议针对任务关键型工作负荷使用单个 VM，因为这会产生单一故障点。 为了提高可用性，请在[可用性集][availability-set]中部署多个 VM。 有关详细信息，请参阅 [Running multiple VMs on Azure][multi-vm]（在 Azure 上运行多个 VM）。 

### <a name="vm-recommendations"></a>VM 建议

Azure 可提供多种虚拟机大小，但建议使用 DS 和 GS 系列，因为相关计算机大小支持[高级存储][premium-storage]。 除非运行专用工作负荷（例如高性能计算），否则请选择其中的一种计算机大小。 有关详细信息，请参阅[虚拟机大小][virtual-machine-sizes]。

如果要将现有工作负荷转移到 Azure，开始时请先使用与本地服务器最匹配的 VM 大小。 然后测量与 CPU、内存和每秒磁盘输入/输出操作次数 (IOPS) 有关的实际工作负荷的性能，并根据需要调整大小。 如果 VM 需要多个 NIC，请注意 NIC 的最大数量取决于 [VM 大小][vm-size-tables]。

在预配 VM 和其他资源时，必须指定区域。 通常应选择离内部用户或客户最近的区域。 但是，并非所有 VM 大小都可在所有区域中使用。 有关详细信息，请参阅[服务（按区域）][services-by-region]。 若要列出特定区域中可用的 VM 大小，请运行以下 Azure 命令行接口 (CLI) 命令：

```
az vm list-sizes --location <location>
```

若要了解如何选择发布的 VM 映像，请参阅[利用 Azure CLI 选择 Linux VM 映像][select-vm-image]。

启用监视和诊断，包括基本运行状况指标、诊断基础结构日志和[启动诊断][boot-diagnostics]。 如果 VM 陷入不可启动状态，启动诊断有助于诊断启动故障。 有关详细信息，请参阅[启用监视和诊断][enable-monitoring]。  

### <a name="disk-and-storage-recommendations"></a>磁盘和存储建议

为获得最佳磁盘 I/O 性能，建议使用[高级存储][premium-storage]，它在固态硬盘 (SSD) 上存储数据。 成本取决于预配磁盘的大小。 IOPS 和吞吐量（即数据传输速率）也取决于磁盘大小，因此在预配磁盘时，请全面考虑三个因素（容量、IOPS 和吞吐量）。 

我们还建议使用[托管磁盘](/azure/storage/storage-managed-disks-overview)。 托管磁盘不需要存储帐户。 只需指定磁盘的大小和类型，它就会以高度可用的方式部署。

如果不使用托管磁盘，请为每个 VM 创建单独的 Azure 存储帐户来存放虚拟硬盘 (VHD)，以避免达到存储帐户的 IOPS 限制。

添加一个或多个数据磁盘。 刚创建的 VHD 尚未格式化， 请登录 VM 格式化该磁盘。 在 Linux shell 中，数据磁盘显示为 `/dev/sdc`、`/dev/sdd` 等。 可以运行 `lsblk` 以列出块设备，包括磁盘。 要使用数据磁盘，请创建一个分区和文件系统，并装载磁盘。 例如：

```bat
# Create a partition.
sudo fdisk /dev/sdc     # Enter 'n' to partition, 'w' to write the change.

# Create a file system.
sudo mkfs -t ext3 /dev/sdc1

# Mount the drive.
sudo mkdir /data1
sudo mount /dev/sdc1 /data1
```

如果不使用[托管磁盘](/azure/storage/storage-managed-disks-overview)并且有大量数据磁盘，请注意存储帐户的总 I/O 限制。 有关详细信息，请参阅[虚拟机磁盘限制][vm-disk-limits]。

在添加数据磁盘时，将为磁盘分配逻辑单元号 (LUN) ID。 或者，可以指定 LUN ID &mdash; 例如，若要更换磁盘并保留相同的 LUN ID，或者应用程序要查找特定 LUN ID。 但请记住，每个磁盘的 LUN ID 必须唯一。

可能会需要更改 I/O 计划程序，以便针对 SSD 的性能进行优化，因为使用高级存储帐户的 VM 的磁盘是 SSD。 常见的建议是对 SSD 使用 NOOP 计划程序，但应使用 [iostat] 等工具来监视工作负荷的磁盘 I/O 性能。

为获得最佳性能，请创建单独的存储帐户来存储诊断日志。 标准的本地冗余存储 (LRS) 帐户足以存储诊断日志。

### <a name="network-recommendations"></a>网络建议

公共 IP 地址可以是动态的或静态的。 默认是动态的。

* 如果需要不会更改的固定 IP 地址（例如，如果需要在 DNS 中创建 A 记录，或者需要将 IP 地址添加到安全列表），请保留[静态 IP 地址][static-ip]。
* 还可以为 IP 地址创建完全限定域名 (FQDN)。 然后，可在 DNS 中注册指向 FQDN 的 [CNAME 记录][cname-record]。 有关详细信息，请参阅[在 Azure 门户中创建完全限定的域名][fqdn]。

所有 NSG 都包含一组[默认规则][nsg-default-rules]，其中包括阻止所有入站 Internet 流量的规则。 无法删除默认规则，但其他规则可以覆盖它们。 要启用 Internet 流量，请创建允许特定端口的入站流量的规则 &mdash; 例如，将端口 80 用于 HTTP。

若要启用 SSH，请向 NSG 添加允许 TCP 端口 22 的入站流量的规则。

## <a name="scalability-considerations"></a>可伸缩性注意事项

可以通过[更改 VM 大小][vm-resize]来扩展或缩小 VM。 要以水平方式横向扩展，请将两个或更多 VM 放在负载均衡器后面。 有关详细信息，请参阅[在 Azure 上运行多个 VM 以实现可伸缩性和可用性][multi-vm]。

## <a name="availability-considerations"></a>可用性注意事项

为了提高可用性，请在可用性集中部署多个 VM。 这样还可提供更高的[服务级别协议][vm-sla] (SLA)。

VM 可能会受到[计划内维护][planned-maintenance]或[计划外维护][manage-vm-availability]的影响。 可以使用 [VM 重新启动日志][reboot-logs]来确定 VM 重新启动是否是由计划内维护导致的。

VHD 存储在 [Azure 存储][azure-storage]中，Azure 存储会复制，实现持久性和可用性。

若要防止在正常操作期间意外数据丢失（例如，由于用户错误），则还应使用 [Blob 快照][blob-snapshot]或其他工具实现时间点备份。

## <a name="manageability-considerations"></a>可管理性注意事项

**资源组。** 将共享相同生命周期的紧密耦合资源放入同一[资源组][resource-manager-overview]中。 资源组可让你以组的形式部署和监视资源，并按资源组汇总计费成本。 还可以删除作为集的资源，这对于测试部署非常有用。 为资源指定有意义的名称。 这样，可更轻松地找到特定资源并了解其角色。 请参阅 [Azure 资源的建议命名约定][naming conventions]。

**SSH**。 在创建 Linux VM 之前，生成 2048 位 RSA 公共/专用密钥对。 创建 VM 时，使用公钥文件。 有关详细信息，请参阅[如何在 Azure 中将 SSH 与 Linux 和 Mac 配合使用][ssh-linux]。

**停止 VM。** Azure 对“已停止”和“已解除分配”状态做了区分。 当 VM 状态为已停止时（而不是当 VM 已解除分配时）将向你收费。 

在 Azure 门户中，“停止”按钮可解除分配 VM。 如果在已登录时通过 OS 关闭，VM 会停止，但不会解除分配，因此仍会产生费用。 

**删除 VM。** 如果删除 VM，则不会删除 VHD。 这意味着可以安全地删除 VM，而不会丢失数据。 但是，仍将向你收取存储费用。 若要删除 VHD，请从 [Blob 存储][blob-storage]中删除相应的文件。

若要防止意外删除，请使用[资源锁][resource-lock]锁定整个资源组或锁定单个资源（如 VM）。

## <a name="security-considerations"></a>安全注意事项

使用 [Azure 安全中心][security-center]可在一个中心视图中获得 Azure 资源的安全状态。 安全中心监视潜在的安全问题，并全面描述了部署的安全运行状况。 安全中心针对每个 Azure 订阅进行配置。 启用安全数据收集，如 [Azure 安全中心快速入门指南][security-center-get-started]中所述。 启用数据收集后，安全中心会自动扫描该订阅下创建的所有 VM。

**修补程序管理。** 如果启用，安全中心会检查是否缺少安全更新和关键更新。 

**反恶意软件。** 如果启用，安全中心会检查是否已安装反恶意软件。 还可以使用安全中心从 Azure 门户中安装反恶意软件。

**操作。** 使用[基于角色的访问控制][rbac] (RBAC) 来控制对部署的 Azure 资源的访问。 RBAC 允许将授权角色分配给开发运营团队的成员。 例如，“读者”角色可以查看 Azure 资源，但不能创建、管理或删除这些资源。 某些角色特定于特定的 Azure 资源类型。 例如，“虚拟机参与者”角色可以执行重启或解除分配 VM、重置管理员密码、创建新的 VM 等操作。 可能对此体系结构有用的其他[内置 RBAC 角色][rbac-roles]包括[开发测试实验室用户][rbac-devtest]和[网络参与者][rbac-network]。 可将用户分配给多个角色，并且可以创建自定义角色以实现更细化的权限。

> [!NOTE]
> RBAC 不限制已登录到 VM 的用户可以执行的操作。 这些权限由来宾 OS 上的帐户类型决定。   

使用[审核日志][audit-logs]可查看预配操作和其他 VM 事件。

**数据加密。** 如果需要加密 OS 磁盘和数据磁盘，请考虑使用 [Azure 磁盘加密][disk-encryption]。 

## <a name="deploy-the-solution"></a>部署解决方案

[GitHub][github-folder] 上提供了此体系结构的部署。 它将部署以下部分：

  * 一个虚拟网络，其中包含用于托管 VM 的名为 **web** 的单个子网。
  * 一个 NSG，其中包含两个用于允许 SSH 和 HTTP 流量发送到 VM 的传入规则。
  * 一个运行 Ubuntu 16.04.3 LTS 最新版本的 VM。
  * 一个示例自定义脚本扩展，它将 Apache HTTP Server 部署到 Ubuntu VM 并格式化两个数据磁盘。

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

1. 导航到已在前面的先决条件步骤中下载的存储库的 `virtual-machines\single-vm\parameters\linux` 文件夹。

2. 打开 `single-vm-v2.json` 文件并在引号之间输入用户名和 SSH 密钥，如下所示，然后保存文件。

  ```bash
  "adminUsername": "",
  "adminsshPublicKey": "",
  ```

3. 运行 `azbb` 以部署示例 VM，如下所示。

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
  ```

有关部署此示例参考体系结构的详细信息，请访问 [GitHub 存储库][git]。

## <a name="next-steps"></a>后续步骤

- 了解有关 [Azure 构建基块][azbbv2]的信息。
- 在 Azure 中部署[多个 VM][multi-vm]。

<!-- links -->
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[multi-vm]: ../virtual-machines-linux/multi-vm.md
[naming conventions]: ../../best-practices/naming-conventions.md
[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[availability-set]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azbbv2]: https://github.com/mspnp/template-building-blocks
[azure-cli-2]: /cli/azure/install-azure-cli?view=azure-cli-latest
[azure-linux]: /azure/virtual-machines/virtual-machines-linux-azure-overview
[azure-storage]: /azure/storage/storage-introduction
[blob-snapshot]: /azure/storage/storage-blob-snapshots
[blob-storage]: /azure/storage/storage-introduction
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: /azure/virtual-machines/virtual-machines-linux-about-disks-vhds
[disk-encryption]: /azure/security/azure-security-disk-encryption
[enable-monitoring]: /azure/monitoring-and-diagnostics/insights-how-to-use-diagnostics
[fqdn]: /azure/virtual-machines/virtual-machines-linux-portal-create-fqdn
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[iostat]: https://en.wikipedia.org/wiki/Iostat
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-default-rules]: /azure/virtual-network/virtual-networks-nsg#default-rules
[OSPatching]: https://github.com/Azure/azure-linux-extensions/tree/master/OSPatching
[planned-maintenance]: /azure/virtual-machines/virtual-machines-linux-planned-maintenance
[premium-storage]: /azure/storage/storage-premium-storage
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rbac-roles]: /azure/active-directory/role-based-access-built-in-roles
[rbac-devtest]: /azure/active-directory/role-based-access-built-in-roles#devtest-labs-user
[rbac-network]: /azure/active-directory/role-based-access-built-in-roles#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resource-lock]: /azure/resource-group-lock-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[security-center]: /azure/security-center/
[security-center-get-started]: /azure/security-center/security-center-get-started
[select-vm-image]: /azure/virtual-machines/virtual-machines-linux-cli-ps-findimage
[services-by-region]: https://azure.microsoft.com/regions/#services
[ssh-linux]: /azure/virtual-machines/virtual-machines-linux-mac-create-ssh-keys
[static-ip]: /azure/virtual-network/virtual-networks-reserved-public-ip
[virtual-machine-sizes]: /azure/virtual-machines/virtual-machines-linux-sizes
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-resize]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[vm-size-tables]: /azure/virtual-machines/virtual-machines-linux-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[readme]: https://github.com/mspnp/reference-architectures/blob/master/virtual-machines/single-vm/README.md
[0]: ./images/single-vm-diagram.png "Azure 中的单一 Linux VM 体系结构"

