---
title: 技术指南：从本地恢复到 Azure
description: 本文可帮助了解和设计从本地基础结构到 Azure 的恢复系统
author: adamglick
ms.date: 08/18/2016
ms.openlocfilehash: 6992e27d148074b3d60c282318741f45974d1afd
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/06/2018
---
[!INCLUDE [header](../_includes/header.md)]

# <a name="azure-resiliency-technical-guidance-recovery-from-on-premises-to-azure"></a><span data-ttu-id="8ec5c-103">Azure 复原技术指南：从本地恢复到 Azure</span><span class="sxs-lookup"><span data-stu-id="8ec5c-103">Azure resiliency technical guidance: Recovery from on-premises to Azure</span></span>
<span data-ttu-id="8ec5c-104">Azure 提供一整套服务，可让你将本地数据中心扩展到 Azure，用于高可用性和灾难恢复目的：</span><span class="sxs-lookup"><span data-stu-id="8ec5c-104">Azure provides a comprehensive set of services for enabling the extension of an on-premises datacenter to Azure for high availability and disaster recovery purposes:</span></span>

* <span data-ttu-id="8ec5c-105">**网络**：通过虚拟专用网络，可以安全地将本地网络扩展到云。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-105">**Networking**: With a virtual private network, you securely extend your on-premises network to the cloud.</span></span>
* <span data-ttu-id="8ec5c-106">**计算**：使用本地 Hyper-V 的客户可以将现有虚拟机 (VM)“提起并移动”到 Azure。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-106">**Compute**: Customers using Hyper-V on-premises can “lift and shift” existing virtual machines (VMs) to Azure.</span></span>
* <span data-ttu-id="8ec5c-107">**存储**：StorSimple 可将文件系统扩展到 Azure 存储。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-107">**Storage**: StorSimple extends your file system to Azure Storage.</span></span> <span data-ttu-id="8ec5c-108">Azure 备份服务提供将文件和 SQL 数据库备份到 Azure 存储的功能。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-108">The Azure Backup service provides backup for files and SQL databases to Azure Storage.</span></span>
* <span data-ttu-id="8ec5c-109">**数据库复制**：使用 SQL 2014（或更高版本）可用性组，可以为本地数据实现高可用性和灾难恢复。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-109">**Database replication**: With SQL Server 2014 (or later) Availability Groups, you can implement high availability and disaster recovery for your on-premises data.</span></span>

## <a name="networking"></a><span data-ttu-id="8ec5c-110">网络</span><span class="sxs-lookup"><span data-stu-id="8ec5c-110">Networking</span></span>
<span data-ttu-id="8ec5c-111">可以使用 Azure 虚拟网络在 Azure 中创建一个逻辑上独立的部分，并可使用 IPsec 连接将其连接到本地数据中心或单个客户端计算机。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-111">You can use Azure Virtual Network to create a logically isolated section in Azure and securely connect it to your on-premises datacenter or a single client machine by using an IPsec connection.</span></span> <span data-ttu-id="8ec5c-112">通过虚拟网络，可轻松地利用 Azure 中按需可缩放的基础架构，同时可连接到本地数据和应用程序，包括 Windows Server、大型机和 UNIX 上运行的系统。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-112">With Virtual Network, you can take advantage of the scalable, on-demand infrastructure in Azure while providing connectivity to data and applications on-premises, including systems running on Windows Server, mainframes, and UNIX.</span></span> <span data-ttu-id="8ec5c-113">有关详细信息，请参阅 [Azure 网络文档](/azure/virtual-network/virtual-networks-overview/)。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-113">See [Azure networking documentation](/azure/virtual-network/virtual-networks-overview/) for more information.</span></span>

