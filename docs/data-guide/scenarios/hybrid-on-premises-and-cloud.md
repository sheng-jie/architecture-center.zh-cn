---
title: 将本地数据解决方案扩展到云
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 66fc225dc123202ba587d82f15ea0883e1bbf3b5
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/14/2018
ms.locfileid: "29289683"
---
# <a name="extending-on-premises-data-solutions-to-the-cloud"></a>将本地数据解决方案扩展到云

当组织将工作负荷和数据转移到云后，其本地数据中心通常可继续发挥重要作用。 术语“混合云”是指公有云与本地数据中心的组合，可以建立跨越两者的集成式 IT 环境。 有些组织使用混合云作为一种途径，在特定的一段时间内将整个数据中心迁移到云中。 还有一些组织使用云服务来扩展其现有的本地基础结构。 

本文介绍管理混合云解决方案中的数据时的注意事项和最佳做法。

## <a name="when-to-use-a-hybrid-solution"></a>何时使用混合解决方案

对于以下情况，可考虑使用混合解决方案：

* 需要采取某种过渡策略，在较长时间内逐步迁移到完全的云解决方案。
* 法规或政策不允许将特定的数据或工作负荷转移到云中。
* 为实现灾难恢复和容错，需要在本地与云环境之间复制数据和服务。
* 为了减少本地数据中心与远程位置之间的延迟，需要将体系结构的一部分托管在 Azure 中。

## <a name="challenges"></a>挑战

* 建立一致的环境以满足安全、管理和开发需求，并避免重复工作。

* 在本地与云环境之间创建可靠、低延迟且安全的数据连接。

* 复制数据并修改应用程序和工具，以便在每个环境中使用正确的数据存储。

* 保护并加密在云中托管、但要从本地访问的数据，反之亦然。

## <a name="on-premises-data-stores"></a>本地数据存储

本地数据存储包括数据库和文件。 有多种原因需要将它们保留在本地。 法规或政策可能不允许将特定的数据或工作负荷转移到云中。 数据主权、隐私或安全因素可能导致更适合将数据保留在本地。 在迁移期间，可将某些数据保留在尚未迁移的应用程序本地。

将应用程序数据放在公有云中的注意事项包括：

* **成本**。 Azure 中的存储成本可能比在本地数据中心维护类似特征的存储的成本要低得多。 当然，许多公司投资购置了高端 SAN，因此，在现有硬件老旧之前，这种成本优势可能不会达到极致。
* **弹性缩放**。 规划和管理本地环境中数据容量的增长可能有一定的难度，尤其是数据增长难以预测时。 这些应用程序可以利用按需容量，以及云中几乎没有极限的存储空间。 对于由大小相对稳定的数据集构成的应用程序而言，不太需要考虑这个因素。
* **灾难恢复**。 Azure 中存储的数据可以在一个 Azure 区域中以及跨地理区域自动复制。 在混合环境中，可以使用相同的技术在本地与基于云的数据存储之间复制。

## <a name="extending-data-stores-to-the-cloud"></a>将数据存储扩展到云

可以采用多种做法将本地数据存储扩展到云。 一种做法是创建本地副本和云副本。 这有助于实现较高程度的容错，但可能需要对应用程序进行更改，以便在故障转移时能够连接到相应的数据存储。

另一种做法是将部分数据转移到云存储，同时，将较新的或者访问较频繁的数据保留在本地。 这种方法可以提高长期存储的经济效率，同时可以减少操作数据集，从而改善数据访问响应时间。

第三种做法是将所有数据保留在本地，并使用云计算来托管应用程序。 为此，可将应用程序托管在云中，并通过安全连接将其连接到本地数据存储。 

## <a name="azure-stack"></a>Azure Stack

对于整个混合云解决方案，请考虑使用 [Microsoft Azure Stack](/azure/azure-stack/)。 Azure Stack 是一种混合云平台，通过它可从数据中心提供 Azure 服务。 这有助于保持本地与 Azure 之间的一致性，因为可以使用相同的工具，且不需要更改代码。 

下面是 Azure 和 Azure Stack 的一些用例：

* **边缘解决方案和断开连接的解决方案**。 在 Azure Stack 本地处理数据，然后在 Azure 中聚合以作进一步分析，并在两者之间使用共同的应用程序逻辑，以此满足延迟和连接要求。 
* **满足各种法规要求的云应用程序**。 在 Azure 中开发和部署应用程序，使用 Azure Stack 在本地灵活部署相同的应用程序，以满足法规或政策要求。
* **本地云应用程序模型**。 使用 Azure 更新和扩展现有应用程序，或构建新应用程序。 在云中的 Azure 与本地 Azure Stack 之间使用一致的 DevOps 流程。

