---
title: "在 Azure 上运行 Linux VM"
description: "如何在 Azure 上运行 Linux VM，并注意可伸缩性、弹性、可管理性和安全性。"
author: telmosampaio
ms.date: 09/06/2017
pnp.series.title: Linux VM workloads
pnp.series.next: multi-vm
pnp.series.prev: ./index
ms.openlocfilehash: f38c53db5df1681a1abc926df9f0b7f929ceb76b
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="run-a-linux-vm-on-azure"></a><span data-ttu-id="fe27c-103">在 Azure 上运行 Linux VM</span><span class="sxs-lookup"><span data-stu-id="fe27c-103">Run a Linux VM on Azure</span></span>

<span data-ttu-id="fe27c-104">此参考体系结构显示一组已经过验证的、在 Azure 上运行 Linux 虚拟机 (VM) 的做法。</span><span class="sxs-lookup"><span data-stu-id="fe27c-104">This reference architecture shows a set of proven practices for running a Linux virtual machine (VM) on Azure.</span></span> <span data-ttu-id="fe27c-105">其中包括有关预配 VM 以及网络和存储组件的建议。</span><span class="sxs-lookup"><span data-stu-id="fe27c-105">It includes recommendations for provisioning the VM along with networking and storage components.</span></span> <span data-ttu-id="fe27c-106">此体系结构可用来运行单个实例，并且是更复杂的体系结构（如 N 层应用程序）的基础。</span><span class="sxs-lookup"><span data-stu-id="fe27c-106">This architecture can be used to run a single instance, and is the basis for more complex architectures such as n-tier applications.</span></span> [<span data-ttu-id="fe27c-107">**部署此解决方案**。</span><span class="sxs-lookup"><span data-stu-id="fe27c-107">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="fe27c-108">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="fe27c-108">![[0]][0]</span></span>

<span data-ttu-id="fe27c-109">下载此体系结构的 [Visio 文件][visio-download]。</span><span class="sxs-lookup"><span data-stu-id="fe27c-109">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="fe27c-110">体系结构</span><span class="sxs-lookup"><span data-stu-id="fe27c-110">Architecture</span></span>

<span data-ttu-id="fe27c-111">在 Azure 中预配 VM 涉及更多移动部件，而不只是 VM 本身。</span><span class="sxs-lookup"><span data-stu-id="fe27c-111">Provisioning a VM in Azure involves more moving parts than just the VM itself.</span></span> <span data-ttu-id="fe27c-112">需要考虑的因素有计算、网络和存储。</span><span class="sxs-lookup"><span data-stu-id="fe27c-112">There are compute, networking, and storage elements that you need to consider.</span></span>

* <span data-ttu-id="fe27c-113">**资源组。**</span><span class="sxs-lookup"><span data-stu-id="fe27c-113">**Resource group.**</span></span> <span data-ttu-id="fe27c-114">[resource-manager-overview]*资源组*][是保存相关资源的容器。</span><span class="sxs-lookup"><span data-stu-id="fe27c-114">A [*resource group*][resource-manager-overview] is a container that holds related resources.</span></span> <span data-ttu-id="fe27c-115">通常需基于解决方案中不同资源的生存期以及要管理资源的人员，为这些资源创建资源组。</span><span class="sxs-lookup"><span data-stu-id="fe27c-115">You usually create resource groups for different resources in a solution based on their lifetime, and who will manage the resources.</span></span> <span data-ttu-id="fe27c-116">对于单一 VM 工作负荷，可以为所有资源创建单个资源组。</span><span class="sxs-lookup"><span data-stu-id="fe27c-116">For a single VM workload, you may create a single resource group for all resources.</span></span>
* <span data-ttu-id="fe27c-117">**VM**。</span><span class="sxs-lookup"><span data-stu-id="fe27c-117">**VM**.</span></span> <span data-ttu-id="fe27c-118">Azure 支持运行大量常用的 Linux 分发，包括 CentOS、Debian、Red Hat Enterprise、Ubuntu 和 FreeBSD。</span><span class="sxs-lookup"><span data-stu-id="fe27c-118">Azure supports running various popular Linux distributions, including CentOS, Debian, Red Hat Enterprise, Ubuntu, and FreeBSD.</span></span> <span data-ttu-id="fe27c-119">有关详细信息，请参阅 [Azure 和 Linux][azure-linux]。</span><span class="sxs-lookup"><span data-stu-id="fe27c-119">For more information, see [Azure and Linux][azure-linux].</span></span> <span data-ttu-id="fe27c-120">可以基于已发布的映像列表或上传到 Azure Blob 存储的虚拟硬盘 (VHD) 文件预配 VM。</span><span class="sxs-lookup"><span data-stu-id="fe27c-120">You can provision a VM from a list of published images or from a virtual hard disk (VHD) file that you upload to Azure Blob storage.</span></span>
* <span data-ttu-id="fe27c-121">**OS 磁盘。**</span><span class="sxs-lookup"><span data-stu-id="fe27c-121">**OS disk.**</span></span> <span data-ttu-id="fe27c-122">OS 磁盘是存储在 [Azure 存储][azure-storage]中的一个 VHD。</span><span class="sxs-lookup"><span data-stu-id="fe27c-122">The OS disk is a VHD stored in [Azure Storage][azure-storage].</span></span> <span data-ttu-id="fe27c-123">这意味着它一直存在，即使主机出现故障，也是如此。</span><span class="sxs-lookup"><span data-stu-id="fe27c-123">That means it persists even if the host machine goes down.</span></span> <span data-ttu-id="fe27c-124">OS 磁盘是 `/dev/sda1`。</span><span class="sxs-lookup"><span data-stu-id="fe27c-124">The OS disk is `/dev/sda1`.</span></span>
* <span data-ttu-id="fe27c-125">**临时磁盘。**</span><span class="sxs-lookup"><span data-stu-id="fe27c-125">**Temporary disk.**</span></span> <span data-ttu-id="fe27c-126">使用临时磁盘创建 VM。</span><span class="sxs-lookup"><span data-stu-id="fe27c-126">The VM is created with a temporary disk.</span></span> <span data-ttu-id="fe27c-127">此磁盘存储在主机的物理驱动器上。</span><span class="sxs-lookup"><span data-stu-id="fe27c-127">This disk is stored on a physical drive on the host machine.</span></span> <span data-ttu-id="fe27c-128">它不保存在 Azure 存储中，并且在重新启动期间以及发生其他 VM 生命周期事件期间可能会被删除。</span><span class="sxs-lookup"><span data-stu-id="fe27c-128">It is *not* saved in Azure Storage and might be deleted during reboots and other VM lifecycle events.</span></span> <span data-ttu-id="fe27c-129">只使用此磁盘存储临时数据，如页面文件或交换文件。</span><span class="sxs-lookup"><span data-stu-id="fe27c-129">Use this disk only for temporary data, such as page or swap files.</span></span> <span data-ttu-id="fe27c-130">临时磁盘是 `/dev/sdb1`，并在 `/mnt/resource` 或 `/mnt` 中装入。</span><span class="sxs-lookup"><span data-stu-id="fe27c-130">The temporary disk is `/dev/sdb1` and is mounted at `/mnt/resource` or `/mnt`.</span></span>
* <span data-ttu-id="fe27c-131">**数据磁盘。**</span><span class="sxs-lookup"><span data-stu-id="fe27c-131">**Data disks.**</span></span> <span data-ttu-id="fe27c-132">[数据磁盘][data-disk]是用于保存应用程序数据的持久性 VHD。</span><span class="sxs-lookup"><span data-stu-id="fe27c-132">A [data disk][data-disk] is a persistent VHD used for application data.</span></span> <span data-ttu-id="fe27c-133">数据磁盘像 OS 磁盘一样，存储在 Azure 存储中。</span><span class="sxs-lookup"><span data-stu-id="fe27c-133">Data disks are stored in Azure Storage, like the OS disk.</span></span>
* <span data-ttu-id="fe27c-134">**虚拟网络 (VNet) 和子网。**</span><span class="sxs-lookup"><span data-stu-id="fe27c-134">**Virtual network (VNet) and subnet.**</span></span> <span data-ttu-id="fe27c-135">Azure 中的每个 VM 都部署在 VNet 中，后者进一步划分为多个子网。</span><span class="sxs-lookup"><span data-stu-id="fe27c-135">Every VM in Azure is deployed into a VNet that is further divided into subnets.</span></span>
* <span data-ttu-id="fe27c-136">**公共 IP 地址。**</span><span class="sxs-lookup"><span data-stu-id="fe27c-136">**Public IP address.**</span></span> <span data-ttu-id="fe27c-137">需要使用公共 IP 地址与 VM 通信 &mdash; 例如，通过 SSH。</span><span class="sxs-lookup"><span data-stu-id="fe27c-137">A public IP address is needed to communicate with the VM &mdash; for example, via SSH.</span></span>
* <span data-ttu-id="fe27c-138">**网络接口 (NIC)**。</span><span class="sxs-lookup"><span data-stu-id="fe27c-138">**Network interface (NIC)**.</span></span> <span data-ttu-id="fe27c-139">NIC 使 VM 能够与虚拟网络进行通信。</span><span class="sxs-lookup"><span data-stu-id="fe27c-139">The NIC enables the VM to communicate with the virtual network.</span></span>
* <span data-ttu-id="fe27c-140">**网络安全组 (NSG)**。</span><span class="sxs-lookup"><span data-stu-id="fe27c-140">**Network security group (NSG)**.</span></span> <span data-ttu-id="fe27c-141">[NSG][nsg] 用于允许/拒绝到子网的网络流量。</span><span class="sxs-lookup"><span data-stu-id="fe27c-141">The [NSG][nsg] is used to allow/deny network traffic to the subnet.</span></span> <span data-ttu-id="fe27c-142">可以将 NSG 与单个 NIC 或与子网相关联。</span><span class="sxs-lookup"><span data-stu-id="fe27c-142">You can associate an NSG with an individual NIC or with a subnet.</span></span> <span data-ttu-id="fe27c-143">如果将 NSG 与一个子网相关联，则 NSG 规则适用于该子网中的所有 VM。</span><span class="sxs-lookup"><span data-stu-id="fe27c-143">If you associate it with a subnet, the NSG rules apply to all VMs in that subnet.</span></span>
* <span data-ttu-id="fe27c-144">**诊断。**</span><span class="sxs-lookup"><span data-stu-id="fe27c-144">**Diagnostics.**</span></span> <span data-ttu-id="fe27c-145">诊断日志记录对于 VM 管理和故障排除至关重要。</span><span class="sxs-lookup"><span data-stu-id="fe27c-145">Diagnostic logging is crucial for managing and troubleshooting the VM.</span></span>

## <a name="recommendations"></a><span data-ttu-id="fe27c-146">建议</span><span class="sxs-lookup"><span data-stu-id="fe27c-146">Recommendations</span></span>

<span data-ttu-id="fe27c-147">此体系结构显示有关在 Azure 中运行 Linux VM 的基准建议。</span><span class="sxs-lookup"><span data-stu-id="fe27c-147">This architecture shows the baseline recommendations for running a Linux VM in Azure.</span></span> <span data-ttu-id="fe27c-148">但是，不建议针对任务关键型工作负荷使用单个 VM，因为这会产生单一故障点。</span><span class="sxs-lookup"><span data-stu-id="fe27c-148">However, we don't recommend using a single VM for mission critical workloads because it creates a single point of failure.</span></span> <span data-ttu-id="fe27c-149">为了提高可用性，请在[可用性集][availability-set]中部署多个 VM。</span><span class="sxs-lookup"><span data-stu-id="fe27c-149">For higher availability, deploy multiple VMs in an [availability set][availability-set].</span></span> <span data-ttu-id="fe27c-150">有关详细信息，请参阅 [Running multiple VMs on Azure][multi-vm]（在 Azure 上运行多个 VM）。</span><span class="sxs-lookup"><span data-stu-id="fe27c-150">For more information, see [Running multiple VMs on Azure][multi-vm].</span></span> 

### <a name="vm-recommendations"></a><span data-ttu-id="fe27c-151">VM 建议</span><span class="sxs-lookup"><span data-stu-id="fe27c-151">VM recommendations</span></span>

<span data-ttu-id="fe27c-152">Azure 可提供多种虚拟机大小，但建议使用 DS 和 GS 系列，因为相关计算机大小支持[高级存储][premium-storage]。</span><span class="sxs-lookup"><span data-stu-id="fe27c-152">Azure offers many different virtual machine sizes, but we recommend the DS- and GS-series because these machine sizes support [Premium Storage][premium-storage].</span></span> <span data-ttu-id="fe27c-153">除非运行专用工作负荷（例如高性能计算），否则请选择其中的一种计算机大小。</span><span class="sxs-lookup"><span data-stu-id="fe27c-153">Select one of these machine sizes unless you have a specialized workload such as high-performance computing.</span></span> <span data-ttu-id="fe27c-154">有关详细信息，请参阅[虚拟机大小][virtual-machine-sizes]。</span><span class="sxs-lookup"><span data-stu-id="fe27c-154">For details, see [virtual machine sizes][virtual-machine-sizes].</span></span>

<span data-ttu-id="fe27c-155">如果要将现有工作负荷转移到 Azure，开始时请先使用与本地服务器最匹配的 VM 大小。</span><span class="sxs-lookup"><span data-stu-id="fe27c-155">If you are moving an existing workload to Azure, start with the VM size that's the closest match to your on-premises servers.</span></span> <span data-ttu-id="fe27c-156">然后测量与 CPU、内存和每秒磁盘输入/输出操作次数 (IOPS) 有关的实际工作负荷的性能，并根据需要调整大小。</span><span class="sxs-lookup"><span data-stu-id="fe27c-156">Then measure the performance of your actual workload with respect to CPU, memory, and disk input/output operations per second (IOPS), and adjust the size if needed.</span></span> <span data-ttu-id="fe27c-157">如果 VM 需要多个 NIC，请注意 NIC 的最大数量取决于 [VM 大小][vm-size-tables]。</span><span class="sxs-lookup"><span data-stu-id="fe27c-157">If you require multiple NICs for your VM, be aware that the maximum number of NICs is a function of the [VM size][vm-size-tables].</span></span>

<span data-ttu-id="fe27c-158">在预配 VM 和其他资源时，必须指定区域。</span><span class="sxs-lookup"><span data-stu-id="fe27c-158">When you provision the VM and other resources, you must specify a region.</span></span> <span data-ttu-id="fe27c-159">通常应选择离内部用户或客户最近的区域。</span><span class="sxs-lookup"><span data-stu-id="fe27c-159">Generally, choose a region closest to your internal users or customers.</span></span> <span data-ttu-id="fe27c-160">但是，并非所有 VM 大小都可在所有区域中使用。</span><span class="sxs-lookup"><span data-stu-id="fe27c-160">However, not all VM sizes may be available in all region.</span></span> <span data-ttu-id="fe27c-161">有关详细信息，请参阅[服务（按区域）][services-by-region]。</span><span class="sxs-lookup"><span data-stu-id="fe27c-161">For details, see [Services by region][services-by-region].</span></span> <span data-ttu-id="fe27c-162">若要列出特定区域中可用的 VM 大小，请运行以下 Azure 命令行接口 (CLI) 命令：</span><span class="sxs-lookup"><span data-stu-id="fe27c-162">To list the VM sizes available in a specific region, run the following Azure command-line interface (CLI) command:</span></span>

```
az vm list-sizes --location <location>
```

<span data-ttu-id="fe27c-163">若要了解如何选择发布的 VM 映像，请参阅[利用 Azure CLI 选择 Linux VM 映像][select-vm-image]。</span><span class="sxs-lookup"><span data-stu-id="fe27c-163">For information about choosing a published VM image, see [Select Linux VM images with the Azure CLI][select-vm-image].</span></span>

<span data-ttu-id="fe27c-164">启用监视和诊断，包括基本运行状况指标、诊断基础结构日志和[启动诊断][boot-diagnostics]。</span><span class="sxs-lookup"><span data-stu-id="fe27c-164">Enable monitoring and diagnostics, including basic health metrics, diagnostics infrastructure logs, and [boot diagnostics][boot-diagnostics].</span></span> <span data-ttu-id="fe27c-165">如果 VM 陷入不可启动状态，启动诊断有助于诊断启动故障。</span><span class="sxs-lookup"><span data-stu-id="fe27c-165">Boot diagnostics can help you diagnose boot failure if your VM gets into a non-bootable state.</span></span> <span data-ttu-id="fe27c-166">有关详细信息，请参阅[启用监视和诊断][enable-monitoring]。</span><span class="sxs-lookup"><span data-stu-id="fe27c-166">For more information, see [Enable monitoring and diagnostics][enable-monitoring].</span></span>  

### <a name="disk-and-storage-recommendations"></a><span data-ttu-id="fe27c-167">磁盘和存储建议</span><span class="sxs-lookup"><span data-stu-id="fe27c-167">Disk and storage recommendations</span></span>

<span data-ttu-id="fe27c-168">为获得最佳磁盘 I/O 性能，建议使用[高级存储][premium-storage]，它在固态硬盘 (SSD) 上存储数据。</span><span class="sxs-lookup"><span data-stu-id="fe27c-168">For best disk I/O performance, we recommend [Premium Storage][premium-storage], which stores data on solid-state drives (SSDs).</span></span> <span data-ttu-id="fe27c-169">成本取决于预配磁盘的大小。</span><span class="sxs-lookup"><span data-stu-id="fe27c-169">Cost is based on the size of the provisioned disk.</span></span> <span data-ttu-id="fe27c-170">IOPS 和吞吐量（即数据传输速率）也取决于磁盘大小，因此在预配磁盘时，请全面考虑三个因素（容量、IOPS 和吞吐量）。</span><span class="sxs-lookup"><span data-stu-id="fe27c-170">IOPS and throughput (that is, data transfer rate) also depend on disk size, so when you provision a disk, consider all three factors (capacity, IOPS, and throughput).</span></span> 

<span data-ttu-id="fe27c-171">我们还建议使用[托管磁盘](/azure/storage/storage-managed-disks-overview)。</span><span class="sxs-lookup"><span data-stu-id="fe27c-171">We also recommend the use of [managed disks](/azure/storage/storage-managed-disks-overview).</span></span> <span data-ttu-id="fe27c-172">托管磁盘不需要存储帐户。</span><span class="sxs-lookup"><span data-stu-id="fe27c-172">Managed disks do not require a storage account.</span></span> <span data-ttu-id="fe27c-173">只需指定磁盘的大小和类型，它就会以高度可用的方式部署。</span><span class="sxs-lookup"><span data-stu-id="fe27c-173">You simply specify the size and type of disk and it is deployed in a highly available way.</span></span>

<span data-ttu-id="fe27c-174">如果不使用托管磁盘，请为每个 VM 创建单独的 Azure 存储帐户来存放虚拟硬盘 (VHD)，以避免达到存储帐户的 IOPS 限制。</span><span class="sxs-lookup"><span data-stu-id="fe27c-174">If you are not using managed disks, create separate Azure storage accounts for each VM to hold the virtual hard disks (VHDs) in order to avoid hitting the IOPS limits for storage accounts.</span></span>

<span data-ttu-id="fe27c-175">添加一个或多个数据磁盘。</span><span class="sxs-lookup"><span data-stu-id="fe27c-175">Add one or more data disks.</span></span> <span data-ttu-id="fe27c-176">刚创建的 VHD 尚未格式化，</span><span class="sxs-lookup"><span data-stu-id="fe27c-176">When you create a VHD, it is unformatted.</span></span> <span data-ttu-id="fe27c-177">请登录 VM 格式化该磁盘。</span><span class="sxs-lookup"><span data-stu-id="fe27c-177">Log in to the VM to format the disk.</span></span> <span data-ttu-id="fe27c-178">在 Linux shell 中，数据磁盘显示为 `/dev/sdc`、`/dev/sdd` 等。</span><span class="sxs-lookup"><span data-stu-id="fe27c-178">In the Linux shell, data disks are displayed as `/dev/sdc`, `/dev/sdd`, and so on.</span></span> <span data-ttu-id="fe27c-179">可以运行 `lsblk` 以列出块设备，包括磁盘。</span><span class="sxs-lookup"><span data-stu-id="fe27c-179">You can run `lsblk` to list the block devices, including the disks.</span></span> <span data-ttu-id="fe27c-180">要使用数据磁盘，请创建一个分区和文件系统，并装载磁盘。</span><span class="sxs-lookup"><span data-stu-id="fe27c-180">To use a data disk, create a partition and file system, and mount the disk.</span></span> <span data-ttu-id="fe27c-181">例如：</span><span class="sxs-lookup"><span data-stu-id="fe27c-181">For example:</span></span>

```bat
# Create a partition.
sudo fdisk /dev/sdc     # Enter 'n' to partition, 'w' to write the change.

# Create a file system.
sudo mkfs -t ext3 /dev/sdc1

# Mount the drive.
sudo mkdir /data1
sudo mount /dev/sdc1 /data1
```

<span data-ttu-id="fe27c-182">如果不使用[托管磁盘](/azure/storage/storage-managed-disks-overview)并且有大量数据磁盘，请注意存储帐户的总 I/O 限制。</span><span class="sxs-lookup"><span data-stu-id="fe27c-182">If you are not using [managed disks](/azure/storage/storage-managed-disks-overview) and have a large number of data disks, be aware of the total I/O limits of the storage account.</span></span> <span data-ttu-id="fe27c-183">有关详细信息，请参阅[虚拟机磁盘限制][vm-disk-limits]。</span><span class="sxs-lookup"><span data-stu-id="fe27c-183">For more information, see [virtual machine disk limits][vm-disk-limits].</span></span>

<span data-ttu-id="fe27c-184">在添加数据磁盘时，将为磁盘分配逻辑单元号 (LUN) ID。</span><span class="sxs-lookup"><span data-stu-id="fe27c-184">When you add a data disk, a logical unit number (LUN) ID is assigned to the disk.</span></span> <span data-ttu-id="fe27c-185">或者，可以指定 LUN ID &mdash; 例如，若要更换磁盘并保留相同的 LUN ID，或者应用程序要查找特定 LUN ID。</span><span class="sxs-lookup"><span data-stu-id="fe27c-185">Optionally, you can specify the LUN ID &mdash; for example, if you're replacing a disk and want to retain the same LUN ID, or you have an application that looks for a specific LUN ID.</span></span> <span data-ttu-id="fe27c-186">但请记住，每个磁盘的 LUN ID 必须唯一。</span><span class="sxs-lookup"><span data-stu-id="fe27c-186">However, remember that LUN IDs must be unique for each disk.</span></span>

<span data-ttu-id="fe27c-187">可能会需要更改 I/O 计划程序，以便针对 SSD 的性能进行优化，因为使用高级存储帐户的 VM 的磁盘是 SSD。</span><span class="sxs-lookup"><span data-stu-id="fe27c-187">You may want to change the I/O scheduler to optimize for performance on SSDs because the disks for VMs with premium storage accounts are SSDs.</span></span> <span data-ttu-id="fe27c-188">常见的建议是对 SSD 使用 NOOP 计划程序，但应使用 [iostat] 等工具来监视工作负荷的磁盘 I/O 性能。</span><span class="sxs-lookup"><span data-stu-id="fe27c-188">A common recommendation is to use the NOOP scheduler for SSDs, but you should use a tool such as [iostat] to monitor disk I/O performance for your workload.</span></span>

<span data-ttu-id="fe27c-189">为获得最佳性能，请创建单独的存储帐户来存储诊断日志。</span><span class="sxs-lookup"><span data-stu-id="fe27c-189">For best performance, create a separate storage account to hold diagnostic logs.</span></span> <span data-ttu-id="fe27c-190">标准的本地冗余存储 (LRS) 帐户足以存储诊断日志。</span><span class="sxs-lookup"><span data-stu-id="fe27c-190">A standard locally redundant storage (LRS) account is sufficient for diagnostic logs.</span></span>

### <a name="network-recommendations"></a><span data-ttu-id="fe27c-191">网络建议</span><span class="sxs-lookup"><span data-stu-id="fe27c-191">Network recommendations</span></span>

<span data-ttu-id="fe27c-192">公共 IP 地址可以是动态的或静态的。</span><span class="sxs-lookup"><span data-stu-id="fe27c-192">The public IP address can be dynamic or static.</span></span> <span data-ttu-id="fe27c-193">默认是动态的。</span><span class="sxs-lookup"><span data-stu-id="fe27c-193">The default is dynamic.</span></span>

* <span data-ttu-id="fe27c-194">如果需要不会更改的固定 IP 地址（例如，如果需要在 DNS 中创建 A 记录，或者需要将 IP 地址添加到安全列表），请保留[静态 IP 地址][static-ip]。</span><span class="sxs-lookup"><span data-stu-id="fe27c-194">Reserve a [static IP address][static-ip] if you need a fixed IP address that won't change &mdash; for example, if you need to create an A record in DNS, or need the IP address to be added to a safe list.</span></span>
* <span data-ttu-id="fe27c-195">还可以为 IP 地址创建完全限定域名 (FQDN)。</span><span class="sxs-lookup"><span data-stu-id="fe27c-195">You can also create a fully qualified domain name (FQDN) for the IP address.</span></span> <span data-ttu-id="fe27c-196">然后，可在 DNS 中注册指向 FQDN 的 [CNAME 记录][cname-record]。</span><span class="sxs-lookup"><span data-stu-id="fe27c-196">You can then register a [CNAME record][cname-record] in DNS that points to the FQDN.</span></span> <span data-ttu-id="fe27c-197">有关详细信息，请参阅[在 Azure 门户中创建完全限定的域名][fqdn]。</span><span class="sxs-lookup"><span data-stu-id="fe27c-197">For more information, see [create a fully qualified domain name in the Azure portal][fqdn].</span></span>

<span data-ttu-id="fe27c-198">所有 NSG 都包含一组[默认规则][nsg-default-rules]，其中包括阻止所有入站 Internet 流量的规则。</span><span class="sxs-lookup"><span data-stu-id="fe27c-198">All NSGs contain a set of [default rules][nsg-default-rules], including a rule that blocks all inbound Internet traffic.</span></span> <span data-ttu-id="fe27c-199">无法删除默认规则，但其他规则可以覆盖它们。</span><span class="sxs-lookup"><span data-stu-id="fe27c-199">The default rules cannot be deleted, but other rules can override them.</span></span> <span data-ttu-id="fe27c-200">要启用 Internet 流量，请创建允许特定端口的入站流量的规则 &mdash; 例如，将端口 80 用于 HTTP。</span><span class="sxs-lookup"><span data-stu-id="fe27c-200">To enable Internet traffic, create rules that allow inbound traffic to specific ports &mdash; for example, port 80 for HTTP.</span></span>

<span data-ttu-id="fe27c-201">若要启用 SSH，请向 NSG 添加允许 TCP 端口 22 的入站流量的规则。</span><span class="sxs-lookup"><span data-stu-id="fe27c-201">To enable SSH, add a rule to the NSG that allows inbound traffic to TCP port 22.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="fe27c-202">可伸缩性注意事项</span><span class="sxs-lookup"><span data-stu-id="fe27c-202">Scalability considerations</span></span>

<span data-ttu-id="fe27c-203">可以通过[更改 VM 大小][vm-resize]来扩展或缩小 VM。</span><span class="sxs-lookup"><span data-stu-id="fe27c-203">You can scale a VM up or down by [changing the VM size][vm-resize].</span></span> <span data-ttu-id="fe27c-204">要以水平方式横向扩展，请将两个或更多 VM 放在负载均衡器后面。</span><span class="sxs-lookup"><span data-stu-id="fe27c-204">To scale out horizontally, put two or more VMs behind a load balancer.</span></span> <span data-ttu-id="fe27c-205">有关详细信息，请参阅[在 Azure 上运行多个 VM 以实现可伸缩性和可用性][multi-vm]。</span><span class="sxs-lookup"><span data-stu-id="fe27c-205">For details, see [Running multiple VMs on Azure for scalability and availability][multi-vm].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="fe27c-206">可用性注意事项</span><span class="sxs-lookup"><span data-stu-id="fe27c-206">Availability considerations</span></span>

<span data-ttu-id="fe27c-207">为了提高可用性，请在可用性集中部署多个 VM。</span><span class="sxs-lookup"><span data-stu-id="fe27c-207">For higher availability, deploy multiple VMs in an availability set.</span></span> <span data-ttu-id="fe27c-208">这样还可提供更高的[服务级别协议][vm-sla] (SLA)。</span><span class="sxs-lookup"><span data-stu-id="fe27c-208">This also provides a higher [service level agreement][vm-sla]  (SLA).</span></span>

<span data-ttu-id="fe27c-209">VM 可能会受到[计划内维护][planned-maintenance]或[计划外维护][manage-vm-availability]的影响。</span><span class="sxs-lookup"><span data-stu-id="fe27c-209">Your VM may be affected by [planned maintenance][planned-maintenance] or [unplanned maintenance][manage-vm-availability].</span></span> <span data-ttu-id="fe27c-210">可以使用 [VM 重新启动日志][reboot-logs]来确定 VM 重新启动是否是由计划内维护导致的。</span><span class="sxs-lookup"><span data-stu-id="fe27c-210">You can use [VM reboot logs][reboot-logs] to determine whether a VM reboot was caused by planned maintenance.</span></span>

<span data-ttu-id="fe27c-211">VHD 存储在 [Azure 存储][azure-storage]中，Azure 存储会复制，实现持久性和可用性。</span><span class="sxs-lookup"><span data-stu-id="fe27c-211">VHDs are stored in [Azure storage][azure-storage], and Azure storage is replicated for durability and availability.</span></span>

<span data-ttu-id="fe27c-212">若要防止在正常操作期间意外数据丢失（例如，由于用户错误），则还应使用 [Blob 快照][blob-snapshot]或其他工具实现时间点备份。</span><span class="sxs-lookup"><span data-stu-id="fe27c-212">To protect against accidental data loss during normal operations (for example, because of user error), you should also implement point-in-time backups, using [blob snapshots][blob-snapshot] or another tool.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="fe27c-213">可管理性注意事项</span><span class="sxs-lookup"><span data-stu-id="fe27c-213">Manageability considerations</span></span>

<span data-ttu-id="fe27c-214">**资源组。**</span><span class="sxs-lookup"><span data-stu-id="fe27c-214">**Resource groups.**</span></span> <span data-ttu-id="fe27c-215">将共享相同生命周期的紧密耦合资源放入同一[资源组][resource-manager-overview]中。</span><span class="sxs-lookup"><span data-stu-id="fe27c-215">Put tightly coupled resources that share the same life cycle into the same [resource group][resource-manager-overview].</span></span> <span data-ttu-id="fe27c-216">资源组可让你以组的形式部署和监视资源，并按资源组汇总计费成本。</span><span class="sxs-lookup"><span data-stu-id="fe27c-216">Resource groups allow you to deploy and monitor resources as a group, and roll up billing costs by resource group.</span></span> <span data-ttu-id="fe27c-217">还可以删除作为集的资源，这对于测试部署非常有用。</span><span class="sxs-lookup"><span data-stu-id="fe27c-217">You can also delete resources as a set, which is very useful for test deployments.</span></span> <span data-ttu-id="fe27c-218">为资源指定有意义的名称。</span><span class="sxs-lookup"><span data-stu-id="fe27c-218">Give resources meaningful names.</span></span> <span data-ttu-id="fe27c-219">这样，可更轻松地找到特定资源并了解其角色。</span><span class="sxs-lookup"><span data-stu-id="fe27c-219">That makes it easier to locate a specific resource and understand its role.</span></span> <span data-ttu-id="fe27c-220">请参阅 [Azure 资源的建议命名约定][naming conventions]。</span><span class="sxs-lookup"><span data-stu-id="fe27c-220">See [Recommended Naming Conventions for Azure Resources][naming conventions].</span></span>

<span data-ttu-id="fe27c-221">**SSH**。</span><span class="sxs-lookup"><span data-stu-id="fe27c-221">**SSH**.</span></span> <span data-ttu-id="fe27c-222">在创建 Linux VM 之前，生成 2048 位 RSA 公共/专用密钥对。</span><span class="sxs-lookup"><span data-stu-id="fe27c-222">Before you create a Linux VM, generate a 2048-bit RSA public-private key pair.</span></span> <span data-ttu-id="fe27c-223">创建 VM 时，使用公钥文件。</span><span class="sxs-lookup"><span data-stu-id="fe27c-223">Use the public key file when you create the VM.</span></span> <span data-ttu-id="fe27c-224">有关详细信息，请参阅[如何在 Azure 中将 SSH 与 Linux 和 Mac 配合使用][ssh-linux]。</span><span class="sxs-lookup"><span data-stu-id="fe27c-224">For more information, see [How to Use SSH with Linux and Mac on Azure][ssh-linux].</span></span>

<span data-ttu-id="fe27c-225">**停止 VM。**</span><span class="sxs-lookup"><span data-stu-id="fe27c-225">**Stopping a VM.**</span></span> <span data-ttu-id="fe27c-226">Azure 对“已停止”和“已解除分配”状态做了区分。</span><span class="sxs-lookup"><span data-stu-id="fe27c-226">Azure makes a distinction between "stopped" and "deallocated" states.</span></span> <span data-ttu-id="fe27c-227">当 VM 状态为已停止时（而不是当 VM 已解除分配时）将向你收费。</span><span class="sxs-lookup"><span data-stu-id="fe27c-227">You are charged when the VM status is stopped, but not when the VM is deallocated.</span></span> 

<span data-ttu-id="fe27c-228">在 Azure 门户中，“停止”按钮可解除分配 VM。</span><span class="sxs-lookup"><span data-stu-id="fe27c-228">In the Azure portal, the **Stop** button deallocates the VM.</span></span> <span data-ttu-id="fe27c-229">如果在已登录时通过 OS 关闭，VM 会停止，但不会解除分配，因此仍会产生费用。</span><span class="sxs-lookup"><span data-stu-id="fe27c-229">If you shut down through the OS while logged in, the VM is stopped but *not* deallocated, so you will still be charged.</span></span> 

<span data-ttu-id="fe27c-230">**删除 VM。**</span><span class="sxs-lookup"><span data-stu-id="fe27c-230">**Deleting a VM.**</span></span> <span data-ttu-id="fe27c-231">如果删除 VM，则不会删除 VHD。</span><span class="sxs-lookup"><span data-stu-id="fe27c-231">If you delete a VM, the VHDs are not deleted.</span></span> <span data-ttu-id="fe27c-232">这意味着可以安全地删除 VM，而不会丢失数据。</span><span class="sxs-lookup"><span data-stu-id="fe27c-232">That means you can safely delete the VM without losing data.</span></span> <span data-ttu-id="fe27c-233">但是，仍将向你收取存储费用。</span><span class="sxs-lookup"><span data-stu-id="fe27c-233">However, you will still be charged for storage.</span></span> <span data-ttu-id="fe27c-234">若要删除 VHD，请从 [Blob 存储][blob-storage]中删除相应的文件。</span><span class="sxs-lookup"><span data-stu-id="fe27c-234">To delete the VHD, delete the file from [Blob storage][blob-storage].</span></span>

<span data-ttu-id="fe27c-235">若要防止意外删除，请使用[资源锁][resource-lock]锁定整个资源组或锁定单个资源（如 VM）。</span><span class="sxs-lookup"><span data-stu-id="fe27c-235">To prevent accidental deletion, use a [resource lock][resource-lock] to lock the entire resource group or lock individual resources, such as the VM.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="fe27c-236">安全注意事项</span><span class="sxs-lookup"><span data-stu-id="fe27c-236">Security considerations</span></span>

<span data-ttu-id="fe27c-237">使用 [Azure 安全中心][security-center]可在一个中心视图中获得 Azure 资源的安全状态。</span><span class="sxs-lookup"><span data-stu-id="fe27c-237">Use [Azure Security Center][security-center] to get a central view of the security state of your Azure resources.</span></span> <span data-ttu-id="fe27c-238">安全中心监视潜在的安全问题，并全面描述了部署的安全运行状况。</span><span class="sxs-lookup"><span data-stu-id="fe27c-238">Security Center monitors potential security issues and provides a comprehensive picture of the security health of your deployment.</span></span> <span data-ttu-id="fe27c-239">安全中心针对每个 Azure 订阅进行配置。</span><span class="sxs-lookup"><span data-stu-id="fe27c-239">Security Center is configured per Azure subscription.</span></span> <span data-ttu-id="fe27c-240">启用安全数据收集，如 [Azure 安全中心快速入门指南][security-center-get-started]中所述。</span><span class="sxs-lookup"><span data-stu-id="fe27c-240">Enable security data collection as described in [Azure Security Center quick start guide][security-center-get-started].</span></span> <span data-ttu-id="fe27c-241">启用数据收集后，安全中心会自动扫描该订阅下创建的所有 VM。</span><span class="sxs-lookup"><span data-stu-id="fe27c-241">When data collection is enabled, Security Center automatically scans any VMs created under that subscription.</span></span>

<span data-ttu-id="fe27c-242">**修补程序管理。**</span><span class="sxs-lookup"><span data-stu-id="fe27c-242">**Patch management.**</span></span> <span data-ttu-id="fe27c-243">如果启用，安全中心会检查是否缺少安全更新和关键更新。</span><span class="sxs-lookup"><span data-stu-id="fe27c-243">If enabled, Security Center checks whether security and critical updates are missing.</span></span> 

<span data-ttu-id="fe27c-244">**反恶意软件。**</span><span class="sxs-lookup"><span data-stu-id="fe27c-244">**Antimalware.**</span></span> <span data-ttu-id="fe27c-245">如果启用，安全中心会检查是否已安装反恶意软件。</span><span class="sxs-lookup"><span data-stu-id="fe27c-245">If enabled, Security Center checks whether antimalware software is installed.</span></span> <span data-ttu-id="fe27c-246">还可以使用安全中心从 Azure 门户中安装反恶意软件。</span><span class="sxs-lookup"><span data-stu-id="fe27c-246">You can also use Security Center to install antimalware software from inside the Azure portal.</span></span>

<span data-ttu-id="fe27c-247">**操作。**</span><span class="sxs-lookup"><span data-stu-id="fe27c-247">**Operations.**</span></span> <span data-ttu-id="fe27c-248">使用[基于角色的访问控制][rbac] (RBAC) 来控制对部署的 Azure 资源的访问。</span><span class="sxs-lookup"><span data-stu-id="fe27c-248">Use [role-based access control][rbac] (RBAC) to control access to the Azure resources that you deploy.</span></span> <span data-ttu-id="fe27c-249">RBAC 允许将授权角色分配给开发运营团队的成员。</span><span class="sxs-lookup"><span data-stu-id="fe27c-249">RBAC lets you assign authorization roles to members of your DevOps team.</span></span> <span data-ttu-id="fe27c-250">例如，“读者”角色可以查看 Azure 资源，但不能创建、管理或删除这些资源。</span><span class="sxs-lookup"><span data-stu-id="fe27c-250">For example, the Reader role can view Azure resources but not create, manage, or delete them.</span></span> <span data-ttu-id="fe27c-251">某些角色特定于特定的 Azure 资源类型。</span><span class="sxs-lookup"><span data-stu-id="fe27c-251">Some roles are specific to particular Azure resource types.</span></span> <span data-ttu-id="fe27c-252">例如，“虚拟机参与者”角色可以执行重启或解除分配 VM、重置管理员密码、创建新的 VM 等操作。</span><span class="sxs-lookup"><span data-stu-id="fe27c-252">For example, the Virtual Machine Contributor role can restart or deallocate a VM, reset the administrator password, create a new VM, and so forth.</span></span> <span data-ttu-id="fe27c-253">可能对此体系结构有用的其他[内置 RBAC 角色][rbac-roles]包括[开发测试实验室用户][rbac-devtest]和[网络参与者][rbac-network]。</span><span class="sxs-lookup"><span data-stu-id="fe27c-253">Other [built-in RBAC roles][rbac-roles] that might be useful for this architecture include [DevTest Labs User][rbac-devtest] and [Network Contributor][rbac-network].</span></span> <span data-ttu-id="fe27c-254">可将用户分配给多个角色，并且可以创建自定义角色以实现更细化的权限。</span><span class="sxs-lookup"><span data-stu-id="fe27c-254">A user can be assigned to multiple roles, and you can create custom roles for even more fine-grained permissions.</span></span>

> [!NOTE]
> <span data-ttu-id="fe27c-255">RBAC 不限制已登录到 VM 的用户可以执行的操作。</span><span class="sxs-lookup"><span data-stu-id="fe27c-255">RBAC does not limit the actions that a user logged into a VM can perform.</span></span> <span data-ttu-id="fe27c-256">这些权限由来宾 OS 上的帐户类型决定。</span><span class="sxs-lookup"><span data-stu-id="fe27c-256">Those permissions are determined by the account type on the guest OS.</span></span>   

<span data-ttu-id="fe27c-257">使用[审核日志][audit-logs]可查看预配操作和其他 VM 事件。</span><span class="sxs-lookup"><span data-stu-id="fe27c-257">Use [audit logs][audit-logs] to see provisioning actions and other VM events.</span></span>

<span data-ttu-id="fe27c-258">**数据加密。**</span><span class="sxs-lookup"><span data-stu-id="fe27c-258">**Data encryption.**</span></span> <span data-ttu-id="fe27c-259">如果需要加密 OS 磁盘和数据磁盘，请考虑使用 [Azure 磁盘加密][disk-encryption]。</span><span class="sxs-lookup"><span data-stu-id="fe27c-259">Consider [Azure Disk Encryption][disk-encryption] if you need to encrypt the OS and data disks.</span></span> 

## <a name="deploy-the-solution"></a><span data-ttu-id="fe27c-260">部署解决方案</span><span class="sxs-lookup"><span data-stu-id="fe27c-260">Deploy the solution</span></span>

<span data-ttu-id="fe27c-261">[GitHub][github-folder] 上提供了此体系结构的部署。</span><span class="sxs-lookup"><span data-stu-id="fe27c-261">A deployment for this architecture is available on [GitHub][github-folder].</span></span> <span data-ttu-id="fe27c-262">它将部署以下部分：</span><span class="sxs-lookup"><span data-stu-id="fe27c-262">It deploys the following:</span></span>

  * <span data-ttu-id="fe27c-263">一个虚拟网络，其中包含用于托管 VM 的名为 **web** 的单个子网。</span><span class="sxs-lookup"><span data-stu-id="fe27c-263">A virtual network with a single subnet named **web** used to host the VM.</span></span>
  * <span data-ttu-id="fe27c-264">一个 NSG，其中包含两个用于允许 SSH 和 HTTP 流量发送到 VM 的传入规则。</span><span class="sxs-lookup"><span data-stu-id="fe27c-264">An NSG with two incoming rules to allow SSH and HTTP traffic to the VM.</span></span>
  * <span data-ttu-id="fe27c-265">一个运行 Ubuntu 16.04.3 LTS 最新版本的 VM。</span><span class="sxs-lookup"><span data-stu-id="fe27c-265">A VM running the latest version of Ubuntu 16.04.3 LTS.</span></span>
  * <span data-ttu-id="fe27c-266">一个示例自定义脚本扩展，它将 Apache HTTP Server 部署到 Ubuntu VM 并格式化两个数据磁盘。</span><span class="sxs-lookup"><span data-stu-id="fe27c-266">A sample custom script extension that deploys Apache HTTP Server to the Ubuntu VM, and formats the two data disks.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="fe27c-267">先决条件</span><span class="sxs-lookup"><span data-stu-id="fe27c-267">Prerequisites</span></span>

<span data-ttu-id="fe27c-268">在将参考体系结构部署到自己的订阅之前，必须执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="fe27c-268">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="fe27c-269">为 [AzureCAT 参考体系结构][ref-arch-repo] GitHub 存储库克隆、下载 zip 文件或创建其分支。</span><span class="sxs-lookup"><span data-stu-id="fe27c-269">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="fe27c-270">确保在计算机上安装了 Azure CLI 2.0。</span><span class="sxs-lookup"><span data-stu-id="fe27c-270">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="fe27c-271">若要安装 CLI，请按照[安装 Azure CLI 2.0][azure-cli-2] 中的说明执行操作。</span><span class="sxs-lookup"><span data-stu-id="fe27c-271">To install the CLI, follow the instructions in [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="fe27c-272">安装 [Azure 构建基块][azbb] npm 包。</span><span class="sxs-lookup"><span data-stu-id="fe27c-272">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="fe27c-273">从命令提示符、bash 提示符或 PowerShell 提示符下通过使用以下命令之一，登录到 Azure 帐户，然后按照提示进行操作。</span><span class="sxs-lookup"><span data-stu-id="fe27c-273">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using one of the commands below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a><span data-ttu-id="fe27c-274">使用 azbb 部署解决方案</span><span class="sxs-lookup"><span data-stu-id="fe27c-274">Deploy the solution using azbb</span></span>

<span data-ttu-id="fe27c-275">若要部署示例单一 VM 工作负荷，请执行下列步骤：</span><span class="sxs-lookup"><span data-stu-id="fe27c-275">To deploy the sample single VM workload, follow these steps:</span></span>

1. <span data-ttu-id="fe27c-276">导航到已在前面的先决条件步骤中下载的存储库的 `virtual-machines\single-vm\parameters\linux` 文件夹。</span><span class="sxs-lookup"><span data-stu-id="fe27c-276">Navigate to the `virtual-machines\single-vm\parameters\linux` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="fe27c-277">打开 `single-vm-v2.json` 文件并在引号之间输入用户名和 SSH 密钥，如下所示，然后保存文件。</span><span class="sxs-lookup"><span data-stu-id="fe27c-277">Open the `single-vm-v2.json` file and enter a username and SSH key between the quotes, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "",
  "adminsshPublicKey": "",
  ```

3. <span data-ttu-id="fe27c-278">运行 `azbb` 以部署示例 VM，如下所示。</span><span class="sxs-lookup"><span data-stu-id="fe27c-278">Run `azbb` to deploy the sample VM as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
  ```

<span data-ttu-id="fe27c-279">有关部署此示例参考体系结构的详细信息，请访问 [GitHub 存储库][git]。</span><span class="sxs-lookup"><span data-stu-id="fe27c-279">For more information on deploying this sample reference architecture, visit our [GitHub repository][git].</span></span>

## <a name="next-steps"></a><span data-ttu-id="fe27c-280">后续步骤</span><span class="sxs-lookup"><span data-stu-id="fe27c-280">Next steps</span></span>

- <span data-ttu-id="fe27c-281">了解有关 [Azure 构建基块][azbbv2]的信息。</span><span class="sxs-lookup"><span data-stu-id="fe27c-281">Learn about our [Azure building Blocks][azbbv2].</span></span>
- <span data-ttu-id="fe27c-282">在 Azure 中部署[多个 VM][multi-vm]。</span><span class="sxs-lookup"><span data-stu-id="fe27c-282">Deploy [multiple VMs][multi-vm] in Azure.</span></span>

<!-- links -->
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[multi-vm]: ../virtual-machines-linux/multi-vm.md
[naming conventions]: ../../best-practices/naming-conventions.md
[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[availability-set]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azbbv2]: https://github.com/mspnp/template-building-blocks
[azure-cli-2]: /cli/azure/install-azure-cli?view=azure-cli-latest
[azure-linux]: /azure/virtual-machines/virtual-machines-linux-azure-overview
[azure-storage]: /azure/storage/storage-introduction
[blob-snapshot]: /azure/storage/storage-blob-snapshots
[blob-storage]: /azure/storage/storage-introduction
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: /azure/virtual-machines/virtual-machines-linux-about-disks-vhds
[disk-encryption]: /azure/security/azure-security-disk-encryption
[enable-monitoring]: /azure/monitoring-and-diagnostics/insights-how-to-use-diagnostics
[fqdn]: /azure/virtual-machines/virtual-machines-linux-portal-create-fqdn
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[iostat]: https://en.wikipedia.org/wiki/Iostat
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-default-rules]: /azure/virtual-network/virtual-networks-nsg#default-rules
[OSPatching]: https://github.com/Azure/azure-linux-extensions/tree/master/OSPatching
[planned-maintenance]: /azure/virtual-machines/virtual-machines-linux-planned-maintenance
[premium-storage]: /azure/storage/storage-premium-storage
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rbac-roles]: /azure/active-directory/role-based-access-built-in-roles
[rbac-devtest]: /azure/active-directory/role-based-access-built-in-roles#devtest-labs-user
[rbac-network]: /azure/active-directory/role-based-access-built-in-roles#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resource-lock]: /azure/resource-group-lock-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[security-center]: /azure/security-center/
[security-center-get-started]: /azure/security-center/security-center-get-started
[select-vm-image]: /azure/virtual-machines/virtual-machines-linux-cli-ps-findimage
[services-by-region]: https://azure.microsoft.com/regions/#services
[ssh-linux]: /azure/virtual-machines/virtual-machines-linux-mac-create-ssh-keys
[static-ip]: /azure/virtual-network/virtual-networks-reserved-public-ip
[virtual-machine-sizes]: /azure/virtual-machines/virtual-machines-linux-sizes
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-resize]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[vm-size-tables]: /azure/virtual-machines/virtual-machines-linux-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[readme]: https://github.com/mspnp/reference-architectures/blob/master/virtual-machines/single-vm/README.md
[0]: ./images/single-vm-diagram.png "Azure 中的单一 Linux VM 体系结构"

