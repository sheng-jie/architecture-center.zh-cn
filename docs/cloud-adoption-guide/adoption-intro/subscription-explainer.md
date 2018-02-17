---
title: "说明：什么是 Azure 订阅？"
description: "说明 Azure 订阅、帐户和产品/服务"
author: abuck
ms.openlocfilehash: 7b88e9489b40b100eecb76602b45901566b3f37f
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/09/2018
---
# <a name="explainer-what-is-an-azure-subscription"></a><span data-ttu-id="a30a4-103">说明：什么是 Azure 订阅？</span><span class="sxs-lookup"><span data-stu-id="a30a4-103">Explainer: What is an Azure subscription?</span></span>

<span data-ttu-id="a30a4-104">在[什么是 Azure Active Directory 租户？](tenant-explainer.md)说明文章中，你已了解 Azure Active Directory 租户中存储的组织的数字标识。</span><span class="sxs-lookup"><span data-stu-id="a30a4-104">In the [what is an Azure Active Directory tenant?](tenant-explainer.md) explainer article, you learned that digital identity for your organization is stored in an Azure Active Directory tenant.</span></span> <span data-ttu-id="a30a4-105">你还了解了 Azure 信任 Azure Active Directory 对用户创建、读取、更新或删除资源的请求进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="a30a4-105">You also learned that Azure trusts Azure Active Directory to authenticate user requests to create, read, update, or delete a resource.</span></span> 

<span data-ttu-id="a30a4-106">我们从根本上了解为何需要限制对资源进行这些操作的访问权限 - 为了防止未经身份验证和未经授权的用户访问我们的资源。</span><span class="sxs-lookup"><span data-stu-id="a30a4-106">We fundamentally understand why it's necessary to restrict access to these operations on a resource - to prevent unauthenticated and unauthorized users from accessing our resources.</span></span> <span data-ttu-id="a30a4-107">但是，这些资源操作具有组织想要控制的其他属性，例如允许用户或用户组创建的资源数，以及运行这些资源的成本。</span><span class="sxs-lookup"><span data-stu-id="a30a4-107">However, these resource operations have other properties that an organization would like to control, such as the number of resources a user or group of users is allowed to create, and, the cost to run those resources.</span></span> 

<span data-ttu-id="a30a4-108">Azure 实现了此控制，它名为**订阅**。</span><span class="sxs-lookup"><span data-stu-id="a30a4-108">Azure implements this control, and it is named a **subscription**.</span></span> <span data-ttu-id="a30a4-109">订阅将用户以及由这些用户创建的资源组合在一起。</span><span class="sxs-lookup"><span data-stu-id="a30a4-109">A subscription groups together users and the resources that have been created by those users.</span></span> <span data-ttu-id="a30a4-110">其中每个资源都会导致该特定资源的[总体限制][subscription-service-limits]。</span><span class="sxs-lookup"><span data-stu-id="a30a4-110">Each of those resources contributes to an [overall limit][subscription-service-limits] on that particular resource.</span></span>

<span data-ttu-id="a30a4-111">组织可以使用订阅管理成本并通过用户、团队、项目或使用许多其他策略管理资源创建。</span><span class="sxs-lookup"><span data-stu-id="a30a4-111">Organizations can use subscriptions to manage costs and creation of resource by users, teams, projects, or using many other strategies.</span></span> <span data-ttu-id="a30a4-112">这些策略将在中间和高级采用阶段文章中进行讨论。</span><span class="sxs-lookup"><span data-stu-id="a30a4-112">These strategies will be discussed in the intermediate and advanced adoption stage articles.</span></span> 

## <a name="next-steps"></a><span data-ttu-id="a30a4-113">后续步骤</span><span class="sxs-lookup"><span data-stu-id="a30a4-113">Next steps</span></span>

* <span data-ttu-id="a30a4-114">现在你已了解 Azure 订阅，请详细了解如何[创建订阅](subscription.md)，然后再创建第一个 Azure 资源。</span><span class="sxs-lookup"><span data-stu-id="a30a4-114">Now that you have learned about Azure subscriptions, learn more about [creating a subscription](subscription.md) before you create your first Azure resources..</span></span>

<!-- Links -->
[azure-get-started]: https://azure.microsoft.com/en-us/get-started/
[azure-offers]: https://azure.microsoft.com/en-us/support/legal/offer-details/
[azure-free-trial]: https://azure.microsoft.com/en-us/offers/ms-azr-0044p/
[azure-change-subscription-offer]: /azure/billing/billing-how-to-switch-azure-offer
[microsoft-account]: https://account.microsoft.com/account
[subscription-service-limits]: /azure/azure-subscription-service-limits
[docs-organizational-account]: https://docs.microsoft.com/en-us/azure/active-directory/sign-up-organization
