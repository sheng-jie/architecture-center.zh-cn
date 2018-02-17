---
title: "说明 - 什么是 Azure 资源组？"
description: "说明资源组的内部 Azure 功能"
author: petertay
ms.openlocfilehash: e7c7334bd88c28f57498486bd2bed3c349565222
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/09/2018
---
# <a name="what-is-an-azure-resource-group"></a>什么是 Azure 资源组？

在[什么是 Azure 资源管理器？](resource-manager-explainer.md)说明文章中，你已了解 Azure 资源管理器在进行创建、读取、更新或删除资源的调用时，需要**资源组标识符**。 此资源组 ID 是指**资源组**。 资源组只是 Azure 资源管理器应用于要组合在一起的资源的标识符。 此资源组 ID 允许 Azure 资源管理器对共享此 ID 的一组资源执行操作。

例如，用户可以指定资源组 ID 对 Azure 资源管理器 RESTful API 进行**删除**调用，而不包括任何特定资源 ID。 Azure 资源管理器在内部 Azure 数据库中查询具有指定资源组 ID 的所有资源，并调用 RESTful API 删除其中每个资源。

资源组不能包含不同订阅中的资源。 这是因为租户 ID 和订阅 ID 之间存在一对多关系 &mdash; 多个订阅可以信任同一个租户提供身份验证和授权，但每个订阅只能信任一个租户。 另外，订阅 ID 和资源组 ID 之间也存在一个对多关系 &mdash; 多个资源组可以属于同一订阅，但每个资源组只能属于一个订阅。 最后，资源组 ID 和资源 ID 之间也存在一个对多关系 &mdash; 单个资源组可以具有多个资源，但每个资源只能属于一个资源组。

## <a name="next-steps"></a>后续步骤

* 现在你已了解 Azure 资源组，请在[关于限制对资源的访问](/azure/active-directory/active-directory-understanding-resource-access?toc=/azure/architecture/cloud-adoption-guide/toc.json)中获取基础知识。 虽然它不属于基础采用阶段，但在中间采用阶段很重要。 然后，可以[创建第一个资源组](/azure/azure-resource-manager/resource-group-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)并查看 [Azure 资源组的设计指南](resource-group.md)。
