---
title: 指南：Azure 资源组设计
description: 作为基础云采用策略一部分的 Azure 资源组设计指南
author: petertay
ms.openlocfilehash: ac6cbb03be8cdba020641d3b9034ad9d20101acf
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/09/2018
---
# <a name="guidance-azure-resource-group-design"></a>指南：Azure 资源组设计

在 Azure 中，[资源组](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview#resource-groups)是对资源进行分组的逻辑容器。 在 Azure 中部署的每个资源必须部署到单个资源组中。

## <a name="design-considerations"></a>设计注意事项

- 同一资源组中的所有资源应具有相同的生命周期。 也就是说，你的做法应当是将具有相同生命周期的资源作为一个组进行部署、更新和删除。 例如，单个 Web 应用程序的计算资源通常部署为单个单元。 但是，与其他 Web 应用程序共享的数据库很可能以不同的生命周期进行管理，并且应当位于其自己的资源组中。
- 资源组可以包含位于不同区域的资源。
- 单个资源组中的所有资源必须与单个订阅相关联。 
- 资源可以在资源组之间移动，但不能移动到包含部署到其他订阅的资源的资源组。
- 资源的组分配不影响与其他资源组中的资源进行连接或交互。 例如，分配到一个资源组的虚拟机可以连接到分配给另一个资源组的数据库（如果它们之间有网络连接）。
- 资源组可用于划分对管理操作的访问控制。 可以在订阅级别或资源组级别应用基于角色的访问控制 (RBAC) 权限。 在资源组级别会继承在订阅级别分配的任何权限。 在中级和高级采用阶段，你将了解有关 RBAC 和资源权限的更多信息。

## <a name="proven-practices"></a>经过验证的做法

- 在基础阶段，你可能只管理几个概念证明 (POC) 项目，每个具有少量资源。 因为 POC 资源通常具有相同的生命周期，所以，可以为这些项目中的每一个创建单个资源组。
- 在中级采用阶段，你将管理多个项目。 不同类型的项目可以从其他资源组设计受益。 如果打算将任何初始 POC 项目提升到生产环境，可以将资源移动到另一个资源组（如果该资源组属于同一订阅）。 因此，在此阶段，应当将这些资源部署到同一订阅，将来可以重新组织资源。

## <a name="next-steps"></a>后续步骤

* 现在，你已了解了适用于基础采用阶段的经过验证的做法，你已能够创建资源组并向其中添加资源。 虽然在此阶段管理的资源较少，但随着添加的资源越来越多，管理资源将变得越来越复杂。 请了解 [Azure 命名约定和标记](/azure/architecture/best-practices/naming-conventions?toc=/azure/architecture/cloud-adoption-guide/toc.json)以便在为中级采用阶段做准备时对资源进行命名和标记。
