---
title: "大计算的体系结构样式"
description: "介绍 Azure 上大计算的体系结构的优点、挑战和最佳做法"
author: MikeWasson
ms.openlocfilehash: b16be4133143d7d73062eeb280b44779c390f387
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="big-compute-architecture-style"></a><span data-ttu-id="52126-103">大计算的体系结构样式</span><span class="sxs-lookup"><span data-stu-id="52126-103">Big compute architecture style</span></span>

<span data-ttu-id="52126-104">术语“大计算”指的是需要大量核心的大规模工作负载，核心数量通常以数百或数千计。</span><span class="sxs-lookup"><span data-stu-id="52126-104">The term *big compute* describes large-scale workloads that require a large number of cores, often numbering in the hundreds or thousands.</span></span> <span data-ttu-id="52126-105">方案包括图像渲染、流体动力学、金融风险建模、石油勘探、药物设计和工程应力分析等等。</span><span class="sxs-lookup"><span data-stu-id="52126-105">Scenarios include image rendering, fluid dynamics, financial risk modeling, oil exploration, drug design, and engineering stress analysis, among others.</span></span>

![](./images/big-compute-logical.png)

<span data-ttu-id="52126-106">以下是大计算应用程序的一些典型特征：</span><span class="sxs-lookup"><span data-stu-id="52126-106">Here are some typical characteristics of big compute applications:</span></span>

- <span data-ttu-id="52126-107">工作可拆分为离散的任务，这些任务可以跨多个核心同时运行。</span><span class="sxs-lookup"><span data-stu-id="52126-107">The work can be split into discrete tasks, which can be run across many cores simultaneously.</span></span>
- <span data-ttu-id="52126-108">各任务都是有限的。</span><span class="sxs-lookup"><span data-stu-id="52126-108">Each task is finite.</span></span> <span data-ttu-id="52126-109">接收一些输入，执行某些处理操作，然后生成输出。</span><span class="sxs-lookup"><span data-stu-id="52126-109">It takes some input, does some processing, and produces output.</span></span> <span data-ttu-id="52126-110">整个应用程序的运行时间（从数分钟到数天）有限。</span><span class="sxs-lookup"><span data-stu-id="52126-110">The entire application runs for a finite amount of time (minutes to days).</span></span> <span data-ttu-id="52126-111">常见模式是突然预配大量核心，在应用程序完成后，核心数量减少到零。</span><span class="sxs-lookup"><span data-stu-id="52126-111">A common pattern is to provision a large number of cores in a burst, and then spin down to zero once the application completes.</span></span> 
- <span data-ttu-id="52126-112">应用程序不需要全天候运行。</span><span class="sxs-lookup"><span data-stu-id="52126-112">The application does not need to stay up 24/7.</span></span> <span data-ttu-id="52126-113">但是，系统必须处理节点故障或应用程序故障。</span><span class="sxs-lookup"><span data-stu-id="52126-113">However, the system must handle node failures or application crashes.</span></span>
- <span data-ttu-id="52126-114">对于某些应用程序，任务是独立的且可并行运行。</span><span class="sxs-lookup"><span data-stu-id="52126-114">For some applications, tasks are independent and can run in parallel.</span></span> <span data-ttu-id="52126-115">在其他情况下，任务紧密耦合，这意味着它们必须交互或交换中间结果。</span><span class="sxs-lookup"><span data-stu-id="52126-115">In other cases, tasks are tightly coupled, meaning they must interact or exchange intermediate results.</span></span> <span data-ttu-id="52126-116">在该情况下，请考虑使用 InfiniBand 和远程直接内存访问 (RDMA) 等高速联网技术。</span><span class="sxs-lookup"><span data-stu-id="52126-116">In that case, consider using high-speed networking technologies such as InfiniBand and remote direct memory access (RDMA).</span></span> 
- <span data-ttu-id="52126-117">可以根据工作负载，使用不同大小的计算密集型 VM（H16r、H16mr 和 A9）。</span><span class="sxs-lookup"><span data-stu-id="52126-117">Depending on your workload, you might use compute-intensive VM sizes (H16r, H16mr, and A9).</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="52126-118">此体系结构适用的情况</span><span class="sxs-lookup"><span data-stu-id="52126-118">When to use this architecture</span></span>

