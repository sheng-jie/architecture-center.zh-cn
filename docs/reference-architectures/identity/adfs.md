---
title: "在 Azure 中实现 Active Directory 联合身份验证服务 (AD FS)"
description: "如何在 Azure 中实现采用 Active Directory 联合身份验证服务授权的安全混合网络体系结构。\n指南,vpn 网关,ExpressRoute,负载均衡器,虚拟网络,active-directory"
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Identity management
pnp.series.prev: adds-forest
cardTitle: Extend AD FS to Azure
ms.openlocfilehash: b8c9ae0621c087c68d449dd13e60046104c01513
ms.sourcegitcommit: 8ab30776e0c4cdc16ca0dcc881960e3108ad3e94
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/08/2017
---
# <a name="extend-active-directory-federation-services-ad-fs-to-azure"></a><span data-ttu-id="f1d83-104">将 Active Directory 联合身份验证服务 (AD FS) 扩展到 Azure</span><span class="sxs-lookup"><span data-stu-id="f1d83-104">Extend Active Directory Federation Services (AD FS) to Azure</span></span>

<span data-ttu-id="f1d83-105">此参考体系结构实现一个安全混合网络，该网络将本地网络扩展到 Azure，并使用 [Active Directory 联合身份验证服务 (AD FS)][active-directory-federation-services] 为 Azure 中运行的组件执行联合身份验证和授权。</span><span class="sxs-lookup"><span data-stu-id="f1d83-105">This reference architecture implements a secure hybrid network that extends your on-premises network to Azure and uses [Active Directory Federation Services (AD FS)][active-directory-federation-services] to perform federated authentication and authorization for components running in Azure.</span></span> [<span data-ttu-id="f1d83-106">**部署此解决方案**。</span><span class="sxs-lookup"><span data-stu-id="f1d83-106">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="f1d83-107">[![0]][0]</span><span class="sxs-lookup"><span data-stu-id="f1d83-107">[![0]][0]</span></span>

<span data-ttu-id="f1d83-108">*下载此体系结构的 [Visio 文件][visio-download]。*</span><span class="sxs-lookup"><span data-stu-id="f1d83-108">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="f1d83-109">AD FS 可以在本地进行承载，但是如果应用程序是其中某些部分在 Azure 中实现的混合体，则在云中复制 AD FS 可能会更加高效。</span><span class="sxs-lookup"><span data-stu-id="f1d83-109">AD FS can be hosted on-premises, but if your application is a hybrid in which some parts are implemented in Azure, it may be more efficient to replicate AD FS in the cloud.</span></span> 

<span data-ttu-id="f1d83-110">该图显示以下方案：</span><span class="sxs-lookup"><span data-stu-id="f1d83-110">The diagram shows the following scenarios:</span></span>

* <span data-ttu-id="f1d83-111">来自合作伙伴组织的应用程序代码访问在 Azure VNet 中承载的 Web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="f1d83-111">Application code from a partner organization accesses a web application hosted inside your Azure VNet.</span></span>
* <span data-ttu-id="f1d83-112">凭据存储在 Active Directory 域服务 (DS) 中的外部已注册用户访问在 Azure VNet 中承载的 Web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="f1d83-112">An external, registered user with credentials stored inside Active Directory Domain Services (DS) accesses a web application hosted inside your Azure VNet.</span></span>
* <span data-ttu-id="f1d83-113">使用授权设备连接到 VNet 的用户执行在 Azure VNet 中承载的 Web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="f1d83-113">A user connected to your VNet using an authorized device executes a web application hosted inside your Azure VNet.</span></span>

<span data-ttu-id="f1d83-114">此体系结构的典型用途包括：</span><span class="sxs-lookup"><span data-stu-id="f1d83-114">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="f1d83-115">在本地运行一部分工作负荷，在 Azure 中运行一部分工作负荷的混合应用程序。</span><span class="sxs-lookup"><span data-stu-id="f1d83-115">Hybrid applications where workloads run partly on-premises and partly in Azure.</span></span>
* <span data-ttu-id="f1d83-116">使用联合授权向合作伙伴组织公开 Web 应用程序的解决方案。</span><span class="sxs-lookup"><span data-stu-id="f1d83-116">Solutions that use federated authorization to expose web applications to partner organizations.</span></span>
* <span data-ttu-id="f1d83-117">支持从组织防火墙外部运行的 Web 浏览器进行访问的系统。</span><span class="sxs-lookup"><span data-stu-id="f1d83-117">Systems that support access from web browsers running outside of the organizational firewall.</span></span>
* <span data-ttu-id="f1d83-118">使用户可以通过从授权外部设备（如远程计算机、笔记本电脑和其他移动设备）进行连接来访问 Web 应用程序的系统。</span><span class="sxs-lookup"><span data-stu-id="f1d83-118">Systems that enable users to access to web applications by connecting from authorized external devices such as remote computers, notebooks, and other mobile devices.</span></span> 

<span data-ttu-id="f1d83-119">此参考体系结构侧重于被动联合，其中由联合服务器决定如何以及何时对用户进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="f1d83-119">This reference architecture focuses on *passive federation*, in which the federation servers decide how and when to authenticate a user.</span></span> <span data-ttu-id="f1d83-120">用户在启动应用程序时提供登录信息。</span><span class="sxs-lookup"><span data-stu-id="f1d83-120">The user provides sign in information when the application is started.</span></span> <span data-ttu-id="f1d83-121">此机制最常由 Web 浏览器使用，涉及将浏览器重定向到对用户进行身份验证的站点的协议。</span><span class="sxs-lookup"><span data-stu-id="f1d83-121">This mechanism is most commonly used by web browsers and involves a protocol that redirects the browser to a site where the user authenticates.</span></span> <span data-ttu-id="f1d83-122">AD FS 还支持主动联合，其中应用程序负责提供凭据，无需进一步的用户交互，但是该方案不在此体系结构的范围内。</span><span class="sxs-lookup"><span data-stu-id="f1d83-122">AD FS also supports *active federation*, where an application takes on responsibility for supplying credentials without further user interaction, but that scenario is outside the scope of this architecture.</span></span>

<span data-ttu-id="f1d83-123">有关其他注意事项，请参阅[选择用于将本地 Active Directory 与 Azure 相集成的解决方案][considerations]。</span><span class="sxs-lookup"><span data-stu-id="f1d83-123">For additional considerations, see [Choose a solution for integrating on-premises Active Directory with Azure][considerations].</span></span> 

## <a name="architecture"></a><span data-ttu-id="f1d83-124">体系结构</span><span class="sxs-lookup"><span data-stu-id="f1d83-124">Architecture</span></span>

<span data-ttu-id="f1d83-125">此体系结构扩展在[将 AD DS 扩展到 Azure][extending-ad-to-azure] 中介绍的实现。</span><span class="sxs-lookup"><span data-stu-id="f1d83-125">This architecture extends the implementation described in [Extending AD DS to Azure][extending-ad-to-azure].</span></span> <span data-ttu-id="f1d83-126">它包含以下组件。</span><span class="sxs-lookup"><span data-stu-id="f1d83-126">It contains the followign components.</span></span>

* <span data-ttu-id="f1d83-127">**AD DS 子网**。</span><span class="sxs-lookup"><span data-stu-id="f1d83-127">**AD DS subnet**.</span></span> <span data-ttu-id="f1d83-128">AD DS 服务器包含在其自己的子网中，具有充当防火墙的网络安全组 (NSG) 规则。</span><span class="sxs-lookup"><span data-stu-id="f1d83-128">The AD DS servers are contained in their own subnet with network security group (NSG) rules acting as a firewall.</span></span>

* <span data-ttu-id="f1d83-129">**AD DS 服务器**。</span><span class="sxs-lookup"><span data-stu-id="f1d83-129">**AD DS servers**.</span></span> <span data-ttu-id="f1d83-130">作为 VM 在 Azure 中运行的域控制器。</span><span class="sxs-lookup"><span data-stu-id="f1d83-130">Domain controllers running as VMs in Azure.</span></span> <span data-ttu-id="f1d83-131">这些服务器提供域中的本地标识的身份验证。</span><span class="sxs-lookup"><span data-stu-id="f1d83-131">These servers provide authentication of local identities within the domain.</span></span>

* <span data-ttu-id="f1d83-132">**AD FS 子网**。</span><span class="sxs-lookup"><span data-stu-id="f1d83-132">**AD FS subnet**.</span></span> <span data-ttu-id="f1d83-133">AD FS 服务器位于其自己的子网中，具有充当防火墙的 NSG 规则。</span><span class="sxs-lookup"><span data-stu-id="f1d83-133">The AD FS servers are located within their own subnet with NSG rules acting as a firewall.</span></span>

* <span data-ttu-id="f1d83-134">**AD FS 服务器**。</span><span class="sxs-lookup"><span data-stu-id="f1d83-134">**AD FS servers**.</span></span> <span data-ttu-id="f1d83-135">AD FS 服务器提供联合授权和身份验证。</span><span class="sxs-lookup"><span data-stu-id="f1d83-135">The AD FS servers provide federated authorization and authentication.</span></span> <span data-ttu-id="f1d83-136">在此体系结构中，它们执行以下任务：</span><span class="sxs-lookup"><span data-stu-id="f1d83-136">In this architecture, they perform the following tasks:</span></span>
  
  * <span data-ttu-id="f1d83-137">代表合作伙伴用户接收包含由合作伙伴联合服务器发出的声明的安全令牌。</span><span class="sxs-lookup"><span data-stu-id="f1d83-137">Receiving security tokens containing claims made by a partner federation server on behalf of a partner user.</span></span> <span data-ttu-id="f1d83-138">AD FS 先验证令牌是否有效，然后将声明传递给在 Azure 中运行的 Web 应用程序以对请求进行授权。</span><span class="sxs-lookup"><span data-stu-id="f1d83-138">AD FS verifies that the tokens are valid before passing the claims to the web application running in Azure to authorize requests.</span></span> 
  
    <span data-ttu-id="f1d83-139">在 Azure 中运行的 Web 应用程序是信赖方。</span><span class="sxs-lookup"><span data-stu-id="f1d83-139">The web application running in Azure is the *relying party*.</span></span> <span data-ttu-id="f1d83-140">合作伙伴联合服务器必须发出由 Web 应用程序理解的声明。</span><span class="sxs-lookup"><span data-stu-id="f1d83-140">The partner federation server must issue claims that are understood by the web application.</span></span> <span data-ttu-id="f1d83-141">合作伙伴联合服务器称为帐户伙伴，因为它们代表合作伙伴组织中经过身份验证的帐户提交访问请求。</span><span class="sxs-lookup"><span data-stu-id="f1d83-141">The partner federation servers are referred to as *account partners*, because they submit access requests on behalf of authenticated accounts in the partner organization.</span></span> <span data-ttu-id="f1d83-142">AD FS 服务器称为资源伙伴，因为它们提供对资源（Web 应用程序）的访问。</span><span class="sxs-lookup"><span data-stu-id="f1d83-142">The AD FS servers are called *resource partners* because they provide access to resources (the web application).</span></span>

  * <span data-ttu-id="f1d83-143">使用 AD DS 和 [Active Directory 设备注册服务][ADDRS]，对运行需要访问 Web 应用程序的 Web 浏览器或设备的外部用户发出的传入请求进行身份验证和授权。</span><span class="sxs-lookup"><span data-stu-id="f1d83-143">Authenticating and authorizing incoming requests from external users running a web browser or device that needs access to web applications, by using AD DS and the [Active Directory Device Registration Service][ADDRS].</span></span>
    
  <span data-ttu-id="f1d83-144">AD FS 服务器配置为通过 Azure 负载均衡器访问的场。</span><span class="sxs-lookup"><span data-stu-id="f1d83-144">The AD FS servers are configured as a farm accessed through an Azure load balancer.</span></span> <span data-ttu-id="f1d83-145">此实现可提高可用性和可伸缩性。</span><span class="sxs-lookup"><span data-stu-id="f1d83-145">This implementation improves availability and scalability.</span></span> <span data-ttu-id="f1d83-146">AD FS 服务器不直接向 Internet 公开。</span><span class="sxs-lookup"><span data-stu-id="f1d83-146">The AD FS servers are not exposed directly to the Internet.</span></span> <span data-ttu-id="f1d83-147">所有 Internet 流量都通过 AD FS Web 应用程序代理服务器和 DMZ（也称为外围网络）进行筛选。</span><span class="sxs-lookup"><span data-stu-id="f1d83-147">All Internet traffic is filtered through AD FS web application proxy servers and a DMZ (also referred to as a perimeter network).</span></span>

  <span data-ttu-id="f1d83-148">有关 AD FS 工作原理的详细信息，请参阅 [Active Directory 联合身份验证服务概述][active-directory-federation-services-overview]。</span><span class="sxs-lookup"><span data-stu-id="f1d83-148">For more information about how AD FS works, see [Active Directory Federation Services Overview][active-directory-federation-services-overview].</span></span> <span data-ttu-id="f1d83-149">此外，[Azure 中的 AD FS 部署][adfs-intro]一文包含实现的详细分步介绍。</span><span class="sxs-lookup"><span data-stu-id="f1d83-149">Also, the article [AD FS deployment in Azure][adfs-intro] contains a detailed step-by-step introduction to implementation.</span></span>

* <span data-ttu-id="f1d83-150">**AD FS 代理子网**。</span><span class="sxs-lookup"><span data-stu-id="f1d83-150">**AD FS proxy subnet**.</span></span> <span data-ttu-id="f1d83-151">AD FS 代理服务器可以包含在其自己的子网中，并具有提供保护的 NSG 规则。</span><span class="sxs-lookup"><span data-stu-id="f1d83-151">The AD FS proxy servers can be contained within their own subnet, with NSG rules providing protection.</span></span> <span data-ttu-id="f1d83-152">此子网中的服务器通过在 Azure 虚拟网络与 Internet 之间提供防火墙的一组网络虚拟设备向 Internet 公开。</span><span class="sxs-lookup"><span data-stu-id="f1d83-152">The servers in this subnet are exposed to the Internet through a set of network virtual appliances that provide a firewall between your Azure virtual network and the Internet.</span></span>

* <span data-ttu-id="f1d83-153">**AD FS Web 应用程序代理 (WAP) 服务器**。</span><span class="sxs-lookup"><span data-stu-id="f1d83-153">**AD FS web application proxy (WAP) servers**.</span></span> <span data-ttu-id="f1d83-154">这些 VM 充当用于来自合作伙伴组织和外部设备的传入请求的 AD FS 服务器。</span><span class="sxs-lookup"><span data-stu-id="f1d83-154">These VMs act as AD FS servers for incoming requests from partner organizations and external devices.</span></span> <span data-ttu-id="f1d83-155">WAP 服务器充当筛选器，使得无法从 Internet 直接访问 AD FS 服务器。</span><span class="sxs-lookup"><span data-stu-id="f1d83-155">The WAP servers act as a filter, shielding the AD FS servers from direct access from the Internet.</span></span> <span data-ttu-id="f1d83-156">与 AD FS 服务器一样，在具有负载均衡的场中部署 WAP 服务器可提供比部署独立服务器集合更高的可用性和可伸缩性。</span><span class="sxs-lookup"><span data-stu-id="f1d83-156">As with the AD FS servers, deploying the WAP servers in a farm with load balancing gives you greater availability and scalability than deploying a collection of stand-alone servers.</span></span>
  
  > [!NOTE]
  > <span data-ttu-id="f1d83-157">有关安装 WAP 服务器的详细信息，请参阅[安装和配置 Web 应用程序代理服务器][install_and_configure_the_web_application_proxy_server]</span><span class="sxs-lookup"><span data-stu-id="f1d83-157">For detailed information about installing WAP servers, see [Install and Configure the Web Application Proxy Server][install_and_configure_the_web_application_proxy_server]</span></span>
  > 
  > 

* <span data-ttu-id="f1d83-158">**合作伙伴组织**。</span><span class="sxs-lookup"><span data-stu-id="f1d83-158">**Partner organization**.</span></span> <span data-ttu-id="f1d83-159">运行的 Web 应用程序请求访问在 Azure 中运行的 Web 应用程序的合作伙伴组织。</span><span class="sxs-lookup"><span data-stu-id="f1d83-159">A partner organization running a web application that requests access to a web application running in Azure.</span></span> <span data-ttu-id="f1d83-160">合作伙伴组织中的联合服务器在本地对请求进行身份验证，并将包含声明的安全令牌提交到在 Azure 中运行的 AD FS。</span><span class="sxs-lookup"><span data-stu-id="f1d83-160">The federation server at the partner organization authenticates requests locally, and submits security tokens containing claims to AD FS running in Azure.</span></span> <span data-ttu-id="f1d83-161">Azure 中的 AD FS 会验证安全令牌，如果有效，则可以将声明传递给在 Azure 中运行的 Web 应用程序以对它们进行授权。</span><span class="sxs-lookup"><span data-stu-id="f1d83-161">AD FS in Azure validates the security tokens, and if valid can pass the claims to the web application running in Azure to authorize them.</span></span>
  
  > [!NOTE]
  > <span data-ttu-id="f1d83-162">还可以使用 Azure 网关配置 VPN 隧道，以便为受信任合作伙伴提供对 AD FS 的直接访问。</span><span class="sxs-lookup"><span data-stu-id="f1d83-162">You can also configure a VPN tunnel using Azure gateway to provide direct access to AD FS for trusted partners.</span></span> <span data-ttu-id="f1d83-163">从这些合作伙伴接收的请求不会经过 WAP 服务器。</span><span class="sxs-lookup"><span data-stu-id="f1d83-163">Requests received from these partners do not pass through the WAP servers.</span></span>
  > 
  > 

<span data-ttu-id="f1d83-164">有关不与 AD FS 相关的体系结构部分的详细信息，请参阅以下内容：</span><span class="sxs-lookup"><span data-stu-id="f1d83-164">For more information about the parts of the architecture that are not related to AD FS, see the following:</span></span>
- <span data-ttu-id="f1d83-165">[在 Azure 中实现安全混合网络体系结构][implementing-a-secure-hybrid-network-architecture]</span><span class="sxs-lookup"><span data-stu-id="f1d83-165">[Implementing a secure hybrid network architecture in Azure][implementing-a-secure-hybrid-network-architecture]</span></span>
- <span data-ttu-id="f1d83-166">[在 Azure 中实现提供 Internet 访问方式的安全混合网络体系结构][implementing-a-secure-hybrid-network-architecture-with-internet-access]</span><span class="sxs-lookup"><span data-stu-id="f1d83-166">[Implementing a secure hybrid network architecture with Internet access in Azure][implementing-a-secure-hybrid-network-architecture-with-internet-access]</span></span>
- <span data-ttu-id="f1d83-167">[在 Azure 中实现采用 Active Directory 标识的安全混合网络体系结构][extending-ad-to-azure]。</span><span class="sxs-lookup"><span data-stu-id="f1d83-167">[Implementing a secure hybrid network architecture with Active Directory identities in Azure][extending-ad-to-azure].</span></span>


## <a name="recommendations"></a><span data-ttu-id="f1d83-168">建议</span><span class="sxs-lookup"><span data-stu-id="f1d83-168">Recommendations</span></span>

<span data-ttu-id="f1d83-169">以下建议适用于大多数方案。</span><span class="sxs-lookup"><span data-stu-id="f1d83-169">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="f1d83-170">除非有优先于这些建议的特定要求，否则请遵循这些建议。</span><span class="sxs-lookup"><span data-stu-id="f1d83-170">Follow these recommendations unless you have a specific requirement that overrides them.</span></span> 

### <a name="vm-recommendations"></a><span data-ttu-id="f1d83-171">VM 建议</span><span class="sxs-lookup"><span data-stu-id="f1d83-171">VM recommendations</span></span>

<span data-ttu-id="f1d83-172">创建具有足够资源来处理预期流量的 VM。</span><span class="sxs-lookup"><span data-stu-id="f1d83-172">Create VMs with sufficient resources to handle the expected volume of traffic.</span></span> <span data-ttu-id="f1d83-173">使用在本地承载 AD FS 的现有计算机的大小作为起点。</span><span class="sxs-lookup"><span data-stu-id="f1d83-173">Use the size of the existing machines hosting AD FS on premises as a starting point.</span></span> <span data-ttu-id="f1d83-174">监视资源利用率。</span><span class="sxs-lookup"><span data-stu-id="f1d83-174">Monitor the resource utilization.</span></span> <span data-ttu-id="f1d83-175">可以调整 VM 的大小，在它们太大时进行缩减。</span><span class="sxs-lookup"><span data-stu-id="f1d83-175">You can resize the VMs and scale down if they are too large.</span></span>

<span data-ttu-id="f1d83-176">遵循[在 Azure 上运行 Windows VM][vm-recommendations] 中列出的建议。</span><span class="sxs-lookup"><span data-stu-id="f1d83-176">Follow the recommendations listed in [Running a Windows VM on Azure][vm-recommendations].</span></span>

### <a name="networking-recommendations"></a><span data-ttu-id="f1d83-177">网络建议</span><span class="sxs-lookup"><span data-stu-id="f1d83-177">Networking recommendations</span></span>

<span data-ttu-id="f1d83-178">使用静态专用 IP 地址为承载 AD FS 和 WAP 服务器的每个 VM 配置网络接口。</span><span class="sxs-lookup"><span data-stu-id="f1d83-178">Configure the network interface for each of the VMs hosting AD FS and WAP servers with static private IP addresses.</span></span>

<span data-ttu-id="f1d83-179">请勿向 AD FS VM 提供公共 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="f1d83-179">Do not give the AD FS VMs public IP addresses.</span></span> <span data-ttu-id="f1d83-180">有关详细信息，请参阅“安全注意事项”部分。</span><span class="sxs-lookup"><span data-stu-id="f1d83-180">For more information, see the Security considerations section.</span></span>

<span data-ttu-id="f1d83-181">为每个 AD FS 和 WAP VM 的网络接口设置首选和辅助域名服务 (DNS) 服务器的 IP 地址，以引用 Active Directory DS VM。</span><span class="sxs-lookup"><span data-stu-id="f1d83-181">Set the IP address of the preferred and secondary domain name service (DNS) servers for the network interfaces for each AD FS and WAP VM to reference the Active Directory DS VMs.</span></span> <span data-ttu-id="f1d83-182">Active Directory DS VM 应运行 DNS。</span><span class="sxs-lookup"><span data-stu-id="f1d83-182">The Active Directory DS VMS should be running DNS.</span></span> <span data-ttu-id="f1d83-183">使每个 VM 可以加入域需要此步骤。</span><span class="sxs-lookup"><span data-stu-id="f1d83-183">This step is necessary to enable each VM to join the domain.</span></span>

### <a name="ad-fs-availability"></a><span data-ttu-id="f1d83-184">AD FS 可用性</span><span class="sxs-lookup"><span data-stu-id="f1d83-184">AD FS availability</span></span> 

<span data-ttu-id="f1d83-185">创建具有至少两台服务器的 AD FS 场以提高服务的可用性。</span><span class="sxs-lookup"><span data-stu-id="f1d83-185">Create an AD FS farm with at least two servers to increase availability of the service.</span></span> <span data-ttu-id="f1d83-186">为场中的每个 AD FS VM 使用不同的存储帐户。</span><span class="sxs-lookup"><span data-stu-id="f1d83-186">Use different storage accounts for each AD FS VM in the farm.</span></span> <span data-ttu-id="f1d83-187">此方法可帮助确保单个存储帐户中的故障不会使整个场不可访问。</span><span class="sxs-lookup"><span data-stu-id="f1d83-187">This approach helps to ensure that a failure in a single storage account does not make the entire farm inaccessible.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="f1d83-188">建议使用[托管磁盘](/azure/storage/storage-managed-disks-overview)。</span><span class="sxs-lookup"><span data-stu-id="f1d83-188">We recommend the use of [managed disks](/azure/storage/storage-managed-disks-overview).</span></span> <span data-ttu-id="f1d83-189">托管磁盘不需要存储帐户。</span><span class="sxs-lookup"><span data-stu-id="f1d83-189">Managed disks do not require a storage account.</span></span> <span data-ttu-id="f1d83-190">只需指定磁盘的大小和类型，它就会以高度可用的方式部署。</span><span class="sxs-lookup"><span data-stu-id="f1d83-190">You simply specify the size and type of disk and it is deployed in a highly available way.</span></span> <span data-ttu-id="f1d83-191">我们的[参考体系结构](/azure/architecture/reference-architectures/)当前未部署托管磁盘，但是[模板构建基块](https://github.com/mspnp/template-building-blocks/wiki)会在版本 2 中更新为部署托管磁盘。</span><span class="sxs-lookup"><span data-stu-id="f1d83-191">Our [reference architectures](/azure/architecture/reference-architectures/) do not currently deploy managed disks but the [template building blocks](https://github.com/mspnp/template-building-blocks/wiki) will be updated to deploy managed disks in version 2.</span></span>

<span data-ttu-id="f1d83-192">为 AD FS 和 WAP VM 创建单独的 Azure 可用性集。</span><span class="sxs-lookup"><span data-stu-id="f1d83-192">Create separate Azure availability sets for the AD FS and WAP VMs.</span></span> <span data-ttu-id="f1d83-193">确保每个集中至少有两个 VM。</span><span class="sxs-lookup"><span data-stu-id="f1d83-193">Ensure that there are at least two VMs in each set.</span></span> <span data-ttu-id="f1d83-194">每个可用性集必须具有至少两个更新域和两个容错域。</span><span class="sxs-lookup"><span data-stu-id="f1d83-194">Each availability set must have at least two update domains and two fault domains.</span></span>

<span data-ttu-id="f1d83-195">按如下所示为 AD FS VM 和 WAP VM 配置负载均衡器：</span><span class="sxs-lookup"><span data-stu-id="f1d83-195">Configure the load balancers for the AD FS VMs and WAP VMs as follows:</span></span>

* <span data-ttu-id="f1d83-196">使用 Azure 负载均衡器提供对 WAP VM 的外部访问，并使用内部负载均衡器在场中的 AD FS 服务器之间分布负载。</span><span class="sxs-lookup"><span data-stu-id="f1d83-196">Use an Azure load balancer to provide external access to the WAP VMs, and an internal load balancer to distribute the load across the AD FS servers in the farm.</span></span>
* <span data-ttu-id="f1d83-197">仅将在端口 443 (HTTPS) 上出现的流量传递给 AD FS/WAP 服务器。</span><span class="sxs-lookup"><span data-stu-id="f1d83-197">Only pass traffic appearing on port 443 (HTTPS) to the AD FS/WAP servers.</span></span>
* <span data-ttu-id="f1d83-198">为负载均衡器提供静态 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="f1d83-198">Give the load balancer a static IP address.</span></span>
* <span data-ttu-id="f1d83-199">使用 TCP 协议而不是 HTTPS 创建运行状况探测。</span><span class="sxs-lookup"><span data-stu-id="f1d83-199">Create a health probe using the TCP protocol rather than HTTPS.</span></span> <span data-ttu-id="f1d83-200">可以对端口 443 执行 ping 操作，以验证 AD FS 服务器是否在正常运行。</span><span class="sxs-lookup"><span data-stu-id="f1d83-200">You can ping port 443 to verify that an AD FS server is functioning.</span></span>
  
  > [!NOTE]
  > <span data-ttu-id="f1d83-201">AD FS 服务器使用服务器名称指示 (SNI) 协议，因此尝试从负载均衡器使用 HTTPS 终结点进行探测会失败。</span><span class="sxs-lookup"><span data-stu-id="f1d83-201">AD FS servers use the Server Name Indication (SNI) protocol, so attempting to probe using an HTTPS endpoint from the load balancer fails.</span></span>
  > 
  > 
* <span data-ttu-id="f1d83-202">将 DNS A 记录添加到 AD FS 负载均衡器的域。</span><span class="sxs-lookup"><span data-stu-id="f1d83-202">Add a DNS *A* record to the domain for the AD FS load balancer.</span></span> <span data-ttu-id="f1d83-203">指定负载均衡器的 IP 地址，并为它提供域中的名称（例如 adfs.contoso.com）。</span><span class="sxs-lookup"><span data-stu-id="f1d83-203">Specify the IP address of the load balancer, and give it a name in the domain (such as adfs.contoso.com).</span></span> <span data-ttu-id="f1d83-204">这是客户端和 WAP 服务器用于访问 AD FS 服务器场的名称。</span><span class="sxs-lookup"><span data-stu-id="f1d83-204">This is the name clients and the WAP servers use to access the AD FS server farm.</span></span>

### <a name="ad-fs-security"></a><span data-ttu-id="f1d83-205">AD FS 安全性</span><span class="sxs-lookup"><span data-stu-id="f1d83-205">AD FS security</span></span> 

<span data-ttu-id="f1d83-206">阻止向 Internet 直接公开 AD FS 服务器。</span><span class="sxs-lookup"><span data-stu-id="f1d83-206">Prevent direct exposure of the AD FS servers to the Internet.</span></span> <span data-ttu-id="f1d83-207">AD FS 服务器是具有完整授权，可授予安全令牌的已加入域的计算机。</span><span class="sxs-lookup"><span data-stu-id="f1d83-207">AD FS servers are domain-joined computers that have full authorization to grant security tokens.</span></span> <span data-ttu-id="f1d83-208">如果服务器受到损害，则恶意用户可以向所有 Web 应用程序并向 AD FS 保护的所有联合服务器颁发完全访问令牌。</span><span class="sxs-lookup"><span data-stu-id="f1d83-208">If a server is compromised, a malicious user can issue full access tokens to all web applications and to all federation servers that are protected by AD FS.</span></span> <span data-ttu-id="f1d83-209">如果系统必须处理不是从受信任合作伙伴站点连接的外部用户发出的请求，请使用 WAP 服务器处理这些请求。</span><span class="sxs-lookup"><span data-stu-id="f1d83-209">If your system must handle requests from external users not connecting from trusted partner sites, use WAP servers to handle these requests.</span></span> <span data-ttu-id="f1d83-210">有关详细信息，请参阅[放置联合服务器代理的位置][where-to-place-an-fs-proxy]。</span><span class="sxs-lookup"><span data-stu-id="f1d83-210">For more information, see [Where to Place a Federation Server Proxy][where-to-place-an-fs-proxy].</span></span>

<span data-ttu-id="f1d83-211">将 AD FS 服务器和 WAP 服务器放置在具有自己的防火墙的单独子网中。</span><span class="sxs-lookup"><span data-stu-id="f1d83-211">Place AD FS servers and WAP servers in separate subnets with their own firewalls.</span></span> <span data-ttu-id="f1d83-212">可以使用 NSG 规则定义防火墙规则。</span><span class="sxs-lookup"><span data-stu-id="f1d83-212">You can use NSG rules to define firewall rules.</span></span> <span data-ttu-id="f1d83-213">如果需要更全面的保护，则可以使用一对子网和网络虚拟设备 (NVA) 在服务器周围实现附加安全外围，如文档[在 Azure 中实现提供 Internet 访问方式的安全混合网络体系结构][implementing-a-secure-hybrid-network-architecture-with-internet-access]中所述。</span><span class="sxs-lookup"><span data-stu-id="f1d83-213">If you require more comprehensive protection you can implement an additional security perimeter around servers by using a pair of subnets and network virtual appliances (NVAs), as described in the document [Implementing a secure hybrid network architecture with Internet access in Azure][implementing-a-secure-hybrid-network-architecture-with-internet-access].</span></span> <span data-ttu-id="f1d83-214">所有防火墙都应允许端口 443 (HTTPS) 上的流量。</span><span class="sxs-lookup"><span data-stu-id="f1d83-214">All firewalls should allow traffic on port 443 (HTTPS).</span></span>

<span data-ttu-id="f1d83-215">限制对 AD FS 和 WAP 服务器的直接登录访问。</span><span class="sxs-lookup"><span data-stu-id="f1d83-215">Restrict direct sign in access to the AD FS and WAP servers.</span></span> <span data-ttu-id="f1d83-216">只有 DevOps 员工才应能够连接。</span><span class="sxs-lookup"><span data-stu-id="f1d83-216">Only DevOps staff should be able to connect.</span></span>

<span data-ttu-id="f1d83-217">请勿将 WAP 服务器加入域。</span><span class="sxs-lookup"><span data-stu-id="f1d83-217">Do not join the WAP servers to the domain.</span></span>

### <a name="ad-fs-installation"></a><span data-ttu-id="f1d83-218">AD FS 安装</span><span class="sxs-lookup"><span data-stu-id="f1d83-218">AD FS installation</span></span> 

<span data-ttu-id="f1d83-219">文章[部署联合服务器场][Deploying_a_federation_server_farm]提供了有关安装和配置 AD FS 的详细说明。</span><span class="sxs-lookup"><span data-stu-id="f1d83-219">The article [Deploying a Federation Server Farm][Deploying_a_federation_server_farm] provides detailed instructions for installing and configuring AD FS.</span></span> <span data-ttu-id="f1d83-220">在场中配置第一台 AD FS 服务器之前执行以下任务：</span><span class="sxs-lookup"><span data-stu-id="f1d83-220">Perform the following tasks before configuring the first AD FS server in the farm:</span></span>

1. <span data-ttu-id="f1d83-221">获取用于执行服务器身份验证的公开受信任证书。</span><span class="sxs-lookup"><span data-stu-id="f1d83-221">Obtain a publicly trusted certificate for performing server authentication.</span></span> <span data-ttu-id="f1d83-222">使用者名称必须包含客户端用于访问联合身份验证服务的名称。</span><span class="sxs-lookup"><span data-stu-id="f1d83-222">The *subject name* must contain the name clients use to access the federation service.</span></span> <span data-ttu-id="f1d83-223">这可以是为负载均衡器注册的 DNS 名称，例如 adfs.contoso.com（出于安全原因，请避免使用通配符名称，如 *.contoso.com）。</span><span class="sxs-lookup"><span data-stu-id="f1d83-223">This can be the DNS name registered for the load balancer, for example, *adfs.contoso.com* (avoid using wildcard names such as **.contoso.com*, for security reasons).</span></span> <span data-ttu-id="f1d83-224">在所有 AD FS 服务器 VM 上使用相同证书。</span><span class="sxs-lookup"><span data-stu-id="f1d83-224">Use the same certificate on all AD FS server VMs.</span></span> <span data-ttu-id="f1d83-225">可以从受信任证书颁发机构购买证书，但如果组织使用 Active Directory 证书服务，则可以创建自己的证书。</span><span class="sxs-lookup"><span data-stu-id="f1d83-225">You can purchase a certificate from a trusted certification authority, but if your organization uses Active Directory Certificate Services you can create your own.</span></span> 
   
    <span data-ttu-id="f1d83-226">使用者可选名称由设备注册服务 (DRS) 用于启用从外部设备进行的访问。</span><span class="sxs-lookup"><span data-stu-id="f1d83-226">The *subject alternative name* is used by the device registration service (DRS) to enable access from external devices.</span></span> <span data-ttu-id="f1d83-227">这应采用 enterpriseregistration.contoso.com 的形式。</span><span class="sxs-lookup"><span data-stu-id="f1d83-227">This should be of the form *enterpriseregistration.contoso.com*.</span></span>
   
    <span data-ttu-id="f1d83-228">有关详细信息，请参阅[为 AD FS 获取并配置安全套接字层 (SSL) 证书][adfs_certificates]。</span><span class="sxs-lookup"><span data-stu-id="f1d83-228">For more information, see [Obtain and Configure a Secure Sockets Layer (SSL) Certificate for AD FS][adfs_certificates].</span></span>

2. <span data-ttu-id="f1d83-229">在域控制器上，为密钥分发服务生成新的根密钥。</span><span class="sxs-lookup"><span data-stu-id="f1d83-229">On the domain controller, generate a new root key for the Key Distribution Service.</span></span> <span data-ttu-id="f1d83-230">将有效时间设置为当前时间减 10 小时（此配置会减少在域中分发和同步密钥时可能发生的延迟）。</span><span class="sxs-lookup"><span data-stu-id="f1d83-230">Set the effective time to the current time minus 10 hours (this configuration reduces the delay that can occur in distributing and synchronizing keys across the domain).</span></span> <span data-ttu-id="f1d83-231">支持创建用于运行 AD FS 服务的组服务帐户需要此步骤。</span><span class="sxs-lookup"><span data-stu-id="f1d83-231">This step is necessary to support creating the group service account that is used to run the AD FS service.</span></span> <span data-ttu-id="f1d83-232">以下 PowerShell 命令演示有关如何执行此操作的示例：</span><span class="sxs-lookup"><span data-stu-id="f1d83-232">The following PowerShell command shows an example of how to do this:</span></span>
   
    ```powershell
    Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
    ```

3. <span data-ttu-id="f1d83-233">将每个 AD FS 服务器 VM 添加到域。</span><span class="sxs-lookup"><span data-stu-id="f1d83-233">Add each AD FS server VM to the domain.</span></span>

> [!NOTE]
> <span data-ttu-id="f1d83-234">若要安装 AD FS，为域运行主域控制器 (PDC) 仿真器灵活单主机操作 (FSMO) 角色的域控制器必须正在运行并且可从 AD FS VM 进行访问。</span><span class="sxs-lookup"><span data-stu-id="f1d83-234">To install AD FS, the domain controller running the primary domain controller (PDC) emulator flexible single master operation (FSMO) role for the domain must be running and accessible from the AD FS VMs.</span></span> <span data-ttu-id="f1d83-235"><<RBC：是否可通过某种方法减少此内容的重复性？>></span><span class="sxs-lookup"><span data-stu-id="f1d83-235"><<RBC: Is there a way to make this less repetitive?>></span></span>
> 
> 

### <a name="ad-fs-trust"></a><span data-ttu-id="f1d83-236">AD FS 信任</span><span class="sxs-lookup"><span data-stu-id="f1d83-236">AD FS trust</span></span> 

<span data-ttu-id="f1d83-237">在 AD FS 安装与任何合作伙伴组织的联合服务器之间建立联合身份验证信任。</span><span class="sxs-lookup"><span data-stu-id="f1d83-237">Establish federation trust between your AD FS installation, and the federation servers of any partner organizations.</span></span> <span data-ttu-id="f1d83-238">配置所需的任何声明筛选和映射。</span><span class="sxs-lookup"><span data-stu-id="f1d83-238">Configure any claims filtering and mapping required.</span></span> 

* <span data-ttu-id="f1d83-239">每个合作伙伴组织的 DevOps 员工必须添加信赖方信任，以便可通过 AD FS 服务器访问 Web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="f1d83-239">DevOps staff at each partner organization must add a relying party trust for the web applications accessible through your AD FS servers.</span></span>
* <span data-ttu-id="f1d83-240">组织中的 DevOps 员工必须配置声明提供方信任，以便使 AD FS 服务器可以信任合作伙伴组织提供的声明。</span><span class="sxs-lookup"><span data-stu-id="f1d83-240">DevOps staff in your organization must configure claims-provider trust to enable your AD FS servers to trust the claims that partner organizations provide.</span></span>
* <span data-ttu-id="f1d83-241">组织中的 DevOps 员工还必须配置 AD FS 以将声明传递给组织的 Web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="f1d83-241">DevOps staff in your organization must also configure AD FS to pass claims on to your organization's web applications.</span></span>
  
<span data-ttu-id="f1d83-242">有关详细信息，请参阅[建立联合身份验证信任][establishing-federation-trust]。</span><span class="sxs-lookup"><span data-stu-id="f1d83-242">For more information, see [Establishing Federation Trust][establishing-federation-trust].</span></span>

<span data-ttu-id="f1d83-243">通过 WAP 服务器使用预身份验证发布组织的 Web 应用程序并将它们提供给外部合作伙伴。</span><span class="sxs-lookup"><span data-stu-id="f1d83-243">Publish your organization's web applications and make them available to external partners by using preauthentication through the WAP servers.</span></span> <span data-ttu-id="f1d83-244">有关详细信息，请参阅[使用 AD FS 预身份验证发布应用程序][publish_applications_using_AD_FS_preauthentication]</span><span class="sxs-lookup"><span data-stu-id="f1d83-244">For more information, see [Publish Applications using AD FS Preauthentication][publish_applications_using_AD_FS_preauthentication]</span></span>

<span data-ttu-id="f1d83-245">AD FS 支持令牌转换和扩大。</span><span class="sxs-lookup"><span data-stu-id="f1d83-245">AD FS supports token transformation and augmentation.</span></span> <span data-ttu-id="f1d83-246">Azure Active Directory 不提供此功能。</span><span class="sxs-lookup"><span data-stu-id="f1d83-246">Azure Active Directory does not provide this feature.</span></span> <span data-ttu-id="f1d83-247">借助 AD FS，在设置信任关系时可以：</span><span class="sxs-lookup"><span data-stu-id="f1d83-247">With AD FS, when you set up the trust relationships, you can:</span></span>

* <span data-ttu-id="f1d83-248">为授权规则配置声明转换。</span><span class="sxs-lookup"><span data-stu-id="f1d83-248">Configure claim transformations for authorization rules.</span></span> <span data-ttu-id="f1d83-249">例如，可以将组安全性从非 Microsoft 合作伙伴组织使用的表示形式映射到 Active Directory DS 可以在组织中授权的某种内容。</span><span class="sxs-lookup"><span data-stu-id="f1d83-249">For example, you can map group security from a representation used by a non-Microsoft partner organization to something that that Active Directory DS can authorize in your organization.</span></span>
* <span data-ttu-id="f1d83-250">将声明从一种格式转换为另一种格式。</span><span class="sxs-lookup"><span data-stu-id="f1d83-250">Transform claims from one format to another.</span></span> <span data-ttu-id="f1d83-251">例如，如果应用程序仅支持 SAML 1.1 声明，则可以从 SAML 2.0 映射到 SAML 1.1。</span><span class="sxs-lookup"><span data-stu-id="f1d83-251">For example, you can map from SAML 2.0 to SAML 1.1 if your application only supports SAML 1.1 claims.</span></span>

### <a name="ad-fs-monitoring"></a><span data-ttu-id="f1d83-252">AD FS 监视</span><span class="sxs-lookup"><span data-stu-id="f1d83-252">AD FS monitoring</span></span> 

<span data-ttu-id="f1d83-253">[适用于 Active Directory 联合身份验证服务 2012 R2 的 Microsoft System Center 管理包][oms-adfs-pack]为联合服务器提供 AD FS 部署的主动和被动监视。</span><span class="sxs-lookup"><span data-stu-id="f1d83-253">The [Microsoft System Center Management Pack for Active Directory Federation Services 2012 R2][oms-adfs-pack] provides both proactive and reactive monitoring of your AD FS deployment for the federation server.</span></span> <span data-ttu-id="f1d83-254">此管理包监视：</span><span class="sxs-lookup"><span data-stu-id="f1d83-254">This management pack monitors:</span></span>

* <span data-ttu-id="f1d83-255">AD FS 服务在其事件日志中记录的事件。</span><span class="sxs-lookup"><span data-stu-id="f1d83-255">Events that the AD FS service records in its event logs.</span></span>
* <span data-ttu-id="f1d83-256">AD FS 性能计数器收集的性能数据。</span><span class="sxs-lookup"><span data-stu-id="f1d83-256">The performance data that the AD FS performance counters collect.</span></span> 
* <span data-ttu-id="f1d83-257">AD FS 系统和 Web 应用程序（信赖方）的总体运行状况，提供针对关键问题和警告的警报。</span><span class="sxs-lookup"><span data-stu-id="f1d83-257">The overall health of the AD FS system and web applications (relying parties), and provides alerts for critical issues and warnings.</span></span> 

## <a name="scalability-considerations"></a><span data-ttu-id="f1d83-258">可伸缩性注意事项</span><span class="sxs-lookup"><span data-stu-id="f1d83-258">Scalability considerations</span></span>

<span data-ttu-id="f1d83-259">从文章[规划 AD FS 部署][plan-your-adfs-deployment]中汇总的以下注意事项为调整 AD FS 场规模提供了起点：</span><span class="sxs-lookup"><span data-stu-id="f1d83-259">The following considerations, summarized from the article [Plan your AD FS deployment][plan-your-adfs-deployment], give a starting point for sizing AD FS farms:</span></span>

* <span data-ttu-id="f1d83-260">如果用户数少于 1000，请勿创建专用服务器，而是改为在云中的每台 Active Directory DS 服务器上安装 AD FS。</span><span class="sxs-lookup"><span data-stu-id="f1d83-260">If you have fewer than 1000 users, do not create dedicated servers, but instead install AD FS on each of the Active Directory DS servers in the cloud.</span></span> <span data-ttu-id="f1d83-261">确保具有至少两台 Active Directory DS 服务器以保持可用性。</span><span class="sxs-lookup"><span data-stu-id="f1d83-261">Make sure that you have at least two Active Directory DS servers to maintain availability.</span></span> <span data-ttu-id="f1d83-262">创建单台 WAP 服务器。</span><span class="sxs-lookup"><span data-stu-id="f1d83-262">Create a single WAP server.</span></span>
* <span data-ttu-id="f1d83-263">如果用户数介于 1000 与 15000 之间，请创建两台专用 AD FS 服务器和两台专用 WAP 服务器。</span><span class="sxs-lookup"><span data-stu-id="f1d83-263">If you have between 1000 and 15000 users, create two dedicated AD FS servers and two dedicated WAP servers.</span></span>
* <span data-ttu-id="f1d83-264">如果用户数介于 15000 与 60000 之间，请创建三到五台专用 AD FS 服务器和至少两台专用 WAP 服务器。</span><span class="sxs-lookup"><span data-stu-id="f1d83-264">If you have between 15000 and 60000 users, create between three and five dedicated AD FS servers and at least two dedicated WAP servers.</span></span>

<span data-ttu-id="f1d83-265">这些注意事项假设在 Azure 中使用双四核 VM（标准 D4_v2 或更好）大小。</span><span class="sxs-lookup"><span data-stu-id="f1d83-265">These considerations assume that you are using dual quad-core VM (Standard D4_v2, or better) sizes in Azure.</span></span>

<span data-ttu-id="f1d83-266">如果使用 Windows 内部数据库存储 AD FS 配置数据，则场中仅限八台 AD FS 服务器。</span><span class="sxs-lookup"><span data-stu-id="f1d83-266">If you are using the Windows Internal Database to store AD FS configuration data, you are limited to eight AD FS servers in the farm.</span></span> <span data-ttu-id="f1d83-267">如果预计在将来需要更多服务器，请使用 SQL Server。</span><span class="sxs-lookup"><span data-stu-id="f1d83-267">If you anticipate that you will need more in the future, use SQL Server.</span></span> <span data-ttu-id="f1d83-268">有关详细信息，请参阅 [AD FS 配置数据库的角色][adfs-configuration-database]。</span><span class="sxs-lookup"><span data-stu-id="f1d83-268">For more information, see [The Role of the AD FS Configuration Database][adfs-configuration-database].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="f1d83-269">可用性注意事项</span><span class="sxs-lookup"><span data-stu-id="f1d83-269">Availability considerations</span></span>

<span data-ttu-id="f1d83-270">可以使用 SQL Server 或 Windows 内部数据库保存 AD FS 配置信息。</span><span class="sxs-lookup"><span data-stu-id="f1d83-270">You can use either SQL Server or the Windows Internal Database to hold AD FS configuration information.</span></span> <span data-ttu-id="f1d83-271">Windows 内部数据库提供基本冗余。</span><span class="sxs-lookup"><span data-stu-id="f1d83-271">The Windows Internal Database provides basic redundancy.</span></span> <span data-ttu-id="f1d83-272">更改仅仅直接写入 AD FS 群集中的一个 AD FS 数据库，而其他服务器使用请求复制使其数据库保持最新状态。</span><span class="sxs-lookup"><span data-stu-id="f1d83-272">Changes are written directly to only one of the AD FS databases in the AD FS cluster, while the other servers use pull replication to keep their databases up to date.</span></span> <span data-ttu-id="f1d83-273">使用 SQL Server 可以提供完整数据库冗余和高可用性（使用故障转移群集或镜像）。</span><span class="sxs-lookup"><span data-stu-id="f1d83-273">Using SQL Server can provide full database redundancy and high availability using failover clustering or mirroring.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="f1d83-274">可管理性注意事项</span><span class="sxs-lookup"><span data-stu-id="f1d83-274">Manageability considerations</span></span>

<span data-ttu-id="f1d83-275">DevOps 员工应准备好执行以下任务：</span><span class="sxs-lookup"><span data-stu-id="f1d83-275">DevOps staff should be prepared to perform the following tasks:</span></span>

* <span data-ttu-id="f1d83-276">管理联合服务器，包括管理 AD FS 场、管理联合服务器上的信任策略以及管理联合身份验证服务使用的证书。</span><span class="sxs-lookup"><span data-stu-id="f1d83-276">Managing the federation servers, including managing the AD FS farm, managing trust policy on the federation servers, and managing the certificates used by the federation services.</span></span>
* <span data-ttu-id="f1d83-277">管理 WAP 服务器，包括管理 WAP 场和证书。</span><span class="sxs-lookup"><span data-stu-id="f1d83-277">Managing the WAP servers including managing the WAP farm and certificates.</span></span>
* <span data-ttu-id="f1d83-278">管理 Web 应用程序，包括配置信赖方、身份验证方法和声明映射。</span><span class="sxs-lookup"><span data-stu-id="f1d83-278">Managing web applications including configuring relying parties, authentication methods, and claims mappings.</span></span>
* <span data-ttu-id="f1d83-279">备份 AD FS 组件。</span><span class="sxs-lookup"><span data-stu-id="f1d83-279">Backing up AD FS components.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="f1d83-280">安全注意事项</span><span class="sxs-lookup"><span data-stu-id="f1d83-280">Security considerations</span></span>

<span data-ttu-id="f1d83-281">AD FS 使用 HTTPS 协议，因此请确保包含 Web 层 VM 的子网的 NSG 规则允许 HTTPS 请求。</span><span class="sxs-lookup"><span data-stu-id="f1d83-281">AD FS utilizes the HTTPS protocol, so make sure that the NSG rules for the subnet containing the web tier VMs permit HTTPS requests.</span></span> <span data-ttu-id="f1d83-282">这些请求可能源自本地网络、包含 Web 层、业务层、数据层、专用 DMZ、公共 DMZ 的子网以及包含 AD FS 服务器的子网。</span><span class="sxs-lookup"><span data-stu-id="f1d83-282">These requests can originate from the on-premises network, the subnets containing the web tier, business tier, data tier, private DMZ, public DMZ, and the subnet containing the AD FS servers.</span></span>

<span data-ttu-id="f1d83-283">请考虑使用一组网络虚拟设备，它们记录有关遍历虚拟网络边缘的流量的详细信息以用于审核。</span><span class="sxs-lookup"><span data-stu-id="f1d83-283">Consider using a set of network virtual appliances that logs detailed information on traffic traversing the edge of your virtual network for auditing purposes.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="f1d83-284">部署解决方案</span><span class="sxs-lookup"><span data-stu-id="f1d83-284">Deploy the solution</span></span>

<span data-ttu-id="f1d83-285">[GitHub][github] 上提供了一个用于部署此参考体系结构的解决方案。</span><span class="sxs-lookup"><span data-stu-id="f1d83-285">A solution is available on [GitHub][github] to deploy this reference architecture.</span></span> <span data-ttu-id="f1d83-286">若要运行部署此解决方案的 Powershell 脚本，需要具有 [Azure CLI][azure-cli] 的最新版本。</span><span class="sxs-lookup"><span data-stu-id="f1d83-286">You will need the latest version of the [Azure CLI][azure-cli] to run the Powershell script that deploys the solution.</span></span> <span data-ttu-id="f1d83-287">若要部署此参考体系结构，请执行以下步骤：</span><span class="sxs-lookup"><span data-stu-id="f1d83-287">To deploy the reference architecture, follow these steps:</span></span>

1. <span data-ttu-id="f1d83-288">将解决方案文件夹从 [GitHub][github] 克隆到本地计算机。</span><span class="sxs-lookup"><span data-stu-id="f1d83-288">Download or clone the solution folder from [GitHub][github] to your local machine.</span></span>

2. <span data-ttu-id="f1d83-289">打开 Azure CLI 并导航到本地解决方案文件夹。</span><span class="sxs-lookup"><span data-stu-id="f1d83-289">Open the Azure CLI and navigate to the local solution folder.</span></span>

3. <span data-ttu-id="f1d83-290">运行以下命令：</span><span class="sxs-lookup"><span data-stu-id="f1d83-290">Run the following command:</span></span>
   
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```
   
    <span data-ttu-id="f1d83-291">将 `<subscription id>` 替换为你的 Azure 订阅 ID。</span><span class="sxs-lookup"><span data-stu-id="f1d83-291">Replace `<subscription id>` with your Azure subscription ID.</span></span>
   
    <span data-ttu-id="f1d83-292">对于 `<location>`，请指定一个 Azure 区域，例如 `eastus` 或 `westus`。</span><span class="sxs-lookup"><span data-stu-id="f1d83-292">For `<location>`, specify an Azure region, such as `eastus` or `westus`.</span></span>
   
    <span data-ttu-id="f1d83-293">`<mode>` 参数控制部署粒度，可以是下列值之一：</span><span class="sxs-lookup"><span data-stu-id="f1d83-293">The `<mode>` parameter controls the granularity of the deployment, and can be one of the following values:</span></span>
   
   * <span data-ttu-id="f1d83-294">`Onpremise`：部署模拟的本地环境。</span><span class="sxs-lookup"><span data-stu-id="f1d83-294">`Onpremise`: Deploys a simulated on-premises environment.</span></span> <span data-ttu-id="f1d83-295">如果没有现有本地网络，或如果要测试此参考体系结构而不更改现有本地网络的配置，则可以使用此部署进行测试和试验。</span><span class="sxs-lookup"><span data-stu-id="f1d83-295">You can use this deployment to test and experiment if you do not have an existing on-premises network, or if you want to test this reference architecture without changing the configuration of your existing on-premises network.</span></span>
   * <span data-ttu-id="f1d83-296">`Infrastructure`：部署 VNet 基础结构和 jumpbox。</span><span class="sxs-lookup"><span data-stu-id="f1d83-296">`Infrastructure`: deploys the VNet infrastructure and jump box.</span></span>
   * <span data-ttu-id="f1d83-297">`CreateVpn`：部署 Azure 虚拟网络网关并将其连接到模拟的本地网络。</span><span class="sxs-lookup"><span data-stu-id="f1d83-297">`CreateVpn`: deploys an Azure virtual network gateway and connects it to the simulated on-premises network.</span></span>
   * <span data-ttu-id="f1d83-298">`AzureADDS`：部署充当 Active Directory DS 服务器的 VM，将 Active Directory 部署到这些 VM，并在 Azure 中创建域。</span><span class="sxs-lookup"><span data-stu-id="f1d83-298">`AzureADDS`: deploys the VMs acting as Active Directory DS servers, deploys Active Directory to these VMs, and creates the domain in Azure.</span></span>
   * <span data-ttu-id="f1d83-299">`AdfsVm`：在 Azure 中部署 AD FS VM 并将它们加入域。</span><span class="sxs-lookup"><span data-stu-id="f1d83-299">`AdfsVm`: deploys the AD FS VMs and joins them to the domain in Azure.</span></span>
   * <span data-ttu-id="f1d83-300">`PublicDMZ`：在 Azure 中部署公共 DMZ。</span><span class="sxs-lookup"><span data-stu-id="f1d83-300">`PublicDMZ`: deploys the public DMZ in Azure.</span></span>
   * <span data-ttu-id="f1d83-301">`ProxyVm`：在 Azure 中部署 AD FS 代理 VM 并将它们加入域。</span><span class="sxs-lookup"><span data-stu-id="f1d83-301">`ProxyVm`: deploys the AD FS proxy VMs and joins them to the domain in Azure.</span></span>
   * <span data-ttu-id="f1d83-302">`Prepare`：部署上述所有部署。</span><span class="sxs-lookup"><span data-stu-id="f1d83-302">`Prepare`: deploys all of the preceding deployments.</span></span> <span data-ttu-id="f1d83-303">**如果在构建全新部署并且没有现有本地基础结构，则这是推荐选项。**</span><span class="sxs-lookup"><span data-stu-id="f1d83-303">**This is the recommended option if you are building an entirely new deployment and you don't have an existing on-premises infrastructure.**</span></span> 
   * <span data-ttu-id="f1d83-304">`Workload`：（可选）部署 Web、业务和数据层 VM 以及支持网络。</span><span class="sxs-lookup"><span data-stu-id="f1d83-304">`Workload`: optionally deploys web, business, and data tier VMs and supporting network.</span></span> <span data-ttu-id="f1d83-305">不包括在 `Prepare` 部署模式中。</span><span class="sxs-lookup"><span data-stu-id="f1d83-305">Not included in the `Prepare` deployment mode.</span></span>
   * <span data-ttu-id="f1d83-306">`PrivateDMZ`：（可选）在 Azure 中，在上面部署的 `Workload` VM 前部署专用 DMZ。</span><span class="sxs-lookup"><span data-stu-id="f1d83-306">`PrivateDMZ`: optionally deploys the private DMZ in Azure in front of the `Workload` VMs deployed above.</span></span> <span data-ttu-id="f1d83-307">不包括在 `Prepare` 部署模式中。</span><span class="sxs-lookup"><span data-stu-id="f1d83-307">Not included in the `Prepare` deployment mode.</span></span>

4. <span data-ttu-id="f1d83-308">等待部署完成。</span><span class="sxs-lookup"><span data-stu-id="f1d83-308">Wait for the deployment to complete.</span></span> <span data-ttu-id="f1d83-309">如果使用 `Prepare` 选项，则部署需要几个小时才能完成，完成时会显示消息 `Preparation is completed. Please install certificate to all AD FS and proxy VMs.`</span><span class="sxs-lookup"><span data-stu-id="f1d83-309">If you used the `Prepare` option, the deployment takes several hours to complete, and finishes with the message `Preparation is completed. Please install certificate to all AD FS and proxy VMs.`</span></span>

5. <span data-ttu-id="f1d83-310">重新启动 jumpbox（ra-adfs-security-rg 组中的 ra-adfs-mgmt-vm1）以允许其 DNS 设置生效。</span><span class="sxs-lookup"><span data-stu-id="f1d83-310">Restart the jump box (*ra-adfs-mgmt-vm1* in the *ra-adfs-security-rg* group) to allow its DNS settings to take effect.</span></span>

6. <span data-ttu-id="f1d83-311">[为 AD FS 获取 SSL 证书][adfs_certificates]并在 AD FS VM 上安装此证书。</span><span class="sxs-lookup"><span data-stu-id="f1d83-311">[Obtain an SSL Certificate for AD FS][adfs_certificates] and install this certificate on the AD FS VMs.</span></span> <span data-ttu-id="f1d83-312">请注意，可以通过 jumpbox 连接到它们。</span><span class="sxs-lookup"><span data-stu-id="f1d83-312">Note that you can connect to them through the jump box.</span></span> <span data-ttu-id="f1d83-313">IP 地址是 10.0.5.4 和 10.0.5.5。</span><span class="sxs-lookup"><span data-stu-id="f1d83-313">The IP addresses are *10.0.5.4* and *10.0.5.5*.</span></span> <span data-ttu-id="f1d83-314">默认用户名为 contoso\testuser，密码为 AweSome@PW。</span><span class="sxs-lookup"><span data-stu-id="f1d83-314">The default username is *contoso\testuser* with password *AweSome@PW*.</span></span>
   
   > [!NOTE]
   > <span data-ttu-id="f1d83-315">Deploy-ReferenceArchitecture.ps1 脚本中的注释此时提供有关使用 `makecert` 命令创建自签名测试证书和颁发机构的详细说明。</span><span class="sxs-lookup"><span data-stu-id="f1d83-315">The comments in the Deploy-ReferenceArchitecture.ps1 script at this point provides detailed instructions for creating a self-signed test certificate and authority using the `makecert` command.</span></span> <span data-ttu-id="f1d83-316">但是，仅将这些步骤作为**测试**来执行，请勿在生产环境中使用 makecert 生成的证书。</span><span class="sxs-lookup"><span data-stu-id="f1d83-316">However, perform these steps as a **test** only and do not use the certificates generated by makecert in a production environment.</span></span>
   > 
   > 

7. <span data-ttu-id="f1d83-317">运行以下 PowerShell 命令以部署 AD FS 服务器场：</span><span class="sxs-lookup"><span data-stu-id="f1d83-317">Run the following PowerShell command to deploy the AD FS server farm:</span></span>
   
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Adfs
    ``` 

8. <span data-ttu-id="f1d83-318">在 jumpbox 上，浏览到 `https://adfs.contoso.com/adfs/ls/idpinitiatedsignon.htm` 以测试 AD FS 安装（可能会收到证书警告，对于此测试，可以忽略它）。</span><span class="sxs-lookup"><span data-stu-id="f1d83-318">On the jump box, browse to `https://adfs.contoso.com/adfs/ls/idpinitiatedsignon.htm` to test the AD FS installation (you may receive a certificate warning that you can ignore for this test).</span></span> <span data-ttu-id="f1d83-319">验证 Contoso Corporation 登录页是否出现。</span><span class="sxs-lookup"><span data-stu-id="f1d83-319">Verify that the Contoso Corporation sign-in page appears.</span></span> <span data-ttu-id="f1d83-320">使用密码 AweS0me@PW，以 contoso\testuser 身份登录。</span><span class="sxs-lookup"><span data-stu-id="f1d83-320">Sign in as *contoso\testuser* with password *AweS0me@PW*.</span></span>

9. <span data-ttu-id="f1d83-321">在 AD FS 代理 VM 上安装 SSL 证书。</span><span class="sxs-lookup"><span data-stu-id="f1d83-321">Install the SSL certificate on the AD FS proxy VMs.</span></span> <span data-ttu-id="f1d83-322">IP 地址是 10.0.6.4 和 10.0.6.5。</span><span class="sxs-lookup"><span data-stu-id="f1d83-322">The IP addresses are *10.0.6.4* and *10.0.6.5*.</span></span>

10. <span data-ttu-id="f1d83-323">运行以下 PowerShell 命令以部署第一台 AD FS 代理服务器：</span><span class="sxs-lookup"><span data-stu-id="f1d83-323">Run the following PowerShell command to deploy the first AD FS proxy server:</span></span>
   
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Proxy1
    ```

11. <span data-ttu-id="f1d83-324">按照脚本显示的说明测试第一台代理服务器的安装。</span><span class="sxs-lookup"><span data-stu-id="f1d83-324">Follow the instructions displayed by the script to test the installation of the first proxy server.</span></span>

12. <span data-ttu-id="f1d83-325">运行以下 PowerShell 命令以部署第二台代理服务器：</span><span class="sxs-lookup"><span data-stu-id="f1d83-325">Run the following PowerShell command to deploy the second proxy server:</span></span>
    
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Proxy2
    ```

13. <span data-ttu-id="f1d83-326">按照脚本显示的说明测试完整代理配置。</span><span class="sxs-lookup"><span data-stu-id="f1d83-326">Follow the instructions displayed by the script to test the complete proxy configuration.</span></span>

## <a name="next-steps"></a><span data-ttu-id="f1d83-327">后续步骤</span><span class="sxs-lookup"><span data-stu-id="f1d83-327">Next steps</span></span>

* <span data-ttu-id="f1d83-328">了解 [Azure Active Directory][aad]。</span><span class="sxs-lookup"><span data-stu-id="f1d83-328">Learn about [Azure Active Directory][aad].</span></span>
* <span data-ttu-id="f1d83-329">了解 [Azure Active Directory B2C][aadb2c]。</span><span class="sxs-lookup"><span data-stu-id="f1d83-329">Learn about [Azure Active Directory B2C][aadb2c].</span></span>

<!-- links -->
[extending-ad-to-azure]: adds-extend-domain.md

[vm-recommendations]: ../virtual-machines-windows/single-vm.md
[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md
[hybrid-azure-on-prem-vpn]: ../hybrid-networking/vpn.md

[azure-cli]: /azure/azure-resource-manager/xplat-cli-azure-resource-manager
[DRS]: https://technet.microsoft.com/library/dn280945.aspx
[where-to-place-an-fs-proxy]: https://technet.microsoft.com/library/dd807048.aspx
[ADDRS]: https://technet.microsoft.com/library/dn486831.aspx
[plan-your-adfs-deployment]: https://msdn.microsoft.com/library/azure/dn151324.aspx
[ad_network_recommendations]: #network_configuration_recommendations_for_AD_DS_VMs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[create_service_account_for_adfs_farm]: https://technet.microsoft.com/library/dd807078.aspx
[adfs-configuration-database]: https://technet.microsoft.com/library/ee913581(v=ws.11).aspx
[active-directory-federation-services]: https://technet.microsoft.com/windowsserver/dd448613.aspx
[security-considerations]: #security-considerations
[recommendations]: #recommendations
[active-directory-federation-services-overview]: https://technet.microsoft.com/library/hh831502(v=ws.11).aspx
[establishing-federation-trust]: https://blogs.msdn.microsoft.com/alextch/2011/06/27/establishing-federation-trust/
[Deploying_a_federation_server_farm]:  https://azure.microsoft.com/documentation/articles/active-directory-aadconnect-azure-adfs/
[install_and_configure_the_web_application_proxy_server]: https://technet.microsoft.com/library/dn383662.aspx
[publish_applications_using_AD_FS_preauthentication]: https://technet.microsoft.com/library/dn383640.aspx
[managing-adfs-components]: https://technet.microsoft.com/library/cc759026.aspx
[oms-adfs-pack]: https://www.microsoft.com/download/details.aspx?id=41184
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[aad]: https://azure.microsoft.com/documentation/services/active-directory/
[aadb2c]: https://azure.microsoft.com/documentation/services/active-directory-b2c/
[adfs-intro]: /azure/active-directory/active-directory-aadconnect-azure-adfs
[github]: https://github.com/mspnp/reference-architectures/tree/master/identity/adfs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[considerations]: ./considerations.md
[visio-download]: https://archcenter.azureedge.net/cdn/identity-architectures.vsdx
[0]: ./images/adfs.png "使用 Active Directory 保护混合网络体系结构的安全"
