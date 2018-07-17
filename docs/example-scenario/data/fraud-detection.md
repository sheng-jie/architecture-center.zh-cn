---
title: 高级分析&mdash;实时欺诈检测
description: 经验证的解决方案，可以使用 Azure 事件中心和流分析实时检测欺诈性活动。
author: alexbuckgit
ms.date: 07/05/2018
ms.openlocfilehash: cf375445b38b0ff7d6fbc400902d5e97b34b4fed
ms.sourcegitcommit: 5d99b195388b7cabba383c49a81390ac48f86e8a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/06/2018
ms.locfileid: "37891267"
---
# <a name="real-time-fraud-detection-on-azure"></a>在 Azure 上实时欺诈检测

本示例方案针对那些需要实时分析数据以检测欺诈性事务或其他异常活动的组织。

可能的应用包括：确定欺诈性信用卡活动或移动电话呼叫。 为了确定异常活动，传统的联机分析系统可能需要数小时来转换并分析数据。

使用完全托管的 Azure 服务（例如事件中心和流分析），公司不需管理各个服务器，既减少了成本，又可充分利用 Microsoft 在云规模的数据引入和实时分析方面的专业技术。 本方案专门解决欺诈性活动的检测问题。 如果有数据分析的其他需求，则应查看可用 [Azure 分析服务][product-category]的列表。

本示例所代表的部分涉及到更广泛的数据处理体系结构和策略。 整个体系结构此方面的其他选项将在本文后面讨论。
 
## <a name="potential-use-cases"></a>可能的用例

以下用例可以考虑本解决方案：

* 在电信场景中检测欺诈性移动电话呼叫。
* 为银行机构确定欺诈性信用卡交易。
* 在零售或电子商务场景中确定欺诈性购物。

## <a name="architecture"></a>体系结构

![从体系结构的角度概要说明实时欺诈检测解决方案的 Azure 组件][architecture-diagram]

本解决方案涉及实时分析管道的后端组件。 数据流经解决方案的情形如下所示：

1. 将移动电话呼叫元数据从源系统发送到 Azure 事件中心实例。 
2. 启动流分析作业，通过事件中心源接收数据。
3. 流分析作业运行转换输入流所需的预定义查询，并根据欺诈事务算法对其进行分析。 该查询使用翻转窗口将流分成不同的时间单元。
4. 流分析作业将已转换的流（代表检测到的欺诈性呼叫）写入到 Azure Blob 存储中的输出接收器。

### <a name="components"></a>组件

* [Azure 事件中心][docs-event-hubs]是一个实时流式处理平台和事件引入服务，每秒能够接收和处理数百万个事件。 事件中心可以处理和存储分布式软件和设备生成的事件、数据或遥测。 在本解决方案中，事件中心接收需要进行欺诈活动分析的所有电话呼叫元数据。
* [Azure 流分析][docs-stream-analytics]是一个事件处理引擎，可以分析从设备和其他数据源流式传输的大量数据。 它还支持从数据流提取信息，以便确定模式和关系。 这些模式可能触发其他下游操作。 在本解决方案中，流分析会转换事件中心的输入流，以便确定欺诈性呼叫。
* [Blob 存储][docs-blob-storage]在本解决方案中用于存储流分析作业的结果。

## <a name="considerations"></a>注意事项

### <a name="alternatives"></a>备选项

进行实时消息引入、数据存储、流处理、分析数据存储以及分析和报告时，有许多技术选择。 有关这些选项及其功能和主要选择标准的概述，请参阅 Azure 数据体系结构指南中的[大数据体系结构：实时处理](/azure/architecture/data-guide/technology-choices/real-time-ingestion)。

另外，也可通过 Azure 中的各种机器学习服务生成更复杂的欺诈检测算法。 有关这些选项的概述，请参阅 [Azure 数据体系结构指南](../../data-guide/index.md)中的[机器学习的技术选择](/azure/architecture/data-guide/technology-choices/data-science-and-machine-learning)。

### <a name="availability"></a>可用性

Azure Monitor 提供了统一的用户界面，可用于监视各种 Azure 服务。 有关详细信息，请参阅[在 Microsoft Azure 中进行监视](/azure/monitoring-and-diagnostics/monitoring-overview)。 事件中心和流分析均与 Azure Monitor 集成在一起。 

