---
title: 订阅治理方案与示例 | Microsoft Docs
description: 提供示例，解释如何针对常见方案实施 Azure 订阅监管。
services: azure-resource-manager
documentationcenter: na
author: rdendtler
manager: timlt
editor: tysonn
ms.assetid: e8fbeeb8-d7a1-48af-804f-6fe1a6024bcb
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/03/2017
ms.author: rodend;karlku;tomfitz
ms.openlocfilehash: 4a4b25e6a6e54fd18c8ae21a88cab39143c3bf48
ms.sourcegitcommit: 4ec010846b9b5545c843a32e08293f906e512302
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/17/2018
ms.locfileid: "34299884"
---
# <a name="examples-of-implementing-azure-enterprise-scaffold"></a>实施 Azure 企业基架的示例
本文通过示例介绍企业如何实施 [Azure 企业基架](subscription-governance.md)的建议事项。 其中使用一家虚构公司 Contoso 来演示常见方案的最佳实践。

## <a name="background"></a>背景
Contoso 是一家跨国公司，为各行业的客户提供供应链解决方案， 服务范围从“软件即服务”模型到本地部署的打包模型，应有尽有。  他们在全球开发软件，在印度、美国和加拿大设立了先进的开发中心。

该公司的 ISV 分部划分为多个独立业务单位，管理重要业务领域的产品。 每个业务单位都有自身的开发人员、产品经理和架构师。

企业技术服务 (ETS) 业务单位提供集中式 IT 功能，管理托管各个业务部门应用程序的多个数据中心。 除了管理数据中心以外，ETS 组织还提供和管理集中式协作（如电子邮件和网站）以及网络/电话服务。 针对未配备操作人员的小型业务单位，他们还要管理面向客户的工作负荷。

本文涉及以下人物：

* Dave 是 ETS Azure 管理员。
* Alice 是 Contoso 供应链业务单位的开发总监。

Contoso 需要构建业务线应用和面向客户的应用。 该公司决定在 Azure 上运行这些应用。 Dave 阅读了[出于合规目的监管订阅](subscription-governance.md)一文，现已准备好实施其中的建议。

## <a name="scenario-1-line-of-business-application"></a>方案 1：业务线应用程序
Contoso 正在构建由世界各地的开发人员使用的源代码管理系统 (BitBucket)。  该应用程序使用基础结构即服务 (IaaS) 来托管软件，由 Web 服务器和一个数据库服务器组成。 开发人员需要访问其开发环境中的服务器，但不需要访问 Azure 中的服务器。 Contoso ETS 希望应用程序所有者和团队能够管理该应用程序。 该应用程序只能在 Contoso 的企业网络中使用。 Dave 需要为此应用程序设置订阅。 该订阅将来还要托管其他与开发人员相关的软件。  

### <a name="naming-standards--resource-groups"></a>命名标准和资源组
Dave 需要创建一个订阅，用于支持所有业务单位常用的开发人员工具。 Dave 需要为订阅和资源组（用于应用程序和网络）创建有意义的名称。 他创建了以下订阅和资源组：

| Item | 名称 | 说明 |
| --- | --- | --- |
| 订阅 |Contoso ETS DeveloperTools Production |支持常用的开发人员工具 |
| 资源组 |bitbucket-prod-rg |包含应用程序 Web 服务器和数据库服务器 |
| 资源组 |corenetworks-prod-rg |包含虚拟网络和站点到站点网关连接 |

### <a name="role-based-access-control"></a>基于角色的访问控制
创建订阅后，Dave 想要确保相应的团队和应用程序所有者可以访问其资源。 Dave 认识到每个团队都有不同的要求。 他利用从 Contoso 本地 Active Directory (AD) 同步到 Azure Active Directory 的组，为团队提供适当的访问级别。

Dave 为订阅分配了以下角色：

