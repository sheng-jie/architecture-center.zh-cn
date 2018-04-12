---
title: 联机分析处理 (OLAP)
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: f5ceea9c9dd03812e92fff811e54316edc22b59c
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/31/2018
---
# <a name="online-analytical-processing-olap"></a>联机分析处理 (OLAP)

联机分析处理 (OLAP) 是一种用于组织大型业务数据库的技术，并且支持复杂分析。 它可用来执行复杂的分析查询，且不会对事务系统产生负面影响。

企业用来存储其所有事务和记录的数据库称为[联机事务处理 (OLTP)](online-transaction-processing.md) 数据库。 这些数据库通常具有一次一个输入的记录。 它们通常包含对组织有价值的大量信息。 但是，用于 OLTP 的数据库不是为分析而设计的。 因此，从这些数据库中检索答案从时间和工作量角度而言成本高昂。 OLAP 系统设计用来以高性能方式从数据中提取此商业智能信息。 这是因为 OLAP 数据库针对高频读取和低频写入进行了优化。

![Azure 中的 OLAP](./images/olap-data-pipeline.png) 

## <a name="when-to-use-this-solution"></a>何时使用此解决方案

在下列应用场景中，请考虑使用 OLAP：

- 你需要快速执行复杂的分析和临时查询，且不能对 OLTP 系统产生负面影响。 
- 你希望为业务用户提供一种简单的方式来基于数据生成报表
- 你希望提供大量聚合，这些聚合将使用户能够快速获得一致的结果。 

OLAP 对于针对大量数据应用聚合计算特别有用。 OLAP 系统针对高频读取应用场景（例如分析和商业智能）进行了优化。 OLAP 允许用户将多维数据分割为切片，可以采用二维方式（例如透视表）查看切片，或者按特定值来筛选数据。 此过程有时称为对数据进行“切片和切块”，并且无论是否在多个数据源之间对数据进行了分区，都可以执行此过程。 这可以帮助用户查明趋势、点模式，以及在不需要知道传统数据分析详细信息的情况下探索数据。

[语义模型](../concepts/semantic-modeling.md)可以帮助业务用户抽象关系复杂性，并且更轻松地快速分析数据。

## <a name="challenges"></a>挑战

对于 OLAP 系统提供的所有好处，它们也带来了一些挑战：

- 尽管 OLTP 系统中的数据不断通过从各种源流入的事务进行更新，但是，OLAP 数据存储通常按更长的间隔进行刷新，具体取决于业务需求。 这意味着 OLAP 系统更适用于战略业务决策，而非适用于立即对更改做出响应。 另外，还需要规划一定级别的数据清理和业务流程来使 OLAP 数据存储保持最新。
- 与 OLTP 系统中使用的传统的规范化关系表不同，OLAP 数据模型通常是多维的。 这导致难以或无法直接映射到实体关系或面向对象的模型，在这种模型中，每个属性映射到一个列。 相反，OLAP 系统通常使用星型或雪花型架构，而非使用传统的规范化。

## <a name="olap-in-azure"></a>Azure 中的 OLAP

在 Azure 中，OLTP 系统（例如 Azure SQL 数据库）中存放的数据将复制到 OLAP 系统（例如 [Azure Analysis Services](/azure/analysis-services/analysis-services-overview)）。 数据探索和可视化工具（例如 [Power BI](https://powerbi.microsoft.com)、Excel 和第三方选件）连接到 Analysis Services 服务器，用户可以通过交互性强且视觉效果丰富的方式来了解建模的数据。 从 OLTP 到 OLAP 的数据流通常是使用 SQL Server Integration Services 安排的，可以使用 [Azure 数据工厂](/azure/data-factory/concepts-integration-runtime)执行这些服务。

## <a name="technology-choices"></a>技术选择

- [联机分析处理 (OLAP) 数据存储](../technology-choices/olap-data-stores.md)

