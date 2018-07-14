---
title: 在 Azure 中运行高可用性 SharePoint Server 2016 场
description: 有关在 Azure 中设置高可用性 SharePoint Server 2016 场的成熟做法。
author: njray
ms.date: 08/01/2017
ms.openlocfilehash: 9fe4fc09cf3babdf3ec8e8f27049f90e0047e9f0
ms.sourcegitcommit: 776b8c1efc662d42273a33de3b82ec69e3cd80c5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2018
ms.locfileid: "38987703"
---
# <a name="run-a-high-availability-sharepoint-server-2016-farm-in-azure"></a>在 Azure 中运行高可用性 SharePoint Server 2016 场

此参考体系结构演示有关使用 MinRole 拓扑和 SQL Server Always On 可用性组，在 Azure 中设置高可用性 SharePoint Server 2016 场的一系列成熟做法。 无需提供面向 Internet 的终结点，即可在安全的虚拟网络中部署 SharePoint 场。 [**部署此解决方案**。](#deploy-the-solution) 

![](./images/sharepoint-ha.png)

下载此体系结构的 [Visio 文件][visio-download]。

## <a name="architecture"></a>体系结构

此体系结构构建在[运行用于 N 层应用程序的 Windows VM][windows-n-tier] 中所示的体系结构基础之上。 它在 Azure 虚拟网络 (VNet) 中部署一个具有高可用性的 SharePoint Server 2016 场。 此体系结构适合用于测试或生产环境、包含 Office 365 的 SharePoint 混合基础结构，或用作灾难恢复方案的基础。

该体系结构包括以下组件：

- **资源组**。 [资源组][resource-group]是保存相关 Azure 资源的容器。 SharePoint 服务器使用一个资源组，独立于 VM 的基础结构组件（例如虚拟网络和负载均衡器）使用另一个资源组。

- **虚拟网络 (VNet)**。 VM 部署在具有唯一 Intranet 地址空间的 VNet 中。 VNet 进一步细分为子网。 

- **虚拟机 (VM)**。 VM 部署到 VNet 中，已将专用静态 IP 地址分配到所有 VM。 建议对运行 SQL Server 和 SharePoint Server 2016 的 VM 使用静态 IP 地址，以免重启后 IP 地址缓存出现问题以及地址发生更改。

- **可用性集**。 将每个 SharePoint 角色的 VM 放入单独的[可用性集][availability-set]，并为每个角色至少预配两个虚拟机 (VM)。 这样，VM 便可以满足更高的服务级别协议 (SLA)。 

- **内部负载均衡器**。 [负载均衡器][load-balancer]将 SharePoint 请求流量从本地网络分配到 SharePoint 场的前端 Web 服务器。 

- **网络安全组 (NSG)**。 对于包含虚拟机的每个子网，需创建一个[网络安全组][nsg]。 使用 NSG 限制 VNet 中的网络流量，以隔离子网。 

- **网关**。 网关在本地网络与 Azure 虚拟网络之间提供连接。 连接可以使用 ExpressRoute 或站点到站点 VPN。 有关详细信息，请参阅[将本地网络连接到 Azure][hybrid-ra]。

- **Windows Server Active Directory (AD) 域控制器**。 此参考体系结构部署 Windows Server AD 域控制器。 这些域控制器在 Azure VNet 中运行，并与本地 Windows Server AD 林建立了信任关系。 针对 SharePoint 场资源发出的客户端 Web 请求在 VNet 中进行身份验证，而不是跨网关连接将该身份验证流量发送到本地网络。 在 DNS 中创建 Intranet A 或 CNAME 记录，使 Intranet 用户能够将 SharePoint 场的名称解析成内部负载均衡器的专用 IP 地址。

  SharePoint Server 2016 也支持使用 [Azure Active Directory 域服务](/azure/active-directory-domain-services/)。 Azure AD 域服务提供托管域服务，因此不需在 Azure 中部署和管理域控制器。

- **SQL Server Always On 可用性组**。 为了实现 SQL Server 数据库的高可用性，我们建议创建 [SQL Server Always On 可用性组][sql-always-on]。 将两个虚拟机用于 SQL Server。 一个虚拟机包含主数据库副本，另一个虚拟机包含次要副本。 

- **多数节点 VM**。 此 VM 可让故障转移群集建立仲裁。 有关详细信息，请参阅[了解故障转移群集中的仲裁配置][sql-quorum]。

- **SharePoint 服务器**。 SharePoint 服务器执行 Web 前端、缓存、应用程序和搜索角色。 

- **Jumpbox**。 也称为[守护主机][bastion-host]。 这是管理员在网络上用来连接其他 VM 的安全 VM。 Jumpbox 中的某个 NSG 只允许来自安全列表中的公共 IP 地址的远程流量。 该 NSG 应允许远程桌面 (RDP) 流量。

## <a name="recommendations"></a>建议

你的要求可能不同于此处描述的体系结构。 请使用以下建议作为入手点。

### <a name="resource-group-recommendations"></a>有关资源组的建议

我们建议根据服务器角色区分资源组，并针对属于全局资源的基础结构组件单独创建一个资源组。 在此体系结构中，SharePoint 资源组来自一个组，SQL Server 和其他实用工具资产来自另一个组。

### <a name="virtual-network-and-subnet-recommendations"></a>有关虚拟网络和子网的建议

对每个 SharePoint 角色使用一个子网，此外，对网关和 Jumpbox 各使用一个子网。 

网关子网必须命名为 *GatewaySubnet*。 从虚拟网络地址空间的最后一个部分中分配网关子网地址空间。 有关详细信息，请参阅[使用 VPN 网关将本地网络连接到 Azure][hybrid-vpn-ra]。

### <a name="vm-recommendations"></a>VM 建议

根据标准 DSv2 虚拟机大小，此体系结构至少需要 38 个核心：

- Standard_DS3_v2 虚拟机上有 8 个 SharePoint 服务器（每个服务器需要 4 个核心）= 32 个核心
- Standard_DS1_v2 虚拟机上有 2 个 Active Directory 域控制器（每个域控制器需要 1 个核心）= 2 个核心
- Standard_DS1_v2 虚拟机上有 2 个 SQL Server VM = 2 个核心
- Standard_DS1_v2 虚拟机上有 1 个多数节点 = 1 个核心
- Standard_DS1_v2 虚拟机上有 1 个管理服务器 = 1 个核心

核心总数取决于所选的 VM 大小。 有关详细信息，请参阅下面的[有关 SharePoint Server 的建议](#sharepoint-server-recommendations)。

确保 Azure 订阅提供足够的 VM 核心配额用于部署，否则部署将会失败。 请参阅 [Azure 订阅和服务限制、配额与约束][quotas]。 
 
### <a name="nsg-recommendations"></a>有关 NSG 的建议

建议针对包含 VM 的每个子网创建一个 NSG，以实现子网隔离。 若要配置子网隔离，请添加 NSG 规则用于定义每个子网允许或拒绝的入站或出站流量。 有关详细信息，请参阅[使用网络安全组筛选网络流量][virtual-networks-nsg]。 

请不要将 NSG 分配到网关子网，否则网关将会停止运行。 

### <a name="storage-recommendations"></a>有关存储的建议

场中 VM 的存储配置应该符合对本地部署使用的相应最佳做法。 SharePoint 服务器应该单独提供一个磁盘用于日志。 托管搜索索引角色的 SharePoint 服务器需要提供额外的磁盘空间用于存储搜索索引。 对于 SQL Server，标准做法是将数据和日志区分开来。 为数据库备份存储添加更多磁盘，并为 [tempdb][tempdb] 单独使用一个磁盘。

为了获得最高可靠性，我们建议使用 [Azure 托管磁盘][managed-disks]。 托管磁盘可确保隔离可用性集中 VM 的磁盘，以避免单一故障点。 

> [!NOTE]
> 目前，此参考体系结构的资源管理器模板不使用托管磁盘。 我们正在计划将该模板更新为使用托管磁盘。

为所有 SharePoint 和 SQL Server VM 使用高级托管磁盘。 可为多数节点服务器、域控制器和管理服务器使用标准托管磁盘。 

### <a name="sharepoint-server-recommendations"></a>有关 SharePoint Server 的建议

在配置 SharePoint 场之前，请确保为每个服务创建了一个 Windows Server Active Directory 服务帐户。 对于此体系结构，至少需要提供以下域级帐户才能隔离每个角色的特权：

- SQL Server 服务帐户
- 安装用户帐户
- 服务器场帐户
- 搜索服务帐户
- 内容访问帐户
- Web 应用池帐户
- 服务应用池帐户
- 缓存超级用户帐户
- 缓存超级读取者帐户

对于除搜索索引器以外的所有角色，我们建议使用 [Standard_DS3_v2][vm-sizes-general] VM 大小。 搜索索引器应至少使用 [Standard_DS13_v2][vm-sizes-memory] 大小。 

> [!NOTE]
> 此参考体系结构的资源管理器模板对搜索索引器使用较小的 DS3 大小来测试部署。 对于生产部署，请使用 DS13 或更大大小。 

对于生产工作负荷，请参阅 [SharePoint Server 2016 的硬件和软件要求][sharepoint-reqs]。 

为了满足每秒最低 200 MB 磁盘吞吐量的支持要求，请确保规划好搜索体系结构。 请参阅[在 SharePoint Server 2013 中规划企业搜索体系结构][sharepoint-search]。 另请遵照[有关在 SharePoint Server 2016 中爬网的最佳做法][sharepoint-crawling]中的准则。

此外，请将搜索组件数据存储在高性能的独立存储卷或分区中。 为了减少负载并提高吞吐量，请配置此体系结构中所需的对象缓存用户帐户。 将 Windows Server 操作系统文件、SharePoint Server 2016 程序文件和诊断日志拆分到具有普通性能的三个独立存储卷或分区中。 

有关这些建议的详细信息，请参阅 [SharePoint Server 2016 中的初始部署管理帐户和服务帐户][sharepoint-accounts]。

### <a name="hybrid-workloads"></a>混合工作负荷

此参考体系结构部署一个可用作 [SharePoint 混合环境][sharepoint-hybrid]的 SharePoint Server 2016 场 &mdash; 也就是说，要将 SharePoint Server 2016 扩展成 Office 365 SharePoint Online。 如果已安装 Office Online Server，请参阅 [Azure 中的 Office Web 应用和 Office Online Server 支持能力][office-web-apps]。

此部署中的默认服务应用程序旨在支持混合工作负荷。 可将所有 SharePoint Server 2016 和 Office 365 混合工作负荷部署到此场，无需对 SharePoint 基础结构进行更改，但有一种例外情况：不得将云混合搜索服务应用程序部署到托管现有搜索拓扑的服务器。 因此，必须将一个或多个基于搜索角色的 VM 添加到该场，以支持此混合方案。

### <a name="sql-server-always-on-availability-groups"></a>SQL Server Always On 可用性组

SharePoint Server 2016 无法使用 Azure SQL 数据库，因此，此体系结构使用了 SQL Server 虚拟机。 为了支持 SQL Server 中的高可用性，我们建议使用 Always On 可用性组。Always On 可用性组指定一组可共同故障转移的数据库，使这些数据库具有高可用性和可恢复性。 在此参考体系结构中，部署期间会创建数据库，但必须手动启用 Always On 可用性组，并将 SharePoint 数据库添加到该可用性组。 有关详细信息，请参阅[创建可用性组和添加 SharePoint 数据库][create-availability-group]。

我们还建议将侦听器 IP 地址（SQL Server 虚拟机内部负载均衡器的专用 IP 地址）添加到群集。

有关建议的 VM 大小，以及 Azure 中运行的 SQL Server 的其他性能建议，请参阅[有关 Azure 虚拟机中 SQL Server 的性能最佳做法][sql-performance]。 另请遵循[有关 SharePoint Server 2016 场中 SQL Server 的最佳做法][sql-sharepoint-best-practices]中的建议。

我们建议将多数节点服务器放在复制伙伴的独立计算机上。 通过该服务器，高安全模式会话中的辅助复制伙伴服务器可以判断是否要启动自动故障转移。 与两个伙伴不同，多数节点服务器不会为数据库提供服务，但只是为自动故障转移提供支持。 

## <a name="scalability-considerations"></a>可伸缩性注意事项

若要纵向扩展现有的服务器，只需更改 VM 大小。 

使用 SharePoint Server 2016 中的 [MinRoles][minroles] 功能，可以基于服务器的角色横向扩展服务器，以及从角色中删除服务器。 将服务器添加到某个角色时，可以指定任意单个角色或一个组合角色。 但是，如果将服务器添加到搜索角色，则还必须使用 PowerShell 重新配置搜索拓扑。 还可以使用 MinRoles 转换角色。 有关详细信息，请参阅[在 SharePoint Server 2016 中管理 MinRole 服务器场][sharepoint-minrole]。

请注意，SharePoint Server 2016 不支持使用虚拟机规模集进行自动缩放。

## <a name="availability-considerations"></a>可用性注意事项

此参考体系结构支持 Azure 区域中的高可用性，因为在一个可用性集中至少为每个角色部署了两个 VM。

为了防范区域性故障，请在不同的 Azure 区域中单独创建一个灾难恢复场。 恢复时间目标 (RTO) 和恢复点目标 (RPO) 确定了设置要求。 有关详细信息，请参阅[选择 SharePoint 2016 的灾难恢复策略][sharepoint-dr]。 次要区域应该是与主要区域*配对的区域*。 如果发生大范围的服务中断，会优先恢复每个配对中的一个区域。 有关详细信息，请参阅[业务连续性和灾难恢复 (BCDR)：Azure 配对区域][paired-regions]。

## <a name="manageability-considerations"></a>可管理性注意事项

若要操作和维护服务器、服务器场和站点，请遵循有关 SharePoint 操作的建议做法。 有关详细信息，请参阅 [针对 SharePoint Server 2016 的操作][sharepoint-ops]。

在 SharePoint 环境中管理 SQL Server 时要考虑的任务可能与通常要对数据库应用程序考虑的任务不同。 最佳做法是使用夜间增量备份，每周完全备份所有的 SQL 数据库。 每隔 15 分钟备份一次事务日志。 另一种做法是针对数据库执行 SQL Server 维护任务，同时禁用内置的 SharePoint 任务。 有关详细信息，请参阅[存储和 SQL Server 容量规划与配置][sql-server-capacity-planning]。 

## <a name="security-considerations"></a>安全注意事项

用于运行 SharePoint Server 2016 的域级服务帐户需要使用 Windows Server AD 域控制器来执行域加入和身份验证过程。 不能将 Azure Active Directory 域服务用于此目的。 为了扩展已在 Intranet 中部署的 Windows Server AD 标识基础结构，此体系结构使用了现有本地 Windows Server AD 林的两个 Windows Server AD 副本域控制器。

此外，做好安全巩固规划始终是明智的做法。 其他建议包括：

- 将规则添加到 NSG 以隔离子网和角色。
- 不要向 VM 分配公共 IP 地址。
- 为了检测入侵和分析工作负荷，请考虑在前端 Web 服务器的前面使用网络虚拟设备，而不要使用内部 Azure 负载均衡器。
- 可以选择使用 IPsec 策略来加密服务器之间的明文流量。 如果同时还在进行子网隔离，请更新网络安全组规则，以允许 IPsec 流量。
- 为 VM 安装反恶意软件代理。

## <a name="deploy-the-solution"></a>部署解决方案

[GitHub][github] 中提供了此参考体系结构的部署脚本。 

可以增量方式部署此体系结构，或一次性部署整个体系结构。 对于首次部署，我们建议采用增量方式，以了解每项部署的作用。 使用以下 *mode* 参数之一指定增量部署。

| Mode           | 作用                                                                                                            |
|----------------|-------------------------------------------------------------------------------------------------------------------------|
| onprem         | （可选）部署用于测试或评估的模拟本地网络环境。 此步骤不会连接到实际本地网络。 |
| infrastructure | 将 SharePoint 2016 网络基础结构和 Jumpbox 部署到 Azure。                                                |
| createvpn      | 部署 SharePoint 和本地网络的虚拟网络网关，并将两者相连接。 仅当运行了 `onprem` 步骤时，才需要运行此步骤。                |
| workload       | 将 SharePoint 服务器部署到 SharePoint 网络。                                                               |
| security       | 将网络安全组部署到 SharePoint 网络。                                                           |
| 本应返回的所有记录的总数，            | 部署上述所有部署。                            


若要使用模拟本地网络环境以增量方式部署体系结构，请依序运行以下步骤：

1. onprem
2. infrastructure
3. createvpn
4. workload
5. security

若要以增量方式部署体系结构但不使用模拟本地网络环境，请依序运行以下步骤：

1. infrastructure
2. workload
3. security

若要通过一个步骤部署所有项目，请使用 `all`。 请注意，整个过程可能需要几个小时。

### <a name="prerequisites"></a>先决条件

* 安装最新版本的 [Azure PowerShell][azure-ps]。

* 在部署此参考体系结构之前，请验证订阅是否具有足够的配额 - 至少 38 个核心。 如果没有足够的配额，请使用 Azure 门户提交支持请求，以获取更高的配额。

* 若要估算此项部署的成本，请参阅 [Azure 定价计算器][azure-pricing]。

### <a name="deploy-the-reference-architecture"></a>部署参考体系结构

1.  将 [GitHub 存储库][github]下载或克隆到本地计算机。

2.  打开 PowerShell 窗口并导航到 `/sharepoint/sharepoint-2016` 文件夹。

3.  运行以下 PowerShell 命令。 对于 \<subscription id\>，请使用自己的 Azure 订阅 ID。 对于 \<location\>，请指定一个 Azure 区域，例如 `eastus` 或 `westus`。 对于 \<mode\>，请指定 `onprem`、`infrastructure`、`createvpn`、`workload`、`security` 或 `all`。

    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```   
4. 根据提示登录到 Azure 帐户。 部署脚本可以需要花费几个小时才能完成，具体时间取决于所选的模式。

5. 部署完成后，请运行脚本来配置 SQL Server Always On 可用性组。 有关详细信息，请参阅[自述文件][readme]。

> [!WARNING]
> 参数文件在不同的位置包含了硬编码的密码 (`AweS0me@PW`)。 在部署之前，请更改这些值。


## <a name="validate-the-deployment"></a>验证部署

部署此参考体系结构之后，所用订阅的下面会列出以下资源组：

| 资源组        | 目的                                                                                         |
|-----------------------|-------------------------------------------------------------------------------------------------|
| ra-onprem-sp2016-rg   | 包含 Active Directory 且与 SharePoint 2016 网络联合的模拟本地网络 |
| ra-sp2016-network-rg  | 用于支持 SharePoint 部署的基础结构                                                 |
| ra-sp2016-workload-rg | SharePoint 和支持性资源                                                             |

### <a name="validate-access-to-the-sharepoint-site-from-the-on-premises-network"></a>验证从本地网络对 SharePoint 站点的访问

1. 在 [Azure 门户][azure-portal]中的“资源组”下，选择 `ra-onprem-sp2016-rg` 资源组。

2. 在资源列表中，选择名为 `ra-adds-user-vm1` 的 VM 资源。 

3. 根据[连接到虚拟机][connect-to-vm]中所述连接到该 VM。 用户名为 `\onpremuser`。

5.  与该 VM 建立远程连接后，请在该 VM 中打开浏览器并导航到 `http://portal.contoso.local`。

6.  在“Windows 安全性”框中，使用用户名 `contoso.local\testuser` 登录到 SharePoint 门户。

执行此项登录会在本地网络使用的 Fabrikam.com 域与 SharePoint 门户使用的 contoso.local 域之间建立隧道。 SharePoint 站点打开后，便会出现根演示站点。

### <a name="validate-jumpbox-access-to-vms-and-check-configuration-settings"></a>验证是否可以通过 Jumpbox 访问 VM，并检查配置设置

1.  在 [Azure 门户][azure-portal]中的“资源组”下，选择 `ra-sp2016-network-rg` 资源组。

2.  在资源列表中，选择名为 `ra-sp2016-jb-vm1` 的 VM 资源，即 Jumpbox。

3. 根据[连接到虚拟机][connect-to-vm]中所述连接到该 VM。 用户名为 `testuser`。

4.  登录到 Jumpbox 后，请从 Jumpbox 打开 RDP 会话。 连接到 VNet 中的其他任何 VM。 用户名为 `testuser`。 可以忽略有关远程计算机安全证书的警告。

5.  打开与 VM 的远程连接后，请查看配置，并使用服务器管理器等管理工具进行更改。

下表显示了已部署的 VM。 

| 资源名称      | 目的                                   | 资源组        | VM 名称                       |
|--------------------|-------------------------------------------|-----------------------|-------------------------------|
| Ra-sp2016-ad-vm1   | Active Directory + DNS                    | Ra-sp2016-network-rg  | Ad1.contoso.local             |
| Ra-sp2016-ad-vm2   | Active Directory + DNS                    | Ra-sp2016-network-rg  | Ad2.contoso.local             |
| Ra-sp2016-fsw-vm1  | SharePoint                                | Ra-sp2016-network-rg  | Fsw1.contoso.local            |
| Ra-sp2016-jb-vm1   | Jumpbox                                   | Ra-sp2016-network-rg  | Jb (use public IP to log on) |
| Ra-sp2016-sql-vm1  | SQL Always On - 故障转移                  | Ra-sp2016-network-rg  | Sq1.contoso.local             |
| Ra-sp2016-sql-vm2  | SQL Always On - 主副本                   | Ra-sp2016-network-rg  | Sq2.contoso.local             |
| Ra-sp2016-app-vm1  | SharePoint 2016 应用程序 MinRole       | Ra-sp2016-workload-rg | App1.contoso.local            |
| Ra-sp2016-app-vm2  | SharePoint 2016 应用程序 MinRole       | Ra-sp2016-workload-rg | App2.contoso.local            |
| Ra-sp2016-dch-vm1  | SharePoint 2016 分布式缓存 MinRole | Ra-sp2016-workload-rg | Dch1.contoso.local            |
| Ra-sp2016-dch-vm2  | SharePoint 2016 分布式缓存 MinRole | Ra-sp2016-workload-rg | Dch2.contoso.local            |
| Ra-sp2016-srch-vm1 | SharePoint 2016 搜索 MinRole            | Ra-sp2016-workload-rg | Srch1.contoso.local           |
| Ra-sp2016-srch-vm2 | SharePoint 2016 搜索 MinRole            | Ra-sp2016-workload-rg | Srch2.contoso.local           |
| Ra-sp2016-wfe-vm1  | SharePoint 2016 Web 前端 MinRole     | Ra-sp2016-workload-rg | Wfe1.contoso.local            |
| Ra-sp2016-wfe-vm2  | SharePoint 2016 Web 前端 MinRole     | Ra-sp2016-workload-rg | Wfe2.contoso.local            |


**此参考体系结构的供稿人** &mdash; Joe Davies、Bob Fox、Neil Hodgkinson、Paul Stork

<!-- links -->

[availability-set]: /azure/virtual-machines/windows/manage-availability
[azure-portal]: https://portal.azure.com
[azure-ps]: /powershell/azure/overview
[azure-pricing]: https://azure.microsoft.com/pricing/calculator/
[bastion-host]: https://en.wikipedia.org/wiki/Bastion_host
[create-availability-group]: https://technet.microsoft.com/library/mt793548(v=office.16).aspx
[connect-to-vm]: /azure/virtual-machines/windows/quick-create-portal#connect-to-virtual-machine
[github]: https://github.com/mspnp/reference-architectures
[hybrid-ra]: ../hybrid-networking/index.md
[hybrid-vpn-ra]: ../hybrid-networking/vpn.md
[load-balancer]: /azure/load-balancer/load-balancer-internal-overview
[managed-disks]: /azure/storage/storage-managed-disks-overview
[minroles]: https://technet.microsoft.com/library/mt346114(v=office.16).aspx
[nsg]: /azure/virtual-network/virtual-networks-nsg
[office-web-apps]: https://support.microsoft.com/help/3199955/office-web-apps-and-office-online-server-supportability-in-azure
[paired-regions]: /azure/best-practices-availability-paired-regions
[readme]: https://github.com/mspnp/reference-architectures/tree/master/sharepoint/sharepoint-2016
[resource-group]: /azure/azure-resource-manager/resource-group-overview
[quotas]: /azure/azure-subscription-service-limits
[sharepoint-accounts]: https://technet.microsoft.com/library/ee662513(v=office.16).aspx
[sharepoint-crawling]: https://technet.microsoft.com/library/dn535606(v=office.16).aspx
[sharepoint-dr]: https://technet.microsoft.com/library/ff628971(v=office.16).aspx
[sharepoint-hybrid]: https://aka.ms/sphybrid
[sharepoint-minrole]: https://technet.microsoft.com/library/mt743705(v=office.16).aspx
[sharepoint-ops]: https://technet.microsoft.com/library/cc262289(v=office.16).aspx
[sharepoint-reqs]: https://technet.microsoft.com/library/cc262485(v=office.16).aspx
[sharepoint-search]: https://technet.microsoft.com/library/dn342836.aspx
[sql-always-on]: /sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server
[sql-performance]: /virtual-machines/windows/sql/virtual-machines-windows-sql-performance
[sql-server-capacity-planning]: https://technet.microsoft.com/library/cc298801(v=office.16).aspx
[sql-quorum]: https://technet.microsoft.com/library/cc731739(v=ws.11).aspx
[sql-sharepoint-best-practices]: https://technet.microsoft.com/library/hh292622(v=office.16).aspx
[tempdb]: /sql/relational-databases/databases/tempdb-database
[virtual-networks-nsg]: /azure/virtual-network/virtual-networks-nsg
[visio-download]: https://archcenter.blob.core.windows.net/cdn/Sharepoint-2016.vsdx
[vm-sizes-general]: /azure/virtual-machines/windows/sizes-general
[vm-sizes-memory]: /azure/virtual-machines/windows/sizes-memory
[windows-n-tier]: ../virtual-machines-windows/n-tier.md
