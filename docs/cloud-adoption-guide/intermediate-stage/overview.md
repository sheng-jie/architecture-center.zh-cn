---
title: 采用 Azure：中间阶段
description: 介绍有关企业采用 Azure 所要掌握的中级知识
author: petertay
ms.openlocfilehash: 39b98595dd615ba1aa36921e48a0b23797bebaa0
ms.sourcegitcommit: b3d74d8a89b2224fc796ce0e89cea447af43a0d4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/11/2018
ms.locfileid: "35291049"
---
# <a name="azure-cloud-adoption-guide-intermediate-overview"></a>Azure 云采用指南：中间阶段概述

在基本采用阶段，我们已了解 Azure 资源调控的基本概念。 基本阶段旨在让客户对 Azure 采用旅程有一个入门性的了解，其中逐步讲解了如何为单个小型团队部署简单的工作负荷。 实际上，拥有多个团队的大型组织会同时处理许多不同的工作负荷。 可以料想到，简单的调控模型并不足以管理更多的复杂组织方案和开发方案。

Azure 采用计划的中间阶段侧重于针对使用多个新 Azure 开发工作负荷的多个团队设计调控模型。  

本指南所述的此阶段面向组织中的以下角色：
- *财务：* 提交到 Azure 的财务信息的所有者，负责制定用于跟踪资源消耗成本（包括计费和费用分摊）的策略与过程。
- *中心 IT：* 负责调控组织的云资源，包括资源管理和访问权限，以及工作负荷运行状况和监视。
- *共享基础结构所有者*：负责在本地与云之间建立网络连接的技术角色。
- *安全运营*：负责实施所需的安全策略，将本地安全边界扩展为涵盖 Azure。 此外，还可能拥有 Azure 中用于存储机密的安全基础结构。
- *工作负荷所有者：* 负责将工作负荷发布到 Azure。 根据组织开发团队的结构，此角色可能是开发主管、程序管理主管或构建工程主管。 发布过程的部分工作可能包括将资源部署到 Azure。
  - *工作负荷参与者：* 负责参与向 Azure 发布工作负荷的工作。 可能需要对 Azure 资源拥有读取访问权限，以便执行性能监视或优化。 不需要拥有创建、更新或删除资源的权限。

## <a name="section-1-azure-concepts-for-multiple-workloads-and-multiple-teams"></a>第 1 部分：有关多个工作负荷和多个团队的 Azure 概念

在基本采用阶段，我们已了解一些有关 Azure 内部结构的基础知识，以及如何创建、读取、更新和删除资源。 我们还了解了标识，以及 Azure 只信任 Azure Active Directory (AD) 对需要访问这些资源的用户进行身份验证和授权。

此外，我们已开始了解如何配置 Azure 的调控工具来管理组织对 Azure 资源的使用。 在基本阶段，我们探讨了如何调控单个团队访问部署简单工作负荷时所需的资源。 实际上，组织往往由多个团队构成，这些团队会同时处理多个工作负荷。 

在开始之前，让我们了解术语“工作负荷”的真正含义。 该术语通常定义一个任意的功能单元，例如应用程序或服务。 我们可以从部署到服务器和其他任何所需服务（例如数据库）的代码项目角度来考虑工作负荷。 对于本地应用程序或服务而言，这种定义非常有效；但在云中，我们需要对其进行延伸。 

在云中，工作负荷不仅涵盖所有项目，而且还包括云资源。 由于存在所谓的**基础结构即代码**概念，我们将云资源包含为定义的一部分。 如“Azure 的工作原理”说明文章中所述，Azure 中的资源由业务流程协调程序服务部署。 业务流程协调程序服务通过 Web API 公开此功能，可以使用 Powershell 等工具、Azure 命令行接口 (CLI) 和 Azure 门户来调用此 Web API。 这意味着，我们可以在机器可读的文件中指定资源，并将该文件连同与应用程序关联的代码项目一起存储。

这样，我们便可以从代码项目和所需云资源的角度来定义工作负荷，从而进一步**隔离**工作负荷。 可以根据资源的组织方式、网络拓扑或其他属性隔离工作负荷。 隔离工作负荷的目的是将工作负荷的特定资源关联到某个团队，使该团队能够独立管理这些资源的所有方面。 这样，多个团队便可以在 Azure 中共享资源管理服务，同时防止意外删除或修改对方的资源。

