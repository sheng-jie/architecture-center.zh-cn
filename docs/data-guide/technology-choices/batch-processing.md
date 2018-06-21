---
title: 选择批处理技术
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 0117798af82f2caa6704dc86e88be57f09c381ea
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/06/2018
ms.locfileid: "30848659"
---
# <a name="choosing-a-batch-processing-technology-in-azure"></a>在 Azure 中选择批处理技术

大数据解决方案经常使用长时间运行的批处理作业来筛选、聚合或者准备数据以用于分析。 这些作业通常涉及从可缩放的存储（例如，HDFS、Azure Data Lake Store 和 Azure 存储）读取源文件，对源文件进行处理，并将输出写入到可缩放存储中的新文件。 

对这类批处理引擎的关键要求是需要它们能够扩展计算能力，以便处理大量数据。 但是，不同于实时处理，批处理会有延迟（从数据引入到计算结果之间的时间），延迟范围为数分钟到数小时。

## <a name="what-are-your-options-when-choosing-a-batch-processing-technology"></a>选择批处理技术时有哪些选项？

在 Azure 中，以下所有数据存储都将满足批处理的核心要求：

- [Azure Data Lake Analytics](/azure/data-lake-analytics/)
- [Azure SQL 数据仓库](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [基于 Spark 的 HDInsight](/azure/hdinsight/spark/apache-spark-overview)
- [基于 Hive 的 HDInsight](/azure/hdinsight/hadoop/hdinsight-use-hive)
- [基于 Hive LLAP 的 HDInsight](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)

## <a name="key-selection-criteria"></a>关键选择条件

若要缩小选择范围，请先回答以下问题：

- 你希望使用托管服务还是由你管理自己的服务器？

- 希望以声明方式还是以命令方式创作批处理逻辑？

- 是否会爆发性地执行批处理？ 如果是，请考虑使用允许暂停群集或其定价模型为按批处理作业的选项。

- 是否需要随批处理查询关系数据存储，例如查找参考数据？ 如果是，请考虑使用允许查询外部关系存储的选项。

## <a name="capability-matrix"></a>功能矩阵

以下各表汇总了功能上的关键差异。 

### <a name="general-capabilities"></a>常规功能

| | Azure Data Lake Analytics | Azure SQL 数据仓库 | 基于 Spark 的 HDInsight | 基于 Hive 的 HDInsight | 基于 Hive LLAP 的 HDInsight |
| --- | --- | --- | --- | --- | --- |
| 是托管服务 | 是 | 是 | 是 <sup>1</sup> | 是 <sup>1</sup> | 是 <sup>1</sup> |
| 支持暂停计算 | 否 | 是 | 否 | 否 | 否 |
| 关系数据存储 | 是 | 是 | 否 | 否 | 否 |
| 可编程性 | U-SQL | T-SQL | Python、Scala、Java、R | HiveQL | HiveQL |
| 编程范例 | 混合使用声明性和命令性方式  | 声明性 | 混合使用声明性和命令性方式 | 声明性 | 声明性 | 
| 定价模型 | 按批处理作业 | 按群集小时 | 按群集小时 | 按群集小时 | 按群集小时 |  

[1] 使用手动配置和缩放。

### <a name="integration-capabilities"></a>集成功能

| | Azure Data Lake Analytics | SQL 数据仓库 | 基于 Spark 的 HDInsight | 基于 Hive 的 HDInsight | 基于 Hive LLAP 的 HDInsight |
| --- | --- | --- | --- | --- | --- |
| 从 Azure Data Lake Store 进行访问 | 是 | 是 | 是 | 是 | 是 |
| 从 Azure 存储进行查询 | 是 | 是 | 是 | 是 | 是 |
| 从外部关系存储进行查询 | 是 | 否 | 是 | 否 | 否 |

### <a name="scalability-capabilities"></a>可伸缩性功能

| | Azure Data Lake Analytics | SQL 数据仓库 | 基于 Spark 的 HDInsight | 基于 Hive 的 HDInsight | 基于 Hive LLAP 的 HDInsight |
| --- | --- | --- | --- | --- | --- |
| 横向扩展粒度  | 按作业 | 按群集 | 按群集 | 按群集 | 按群集 |
| 快速横向扩展（少于 1 分钟） | 是 | 是 | 否 | 否 | 否 |
| 数据的内存中缓存 | 否 | 是 | 是 | 否 | 是 | 

### <a name="security-capabilities"></a>安全功能

| | Azure Data Lake Analytics | SQL 数据仓库 | 基于 Spark 的 HDInsight | Apache Hive on HDInsight | HDInsight 上的 Hive LLAP |
| --- | --- | --- | --- | --- | --- |
| 身份验证  | Azure Active Directory (Azure AD) | SQL / Azure AD | 否 | 本地 / Azure AD <sup>1</sup> | 本地 / Azure AD <sup>1</sup> |
| 授权  | 是 | 是| 否 | 是 <sup>1</sup> | 是 <sup>1</sup> |
| 审核  | 是 | 是 | 否 | 是 <sup>1</sup> | 是 <sup>1</sup> |
| 静态数据加密 | 是| 是 <sup>2</sup> | 是 | 是 | 是 |
| 行级别安全性 | 否 | 是 | 否 | 是 <sup>1</sup> | 是 <sup>1</sup> |
| 支持防火墙 | 是 | 是 | 是 | 是 <sup>3</sup> | 是 <sup>3</sup> |
| 动态数据掩码 | 否 | 否 | 否 | 是 <sup>1</sup> | 是 <sup>1</sup> |

[1] 需要使用[已加入域的 HDInsight 群集](/azure/hdinsight/domain-joined/apache-domain-joined-introduction)。

[2] 需要使用透明数据加密 (TDE) 来加密和解密静止数据。

[3] [在 Azure 虚拟网络中使用时](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)受支持。