- <span data-ttu-id="52126-119">模拟和数字运算等计算密集型操作。</span><span class="sxs-lookup"><span data-stu-id="52126-119">Computationally intensive operations such as simulation and number crunching.</span></span>
- <span data-ttu-id="52126-120">计算密集型模拟，须拆分到多台计算机（10 - 1000 台）的 CPU 中。</span><span class="sxs-lookup"><span data-stu-id="52126-120">Simulations that are computationally intensive and must be split across CPUs in multiple computers (10-1000s).</span></span>
- <span data-ttu-id="52126-121">对一台计算机的内存要求过高的模拟，须拆分到多台计算机中。</span><span class="sxs-lookup"><span data-stu-id="52126-121">Simulations that require too much memory for one computer, and must be split across multiple computers.</span></span>
- <span data-ttu-id="52126-122">长时间运行的计算，在一台计算机上完成计算会花费过长时间。</span><span class="sxs-lookup"><span data-stu-id="52126-122">Long-running computations that would take too long to complete on a single computer.</span></span>
- <span data-ttu-id="52126-123">必须运行 100 次或 1000 次的较小型计算，如 Monte Carlo 模拟。</span><span class="sxs-lookup"><span data-stu-id="52126-123">Smaller computations that must be run 100s or 1000s of times, such as Monte Carlo simulations.</span></span>

## <a name="benefits"></a><span data-ttu-id="52126-124">优点</span><span class="sxs-lookup"><span data-stu-id="52126-124">Benefits</span></span>

- <span data-ttu-id="52126-125">高性能和“[易并行][embarrassingly-parallel]”处理。</span><span class="sxs-lookup"><span data-stu-id="52126-125">High performance with "[embarrassingly parallel][embarrassingly-parallel]" processing.</span></span>
- <span data-ttu-id="52126-126">可以利用数百或数千个计算机核心更快地解决大型问题。</span><span class="sxs-lookup"><span data-stu-id="52126-126">Can harness hundreds or thousands of computer cores to solve large problems faster.</span></span>
- <span data-ttu-id="52126-127">可以通过 InfiniBand 高速专用网络，访问高性能专用硬件。</span><span class="sxs-lookup"><span data-stu-id="52126-127">Access to specialized high-performance hardware, with dedicated high-speed InfiniBand networks.</span></span>
- <span data-ttu-id="52126-128">可以根据工作需要预配 VM，然后再将它们关闭。</span><span class="sxs-lookup"><span data-stu-id="52126-128">You can provision VMs as needed to do work, and then tear them down.</span></span> 

## <a name="challenges"></a><span data-ttu-id="52126-129">挑战</span><span class="sxs-lookup"><span data-stu-id="52126-129">Challenges</span></span>

- <span data-ttu-id="52126-130">管理 VM 基础结构。</span><span class="sxs-lookup"><span data-stu-id="52126-130">Managing the VM infrastructure.</span></span>
- <span data-ttu-id="52126-131">管理数字运算量。</span><span class="sxs-lookup"><span data-stu-id="52126-131">Managing the volume of number crunching.</span></span> 
- <span data-ttu-id="52126-132">及时预配数千个核心。</span><span class="sxs-lookup"><span data-stu-id="52126-132">Provisioning thousands of cores in a timely manner.</span></span>
- <span data-ttu-id="52126-133">对于紧密耦合的任务，添加更多核心可能会减少返回量。</span><span class="sxs-lookup"><span data-stu-id="52126-133">For tightly coupled tasks, adding more cores can have diminishing returns.</span></span> <span data-ttu-id="52126-134">可能需要进行试验来找到最适宜的核心数。</span><span class="sxs-lookup"><span data-stu-id="52126-134">You may need to experiment to find the optimum number of cores.</span></span>

## <a name="big-compute-using-azure-batch"></a><span data-ttu-id="52126-135">使用 Azure Batch 的大计算</span><span class="sxs-lookup"><span data-stu-id="52126-135">Big compute using Azure Batch</span></span>

<span data-ttu-id="52126-136">[Azure Batch][batch] 是一个托管服务，适用于运行大规模高性能计算 (HPC) 应用程序。</span><span class="sxs-lookup"><span data-stu-id="52126-136">[Azure Batch][batch] is a managed service for running large-scale high-performance computing (HPC) applications.</span></span>

<span data-ttu-id="52126-137">使用 Azure Batch 配置 VM 池并上传应用程序和数据文件。</span><span class="sxs-lookup"><span data-stu-id="52126-137">Using Azure Batch, you configure a VM pool, and upload the applications and data files.</span></span> <span data-ttu-id="52126-138">然后 Batch 服务预配 VM、将任务分配给 VM、运行任务并监视进度。</span><span class="sxs-lookup"><span data-stu-id="52126-138">Then the Batch service provisions the VMs, assign tasks to the VMs, runs the tasks, and monitors the progress.</span></span> <span data-ttu-id="52126-139">Batch 可以根据工作负载横向扩展 VM。</span><span class="sxs-lookup"><span data-stu-id="52126-139">Batch can automatically scale out the VMs in response to the workload.</span></span> <span data-ttu-id="52126-140">Batch 还提供作业计划。</span><span class="sxs-lookup"><span data-stu-id="52126-140">Batch also provides job scheduling.</span></span>

![](./images/big-compute-batch.png) 