## <a name="compute"></a><span data-ttu-id="8ec5c-114">计算</span><span class="sxs-lookup"><span data-stu-id="8ec5c-114">Compute</span></span>
<span data-ttu-id="8ec5c-115">使用本地 Hyper-V 的客户可以将现有的虚拟机“提起并移动”到 Azure 和运行 Windows Server 2012（或更高版本）的服务提供程序，而不需要对 VM 进行更改或转换 VM 格式。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-115">If you're using Hyper-V on-premises, you can “lift and shift” existing virtual machines to Azure and service providers running Windows Server 2012 (or later), without making changes to the VM or converting VM formats.</span></span> <span data-ttu-id="8ec5c-116">有关详细信息，请参阅[关于 Azure 虚拟机的磁盘和 VHD](/azure/virtual-machines/virtual-machines-linux-about-disks-vhds/?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-116">For more information, see [About disks and VHDs for Azure virtual machines](/azure/virtual-machines/virtual-machines-linux-about-disks-vhds/?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).</span></span>

## <a name="azure-site-recovery"></a><span data-ttu-id="8ec5c-117">Azure Site Recovery</span><span class="sxs-lookup"><span data-stu-id="8ec5c-117">Azure Site Recovery</span></span>
<span data-ttu-id="8ec5c-118">如果希望灾难恢复作为一项服务 (DRaaS)，Azure 提供了 [Azure Site Recovery](https://azure.microsoft.com/services/site-recovery/)。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-118">If you want disaster recovery as a service (DRaaS), Azure provides [Azure Site Recovery](https://azure.microsoft.com/services/site-recovery/).</span></span> <span data-ttu-id="8ec5c-119">Azure Site Recovery 为 VMware、Hyper-V 和物理服务器提供全面的保护。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-119">Azure Site Recovery offers comprehensive protection for VMware, Hyper-V, and physical servers.</span></span> <span data-ttu-id="8ec5c-120">借助 Azure Site Recovery，可以使用另一台本地服务器或 Azure 作为恢复站点。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-120">With Azure Site Recovery, you can use another on-premises server or Azure as your recovery site.</span></span> <span data-ttu-id="8ec5c-121">有关 Azure Site Recovery 的详细信息，请参阅 [Azure Site Recovery 文档](https://azure.microsoft.com/documentation/services/site-recovery/)。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-121">For more information on Azure Site Recovery, see the [Azure Site Recovery documentation](https://azure.microsoft.com/documentation/services/site-recovery/).</span></span>

## <a name="storage"></a><span data-ttu-id="8ec5c-122">存储</span><span class="sxs-lookup"><span data-stu-id="8ec5c-122">Storage</span></span>
<span data-ttu-id="8ec5c-123">可以使用多个选项将 Azure 用作本地数据的备份站点。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-123">There are several options for using Azure as a backup site for on-premises data.</span></span>

### <a name="storsimple"></a><span data-ttu-id="8ec5c-124">StorSimple</span><span class="sxs-lookup"><span data-stu-id="8ec5c-124">StorSimple</span></span>
<span data-ttu-id="8ec5c-125">StorSimple 可安全、透明地整合本地应用程序的云存储。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-125">StorSimple securely and transparently integrates cloud storage for on-premises applications.</span></span> <span data-ttu-id="8ec5c-126">它还提供单个设备来实现高性能的分层本地和云存储、实时存档、基于云的数据保护和灾难恢复。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-126">It also offers a single appliance that delivers high-performance tiered local and cloud storage, live archiving, cloud-based data protection, and disaster recovery.</span></span> <span data-ttu-id="8ec5c-127">有关详细信息，请参阅 [StorSimple 产品页](https://azure.microsoft.com/services/storsimple/)。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-127">For more information, see the [StorSimple product page](https://azure.microsoft.com/services/storsimple/).</span></span>

### <a name="azure-backup"></a><span data-ttu-id="8ec5c-128">Azure 备份</span><span class="sxs-lookup"><span data-stu-id="8ec5c-128">Azure Backup</span></span>
<span data-ttu-id="8ec5c-129">Azure 备份让可以使用 Windows Server 2012（或更高版本）、Windows Server 2012 Essentials（或更高版本）和 System Center 2012 Data Protection Manager（或更高版本）中熟悉的备份工具进行云备份。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-129">Azure Backup enables cloud backups by using the familiar backup tools in Windows Server 2012 (or later), Windows Server 2012 Essentials (or later), and System Center 2012 Data Protection Manager (or later).</span></span> <span data-ttu-id="8ec5c-130">这些工具提供了独立于备份存储位置（无论是本地磁盘还是 Azure 存储）的备份管理工作流。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-130">These tools provide a workflow for backup management that is independent of the storage location of the backups, whether a local disk or Azure Storage.</span></span> <span data-ttu-id="8ec5c-131">数据备份到云后，经过授权的用户可以轻松地恢复到任何服务器的备份。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-131">After data is backed up to the cloud, authorized users can easily recover backups to any server.</span></span>

<span data-ttu-id="8ec5c-132">使用增量备份时，只会将文件更改传输到云。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-132">With incremental backups, only changes to files are transferred to the cloud.</span></span> <span data-ttu-id="8ec5c-133">这有助于高效使用存储空间、降低带宽消耗并支持多个数据版本的时间点恢复。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-133">This helps to efficiently use storage space, reduce bandwidth consumption, and support point-in-time recovery of multiple versions of the data.</span></span> <span data-ttu-id="8ec5c-134">也可以选择使用其他功能，如数据保留策略、数据压缩和数据传输限制。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-134">You can also choose to use additional features, such as data retention policies, data compression, and data transfer throttling.</span></span> <span data-ttu-id="8ec5c-135">使用 Azure 作为备份位置有一个明显的优点，那就是自动在“场外”备份。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-135">Using Azure as the backup location has the obvious advantage that the backups are automatically “offsite”.</span></span> <span data-ttu-id="8ec5c-136">这样就不再需要额外对现场备份媒体进行保护了。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-136">This eliminates the extra requirements to secure and protect on-site backup media.</span></span>

<span data-ttu-id="8ec5c-137">有关详细信息，请参阅[什么是 Azure 备份？](/azure/backup/backup-introduction-to-azure-backup/)和[为 DPM 数据配置 Azure 备份](https://technet.microsoft.com/library/jj728752.aspx)。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-137">For more information, see [What is Azure Backup?](/azure/backup/backup-introduction-to-azure-backup/) and [Configure Azure Backup for DPM data](https://technet.microsoft.com/library/jj728752.aspx).</span></span>

## <a name="database"></a><span data-ttu-id="8ec5c-138">数据库</span><span class="sxs-lookup"><span data-stu-id="8ec5c-138">Database</span></span>
<span data-ttu-id="8ec5c-139">可使用 AlwaysOn 可用性组、数据库镜像、日志传送以及备份和还原与 Azure Blob 存储配合使用，在混合 IT 环境中为 SQL Server 数据库提供灾难恢复解决方案。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-139">You can have a disaster recovery solution for your SQL Server databases in a hybrid-IT environment by using AlwaysOn Availability Groups, database mirroring, log shipping, and backup and restore with Azure Blob storage.</span></span> <span data-ttu-id="8ec5c-140">所有这些解决方案都使用 Azure 虚拟机上运行的 SQL Server。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-140">All of these solutions use SQL Server running on Azure Virtual Machines.</span></span>

<span data-ttu-id="8ec5c-141">AlwaysOn 可用性组可在本地和云中都有数据库副本的混合 IT 环境中使用。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-141">AlwaysOn Availability Groups can be used in a hybrid-IT environment where database replicas exist both on-premises and in the cloud.</span></span> <span data-ttu-id="8ec5c-142">下图显示了此特点。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-142">This is shown in the following diagram.</span></span>

![混合云体系结构中的 SQL Server AlwaysOn 可用性组](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-3.png)

<span data-ttu-id="8ec5c-144">在基于证书的设置中，数据库镜像也可以横跨本地服务器和云。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-144">Database mirroring can also span on-premises servers and the cloud in a certificate-based setup.</span></span> <span data-ttu-id="8ec5c-145">下图演示了此概念。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-145">The following diagram illustrates this concept.</span></span>

![混合云体系结构中的 SQL Server 数据库镜像](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-4.png)

<span data-ttu-id="8ec5c-147">日志传送可用于同步本地数据库和 Azure 虚拟机中的 SQL Server 数据库。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-147">Log shipping can be used to synchronize an on-premises database with a SQL Server database in an Azure virtual machine.</span></span>

![混合云体系结构中的 SQL Server 日志传送](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-5.png)

<span data-ttu-id="8ec5c-149">最后，可以直接将本地数据库备份到 Azure Blob 存储。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-149">Finally, you can back up an on-premises database directly to Azure Blob storage.</span></span>

![在混合云体系结构中将 SQL Server 备份到 Azure Blob 存储](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-6.png)

<span data-ttu-id="8ec5c-151">有关详细信息，请参阅 [Azure 虚拟机中 SQL Server 的高可用性和灾难恢复](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/)和 [Azure 虚拟机中 SQL Server 的备份和还原](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-backup-recovery/)。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-151">For more information, see [High availability and disaster recovery for SQL Server in Azure virtual machines](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/) and [Backup and restore for SQL Server in Azure virtual machines](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-backup-recovery/).</span></span>

## <a name="checklists-for-on-premises-recovery-in-microsoft-azure"></a><span data-ttu-id="8ec5c-152">Microsoft Azure 中的本地恢复清单</span><span class="sxs-lookup"><span data-stu-id="8ec5c-152">Checklists for on-premises recovery in Microsoft Azure</span></span>
### <a name="networking"></a><span data-ttu-id="8ec5c-153">网络</span><span class="sxs-lookup"><span data-stu-id="8ec5c-153">Networking</span></span>
1. <span data-ttu-id="8ec5c-154">查看本文档的“网络”部分。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-154">Review the Networking section of this document.</span></span>
2. <span data-ttu-id="8ec5c-155">使用虚拟网络安全地将本地部署与云相连。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-155">Use Virtual Network to securely connect on-premises to the cloud.</span></span>

### <a name="compute"></a><span data-ttu-id="8ec5c-156">计算</span><span class="sxs-lookup"><span data-stu-id="8ec5c-156">Compute</span></span>
1. <span data-ttu-id="8ec5c-157">查看本文档的“计算”部分。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-157">Review the Compute section of this document.</span></span>
2. <span data-ttu-id="8ec5c-158">在 Hyper-V 和 Azure 之间重新定位 VM。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-158">Relocate VMs between Hyper-V and Azure.</span></span>

### <a name="storage"></a><span data-ttu-id="8ec5c-159">存储</span><span class="sxs-lookup"><span data-stu-id="8ec5c-159">Storage</span></span>
1. <span data-ttu-id="8ec5c-160">查看本文档的“存储”部分。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-160">Review the Storage section of this document.</span></span>
2. <span data-ttu-id="8ec5c-161">通过 StorSimple 服务来使用云存储。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-161">Take advantage of StorSimple services for using cloud storage.</span></span>
3. <span data-ttu-id="8ec5c-162">使用 Azure 备份服务。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-162">Use the Azure Backup service.</span></span>

### <a name="database"></a><span data-ttu-id="8ec5c-163">数据库</span><span class="sxs-lookup"><span data-stu-id="8ec5c-163">Database</span></span>
1. <span data-ttu-id="8ec5c-164">查看本文档的“数据库”部分。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-164">Review the Database section of this document.</span></span>
2. <span data-ttu-id="8ec5c-165">考虑使用 Azure VM 上的 SQL Server 作为备份。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-165">Consider using SQL Server on Azure VMs as the backup.</span></span>
3. <span data-ttu-id="8ec5c-166">设置 AlwaysOn 可用性组。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-166">Set up AlwaysOn Availability Groups.</span></span>
4. <span data-ttu-id="8ec5c-167">配置基于证书的数据库镜像。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-167">Configure certificate-based database mirroring.</span></span>
5. <span data-ttu-id="8ec5c-168">使用日志传送。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-168">Use log shipping.</span></span>
6. <span data-ttu-id="8ec5c-169">将本地数据库备份到 Azure Blob 存储。</span><span class="sxs-lookup"><span data-stu-id="8ec5c-169">Back up on-premises databases to Azure Blob storage.</span></span>


