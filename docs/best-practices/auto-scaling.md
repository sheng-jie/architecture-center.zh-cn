---
title: 自动缩放指南
description: 有关如何自动缩放以动态分配应用程序所需的资源的指南。
author: dragon119
ms.date: 05/17/2017
pnp.series.title: Best Practices
ms.openlocfilehash: a8489aaabab2b8523fbc9f026f4f435bb6d1ad29
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/23/2018
---
# <a name="autoscaling"></a>自动缩放
[!INCLUDE [header](../_includes/header.md)]

自动缩放是动态分配资源以满足性能需求的过程。 当工作量增大时，应用程序可能需要额外的资源来维持所需的性能级别和满足服务级别协议 (SLA)。 当需求降低，不再需要额外的资源时，可以取消分配资源，最大程度地降低成本。

自动缩放可以利用云托管环境的弹性，同时可以降低管理开销。 操作员不必持续监视系统性能，只在需要时才做有关添加或删除资源的决策。

应用程序缩放有两种主要的方式： 

* 垂直缩放，也称为增加和减少，表示改变资源的容量。 例如，可将应用程序移动到更大的 VM 中。 垂直缩放在重新部署时通常会要求系统暂时不可用。 因此，自动化垂直缩放并不常见。
* 水平缩放，也称为扩大和缩小，表示添加或删除资源的实例。 在预配新资源时，应用程序无需中断，可持续运行。 当预配过程完成时，解决方案就已部署在这些额外资源上。 如果需求降低，额外的资源可以完全关闭并解除分配。 

许多基于云的系统（包括 Microsoft Azure）支持自动水平缩放。 本文的余下内容重点介绍水平缩放。

> [!NOTE]
> 自动缩放主要适用于计算资源。 虽然可以水平缩放数据库或消息队列，但这通常涉及非自动的[数据分区][data-partitioning]。
>

## <a name="overview"></a>概述

自动缩放策略通常包括以下部分：

* 位于应用程序、服务和基础结构级别的检测和监视系统。 这些系统可捕获响应时间、队列长度、CPU 利用率和内存使用量等关键指标。
* 根据预定义的阈值或计划来评估这些指标并决定是否缩放的决策逻辑。
* 缩放系统的组件。
* 测试、监视和优化自动缩放策略，以确保它按预期工作。

Azure 提供用于处理常见方案的内置自动缩放机制。 如果某个特定服务或技术没有内置自动缩放功能，或者你有超出其功能的特定自动缩放需求，则可考虑自定义实现。 自定义实现将收集操作和系统指标，分析这些指标，然后相应地缩放资源。

## <a name="configure-autoscaling-for-an-azure-solution"></a>配置 Azure 解决方案的自动缩放

Azure 为大多数计算选项提供内置自动缩放功能。

* 虚拟机使用 [VM 规模集][vm-scale-sets]支持自动缩放，这是以组的形式管理 Azure 虚拟机集的方法。 请参阅[如何使用自动缩放和虚拟机规模集][vm-scale-sets-autoscale]。

* Service Fabric 也通过 VM 规模集来支持自动缩放。 Service Fabric 群集中的每个节点类型会设置为单独的 VM 规模集。 这样，每个节点类型都可以独立地扩大和缩小。 请参阅[使用自动缩放规则扩大和缩小 Service Fabric 群集][service-fabric-autoscale]。

* Azure 应用服务具有内置自动缩放功能。 自动缩放设置应用于应用服务中的所有应用。 请参阅[手动或自动缩放实例计数][app-service-autoscale]。

* Azure 云服务在角色级别具有内置自动缩放功能。 请参阅[如何在门户中为云服务配置自动缩放][cloud-services-autoscale]。

这些计算选项都使用 [Azure Monitor 自动缩放][monitoring]来提供一组通用的自动缩放功能。

* Azure Functions 与以前的计算选项不同，因为无需配置任何自动缩放规则。 相反，当代码正在运行时，Azure Functions 会自动分配计算能力，根据需要进行横向扩展，以处理负载。 有关详细信息，请参阅[为 Azure Functions 选择正确的托管计划][functions-scale]。

