---
title: "面向 AWS 专业人员的 Azure"
description: "了解 Microsoft Azure 帐户、平台和服务的基础知识。 另外，了解 AWS 与 Azure 平台之间的重要相似之处和差别。 在 Azure 中利用 AWS 方面的经验。"
keywords: "AWS 专家, Azure 比较, AWS 比较, azure 与 aws 之间的差别, azure 与 aws"
author: lbrader
ms.date: 03/24/2017
pnp.series.title: Azure for AWS Professionals
ms.openlocfilehash: 75fda82ee5ca7ca3665501fe428d1d01995e7422
ms.sourcegitcommit: c53adf50d3a787956fc4ebc951b163a10eeb5d20
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/23/2017
---
# <a name="azure-for-aws-professionals"></a><span data-ttu-id="38a72-106">面向 AWS 专业人员的 Azure</span><span class="sxs-lookup"><span data-stu-id="38a72-106">Azure for AWS Professionals</span></span>

<span data-ttu-id="38a72-107">本文可帮助 Amazon Web Services (AWS) 专家了解 Microsoft Azure 帐户、平台和服务的基础知识。</span><span class="sxs-lookup"><span data-stu-id="38a72-107">This article helps Amazon Web Services (AWS) experts understand the basics of Microsoft Azure accounts, platform, and services.</span></span> <span data-ttu-id="38a72-108">其中介绍了 AWS 与 Azure 平台之间的重要相似之处和差别。</span><span class="sxs-lookup"><span data-stu-id="38a72-108">It also covers key similarities and differences between the AWS and Azure platforms.</span></span>

<span data-ttu-id="38a72-109">学习内容：</span><span class="sxs-lookup"><span data-stu-id="38a72-109">You'll learn:</span></span>

* <span data-ttu-id="38a72-110">帐户和资源在 Azure 中的组织方式。</span><span class="sxs-lookup"><span data-stu-id="38a72-110">How accounts and resources are organized in Azure.</span></span>
* <span data-ttu-id="38a72-111">如何在 Azure 中构建可用的解决方案。</span><span class="sxs-lookup"><span data-stu-id="38a72-111">How available solutions are structured in Azure.</span></span>
* <span data-ttu-id="38a72-112">如何 Azure 服务与 AWS 服务之间有哪些重要差别。</span><span class="sxs-lookup"><span data-stu-id="38a72-112">How the major Azure services differ from AWS services.</span></span>

<span data-ttu-id="38a72-113">长久以来，Azure 和 AWS 各自建立了自身的功能，因此，两者的实施方式和设计具有重要差别。</span><span class="sxs-lookup"><span data-stu-id="38a72-113">Azure and AWS built their capabilities independently over time so that each has important implementation and design differences.</span></span>

## <a name="overview"></a><span data-ttu-id="38a72-114">概述</span><span class="sxs-lookup"><span data-stu-id="38a72-114">Overview</span></span>

<span data-ttu-id="38a72-115">像 AWS 一样，Microsoft Azure 是围绕一组核心计算、存储、数据库和网络服务构建的。</span><span class="sxs-lookup"><span data-stu-id="38a72-115">Like AWS, Microsoft Azure is built around a core set of compute, storage, database, and networking services.</span></span> <span data-ttu-id="38a72-116">在许多情况下，这两个平台在所提供的产品和服务方面具有基本的相似性。</span><span class="sxs-lookup"><span data-stu-id="38a72-116">In many cases, both platforms offer a basic equivalence between the products and services they offer.</span></span> <span data-ttu-id="38a72-117">AWS 和 Azure 都允许基于 Windows 或 Linux 主机构建高可用性解决方案。</span><span class="sxs-lookup"><span data-stu-id="38a72-117">Both AWS and Azure allow you to build highly available solutions based on Windows or Linux hosts.</span></span> <span data-ttu-id="38a72-118">因此，如果你习惯于使用 Linux 和 OSS 技术进行开发，这两个平台都可满足要求。</span><span class="sxs-lookup"><span data-stu-id="38a72-118">So, if you're used to development using Linux and OSS technology, both platforms can do the job.</span></span>

<span data-ttu-id="38a72-119">尽管这两个平台的功能类似，但提供这些功能的资源通常以不同的方式进行组织。</span><span class="sxs-lookup"><span data-stu-id="38a72-119">While the capabilities of both platforms are similar, the resources that provide those capabilities are often organized differently.</span></span> <span data-ttu-id="38a72-120">用于构建解决方案的服务之间的具体一对一关系有时并不明确。</span><span class="sxs-lookup"><span data-stu-id="38a72-120">Exact one-to-one relationships between the services required to build a solution are not always clear.</span></span> <span data-ttu-id="38a72-121">另外，在某些情况下，特定的服务可能已在其中一个平台上提供，但没有在另一个平台上提供。</span><span class="sxs-lookup"><span data-stu-id="38a72-121">There are also cases where a particular service might be offered on one platform, but not the other.</span></span> <span data-ttu-id="38a72-122">请参阅[类似的 Azure 和 AWS 服务图表](services.md)。</span><span class="sxs-lookup"><span data-stu-id="38a72-122">See [charts of comparable Azure and AWS services](services.md).</span></span>

## <a name="accounts-and-subscriptions"></a><span data-ttu-id="38a72-123">帐户和订阅</span><span class="sxs-lookup"><span data-stu-id="38a72-123">Accounts and subscriptions</span></span>

<span data-ttu-id="38a72-124">可以根据组织的规模和需求，以多个定价选项购买 Azure 服务。</span><span class="sxs-lookup"><span data-stu-id="38a72-124">Azure services can be purchased using several pricing options, depending on your organization's size and needs.</span></span> <span data-ttu-id="38a72-125">有关详细信息，请参阅[定价概述](https://azure.microsoft.com/pricing/)页。</span><span class="sxs-lookup"><span data-stu-id="38a72-125">See the [pricing overview](https://azure.microsoft.com/pricing/) page for details.</span></span>

<span data-ttu-id="38a72-126">[Azure 订阅](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-infrastructure-subscription-accounts-guidelines/)是一组资源，其中分配的所有者负责进行计费和权限管理。</span><span class="sxs-lookup"><span data-stu-id="38a72-126">[Azure subscriptions](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-infrastructure-subscription-accounts-guidelines/) are a grouping of resources with an assigned owner responsible for billing and permissions management.</span></span> <span data-ttu-id="38a72-127">订阅是独立于其所有者帐户存在的，可根据需要将其重新分配给新的所有者；而 AWS 则与此不同，其中，在 AWS 帐户下创建的所有资源会绑定到该帐户。</span><span class="sxs-lookup"><span data-stu-id="38a72-127">Unlike AWS, where any resources created under the AWS account are tied to that account, subscriptions exist independently of their owner accounts, and can be reassigned to new owners as needed.</span></span>

<span data-ttu-id="38a72-128">![AWS 帐户与 Azure 订阅的结构和所有权比较](./images/azure-aws-account-compare.png "AWS 帐户与 Azure 订阅的结构和所有权比较")
</span><span class="sxs-lookup"><span data-stu-id="38a72-128">![Comparison of structure and ownership of AWS accounts and Azure subscriptions](./images/azure-aws-account-compare.png "Comparison of structure and ownership of AWS accounts and Azure subscriptions")
</span></span><br/><span data-ttu-id="38a72-129">*AWS 帐户与 Azure 订阅的结构和所有权比较*
</span><span class="sxs-lookup"><span data-stu-id="38a72-129">*Comparison of structure and ownership of AWS accounts and Azure subscriptions*
</span></span><br/><br/>

<span data-ttu-id="38a72-130">订阅分配给三种类型的管理员帐户：</span><span class="sxs-lookup"><span data-stu-id="38a72-130">Subscriptions are assigned three types of administrator accounts:</span></span>

-   <span data-ttu-id="38a72-131">**帐户管理员** - 针对订阅中所用资源计费的订阅所有者和帐户。</span><span class="sxs-lookup"><span data-stu-id="38a72-131">**Account Administrator** - The subscription owner and the account billed for the resources used in the subscription.</span></span> <span data-ttu-id="38a72-132">只能通过转让订阅所有权来更改帐户管理员。</span><span class="sxs-lookup"><span data-stu-id="38a72-132">The account administrator can only be changed by transferring ownership of the subscription.</span></span>

-   <span data-ttu-id="38a72-133">**服务管理员** - 此帐户有权在订阅中创建和管理资源，但不负责计费。</span><span class="sxs-lookup"><span data-stu-id="38a72-133">**Service Administrator** - This account has rights to create and manage resources in the subscription, but is not responsible for billing.</span></span> <span data-ttu-id="38a72-134">默认情况下，会将帐户管理员和服务管理员分配到同一个帐户。</span><span class="sxs-lookup"><span data-stu-id="38a72-134">By default, the account administrator and service administrator are assigned to the same account.</span></span> <span data-ttu-id="38a72-135">帐户管理员可将一个单独的用户分配到服务管理员帐户，用于管理订阅的技术和操作方面。</span><span class="sxs-lookup"><span data-stu-id="38a72-135">The account administrator can assign a separate user to the service administrator account for managing the technical and operational aspects of a subscription.</span></span> <span data-ttu-id="38a72-136">每个订阅只有一个服务管理员。</span><span class="sxs-lookup"><span data-stu-id="38a72-136">There is only one service administrator per subscription.</span></span>

-   <span data-ttu-id="38a72-137">**协同管理员** - 可将多个协同管理员帐户分配到一个订阅。</span><span class="sxs-lookup"><span data-stu-id="38a72-137">**Co-administrator** - There can be multiple co-administrator accounts assigned to a subscription.</span></span> <span data-ttu-id="38a72-138">协同管理员无法更改服务管理员，但对订阅资源和用户拥有完全控制权。</span><span class="sxs-lookup"><span data-stu-id="38a72-138">Co-administrators cannot change the service administrator, but otherwise have full control over subscription resources and users.</span></span>

<span data-ttu-id="38a72-139">在订阅级别下，还可将用户角色和单个权限分配到特定的资源，类似于在 AWS 中向 IAM 用户和组授予权限。</span><span class="sxs-lookup"><span data-stu-id="38a72-139">Below the subscription level user roles and individual permissions can also be assigned to specific resources, similarly to how permissions are granted to IAM users and groups in AWS.</span></span> <span data-ttu-id="38a72-140">在 Azure 中，所有用户帐户都与 Microsoft 帐户或组织帐户（通过 Azure Active Directory 管理的帐户）相关联。</span><span class="sxs-lookup"><span data-stu-id="38a72-140">In Azure all user accounts are associated with either a Microsoft Account or Organizational Account (an account managed through an Azure Active Directory).</span></span>

