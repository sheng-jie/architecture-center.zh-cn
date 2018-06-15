---
title: 选择数据传输技术
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 53dcf8a69ad8ae100dbdbb230a9280efd419342a
ms.sourcegitcommit: 85334ab0ccb072dac80de78aa82bcfa0f0044d3f
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/11/2018
ms.locfileid: "35252747"
---
# <a name="transferring-data-to-and-from-azure"></a>将数据传入和传出 Azure

可以根据需要，使用多个选项将数据传入和传出 Azure。

## <a name="physical-transfer"></a>物理传输

在以下情况下，使用物理硬件将数据传入 Azure 是不错的做法：

- 网络较慢或不可靠。
- 提高网络带宽的费用高昂。
- 安全或组织政策不允许在处理敏感数据时使用出站连接。 

如果主要关注数据传输花费的时间，可以运行一项测试来验证网络传输速度是否确实比物理传输更慢。

可以使用两个主要选项以物理方式将数据传输到 Azure：
- **Azure 导入/导出**。 使用 [Azure 导入/导出服务](/azure/storage/common/storage-import-export-service)可将内部 SATA HDD 或 SDD 寄送到 Azure 数据中心，从而安全地将大量数据传输到 Azure Blob 存储。 还可以使用此服务将数据从 Azure 存储传输到硬盘驱动器，然后寄回以便在本地加载。

