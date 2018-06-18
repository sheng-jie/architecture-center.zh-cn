---
title: 说明：Azure 如何工作？
description: 介绍了 Azure 的内部运行
author: petertay
ms.openlocfilehash: 1cebcc001b8d2ae93d8b0271c48d54617281c7c2
ms.sourcegitcommit: b3d74d8a89b2224fc796ce0e89cea447af43a0d4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/11/2018
ms.locfileid: "35290503"
---
# <a name="explainer-how-does-azure-work"></a>说明：Azure 如何工作？

Azure 是 Microsoft 的公有云平台。 Azure 提供了一个大型的服务集合，包括平台即服务 (PaaS)、基础结构即服务 (IaaS)、数据库即服务 (DBaaS) 以及许多其他服务。 但是，确切而言，什么是 Azure，它如何工作？

与其他云平台一样，Azure 依赖于称为**虚拟化**的技术。 可以在软件中仿真大多数计算机硬件，因为大多数计算机硬件只是在硅片中永久或半永久编码的一组指令。 使用将软件指令映射为硬件指令的仿真层，虚拟化的硬件可以在软件中执行，就像它是实际硬件本身一样。

本质上来说，云是位于一个或多个数据中心内的一组物理服务器，它们代表客户执行虚拟化硬件。 那么，云如何同时为数百万客户创建、启动、停止和删除虚拟化硬件的数百万实例？

为了理解这一点，让我们看一下数据中心内硬件的体系结构。  每个数据中心内是位于服务器机架中的服务器集合。 每个服务器机架包含许多服务器**刀片**和一个提供网络连接的网络交换机，以及一个用于供电的配电单元 (PDU)。 机架有时组合到一起形成更大的单元，称为**群集**。 

在每个机架或群集中，大多数服务器设计为代表用户运行这些虚拟化硬件实例。 但是，有些服务器运行称为结构控制器的云管理软件。 **结构控制器**是一个有许多职责的分布式应用程序。 它分配服务，监视服务器和在其上运行的服务的运行状况，并且在服务器发生故障时将其修复。

结构控制器的每个实例连接到运行云业务流程软件的另一组服务器，通常称为**前端**。 前端托管着用于云执行的所有功能的 Web 服务、RESTful API 和内部 Azure 数据库。 

例如，前端托管的服务对客户请求进行处理来分配 Azure 资源（例如[虚拟网络][vnet]、[虚拟机][vms]）和服务（例如 [Cosmos DB][cosmosdb]）。 首先，前端对用户进行校验并验证用户是否有权分配所请求的资源。 如果有，则前端将请求数据库分配一个具有足够容量的服务器机架，然后指示机架上的结构控制器来分配资源。

因此，非常简单，Azure 是服务器和网络硬件的巨大集合，并且其中还包含一组复杂的分布式应用程序，它们安排这些服务器上的虚拟化硬件和软件的配置和操作。 并且，正是此业务流程安排使得 Azure 如此强大 - 用户不再负责维护和升级硬件，Azure 在幕后执行所有这些操作。 

## <a name="next-steps"></a>后续步骤

* 了解 Azure 的内部功能后，接下来请了解[资源访问调控](governance-explainer.md)。 然后，转到采用 Azure 的第一个步骤，即[了解 Azure 中的数字标识](tenant-explainer.md)。 完成该步骤后，便可以[在 Azure AD 中创建第一个用户][docs-add-users-to-aad]。

<!-- Links -->

[cosmosdb]: /azure/cosmos-db/introduction
[docs-add-users-to-aad]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[vms]: /azure/virtual-machines/
[vnet]: /azure/virtual-network/virtual-networks-overview
