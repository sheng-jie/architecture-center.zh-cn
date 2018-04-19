---
title: 采用 Azure：基础
description: 说明企业采用 Azure 所需的基线知识级别
author: petertay
ms.openlocfilehash: e9421b610e4eb07a3ed37bca56e513b0689484ef
ms.sourcegitcommit: 9ba82cf84cee06ccba398ec04c51dab0e1ca8974
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/13/2018
---
# <a name="adopting-azure-foundational"></a>采用 Azure：基础

采用 Azure 是企业具有组织成熟度的第一阶段。 此阶段结束时，组织中的人员将可以向 Azure 部署简单的工作负荷。

下表包含用于完成基础采用阶段的任务。 该列表是渐进式的，以便按顺序完成每个任务。 如果以前已完成某个任务，请转到该列表中的下一个任务。 

1. 了解 Azure 的内部：
    - **说明：**[Azure 的工作原理](azure-explainer.md)
2. 了解 Azure 中的企业数字标识：
    - **说明：**[Azure Active Directory 租户是什么？](tenant-explainer.md)
    - **如何：**[获取 Azure Active Directory 租户](/azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **指南：**[Azure AD 租户设计指南](tenant.md)
    - **如何：**[向 Azure Active Directory 添加新用户](/azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json)    
3. 了解 Azure 中的订阅：
    - **说明：**[什么是 Azure 订阅？](subscription-explainer.md)
    - **指南：**[Azure 订阅设计](subscription.md)
4. 了解 Azure 中的资源管理： 
    - **说明：**[什么是 Azure 资源管理器？](resource-manager-explainer.md)
    - **说明：**[什么是 Azure 资源组？](resource-group-explainer.md)
    - **说明：**[了解 Azure 中的资源访问权限](/azure/active-directory/active-directory-understanding-resource-access?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **如何：**[使用 Azure 门户创建 Azure 资源组](/azure/azure-resource-manager/resource-group-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **指南：**[Azure 资源组设计指南](resource-group.md)
    - **指南：**[Azure 资源的命名约定](/azure/architecture/best-practices/naming-conventions?toc=/azure/architecture/cloud-adoption-guide/toc.json)
5. 部署基本 Azure 体系结构：
    - 在 [Azure 计算选项概述](/azure/architecture/guide/technology-choices/compute-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)中了解不同类型的 Azure 计算选项，例如基础结构即服务 (IaaS) 和平台即服务 (PaaS)。
    - 既然已了解不同类型的 Azure 计算选项，请选取 PaaS Web 应用程序或 IaaS 虚拟机作为 Azure 中的第一个资源：
    - PaaS：平台即服务简介：
        - **如何：**[将基本 Web 应用程序部署到 Azure](/azure/app-service/app-service-web-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **指南：** 用于将[基本 Web 应用程序](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json)部署到 Azure 的经过验证的做法
    - IaaS：虚拟网络简介：
        - **说明：**[Azure 虚拟网络](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **如何：**[使用门户将虚拟网络部署到 Azure](/azure/virtual-network/virtual-networks-create-vnet-arm-pportal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - IasS：部署单一虚拟机 (VM) 工作负荷（Windows 和 Linux）：
        - **如何：**[使用门户将 Windows VM 部署到 Azure](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **指南：**[用于在 Azure 上运行 Windows VM 的经过验证的做法](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **如何：**[使用门户将 Linux VM 部署到 Azure](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **指南：**[用于在 Azure 上运行 Linux VM 的经过验证的做法](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)
