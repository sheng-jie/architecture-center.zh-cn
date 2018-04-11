---
title: 软件质量的要素
description: 介绍软件质量的五大构成要素：可伸缩性、可用性、复原能力、管理和安全性。
author: MikeWasson
ms.openlocfilehash: 1d5e30602cafa0d39f92de3101974e77ae258595
ms.sourcegitcommit: a7aae13569e165d4e768ce0aaaac154ba612934f
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/30/2018
---
# <a name="pillars-of-software-quality"></a>软件质量的构成要素 

一个成功的云应用程序应注重软件质量的五大构成要素：可伸缩性、可用性、复原能力、管理和安全性。

| 构成要素 | 说明 |
|--------|-------------|
| 可伸缩性 | 系统处理增加的负载的能力。 |
| 可用性 | 系统正常工作时间所占的比例。 |
| 复原能力 | 系统从故障中恢复并继续正常运行的能力。 |
| 管理 | 让系统在生产环境中持续运行的操作过程。 |
| “安全” | 保护应用程序和数据免受威胁。 |

## <a name="scalability"></a>可伸缩性

可伸缩性是指系统处理增加的负载的能力。 应用程序可通过两种主要方式进行扩展。 垂直扩展（*纵向*扩展）指增加资源的容量，例如通过使用更大的 VM。 水平扩展（*横向*扩展）指添加资源的新实例，比如 VM 或数据库副本。 

水平扩展相较垂直扩展具有明显优势：

- 真正的云规模。 可将应用程序设计为在数百个甚至数千个节点上运行，其规模是在单个节点上无法达到的。
- 水平扩展具有弹性。 如果负载增加，可以添加更多实例；在较安静的时间段，则可以删除实例。
- 可以按计划或为响应负载变化，自动触发横向扩展。 
- 横向扩展可能比纵向扩展更便宜。 运行多个小型 VM 比运行单个大型 VM 的成本更低。 
- 水平扩展还可通过添加冗余提高复原能力。 如果某个实例出现故障，应用程序将继续运行。

垂直扩展的一个优点是，扩展时不必对应用程序进行任何更改。 但会在某个时候达到极限，即，再也无法纵向扩展。 这时，任何进一步的扩展都只能是水平扩展。 

必须将水平扩展设计到系统中。 例如，可通过将 VM 放在负载均衡器后面来横向扩展 VM。 但池中的每个 VM 都必须能够处理任何客户端请求，因此应用程序必须无状态或将状态存储在外部（例如，在分布式缓存中）。 托管 PaaS 服务通常内置水平扩展和自动扩展。 能轻松扩展这些服务是使用 PaaS 服务的主要优点。

不过，只添加更多实例并不意味着就扩展了应用程序。 它可能只是将瓶颈推到了其他地方。 例如，如果扩展 Web 前端以处理更多客户端请求，则可能在数据库中触发锁争用。 然后，你就得考虑其他对策，比如乐观并发或数据分区，以提高数据库的吞吐量。

始终执行性能和负载测试以发现这些潜在瓶颈。 系统的有状态部分（如数据库）是导致瓶颈最常见的原因，因此在设计水平扩展时需谨慎。 解决一个瓶颈可能会暴露其他位置的其他瓶颈。

使用[可伸缩性清单][scalability-checklist]从可伸缩性角度审查你的设计。

### <a name="scalability-guidance"></a>可伸缩性指南

- [可伸缩性和性能的设计模式][scalability-patterns]
- 最佳做法：[自动缩放][autoscale]、[后台作业][background-jobs]、[缓存][caching]、[CDN][cdn]、[数据分区][data-partitioning]

## <a name="availability"></a>可用性

可用性指系统正常工作时间所占的比例。 通常通过运行时间百分比衡量。 应用程序错误、基础结构问题和系统负载都会降低可用性。 

