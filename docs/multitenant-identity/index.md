---
title: "多租户应用程序的标识管理"
description: "有关多租户应用程序中的身份验证、授权和标识管理的最佳做法。"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.next: tailspin
ms.openlocfilehash: c363ac01e798b522fa95f39586e28fe3af5fae4a
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="manage-identity-in-multitenant-applications"></a><span data-ttu-id="8fb0e-103">管理多租户应用程序中的标识</span><span class="sxs-lookup"><span data-stu-id="8fb0e-103">Manage Identity in Multitenant Applications</span></span>

<span data-ttu-id="8fb0e-104">本系列文章介绍有关使用 Azure AD 进行身份验证和标识管理时，适用于多租户的最佳做法。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-104">This series of articles describes best practices for multitenancy, when using Azure AD for authentication and identity management.</span></span>

<span data-ttu-id="8fb0e-105">[![GitHub](../_images/github.png) 示例代码][sample application]</span><span class="sxs-lookup"><span data-stu-id="8fb0e-105">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="8fb0e-106">构建多租户应用程序时，首要的难题之一是管理用户标识，因为目前每个用户都属于某个租户。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-106">When you're building a multitenant application, one of the first challenges is managing user identities, because now every user belongs to a tenant.</span></span> <span data-ttu-id="8fb0e-107">例如：</span><span class="sxs-lookup"><span data-stu-id="8fb0e-107">For example:</span></span>

* <span data-ttu-id="8fb0e-108">用户使用其组织凭据登录。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-108">Users sign in with their organizational credentials.</span></span>
* <span data-ttu-id="8fb0e-109">用户应该有权访问其所在组织的数据，但不能访问属于其他租户的数据。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-109">Users should have access to their organization's data, but not data that belongs to other tenants.</span></span>
* <span data-ttu-id="8fb0e-110">组织可以注册应用程序，然后将应用程序角色分配到其成员。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-110">An organization can sign up for the application, and then assign application roles to its members.</span></span>

<span data-ttu-id="8fb0e-111">Azure Active Directory (Azure AD) 提供一些极佳的功能用于支持所有这些方案。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-111">Azure Active Directory (Azure AD) has some great features that support all of these scenarios.</span></span>

<span data-ttu-id="8fb0e-112">我们创建了某个多租户应用程序的完整[端到端实现][sample application]，以辅导学习本系列文章。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-112">To accompany this series of articles, we created a complete [end-to-end implementation][sample application] of a multitenant application.</span></span> <span data-ttu-id="8fb0e-113">这些文章体现了我们在构建应用程序的过程中所获得的经验。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-113">The articles reflect what we learned in the process of building the application.</span></span> <span data-ttu-id="8fb0e-114">若要开始使用该应用程序，请参阅[运行 Surveys 应用程序][running-the-app]。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-114">To get started with the application, see [Run the Surveys application][running-the-app].</span></span>

## <a name="introduction"></a><span data-ttu-id="8fb0e-115">介绍</span><span class="sxs-lookup"><span data-stu-id="8fb0e-115">Introduction</span></span>

<span data-ttu-id="8fb0e-116">假设你要编写一个将在云中托管的企业 SaaS 应用程序。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-116">Let's say you're writing an enterprise SaaS application to be hosted in the cloud.</span></span> <span data-ttu-id="8fb0e-117">当然，该应用程序将会包含用户：</span><span class="sxs-lookup"><span data-stu-id="8fb0e-117">Of course, the application will have users:</span></span>

![用户](./images/users.png)

<span data-ttu-id="8fb0e-119">但是，这些用户属于组织：</span><span class="sxs-lookup"><span data-stu-id="8fb0e-119">But those users belong to organizations:</span></span>

![组织用户](./images/org-users.png)

<span data-ttu-id="8fb0e-121">示例：Tailspin 在其 SaaS 应用程序中销售订阅。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-121">Example: Tailspin sells subscriptions to its SaaS application.</span></span> <span data-ttu-id="8fb0e-122">Contoso 和 Fabrikam 注册了该应用。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-122">Contoso and Fabrikam sign up for the app.</span></span> <span data-ttu-id="8fb0e-123">当 Alice (`alice@contoso`) 登录时，该应用程序应该知道 Alice 属于 Contoso。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-123">When Alice (`alice@contoso`) signs in, the application should know that Alice is part of Contoso.</span></span>

* <span data-ttu-id="8fb0e-124">Alice 应该有权访问 Contoso 数据。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-124">Alice *should* have access to Contoso data.</span></span>
* <span data-ttu-id="8fb0e-125">Alice 不应有权访问 Fabrikam 数据。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-125">Alice *should not* have access to Fabrikam data.</span></span>

<span data-ttu-id="8fb0e-126">本指南将会介绍如何使用 [Azure Active Directory][AzureAD] (Azure AD) 处理登录和身份验证，从而在多租户应用程序中管理用户标识。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-126">This guidance will show you how to manage user identities in a multitenant application, using [Azure Active Directory][AzureAD] (Azure AD) to handle sign-in and authentication.</span></span>