## <a name="sql-server-data-stores"></a>SQL Server 数据存储

如果在本地运行 SQL Server，可以使用 Microsoft Azure Blob 存储服务进行备份和还原。 有关详细信息，请参阅[使用 Microsoft Azure Blob 存储服务执行 SQL Server 备份和还原](/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service)。 此功能提供无限的场外存储，并可以在本地运行的 SQL Server 与 Azure 中的虚拟机上运行的 SQL Server 之间共享相同的备份。 

[Azure SQL 数据库](/azure/sql-database/)是托管的关系数据库即服务。 由于 Azure SQL 数据库使用 Microsoft SQL Server 引擎，因此应用程序可以使用这两种技术以相同的方式访问数据。 Azure SQL 数据库还能以有利的方式与 SQL Server 相结合。 例如，应用程序可以通过 [SQL Server Stretch Database](/sql/sql-server/stretch-database/stretch-database) 功能访问 SQL Server 数据库中的单个表，同时将该表的部分或所有行存储在 Azure SQL 数据库中。 此技术可自动将定义的一段时间内未访问过的数据转移到云中。 读取此数据的应用程序并不知道有任何数据已转移到云中。

如果需要让数据保持同步，将数据存储保留在本地和云中可能有难度。 若要解决此难题，可以使用 [SQL 数据同步](/azure/sql-database/sql-database-sync-data)这项基于 Azure SQL 数据库的服务，它可以跨多个 Azure SQL 数据库和 SQL Server 实例双向同步选定数据。 尽管使用数据同步可以轻松地在各种数据存储之间保持数据的最新状态，但不应将它用于灾难恢复，或者从本地 SQL Sever 迁移到 Azure SQL 数据库。

若要实现灾难恢复和业务连续性，可以使用 [AlwaysOn 可用性组](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server)在两个或更多个 SQL Server 实例之间复制数据，其中的一些实例可以在另一地理区域中的 Azure 虚拟机上运行。

## <a name="network-shares-and-file-based-data-stores"></a>网络共享和基于文件的数据存储

在混合云体系结构中，组织往往将较新的文件保留在本地，并将较旧的文件存档到云中。 这种方法有时称作文件分层，可实现本地和云中两组文件的无缝访问。 此方法有助于最大程度地减少网络带宽用量，以及访问最频繁的新文件的访问时间。 同时，可以利用基于云的存储来存档数据。 

组织可能还希望将网络共享完全转移到云中。 例如，如果访问网络共享的应用程序也位于云中，则最好采用这种做法。 可以使用[数据业务流程](../technology-choices/pipeline-orchestration-data-movement.md)工具完成此过程。


[Azure StorSimple](/azure/storsimple/) 提供最完整的集成存储解决方案用于管理本地设备与 Azure 云存储之间的存储任务。 StorSimple 是一种经济高效、易于管理的存储区域网络 (SAN) 解决方案，可以消除与企业存储和数据保护相关的很多问题和开支。 它使用专有的 StorSimple 8000 系列设备，与云服务集成，并提供一套集成式管理工具。

将本地网络共享与基于云的文件存储配合使用的另一种方法是借助 [Azure 文件](/azure/storage/files/storage-files-introduction)。 Azure 文件提供完全托管的文件共享，可以使用标准的[服务器消息块](https://msdn.microsoft.com/library/windows/desktop/aa365233.aspx?f=255&MSPPError=-2147217396) (SMB) 协议（有时称为 CIFS）来访问它。 可以在本地计算机上将 Azure 文件装载为文件共享，或者在访问本地文件或网络共享文件的现有应用程序中使用 Azure 文件。

若要将 Azure 文件中的文件共享与本地 Windows Server 同步，可以使用 [Azure 文件同步](/azure/storage/files/storage-sync-files-planning)。Azure 文件同步的一个主要优势是能够在本地文件服务器与 Azure 文件之间将文件分层。 这样，便可以在本地保留最新的和最近访问的文件。 

有关详细信息，请参阅[确定何时使用 Azure Blob 存储、Azure 文件或 Azure 磁盘](/azure/storage/common/storage-decide-blobs-files-disks)。

## <a name="hybrid-networking"></a>混合网络

本文重点介绍了混合数据解决方案，但是，还要考虑如何将本地网络扩展到 Azure。 有关混合解决方案的此方面，请参阅以下主题：

- [选择用于将本地网络连接到 Azure 的解决方案](../../reference-architectures/hybrid-networking/considerations.md)
- [混合网络参考体系结构](../../reference-architectures/hybrid-networking/index.md)

