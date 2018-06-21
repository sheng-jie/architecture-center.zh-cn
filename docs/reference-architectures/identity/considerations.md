---
title: 选择用于将本地 Active Directory 与 Azure 相集成的解决方案。
description: 比较用于将本地 Active Directory 与 Azure 相集成的参考体系结构。
ms.date: 04/06/2017
ms.openlocfilehash: 413a5463d90547197c4b6834d353b4ecf61483ee
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
ms.locfileid: "24541651"
---
# <a name="choose-a-solution-for-integrating-on-premises-active-directory-with-azure"></a>选择用于将本地 Active Directory 与 Azure 相集成的解决方案

本文比较用于将本地 Active Directory (AD) 环境与 Azure 相集成的各种选项。 我们为每个选项提供了参考体系结构和可部署的解决方案。

许多组织使用 [Active Directory 域服务 (AD DS)][active-directory-domain-services] 来验证与用户、计算机、应用程序或包含在安全边界中的其他资源相关联的身份。 目录和标识服务通常托管在本地，但如果应用程序有一部分托管本地，还有一部分托管在 Azure 中，则将来自 Azure 的身份验证请求发回到本地时，可能会出现延迟。 在 Azure 中实施目录和标识服务可以降低此延迟。

Azure 提供两种解决方案用于在 Azure 中实施目录和标识服务： 

* 使用 [Azure AD][azure-active-directory] 可在云中创建 Active Directory 域，并将其连接到本地 Active Directory 域。 [Azure AD Connect][azure-ad-connect] 可将本地目录与 Azure AD 相集成。

* 通过在 Azure 中部署一个运行 AD DS（作为域控制器）的 VM，将现有的本地 Active Directory 基础结构扩展到 Azure。 如果本地网络和 Azure 虚拟网络 (VNet) 通过 VPN 或 ExpressRoute 连接进行连接，则往往会使用此体系结构。 此体系结构存在多种可能的变通形式： 

    - 在 Azure 中创建一个域，并将其加入本地 AD 林。
    - 在 Azure 中创建受本地林中的域信任的独立林。
    - 将 Active Directory 联合身份验证服务 (AD FS) 部署复制到 Azure。 

后续部分更详细介绍了其中的每个选项。

## <a name="integrate-your-on-premises-domains-with-azure-ad"></a>将本地域与 Azure AD 集成

使用 Azure Active Directory (Azure AD) 在 Azure 中创建域，并将其链接到本地 AD 域。 

Azure AD 目录不是本地目录的扩展， 而是包含相同对象和标识的副本。 在本地对这些项所做的更改会复制到 Azure AD，但在 Azure AD 中所做的更改不会复制回到本地域。

也可以只使用 Azure AD 而不使用本地目录。 在这种情况下，Azure AD 充当所有标识信息的主要源，而不包含从本地目录复制的数据。


**优点**

* 无需在云中维护 AD 基础结构。 Azure AD 完全由 Microsoft 管理和维护。
* Azure AD 提供本地所提供的相同标识信息。
* 身份验证可以在 Azure 中发生，从而减少了外部应用程序和用户访问本地域的需要。

**挑战**

* 标识服务限制为由用户和组访问。 无法对服务和计算机帐户进行身份验证。
* 必须配置与本地域之间的连接，使 Azure AD 目录保持同步。 
* 可能需要重新编写应用程序才能通过 Azure AD 启用身份验证。

**[了解详细信息...][aad]**

## <a name="ad-ds-in-azure-joined-to-an-on-premises-forest"></a>Azure 中已加入本地林的 AD DS

将 AD 域服务 (AD DS) 服务器部署到 Azure。 在 Azure 中创建一个域，并将其加入本地 AD 林。 

如果需要使用 Azure AD 当前未实现的 AD DS 功能，请考虑此选项。 

**优点**

* 可访问本地所提供的相同标识信息。
* 可对本地和 Azure 中的用户、服务和计算机帐户进行身份验证。
* 无需管理独立的 AD 林。 Azure 中的域可属于本地林。
* 可对 Azure 中的域应用本地组策略对象所定义的组策略。

**挑战**

* 必须在云中部署和管理自己的 AD DS 服务器与域。
* 云中的域服务器与本地运行的服务器之间可能存在一定的同步延迟。

**[了解详细信息...][ad-ds]**

## <a name="ad-ds-in-azure-with-a-separate-forest"></a>Azure 中使用独立林的 AD DS

将 AD 域服务 (AD DS) 服务器部署到 Azure，但创建一个独立于本地林的独立 Active Directory [林][ad-forest-defn]。 此林受本地林中的域的信任。

此体系结构的典型用途包括为云中拥有的对象和标识维护安全隔离，以及将各个域从本地迁移到云。

**优点**

* 可以实施本地标识和隔离仅限 Azure 的标识。
* 无需从本地 AD 林复制到 Azure。

**挑战**

* 在 Azure 中针对本地标识执行身份验证需要额外与本地 AD 服务器建立网络跃点。
* 必须在云中部署 AD DS 服务器和林，并在林之间建立适当的信任关系。

**[了解详细信息...][ad-ds-forest]**

## <a name="extend-ad-fs-to-azure"></a>将 AD FS 扩展到 Azure

将 Active Directory 联合身份验证服务 (AD FS) 部署复制到 Azure，以针对 Azure 中运行的组件执行联合身份验证和授权。 

此体系结构的典型用途：

* 对合作伙伴组织中的用户执行身份验证和授权。
* 允许用户从组织防火墙外部运行的 Web 浏览器进行身份验证。
* 允许用户通过已授权的外部设备（例如移动设备）建立连接。 

**优点**

* 可以利用声明感知的应用程序。
* 能够信任外部合作伙伴，从而完成身份验证。
* 与大量的身份验证协议兼容。

**挑战**

* 必须在 Azure 中部署自己的 AD DS、AD FS 和 AD FS Web 应用程序代理服务器。
* 此体系结构的配置可能比较复杂。

**[了解详细信息...][adfs]**

<!-- links -->

[aad]: ./azure-ad.md
[ad-ds]: ./adds-extend-domain.md
[ad-ds-forest]: ./adds-forest.md
[ad-forest-defn]: https://msdn.microsoft.com/library/ms676906.aspx
[adfs]: ./adfs.md

[active-directory-domain-services]: https://technet.microsoft.com/library/dd448614.aspx
[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/active-directory-aadconnect
