---
title: "选择 OLAP 数据存储"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: f3041b95696c9408a2c9ab747fe1ec3041db0743
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-an-olap-data-store-in-azure"></a>在 Azure 中选择 OLAP 数据存储

联机分析处理 (OLAP) 是一种用于组织大型业务数据库的技术，并且支持复杂分析。 本主题对 Azure 中针对 OLAP 解决方案的选项进行了比较。

> [!NOTE]
> 有关何时使用 OLAP 数据存储的详细信息，请参阅[联机分析处理](../scenarios/online-analytical-processing.md)。

## <a name="what-are-your-options-when-choosing-an-olap-data-store"></a>选择 OLAP 数据存储时有哪些选项？

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

| | Azure Analysis Services | SQL Server Analysis Services | 具有列存储索引的 SQL Server | 具有列存储索引的 Azure SQL 数据库 |
| --- | --- | --- | --- | --- |
| 用于实现高可用性的的冗余区域服务器  | 是 | 否 | 是 | 是 |
| 支持查询横向扩展  | 是 | 否 | 是 | 否 |
| 动态可伸缩性（纵向扩展）  | 是 | 否 | 是 | 否 |

