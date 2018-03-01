---
title: "指南：Azure AD 租户设计"
description: "作为基础云采用策略一部分的 Azure 租户设计指南"
author: telmosampaio
ms.openlocfilehash: 9ac52e9fd44bd8b9c777625002d5960f4f269be2
ms.sourcegitcommit: 29fbcb1eec44802d2c01b6d3bcf7d7bd0bae65fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/27/2018
---
# <a name="guidance-azure-ad-tenant-design"></a><span data-ttu-id="bb632-103">指南：Azure AD 租户设计</span><span class="sxs-lookup"><span data-stu-id="bb632-103">Guidance: Azure AD tenant design</span></span>

<span data-ttu-id="bb632-104">Azure AD 租户提供了用于一个或多个 [Azure 订阅](subscription-explainer.md)的数字标识服务和命名空间。</span><span class="sxs-lookup"><span data-stu-id="bb632-104">An Azure AD tenant provides digital identity services and namespaces used for one or more [Azure subscriptions](subscription-explainer.md).</span></span> <span data-ttu-id="bb632-105">如果你按照基础采用大纲操作，你已学习了[如何获取 Azure AD 租户][how-to-get-aad-tenant]。</span><span class="sxs-lookup"><span data-stu-id="bb632-105">If you are following the foundational adoption outline, you have already learned [how to get an Azure AD tenant][how-to-get-aad-tenant].</span></span> 

## <a name="design-considerations"></a><span data-ttu-id="bb632-106">设计注意事项</span><span class="sxs-lookup"><span data-stu-id="bb632-106">Design considerations</span></span>

- <span data-ttu-id="bb632-107">在基础采用阶段，可以从单个 Azure AD 租户开始。</span><span class="sxs-lookup"><span data-stu-id="bb632-107">At the foundational adoption stage, you can begin with a single Azure AD tenant.</span></span> <span data-ttu-id="bb632-108">如果你的组织已有 Office 365 订阅或 Azure 订阅，你已有可以使用的 Azure AD 租户。</span><span class="sxs-lookup"><span data-stu-id="bb632-108">If your organization has an existing Office 365 subscription or an Azure subscription, you already have an Azure AD tenant that you can use.</span></span> <span data-ttu-id="bb632-109">如果你没有上述租户，可以详细了解如何[如何获取 Azure AD 租户][how-to-get-aad-tenant]。</span><span class="sxs-lookup"><span data-stu-id="bb632-109">If you do not have either or of these, you can learn more about [how to get an Azure AD tenant][how-to-get-aad-tenant].</span></span> 
- <span data-ttu-id="bb632-110">在中间和高级采用阶段，你将学习如何将本地目录与 Azure AD 同步或联合。</span><span class="sxs-lookup"><span data-stu-id="bb632-110">In the intermediate and advanced adoption stages, you will learn how to synchronize or federate on-premises directories with Azure AD.</span></span> <span data-ttu-id="bb632-111">这样便可以在 Azure AD 中使用本地数字标识。</span><span class="sxs-lookup"><span data-stu-id="bb632-111">This will allow you to use on-premises digital identity in Azure AD.</span></span> <span data-ttu-id="bb632-112">但是，在基础阶段，你将添加仅在单个 Azure AD 租户中具有标识的新用户。</span><span class="sxs-lookup"><span data-stu-id="bb632-112">However, at the foundational stage, you will be adding new users that only have identity your single Azure AD tenant.</span></span> <span data-ttu-id="bb632-113">你将负责管理这些标识。</span><span class="sxs-lookup"><span data-stu-id="bb632-113">You will be responsible for managing those identities.</span></span> <span data-ttu-id="bb632-114">例如，你将需要加入新 Azure AD 用户、分离你不再想让其访问 Azure 资源的 Azure AD 用户，以及对用户权限进行其他更改。</span><span class="sxs-lookup"><span data-stu-id="bb632-114">For example, you will have to on-board new Azure AD users, off-board Azure AD users that you no longer wish to have access to Azure resources, and other changes to user permissions.</span></span>

## <a name="next-steps"></a><span data-ttu-id="bb632-115">后续步骤</span><span class="sxs-lookup"><span data-stu-id="bb632-115">Next Steps</span></span>

* <span data-ttu-id="bb632-116">现在你已有 Azure AD 租户，请了解[如何添加用户][azure-ad-add-user]。</span><span class="sxs-lookup"><span data-stu-id="bb632-116">Now that you have an Azure AD tenant, learn [how to add a user][azure-ad-add-user].</span></span> <span data-ttu-id="bb632-117">将一个或多个新用户添加到 Azure AD 租户后，下一步是学习有关 [Azure 订阅](subscription-explainer.md)的内容。</span><span class="sxs-lookup"><span data-stu-id="bb632-117">After you have added one or more new users to your Azure AD tenant, your next step is learning about [Azure subscriptions](subscription-explainer.md).</span></span>

<!-- Links -->

[azure-ad-add-user]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-manage-azure-ad]: /azure/active-directory/active-directory-administer?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-associate-subscription]: /azure/active-directory/active-directory-how-subscriptions-associated-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
