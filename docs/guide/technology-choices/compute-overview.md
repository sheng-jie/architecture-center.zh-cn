---
title: Azure 计算选项概述
description: Azure 计算选项概述
author: MikeWasson
ms.date: 06/13/2018
ms.openlocfilehash: ceb70f8eeff42e6cadb8a63c2f36986f26322201
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206524"
---
# <a name="overview-of-azure-compute-options"></a>Azure 计算选项概述

术语“计算”指的是计算资源（应用程序在这些资源上运行）的承载模型。 

## <a name="overview"></a>概述

一端是**服务架构**（Infrastructure-as-a-Service，IaaS）。 使用 IaaS 可以预配所需的 VM 以及关联的网络和存储组件。 然后将需要的任何软件和应用程序部署到这些 VM 上。 除非 Microsoft 管理基础结构，否则该模型最接近传统的本地环境。 仍由你管理单独的 VM。  

平台即服务（Platform-as-a-Service，PaaS）提供托管的承载环境，可在其中部署应用程序而无需管理 VM 或网络资源。 例如，指定实例计数而不是创建单独的 VM，服务将预配、配置并管理必需的资源。 Azure App Service 是 PaaS 服务的一个示例。

从 IaaS 到纯 PaaS 存在一个范围。 例如，Azure VM 可以使用 VM 规模集自动缩放。 此自动缩放功能严格说来，并非 PaaS，但它是可能在 PaaS 服务中找到的管理功能类型。

借助功能即服务（Functions-as-a-Service，简称 FaaS），更不需要担心承载环境。 只需部署代码，服务便会自动运行它，而无需创建计算实例并向其部署代码。 无需管理计算资源。 这些服务使用无服务器体系结构，并且无缝地纵向扩展或减少到处理流量所需的级别。 Azure Functions 是 FaaS 服务。

IaaS 提供最大的控制力、灵活性和可移植性。 FaaS 具有简便性，提供弹性缩放能力，可能会节省成本，因为用户只为代码运行的时间支付费用。 PaaS 则位于这两者之间。 通常情况下，服务提供的灵活性越高，用户配置和管理资源的责任越大。 FaaS 服务自动管理运行应用程序的几乎所有方面，而 IaaS 解决方案需要用户预配、配置并管理自己创建的 VM 和网络组件。

## <a name="azure-compute-options"></a>Azure 计算选项

以下是 Azure 中目前提供的主要计算选项：

- [虚拟机](/azure/virtual-machines/)是 IaaS 服务，允许在虚拟网络 (VNet) 内部署和管理 VM。
- [应用服务](/azure/app-service/app-service-value-prop-what-is)是托管型 PaaS 产品/服务，用于托管 Web 应用、移动应用后端、RESTful API 或自动化业务流程。
- [Service Fabric](/azure/service-fabric/service-fabric-overview) 是可在多个环境（包括 Azure 或本地环境）中运行的分布式系统平台。 Service Fabric 是跨计算机群集的微服务业务流程协调程序。 
- [Azure 容器服务](/azure/container-service/container-service-intro)使用户可以创建、配置和管理预配为运行容器化应用程序的 VM 群集。
- [Azure 容器实例](/azure/container-instances/container-instances-overview)提供了在 Azure 中运行容器的最简捷方式，既无需预配任何虚拟机，也不必采用更高级的服务。
- [Azure Functions](/azure/azure-functions/functions-overview) 是托管的 FaaS 服务。
- [Azure Batch](/azure/batch/batch-technical-overview) 是一个托管服务，适用于运行大规模并行和高性能计算 (HPC) 应用程序。
- [云服务](/azure/cloud-services/cloud-services-choose-me)是用于运行云应用程序的托管服务。 它使用 PaaS 承载模型。 

选择计算选项时，应考虑以下因素：

- 承载模型。 服务承载方式是什么？ 此承载环境具有哪些要求和限制？ 
- DevOps。 是否存在用于应用程序升级的内置支持？ 部署模型是什么？
- 可伸缩性。 服务如何处理实例的添加或删除？ 是否能基于负载和其他指标自动缩放？ 
- 可用性。 什么是服务 SLA？ 
- 成本。 除了服务本身的成本，还要考虑管理在该服务上构建的解决方案的操作成本。 例如，IaaS 解决方案的操作成本可能更高。
- 每项服务的总体限制有哪些？ 
- 哪类应用程序体系结构适用于此服务？ 

## <a name="next-steps"></a>后续步骤

若要帮助选择应用程序的计算服务，请使用 [Azure 计算服务的决策树](./compute-decision-tree.md)

有关 Azure 中计算选项的详细比较信息，请参阅[选择 Azure 计算服务的条件](./compute-comparison.md)。
