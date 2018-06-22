---
title: Azure 计算服务的决策树
description: 用于选择计算服务的流程图
author: MikeWasson
ms.date: 06/13/2018
ms.openlocfilehash: 60bb84d4bf210888d3d43498db043b6e452f6a80
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206547"
---
# <a name="decision-tree-for-azure-compute-services"></a>Azure 计算服务的决策树

Azure 提供多种方式来托管应用程序代码。 术语“计算”指的是计算资源（应用程序在这些资源上运行）的承载模型。 以下流程图将会帮助你选择应用程序的计算服务。 该流程图将引导你创建一组关键决策条件用于访问建议。 

**请将此流程图视为起点。** 每个应用程序有独特的要求，因此请将该建议作为起点。 然后执行更详细的评估，例如查看：
 
- 特征集
- [服务限制](/azure/azure-subscription-service-limits)
- [成本](https://azure.microsoft.com/pricing/)
- [SLA](https://azure.microsoft.com/support/legal/sla/)
- [区域可用性](https://azure.microsoft.com/global-infrastructure/services/)
- 开发人员生态系统和团队技能
- [计算比较表](./compute-comparison.md)

如果应用程序包括多个工作负荷，请单独评估每个工作负荷。 完整的解决方案可能会合并两个或更多个计算服务。

## <a name="flowchart"></a>流程图

![](../images/compute-decision-tree.svg)

## <a name="definitions"></a>定义

- “绿色领域”是指全新的从头开始生成的软件项目。 它不包括旧代码。 

- “褐色领域”是指在现有应用程序基础上生成的软件项目。 它可能继承旧代码或框架。

- “直接迁移”是一种将工作负荷迁移到云的策略，不重新设计应用程序，也不进行代码更改。 也称“重新托管”。 有关详细信息，请参阅 [Azure 迁移中心](https://azure.microsoft.com/migration/)。

- “云优化”是一种云迁移策略，需重构应用程序，以便充分利用云原生特性和功能。

## <a name="next-steps"></a>后续步骤

若要考虑其他条件，请参阅[选择 Azure 计算服务的条件](./compute-comparison.md)。
