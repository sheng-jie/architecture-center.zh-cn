---
title: 选择数据存储技术
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: b14611a2dc34bcb145cf420441795d4124e7baeb
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/06/2018
ms.locfileid: "30847203"
---
# <a name="choosing-a-big-data-storage-technology-in-azure"></a>在 Azure 中选择大数据存储技术

本主题将大数据解决方案的数据存储选项进行比较 &mdash; 具体而言，将用于批量数据引入和批处理的数据存储，与用于[分析数据存储](./analytical-data-stores.md)或[实时流引入](./real-time-ingestion.md)的数据存储进行比较。

## <a name="what-are-your-options-when-choosing-data-storage-in-azure"></a>在 Azure 中选择数据存储时有哪些选项？

可以根据需要，使用多个选项将数据引入 Azure：

**文件存储**

- [Azure 存储 Blob](/azure/storage/blobs/storage-blobs-introduction)
- [Azure Data Lake Store](/azure/data-lake-store/)

**NoSQL 数据库**

- [Azure Cosmos DB](/azure/cosmos-db/)
- [HDInsight 上的 HBase](http://hbase.apache.org/)

## <a name="azure-storage-blobs"></a>Azure 存储 Blob

Azure 存储是高度可用、安全、持久、可缩放和冗余的托管存储服务。 Microsoft 为你负责维护并处理关键问题。 Azure 存储是 Azure 提供的最普及的存储解决方案，可配合大量的服务和工具使用。

可以使用各种 Azure 存储服务来存储数据。 存储许多数据源中的 Blob 的最灵活选项是 [Blob 存储](/azure/storage/blobs/storage-blobs-introduction)。 Blob 本质上是文件。 它们可以存储图片、文档、HTML 文件、虚拟硬盘 (VHD)、大数据（例如日志）、数据库备份 &mdash; 几乎包括任何内容。 Blob 存储在类似于文件夹的容器中。 一个容器包含一组 Blob 集。 一个存储帐户可以包含无限数量的容器，一个容器可以存储无限数量的 Blob。

因为灵活、高度可用且成本低廉，Azure 存储非常适合大数据和分析解决方案。 它为不同的用例提供热、冷和存档存储层。 有关详细信息，请参阅 [Azure Blob 存储：热、冷和存档存储层](/azure/storage/blobs/storage-blob-storage-tiers)。

可以从 Hadoop（HDInsight 已随附）访问 Azure Blob 存储。 HDInsight 可将 Azure 存储中的 Blob 容器用作群集的默认文件系统。 通过 WASB 驱动程序提供的 Hadoop 分布式的文件系统 (HDFS) 接口，可以针对作为 Blob 存储的结构化或非结构化数据直接运行 HDInsight 中的整套组件。 此外，可以使用 Azure SQL 数据仓库的 PolyBase 功能访问 Azure Blob 存储。

使 Azure 存储成为不错选项的其他功能包括：

- [多个并发策略](/azure/storage/common/storage-concurrency?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)。
- [灾难恢复和高可用性选项](/azure/storage/common/storage-disaster-recovery-guidance?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)。
- [静态加密](/azure/storage/common/storage-service-encryption?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)。
- [基于角色的访问控制 (RBAC)](/azure/storage/common/storage-security-guide?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#management-plane-security)，使用 Azure Active Directory 用户和组控制访问。

## <a name="azure-data-lake-store"></a>Azure Data Lake Store

[Azure Data Lake Store](/azure/data-lake-store/) 是一个企业范围的超大规模存储库，适用于大数据分析工作负荷。 使用 Data Lake 可以在单个[安全](/azure/data-lake-store/data-lake-store-overview#DataLakeStoreSecurity)位置捕获任何大小、类型和引入速度的数据进行操作和探索分析。

Data Lake Store 不对帐户大小、文件大小或 Data Lake 中可存储的数据量施加任何限制。 通过创建多个副本来长期存储数据，数据在 Data Lake 中的存储持续时间没有限制。 除了创建文件的多个副本来防范任何意外的故障以外，Data Lake 还会在大量独立存储服务器之间分布文件的各个部分。 这可改善执行数据分析时并行读取文件的吞吐量。

使用与 WebHDFS 兼容的 REST API，可以从 Hadoop（HDInsight 已随附）访问 Data Lake Store。 如果单个文件的大小或总文件大小超过了 Azure 存储支持的限制，可以考虑使用此技术来代替 Azure 存储。 但是，在使用 Data Lake Store 作为 HDInsight 群集的主要存储时，应该遵照[性能优化准则](/azure/data-lake-store/data-lake-store-performance-tuning-guidance#optimizing-io-intensive-jobs-on-hadoop-and-spark-workloads-on-hdinsight)，以及有关 [Spark](/azure/data-lake-store/data-lake-store-performance-tuning-spark)、[Hive](/azure/data-lake-store/data-lake-store-performance-tuning-hive)、[MapReduce](/azure/data-lake-store/data-lake-store-performance-tuning-mapreduce) 和 [Storm](/azure/data-lake-store/data-lake-store-performance-tuning-storm) 的具体准则。 此外，请务必检查 Data Lake Store 的[区域可用性](https://azure.microsoft.com/regions/#services)，因为它的适用区域不如 Azure 存储那么多，并且它必须与 HDInsight 群集位于同一区域。

Data Lake Store 与 Azure Data Lake Analytics 相结合，专为存储数据分析而设计，并已针对数据分析方案优化了性能。 此外，可以使用 Azure SQL 数据仓库的 PolyBase 功能访问 Data Lake Store。

## <a name="azure-cosmos-db"></a>Azure Cosmos DB

[Azure Cosmos DB](/azure/cosmos-db/) 由 Microsoft 提供，是全球分布式多模型数据库。 Cosmos DB 保证全球任意位置在第 99 个百分位为个位数毫秒的延迟，提供多种定义明确的一致性模型以微调性能，并保证多宿主功能的高可用性。

Azure Cosmos DB 与架构无关。 它自动为所有数据编制索引，无需客户管理架构和索引。 它也是多模型、本地支持文档、键值、图和列-系列的数据模型。 

Azure Cosmos DB 功能：

- [异地复制](/azure/cosmos-db/distribute-data-globally)
- 在全球范围内[灵活缩放吞吐量和存储](/azure/cosmos-db/partition-data)
- [五个妥善定义的一致性级别](/azure/cosmos-db/consistency-levels)

## <a name="hbase-on-hdinsight"></a>HDInsight 上的 HBase

[Apache HBase](http://hbase.apache.org/) 是一种开源 NoSQL 数据库，它构建于 Hadoop 基础之上，并基于 Google BigTable 模型化。 HBase 针对按列系列组织的无架构数据库中的大量非结构化和结构化数据提供随机访问和强一致性。

数据存储在表的各行中，行中的数据按列系列分组。 HBase 是无架构数据库，也就是说，在使用其数据前，不必定义列以及列中存储的数据类型。 开放源代码可进行线性伸缩，以处理上千节点上数 PB 的数据。 开放源代码可依赖数据冗余、批处理以及 Hadoop 生态系统中的分布式应用程序提供的其他功能。

[HDInsight 实施](/azure/hdinsight/hbase/apache-hbase-overview)利用 HBase 的横向扩展架构来提供表自动分片、使读写操作保持高度的一致性，以及支持自动故障转移。 性能可通过对读取使用内存中缓存并对写入使用高吞吐量流式处理来提高。 在大多数情况下，我们会[在虚拟网络中创建 HBase 群集](/azure/hdinsight/hbase/apache-hbase-provision-vnet)，使其他 HDInsight 群集和应用程序可以直接访问表。

## <a name="key-selection-criteria"></a>关键选择条件

若要缩小选择范围，请先回答以下问题：

- 是否需要对任何类型的文本或二进制数据使用基于云的高速托管存储？ 如果是，请选择一个文件存储选项。

- 是否需要针对并行分析工作负荷和高吞吐量/IOPS 优化的文件存储？ 如果是，请选择优化了分析工作负荷性能的选项。

- 是否需要在无架构数据库中存储非结构化或半结构化数据？ 如果是，请选择一个非关系数据库选项。 比较索引和数据库模型的选项。 根据需要存储的数据类型，主数据库模型可能是最重要的因素。

- 是否可以在你的区域中使用该服务？ 检查每个 Azure 服务的区域可用性。 参阅[各区域的产品可用性](https://azure.microsoft.com/regions/services/)。

## <a name="capability-matrix"></a>功能矩阵

以下各表汇总了功能上的关键差异。

### <a name="file-storage-capabilities"></a>文件存储功能

|  | Azure Data Lake Store | Azure Blob 存储容器 |
| --- | --- | --- |
| 目的 | 大数据分析工作负荷的优化存储 |用于多种存储方案的常规用途对象存储 |
| 用例 | Batch、流分析和机器学习数据，例如日志文件、IoT 数据、点击流、大型数据集 | 任何类型的文本或二进制数据，例如应用程序后端、备份数据、流式处理媒体存储和常规用途数据 |
| 结构 | 分层文件系统 | 具有平面命名空间的对象存储 |
| 身份验证 | 基于 [Azure Active Directory 标识](/azure/active-directory/active-directory-authentication-scenarios) | 基于共享机密 - [帐户访问密钥](/azure/storage/common/storage-create-storage-account#manage-your-storage-account)、[共享访问签名密钥](/azure/storage/common/storage-dotnet-shared-access-signature-part-1)和[基于角色的访问控制 (RBAC)](/azure/security/security-storage-overview) |
| 身份验证协议 | OAuth 2.0。 调用必须包含 Azure Active Directory 发布的有效的 JWT（JSON Web 令牌） | 基于哈希的消息身份验证代码 (HMAC)。 调用必须包含 Base64 编码的 SHA-256 哈希作为 HTTP 请求的一部分。 |
| 授权 | POSIX 访问控制列表 (ACL)。 可设置基于 Azure Active Directory 标识的 ACL 为文件和文件夹级别。 | 对于帐户级别授权 – 使用[帐户访问密钥](/azure/storage/common/storage-create-storage-account#manage-your-storage-account)。 对于帐户、容器 或 blob 授权 - 使用[共享访问签名密钥](/azure/storage/common/storage-dotnet-shared-access-signature-part-1)。 |
| 审核 | 可用。  |可用 |
| 静态加密 | 透明、服务器端 | 透明、服务器端；客户端加密 |
| Developer SDK | .NET、Java、Python、Node.js | .Net、Java、Python、Node.js、C++、Ruby |
| 分析工作负荷性能 | 并行分析工作负荷的优化性能、高吞吐量和 IOPS | 未进行分析工作负荷优化 |
| 大小限制 | 无帐户大小、文件大小或文件数量限制 | [此处](/azure/azure-subscription-service-limits#storage-limits)记录有具体限制 |
| 异地冗余 | 本地冗余（一个 Azure 区域中数据的多个副本） | 本地冗余 (LRS)、全局冗余 (GRS)、读取访问全局冗余 (RA-GRS)。 详细信息参见[此处](/azure/storage/common/storage-redundancy) |

### <a name="nosql-database-capabilities"></a>NoSQL 数据库功能

|                                    |                                           Azure Cosmos DB                                           |                                                             HDInsight 上的 HBase                                                             |
|------------------------------------|-----------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
|       主数据库模型       |                      文档存储、图形、键-值存储、宽列存储                      |                                                             宽列存储                                                              |
|         辅助索引          |                                                 是                                                 |                                                                     否                                                                     |
|        SQL 语言支持        |                                                 是                                                 |                                     是（使用 [Phoenix](http://phoenix.apache.org/) JDBC 驱动程序）                                      |
|            一致性             |                   非常、有限过期、会话、一致前缀或最终                   |                                                                   非常                                                                   |
| 本机 Azure Functions 集成 |                        [是](/azure/cosmos-db/serverless-computing-database)                        |                                                                     否                                                                     |
|   自动全局分发    |                          [是](/azure/cosmos-db/distribute-data-globally)                           | 使用最终一致性时，无法跨区域[配置 HBase 群集复制](/azure/hdinsight/hbase/apache-hbase-replication) |
|           定价模型            | 按秒计费的弹性可缩放请求单位 (RU)、弹性可缩放的存储 |                              HDInsight 群集（横向缩放节点）和存储按分钟计费                               |