## <a name="what-is-multitenancy"></a><span data-ttu-id="8fb0e-127">什么是多租户？</span><span class="sxs-lookup"><span data-stu-id="8fb0e-127">What is multitenancy?</span></span>
<span data-ttu-id="8fb0e-128">租户是一组用户。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-128">A *tenant* is a group of users.</span></span> <span data-ttu-id="8fb0e-129">在 SaaS 应用程序中，租户是应用程序的订阅者或客户。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-129">In a SaaS application, the tenant is a subscriber or customer of the application.</span></span> <span data-ttu-id="8fb0e-130">多租户是一种体系结构，其中的多个租户共享应用的同一个物理实例。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-130">*Multitenancy* is an architecture where multiple tenants share the same physical instance of the app.</span></span> <span data-ttu-id="8fb0e-131">尽管租户共享物理资源（例如 VM 或存储），但每个租户可获取自身的应用逻辑实例。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-131">Although tenants share physical resources (such as VMs or storage), each tenant gets its own logical instance of the app.</span></span>

<span data-ttu-id="8fb0e-132">通常，应用程序数据在租户中的用户之间共享，而不是与其他租户共享。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-132">Typically, application data is shared among the users within a tenant, but not with other tenants.</span></span>

![多租户](./images/multitenant.png)

<span data-ttu-id="8fb0e-134">现在我们将此体系结构与单租户体系结构进行比较，后者的每个租户具有专用的物理实例。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-134">Compare this architecture with a single-tenant architecture, where each tenant has a dedicated physical instance.</span></span> <span data-ttu-id="8fb0e-135">在单租户体系结构中，可通过运转应用的新实例来添加租户。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-135">In a single-tenant architecture, you add tenants by spinning up new instances of the app.</span></span>

![单租户](./images/single-tenant.png)

### <a name="multitenancy-and-horizontal-scaling"></a><span data-ttu-id="8fb0e-137">多租户和水平扩展</span><span class="sxs-lookup"><span data-stu-id="8fb0e-137">Multitenancy and horizontal scaling</span></span>
<span data-ttu-id="8fb0e-138">若要在云中实现扩展，常见的做法是添加更多的物理实例。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-138">To achieve scale in the cloud, it’s common to add more physical instances.</span></span> <span data-ttu-id="8fb0e-139">这称为“水平扩展”或“横向扩展”。以某个 Web 应用为例。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-139">This is known as *horizontal scaling* or *scaling out*. Consider a web app.</span></span> <span data-ttu-id="8fb0e-140">若要处理更多流量，可以添加更多的服务器 VM 并将其放在负载均衡器的后面。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-140">To handle more traffic, you can add more server VMs and put them behind a load balancer.</span></span> <span data-ttu-id="8fb0e-141">每个 VM 运行该 Web 应用的独立物理实例。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-141">Each VM runs a separate physical instance of the web app.</span></span>

![对网站进行负载均衡](./images/load-balancing.png)

<span data-ttu-id="8fb0e-143">可将任何请求路由到任何实例。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-143">Any request can be routed to any instance.</span></span> <span data-ttu-id="8fb0e-144">整个系统充当单个逻辑实例。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-144">Together, the system functions as a single logical instance.</span></span> <span data-ttu-id="8fb0e-145">可以解除 VM 或运转新的 VM，不会对用户造成影响。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-145">You can tear down a VM or spin up a new VM, without affecting users.</span></span> <span data-ttu-id="8fb0e-146">在此体系结构中，每个物理实例采用多租户体系结构，我们可以通过添加更多的实例进行扩展。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-146">In this architecture, each physical instance is multi-tenant, and you scale by adding more instances.</span></span> <span data-ttu-id="8fb0e-147">某个实例出现故障不应影响任何租户。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-147">If one instance goes down, it should not affect any tenant.</span></span>

## <a name="identity-in-a-multitenant-app"></a><span data-ttu-id="8fb0e-148">多租户应用中的标识</span><span class="sxs-lookup"><span data-stu-id="8fb0e-148">Identity in a multitenant app</span></span>
<span data-ttu-id="8fb0e-149">在多租户应用中，必须考虑租户上下文中的用户。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-149">In a multitenant app, you must consider users in the context of tenants.</span></span>

<span data-ttu-id="8fb0e-150">**身份验证**</span><span class="sxs-lookup"><span data-stu-id="8fb0e-150">**Authentication**</span></span>

* <span data-ttu-id="8fb0e-151">用户使用其组织凭据登录到应用。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-151">Users sign into the app with their organization credentials.</span></span> <span data-ttu-id="8fb0e-152">他们无需为该应用创建新的用户配置文件。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-152">They don't have to create new user profiles for the app.</span></span>
* <span data-ttu-id="8fb0e-153">同一组织中的用户属于同一个租户。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-153">Users within the same organization are part of the same tenant.</span></span>
* <span data-ttu-id="8fb0e-154">当某个用户登录时，应用程序知道该用户属于哪个租户。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-154">When a user signs in, the application knows which tenant the user belongs to.</span></span>

