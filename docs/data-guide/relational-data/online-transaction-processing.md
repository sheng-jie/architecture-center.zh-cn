---
title: 联机事务处理 (OLTP)
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 8650b919fc1a59240343015493a1fe41c8729a72
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/06/2018
---
# <a name="online-transaction-processing-oltp"></a>联机事务处理 (OLTP)

使用计算机系统对事务数据进行管理称为联机事务处理 (OLTP)。 OLTP 系统在组织的日常运营中发生业务交互时对其进行记录，并且支持对该数据进行查询来做出推断。

## <a name="transactional-data"></a>事务数据

事务数据是指一种信息，用于跟踪与组织活动相关的交互。 这些交互通常是业务事务，如从客户收到的付款、对供应商进行的付款、从库存移动的产品、接受的订单或交付的服务。 表示事务本身的事务事件通常包含时间维度、一些数值和对其他数据的引用。 

事务通常需要*原子性*和*一致性*。 原子性意味着整个事务始终作为一个工作单元成功或失败，永远不会处于半完成状态。 如果无法完成某个事务，数据库系统必须回退任何已作为该事务的一部分完成的步骤。 在传统 RDBMS 中，如果某个事务无法完成，这种回退会自动发生。 一致性意味着事务始终让数据处于有效状态。 （这些是原子性和一致性的非常不正式的说明。 这些属性有更正式的定义，如 [ACID](https://en.wikipedia.org/wiki/ACID)。）

事务数据库可以使用各种锁定策略（如悲观锁定）支持事务的强一致性，以确保所有用户和进程的所有数据在企业的上下文中具有强一致性。 

使用事务数据的最常见部署体系结构是 3 层体系结构中的数据存储层。 3 层体系结构通常包含表示层、业务逻辑层和数据存储层。 相关的部署体系结构是 [N 层](/azure/architecture/guide/architecture-styles/n-tier)体系结构，它可能具有多个处理业务逻辑的中间层。

## <a name="typical-traits-of-transactional-data"></a>事务数据的典型特征

事务数据往往具有以下特征：

| 要求 | 说明 |
| --- | --- |
| 规范化 | 高度规范化 |
| 架构 | 写入时架构，强制实施|
| 一致性 | 强一致性，保证 ACID |
| 完整性 | 高完整性 |
| 使用事务 | 是 |
| 锁定策略 | 悲观或乐观|
| 可更新 | 是 |
| 可追加 | 是 |
| 工作负载 | 大量写入，适度读取 |
| 索引 | 主要和辅助索引 |
| 基准大小 | 中小型 |
| 模型 | 关系 |
| 数据形状 | 表格 |
| 查询灵活性 | 高度灵活 |
| 缩放 | 小型 (MB) 到大型（几 TB） | 

## <a name="when-to-use-this-solution"></a>何时使用此解决方案

如果需要高效地处理和存储业务事务并立即使其能够以一致的方式供客户端应用程序使用，请选择 OLTP。 当处理过程中任何明显的延迟都会对业务的日常运营造成负面影响时，请使用此体系结构。

OLTP 系统设计用于高效地处理和存储事务，以及查询事务数据。 通过 OLTP 系统高效地处理和存储各个事务有一部分是通过数据规范化完成的 &mdash; 也就是说，将数据拆分为冗余较少的块。 这可以提高效率，因为它使得 OLTP 系统能够独立处理大量的事务，并且避免了在存在冗余数据的情况下维护数据完整性所需执行的额外处理。

## <a name="challenges"></a>挑战
实现和使用 OLTP 系统可能会带来一些挑战：

- OLTP 系统并非非常适合用于处理对大量数据的聚合，但是有一些例外，例如，进行了良好规划的基于 SQL Server 的解决方案。 对数据的分析依赖于对数百个单独事务的聚合计算，对于 OLTP 系统而言，这会消耗大量的资源。 它们可能执行较慢，并且可能会因为阻塞数据库中的其他事务在而导致减速。
- 在对高度规范化的数据进行分析和报告时，查询通常比较复杂，因为大多数查询需要使用联接来使数据非规范化。 此外，在 OLTP 系统中，数据库对象的命名约定通常简洁而精炼。 OLTP 系统中增强的规范化与简洁的命名约定共同使得业务用户在没有 DBA 或数据开发者帮助的情况下难以执行查询。
- 无限期地存储事务历史记录以及在任何一个表中存储太多数据都会导致查询性能变慢，具体取决于所存储的事务数。 常见的解决方案是在 OLTP 系统中维护一个相关时间范围（例如当前会计年度）并将历史数据卸载到其他系统，例如数据市场或[数据仓库](./data-warehousing.md)。

## <a name="oltp-in-azure"></a>Azure 中的 OLTP

[应用服务 Web 应用](/azure/app-service/app-service-web-overview)中托管的应用程序（例如网站）、在应用服务中运行的 REST API、移动或桌面应用程序通常通过 REST API 中介与 OLTP 系统进行通信。

实际上，大多数工作负荷不单纯是 OLTP。 通常还存在分析组件。 此外，对实时报告的需求也在增加，例如针对运营系统运行报表。 这也称为 HTAP（混合事务性和分析处理）。 有关详细信息，请参阅[联机分析处理 (OLAP)](./online-analytical-processing.md)。

在 Azure 中，以下所有数据存储都将满足 OLTP 和事务数据处理的核心要求：

- [Azure SQL 数据库](/azure/sql-database/)
- [Azure 虚拟机中的 SQL Server](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json)
- [Azure Database for MySQL](/azure/mysql/)
- [Azure Database for PostgreSQL](/azure/postgresql/)

## <a name="key-selection-criteria"></a>关键选择条件

若要缩小选择范围，请先回答以下问题：

- 你希望使用托管服务还是由你管理自己的服务器？

- 解决方案是否对 Microsoft SQL Server、MySQL 或 PostgreSQL 兼容性具有特定的依赖关系？ 你的应用程序可能会根据以下因素限制可以选择的数据存储：它支持用于与数据存储进行通信的驱动程序，或者它针对使用哪个数据库所做的假设。

- 对写入吞吐量的要求是否特别高？ 如果是，请选择一个提供内存中表的选项。 

- 解决方案是否为多租户的？ 如果是，请考虑使用支持容量池的选项，在容量池中，多个数据库实例从弹性资源池拉取资源，而非每个数据库使用固定资源。 这可以帮助你更好地将容量分布到所有数据库实例中，并且可以使解决方案更经济高效。

- 是否需要在多个区域中以低延迟读取数据？ 如果是，请选择一个支持可读取的次要副本的选项。

- 数据库是否需要跨地理区域高度可用？ 如果是，请选择一个支持异地复制的选项。 另外，请考虑使用支持从主要副本自动故障转移到次要副本的选项。

- 数据库是否有特定的安全需求？ 如果是，请考虑使用提供了诸如行级安全性、数据掩码和透明数据加密等功能的选项。

## <a name="capability-matrix"></a>功能矩阵

以下各表汇总了功能上的关键差异。

### <a name="general-capabilities"></a>常规功能 

|                              | Azure SQL 数据库 | Azure 虚拟机中的 SQL Server | Azure Database for MySQL | Azure Database for PostgreSQL |
|------------------------------|--------------------|----------------------------------------|--------------------------|-------------------------------|
|      是托管服务      |        是         |                   否                   |           是            |              是              |
|       在平台上运行       |        不适用         |         Windows、Linux、Docker         |           不适用            |              不适用              |
| 可编程性 <sup>1</sup> |   T-SQL、.NET、R   |         T-SQL、.NET、R、Python         |  T-SQL、.NET、R、Python  |              SQL              |

[1] 不包括客户端驱动程序支持，该支持允许通过许多编程语言连接到并使用 OLTP 数据存储。

### <a name="scalability-capabilities"></a>可伸缩性功能

| | Azure SQL 数据库 | Azure 虚拟机中的 SQL Server| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- |
| 最大数据库实例大小 | [4 TB](/azure/sql-database/sql-database-resource-limits) | 256 TB | [1 TB](/azure/mysql/concepts-limits) | [1 TB](/azure/postgresql/concepts-limits) |
| 支持容量池  | 是 | 是 | 否 | 否 |
| 支持群集横向扩展  | 否 | 是 | 否 | 否 |
| 动态可伸缩性（纵向扩展）  | 是 | 否 | 是 | 是 |

### <a name="analytic-workload-capabilities"></a>分析工作负荷功能

| | Azure SQL 数据库 | Azure 虚拟机中的 SQL Server| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| 临时表 | 是 | 是 | 否 | 否 |
| 内存中（内存优化的）表 | 是 | 是 | 否 | 否 |
| 列存储支持 | 是 | 是 | 否 | 否 |
| 自适应查询处理 | 是 | 是 | 否 | 否 |

### <a name="availability-capabilities"></a>可用性功能

| | Azure SQL 数据库 | Azure 虚拟机中的 SQL Server| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| 可读次要副本 | 是 | 是 | 否 | 否 | 
| 异地复制 | 是 | 是 | 否 | 否 | 
| 自动故障转移到次要副本 | 是 | 否 | 否 | 否|
| 时间点还原 | 是 | 是 | 是 | 是 |

### <a name="security-capabilities"></a>安全功能

|                                                                                                             | Azure SQL 数据库 | Azure 虚拟机中的 SQL Server | Azure Database for MySQL | Azure Database for PostgreSQL |
|-------------------------------------------------------------------------------------------------------------|--------------------|----------------------------------------|--------------------------|-------------------------------|
|                                             行级安全性                                              |        是         |                  是                   |           是            |              是              |
|                                                数据屏蔽                                                 |        是         |                  是                   |            否            |              否               |
|                                         透明数据加密                                         |        是         |                  是                   |           是            |              是              |
|                                  限制访问，仅限特定 IP 地址进行访问                                   |        是         |                  是                   |           是            |              是              |
|                                  限制访问，仅允许访问 VNET                                  |        是         |                  是                   |            否            |              否               |
|                                    Azure Active Directory 身份验证                                    |        是         |                  是                   |            否            |              否               |
|                                       Active Directory 身份验证                                       |         否         |                  是                   |            否            |              否               |
|                                         多重身份验证                                         |        是         |                  是                   |            否            |              否               |
| 支持[始终加密](/sql/relational-databases/security/encryption/always-encrypted-database-engine) |        是         |                  是                   |           是            |              否               |
|                                                 专用 IP                                                  |         否         |                  是                   |           是            |              否               |