云应用程序应具有一个服务级别目标 (SLO)，以明确定义预期的可用性以及如何衡量可用性。 定义可用性时，请查看关键路径。 Web 前端可能能够处理客户端请求，但如果每个事务都因无法连接到数据库而失败，用户将无法使用该应用程序。 

通常以“9s”的方式描述可用性 &mdash; 例如，“四个 9”意味着 99.99% 的运行时间。 下表展示不同可用性级别的潜在累积故障时间。

| 运行时间百分比 | 每周故障时间 | 每月故障时间 | 每年故障时间 |
|----------|-------------------|--------------------|-------------------|
| 99% | 1.68 小时 | 7.2 小时 | 3.65 天 |
| 99.9% | 10 分钟 | 43.2 分钟 | 8.76 小时 |
| 99.95% | 5 分钟 | 21.6 分钟 | 4.38 小时 |
| 99.99% | 1 分钟 | 4.32 分钟 | 52.56 分钟 |
| 99.999% | 6 秒 | 26 秒 | 5.26 分钟 |

请注意，99% 的运行时间意味着每周将近 2 小时的服务中断时间。 对于许多应用程序，特别是面向使用者的应用程序，这是一个不可接受的 SLO。 另一方面，五个 9 (99.999%) 意味着一*年*内的故障时间不能超过 5 分钟。 以这么快的速度检测一次中断都很难做到，更别说解决问题了。 若要获取非常高的可用性（99.99% 或更高），不能依靠手动干预从故障中恢复。 应用程序必须自我诊断和自我修复，此时复原能力就变得至关重要。

在 Azure 中，服务级别协议 (SLA) 描述 Microsoft 关于运行时间和连接方面的承诺。 如果针对特定服务的 SLA 为 99.95%，则意味着该服务应该在 99.95% 的时间内可用。

应用程序通常依赖于多个服务。 一般来说，任一服务发生故障的概率是独立的。 例如，假设应用程序依赖于两个服务，每个服务的 SLA 都为 99.9%。 那么，这两个服务的复合 SLA 为 99.9% &times; 99.9% &asymp; 99.8%，或略小于单独的每个服务。 

使用[可用性清单][availability-checklist]从可用性角度审查你的设计。

### <a name="availability-guidance"></a>可用性指南

- [可用性的设计模式][availability-patterns]
- 最佳做法：[自动缩放][autoscale]、[后台作业][background-jobs]

## <a name="resiliency"></a>复原能力

复原能力是指系统从故障中恢复并继续正常运行的能力。 复原能力的目标是在故障发生后将应用程序恢复到可完全正常运行的状态。 复原能力与可用性密切相关。

传统应用程序开发一直将焦点放在如何缩短平均故障间隔时间 (MTBF) 上， 并尝试各种办法防止系统出现故障。 在云计算中，必须采用不同的思维方式，原因如下：

- 分布式系统很复杂，一个点的故障可能在整个系统中级联。
- 云环境通过使用商用硬件保持低成本，因此必须预料到偶尔的硬件故障。 
- 应用程序通常依赖于外部服务，这些服务可能会变得暂时不可用或限制大量用户。 
- 现在的用户都希望应用程序能够全天候可用，永不下线。

所有这些因素都意味着设计云应用程序时必须预料到偶发故障并从中恢复。 Azure 已向平台内置许多复原功能。 例如， 

- Azure 存储、SQL 数据库和 Cosmos DB 都在区域内以及跨区域提供内置数据复制。
- Azure 托管磁盘自动放置在不同的存储缩放单位，以限制硬件故障的影响。
- 可用性集中的 VM 分布在多个容错域。 容错域是指一组共享公共电源和网络交换机的 VM。 跨容错域分布 VM 可限制物理硬件故障、网络中断或断电的影响。