<span data-ttu-id="8fb0e-155">**授权**</span><span class="sxs-lookup"><span data-stu-id="8fb0e-155">**Authorization**</span></span>

* <span data-ttu-id="8fb0e-156">为用户操作（例如查看资源）授权时，该应用必须考虑该用户的租户。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-156">When authorizing a user's actions (say, viewing a resource), the app must take into account the user's tenant.</span></span>
* <span data-ttu-id="8fb0e-157">可在应用程序中为用户分配角色，例如“管理员”或“标准用户”。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-157">Users might be assigned roles within the application, such as "Admin" or "Standard User".</span></span> <span data-ttu-id="8fb0e-158">角色分配应该由客户管理，而不是由 SaaS 提供商管理。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-158">Role assignments should be managed by the customer, not by the SaaS provider.</span></span>

<span data-ttu-id="8fb0e-159">**示例。**</span><span class="sxs-lookup"><span data-stu-id="8fb0e-159">**Example.**</span></span> <span data-ttu-id="8fb0e-160">Contoso 员工 Alice 在其浏览器中导航到应用程序，并单击“登录”按钮。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-160">Alice, an employee at Contoso, navigates to the application in her browser and clicks the “Log in” button.</span></span> <span data-ttu-id="8fb0e-161">她重定向到了登录屏幕，并在其中输入了其企业凭据（用户名和密码）。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-161">She is redirected to a login screen where she enters her corporate credentials (username and password).</span></span> <span data-ttu-id="8fb0e-162">此时，她以 `alice@contoso.com` 的身份登录到了应用。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-162">At this point, she is logged into the app as `alice@contoso.com`.</span></span> <span data-ttu-id="8fb0e-163">应用程序也知道 Alice 是此应用程序的管理员用户。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-163">The application also knows that Alice is an admin user for this application.</span></span> <span data-ttu-id="8fb0e-164">由于她是管理员，因此可以查看属于 Contoso 的所有资源的列表。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-164">Because she is an admin, she can see a list of all the resources that belong to Contoso.</span></span> <span data-ttu-id="8fb0e-165">但是，她无法查看 Fabrikam 的资源，因为她只是她所在租户中的管理员。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-165">However, she cannot view Fabrikam's resources, because she is an admin only within her tenant.</span></span>

<span data-ttu-id="8fb0e-166">本指南将会专门介绍如何使用 Azure AD 进行标识管理。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-166">In this guidance, we'll look specifically at using Azure AD for identity management.</span></span>

* <span data-ttu-id="8fb0e-167">假设客户将其用户配置文件存储在 Azure AD（包括 Office365 和 Dynamics CRM 租户）中</span><span class="sxs-lookup"><span data-stu-id="8fb0e-167">We assume the customer stores their user profiles in Azure AD (including Office365 and Dynamics CRM tenants)</span></span>
* <span data-ttu-id="8fb0e-168">具有本地 Active Directory (AD) 的客户可以使用 [Azure AD Connect][ADConnect] 将其本地 AD 与 Azure AD 同步。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-168">Customers with on-premise Active Directory (AD) can use [Azure AD Connect][ADConnect] to sync their on-premise AD with Azure AD.</span></span>

<span data-ttu-id="8fb0e-169">如果具有本地 AD 的客户无法使用 Azure AD Connect（出于企业 IT 政策或其他方面的原因），SaaS 提供商可以通过 Active Directory 联合身份验证服务 (AD FS) 来与客户的 AD 联合。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-169">If a customer with on-premise AD cannot use Azure AD Connect (due to corporate IT policy or other reasons), the SaaS provider can federate with the customer's AD through Active Directory Federation Services (AD FS).</span></span> <span data-ttu-id="8fb0e-170">[与客户的 AD FS 联合]中介绍了此选项。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-170">This option is described in [Federating with a customer's AD FS].</span></span>

<span data-ttu-id="8fb0e-171">本指南不考虑多租户的其他方面，例如数据分区、每个租户的配置，等等。</span><span class="sxs-lookup"><span data-stu-id="8fb0e-171">This guidance does not consider other aspects of multitenancy such as data partitioning, per-tenant configuration, and so forth.</span></span>

<span data-ttu-id="8fb0e-172">[**下一篇**][tailpin]</span><span class="sxs-lookup"><span data-stu-id="8fb0e-172">[**Next**][tailpin]</span></span>



<!-- Links -->
[ADConnect]: /azure/active-directory/active-directory-aadconnect
[AzureAD]: /azure/active-directory

[与客户的 AD FS 联合]: adfs.md
[tailpin]: tailspin.md

[running-the-app]: ./run-the-app.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