最后，自定义自动缩放解决方案有时非常有用。 例如，可使用 Azure 诊断和基于应用程序的指标，以及自定义代码来监视和导出应用程序指标。 然后，可根据这些指标定义自定义规则，并使用资源管理器 REST API 来触发自动缩放。 但是，自定义解决方案并不容易实施，只应在前述方法都无法满足要求时才加以考虑。

如果平台的内置自动缩放功能可以符合要求，就使用此内置功能。 否则，请仔细考虑是否真正需要更复杂的缩放功能。 其他要求的示例包括更高粒度的控制、检测缩放触发事件的其他方式、跨订阅缩放和缩放其他类型的资源。

## <a name="use-azure-monitor-autoscale"></a>使用 Azure Monitor 自动缩放

[Azure Monitor 自动缩放][monitoring]为 VM 规模集、Azure 应用服务和 Azure 云服务提供一套通用的自动缩放功能。 可按计划，也可根据运行时指标（如 CPU 或内存使用率）执行缩放。 示例:

- 在工作日扩大到 10 个实例，在周六和周日缩小到 4 个实例。 
- 如果 CPU 平均使用率在 70% 以上，则扩大一个实例；如果 CPU 使用率低于 50%，则缩小一个实例。
- 如果队列中消息数量超过特定阈值，则扩大一个实例。

有关内置指标列表，请参阅 [Azure Monitor 自动缩放常用指标][autoscale-metrics]。 还可通过使用 Application Insights 来实现自定义指标。 

