---
title: "运行用于 N 层体系结构的 Windows VM"
description: "如何在 Azure 上实现多层体系结构，并且特别注意可用性、安全性、可伸缩性和可管理性安全。"
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Windows VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: 0654239a5bbd966a2aa776415b7f15ae723ffd63
ms.sourcegitcommit: c9e6d8edb069b8c513de748ce8114c879bad5f49
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/08/2018
---
# <a name="run-windows-vms-for-an-n-tier-application"></a>运行用于 N 层应用程序的 Windows VM

此参考体系结构显示了一组经过实践检验的、运行用于 N 层应用程序的 Windows 虚拟机 (VM) 的做法。 [**部署此解决方案**。](#deploy-the-solution) 

![[0]][0]

*下载此体系结构的 [Visio 文件][visio-download]。*

## <a name="architecture"></a>体系结构 

有许多方法可用来实现 N 层体系结构。 关系图中显示了一个典型的 3 层 Web 应用程序。 此体系结构基于[运行负载均衡的 VM 以提高可伸缩性和可用性][multi-vm]中所述的内容。 Web 层和业务层都使用负载均衡的 VM。

* **可用性集。** 为每个层创建一个[可用性集][azure-availability-sets]，并且在每个层中至少预配两个 VM。 这样，VM 便可以满足 VM 的更高[服务级别协议 (SLA)][vm-sla]。 可在可用性集中部署单个 VM，但单个 VM 不具备 SLA 保证的资格，除非单个 VM 针对所有 OS 和数据磁盘使用 Azure 高级存储。  
* **子网。** 为每个层创建一个单独的子网。 使用 [CIDR] 表示法指定地址范围和子网掩码。 
* **负载均衡器。** 使用[面向 Internet 的负载均衡器][load-balancer-external]将传入的 Internet 流量分布到 Web 层，使用[内部负载均衡器][load-balancer-internal]将来自 Web 层的网络流量分布到业务层。
* **Jumpbox。** 也称为[守护主机]。 网络上的一个安全 VM，管理员使用它来连接到其他 VM。 Jumpbox 中的某个 NSG 只允许来自安全列表中的公共 IP 地址的远程流量。 该 NSG 应允许远程桌面 (RDP) 流量。
* **监视。** 可以使用 [Nagios]、[Zabbix] 或 [Icinga] 等监视软件深入了解响应时间、VM 运行时间和系统的整体运行状况。 在置于单独的管理子网中的 VM 上安装监视软件。
* **NSG。** 使用[网络安全组][nsg] (NSG) 来限制 VNet 中的网络流量。 例如，在此处显示的 3 层体系结构中，数据库层不接受来自 Web 前端的流量，仅接受来自业务层和管理子网的流量。
* **SQL Server Always On 可用性组。** 通过启用复制和故障转移，在数据层提供高可用性。
* **Active Directory 域服务 (AD DS) 服务器**。 在 Windows Server 2016 之前，SQL Server Always On 可用性组必须加入到域中。 这是因为可用性组依赖于 Windows Server 故障转移群集 (WSFC) 技术。 Windows Server 2016 引入了在没有 Active Directory 的情况下创建故障转移群集的能力，在这种情况下，AD DS 服务器不是此体系结构所必需的。 有关详细信息，请参阅 [What's new in Failover Clustering in Windows Server 2016][wsfc-whats-new]（Windows Server 2016 中的故障转移群集的新增功能）。
* **Azure DNS**。 [Azure DNS][azure-dns] 是 DNS 域的托管服务，它使用 Microsoft Azure 基础结构提供名称解析。 通过在 Azure 中托管域，可以使用与其他 Azure 服务相同的凭据、API、工具和计费来管理 DNS 记录。

## <a name="recommendations"></a>建议

你的要求可能不同于此处描述的体系结构。 请使用以下建议作为入手点。 

### <a name="vnet--subnets"></a>VNet/子网

在创建 VNet 时，确定每个子网中的资源需要多少 IP 地址。 使用 [CIDR] 表示法为所需的 IP 地址指定子网掩码和足够大的 VNet 地址范围。 使用标准[专用 IP 地址块][private-ip-space]内的一个地址空间，这些地址块为 10.0.0.0/8、172.16.0.0/12 和 192.168.0.0/16。

如果以后需要在 VNet 与本地网络之间设置一个网关，请选择一个不与你的本地网络重叠的地址范围。 在创建 VNet 后，将无法更改地址范围。

在设计子网时一定要牢记功能和安全要求。 同一层或同一角色中的所有 VM 应当置于同一子网，这可能是一个安全边界。 有关设计 VNet 和子网的详细信息，请参阅[规划和设计 Azure 虚拟网络][plan-network]。

针对每个子网，采用 CIDR 表示法指定子网的地址空间。 例如，“10.0.0.0/24”将创建包含 256 个 IP 地址的范围。 VM 可以使用其中的 251 个；5 个会被保留。 请确保地址范围在子网之间不重叠。 请参阅[虚拟网络常见问题解答][vnet faq]。

### <a name="network-security-groups"></a>网络安全组

使用 NSG 规则限制各个层之间的流量。 例如，在上面显示的 3 层体系结构中，Web 层不直接与数据库层进行通信。 为强制实现此目的，数据库层应当阻止来自 Web 层子网的传入流量。  

1. 创建一个 NSG 并将其关联到数据库层子网。
2. 添加一个拒绝来自 VNet 的所有入站流量的规则。 （在规则中使用 `VIRTUAL_NETWORK` 标记。） 
3. 添加一个具有更高优先级的规则，用以允许来自业务层子网的入站流量。 此规则将替代前面的规则，并允许业务层与数据库层进行通信。
4. 添加一个允许来自数据库层子网自身的入站流量的规则。 此规则允许在数据库层中的各个 VM 之间进行通信，这是进行数据库复制和故障转移所必需的。
5. 添加一个允许来自 jumpbox 子网的 RDP 流量的规则。 此规则允许管理员从 jumpbox 连接到数据库层。
   
   > [!NOTE]
   > NSG 包含允许来自 VNet 中的任何入站流量的默认规则。 这些规则无法删除，但可以通过创建优先级更高的规则来替代它们。
   > 
   > 

### <a name="load-balancers"></a>负载均衡器

外部负载均衡器将 Internet 流量分布到 Web 层。 为此负载均衡器创建一个公共 IP 地址。 请参阅[创建面向 Internet 的负载均衡器][lb-external-create]。

内部负载均衡器将网络流量从 Web 层分布到业务层。 若要为此负载均衡器提供专用 IP 地址，请创建一个前端 IP 配置，并将其与业务层的子网相关联。 请参阅[开始创建内部负载均衡器][lb-internal-create]。

### <a name="sql-server-always-on-availability-groups"></a>SQL Server Always On 可用性组

建议使用 [Always On 可用性组][sql-alwayson]以实现高可用性。 在 Windows Server 2016 之前，Always On 可用性组需要一个域控制器，并且可用性组中的所有节点必须在同一 AD 域中。

其他层通过[可用性组侦听程序][sql-alwayson-listeners]连接到数据库。 该侦听程序使得 SQL 客户端能够在不知道 SQL Server 物理实例名称的情况下进行连接。 访问数据库的 VM 必须加入域。 客户端（在本例中为另一个层）使用 DNS 将该侦听程序的虚拟网络名称解析为 IP 地址。

如下所述配置 SQL Server Always On 可用性组：

1. 创建一个 Windows Server 故障转移群集 (WSFC) 群集、一个 SQL Server Always On 可用性组和一个主要副本。 有关详细信息，请参阅 [Always On 可用性组入门 (SQL Server)][sql-alwayson-getting-started]。 
2. 创建一个具有静态专用 IP 地址的内部负载均衡器。
3. 创建一个可用性组侦听程序，并将该侦听程序的 DNS 名称映射到一个内部负载均衡器的 IP 地址。 
4. 为 SQL Server 侦听端口（默认情况下为 TCP 端口 1433）创建一个负载均衡器规则。 该负载均衡器规则必须启用*浮动 IP*，也称为“直接服务器返回”。 这将导致 VM 直接回复客户端，从而实现到主要副本的直接连接。
  
  > [!NOTE]
  > 当启用了浮动 IP 时，前端端口号必须与负载均衡器规则中的后端端口号相同。
  > 
  > 

当 SQL 客户端尝试连接时，负载均衡器会将连接请求路由到主要副本。 如果发生到其他副本的故障转移，则负载均衡器会自动将后续请求路由到新的主要副本。 有关详细信息，请参阅[Configure an ILB listener for SQL Server Always On Availability Groups][sql-alwayson-ilb]（为 SQL Server Always On 可用性组配置 ILB 侦听程序）。

在故障转移期间，现有的客户端连接将关闭。 在故障转移完成后，新连接将被路由到新的主要副本。

如果应用程序执行的读取操作显著多于写入操作，则可以将一些只读查询转移到次要副本。 请参阅[Using a Listener to Connect to a Read-Only Secondary Replica (Read-Only Routing)][sql-alwayson-read-only-routing]（使用侦听程序连接到只读次要副本（只读路由））。

通过执行可用性组的[强制手动故障转移][sql-alwayson-force-failover]来测试部署。

### <a name="jumpbox"></a>Jumpbox

Jumpbox 的性能要求非常低，因此请为 jumpbox 选择一个较小的 VM 大小，例如标准 A1。 

为 jumpbox 创建一个[公共 IP 地址]。 将 jumpbox 放置在与其他 VM 相同的 VNet 中，但将其置于一个单独的管理子网中。

不要允许通过公共 Internet 对运行应用程序工作负荷的 VM 进行 RDP 访问。 相反，对这些 VM 的所有 RDP 访问都必须通过 jumpbox 进行。 管理员登录到 jumpbox，然后从 jumpbox 登录到其他 VM。 Jumpbox 允许来自 Internet 的 RDP 流量，但仅允许来自已知的安全 IP 地址的流量。

若要保护 jumpbox，请创建一个 NSG 并将其应用于 jumpbox 子网。 添加一个 NSG 规则，使其仅允许来自一组安全的公共 IP 地址的 RDP 连接。 可以将 NSG 附加到子网或 jumpbox NIC。 在本例中，我们建议将其附加到 NIC，以便仅允许将 RDP 流量发送到 jumpbox，即使你将其他 VM 添加到了同一子网。

为其他子网配置 NSG 以允许来自管理子网的 RDP 流量。

## <a name="availability-considerations"></a>可用性注意事项

在数据库层，具有多个 VM 不会自动转换为高度可用的数据库。 对于关系数据库，通常需要使用复制和故障转移来实现高可用性。 对于 SQL Server，建议使用 [Always On 可用性组][sql-alwayson]。 

如果需要的可用性高于 [VM 的 Azure SLA][vm-sla] 提供的可用性，请跨两个区域复制应用程序，并使用 Azure 流量管理器进行故障转移。 有关详细信息，请参阅[在多个区域中运行 Windows VM 以实现高可用性][multi-dc]。   

## <a name="security-considerations"></a>安全注意事项

加密静态的敏感数据并使用 [Azure Key Vault][azure-key-vault] 管理数据库加密密钥。 Key Vault 可以将加密密钥存储在硬件安全模块 (HSM) 中。 有关详细信息，请参阅 [Configure Azure Key Vault Integration for SQL Server on Azure VMs][sql-keyvault]（为 Azure VM 上的 SQL Server 配置 Azure Key Vault 集成）。另外，建议将应用程序机密（例如数据库连接字符串）也存储在 Key Vault 中。

请考虑添加一个网络虚拟设备 (NVA) 以在 Internet 与 Azure 虚拟网络之间创建一个外围网络。 NVA 是虚拟设备的一个通用术语，可以执行与网络相关的任务，例如防火墙、包检查、审核和自定义路由。 有关详细信息，请参阅[在 Azure 与 Internet 之间实现外围网络][dmz]。

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

若要为 N 层应用程序参考体系结构部署 Windows VM，请按照以下步骤操作：

1. 导航到在以上前决条件的第 1 步中克隆的存储库的 `virtual-machines\n-tier-windows` 文件夹。

2. 参数文件为部署中的每个 VM 指定默认管理员用户名称和密码。 必须在部署参考体系结构前，对其进行更改。 打开 `n-tier-windows.json` 文件，然后将每个“adminUsername”和“adminPassword”字段替换为新设置。
  
  > [!NOTE]
  > 此部署期间，多个脚本在一些 VirtualMachine 对象的 VirtualMachineExtension 对象和 extensions 设置中运行。 这些脚本中的一些需要你刚更改的管理员用户名和密码。 建议查看这些脚本，确保指定正确的凭据。 如果未指定正确的凭据，部署可能失败。
  > 
  > 

保存文件。

3. 使用 azbb 命令行工具部署参考体系结构，如下所示。

  ```bash
  azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-windows.json --deploy
  ```

若要详细了解如何使用 Azure 构建基块部署此示例参考体系结构，请访问 [GitHub 存储库][git]。


<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-dc]: multi-region-application.md
[multi-vm]: multi-vm.md
[n-tier]: n-tier.md
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-windows-manage-availability#configure-each-application-tier-into-separate-availability-sets
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault
[守护主机]: https://en.wikipedia.org/wiki/Bastion_host
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-windows
[lb-external-create]: /azure/load-balancer/load-balancer-get-started-internet-portal
[lb-internal-create]: /azure/load-balancer/load-balancer-get-started-ilb-arm-portal
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[公共 IP 地址]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[sql-alwayson]: https://msdn.microsoft.com/library/hh510230.aspx
[sql-alwayson-force-failover]: https://msdn.microsoft.com/library/ff877957.aspx
[sql-alwayson-getting-started]: https://msdn.microsoft.com/library/gg509118.aspx
[sql-alwayson-ilb]: /azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-alwayson-int-listener
[sql-alwayson-listeners]: https://msdn.microsoft.com/library/hh213417.aspx
[sql-alwayson-read-only-routing]: https://technet.microsoft.com/library/hh213417.aspx#ConnectToSecondary
[sql-keyvault]: /azure/virtual-machines/virtual-machines-windows-ps-sql-keyvault
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[wsfc-whats-new]: https://technet.microsoft.com/windows-server-docs/failover-clustering/whats-new-in-failover-clustering
[Nagios]: https://www.nagios.org/
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[0]: ./images/n-tier-diagram.png "使用 Microsoft Azure 的 N 层体系结构"