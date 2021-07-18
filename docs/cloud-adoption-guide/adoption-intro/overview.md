---
title: 采用 Azure：基础
description: 介绍企业采用 Azure 所要掌握的基础知识
author: petertay
ms.openlocfilehash: b5a0a4a2c4ed1d06c97774b0eca643a89a5a2110
ms.sourcegitcommit: c704d5d51c8f9bbab26465941ddcf267040a8459
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/24/2018
ms.locfileid: "39229161"
---
# <a name="adopting-microsoft-azure-foundational"></a>采用 Microsoft Azure：基础

对于刚刚涉入云技术的组织而言，可能很难确定要在哪个适当的时机开启采用之旅。 基础采用阶段的目标是提供一个起点。 组织中的个人在经历此阶段后，将会获得所有必要的知识和技能，可将简单工作负荷的计算资源部署到 Azure。 

> [!NOTE]
> 本指南不会介绍应用程序开发。 有关在 Azure 上开发应用程序的详细信息，请参阅 [Azure 应用程序体系结构指南](/azure/architecture/guide/)。

本指南所述的此阶段面向组织中的以下角色：

- *财务：* 提交到 Azure 的财务信息的所有者，负责制定用于跟踪资源消耗成本（包括计费和费用分摊）的策略与过程。
- 中心 IT：负责调控组织的云资源，包括资源管理和访问权限，以及工作负荷运行状况和监视。
- 工作负荷所有者：将工作负荷部署到 Azure 时所涉及到的所有开发角色，包括开发人员、测试人员和构建工程师。

## <a name="section-1-azure-basics"></a>第 1 部分：Azure 基础知识

本简介部分面向财务和中心 IT 角色。 本部分的重点是让读者在准备好学习[云调控的概念](governance-explainer.md)之前，对 [Azure 的工作原理](azure-explainer.md)有一个基本的了解。 此外，组织中的工作负荷所有者也很适合阅读此内容，因为本部分可帮助他们了解如何管理资源访问权限。

## <a name="section-2-governance-design-guide"></a>第 2 部分：调控设计指南

了解 Azure 的工作原理以及云调控的基础知识后，采用 Azure 的第一步是了解 Azure 中的[资源访问管理](azure-resource-access.md)。 本文介绍发出资源访问请求的 Azure 服务，以及用于验证这些请求的控制措施。

下一步是了解如何为单个团队[设计调控模型](governance-how-to.md)。 本文将介绍如何配置前面所了解的资源访问管理服务和控制措施。

## <a name="section-3-implementing-a-basic-resource-access-management-model"></a>第 3 部分：实施基本的资源访问管理模型

采用旅程的最后一步是了解如何实施前面设计的调控模型。 

若要开始，组织需要一个 Azure 帐户。 如果组织的某个现有 [Microsoft 企业协议](https://www.microsoft.com/licensing/licensing-programs/enterprise.aspx)不包括 Azure，可以通过做出前期货币承诺来添加 Azure。 有关详细信息，请参阅[为企业获得 Azure 许可](https://azure.microsoft.com/pricing/enterprise-agreement/)。 

创建 Azure 帐户时，需将组织中的某人指定为 Azure **帐户所有者**。 然后，会默认创建一个 Azure Active Directory (Azure AD) 租户。 Azure **帐户所有者**必须为组织中的**工作负荷所有者**[创建用户帐户](/azure/active-directory/add-users-azure-active-directory)。 

接下来，Azure **帐户所有者**必须[创建一个订阅](https://docs.microsoft.com/partner-center/create-a-new-subscription)，并将其[关联到 Azure AD 租户](/azure/active-directory/fundamentals/active-directory-how-subscriptions-associated-directory)。

最后，在创建订阅并将其关联到 Azure AD 租户后，可[将**工作负荷所有者**添加到具有内置**所有者**角色的订阅](/azure/billing/billing-add-change-azure-subscription-administrator#add-an-rbac-owner-for-a-subscription-in-azure-portal)。

## <a name="section-4-deploy-a-basic-workload-architecture-to-azure"></a>第 4 部分：将基本工作负荷体系结构部署到 Azure

本部分的受众是工作负荷所有者角色。 工作负荷所有者定义其工作负荷的计算和网络要求，根据这些要求选择正确的资源，并将资源部署到 Azure。 

在基础采用阶段，工作负荷所有者可以选择基本 Web 应用程序或虚拟网络 (VNet) 和虚拟机 (VM)。 有关 Azure 中这些不同类型的计算选项的详细信息，请查看 [Azure 计算选项概述](/azure/architecture/guide/technology-choices/compute-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)。

无论选择哪个计算选项，其中的每个部署都需要一个**资源组**。 **工作负荷所有者**必须[创建资源组](/azure/azure-resource-manager/vs-azure-tools-resource-groups-deployment-projects-create-deploy)。 在部署过程中，**工作负荷所有者**指定资源组的名称。 此名称将在后续部分中使用。

### <a name="basic-web-application-paas"></a>基本 Web 应用程序 (PaaS)

对于基本 Web 应用程序，请从 [Web 应用文档](/azure/app-service?toc=/azure/architecture/cloud-adoption-guide/toc.json)中选择一篇为时 5 分钟的快速入门，并遵循其中的步骤。 

> [!NOTE]
> 某些快速入门默认会部署资源组。 在这种情况下，**工作负荷所有者**不需要显式创建资源组。 否则，应将 Web 应用程序部署到前面创建的资源组。

部署简单的工作负荷后，可以详细了解有关将[基本 Web 应用程序](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json)部署到 Azure 的成熟做法。

### <a name="single-windows-or-linux-vm-iaas"></a>单个 Windows 或 Linux VM (IaaS)

对于虚拟机上运行的简单工作负荷，第一步是部署虚拟网络。 Azure 中的所有 IaaS 资源（例如虚拟机、负载均衡器和网关）都需要虚拟网络。 了解 [Azure 虚拟网络](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)，然后遵循相应的步骤，[使用门户将虚拟网络部署到 Azure](/azure/virtual-network/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)。 在 Azure 门户中指定虚拟网络的设置时，请指定前面创建的资源组的名称。

下一步是确定是否部署单个 Windows 或 Linux VM。 对于 Windows VM，请遵循相应的步骤，[使用门户将 Windows VM 部署到 Azure](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)。 同样，在 Azure 门户中指定虚拟机的设置时，请指定前面创建的资源组的名称。

遵循这些步骤并部署 VM 后，可以了解[有关在 Azure 上运行 Windows VM 的成熟做法](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)。 对于 Linux VM，请遵循相应的步骤，[使用门户将 Linux VM 部署到 Azure](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)。 还可以详细了解[有关在 Azure 上运行 Linux VM 的成熟做法](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)。

## <a name="next-steps"></a>后续步骤

云准备工作的下一阶段是[**中间阶段**](../intermediate-stage/overview.md)。 在中间阶段，你将了解如何为多个团队扩展运行多个工作负荷和管理模型的本地网络。