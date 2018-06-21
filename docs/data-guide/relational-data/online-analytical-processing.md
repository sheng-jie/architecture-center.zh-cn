---
title: 联机分析处理 (OLAP)
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 92b71934f2081e95c3c9b0d4dc9edeb3885b12e8
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/06/2018
ms.locfileid: "30846801"
---
# <a name="online-analytical-processing-olap"></a>联机分析处理 (OLAP)

联机分析处理 (OLAP) 是一种用于组织大型业务数据库的技术，并且支持复杂分析。 它可用来执行复杂的分析查询，且不会对事务系统产生负面影响。

企业用来存储其所有事务和记录的数据库称为[联机事务处理 (OLTP)](online-transaction-processing.md) 数据库。 这些数据库通常具有一次一个输入的记录。 它们通常包含对组织有价值的大量信息。 但是，用于 OLTP 的数据库不是为分析而设计的。 因此，从这些数据库中检索答案从时间和工作量角度而言成本高昂。 OLAP 系统设计用来以高性能方式从数据中提取此商业智能信息。 这是因为 OLAP 数据库针对高频读取和低频写入进行了优化。

![Azure 中的 OLAP](../images/olap-data-pipeline.png) 

## <a name="semantic-modeling"></a>语义建模

语义数据模型是一个概念模型，描述了它包含的数据元素的含义。 各个组织经常使用其自己的术语来表示事物，有时使用同义词，甚至同一术语具有不同的含义。 例如，库存数据库可能使用资产 ID 和序列号来跟踪设备部件，但销售数据库可能将序列号称为资产 ID。 如果没有对关系进行描述的模型，则没有简单的方法来将这些值关联起来。 

语义建模基于数据库架构提供了一定级别的抽象，因此，用户不需要知道基础数据结构。 这使得最终用户可以更容易地查询数据，不需要基于基础架构执行聚合和联接。 此外，通常会将列重命名为对用户更友好的名称，使数据的上下文和含义更加明确。

语义建模主要用于高频读取应用场景中，例如分析和商业智能 (OLAP)，与之相对的是高频写入事务数据处理 (OLTP)。 这主要是由典型语义层的性质导致的：

- 设置了聚合行为，以便报告工具可以正确显示它们。
- 定义了业务逻辑和计算。
- 包括了面向时间的计算。
- 数据通常是从多个源集成的。 

传统上，由于这些原因，语义层放置在数据仓库之上。

![位于数据仓库与报告工具之间的语义层的示例关系图](../images/semantic-modeling.png)

有两种主要类型的语义模型：

* **表格**。 使用关系建模构造（模型、表、列）。 在内部，元数据是从 OLAP 建模构造（多维数据集、维、度量值）继承的。 代码和脚本使用 OLAP 元数据。
* **多维**。 使用传统的 OLAP 建模构造（多维数据集、维、度量值）。

