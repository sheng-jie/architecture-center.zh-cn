---
title: 了解 Azure 中的资源访问管理
description: 解释 Azure 中的资源访问管理构造：Azure 资源管理器、订阅、资源组和资源
author: petertay
ms.openlocfilehash: 02e25e943822ba065d360d687d34283d86a805bd
ms.sourcegitcommit: c704d5d51c8f9bbab26465941ddcf267040a8459
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/24/2018
ms.locfileid: "39229457"
---
# <a name="understanding-resource-access-management-in-azure"></a>了解 Azure 中的资源访问管理

在[什么是资源调控？](governance-explainer.md)中我们已了解到，调控是指根据组织的目标和要求，对 Azure 资源的使用持续进行管理、监视和审核的过程。 在继续学习如何设计调控模型之前，必须了解 Azure 中的资源访问管理控制措施。 这些资源访问管理控制措施的配置构成了调控模型的基础。

首先，让我们进一步探讨资源在 Azure 中的部署方式。 

## <a name="what-is-an-azure-resource"></a>什么是 Azure 资源？

在 Azure 中，术语“资源”是指 Azure 管理的实体。 例如，虚拟机、虚拟网络和存储帐户都称为 Azure 资源。

![](../_images/governance-1-9.png)   
*图 1.资源。*

## <a name="what-is-an-azure-resource-group"></a>什么是 Azure 资源组？

Azure 中的每个资源必须属于某个[资源组](/azure/azure-resource-manager/resource-group-overview#resource-groups)。 资源组只是一个逻辑构造，它将多个资源分组到一起，以便可将它们作为单个实体进行管理。 举例来说，具有类似生命周期的资源（例如，[n 层应用程序](/azure/architecture/guide/architecture-styles/n-tier)的资源）可以作为一个组进行创建或删除。 

![](../_images/governance-1-10.png)   
*图 2.资源组包含资源。* 

资源组及其包含的资源与 Azure **订阅**相关联。 

## <a name="what-is-an-azure-subscription"></a>什么是 Azure 订阅？

Azure 订阅与资源组的类似之处在于，它也是一个逻辑构造，可将资源组及其资源分组到一起。 但是，Azure 订阅还与 Azure 资源管理器使用的控制措施相关联。 这是什么意思呢？ 让我们进一步探讨 Azure 资源管理器，以了解它与 Azure 订阅之间的关系。

![](../_images/governance-1-11.png)   
*图 3.Azure 订阅。*

## <a name="what-is-azure-resource-manager"></a>什么是 Azure 资源管理器？

在 [Azure 的工作原理](azure-explainer.md)中我们已了解到，Azure 包含一个“前端”，该前端中的许多服务协调 Azure 的所有功能。 其中的一个服务是 [Azure 资源管理器](/azure/azure-resource-manager/)，此服务托管客户端用来管理资源的 RESTful API。 

![](../_images/governance-1-12.png)   
*图 4.Azure 资源管理器。*

下图显示了三个客户端：[Powershell](/powershell/azure/overview)、[Azure 门户](https://portal.azure.com)和 [Azure 命令行接口 (CLI)](/cli/azure)：

![](../_images/governance-1-13.png)   
*图 5.Azure 客户端连接到 Azure 资源管理器 RESTful API。*

尽管这些客户端使用 RESTful API 连接到 Azure 资源管理器，但 Azure 资源管理器不包含直接管理资源的功能。 Azure 中的大多数资源类型具有自身的[**资源提供程序**](/azure/azure-resource-manager/resource-group-overview#terminology)。 

![](../_images/governance-1-14.png)   
*图 6.Azure 资源提供程序。*

当客户端发出管理特定资源的请求时，Azure 资源管理器会连接到该资源类型的资源提供程序以完成该请求。 例如，如果客户端发出管理虚拟机资源的请求，则 Azure 资源管理器会连接到 **Microsoft.compute** 资源提供程序。 

![](../_images/governance-1-15.png)   
*图 7.Azure 资源管理器连接到 **Microsoft.compute** 资源提供程序，以管理客户端请求中指定的资源。*

请注意，Azure 资源管理器要求客户端指定订阅和资源组的标识符，以管理虚拟机资源。 

了解 Azure 资源管理器工作原理后，让我们返回到有关 Azure 订阅如何与 Azure 资源管理器所用控制措施相关联的内容。 在 Azure 资源管理器执行任何资源管理请求之前，需要先检查一组控制措施。 

第一个控制措施是请求必须由已验证的用户发出，并且 Azure 资源管理器与 [Azure Active Directory (Azure AD)](/azure/active-directory/) 之间建立了可信的关系，这样才能提供用户标识功能。

![](../_images/governance-1-16.png)   
*图 8.Azure Active Directory。*

在 Azure AD 中，用户划分到**租户**。 租户是一个逻辑构造，它通常代表一个与组织关联的安全专用的 Azure AD 实例。 每个订阅与一个 Azure AD 租户相关联。

![](../_images/governance-1-17.png)   
*图 9.与订阅关联的 Azure AD 租户。*

管理特定订阅中的资源的每个客户端请求要求用户在关联的 Azure AD 租户中有一个帐户。 

下一个控制措施是检查用户是否拥有发出请求的足够权限。 权限是使用[基于角色的访问控制 (RBAC)](/azure/role-based-access-control/) 分配给用户的。

![](../_images/governance-1-18.png)   
*图 10.为租户中的每个用户分配一个或多个 RBAC 角色。*

RBAC 角色指定用户对特定的资源拥有的一组权限。 将角色分配给用户时，会应用这些权限。 例如，[内置的**所有者**角色](/azure/role-based-access-control/built-in-roles#owner)允许用户对资源执行任何操作。

下一个控制措施是检查为 [Azure 资源策略](/azure/azure-policy/)指定的设置是否允许该请求。 Azure 资源策略指定允许对特定资源执行的操作。 例如，Azure 资源策略可以指定只允许用户部署特定类型的虚拟机。

![](../_images/governance-1-19.png)   
*图 11.Azure 资源策略。*

下一个控制措施是检查请求是否未超过 [Azure 订阅限制](/azure/azure-subscription-service-limits)。 例如，每个订阅限制为包含 980 个资源组。 达到该限制后，如果收到部署更多资源组的请求，则会拒绝该请求。

![](../_images/governance-1-20.png)   
*图 12.Azure 资源限制。* 

最后一个控制措施是检查请求是否在与订阅关联的财务承诺范围内。 例如，如果请求是部署虚拟机，则 Azure 资源管理器会验证订阅是否包含足够的付款信息。

![](../_images/governance-1-21.png)   
*图 13.财务承诺与订阅相关联。*

# <a name="summary"></a>摘要

本文已介绍如何使用 Azure 资源管理器在 Azure 中管理资源访问权限。

# <a name="next-steps"></a>后续步骤

了解如何在 Azure 中管理资源访问权限后，接下来请学习如何使用这些服务[设计调控模型](governance-how-to.md)。