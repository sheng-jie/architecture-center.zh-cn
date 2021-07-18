---
title: Azure 服务的复原能力检查表
description: 一个检查表，提供各种 Azure 服务的复原能力指南。
author: petertaylor9999
ms.date: 03/02/2018
ms.custom: resiliency, checklist
ms.openlocfilehash: 25d961d6bb753b1f515fc073e51bbb912cc59db7
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/08/2018
ms.locfileid: "29783506"
---
# <a name="resiliency-checklist-for-specific-azure-services"></a>特定 Azure 服务的复原能力检查表

复原能力是指系统能够在发生故障后进行恢复，然后继续正常运行，它是[软件质量的构成要素](../guide/pillars.md)之一。 每项技术都有自己的特定故障模式，这是在设计和实现应用程序时必须考虑的。 请使用此检查表查看特定 Azure 服务的复原能力注意事项。 另请查看[常规复原能力检查表](./resiliency.md)。

## <a name="app-service"></a>应用服务

**使用“标准”或“高级”层。** 这些层支持过渡槽位和自动备份。 有关详细信息，请参阅 [Azure 应用服务计划深入概述](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/)

**避免纵向扩展或缩减。** 应选择满足典型负载下的性能要求的层和实例大小，然后[横向扩展](/azure/app-service-web/web-sites-scale/)实例来处理流量的更改。 纵向扩展或缩减可能触发应用程序重启。  

**将配置存储为应用设置。** 使用应用设置将配置设置保存为应用设置。 在资源管理器模板中或使用 PowerShell 定义设置，以便可以在更可靠的自动部署/更新过程中应用这些设置。 有关详细信息，请参阅[在 Azure 应用服务中配置 Web 应用](/azure/app-service-web/web-sites-configure/)。

**为生产和测试创建单独的应用服务计划。** 不要使用生产部署中的槽位进行测试。  同一应用服务计划中的所有应用共享相同的 VM 实例。 如果将生产和测试部署放在同一个计划中，可能会对生产部署产生负面影响。 例如，负载测试可能使实时生产站点降级。 如果将测试部署放入单独的计划，则可将其与生产版本相隔离。  

**将 Web 应用与 Web API 相区分。** 如果解决方案包含 Web 前端和 Web API，请考虑将它们分解为单独的应用服务应用。 使用这种设计可以更方便地按工作负荷分解解决方案。 可以在单独的应用服务计划中运行 Web 应用和 API，以便对它们进行单独缩放。 如果一开始不需要该级别的可伸缩性，可以先将应用部署到同一计划中，以后再根据需要将其移至独立的计划中。

**避免使用应用服务备份功能来备份 Azure SQL 数据库。** 应该使用 [SQL 数据库自动备份][sql-backup]。 应用服务备份会将数据库导出到 SQL .bacpac 文件，这会消耗 DTU。  

**部署到过渡槽位。** 创建部署槽位用于过渡。 将应用程序更新部署到过渡槽位，并在交换到生产环境之前验证部署。 这可以减少生产环境中出现更新不当的可能性。 此外，可确保所有实例在交换到生产环境之前已预热。 许多应用程序的预热和冷启动时间很长。 有关详细信息，请参阅[为 Azure 应用服务中的 Web 应用设置过渡环境](/azure/app-service-web/web-sites-staged-publishing/)。

**创建一个部署槽位用于保存上次已知正常 (LKG) 的部署。** 将更新部署到生产环境时，请将前一个生产部署移到 LKG 槽位中。 这可以更方便地回滚错误部署。 如果以后发现问题，可以快速还原到 LKG 版本。 有关详细信息，请参阅[基本 Web 应用程序](../reference-architectures/app-service-web-app/basic-web-app.md)。

**启用诊断日志记录**，包括应用程序日志记录和 Web 服务器日志记录。 日志记录对于监视和诊断很重要。 请参阅[在 Azure 应用服务中为 Web 应用启用诊断日志记录](/azure/app-service-web/web-sites-enable-diagnostic-log/)

**记录 Blob 存储的日志。** 这可以更方便地收集和分析数据。

**为日志创建单独的存储帐户。** 不要对日志和应用程序数据使用相同的存储帐户。 这有助于防止日志记录降低应用程序的性能。

**监视性能。** 使用 [New Relic](http://newrelic.com/) 或 [Application Insights](/azure/application-insights/app-insights-overview/) 等性能监视服务来监视承受负载时的应用程序性能和行为。  性能监视提供应用程序的实时深入信息。 可以使用它来诊断问题，以及执行故障根本原因分析。

## <a name="application-gateway"></a>应用程序网关

**至少预配两个实例。** 部署至少包含两个实例的应用程序网关。 单个实例会成为单一故障点。 使用两个或更多个实例实现冗余和可伸缩性。 若要满足 [SLA](https://azure.microsoft.com/support/legal/sla/application-gateway/v1_0/)，必须预配两个或更多个中型或大型实例。

## <a name="cosmos-db"></a>Cosmos DB

**跨区域复制数据库。** Cosmos DB 允许将任意数量的 Azure 区域与 Cosmos DB 数据库帐户关联。 Cosmos DB 数据库可以包含一个写入区域和多个读取区域。 如果写入区域发生故障，可以从另一个副本读取。 客户端 SDK 会自动处理此操作。 还可以将写入区域故障转移到另一个区域。 有关详细信息，请参阅[如何使用 Azure Cosmos DB 进行全局数据分配](/azure/cosmos-db/distribute-data-globally)。

## <a name="event-hubs"></a>事件中心

**使用检查点**。  事件使用者应该根据某种预定义的间隔将其当前位置写入持久性存储。 这样，如果使用者遇到故障（例如，使用者崩溃，或主机故障），则新实例可以继续从上一个记录的位置读取流。 有关详细信息，请参阅[事件使用者](/azure/event-hubs/event-hubs-features#event-consumers)。

**处理重复消息。** 如果某个事件使用者故障，则会从上一个记录的检查点恢复消息处理。 在上一个检查点之后已经处理的任何消息都会重新处理。 因此，消息处理逻辑必须是幂等的，否则应用程序必须能够删除重复消息。

**处理异常。** 事件使用者通常以循环的方式处理一批消息。 应该在该处理循环内处理异常，避免在单个消息导致异常的情况下丢失一整批消息。

**使用死信队列。** 如果处理消息时出现非暂时性故障，请将该消息置于死信队列中，以便跟踪状态。 可以根据情况在以后重试消息、应用补偿事务，或者执行其他操作。 请注意，事件中心没有任何内置的死信队列功能。 可以使用 Azure 队列存储或服务总线来实现死信队列，也可以使用 Azure Functions 或某个其他的事件处理机制。  

**通过故障转移到辅助的事件中心命名空间实施灾难恢复。** 有关详细信息，请参阅 [Azure 事件中心异地灾难恢复](/azure/event-hubs/event-hubs-geo-dr)。

## <a name="redis-cache"></a>Redis 缓存

**配置异地复制**。 异地复制提供一种用于链接两个高级层 Azure Redis 缓存实例的机制。 写入主缓存的数据将复制到辅助只读缓存。 有关详细信息，请参阅[如何为 Azure Redis 缓存配置异地复制功能](/azure/redis-cache/cache-how-to-geo-replication)

**配置数据持久性。** 使用 Redis 持久性可以在 Redis 中持久存储数据。 还可以获取快照并备份数据，以便在出现硬件故障时进行加载。 有关详细信息，请参阅[如何为高级 Azure Redis 缓存配置数据持久性](/azure/redis-cache/cache-how-to-premium-persistence)

如果将 Redis 缓存使用临时数据缓存而不是持久性存储，则这些建议可能不适用。 

## <a name="search"></a>搜索

**预配多个副本。** 至少使用两个副本实现读取高可用性，或至少使用三个副本实现读写高可用性。

**为多区域部署配置索引器。** 若要进行多区域部署，请考虑使用索引连续性选项。

  * 如果数据源是异地复制的，通常应该将每个区域性 Azure 搜索服务的每个索引器指向其本地数据源副本。 但是，对于存储在 Azure SQL 数据库的大型数据集，不建议使用此方法。 原因在于，Azure 搜索无法从辅助 SQL 数据库副本执行增量索引，而只能从主副本执行。 应该将所有索引器指向主副本。 故障转移后，请将 Azure 搜索索引器指向新的主副本。  
  * 如果数据源不是异地复制的，请将多个索引器指向同一个数据源，以便多个区域中的 Azure 搜索服务可从数据源连续独立地编制索引。 有关详细信息，请参阅 [Azure 搜索性能和优化注意事项][search-optimization]。

## <a name="storage"></a>存储

**对于应用程序数据，请使用读取访问异地冗余存储 (RA-GRS)。** RA-GRS 存储将数据复制到次要区域，并提供从次要区域的只读访问权限。 如果主要区域中发生存储故障，应用程序可以从次要区域读取数据。 有关详细信息，请参阅 [Azure 存储复制](/azure/storage/storage-redundancy/)。

**对于 VM 磁盘，请使用托管磁盘。** [托管磁盘][managed-disks]为可用性集中的 VM 提供更高的可靠性，因为这些磁盘彼此充分隔离，可避免单一故障点。 此外，托管磁盘不受存储帐户中创建的 VHD 的 IOPS 限制。 有关详细信息，请参阅[在 Azure 中管理 Windows 虚拟机的可用性][vm-manage-availability]。

**对于队列存储，请在另一个区域中创建备份队列。** 对于队列存储，只读副本的作用有限，因为无法将项排队或取消排队。 应在另一个区域中的某个存储帐户中创建备份队列。 如果发生存储故障，应用程序可以使用备份队列，直到主要区域重新可用。 这样，应用程序仍可处理新请求。  

## <a name="sql-database"></a>SQL 数据库

**使用“标准”或“高级”层。** 这些层提供更长的时间点还原期限（35 天）。 有关详细信息，请参阅 [SQL 数据库选项和性能](/azure/sql-database/sql-database-service-tiers/)。

**启用 SQL 数据库审核。** 审核可用于诊断恶意攻击或人为错误。 有关详细信息，请参阅 [SQL 数据库审核入门](/azure/sql-database/sql-database-auditing-get-started/)。

**使用活动异地复制**使用活动异地复制在一个不同的区域中创建可读取的辅助副本。  如果主数据库出现故障，或者只是需要脱机，可以手动故障转移到任何辅助数据库。  在故障转移之前，辅助数据库将保持只读状态。  有关详细信息，请参阅 [SQL 数据库活动异地复制](/azure/sql-database/sql-database-geo-replication-overview/)。

**使用分片。** 考虑使用分片将数据库横向分区。 分片可提供故障隔离。 有关详细信息，请参阅[使用 Azure SQL 数据库进行横向扩展](/azure/sql-database/sql-database-elastic-scale-introduction/)。

**发生人为错误后使用时间点还原进行恢复。**  时间点还原可将数据库还原到以前的某个时间点。 有关详细信息，请参阅[使用自动数据库备份恢复 Azure SQL 数据库][sql-restore]。

**发生服务中断后使用异地还原进行恢复。** 异地还原可从异地冗余的备份还原数据库。  有关详细信息，请参阅[使用自动数据库备份恢复 Azure SQL 数据库][sql-restore]。

## <a name="sql-server-running-in-a-vm"></a>VM 中运行的 SQL Server

**复制数据库。** 使用 SQL Server Always On 可用性组复制数据库。 当某个 SQL Server 实例发生故障时可提供高可用性。 有关详细信息，请参阅[对 N 层应用程序运行 Windows VM](../reference-architectures/virtual-machines-windows/n-tier.md)

**备份数据库**。 如果已在使用 [Azure 备份](https://azure.microsoft.com/documentation/services/backup/)来备份 VM，请考虑[使用 DPM 为 SQL 工作负荷配置 Azure 备份](/azure/backup/backup-azure-backup-sql/)。 使用此方法时，将为组织配置一个备份管理员角色，并为 VM 和 SQL Server 创建统一的恢复过程。 否则，请使用 [Microsoft Azure 的 SQL Server 托管备份](https://msdn.microsoft.com/library/dn449496.aspx)。

## <a name="traffic-manager"></a>流量管理器

**执行手动故障回复。** 流量管理器故障转移后，请执行手动故障回复而不是自动故障回复。 在故障回复之前，请验证是否所有应用程序子系统的运行状况都正常。  否则，可能会造成应用程序在数据中心之间来回转移的情况。 有关详细信息，请参阅[在多个区域中运行 VM 以实现高可用性](../reference-architectures/virtual-machines-windows/multi-region-application.md)。

**创建运行状况探测终结点。** 创建一个自定义终结点用于报告应用程序的总体运行状况。 这样，当任何关键路径（而不仅仅是前端）发生故障时，流量管理器可故障转移。 如果任何关键依赖项不正常或不可访问，终结点应返回 HTTP 错误代码。 但是，不会报告非关键服务发生的错误。 否则，运行状况探测可能触发不必要的故障转移，并产生误报。 有关详细信息，请参阅[流量管理器终结点监视和故障转移](/azure/traffic-manager/traffic-manager-monitoring/)。

## <a name="virtual-machines"></a>虚拟机

**避免在单个 VM 上运行生产工作负荷。** 单个 VM 部署不能弹性应对计划内或计划外维护。 应该将多个 VM 放入一个可用性集或 [VM 规模集](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/)，并在其前面使用负载均衡器。

**预配 VM 时指定可用性集。** 目前，没有任何方法可以在预配 VM 后将 VM 添加到可用性集。 将新的 VM 添加到现有的可用性集时，请务必为该 VM 创建一个 NIC，并将该 NIC 添加到负载均衡器上的后端地址池。 否则，负载均衡器不会将网络流量路由到该 VM。

**将每个应用程序层放入不同的可用性集。** 在 N 层应用程序中，不要将不同层中的 VM 放入同一个可用性集。 可用性集中的 VM 放置在不同的容错域 (FD) 和更新域 (UD) 中。 但是，为了获得 FD 与 UD 的冗余优势，可用性集中的每个 VM 必须能够处理相同的客户端请求。

**根据性能要求选择适当的 VM 大小。** 将现有工作负荷转移到 Azure 时，首先请使用与本地服务器最匹配的 VM 大小。 然后测量与 CPU、内存和磁盘 IOPS 有关的实际工作负荷的性能，并根据需要调整大小。 这有助于确保应用程序在云环境中的行为符合预期。 此外，如果需要多个 NIC，请注意每种大小的 NIC 限制。

**对 VHD 使用托管磁盘。** [托管磁盘][managed-disks]为可用性集中的 VM 提供更高的可靠性，因为这些磁盘彼此充分隔离，可避免单一故障点。 此外，托管磁盘不受存储帐户中创建的 VHD 的 IOPS 限制。 有关详细信息，请参阅[在 Azure 中管理 Windows 虚拟机的可用性][vm-manage-availability]。

**将应用程序安装在数据磁盘而不是 OS 磁盘上。** 否则，可能会达到磁盘大小限制。

**使用 Azure 备份来备份 VM。** 备份可防范意外的数据丢失。 有关详细信息，请参阅[使用恢复服务保管库保护 Azure VM](/azure/backup/backup-azure-vms-first-look-arm/)。

**启用监视日志**，包括基本运行状况指标、基础结构日志和[启动诊断][boot-diagnostics]。 如果 VM 陷入不可启动状态，启动诊断有助于诊断启动故障。 有关详细信息，请参阅 [Azure 诊断日志概述][diagnostics-logs]。

**使用 AzureLogCollector 扩展。** （仅限 Windows VM。）此扩展可聚合 Azure 平台日志并将其上传到 Azure 存储，而无需操作员远程登录到 VM。 有关详细信息，请参阅 [AzureLogCollector 扩展](/azure/virtual-machines/virtual-machines-windows-log-collector-extension/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

## <a name="virtual-network"></a>虚拟网络

**要将公共 IP 地址加入允许列表或阻止，请将 NSG 添加到子网。** 阻止恶意用户的访问，或只允许有权访问应用程序的用户访问。  

**创建自定义运行状况探测。** 负载均衡器运行状况探测可以测试 HTTP 或 TCP。 如果 VM 运行 HTTP 服务器，HTTP 探测的运行状况指示效果比 TCP 探测更好。 对于 HTTP 探测，请使用自定义终结点来报告应用程序（包括所有关键依赖项）的总体运行状况。 有关详细信息，请参阅 [Azure 负载均衡器概述](/azure/load-balancer/load-balancer-overview/)。

**不要阻止运行状况探测。** 负载均衡器运行状况探测信息从已知 IP 地址 168.63.129.16 发送。 不要在任何防火墙策略或网络安全组 (NSG) 规则中阻止与此 IP 之间的流量。 阻止运行状况探测会导致负载均衡器从轮转项目中删除 VM。

**启用负载均衡器日志记录。** 这些日志显示后端有多少个 VM 由于失败的探测响应而未收到网络流量。 有关详细信息，请参阅 [Azure 负载均衡器的 Log Analytics](/azure/load-balancer/load-balancer-monitor-log/)。

<!-- links -->
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[diagnostics-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs/
[managed-disks]: /azure/storage/storage-managed-disks-overview
[search-optimization]: /azure/search/search-performance-optimization/
[sql-backup]: /azure/sql-database/sql-database-automated-backups/
[sql-restore]: /azure/sql-database/sql-database-recovery-using-backups/
[vm-manage-availability]: /azure/virtual-machines/windows/manage-availability#use-managed-disks-for-vms-in-an-availability-set