这种隔离还成就了另一个称作 [DevOps](https://azure.microsoft.com/solutions/devops/) 的概念。 DevOps 包括软件开发实践（例如上述软件开发和 IT 运营），但会尽量添加更多的自动化功能。 DevOps 的原理之一称为持续集成和持续交付 (CI/CD)。 持续集成是指每当开发人员提交代码更改时运行的自动生成过程，持续交付是指将此代码部署到各种**环境**（例如，部署到**开发环境**进行测试，或部署到**生产环境**以完成最终部署）的自动化过程。

## <a name="section-2-governance-design-for-multiple-teams-and-multiple-workloads"></a>第2 部分：针对多个团队和多个工作负荷的调控设计

Azure 云采用指南的[基本阶段](/azure/architecture/cloud-adoption-guide/adoption-intro/overview)已介绍云调控的概念。 其中介绍了如何针对使用单个工作负荷的单个团队设计简单的调控模型。 

在中间阶段，[调控设计指南](governance-design-guide.md)在基本概念的基础上做了延伸，添加了多个团队、多个工作负荷和多个环境。 完成本文档中的示例后，可以运用设计原理来设计和实施组织的调控模型。

## <a name="section-3-implementing-a-resource-management-model"></a>第 3 部分：实施资源管理模型

组织的云调控模型代表 Azure 资源访问管理工具、人员和定义的访问管理规则之间的交集。 调控设计指南已介绍有关用于调控对 Azure 资源的访问的多个不同模型。 现在，让我们执行所需的每个步骤，针对设计指南中的每个**共享基础结构**、**生产**和**开发**环境，实施包含一个订阅的资源管理模型。 我们将为所有三个环境创建一个**订阅所有者**。 每个工作负荷将在某个**资源组**中隔离，该资源组包含具有**参与者**角色的**工作负荷所有者**。

> [!NOTE]
> 请阅读[了解 Azure 中的资源访问][understand-resource-access-in-azure]，详细了解 Azure 帐户与订阅之间的关系。 

执行以下步骤:

1. 如果组织没有 [Azure 帐户](/azure/active-directory/sign-up-organization)，请创建一个帐户。 注册 Azure 帐户的人员将成为 Azure 帐户管理员，组织的领导层必须选择一个充当此角色的人员。 此人将会负责：
  * 创建订阅，以及
  * 创建和管理用于存储这些订阅的用户标识的 [Azure Active Directory (AD)](/azure/active-directory/active-directory-whatis) 租户。    
2. 组织领导团队确定哪个人员承担以下责任：
  * 管理用户标识；创建组织的 Azure 帐户时，默认会创建 [Azure AD 租户](/azure/active-directory/develop/active-directory-howto-tenant)，并且默认会将帐户管理员添加为 [Azure AD 全局管理员](/azure/active-directory/active-directory-assign-admin-roles-azure-portal#details-about-the-global-administrator-role)。 组织可以选择另一个用户来管理用户标识，只需[将 Azure AD 全局管理员角色分配到该用户](/azure/active-directory/active-directory-users-assign-role-azure-portal)即可。 
  * 订阅，表示这些用户将负责：
    * 管理与该订阅中的资源用量相关的成本
    * 实施和维护用于资源访问的最低权限模型，以及
    * 跟踪服务限制。
  * 共享基础结构服务（如果组织决定使用此模型），表示此用户将负责：
    * 建立本地到 Azure 的网络连接，以及 
    * 通过虚拟网络对等互连拥有 Azure 中的网络连接。
  * 工作负荷所有者。 
3. Azure AD 全局管理员为以下人员[创建新用户帐户](/azure/active-directory/add-users-azure-active-directory)：
  * 与每个环境关联的每个订阅的**订阅所有者**。 请注意，仅当订阅**服务管理员**的任务不包括管理每个订阅/环境的资源访问权限时，才需要创建帐户。
  * **网络操作用户**，以及
  * **工作负荷所有者**。
4. Azure 帐户管理员可以使用 [Azure 帐户门户](https://account.azure.com)创建以下三个订阅：
  * **共享基础结构**环境的订阅；
  * **生产**环境的订阅； 
  * **开发**环境的订阅。 
5. Azure 帐户管理员[将订阅服务所有者添加到每个订阅](/azure/billing/billing-add-change-azure-subscription-administrator#add-an-rbac-owner-admin-for-a-subscription-in-azure-portal)。
6. 为**工作负荷所有者**创建审批过程，以请求创建资源组。 可通过许多方式实施审批过程，例如，通过电子邮件，或者使用 [Sharepoint 工作流](https://support.office.com/article/introduction-to-sharepoint-workflow-07982276-54e8-4e17-8699-5056eff4d9e3)等过程管理工具。 审批过程可以遵循以下步骤：
  1. **工作负荷所有者**为**开发**环境和/或**生产**环境中的所需 Azure 资源准备物料清单，并将其提交给**订阅所有者**。
  2. **订阅所有者**审查物料清单并验证请求的资源，以确保请求的资源适合规划的用途 - 例如，检查请求的[虚拟机大小](/azure/virtual-machines/windows/sizes)是否正确。
  3. 如果请求未获批准，则会通知**工作负荷所有者**。 如果请求获批准，**订阅所有者**将遵循组织的[命名约定](/azure/architecture/best-practices/naming-conventions)[创建请求的资源组](/azure/azure-resource-manager/resource-group-portal#manage-resource-groups)，添加具有[**参与者**角色](/azure/role-based-access-control/built-in-roles#contributor)的[**工作负荷所有者**](/azure/role-based-access-control/role-assignments-portal#add-access)，并通知**工作负荷所有者**已创建资源组。
7. 为工作负荷所有者创建审批过程，以请求共享基础结构所有者建立虚拟网络对等连接。 与前面的步骤一样，可以使用电子邮件或过程管理工具来实施此审批过程。

实施调控模型后，可以部署共享基础结构服务。

## <a name="section-4-deploy-shared-infrastructure-services"></a>第 4 部分：部署共享基础结构服务

组织可以使用多种[混合网络参考体系结构](/azure/architecture/reference-architectures/hybrid-networking/)将本地网络连接到 Azure。 其中的每种参考体系结构包含一个需要订阅标识符的部署。 在部署期间，请指定与**共享基础结构**环境关联的订阅的订阅标识符。 此外，还需要编辑模板文件，以指定**网络操作**用户管理的资源组；或者，可以在部署中使用默认资源组，并将具有**参与者**角色的**网络操作**用户添加到这些资源组。

<!-- links -->
[understand-resource-access-in-azure]: /azure/role-based-access-control/rbac-and-directory-admin-roles