若要了解其他可用性注意事项，请参阅 Azure 体系结构中心的[可用性核对清单][availability]。

### <a name="scalability"></a>可伸缩性

本解决方案的组件设计用于超大规模的引入和大规模并行实时分析。 Azure 事件中心高度可缩放，每秒能够接收和处理数百万个事件且延迟很低。  事件中心可[自动增加](/azure/event-hubs/event-hubs-auto-inflate)吞吐量单元数，以便满足使用需求。 Azure 流分析可以分析多个源提供的大量流数据。 若要纵向扩展流分析，可以增加分配的用于执行流作业的[流单元](/azure/stream-analytics/stream-analytics-streaming-unit-consumption)数。

有关如何设计可缩放解决方案的通用指南，请参阅 Azure 体系结构中心的[可伸缩性核对清单][scalability]。

### <a name="security"></a>“安全”

Azure 事件中心通过[身份验证和安全模型][docs-event-hubs-security-model]来确保数据安全，该模型基于共享访问签名 (SAS) 令牌与事件发布者的组合。 事件发布者定义事件中心的虚拟终结点。 发布者只能用于将消息发送到事件中心。 无法从发布者接收消息。

若需安全解决方案的通用设计指南，请参阅 [Azure 安全性文档][security]。

### <a name="resiliency"></a>复原

若需可复原解决方案的通用设计指南，请参阅[设计适用于 Azure 的可复原应用程序][resiliency]。

## <a name="deploy-the-solution"></a>部署解决方案

若要部署本解决方案，可以按照此[分步教程][tutorial]的说明操作，手动部署解决方案的每个组件。 本教程还提供了一个 .NET 客户端应用程序，用于生成示例电话呼叫元数据并将该数据发送到事件中心实例。 

## <a name="pricing"></a>定价

为了方便用户了解运行本解决方案的成本，我们已在成本计算器中预配置了所有服务。 若要了解自己的特定用例的定价变化情况，请按预期的数据量更改相应的变量。

我们已根据你预期接收的流量提供了三个示例性的成本配置文件：

* [小][small-pricing]：每月通过一个标准的流单元处理一百万个事件。
* [中][medium-pricing]：每月通过五个标准的流单元处理一亿个事件。
* [大][large-pricing]：每月通过 20 个标准的流单元处理 9.99 亿个事件。

## <a name="related-resources"></a>相关资源

可以通过机器学习模型生成更复杂的欺诈检测方案。 若要了解使用 Machine Learning Server 生成的解决方案，请参阅[使用 Machine Learning Server 进行的欺诈检测][r-server-fraud-detection]。 若要了解使用 Machine Learning Server 的其他解决方案模板，请参阅[数据科学方案和解决方案模板][docs-r-server-sample-solutions]。 若要了解使用 Azure Data Lake Analytics 的示例解决方案，请参阅[使用 Azure Data Lake 和 R 进行欺诈检测][technet-fraud-detection]。  

<!-- links -->
[product-category]: https://azure.microsoft.com/product-categories/analytics/
[tutorial]: /azure/stream-analytics/stream-analytics-real-time-fraud-detection
[small-pricing]: https://azure.com/e/74149ec312c049ccba79bfb3cfa67606
[medium-pricing]: https://azure.com/e/4fc94f7376de484d8ae67a6958cae60a
[large-pricing]: https://azure.com/e/7da8804396f9428a984578700003ba42
[architecture-diagram]: ./images/architecture-diagram-fraud-detection.png
[docs-event-hubs]: /azure/event-hubs/event-hubs-what-is-event-hubs
[docs-event-hubs-security-model]: /azure/event-hubs/event-hubs-authentication-and-security-model-overview
[docs-stream-analytics]: /azure/stream-analytics/stream-analytics-introduction
[docs-blob-storage]: /azure/storage/blobs/storage-blobs-introduction
[docs-r-server-sample-solutions]: /machine-learning-server/r/sample-solutions
[r-server-fraud-detection]: https://microsoft.github.io/r-server-fraud-detection/
[technet-fraud-detection]: https://blogs.technet.microsoft.com/machinelearning/2017/06/28/using-azure-data-lake-and-r-for-fraud-detection/
[availability]: /azure/architecture/checklist/availability
[scalability]: /azure/architecture/checklist/scalability
[resiliency]: ../../resiliency/index.md
[security]: /azure/security/

