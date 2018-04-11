---
title: 选择用于将本地 Active Directory 与 Azure 相集成的解决方案。
description: 比较用于将本地 Active Directory 与 Azure 相集成的参考体系结构。
ms.date: 04/06/2017
ms.openlocfilehash: 413a5463d90547197c4b6834d353b4ecf61483ee
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="choose-a-solution-for-integrating-on-premises-active-directory-with-azure"></a><span data-ttu-id="e9654-103">选择用于将本地 Active Directory 与 Azure 相集成的解决方案</span><span class="sxs-lookup"><span data-stu-id="e9654-103">Choose a solution for integrating on-premises Active Directory with Azure</span></span>

<span data-ttu-id="e9654-104">本文比较用于将本地 Active Directory (AD) 环境与 Azure 相集成的各种选项。</span><span class="sxs-lookup"><span data-stu-id="e9654-104">This article compares options for integrating your on-premises Active Directory (AD) environment with an Azure network.</span></span> <span data-ttu-id="e9654-105">我们为每个选项提供了参考体系结构和可部署的解决方案。</span><span class="sxs-lookup"><span data-stu-id="e9654-105">We provide a reference architecture and a deployable solution for each option.</span></span>

<span data-ttu-id="e9654-106">许多组织使用 [Active Directory 域服务 (AD DS)][active-directory-domain-services] 来验证与用户、计算机、应用程序或包含在安全边界中的其他资源相关联的身份。</span><span class="sxs-lookup"><span data-stu-id="e9654-106">Many organizations use [Active Directory Domain Services (AD DS)][active-directory-domain-services] to authenticate identities associated with users, computers, applications, or other resources that are included in a security boundary.</span></span> <span data-ttu-id="e9654-107">目录和标识服务通常托管在本地，但如果应用程序有一部分托管本地，还有一部分托管在 Azure 中，则将来自 Azure 的身份验证请求发回到本地时，可能会出现延迟。</span><span class="sxs-lookup"><span data-stu-id="e9654-107">Directory and identity services are typically hosted on-premises, but if your application is hosted partly on-premises and partly in Azure, there may be latency sending authentication requests from Azure back to on-premises.</span></span> <span data-ttu-id="e9654-108">在 Azure 中实施目录和标识服务可以降低此延迟。</span><span class="sxs-lookup"><span data-stu-id="e9654-108">Implementing directory and identity services in Azure can reduce this latency.</span></span>

<span data-ttu-id="e9654-109">Azure 提供两种解决方案用于在 Azure 中实施目录和标识服务：</span><span class="sxs-lookup"><span data-stu-id="e9654-109">Azure provides two solutions for implementing directory and identity services in Azure:</span></span> 

* <span data-ttu-id="e9654-110">使用 [Azure AD][azure-active-directory] 可在云中创建 Active Directory 域，并将其连接到本地 Active Directory 域。</span><span class="sxs-lookup"><span data-stu-id="e9654-110">Use [Azure AD][azure-active-directory] to create an Active Directory domain in the cloud and connect it to your on-premises Active Directory domain.</span></span> <span data-ttu-id="e9654-111">[Azure AD Connect][azure-ad-connect] 可将本地目录与 Azure AD 相集成。</span><span class="sxs-lookup"><span data-stu-id="e9654-111">[Azure AD Connect][azure-ad-connect] integrates your on-premises directories with Azure AD.</span></span>

