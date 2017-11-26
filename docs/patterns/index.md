---
title: "云设计模式"
description: "Microsoft Azure 的云设计模式"
keywords: Azure
ms.openlocfilehash: bf9fb2555f5c80cab9e4616ba52155bf1284d26f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="cloud-design-patterns"></a>云设计模式

这些设计模式可用于在云中构建可靠且可缩放的安全应用程序。

每种模式描述了该模式解决的问题、有关应用该模式的注意事项，以及基于 Microsoft Azure 的示例。 大多数模式都包含了代码示例或代码片段，演示如何在 Azure 中实现该模式。 但是，无论托管在 Azure 还是其他云平台中，大多数模式都与任一分布式系统相关。

## <a name="challenges-in-cloud-development"></a>云中开发的难题

<table>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/availability.md"><img src="_images/category/availability.svg" alt="Availability" /></a></td>
    <td>
        <h3><a href="./category/availability.md">可用性</a></h3>
        <p>可用性指系统正常工作时间所占的比例，通常以运行时间百分比度量。 可用性受系统错误、基础结构问题、恶意攻击和系统负载的影响。  云应用程序通常向用户提供服务级别协议 (SLA)，因此，它们在设计上必须能够最大程度地保持可用性。</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/data-management.md"><img src="_images/category/data-management.svg" alt="Data Management" /></a></td>
    <td>
        <h3><a href="./category/data-management.md">数据管理</a></h3>
        <p>数据管理是云应用程序的关键要素，影响大部分质量属性。 出于性能、可伸缩性或可用性方面的原因，数据通常托管在不同的位置并跨多个服务器，这可能会带来一系列的挑战。 例如，必须保持数据一致性，通常需要将不同位置的数据进行同步。</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/design-implementation.md"><img src="_images/category/design-implementation.svg" alt="Design and Implementation" /></a></td>
    <td>
        <h3><a href="./category/design-implementation.md">设计和实施</a></h3>
        <p>合理的设计包括很多因素（例如组件设计和部署中的一致性与连贯性）、可简化管理和部署的可维护性，以及可允许在其他应用程序和其他方案中使用的组件和子系统的可重用性。 在设计和实施阶段做出的决策对质量和云托管的应用程序与服务的总拥有成本有着巨大影响。</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/messaging.md"><img src="_images/category/messaging.svg" alt="Messaging" /></a></td>
    <td>
        <h3><a href="./category/messaging.md">消息传送</a></h3>
        <p>云应用程序的分布性要求消息基础结构在理想情况下能以松散耦合的方式连接组件和服务，从而将可伸缩性最大化。 异步消息受到广泛使用并提供了诸多好处，但也带来了许多挑战，如消息排序、有害消息管理和幂等性等。</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/management-monitoring.md"><img src="_images/category/management-monitoring.svg" alt="Management and Monitoring" /></a></td>
    <td>
        <h3><a href="./category/management-monitoring.md">管理和监视</a></h3>
        <p>云应用程序在远程数据中心内运行，在此中心内，无法完全控制基础架构，或者在某些情况下无法控制操作系统。 与本地部署相比，管理和监视难度更大。 应用程序必须公开运行时信息，以便管理员和操作员管理和监视系统，支持不断变化的业务要求和定制，而无需停止或重新部署应用程序。</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/performance-scalability.md"><img src="_images/category/performance-scalability.svg" alt="Performance and Scalability" /></a></td>
    <td>
        <h3><a href="./category/performance-scalability.md">性能和可伸缩性</a></h3>
        <p>性能是指系统在给定的时间间隔内执行任何操作的响应能力，可伸缩性则是系统能够应对负载增大且不影响性能或随时增加可用资源的能力。 云应用程序往往会遇到可变工作负载和活动高峰。 预测这些变化（尤其是在多租户方案中）几乎是不可能的。 应用程序应该能够在限制范围内扩展以满足需求高峰，并在需求减少时缩减。 可伸缩性不仅涉及计算实例，而且还涉及其他要素，例如数据存储、消息传送基础结构，等等。</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/resiliency.md"><img src="_images/category/resiliency.svg" alt="Resiliency" /></a></td>
    <td>
        <h3><a href="./category/resiliency.md">复原能力</a></h3>
        <p>复原能力是指系统能够在发生故障后进行恰当处理，然后恢复正常。 云托管的性质包括：应用程序通常是多租户的，它们使用共享的平台服务，会竞争资源和带宽，通过 Internet 通信，并在市售硬件上运行。正因为这种性质，同时出现暂时性故障和较持久性故障的可能性就会增大。 为了保持复原能力，有必要检测故障并快速高效地进行恢复。</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/security.md"><img src="_images/category/security.svg" alt="Security" /></a></td>
    <td>
        <h3><a href="./category/security.md">安全性</a></h3>
        <p>安全性是防止超出设计使用范围的恶意或意外操作，并防止泄露或丢失信息的系统能力。 云应用程序暴露在受信任本地边界之外的 Internet 上，通常向公众开放，并可能服务于不受信任的用户。 应用程序的设计和部署必须防范它们受到恶意攻击，将访问权限限制给经过批准的用户，并保护敏感数据。</p>
    </td>
