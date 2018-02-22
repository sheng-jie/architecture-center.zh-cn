---
title: "选择分析数据存储"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: b2e5e63982d4b89b95cd28e596d3b882a4a2263e
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-an-analytical-data-store-in-azure"></a>在 Azure 中选择分析数据存储

在[大数据](../concepts/big-data.md)体系结构中，通常需要使用分析数据存储以可供使用分析工具进行查询的结构化格式提供处理的数据。 同时支持对热路径和冷路径进行查询的分析数据存储统称为服务层或数据服务存储。

服务层对来自热路径和冷路径的已处理数据进行处理。 在 [lambda 体系结构](../concepts/big-data.md#lambda-architecture)中，服务层又细分为一个_速度服务_层（该层存储着以增量方式处理的数据）和一个_批处理服务_层（该层包含已进行批处理的输出）。 服务层要求强力支持低延迟的随机读取。 速度层的数据存储还应当支持随机写入，因为将数据批量加载到此存储中会引起非期望的延迟。 另一方面，批处理层的数据存储不需要支持随机写入，但需要支持批量写入。

没有通用于所有数据存储任务的单一最佳数据管理选项。 不同的数据管理解决方案针对不同的任务进行了优化。 大多数实际云应用和大数据流程都具有各种数据存储要求，并且通常使用各种数据存储解决方案的组合。

## <a name="what-are-your-options-when-choosing-an-analytical-data-store"></a>在选择分析数据存储时有哪些选项？

对于 Azure 中的数据服务存储，有多个选项可用，具体取决于你的需求：

- [SQL 数据仓库](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [Azure SQL 数据库](/azure/sql-database/)
- [Azure VM 中的 SQL Server](/sql/sql-server/sql-server-technical-documentation)
- [HBase/Phoenix on HDInsight](/azure/hdinsight/hbase/apache-hbase-overview)
- [Hive LLAP on HDInsight](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)
- [Azure Analysis Services](/azure/analysis-services/analysis-services-overview)
- [Azure Cosmos DB](/azure/cosmos-db/)

这些选项提供了针对不同类型的任务进行了优化的各种数据库模型：

- [键/值](https://msdn.microsoft.com/library/dn313285.aspx#sec7)数据库存放着每个键值的单一序列化对象。 它们适用于存储大量数据，其中你希望为每个给定键值获取一个项，并且不必对该项的其他属性进行查询。
- [文档](https://msdn.microsoft.com/library/dn313285.aspx#sec8)数据库是键/值数据库，其中的值为“文档”。 此上下文中的“文档”是指已命名的字段和值的集合。 数据库通常以诸如 XML、YAML、JSON 或 BSON 的格式存储数据，但可以使用纯文本。 文档数据库可以按非键字段进行查询并定义辅助索引来使查询更为高效。 这使得文档数据库更适用于需要基于比文档键的值更复杂的条件来检索数据的应用程序。 例如，你可以按产品 ID、客户 ID 或客户名称等字段进行查询。
- [列系列](https://msdn.microsoft.com/library/dn313285.aspx#sec9)数据库是键/值数据存储，它们将数据存储的结构安排为称为列系统的相关列的集合。 例如，人口普查数据库可能有一组列用于人员的姓名（名字、中间名、姓氏），一组列用于人员的地址，一组列用于人员的个人资料信息（出生日期、性别）。 该数据库可以在单独的分区中存储每个列系列，同时为人员保留与同一键相关的所有数据。 应用程序可以读取单个列系列，而无需读取实体的所有数据。
- [图形](https://msdn.microsoft.com/library/dn313285.aspx#sec10)数据库将信息存储为对象和关系的集合。 图形数据库可以高效地执行在对象网络以及它们之间的关系中进行遍历的查询。 例如，对象可以是人力资源数据库中的员工，并且你可能希望使“查找直接或间接为 Scott 工作的所有员工”这类查询更为容易。

## <a name="key-selection-criteria"></a>关键选择条件

若要缩小选择范围，请先回答以下问题：

- 是否需要使用作为数据的热路径提供服务的服务存储？ 如果是，请将选项范围缩小到针对速度服务层进行了优化的那些选项。

- 是否需要支持大规模并行处理 (MPP)？在这类处理中，会自动在多个流程或节点之间分布查询。 如果是，请选择一个支持查询横向扩展的选项。

- 是否想要使用关系数据存储？ 如果是，请将选项范围缩小到具有关系数据库模型的那些选项。 但是，请注意，某些非关系存储支持使用 SQL 语法进行查询，并且可以使用 PolyBase 之类的工具来查询非关系数据存储。

## <a name="capability-matrix"></a>功能矩阵

以下各表汇总了功能上的关键差异。

### <a name="general-capabilities"></a>常规功能

| | SQL 数据库 | SQL 数据仓库 | HBase/Phoenix on HDInsight | Hive LLAP on HDInsight | Azure Analysis Services | Cosmos DB |
| --- | --- | --- | --- | --- | --- | --- |
| 是托管服务 | 是 | 是 | 是 <sup>1</sup> | 是 <sup>1</sup> | 是 | 是 |
| 主数据库模型 | 关系（使用列存储索引时的纵栏格式） | 具有纵栏存储的关系表 | 宽列存储 | Hive/内存中 | 表格/MOLAP 语义模型 | 文档存储、图形、键-值存储、宽列存储 |
| SQL 语言支持 | 是 | 是 | 是（使用 [Phoenix](http://phoenix.apache.org/) JDBC 驱动程序） | 是 | 否 | 是 |
| 针对速度服务层进行了优化 | 是 <sup>2</sup> | 否 | 是 | 是 | 否 | 是 |

[1] 使用手动配置和缩放。

[2] 使用内存优化的表和哈希或非聚集索引。
 
### <a name="scalability-capabilities"></a>可伸缩性功能

| | SQL 数据库 | SQL 数据仓库 | HBase/Phoenix on HDInsight | Hive LLAP on HDInsight | Azure Analysis Services | Cosmos DB |
| --- | --- | --- | --- | --- | --- | --- |
| 用于实现高可用性的的冗余区域服务器  | 是 | 是 | 是 | 否 | 否 | 是 | 是 |
| 支持查询横向扩展  | 否 | 是 | 是 | 是 | 是 | 是 |
| 动态可伸缩性（纵向扩展）  | 是 | 是 | 否 | 否 | 是 | 是 |
| 支持数据的内存中缓存 | 是 | 是 | 否 | 是 | 是 | 否 |

### <a name="security-capabilities"></a>安全功能

| | SQL 数据库 | SQL 数据仓库 | HBase/Phoenix on HDInsight | Hive LLAP on HDInsight | Azure Analysis Services | Cosmos DB |
| --- | --- | --- | --- | --- | --- | --- |
| 身份验证  | SQL / Azure Active Directory (Azure AD) | SQL / Azure AD | 本地 / Azure AD <sup>1</sup> | 本地 / Azure AD <sup>1</sup> | Azure AD | 数据库用户 / Azure AD，通过访问控制 (IAM) |
| 静态数据加密 | 是 <sup>2</sup> | 是 <sup>2</sup> | 是 <sup>1</sup> | 是 <sup>1</sup> | 是 | 是 |
| 行级别安全性 | 是 | 否 | 是 <sup>1</sup> | 是 <sup>1</sup> | 是（通过模型中的对象级安全性） | 否 |
| 支持防火墙 | 是 | 是 | 是 <sup>3</sup> | 是 <sup>3</sup> | 是 | 是 |
| 动态数据掩码 | 是 | 否 | 是 <sup>1</sup> | 是 * | 否 | 否 |

[1] 需要使用[已加入域的 HDInsight 群集](/azure/hdinsight/domain-joined/apache-domain-joined-introduction)。

[2] 需要使用透明数据加密 (TDE) 来加密和解密静止数据。

[3] 在 Azure 虚拟网络中使用时。 请参阅[使用 Azure 虚拟网络扩展 Azure HDInsight](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)。
