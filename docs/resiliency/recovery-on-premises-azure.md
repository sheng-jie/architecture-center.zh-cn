---
title: 技术指南：从本地恢复到 Azure
description: 本文可帮助了解和设计从本地基础结构到 Azure 的恢复系统
author: adamglick
ms.date: 08/18/2016
ms.openlocfilehash: 6992e27d148074b3d60c282318741f45974d1afd
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/06/2018
ms.locfileid: "30847809"
---
[!INCLUDE [header](../_includes/header.md)]

# <a name="azure-resiliency-technical-guidance-recovery-from-on-premises-to-azure"></a>Azure 复原技术指南：从本地恢复到 Azure
Azure 提供一整套服务，可让你将本地数据中心扩展到 Azure，用于高可用性和灾难恢复目的：

* **网络**：通过虚拟专用网络，可以安全地将本地网络扩展到云。
* **计算**：使用本地 Hyper-V 的客户可以将现有虚拟机 (VM)“提起并移动”到 Azure。
* **存储**：StorSimple 可将文件系统扩展到 Azure 存储。 Azure 备份服务提供将文件和 SQL 数据库备份到 Azure 存储的功能。
* **数据库复制**：使用 SQL 2014（或更高版本）可用性组，可以为本地数据实现高可用性和灾难恢复。

## <a name="networking"></a>网络
可以使用 Azure 虚拟网络在 Azure 中创建一个逻辑上独立的部分，并可使用 IPsec 连接将其连接到本地数据中心或单个客户端计算机。 通过虚拟网络，可轻松地利用 Azure 中按需可缩放的基础架构，同时可连接到本地数据和应用程序，包括 Windows Server、大型机和 UNIX 上运行的系统。 有关详细信息，请参阅 [Azure 网络文档](/azure/virtual-network/virtual-networks-overview/)。

## <a name="compute"></a>计算
使用本地 Hyper-V 的客户可以将现有的虚拟机“提起并移动”到 Azure 和运行 Windows Server 2012（或更高版本）的服务提供程序，而不需要对 VM 进行更改或转换 VM 格式。 有关详细信息，请参阅[关于 Azure 虚拟机的磁盘和 VHD](/azure/virtual-machines/virtual-machines-linux-about-disks-vhds/?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

## <a name="azure-site-recovery"></a>Azure Site Recovery
如果希望灾难恢复作为一项服务 (DRaaS)，Azure 提供了 [Azure Site Recovery](https://azure.microsoft.com/services/site-recovery/)。 Azure Site Recovery 为 VMware、Hyper-V 和物理服务器提供全面的保护。 借助 Azure Site Recovery，可以使用另一台本地服务器或 Azure 作为恢复站点。 有关 Azure Site Recovery 的详细信息，请参阅 [Azure Site Recovery 文档](https://azure.microsoft.com/documentation/services/site-recovery/)。

## <a name="storage"></a>存储
可以使用多个选项将 Azure 用作本地数据的备份站点。

### <a name="storsimple"></a>StorSimple
StorSimple 可安全、透明地整合本地应用程序的云存储。 它还提供单个设备来实现高性能的分层本地和云存储、实时存档、基于云的数据保护和灾难恢复。 有关详细信息，请参阅 [StorSimple 产品页](https://azure.microsoft.com/services/storsimple/)。

### <a name="azure-backup"></a>Azure 备份
Azure 备份让可以使用 Windows Server 2012（或更高版本）、Windows Server 2012 Essentials（或更高版本）和 System Center 2012 Data Protection Manager（或更高版本）中熟悉的备份工具进行云备份。 这些工具提供了独立于备份存储位置（无论是本地磁盘还是 Azure 存储）的备份管理工作流。 数据备份到云后，经过授权的用户可以轻松地恢复到任何服务器的备份。

使用增量备份时，只会将文件更改传输到云。 这有助于高效使用存储空间、降低带宽消耗并支持多个数据版本的时间点恢复。 也可以选择使用其他功能，如数据保留策略、数据压缩和数据传输限制。 使用 Azure 作为备份位置有一个明显的优点，那就是自动在“场外”备份。 这样就不再需要额外对现场备份媒体进行保护了。

有关详细信息，请参阅[什么是 Azure 备份？](/azure/backup/backup-introduction-to-azure-backup/)和[为 DPM 数据配置 Azure 备份](https://technet.microsoft.com/library/jj728752.aspx)。

## <a name="database"></a>数据库
可使用 AlwaysOn 可用性组、数据库镜像、日志传送以及备份和还原与 Azure Blob 存储配合使用，在混合 IT 环境中为 SQL Server 数据库提供灾难恢复解决方案。 所有这些解决方案都使用 Azure 虚拟机上运行的 SQL Server。

AlwaysOn 可用性组可在本地和云中都有数据库副本的混合 IT 环境中使用。 下图显示了此特点。

![混合云体系结构中的 SQL Server AlwaysOn 可用性组](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-3.png)

在基于证书的设置中，数据库镜像也可以横跨本地服务器和云。 下图演示了此概念。

![混合云体系结构中的 SQL Server 数据库镜像](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-4.png)

日志传送可用于同步本地数据库和 Azure 虚拟机中的 SQL Server 数据库。

![混合云体系结构中的 SQL Server 日志传送](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-5.png)

最后，可以直接将本地数据库备份到 Azure Blob 存储。

![在混合云体系结构中将 SQL Server 备份到 Azure Blob 存储](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-6.png)

有关详细信息，请参阅 [Azure 虚拟机中 SQL Server 的高可用性和灾难恢复](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/)和 [Azure 虚拟机中 SQL Server 的备份和还原](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-backup-recovery/)。

## <a name="checklists-for-on-premises-recovery-in-microsoft-azure"></a>Microsoft Azure 中的本地恢复清单
### <a name="networking"></a>网络
1. 查看本文档的“网络”部分。
2. 使用虚拟网络安全地将本地部署与云相连。

### <a name="compute"></a>计算
1. 查看本文档的“计算”部分。
2. 在 Hyper-V 和 Azure 之间重新定位 VM。

### <a name="storage"></a>存储
1. 查看本文档的“存储”部分。
2. 通过 StorSimple 服务来使用云存储。
3. 使用 Azure 备份服务。

### <a name="database"></a>数据库
1. 查看本文档的“数据库”部分。
2. 考虑使用 Azure VM 上的 SQL Server 作为备份。
3. 设置 AlwaysOn 可用性组。
4. 配置基于证书的数据库镜像。
5. 使用日志传送。
6. 将本地数据库备份到 Azure Blob 存储。