* <span data-ttu-id="e9654-112">通过在 Azure 中部署一个运行 AD DS（作为域控制器）的 VM，将现有的本地 Active Directory 基础结构扩展到 Azure。</span><span class="sxs-lookup"><span data-stu-id="e9654-112">Extend your existing on-premises Active Directory infrastructure to Azure, by deploying a VM in Azure that runs AD DS as a domain controller.</span></span> <span data-ttu-id="e9654-113">如果本地网络和 Azure 虚拟网络 (VNet) 通过 VPN 或 ExpressRoute 连接进行连接，则往往会使用此体系结构。</span><span class="sxs-lookup"><span data-stu-id="e9654-113">This architecture is more common when the on-premises network and the Azure virtual network (VNet) are connected by a VPN or ExpressRoute connection.</span></span> <span data-ttu-id="e9654-114">此体系结构存在多种可能的变通形式：</span><span class="sxs-lookup"><span data-stu-id="e9654-114">Several variations of this architecture are possible:</span></span> 

    - <span data-ttu-id="e9654-115">在 Azure 中创建一个域，并将其加入本地 AD 林。</span><span class="sxs-lookup"><span data-stu-id="e9654-115">Create a domain in Azure and join it to your on-premises AD forest.</span></span>
    - <span data-ttu-id="e9654-116">在 Azure 中创建受本地林中的域信任的独立林。</span><span class="sxs-lookup"><span data-stu-id="e9654-116">Create a separate forest in Azure that is trusted by domains in your on-premises forest.</span></span>
    - <span data-ttu-id="e9654-117">将 Active Directory 联合身份验证服务 (AD FS) 部署复制到 Azure。</span><span class="sxs-lookup"><span data-stu-id="e9654-117">Replicate an Active Directory Federation Services (AD FS) deployment to Azure.</span></span> 

<span data-ttu-id="e9654-118">后续部分更详细介绍了其中的每个选项。</span><span class="sxs-lookup"><span data-stu-id="e9654-118">The next sections describe each of these options in more detail.</span></span>

## <a name="integrate-your-on-premises-domains-with-azure-ad"></a><span data-ttu-id="e9654-119">将本地域与 Azure AD 集成</span><span class="sxs-lookup"><span data-stu-id="e9654-119">Integrate your on-premises domains with Azure AD</span></span>

<span data-ttu-id="e9654-120">使用 Azure Active Directory (Azure AD) 在 Azure 中创建域，并将其链接到本地 AD 域。</span><span class="sxs-lookup"><span data-stu-id="e9654-120">Use Azure Active Directory (Azure AD) to create a domain in Azure and link it to an on-premises AD domain.</span></span> 

<span data-ttu-id="e9654-121">Azure AD 目录不是本地目录的扩展，</span><span class="sxs-lookup"><span data-stu-id="e9654-121">The Azure AD directory is not an extension of an on-premises directory.</span></span> <span data-ttu-id="e9654-122">而是包含相同对象和标识的副本。</span><span class="sxs-lookup"><span data-stu-id="e9654-122">Rather, it's a copy that contains the same objects and identities.</span></span> <span data-ttu-id="e9654-123">在本地对这些项所做的更改会复制到 Azure AD，但在 Azure AD 中所做的更改不会复制回到本地域。</span><span class="sxs-lookup"><span data-stu-id="e9654-123">Changes made to these items on-premises are copied to Azure AD, but changes made in Azure AD are not replicated back to the on-premises domain.</span></span>

<span data-ttu-id="e9654-124">也可以只使用 Azure AD 而不使用本地目录。</span><span class="sxs-lookup"><span data-stu-id="e9654-124">You can also use Azure AD without using an on-premises directory.</span></span> <span data-ttu-id="e9654-125">在这种情况下，Azure AD 充当所有标识信息的主要源，而不包含从本地目录复制的数据。</span><span class="sxs-lookup"><span data-stu-id="e9654-125">In this case, Azure AD acts as the primary source of all identity information, rather than containing data replicated from an on-premises directory.</span></span>


<span data-ttu-id="e9654-126">**优点**</span><span class="sxs-lookup"><span data-stu-id="e9654-126">**Benefits**</span></span>

