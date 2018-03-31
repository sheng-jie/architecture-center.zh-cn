---
title: "联机事务处理 (OLTP)"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 07e7f680c8ee5e8589ff7cd2236ff95f6ee84f4c
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/14/2018
---
# <a name="online-transaction-processing-oltp"></a>联机事务处理 (OLTP)

使用计算机系统对[事务数据](../concepts/transactional-data.md)进行管理称为联机事务处理 (OLTP)。 OLTP 系统在组织的日常运营中发生业务交互时对其进行记录，并且支持对该数据进行查询来做出推断。

![Azure 中的 OLTP](./images/oltp-data-pipeline.png)

## <a name="when-to-use-this-solution"></a>何时使用此解决方案

如果需要高效地处理和存储业务事务并立即使其能够以一致的方式供客户端应用程序使用，请选择 OLTP。 当处理过程中任何明显的延迟都会对业务的日常运营造成负面影响时，请使用此体系结构。

OLTP 系统设计用于高效地处理和存储事务，以及查询事务数据。 通过 OLTP 系统高效地处理和存储各个事务有一部分是通过数据规范化完成的 &mdash; 也就是说，将数据拆分为冗余较少的块。 这可以提高效率，因为它使得 OLTP 系统能够独立处理大量的事务，并且避免了在存在冗余数据的情况下维护数据完整性所需执行的额外处理。

## <a name="challenges"></a>挑战
实现和使用 OLTP 系统可能会带来一些挑战：

- OLTP 系统并非非常适合用于处理对大量数据的聚合，但是有一些例外，例如，进行了良好规划的基于 SQL Server 的解决方案。 对数据的分析依赖于对数百个单独事务的聚合计算，对于 OLTP 系统而言，这会消耗大量的资源。 它们可能执行较慢，并且可能会因为阻塞数据库中的其他事务在而导致减速。
- 在对高度规范化的数据进行分析和报告时，查询通常比较复杂，因为大多数查询需要使用联接来使数据非规范化。 此外，在 OLTP 系统中，数据库对象的命名约定通常简洁而精炼。 OLTP 系统中增强的规范化与简洁的命名约定共同使得业务用户在没有 DBA 或数据开发者帮助的情况下难以执行查询。
- 无限期地存储事务历史记录以及在任何一个表中存储太多数据都会导致查询性能变慢，具体取决于所存储的事务数。 常见的解决方案是在 OLTP 系统中维护一个相关时间范围（例如当前会计年度）并将历史数据卸载到其他系统，例如数据市场或[数据仓库](../technology-choices/data-warehouses.md)。

## <a name="oltp-in-azure"></a>Azure 中的 OLTP

[应用服务 Web 应用](/azure/app-service/app-service-web-overview)中托管的应用程序（例如网站）、在应用服务中运行的 REST API、移动或桌面应用程序通常通过 REST API 中介与 OLTP 系统进行通信。

实际上，大多数工作负荷不单纯是 OLTP。 通常还存在[分析组件](../scenarios/online-analytical-processing.md)。 此外，对实时报告的需求也在增加，例如针对运营系统运行报表。 这也称为 HTAP（混合事务性和分析处理）。 有关详细信息，请参阅[联机分析处理 (OLAP) 数据存储](../technology-choices/olap-data-stores.md)。

## <a name="technology-choices"></a>技术选择

数据存储：

- [Azure SQL 数据库](/azure/sql-database/)
- [Azure VM 中的 SQL Server](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json)
- [Azure Database for MySQL](/azure/mysql/)
- [Azure Database for PostgreSQL](/azure/postgresql/)

有关详细信息，请参阅[选择 OLTP 数据存储](../technology-choices/oltp-data-stores.md)

数据源：

- [应用服务](/azure/app-service/)
- [移动应用](/azure/app-service-mobile/)

