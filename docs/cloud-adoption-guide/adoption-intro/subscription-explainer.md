---
title: 说明：什么是 Azure 订阅？
description: 说明 Azure 订阅、帐户和产品/服务
author: alexbuckgit
ms.openlocfilehash: 1650d90d6f78b46b7fe4128d2dab6a80bd6cca78
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/23/2018
ms.locfileid: "29478300"
---
# <a name="explainer-what-is-an-azure-subscription"></a>说明：什么是 Azure 订阅？

在[什么是 Azure Active Directory 租户？](tenant-explainer.md)说明文章中，你已了解 Azure Active Directory 租户中存储的组织的数字标识。 你还了解了 Azure 信任 Azure Active Directory 对用户创建、读取、更新或删除资源的请求进行身份验证。 

我们从根本上了解为何需要限制对资源进行这些操作的访问权限 - 为了防止未经身份验证和未经授权的用户访问我们的资源。 但是，这些资源操作具有组织想要控制的其他属性，例如允许用户或用户组创建的资源数，以及运行这些资源的成本。 

Azure 实现了此控制，它名为**订阅**。 订阅将用户以及由这些用户创建的资源组合在一起。 其中每个资源都会导致该特定资源的[总体限制][subscription-service-limits]。

组织可以使用订阅管理成本并通过用户、团队、项目或使用许多其他策略管理资源创建。 这些策略将在中级和高级采用阶段文章中进行讨论。 

## <a name="next-steps"></a>后续步骤

* 现在你已了解 Azure 订阅，请详细了解如何[创建订阅](subscription.md)，然后再创建第一个 Azure 资源。

<!-- Links -->
[azure-get-started]: https://azure.microsoft.com/get-started/
[azure-offers]: https://azure.microsoft.com/support/legal/offer-details/
[azure-free-trial]: https://azure.microsoft.com/offers/ms-azr-0044p/
[azure-change-subscription-offer]: /azure/billing/billing-how-to-switch-azure-offer
[microsoft-account]: https://account.microsoft.com/account
[subscription-service-limits]: /azure/azure-subscription-service-limits
[docs-organizational-account]: https://docs.microsoft.com/azure/active-directory/sign-up-organization