</tr>
</table>

## <a name="catalog-of-patterns"></a>模式目录

| 模式 | 摘要 |
| ------- | ------- |
| [代表](./ambassador.md) | 创建代表客户服务或应用程序发送网络请求的帮助程序服务。 |
| [防损层](./anti-corruption-layer.md) | 在现代应用程序与传统系统之间实施外观或适配器层。 |
| [用于前端的后端](./backends-for-frontends.md) | 创建单独的后端服务，供特定的前端应用程序或接口使用。 |
| [隔层](./bulkhead.md) | 将应用程序的元素隔离到池中，这样，如果一个元素发生失败，其他元素可继续工作。 |
| [缓存端](./cache-aside.md) | 将数据按需从数据存储加载到缓存中 |
| [断路器](./circuit-breaker.md) | 连接到远程服务或资源时处理故障，此类故障所需修复时间不定。 |
| [CQRS](./cqrs.md) | 使用独立接口将读取数据的操作与更新数据的操作分离。 |
| [补偿事务](./compensating-transaction.md) | 撤销一系列会共同定义最终一致操作的工作。 |
| [竞争性使用者](./competing-consumers.md) | 使多个并发使用者能够处理同一消息通道上收到的消息。 |
| [计算资源合并](./compute-resource-consolidation.md) | 将多个任务或操作合并到单个计算单元 |
| [事件溯源](./event-sourcing.md) | 使用只追加存储来记录描述域中数据采取的操作的完整系列事件。 |
| [外部配置存储](./external-configuration-store.md) | 将配置信息从应用程序部署包移出，移到一个集中的位置。 |
| [联合标识](./federated-identity.md) | 将身份验证委托给外部标识提供者。 |
| [守护程序](./gatekeeper.md) | 通过使用专用的主机实例保护应用程序和服务，该实例用于充当客户端和应用程序或服务之间的中转站、验证和整理请求，并在它们之间传递请求和数据。 |
| [网关聚合](./gateway-aggregation.md) | 使用网关可将多个单独请求聚合成一个请求。 |
| [网关卸载](./gateway-offloading.md) | 将共享或专用服务功能卸载到网关代理。 |
| [网关路由](./gateway-routing.md) | 使用单个终结点将请求路由到多个服务。 |
| [运行状况终结点监视](./health-endpoint-monitoring.md) | 在应用程序中实施可让外部工具通过公开终结点定期访问的功能检查。 |
| [索引表](./index-table.md) | 基于数据存储中经常由查询引用的字段创建索引。 |
| [领导选拔](./leader-election.md) | 通过选拔一个实例作为领导来负责管理其他实例，协调分布式应用程序中协作性任务实例集合所执行的操作。 |
| [具体化视图](./materialized-view.md) | 当未针对所需的查询操作完美设置数据的格式时，在一个或多个数据存储中基于数据生成预填充的视图。 |
| [管道和筛选器](./pipes-and-filters.md) | 将一个执行复杂处理的任务分解为一系列可重复使用的单个元素。 |
| [优先级队列](./priority-queue.md) | 为发送到服务的请求确定优先级，以便高优先级请求能够得到比低优先级请求更快速地接收和处理。 |
| [基于队列的负载调控](./queue-based-load-leveling.md) | 使用队列在任务与所调用的服务之间充当缓冲，从而缓解间歇性负载过大现象。 |
| [重试](./retry.md) | 当应用程序尝试连接到服务或网络资源时，使应用程序能够通过以透明方式重试先前失败的操作来处理预期的临时故障。 |
| [计划程序代理监督程序](./scheduler-agent-supervisor.md) | 跨一组分布式服务和其他远程资源协调一组操作。 |
| [分片](./sharding.md) | 将数据存储划分为一组水平分区或分片。 |
| [Sidecar](./sidecar.md) | 将应用程序的组件部署到单独的进程或容器中，以提供隔离和封装。 |
| [静态内容托管](./static-content-hosting.md) | 将静态内容部署到基于云的存储服务，再由后者将它们直接传送给客户端。 |
| [Strangler](./strangler.md) | 通过将特定的功能片断逐渐取代为新的应用程序和服务，逐步迁移传统系统。 |
| [限制](./throttling.md) | 控制应用程序实例、单个租户或整个服务对资源的消耗。 |
| [附属密钥](./valet-key.md) | 使用令牌或密钥为客户端提供对特定资源或服务的受限直接访问权限。 |