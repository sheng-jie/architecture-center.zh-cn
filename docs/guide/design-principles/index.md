---
title: Azure 应用程序的设计原则
description: Azure 应用程序的设计原则
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 57b04839e14804ad97fc9c86e1f9c4fe6e0da472
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="design-principles-for-azure-applications"></a>Azure 应用程序的设计原则

遵循这些设计原则可以提高应用程序的可伸缩性、复原能力和易管理性。 

**[自我修复设计](self-healing.md)**。 在分布式系统中，故障时有发生。 设计应用程序以在故障发生时进行自我修复。

**[实现全面冗余](redundancy.md)**。 在应用程序中构建冗余，以避免出现单一故障点。
 
**[尽量减少协调](minimize-coordination.md)**。 最大程度地减少应用程序服务之间的协调以实现可伸缩性。
 
**[横向扩展设计](scale-out.md)**。合理设计应用程序，以便能够通过按需添加或删除新实例对应用程序进行横向缩放。

**[通过分区解决限制](partition.md)**。 使用分区来解决数据库、网络和计算限制。

**[运营设计](design-for-operations.md)**。 合理设计应用程序，使运营团队获得所需的工具。

**[使用托管服务](managed-services.md)**。 尽量使用平台即服务 (PaaS) 而不是基础结构即服务 (IaaS)。

**[使用最佳的数据存储完成作业](use-the-best-data-store.md)**。 选择最适合数据的存储技术，并了解如何使用该技术。 
 
**[演变设计](design-for-evolution.md)**。 所有成功的应用程序会不断变化。 演变设计是持续创新的关键。

**[根据业务需求构建](build-for-business.md)**。 每个设计决策必须与业务要求相称。

