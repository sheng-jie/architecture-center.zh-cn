---
title: 多租户应用程序的标识管理
description: 有关多租户应用程序中的身份验证、授权和标识管理的最佳做法。
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
# <a name="manage-identity-in-multitenant-applications"></a>管理多租户应用程序中的标识

本系列文章介绍有关使用 Azure AD 进行身份验证和标识管理时，适用于多租户的最佳做法。

[![GitHub](../_images/github.png) 示例代码][sample application]

构建多租户应用程序时，首要的难题之一是管理用户标识，因为目前每个用户都属于某个租户。 例如：

* 用户使用其组织凭据登录。
* 用户应该有权访问其所在组织的数据，但不能访问属于其他租户的数据。
* 组织可以注册应用程序，然后将应用程序角色分配到其成员。

Azure Active Directory (Azure AD) 提供一些极佳的功能用于支持所有这些方案。

我们创建了某个多租户应用程序的完整[端到端实现][sample application]，以辅导学习本系列文章。 这些文章体现了我们在构建应用程序的过程中所获得的经验。 若要开始使用该应用程序，请参阅[运行 Surveys 应用程序][running-the-app]。

## <a name="introduction"></a>介绍

假设你要编写一个将在云中托管的企业 SaaS 应用程序。 当然，该应用程序将会包含用户：

![用户](./images/users.png)

但是，这些用户属于组织：

![组织用户](./images/org-users.png)

示例：Tailspin 在其 SaaS 应用程序中销售订阅。 Contoso 和 Fabrikam 注册了该应用。 当 Alice (`alice@contoso`) 登录时，该应用程序应该知道 Alice 属于 Contoso。

* Alice 应该有权访问 Contoso 数据。
* Alice 不应有权访问 Fabrikam 数据。

本指南将会介绍如何使用 [Azure Active Directory][AzureAD] (Azure AD) 处理登录和身份验证，从而在多租户应用程序中管理用户标识。

## <a name="what-is-multitenancy"></a>什么是多租户？
租户是一组用户。 在 SaaS 应用程序中，租户是应用程序的订阅者或客户。 多租户是一种体系结构，其中的多个租户共享应用的同一个物理实例。 尽管租户共享物理资源（例如 VM 或存储），但每个租户可获取自身的应用逻辑实例。

通常，应用程序数据在租户中的用户之间共享，而不是与其他租户共享。

![多租户](./images/multitenant.png)

现在我们将此体系结构与单租户体系结构进行比较，后者的每个租户具有专用的物理实例。 在单租户体系结构中，可通过运转应用的新实例来添加租户。

![单租户](./images/single-tenant.png)

### <a name="multitenancy-and-horizontal-scaling"></a>多租户和水平扩展
若要在云中实现扩展，常见的做法是添加更多的物理实例。 这称为“水平扩展”或“横向扩展”。以某个 Web 应用为例。 若要处理更多流量，可以添加更多的服务器 VM 并将其放在负载均衡器的后面。 每个 VM 运行该 Web 应用的独立物理实例。

![对网站进行负载均衡](./images/load-balancing.png)

可将任何请求路由到任何实例。 整个系统充当单个逻辑实例。 可以解除 VM 或运转新的 VM，不会对用户造成影响。 在此体系结构中，每个物理实例采用多租户体系结构，我们可以通过添加更多的实例进行扩展。 某个实例出现故障不应影响任何租户。

## <a name="identity-in-a-multitenant-app"></a>多租户应用中的标识
在多租户应用中，必须考虑租户上下文中的用户。

**身份验证**

* 用户使用其组织凭据登录到应用。 他们无需为该应用创建新的用户配置文件。
* 同一组织中的用户属于同一个租户。
* 当某个用户登录时，应用程序知道该用户属于哪个租户。

**授权**

* 为用户操作（例如查看资源）授权时，该应用必须考虑该用户的租户。
* 可在应用程序中为用户分配角色，例如“管理员”或“标准用户”。 角色分配应该由客户管理，而不是由 SaaS 提供商管理。

**示例。** Contoso 员工 Alice 在其浏览器中导航到应用程序，并单击“登录”按钮。 她重定向到了登录屏幕，并在其中输入了其企业凭据（用户名和密码）。 此时，她以 `alice@contoso.com` 的身份登录到了应用。 应用程序也知道 Alice 是此应用程序的管理员用户。 由于她是管理员，因此可以查看属于 Contoso 的所有资源的列表。 但是，她无法查看 Fabrikam 的资源，因为她只是她所在租户中的管理员。

本指南将会专门介绍如何使用 Azure AD 进行标识管理。

* 假设客户将其用户配置文件存储在 Azure AD（包括 Office365 和 Dynamics CRM 租户）中
* 具有本地 Active Directory (AD) 的客户可以使用 [Azure AD Connect][ADConnect] 将其本地 AD 与 Azure AD 同步。

如果具有本地 AD 的客户无法使用 Azure AD Connect（出于企业 IT 政策或其他方面的原因），SaaS 提供商可以通过 Active Directory 联合身份验证服务 (AD FS) 来与客户的 AD 联合。 [与客户的 AD FS 联合]中介绍了此选项。

本指南不考虑多租户的其他方面，例如数据分区、每个租户的配置，等等。

[**下一篇**][tailpin]



<!-- Links -->
[ADConnect]: /azure/active-directory/active-directory-aadconnect
[AzureAD]: /azure/active-directory

[与客户的 AD FS 联合]: adfs.md
[tailpin]: tailspin.md

[running-the-app]: ./run-the-app.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
