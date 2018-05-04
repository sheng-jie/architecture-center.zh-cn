---
title: 在 Azure 上运行 Jenkins 服务器
description: 本参考体系结构演示如何在 Azure 上部署和运行使用单一登录 (SSO) 保护的可伸缩企业级 Jenkins 服务器。
author: njray
ms.date: 01/21/18
ms.openlocfilehash: 5f9c54e71a8750e88de1ae633ccc1316f8375d3a
ms.sourcegitcommit: 0de300b6570e9990e5c25efc060946cb9d079954
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/03/2018
---
# <a name="run-a-jenkins-server-on-azure"></a><span data-ttu-id="78892-103">在 Azure 上运行 Jenkins 服务器</span><span class="sxs-lookup"><span data-stu-id="78892-103">Run a Jenkins server on Azure</span></span>

<span data-ttu-id="78892-104">本参考体系结构演示如何在 Azure 上部署和运行使用单一登录 (SSO) 保护的可伸缩企业级 Jenkins 服务器。</span><span class="sxs-lookup"><span data-stu-id="78892-104">This reference architecture shows how to deploy and operate a scalable, enterprise-grade Jenkins server on Azure secured with single sign-on (SSO).</span></span> <span data-ttu-id="78892-105">本体系结构还使用 Azure Monitor 来监视 Jenkins 服务器的状态。</span><span class="sxs-lookup"><span data-stu-id="78892-105">The architecture also uses Azure Monitor to monitor the state of the Jenkins server.</span></span> [<span data-ttu-id="78892-106">**部署此解决方案**。</span><span class="sxs-lookup"><span data-stu-id="78892-106">**Deploy this solution**.</span></span>](#deploy-the-solution)

![Azure 上运行的 Jenkins 服务器][0]

<span data-ttu-id="78892-108">下载包含本体系结构关系图的 [Visio 文件](https://archcenter.blob.core.windows.net/cdn/Jenkins-architecture.vsdx)。</span><span class="sxs-lookup"><span data-stu-id="78892-108">*Download a [Visio file](https://archcenter.blob.core.windows.net/cdn/Jenkins-architecture.vsdx) that contains this architecture diagram.*</span></span>

<span data-ttu-id="78892-109">本体系结构支持对 Azure 服务实现灾难恢复，但不包括涉及到多个主节点或高可用性 (HA) 功能，且可避免停机的其他高级横向扩展方案。</span><span class="sxs-lookup"><span data-stu-id="78892-109">This architecture supports disaster recovery with Azure services but does not cover more advanced scale-out scenarios involving multiple masters or high availability (HA) with no downtime.</span></span> <span data-ttu-id="78892-110">有关各种 Azure 组件的一般见解，包括有关在 Azure 上构建 CI/CD 管道的分步教程，请参阅 [Azure 上的 Jenkins][jenkins-on-azure]。</span><span class="sxs-lookup"><span data-stu-id="78892-110">For general insights about the various Azure components, including a step-by-step tutorial about building out a CI/CD pipeline on Azure, see [Jenkins on  Azure][jenkins-on-azure].</span></span>

<span data-ttu-id="78892-111">本文档重点介绍支持 Jenkins 所要执行的核心 Azure 操作，包括使用 Azure 存储来维护生成项目、实现 SSO 所需的安全项、可集成的其他服务，以及管道的可伸缩性。</span><span class="sxs-lookup"><span data-stu-id="78892-111">The focus of this document is on the core Azure operations needed to support Jenkins, including the use of Azure Storage to maintain build artifacts, the security items needed for SSO, other services that can be integrated, and scalability for the pipeline.</span></span> <span data-ttu-id="78892-112">本体系结构在设计上使用现有的源代码管理存储库。</span><span class="sxs-lookup"><span data-stu-id="78892-112">The architecture is designed to work with an existing source control repository.</span></span> <span data-ttu-id="78892-113">例如，一种常见方案是基于 GitHub 提交内容启动 Jenkins 作业。</span><span class="sxs-lookup"><span data-stu-id="78892-113">For example, a common scenario is to start Jenkins jobs based on GitHub commits.</span></span>

## <a name="architecture"></a><span data-ttu-id="78892-114">体系结构</span><span class="sxs-lookup"><span data-stu-id="78892-114">Architecture</span></span>

<span data-ttu-id="78892-115">该体系结构包括以下组件：</span><span class="sxs-lookup"><span data-stu-id="78892-115">The architecture consists of the following components:</span></span>

- <span data-ttu-id="78892-116">**资源组。**</span><span class="sxs-lookup"><span data-stu-id="78892-116">**Resource group.**</span></span> <span data-ttu-id="78892-117">[资源组][rg]用于对 Azure 资产进行分组，以便可以根据生存期、所有者和其他条件对其进行管理。</span><span class="sxs-lookup"><span data-stu-id="78892-117">A [resource group][rg] is used to group Azure assets so they can be managed by lifetime, owner, and other criteria.</span></span> <span data-ttu-id="78892-118">使用资源组能够以组的形式部署和监视 Azure 资产，并按资源组跟踪计费成本。</span><span class="sxs-lookup"><span data-stu-id="78892-118">Use resource groups to deploy and monitor Azure assets as a group and track billing costs by resource group.</span></span> <span data-ttu-id="78892-119">还可以删除作为集的资源，这对于测试部署非常有用。</span><span class="sxs-lookup"><span data-stu-id="78892-119">You can also delete resources as a set, which is very useful for test deployments.</span></span>

- <span data-ttu-id="78892-120">**Jenkins 服务器**。</span><span class="sxs-lookup"><span data-stu-id="78892-120">**Jenkins server**.</span></span> <span data-ttu-id="78892-121">部署一个用于运行 [Jenkins][azure-market] 的虚拟机。该虚拟机用作自动化服务器并充当 Jenkins 主节点。</span><span class="sxs-lookup"><span data-stu-id="78892-121">A virtual machine is deployed to run [Jenkins][azure-market] as an automation server and serve as Jenkins Master.</span></span> <span data-ttu-id="78892-122">本参考体系结构使用 [Azure 上的 Jenkins 解决方案模板][solution]，该模板安装在 Azure 上的 Linux (Ubuntu 16.04 LTS) 虚拟机中。</span><span class="sxs-lookup"><span data-stu-id="78892-122">This reference architecture uses the [solution template for Jenkins on Azure][solution], installed on a Linux (Ubuntu 16.04 LTS) virtual machine on Azure.</span></span> <span data-ttu-id="78892-123">Azure Marketplace 中提供了其他 Jenkins 产品。</span><span class="sxs-lookup"><span data-stu-id="78892-123">Other Jenkins offerings are available in the Azure Marketplace.</span></span>

  > [!NOTE]
  > <span data-ttu-id="78892-124">已在 VM 上安装充当 Jenkins 的反向代理的 Nginx。</span><span class="sxs-lookup"><span data-stu-id="78892-124">Nginx is installed on the VM to act as a reverse proxy to Jenkins.</span></span> <span data-ttu-id="78892-125">可将 Nginx 配置为对 Jenkins 服务器启用 SSL。</span><span class="sxs-lookup"><span data-stu-id="78892-125">You can configure Nginx to enable SSL for the Jenkins server.</span></span>
  > 
  > 

- <span data-ttu-id="78892-126">虚拟网络。</span><span class="sxs-lookup"><span data-stu-id="78892-126">**Virtual network**.</span></span> <span data-ttu-id="78892-127">[虚拟网络][vnet]可将 Azure 资源相互连接，并提供逻辑隔离。</span><span class="sxs-lookup"><span data-stu-id="78892-127">A [virtual network][vnet] connects Azure resources to each other and provides logical isolation.</span></span> <span data-ttu-id="78892-128">在本体系结构中，Jenkins 服务器在虚拟网络中运行。</span><span class="sxs-lookup"><span data-stu-id="78892-128">In this architecture, the Jenkins server runs in a virtual network.</span></span>

- <span data-ttu-id="78892-129">**子网**。</span><span class="sxs-lookup"><span data-stu-id="78892-129">**Subnets**.</span></span> <span data-ttu-id="78892-130">Jenkins 服务器在[子网][subnet]中隔离，因此我们可以更轻松地管理和隔离网络流量，且不影响性能。</span><span class="sxs-lookup"><span data-stu-id="78892-130">The Jenkins server is isolated in a [subnet][subnet] to make it easier to manage and segregate network traffic without impacting performance.</span></span>

- <span data-ttu-id="78892-131"><strong>NSG</strong>。</span><span class="sxs-lookup"><span data-stu-id="78892-131"><strong>NSGs</strong>.</span></span> <span data-ttu-id="78892-132">使用[网络安全组][nsg] (NSG) 可以限制从 Internet 发往虚拟网络子网的网络流量。</span><span class="sxs-lookup"><span data-stu-id="78892-132">Use [network security groups][nsg] (NSGs) to restrict network traffic from the Internet to the subnet of a virtual network.</span></span>

- <span data-ttu-id="78892-133">**托管磁盘**。</span><span class="sxs-lookup"><span data-stu-id="78892-133">**Managed disks**.</span></span> <span data-ttu-id="78892-134">[托管磁盘][managed-disk]是一种持久性虚拟硬盘 (VHD)，可用作应用程序存储，同时还可维护 Jenkins 服务器的状态并提供灾难恢复。</span><span class="sxs-lookup"><span data-stu-id="78892-134">A [managed disk][managed-disk] is a persistent virtual hard disk (VHD) used for application storage and also to maintain the state of the Jenkins server and provide disaster recovery.</span></span> <span data-ttu-id="78892-135">数据磁盘存储在 Azure 存储中。</span><span class="sxs-lookup"><span data-stu-id="78892-135">Data disks are stored in Azure Storage.</span></span> <span data-ttu-id="78892-136">为实现高性能，我们建议使用[高级存储][premium]。</span><span class="sxs-lookup"><span data-stu-id="78892-136">For high performance, [premium storage][premium] is recommended.</span></span>

- <span data-ttu-id="78892-137">**Azure Blob 存储**。</span><span class="sxs-lookup"><span data-stu-id="78892-137">**Azure Blob Storage**.</span></span> <span data-ttu-id="78892-138">[Windows Azure 存储插件][configure-storage]使用 Azure Blob 存储来存储所创建的并与其他 Jenkins 生成组件共享的生成项目。</span><span class="sxs-lookup"><span data-stu-id="78892-138">The [Windows Azure Storage plugin][configure-storage] uses Azure Blob  Storage to store the build artifacts that are created and shared with other Jenkins builds.</span></span>

- <span data-ttu-id="78892-139"><strong>Azure Active Directory (Azure AD)</strong>。</span><span class="sxs-lookup"><span data-stu-id="78892-139"><strong>Azure Active Directory (Azure AD)</strong>.</span></span> <span data-ttu-id="78892-140">[Azure AD][azure-ad] 支持用户身份验证，并允许设置 SSO。</span><span class="sxs-lookup"><span data-stu-id="78892-140">[Azure AD][azure-ad] supports user authentication, allowing you to set up SSO.</span></span> <span data-ttu-id="78892-141">Azure AD [服务主体][service-principal]使用[基于角色的访问控制][rbac] (RBAC) 为工作流中的每个角色授权定义策略和权限。</span><span class="sxs-lookup"><span data-stu-id="78892-141">Azure AD [service principals][service-principal] define the policy and permissions for each role authorization in the workflow, using [role-based access control][rbac] (RBAC).</span></span> <span data-ttu-id="78892-142">每个服务主体与某个 Jenkins 作业相关联。</span><span class="sxs-lookup"><span data-stu-id="78892-142">Each service principal is associated with a Jenkins job.</span></span>

- <span data-ttu-id="78892-143">**Azure Key Vault**。</span><span class="sxs-lookup"><span data-stu-id="78892-143">**Azure Key Vault.**</span></span> <span data-ttu-id="78892-144">为了在需要机密时管理用于预配 Azure 资源的机密和加密密钥，本体系结构使用了 [Key Vault][key-vault]。</span><span class="sxs-lookup"><span data-stu-id="78892-144">To manage secrets and cryptographic keys used to provision Azure resources when secrets are required, this architecture uses [Key Vault][key-vault].</span></span> <span data-ttu-id="78892-145">有关存储与管道中应用程序关联的机密的其他帮助，请参阅适用于 Jenkins 的 [Azure 凭据][configure-credential]插件。</span><span class="sxs-lookup"><span data-stu-id="78892-145">For added help storing secrets associated with the application in the pipeline, see also the [Azure Credentials][configure-credential] plugin for Jenkins.</span></span>

- <span data-ttu-id="78892-146">**Azure 监视服务**。</span><span class="sxs-lookup"><span data-stu-id="78892-146">**Azure monitoring services**.</span></span> <span data-ttu-id="78892-147">此服务[监视][monitor]托管 Jenkins 的 Azure 虚拟机。</span><span class="sxs-lookup"><span data-stu-id="78892-147">This service [monitors][monitor] the Azure virtual machine hosting Jenkins.</span></span> <span data-ttu-id="78892-148">此部署监视虚拟机状态和 CPU 利用率，并发送警报。</span><span class="sxs-lookup"><span data-stu-id="78892-148">This deployment monitors the virtual machine status and CPU utilization and sends alerts.</span></span>

## <a name="recommendations"></a><span data-ttu-id="78892-149">建议</span><span class="sxs-lookup"><span data-stu-id="78892-149">Recommendations</span></span>

<span data-ttu-id="78892-150">以下建议适用于大多数方案。</span><span class="sxs-lookup"><span data-stu-id="78892-150">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="78892-151">除非有优先于这些建议的特定要求，否则请遵循这些建议。</span><span class="sxs-lookup"><span data-stu-id="78892-151">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="azure-ad"></a><span data-ttu-id="78892-152">Azure AD</span><span class="sxs-lookup"><span data-stu-id="78892-152">Azure AD</span></span>

<span data-ttu-id="78892-153">Azure 订阅的 [Azure AD][azure-ad] 租户用于对 Jenkins 用户启用 SSO，并设置可让 Jenkins 作业访问 Azure 资源的[服务主体][service-principal]。</span><span class="sxs-lookup"><span data-stu-id="78892-153">The [Azure AD][azure-ad] tenant for your Azure subscription is used to enable SSO for Jenkins users and set up [service principals][service-principal] that enable Jenkins jobs to access Azure resources.</span></span>

<span data-ttu-id="78892-154">SSO 身份验证和授权由 Jenkins 服务器上安装的 Azure AD 插件实现。</span><span class="sxs-lookup"><span data-stu-id="78892-154">SSO authentication and authorization are implemented by the Azure AD plugin  installed on the Jenkins server.</span></span> <span data-ttu-id="78892-155">登录到 Jenkins 服务器时，可以借助 SSO 使用 Azure AD 中的组织凭据进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="78892-155">SSO allows you to authenticate using your organization credentials from Azure AD when logging on to the Jenkins server.</span></span> <span data-ttu-id="78892-156">配置 Azure AD 插件时，可以指定用户对 Jenkins 服务器的授权访问级别。</span><span class="sxs-lookup"><span data-stu-id="78892-156">When configuring the Azure AD plugin, you can specify the level of a user’s authorized access to the Jenkins server.</span></span>

<span data-ttu-id="78892-157">若要提供有权访问 Azure 资源的 Jenkins 作业，Azure AD 管理员可以创建服务主体。</span><span class="sxs-lookup"><span data-stu-id="78892-157">To provide Jenkins jobs with access to Azure resources, an Azure AD administrator creates service principals.</span></span> <span data-ttu-id="78892-158">这些授权应用程序（在本例中为 Jenkins 作业）[在经过身份验证且获授权的情况下可以访问][ad-sp] Azure 资源。</span><span class="sxs-lookup"><span data-stu-id="78892-158">These grant applications—in this case, the Jenkins jobs—[authenticated, authorized access][ad-sp] to Azure resources.</span></span>

<span data-ttu-id="78892-159">[RBAC][rbac] 进一步通过用户或服务主体的角色来定义和控制他们（它们）对 Azure 资源的访问。</span><span class="sxs-lookup"><span data-stu-id="78892-159">[RBAC][rbac] further defines and controls access to Azure resources for users or service principals through their assigned role.</span></span> <span data-ttu-id="78892-160">支持内置角色和自定义角色。</span><span class="sxs-lookup"><span data-stu-id="78892-160">Both built-in and custom roles are supported.</span></span> <span data-ttu-id="78892-161">角色还有助于保护管道，并确保正确分配和授权用户或代理的责任。</span><span class="sxs-lookup"><span data-stu-id="78892-161">Roles also help secure the pipeline and ensure that a user’s or agent’s responsibilities are assigned and authorized correctly.</span></span> <span data-ttu-id="78892-162">此外，可以设置 RBAC 来限制对 Azure 资产的访问。</span><span class="sxs-lookup"><span data-stu-id="78892-162">In addition, RBAC can be set up to limit access to Azure assets.</span></span> <span data-ttu-id="78892-163">例如，可将用户限制为只能使用特定资源组中的资产。</span><span class="sxs-lookup"><span data-stu-id="78892-163">For example, a user can be limited to working with only the assets in a particular resource group.</span></span>

### <a name="storage"></a><span data-ttu-id="78892-164">存储</span><span class="sxs-lookup"><span data-stu-id="78892-164">Storage</span></span>

<span data-ttu-id="78892-165">使用从 Azure Marketplace 安装的 Jenkins [Windows Azure 存储插件][storage-plugin]能够存储可与其他生成和测试组件共享的生成项目。</span><span class="sxs-lookup"><span data-stu-id="78892-165">Use the Jenkins [Windows Azure Storage plugin][storage-plugin], which is installed from the Azure Marketplace, to store build artifacts that can be shared with other builds and tests.</span></span> <span data-ttu-id="78892-166">必须先配置一个 Azure 存储帐户，然后 Jenkins 作业才能使用此插件。</span><span class="sxs-lookup"><span data-stu-id="78892-166">An Azure Storage account must be configured before this plugin can be used by the Jenkins jobs.</span></span>

### <a name="jenkins-azure-plugins"></a><span data-ttu-id="78892-167">Jenkins Azure 插件</span><span class="sxs-lookup"><span data-stu-id="78892-167">Jenkins Azure plugins</span></span>

<span data-ttu-id="78892-168">Azure 上的 Jenkins 解决方案模板会安装多个 Azure 插件。</span><span class="sxs-lookup"><span data-stu-id="78892-168">The solution template for Jenkins on Azure installs several Azure plugins.</span></span> <span data-ttu-id="78892-169">Azure DevOps 团队会生成并维护该解决方案模板和以下插件，这些插件使用 Azure Marketplace 中的其他 Jenkins 产品，以及本地安装的 Jenkins 主节点：</span><span class="sxs-lookup"><span data-stu-id="78892-169">The Azure DevOps Team builds and maintains the solution template and the following plugins, which work with other Jenkins offerings in Azure Marketplace as well as any Jenkins master set up on premises:</span></span>

-   <span data-ttu-id="78892-170">[Azure AD 插件][configure-azure-ad]可让 Jenkins 服务器基于 Azure AD 支持用户的 SSO。</span><span class="sxs-lookup"><span data-stu-id="78892-170">[Azure AD plugin][configure-azure-ad] allows the Jenkins server to support SSO for users based on Azure AD.</span></span>

-   <span data-ttu-id="78892-171">[Azure VM 代理][configure-agent]插件使用 Azure 资源管理器模板在 Azure 虚拟机中创建 Jenkins 代理。</span><span class="sxs-lookup"><span data-stu-id="78892-171">[Azure VM Agents][configure-agent] plugin uses an Azure Resource Manager template to create Jenkins agents in Azure virtual machines.</span></span>

-   <span data-ttu-id="78892-172">[Azure 凭据][configure-credential]插件可用于在 Jenkins 中存储 Azure 服务主体凭据。</span><span class="sxs-lookup"><span data-stu-id="78892-172">[Azure Credentials][configure-credential] plugin allows you to store Azure service principal credentials in Jenkins.</span></span>

-   <span data-ttu-id="78892-173">[Windows Azure 存储插件][configure-storage]可将生成项目上传到 [Azure Blob 存储][blob]并从中下载生成依赖项。</span><span class="sxs-lookup"><span data-stu-id="78892-173">[Windows Azure Storage plugin][configure-storage] uploads build artifacts to, or downloads build dependencies from, [Azure Blob storage][blob].</span></span>

<span data-ttu-id="78892-174">我们还建议查看使用 Azure 资源的所有可用 Azure 插件列表，该列表不断扩充。</span><span class="sxs-lookup"><span data-stu-id="78892-174">We also recommend reviewing the growing list of all available Azure plugins that work with Azure resources.</span></span> <span data-ttu-id="78892-175">若要查看最新列表，请访问 [Jenkins 插件索引][index]并搜索 Azure。</span><span class="sxs-lookup"><span data-stu-id="78892-175">To see all the latest list, visit [Jenkins Plugin Index][index] and search for Azure.</span></span> <span data-ttu-id="78892-176">例如，可将以下插件用于部署：</span><span class="sxs-lookup"><span data-stu-id="78892-176">For example, the following plugins are available for deployment:</span></span>

-   <span data-ttu-id="78892-177">借助 [Azure 容器代理][container-agents]可在 Jenkins 中以代理的形式运行容器。</span><span class="sxs-lookup"><span data-stu-id="78892-177">[Azure Container Agents][container-agents] helps you to run a container as an agent in Jenkins.</span></span>

-   <span data-ttu-id="78892-178">[Kubernetes 持续部署](https://aka.ms/azjenkinsk8s)可将资源配置部署到 Kubernetes 群集。</span><span class="sxs-lookup"><span data-stu-id="78892-178">[Kubernetes Continuous Deploy](https://aka.ms/azjenkinsk8s) deploys resource configurations to a Kubernetes cluster.</span></span>

-   <span data-ttu-id="78892-179">[Azure 容器服务][acs]可以使用 Kubernetes、包含 Marathon 的 DC/OS 或 Docker Swarm 将配置部署到 Azure 容器服务。</span><span class="sxs-lookup"><span data-stu-id="78892-179">[Azure Container Service][acs] deploys configurations to Azure Container Service with Kubernetes, DC/OS with Marathon, or Docker Swarm.</span></span>

-   <span data-ttu-id="78892-180">[Azure Functions][functions] 可将项目部署到 Azure 函数。</span><span class="sxs-lookup"><span data-stu-id="78892-180">[Azure Functions][functions] deploys your project to Azure Function.</span></span>

-   <span data-ttu-id="78892-181">[Azure 应用服务][app-service]可部署到 Azure 应用服务。</span><span class="sxs-lookup"><span data-stu-id="78892-181">[Azure App Service][app-service] deploys to Azure App Service.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="78892-182">可伸缩性注意事项</span><span class="sxs-lookup"><span data-stu-id="78892-182">Scalability considerations</span></span>

<span data-ttu-id="78892-183">可以缩放 Jenkins 以支持极大型工作负荷。</span><span class="sxs-lookup"><span data-stu-id="78892-183">Jenkins can scale to support very large workloads.</span></span> <span data-ttu-id="78892-184">对于弹性生成，请不要在 Jenkins 主服务器上运行生成。</span><span class="sxs-lookup"><span data-stu-id="78892-184">For elastic builds, do not run builds on the Jenkins master server.</span></span> <span data-ttu-id="78892-185">应将生成任务卸载到 Jenkins 代理，以便按需弹性缩放。</span><span class="sxs-lookup"><span data-stu-id="78892-185">Instead, offload build tasks to Jenkins agents, which can be elastically scaled in and out as need.</span></span> <span data-ttu-id="78892-186">缩放代理时，请考虑两个选项：</span><span class="sxs-lookup"><span data-stu-id="78892-186">Consider two options for scaling agents:</span></span>

- <span data-ttu-id="78892-187">使用 [Azure VM 代理][vm-agent]插件创建可在 Azure VM 中运行的 Jenkins 代理。</span><span class="sxs-lookup"><span data-stu-id="78892-187">Use the [Azure VM Agents][vm-agent] plugin to create Jenkins agents that run in Azure VMs.</span></span> <span data-ttu-id="78892-188">此插件可让代理实现弹性扩展，并可使用不同类型的虚拟机。</span><span class="sxs-lookup"><span data-stu-id="78892-188">This plugin enables elastic scale-out for agents and can use distinct types of virtual machines.</span></span> <span data-ttu-id="78892-189">可以从 Azure Marketplace 中选择不同的基础映像，也可以使用自定义映像。</span><span class="sxs-lookup"><span data-stu-id="78892-189">You can select a different base image from Azure Marketplace or use a custom image.</span></span> <span data-ttu-id="78892-190">有关 Jenkins 代理如何缩放的详细信息，请参阅 Jenkins 文档中的 [Architecting for Scale][scale]（构建可缩放的解决方案）。</span><span class="sxs-lookup"><span data-stu-id="78892-190">For details about how the Jenkins agents scale, see [Architecting for Scale][scale] in the Jenkins documentation.</span></span>

- <span data-ttu-id="78892-191">使用 [Azure 容器代理][container-agents]插件在[包含 Kubernetes 的 Azure 容器服务](/azure/container-service/kubernetes/)或 [Azure 容器实例](/azure/container-instances/)中以代理形式运行容器。</span><span class="sxs-lookup"><span data-stu-id="78892-191">Use the [Azure Container Agents][container-agents] plugin to run a container as an agent in either [Azure Container Service with Kubernetes](/azure/container-service/kubernetes/), or [Azure Container Instances](/azure/container-instances/).</span></span>

<span data-ttu-id="78892-192">虚拟机的缩放成本通常比容器更高。</span><span class="sxs-lookup"><span data-stu-id="78892-192">Virtual machines generally cost more to scale than containers.</span></span> <span data-ttu-id="78892-193">但是，若要使用容器进行缩放，则必须配合容器运行生成过程。</span><span class="sxs-lookup"><span data-stu-id="78892-193">To use containers for scaling, however, your build process must run with containers.</span></span>

<span data-ttu-id="78892-194">此外，可以使用 Azure 存储来共享生成项目，供其他生成代理在管道的下一阶段中使用。</span><span class="sxs-lookup"><span data-stu-id="78892-194">Also, use Azure Storage to share build artifacts that may be used in the next stage of the pipeline by other build agents.</span></span>

### <a name="scaling-the-jenkins-server"></a><span data-ttu-id="78892-195">缩放 Jenkins 服务器</span><span class="sxs-lookup"><span data-stu-id="78892-195">Scaling the Jenkins server</span></span> 

<span data-ttu-id="78892-196">可以通过更改 VM 大小来扩展或缩减 Jenkins 服务器 VM。</span><span class="sxs-lookup"><span data-stu-id="78892-196">You can scale the Jenkins server VM up or down by changing the VM size.</span></span> <span data-ttu-id="78892-197">[Azure 上的 Jenkins 解决方案模板][azure-market]默认指定了 DS2 v2 大小（两个 CPU，7 GB RAM）。</span><span class="sxs-lookup"><span data-stu-id="78892-197">The [solution template for Jenkins on Azure][azure-market] specifies the DS2 v2 size (with two CPUs, 7 GB) by default.</span></span> <span data-ttu-id="78892-198">此大小可以处理中小型团队工作负荷。</span><span class="sxs-lookup"><span data-stu-id="78892-198">This size handles a small to medium team workload.</span></span> <span data-ttu-id="78892-199">构建服务器时，可以通过选择不同的选项来更改 VM 大小。</span><span class="sxs-lookup"><span data-stu-id="78892-199">Change the VM size by choosing a different option when building out the server.</span></span> 

<span data-ttu-id="78892-200">哪个服务器大小合适取决于预期工作负荷的大小。</span><span class="sxs-lookup"><span data-stu-id="78892-200">Selecting the correct server size depends on the size of the expected workload.</span></span> <span data-ttu-id="78892-201">Jenkins 社区维护了一套[选择指南][selection-guide]，以帮助确定最符合要求的配置。</span><span class="sxs-lookup"><span data-stu-id="78892-201">The Jenkins community maintains a [selection guide][selection-guide] to help identify the configuration that best meets your requirements.</span></span> <span data-ttu-id="78892-202">Azure 提供许多 [Linux VM 大小][sizes-linux]，可满足任何要求。</span><span class="sxs-lookup"><span data-stu-id="78892-202">Azure offers many [sizes for Linux VMs][sizes-linux] to meet any requirements.</span></span> <span data-ttu-id="78892-203">有关缩放 Jenkins 主节点的详细信息，请参阅 Jenkins 社区编写的[最佳做法][best-practices]。</span><span class="sxs-lookup"><span data-stu-id="78892-203">For more information about scaling the Jenkins master, refer to the Jenkins community of [best practices][best-practices], which also includes details about scaling Jenkins master.</span></span>


## <a name="availability-considerations"></a><span data-ttu-id="78892-204">可用性注意事项</span><span class="sxs-lookup"><span data-stu-id="78892-204">Availability considerations</span></span>

<span data-ttu-id="78892-205">Jenkins 服务器上下文中的可用性指的是能够恢复与工作流关联的任何状态信息，如测试结果、你已创建的库或其他项目。</span><span class="sxs-lookup"><span data-stu-id="78892-205">Availability in the context of a Jenkins server means being able to recover any state information associated with your workflow, such as test results, libraries you have created, or other artifacts.</span></span> <span data-ttu-id="78892-206">必须维护关键的工作流状态或项目才能在 Jenkins 服务器出现故障时恢复工作流。</span><span class="sxs-lookup"><span data-stu-id="78892-206">Critical workflow state or artifacts must be maintained to recover the workflow if the Jenkins server goes down.</span></span> <span data-ttu-id="78892-207">若要评估可用性要求，请考虑两个常见指标：</span><span class="sxs-lookup"><span data-stu-id="78892-207">To assess your availability requirements, consider two common metrics:</span></span>

-   <span data-ttu-id="78892-208">恢复时间目标 (RTO) 指定允许 Jenkins 发生故障多长时间。</span><span class="sxs-lookup"><span data-stu-id="78892-208">Recovery Time Objective (RTO) specifies how long you can go without Jenkins.</span></span>

-   <span data-ttu-id="78892-209">恢复点目标 (RPO) 表示服务中断影响到 Jenkins 时，允许丢失多少数据。</span><span class="sxs-lookup"><span data-stu-id="78892-209">Recovery Point Objective (RPO) indicates how much data you can afford to lose if a disruption in service affects Jenkins.</span></span>

<span data-ttu-id="78892-210">在实践中，RTO 和 RPO 涉及到冗余和备份。</span><span class="sxs-lookup"><span data-stu-id="78892-210">In practice, RTO and RPO imply redundancy and backup.</span></span> <span data-ttu-id="78892-211">可用性对于硬件恢复而言不是一个问题（这是 Azure 的部分责任），它旨在确保维持 Jenkins 服务器的状态。</span><span class="sxs-lookup"><span data-stu-id="78892-211">Availability is not a question of hardware recovery—that is part of Azure—but rather ensuring you maintain the state of your Jenkins server.</span></span> <span data-ttu-id="78892-212">Microsoft 提供了用于单个 VM 实例的[服务级别协议][sla] (SLA)。</span><span class="sxs-lookup"><span data-stu-id="78892-212">Microsoft offers a [service level agreement][sla] (SLA) for single VM instances.</span></span> <span data-ttu-id="78892-213">如果此 SLA 无法满足你的运行时间要求，请确保做好灾难恢复的规划，或考虑使用[多主节点 Jenkins 服务器][multi-master]部署（本文档未介绍）。</span><span class="sxs-lookup"><span data-stu-id="78892-213">If this SLA doesn't meet your uptime requirements, make sure you have a plan for disaster recovery, or consider using a [multi-master Jenkins server][multi-master] deployment (not covered in this document).</span></span>

<span data-ttu-id="78892-214">考虑使用部署过程步骤 7 中的灾难恢复[脚本][disaster]，来创建包含托管磁盘的 Azure 存储帐户用于存储 Jenkins 服务器状态。</span><span class="sxs-lookup"><span data-stu-id="78892-214">Consider using the disaster recovery [scripts][disaster] in step 7 of the deployment to create an Azure Storage account with managed disks to store the Jenkins server state.</span></span> <span data-ttu-id="78892-215">如果 Jenkins 发生故障，可将它还原到此单独存储帐户中存储的状态。</span><span class="sxs-lookup"><span data-stu-id="78892-215">If Jenkins goes down, it can be restored to the state stored in this separate storage account.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="78892-216">安全注意事项</span><span class="sxs-lookup"><span data-stu-id="78892-216">Security considerations</span></span>

<span data-ttu-id="78892-217">使用以下方法有助于在基本 Jenkins 服务器上锁定安全状态，因为它的基本状态是不安全的。</span><span class="sxs-lookup"><span data-stu-id="78892-217">Use the following approaches to help lock down security on a basic Jenkins server, since in its basic state, it is not secure.</span></span>

-   <span data-ttu-id="78892-218">设置登录 Jenkins 服务器的安全途径。</span><span class="sxs-lookup"><span data-stu-id="78892-218">Set up a secure way to log into the Jenkins server.</span></span> <span data-ttu-id="78892-219">本体系结构使用 HTTP 和公共 IP，但默认情况下 HTTP 不安全。</span><span class="sxs-lookup"><span data-stu-id="78892-219">This architecture uses HTTP and has a public IP, but HTTP is not secure by default.</span></span> <span data-ttu-id="78892-220">考虑在 [Nginx 服务器上设置 HTTPS][nginx] 用于安全登录。</span><span class="sxs-lookup"><span data-stu-id="78892-220">Consider setting up [HTTPS on the Nginx server][nginx] being used for a secure logon.</span></span>

    > [!NOTE]
    > <span data-ttu-id="78892-221">将 SSL 添加到服务器时，请针对 Jenkins 子网创建一条 NSG 规则，以打开端口 443。</span><span class="sxs-lookup"><span data-stu-id="78892-221">When adding SSL to your server, create an NSG rule for the Jenkins subnet to open port 443.</span></span> <span data-ttu-id="78892-222">有关详细信息，请参阅[如何使用 Azure 门户向虚拟机开放端口][port443]。</span><span class="sxs-lookup"><span data-stu-id="78892-222">For more information, see [How to open ports to a virtual machine with the Azure portal][port443].</span></span>
    > 

-   <span data-ttu-id="78892-223">确保 Jenkins 配置可防止跨站点请求伪造（“管理 Jenkins”\>“配置全局安全性”）。</span><span class="sxs-lookup"><span data-stu-id="78892-223">Ensure that the Jenkins configuration prevents cross site request forgery (Manage Jenkins \> Configure Global Security).</span></span> <span data-ttu-id="78892-224">这是 Microsoft Jenkins 服务器的默认设置。</span><span class="sxs-lookup"><span data-stu-id="78892-224">This is the default for Microsoft Jenkins Server.</span></span>

-   <span data-ttu-id="78892-225">使用[矩阵授权策略插件][matrix]配置对 Jenkins 仪表板的只读访问。</span><span class="sxs-lookup"><span data-stu-id="78892-225">Configure read-only access to the Jenkins dashboard by using the [Matrix Authorization Strategy Plugin][matrix].</span></span>

-   <span data-ttu-id="78892-226">安装 [Azure 凭据][configure-credential]插件，使用 Key Vault 来处理 Azure 资产、管道中的代理以及第三方组件的机密。</span><span class="sxs-lookup"><span data-stu-id="78892-226">Install the [Azure Credentials][configure-credential] plugin to use Key Vault to handle secrets for the Azure assets, the agents in the pipeline, and third-party components.</span></span>

-   <span data-ttu-id="78892-227">使用 RBAC 将服务主体的访问权限限制为运行作业的最低必需权限。</span><span class="sxs-lookup"><span data-stu-id="78892-227">Use RBAC to restrict the access of the service principal to the minimum required to run the jobs.</span></span> <span data-ttu-id="78892-228">这样可以限制恶意作业造成的损害的范围。</span><span class="sxs-lookup"><span data-stu-id="78892-228">This helps limit the scope of damage from a rogue job.</span></span>

<span data-ttu-id="78892-229">Jenkins 作业通常需要使用机密来访问需要授权的 Azure 服务，例如 Azure 容器服务。</span><span class="sxs-lookup"><span data-stu-id="78892-229">Jenkins jobs often require secrets to access Azure services that require authorization, such as Azure Container Service.</span></span> <span data-ttu-id="78892-230">结合 [Azure 凭据插件][configure-credential]使用 [Key Vault][key-vault] 来安全管理这些机密。</span><span class="sxs-lookup"><span data-stu-id="78892-230">Use [Key Vault][key-vault] along with the [Azure Credential plugin][configure-credential] to manage these secrets securely.</span></span> <span data-ttu-id="78892-231">使用 Key Vault 存储服务主体凭据、密码、令牌和其他机密。</span><span class="sxs-lookup"><span data-stu-id="78892-231">Use Key Vault to store service principal credentials, passwords, tokens, and other secrets.</span></span>

<span data-ttu-id="78892-232">若要在一个中心视图中查看 Azure 资源的安全状态，请使用 [Azure 安全中心][security-center]。</span><span class="sxs-lookup"><span data-stu-id="78892-232">To get a central view of the security state of your Azure resources, use [Azure Security Center][security-center].</span></span> <span data-ttu-id="78892-233">安全中心监视潜在的安全问题，并全面描述了部署的安全运行状况。</span><span class="sxs-lookup"><span data-stu-id="78892-233">Security Center monitors potential security issues and provides a comprehensive picture of the security health of your deployment.</span></span> <span data-ttu-id="78892-234">安全中心针对每个 Azure 订阅进行配置。</span><span class="sxs-lookup"><span data-stu-id="78892-234">Security Center is configured per Azure subscription.</span></span> <span data-ttu-id="78892-235">启用安全数据收集，如 [Azure 安全中心快速入门指南][quick-start]中所述。</span><span class="sxs-lookup"><span data-stu-id="78892-235">Enable security data collection as described in the [Azure Security Center quick start guide][quick-start].</span></span> <span data-ttu-id="78892-236">启用数据收集后，安全中心会自动扫描该订阅下创建的所有虚拟机。</span><span class="sxs-lookup"><span data-stu-id="78892-236">When data collection is enabled, Security Center automatically scans any virtual machines created under that subscription.</span></span>

<span data-ttu-id="78892-237">Jenkins 服务器具有自身的用户管理系统，Jenkins 社区提供了有关[在 Azure 上保护 Jenkins 实例][secure-jenkins]的最佳做法。</span><span class="sxs-lookup"><span data-stu-id="78892-237">The Jenkins server has its own user management system, and the Jenkins community provides best practices for [securing a Jenkins instance on Azure][secure-jenkins].</span></span> <span data-ttu-id="78892-238">Azure 上的 Jenkins 解决方案模板实施这些最佳做法。</span><span class="sxs-lookup"><span data-stu-id="78892-238">The solution template for Jenkins on Azure implements these best practices.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="78892-239">可管理性注意事项</span><span class="sxs-lookup"><span data-stu-id="78892-239">Manageability considerations</span></span>

<span data-ttu-id="78892-240">使用资源组来组织已部署的 Azure 资源。</span><span class="sxs-lookup"><span data-stu-id="78892-240">Use resource groups to organize the Azure resources that are deployed.</span></span> <span data-ttu-id="78892-241">在单独的资源组中部署生产环境和开发/测试环境，以便可以监视每个环境的资源，并按资源组汇总计费成本。</span><span class="sxs-lookup"><span data-stu-id="78892-241">Deploy production environments and development/test environments in separate resource groups, so that you can monitor each environment’s resources and roll up billing costs by resource group.</span></span> <span data-ttu-id="78892-242">还可以删除作为集的资源，这对于测试部署非常有用。</span><span class="sxs-lookup"><span data-stu-id="78892-242">You can also delete resources as a set, which is very useful for test deployments.</span></span>

<span data-ttu-id="78892-243">Azure 提供多种功能用于[监视和诊断][monitoring-diag]整个基础结构。</span><span class="sxs-lookup"><span data-stu-id="78892-243">Azure provides several features for [monitoring and diagnostics][monitoring-diag] of the overall infrastructure.</span></span> <span data-ttu-id="78892-244">为了监视 CPU 使用率，本体系结构部署了 Azure Monitor。</span><span class="sxs-lookup"><span data-stu-id="78892-244">To monitor CPU usage, this architecture deploys Azure Monitor.</span></span> <span data-ttu-id="78892-245">例如，可以使用 Azure Monitor 来监视 CPU 使用率，并在 CPU 使用率超过 80% 时发送通知。</span><span class="sxs-lookup"><span data-stu-id="78892-245">For example, you can use Azure Monitor to monitor CPU utilization, and send a notification if CPU usage exceeds 80 percent.</span></span> <span data-ttu-id="78892-246">（CPU 使用率较高意味着可能需要扩展 Jenkins 服务器 VM。）此外，当 VM 发生故障或不可用时，可向指定的用户发送通知。</span><span class="sxs-lookup"><span data-stu-id="78892-246">(High CPU usage indicates that you might want to scale up the Jenkins server VM.) You can also notify a designated user if the VM fails or becomes unavailable.</span></span>

## <a name="communities"></a><span data-ttu-id="78892-247">社区</span><span class="sxs-lookup"><span data-stu-id="78892-247">Communities</span></span>

<span data-ttu-id="78892-248">社区可以解答问题，并帮助设置成功的部署。</span><span class="sxs-lookup"><span data-stu-id="78892-248">Communities can answer questions and help you set up a successful deployment.</span></span> <span data-ttu-id="78892-249">请注意以下几点：</span><span class="sxs-lookup"><span data-stu-id="78892-249">Consider the following:</span></span>

-   [<span data-ttu-id="78892-250">Jenkins 社区博客</span><span class="sxs-lookup"><span data-stu-id="78892-250">Jenkins Community Blog</span></span>](https://jenkins.io/node/)
-   [<span data-ttu-id="78892-251">Azure 论坛</span><span class="sxs-lookup"><span data-stu-id="78892-251">Azure Forum</span></span>](https://azure.microsoft.com/support/forums/)
-   [<span data-ttu-id="78892-252">Stack Overflow Jenkins</span><span class="sxs-lookup"><span data-stu-id="78892-252">Stack Overflow Jenkins</span></span>](https://stackoverflow.com/tags/jenkins/info)

<span data-ttu-id="78892-253">有关 Jenkins 社区编写的其他最佳做法，请访问 [Jenkins 最佳做法][jenkins-best]。</span><span class="sxs-lookup"><span data-stu-id="78892-253">For more best practices from the Jenkins community, visit [Jenkins best practices][jenkins-best].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="78892-254">部署解决方案</span><span class="sxs-lookup"><span data-stu-id="78892-254">Deploy the solution</span></span>

<span data-ttu-id="78892-255">若要部署本体系结构，请执行以下步骤安装 [Azure 上的 Jenkins 解决方案模板][azure-market]，然后安装所需的脚本，用于设置后续步骤中所述的监视和灾难恢复。</span><span class="sxs-lookup"><span data-stu-id="78892-255">To deploy this architecture, follow the steps below to install the [solution template for Jenkins on Azure][azure-market], then install the scripts that set up monitoring and disaster recovery in the steps below.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="78892-256">先决条件</span><span class="sxs-lookup"><span data-stu-id="78892-256">Prerequisites</span></span>

- <span data-ttu-id="78892-257">本参考体系结构需要 Azure 订阅。</span><span class="sxs-lookup"><span data-stu-id="78892-257">This reference architecture requires an Azure subscription.</span></span> 
- <span data-ttu-id="78892-258">若要创建 Azure 服务主体，必须对与部署的 Jenkins 服务器相关联的 Azure AD 租户拥有管理权限。</span><span class="sxs-lookup"><span data-stu-id="78892-258">To create an Azure service principal, you must have admin rights to the Azure AD tenant that is associated with the deployed Jenkins server.</span></span>
- <span data-ttu-id="78892-259">本文档中的说明假设 Jenkins 管理员也是拥有最低“参与者”特权的 Azure 用户。</span><span class="sxs-lookup"><span data-stu-id="78892-259">These instructions assume that the Jenkins administrator is also an Azure user with at least Contributor privileges.</span></span>

### <a name="step-1-deploy-the-jenkins-server"></a><span data-ttu-id="78892-260">步骤 1：部署 Jenkins 服务器</span><span class="sxs-lookup"><span data-stu-id="78892-260">Step 1: Deploy the Jenkins server</span></span>

1.  <span data-ttu-id="78892-261">在 Web 浏览器中打开[用于 Jenkins 的 Azure Marketplace 映像][azure-market]，然后从页面左侧选择“立即获取”。</span><span class="sxs-lookup"><span data-stu-id="78892-261">Open the [Azure Marketplace image for Jenkins][azure-market] in your web browser and select **GET IT NOW** from the left side of the page.</span></span>

2.  <span data-ttu-id="78892-262">查看定价详细信息并选择“继续”，然后选择“创建”，在 Azure 门户中配置 Jenkins 服务器。</span><span class="sxs-lookup"><span data-stu-id="78892-262">Review the pricing details and select **Continue**, then select **Create** to configure the Jenkins server in the Azure portal.</span></span>

<span data-ttu-id="78892-263">有关详细说明，请参阅[通过 Azure 门户在 Azure Linux VM 上创建 Jenkins 服务器][create-jenkins]。</span><span class="sxs-lookup"><span data-stu-id="78892-263">For detailed instructions, see [Create a Jenkins server on an Azure Linux VM from the Azure portal][create-jenkins].</span></span> <span data-ttu-id="78892-264">对于本参考体系结构，使用管理员登录名便足以启动和运行服务器。</span><span class="sxs-lookup"><span data-stu-id="78892-264">For this reference architecture, it is sufficient to get the server up and running with the admin logon.</span></span> <span data-ttu-id="78892-265">然后，可将服务器预配为使用其他各种服务。</span><span class="sxs-lookup"><span data-stu-id="78892-265">Then you can provision it to use various other services.</span></span>

### <a name="step-2-set-up-sso"></a><span data-ttu-id="78892-266">步骤 2：设置 SSO</span><span class="sxs-lookup"><span data-stu-id="78892-266">Step 2: Set up SSO</span></span>

<span data-ttu-id="78892-267">该步骤由 Jenkins 管理员运行。该管理员还必须在订阅的 Azure AD 目录中有一个用户帐户，并且必须分配有“参与者”角色。</span><span class="sxs-lookup"><span data-stu-id="78892-267">The step is run by the Jenkins administrator, who must also have a user account in the subscription’s Azure AD directory and must be assigned the Contributor role.</span></span>

<span data-ttu-id="78892-268">在 Jenkins 服务器中使用来自 Jenkins 更新中心的 [Azure AD 插件][configure-azure-ad]，然后遵照说明设置 SSO。</span><span class="sxs-lookup"><span data-stu-id="78892-268">Use the [Azure AD Plugin][configure-azure-ad] from the Jenkins Update Center in the Jenkins server and follow the instructions to set up SSO.</span></span>

### <a name="step-3-provision-jenkins-server-with-azure-vm-agent-plugin"></a><span data-ttu-id="78892-269">步骤 3：使用 Azure VM 代理插件预配 Jenkins 服务器</span><span class="sxs-lookup"><span data-stu-id="78892-269">Step 3: Provision Jenkins server with Azure VM Agent plugin</span></span>

<span data-ttu-id="78892-270">Jenkins 管理员运行此步骤来设置已安装的 Azure VM 代理插件。</span><span class="sxs-lookup"><span data-stu-id="78892-270">The step is run by the Jenkins administrator to set up the Azure VM Agent plugin, which is already installed.</span></span>

<span data-ttu-id="78892-271">[遵循以下步骤配置插件][configure-agent]。</span><span class="sxs-lookup"><span data-stu-id="78892-271">[Follow these steps to configure the plugin][configure-agent].</span></span> <span data-ttu-id="78892-272">有关为插件设置服务主体的教程，请参阅[使用 Azure VM 代理根据需求缩放 Jenkins 部署][scale-agent]。</span><span class="sxs-lookup"><span data-stu-id="78892-272">For a tutorial about setting up service principals for the plugin, see [Scale your  Jenkins deployments to meet demand with Azure VM agents][scale-agent].</span></span>

### <a name="step-4-provision-jenkins-server-with-azure-storage"></a><span data-ttu-id="78892-273">步骤 4：使用 Azure 存储预配 Jenkins 服务器</span><span class="sxs-lookup"><span data-stu-id="78892-273">Step 4: Provision Jenkins server with Azure Storage</span></span>

<span data-ttu-id="78892-274">Jenkins 管理员运行此步骤来设置已安装的 Windows Azure 存储插件。</span><span class="sxs-lookup"><span data-stu-id="78892-274">The step is run by the Jenkins administrator, who sets up the Windows Azure Storage Plugin, which is already installed.</span></span>

<span data-ttu-id="78892-275">[遵循这些步骤配置插件][configure-storage]。</span><span class="sxs-lookup"><span data-stu-id="78892-275">[Follow these steps to configure the plugin][configure-storage].</span></span>

### <a name="step-5-provision-jenkins-server-with-azure-credential-plugin"></a><span data-ttu-id="78892-276">步骤 5：使用 Azure 凭据插件预配 Jenkins 服务器</span><span class="sxs-lookup"><span data-stu-id="78892-276">Step 5: Provision Jenkins server with Azure Credential plugin</span></span>

<span data-ttu-id="78892-277">Jenkins 管理员运行此步骤来设置已安装的 Azure 凭据插件。</span><span class="sxs-lookup"><span data-stu-id="78892-277">The step is run by the Jenkins administrator to set up the Azure Credential plugin, which is already installed.</span></span>

<span data-ttu-id="78892-278">[遵循这些步骤配置插件][configure-credential]。</span><span class="sxs-lookup"><span data-stu-id="78892-278">[Follow these steps to configure the plugin][configure-credential].</span></span>

### <a name="step-6-provision-jenkins-server-for-monitoring-by-the-azure-monitor-service"></a><span data-ttu-id="78892-279">步骤 6：预配 Jenkins 服务器以通过 Azure Monitor 服务进行监视</span><span class="sxs-lookup"><span data-stu-id="78892-279">Step 6: Provision Jenkins server for monitoring by the Azure Monitor Service</span></span>

<span data-ttu-id="78892-280">若要设置 Jenkins 服务器监视，请遵照[在 Azure Monitor 中针对 Azure 服务创建指标警报][create-metric]中的说明。</span><span class="sxs-lookup"><span data-stu-id="78892-280">To set up monitoring for your Jenkins server, follow the instructions in [Create metric alerts in Azure Monitor for Azure services][create-metric].</span></span>

### <a name="step-7-provision-jenkins-server-with-managed-disks-for-disaster-recovery"></a><span data-ttu-id="78892-281">步骤 7：使用托管磁盘预配 Jenkins 服务器以实现灾难恢复</span><span class="sxs-lookup"><span data-stu-id="78892-281">Step 7: Provision Jenkins server with Managed Disks for disaster recovery</span></span>

<span data-ttu-id="78892-282">Microsoft Jenkins 产品组已创建灾难恢复脚本，这些脚本可以生成用于保存 Jenkins 状态的托管磁盘。</span><span class="sxs-lookup"><span data-stu-id="78892-282">The Microsoft Jenkins product group has created disaster recovery scripts that build a managed disk used to save the Jenkins state.</span></span> <span data-ttu-id="78892-283">如果服务器发生故障，可将它还原到最新状态。</span><span class="sxs-lookup"><span data-stu-id="78892-283">If the server goes down, it can be restored to its latest state.</span></span>

<span data-ttu-id="78892-284">从 [GitHub][disaster] 下载并运行灾难恢复脚本。</span><span class="sxs-lookup"><span data-stu-id="78892-284">Download and run the disaster recovery scripts from [GitHub][disaster].</span></span>

[acs]: https://aka.ms/azjenkinsacs
[ad-sp]: /azure/active-directory/develop/active-directory-integrating-applications
[app-service]: https://plugins.jenkins.io/azure-app-service
[azure-ad]: /azure/active-directory/
[azure-market]: https://azuremarketplace.microsoft.com/marketplace/apps/azure-oss.jenkins?tab=Overview
[best-practices]: https://jenkins.io/doc/book/architecting-for-scale/
[blob]: /azure/storage/common/storage-java-jenkins-continuous-integration-solution
[configure-azure-ad]: https://plugins.jenkins.io/azure-ad
[configure-agent]: https://plugins.jenkins.io/azure-vm-agents
[configure-credential]: https://plugins.jenkins.io/azure-credentials
[configure-storage]: https://plugins.jenkins.io/windows-azure-storage
[container-agents]: https://aka.ms/azcontaineragent
[create-jenkins]: /azure/jenkins/install-jenkins-solution-template
[create-metric]: /azure/monitoring-and-diagnostics/insights-alerts-portal
[disaster]: https://github.com/Azure/jenkins/tree/master/disaster_recovery
[functions]: https://aka.ms/azjenkinsfunctions
[index]: https://plugins.jenkins.io
[jenkins-best]: https://wiki.jenkins.io/display/JENKINS/Jenkins+Best+Practices
[jenkins-on-azure]: /azure/jenkins/
[key-vault]: /azure/key-vault/
[managed-disk]: /azure/virtual-machines/linux/managed-disks-overview
[matrix]: https://plugins.jenkins.io/matrix-auth
[monitor]: /azure/monitoring-and-diagnostics/
[monitoring-diag]: /azure/architecture/best-practices/monitoring
[multi-master]: https://jenkins.io/doc/book/architecting-for-scale/
[nginx]: https://www.digitalocean.com/community/tutorials/how-to-create-an-ssl-certificate-on-nginx-for-ubuntu-14-04
[nsg]: /azure/virtual-network/virtual-networks-nsg
[quick-start]: /azure/security-center/security-center-get-started
[port443]: /azure/virtual-machines/windows/nsg-quickstart-portal
[premium]: /azure/virtual-machines/linux/premium-storage
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rg]: /azure/azure-resource-manager/resource-group-overview
[scale]: https://jenkins.io/doc/book/architecting-for-scale/
[scale-agent]: /azure/jenkins/jenkins-azure-vm-agents
[selection-guide]: https://jenkins.io/doc/book/hardware-recommendations/
[service-principal]: /azure/active-directory/develop/active-directory-application-objects
[secure-jenkins]: https://jenkins.io/blog/2017/04/20/secure-jenkins-on-azure/
[security-center]: /azure/security-center/security-center-intro
[sizes-linux]: /azure/virtual-machines/linux/sizes?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json
[solution]: https://azure.microsoft.com/blog/announcing-the-solution-template-for-jenkins-on-azure/
[sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/
[storage-plugin]: https://wiki.jenkins.io/display/JENKINS/Windows+Azure+Storage+Plugin
[subnet]: /azure/virtual-network/virtual-network-manage-subnet
[vm-agent]: https://wiki.jenkins.io/display/JENKINS/Azure+VM+Agents+plugin
[vnet]: /azure/virtual-network/virtual-networks-overview
[0]: ./images/jenkins-server.png 
