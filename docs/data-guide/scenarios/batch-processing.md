---
title: "批处理"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: d3d3b92034c251586ecc9caff2785ecd0808b2a7
ms.sourcegitcommit: 943e671a8d522cef5ddc8c6e04848134b03c2de4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/05/2018
---
# <a name="batch-processing"></a>批处理

一种常见的大数据方案是批处理静态数据。 在此方案中，源数据通过源应用程序本身或业务流程工作流载入数据存储。 然后，业务流程工作流启动的并行化作业就地处理该数据。 处理过程可以包括多个迭代步骤。然后，转换的结果将载入可由分析和报告组件查询的分析数据存储。

例如，可将 Web 服务器的日志复制到某个文件夹，然后在夜间对其进行处理，以生成 Web 活动的每日报告。

![](./images/batch-pipeline.png)

## <a name="when-to-use-this-solution"></a>何时使用此解决方案

从简单的数据转换到更完整的 ETL（提取-转换-加载）管道等各种场合，都可以使用批处理。 在大数据上下文中，批处理可以处理极大的数据集，其中的计算会消耗很长时间。 （有关示例，请参阅 [Lambda 体系结构](../concepts/big-data.md#lambda-architecture)。）批处理通常促成进一步的交互式探索，为机器学习提供随时可建模的数据，或者将数据写入针对分析和可视化进行优化的数据存储。

批处理的一个示例是将大量的平面半结构化 CSV 或 JSON 文件转换为架构化和结构化格式供进一步查询。 通常，会将数据从用于引入的原始格式（例如 CSV）转换为查询性能更好的二进制格式，因为二进制以纵栏表格式存储数据，并且通常提供索引和有关数据的内联统计信息。

## <a name="challenges"></a>挑战

- **数据格式和编码**。 如果文件使用意外的格式或编码，会发生一些很难调试的问题。 例如，源文件可能混用 UTF-16 和 UTF-8 编码，或者包含意外的分隔符（空格与制表符）或意外的字符。 另一个常见例子是包含解释为分隔符的制表符、空格或逗号的文本字段。 数据加载和分析逻辑必须足够灵活，可以检测和处理这些问题。

- **协调时间切片。** 通常，源数据被放置在反映处理窗口的文件夹层次结构中，并按年、月、日、小时等属性进行组织。 在某些情况下，数据可能延迟到达。 例如，假设某个 Web 服务器发生故障，3 月 7 日生成的日志可能要到 3 月 9 日才会出现在文件夹中供处理。 会不会因为延迟到达而将其忽略？ 下游处理逻辑是否可以处理无序的记录？

## <a name="architecture"></a>体系结构

批处理体系结构包含以下逻辑组件，如上图所示。

- **数据存储。** 通常是一个分布式文件存储，可充当大量采用各种格式的大型文件的存储库。 一般情况下，这种类型的存储通常称为 Data Lake。 

- **批处理。** 大数据的庞大性通常意味着，解决方案必须使用长时间运行的批处理作业来处理数据文件，以便筛选、聚合和准备用于分析的数据。 这些作业通常涉及读取源文件、对它们进行处理，以及将输出写入到新文件。 

- **分析数据存储。** 许多大数据解决方案会先准备用于分析的数据，然后以结构化格式提供已处理的数据供分析工具查询。 

- **分析和报告。** 大多数大数据解决方案的目标是通过分析和报告提供对数据的见解。 

- **业务流程。** 使用批处理时，通常需要通过某种业务流程将数据迁移或复制到数据存储、批处理、分析数据存储和报告层。

## <a name="technology-choices"></a>技术选择

建议对 Azure 中的批处理解决方案选择以下技术。

### <a name="data-storage"></a>数据存储

- **Azure 存储 Blob 容器**。 许多现有的 Azure 业务流程已在利用 Azure Blob 存储，因此它是大数据存储的适当选择。
- **Azure Data Lake Store**。 Azure Data Lake Store 为任何大小的文件提供几乎无限的存储，并提供广泛的安全选项，因此非常适合极大规模的大数据解决方案，让它们在一个集中式存储中存储异类格式的数据。

有关详细信息，请参阅[数据存储](../technology-choices/data-storage.md)。

### <a name="batch-processing"></a>批处理

- **U-SQL**。 U-SQL 是 Azure Data Lake Analytics 使用的查询处理语言。 它将 SQL 的声明性 C# 的过程扩展性相结合，利用并行度来实现高效的大规模数据处理。
- **Hive**。 Hive 是类似 SQL 的语言，受大多数 Hadoop 分发版（包括 HDInsight）的支持。 使用它可以处理任何 HDFS 兼容存储（包括 Azure Blob 存储和 Azure Data Lake Store）中的数据。
- **Pig**。 Pig 是许多 Hadoop 分发版（包括 HDInsight）中使用的声明性大数据处理语言。 它特别适合用于处理非结构化或半结构化数据。
- **Spark**。 Spark 引擎支持以各种语言（包括 Java、Scala 和 Python）编写的批处理程序。 Spark 使用分布式体系结构跨多个工作节点并行处理数据。

有关详细信息，请参阅[批处理](../technology-choices/batch-processing.md)。

### <a name="analytical-data-store"></a>分析数据存储

- **SQL 数据仓库**。 Azure SQL 数据仓库是经过优化的基于 SQL Server 数据库技术的托管服务，支持大规模数据仓库工作负荷。
- **Spark SQL**。 Spark SQL 是在 Spark 基础上构建的 API，支持创建可使用 SQL 语法查询的数据帧和表。
- **HBase**。 HBase 是一种低延迟的 NoSQL 存储，为查询结构化和半结构化数据提供高性能、灵活的选项。
- **Hive**。 除了用于批处理以外，Hive 还提供概念上类似于典型关系数据库管理系统的数据库体系结构。 借助 Hive 查询性能的创新改进（例如 Tez 引擎和 Stinger 方案），在某些场合下，可以有效利用 Hive 表作为分析查询的源。

有关详细信息，请参阅[分析数据存储](../technology-choices/analytical-data-stores.md)。

### <a name="analytics-and-reporting"></a>分析和报告

- **Azure Analysis Services**。 许多大数据解决方案包含集中式联机分析处理 (OLAP) 数据模型（通常称为多维数据集）并据此生成报告、仪表板和交互式“切片和切块”，因此可以模拟传统的企业商业智能体系结构。 Azure Analysis Services 支持创建多维和表格模型来满足此需求。
- **Power BI**。 数据分析师可以使用 Power BI，基于 OLAP 模型中的数据模型或者直接从分析数据存储创建交互式数据可视化效果。
- **Microsoft Excel**。 Microsoft Excel 是全球最广泛使用的软件应用程序之一，提供大量的数据分析和可视化功能。 数据分析师可以使用 Excel 从分析数据存储构建文档数据模型，或者将 OLAP 数据模型中的数据检索到交互式数据透视表和图表。

有关详细信息，请参阅[分析和报告](../technology-choices/analysis-visualizations-reporting.md)。

### <a name="orchestration"></a>业务流程

- **Azure 数据工厂** 使用 Azure 数据工厂管道可以定义一系列根据重复临时窗口计划的活动。 这些活动可以在按需 HDInsight 群集中启动数据复制操作，以及：Hive、Pig、MapReduce 或 Spark 作业；Azure Date Lake Analytics 中的 U-SQL 作业；Azure SQL 数据仓库或 Azure SQL 数据库中的存储过程。
- **Oozie** 和 **Sqoop**。 Oozie 是 Apache Hadoop 生态系统的作业自动化引擎，可用于启动数据复制操作和 Hive、Pig 与 MapReduce 作业来处理数据，以及启动 Sqoop 作业在 HDFS 与 SQL 数据库之间复制数据。

有关详细信息，请参阅[管道业务流程](../technology-choices/pipeline-orchestration-data-movement.md)
