---
title: "在 Azure 上运行用于 N 层应用程序的 Linux VM"
description: "如何在 Microsoft Azure 中运行用于 N 层体系结构的 Linux VM。"
author: MikeWasson
ms.date: 11/22/2017
pnp.series.title: Linux VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: e875a58aa83339560fd1de5b03a960f071883927
ms.sourcegitcommit: c9e6d8edb069b8c513de748ce8114c879bad5f49
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/08/2018
---
# <a name="run-linux-vms-for-an-n-tier-application"></a>运行用于 N 层应用程序的 Linux VM

此参考体系结构显示了一组经过实践检验的、运行用于 N 层应用程序的 Linux 虚拟机 (VM) 的做法。 [**部署此解决方案**。](#deploy-the-solution)  

![[0]][0]

*下载此体系结构的 [Visio 文件][visio-download]。*

## <a name="architecture"></a>体系结构

有许多方法可用来实现 N 层体系结构。 关系图中显示了一个典型的 3 层 Web 应用程序。 此体系结构基于[运行负载均衡的 VM 以提高可伸缩性和可用性][multi-vm]中所述的内容。 Web 层和业务层都使用负载均衡的 VM。

* **可用性集。** 为每个层创建一个[可用性集][azure-availability-sets]，并且在每个层中至少预配两个 VM。  这样，VM 便可以满足 VM 的更高[服务级别协议 (SLA)][vm-sla]。 可在可用性集中部署单个 VM，但单个 VM 不具备 SLA 保证的资格，除非单个 VM 针对所有 OS 和数据磁盘使用 Azure 高级存储。  
* **子网。** 为每个层创建一个单独的子网。 使用 [CIDR] 表示法指定地址范围和子网掩码。 
* **负载均衡器。** 使用[面向 Internet 的负载均衡器][load-balancer-external]将传入的 Internet 流量分布到 Web 层，使用[内部负载均衡器][load-balancer-internal]将来自 Web 层的网络流量分布到业务层。
* **Azure DNS**。 [Azure DNS][azure-dns] 是 DNS 域的托管服务，它使用 Microsoft Azure 基础结构提供名称解析。 通过在 Azure 中托管域，可以使用与其他 Azure 服务相同的凭据、API、工具和计费来管理 DNS 记录。
* **Jumpbox。** 也称为[守护主机]。 网络上的一个安全 VM，管理员使用它来连接到其他 VM。 Jumpbox 中的某个 NSG 只允许来自安全列表中的公共 IP 地址的远程流量。 NSG 应允许安全外壳 (SSH) 流量。
* **监视。** 可以使用 [Nagios]、[Zabbix] 或 [Icinga] 等监视软件深入了解响应时间、VM 运行时间和系统的整体运行状况。 在置于单独的管理子网中的 VM 上安装监视软件。
* **NSG。** 使用[网络安全组][nsg] (NSG) 来限制 VNet 中的网络流量。 例如，在此处显示的 3 层体系结构中，数据库层不接受来自 Web 前端的流量，仅接受来自业务层和管理子网的流量。
* **Apache Cassandra 数据库**。 通过启用复制和故障转移，在数据层提供高可用性。

## <a name="recommendations"></a>建议

你的要求可能不同于此处描述的体系结构。 请使用以下建议作为入手点。 

### <a name="vnet--subnets"></a>VNet/子网

在创建 VNet 时，确定每个子网中的资源需要多少 IP 地址。 使用 [CIDR] 表示法为所需的 IP 地址指定子网掩码和足够大的 VNet 地址范围。 使用标准[专用 IP 地址块][private-ip-space]内的一个地址空间，这些地址块为 10.0.0.0/8、172.16.0.0/12 和 192.168.0.0/16。

如果以后需要在 VNet 与本地网络之间设置一个网关，请选择一个不与本地网络重叠的地址范围。 在创建 VNet 后，将无法更改地址范围。

在设计子网时一定要牢记功能和安全要求。 同一层或同一角色中的所有 VM 应当置于同一子网，这可能是一个安全边界。 有关设计 VNet 和子网的详细信息，请参阅[规划和设计 Azure 虚拟网络][plan-network]。

针对每个子网，采用 CIDR 表示法指定子网的地址空间。 例如，“10.0.0.0/24”将创建包含 256 个 IP 地址的范围。 VM 可以使用其中的 251 个；5 个会被保留。 请确保地址范围在子网之间不重叠。 请参阅[虚拟网络常见问题解答][vnet faq]。

### <a name="network-security-groups"></a>网络安全组

使用 NSG 规则限制各个层之间的流量。 例如，在上面显示的 3 层体系结构中，Web 层不直接与数据库层进行通信。 为强制实现此目的，数据库层应当阻止来自 Web 层子网的传入流量。  

1. 创建一个 NSG 并将其关联到数据库层子网。
2. 添加一个拒绝来自 VNet 的所有入站流量的规则。 （在规则中使用 `VIRTUAL_NETWORK` 标记。） 
3. 添加一个具有更高优先级的规则，用以允许来自业务层子网的入站流量。 此规则将替代前面的规则，并允许业务层与数据库层进行通信。
4. 添加一个允许来自数据库层子网自身的入站流量的规则。 此规则允许在数据库层中的各个 VM 之间进行通信，这是进行数据库复制和故障转移所必需的。
5. 添加一个允许来自 jumpbox 子网的 SSH 流量的规则。 此规则允许管理员从 jumpbox 连接到数据库层。
   
   > [!NOTE]
   > NSG 包含允许来自 VNet 的任何入站流量的[默认规则][nsg-rules]。 这些规则无法删除，但可以通过创建优先级更高的规则来替代它们。
   > 
   > 

### <a name="load-balancers"></a>负载均衡器

外部负载均衡器将 Internet 流量分布到 Web 层。 为此负载均衡器创建一个公共 IP 地址。 请参阅[创建面向 Internet 的负载均衡器][lb-external-create]。

内部负载均衡器将网络流量从 Web 层分布到业务层。 若要为此负载均衡器提供专用 IP 地址，请创建一个前端 IP 配置，并将其与业务层的子网相关联。 请参阅[开始创建内部负载均衡器][lb-internal-create]。

### <a name="cassandra"></a>Cassandra

我们建议将 [DataStax Enterprise][datastax] 用于生产用途，但这些建议适用于任何 Cassandra 版本。 有关在 Azure 中运行 DataStax 的详细信息，请参阅[适用于 Azure 的 DataStax Enterprise 部署指南][cassandra-in-azure]。 

将 Cassandra 群集的 VM 放置在一个可用性集中，以确保 Cassandra 副本分布在多个容错域和升级域中。 有关容错域和升级域的详细信息，请参阅[管理虚拟机的可用性][azure-availability-sets]。 

为每个可用性集配置三个容错域（最大数目），并且为每个可用性集配置 18 个升级域。 这提供了仍可均匀分布在容错域中的升级域的最大数量。   

以机架感知模式配置节点。 将故障域映射到 `cassandra-rackdc.properties` 文件中的机架。

群集前面不需要负载均衡器。 客户端会直接连接到群集中的节点。

### <a name="jumpbox"></a>Jumpbox

Jumpbox 的性能要求非常低，因此请为 jumpbox 选择一个较小的 VM 大小，例如标准 A1。 

为 jumpbox 创建一个[公共 IP 地址]。 将 jumpbox 放置在与其他 VM 相同的 VNet 中，但将其置于一个单独的管理子网中。

不要允许从公共 Internet 对运行应用程序工作负荷的 VM 进行 SSH 访问。 相反，对这些 VM 的所有 SSH 访问都必须通过 jumpbox 进行。 管理员登录到 jumpbox，然后从 jumpbox 登录到其他 VM。 Jumpbox 允许来自 Internet 的 SSH 流量，但仅允许来自已知的安全 IP 地址的流量。

若要保护 jumpbox，请创建一个 NSG 并将其应用于 jumpbox 子网。 添加一个 NSG 规则，使其仅允许来自一组安全的公共 IP 地址的 SSH 连接。 可以将 NSG 附加到子网或 jumpbox NIC。 在本例中，我们建议将其附加到 NIC，以便仅允许将 SSH 流量发送到 jumpbox，即使将其他 VM 添加到同一子网也是如此。

为其他子网配置 NSG 以允许来自管理子网的 SSH 流量。

## <a name="availability-considerations"></a>可用性注意事项

将每个层或 VM 角色放入单独的可用性集中。 

在数据库层，具有多个 VM 不会自动转换为高度可用的数据库。 对于关系数据库，通常需要使用复制和故障转移来实现高可用性。  

如果需要的可用性高于 [VM 的 Azure SLA][vm-sla] 提供的可用性，请跨两个区域复制应用程序，并使用 Azure 流量管理器进行故障转移。 有关详细信息，请参阅[在多个区域中运行 Linux VM 以实现高可用性][multi-dc]。  

## <a name="security-considerations"></a>安全注意事项

请考虑添加一个网络虚拟设备 (NVA) 以在公共 Internet 与 Azure 虚拟网络之间创建一个外围网络。 NVA 是虚拟设备的一个通用术语，该虚拟设备可以执行与网络相关的任务，例如防火墙、包检查、审核和自定义路由。 有关详细信息，请参阅[在 Azure 与 Internet 之间实现外围网络][dmz]。

## <a name="scalability-considerations"></a>可伸缩性注意事项

负载均衡器将网络流量分布到 Web 层和业务层。 可以通过添加新的 VM 实例进行水平缩放。 请注意，可以根据负载分别对 Web 层和业务层进行缩放。 为减少由于需要维护客户端相关性而可能导致的复杂情况，Web 层中的 VM 应当是无状态的。 托管着业务逻辑的 VM 也应当是无状态的。

## <a name="manageability-considerations"></a>可管理性注意事项

可以通过使用 [Azure 自动化][azure-administration]、[Microsoft Operations Management Suite][operations-management-suite]、[Chef][chef] 或 [Puppet][puppet] 等集中式管理工具来简化整个系统的管理。 这些工具可以整合从多个 VM 捕获的诊断和运行状况信息，使用户可以全面了解该系统。

## <a name="deploy-the-solution"></a>部署解决方案

[GitHub][github-folder] 中提供了此参考体系结构的部署。 

### <a name="prerequisites"></a>系统必备

在将参考体系结构部署到自己的订阅之前，必须执行以下步骤。

1. 为 [AzureCAT 参考体系结构][ref-arch-repo] GitHub 存储库克隆、下载 zip 文件或创建其分支。

2. 确保在计算机上安装了 Azure CLI 2.0。 若要安装 CLI，请按照[安装 Azure CLI 2.0][azure-cli-2] 中的说明执行操作。

3. 安装 [Azure 构建基块][azbb] npm 包。

  ```bash
  npm install -g @mspnp/azure-building-blocks
  ```

4. 从命令提示符、bash 提示符或 PowerShell 提示符下通过使用以下命令之一，登录到 Azure 帐户，然后按照提示进行操作。

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a>使用 azbb 部署解决方案

若要为 N 层应用程序参考体系结构部署 Linux VM，请执行以下步骤：

1. 导航到在以上前决条件的第 1 步中克隆的存储库的 `virtual-machines\n-tier-linux` 文件夹。

2. 参数文件为部署中的每个 VM 指定默认管理员用户名称和密码。 必须在部署参考体系结构前，对其进行更改。 打开 `n-tier-linux.json` 文件，然后将每个“adminUsername”和“adminPassword”字段替换为新设置。   保存文件。

3. 使用 azbb 命令行工具部署参考体系结构，如下所示。

  ```bash
  azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-linux.json --deploy
  ```

若要详细了解如何使用 Azure 构建基块部署此示例参考体系结构，请访问 [GitHub 存储库][git]。

<!-- links -->
[multi-dc]: multi-region-application.md
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-vm]: ./multi-vm.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[守护主机]: https://en.wikipedia.org/wiki/Bastion_host
[cassandra-in-azure]: https://docs.datastax.com/en/datastax_enterprise/4.5/datastax_enterprise/install/installAzure.html
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[datastax]: http://www.datastax.com/products/datastax-enterprise
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-linux
[lb-external-create]: /azure/load-balancer/load-balancer-get-started-internet-portal
[lb-internal-create]: /azure/load-balancer/load-balancer-get-started-ilb-arm-portal
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-rules]: /azure/azure-resource-manager/best-practices-resource-manager-security#network-security-groups
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[公共 IP 地址]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[Nagios]: https://www.nagios.org/
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[0]: ./images/n-tier-diagram.png "使用 Microsoft Azure 的 N 层体系结构"

