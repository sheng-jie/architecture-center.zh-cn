---
title: "选择数据仓库"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 9cb3d4d0196b02da76d85c7f7f0e4a2a69d531e9
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-data-warehouse-in-azure"></a>在 Azure 中选择数据仓库

数据仓库是一个集中式、组织有序的关系存储库，其中保存了一个或多个不同源的集成数据。 本主题对 Azure 中的数据仓库选项做了对比。

> [!NOTE]
> 有关何时使用哪种数据仓库的详细信息，请参阅[数据仓库和数据市场](../scenarios/data-warehousing.md)。

## <a name="what-are-your-options-when-choosing-a-data-warehouse"></a>有哪些数据仓库选项？

可以根据需要，使用多个选项在 Azure 中实施数据仓库。 以下列表划分为两个类别：[对称多处理](https://en.wikipedia.org/wiki/Symmetric_multiprocessing) (SMP) 和[大规模并行处理](https://en.wikipedia.org/wiki/Massively_parallel) (MPP)。 

SMP：

- [Azure SQL 数据库](/azure/sql-database/)
- [虚拟机中的 SQL Server](/sql/sql-server/sql-server-technical-documentation)

MPP：

- [Azure 数据仓库](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [HDInsight 上的 Apache Hive](/azure/hdinsight/hadoop/hdinsight-use-hive)
- [HDInsight 上的交互式查询 (Hive LLAP)](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)

经验法规是，基于 SMP 的仓库最适合中小型数据集（最大 4-100 TB），而 MPP 通常适合大数据。 小、中、大数据的界限在一定程度上根据组织的定义和支持基础结构而定。 （请参阅[选择 OLTP 数据存储](oltp-data-stores.md#scalability-capabilities)。） 

除了数据大小以外，工作负荷模式的类型也是一个很大的决定因素。 例如，使用 SMP 解决方案运行复杂的查询可能速度很慢，而要改用 MPP 解决方案。 基于 MPP 系统在处理小型数据时可能会降低性能，因为作业在节点之间分配和合并。 如果数据大小已超过 1 TB 并且预期会不断增长，可考虑选择 MPP 解决方案。 但是，如果数据大小小于 1 TB，但工作负荷超过了 SMP 解决方案的可用资源，则 MPP 也可能是最佳选择。

由数据仓库访问或存储的数据可能来自众多的数据源，包括 [Azure Data Lake Store](/azure/data-lake-store/) 等数据湖。 请观看视频课程 [Azure Data Lake and Azure Data Warehouse: Applying Modern Practices to Your App](https://azure.microsoft.com/resources/videos/build-2016-azure-data-lake-and-azure-data-warehouse-applying-modern-practices-to-your-app/)（Azure Data Lake 和 Azure 数据仓库：将新式做法运用到应用中），其中比较了可以利用 Azure Data Lake 的各种 MPP 服务的不同优势。

SMP 系统的特征是提供共享所有资源（CPU/内存/磁盘）的单个关系数据库管理系统实例。 可以纵向扩展 SMP 系统。 对于 VM 上运行的 SQL Server，可以纵向扩展 VM 大小。 对于 Azure SQL 数据库，可以通过选择不同的服务层来纵向扩展。 

可以通过添加更多计算节点（它们具有自身的 CPU、内存和 I/O 子系统）来横向扩展 MPP 系统。 服务器的纵向扩展存在物理限制，从这一点看，根据工作负荷，横向扩展更有利。 但是，MPP 解决方案需要不同的技能组合，因为数据的查询、建模和分区存在差异，此外，并行处理存在一些独特的考虑因素。 

在决定要使用哪个 SMP 解决方案时，请参阅 [Azure SQL 数据库和 Azure VM 中的 SQL Server 详述](/azure/sql-database/sql-database-paas-vs-sql-server-iaas#a-closer-look-at-azure-sql-database-and-sql-server-on-azure-vms)。 

Azure SQL 数据仓库还可用于小型和中型数据集，其中的工作负荷是计算和内存密集型的。 阅读有关 SQL 数据仓库模式和常见方案的详细信息：

- [SQL 数据仓库模式和对立模式](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/azure-sql-data-warehouse-workload-patterns-and-anti-patterns/)
- [Azure SQL 数据仓库加载模式和策略](https://blogs.msdn.microsoft.com/sqlcat/2017/05/17/azure-sql-data-warehouse-loading-patterns-and-strategies/)
- [将数据迁移到 Azure SQL 数据仓库](https://blogs.msdn.microsoft.com/sqlcat/2016/08/18/migrating-data-to-azure-sql-data-warehouse-in-practice/)
- [使用 Azure SQL 数据仓库的常见 ISV 应用程序模式](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/common-isv-application-patterns-using-azure-sql-data-warehouse/)

## <a name="key-selection-criteria"></a>关键选择条件

若要缩小选择范围，请先回答以下问题：

- 你希望使用托管服务还是由你管理自己的服务器？

- 是否要处理极大型数据集，或非常复杂且长时间运行的查询？ 如果是，请考虑 MPP 选项。 

- 对于大型数据集，数据源是结构化还是非结构化的？ 非结构化数据可能需要在 HDInsight 上的 Spark、Azure Databricks、HDInsight 上的 Hive LLAP 或 Azure Data Lake Analytics 等大数据环境中处理。 所有这些工具可以充当 ELT（提取、加载、转换）和 ETL（提取、转换、加载）引擎。 它们可将处理的数据输出为结构化数据，以便更方便地载入 SQL 数据仓库或其他选项之一。 对于结构化数据，SQL 数据仓库针对需要超高性能的计算密集型工作负荷提供一个称为“计算优化”的性能层。

- 是否需要将历史数据与当前的操作数据分开？ 如果是，请选择需要[业务流程](pipeline-orchestration-data-movement.md)的选项。 这些独立的仓库已针对重度读取访问进行优化，最适合用作单独的历史数据存储。

- 除了 OLTP 数据存储外，是否需要集成来自其他多个源的数据？ 如果是，请考虑可以轻松集成多个数据源的选项。 

- 是否有多租户要求？ 如果是，SQL 数据仓库不能很好地满足此要求。 有关详细信息，请参阅 [SQL 数据仓库模式和对立模式](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/azure-sql-data-warehouse-workload-patterns-and-anti-patterns/)。

- 是否偏向于关系数据存储？ 如果是，请将选项缩小为提供关系数据存储的产品，但另请注意，可以根据需要使用 PolyBase 等工具来查询非关系数据存储。 但是，如果决定使用 PolyBase，请针对工作负荷的非结构化数据集运行性能测试。

- 是否有实时报告要求？ 如果在执行大量的单一实例插入时需要快速的查询响应时间，请将选项缩小为可以支持实时报告的产品。

- 是否需要支持大量并发用户和连接？ 能否支持大量并发用户/连接取决于多个因素。 

    - 对于 Azure SQL 数据库，请根据服务层参阅[规定的资源限制](/azure/sql-database/sql-database-resource-limits)。 
    
    - SQL Server 最多允许 32,767 个用户连接。 在 VM 上运行时，性能取决于 VM 大小和其他因素。 
    
    - SQL 数据仓库对并发查询数和并发连接数施加了限制。 有关详细信息，请参阅 [SQL 数据仓库中的并发性和工作负荷管理](/azure/sql-data-warehouse/sql-data-warehouse-develop-concurrency)。 考虑使用互补性的服务，例如 [Azure Analysis Services](/azure/analysis-services/analysis-services-overview) 来克服 SQL 数据仓库的限制。

- 使用的是哪种类型的工作负荷？ 一般情况下，基于 MPP 仓库解决方案最适合用于面向批处理的分析工作负荷。 如果工作负荷是事务性的，并且附带了许多小规模读/写操作或多个逐行操作，请考虑使用某个 SMP 选项。 但如果在 HDInsight 群集中使用流处理（例如 Spark 流），或者在 Hive 表中存储数据，则不适用此原则。

## <a name="capability-matrix"></a>功能矩阵

以下各表汇总了功能上的关键差异。

### <a name="general-capabilities"></a>常规功能

| | Azure SQL 数据库 | SQL Server (VM) | SQL 数据仓库 | HDInsight 上的 Apache Hive | HDInsight 上的 Hive LLAP |
| --- | --- | --- | --- | --- | --- | -- |
| 是托管服务 | 是 | 否 | 是 | 是 <sup>1</sup> | 是 <sup>1</sup> |
| 需要数据业务流程（保存数据/历史数据的副本） | 否 | 否 | 是 | 是 | 是 |
| 轻松集成多个数据源 | 否 | 否 | 是 | 是 | 是 |
| 支持暂停计算 | 否 | 否 | 是 | 否 <sup>2</sup> | 否 <sup>2</sup> |
| 关系数据存储 | 是 | 是 |  是 | 否 | 否 |
| 实时报告 | 是 | 是 | 否 | 否 | 是 |
| 灵活的备份还原点 | 是 | 是 | 否 <sup>3</sup> | 是 <sup>4</sup> | 是 <sup>4</sup> |
| SMP/MPP | SMP | SMP | MPP | MPP | MPP |

[1] 手动配置和缩放。

[2] 不需 HDInsight 群集时可将其删除，然后重新创建。 将外部数据存储附加到群集，以便在删除群集后可以保留数据。 可以通过创建按需 HDInsight 群集来处理工作负荷，使用 Azure 数据工厂将群集的生命周期自动化，然后在处理完成后删除该群集。

[3] 使用 SQL 数据仓库时，可将数据库还原到过去 7 天的任何可用还原点。 快照 4 到 8 小时启动一次，可供使用 7 天。 快照超过 7 天将过期，其还原点不再可用。

[4] 考虑使用可按需备份和还原的[外部 Hive 元存储](/azure/hdinsight/hdinsight-hadoop-provision-linux-clusters#use-hiveoozie-metastore)。 可对数据或第三方 HDInsight 备份和还原解决方案使用适用于 Blob 存储或 Data Lake Store 的标准备份和还原选项，例如，可以使用 [Imanis Data](https://azure.microsoft.com/blog/imanis-data-cloud-migration-backup-for-your-big-data-applications-on-azure-hdinsight/) 来提高灵活性和易用性。

### <a name="scalability-capabilities"></a>可伸缩性功能

| | Azure SQL 数据库 | SQL Server (VM) |  SQL 数据仓库 | HDInsight 上的 Apache Hive | HDInsight 上的 Hive LLAP |
| --- | --- | --- | --- | --- | --- | -- |
| 用于实现高可用性的的冗余区域服务器  | 是 | 是 | 是 | 否 | 否 |
| 支持查询横向扩展（分布式查询）  | 否 | 否 | 是 | 是 | 是 |
| 动态可伸缩性（纵向扩展）  | 是 | 否 | 是 <sup>1</sup> | 否 | 否 |
| 支持数据的内存中缓存 | 是 |  是 | 否 | 是 | 是 |

[1] SQL 数据仓库允许通过调整数据仓库单位 (DWU) 数目来纵向扩展和缩减。 请参阅[管理 Azure SQL 数据仓库中的计算能力](/azure/sql-data-warehouse/sql-data-warehouse-manage-compute-overview)。

### <a name="security-capabilities"></a>安全功能

| | Azure SQL 数据库 | 虚拟机中的 SQL Server | SQL 数据仓库 | HDInsight 上的 Apache Hive | HDInsight 上的 Hive LLAP |
| --- | --- | --- | --- | --- | --- | -- |
| 身份验证  | SQL / Azure Active Directory (Azure AD) | SQL / Azure AD / Active Directory | SQL / Azure AD | 本地 / Azure AD <sup>1</sup> | 本地 / Azure AD <sup>1</sup> |
| 授权  | 是 | 是 | 是 | 是 | 是 <sup>1</sup> | 是 <sup>1</sup> |
| 审核  | 是 | 是 | 是 | 是 | 是 <sup>1</sup> | 是 <sup>1</sup> |
| 静态数据加密 | 是 <sup>2</sup> | 是 <sup>2</sup> | 是 <sup>2</sup> | 是 <sup>2</sup> | 是 <sup>1</sup> | 是 <sup>1</sup> |
| 行级别安全性 | 是 | 是 | 是 | 否 | 是 <sup>1</sup> | 是 <sup>1</sup> |
| 支持防火墙 | 是 | 是 | 是 | 是 | 是 <sup>3</sup> | 是 <sup>3</sup> |
| 动态数据掩码 | 是 | 是 | 是 | 否 | 是 <sup>1</sup> | 是 <sup>1</sup> |

[1] 需要使用[已加入域的 HDInsight 群集](/azure/hdinsight/domain-joined/apache-domain-joined-introduction)。

[2] 需要使用透明数据加密 (TDE) 来加密和解密静止数据。

[3] [在 Azure 虚拟网络中使用时](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)受支持。

阅读有关保护数据仓库的详细信息：

* [保护 SQL 数据库](/azure/sql-database/sql-database-security-overview#connection-security)
* [保护 SQL 数据仓库中的数据库](/azure/sql-data-warehouse/sql-data-warehouse-overview-manage-security)
* [使用 Azure 虚拟网络扩展 Azure HDInsight](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)
* [已加入域的 HDInsight 群集的企业级 Hadoop 安全性](/azure/hdinsight/domain-joined/apache-domain-joined-introduction)