<span data-ttu-id="38a72-141">与 AWS 帐户一样，订阅具有默认的服务配额和限制。</span><span class="sxs-lookup"><span data-stu-id="38a72-141">Like AWS accounts, subscriptions have default service quotas and limits.</span></span> <span data-ttu-id="38a72-142">有关这些限制的完整列表，请参阅 [Azure 订阅和服务限制、配额与约束](https://azure.microsoft.com/documentation/articles/azure-subscription-service-limits/)。</span><span class="sxs-lookup"><span data-stu-id="38a72-142">For a full list of these limits, see [Azure subscription and service limits, quotas, and constraints](https://azure.microsoft.com/documentation/articles/azure-subscription-service-limits/).</span></span>
<span data-ttu-id="38a72-143">可以通过[在管理门户中提出支持请求](https://blogs.msdn.microsoft.com/girishp/2015/09/20/increasing-core-quota-limits-in-azure/)，将这些限制提高到最大值。</span><span class="sxs-lookup"><span data-stu-id="38a72-143">These limits can be increased up to the maximum by [filing a support request in the management portal](https://blogs.msdn.microsoft.com/girishp/2015/09/20/increasing-core-quota-limits-in-azure/).</span></span>

### <a name="see-also"></a><span data-ttu-id="38a72-144">另请参阅</span><span class="sxs-lookup"><span data-stu-id="38a72-144">See also</span></span>

-   [<span data-ttu-id="38a72-145">如何添加或更改 Azure 管理员角色</span><span class="sxs-lookup"><span data-stu-id="38a72-145">How to add or change Azure administrator roles</span></span>](https://azure.microsoft.com/documentation/articles/billing-add-change-azure-subscription-administrator/)

-   [<span data-ttu-id="38a72-146">如何下载 Azure 帐单发票和每日使用数据</span><span class="sxs-lookup"><span data-stu-id="38a72-146">How to download your Azure billing invoice and daily usage data</span></span>](https://azure.microsoft.com/documentation/articles/billing-download-azure-invoice-daily-usage-date/)

## <a name="resource-management"></a><span data-ttu-id="38a72-147">资源管理</span><span class="sxs-lookup"><span data-stu-id="38a72-147">Resource management</span></span>

<span data-ttu-id="38a72-148">在 Azure 中，术语“资源”的用法与在 AWS 中一样，表示可在平台中创建或配置的任何计算实例、存储对象、网络设备或其他实体。</span><span class="sxs-lookup"><span data-stu-id="38a72-148">The term "resource" in Azure is used in the same way as in AWS, meaning any compute instance, storage object, networking device, or other entity you can create or configure within the platform.</span></span>

<span data-ttu-id="38a72-149">可使用以下两种模型之一部署和管理 Azure 资源：[Azure 资源管理器模型](/azure/azure-resource-manager/resource-group-overview)或早期的 Azure [经典部署模型](/azure/azure-resource-manager/resource-manager-deployment-model)。</span><span class="sxs-lookup"><span data-stu-id="38a72-149">Azure resources are deployed and managed using one of two models: [Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview), or the older Azure [classic deployment model](/azure/azure-resource-manager/resource-manager-deployment-model).</span></span>
<span data-ttu-id="38a72-150">任何新资源都是使用资源管理器模型创建的。</span><span class="sxs-lookup"><span data-stu-id="38a72-150">Any new resources are created using the Resource Manager model.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="38a72-151">资源组</span><span class="sxs-lookup"><span data-stu-id="38a72-151">Resource groups</span></span>

<span data-ttu-id="38a72-152">Azure 和 AWS 中都包含称作“资源组”的实体，这些实体用于组织 VM、存储和虚拟网络设备等资源。</span><span class="sxs-lookup"><span data-stu-id="38a72-152">Both Azure and AWS have entities called "resource groups" that organize resources such as VMs, storage, and virtual networking devices.</span></span> <span data-ttu-id="38a72-153">但是，不能直接将 [Azure 资源组](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/)与 AWS 资源组进行比较。</span><span class="sxs-lookup"><span data-stu-id="38a72-153">However, [Azure resource groups](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/) are not directly comparable to AWS resource groups.</span></span>

<span data-ttu-id="38a72-154">AWS 允许在多个资源组中标记某个资源，而一个 Azure 资源始终与一个资源组相关联。</span><span class="sxs-lookup"><span data-stu-id="38a72-154">While AWS allows a resource to be tagged into multiple resource groups, an Azure resource is always associated with one resource group.</span></span> <span data-ttu-id="38a72-155">在一个资源组中创建的资源可以转移到另一个组，但始终只能在一个资源组中。</span><span class="sxs-lookup"><span data-stu-id="38a72-155">A resource created in one resource group can be moved to another group, but can only be in one resource group at a time.</span></span> <span data-ttu-id="38a72-156">资源组是 Azure 资源管理器使用的基本分组方式。</span><span class="sxs-lookup"><span data-stu-id="38a72-156">Resource groups are the fundamental grouping used by Azure Resource Manager.</span></span>

<span data-ttu-id="38a72-157">也可以使用[标记](https://azure.microsoft.com/documentation/articles/resource-group-using-tags/)来组织资源。</span><span class="sxs-lookup"><span data-stu-id="38a72-157">Resources can also be organized using [tags](https://azure.microsoft.com/documentation/articles/resource-group-using-tags/).</span></span>
<span data-ttu-id="38a72-158">标记是一些键值对，用于将整个订阅中的资源分组，不管资源组的成员身份如何。</span><span class="sxs-lookup"><span data-stu-id="38a72-158">Tags are key-value pairs that allow you to group resources across your subscription irrespective of resource group membership.</span></span>

### <a name="management-interfaces"></a><span data-ttu-id="38a72-159">管理界面</span><span class="sxs-lookup"><span data-stu-id="38a72-159">Management interfaces</span></span>

<span data-ttu-id="38a72-160">Azure 提供多种方式用于管理资源：</span><span class="sxs-lookup"><span data-stu-id="38a72-160">Azure offers several ways to manage your resources:</span></span>

-   <span data-ttu-id="38a72-161">[Web 界面](https://azure.microsoft.com/documentation/articles/resource-group-portal/)。</span><span class="sxs-lookup"><span data-stu-id="38a72-161">[Web interface](https://azure.microsoft.com/documentation/articles/resource-group-portal/).</span></span>
    <span data-ttu-id="38a72-162">Azure 门户针对 Azure 资源提供类似于 AWS 仪表板的基于 Web 的完整管理界面。</span><span class="sxs-lookup"><span data-stu-id="38a72-162">Like the AWS Dashboard, the Azure portal provides a full web-based management interface for Azure resources.</span></span>

-   <span data-ttu-id="38a72-163">[REST API](https://azure.microsoft.com/documentation/articles/resource-manager-rest-api/)。</span><span class="sxs-lookup"><span data-stu-id="38a72-163">[REST API](https://azure.microsoft.com/documentation/articles/resource-manager-rest-api/).</span></span>
    <span data-ttu-id="38a72-164">使用 Azure 资源管理器 REST API 能以编程方式访问 Azure 门户中提供的大部分功能。</span><span class="sxs-lookup"><span data-stu-id="38a72-164">The Azure Resource Manager REST API provides programmatic access to most of the features available in the Azure portal.</span></span>

-   <span data-ttu-id="38a72-165">[命令行](https://azure.microsoft.com/documentation/articles/xplat-cli-azure-resource-manager/)。</span><span class="sxs-lookup"><span data-stu-id="38a72-165">[Command Line](https://azure.microsoft.com/documentation/articles/xplat-cli-azure-resource-manager/).</span></span>
    <span data-ttu-id="38a72-166">Azure CLI 2.0 工具提供一个可以创建和管理 Azure 资源的命令行接口。</span><span class="sxs-lookup"><span data-stu-id="38a72-166">The Azure CLI 2.0 tool provides a command-line interface capable of creating and managing Azure resources.</span></span> <span data-ttu-id="38a72-167">Azure CLI 适用于 [Windows、Linux 和 Mac OS](https://aka.ms/azurecli2)。</span><span class="sxs-lookup"><span data-stu-id="38a72-167">Azure CLI is available for [Windows, Linux, and Mac OS](https://aka.ms/azurecli2).</span></span>

-   <span data-ttu-id="38a72-168">[PowerShell](https://azure.microsoft.com/documentation/articles/powershell-azure-resource-manager/)。</span><span class="sxs-lookup"><span data-stu-id="38a72-168">[PowerShell](https://azure.microsoft.com/documentation/articles/powershell-azure-resource-manager/).</span></span>
    <span data-ttu-id="38a72-169">借助 PowerShell 的 Azure 模块，可以使用脚本执行自动化管理任务。</span><span class="sxs-lookup"><span data-stu-id="38a72-169">The Azure modules for PowerShell allow you to execute automated management tasks using a script.</span></span> <span data-ttu-id="38a72-170">PowerShell 适用于 [Windows、Linux 和 Mac OS](https://github.com/PowerShell/PowerShell)。</span><span class="sxs-lookup"><span data-stu-id="38a72-170">PowerShell is available for [Windows, Linux, and Mac OS](https://github.com/PowerShell/PowerShell).</span></span>

-   <span data-ttu-id="38a72-171">[模板](https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/)。</span><span class="sxs-lookup"><span data-stu-id="38a72-171">[Templates](https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/).</span></span>
    <span data-ttu-id="38a72-172">Azure 资源管理器模板提供类似于 AWS CloudFormation 服务的基于 JSON 模板的资源管理功能。</span><span class="sxs-lookup"><span data-stu-id="38a72-172">Azure Resource Manager templates provide similar JSON template-based resource management capabilities to the AWS CloudFormation service.</span></span>

<span data-ttu-id="38a72-173">在其中每个接口中，都是围绕资源组创建、部署或修改 Azure 资源。</span><span class="sxs-lookup"><span data-stu-id="38a72-173">In each of these interfaces, the resource group is central to how Azure resources get created, deployed, or modified.</span></span> <span data-ttu-id="38a72-174">这类似于在执行 CloudFormation 部署过程中，“堆栈”在分组 AWS 资源时所扮演的角色。</span><span class="sxs-lookup"><span data-stu-id="38a72-174">This is similar to the role a "stack" plays in grouping AWS resources during CloudFormation deployments.</span></span>

<span data-ttu-id="38a72-175">这些接口的语法和结构与其 AWS 等效接口不同，但提供类似的功能。</span><span class="sxs-lookup"><span data-stu-id="38a72-175">The syntax and structure of these interfaces are different from their AWS equivalents, but they provide comparable capabilities.</span></span> <span data-ttu-id="38a72-176">此外，在 AWS 中使用的许多第三方管理工具，例如 [Hashicorp 的 Terraform](https://www.terraform.io/docs/providers/azurerm/) 和 [Netflix Spinnaker](http://www.spinnaker.io/)，在 Azure 中也可用。</span><span class="sxs-lookup"><span data-stu-id="38a72-176">In addition, many third party management tools used on AWS, like [Hashicorp's Terraform](https://www.terraform.io/docs/providers/azurerm/) and [Netflix Spinnaker](http://www.spinnaker.io/), are also available on Azure.</span></span>

### <a name="see-also"></a><span data-ttu-id="38a72-177">另请参阅</span><span class="sxs-lookup"><span data-stu-id="38a72-177">See also</span></span>

-   [<span data-ttu-id="38a72-178">Azure 资源组准则</span><span class="sxs-lookup"><span data-stu-id="38a72-178">Azure resource group guidelines</span></span>](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/)

## <a name="regions-and-zones-high-availability"></a><span data-ttu-id="38a72-179">区域 (Region) 和局部区域 (Zone)（高可用性）</span><span class="sxs-lookup"><span data-stu-id="38a72-179">Regions and zones (high availability)</span></span>

<span data-ttu-id="38a72-180">在 AWS 中，可用性是围绕可用性区域的概念建立的。</span><span class="sxs-lookup"><span data-stu-id="38a72-180">In AWS, availability centers around the concept of Availability Zones.</span></span> <span data-ttu-id="38a72-181">在 Azure 中，构建高可用性解决方案时，会同时涉及到容错域和可用性集。</span><span class="sxs-lookup"><span data-stu-id="38a72-181">In Azure, fault domains and availability sets are all involved in building highly available solutions.</span></span> <span data-ttu-id="38a72-182">配对的区域提供附加的灾难恢复功能。</span><span class="sxs-lookup"><span data-stu-id="38a72-182">Paired regions provide additional disaster recovery capabilities.</span></span>

### <a name="availability-zones-azure-fault-domains-and-availability-sets"></a><span data-ttu-id="38a72-183">可用性区域、Azure 容错域和可用性集</span><span class="sxs-lookup"><span data-stu-id="38a72-183">Availability Zones, Azure fault domains, and availability sets</span></span>

<span data-ttu-id="38a72-184">在 AWS 中，一个区域划分为两个或更多个可用性区域。</span><span class="sxs-lookup"><span data-stu-id="38a72-184">In AWS, a region is divided into two or more Availability Zones.</span></span> <span data-ttu-id="38a72-185">可用性区域对应于某个地理区域中物理隔离的数据中心。</span><span class="sxs-lookup"><span data-stu-id="38a72-185">An Availability Zone corresponds with a physically isolated datacenter in the geographic region.</span></span>
<span data-ttu-id="38a72-186">如果将应用程序服务器部署到独立的可用性区域，发生影响到一个区域的硬件故障或连接中断时，不会影响到其他区域中托管的任何服务器。</span><span class="sxs-lookup"><span data-stu-id="38a72-186">If you deploy your application servers to separate Availability Zones, a hardware or connectivity outage affecting one zone does not impact any servers hosted in other zones.</span></span>

<span data-ttu-id="38a72-187">在 Azure 中，[容错域](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-manage-availability/)定义一组共享物理电源和网络交换机的 VM。</span><span class="sxs-lookup"><span data-stu-id="38a72-187">In Azure, a [fault domain](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-manage-availability/) defines a group of VMs that shares a physical power source and network switch.</span></span>
<span data-ttu-id="38a72-188">可以使用[可用性集](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-manage-availability/)将 VM 分散到多个容错域中。</span><span class="sxs-lookup"><span data-stu-id="38a72-188">You use [availability sets](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-manage-availability/) to spread VMs across multiple fault domains.</span></span> <span data-ttu-id="38a72-189">如果将实例分配到同一个可用性集，Azure 会在多个容错域之间均匀分配这些实例。</span><span class="sxs-lookup"><span data-stu-id="38a72-189">When instances are assigned to the same availability set, Azure distributes them evenly across several fault domains.</span></span> <span data-ttu-id="38a72-190">如果一个容错域发生电源故障或网络中断，另一个容错域至少包含其中的一部分 VM，因此不受中断的影响。</span><span class="sxs-lookup"><span data-stu-id="38a72-190">If a power failure or network outage occurs in one fault domain, at least some of the set's VMs are in another fault domain and unaffected by the outage.</span></span>

<span data-ttu-id="38a72-191">![AWS 可用性区域与 Azure 容错域和可用性集的比较](./images/zone-fault-domains.png "AWS 可用性区域与 Azure 容错域和可用性集的比较")
</span><span class="sxs-lookup"><span data-stu-id="38a72-191">![AWS Availability Zones comparison to Azure fault domains and availability sets](./images/zone-fault-domains.png "AWS Availability Zones compared with Azure fault domains and availability sets")
</span></span><br/><span data-ttu-id="38a72-192">*AWS 可用性区域与 Azure 容错域和可用性集的比较*
</span><span class="sxs-lookup"><span data-stu-id="38a72-192">*AWS Availability Zones compared with Azure fault domains and availability sets*
</span></span><br/><br/>

<span data-ttu-id="38a72-193">应该根据实例在应用程序中的角色来组织可用性集，确保每个角色都有一个正常运行的实例。</span><span class="sxs-lookup"><span data-stu-id="38a72-193">Availability sets should be organized by the instance's role in your application to ensure one instance in each role is operational.</span></span> <span data-ttu-id="38a72-194">例如，在标准的三层 Web 应用程序中，可以针对前端、应用程序和数据实例分别创建一个可用性集。</span><span class="sxs-lookup"><span data-stu-id="38a72-194">For example, in a standard three-tier web application, you would want to create a separate availability set for front-end, application, and data instances.</span></span>

<span data-ttu-id="38a72-195">![每个应用程序角色的 Azure 可用性集](./images/three-tier-example.png "每个应用程序角色的可用性集")
</span><span class="sxs-lookup"><span data-stu-id="38a72-195">![Azure availability sets for each application role](./images/three-tier-example.png "Availability sets for each application role")
</span></span><br/><span data-ttu-id="38a72-196">*每个应用程序角色的 Azure 可用性集*
</span><span class="sxs-lookup"><span data-stu-id="38a72-196">*Azure availability sets for each application role*
</span></span><br/><br/>

<span data-ttu-id="38a72-197">将 VM 实例添加到可用性集后，还会为这些实例分配一个[更新域](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-manage-availability/)。</span><span class="sxs-lookup"><span data-stu-id="38a72-197">When VM instances are added to availability sets, they are also assigned an [update domain](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-manage-availability/).</span></span>
<span data-ttu-id="38a72-198">更新域是针对同时执行的计划内维护事件设置的一组 VM。</span><span class="sxs-lookup"><span data-stu-id="38a72-198">An update domain is a group of VMs that are set for planned maintenance events at the same time.</span></span> <span data-ttu-id="38a72-199">在多个更新域之间分配 VM 可以确保在任意给定时间，计划内更新和修补事件只会影响其中的一部分 VM。</span><span class="sxs-lookup"><span data-stu-id="38a72-199">Distributing VMs across multiple update domains ensures that planned update and patching events affect only a subset of these VMs at any given time.</span></span>

### <a name="paired-regions"></a><span data-ttu-id="38a72-200">配对区域</span><span class="sxs-lookup"><span data-stu-id="38a72-200">Paired regions</span></span>

<span data-ttu-id="38a72-201">在 Azure 中，可以使用[配对区域](https://azure.microsoft.com/documentation/articles/best-practices-availability-paired-regions/)来支持两个预定义地理区域之间的冗余，这样，即使服务中断影响了整个 Azure 区域，也能确保解决方案保持可用。</span><span class="sxs-lookup"><span data-stu-id="38a72-201">In Azure, you use [paired regions](https://azure.microsoft.com/documentation/articles/best-practices-availability-paired-regions/) to support redundancy across two predefined geographic regions, ensuring that even if an outage affects an entire Azure region, your solution is still available.</span></span>

<span data-ttu-id="38a72-202">配对区域通常至少相隔 300 英里的距离，这与 AWS 可用性区域不同，后者虽然是物理独立的数据中心，但可能位于相对邻近的地理区域中。</span><span class="sxs-lookup"><span data-stu-id="38a72-202">Unlike AWS Availability Zones, which are physically separate datacenters but may be in relatively nearby geographic areas, paired regions are usually separated by at least 300 miles.</span></span> <span data-ttu-id="38a72-203">保持这种远距离的目的是确保大规模的灾难只会影响配对中的一个区域。</span><span class="sxs-lookup"><span data-stu-id="38a72-203">This is intended to ensure larger scale disasters only impact one of the regions in the pair.</span></span> <span data-ttu-id="38a72-204">可将邻近的配对设置为同步数据库和存储服务数据，并对其进行适当的配置，以便每次只将平台更新部署到配对中的一个区域。</span><span class="sxs-lookup"><span data-stu-id="38a72-204">Neighboring pairs can be set to sync database and storage service data, and are configured so that platform updates are rolled out to only one region in the pair at a time.</span></span>

<span data-ttu-id="38a72-205">Azure [异地冗余存储](https://azure.microsoft.com/documentation/articles/storage-redundancy/#geo-redundant-storage)会自动备份到相应的配对区域。</span><span class="sxs-lookup"><span data-stu-id="38a72-205">Azure [geo-redundant storage](https://azure.microsoft.com/documentation/articles/storage-redundancy/#geo-redundant-storage) is automatically backed up to the appropriate paired region.</span></span> <span data-ttu-id="38a72-206">对于其他所有资源而言，使用配对区域创建完全冗余的解决方案意味着需要在两个区域中创建该解决方案的完整副本。</span><span class="sxs-lookup"><span data-stu-id="38a72-206">For all other resources, creating a fully redundant solution using paired regions means creating a full copy of your solution in both regions.</span></span>

### <a name="see-also"></a><span data-ttu-id="38a72-207">另请参阅</span><span class="sxs-lookup"><span data-stu-id="38a72-207">See also</span></span>

-   [<span data-ttu-id="38a72-208">Azure 中虚拟机的区域和可用性</span><span class="sxs-lookup"><span data-stu-id="38a72-208">Regions and availability for virtual machines in Azure</span></span>](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-regions-and-availability/)

-   [<span data-ttu-id="38a72-209">Azure 应用程序的高可用性</span><span class="sxs-lookup"><span data-stu-id="38a72-209">High availability for Azure applications</span></span>](../resiliency/high-availability-azure-applications.md)

-   [<span data-ttu-id="38a72-210">Azure 应用程序的灾难恢复</span><span class="sxs-lookup"><span data-stu-id="38a72-210">Disaster recovery for Azure applications</span></span>](../resiliency/disaster-recovery-azure-applications.md)

-   [<span data-ttu-id="38a72-211">Azure 中 Linux 虚拟机的计划内维护</span><span class="sxs-lookup"><span data-stu-id="38a72-211">Planned maintenance for Linux virtual machines in Azure</span></span>](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-planned-maintenance/)

## <a name="services"></a><span data-ttu-id="38a72-212">服务</span><span class="sxs-lookup"><span data-stu-id="38a72-212">Services</span></span>

<span data-ttu-id="38a72-213">请参阅 [AWS 和 Azure 服务的完整比较对照表](https://aka.ms/azure4aws-services)，了解平台之间所有服务映射方式的完整列表。</span><span class="sxs-lookup"><span data-stu-id="38a72-213">Consult the [complete AWS and Azure service comparison matrix](https://aka.ms/azure4aws-services) for a full listing of how all services map between platforms.</span></span>

<span data-ttu-id="38a72-214">并非所有 Azure 产品和服务在所有区域中都可用。</span><span class="sxs-lookup"><span data-stu-id="38a72-214">Not all Azure products and services are available in all regions.</span></span> <span data-ttu-id="38a72-215">有关详细信息，请参阅[各区域中推出的产品](https://azure.microsoft.com/regions/services/)页。</span><span class="sxs-lookup"><span data-stu-id="38a72-215">Consult the [Products by Region](https://azure.microsoft.com/regions/services/) page for details.</span></span> <span data-ttu-id="38a72-216">可在[服务级别协议](https://azure.microsoft.com/support/legal/sla/)页上找到每个 Azure 产品或服务的运行时间保证以及停机时间信用政策。</span><span class="sxs-lookup"><span data-stu-id="38a72-216">You can find the uptime guarantees and downtime credit policies for each Azure product or service on the [Service Level Agreements](https://azure.microsoft.com/support/legal/sla/) page.</span></span>

<span data-ttu-id="38a72-217">以下部分提供 AWS 与 Azure 平台中常用功能和服务的差别的简要说明。</span><span class="sxs-lookup"><span data-stu-id="38a72-217">The following sections provide a brief explanation of how commonly used features and services differ between the AWS and Azure platforms.</span></span>

### <a name="compute-services"></a><span data-ttu-id="38a72-218">计算服务</span><span class="sxs-lookup"><span data-stu-id="38a72-218">Compute services</span></span>

#### <a name="ec2-instances-and-azure-virtual-machines"></a><span data-ttu-id="38a72-219">EC2 实例和 Azure 虚拟机</span><span class="sxs-lookup"><span data-stu-id="38a72-219">EC2 Instances and Azure virtual machines</span></span>

<span data-ttu-id="38a72-220">尽管 AWS 实例类型与 Azure 虚拟机大小的划分方式类似，但 RAM、CPU 和存储功能方面存在一些差别。</span><span class="sxs-lookup"><span data-stu-id="38a72-220">Although AWS instance types and Azure virtual machine sizes breakdown in a similar way, there are differences in the RAM, CPU, and storage capabilities.</span></span>

-   [<span data-ttu-id="38a72-221">Amazon EC2 实例类型</span><span class="sxs-lookup"><span data-stu-id="38a72-221">Amazon EC2 Instance Types</span></span>](https://aws.amazon.com/ec2/instance-types/)

-   [<span data-ttu-id="38a72-222">Azure 中虚拟机的大小 (Windows)</span><span class="sxs-lookup"><span data-stu-id="38a72-222">Sizes for virtual machines in Azure (Windows)</span></span>](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/)

-   [<span data-ttu-id="38a72-223">Azure 中虚拟机的大小 (Linux)</span><span class="sxs-lookup"><span data-stu-id="38a72-223">Sizes for virtual machines in Azure (Linux)</span></span>](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-sizes/)

<span data-ttu-id="38a72-224">与 AWS 的按秒计费不同，Azure 按需 VM 是按分钟计费的。</span><span class="sxs-lookup"><span data-stu-id="38a72-224">Unlike AWS' per second billing, Azure on-demand VMs are billed by the minute.</span></span>

<span data-ttu-id="38a72-225">Azure 不提供与 EC2 Spot 实例或专用主机相同的功能。</span><span class="sxs-lookup"><span data-stu-id="38a72-225">Azure has no equivalent to EC2 Spot Instances or Dedicated Hosts.</span></span>

#### <a name="ebs-and-azure-storage-for-vm-disks"></a><span data-ttu-id="38a72-226">EBS 与用作 VM 磁盘的 Azure 存储</span><span class="sxs-lookup"><span data-stu-id="38a72-226">EBS and Azure Storage for VM disks</span></span>

<span data-ttu-id="38a72-227">Azure VM 的持久性数据存储是通过驻留在 Blob 存储中的[数据磁盘](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-about-disks-vhds/)提供的。</span><span class="sxs-lookup"><span data-stu-id="38a72-227">Durable data storage for Azure VMs is provided by [data disks](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-about-disks-vhds/) residing in blob storage.</span></span> <span data-ttu-id="38a72-228">这类似于 EC2 实例在弹性块存储 (EBS) 中存储磁盘卷。</span><span class="sxs-lookup"><span data-stu-id="38a72-228">This is similar to how EC2 instances store disk volumes on Elastic Block Store (EBS).</span></span> <span data-ttu-id="38a72-229">与 EC2 实例存储（也称为临时性存储）一样，[Azure 临时存储](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/)也为 VM 提供低延迟的临时读写存储。</span><span class="sxs-lookup"><span data-stu-id="38a72-229">[Azure temporary storage](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/) also provides VMs the same low-latency temporary read-write storage as EC2 Instance Storage (also called ephemeral storage).</span></span>

<span data-ttu-id="38a72-230">使用 [Azure 高级存储](https://docs.microsoft.com/azure/storage/storage-premium-storage)可以支持更高性能的磁盘 IO。</span><span class="sxs-lookup"><span data-stu-id="38a72-230">Higher performance disk IO is supported using [Azure premium storage](https://docs.microsoft.com/azure/storage/storage-premium-storage).</span></span>
<span data-ttu-id="38a72-231">这类似于 AWS 提供的预配 IOPS 存储选项。</span><span class="sxs-lookup"><span data-stu-id="38a72-231">This is similar to the Provisioned IOPS storage options provided by AWS.</span></span>

#### <a name="lambda-azure-functions-azure-web-jobs-and-azure-logic-apps"></a><span data-ttu-id="38a72-232">Lambda、Azure Functions、Azure Web 作业和 Azure 逻辑应用</span><span class="sxs-lookup"><span data-stu-id="38a72-232">Lambda, Azure Functions, Azure Web-Jobs, and Azure Logic Apps</span></span>

<span data-ttu-id="38a72-233">[Azure Functions](https://azure.microsoft.com/services/functions/) 基本上相当于 AWS Lambda，提供无服务器的按需代码。</span><span class="sxs-lookup"><span data-stu-id="38a72-233">[Azure Functions](https://azure.microsoft.com/services/functions/) is the primary equivalent of AWS Lambda in providing serverless, on-demand code.</span></span>
<span data-ttu-id="38a72-234">但是，Lambda 功能还与其他 Azure 服务重叠：</span><span class="sxs-lookup"><span data-stu-id="38a72-234">However, Lambda functionality also overlaps with other Azure services:</span></span>

-   <span data-ttu-id="38a72-235">[Web 作业](https://azure.microsoft.com/documentation/articles/web-sites-create-web-jobs/) - 用于创建计划或连续运行后台任务。</span><span class="sxs-lookup"><span data-stu-id="38a72-235">[WebJobs](https://azure.microsoft.com/documentation/articles/web-sites-create-web-jobs/) - allow you to create scheduled or continuously running background tasks.</span></span>

-   <span data-ttu-id="38a72-236">[逻辑应用](https://azure.microsoft.com/services/logic-apps/) - 提供通信、集成和业务规则管理服务。</span><span class="sxs-lookup"><span data-stu-id="38a72-236">[Logic Apps](https://azure.microsoft.com/services/logic-apps/) - provides communications, integration, and business rule management services.</span></span>

#### <a name="autoscaling-azure-vm-scaling-and-azure-app-service-autoscale"></a><span data-ttu-id="38a72-237">自动缩放、Azure VM 缩放和 Azure 应用服务自动缩放</span><span class="sxs-lookup"><span data-stu-id="38a72-237">Autoscaling, Azure VM scaling, and Azure App Service Autoscale</span></span>

<span data-ttu-id="38a72-238">Azure 中的自动缩放由两个服务进行处理：</span><span class="sxs-lookup"><span data-stu-id="38a72-238">Autoscaling in Azure is handled by two services:</span></span>

-   <span data-ttu-id="38a72-239">[VM 规模集](https://azure.microsoft.com/documentation/articles/virtual-machine-scale-sets-overview/) - 用于部署和管理一组相同的 VM。</span><span class="sxs-lookup"><span data-stu-id="38a72-239">[VM scale sets](https://azure.microsoft.com/documentation/articles/virtual-machine-scale-sets-overview/) - allow you to deploy and manage an identical set of VMs.</span></span> <span data-ttu-id="38a72-240">可以根据性能需求自动缩放实例数。</span><span class="sxs-lookup"><span data-stu-id="38a72-240">The number of instances can autoscale based on performance needs.</span></span>

-   <span data-ttu-id="38a72-241">[应用服务自动缩放](https://azure.microsoft.com/documentation/articles/web-sites-scale/) - 提供自动缩放 Azure 应用服务解决方案的功能。</span><span class="sxs-lookup"><span data-stu-id="38a72-241">[App Service Autoscale](https://azure.microsoft.com/documentation/articles/web-sites-scale/) - provides the capability to autoscale Azure App Service solutions.</span></span>


#### <a name="container-service"></a><span data-ttu-id="38a72-242">容器服务</span><span class="sxs-lookup"><span data-stu-id="38a72-242">Container Service</span></span>
<span data-ttu-id="38a72-243">[Azure 容器服务](https://docs.microsoft.com/azure/container-service/container-service-intro)支持通过 Docker Swarm、Kubernetes 或 DC/OS 管理的 Docker 容器。</span><span class="sxs-lookup"><span data-stu-id="38a72-243">The [Azure Container Service](https://docs.microsoft.com/azure/container-service/container-service-intro) supports Docker containers managed through Docker Swarm, Kubernetes, or DC/OS.</span></span>

#### <a name="other-compute-services"></a><span data-ttu-id="38a72-244">其他计算服务</span><span class="sxs-lookup"><span data-stu-id="38a72-244">Other compute services</span></span> 


<span data-ttu-id="38a72-245">Azure 提供多个计算服务，在 AWS 中没有直接与此类似的服务：</span><span class="sxs-lookup"><span data-stu-id="38a72-245">Azure offers several compute services that do not have direct equivalents in AWS:</span></span>

-   <span data-ttu-id="38a72-246">[Azure Batch](https://azure.microsoft.com/documentation/articles/batch-technical-overview/) - 用于跨可缩放的虚拟机集合管理计算密集型工作。</span><span class="sxs-lookup"><span data-stu-id="38a72-246">[Azure Batch](https://azure.microsoft.com/documentation/articles/batch-technical-overview/) - allows you to manage compute-intensive work across a scalable collection of virtual machines.</span></span>

-   <span data-ttu-id="38a72-247">[Service Fabric](https://azure.microsoft.com/documentation/articles/service-fabric-overview/) - 用于开发和托管可缩放[微服务](https://azure.microsoft.com/documentation/articles/service-fabric-overview-microservices/)解决方案的平台。</span><span class="sxs-lookup"><span data-stu-id="38a72-247">[Service Fabric](https://azure.microsoft.com/documentation/articles/service-fabric-overview/) - platform for developing and hosting scalable [microservice](https://azure.microsoft.com/documentation/articles/service-fabric-overview-microservices/) solutions.</span></span>

#### <a name="see-also"></a><span data-ttu-id="38a72-248">另请参阅</span><span class="sxs-lookup"><span data-stu-id="38a72-248">See also</span></span>

-   [<span data-ttu-id="38a72-249">使用门户在 Azure 上创建 Linux VM</span><span class="sxs-lookup"><span data-stu-id="38a72-249">Create a Linux VM on Azure using the Portal</span></span>](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-quick-create-portal/)

-   [<span data-ttu-id="38a72-250">Azure 参考体系结构：在 Azure 上运行 Linux VM</span><span class="sxs-lookup"><span data-stu-id="38a72-250">Azure Reference Architecture: Running a Linux VM on Azure</span></span>](https://azure.microsoft.com/documentation/articles/guidance-compute-single-vm-linux/)

-   [<span data-ttu-id="38a72-251">Azure 应用服务中的 Node.js Web 应用入门</span><span class="sxs-lookup"><span data-stu-id="38a72-251">Get started with Node.js web apps in Azure App Service</span></span>](https://azure.microsoft.com/documentation/articles/app-service-web-nodejs-get-started/)

-   [<span data-ttu-id="38a72-252">Azure 参考体系结构：基本 Web 应用程序</span><span class="sxs-lookup"><span data-stu-id="38a72-252">Azure Reference Architecture: Basic web application</span></span>](https://azure.microsoft.com/documentation/articles/guidance-web-apps-basic/)

-   [<span data-ttu-id="38a72-253">创建第一个 Azure 函数</span><span class="sxs-lookup"><span data-stu-id="38a72-253">Create your first Azure Function</span></span>](https://azure.microsoft.com/documentation/articles/functions-create-first-azure-function/)

### <a name="storage"></a><span data-ttu-id="38a72-254">存储</span><span class="sxs-lookup"><span data-stu-id="38a72-254">Storage</span></span>

#### <a name="s3ebsefs-and-azure-storage"></a><span data-ttu-id="38a72-255">S3/EBS/EFS 与 Azure 存储</span><span class="sxs-lookup"><span data-stu-id="38a72-255">S3/EBS/EFS and Azure Storage</span></span>

<span data-ttu-id="38a72-256">在 AWS 平台中，云存储主要划分为以下三个服务：</span><span class="sxs-lookup"><span data-stu-id="38a72-256">In the AWS platform, cloud storage is primarily broken down into three services:</span></span>

-   <span data-ttu-id="38a72-257">**简单存储服务 (S3)** - 基本对象存储。</span><span class="sxs-lookup"><span data-stu-id="38a72-257">**Simple Storage Service (S3)** - basic object storage.</span></span> <span data-ttu-id="38a72-258">借助该服务能够通过可在 Internet 上访问的 API 使用数据。</span><span class="sxs-lookup"><span data-stu-id="38a72-258">Makes data available through an Internet accessible API.</span></span>

-   <span data-ttu-id="38a72-259">**弹性块存储 (EBS)** - 块级存储，旨在由单个 VM 访问。</span><span class="sxs-lookup"><span data-stu-id="38a72-259">**Elastic Block Storage (EBS)** - block level storage, intended for access by a single VM.</span></span>

-   <span data-ttu-id="38a72-260">**弹性文件系统 (EFS)** - 供多达数千个 EC2 实例用作共享存储的文件存储。</span><span class="sxs-lookup"><span data-stu-id="38a72-260">**Elastic File System (EFS)** - file storage meant for use as shared storage for up to thousands of EC2 instances.</span></span>

<span data-ttu-id="38a72-261">在 Azure 存储中，使用已绑定到订阅的[存储帐户](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/)可以创建和管理以下存储服务：</span><span class="sxs-lookup"><span data-stu-id="38a72-261">In Azure Storage, subscription-bound [storage accounts](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/) allow you to create and manage the following storage services:</span></span>

-   <span data-ttu-id="38a72-262">[Blob 存储](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/) - 存储任何类型的文本或二进制数据，例如文档、媒体文件或应用程序安装程序。</span><span class="sxs-lookup"><span data-stu-id="38a72-262">[Blob storage](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/) - stores any type of text or binary data, such as a document, media file, or application installer.</span></span> <span data-ttu-id="38a72-263">可以设置 Blob 存储进行私人访问，或者在 Internet 上公开共享内容。</span><span class="sxs-lookup"><span data-stu-id="38a72-263">You can set Blob storage for private access or share contents publicly to the Internet.</span></span> <span data-ttu-id="38a72-264">Blob 存储的用途与 AWS S3 和 EBS 相同。</span><span class="sxs-lookup"><span data-stu-id="38a72-264">Blob storage serves the same purpose as both AWS S3 and EBS.</span></span>
-   <span data-ttu-id="38a72-265">[表存储](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/) - 存储结构化数据集。</span><span class="sxs-lookup"><span data-stu-id="38a72-265">[Table storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/) - stores structured datasets.</span></span> <span data-ttu-id="38a72-266">表存储是一个 NoSQL“键-属性”数据存储，用于实现快速开发以及快速访问大量数据。</span><span class="sxs-lookup"><span data-stu-id="38a72-266">Table storage is a NoSQL key-attribute data store that allows for rapid development and fast access to large quantities of data.</span></span> <span data-ttu-id="38a72-267">类似于 AWS 的 SimpleDB 和 DynamoDB 服务。</span><span class="sxs-lookup"><span data-stu-id="38a72-267">Similar to AWS' SimpleDB and DynamoDB services.</span></span>

-   <span data-ttu-id="38a72-268">[队列存储](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) - 为工作流处理和云服务组件之间的通信提供消息传送。</span><span class="sxs-lookup"><span data-stu-id="38a72-268">[Queue storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) - provides messaging for workflow processing and for communication between components of cloud services.</span></span>

-   <span data-ttu-id="38a72-269">[文件存储](https://azure.microsoft.com/documentation/articles/storage-java-how-to-use-file-storage/) - 使用标准服务器消息块 (SMB) 协议为传统应用程序提供共享存储。</span><span class="sxs-lookup"><span data-stu-id="38a72-269">[File storage](https://azure.microsoft.com/documentation/articles/storage-java-how-to-use-file-storage/) - offers shared storage for legacy applications using the standard server message block (SMB) protocol.</span></span> <span data-ttu-id="38a72-270">文件存储的使用方式与 AWS 平台中的 EFS 类似。</span><span class="sxs-lookup"><span data-stu-id="38a72-270">File storage is used in a similar manner to EFS in the AWS platform.</span></span>




 
#### <a name="glacier-and-azure-storage"></a><span data-ttu-id="38a72-271">Glacier 与 Azure 存储</span><span class="sxs-lookup"><span data-stu-id="38a72-271">Glacier and Azure Storage</span></span> 
<span data-ttu-id="38a72-272">[Azure 存储标准存档](/azure/storage/blobs/storage-blob-storage-tiers)提供与 AWS 的长期存档 Glacier 存储直接相同的功能。</span><span class="sxs-lookup"><span data-stu-id="38a72-272">[Azure Storage Standard Archive](/azure/storage/blobs/storage-blob-storage-tiers) offers a direct equivalent to AWS' long-term archival Glacier storage.</span></span> <span data-ttu-id="38a72-273">针对不经常访问且长期留存的数据，Azure 提供 [Azure 冷 Blob 存储层](/azure/storage/blobs/storage-blob-storage-tiers)。</span><span class="sxs-lookup"><span data-stu-id="38a72-273">For data that is infrequently accessed and long-lived Azure offers the [Azure cool blob storage tier](/azure/storage/blobs/storage-blob-storage-tiers).</span></span>
<span data-ttu-id="38a72-274">与标准 Blob 存储相比，冷存储更经济节省但性能更低，类似于 AWS 的 S3 - Infrequent Access。</span><span class="sxs-lookup"><span data-stu-id="38a72-274">Cool storage provides cheaper, lower performance storage than standard blob storage and is comparable to AWS' S3 - Infrequent Access.</span></span>

#### <a name="see-also"></a><span data-ttu-id="38a72-275">另请参阅</span><span class="sxs-lookup"><span data-stu-id="38a72-275">See also</span></span>

-   [<span data-ttu-id="38a72-276">Microsoft Azure 存储性能和可伸缩性清单</span><span class="sxs-lookup"><span data-stu-id="38a72-276">Microsoft Azure Storage Performance and Scalability Checklist</span></span>](https://azure.microsoft.com/documentation/articles/storage-performance-checklist/)

-   [<span data-ttu-id="38a72-277">Azure 存储安全指南</span><span class="sxs-lookup"><span data-stu-id="38a72-277">Azure Storage security guide</span></span>](https://azure.microsoft.com/documentation/articles/storage-security-guide/)

-   [<span data-ttu-id="38a72-278">模式和做法：内容交付网络 (CDN) 指南</span><span class="sxs-lookup"><span data-stu-id="38a72-278">Patterns & Practices: Content Delivery Network (CDN) guidance</span></span>](https://azure.microsoft.com/documentation/articles/best-practices-cdn/)

### <a name="networking"></a><span data-ttu-id="38a72-279">联网</span><span class="sxs-lookup"><span data-stu-id="38a72-279">Networking</span></span>

#### <a name="elastic-load-balancing-azure-load-balancer-and-azure-application-gateway"></a><span data-ttu-id="38a72-280">弹性负载均衡、Azure 负载均衡器和 Azure 应用程序网关</span><span class="sxs-lookup"><span data-stu-id="38a72-280">Elastic Load Balancing, Azure Load Balancer, and Azure Application Gateway</span></span>

<span data-ttu-id="38a72-281">Azure 提供两个类似于弹性负载均衡的服务：</span><span class="sxs-lookup"><span data-stu-id="38a72-281">The Azure equivalents of the two Elastic Load Balancing services are:</span></span>

-   <span data-ttu-id="38a72-282">[负载均衡器](https://azure.microsoft.com/documentation/articles/load-balancer-overview/) - 提供与 AWS 经典负载均衡器相同的功能，可用于在网络级别分配多个 VM 的流量。</span><span class="sxs-lookup"><span data-stu-id="38a72-282">[Load Balancer](https://azure.microsoft.com/documentation/articles/load-balancer-overview/) - provides the same capabilities as the AWS Classic Load Balancer, allowing you to distribute traffic for multiple VMs at the network level.</span></span> <span data-ttu-id="38a72-283">此外，它还提供故障转移功能。</span><span class="sxs-lookup"><span data-stu-id="38a72-283">It also provides failover capability.</span></span>

-   <span data-ttu-id="38a72-284">[应用程序网关](https://azure.microsoft.com/documentation/articles/application-gateway-introduction/) - 提供应用程序级别的基于规则的路由，类似于 AWS 应用程序负载均衡器。</span><span class="sxs-lookup"><span data-stu-id="38a72-284">[Application Gateway](https://azure.microsoft.com/documentation/articles/application-gateway-introduction/) - offers application-level rule-based routing comparable to the AWS Application Load Balancer.</span></span>

#### <a name="route-53-azure-dns-and-azure-traffic-manager"></a><span data-ttu-id="38a72-285">Route 53、Azure DNS 和 Azure 流量管理器</span><span class="sxs-lookup"><span data-stu-id="38a72-285">Route 53, Azure DNS, and Azure Traffic Manager</span></span>

<span data-ttu-id="38a72-286">在 AWS 中，Route 53 提供 DNS 名称管理，以及 DNS 级别的流量路由和故障转移服务。</span><span class="sxs-lookup"><span data-stu-id="38a72-286">In AWS, Route 53 provides both DNS name management and DNS-level traffic routing and failover services.</span></span> <span data-ttu-id="38a72-287">在 Azure 中，这些功能通过两个服务进行处理：</span><span class="sxs-lookup"><span data-stu-id="38a72-287">In Azure this is handled through two services:</span></span>

-   <span data-ttu-id="38a72-288">[Azure DNS](https://azure.microsoft.com/documentation/services/dns/) - 提供域和 DNS 管理。</span><span class="sxs-lookup"><span data-stu-id="38a72-288">[Azure DNS](https://azure.microsoft.com/documentation/services/dns/) - provides domain and DNS management.</span></span>

-   <span data-ttu-id="38a72-289">[流量管理器](https://azure.microsoft.com/documentation/articles/traffic-manager-overview/) - 提供 DNS 级流量路由、负载均衡和故障转移功能。</span><span class="sxs-lookup"><span data-stu-id="38a72-289">[Traffic Manager](https://azure.microsoft.com/documentation/articles/traffic-manager-overview/) - provides DNS level traffic routing, load balancing, and failover capabilities.</span></span>

#### <a name="direct-connect-and-azure-expressroute"></a><span data-ttu-id="38a72-290">直接连接和 Azure ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="38a72-290">Direct Connect and Azure ExpressRoute</span></span>

<span data-ttu-id="38a72-291">Azure 通过其 [ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/) 服务提供类似的站点到站点专用连接。</span><span class="sxs-lookup"><span data-stu-id="38a72-291">Azure provides similar site-to-site dedicated connections through its [ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/) service.</span></span> <span data-ttu-id="38a72-292">通过 ExpressRoute 可以使用专用网络连接将本地网络直接连接到 Azure 资源。</span><span class="sxs-lookup"><span data-stu-id="38a72-292">ExpressRoute allows you to connect your local network directly to Azure resources using a dedicated private network connection.</span></span> <span data-ttu-id="38a72-293">Azure 还以更低的价格提供更方便的[站点到站点 VPN 连接](https://azure.microsoft.com/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/)。</span><span class="sxs-lookup"><span data-stu-id="38a72-293">Azure also offers more conventional [site-to-site VPN connections](https://azure.microsoft.com/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/) at a lower cost.</span></span>

#### <a name="see-also"></a><span data-ttu-id="38a72-294">另请参阅</span><span class="sxs-lookup"><span data-stu-id="38a72-294">See also</span></span>

-   [<span data-ttu-id="38a72-295">使用 Azure 门户创建虚拟网络</span><span class="sxs-lookup"><span data-stu-id="38a72-295">Create a virtual network using the Azure portal</span></span>](https://azure.microsoft.com/documentation/articles/virtual-networks-create-vnet-arm-pportal/)

-   [<span data-ttu-id="38a72-296">规划和设计 Azure 虚拟网络</span><span class="sxs-lookup"><span data-stu-id="38a72-296">Plan and design Azure Virtual Networks</span></span>](https://azure.microsoft.com/documentation/articles/virtual-network-vnet-plan-design-arm/)

-   [<span data-ttu-id="38a72-297">Azure 网络安全最佳做法</span><span class="sxs-lookup"><span data-stu-id="38a72-297">Azure Network Security Best Practices</span></span>](https://azure.microsoft.com/documentation/articles/azure-security-network-security-best-practices/)

### <a name="database-services"></a><span data-ttu-id="38a72-298">数据库服务</span><span class="sxs-lookup"><span data-stu-id="38a72-298">Database services</span></span>

#### <a name="rds-and-azure-relational-database-services"></a><span data-ttu-id="38a72-299">RDS 和 Azure 关系数据库服务</span><span class="sxs-lookup"><span data-stu-id="38a72-299">RDS and Azure relational database services</span></span>

<span data-ttu-id="38a72-300">Azure 提供多个与 AWS 的关系数据库服务 (RDS) 功能相同的不同关系数据库服务。</span><span class="sxs-lookup"><span data-stu-id="38a72-300">Azure provides several different relational database services that are the equivalent of AWS' Relational Database Service (RDS).</span></span>

-   [<span data-ttu-id="38a72-301">SQL 数据库</span><span class="sxs-lookup"><span data-stu-id="38a72-301">SQL Database</span></span>](https://docs.microsoft.com/azure/sql-database/sql-database-technical-overview)
-   [<span data-ttu-id="38a72-302">Azure Database for MySQL</span><span class="sxs-lookup"><span data-stu-id="38a72-302">Azure Database for MySQL</span></span>](https://docs.microsoft.com/azure/mysql/overview)
-   [<span data-ttu-id="38a72-303">Azure Database for PostgreSQL</span><span class="sxs-lookup"><span data-stu-id="38a72-303">Azure Database for PostgreSQL</span></span>](https://docs.microsoft.com/azure/postgresql/overview)

<span data-ttu-id="38a72-304">可以使用 Azure VM 实例部署其他数据库引擎，例如 [SQL Server](https://azure.microsoft.com/services/virtual-machines/sql-server/)、[Oracle](https://azure.microsoft.com/campaigns/oracle/) 和 [MySQL](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-classic-mysql-2008r2/)。</span><span class="sxs-lookup"><span data-stu-id="38a72-304">Other database engines such as [SQL Server](https://azure.microsoft.com/services/virtual-machines/sql-server/), [Oracle](https://azure.microsoft.com/campaigns/oracle/), and [MySQL](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-classic-mysql-2008r2/) can be deployed using Azure VM Instances.</span></span>

<span data-ttu-id="38a72-305">AWS RDS 的费用根据实例使用的硬件资源确定，例如 CPU、RAM、存储和网络带宽。</span><span class="sxs-lookup"><span data-stu-id="38a72-305">Costs for AWS RDS are determined by the amount of hardware resources that your instance uses, like CPU, RAM, storage, and network bandwidth.</span></span> <span data-ttu-id="38a72-306">在 Azure 数据库服务中，费用取决于数据库大小、并发连接数和吞吐量级别。</span><span class="sxs-lookup"><span data-stu-id="38a72-306">In the Azure database services, cost depends on your database size, concurrent connections, and throughput levels.</span></span>

#### <a name="see-also"></a><span data-ttu-id="38a72-307">另请参阅</span><span class="sxs-lookup"><span data-stu-id="38a72-307">See also</span></span>

-   [<span data-ttu-id="38a72-308">Azure SQL 数据库教程</span><span class="sxs-lookup"><span data-stu-id="38a72-308">Azure SQL Database Tutorials</span></span>](https://azure.microsoft.com/documentation/articles/sql-database-explore-tutorials/)

-   [<span data-ttu-id="38a72-309">使用 Azure 门户为 Azure SQL 数据库配置异地复制</span><span class="sxs-lookup"><span data-stu-id="38a72-309">Configure geo-replication for Azure SQL Database with the Azure portal</span></span>](https://azure.microsoft.com/documentation/articles/sql-database-geo-replication-portal/)

-   [<span data-ttu-id="38a72-310">DocumentDB 简介：一种 NoSQL JSON 数据库</span><span class="sxs-lookup"><span data-stu-id="38a72-310">Introduction to Cosmos DB: A NoSQL JSON Database</span></span>](https://azure.microsoft.com/documentation/articles/documentdb-introduction/)

-   [<span data-ttu-id="38a72-311">如何通过 Node.js 使用 Azure 表存储</span><span class="sxs-lookup"><span data-stu-id="38a72-311">How to use Azure Table storage from Node.js</span></span>](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/)

### <a name="security-and-identity"></a><span data-ttu-id="38a72-312">安全和标识</span><span class="sxs-lookup"><span data-stu-id="38a72-312">Security and identity</span></span>

#### <a name="directory-service-and-azure-active-directory"></a><span data-ttu-id="38a72-313">目录服务和 Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="38a72-313">Directory service and Azure Active Directory</span></span>

<span data-ttu-id="38a72-314">Azure 将目录服务划分为以下服务：</span><span class="sxs-lookup"><span data-stu-id="38a72-314">Azure splits up directory services into the following offerings:</span></span>

-   <span data-ttu-id="38a72-315">[Azure Active Directory](https://azure.microsoft.com/documentation/articles/active-directory-whatis/) - 基于云的目录和标识管理服务。</span><span class="sxs-lookup"><span data-stu-id="38a72-315">[Azure Active Directory](https://azure.microsoft.com/documentation/articles/active-directory-whatis/) - cloud based directory and identity management service.</span></span>

-   <span data-ttu-id="38a72-316">[Azure Active Directory B2B](https://azure.microsoft.com/documentation/articles/active-directory-b2b-collaboration-overview/) - 用于从合作伙伴管理的标识访问你的企业应用程序。</span><span class="sxs-lookup"><span data-stu-id="38a72-316">[Azure Active Directory B2B](https://azure.microsoft.com/documentation/articles/active-directory-b2b-collaboration-overview/) - enables access to your corporate applications from partner-managed identities.</span></span>

-   <span data-ttu-id="38a72-317">[Azure Active Directory B2C](https://azure.microsoft.com/documentation/articles/active-directory-b2c-overview/) - 针对消费型应用程序提供单一登录和用户管理支持的服务。</span><span class="sxs-lookup"><span data-stu-id="38a72-317">[Azure Active Directory B2C](https://azure.microsoft.com/documentation/articles/active-directory-b2c-overview/) - service offering support for single sign-on and user management for consumer facing applications.</span></span>

-   <span data-ttu-id="38a72-318">[Azure Active Directory 域服务](https://azure.microsoft.com/documentation/articles/active-directory-ds-overview/) - 托管的域控制器服务，可实现与 Active Directory 兼容的域加入和用户管理功能。</span><span class="sxs-lookup"><span data-stu-id="38a72-318">[Azure Active Directory Domain Services](https://azure.microsoft.com/documentation/articles/active-directory-ds-overview/) - hosted domain controller service, allowing Active Directory compatible domain join and user management functionality.</span></span>

#### <a name="web-application-firewall"></a><span data-ttu-id="38a72-319">Web 应用程序防火墙</span><span class="sxs-lookup"><span data-stu-id="38a72-319">Web application firewall</span></span>

<span data-ttu-id="38a72-320">除了[应用程序网关 Web 应用程序防火墙](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/)以外，还可以使用来自 [Barracuda Networks](https://azure.microsoft.com/marketplace/partners/barracudanetworks/waf/) 等第三方供应商的 [Web 应用程序防火墙](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/)。</span><span class="sxs-lookup"><span data-stu-id="38a72-320">In addition to the [Application Gateway Web Application Firewall](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/), you can also [use web application firewalls](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/) from third-party vendors like [Barracuda Networks](https://azure.microsoft.com/marketplace/partners/barracudanetworks/waf/).</span></span>

#### <a name="see-also"></a><span data-ttu-id="38a72-321">另请参阅</span><span class="sxs-lookup"><span data-stu-id="38a72-321">See also</span></span>

-   [<span data-ttu-id="38a72-322">Microsoft Azure 安全入门</span><span class="sxs-lookup"><span data-stu-id="38a72-322">Getting started with Microsoft Azure security</span></span>](https://azure.microsoft.com/documentation/articles/azure-security-getting-started/)

-   [<span data-ttu-id="38a72-323">Azure 标识管理和访问控制安全最佳实践</span><span class="sxs-lookup"><span data-stu-id="38a72-323">Azure Identity Management and access control security best practices</span></span>](https://azure.microsoft.com/documentation/articles/azure-security-identity-management-best-practices/)

### <a name="application-and-messaging-services"></a><span data-ttu-id="38a72-324">应用程序和消息传送服务</span><span class="sxs-lookup"><span data-stu-id="38a72-324">Application and messaging services</span></span>

#### <a name="simple-email-service"></a><span data-ttu-id="38a72-325">简单电子邮件服务</span><span class="sxs-lookup"><span data-stu-id="38a72-325">Simple Email Service</span></span>

<span data-ttu-id="38a72-326">AWS 提供简单电子邮件服务 (SES) 用于发送通知、交易或市场营销电子邮件。</span><span class="sxs-lookup"><span data-stu-id="38a72-326">AWS provides the Simple Email Service (SES) for sending notification, transactional, or marketing emails.</span></span> <span data-ttu-id="38a72-327">在 Azure 中，[Sendgrid](https://sendgrid.com/partners/azure/) 等第三方解决方案可提供电子邮件服务。</span><span class="sxs-lookup"><span data-stu-id="38a72-327">In Azure, third-party solutions like [Sendgrid](https://sendgrid.com/partners/azure/) provide email services.</span></span>

#### <a name="simple-queueing-service"></a><span data-ttu-id="38a72-328">简单队列服务</span><span class="sxs-lookup"><span data-stu-id="38a72-328">Simple Queueing Service</span></span>

<span data-ttu-id="38a72-329">AWS 简单队列服务 (SQS) 提供一个消息传送系统用于连接 AWS 平台中的应用程序、服务和设备。</span><span class="sxs-lookup"><span data-stu-id="38a72-329">AWS Simple Queueing Service (SQS) provides a messaging system for connecting applications, services, and devices within the AWS platform.</span></span> <span data-ttu-id="38a72-330">在 Azure 中，有两个服务提供类似的功能：</span><span class="sxs-lookup"><span data-stu-id="38a72-330">Azure has two services that provide similar functionality:</span></span>

-   <span data-ttu-id="38a72-331">[队列存储](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) - 一个云消息传送服务，可在 Azure 平台中的应用程序组件之间实现通信。</span><span class="sxs-lookup"><span data-stu-id="38a72-331">[Queue storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) - a cloud messaging service that allows communication between application components within the Azure platform.</span></span>

-   <span data-ttu-id="38a72-332">[服务总线](https://azure.microsoft.com/en-us/services/service-bus/) - 一个更可靠的消息传送系统，用于连接应用程序、服务和设备。</span><span class="sxs-lookup"><span data-stu-id="38a72-332">[Service Bus](https://azure.microsoft.com/en-us/services/service-bus/) - a more robust messaging system for connecting applications, services, and devices.</span></span> <span data-ttu-id="38a72-333">使用相关的[服务总线中继](https://docs.microsoft.com/en-us/azure/service-bus-relay/relay-what-is-it)，服务总线还可以连接到远程托管的应用程序和服务。</span><span class="sxs-lookup"><span data-stu-id="38a72-333">Using the related [Service Bus relay](https://docs.microsoft.com/en-us/azure/service-bus-relay/relay-what-is-it), Service Bus can also connect to remotely hosted applications and services.</span></span>

#### <a name="device-farm"></a><span data-ttu-id="38a72-334">设备场</span><span class="sxs-lookup"><span data-stu-id="38a72-334">Device Farm</span></span>

<span data-ttu-id="38a72-335">AWS 设备场提供跨设备测试服务。</span><span class="sxs-lookup"><span data-stu-id="38a72-335">The AWS Device Farm provides cross-device testing services.</span></span> <span data-ttu-id="38a72-336">在 Azure 中，[Xamarin Test Cloud](https://www.xamarin.com/test-cloud) 针对移动设备提供类似的跨设备前端测试。</span><span class="sxs-lookup"><span data-stu-id="38a72-336">In Azure, [Xamarin Test Cloud](https://www.xamarin.com/test-cloud) provides similar cross-device front-end testing for mobile devices.</span></span>

<span data-ttu-id="38a72-337">除了前端测试以外，[Azure 开发测试实验室](https://azure.microsoft.com/services/devtest-lab/)还为 Linux 和 Windows 环境提供后端测试资源。</span><span class="sxs-lookup"><span data-stu-id="38a72-337">In addition to front-end testing, the [Azure DevTest Labs](https://azure.microsoft.com/services/devtest-lab/) provides back end testing resources for Linux and Windows environments.</span></span>

#### <a name="see-also"></a><span data-ttu-id="38a72-338">另请参阅</span><span class="sxs-lookup"><span data-stu-id="38a72-338">See also</span></span>

-   [<span data-ttu-id="38a72-339">如何通过 Node.js 使用队列存储</span><span class="sxs-lookup"><span data-stu-id="38a72-339">How to use Queue storage from Node.js</span></span>](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/)

-   [<span data-ttu-id="38a72-340">如何使用 Service Bus 队列</span><span class="sxs-lookup"><span data-stu-id="38a72-340">How to use Service Bus queues</span></span>](https://azure.microsoft.com/documentation/articles/service-bus-nodejs-how-to-use-queues/)

### <a name="analytics-and-big-data"></a><span data-ttu-id="38a72-341">分析和大数据</span><span class="sxs-lookup"><span data-stu-id="38a72-341">Analytics and big data</span></span>

<span data-ttu-id="38a72-342">[Cortana Intelligence Suite](https://azure.microsoft.com/suites/cortana-intelligence-suite/) 是 Azure 提供的产品和服务包，旨在捕获、组织、分析和可视化大量数据。</span><span class="sxs-lookup"><span data-stu-id="38a72-342">[The Cortana Intelligence Suite](https://azure.microsoft.com/suites/cortana-intelligence-suite/) is Azure's package of products and services designed to capture, organize, analyze, and visualize large amounts of data.</span></span> <span data-ttu-id="38a72-343">Cortana 套件包括以下服务：</span><span class="sxs-lookup"><span data-stu-id="38a72-343">The Cortana suite consists of the following services:</span></span>

-   <span data-ttu-id="38a72-344">[HDInsight](https://azure.microsoft.com/documentation/services/hdinsight/) - 包含 Hadoop、Spark、Storm 或 HBase 的托管 Apache 分发版。</span><span class="sxs-lookup"><span data-stu-id="38a72-344">[HDInsight](https://azure.microsoft.com/documentation/services/hdinsight/) - managed Apache distribution that includes Hadoop, Spark, Storm, or HBase.</span></span>

-   <span data-ttu-id="38a72-345">[数据工厂](https://azure.microsoft.com/documentation/services/data-factory/) - 提供数据业务流程和数据管道功能。</span><span class="sxs-lookup"><span data-stu-id="38a72-345">[Data Factory](https://azure.microsoft.com/documentation/services/data-factory/) - provides data orchestration and data pipeline functionality.</span></span>

-   <span data-ttu-id="38a72-346">[SQL 数据仓库](https://azure.microsoft.com/documentation/services/sql-data-warehouse/) - 大规模关系型数据存储。</span><span class="sxs-lookup"><span data-stu-id="38a72-346">[SQL Data Warehouse](https://azure.microsoft.com/documentation/services/sql-data-warehouse/) - large-scale relational data storage.</span></span>

-   <span data-ttu-id="38a72-347">[Data Lake Store](https://azure.microsoft.com/documentation/services/data-lake-store/) - 针对大数据分析工作负荷优化的大规模存储。</span><span class="sxs-lookup"><span data-stu-id="38a72-347">[Data Lake Store](https://azure.microsoft.com/documentation/services/data-lake-store/) - large-scale storage optimized for big data analytics workloads.</span></span>

-   <span data-ttu-id="38a72-348">[机器学习](https://azure.microsoft.com/documentation/services/machine-learning/) - 用于基于数据生成和应用预测分析。</span><span class="sxs-lookup"><span data-stu-id="38a72-348">[Machine Learning](https://azure.microsoft.com/documentation/services/machine-learning/) - used to build and apply predictive analytics on data.</span></span>

-   <span data-ttu-id="38a72-349">[流分析](https://azure.microsoft.com/documentation/services/stream-analytics/) - 实时数据分析。</span><span class="sxs-lookup"><span data-stu-id="38a72-349">[Stream Analytics](https://azure.microsoft.com/documentation/services/stream-analytics/) - real-time data analysis.</span></span>

-   <span data-ttu-id="38a72-350">[Data Lake Analytics](https://azure.microsoft.com/documentation/articles/data-lake-analytics-overview/) - 经过优化的、可与 Data Lake Store 配合使用的大规模分析服务</span><span class="sxs-lookup"><span data-stu-id="38a72-350">[Data Lake Analytics](https://azure.microsoft.com/documentation/articles/data-lake-analytics-overview/) - large-scale analytics service optimized to work with Data Lake Store</span></span>

-   <span data-ttu-id="38a72-351">[PowerBI](https://powerbi.microsoft.com/) - 用于 Power 数据可视化。</span><span class="sxs-lookup"><span data-stu-id="38a72-351">[PowerBI](https://powerbi.microsoft.com/) - used to power data visualization.</span></span>

#### <a name="see-also"></a><span data-ttu-id="38a72-352">另请参阅</span><span class="sxs-lookup"><span data-stu-id="38a72-352">See also</span></span>

-   [<span data-ttu-id="38a72-353">Cortana Intelligence 库</span><span class="sxs-lookup"><span data-stu-id="38a72-353">Cortana Intelligence Gallery</span></span>](https://gallery.cortanaintelligence.com/)

-   [<span data-ttu-id="38a72-354">了解 Microsoft 大数据解决方案</span><span class="sxs-lookup"><span data-stu-id="38a72-354">Understanding Microsoft big data solutions</span></span>](https://msdn.microsoft.com/library/dn749804.aspx)

-   [<span data-ttu-id="38a72-355">Azure Data Lake 和 Azure HDInsight 博客</span><span class="sxs-lookup"><span data-stu-id="38a72-355">Azure Data Lake & Azure HDInsight Blog</span></span>](https://blogs.msdn.microsoft.com/azuredatalake/)

### <a name="internet-of-things"></a><span data-ttu-id="38a72-356">物联网</span><span class="sxs-lookup"><span data-stu-id="38a72-356">Internet of Things</span></span>

#### <a name="see-also"></a><span data-ttu-id="38a72-357">另请参阅</span><span class="sxs-lookup"><span data-stu-id="38a72-357">See also</span></span>

-   [<span data-ttu-id="38a72-358">Azure IoT 中心入门</span><span class="sxs-lookup"><span data-stu-id="38a72-358">Get started with Azure IoT Hub</span></span>](https://azure.microsoft.com/documentation/articles/iot-hub-csharp-csharp-getstarted/)

-   [<span data-ttu-id="38a72-359">IoT 中心与事件中心的比较</span><span class="sxs-lookup"><span data-stu-id="38a72-359">Comparison of IoT Hub and Event Hubs</span></span>](https://azure.microsoft.com/documentation/articles/iot-hub-compare-event-hubs/)

### <a name="mobile-services"></a><span data-ttu-id="38a72-360">移动服务</span><span class="sxs-lookup"><span data-stu-id="38a72-360">Mobile services</span></span>

#### <a name="notifications"></a><span data-ttu-id="38a72-361">通知</span><span class="sxs-lookup"><span data-stu-id="38a72-361">Notifications</span></span>

<span data-ttu-id="38a72-362">通知中心不支持发送短信或电子邮件，因此，对于这些传送类型，需要使用第三方服务。</span><span class="sxs-lookup"><span data-stu-id="38a72-362">Notification Hubs do not support sending SMS or email messages, so third-party services are needed for those delivery types.</span></span>

#### <a name="see-also"></a><span data-ttu-id="38a72-363">另请参阅</span><span class="sxs-lookup"><span data-stu-id="38a72-363">See also</span></span>

-   [<span data-ttu-id="38a72-364">创建 Android 应用</span><span class="sxs-lookup"><span data-stu-id="38a72-364">Create an Android app</span></span>](https://azure.microsoft.com/documentation/articles/app-service-mobile-android-get-started/)

-   [<span data-ttu-id="38a72-365">Azure 移动应用中的身份验证和授权</span><span class="sxs-lookup"><span data-stu-id="38a72-365">Authentication and Authorization in Azure Mobile Apps</span></span>](https://azure.microsoft.com/documentation/articles/app-service-mobile-auth/)

-   [<span data-ttu-id="38a72-366">使用 Azure 通知中心发送推送通知</span><span class="sxs-lookup"><span data-stu-id="38a72-366">Sending push notifications with Azure Notification Hubs</span></span>](https://azure.microsoft.com/documentation/articles/notification-hubs-android-push-notification-google-fcm-get-started/)

### <a name="management-and-monitoring"></a><span data-ttu-id="38a72-367">监视和管理</span><span class="sxs-lookup"><span data-stu-id="38a72-367">Management and monitoring</span></span>

#### <a name="see-also"></a><span data-ttu-id="38a72-368">另请参阅</span><span class="sxs-lookup"><span data-stu-id="38a72-368">See also</span></span>
-   [<span data-ttu-id="38a72-369">监视和诊断指南</span><span class="sxs-lookup"><span data-stu-id="38a72-369">Monitoring and diagnostics guidance</span></span>](https://azure.microsoft.com/documentation/articles/best-practices-monitoring/)

-   [<span data-ttu-id="38a72-370">有关创建 Azure 资源管理器模板的最佳做法</span><span class="sxs-lookup"><span data-stu-id="38a72-370">Best practices for creating Azure Resource Manager templates</span></span>](https://azure.microsoft.com/documentation/articles/resource-manager-template-best-practices/)

-   [<span data-ttu-id="38a72-371">Azure 资源管理器快速入门模板</span><span class="sxs-lookup"><span data-stu-id="38a72-371">Azure Resource Manager Quickstart templates</span></span>](https://azure.microsoft.com/documentation/templates/)


## <a name="next-steps"></a><span data-ttu-id="38a72-372">后续步骤</span><span class="sxs-lookup"><span data-stu-id="38a72-372">Next steps</span></span>

-   [<span data-ttu-id="38a72-373">AWS 和 Azure 服务的完整比较对照表</span><span class="sxs-lookup"><span data-stu-id="38a72-373">Complete AWS and Azure service comparison matrix</span></span>](https://aka.ms/azure4aws-services)

-   [<span data-ttu-id="38a72-374">交互式 Azure 平台概观</span><span class="sxs-lookup"><span data-stu-id="38a72-374">Interactive Azure Platform Big Picture</span></span>](http://azureplatform.azurewebsites.net/)

-   [<span data-ttu-id="38a72-375">Azure 入门</span><span class="sxs-lookup"><span data-stu-id="38a72-375">Get started with Azure</span></span>](https://azure.microsoft.com/get-started/)

-   [<span data-ttu-id="38a72-376">Azure 解决方案体系结构</span><span class="sxs-lookup"><span data-stu-id="38a72-376">Azure solution architectures</span></span>](https://azure.microsoft.com/solutions/architecture/)

-   [<span data-ttu-id="38a72-377">Azure 参考体系结构</span><span class="sxs-lookup"><span data-stu-id="38a72-377">Azure Reference Architectures</span></span>](https://azure.microsoft.com/documentation/articles/guidance-architecture/)

-   [<span data-ttu-id="38a72-378">模式和做法：Azure 指南</span><span class="sxs-lookup"><span data-stu-id="38a72-378">Patterns & Practices: Azure Guidance</span></span>](https://azure.microsoft.com/documentation/articles/guidance/)

-   [<span data-ttu-id="38a72-379">免费在线课程：面向 AWS 专家的 Microsoft Azure</span><span class="sxs-lookup"><span data-stu-id="38a72-379">Free Online Course: Microsoft Azure for AWS Experts</span></span>](http://aka.ms/azureforaws)
