---
title: 从数据损坏或意外删除中恢复
description: 本文可帮助了解如何从数据损坏或意外数据删除中恢复，并可帮助设计有复原能力和高可用性的容错应用程序，以及对灾难恢复进行规划
author: MikeWasson
ms.date: 01/10/2018
ms.openlocfilehash: 76d2f996750d5a67b67bd5dc4977580f3b8abbc3
ms.sourcegitcommit: 3d6dba524cc7661740bdbaf43870de7728d60a01
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/11/2018
---
# <a name="recover-from-data-corruption-or-accidental-deletion"></a>从数据损坏或意外删除中恢复 

健全的业务连续性计划的一部分是针对数据损坏或意外删除制定计划。 下面提供有关应用程序错误或操作人员失误造成数据损坏或意外删除后进行恢复的信息。

## <a name="virtual-machines"></a>虚拟机

若要在发生应用程序错误或意外删除后保护 Azure 虚拟机 (VM)，可以使用 [Azure 备份](/azure/backup/)。 使用 Azure 备份可跨多个 VM 磁盘创建一致的备份。 此外，可以跨区域复制备份保管库，以便在发生区域服务中断时进行恢复。

## <a name="storage"></a>存储

Azure 存储通过自动化副本提供数据复原能力。 但是，这不会防止应用程序代码或用户有意或无意损坏数据。 在遇到应用程序或用户错误时保持数据保真需要更为先进的技术，如将数据和审核日志复制到辅助存储位置。 

- **块 Blob**。 创建每个块 Blob 的时间点快照。 有关详细信息，请参阅 [Creating a Snapshot of a Blob](/rest/api/storageservices/creating-a-snapshot-of-a-blob)（创建 Blob 的快照）。 对于每个快照，只需要支付存储自上次快照状态以来 Blob 内的更改所需的存储空间费用。 快照依赖于它们所基于的原始 Blob 的存在，因此建议复制到另一个 Blob 甚至另一个存储帐户。 这可以确保正确保护备份数据不受意外删除的影响。 可以使用 [AzCopy](/azure/storage/common/storage-use-azcopy) 或 [Azure PowerShell](/azure/storage/common/storage-powershell-guide-full) 将 Blob 复制到其他存储帐户。

- **文件**。 使用[共享快照（预览版）](/azure/storage/files/storage-how-to-use-files-snapshots)、AzCopy 或 PowerShell 将文件复制到其他存储帐户。

- **表**。 使用 AzCopy 将表数据导出到其他区域中的其他存储帐户。

## <a name="database"></a>数据库

### <a name="azure-sql-database"></a>Azure SQL 数据库 

SQL 数据库每周自动执行完整数据库备份，每小时自动执行差异数据库备份，每 5 到 10 分钟自动执行事务日志备份，防止企业丢失数据。 使用时间点还原将数据库还原到以前的某个时间。 有关详细信息，请参阅：

- [使用自动数据库备份恢复 Azure SQL 数据库](/azure/sql-database/sql-database-recovery-using-backups)

- [使用 Azure SQL 数据库确保业务连续性的相关概述](/azure/sql-database/sql-database-business-continuity)

### <a name="sql-server-on-vms"></a>VM 上的 SQL Server

对于 VM 上运行的 SQL Server，可以使用两个选项：传统备份和日志传送。 使用传统备份可以还原到某个特定的时间点，但恢复过程较慢。 还原传统备份要求从初始的完整备份开始，然后应用在该备份之后创建的所有备份。 第二个选项是配置日志传送会话，延迟日志备份的还原（例如，延迟两小时）。 这就为从主要数据库发生的错误恢复提供了时间窗口。

### <a name="azure-cosmos-db"></a>Azure Cosmos DB

Azure Cosmos DB 会定期自动创建备份。 备份单独存储在另一个存储服务中并在全球复制，以便在发生区域性的灾难时可以复原。 如果意外删除了数据库或集合，可以提交支持票证或联系 Azure 支持，以便从上一次自动备份中还原数据。 有关详细信息，请参阅[使用 Azure Cosmos DB 自动执行在线备份和还原](/azure/cosmos-db/online-backup-and-restore)。

### <a name="azure-database-for-mysql-azure-database-for-postresql"></a>Azure Database for MySQL、Azure Database for PostreSQL

使用 Azure Database for MySQL 或 Azure Database for PostreSQL 时，数据库服务每隔 5 分钟自动备份服务。 通过此自动备份功能，用户可将服务器及其所有数据库还原到某个较早时间点，并还原到新服务器。 有关详细信息，请参阅：

- [如何使用 Azure 门户在 Azure Database for MySQL 中备份和还原服务器](/azure/mysql/howto-restore-server-portal)

- [如何使用 Azure 门户在 Azure Database for PostgreSQL 中备份和还原服务器](/azure/postgresql/howto-restore-server-portal)

