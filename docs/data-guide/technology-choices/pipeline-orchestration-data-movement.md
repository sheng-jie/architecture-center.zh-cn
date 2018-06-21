---
title: 选择数据管道业务流程技术
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 17aeb871bc815793295ed610795e5e83de72c637
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/14/2018
ms.locfileid: "29288799"
---
# <a name="choosing-a-data-pipeline-orchestration-technology-in-azure"></a>在 Azure 中选择数据管道业务流程技术

大多数大数据解决方案包括重复数据处理操作，这些操作封装在工作流中。 管道业务流程协调程序是一个工具，可帮助自动执行这些工作流。 业务流程协调程序可以计划作业、执行工作流和协调任务之间的依赖关系。

## <a name="what-are-your-options-for-data-pipeline-orchestration"></a>数据管道业务流程有哪些选项？

在 Azure 中，以下服务和工具可满足管道业务流程、控制流和数据移动的核心要求：

- [Azure 数据工厂](/azure/data-factory/)
- [Oozie on HDInsight](/azure/hdinsight/hdinsight-use-oozie-linux-mac)
- [SQL Server Integration Services (SSIS)](/sql/integration-services/sql-server-integration-services)

这些服务和工具可以彼此独立地使用，也可以一起使用以创建混合解决方案。 例如，Azure 数据工厂 V2 中的集成运行时 (IR) 可以以本机方式在托管 Azure 计算环境中执行 SSIS 包。 尽管这些服务在功能上有某种重叠，但有几个关键差异。

## <a name="key-selection-criteria"></a>关键选择条件

若要缩小选择范围，请先回答以下问题：

- 是否需要使用大数据功能来移动和转换数据？ 通常，这意味着几 GB 到几 TB 的数据。 如果是，则将选项的范围缩小到最适合大数据的选项。

- 是否需要可以大规模运行的托管服务？ 如果是，则选择不受本地处理能力限制的基于云的服务之一。

- 某些数据源是否位于本地？ 如果是，请查找可以同时使用云和本地数据源或目标的选项。

- 源数据是否存储在 HDFS 文件系统上的 Blob 存储中？ 如果是这样，请选择支持 Hive 查询的选项。

## <a name="capability-matrix"></a>功能矩阵

下表汇总了功能上的关键差异。

### <a name="general-capabilities"></a>常规功能

| | Azure 数据工厂 | SQL Server Integration Services (SSIS) | Oozie on HDInsight
| --- | --- | --- | --- |
| 托管 | 是 | 否 | 是 |
| 基于云 | 是 | 否（本地） | 是 |
| 先决条件 | Azure 订阅 | SQL Server  | Azure 订阅、HDInsight 群集 |
| 管理工具 | Azure 门户、PowerShell、CLI、.NET SDK | SSMS、PowerShell | Bash shell、Oozie REST API、Oozie Web UI |
| 定价 | 按使用情况付费 | 许可/为功能付费 | 在运行 HDInsight 群集之上无需额外付费 |

### <a name="pipeline-capabilities"></a>管道功能

| | Azure 数据工厂 | SQL Server Integration Services (SSIS) | Oozie on HDInsight
| --- | --- | --- | --- |
| 复制数据 | 是 | 是 | 是 |
| 自定义转换 | 是 | 是 | 是（MapReduce、Pig 和 Hive 作业） |
| Azure 机器学习评分 | 是 | 是（使用脚本） | 否 |
| HDInsight 按需 | 是 | 否 | 否 |
| Azure 批处理 | 是 | 否 | 否 |
| Pig、Hive、MapReduce | 是 | 否 | 是 |
| Spark | 是 | 否 | 否 |
| 执行 SSIS 包 | 是 | 是 | 否 |
| 控制流 | 是 | 是 | 是 |
| 访问本地数据 | 是 | 是 | 否 |

### <a name="scalability-capabilities"></a>可伸缩性功能

| | Azure 数据工厂 | SQL Server Integration Services (SSIS) | Oozie on HDInsight
| --- | --- | --- | --- |
| 纵向扩展 | 是 | 否 | 否 |
| 向外扩展 | 是 | 否 | 是（通过将工作节点添加到群集） |
| 针对大数据进行了优化 | 是 | 否 | 是 |