* <span data-ttu-id="e9654-127">无需在云中维护 AD 基础结构。</span><span class="sxs-lookup"><span data-stu-id="e9654-127">You don't need to maintain an AD infrastructure in the cloud.</span></span> <span data-ttu-id="e9654-128">Azure AD 完全由 Microsoft 管理和维护。</span><span class="sxs-lookup"><span data-stu-id="e9654-128">Azure AD is entirely managed and maintained by Microsoft.</span></span>
* <span data-ttu-id="e9654-129">Azure AD 提供本地所提供的相同标识信息。</span><span class="sxs-lookup"><span data-stu-id="e9654-129">Azure AD provides the same identity information that is available on-premises.</span></span>
* <span data-ttu-id="e9654-130">身份验证可以在 Azure 中发生，从而减少了外部应用程序和用户访问本地域的需要。</span><span class="sxs-lookup"><span data-stu-id="e9654-130">Authentication can happen in Azure, reducing the need for external applications and users to contact the on-premises domain.</span></span>

<span data-ttu-id="e9654-131">**挑战**</span><span class="sxs-lookup"><span data-stu-id="e9654-131">**Challenges**</span></span>

* <span data-ttu-id="e9654-132">标识服务限制为由用户和组访问。</span><span class="sxs-lookup"><span data-stu-id="e9654-132">Identity services are limited to users and groups.</span></span> <span data-ttu-id="e9654-133">无法对服务和计算机帐户进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="e9654-133">There is no ability to authenticate service and computer accounts.</span></span>
* <span data-ttu-id="e9654-134">必须配置与本地域之间的连接，使 Azure AD 目录保持同步。</span><span class="sxs-lookup"><span data-stu-id="e9654-134">You must configure connectivity with your on-premises domain to keep the Azure AD directory synchronized.</span></span> 
* <span data-ttu-id="e9654-135">可能需要重新编写应用程序才能通过 Azure AD 启用身份验证。</span><span class="sxs-lookup"><span data-stu-id="e9654-135">Applications may need to be rewritten to enable authentication through Azure AD.</span></span>

<span data-ttu-id="e9654-136">**[了解详细信息...][aad]**</span><span class="sxs-lookup"><span data-stu-id="e9654-136">**[Read more...][aad]**</span></span>

## <a name="ad-ds-in-azure-joined-to-an-on-premises-forest"></a><span data-ttu-id="e9654-137">Azure 中已加入本地林的 AD DS</span><span class="sxs-lookup"><span data-stu-id="e9654-137">AD DS in Azure joined to an on-premises forest</span></span>

<span data-ttu-id="e9654-138">将 AD 域服务 (AD DS) 服务器部署到 Azure。</span><span class="sxs-lookup"><span data-stu-id="e9654-138">Deploy AD Domain Services (AD DS) servers to Azure.</span></span> <span data-ttu-id="e9654-139">在 Azure 中创建一个域，并将其加入本地 AD 林。</span><span class="sxs-lookup"><span data-stu-id="e9654-139">Create a domain in Azure and join it to your on-premises AD forest.</span></span> 

<span data-ttu-id="e9654-140">如果需要使用 Azure AD 当前未实现的 AD DS 功能，请考虑此选项。</span><span class="sxs-lookup"><span data-stu-id="e9654-140">Consider this option if you need to use AD DS features that are not currently implemented by Azure AD.</span></span> 

<span data-ttu-id="e9654-141">**优点**</span><span class="sxs-lookup"><span data-stu-id="e9654-141">**Benefits**</span></span>

* <span data-ttu-id="e9654-142">可访问本地所提供的相同标识信息。</span><span class="sxs-lookup"><span data-stu-id="e9654-142">Provides access to the same identity information that is available on-premises.</span></span>
* <span data-ttu-id="e9654-143">可对本地和 Azure 中的用户、服务和计算机帐户进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="e9654-143">You can authenticate user, service, and computer accounts on-premises and in Azure.</span></span>
* <span data-ttu-id="e9654-144">无需管理独立的 AD 林。</span><span class="sxs-lookup"><span data-stu-id="e9654-144">You don't need to manage a separate AD forest.</span></span> <span data-ttu-id="e9654-145">Azure 中的域可属于本地林。</span><span class="sxs-lookup"><span data-stu-id="e9654-145">The domain in Azure can belong to the on-premises forest.</span></span>
* <span data-ttu-id="e9654-146">可对 Azure 中的域应用本地组策略对象所定义的组策略。</span><span class="sxs-lookup"><span data-stu-id="e9654-146">You can apply group policy defined by on-premises Group Policy Objects to the domain in Azure.</span></span>