话虽如此，你仍需构建应用程序的复原能力。 复原策略可应用于体系结构的所有级别。 有些缓解措施本质上更具战术意义 &mdash; 例如，在暂时性网络故障后重试远程调用。 其他缓解措施则更具战略意义，比如将整个应用程序故障转移到次要区域。 战术性缓解措施可以带来很大变化。 整个区域都发生中断的情况很少见，像网络拥塞这样的暂时性问题则更常见 &mdash; 因此先锁定这些问题。 正确的监视和诊断也很重要，它们都能检测到正在发生的故障并找到根本原因。

设计可复原的应用程序时，必须了解可用性要求。 可以接受多长的故障时间？ 这在一定程度上取决于成本。 潜在的停机会给业务造成多大的损失？ 使应用程序保持高可用性需要投入多少资金？

使用[复原能力清单][resiliency-checklist]从复原能力角度审查你的设计。

### <a name="resiliency-guidance"></a>复原指南

- [设计适用于 Azure 的弹性应用程序][resiliency]
- [复原能力的设计模式][resiliency-patterns]
- 最佳做法：[暂时性错误处理][transient-fault-handling]、[特定服务重试指南][retry-service-specific]

## <a name="management-and-devops"></a>管理和 DevOps

此构成要素涵盖让应用程序在生产环境中持续运行的操作过程。

部署必须可靠且可预测。 它们应实现自动化，以减少人为失误的可能性。 它们应当是一个快速、例行的过程，这样就不会拖慢新功能或 bug 修复的发布。 如果更新出现问题，你必须能够快速回滚或前滚，这一点也同样重要。

监视和诊断至关重要。 云应用程序在远程数据中心内运行，在此中心内，无法完全控制基础结构，或者在某些情况下无法控制操作系统。 在大型应用程序中，不可能登录到 VM 来解决问题或仔细查看日志文件。 使用 PaaS 服务时，可能根本就没有可登录的专用 VM。 通过监视和诊断，你可以深入了解系统，以便知道故障在何时及何处出现。 所有系统都必须可观测。 可使用常见的一致日志记录架构，以便跨系统关联事件。

监视和诊断过程包含多个不同的阶段：

- 检测。 根据应用程序日志、Web 服务器日志、Azure 平台内置的诊断以及其他来源生成原始数据。
- 收集和存储。 将数据整合到一个位置。
- 分析和诊断。 用于解决问题，查看总体运行状况。
- 可视化和警报。 使用遥测数据发现趋势或向运营团队发出警报。

使用 [DevOps 清单][devops-checklist]从管理和 DevOps 角度审查你的设计。

### <a name="management-and-devops-guidance"></a>管理和 DevOps 指南

- [管理和监视的设计模式][management-patterns]
- 最佳做法：[监视和诊断][monitoring]

## <a name="security"></a>“安全”

你必须考虑从设计和实现到部署和操作的整个应用程序生命周期的安全性。 Azure 平台会提供保护以应对各种威胁，如网络入侵和 DDoS 攻击。 但你仍需在应用程序和 DevOps 过程中构建安全性。

下面是一些需要考虑的较广泛的安全领域。 

### <a name="identity-management"></a>身份管理

请考虑使用 Azure Active Directory (Azure AD) 对用户进行身份验证和授权。 Azure AD 是一项完全托管的标识和访问管理服务。 该服务可用于创建仅存在于 Azure 的域，或与本地 Active Directory 标识集成。 Azure AD 还与 Office365、Dynamics CRM Online 和许多第三方 SaaS 应用程序集成。 对于面向使用者的应用程序，Azure Active Directory B2C 允许用户使用其现有社交帐户（如 Facebook、Google 或 LinkedIn）进行身份验证，或者创建由 Azure AD 管理的新用户帐户。

若要将本地 Active Directory 环境与 Azure 网络集成，可通过多种方法实现，具体视你的要求而定。 有关详细信息，请参阅我们的[标识管理][identity-ref-arch]参考体系结构。

### <a name="protecting-your-infrastructure"></a>保护基础结构 