相关的 Azure 服务：
- [Azure Analysis Services](https://azure.microsoft.com/services/analysis-services/)

## <a name="example-use-case"></a>示例用例

某个组织将数据存储在大型数据库中。 它希望使此数据可供业务用户和客户用来创建其自己的报表以及执行某些分析。 一种选择是向那些用户授予对数据库的直接访问权限。 但是，这样做有几个缺点，包括安全性管理和访问控制。 此外，数据库的设计（包括表和列的名称）对用户而言可能难以理解。 用户将需要知道要查询哪些表，应当如何联接那些表，还需要知道为获得正确结果而必须应用的其他业务逻辑。 用户甚至还需要了解 SQL 之类的查询语言才能入门。 通常，这将导致多个用户在报告相同的指标时出现不同的结果。

另一种选择是将用户需要的所有信息封装到一个语义模型中。 用户可以使用他们选择的报告工具更轻松地查询语义模型。 语义模型提供的数据是从数据仓库中拉取的，这确保了所有用户看到的是单一版本的真实数据。 语义模型还提供了友好的表名和列名、表之间的关系、说明、计算以及行级安全性。

## <a name="typical-traits-of-semantic-modeling"></a>语义建模的典型特征

语义建模和分析处理通常具有以下特征：

| 要求 | 说明 |
| --- | --- |
| 架构 | 写入时架构，强制实施|
| 使用事务 | 否 |
| 锁定策略 | 无 |
| 可更新 | 否（通常需要重新计算多维数据集） |
| 可追加 | 否（通常需要重新计算多维数据集） |
| 工作负载 | 高频读取，只读的 |
| 索引 | 多维索引 |
| 基准大小 | 中小型 |
| 模型 | 多维 |
| 数据形状：| 多维数据集或星型/雪花型架构 |
| 查询灵活性 | 高度灵活 |
| 规模： | 大型（数十到数百 GB） |

## <a name="when-to-use-this-solution"></a>何时使用此解决方案

在下列应用场景中，请考虑使用 OLAP：

- 你需要快速执行复杂的分析和临时查询，且不能对 OLTP 系统产生负面影响。 
- 你希望为业务用户提供一种简单的方式来基于数据生成报表
- 你希望提供大量聚合，这些聚合将使用户能够快速获得一致的结果。 

OLAP 对于针对大量数据应用聚合计算特别有用。 OLAP 系统针对高频读取应用场景（例如分析和商业智能）进行了优化。 OLAP 允许用户将多维数据分割为切片，可以采用二维方式（例如透视表）查看切片，或者按特定值来筛选数据。 此过程有时称为对数据进行“切片和切块”，并且无论是否在多个数据源之间对数据进行了分区，都可以执行此过程。 这可以帮助用户查明趋势、点模式，以及在不需要知道传统数据分析详细信息的情况下探索数据。

语义模型可以帮助业务用户抽象关系复杂性，并且更轻松地快速分析数据。

## <a name="challenges"></a>挑战

对于 OLAP 系统提供的所有好处，它们也带来了一些挑战：

- 尽管 OLTP 系统中的数据不断通过从各种源流入的事务进行更新，但是，OLAP 数据存储通常按更长的间隔进行刷新，具体取决于业务需求。 这意味着 OLAP 系统更适用于战略业务决策，而非适用于立即对更改做出响应。 另外，还需要规划一定级别的数据清理和业务流程来使 OLAP 数据存储保持最新。
- 与 OLTP 系统中使用的传统的规范化关系表不同，OLAP 数据模型通常是多维的。 这导致难以或无法直接映射到实体关系或面向对象的模型，在这种模型中，每个属性映射到一个列。 相反，OLAP 系统通常使用星型或雪花型架构，而非使用传统的规范化。

## <a name="olap-in-azure"></a>Azure 中的 OLAP

在 Azure 中，OLTP 系统（例如 Azure SQL 数据库）中存放的数据将复制到 OLAP 系统（例如 [Azure Analysis Services](/azure/analysis-services/analysis-services-overview)）。 数据探索和可视化工具（例如 [Power BI](https://powerbi.microsoft.com)、Excel 和第三方选件）连接到 Analysis Services 服务器，用户可以通过交互性强且视觉效果丰富的方式来了解建模的数据。 从 OLTP 到 OLAP 的数据流通常是使用 SQL Server Integration Services 安排的，可以使用 [Azure 数据工厂](/azure/data-factory/concepts-integration-runtime)执行这些服务。

在 Azure 中，以下所有数据存储都将满足 OLAP 的核心要求：

- [具有列存储索引的 SQL Server](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics)
- [Azure Analysis Services](/azure/analysis-services/analysis-services-overview)
- [SQL Server Analysis Services (SSAS)](/sql/analysis-services/analysis-services)

SQL Server Analysis Services (SSAS) 为商业智能应用程序提供 OLAP 和数据挖掘功能。 可以将 SSAS 安装在本地服务器上，也可以将其安装在 Azure 中的虚拟机内的主机上。 Azure Analysis Services 是一项完全托管的服务，提供与 SSAS 相同的主要功能。 Azure Analysis Services 支持连接到云中的[各种数据源](/azure/analysis-services/analysis-services-datasource)和组织中的本地数据源。

聚集列存储索引在 SQL Server 2014 及更高版本中可用，并且最适用于 OLAP 工作负荷。 但是，从 SQL Server 2016（包括 Azure SQL 数据库）开始，可以通过使用可更新的非聚集列存储索引来利用混合事务/分析处理 (HTAP)。 HTAP 使得你可以在同一平台上执行 OLTP 和 OLAP 处理，这样将不需要存储数据的多个副本，并且不需要具有不同的 OLTP 和 OLAP 系统。 有关详细信息，请参阅 [Get started with Columnstore for real-time operational analytics](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics)（用于实时运营分析的列存储入门）。

## <a name="key-selection-criteria"></a>关键选择条件

若要缩小选择范围，请先回答以下问题：

- 你希望使用托管服务还是由你管理自己的服务器？

- 是否要求使用 Azure Active Directory (Azure AD) 进行安全的身份验证？

- 是否要执行实时分析？ 如果是，请将选项范围缩小到支持实时分析的那些选项。 

    此上下文中的“实时分析”适用于将同时运行运营和分析工作负荷的单个数据源，例如企业资源规划 (ERP) 应用程序。 如果需要集成来自多个源的数据，或者需要使用预先聚合的数据（例如多维数据集）来获得极致分析性能，你可能还需要一个单独的数据仓库。

- 是否需要使用预先聚合的数据，例如，提供使分析对业务用户而言更友好的语义模型？ 如果是，请选择一个支持多维数据集或表格语义模型的选项。 

    提供聚合可以帮助用户一致地计算数据聚合。 当处理许多行中的多个列时，预先聚合的数据还可以提供很大的性能提升。 数据可以在多维数据集中预先聚合，也可以在表格语义模型中预先聚合。

- 除了 OLTP 数据存储外，是否需要集成来自其他多个源的数据？ 如果是，请考虑可以轻松集成多个数据源的选项。

## <a name="capability-matrix"></a>功能矩阵

以下各表汇总了功能上的关键差异。

### <a name="general-capabilities"></a>常规功能

| | Azure Analysis Services | SQL Server Analysis Services | 具有列存储索引的 SQL Server | 具有列存储索引的 Azure SQL 数据库 |
| --- | --- | --- | --- | --- |
| 是托管服务 | 是 | 否 | 否 | 是 |
| 支持多维数据集 | 否 | 是 | 否 | 否 |
| 支持表格语义模型 | 是 | 是 | 否 | 否 |
| 轻松集成多个数据源 | 是 | 是 | 否 <sup>1</sup> | 否 <sup>1</sup> |
| 支持实时分析 | 否 | 否 | 是 | 是 |
| 要求具有从多个源复制数据的流程 | 是 | 是 | 否 | 否 |
| Azure AD 集成 | 是 | 否 | 否 <sup>2</sup> | 是 |

[1] 虽然 SQL Server 和 Azure SQL 数据库无法用来从多个外部数据源进行查询以及与之进行集成，但仍然可以构建一个管道并通过它使用 [SSIS](/sql/integration-services/sql-server-integration-services) 或 [Azure 数据工厂](/azure/data-factory/)执行此操作。 Azure VM 中托管的 SQL Server 具有其他选项，例如链接服务器和 [PolyBase](/sql/relational-databases/polybase/polybase-guide)。 有关详细信息，请参阅[管道业务流程、控制流和数据移动](../technology-choices/pipeline-orchestration-data-movement.md)。

[2] 不支持使用 Azure AD 帐户连接到在 Azure 虚拟机上运行的 SQL Server。 请改用域 Active Directory 帐户。

### <a name="scalability-capabilities"></a>可伸缩性功能

|                                                  | Azure Analysis Services | SQL Server Analysis Services | 具有列存储索引的 SQL Server | 具有列存储索引的 Azure SQL 数据库 |
|--------------------------------------------------|-------------------------|------------------------------|-------------------------------------|---------------------------------------------|
| 用于实现高可用性的的冗余区域服务器 |           是           |              否              |                 是                 |                     是                     |
|             支持查询横向扩展             |           是           |              否              |                 是                 |                     否                      |
|          动态可伸缩性（纵向扩展）          |           是           |              否              |                 是                 |                     否                      |