<span data-ttu-id="e9654-147">**挑战**</span><span class="sxs-lookup"><span data-stu-id="e9654-147">**Challenges**</span></span>

* <span data-ttu-id="e9654-148">必须在云中部署和管理自己的 AD DS 服务器与域。</span><span class="sxs-lookup"><span data-stu-id="e9654-148">You must deploy and manage your own AD DS servers and domain in the cloud.</span></span>
* <span data-ttu-id="e9654-149">云中的域服务器与本地运行的服务器之间可能存在一定的同步延迟。</span><span class="sxs-lookup"><span data-stu-id="e9654-149">There may be some synchronization latency between the domain servers in the cloud and the servers running on-premises.</span></span>

<span data-ttu-id="e9654-150">**[了解详细信息...][ad-ds]**</span><span class="sxs-lookup"><span data-stu-id="e9654-150">**[Read more...][ad-ds]**</span></span>

## <a name="ad-ds-in-azure-with-a-separate-forest"></a><span data-ttu-id="e9654-151">Azure 中使用独立林的 AD DS</span><span class="sxs-lookup"><span data-stu-id="e9654-151">AD DS in Azure with a separate forest</span></span>

<span data-ttu-id="e9654-152">将 AD 域服务 (AD DS) 服务器部署到 Azure，但创建一个独立于本地林的独立 Active Directory [林][ad-forest-defn]。</span><span class="sxs-lookup"><span data-stu-id="e9654-152">Deploy AD Domain Services (AD DS) servers to Azure, but create a separate Active Directory [forest][ad-forest-defn] that is separate from the on-premises forest.</span></span> <span data-ttu-id="e9654-153">此林受本地林中的域的信任。</span><span class="sxs-lookup"><span data-stu-id="e9654-153">This forest is trusted by domains in your on-premises forest.</span></span>

<span data-ttu-id="e9654-154">此体系结构的典型用途包括为云中拥有的对象和标识维护安全隔离，以及将各个域从本地迁移到云。</span><span class="sxs-lookup"><span data-stu-id="e9654-154">Typical uses for this architecture include maintaining security separation for objects and identities held in the cloud, and migrating individual domains from on-premises to the cloud.</span></span>

<span data-ttu-id="e9654-155">**优点**</span><span class="sxs-lookup"><span data-stu-id="e9654-155">**Benefits**</span></span>

* <span data-ttu-id="e9654-156">可以实施本地标识和隔离仅限 Azure 的标识。</span><span class="sxs-lookup"><span data-stu-id="e9654-156">You can implement on-premises identities and separate Azure-only identities.</span></span>
* <span data-ttu-id="e9654-157">无需从本地 AD 林复制到 Azure。</span><span class="sxs-lookup"><span data-stu-id="e9654-157">You don't need to replicate from the on-premises AD forest to Azure.</span></span>

<span data-ttu-id="e9654-158">**挑战**</span><span class="sxs-lookup"><span data-stu-id="e9654-158">**Challenges**</span></span>

* <span data-ttu-id="e9654-159">在 Azure 中针对本地标识执行身份验证需要额外与本地 AD 服务器建立网络跃点。</span><span class="sxs-lookup"><span data-stu-id="e9654-159">Authentication within Azure for on-premises identities requires extra network hops to the on-premises AD servers.</span></span>
* <span data-ttu-id="e9654-160">必须在云中部署 AD DS 服务器和林，并在林之间建立适当的信任关系。</span><span class="sxs-lookup"><span data-stu-id="e9654-160">You must deploy your own AD DS servers and forest in the cloud, and establish the appropriate trust relationships between forests.</span></span>