- **Azure Data Box**。 [Azure Data Box](https://azure.microsoft.com/services/storage/databox/) 是 Microsoft 提供的一种设备，非常类似于 Azure 导入/导出服务。 Microsoft 将会寄送专有、安全、且防篡改的传输设备，并会安排往返物流，在门户中可以跟踪物流状态。 Azure Data Box 服务的一项优势是易用。 无需购买多块硬盘、配备好硬盘，并将文件传输到每块硬盘。 许多行业领先的 Azure 合作伙伴都支持 Azure Data Box，因此可以更轻松地从其产品将数据无缝离线传输到云中。 

## <a name="command-line-tools-and-apis"></a>命令行工具和 API

如果想要以脚本和编程方式传输数据，请考虑这些选项。

- **Azure CLI**。 [Azure CLI](/azure/hdinsight/hdinsight-upload-data#commandline) 是一个跨平台工具，可用于管理 Azure 服务，以及将数据上传到 Azure 存储。 

- **AzCopy**。 在 [Windows](/azure/storage/common/storage-use-azcopy?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) 或 [Linux](/azure/storage/common/storage-use-azcopy-linux?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) 命令行中使用 AzCopy 可以轻松地向/从 Azure Blob、文件和表存储复制数据，同时保证最佳的性能。 AzCopy 支持并发度和并行度，并且可以在复制操作中断后进行恢复。 另外，它的速度比其他大多数选项更快。 [Microsoft Azure 存储数据移动库](/azure/storage/common/storage-use-data-movement-library)是 AzCopy 依赖的核心框架，可实现编程式访问。 它以 .NET Core 库的形式提供。 

- **PowerShell**。 习惯使用 PowerShell 的 Windows 管理员可以选择 [`Start-AzureStorageBlobCopy` PowerShell cmdlet](/powershell/module/azure.storage/start-azurestorageblobcopy?view=azurermps-5.0.0)。  

- **AdlCopy**。 使用 [AdlCopy](/azure/data-lake-store/data-lake-store-copy-data-azure-storage-blob) 可将数据从 Azure 存储 Blob 复制到 Data Lake Store。 此外，它还可用于在两个 Azure Data Lake Store 帐户之间复制数据。 但是，无法使用它将数据从 Data Lake Store 复制到 Azure 存储 Blob。

- **Distcp**。 如果你的 HDInsight 群集有权访问 Data Lake Store，则你可以使用 [Distcp](/azure/data-lake-store/data-lake-store-copy-data-wasb-distcp) 等 Hadoop 生态系统工具在 HDInsight 群集存储 (WASB) 与 Data Lake Store 帐户之间来回复制数据。

- **Sqoop**。 [Sqoop](/azure/hdinsight/hadoop/hdinsight-use-sqoop) 是一个 Apache 项目，也是 Hadoop 生态系统的一部分。 所有 HDInsight 群集上已预装 Sqoop。 使用 Sqoop 可以在 HDInsight 群集与关系数据库（例如 SQL、Oracle、MySQL 等）之间传输数据。 Sqoop 是相关工具的集合，包括导入和导出。 Sqoop 使用 Azure 存储 Blob 或 Data Lake Store 附加存储来配合 HDInsight 群集。

- **PolyBase**。 [PolyBase](/sql/relational-databases/polybase/get-started-with-polybase) 技术可以通过 T-SQL 语言访问数据库外部的数据。 在 SQL Server 2016 中，可以使用 PolyBase 针对 Hadoop 中的外部数据运行查询，或者从 Azure Blob 存储中导入/导出数据。 在 Azure SQL 数据仓库中，可以从 Azure Blob 存储和 Azure Data Lake Store 中导入/导出数据。 目前，PolyBase 是将数据导入 SQL 数据仓库的最快方法。

- **Hadoop 命令行**。 如果数据驻留在 HDInsight 群集头节点中，则可以使用 `hadoop -copyFromLocal` 命令将该数据复制到群集的附加存储，例如 Azure 存储 Blob 或 Azure Data Lake Store。 若要使用 Hadoop 命令，必须先连接到头节点。 连接后，可将文件上传到存储。

## <a name="graphical-interface"></a>图形界面

如果只是要传输少量的文件或数据对象，且不需要将该过程自动化，可以考虑以下选项。

- **Azure 存储资源管理器**。 [Azure 存储资源管理器](https://azure.microsoft.com/features/storage-explorer/)是一个跨平台工具，可用于管理 Azure 存储帐户的内容。 使用它可以上传、下载和管理 Blob、文件、队列、表以及 Azure Cosmos DB 实体。 将它与 Blob 存储配合使用可以管理 Blob 和文件夹，以及在本地文件系统与 Blob 存储之间，或者在存储帐户之间上传和下载 Blob。

- **Azure 门户**。 Blob 存储和 Data Lake Store 都提供基于 Web 的界面用于浏览文件，以及逐个地上传新文件。 如果你希望不必安装任何工具或发出命令就能快速浏览文件，或者只是要上传少量的新文件，则此选项很合适。

## <a name="data-pipeline"></a>数据管道

**Azure 数据工厂** [Azure 数据工厂](/azure/data-factory/)是一个托管服务，最适合用于在许多 Azure 服务和/或本地之间定期传输文件。 可以使用 Azure 数据工厂创建和计划数据驱动型工作流（称为管道），以便从不同的数据存储引入数据。 它可以使用计算服务（例如 Azure HDInsight Hadoop、Spark、Azure Data Lake Analytics、Azure 机器学习）处理和转换数据。 创建数据驱动的工作流用于[协调](../technology-choices/pipeline-orchestration-data-movement.md)和自动化数据移动和数据转换。

## <a name="key-selection-criteria"></a>关键选择条件

对于数据传输方案，请先回答以下问题来选择适合需要的系统：

- 是否需要传输极大量的数据，而通过 Internet 连接传输会花费很长时间、不可靠或过于昂贵？ 如果是，请考虑物理传输。

- 是否偏向于编写数据传输任务的脚本，以便可以重复使用？ 如果是，请选择某个命令行选项或 Azure 数据工厂。

- 是否需要通过网络连接传输极大量的数据？ 如果是，请选择针对大数据进行优化的选项。

- 是否需要将数据传入或传出关系数据库？ 如果是，请选择支持一个或多个关系数据库的选项。 请注意，其中的一些选项还需要 Hadoop 群集。

- 是否需要自动化的数据管道或工作流业务流程？ 如果是，请考虑 Azure 数据工厂。

## <a name="capability-matrix"></a>功能矩阵

以下各表汇总了功能上的关键差异。

### <a name="physical-transfer"></a>物理传输

| | Azure 导入/导出服务 | Azure Data Box |
| --- | --- | --- |
| 外形规格 | 内部 SATA HDD 或 SDD | 安全、防篡改、单个硬件设备 |
| Microsoft 安排寄送物流 | 否 | 是 |
| 与合作伙伴产品集成 | 否 | 是 |
| 定制设备 | 否 | 是 |

### <a name="command-line-tools"></a>命令行工具

**Hadoop/HDInsight**

| | Distcp | Sqoop | Hadoop CLI |
| --- | --- | --- | --- |
| 针对大数据优化 | 是 | 是 |  是 |
| 复制到关系数据库 |  否 | 是 | 否 |
| 从关系数据库复制 |  否 | 是 | 否 |
| 复制到 Blob 存储 |  是 | 是 | 是 |
| 从 Blob 存储复制 | 是 |  是 | 否 |
| 复制到 Data Lake Store | 是 | 是 | 是 |
| 从 Data Lake Store 复制 | 是 | 是 | 否 |

**其他**

| | Azure CLI | AzCopy | PowerShell | AdlCopy | PolyBase |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 兼容的平台 | Linux、OS X、Windows | Linux、Windows | Windows | Linux、OS X、Windows | SQL Server、Azure SQL 数据仓库 | 
| 针对大数据优化 | 否 | 否 | 否 | 是 <sup>1</sup> | 是 <sup>2</sup> |
| 复制到关系数据库 | 否 | 否 | 否 | 否 | 是 | 
| 从关系数据库复制 | 否 | 否 | 否 | 否 | 是 | 
| 复制到 Blob 存储 | 是 | 是 | 是 | 否 | 是 | 
| 从 Blob 存储复制 | 是 | 是 | 是 | 是 | 是 |
| 复制到 Data Lake Store | 否 | 否 | 是 | 是 |  是 | 
| 从 Data Lake Store 复制 | 否 | 否 | 是 | 是 | 是 | 


[1] AdlCopy 经过优化，配合 Data Lake Analytics 帐户使用时可传输大数据。

[2] 可以通过将计算工作负荷推送到 Hadoop，并使用 [PolyBase 横向扩展组](/sql/relational-databases/polybase/polybase-scale-out-groups)在 SQL Server 实例与 Hadoop 节点之间启用并行数据传输，来[提高 PolyBase 的性能](/sql/relational-databases/polybase/polybase-guide#performance)。

### <a name="graphical-interface-and-azure-data-factory"></a>图形界面和 Azure 数据工厂

| | Azure 存储资源管理器 | Azure 门户 * | Azure 数据工厂 |
| --- | --- | --- | --- |
| 针对大数据优化 | 否 | 否 | 是 | 
| 复制到关系数据库 | 否 | 否 | 是 |
| 复制到关系数据库 | 否 | 否 | 是 |
| 复制到 Blob 存储 | 是 | 否 | 是 |
| 从 Blob 存储复制 | 是 | 否 | 是 |
| 复制到 Data Lake Store | 否 | 否 | 是 |
| 从 Data Lake Store 复制 | 否 | 否 | 是 |
| 上传到 Blob 存储 | 是 | 是 | 是 |
| 上传到 Data Lake Store | 是 | 是 | 是 |
| 协调数据传输 | 否 | 否 | 是 |
| 自定义数据转换 | 否 | 否 | 是 |
| 定价模型 | 免费 | 免费 | 按使用情况付费 |

\* 在此语境中，Azure 门户表示使用适用于 Blob 存储和 Data Lake Store 的基于 Web 的浏览工具。

