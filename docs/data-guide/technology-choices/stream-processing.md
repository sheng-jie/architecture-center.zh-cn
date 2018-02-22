---
title: "选择流处理技术"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: e06f46e2951159219bd8cc430102e2ec0c5d6d4d
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-stream-processing-technology-in-azure"></a>在 Azure 中选择流处理技术

本文对 Azure 中用于实时流处理的各种技术选择进行了比较。

实时流处理使用来自队列或基于文件的存储的消息，对消息进行处理，并将结果转发到其他消息队列、文件存储或数据库。 处理可能包括对消息进行查询、筛选和聚合。 流处理引擎必须能够使用无限的数据流并以最低的延迟生成结果。 有关详细信息，请参阅[实时处理](../scenarios/real-time-processing.md)。

## <a name="what-are-your-options-when-choosing-a-technology-for-real-time-processing"></a>选择用于实时处理的技术时有哪些选项？
在 Azure 中，下列所有数据存储都将满足支持实时处理的核心要求：
- [Azure 流分析](/azure/stream-analytics/)
- [基于 Spark Streaming 的 HDInsight](/azure/hdinsight/spark/apache-spark-streaming-overview)
- [基于 Storm 的 HDInsight](/azure/hdinsight/storm/apache-storm-overview)
- [Azure Functions](/azure/azure-functions/functions-overview)
- [Azure 应用服务 WebJobs](/azure/app-service/web-sites-create-web-jobs)

## <a name="key-selection-criteria"></a>关键选择条件

对于实时处理方案，通过回答以下问题开始选择适合需求的合适服务：

- 希望使用声明性方式还是命令性方式来创作流处理逻辑？

- 是否需要对时间处理或窗口化的内置支持？

- 数据是否会以 Avro、JSON 或 CSV 之外的格式到达？ 如果是，请考虑使用自定义代码支持任何格式的选项。

- 是否需要将处理能力扩展为超出 1 GB/s？ 如果是，请考虑通过群集大小进行缩放的选项。 

## <a name="capability-matrix"></a>功能矩阵

以下各表汇总了功能上的关键差异。 

### <a name="general-capabilities"></a>常规功能
| | Azure 流分析 | 基于 Spark Streaming 的 HDInsight | 基于 Storm 的 HDInsight | Azure Functions | Azure 应用服务 Web 作业 |
| --- | --- | --- | --- | --- | --- | 
| 可编程性 | 流分析查询语言，JavaScript | Scala、Python、Java | Java、C# | C#、F#、Node.js | C#、Node.js、PHP、Java、Python |
| 编程范例 | 声明性 | 混合使用声明性和命令性方式 | 命令性 | 命令性 | 命令性 |    
| 定价模型 | 按流单元 | 按群集小时 | 按群集小时 | 按函数执行和资源消耗 | 按应用服务计划小时 |  

### <a name="integration-capabilities"></a>集成功能
| | Azure 流分析 | 基于 Spark Streaming 的 HDInsight | 基于 Storm 的 HDInsight | Azure Functions | Azure 应用服务 Web 作业 |
| --- | --- | --- | --- | --- | --- | 
| 输入 | [流分析输入](/azure/stream-analytics/stream-analytics-define-inputs)  | 事件中心、IoT 中心、Kafka、HDFS  | 事件中心、IoT 中心、存储 Blob、Azure Data Lake Store  | [支持的绑定](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | 服务总线、存储队列、存储 Blob、事件中心、WebHook、Cosmos DB、文件 |
| 接收器 |  [流分析输出](/azure/stream-analytics/stream-analytics-define-outputs) | HDFS | 事件中心、服务总线、Kafka | [支持的绑定](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | 服务总线、存储队列、存储 Blob、事件中心、WebHook、Cosmos DB、文件 | 

### <a name="processing-capabilities"></a>处理功能
| | Azure 流分析 | 基于 Spark Streaming 的 HDInsight | 基于 Storm 的 HDInsight | Azure Functions | Azure 应用服务 Web 作业 |
| --- | --- | --- | --- | --- | --- | 
| 内置临时/窗口化支持 | 是 | 是 | 是 | 否 | 否 |
| 输入数据格式 | Avro、JSON 或 CSV、UTF-8 编码 | 使用自定义代码的任何格式 | 使用自定义代码的任何格式 | 使用自定义代码的任何格式 | 使用自定义代码的任何格式 |
| 可伸缩性 | [查询分区](/azure/stream-analytics/stream-analytics-parallelization) | 按群集大小界定 | 按群集大小界定 | 最多 200 个并行的函数应用实例处理 | 按应用服务计划容量界定 | 
| 支持延迟到达和乱序事件处理 | 是 | 是 | 是 | 否 | 否 |

另请参阅：

- [选择实时消息引入技术](./real-time-ingestion.md)
- [比较 Apache Storm 和 Azure 流分析](/azure/stream-analytics/stream-analytics-comparison-storm)
- [实时处理](../scenarios/real-time-processing.md)
