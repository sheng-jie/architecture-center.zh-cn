---
title: "指南：Azure AD 租户设计"
description: "作为基础云采用策略一部分的 Azure 租户设计指南"
author: telmosampaio
ms.openlocfilehash: 5bf710f74bde04e041f2896b4a638c3513e4aa44
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/09/2018
---
# <a name="guidance-azure-ad-tenant-design"></a>指南：Azure AD 租户设计

Azure AD 租户提供了用于一个或多个 [Azure 订阅](subscription-explainer.md)的数字标识服务和命名空间。 如果你按照基础采用大纲操作，你已学习了[如何获取 Azure AD 租户][how-to-get-aad-tenant]。 

## <a name="design-considerations"></a>设计注意事项

- 在基础采用阶段，可以从单个 Azure AD 租户开始。 如果你的组织已有 Office 365 订阅或 Azure 订阅，你已有可以使用的 Azure AD 租户。 如果你没有上述租户，可以详细了解如何[如何获取 Azure AD 租户][how-to-get-aad-tenant]。 
- 在中间和高级采用阶段，你将学习如何将本地目录与 Azure AD 同步或联合。 这样便可以在 Azure AD 中使用本地数字标识。 但是，在基础阶段，你将添加仅在单个 Azure AD 租户中具有标识的新用户。 你负责管理这些标识。 例如，你将需要加入新 Azure AD 用户、分离你不再想让其访问 Azure 资源的 Azure AD 用户，以及对用户权限进行其他更改。

## <a name="next-steps"></a>后续步骤

* 现在你已有 Azure AD 租户，请了解[如何添加用户][azure-ad-add-user]。 将一个或多个新用户添加到 Azure AD 租户后，下一步是学习有关 [Azure 订阅](subscription-explainer.md)的内容。

<!-- Links -->

[azure-ad-add-user]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-manage-azure-ad]: /azure/active-directory/active-directory-administer?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-associate-subscription]: /azure/active-directory/active-directory-how-subscriptions-associated-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json