<span data-ttu-id="e9654-161">**[了解详细信息...][ad-ds-forest]**</span><span class="sxs-lookup"><span data-stu-id="e9654-161">**[Read more...][ad-ds-forest]**</span></span>

## <a name="extend-ad-fs-to-azure"></a><span data-ttu-id="e9654-162">将 AD FS 扩展到 Azure</span><span class="sxs-lookup"><span data-stu-id="e9654-162">Extend AD FS to Azure</span></span>

<span data-ttu-id="e9654-163">将 Active Directory 联合身份验证服务 (AD FS) 部署复制到 Azure，以针对 Azure 中运行的组件执行联合身份验证和授权。</span><span class="sxs-lookup"><span data-stu-id="e9654-163">Replicate an Active Directory Federation Services (AD FS) deployment to Azure, to perform federated authentication and authorization for components running in Azure.</span></span> 

<span data-ttu-id="e9654-164">此体系结构的典型用途：</span><span class="sxs-lookup"><span data-stu-id="e9654-164">Typical uses for this architecture:</span></span>

* <span data-ttu-id="e9654-165">对合作伙伴组织中的用户执行身份验证和授权。</span><span class="sxs-lookup"><span data-stu-id="e9654-165">Authenticate and authorize users from partner organizations.</span></span>
* <span data-ttu-id="e9654-166">允许用户从组织防火墙外部运行的 Web 浏览器进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="e9654-166">Allow users to authenticate from web browsers running outside of the organizational firewall.</span></span>
* <span data-ttu-id="e9654-167">允许用户通过已授权的外部设备（例如移动设备）建立连接。</span><span class="sxs-lookup"><span data-stu-id="e9654-167">Allow users to connect from authorized external devices such as mobile devices.</span></span> 

<span data-ttu-id="e9654-168">**优点**</span><span class="sxs-lookup"><span data-stu-id="e9654-168">**Benefits**</span></span>

* <span data-ttu-id="e9654-169">可以利用声明感知的应用程序。</span><span class="sxs-lookup"><span data-stu-id="e9654-169">You can leverage claims-aware applications.</span></span>
* <span data-ttu-id="e9654-170">能够信任外部合作伙伴，从而完成身份验证。</span><span class="sxs-lookup"><span data-stu-id="e9654-170">Provides the ability to trust external partners for authentication.</span></span>
* <span data-ttu-id="e9654-171">与大量的身份验证协议兼容。</span><span class="sxs-lookup"><span data-stu-id="e9654-171">Compatibility with large set of authentication protocols.</span></span>

<span data-ttu-id="e9654-172">**挑战**</span><span class="sxs-lookup"><span data-stu-id="e9654-172">**Challenges**</span></span>

* <span data-ttu-id="e9654-173">必须在 Azure 中部署自己的 AD DS、AD FS 和 AD FS Web 应用程序代理服务器。</span><span class="sxs-lookup"><span data-stu-id="e9654-173">You must deploy your own AD DS, AD FS, and AD FS Web Application Proxy servers in Azure.</span></span>
* <span data-ttu-id="e9654-174">此体系结构的配置可能比较复杂。</span><span class="sxs-lookup"><span data-stu-id="e9654-174">This architecture can be complex to configure.</span></span>

<span data-ttu-id="e9654-175">**[了解详细信息...][adfs]**</span><span class="sxs-lookup"><span data-stu-id="e9654-175">**[Read more...][adfs]**</span></span>

<!-- links -->

[aad]: ./azure-ad.md
[ad-ds]: ./adds-extend-domain.md
[ad-ds-forest]: ./adds-forest.md
[ad-forest-defn]: https://msdn.microsoft.com/library/ms676906.aspx
[adfs]: ./adfs.md

[active-directory-domain-services]: https://technet.microsoft.com/library/dd448614.aspx
[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/active-directory-aadconnect
