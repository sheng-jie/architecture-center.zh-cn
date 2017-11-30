---
title: "在 Azure 上运行负载均衡的 VM 以提高可伸缩性和可用性"
description: "如何在 Azure 上运行多个 Windows VM 以提高可伸缩性和可用性。"
author: telmosampaio
ms.date: 09/07/2017
pnp.series.title: Windows VM workloads
pnp.series.next: n-tier
pnp.series.prev: single-vm
ms.openlocfilehash: d38cfb41255c547f1f1e87ef289c7a79033df778
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="run-load-balanced-vms-for-scalability-and-availability"></a><span data-ttu-id="0a775-103">运行负载均衡的 VM 以提高可伸缩性和可用性</span><span class="sxs-lookup"><span data-stu-id="0a775-103">Run load-balanced VMs for scalability and availability</span></span>

<span data-ttu-id="0a775-104">对于在负载均衡器后的规模集中运行多个 Windows 虚拟机 (VM) 以提高可用性和可伸缩性，此参考体系结构显示了一组已经过验证的做法。</span><span class="sxs-lookup"><span data-stu-id="0a775-104">This reference architecture shows a set of proven practices for running several Windows virtual machines (VMs) in a scale set behind a load balancer, to improve availability and scalability.</span></span> <span data-ttu-id="0a775-105">此体系结构可用于任何无状态工作负荷（如 Web 服务器），并且是用于部署 n 层应用程序的构建基块。</span><span class="sxs-lookup"><span data-stu-id="0a775-105">This architecture can be used for any stateless workload, such as a web server, and is a building block for deploying n-tier applications.</span></span> [<span data-ttu-id="0a775-106">**部署此解决方案**。</span><span class="sxs-lookup"><span data-stu-id="0a775-106">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="0a775-107">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="0a775-107">![[0]][0]</span></span>

<span data-ttu-id="0a775-108">*下载此体系结构的 [Visio 文件][visio-download]。*</span><span class="sxs-lookup"><span data-stu-id="0a775-108">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="0a775-109">体系结构</span><span class="sxs-lookup"><span data-stu-id="0a775-109">Architecture</span></span>

<span data-ttu-id="0a775-110">此体系结构基于[在 Azure 上运行 Windows VM][single vm] 中显示的体系结构。</span><span class="sxs-lookup"><span data-stu-id="0a775-110">This architecture builds on the one shown in [Run a Windows VM on Azure][single vm].</span></span> <span data-ttu-id="0a775-111">那里的建议也适用于此体系结构。</span><span class="sxs-lookup"><span data-stu-id="0a775-111">The recommendations there also apply to this architecture.</span></span>

<span data-ttu-id="0a775-112">在此体系结构中，工作负荷分布于多个 VM 实例上。</span><span class="sxs-lookup"><span data-stu-id="0a775-112">In this architecture, a workload is distributed across several VM instances.</span></span> <span data-ttu-id="0a775-113">有单个公共 IP 地址，并且 Internet 流量通过负载均衡器分布到各个 VM。</span><span class="sxs-lookup"><span data-stu-id="0a775-113">There is a single public IP address, and Internet traffic is distributed to the VMs using a load balancer.</span></span> <span data-ttu-id="0a775-114">此体系结构可用于单层应用程序，例如无状态 Web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="0a775-114">This architecture can be used for a single-tier application, such as a stateless web application.</span></span>

<span data-ttu-id="0a775-115">此体系结构具有以下组件：</span><span class="sxs-lookup"><span data-stu-id="0a775-115">The architecture has the following components:</span></span>

* <span data-ttu-id="0a775-116">**资源组。**</span><span class="sxs-lookup"><span data-stu-id="0a775-116">**Resource group.**</span></span> <span data-ttu-id="0a775-117">[*资源组*][resource-manager-overview]用于对资源进行分组，以便它们可以按生存期、所有者和其他条件进行管理。</span><span class="sxs-lookup"><span data-stu-id="0a775-117">[*Resource groups*][resource-manager-overview] are used to group resources so they can be managed by lifetime, owner, and other criteria.</span></span>
* <span data-ttu-id="0a775-118">**虚拟网络 (VNet) 和子网。**</span><span class="sxs-lookup"><span data-stu-id="0a775-118">**Virtual network (VNet) and subnet.**</span></span> <span data-ttu-id="0a775-119">Azure 中的每个 VM 都部署在 VNet 中，后者进一步划分为多个子网。</span><span class="sxs-lookup"><span data-stu-id="0a775-119">Every VM in Azure is deployed into a VNet that is further divided into subnets.</span></span>
* <span data-ttu-id="0a775-120">**Azure 负载均衡器**。</span><span class="sxs-lookup"><span data-stu-id="0a775-120">**Azure Load Balancer**.</span></span> <span data-ttu-id="0a775-121">[负载均衡器]将传入 Internet 请求分布到各个 VM 实例。</span><span class="sxs-lookup"><span data-stu-id="0a775-121">The [load balancer] distributes incoming Internet requests to the VM instances.</span></span> 
* <span data-ttu-id="0a775-122">**公共 IP 地址**。</span><span class="sxs-lookup"><span data-stu-id="0a775-122">**Public IP address**.</span></span> <span data-ttu-id="0a775-123">负载均衡器需要使用一个公共 IP 地址来接收 Internet 流量。</span><span class="sxs-lookup"><span data-stu-id="0a775-123">A public IP address is needed for the load balancer to receive Internet traffic.</span></span>
* <span data-ttu-id="0a775-124">**VM 规模集**。</span><span class="sxs-lookup"><span data-stu-id="0a775-124">**VM scale set**.</span></span> <span data-ttu-id="0a775-125">[VM 规模集][vm-scaleset]是一组用于托管工作负荷的相同 VM。</span><span class="sxs-lookup"><span data-stu-id="0a775-125">A [VM scale set][vm-scaleset] is a set of identical VMs used to host a workload.</span></span> <span data-ttu-id="0a775-126">规模集允许以手动方式或基于预定义规则增加或减少 VM 的数量。</span><span class="sxs-lookup"><span data-stu-id="0a775-126">Scale sets allow the number of VMs to be scaled in or out manually, or based on predefined rules.</span></span>
* <span data-ttu-id="0a775-127">**可用性集**。</span><span class="sxs-lookup"><span data-stu-id="0a775-127">**Availability set**.</span></span> <span data-ttu-id="0a775-128">[可用性集][availability set]包含 VM，使 VM 符合更高[服务级别协议 (SLA)][vm-sla] 的要求。</span><span class="sxs-lookup"><span data-stu-id="0a775-128">The [availability set][availability set] contains the VMs, making the VMs eligible for a higher [service level agreement (SLA)][vm-sla].</span></span> <span data-ttu-id="0a775-129">为应用更高级别的 SLA，可用性集必须包含至少两个 VM。</span><span class="sxs-lookup"><span data-stu-id="0a775-129">For the higher SLA to apply, the availability set must include a minimum of two VMs.</span></span> <span data-ttu-id="0a775-130">可用性集隐含于规模集中。</span><span class="sxs-lookup"><span data-stu-id="0a775-130">Availability sets are implicit in scale sets.</span></span> <span data-ttu-id="0a775-131">如果在规模集外创建 VM，需要独立创建可用性集。</span><span class="sxs-lookup"><span data-stu-id="0a775-131">If you create VMs outside a scale set, you need to create the availability set independently.</span></span>
* <span data-ttu-id="0a775-132">**托管磁盘**。</span><span class="sxs-lookup"><span data-stu-id="0a775-132">**Managed disks**.</span></span> <span data-ttu-id="0a775-133">Azure 托管磁盘管理 VM 磁盘的虚拟硬盘 (VHD) 文件。</span><span class="sxs-lookup"><span data-stu-id="0a775-133">Azure Managed Disks manage the virtual hard disk (VHD) files for the VM disks.</span></span> 
* <span data-ttu-id="0a775-134">**存储**。</span><span class="sxs-lookup"><span data-stu-id="0a775-134">**Storage**.</span></span> <span data-ttu-id="0a775-135">创建一个 Azure 存储帐户来保存 VM 的诊断日志。</span><span class="sxs-lookup"><span data-stu-id="0a775-135">Create an Azure Storage acount to hold diagnostic logs for the VMs.</span></span>

## <a name="recommendations"></a><span data-ttu-id="0a775-136">建议</span><span class="sxs-lookup"><span data-stu-id="0a775-136">Recommendations</span></span>

<span data-ttu-id="0a775-137">你的要求可能与此处描述的体系结构不同。</span><span class="sxs-lookup"><span data-stu-id="0a775-137">Your requirements might differ from the architecture described here.</span></span> <span data-ttu-id="0a775-138">请使用以下建议作为入手点。</span><span class="sxs-lookup"><span data-stu-id="0a775-138">Use these recommendations as a starting point.</span></span> 

### <a name="availability-and-scalability-recommendations"></a><span data-ttu-id="0a775-139">可用性和可伸缩性建议</span><span class="sxs-lookup"><span data-stu-id="0a775-139">Availability and scalability recommendations</span></span>

<span data-ttu-id="0a775-140">实现可用性和可伸缩性的一个选项是使用[虚拟机规模集][vmss]。</span><span class="sxs-lookup"><span data-stu-id="0a775-140">An option for availability and scalability is to use a [virtual machine scale set][vmss].</span></span> <span data-ttu-id="0a775-141">VM 规模集可帮助部署和管理一组相同的 VM。</span><span class="sxs-lookup"><span data-stu-id="0a775-141">VM scale sets help you to deploy and manage a set of identical VMs.</span></span> <span data-ttu-id="0a775-142">规模集支持基于性能指标自动缩放。</span><span class="sxs-lookup"><span data-stu-id="0a775-142">Scale sets support autoscaling based on performance metrics.</span></span> <span data-ttu-id="0a775-143">VM 上的负载增加时，会自动向负载均衡器添加更多 VM。</span><span class="sxs-lookup"><span data-stu-id="0a775-143">As the load on the VMs increases, additional VMs are automatically added to the load balancer.</span></span> <span data-ttu-id="0a775-144">如果你需要快速横向扩展 VM，或者需要进行自动缩放，请考虑使用规模集。</span><span class="sxs-lookup"><span data-stu-id="0a775-144">Consider scale sets if you need to quickly scale out VMs, or need to autoscale.</span></span>

<span data-ttu-id="0a775-145">默认情况下，规模集使用“过度预配”，这意味着规模集最初会预配多于你要求的数量的 VM，然后删除额外的 VM。</span><span class="sxs-lookup"><span data-stu-id="0a775-145">By default, scale sets use "overprovisioning," which means the scale set initially provisions more VMs than you ask for, then deletes the extra VMs.</span></span> <span data-ttu-id="0a775-146">这会提高预配 VM 时的整体成功率。</span><span class="sxs-lookup"><span data-stu-id="0a775-146">This improves the overall success rate when provisioning the VMs.</span></span> <span data-ttu-id="0a775-147">如果不使用[托管磁盘](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-managed-disks)，则建议每个存储帐户在启用过度预配的情况下不超过 20 个 VM，在禁用过度预配的情况下不超过 40 个 VM。</span><span class="sxs-lookup"><span data-stu-id="0a775-147">If you are not using [managed disks](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-managed-disks), we recommend no more than 20 VMs per storage account with overprovisioning enabled, or no more than 40 VMs with overprovisioning disabled.</span></span>

<span data-ttu-id="0a775-148">有两种基本方法可用来配置规模集中部署的 VM：</span><span class="sxs-lookup"><span data-stu-id="0a775-148">There are two basic ways to configure VMs deployed in a scale set:</span></span>

- <span data-ttu-id="0a775-149">在预配 VM 后使用扩展对其进行配置。</span><span class="sxs-lookup"><span data-stu-id="0a775-149">Use extensions to configure the VM after it is provisioned.</span></span> <span data-ttu-id="0a775-150">使用此方法时，启动新 VM 实例的所需时间可能会长于启动不带扩展的 VM 的所需时间。</span><span class="sxs-lookup"><span data-stu-id="0a775-150">With this approach, new VM instances may take longer to start up than a VM with no extensions.</span></span>

- <span data-ttu-id="0a775-151">使用自定义磁盘映像部署[托管磁盘](/azure/storage/storage-managed-disks-overview)。</span><span class="sxs-lookup"><span data-stu-id="0a775-151">Deploy a [managed disk](/azure/storage/storage-managed-disks-overview) with a custom disk image.</span></span> <span data-ttu-id="0a775-152">此选项的部署速度可能更快。</span><span class="sxs-lookup"><span data-stu-id="0a775-152">This option may be quicker to deploy.</span></span> <span data-ttu-id="0a775-153">但是，它要求将映像保持最新。</span><span class="sxs-lookup"><span data-stu-id="0a775-153">However, it requires you to keep the image up to date.</span></span>

<span data-ttu-id="0a775-154">有关其他注意事项，请参阅[规模集的设计注意事项][vmss-design]。</span><span class="sxs-lookup"><span data-stu-id="0a775-154">For additional considerations, see [Design considerations for scale sets][vmss-design].</span></span>

> [!TIP]
> <span data-ttu-id="0a775-155">在使用任何自动缩放解决方案时，请早早提前使用生产级工作负荷测试它。</span><span class="sxs-lookup"><span data-stu-id="0a775-155">When using any autoscale solution, test it with production-level workloads well in advance.</span></span>

<span data-ttu-id="0a775-156">如果不使用规模集，请考虑至少使用可用性集。</span><span class="sxs-lookup"><span data-stu-id="0a775-156">If you do not use a scale set, consider at least using an availability set.</span></span> <span data-ttu-id="0a775-157">请在可用性集中至少创建两个 VM 以支持 [Azure VM 的可用性 SLA][vm-sla]。</span><span class="sxs-lookup"><span data-stu-id="0a775-157">Create at least two VMs in the availability set, to support the [availability SLA for Azure VMs][vm-sla].</span></span> <span data-ttu-id="0a775-158">Azure 负载均衡器还要求各个负载均衡 VM 属于同一可用性集。</span><span class="sxs-lookup"><span data-stu-id="0a775-158">The Azure load balancer also requires that load-balanced VMs belong to the same availability set.</span></span>

<span data-ttu-id="0a775-159">每个 Azure 订阅都有适用的默认限制，包括每个区域的最大 VM 数量。</span><span class="sxs-lookup"><span data-stu-id="0a775-159">Each Azure subscription has default limits in place, including a maximum number of VMs per region.</span></span> <span data-ttu-id="0a775-160">可以通过提出支持请求来提高上限。</span><span class="sxs-lookup"><span data-stu-id="0a775-160">You can increase the limit by filing a support request.</span></span> <span data-ttu-id="0a775-161">有关详细信息，请参阅 [Azure 订阅和服务限制、配额与约束][subscription-limits]。</span><span class="sxs-lookup"><span data-stu-id="0a775-161">For more information, see [Azure subscription and service limits, quotas, and constraints][subscription-limits].</span></span>

### <a name="network-recommendations"></a><span data-ttu-id="0a775-162">网络建议</span><span class="sxs-lookup"><span data-stu-id="0a775-162">Network recommendations</span></span>

<span data-ttu-id="0a775-163">将各个 VM 置于同一子网内。</span><span class="sxs-lookup"><span data-stu-id="0a775-163">Place the VMs within the same subnet.</span></span> <span data-ttu-id="0a775-164">不要将 VM 直接向 Internet 公开，而是改为给每个 VM 提供专用 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="0a775-164">Do not expose the VMs directly to the Internet, but instead give each VM a private IP address.</span></span> <span data-ttu-id="0a775-165">客户端使用负载均衡器的公共 IP 地址进行连接。</span><span class="sxs-lookup"><span data-stu-id="0a775-165">Clients connect using the public IP address of the load balancer.</span></span>

<span data-ttu-id="0a775-166">如果需要登录到负载均衡器后面的 VM，请考虑将单个 VM 添加为守护主机/jumpbox 并使其具有可以登录到的公共 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="0a775-166">If you need to log into the VMs behind the load balancer, consider adding a single VM as a bastion host/jumpbox with a public IP address you can log into.</span></span> <span data-ttu-id="0a775-167">然后从 jumpbox 登录到负载均衡器后面的 VM。</span><span class="sxs-lookup"><span data-stu-id="0a775-167">And then log into the VMs behind the load balancer from the jumpbox.</span></span> <span data-ttu-id="0a775-168">另外，也可以通过在负载均衡器中配置入站 NAT 规则来实现同一目的。</span><span class="sxs-lookup"><span data-stu-id="0a775-168">Alternatively, configure inbound NAT rules in the load balancer for the same purpose.</span></span> <span data-ttu-id="0a775-169">但是，托管 n 层工作负荷或多个工作负荷时，使用 jumpbox 是更好的解决方案。</span><span class="sxs-lookup"><span data-stu-id="0a775-169">However, having a jumpbox is a better solution when you are hosting n-tier workloads, or multiple workloads.</span></span>

### <a name="load-balancer-recommendations"></a><span data-ttu-id="0a775-170">负载均衡器建议</span><span class="sxs-lookup"><span data-stu-id="0a775-170">Load balancer recommendations</span></span>

<span data-ttu-id="0a775-171">将可用性集中的所有 VM 都添加到负载均衡器的后端地址池。</span><span class="sxs-lookup"><span data-stu-id="0a775-171">Add all VMs in the availability set to the back-end address pool of the load balancer.</span></span>

<span data-ttu-id="0a775-172">定义用于将网络流量定向到 VM 的负载均衡器规则。</span><span class="sxs-lookup"><span data-stu-id="0a775-172">Define load balancer rules to direct network traffic to the VMs.</span></span> <span data-ttu-id="0a775-173">例如，若要启用 HTTP 流量，请创建将前端配置中的端口 80 映射到后端地址池上的端口 80 的规则。</span><span class="sxs-lookup"><span data-stu-id="0a775-173">For example, to enable HTTP traffic, create a rule that maps port 80 from the front-end configuration to port 80 on the back-end address pool.</span></span> <span data-ttu-id="0a775-174">当客户端将 HTTP 请求发送到端口 80 时，负载均衡器会通过使用包括源 IP 地址的[哈希算法][load balancer hashing]选择后端 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="0a775-174">When a client sends an HTTP request to port 80, the load balancer selects a back-end IP address by using a [hashing algorithm][load balancer hashing] that includes the source IP address.</span></span> <span data-ttu-id="0a775-175">这样，客户端请求就会分布在所有 VM 上。</span><span class="sxs-lookup"><span data-stu-id="0a775-175">In that way, client requests are distributed across all the VMs.</span></span>

<span data-ttu-id="0a775-176">若要将流量路由到特定 VM，请使用 NAT 规则。</span><span class="sxs-lookup"><span data-stu-id="0a775-176">To route traffic to a specific VM, use NAT rules.</span></span> <span data-ttu-id="0a775-177">例如，若要为 VM 启用 RDP，请为每个 VM 创建单独的 NAT 规则。</span><span class="sxs-lookup"><span data-stu-id="0a775-177">For example, to enable RDP to the VMs, create a separate NAT rule for each VM.</span></span> <span data-ttu-id="0a775-178">每个规则应将不重复的端口号映射到端口 3389（RDP 的默认端口）。</span><span class="sxs-lookup"><span data-stu-id="0a775-178">Each rule should map a distinct port number to port 3389, the default port for RDP.</span></span> <span data-ttu-id="0a775-179">例如，将端口 50001 用于“VM1”，将端口 50002 用于“VM2”，依此类推。</span><span class="sxs-lookup"><span data-stu-id="0a775-179">For example, use port 50001 for "VM1," port 50002 for "VM2," and so on.</span></span> <span data-ttu-id="0a775-180">将 NAT 规则分配给 VM 上的 NIC。</span><span class="sxs-lookup"><span data-stu-id="0a775-180">Assign the NAT rules to the NICs on the VMs.</span></span>

### <a name="storage-account-recommendations"></a><span data-ttu-id="0a775-181">存储帐户建议</span><span class="sxs-lookup"><span data-stu-id="0a775-181">Storage account recommendations</span></span>

<span data-ttu-id="0a775-182">为避免达到存储帐户的每秒输入输出/操作数 [(IOPS) 限制][vm-disk-limits]，请为每个 VM 创建单独的 Azure 存储帐户来存放虚拟硬盘 (VHD)。</span><span class="sxs-lookup"><span data-stu-id="0a775-182">Create separate Azure storage accounts for each VM to hold the virtual hard disks (VHDs), in order to avoid hitting the input/output operations per second [(IOPS) limits][vm-disk-limits] for storage accounts.</span></span>

<span data-ttu-id="0a775-183">建议将[托管磁盘](/azure/storage/storage-managed-disks-overview)与[高级存储][premium]配合使用。</span><span class="sxs-lookup"><span data-stu-id="0a775-183">We recommend the use of [managed disks](/azure/storage/storage-managed-disks-overview) with [premium storage][premium].</span></span> <span data-ttu-id="0a775-184">托管磁盘不需要存储帐户。</span><span class="sxs-lookup"><span data-stu-id="0a775-184">Managed disks do not require a storage account.</span></span> <span data-ttu-id="0a775-185">只需指定磁盘的大小和类型，它就会以高度可用的方式部署。</span><span class="sxs-lookup"><span data-stu-id="0a775-185">You simply specify the size and type of disk and it is deployed in a highly available way.</span></span>

<span data-ttu-id="0a775-186">为诊断日志创建一个存储帐户。</span><span class="sxs-lookup"><span data-stu-id="0a775-186">Create one storage account for diagnostic logs.</span></span> <span data-ttu-id="0a775-187">此存储帐户可由所有 VM 共享。</span><span class="sxs-lookup"><span data-stu-id="0a775-187">This storage account can be shared by all the VMs.</span></span> <span data-ttu-id="0a775-188">这可以是使用标准磁盘的非托管存储帐户。</span><span class="sxs-lookup"><span data-stu-id="0a775-188">This can be an unmanaged storage account using standard disks.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="0a775-189">可用性注意事项</span><span class="sxs-lookup"><span data-stu-id="0a775-189">Availability considerations</span></span>

<span data-ttu-id="0a775-190">可用性集使应用程序在发生计划内和计划外维护事件后能够更好地进行复原。</span><span class="sxs-lookup"><span data-stu-id="0a775-190">The availability set makes your application more resilient to both planned and unplanned maintenance events.</span></span>

* <span data-ttu-id="0a775-191">*计划内维护*在 Microsoft 更新基础平台时进行，有时会导致 VM 重新启动。</span><span class="sxs-lookup"><span data-stu-id="0a775-191">*Planned maintenance* occurs when Microsoft updates the underlying platform, sometimes causing VMs to be restarted.</span></span> <span data-ttu-id="0a775-192">Azure 可确保可用性集中的 VM 不会全部同时重新启动。</span><span class="sxs-lookup"><span data-stu-id="0a775-192">Azure makes sure the VMs in an availability set are not all restarted at the same time.</span></span> <span data-ttu-id="0a775-193">至少有一个 VM 在其他 VM 重新启动时会保持运行。</span><span class="sxs-lookup"><span data-stu-id="0a775-193">At least one is kept running while others are restarting.</span></span>
* <span data-ttu-id="0a775-194">*计划外维护*在发生硬件故障时进行。</span><span class="sxs-lookup"><span data-stu-id="0a775-194">*Unplanned maintenance* happens if there is a hardware failure.</span></span> <span data-ttu-id="0a775-195">Azure 会确保跨多个服务器机架预配可用性集中的 VM。</span><span class="sxs-lookup"><span data-stu-id="0a775-195">Azure makes sure that VMs in an availability set are provisioned across more than one server rack.</span></span> <span data-ttu-id="0a775-196">这有助于减少硬件故障、网络中断、电力中断等事件的影响。</span><span class="sxs-lookup"><span data-stu-id="0a775-196">This helps to reduce the impact of hardware failures, network outages, power interruptions, and so on.</span></span>

<span data-ttu-id="0a775-197">有关详细信息，请参阅[管理虚拟机的可用性][availability set]。</span><span class="sxs-lookup"><span data-stu-id="0a775-197">For more information, see [Manage the availability of virtual machines][availability set].</span></span> <span data-ttu-id="0a775-198">下面的视频还包含一个不错的可用性集概述：[如何配置可用性集以缩放 VM][availability set ch9]。</span><span class="sxs-lookup"><span data-stu-id="0a775-198">The following video also has a good overview of availability sets: [How Do I Configure an Availability Set to Scale VMs][availability set ch9].</span></span>

> [!WARNING]
> <span data-ttu-id="0a775-199">务必在预配 VM 时配置可用性集。</span><span class="sxs-lookup"><span data-stu-id="0a775-199">Make sure to configure the availability set when you provision the VM.</span></span> <span data-ttu-id="0a775-200">目前，没有任何方法可以在预配 VM 后将资源管理器 VM 添加到可用性集。</span><span class="sxs-lookup"><span data-stu-id="0a775-200">Currently, there is no way to add a Resource Manager VM to an availability set after the VM is provisioned.</span></span>

<span data-ttu-id="0a775-201">负载均衡器使用[运行状况探测]监视 VM 实例的可用性。</span><span class="sxs-lookup"><span data-stu-id="0a775-201">The load balancer uses [health probes] to monitor the availability of VM instances.</span></span> <span data-ttu-id="0a775-202">如果探测在超时期限内无法到达实例，负载均衡器会停止向该 VM 发送流量。</span><span class="sxs-lookup"><span data-stu-id="0a775-202">If a probe cannot reach an instance within a timeout period, the load balancer stops sending traffic to that VM.</span></span> <span data-ttu-id="0a775-203">但是，负载均衡器将继续探测，并且如果 VM 再次变得可用，负载均衡器会继续向该 VM 发送流量。</span><span class="sxs-lookup"><span data-stu-id="0a775-203">However, the load balancer will continue to probe, and if the VM becomes available again, the load balancer resumes sending traffic to that VM.</span></span>

<span data-ttu-id="0a775-204">下面是有关负载均衡器运行状况探测的一些建议：</span><span class="sxs-lookup"><span data-stu-id="0a775-204">Here are some recommendations on load balancer health probes:</span></span>

* <span data-ttu-id="0a775-205">探测可以测试 HTTP 或 TCP。</span><span class="sxs-lookup"><span data-stu-id="0a775-205">Probes can test either HTTP or TCP.</span></span> <span data-ttu-id="0a775-206">如果 VM 运行 HTTP 服务器，请创建 HTTP 探测。</span><span class="sxs-lookup"><span data-stu-id="0a775-206">If your VMs run an HTTP server, create an HTTP probe.</span></span> <span data-ttu-id="0a775-207">否则，请创建 TCP 探测。</span><span class="sxs-lookup"><span data-stu-id="0a775-207">Otherwise create a TCP probe.</span></span>
* <span data-ttu-id="0a775-208">对于 HTTP 探测，请指定指向 HTTP 终结点的路径。</span><span class="sxs-lookup"><span data-stu-id="0a775-208">For an HTTP probe, specify the path to an HTTP endpoint.</span></span> <span data-ttu-id="0a775-209">探测将检查是否有来自此路径的 HTTP 200 响应。</span><span class="sxs-lookup"><span data-stu-id="0a775-209">The probe checks for an HTTP 200 response from this path.</span></span> <span data-ttu-id="0a775-210">这可以是根路径 ("/")，也可以是一个运行状况监视终结点，该终结点实现某些自定义逻辑来检查应用程序运行状况。</span><span class="sxs-lookup"><span data-stu-id="0a775-210">This can be the root path ("/"), or a health-monitoring endpoint that implements some custom logic to check the health of the application.</span></span> <span data-ttu-id="0a775-211">终结点必须允许匿名 HTTP 请求。</span><span class="sxs-lookup"><span data-stu-id="0a775-211">The endpoint must allow anonymous HTTP requests.</span></span>
* <span data-ttu-id="0a775-212">探测是从[已知][health-probe-ip] IP 地址 168.63.129.16 发送的。</span><span class="sxs-lookup"><span data-stu-id="0a775-212">The probe is sent from a [known][health-probe-ip] IP address, 168.63.129.16.</span></span> <span data-ttu-id="0a775-213">请确保任何防火墙策略或网络安全组 (NSG) 规则都未阻止与此 IP 之间的流量。</span><span class="sxs-lookup"><span data-stu-id="0a775-213">Make sure you don't block traffic to or from this IP in any firewall policies or network security group (NSG) rules.</span></span>
* <span data-ttu-id="0a775-214">使用[运行状况探测日志][health probe log]查看运行状况探测的状态。</span><span class="sxs-lookup"><span data-stu-id="0a775-214">Use [health probe logs][health probe log] to view the status of the health probes.</span></span> <span data-ttu-id="0a775-215">请在 Azure 门户中为每个负载均衡器启用日志记录。</span><span class="sxs-lookup"><span data-stu-id="0a775-215">Enable logging in the Azure portal for each load balancer.</span></span> <span data-ttu-id="0a775-216">日志将写入到 Azure Blob 存储。</span><span class="sxs-lookup"><span data-stu-id="0a775-216">Logs are written to Azure Blob storage.</span></span> <span data-ttu-id="0a775-217">这些日志显示后端有多少个 VM 由于失败的探测响应而未收到网络流量。</span><span class="sxs-lookup"><span data-stu-id="0a775-217">The logs show how many VMs on the back end are not receiving network traffic due to failed probe responses.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="0a775-218">可管理性注意事项</span><span class="sxs-lookup"><span data-stu-id="0a775-218">Manageability considerations</span></span>

<span data-ttu-id="0a775-219">如果使用多个 VM ，那么自动执行过程会非常重要，这样过程将非常可靠且可重复。</span><span class="sxs-lookup"><span data-stu-id="0a775-219">With multiple VMs, it is important to automate processes so they are reliable and repeatable.</span></span> <span data-ttu-id="0a775-220">可以使用 [Azure 自动化][azure-automation]自动执行部署、OS 修补和其他任务。</span><span class="sxs-lookup"><span data-stu-id="0a775-220">You can use [Azure Automation][azure-automation] to automate deployment, OS patching, and other tasks.</span></span> <span data-ttu-id="0a775-221">[Azure 自动化][azure-automation]是一项自动化服务，它基于可以用于此用途的 Windows Powershell。</span><span class="sxs-lookup"><span data-stu-id="0a775-221">[Azure Automation][azure-automation] is an automation service based on Windows Powershell that can be used for this.</span></span> <span data-ttu-id="0a775-222">TechNet 上的 [Runbook 库]中提供了示例自动化脚本。</span><span class="sxs-lookup"><span data-stu-id="0a775-222">Example automation scripts are available from the [Runbook Gallery] on TechNet.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="0a775-223">安全注意事项</span><span class="sxs-lookup"><span data-stu-id="0a775-223">Security considerations</span></span>

<span data-ttu-id="0a775-224">虚拟网络是 Azure 中的流量隔离边界。</span><span class="sxs-lookup"><span data-stu-id="0a775-224">Virtual networks are a traffic isolation boundary in Azure.</span></span> <span data-ttu-id="0a775-225">一个 VNet 中的 VM 无法直接与其他 VNet 中的 VM 通信。</span><span class="sxs-lookup"><span data-stu-id="0a775-225">VMs in one VNet cannot communicate directly to VMs in a different VNet.</span></span> <span data-ttu-id="0a775-226">同一个 VNet 中的 VM 之间可以通信，除非你创建[网络安全组][nsg] (NSG) 来限制流量。</span><span class="sxs-lookup"><span data-stu-id="0a775-226">VMs within the same VNet can communicate, unless you create [network security groups][nsg] (NSGs) to restrict traffic.</span></span> <span data-ttu-id="0a775-227">有关详细信息，请参阅 [Microsoft 云服务和网络安全性][network-security]。</span><span class="sxs-lookup"><span data-stu-id="0a775-227">For more information, see [Microsoft cloud services and network security][network-security].</span></span>

<span data-ttu-id="0a775-228">对于传入 Internet 流量，负载均衡器规则定义哪些流量可以到达后端。</span><span class="sxs-lookup"><span data-stu-id="0a775-228">For incoming Internet traffic, the load balancer rules define which traffic can reach the back end.</span></span> <span data-ttu-id="0a775-229">但是，负载均衡器规则不支持 IP 安全列表，因此如果要将某些公共 IP 地址添加到安全列表，请将 NSG 添加到子网。</span><span class="sxs-lookup"><span data-stu-id="0a775-229">However, load balancer rules don't support IP safe lists, so if you want to add certain public IP addresses to a safe list, add an NSG to the subnet.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="0a775-230">部署解决方案</span><span class="sxs-lookup"><span data-stu-id="0a775-230">Deploy the solution</span></span>

<span data-ttu-id="0a775-231">[GitHub][github-folder] 上提供了此体系结构的部署。</span><span class="sxs-lookup"><span data-stu-id="0a775-231">A deployment for this architecture is available on [GitHub][github-folder].</span></span> <span data-ttu-id="0a775-232">它部署以下项：</span><span class="sxs-lookup"><span data-stu-id="0a775-232">It deploys the following:</span></span>

  * <span data-ttu-id="0a775-233">一个虚拟网络，其中包含用来托管 VM 的名为 **web** 的单个子网。</span><span class="sxs-lookup"><span data-stu-id="0a775-233">A virtual network with a single subnet named **web** used to host the VMs.</span></span>
  * <span data-ttu-id="0a775-234">一个 VM 规模集，其中包含运行 Windows Server 2016 Datacenter Edition 最新版本的 VM。</span><span class="sxs-lookup"><span data-stu-id="0a775-234">A VM scale set that contains VMs running the latest version of Windows Server 2016 Datacenter Edition.</span></span> <span data-ttu-id="0a775-235">启用自动缩放。</span><span class="sxs-lookup"><span data-stu-id="0a775-235">Autoscale is enabled.</span></span>
  * <span data-ttu-id="0a775-236">一个位于 VM 规模集前面的负载均衡器。</span><span class="sxs-lookup"><span data-stu-id="0a775-236">A load balancer that sits in front of the VM scale set.</span></span>
  * <span data-ttu-id="0a775-237">一个 NSG，其中包含用于允许发送到 VM 规模集的 HTTP 流量的传入规则。</span><span class="sxs-lookup"><span data-stu-id="0a775-237">An NSG with incoming rules to allow HTTP traffic to the VM scale set.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="0a775-238">先决条件</span><span class="sxs-lookup"><span data-stu-id="0a775-238">Prerequisites</span></span>

<span data-ttu-id="0a775-239">在将参考体系结构部署到自己的订阅之前，必须执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="0a775-239">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="0a775-240">为 [AzureCAT 参考体系结构][ref-arch-repo] GitHub 存储库克隆、下载 zip 文件或创建其分支。</span><span class="sxs-lookup"><span data-stu-id="0a775-240">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="0a775-241">确保在计算机上安装了 Azure CLI 2.0。</span><span class="sxs-lookup"><span data-stu-id="0a775-241">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="0a775-242">若要安装 CLI，请按照[安装 Azure CLI 2.0][azure-cli-2] 中的说明执行操作。</span><span class="sxs-lookup"><span data-stu-id="0a775-242">To install the CLI, follow the instructions in [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="0a775-243">安装 [Azure 构建基块][azbb] npm 包。</span><span class="sxs-lookup"><span data-stu-id="0a775-243">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="0a775-244">从命令提示符、bash 提示符或 PowerShell 提示符下通过使用以下命令之一，登录到 Azure 帐户，然后按照提示进行操作。</span><span class="sxs-lookup"><span data-stu-id="0a775-244">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using one of the commands below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a><span data-ttu-id="0a775-245">使用 azbb 部署解决方案</span><span class="sxs-lookup"><span data-stu-id="0a775-245">Deploy the solution using azbb</span></span>

<span data-ttu-id="0a775-246">若要部署示例单一 VM 工作负荷，请执行下列步骤：</span><span class="sxs-lookup"><span data-stu-id="0a775-246">To deploy the sample single VM workload, follow these steps:</span></span>

1. <span data-ttu-id="0a775-247">导航到已在前面的先决条件步骤中下载的存储库的 `virtual-machines\multi-vm\parameters\windows` 文件夹。</span><span class="sxs-lookup"><span data-stu-id="0a775-247">Navigate to the `virtual-machines\multi-vm\parameters\windows` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="0a775-248">打开 `multi-vm-v2.json` 文件，输入括在引号中的用户名和密码，如下所示，然后保存文件。</span><span class="sxs-lookup"><span data-stu-id="0a775-248">Open the `multi-vm-v2.json` file and enter a username and password between the quotes, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "",
  "adminPassword": "",
  ```

3. <span data-ttu-id="0a775-249">运行 `azbb` 以部署 VM，如下所示。</span><span class="sxs-lookup"><span data-stu-id="0a775-249">Run `azbb` to deploy the VMs as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p multi-vm-v2.json --deploy
  ```

<span data-ttu-id="0a775-250">有关部署此示例参考体系结构的详细信息，请访问 [GitHub 存储库][git]。</span><span class="sxs-lookup"><span data-stu-id="0a775-250">For more information on deploying this sample reference architecture, visit our [GitHub repository][git].</span></span>

<!-- Links -->
[github-folder]: http://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[n-tier-linux]: ../virtual-machines-linux/n-tier.md
[n-tier-windows]: n-tier.md
[single vm]: single-vm.md
[premium]: /azure/storage/common/storage-premium-storage
[naming conventions]: /azure/guidance/guidance-naming-conventions
[vm-scaleset]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[availability set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[availability set ch9]: https://channel9.msdn.com/Series/Microsoft-Azure-Fundamentals-Virtual-Machines/08
[azure-automation]: https://azure.microsoft.com/documentation/services/automation/
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-automation]: /azure/automation/automation-intro
[bastion host]: https://en.wikipedia.org/wiki/Bastion_host
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[health probe log]: /azure/load-balancer/load-balancer-monitor-log
[运行状况探测]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[负载均衡器]: /azure/load-balancer/load-balancer-get-started-internet-arm-cli
[load balancer hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[network-security]: /azure/best-practices-network-security
[nsg]: /azure/virtual-network/virtual-networks-nsg
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview 
[Runbook 库]: /azure/automation/automation-runbook-gallery#runbooks-in-runbook-gallery
[subscription-limits]: /azure/azure-subscription-service-limits
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_2/
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[vmss-quickstart]: https://azure.microsoft.com/documentation/templates/?term=scale+set
[VM-sizes]: https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/
[0]: ./images/multi-vm-diagram.png "Azure 上的多 VM 解决方案的体系结构，其中包含带有两个 VM 和一个负载均衡器的可用性集"
[azure-cli-2]: /azure/install-azure-cli?view=azure-cli-latest
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Template-Building-Blocks-Version-2-(Windows)
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm