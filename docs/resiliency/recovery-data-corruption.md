---
title: "从数据损坏或意外删除中恢复"
description: "本文可帮助了解如何从数据损坏或意外数据删除中恢复，并可帮助设计有复原能力和高可用性的容错应用程序，以及对灾难恢复进行规划"
author: adamglick
ms.date: 08/18/2016
ms.openlocfilehash: b75c774f85c42f64472167897f08a7302ab50a3f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
[!INCLUDE [header](../_includes/header.md)]
# <a name="azure-resiliency-technical-guidance-recovery-from-data-corruption-or-accidental-deletion"></a>Azure 复原技术指南 - 数据损坏或意外删除后进行恢复
健全的业务连续性计划的一部分是针对数据损坏或意外删除制定计划。 下面提供有关应用程序错误或操作人员失误造成数据损坏或意外删除后进行恢复的信息。

## <a name="virtual-machines"></a>虚拟机
若要在发生应用程序错误或意外删除后保护 Azure 虚拟机（有时称基础结构即服务 VM），可以使用 [Azure 备份](https://azure.microsoft.com/services/backup/)。 使用 Azure 备份可跨多个 VM 磁盘创建一致的备份。 此外，可以跨区域复制备份保管库，以便在发生区域服务中断时进行恢复。

## <a name="storage"></a>存储
请注意，尽管 Azure 存储通过自动化副本提供了数据复原功能，但这不会防止应用程序代码（或开发人员/用户）因为意外或无心的删除、更新等操作而造成数据损坏。 在遇到应用程序或用户错误时保持数据保真需要更为先进的技术，如将数据和审核日志复制到辅助存储位置。 开发人员可以利用 blob [快照功能](https://msdn.microsoft.com/library/azure/ee691971.aspx)，它可以创建 blob 内容的只读时间点快照。 这可以用作 Azure 存储 Blob 数据保真解决方案的基础。

### <a name="blob-and-table-storage-backup"></a>Blob 和表存储备份
尽管 Blob 和表具有很高的持久性，但它们始终代表数据的当前状态。 从意外的数据修改或删除中恢复可能需要将数据还原到以前的状态。 这可以通过利用 Azure 提供的功能来存储和保留时间点副本来实现。

对于 Azure Blob，可以使用 [blob 快照功能](https://msdn.microsoft.com/library/ee691971.aspx)执行时间点备份。 对于每个快照，只需要支付存储自上次快照状态以来 Blob 内的更改所需的存储空间费用。 快照依赖于它们所基于的原始 Blob 的存在，因此建议复制到另一个 Blob 甚至另一个存储帐户。 这可以确保正确保护备份数据不受意外删除的影响。 对于 Azure 表，可以将时间点备份保存到另一个表或 Azure Blob。 有关执行表和 Blob 的应用程序级备份的更详细指南和示例，请参阅以下文章：

* [防止表受到应用程序错误的影响](https://blogs.msdn.microsoft.com/windowsazurestorage/2010/05/03/protecting-your-tables-against-application-errors/)
* [防止 Blob 受到应用程序错误的影响](https://blogs.msdn.microsoft.com/windowsazurestorage/2010/04/29/protecting-your-blobs-against-application-errors/)

## <a name="database"></a>数据库
Azure SQL 数据库提供了多个[业务连续性](/azure/sql-database/sql-database-business-continuity/)（备份、还原）选项。 可以使用[数据库复制](/azure/sql-database/sql-database-copy/)功能或通过[导出](/azure/sql-database/sql-database-export/)和[导入](https://msdn.microsoft.com/library/hh710052.aspx) SQL Server bacpac 文件来复制数据库。 数据库复制提供事务一致的结果，而 bacpac（通过导入/导出服务）则不会提供事务一致的结果。 这两种选项都在数据中心中作为基于队列的服务运行，当前不提供完成时间 SLA。

> [!NOTE]
> 数据库复制和导入/导出服务会对源数据库形成极大的负载。 它们可能会触发资源争用或限制事件。
> 
> 

### <a name="sql-database-backup"></a>SQL 数据库备份
Microsoft Azure SQL 数据库的时间点备份是通过[复制 Azure SQL 数据库](/azure/sql-database/sql-database-copy/)来实现的。 可以使用此命令在同一逻辑数据库服务器或不同服务器上创建数据库的事务一致副本。 无论是哪种情况，数据库副本都完全独立于源数据库并发挥全部功能。 创建的每个副本表示一个时间点恢复选项。 可以通过将新数据库重命名为源数据库名称，完全恢复数据库状态。 或者，可以通过使用 Transact-SQL 查询从新数据库恢复数据的特定子集。 有关 SQL Database 的其他详细信息，请参阅[使用 Azure SQL 数据库确保业务连续性的相关概述](/azure/sql-database/sql-database-business-continuity/)。

### <a name="sql-server-on-virtual-machines-backup"></a>虚拟机上的 SQL Server 备份
对于在 Azure 基础结构即服务虚拟机（通常称为 IaaS 或 Iaas VM）上使用的 SQL Server，有两个选项可供使用：传统备份和日志传送。 通过使用传统备份可以还原到某个特定时间点，但恢复过程较慢。 还原传统备份要求从初始的完整备份开始，然后应用在该备份之后创建的所有备份。 第二个选项是配置日志传送会话，延迟日志备份的还原（例如，延迟两小时）。 这就为从主要数据库发生的错误恢复提供了时间窗口。

## <a name="other-azure-platform-services"></a>其他 Azure 平台服务
某些 Azure 平台服务将信息存储在用户控制的存储帐户或 SQL 数据库中。 如果帐户或存储资源遭到删除或损毁，这可能会造成服务的严重错误。 在这些情况下，请务必维护在这些资源已删除或损坏时可用于重新创建它们的备份。

对于 Azure 网站和 Azure 移动服务，必须备份和维护关联的数据库。 对于 Azure 媒体服务和虚拟机，必须维护相关的 Azure 存储帐户和该帐户中的所有资源。 例如，对于虚拟机，必须备份和管理 Azure Blob 存储中的 VM 磁盘。

## <a name="checklists-for-data-corruption-or-accidental-deletion"></a>数据损坏和意外删除清单
## <a name="virtual-machines-checklist"></a>虚拟机清单
1. 查看本文档的“虚拟机”部分。
2. 使用 Azure 备份（或者在自己的备份系统中使用 Azure Blob 存储和 VHD 快照）来备份和维护 VM 磁盘。

## <a name="storage-checklist"></a>存储清单
1. 查看本文档的“存储”部分。
2. 定期备份重要的存储资源。
3. 考虑对 Blob 使用快照功能。

## <a name="database-checklist"></a>数据库清单
1. 查看本文档的“数据库”部分。
2. 使用“数据库复制”命令创建时间点备份。

## <a name="sql-server-on-virtual-machines-backup-checklist"></a>虚拟机上的 SQL Server 备份清单
1. 查看本文档的“虚拟机上的 SQL Server 备份”部分。
2. 使用传统备份和还原技术。
3. 创建延迟的日志传送会话。

## <a name="web-apps-checklist"></a>Web 应用清单
1. 备份和维护关联的数据库（如果有）。

## <a name="media-services-checklist"></a>媒体服务清单
1. 备份和维护关联的存储资源。

## <a name="more-information"></a>详细信息
有关 Azure 中的备份和还原功能的详细信息，请参阅[存储、备份和恢复方案](https://azure.microsoft.com/documentation/scenarios/storage-backup-recovery/)。