控制对已部署的 Azure 资源的访问。 每个 Azure 订阅都与某个 Azure AD 租户存在[信任关系][ad-subscriptions]。 可使用[基于角色的访问控制][rbac] (RBAC) 向组织内的用户授予对 Azure 资源的正确权限。 通过向用户或组分配 RBAC 角色，授予对特定范围的访问权限。 该范围可以是订阅、资源组或单个资源。 [审核][resource-manager-auditing]对基础结构的所有更改。 

### <a name="application-security"></a>应用程序安全性

一般来说，应用程序开发的安全性最佳做法在云端仍然适用。 其中包括随处使用 SSL、防止 CSRF 和 XSS 攻击、阻止 SQL 注入攻击等等。 

云应用程序通常使用具有访问密钥的托管服务。 绝不要将这些服务签入源控件中。 请考虑将应用程序密码存储到 Azure Key Vault 中。

### <a name="data-sovereignty-and-encryption"></a>数据自主性和加密

使用 Azure 的高可用性时，确保数据一直位于正确的地缘政治区域中。 Azure 的异地复制存储采用了同一地缘政治区域中的[配对区域][paired-region]这一概念。 

使用 Key Vault 保护加密密钥和密码。 通过使用 Key Vault，可以利用受硬件安全模块 (HSM) 保护的密钥来加密密钥和密码。 许多 Azure 存储和 DB 服务支持静态数据加密，包括 [Azure 存储][storage-encryption]、[Azure SQL 数据库][sql-db-encryption]、[Azure SQL 数据仓库][data-warehouse-encryption]和 [Cosmos DB][cosmosdb-encryption]。

### <a name="security-resources"></a>安全性资源

- [Azure 安全中心][security-center]为 Azure 订阅提供集成的安全监视和策略管理。 
- [Azure 安全文档][security-documentation]
- [Microsoft 信任中心][trust-center]



<!-- links -->

[dr-guidance]: ../resiliency/disaster-recovery-azure-applications.md
[identity-ref-arch]: ../reference-architectures/identity/index.md
[resiliency]: ../resiliency/index.md

[ad-subscriptions]: /azure/active-directory/active-directory-how-subscriptions-associated-directory
[data-warehouse-encryption]: /azure/data-lake-store/data-lake-store-security-overview#data-protection
[cosmosdb-encryption]: /azure/cosmos-db/database-security
[rbac]: /azure/active-directory/role-based-access-control-what-is
[paired-region]: /azure/best-practices-availability-paired-regions
[resource-manager-auditing]: /azure/azure-resource-manager/resource-group-audit
[security-blog]: https://azure.microsoft.com/blog/tag/security/
[security-center]: https://azure.microsoft.com/services/security-center/
[security-documentation]: /azure/security/
[sql-db-encryption]: /azure/sql-database/sql-database-always-encrypted-azure-key-vault
[storage-encryption]: /azure/storage/storage-service-encryption
[trust-center]: https://azure.microsoft.com/support/trust-center/
 

<!-- patterns -->
[availability-patterns]: ../patterns/category/availability.md
[management-patterns]: ../patterns/category/management-monitoring.md
[resiliency-patterns]: ../patterns/category/resiliency.md
[scalability-patterns]: ../patterns/category/performance-scalability.md


<!-- practices -->
[autoscale]: ../best-practices/auto-scaling.md
[background-jobs]: ../best-practices/background-jobs.md
[caching]: ../best-practices/caching.md
[cdn]: ../best-practices/cdn.md
[data-partitioning]: ../best-practices/data-partitioning.md
[monitoring]: ../best-practices/monitoring.md
[retry-service-specific]: ../best-practices/retry-service-specific.md
[transient-fault-handling]: ../best-practices/transient-faults.md


<!-- checklist -->
[availability-checklist]: ../checklist/availability.md
[devops-checklist]: ../checklist/dev-ops.md
[resiliency-checklist]: ../checklist/resiliency.md
[scalability-checklist]: ../checklist/scalability.md
