---
title: 说明：什么是 Azure 资源管理器？
description: 说明 Azure 资源管理器的内部功能
author: petertay
ms.openlocfilehash: 60f09901bdc4b292abd73335b78c7d56a76f27a6
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/09/2018
---
# <a name="explainer-what-is-azure-resource-manager"></a>说明：什么是 Azure 资源管理器？

在 [Azure 的工作原理](azure-explainer.md)说明中，你已了解 Azure 的内部体系结构。 此体系结构包括托管分布式应用程序（管理内部 Azure 服务）的前端。

Azure 前端包括一个称为 Azure 资源管理器的服务。 Azure 资源管理器负责管理 Azure 中托管的资源从创建到删除的生命周期。 有多种方法可与 Azure 资源管理器进行交互 &mdash; 使用 Powershell、Azure 命令行接口、SDK，但上述每个工具都只是 Azure 资源管理器托管的 RESTful API 的包装。

Azure 资源管理器提供的 RESTful API 是一组基于**资源提供程序**的一致接口。 资源提供程序只是在 Azure 中创建、读取、更新和删除资源的服务。 事实上，RESTful API 包括上述每种功能的方法。 

RESTful API 需要用户访问令牌、**订阅 ID** 和一项新内容（即 **资源组 ID**）。 我们将在[资源组说明](resource-group-explainer.md)一文中介绍资源组。 Azure 资源管理器还需要**租户 ID**，该 ID 作为访问令牌的一部分进行编码。 

收到创建资源的有效 API 调用时，Azure 资源管理器将在指定的区域中查找容量，并将任何所需的文件复制到暂存位置。 然后，将请求发送到机架中的结构控制器，结构控制器将分配资源。 结构控制器以成功或失败通知以及新创建的资源的**资源 ID** 对请求做出响应。 这四个 ID 存储在 Azure 内部，并一起作为已部署资源的唯一标识符。

## <a name="next-steps"></a>后续步骤

* 现在你已了解 Azure 资源管理器的内部功能，了解[资源组](resource-group-explainer.md)可帮助你创建第一个资源组。
