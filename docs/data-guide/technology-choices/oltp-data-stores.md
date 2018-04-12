---
title: 选择 OLTP 数据存储
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 1c27d7d5f3b78f40822de6b77664dbf49b1367f6
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/31/2018
---
# <a name="choosing-an-oltp-data-store-in-azure"></a>在 Azure 中选择 OLTP 数据存储

联机事务处理 (OLTP) 是指对事务数据和事务处理进行管理。 本主题对 Azure 中针对 OLTP 解决方案的选项进行了比较。

> [!NOTE]
> 有关何时使用 OLTP 数据存储的详细信息，请参阅[联机事务处理](../scenarios/online-analytical-processing.md)。

## <a name="what-are-your-options-when-choosing-an-oltp-data-store"></a>在选择 OLTP 数据存储时有哪些选项？

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
| | Azure SQL 数据库 | Azure 虚拟机中的 SQL Server | Azure Database for MySQL | Azure Database for PostgreSQL |
| --- | --- | --- | --- | --- | --- |
| 是托管服务 | 是 | 否 | 是 | 是 |
| 在平台上运行 | 不适用 | Windows、Linux、Docker | 不适用 | 不适用 |
| 可编程性 <sup>1</sup> | T-SQL、.NET、R | T-SQL、.NET、R、Python | T-SQL、.NET、R、Python | SQL | SQL |

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
| | Azure SQL 数据库 | Azure 虚拟机中的 SQL Server| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| 行级安全性 | 是 | 是 | 是 | 是 |
| 数据屏蔽 | 是 | 是 | 否 | 否 |
| 透明数据加密 | 是 | 是 | 是 | 是 |
| 限制访问，仅限特定 IP 地址进行访问 | 是 | 是 | 是 | 是 |
| 限制访问，仅允许访问 VNET | 是 | 是 | 否 | 否 |
| Azure Active Directory 身份验证 | 是 | 是 | 否 | 否 |
| Active Directory 身份验证 | 否 | 是 | 否 | 否 |
| 多重身份验证 | 是 | 是 | 否 | 否 |
| 支持[始终加密](/sql/relational-databases/security/encryption/always-encrypted-database-engine) | 是 | 是 | 是 | 否 | 否 |
| 专用 IP | 否 | 是 | 是 | 否 | 否 |

