---
title: "Azure 数据体系结构指南"
description: 
author: zoinerTejada
ms:date: 02/12/2018
layout: LandingPage
ms.openlocfilehash: 848601f27faf56ea069852d8983e4d10fbad9d77
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/14/2018
---
# <a name="azure-data-architecture-guide"></a>Azure 数据体系结构指南

本指南提供了在 Microsoft Azure 上设计以数据为中心的解决方案的结构化方法。 该方法基于我们从客户互动中获得的经过验证的做法。

## <a name="introduction"></a>介绍

云正在改变应用程序的设计方式，包括如何处理和存储数据。 “polyglot 持久性”解决方案不是用于处理解决方案的所有数据的单一通用数据库，它们使用多个专用的数据存储，每个都进行了优化来提供特定功能。 因此，解决方案中的数据透视也发生了变化。 不再有多个在单一数据层中进行读取和写入的业务逻辑层。 相反，解决方案围绕“数据管道”而设计，数据管道描述了数据如何流经解决方案，在哪里处理数据，在哪里存储数据，以及管道中的下一组件如何使用数据。 

## <a name="how-this-guide-is-structured"></a>本指南的结构

本指南的结构基于一个基本透视：*关系*数据与*非关系*数据之间的不同。 

![](./images/guide-steps.svg)

关系数据通常存储在传统 RDBMS 或数据仓库中。 它有一个预定义的架构（“写入时架构”），其中包含一组用于维护引用完整性的约束。 大多数关系数据库使用结构化查询语言 (SQL) 进行查询。 使用关系数据库的解决方案包括联机事务处理 (OLTP) 和联机分析处理 (OLAP)。

非关系数据是不使用传统 RDBMS 系统中存在的[关系模型](https://en.wikipedia.org/wiki/Relational_model)的任何数据。 这可能包括键-值数据、JSON 数据、图形数据、时序数据以及其他数据类型。 术语 *NoSQL* 指的是设计用来存放各种类型的非关系数据的数据库。 不过，此术语不是完全正确，因为许多非关系数据存储支持 SQL 兼容查询。 在讨论“大数据”解决方案时经常会提到非关系数据和 NoSQL 数据库。 大数据体系结构设计用来处理对传统数据库系统而言太大或太复杂的数据的引入、处理和分析。 

在这两个主要类别中，数据体系结构指南都包含以下各部分：

- **概念。** 概述文章，其中介绍了在使用此类型的数据时需要理解的主要概念。
- **方案。** 具有代表性的一组数据方案，并且讨论了方案的相关 Azure 服务和合适的体系结构。
- **技术选择。** 详细比较了 Azure 中可用的各种数据技术，包括开源选项。 在每个类别中，我们都介绍了关键选择条件和功能矩阵，以帮助你选择适合你的方案的合适技术。

本指南的目的不是讲授数据科学或数据库理论 &mdash; 你可以找到有关这些主题的完整丛书。 相反，其目的是帮助你选择适用于你的方案的合适数据体系结构或数据管道，然后选择最能满足要求的 Azure 服务和技术。 如果你的脑海中已有了体系结构，则可以直接跳到技术选择。

## <a name="traditional-rdbms"></a>传统 RDBMS

### <a name="concepts"></a>概念

- [关系数据](./concepts/relational-data.md) 
- [事务数据](./concepts/transactional-data.md) 
- [语义建模](./concepts/semantic-modeling.md) 

### <a name="scenarios"></a>方案

- [联机分析处理 (OLAP)](./scenarios/online-analytical-processing.md)
- [联机事务处理 (OLTP)](./scenarios/online-transaction-processing.md) 
- [数据仓库和数据市场](./scenarios/data-warehousing.md)
- [ETL](./scenarios/etl.md) 

## <a name="big-data-and-nosql"></a>大数据和 NoSQL

### <a name="concepts"></a>概念

- [非关系数据存储](./concepts/non-relational-data.md)
- [使用 CSV 和 JSON 文件](./concepts/csv-and-json.md)
- [大数据体系结构](./concepts/big-data.md)
- [高级分析](./concepts/advanced-analytics.md) 
- [规模化机器学习](./concepts/machine-learning-at-scale.md)

### <a name="scenarios"></a>方案

- [批处理](./scenarios/batch-processing.md)
- [实时处理](./scenarios/real-time-processing.md)
- [自由格式文本搜索](./scenarios/search.md)
- [交互式数据探索](./scenarios/interactive-data-exploration.md)
- [自然语言处理](./scenarios/natural-language-processing.md)
- [时序解决方案](./scenarios/time-series.md)

## <a name="cross-cutting-concerns"></a>横切关注点

- [数据传输](./scenarios/data-transfer.md) 
- [将本地数据解决方案扩展到云](./scenarios/hybrid-on-premises-and-cloud.md) 
- [保护数据解决方案](./scenarios/securing-data-solutions.md) 