## <a name="big-compute-running-on-virtual-machines"></a><span data-ttu-id="52126-141">在虚拟机上运行的大计算</span><span class="sxs-lookup"><span data-stu-id="52126-141">Big compute running on Virtual Machines</span></span>

<span data-ttu-id="52126-142">可以使用 [Microsoft HPC Pack][hpc-pack] 管理 VM 群集以及计划并监视 HPC 作业。</span><span class="sxs-lookup"><span data-stu-id="52126-142">You can use [Microsoft HPC Pack][hpc-pack] to administer a cluster of VMs, and schedule and monitor HPC jobs.</span></span> <span data-ttu-id="52126-143">在此方法中，必须预配并管理 VM 和网络基础结构。</span><span class="sxs-lookup"><span data-stu-id="52126-143">With this approach, you must provision and manage the VMs and network infrastructure.</span></span> <span data-ttu-id="52126-144">如果有现有的 HPC 工作负载且要将其部分或全部移动到 Azure，请考虑使用此方法。</span><span class="sxs-lookup"><span data-stu-id="52126-144">Consider this approach if you have existing HPC workloads and want to move some or all it to Azure.</span></span> <span data-ttu-id="52126-145">可将整个 HPC 群集移动到 Azure，或本地保存 HPC 群集但对迸发容量使用 Azure。</span><span class="sxs-lookup"><span data-stu-id="52126-145">You can move the entire HPC cluster to Azure, or keep your HPC cluster on-premises but use Azure for burst capacity.</span></span> <span data-ttu-id="52126-146">有关详细信息，请参阅[适用于大规模计算工作负载的 Batch 和 HPC 解决方案][batch-hpc-solutions]。</span><span class="sxs-lookup"><span data-stu-id="52126-146">For more information, see [Batch and HPC solutions for large-scale computing workloads][batch-hpc-solutions].</span></span>

### <a name="hpc-pack-deployed-to-azure"></a><span data-ttu-id="52126-147">部署到 Azure 的 HPC Pack</span><span class="sxs-lookup"><span data-stu-id="52126-147">HPC Pack deployed to Azure</span></span>

<span data-ttu-id="52126-148">在此方案中，HPC 群集完全是在 Azure 中创建的。</span><span class="sxs-lookup"><span data-stu-id="52126-148">In this scenario, the HPC cluster is created entirely within Azure.</span></span>

![](./images/big-compute-iaas.png) 
 
<span data-ttu-id="52126-149">头节点向群集提供管理和作业计划服务。</span><span class="sxs-lookup"><span data-stu-id="52126-149">The head node provides management and job scheduling services to the cluster.</span></span> <span data-ttu-id="52126-150">对于紧密耦合的任务，所使用的 RDMA 网络需提供极高的带宽，并在 VM 之间提供低延迟的通信。</span><span class="sxs-lookup"><span data-stu-id="52126-150">For tightly coupled tasks, use an RDMA network that provides very high bandwidth, low latency communication between VMs.</span></span> <span data-ttu-id="52126-151">有关详细信息，请参阅[在 Azure 中部署 HPC Pack 2016 群集][deploy-hpc-azure]。</span><span class="sxs-lookup"><span data-stu-id="52126-151">For more information see [Deploy an HPC Pack 2016 cluster in Azure][deploy-hpc-azure].</span></span>

### <a name="burst-an-hpc-cluster-to-azure"></a><span data-ttu-id="52126-152">将 HPC 群集迸发到 Azure</span><span class="sxs-lookup"><span data-stu-id="52126-152">Burst an HPC cluster to Azure</span></span>

<span data-ttu-id="52126-153">在此方案中，组织在本地运行 HPC Pack，并对迸发容量使用 Azure VM。</span><span class="sxs-lookup"><span data-stu-id="52126-153">In this scenario, an organization is running HPC Pack on-premises, and uses Azure VMs for burst capacity.</span></span> <span data-ttu-id="52126-154">群集的头节点位于本地。</span><span class="sxs-lookup"><span data-stu-id="52126-154">The cluster head node is on-premises.</span></span> <span data-ttu-id="52126-155">ExpressRoute 或 VPN 网关将本地网络连接到 Azure VNet。</span><span class="sxs-lookup"><span data-stu-id="52126-155">ExpressRoute or VPN Gateway connects the on-premises network to the Azure VNet.</span></span>

![](./images/big-compute-hybrid.png) 


[batch]: /azure/batch/
[batch-hpc-solutions]: /azure/batch/batch-hpc-solutions
[deploy-hpc-azure]: /azure/virtual-machines/windows/hpcpack-2016-cluster
[embarrassingly-parallel]: https://en.wikipedia.org/wiki/Embarrassingly_parallel
[hpc-pack]: https://technet.microsoft.com/library/cc514029

 