| 角色 | 已分配到 | 说明 |
| --- | --- | --- |
| [所有者](/azure/role-based-access-control/built-in-roles#owner) |Contoso AD 中的托管 ID |此 ID 是通过 Contoso 的标识管理工具，配合适时使用 (JIT) 访问权限控制的，可确保订阅所有者的访问完全受到审核 |
| [安全读者](/azure/role-based-access-control/built-in-roles#security-reader) |安全与风险管理部门 |此角色允许用户查看 Azure 安全中心及资源状态 |
| [网络参与者](/azure/role-based-access-control/built-in-roles#network-contributor) |网络团队 |此角色允许 Contoso 的网络团队管理站点到站点 VPN 和虚拟网络 |
| *自定义角色* |应用程序所有者 |Dave 创建了一个可授权修改资源组中资源的角色。 有关详细信息，请参阅 [Custom Roles in Azure RBAC](/azure/role-based-access-control/custom-roles)（Azure RBAC 中的自定义角色） |

### <a name="policies"></a>策略
在管理订阅中的资源方面，Dave 提出以下要求：

* 由于开发工具为世界各地的开发人员提供支持，因此他不希望阻止用户在任何区域创建资源。 但是，他需要知道资源是在哪里创建的。
* 另外，他还关心成本。 因此，他想要防止应用程序所有者不必要地创建成本不菲的虚拟机。  
* 由于此应用程序为许多业务单位的开发人员提供服务，因此他想要使用业务单元和应用程序所有者标记每个资源。 使用这些标记，ETS 可向相应的团队计费。

他创建了以下 [Azure 策略](/azure/azure-policy/azure-policy-introduction)：

| 字段 | 效果 | 说明 |
| --- | --- | --- |
| location |audit |审核任何区域的资源创建活动 |
| type |deny |拒绝创建 G 系列虚拟机 |
| 标记 |deny |要求使用应用程序所有者标记 |
| 标记 |deny |要求使用成本中心标记 |
| 标记 |append |将标记名称 **BusinessUnit** 和标记值 **ETS** 追加到所有资源 |

### <a name="resource-tags"></a>资源标记
Dave 知道，若要识别 BitBucket 实现的成本中心，就需要获得有关帐单的具体信息。 此外，Dave 想要知道 ETS 拥有的所有资源。

他向资源组和资源添加了以下[标记](/azure/azure-resource-manager/resource-group-using-tags)。

| 标记名称 | 标记值 |
| --- | --- |
| ApplicationOwner |此应用程序的管理人员姓名 |
| CostCenter |支付 Azure 使用费的组所在的成本中心 |
| BusinessUnit |**ETS**（与订阅关联的业务单位） |

### <a name="core-network"></a>核心网络
Contoso ETS 信息安全与风险管理团队评审了 Dave 规划的将应用程序转移到 Azure 的提案。 他们想要确保该应用程序不会在 Internet 上公开。  将来，Dave 还要将一些开发人员应用转移到 Azure。 这些应用需要公共接口。  为了满足这些要求，他提供了内部和外部虚拟网络以及一个网络安全组来限制访问。

他使用了以下资源：

| 资源类型 | 名称 | 说明 |
| --- | --- | --- |
| 虚拟网络 |internal-vnet |用于 BitBucket 应用程序，通过 ExpressRoute 连接到 Contoso 的企业网络。  子网 (`bitbucket`) 提供具有特定 IP 地址空间的应用程序 |
| 虚拟网络 |external-vnet |用于将来部署的、需要面向公众的终结点的应用程序 |
| 网络安全组 |bitbucket-nsg |只允许通过应用程序所在子网 (`bitbucket`) 端口 443 建立连接，确保最大程度地减少此工作负荷的受攻击面 |

### <a name="resource-locks"></a>资源锁
Dave 认识到，从 Contoso 的企业网络与内部虚拟网络建立的连接必须避免出现任何意料不到的脚本或意外的删除行为。

他创建了以下[资源锁](/azure/azure-resource-manager/resource-group-lock-resources)：

| 锁类型 | 资源 | 说明 |
| --- | --- | --- |
| **CanNotDelete** |internal-vnet |阻止用户删除虚拟网络或子网，但不阻止添加新子网 |

### <a name="azure-automation"></a>Azure 自动化
Dave 没有为此应用程序设置自动化。 虽然他创建了 Azure 自动化帐户，但从来没有用过。

### <a name="azure-security-center"></a>Azure 安全中心
Contoso IT 服务管理部门需要快速识别和处理威胁。 他们还希望了解可能存在哪些问题。  

为了满足这些要求，Dave 启用了 [Azure 安全中心](/azure/security-center/security-center-intro)，并向“安全读者”角色提供了访问权限。

## <a name="scenario-2-customer-facing-app"></a>方案 2：面向客户的应用
供应链业务单位的业务领导发现，他们可以借助多种机会，使用会员卡来提高 Contoso 客户的参与度。 Alice 的团队必须创建此应用程序，他们认定，Azure 能够帮助实现该业务需求。 Alice 与来自 ETS 的 Dave 合作，配置了两个订阅用于开发和运行此应用程序。

### <a name="azure-subscriptions"></a>Azure 订阅
Dave 登录到 Azure 企业门户，发现供应链部门已经存在。  不过，由于此项目是供应链团队在 Azure 中的第一个开发项目，Dave 认识到 Alice 的开发团队需要一个新帐户。  他为她的团队创建了“R&D”帐户，并向 Alice 分配了访问权限。 Alice 通过 Azure 门户登录并创建了两个订阅：分别用于托管开发服务器和托管生产服务器。  创建以下订阅时，她遵循了以前建立的命名标准：

| 订阅用途 | 名称 |
| --- | --- |
| 开发 |Contoso SupplyChain ResearchDevelopment LoyaltyCard Development |
| 生产 |Contoso SupplyChain Operations LoyaltyCard Production |

### <a name="policies"></a>策略
Dave 和 Alice 就此应用程序展开了讨论，发现此应用程序只能为北美区域的客户提供服务。  Alice 及其团队打算使用 Azure 的应用程序服务环境和 Azure SQL 来创建该应用程序。 在开发过程中，他们可能需要创建虚拟机。  Alice 想要确保其开发人员在探讨和检查问题时可以获得所需的资源，而无需求助于 ETS。

针对**开发订阅**，他们创建了以下策略：

| 字段 | 效果 | 说明 |
| --- | --- | --- |
| location |audit |审核任何区域的资源创建活动 |

该策略不会限制用户可在开发环境中创建的 SKU 类型，并且不要求对任何资源组或资源使用标记。

针对**生产订阅**，他们创建了以下策略：

| 字段 | 效果 | 说明 |
| --- | --- | --- |
| location |deny |拒绝在美国数据中心以外创建任何资源 |
| 标记 |deny |要求使用应用程序所有者标记 |
| 标记 |deny |要求使用部门标记 |
| 标记 |append |将指明生产环境的标记追加到每个资源组 |

该策略不会限制用户可在生产环境中创建的 SKU 类型。

### <a name="resource-tags"></a>资源标记
Dave 知道，他需要获得具体的信息才能识别正确的业务组来完成计费和确定所有权。 他为资源组和资源定义了资源标记。

| 标记名称 | 标记值 |
| --- | --- |
| ApplicationOwner |此应用程序的管理人员姓名 |
| 系 |支付 Azure 使用费的组所在的成本中心 |
| EnvironmentType |Production（尽管订阅的名称中已经包含 Production，但如果添加此标记，在门户或帐单中查找资源时可以方便识别） |

### <a name="core-networks"></a>核心网络
Contoso ETS 信息安全与风险管理团队评审了 Dave 规划的将应用程序转移到 Azure 的提案。 他们想要确保会员卡应用程序在外围网络中受到适当隔离和保护。  为了满足此要求，Dave 和 Alice 创建了外部虚拟网络和网络安全组，用于从 Contoso 企业网络中隔离会员卡应用程序。  

针对**开发订阅**，他们创建了：

| 资源类型 | 名称 | 说明 |
| --- | --- | --- |
| 虚拟网络 |internal-vnet |为 Contoso 会员卡开发环境提供服务，通过 ExpressRoute 连接到 Contoso 的企业网络 |

针对**生产订阅**，他们创建了：

| 资源类型 | 名称 | 说明 |
| --- | --- | --- |
| 虚拟网络 |external-vnet |托管会员卡应用程序，不直接连接到 Contoso 的 ExpressRoute。 代码通过源代码管理系统直接推送到 PaaS 服务 |
| 网络安全组 |loyaltycard-nsg |只允许通过 TCP 443 接收入站通信，确保最大程度地减少此工作负荷的受攻击面。  另外，Contoso 正在调研是否要 Web 应用程序防火墙来增强保护 |

### <a name="resource-locks"></a>资源锁
Dave 和 Alice 经过商讨，决定对环境中的某些资源添加资源锁，防止在推送不当代码的过程中出现意外删除。

他们创建了以下锁：

| 锁类型 | 资源 | 说明 |
| --- | --- | --- |
| **CanNotDelete** |external-vnet |阻止用户删除虚拟网络或子网。 该锁不会阻止添加新子网 |

### <a name="azure-automation"></a>Azure 自动化
Alice 及其开发团队创建了大量的 Runbook 来管理此应用程序的环境。 使用 Runbook 可以添加/删除应用程序节点及其他 DevOps 任务。

为了使用这些 Runbook，它们启用了[自动化](/azure/automation/automation-intro)。

### <a name="azure-security-center"></a>Azure 安全中心
Contoso IT 服务管理部门需要快速识别和处理威胁。 他们还希望了解可能存在哪些问题。  

为了满足这些要求，Dave 启用了 Azure 安全中心。 他确保 Azure 安全中心正在监视这些资源，并向 DevOps 和安全团队提供了访问权限。

## <a name="next-steps"></a>后续步骤
* 若要了解如何创建资源管理器模板，请参阅[创建 Azure 资源管理器模板的最佳实践](/azure/azure-resource-manager/resource-manager-template-best-practices)。
