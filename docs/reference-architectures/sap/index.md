---
title: 在 Azure 上部署 SAP NetWeaver 和 SAP HANA
description: 有关在 Azure 上的高可用性环境中运行 SAP HANA 的成熟做法。
author: njray
ms.date: 06/29/2017
ms.openlocfilehash: 33171164c59a520a87ef3209c5bb1b208377221c
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/30/2018
---
# <a name="deploy-sap-netweaver-and-sap-hana-on-azure"></a>在 Azure 上部署 SAP NetWeaver 和 SAP HANA

此参考体系结构演示有关在 Azure 上的高可用性环境中运行 SAP HANA 的一套成熟做法。 [**部署此解决方案**。](#deploy-the-solution)

![0][0]

下载此体系结构的 [Visio 文件][visio-download]。

> [!NOTE]
> 部署此参考体系结构需要获取 SAP 产品和其他非 Microsoft 技术的相应许可。 有关 Microsoft 与 SAP 之间的合作关系的信息，请参阅 [Azure 上的 SAP HANA][sap-hana-on-azure]。

## <a name="architecture"></a>体系结构

该体系结构包括以下组件。

- **虚拟网络 (VNet)**。 VNet 是 Azure 中逻辑隔离的网络的表示形式。 此参考体系结构中的所有 VM 将部署到同一 VNet。 VNet 进一步细分为子网。 为每个层（包括应用程序 (SAP NetWeaver)、数据库 (SAP HANA)、管理 (Jumpbox) 和 Active Directory）创建单独的子网。

- **虚拟机 (VM)**。 此体系结构的 VM 将分组到多个不同的层。

    - **SAP NetWeaver**。 包括 SAP ASCS、SAP Web 调度程序和 SAP 应用程序服务器。 
    
    - **SAP HANA**。 此参考体系结构对单个 [GS5][vm-sizes-mem] 实例上运行的数据库层使用 SAP HANA。 SAP HANA 经认证适合用于 GS5 或 [Azure 上的 SAP HANA 大型实例][azure-large-instances] 中的生产 OLAP 工作负荷。 此参考体系结构适用于 G 系列和 M 系列中的 Azure 虚拟机。 有关 Azure 上的 SAP HANA 大型实例的信息，请参阅 [Azure 上的 SAP HANA（大型实例）概述和体系结构][azure-large-instances]。
   
    - **Jumpbox**。 也称为守护主机。 这是管理员在网络上用来连接其他 VM 的安全 VM。 
     
    - **Windows Server Active Directory (AD) 域控制器。** 域控制器用于配置 Windows Server 故障转移群集（请参阅下文）。
 
- **可用性集**。 将 SAP Web 调度程序、SAP 应用程序服务器和 SAP ACSC 角色的 VM 放入单独的可用性集，并为每个角色至少预配两个 VM。 这样，VM 便可以满足更高的服务级别协议 (SLA)。
    
- **NIC。** 运行 SAP NetWeaver 和 SAP HANA 的 VM 需要两个网络接口 (NIC)。 每个 NIC 分配到不同的子网，以分离不同类型的流量。 有关详细信息，请参阅下面的[建议](#recommendations)。

- **Windows Server 故障转移群集**。 将运行 SAP ACSC 的 VM 配置为故障转移群集，以实现高可用性。 为了支持故障转移群集，SIOS DataKeeper Cluster Edition 会通过复制群集节点拥有的独立磁盘来执行群集共享卷 (CSV) 功能。 有关详细信息，请参阅[在 Microsoft 平台上运行 SAP 应用程序][running-sap]。
    
- **负载均衡器。** 使用两个 [Azure 负载均衡器][azure-lb]实例。 示意图左侧显示的第一个负载均衡器将流量分配到 SAP Web 调度程序 VM。 此配置可根据 [SAP Web 调度程序的高可用性][sap-dispatcher-ha]中所述实现并行 Web 调度程序选项。 右侧显示的第二个负载均衡器通过将传入的连接定向到活动/正常的节点，在 Windows 服务器故障转移群集中启用故障转移。

- **VPN 网关。** VPN 网关将本地网络扩展到 Azure VNet。 也可以使用 ExpressRoute，该选项使用不绕道公共 Internet 的专用连接。 示例解决方案中未部署网关。 有关详细信息，请参阅[将本地网络连接到 Azure][hybrid-networking]。

## <a name="recommendations"></a>建议

你的要求可能不同于此处描述的体系结构。 请使用以下建议作为入手点。

### <a name="load-balancers"></a>负载均衡器

[SAP Web 调度程序][sap-dispatcher]处理对发往双堆栈服务器（ABAP 和 Java）的 HTTP(S) 流量的负载均衡。 多年以来，SAP 一直在提供有口皆碑的单堆栈应用程序服务器，当今只有极少数应用程序还在双堆栈部署模型中运行。 体系结构示意图中所示的 Azure 负载均衡器为 SAP Web 调度程序实施高可用性群集。

对发往应用程序服务器的流量执行的负载均衡在 SAP 内部处理。 对于通过 DIAG 和远程函数调用 (RFC) 连接到 SAP 服务器的 SAPGUI 客户端发出的流量，SCS 消息服务器通过创建 SAP 应用服务器[登录组][logon-groups]进行负载均衡。 

SMLG 是一个 SAP ABAP 事务，用于管理 SAP 中心服务的登录负载均衡功能。 登录组的后端池包含多个 ABAP 应用程序服务器。 访问 ASCS 群集服务的客户端通过前端 IP 地址连接到 Azure 负载均衡器。 ASCS 群集虚拟网络名称也包含 IP 地址。 可以选择性地将此地址与 Azure 负载均衡器上的其他 IP 地址相关联，以便能够远程管理群集。  

### <a name="nics"></a>NIC

SAP 布局管理功能需要分离不同 NIC 上的服务器流量。 例如，业务数据应与管理流量和备份流量相隔离。 将多个 NIC 分配到不同的子网即可实现此数据分离。 有关详细信息，请参阅[为 SAP NetWeaver 和 SAP HANA 构建高可用性][sap-ha] (PDF) 中的“网络”。

将管理 NIC 分配到管理子网，将数据通信 NIC 分配到独立的子网。 有关配置详细信息，请参阅[创建并管理具有多个 NIC 的 Windows 虚拟机][multiple-vm-nics]。

### <a name="azure-storage"></a>Azure 存储

对于所有数据库服务器 VM，我们建议使用 Azure 高级存储，以保持一致的读/写延迟。 对于 SAP 应用程序服务器，包括 (A)SCS 虚拟机，可以使用标准 Azure 存储，因为应用程序执行在内存中发生，只将磁盘用于日志记录。

为了获得最高可靠性，我们建议使用 [Azure 托管磁盘][managed-disks]。 托管磁盘可确保隔离可用性集中 VM 的磁盘，以避免单一故障点。

> [!NOTE]
> 目前，此参考体系结构的资源管理器模板不使用托管磁盘。 我们正在计划将该模板更新为使用托管磁盘。

若要实现较高的 IOPS 和磁盘带宽吞吐量，可向 Azure 存储布局应用优化存储卷性能时采用的常见做法。 例如，将多个磁盘一起条带化以创建更大的磁盘卷可以提高 IO 性能。 针对不经常更改的存储内容启用读取缓存可增强数据检索的速度。 有关性能要求的详细信息，请参阅 [SAP 说明 1943937 - 硬件配置检查工具][sap-1943937]。

对于备份数据存储，我们建议使用 Azure Blob 存储的[冷存储层][cool-blob-storage]。 冷存储层是存储不经常访问且长期留存的数据的经济高效方式。

## <a name="scalability-considerations"></a>可伸缩性注意事项

在 SAP 应用程序层，Azure 提供多种虚拟机大小供纵向扩展。 有关详尽列表，请参阅 [SAP 说明 1928533 - Azure 上的 SAP 应用程序：支持的产品和 Azure VM 类型][sap-1928533]。 通过将更多 VM 添加到可用性集进行横向扩展。

对于同时包含 OLTP 和 OLAP SAP 应用程序的 Azure 虚拟机上的 SAP HANA，SAP 认证的虚拟机大小是 GS5，其中只能包含一个 VM 实例。 对于共置在 Microsoft Azure 认证数据中心内物理服务器上的 SAP HANA 所用的较大型工作负荷，Microsoft 还提供 [Azure 大型实例][azure-large-instances]，其中目前提供多达 4 TB 的内存容量用于托管单个实例。 凭借多达 32 TB 的总内存容量，还可以实现多节点配置。

## <a name="availability-considerations"></a>可用性注意事项

在集中式数据库上实施的这种 SAP 应用程序分布式安装中，将会复制基本安装以实现高可用性。 对于体系结构的每个层，高可用性设计各不相同：

- **Web 调度程序。** 高可用性是使用处理 SAP 应用程序流量的冗余 SAP Web 调度程序实例实现的。 请参阅 SAP 文档中的 [SAP Web 调度程序][swd]。

- **ASCS。** 若要对 Azure Windows 虚拟机上的 ASCS 实现高可用性，可将 Windows 服务器故障转移群集与 SIOS DataKeeper 配合使用，以实施群集共享卷。 有关实施方式的详细信息，请参阅[在 Azure 上组建 SAP ASCS 群集][clustering]。

- **应用程序服务器。** 可通过对应用程序服务器池中的流量进行负载均衡来实现高可用性。

- **数据库层。** 此参考体系结构部署单个 SAP HANA 数据库实例。 为实现高可用性，可部署多个实例，并使用 HANA 系统复制 (HSR) 来实现手动故障转移。 若要实现自动故障转移，需要特定 Linux 分发版的 HA 扩展。

### <a name="disaster-recovery-considerations"></a>灾难恢复注意事项

每个层使用不同的策略提供灾难恢复 (DR) 保护。

- **应用程序服务器。** SAP 应用程序服务器不包含业务数据。 在 Azure 上，简单的 DR 策略是在另一个区域中创建 SAP 应用程序服务器。 在主要应用程序服务器上进行任何配置更改或内核更新，都必须将相同的更改复制到 DR 区域中的 VM。 例如，将内核可执行文件复制到 DR VM。

- **SAP 中心服务。** SAP 应用程序堆栈的此组件也不会保存业务数据。 可以在 DR 区域中生成一个用于运行 SCS 角色的 VM。 在主要 SCS 节点中，要同步的唯一内容是 **/sapmnt** 共享内容。 此外，如果主要 SCS 服务器上发生配置更改或内核更新，必须在 DR SCS 上重复这些操作。 若要同步两个服务器，只需使用定期计划的复制作业将 **/sapmnt** 复制到 DR 端。 有关生成、复制和测试故障转移过程的详细信息，请下载[ SAP NetWeaver：生成基于 Hyper-V 和 Microsoft Azure 的灾难恢复解决方案][sap-netweaver-dr]，并参阅“4.3. SAP SPOF 层 (ASCS)”。

- **数据库层。** 使用 HANA 支持的复制解决方案，例如 HSR 或存储复制。 

## <a name="manageability-considerations"></a>可管理性注意事项

SAP HANA 提供一个使用底层 Azure 基础结构的备份功能。 若要备份 Azure 虚拟机上运行的 SAP HANA 数据库，可同时使用 SAP HANA 快照和 Azure 存储快照，以确保备份文件的一致性。 有关详细信息，请参阅 [Azure 虚拟机上的 SAP HANA 备份指南][hana-backup]和 [Azure 备份服务常见问题解答][backup-faq]。

Azure 提供多种功能用于[监视和诊断][monitoring]整个基础结构。 此外，Azure Operations Management Suite (OMS) 可处理 Azure 虚拟机（Linux 或 Windows）的增强型监视。

若要针对 SAP 基础结构的资源和服务性能提供基于 SAP 的监视，可使用 Azure SAP 增强型监视扩展。 此扩展会将 Azure 监视统计信息馈送到 SAP 应用程序，以执行操作系统监视和 DBA Cockpit 功能。 

## <a name="security-considerations"></a>安全注意事项

SAP 具有自身的用户管理引擎 (UME)，可在 SAP 应用程序中控制基于角色的访问和授权。 有关详细信息，请参阅 [SAP HANA 安全性 - 概述][sap-security]。 （需要创建一个 SAP 服务 Marketplace 帐户进行访问。）

在基础结构安全性方面，传输中数据和静态数据会受到保护。 可参考 [Azure 虚拟机 (VM) 上的 SAP NetWeaver - 规划和实施指南][netweaver-on-azure]的“安全注意事项”部分解决网络安全问题。 该指南还指定了为使应用程序能够通信，而必须在防火墙中打开的网络端口。 

若要加密 Windows 和 Linux IaaS 虚拟机磁盘，可以使用 [Azure 磁盘加密][disk-encryption]。 Azure 磁盘加密使用 Windows 的 BitLocker 功能和 Linux 的 DM-Crypt 功能，为操作系统和数据磁盘提供卷加密。 该解决方案还可与 Azure Key Vault 配合工作，帮助管理 Key Vault 订阅中的磁盘加密密钥和机密。 虚拟机磁盘上的数据将在 Azure 存储中静态加密。

对于 SAP HANA 静态数据加密，我们建议使用 SAP HANA 本机加密技术。

> [!NOTE]
> 请不要在同一台服务器上同时使用 HANA 静态数据加密和 Azure 磁盘加密。

考虑使用[网络安全组][nsg] (NSG) 来限制 VNet 中不同子网之间的流量。

## <a name="deploy-the-solution"></a>部署解决方案 

[GitHub][github] 中提供了此参考体系结构的部署脚本。


### <a name="prerequisites"></a>先决条件

- 必须有权访问 SAP 软件下载中心才能完成安装。
 
- 安装最新版本的 [Azure PowerShell][azure-ps]。 

- 此部署需要 51 个核心。 在部署之前，请验证订阅是否有足够的 VM 核心配额。 如果没有，请使用 Azure 门户提交支持请求，以获取更高的配额。
 
- 此部署使用 GS 系列 VM。 在[此处][region-availability]检查 GS 系列是否在每个区域中可用。

- 若要估算此项部署的成本，请参阅 [Azure 定价计算器][azure-pricing]。 
 
此参考体系结构部署以下 VM：

| 资源名称 | VM 大小 | 目的  |
|---------------|---------|----------|
| `ra-sapApps-scs-vm1` ... `ra-sapApps-scs-vmN` | DS11v2 | SAP 中心服务 |
| `ra-sapApps-vm1` ... `ra-sapApps-vmN` | DS11v2 | SAP NetWeaver 应用程序 |
| `ra-sap-wdp-vm1` ... `ra-sap-wdp-vmN` | DS11v2 | SAP Web 调度程序 |
| `ra-sap-data-vm1` | GS5 | SAP HANA 数据库实例 |
| `ra-sap-jumpbox-vm1` | DS1V2 | Jumpbox |

部署单个 SAP HANA 实例。 对于应用程序 VM，需在模板参数中指定要部署的实例数。

### <a name="deploy-sap-infrastructure"></a>部署 SAP 基础结构

可以增量方式部署此体系结构，或一次性部署整个体系结构。 对于首次部署，我们建议采用增量方式，以了解每个部署步骤的作用。 使用以下 *mode* 参数之一指定增量部署

| Mode           | 作用                                                                                                            |
|----------------|-----------------------------------------------------|
| infrastructure | 在 Azure 中部署网络基础结构。        |
| workload       | 将 SAP 服务器部署到网络。             |
| 本应返回的所有记录的总数，            | 部署上述所有部署。              |

若要部署解决方案，请执行以下步骤：

1. 将 [GitHub 存储库][github]下载或克隆到本地计算机。

2. 打开 PowerShell 窗口并导航到 `/sap/sap-hana/` 文件夹。

3. 运行以下 Azure PowerShell cmdlet。 对于 `<subscription id>`，请使用自己的 Azure 订阅 ID。 对于 `<location>`，请指定一个 Azure 区域，例如 `eastus` 或 `westus`。 对于 `<mode>`，请指定上面所列的模式之一。

    ```powershell
     .\Deploy-ReferenceArchitecture -SubscriptionId <subscription id> -Location <location> -ResourceGroupName <resource group> <mode>
    ```

4.  根据提示登录到 Azure 帐户。 

部署脚本可以需要花费几个小时才能完成，具体时间取决于所选的模式。

> [!WARNING]
> 参数文件在不同的位置包含了硬编码的密码 (`AweS0me@PW`)。 在部署之前，请更改这些值。
 
### <a name="configure-sap-applications-and-database"></a>配置 SAP 应用程序和数据库

部署 SAP 基础结构后，请按如下所述在虚拟机上安装并配置 SAP 应用程序和 HANA 数据库。

> [!NOTE]
> 如需 SAP 安装说明，必须使用 SAP 支持门户用户名和密码来下载 [SAP 安装指南][sap-guide]。

1. 登录到 Jumpbox (`ra-sap-jumpbox-vm1`)。 将使用 Jumpbox 登录到其他 VM。 

2.  对于名为 `ra-sap-wdp-vm1` ... `ra-sap-wdp-vmN` 的每个 VM，请登录到该 VM，然后使用 [Web 调度程序安装][sap-dispatcher-install] wiki 中所述的步骤安装并配置 SAP Web 调度程序实例。

3.  登录到名为 `ra-sap-data-vm1` 的 VM。 使用 [SAP HANA 服务器安装和更新指南][hana-guide]安装并配置 SAP Hana 数据库实例。

4. 对于名为 `ra-sapApps-scs-vm1` ... `ra-sapApps-scs-vmN` 的每个 VM，请登录到该 VM，然后使用 [SAP 安装指南][sap-guide]安装并配置 SAP 中心服务 (SCS)。

5.  对于名为 `ra-sapApps-vm1` ... `ra-sapApps-vmN` 的每个 VM，请登录到该 VM，然后使用 [SAP 安装指南][sap-guide]安装并配置 SAP NetWeaver 应用程序。

**此参考体系结构的供稿人** &mdash; Rick Rainey、Ross Sponholtz、Ben Trinh

[azure-large-instances]: /azure/virtual-machines/workloads/sap/hana-overview-architecture
[azure-lb]: /azure/load-balancer/load-balancer-overview
[azure-pricing]: https://azure.microsoft.com/pricing/calculator/
[azure-ps]: /powershell/azure/overview
[backup-faq]: /azure/backup/backup-azure-backup-faq
[clustering]: https://blogs.msdn.microsoft.com/saponsqlserver/2015/05/20/clustering-sap-ascs-instance-using-windows-server-failover-cluster-on-microsoft-azure-with-sios-datakeeper-and-azure-internal-load-balancer/
[cool-blob-storage]: /azure/storage/storage-blob-storage-tiers
[disk-encryption]: /azure/security/azure-security-disk-encryption
[github]: https://github.com/mspnp/reference-architectures/tree/master/sap/sap-hana
[hana-backup]: /azure/virtual-machines/workloads/sap/sap-hana-backup-guide
[hana-guide]: https://help.sap.com/viewer/2c1988d620e04368aa4103bf26f17727/2.0.01/en-US/7eb0167eb35e4e2885415205b8383584.html
[hybrid-networking]: ../hybrid-networking/index.md
[logon-groups]: https://wiki.scn.sap.com/wiki/display/SI/ABAP+Logon+Group+based+Load+Balancing
[managed-disks]: /azure/storage/storage-managed-disks-overview
[monitoring]: /azure/architecture/best-practices/monitoring
[multiple-vm-nics]: /azure/virtual-machines/windows/multiple-nics
[netweaver-on-azure]: /azure/virtual-machines/workloads/sap/planning-guide
[nsg]: /azure/virtual-network/virtual-networks-nsg
[region-availability]: https://azure.microsoft.com/regions/services/
[running-SAP]: https://blogs.msdn.microsoft.com/saponsqlserver/2016/06/07/sap-on-sql-general-update-for-customers-partners-june-2016/
[sap-1943937]: https://launchpad.support.sap.com/#/notes/1943937
[sap-1928533]: https://launchpad.support.sap.com/#/notes/1928533
[sap-dispatcher]: https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/8fe37933114e6fe10000000a421937/frameset.htm
[sap-dispatcher-ha]: https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/9a9a6b48c673e8e10000000a42189b/frameset.htm
[sap-dispatcher-install]: https://wiki.scn.sap.com/wiki/display/SI/Web+Dispatcher+Installation
[sap-guide]: https://service.sap.com/instguides
[sap-ha]: https://support.sap.com/content/dam/SAAP/SAP_Activate/AGS_70.pdf
[sap-hana-on-azure]: https://azure.microsoft.com/services/virtual-machines/sap-hana/
[sap-netweaver-dr]: http://download.microsoft.com/download/9/5/6/956FEDC3-702D-4EFB-A7D3-2DB7505566B6/SAP%20NetWeaver%20-%20Building%20an%20Azure%20based%20Disaster%20Recovery%20Solution%20V1_5%20.docx
[sap-security]: https://archive.sap.com/documents/docs/DOC-62943
[visio-download]: https://archcenter.blob.core.windows.net/cdn/SAP-HANA-architecture.vsdx
[vm-sizes-mem]: /azure/virtual-machines/windows/sizes-memory
[swd]: https://help.sap.com/doc/saphelp_nw70ehp2/7.02.16/en-us/48/8fe37933114e6fe10000000a421937/frameset.htm
[0]: ./images/sap-hana.png "使用 Microsoft Azure 的 SAP HANA 体系结构"