可通过使用 PowerShell、Azure CLI、Azure 资源管理器模板或 Azure 门户来配置自动缩放。 要实现更细化的控制，请使用 [Azure 资源管理器 REST API](https://msdn.microsoft.com//library/azure/dn790568.aspx)。 [Azure 监视服务管理库](http://www.nuget.org/packages/Microsoft.WindowsAzure.Management.Monitoring)和 [Microsoft Insights 库](https://www.nuget.org/packages/Microsoft.Azure.Insights/)（预览版）是可让用户从不同的资源收集指标以及利用 REST API 执行自动缩放的 SDK。 对于不支持 Azure 资源管理器的资源，或者使用 Azure 云服务时，可以使用服务管理 REST API 进行自动缩放。 在其他所有情况下，请使用 Azure 资源管理器。

使用 Azure 自动缩放时，请注意以下几点：

* 考虑是否可以足够精确地预测应用程序负载，使用计划的自动缩放添加和删除实例以满足需求的预期高峰。 如果不可行，请根据运行时指标使用被动自动缩放，以处理无法预测的需求变化。 通常情况下，可以组合使用这些方法。 例如，如果知道应用程序何时最繁忙，则可以创建一个策略，以根据时间计划添加资源。 这有助于确保容量在需要时可供使用，并且不会在启动新实例时发生延迟。 对于每个计划的规则，请定义在该期间允许被动自动缩放的指标，以确保应用程序能够处理持续但无法预测的需求高峰。
* 通常很难了解指标与容量要求之间的关系，尤其是在最初部署应用程序后。 在一开始多预配一些附加容量，监视并调整自动缩放规则，使容量更接近实际负载的需要。
* 配置自动缩放规则，然后监视应用程序在一段时间内的性能。 如果需要，请使用这种监视的结果来调整系统的缩放方式。 但请记住，自动缩放不是即时起效的过程。 它需要时间来对指标（例如平均 CPU 利用率超过或低于指定的阈值）做出反应。
* 使用基于测得触发器属性（例如 CPU 使用量或队列长度）的检测机制的自动缩放规则使用一段时间内的聚合值而不是即时值来触发自动缩放操作。 默认情况下，聚合是值的平均值。 这可以防止系统反应太快，或导致快速震荡。 这还可以使自动启动的新实例顺利进入运行模式，避免当新实例正在启动时，又发生其他自动缩放操作。 对于 Azure 云服务和 Azure 虚拟机，聚合的默认期限为 45 分钟，指标需要经过这段时间后才为了响应需求高峰而触发自动缩放。 可以使用 SDK 更改聚合期限，但请注意，低于 25 分钟可能导致不可预测的结果（有关详细信息，请参阅 [Auto Scaling Cloud Services on CPU Percentage with the Azure Monitoring Services Management Library](http://rickrainey.com/2013/12/15/auto-scaling-cloud-services-on-cpu-percentage-with-the-windows-azure-monitoring-services-management-library/)（使用 Azure 监视服务管理库根据 CPU 百分比自动缩放云服务））。 对于 Web 应用，平均期限要短得多，这样便可以在平均触发测量值更改约五分钟后提供新实例。
* 如果使用 SDK 而不是门户配置自动缩放，则可以指定更详细的计划，在执行该计划期间，规则将处于活动状态。 还可以创建自己的度量值，并将其与自动缩放规则中的现有度量值一起使用，或单独使用。 例如，建议使用备选计数器，如每秒的请求数或平均内存可用性，或使用测量特定业务流程的自定义计数器。
* 自动缩放 Service Fabric 时，由于群集中的节点类型由后端的 VM 规模集构成，因此需要为每个节点类型设置自动缩放规则。 在设置自动缩放之前请考虑必须具有的节点数。 对于主节点类型所必须具有的最小节点数受所选择的可靠性级别影响。 有关详细信息，请参阅[使用自动缩放规则扩大或缩小 Service Fabric 群集](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-scale-up-down)。
* 可以使用门户将 SQL 数据库实例和队列等资源链接到云服务实例。 这样，便可以更轻松地访问每个链接资源的各个手动和自动缩放配置选项。 有关详细信息，请参阅[如何：将资源链接到云服务](/azure/cloud-services/cloud-services-how-to-manage)。
* 配置多个策略和规则时，它们可能相互冲突。 自动缩放使用以下冲突解决规则来确保始终有足够的实例在运行状态：
  * 向外缩放操作始终优先于向内缩放操作。
  * 当向外缩放操作发生冲突时，使实例数增幅最大的规则优先。
  * 当向内缩放操作发生冲突时，使实例数降幅最小的规则优先。
* 在应用服务环境中，可使用任何辅助池或前端指标来定义自动缩放规则。 有关详细信息，请参阅[自动缩放和应用服务环境](/azure/app-service/app-service-environment-auto-scale)。

## <a name="application-design-considerations"></a>应用程序设计注意事项
自动缩放不是即时见效的解决方案。 只是将资源添加到系统或运行进程的更多实例并不能保证提高系统性能。 设计自动缩放策略时，请注意以下几点：

* 系统必须设计为支持水平缩放。 不要在实例相关性方面做出假设；不要设计需要代码始终在特定的进程实例中运行的解决方案。 水平缩放云服务或网站时，不要假设一系列来自同一源的请求始终路由到同一实例。 出于相同原因，请将服务设计为无状态，以避免需要将一系列来自应用程序的请求始终路由到同一服务实例。 在设计从队列读取并处理消息的服务时，不要假设哪个服务实例处理哪个特定消息。 自动缩放可能会在队列长度增大时启动其他服务实例。 [使用者竞争模式][competing-consumers]说明了如何解决这种情况。
* 如果解决方案实施长时间运行的任务，请将此任务设计为同时支持向外和向内缩放。 如果不保持应有的谨慎，这种任务在系统向内缩放时会阻止进程实例完全关闭；如果进程被强行终止，则可能会丢失数据。 理想的情况是，重构长时间运行的任务并分解处理逻辑，使其以较小且不连续的块执行。 [管道和筛选器模式][pipes-and-filters]提供了有关如何实现此目的的示例。
* 或者，可以实施检查点机制，用于定期记录任务的状态信息，并将此状态保存在运行任务的任何进程实例可以访问的持久性存储中。 这样，如果进程关闭，它所执行的工作可以使用另一个实例，从最后一个检查点继续进行。
* 当后台任务在独立的计算实例（例如云服务托管应用程序的辅助角色）上运行时，可能需要使用不同的缩放策略来缩放应用程序的不同部分。 例如，可能需要部署其他用户界面 (UI) 计算实例而不增加后台计算实例数，或是相反。 如果提供不同级别的服务（例如基本和高级服务包），可能需要使用比基本服务包更主动的高级服务包的计算资源才能符合 SLA。
* 考虑使用 UI 和后台计算实例通信的队列长度作为自动缩放策略的条件。 这可以最好地反映当前负载与后台任务处理容量之间的不平衡或差异。
* 如果自动缩放策略基于度量业务进程（例如每小时订单数或复杂事务的平均执行时间）的计数器，请确保完全了解这些计数器类型的结果与实际计算容量要求之间的关系。 可能需要缩放多个组件或计算单位来应对业务进程计数器的变化。  
* 若要防止系统过度地向外缩放并避免由于运行数千个实例而带来的成本开销，请考虑限制可以自动添加的实例数上限。 大多数自动缩放机制允许指定规则的实例数上限和下限。 此外，如果部署的实例数已达到上限而系统仍然过载，请考虑适当地降低系统提供的功能。
* 请记住，自动缩放可能不是处理工作负荷中突发高峰的最适当机制。 设置并启动新服务实例或将资源添加到系统都需要花费时间，而当这些附加资源可供使用时，高峰需求可能已成为过去。 在这种情况下，限制服务可能更适合。 有关详细信息，请参阅[限制模式][throttling]。
* 相比之下，如果希望在事务量快速波动时有足够的容量处理所有请求，并且成本不是主要考虑因素，那么，请考虑使用激进的自动缩放策略来更快速地启动附加实例。 还可以使用计划的策略在最大负载来临前预先启动足量的实例。
* 自动缩放机制应该监视自动缩放过程，并记录每个自动缩放事件的详细信息（触发的事件、添加或删除了哪些资源，以及时间）。 在创建自定义自动缩放机制时，请确保它包含此功能。 分析信息以帮助度量自动缩放策略的有效性，并根据需要进行优化。 在短时间内使用模式变得明显，以及长期业务拓展或对应用程序的要求变化时，可以进行优化。 如果应用程序达到定义的自动缩放上限，机制可以提醒操作人员，让操作人员手动启动其他资源（如果必要）。 请注意，在这种情况下，操作人员可能还要负责在工作负荷减轻后手动删除这些资源。

## <a name="related-patterns-and-guidance"></a>相关模式和指南
实施自动缩放时，以下模式和指南也可能与方案相关：

* [限制模式][throttling]。 此模式描述当需求增大而对资源产生极大负载时，应用程序如何继续工作并满足 SLA。 限制可与自动缩放配合使用，以避免系统向外缩放时失控。
* [使用者竞争模式][competing-consumers]。 此模式描述如何实施服务实例池，以便处理来自任何应用程序实例的消息。 自动缩放可用于启动和停止服务实例，以符合预期的工作负荷。 此模式可让系统同时处理多个消息，以优化吞吐量、提高伸缩性和可用性，以及平衡工作负荷。
* [监视和诊断](./monitoring.md)。 检测和遥测在收集信息以促成自动缩放过程方面至关重要。


<!-- links -->

[monitoring]: /azure/monitoring-and-diagnostics/monitoring-overview-autoscale
[app-service-autoscale]: /azure/monitoring-and-diagnostics/insights-how-to-scale?toc=%2fazure%2fapp-service-web%2ftoc.json#scaling-based-on-a-pre-set-metric
[app-service-plan]: /azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview
[autoscale-metrics]: /azure/monitoring-and-diagnostics/insights-autoscale-common-metrics
[cloud-services-autoscale]: /azure/cloud-services/cloud-services-how-to-scale-portal
[competing-consumers]: ../patterns/competing-consumers.md
[data-partitioning]: ./data-partitioning.md
[functions-scale]: /azure/azure-functions/functions-scale
[link-resource-to-cloud-service]: /azure/cloud-services/cloud-services-how-to-manage#how-to-link-a-resource-to-a-cloud-service
[pipes-and-filters]: ../patterns/pipes-and-filters.md
[service-fabric-autoscale]: /azure/service-fabric/service-fabric-cluster-scale-up-down
[throttling]: ../patterns/throttling.md
[vm-scale-sets]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vm-scale-sets-autoscale]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview
