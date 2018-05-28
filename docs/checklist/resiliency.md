---
title: 复原能力清单
description: 为设计过程中的复原能力考虑因素提供指导的清单。
author: petertaylor9999
ms.date: 01/10/2018
ms.custom: resiliency, checklist
ms.openlocfilehash: ca4bf77c9348f6c656348d9cd61d3a1241d69ba8
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/08/2018
---
# <a name="resiliency-checklist"></a>复原能力清单

复原能力是指系统能够在发生故障后进行恢复，然后继续正常运行，它是[软件质量的构成要素](../guide/pillars.md)之一。 根据复原能力设计应用程序需要规划和缓解可能发生的各种故障模式。 核对此清单可以从复原能力的角度审查应用程序的架构。 另请查看[特定 Azure 服务的复原能力检查表](./resiliency-per-service.md)。

## <a name="requirements"></a>要求

**定义客户的可用性要求。** 客户对应用程序的组件有可用性要求，这会影响应用程序设计。 让客户认可应用程序每个组成部分的可用性目标，否则设计可能不符合客户期望。 有关详细信息，请参阅[定义复原要求](../resiliency/index.md#defining-your-resiliency-requirements)。

## <a name="application-design"></a>应用程序设计

**对应用程序执行故障模式分析 (FMA)。** FMA 是在设计阶段提前将复原能力整合到应用程序的过程。 有关详细信息，请参阅[故障模式分析][fma]。 FMA 的目标包括：  

* 识别应用程序可能遇到的故障类型。
* 捕获每种故障对应用程序造成的潜在影响。
* 确定恢复策略。
  

**部署服务的多个实例。** 如果应用程序依赖于服务的单个实例，则会造成单一故障点。 预配多个实例能够提高复原能力和可伸缩性。 对于 [Azure 应用服务](/azure/app-service/app-service-value-prop-what-is/)，请选择提供多个实例的[应用服务计划](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/)。 对于 Azure 云服务，请将每个角色配置为使用[多个实例](/azure/cloud-services/cloud-services-choose-me/#scaling-and-management)。 对于 [Azure 虚拟机 (VM)](/azure/virtual-machines/virtual-machines-windows-about/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)，请确保 VM 体系结构包含多个 VM，并且每个 VM 包含在[可用性集][availability-sets]中。   

**使用自动缩放来响应负载增加。** 如果应用程序未配置为随着负载的增加而自动横向扩展，如果用户的请求饱和，那么应用程序的服务可能会失败。 有关详细信息，请参阅以下文章：

* 一般信息：[可伸缩性清单](./scalability.md)
* Azure 应用服务：[手动或自动缩放实例计数][app-service-autoscale]
* 云服务：[如何自动缩放云服务][cloud-service-autoscale]
* 虚拟机：[自动缩放和虚拟机规模集][vmss-autoscale]

**使用负载均衡来分发请求。** 负载均衡通过从循环列表中删除不正常的实例，将应用程序请求分发到正常的服务实例。 如果服务使用 Azure 应用服务或 Azure 云服务，则已负载均衡。 但是，如果应用程序使用 Azure VM，则你需要预配负载均衡器。 有关更多详细信息，请参阅 [Azure 负载均衡器](/azure/load-balancer/load-balancer-overview/)概述。

**将 Azure 应用程序网关配置为使用多个实例。** 根据应用程序的要求，[Azure 应用程序网关](/azure/application-gateway/application-gateway-introduction/)可能更适合用于将请求分发到应用服务。 但是，应用程序网关服务的单个实例不享有 SLA 保障，因此，洱应用程序网关实例发生故障时，应用程序也可能发生故障。 预配多个中型或大型应用程序网关实例，保证根据 [SLA](https://azure.microsoft.com/support/legal/sla/application-gateway/v1_0/) 的条款提供服务可用性。

**为每个应用层使用 Azure 可用性集。** 将实例放入[可用性集][availability-sets]可提供更高的 [SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/)。 

**考虑跨多个区域部署应用程序。** 如果将应用程序部署到单个区域，在发生整个区域不可用的情况时（这种情况很罕见），应用程序也不可用。 根据应用程序的 SLA 条款，这种情况可能不可接受。 为此，请考虑跨多个区域部署应用程序及其服务。 多区域部署可以使用主动-主动模式（将请求分发到多个活动实例）或主动-被动模式（预留一个“热”实例，以防主实例发生故障）。 我们建议跨区域对部署应用程序服务的多个实例。 有关详细信息，请参阅[业务连续性和灾难恢复 (BCDR)：Azure 配对区域](/azure/best-practices-availability-paired-regions)。

**使用 Azure 流量管理器将应用程序的流量路由到不同的区域。**  [Azure 流量管理器][traffic-manager]在 DNS 级别执行负载均衡，根据指定的[流量路由][traffic-manager-routing]方法和应用程序终结点的运行状况将流量路由到不同区域。 如果不使用流量管理器，则受限于部署的单个区域，从而会限制规模，增大用户的延迟，并导致应用程序在发生区域范围的服务中断时停机。

**为负载均衡器和流量管理器配置并测试运行健康状况探测。** 确保运行状况逻辑检查系统关键部件，并相应地对运行状况探测做出响应。

* [Azure 流量管理器][traffic-manager]和 [Azure 负载均衡器][load-balancer]的运行状况探测充当特定的功能。 对于流量管理器，运行状况探测确定是否故障转移到另一个区域。 对于负载均衡器，它确定是否从轮转项目中删除某个 VM。      
* 对于流量管理器探测，运行状况终结点应检查所有关键依赖项，它们部署在同一区域，其故障应触发故障转移到另一个区域。  
* 对于负载均衡器，运行状况终结点应报告 VM 的运行状况。 不要包含其他层或外部服务。 否则，在 VM 外部发生的故障会导致负载均衡器从轮转项目中删除该 VM。
* 有关在应用程序中实施运行状况监视的指导，请参阅[运行状况终结点监视模式](https://msdn.microsoft.com/library/dn589789.aspx)。

**监视第三方服务。** 如果应用程序依赖于第三方服务，请确定这些第三方服务可能会在哪个位置出现何种故障，以及这些故障对应用程序造成的影响。 第三方服务可能不包括监视和诊断，因此，必须记录其调用，并使用唯一标识符将其与应用程序的运行状况和诊断日志记录相关联。 有关监视和诊断的成熟做法的详细信息，请参阅[监视和诊断指南][monitoring-and-diagnostics-guidance]。

**确保使用的任何第三方服务提供 SLA。** 如果应用程序依赖于某个第三方服务，但该服务不以 SLA 的形式保证可用性，则也无法保证应用程序的可用性。 SLA 只与应用程序的最低可用性组件一样高。

**在适用的情况下，请实施复原模式以进行远程操作。** 如果应用程序依赖于远程服务之间的通信，请遵循处理暂时性故障的设计模式，例如[重试模式][retry-pattern]和[断路器模式][circuit-breaker]。 有关详细信息，请参阅[复原策略](../resiliency/index.md#resiliency-strategies)。

**尽量执行异步操作。** 同步操作可能会独占资源，并在调用方等待进程完成时阻塞其他操作。 设计应用程序的每个组成部分，以尽量执行异步操作。 有关如何在 C# 中实现异步编程的详细信息，请参阅[使用 async 和 await 进行异步编程][asynchronous-c-sharp]。

## <a name="data-management"></a>数据管理

**了解应用程序数据源的复制方法。** 应用程序数据将存储在不同的数据源中，因此具有不同的可用性要求。 评估 Azure 中每种数据存储的复制方法，包括 [Azure 存储复制](/azure/storage/storage-redundancy/)和 [SQL 数据库活动异地复制](/azure/sql-database/sql-database-geo-replication-overview/)，确保满足应用程序的数据要求。

**确保没有任何用户帐户同时有权访问生产和备份数据。** 如果单个用户帐户同时有权写入生产和备份源，则将会透露数据备份。 恶意用户可能有意删除所有数据，而普通用户可能意外删除数据。 将应用程序设计为限制每个用户帐户的权限，以便只有需要写访问权限的用户拥有写访问权限，并且只能写入生产或备份数据，但不能同时写入两者。

**阐述并测试数据源故障转移和故障恢复过程。** 当数据源发生灾难性的故障时，人工操作员必须遵循一套阐述的说明故障转移到新数据源。 如果阐述的步骤存在错误，操作员将无法成功遵循它们和故障转移资源。 定期测试说明步骤，以验证操作员是否能够遵循它们成功故障转移和故障恢复数据源。

**验证数据备份。** 定期通过运行脚本验证数据完整性、架构和查询，验证备份数据是否符合预期。 如果某个备份对还原数据源没有任何帮助，则就没有必要保留该备份。 记录并报告任何不一致情况，以便能够修复备份服务。

**考虑使用异地冗余的存储帐户类型。** Azure 存储帐户中存储的数据始终在本地复制。 但是，在预配存储帐户时，有多个复制策略可供选择。 选择 [Azure 读取访问异地冗余存储 (RA-GRS)](/azure/storage/storage-redundancy/#read-access-geo-redundant-storage) 可以在整个区域不可用时（这种情况很罕见）保护应用程序数据。

> [!NOTE]
> 对于 VM，请不要依赖于 RA-GRS 复制来还原 VM 磁盘（VHD 文件）， 而应该使用 [Azure 备份][azure-backup]。   
>
>

## <a name="security"></a>“安全”

**针对分布式拒绝服务 (DDoS) 攻击实施应用程序级保护。** Azure 服务可在网络层防范 DDos 攻击。 但是，Azure 无法防范应用程序层攻击，因为很难将真实用户请求与恶意用户请求区分开来。 有关如何防范应用程序层 DDoS 攻击的详细信息，请参阅 [Microsoft Azure 网络安全性](http://download.microsoft.com/download/C/A/3/CA3FC5C0-ECE0-4F87-BF4B-D74064A00846/AzureNetworkSecurity_v3_Feb2015.pdf)中的“防范 DDoS”部分（PDF 下载）。

**对应用程序资源的访问权限实施最低特权原则。** 尽量限制对应用程序资源的默认访问权限。 只在经过批准后才授予更高级别的权限。 默认授予对应用程序资源的过高访问权限可能导致某人有意或无意删除资源。 Azure 提供[基于角色的访问控制](/azure/active-directory/role-based-access-built-in-roles/)用于管理用户特权，但必须验证具有自身权限系统（例如 SQL Server）的其他资源的最低特权。

## <a name="testing"></a>测试

**对应用程序执行故障转移和故障恢复测试。** 如果不全面测试故障转移和故障回复，则无法确定在灾难恢复过程中，应用程序中的依赖服务以同步方式恢复正常运行。 确保应用程序的依赖服务按正确的顺序故障转移和故障恢复。

**对应用程序执行故障注入测试。** 应用程序可能出于多种不同的原因而发生故障，例如，证书过期、VM 中系统资源耗尽或存储故障。 在尽可能接近生产环境的环境中，通过模拟或触发实际故障来测试应用程序。 例如，删除证书、人为地消耗系统资源，或删除存储源。 验证应用程序是否能够从所有类型的故障（单独或组合的故障）中恢复。 检查故障是否不会通过系统传播或引发连锁故障。

**使用合成数据和真实用户数据在生产环境中运行测试。** 测试环境和生产环境很少完全相同，因此，必须使用蓝/绿部署或金丝雀部署方法在生产环境中测试应用程序。 这样，便可以在真实负载下在生产环境中测试应用程序，并确保它在完全部署后可按预期运行。

## <a name="deployment"></a>部署

**阐述应用程序的发布过程。** 如果不提供详细的发布过程文档，操作员可能部署错误的更新，或不当地配置应用程序的设置。 明确定义和阐述发布过程，并确保将其提供给整个运营团队。 

**自动执行应用程序部署过程。** 如果需要操作人员手动部署应用程序，则人为错误可能导致部署失败。 

**设计发布过程，以便尽量提高应用程序可用性。** 如果发布过程要求服务在部署期间脱机，应用程序只能在重新联机后才可用。 使用[蓝/绿](http://martinfowler.com/bliki/BlueGreenDeployment.html)或[金丝雀发布](http://martinfowler.com/bliki/CanaryRelease.html)部署方法将应用程序部署到生产环境。 这两种方法涉及到连同生产代码一起部署发布代码，因此，在发生故障时，发布代码的用户可重定向到生产代码。

**记录并审核应用程序的部署。** 如果使用分阶段部署方法（例如蓝/绿或金丝雀发布方法），则生产环境中会运行的应用程序的多个版本。 如果出现问题，确定应用程序的哪个版本导致该问题至关重要。 实施可靠的日志记录策略来尽量多地捕获版本特定的信息。

**为部署创建回滚计划。** 应用程序部署可能失败，并导致应用程序不可用。 请设计回滚过程，以恢复到上次已知正确的版本并最小化停机时间。 

## <a name="operations"></a>操作

**实施有关在应用程序中进行监视和警报的最佳做法。** 不采用适当的监视、诊断和警报措施，就无法检测应用程序中的故障并提醒操作员解决故障。 有关详细信息，请参阅[监视和诊断指南][monitoring-and-diagnostics-guidance]。

**度量远程调用统计信息，并将信息提供给应用程序团队。**  如果不实时跟踪和报告远程调用统计信息并提供查看此信息的简单办法，则运营团队不能即时深入了解应用程序的运行状况。 如果仅度量平均远程调用时间，则没有足够的信息揭露服务中的问题。 汇总远程调用指标，例如延迟、吞吐量和 99 和 95 百分位中的错误。 针对指标执行统计分析，以发现每个百分位中发生的错误。

**跟踪适当时间范围内的暂时性异常和重试数目。** 如果不跟踪和监视各时间段的暂时性异常和重试，应用程序的重试逻辑中可能会隐藏问题或故障。 也就是说，如果监视和日志记录仅显示操作的成功或失败结果，则由于异常而必须多次重试的操作将会隐藏。 在一段时间内异常的增加趋势指示此服务有问题，并可能发生故障。 有关详细信息，请参阅[重试服务指南][retry-service-guidance]。

**实施提醒操作员的提前警告系统。** 识别应用程序运行状况的关键性能指标，例如暂时性异常和远程调用延迟，并设置其中每个的适当阈值。 达到阈值时，将警报发送到操作员。 在它们变得严重并且需要恢复响应之前，设置确定问题的阈值级别。

**确保团队的多个人员经过培训，能够监视应用程序并执行任何手动恢复步骤。** 如果团队中只有一个操作员可以监视应用程序和启动恢复步骤，此人将成为单一故障点。 培训多个检测和恢复人员，并确保始终至少有一个处于活动状态。

**确保应用程序不会达到 [Azure 订阅限制](/azure/azure-subscription-service-limits/)。** Azure 订阅限制特定的资源类型，例如资源组数量、内核数量和存储帐户数量。  如果应用程序的要求超过 Azure 订阅限制，请创建另一个 Azure 订阅并预配足够的资源。

**确保应用程序不会达到[每个服务的限制](/azure/azure-subscription-service-limits/)。** 每个 Azure 服务存在消耗量限制 &mdash; 例如，存储、吞吐量、连接、每秒请求数和其他指标的限制。 如果应用程序尝试使用的资源超过这些限制，则会发生故障。 这会导致服务限制，并可能给受影响用户造成停机。 根据特定的服务和应用程序要求，通常可以通过纵向扩展（例如，选择另一个定价层）或横向扩展（添加新实例）避免这些限制。  

**将应用程序的存储要求设计为处在Azure 存储可伸缩性和性能目标的范围内。** Azure 存储需在预定义的可伸缩性和性能目标内运行，因此请将应用程序设计为使用这些目标内的存储。 如果超出这些目标，应用程序会受到存储限制。 若要解决此问题，请预配更多的存储帐户。 如果达到存储帐户限制，请预配更多的 Azure 订阅，然后预配更多的存储帐户。 有关详细信息，请参阅 [Azure 存储可伸缩性和性能目标](/azure/storage/storage-scalability-targets/)。

**为应用程序选择适当的 VM 大小。** 在生产环境中度量 VM 的实际 CPU、内存、磁盘和 I/O，并验证选择的 VM 大小是否足够。 如果不足，当 VM 接近其限制时，应用程序可能遇到容量问题。 [Azure 中虚拟机的大小](/azure/virtual-machines/virtual-machines-windows-sizes/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)中详细介绍了 VM 大小。

**确定应用程序的工作负荷在一段时间内是稳定还是波动。** 如果工作负荷在一段时间内波动，请使用 Azure VM 规模集自动缩放 VM 实例的数目。 否则，必须手动增加或减少 VM 的数目。 有关详细信息，请参阅[虚拟机规模集概述](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/)。

**为 Azure SQL 数据库选择适当的服务层。** 如果应用程序使用 Azure SQL 数据库，请确保选择适当的服务层。 如果选择的层无法应对应用程序的数据库事务单位 (DTU) 要求，则数据将受到限制。 有关选择正确服务计划的详细信息，请参阅 [SQL 数据库选项和性能：了解每个服务层提供的功能](/azure/sql-database/sql-database-service-tiers/)。

**创建与 Azure 支持部门交互的过程。** 如果在需要联系支持人员之前未设置联系 [Azure 支持部门](https://azure.microsoft.com/support/plans/)的过程，则停机时间将会延长，因为第一次需要导航支持过程。 在设计应用程序复原能力的过程中，从一开始就包括与支持部门联系的过程，以及升级事务的过程。

**确保应用程序不会使用超过每个订阅的最大存储帐户数。** Azure 允许为每个订阅最多使用 200 个存储帐户。 如果应用程序所需的存储帐户数超过订阅中当前提供的数目，则需要创建新的订阅，并在其中创建更多的存储帐户。 有关详细信息，请参阅 [Azure 订阅和服务限制、配额与约束](/azure/azure-subscription-service-limits/#storage-limits)。

**确保应用程序不超过虚拟机磁盘的可伸缩性目标。** Azure IaaS VM 根据多种因素（包括 VM 大小和存储帐户类型）支持附加一些数据磁盘。 如果应用程序超过虚拟机磁盘的可伸缩性目标，请预配更多的存储帐户并在其中创建虚拟机磁盘。 有关详细信息，请参阅 [Azure 存储可伸缩性和性能目标](/azure/storage/storage-scalability-targets/#scalability-targets-for-virtual-machine-disks)

## <a name="telemetry"></a>遥测

**当应用程序在生产环境中运行时记录遥测数据。** 当应用程序在生产环境中运行时捕获可靠的遥测信息，否则没有足够的信息来诊断应用程序为用户提供服务时出现的问题的原因。 有关详细信息，请参阅[监视和诊断][monitoring-and-diagnostics-guidance]。

**使用异步模式实施日志记录。** 如果日志记录操作是同步的，则可能会阻止应用程序代码。 确保将日志记录操作实施为异步操作。

**跨服务边界关联日志数据。** 在典型的 N 层应用程序中，用户请求可能遍历多个服务边界。 例如，某个用户请求通常源自于 Web 层并传递到业务层，最后保留在数据层。 在更复杂的情况下，可将用户请求分发到多个不同的服务和数据存储。 确保日志记录系统跨服务边界关联调用，以便可以跟踪整个应用程序中的请求。

## <a name="azure-resources"></a>Azure 资源

**使用 Azure 资源管理器模板预配资源。** 使用资源管理器模板可以更轻松地通过 PowerShell 或 Azure CLI 自动完成部署，使部署过程更可靠。 有关详细信息，请参阅 [Azure 资源管理器概述][resource-manager]。

**为资源指定有意义的名称。** 为资源指定有意义的名称可以更轻松地找到特定资源并了解其角色。 有关详细信息，请参阅 [Azure 资源的建议命名约定](../best-practices/naming-conventions.md)。

**使用基于角色的访问控制 (RBAC)。** 使用 RBAC 控制对已部署的 Azure 资源的访问。 RBAC 允许向 DevOps 团队成员分配授权角色，防止意外删除或更改已部署的资源。 有关详细信息，请参阅 [Azure 门户中的访问管理入门](/azure/active-directory/role-based-access-control-what-is/)。

**对 VM 等关键资源使用资源锁。** 资源锁可防止操作员意外删除资源。 有关详细信息，请参阅 [使用 Azure 资源管理器锁定资源](/azure/azure-resource-manager/resource-group-lock-resources/)

**选择区域对。** 部署到两个区域时，请从相同的区域对中选择区域。 如果发生大范围的服务中断，会优先恢复每个配对中的一个区域。 某些服务（例如异地冗余存储）提供自动复制到配对区域的功能。 有关详细信息，请参阅[业务连续性和灾难恢复 (BCDR)：Azure 配对区域](/azure/best-practices-availability-paired-regions)

**按功能和生命周期组织资源组。**  一般情况下，资源组应包含具有相同生命周期的资源。 这样可以更方便地管理部署、删除测试部署和分配访问权限，减少意外删除或修改生产部署的可能性。 为生产、开发和测试环境创建单独的资源组。 在多区域部署中，请将每个区域的资源放入单独的资源组。 这可以更方便地重新部署一个区域，而不影响其他区域。

## <a name="next-steps"></a>后续步骤

- [特定 Azure 服务的复原能力检查表](./resiliency-per-service.md)
- [故障模式分析](../resiliency/failure-mode-analysis.md)


<!-- links -->
[app-service-autoscale]: /azure/monitoring-and-diagnostics/insights-how-to-scale/
[asynchronous-c-sharp]: /dotnet/articles/csharp/async
[availability-sets]:/azure/virtual-machines/virtual-machines-windows-manage-availability/
[azure-backup]: https://azure.microsoft.com/documentation/services/backup/
[circuit-breaker]: ../patterns/circuit-breaker.md
[cloud-service-autoscale]: /azure/cloud-services/cloud-services-how-to-scale/
[fma]: ../resiliency/failure-mode-analysis.md
[load-balancer]: /azure/load-balancer/load-balancer-overview/
[monitoring-and-diagnostics-guidance]: ../best-practices/monitoring.md
[resource-manager]: /azure/azure-resource-manager/resource-group-overview/
[retry-pattern]: ../patterns/retry.md
[retry-service-guidance]: ../best-practices/retry-service-specific.md
[traffic-manager]: /azure/traffic-manager/traffic-manager-overview/
[traffic-manager-routing]: /azure/traffic-manager/traffic-manager-routing-methods/
[vmss-autoscale]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview/
