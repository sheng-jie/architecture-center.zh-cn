---
title: "复原能力查检表"
description: "为设计过程中的复原能力考虑因素提供指导的查检表。"
author: petertaylor9999
ms.date: 01/10/2018
ms.custom: resiliency, checklist
ms.openlocfilehash: 66ff802c1f7b35db147ffe4279982c827570c3c1
ms.sourcegitcommit: 3d6dba524cc7661740bdbaf43870de7728d60a01
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/11/2018
---
# <a name="resiliency-checklist"></a>复原能力查检表

复原能力是指系统能够在发生故障后进行恢复，然后继续正常运行，它是[软件质量的构成要素](../guide/pillars.md)之一。 根据复原能力设计应用程序需要规划和缓解可能发生的各种故障模式。 使用此核对清单可以从复原能力的角度审查应用程序的体系结构。 

## <a name="requirements"></a>要求

**定义客户的可用性要求。** 客户对应用程序的组件有可用性要求，这会影响应用程序设计。 让客户认可应用程序每个组成部分的可用性目标，否则设计可能不符合客户期望。 有关详细信息，请参阅[定义复原要求](../resiliency/index.md#defining-your-resiliency-requirements)。

## <a name="application-design"></a>应用程序设计

**对应用程序执行故障模式分析 (FMA)。** FMA 是在设计阶段提前将复原能力整合到应用程序的过程。 有关详细信息，请参阅[故障模式分析][fma]。 FMA 的目标包括：  

* 识别应用程序可能遇到的故障类型。
* 捕获每种故障对应用程序造成的潜在影响。
* 确定恢复策略。
  

**部署服务的多个实例。** 如果应用程序依赖于服务的单个实例，则会造成单一故障点。 预配多个实例能够提高复原能力和可伸缩性。 对于 [Azure 应用服务](/azure/app-service/app-service-value-prop-what-is/)，请选择提供多个实例的[应用服务计划](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/)。 对于 Azure 云服务，请将每个角色配置为使用[多个实例](/azure/cloud-services/cloud-services-choose-me/#scaling-and-management)。 对于 [Azure 虚拟机 (VM)](/azure/virtual-machines/virtual-machines-windows-about/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)，请确保 VM 体系结构包含多个 VM，并且每个 VM 包含在[可用性集][availability-sets]中。   

**使用自动缩放来响应负载增加。** 如果应用程序未配置为随着负载的增加而自动横向扩展，当应用程序的服务中填满用户请求时，将会发生故障。 有关详细信息，请参阅以下文章：

* 一般信息：[可伸缩性查检表](./scalability.md)
* Azure 应用服务：[手动或自动缩放实例计数][app-service-autoscale]
* 云服务：[如何自动缩放云服务][cloud-service-autoscale]
* 虚拟机：[自动缩放和虚拟机规模集][vmss-autoscale]

**使用负载均衡分发请求。** 负载均衡通过从轮转项目中删除不正常的实例，将应用程序请求分发到正常的服务实例。 如果服务使用 Azure 应用服务或 Azure 云服务，则已均衡负载。 但是，如果应用程序使用 Azure VM，则你需要预配负载均衡器。 有关更多详细信息，请参阅 [Azure 负载均衡器](/azure/load-balancer/load-balancer-overview/)概述。

**将 Azure 应用程序网关配置为使用多个实例。** 根据应用程序的要求，[Azure 应用程序网关](/azure/application-gateway/application-gateway-introduction/)可能更适合用于将请求分发到应用程序的服务。 但是，应用程序网关服务的单个实例不享有 SLA 保障，因此，洱应用程序网关实例发生故障时，应用程序也可能发生故障。 预配多个中型或大型应用程序网关实例，保证根据 [SLA](https://azure.microsoft.com/support/legal/sla/application-gateway/v1_0/) 的条款提供服务可用性。

**为每个应用层使用 Azure 可用性集。** 将实例放入[可用性集][availability-sets]可提供更高的 [SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/)。 

**考虑跨多个区域部署应用程序。** 如果将应用程序部署到单个区域，在发生整个区域不可用的情况时（这种情况很罕见），应用程序也不可用。 根据应用程序的 SLA 条款，这种情况可能不可接受。 为此，请考虑跨多个区域部署应用程序及其服务。 多区域部署可以使用主动-主动模式（将请求分发到多个活动实例）或主动-被动模式（预留一个“热”实例，以防主实例发生故障）。 我们建议跨区域对部署应用程序服务的多个实例。 有关详细信息，请参阅[业务连续性和灾难恢复 (BCDR)：Azure 配对区域](/azure/best-practices-availability-paired-regions)。

**使用 Azure 流量管理器将应用程序的流量路由到不同的区域。**  [Azure 流量管理器][traffic-manager]在 DNS 级别执行负载均衡，根据指定的[流量路由][traffic-manager-routing]方法和应用程序终结点的运行状况将流量路由到不同区域。 如果不使用流量管理器，则受限于部署的单个区域，从而会限制规模，增大用户的延迟，并导致应用程序在发生区域范围的服务中断时停机。

**为负载均衡器和流量管理器配置并测试运行状况探测。** 确保运行状况逻辑检查系统关键部件，并相应地对运行状况探测做出响应。

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

**阐述并测试数据源故障转移和故障回复过程。** 当数据源发生灾难性的故障时，人工操作员必须遵循一套阐述的说明故障转移到新数据源。 如果阐述的步骤存在错误，操作员将无法成功遵循它们和故障转移资源。 定期测试说明步骤，以验证操作员是否能够遵循它们成功故障转移和故障回复数据源。

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

**对应用程序执行故障转移和故障回复测试。** 如果不全面测试故障转移和故障回复，则无法确定在灾难恢复过程中，应用程序中的依赖服务以同步方式恢复正常运行。 确保应用程序的依赖服务按正确的顺序故障转移和故障回复。

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

## <a name="azure-services"></a>Azure 服务
以下查检表项目适用于 Azure 中的特定服务。

- [应用服务](#app-service)
- [应用程序网关](#application-gateway)
- [Cosmos DB](#cosmos-db)
- [Redis 缓存](#redis-cache)
- [搜索](#search)
- [存储](#storage)
- [SQL 数据库](#sql-database)
- [VM 中运行的 SQL Server](#sql-server-running-in-a-vm)
- [流量管理器](#traffic-manager)
- [虚拟机](#virtual-machines)
- [虚拟网络](#virtual-network)

### <a name="app-service"></a>应用服务

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

### <a name="application-gateway"></a>应用程序网关

**至少预配两个实例。** 部署至少包含两个实例的应用程序网关。 单个实例会成为单一故障点。 使用两个或更多个实例实现冗余和可伸缩性。 若要满足 [SLA](https://azure.microsoft.com/support/legal/sla/application-gateway/v1_0/)，必须预配两个或更多个中型或大型实例。

### <a name="cosmos-db"></a>Cosmos DB

**跨区域复制数据库。** Cosmos DB 允许将任意数量的 Azure 区域与 Cosmos DB 数据库帐户关联。 Cosmos DB 数据库可以包含一个写入区域和多个读取区域。 如果写入区域发生故障，可以从另一个副本读取。 客户端 SDK 会自动处理此操作。 还可以将写入区域故障转移到另一个区域。 有关详细信息，请参阅[如何使用 Azure Cosmos DB 在全球范围内分发数据？](/azure/documentdb/documentdb-distribute-data-globally)

### <a name="redis-cache"></a>Redis 缓存

**配置异地复制**。 异地复制提供一种用于链接两个高级层 Azure Redis 缓存实例的机制。 写入主缓存的数据将复制到辅助只读缓存。 有关详细信息，请参阅[如何为 Azure Redis 缓存配置异地复制功能](/azure/redis-cache/cache-how-to-geo-replication)

**配置数据持久性。** 使用 Redis 持久性可以在 Redis 中持久存储数据。 还可以获取快照并备份数据，以便在出现硬件故障时进行加载。 有关详细信息，请参阅[如何为高级 Azure Redis 缓存配置数据持久性](/azure/redis-cache/cache-how-to-premium-persistence)

如果将 Redis 缓存使用临时数据缓存而不是持久性存储，则这些建议可能不适用。 

### <a name="search"></a>搜索

**预配多个副本。** 至少使用两个副本实现读取高可用性，或至少使用三个副本实现读写高可用性。

**为多区域部署配置索引器。** 若要进行多区域部署，请考虑使用索引连续性选项。

  * 如果数据源是异地复制的，通常应该将每个区域性 Azure 搜索服务的每个索引器指向其本地数据源副本。 但是，对于存储在 Azure SQL 数据库的大型数据集，不建议使用此方法。 原因在于，Azure 搜索无法从辅助 SQL 数据库副本执行增量索引，而只能从主副本执行。 应该将所有索引器指向主副本。 故障转移后，请将 Azure 搜索索引器指向新的主副本。  
  * 如果数据源不是异地复制的，请将多个索引器指向同一个数据源，以便多个区域中的 Azure 搜索服务可从数据源连续独立地编制索引。 有关详细信息，请参阅 [Azure 搜索性能和优化注意事项][search-optimization]。

### <a name="storage"></a>存储

**对于应用程序数据，请使用读取访问异地冗余存储 (RA-GRS)。** RA-GRS 存储将数据复制到次要区域，并提供从次要区域的只读访问权限。 如果主要区域中发生存储故障，应用程序可以从次要区域读取数据。 有关详细信息，请参阅 [Azure 存储复制](/azure/storage/storage-redundancy/)。

**对于 VM 磁盘，请使用托管磁盘。** [托管磁盘][managed-disks]为可用性集中的 VM 提供更高的可靠性，因为这些磁盘彼此充分隔离，可避免单一故障点。 此外，托管磁盘不受存储帐户中创建的 VHD 的 IOPS 限制。 有关详细信息，请参阅[在 Azure 中管理 Windows 虚拟机的可用性][vm-manage-availability]。

**对于队列存储，请在另一个区域中创建备份队列。** 对于队列存储，只读副本的作用有限，因为无法将项排队或取消排队。 应在另一个区域中的某个存储帐户中创建备份队列。 如果发生存储故障，应用程序可以使用备份队列，直到主要区域重新可用。 这样，应用程序仍可处理新请求。  

### <a name="sql-database"></a>SQL 数据库

**使用“标准”或“高级”层。** 这些层提供更长的时间点还原期限（35 天）。 有关详细信息，请参阅 [SQL 数据库选项和性能](/azure/sql-database/sql-database-service-tiers/)。

**启用 SQL 数据库审核。** 审核可用于诊断恶意攻击或人为错误。 有关详细信息，请参阅 [SQL 数据库审核入门](/azure/sql-database/sql-database-auditing-get-started/)。

**使用活动异地复制**使用活动异地复制在一个不同的区域中创建可读取的辅助副本。  如果主数据库出现故障，或者只是需要脱机，可以手动故障转移到任何辅助数据库。  在故障转移之前，辅助数据库将保持只读状态。  有关详细信息，请参阅 [SQL 数据库活动异地复制](/azure/sql-database/sql-database-geo-replication-overview/)。

**使用分片。** 考虑使用分片将数据库横向分区。 分片可提供故障隔离。 有关详细信息，请参阅[使用 Azure SQL 数据库进行横向扩展](/azure/sql-database/sql-database-elastic-scale-introduction/)。

**发生人为错误后使用时间点还原进行恢复。**  时间点还原可将数据库还原到以前的某个时间点。 有关详细信息，请参阅[使用自动数据库备份恢复 Azure SQL 数据库][sql-restore]。

**发生服务中断后使用异地还原进行恢复。** 异地还原可从异地冗余的备份还原数据库。  有关详细信息，请参阅[使用自动数据库备份恢复 Azure SQL 数据库][sql-restore]。

### <a name="sql-server-running-in-a-vm"></a>VM 中运行的 SQL Server

**复制数据库。** 使用 SQL Server Always On 可用性组复制数据库。 当某个 SQL Server 实例发生故障时可提供高可用性。 有关详细信息，请参阅[对 N 层应用程序运行 Windows VM](../reference-architectures/virtual-machines-windows/n-tier.md)

**备份数据库**。 如果已在使用 [Azure 备份](https://azure.microsoft.com/documentation/services/backup/)来备份 VM，请考虑[使用 DPM 为 SQL 工作负荷配置 Azure 备份](/azure/backup/backup-azure-backup-sql/)。 使用此方法时，将为组织配置一个备份管理员角色，并为 VM 和 SQL Server 创建统一的恢复过程。 否则，请使用 [Microsoft Azure 的 SQL Server 托管备份](https://msdn.microsoft.com/library/dn449496.aspx)。

### <a name="traffic-manager"></a>流量管理器

**执行手动故障回复。** 流量管理器故障转移后，请执行手动故障回复而不是自动故障回复。 在故障回复之前，请验证是否所有应用程序子系统的运行状况都正常。  否则，可能会造成应用程序在数据中心之间来回转移的情况。 有关详细信息，请参阅[在多个区域中运行 VM 以实现高可用性](../reference-architectures/virtual-machines-windows/multi-region-application.md)。

**创建运行状况探测终结点。** 创建一个自定义终结点用于报告应用程序的总体运行状况。 这样，当任何关键路径（而不仅仅是前端）发生故障时，流量管理器可故障转移。 如果任何关键依赖项不正常或不可访问，终结点应返回 HTTP 错误代码。 但是，不会报告非关键服务发生的错误。 否则，运行状况探测可能触发不必要的故障转移，并产生误报。 有关详细信息，请参阅[流量管理器终结点监视和故障转移](/azure/traffic-manager/traffic-manager-monitoring/)。

### <a name="virtual-machines"></a>虚拟机

**避免在单个 VM 上运行生产工作负荷。** 单个 VM 部署不能弹性应对计划内或计划外维护。 应该将多个 VM 放入一个可用性集或 [VM 规模集](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/)，并在其前面使用负载均衡器。

**预配 VM 时指定可用性集。** 目前，没有任何方法可以在预配 VM 后将 VM 添加到可用性集。 将新的 VM 添加到现有的可用性集时，请务必为该 VM 创建一个 NIC，并将该 NIC 添加到负载均衡器上的后端地址池。 否则，负载均衡器不会将网络流量路由到该 VM。

**将每个应用程序层放入不同的可用性集。** 在 N 层应用程序中，不要将不同层中的 VM 放入同一个可用性集。 可用性集中的 VM 放置在不同的容错域 (FD) 和更新域 (UD) 中。 但是，为了获得 FD 与 UD 的冗余优势，可用性集中的每个 VM 必须能够处理相同的客户端请求。

**根据性能要求选择适当的 VM 大小。** 将现有工作负荷转移到 Azure 时，首先请使用与本地服务器最匹配的 VM 大小。 然后测量与 CPU、内存和磁盘 IOPS 有关的实际工作负荷的性能，并根据需要调整大小。 这有助于确保应用程序在云环境中的行为符合预期。 此外，如果需要多个 NIC，请注意每种大小的 NIC 限制。

**对 VHD 使用托管磁盘。** [托管磁盘][managed-disks]为可用性集中的 VM 提供更高的可靠性，因为这些磁盘彼此充分隔离，可避免单一故障点。 此外，托管磁盘不受存储帐户中创建的 VHD 的 IOPS 限制。 有关详细信息，请参阅[在 Azure 中管理 Windows 虚拟机的可用性][vm-manage-availability]。

**将应用程序安装在数据磁盘而不是 OS 磁盘上。** 否则，可能会达到磁盘大小限制。

**使用 Azure 备份来备份 VM。** 备份可防范意外的数据丢失。 有关详细信息，请参阅[使用恢复服务保管库保护 Azure VM](/azure/backup/backup-azure-vms-first-look-arm/)。

**启用监视日志**，包括基本运行状况指标、基础结构日志和[启动诊断][boot-diagnostics]。 如果 VM 陷入不可启动状态，启动诊断有助于诊断启动故障。 有关详细信息，请参阅 [Azure 诊断日志概述][diagnostics-logs]。

**使用 AzureLogCollector 扩展。** （仅限 Windows VM。）此扩展可聚合 Azure 平台日志并将其上传到 Azure 存储，而无需操作员远程登录到 VM。 有关详细信息，请参阅 [AzureLogCollector 扩展](/azure/virtual-machines/virtual-machines-windows-log-collector-extension/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

### <a name="virtual-network"></a>虚拟网络

**要将公共 IP 地址加入白名单或阻止，请将 NSG 添加到子网。** 阻止恶意用户的访问，或只允许有权访问应用程序的用户访问。  

**创建自定义运行状况探测。** 负载均衡器运行状况探测可以测试 HTTP 或 TCP。 如果 VM 运行 HTTP 服务器，HTTP 探测的运行状况指示效果比 TCP 探测更好。 对于 HTTP 探测，请使用自定义终结点来报告应用程序（包括所有关键依赖项）的总体运行状况。 有关详细信息，请参阅 [Azure 负载均衡器概述](/azure/load-balancer/load-balancer-overview/)。

**不要阻止运行状况探测。** 负载均衡器运行状况探测信息从已知 IP 地址 168.63.129.16 发送。 不要在任何防火墙策略或网络安全组 (NSG) 规则中阻止与此 IP 之间的流量。 阻止运行状况探测会导致负载均衡器从轮转项目中删除 VM。

**启用负载均衡器日志记录。** 这些日志显示后端有多少个 VM 由于失败的探测响应而未收到网络流量。 有关详细信息，请参阅 [Azure 负载均衡器的 Log Analytics](/azure/load-balancer/load-balancer-monitor-log/)。


<!-- links -->
[app-service-autoscale]: /azure/monitoring-and-diagnostics/insights-how-to-scale/
[asynchronous-c-sharp]: /dotnet/articles/csharp/async
[availability-sets]:/azure/virtual-machines/virtual-machines-windows-manage-availability/
[azure-backup]: https://azure.microsoft.com/documentation/services/backup/
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[circuit-breaker]: ../patterns/circuit-breaker.md
[cloud-service-autoscale]: /azure/cloud-services/cloud-services-how-to-scale/
[diagnostics-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs/
[fma]: ../resiliency/failure-mode-analysis.md
[resilient-deployment]: ../resiliency/index.md#resilient-deployment
[load-balancer]: /azure/load-balancer/load-balancer-overview/
[managed-disks]: /azure/storage/storage-managed-disks-overview
[monitoring-and-diagnostics-guidance]: ../best-practices/monitoring.md
[resource-manager]: /azure/azure-resource-manager/resource-group-overview/
[retry-pattern]: ../patterns/retry.md
[retry-service-guidance]: ../best-practices/retry-service-specific.md
[search-optimization]: /azure/search/search-performance-optimization/
[sql-backup]: /azure/sql-database/sql-database-automated-backups/
[sql-restore]: /azure/sql-database/sql-database-recovery-using-backups/
[traffic-manager]: /azure/traffic-manager/traffic-manager-overview/
[traffic-manager-routing]: /azure/traffic-manager/traffic-manager-routing-methods/
[vm-manage-availability]: /azure/virtual-machines/windows/manage-availability#use-managed-disks-for-vms-in-an-availability-set
[vmss-autoscale]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview/
