---
title: "在 Azure 中创建 AD DS 资源林"
description: "如何在 Azure 中创建受信任的 Active Directory 域。\n指南,vpn 网关,ExpressRoute,负载均衡器,虚拟网络,active-directory"
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Identity management
pnp.series.prev: adds-extend-domain
pnp.series.next: adfs
cardTitle: Create an AD DS forest in Azure
ms.openlocfilehash: b946afa91e8bd303c51f97e18be170c4105cc8c5
ms.sourcegitcommit: 8ab30776e0c4cdc16ca0dcc881960e3108ad3e94
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/08/2017
---
# <a name="create-an-active-directory-domain-services-ad-ds-resource-forest-in-azure"></a><span data-ttu-id="00877-104">在 Azure 中创建 Active Directory 域服务 (AD DS) 资源林</span><span class="sxs-lookup"><span data-stu-id="00877-104">Create an Active Directory Domain Services (AD DS) resource forest in Azure</span></span>

<span data-ttu-id="00877-105">此参考体系结构展示了如何在 Azure 中创建本地 AD 林中的域信任的一个单独 Active Directory 域。</span><span class="sxs-lookup"><span data-stu-id="00877-105">This reference architecture shows how to create a separate Active Directory domain in Azure that is trusted by domains in your on-premises AD forest.</span></span> [<span data-ttu-id="00877-106">**部署此解决方案**。</span><span class="sxs-lookup"><span data-stu-id="00877-106">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="00877-107">[![0]][0]</span><span class="sxs-lookup"><span data-stu-id="00877-107">[![0]][0]</span></span> 

<span data-ttu-id="00877-108">*下载此体系结构的 [Visio 文件][visio-download]。*</span><span class="sxs-lookup"><span data-stu-id="00877-108">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="00877-109">Active Directory 域服务 (AD DS) 以分层结构存储标识信息。</span><span class="sxs-lookup"><span data-stu-id="00877-109">Active Directory Domain Services (AD DS) stores identity information in a hierarchical structure.</span></span> <span data-ttu-id="00877-110">分层结构中的顶层节点称为林。</span><span class="sxs-lookup"><span data-stu-id="00877-110">The top node in the hierarchical structure is known as a forest.</span></span> <span data-ttu-id="00877-111">林包含域，域包含其他类型的对象。</span><span class="sxs-lookup"><span data-stu-id="00877-111">A forest contains domains, and domains contain other types of objects.</span></span> <span data-ttu-id="00877-112">此参考体系结构在 Azure 中创建与本地域之间具有单向传出信任关系的 AD DS 林。</span><span class="sxs-lookup"><span data-stu-id="00877-112">This reference architecture creates an AD DS forest in Azure with a one-way outgoing trust relationship with an on-premises domain.</span></span> <span data-ttu-id="00877-113">Azure 中的林包含本地不存在的一个域。</span><span class="sxs-lookup"><span data-stu-id="00877-113">The forest in Azure contains a domain that does not exist on-premises.</span></span> <span data-ttu-id="00877-114">由于信任关系，将信任针对本地域的登录，允许其访问该单独 Azure 域中的资源。</span><span class="sxs-lookup"><span data-stu-id="00877-114">Because of the trust relationship, logons made against on-premises domains can be trusted for access to resources in the separate Azure domain.</span></span> 

<span data-ttu-id="00877-115">此体系结构的典型用途包括为云中拥有的对象和标识维护安全隔离，以及将各个域从本地迁移到云。</span><span class="sxs-lookup"><span data-stu-id="00877-115">Typical uses for this architecture include maintaining security separation for objects and identities held in the cloud, and migrating individual domains from on-premises to the cloud.</span></span> 

<span data-ttu-id="00877-116">有关其他注意事项，请参阅[选择用于将本地 Active Directory 与 Azure 相集成的解决方案][considerations]。</span><span class="sxs-lookup"><span data-stu-id="00877-116">For additional considerations, see [Choose a solution for integrating on-premises Active Directory with Azure][considerations].</span></span> 

## <a name="architecture"></a><span data-ttu-id="00877-117">体系结构</span><span class="sxs-lookup"><span data-stu-id="00877-117">Architecture</span></span>

<span data-ttu-id="00877-118">该体系结构具有以下组件。</span><span class="sxs-lookup"><span data-stu-id="00877-118">The architecture has the following components.</span></span>

* <span data-ttu-id="00877-119">**本地网络**。</span><span class="sxs-lookup"><span data-stu-id="00877-119">**On-premises network**.</span></span> <span data-ttu-id="00877-120">本地网络包含其自己的 Active Directory 林和域。</span><span class="sxs-lookup"><span data-stu-id="00877-120">The on-premises network contains its own Active Directory forest and domains.</span></span>
* <span data-ttu-id="00877-121">**Active Directory 服务器**。</span><span class="sxs-lookup"><span data-stu-id="00877-121">**Active Directory servers**.</span></span> <span data-ttu-id="00877-122">它们是域控制器，用于实现在云中作为 VM 运行的域服务。</span><span class="sxs-lookup"><span data-stu-id="00877-122">These are domain controllers implementing domain services running as VMs in the cloud.</span></span> <span data-ttu-id="00877-123">这些服务器托管的林中包含独立于本地域的一个或多个域。</span><span class="sxs-lookup"><span data-stu-id="00877-123">These servers host a forest containing one or more domains, separate from those located on-premises.</span></span>
* <span data-ttu-id="00877-124">**单向信任关系**。</span><span class="sxs-lookup"><span data-stu-id="00877-124">**One-way trust relationship**.</span></span> <span data-ttu-id="00877-125">关系图中的示例显示了从 Azure 中的域到本地域的单向信任。</span><span class="sxs-lookup"><span data-stu-id="00877-125">The example in the diagram shows a one-way trust from the domain in Azure to the on-premises domain.</span></span> <span data-ttu-id="00877-126">此关系允许本地用户访问 Azure 中的域的资源，但无法反向访问。</span><span class="sxs-lookup"><span data-stu-id="00877-126">This relationship enables on-premises users to access resources in the domain in Azure, but not the other way around.</span></span> <span data-ttu-id="00877-127">如果云用户也需要访问本地资源，则可以创建双向信任。</span><span class="sxs-lookup"><span data-stu-id="00877-127">It is possible to create a two-way trust if cloud users also require access to on-premises resources.</span></span>
* <span data-ttu-id="00877-128">**Active Directory 子网**。</span><span class="sxs-lookup"><span data-stu-id="00877-128">**Active Directory subnet**.</span></span> <span data-ttu-id="00877-129">AD DS 服务器托管在一个单独的子网中。</span><span class="sxs-lookup"><span data-stu-id="00877-129">The AD DS servers are hosted in a separate subnet.</span></span> <span data-ttu-id="00877-130">网络安全组 (NSG) 规则对 AD DS 服务器进行保护，并提供防火墙以阻止来自意外来源的流量。</span><span class="sxs-lookup"><span data-stu-id="00877-130">Network security group (NSG) rules protect the AD DS servers and provide a firewall against traffic from unexpected sources.</span></span>
* <span data-ttu-id="00877-131">**Azure 网关**。</span><span class="sxs-lookup"><span data-stu-id="00877-131">**Azure gateway**.</span></span> <span data-ttu-id="00877-132">Azure 网关在本地网络与 Azure VNet 之间提供连接。</span><span class="sxs-lookup"><span data-stu-id="00877-132">The Azure gateway provides a connection between the on-premises network and the Azure VNet.</span></span> <span data-ttu-id="00877-133">这可以是 [VPN 连接][azure-vpn-gateway]，也可以是 [Azure ExpressRoute][azure-expressroute]。</span><span class="sxs-lookup"><span data-stu-id="00877-133">This can be a [VPN connection][azure-vpn-gateway] or [Azure ExpressRoute][azure-expressroute].</span></span> <span data-ttu-id="00877-134">有关详细信息，请参阅[在 Azure 中实现安全的混合网络体系结构][implementing-a-secure-hybrid-network-architecture]。</span><span class="sxs-lookup"><span data-stu-id="00877-134">For more information, see [Implementing a secure hybrid network architecture in Azure][implementing-a-secure-hybrid-network-architecture].</span></span>

## <a name="recommendations"></a><span data-ttu-id="00877-135">建议</span><span class="sxs-lookup"><span data-stu-id="00877-135">Recommendations</span></span>

<span data-ttu-id="00877-136">有关在 Azure 中实施 Active Directory 的具体建议，请参阅以下文章：</span><span class="sxs-lookup"><span data-stu-id="00877-136">For specific recommendations on implementing Active Directory in Azure, see the following articles:</span></span>

- <span data-ttu-id="00877-137">[将 Active Directory 域服务 (AD DS) 扩展到 Azure][adds-extend-domain]。</span><span class="sxs-lookup"><span data-stu-id="00877-137">[Extending Active Directory Domain Services (AD DS) to Azure][adds-extend-domain].</span></span> 
- <span data-ttu-id="00877-138">[在 Azure 虚拟机上部署 Windows Server Active Directory 的指南][ad-azure-guidelines]。</span><span class="sxs-lookup"><span data-stu-id="00877-138">[Guidelines for Deploying Windows Server Active Directory on Azure Virtual Machines][ad-azure-guidelines].</span></span>

### <a name="trust"></a><span data-ttu-id="00877-139">信任</span><span class="sxs-lookup"><span data-stu-id="00877-139">Trust</span></span>

<span data-ttu-id="00877-140">本地域包含在与云中的域不同的林中。</span><span class="sxs-lookup"><span data-stu-id="00877-140">The on-premises domains are contained within a different forest from the domains in the cloud.</span></span> <span data-ttu-id="00877-141">若要在云中启用对本地用户的身份验证，Azure 中的域必须信任本地林中的登录域。</span><span class="sxs-lookup"><span data-stu-id="00877-141">To enable authentication of on-premises users in the cloud, the domains in Azure must trust the logon domain in the on-premises forest.</span></span> <span data-ttu-id="00877-142">同样，如果云为外部用户提供了登录域，则本地林可能需要信任云域。</span><span class="sxs-lookup"><span data-stu-id="00877-142">Similarly, if the cloud provides a logon domain for external users, it may be necessary for the on-premises forest to trust the cloud domain.</span></span>

<span data-ttu-id="00877-143">可以通过[创建林信任][creating-forest-trusts]在林级别建立信任，或者通过[创建外部信任][creating-external-trusts]在域级别建立信任。</span><span class="sxs-lookup"><span data-stu-id="00877-143">You can establish trusts at the forest level by [creating forest trusts][creating-forest-trusts], or at the domain level by [creating external trusts][creating-external-trusts].</span></span> <span data-ttu-id="00877-144">林级别信任在两个林中的所有域之间创建关系。</span><span class="sxs-lookup"><span data-stu-id="00877-144">A forest level trust creates a relationship between all domains in two forests.</span></span> <span data-ttu-id="00877-145">外部域级别信任仅在两个指定的域之间创建关系。</span><span class="sxs-lookup"><span data-stu-id="00877-145">An external domain level trust only creates a relationship between two specified domains.</span></span> <span data-ttu-id="00877-146">只应当在不同林中的域之间创建外部域级别信任。</span><span class="sxs-lookup"><span data-stu-id="00877-146">You should only create external domain level trusts between domains in different forests.</span></span>

<span data-ttu-id="00877-147">信任可以是单向的，也可以是双向的：</span><span class="sxs-lookup"><span data-stu-id="00877-147">Trusts can be unidirectional (one-way) or bidirectional (two-way):</span></span>

* <span data-ttu-id="00877-148">单向信任允许一个域或林（称为*传入*域或林）中的用户访问另一个域或林（*传出*域或林）中拥有的资源。</span><span class="sxs-lookup"><span data-stu-id="00877-148">A one-way trust enables users in one domain or forest (known as the *incoming* domain or forest) to access the resources held in another (the *outgoing* domain or forest).</span></span>
* <span data-ttu-id="00877-149">双向信任允许任一域或林中的用户访问另一个域或林中拥有的资源。</span><span class="sxs-lookup"><span data-stu-id="00877-149">A two-way trust enables users in either domain or forest to access resources held in the other.</span></span>

<span data-ttu-id="00877-150">下表总结了一些简单方案的信任配置：</span><span class="sxs-lookup"><span data-stu-id="00877-150">The following table summarizes trust configurations for some simple scenarios:</span></span>

| <span data-ttu-id="00877-151">场景</span><span class="sxs-lookup"><span data-stu-id="00877-151">Scenario</span></span> | <span data-ttu-id="00877-152">本地信任</span><span class="sxs-lookup"><span data-stu-id="00877-152">On-premises trust</span></span> | <span data-ttu-id="00877-153">云信任</span><span class="sxs-lookup"><span data-stu-id="00877-153">Cloud trust</span></span> |
| --- | --- | --- |
| <span data-ttu-id="00877-154">本地用户需要访问云中的资源，但云中的用户不需要访问本地资源</span><span class="sxs-lookup"><span data-stu-id="00877-154">On-premises users require access to resources in the cloud, but not vice versa</span></span> |<span data-ttu-id="00877-155">单向、传入</span><span class="sxs-lookup"><span data-stu-id="00877-155">One-way, incoming</span></span> |<span data-ttu-id="00877-156">单向、传出</span><span class="sxs-lookup"><span data-stu-id="00877-156">One-way, outgoing</span></span> |
| <span data-ttu-id="00877-157">云中的用户需要访问本地资源，本地用户不需要访问云中的资源</span><span class="sxs-lookup"><span data-stu-id="00877-157">Users in the cloud require access to resources located on-premises, but not vice versa</span></span> |<span data-ttu-id="00877-158">单向、传出</span><span class="sxs-lookup"><span data-stu-id="00877-158">One-way, outgoing</span></span> |<span data-ttu-id="00877-159">单向、传入</span><span class="sxs-lookup"><span data-stu-id="00877-159">One-way, incoming</span></span> |
| <span data-ttu-id="00877-160">云中的和本地用户都需要访问云中和本地拥有的资源。</span><span class="sxs-lookup"><span data-stu-id="00877-160">Users in the cloud and on-premises both requires access to resources held in the cloud and on-premises</span></span> |<span data-ttu-id="00877-161">双向、传入和传出</span><span class="sxs-lookup"><span data-stu-id="00877-161">Two-way, incoming and outgoing</span></span> |<span data-ttu-id="00877-162">双向、传入和传出</span><span class="sxs-lookup"><span data-stu-id="00877-162">Two-way, incoming and outgoing</span></span> |

## <a name="scalability-considerations"></a><span data-ttu-id="00877-163">可伸缩性注意事项</span><span class="sxs-lookup"><span data-stu-id="00877-163">Scalability considerations</span></span>

<span data-ttu-id="00877-164">Active Directory 能够针对属于同一域的域控制器自动进行缩放。</span><span class="sxs-lookup"><span data-stu-id="00877-164">Active Directory is automatically scalable for domain controllers that are part of the same domain.</span></span> <span data-ttu-id="00877-165">请求被分布到域中的所有控制器中。</span><span class="sxs-lookup"><span data-stu-id="00877-165">Requests are distributed across all controllers within a domain.</span></span> <span data-ttu-id="00877-166">可以添加另一个域控制器，并且它将与该域自动同步。</span><span class="sxs-lookup"><span data-stu-id="00877-166">You can add another domain controller, and it synchronizes automatically with the domain.</span></span> <span data-ttu-id="00877-167">不要配置单独的负载均衡器来将流量定向到该域中的控制器。</span><span class="sxs-lookup"><span data-stu-id="00877-167">Do not configure a separate load balancer to direct traffic to controllers within the domain.</span></span> <span data-ttu-id="00877-168">确保所有域控制器都有足够的内存和存储资源来处理域数据库。</span><span class="sxs-lookup"><span data-stu-id="00877-168">Ensure that all domain controllers have sufficient memory and storage resources to handle the domain database.</span></span> <span data-ttu-id="00877-169">使所有域控制器 VM 具有相同的大小。</span><span class="sxs-lookup"><span data-stu-id="00877-169">Make all domain controller VMs the same size.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="00877-170">可用性注意事项</span><span class="sxs-lookup"><span data-stu-id="00877-170">Availability considerations</span></span>

<span data-ttu-id="00877-171">请至少为每个域预配两个域控制器。</span><span class="sxs-lookup"><span data-stu-id="00877-171">Provision at least two domain controllers for each domain.</span></span> <span data-ttu-id="00877-172">这可以在服务器之间实现自动复制。</span><span class="sxs-lookup"><span data-stu-id="00877-172">This enables automatic replication between servers.</span></span> <span data-ttu-id="00877-173">为充当 Active Directory 服务器（用于处理每个域）的 VM 创建一个可用性集。</span><span class="sxs-lookup"><span data-stu-id="00877-173">Create an availability set for the VMs acting as Active Directory servers handling each domain.</span></span> <span data-ttu-id="00877-174">至少在此可用性集中放置两个服务器。</span><span class="sxs-lookup"><span data-stu-id="00877-174">Put at least two servers in this availability set.</span></span>

<span data-ttu-id="00877-175">另外，请考虑将每个域中的一个或多个服务器指定为[备用操作主机][standby-operations-masters]，以应对作为灵活单一主机操作 (FSMO) 角色连接到服务器时失败的情况。</span><span class="sxs-lookup"><span data-stu-id="00877-175">Also, consider designating one or more servers in each domain as [standby operations masters][standby-operations-masters] in case connectivity to a server acting as a flexible single master operation (FSMO) role fails.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="00877-176">可管理性注意事项</span><span class="sxs-lookup"><span data-stu-id="00877-176">Manageability considerations</span></span>

<span data-ttu-id="00877-177">有关管理和监视注意事项的详细信息，请参阅[将 Active Directory 扩展到 Azure][adds-extend-domain]。</span><span class="sxs-lookup"><span data-stu-id="00877-177">For information about management and monitoring considerations, see [Extending Active Directory to Azure][adds-extend-domain].</span></span> 
 
<span data-ttu-id="00877-178">有关详细信息，请参阅[监视 Active Directory][monitoring_ad]。</span><span class="sxs-lookup"><span data-stu-id="00877-178">For additional information, see [Monitoring Active Directory][monitoring_ad].</span></span> <span data-ttu-id="00877-179">可以在管理子网中的监视服务器上安装 [Microsoft Systems Center][microsoft_systems_center] 之类的工具来帮助执行这些任务。</span><span class="sxs-lookup"><span data-stu-id="00877-179">You can install tools such as [Microsoft Systems Center][microsoft_systems_center] on a monitoring server in the management subnet to help perform these tasks.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="00877-180">安全注意事项</span><span class="sxs-lookup"><span data-stu-id="00877-180">Security considerations</span></span>

<span data-ttu-id="00877-181">林级别信任是可传递的。</span><span class="sxs-lookup"><span data-stu-id="00877-181">Forest level trusts are transitive.</span></span> <span data-ttu-id="00877-182">如果在一个本地林与云中的一个林之间建立了林级别信任，则此信任将扩展到在任一林中创建的其他新域。</span><span class="sxs-lookup"><span data-stu-id="00877-182">If you establish a forest level trust between an on-premises forest and a forest in the cloud, this trust is extended to other new domains created in either forest.</span></span> <span data-ttu-id="00877-183">如果出于安全目的而使用域提供隔离，请考虑仅在域级别创建信任。</span><span class="sxs-lookup"><span data-stu-id="00877-183">If you use domains to provide separation for security purposes, consider creating trusts at the domain level only.</span></span> <span data-ttu-id="00877-184">域级别信任是不可传递的。</span><span class="sxs-lookup"><span data-stu-id="00877-184">Domain level trusts are non-transitive.</span></span>

<span data-ttu-id="00877-185">有关特定于 Active Directory 的安全注意事项，请参阅[将 Active Directory 扩展到 Azure][adds-extend-domain] 中的安全注意事项部分。</span><span class="sxs-lookup"><span data-stu-id="00877-185">For Active Directory-specific security considerations, see the security considerations section in [Extending Active Directory to Azure][adds-extend-domain].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="00877-186">部署解决方案</span><span class="sxs-lookup"><span data-stu-id="00877-186">Deploy the solution</span></span>

<span data-ttu-id="00877-187">[GitHub][github] 上提供了一个用于部署此参考体系结构的解决方案。</span><span class="sxs-lookup"><span data-stu-id="00877-187">A solution is available on [GitHub][github] to deploy this reference architecture.</span></span> <span data-ttu-id="00877-188">若要运行部署此解决方案的 Powershell 脚本，需要具有 Azure CLI 的最新版本。</span><span class="sxs-lookup"><span data-stu-id="00877-188">You will need the latest version of the Azure CLI to run the Powershell script that deploys the solution.</span></span> <span data-ttu-id="00877-189">若要部署此参考体系结构，请执行以下步骤：</span><span class="sxs-lookup"><span data-stu-id="00877-189">To deploy the reference architecture, follow these steps:</span></span>

1. <span data-ttu-id="00877-190">将解决方案文件夹从 [GitHub][github] 克隆到本地计算机。</span><span class="sxs-lookup"><span data-stu-id="00877-190">Download or clone the solution folder from [GitHub][github] to your local machine.</span></span>

2. <span data-ttu-id="00877-191">打开 Azure CLI 并导航到本地解决方案文件夹。</span><span class="sxs-lookup"><span data-stu-id="00877-191">Open the Azure CLI and navigate to the local solution folder.</span></span>

3. <span data-ttu-id="00877-192">运行以下命令：</span><span class="sxs-lookup"><span data-stu-id="00877-192">Run the following command:</span></span>
   
    ```Powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```
   
    <span data-ttu-id="00877-193">将 `<subscription id>` 替换为你的 Azure 订阅 ID。</span><span class="sxs-lookup"><span data-stu-id="00877-193">Replace `<subscription id>` with your Azure subscription ID.</span></span>
   
    <span data-ttu-id="00877-194">对于 `<location>`，请指定一个 Azure 区域，例如 `eastus` 或 `westus`。</span><span class="sxs-lookup"><span data-stu-id="00877-194">For `<location>`, specify an Azure region, such as `eastus` or `westus`.</span></span>
   
    <span data-ttu-id="00877-195">`<mode>` 参数控制部署粒度，可以是下列值之一：</span><span class="sxs-lookup"><span data-stu-id="00877-195">The `<mode>` parameter controls the granularity of the deployment, and can be one of the following values:</span></span>
   
   * <span data-ttu-id="00877-196">`Onpremise`：部署模拟的本地环境。</span><span class="sxs-lookup"><span data-stu-id="00877-196">`Onpremise`: deploys the simulated on-premises environment.</span></span>
   * <span data-ttu-id="00877-197">`Infrastructure`：在 Azure 中部署 VNet 基础结构和 jumpbox。</span><span class="sxs-lookup"><span data-stu-id="00877-197">`Infrastructure`: deploys the VNet infrastructure and jump box in Azure.</span></span>
   * <span data-ttu-id="00877-198">`CreateVpn`：部署 Azure 虚拟网络网关并将其连接到模拟的本地网络。</span><span class="sxs-lookup"><span data-stu-id="00877-198">`CreateVpn`: deploys the Azure virtual network gateway and connects it to the simulated on-premises network.</span></span>
   * <span data-ttu-id="00877-199">`AzureADDS`：部署充当 Active Directory DS 服务器的 VM，将 Active Directory 部署到这些 VM，并在 Azure 中部署域。</span><span class="sxs-lookup"><span data-stu-id="00877-199">`AzureADDS`: deploys the VMs acting as Active Directory DS servers, deploys Active Directory to these VMs, and deploys the domain in Azure.</span></span>
   * <span data-ttu-id="00877-200">`WebTier`：部署 Web 层 VM 和负载均衡器。</span><span class="sxs-lookup"><span data-stu-id="00877-200">`WebTier`: deploys the web tier VMs and load balancer.</span></span>
   * <span data-ttu-id="00877-201">`Prepare`：部署上述所有部署。</span><span class="sxs-lookup"><span data-stu-id="00877-201">`Prepare`: deploys all of the preceding deployments.</span></span> <span data-ttu-id="00877-202">**如果没有现有的本地网络，但是希望如上所述部署完整的参考体系结构以用于测试或评估，则这是建议使用的选项。**</span><span class="sxs-lookup"><span data-stu-id="00877-202">**This is the recommended option if If you do not have an existing on-premises network but you want to deploy the complete reference architecture described above for testing or evaluation.**</span></span> 
   * <span data-ttu-id="00877-203">`Workload`：部署业务和数据层 VM 和负载均衡器。</span><span class="sxs-lookup"><span data-stu-id="00877-203">`Workload`: deploys the business and data tier VMs and load balancers.</span></span> <span data-ttu-id="00877-204">注意，`Prepare` 部署中未包括这些 VM。</span><span class="sxs-lookup"><span data-stu-id="00877-204">Note that these VMs are not included in the `Prepare` deployment.</span></span>

4. <span data-ttu-id="00877-205">等待部署完成。</span><span class="sxs-lookup"><span data-stu-id="00877-205">Wait for the deployment to complete.</span></span> <span data-ttu-id="00877-206">如果要部署 `Prepare` 部署，则将需要花费几个小时。</span><span class="sxs-lookup"><span data-stu-id="00877-206">If you are deploying the `Prepare` deployment, it will take several hours.</span></span>
     
5. <span data-ttu-id="00877-207">如果使用模拟的本地配置，请配置传入信任关系：</span><span class="sxs-lookup"><span data-stu-id="00877-207">If you are using the simulated on-premises configuration, configure the incoming trust relationship:</span></span>
   
   1. <span data-ttu-id="00877-208">连接到 jumpbox（*ra-adtrust-security-rg* 资源组中的 *ra-adtrust-mgmt-vm1*）。</span><span class="sxs-lookup"><span data-stu-id="00877-208">Connect to the jump box (*ra-adtrust-mgmt-vm1* in the *ra-adtrust-security-rg* resource group).</span></span> <span data-ttu-id="00877-209">以 *testuser* 身份和密码 *AweS0me@PW* 登录。</span><span class="sxs-lookup"><span data-stu-id="00877-209">Log in as *testuser* with password *AweS0me@PW*.</span></span>
   2. <span data-ttu-id="00877-210">在 jumpbox 上，在 *contoso.com* 域（本地域）中的第一个 VM 上打开一个 RDP 会话。</span><span class="sxs-lookup"><span data-stu-id="00877-210">On the jump box open an RDP session on the first VM in the *contoso.com* domain (the on-premises domain).</span></span> <span data-ttu-id="00877-211">此 VM 具有 IP 地址 192.168.0.4。</span><span class="sxs-lookup"><span data-stu-id="00877-211">This VM has the IP address 192.168.0.4.</span></span> <span data-ttu-id="00877-212">用户名为 *contoso\testuser*，密码为 *AweS0me@PW*。</span><span class="sxs-lookup"><span data-stu-id="00877-212">The username is *contoso\testuser* with password *AweS0me@PW*.</span></span>
   3. <span data-ttu-id="00877-213">下载 [incoming-trust.ps1][incoming-trust] 脚本并运行它来创建来自 *treyresearch.com* 域的传入信任。</span><span class="sxs-lookup"><span data-stu-id="00877-213">Download the [incoming-trust.ps1][incoming-trust] script and run it to create the incoming trust from the *treyresearch.com* domain.</span></span>

6. <span data-ttu-id="00877-214">如果你使用自己的本地基础结构：</span><span class="sxs-lookup"><span data-stu-id="00877-214">If you are using your own on-premises infrastructure:</span></span>
   
   1. <span data-ttu-id="00877-215">则下载 [incoming-trust.ps1][incoming-trust] 脚本。</span><span class="sxs-lookup"><span data-stu-id="00877-215">Download the [incoming-trust.ps1][incoming-trust] script.</span></span>
   2. <span data-ttu-id="00877-216">编辑脚本并将 `$TrustedDomainName` 变量的值替换为你自己的域的值。</span><span class="sxs-lookup"><span data-stu-id="00877-216">Edit the script and replace the value of the `$TrustedDomainName` variable with the name of your own domain.</span></span>
   3. <span data-ttu-id="00877-217">运行该脚本。</span><span class="sxs-lookup"><span data-stu-id="00877-217">Run the script.</span></span>

7. <span data-ttu-id="00877-218">从 jumpbox 中，连接到 *treyresearch.com* 域（云中的域）中的第一个 VM。</span><span class="sxs-lookup"><span data-stu-id="00877-218">From the jump-box, connect to the first VM in the *treyresearch.com* domain (the domain in the cloud).</span></span> <span data-ttu-id="00877-219">此 VM 具有 IP 地址 10.0.4.4。</span><span class="sxs-lookup"><span data-stu-id="00877-219">This VM has the IP address 10.0.4.4.</span></span> <span data-ttu-id="00877-220">用户名为 *treyresearch\testuser*，密码为 *AweS0me@PW*。</span><span class="sxs-lookup"><span data-stu-id="00877-220">The username is *treyresearch\testuser* with password *AweS0me@PW*.</span></span>

8. <span data-ttu-id="00877-221">下载 [outgoing-trust.ps1][outgoing-trust] 脚本并运行它来创建来自 *treyresearch.com* 域的传入信任。</span><span class="sxs-lookup"><span data-stu-id="00877-221">Download the [outgoing-trust.ps1][outgoing-trust] script and run it to create the incoming trust from the *treyresearch.com* domain.</span></span> <span data-ttu-id="00877-222">如果你在使用自己的本地计算机，请先编辑该脚本。</span><span class="sxs-lookup"><span data-stu-id="00877-222">If you are using your own on-premises machines, then edit the script first.</span></span> <span data-ttu-id="00877-223">将 `$TrustedDomainName` 变量设置为你的本地域的名称，在 `$TrustedDomainDnsIpAddresses` 变量中指定此域的 Active Directory DS 服务器的 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="00877-223">Set the `$TrustedDomainName` variable to the name of your on-premises domain, and specify the IP addresses of the Active Directory DS servers for this domain in the `$TrustedDomainDnsIpAddresses` variable.</span></span>

9. <span data-ttu-id="00877-224">等待几分钟时间，直到前面的步骤完成，然后连接到本地 VM 并执行[验证信任][verify-a-trust]一文中列出的步骤来确定是否正确配置了 *contoso.com* 与 *treyresearch.com* 域之间的信任关系。</span><span class="sxs-lookup"><span data-stu-id="00877-224">Wait a few minutes for the previous steps to complete, then connect to an on-premises VM and perform the steps outlined in the article [Verify a Trust][verify-a-trust] to determine whether the trust relationship between the *contoso.com* and *treyresearch.com* domains is correctly configured.</span></span>

## <a name="next-steps"></a><span data-ttu-id="00877-225">后续步骤</span><span class="sxs-lookup"><span data-stu-id="00877-225">Next steps</span></span>

* <span data-ttu-id="00877-226">了解[将本地 AD DS 域扩展到 Azure][adds-extend-domain] 的最佳做法</span><span class="sxs-lookup"><span data-stu-id="00877-226">Learn the best practices for [extending your on-premises AD DS domain to Azure][adds-extend-domain]</span></span>
* <span data-ttu-id="00877-227">了解在 Azure 中[创建 AD FS 基础结构][adfs]的最佳做法。</span><span class="sxs-lookup"><span data-stu-id="00877-227">Learn the best practices for [creating an AD FS infrastructure][adfs] in Azure.</span></span>

<!-- links -->
[adds-extend-domain]: adds-extend-domain.md
[adfs]: adfs.md

[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[running-VMs-for-an-N-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[ad-azure-guidelines]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[azure-expressroute]: https://azure.microsoft.com/documentation/articles/expressroute-introduction/
[azure-vpn-gateway]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-vpngateways/
[considerations]: ./considerations.md
[creating-external-trusts]: https://technet.microsoft.com/library/cc816837(v=ws.10).aspx
[creating-forest-trusts]: https://technet.microsoft.com/library/cc816810(v=ws.10).aspx
[github]: https://github.com/mspnp/reference-architectures/tree/master/identity/adds-forest
[incoming-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/incoming-trust.ps1
[microsoft_systems_center]: https://www.microsoft.com/server-cloud/products/system-center-2016/
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[solution-script]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/Deploy-ReferenceArchitecture.ps1
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[outgoing-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/outgoing-trust.ps1
[verify-a-trust]: https://technet.microsoft.com/library/cc753821.aspx
[visio-download]: https://archcenter.azureedge.net/cdn/identity-architectures.vsdx
[0]: ./images/adds-forest.png "使用独立的 Active Directory 域保护混合网络体系结构的安全"