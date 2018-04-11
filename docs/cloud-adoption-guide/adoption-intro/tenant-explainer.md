---
title: 说明：Azure Active Directory 租户是什么？
description: 介绍了在 Azure 中提供标识即服务 (IDaaS) 的 Azure Active Directory 的内部运行
author: petertay
ms.openlocfilehash: ce5a33b92047e1f360eee8fcbc7a726bcf8cd19f
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/09/2018
---
# <a name="explainer-what-is-an-azure-active-directory-tenant"></a>说明：Azure Active Directory 租户是什么？

在 [Azure 如何工作？](azure-explainer.md)说明文章中，我们已知道了 Azure 是在虚拟化硬件和软件中以用户身份运行的服务器和网络硬件的集合。 我们还知道了这些服务器中有一些运行分布式业务流程应用程序来管理 Azure 资源的创建、读取、更新和删除。

但是，如你所料，Azure 不允许任何人对资源执行这些操作之一。 Azure 仅允许使用称作 **Azure Active Directory** (Azure AD) 的可信数字标识服务来访问这些操作。 Azure AD 存储着用户名、密码、配置文件数据和其他信息。 Azure AD 用户划分到各个**租户**中。 租户是一个逻辑构造，它代表通常与组织关联的一个安全专用的 Azure AD 实例。

为了创建租户，Azure 需要一个**特权帐户**。 此特权帐户与一个 Azure 帐户或企业协议相关联。 这两者都是计费构造，并且不存储在 Azure AD 中 &mdash; 这些帐户存储在高度安全的计费数据库中。 

在创建租户后，会为租户生成一个**租户 ID** 并且会将其保存在高度安全的内部 Azure AD 数据库中。 然后，特权帐户所有者可以登录到 Azure 门户并向新创建的 Azure AD 租户添加用户。 

大多数企业已经有至少一个标识管理服务，通常是 Active Directory 域服务 (AD DS)。 Azure AD 能够同步或联合来自 AD DS 的用户标识，因此企业不需要分别在两个环境中管理标识。 在有关数字标识的中级和高级采用阶段的文章中将更详细地介绍这方面的内容。

## <a name="next-steps"></a>后续步骤

* 现在，你已了解了 Azure AD 租户，基础采用阶段中的第一个步骤是了解[如何获取 Azure Active Directory 租户][how-to-get-aad-tenant]。 然后，查看 [Azure AD 租户设计指南](tenant.md)。

<!-- Links -->
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json