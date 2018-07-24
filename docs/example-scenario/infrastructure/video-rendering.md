---
title: Azure 上的 3D 视频渲染
description: 使用 Azure Batch 服务在 Azure 中运行本机 HPC 工作负荷
author: adamboeglin
ms.date: 07/13/2018
ms.openlocfilehash: b3af0641642d7ec4b022e8c96f51693eeb0adee4
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060998"
---
# <a name="3d-video-rendering-on-azure"></a>Azure 上的 3D 视频渲染

3D 渲染是很费时的过程，需要大量的 CPU 时间才能完成。  在一台计算机中，从静态资产生成视频文件可能需要数小时甚至数天，具体取决于所要生成的视频的长度和复杂程度。  许多公司会购买昂贵的高端台式机来执行这些任务，或者会投资可以向其提交作业的大型渲染场。  但是，如果使用 Azure Batch，则可根据需要随意启用或关闭该功能，根本不需要任何资本投资。

不管是选择 Windows Server 还是选择 Linux 计算节点，都可以通过 Batch 获得始终如一的管理体验并进行作业计划。 有了 Batch，你就可以使用现有的 Windows 或 Linux 应用程序（包括 AutoDesk Maya 和 Blender）在 Azure 中运行大规模的渲染作业。

## <a name="related-use-cases"></a>相关的用例

考虑将本方案用于以下类似的用例：

* 3D 建模
* Visual FX (VFX) 渲染
* 视频转码
* 图像处理、颜色校正和重设大小

## <a name="architecture"></a>体系结构

![从体系结构的角度概要说明使用 Azure Batch 的云原生 HPC 解决方案中涉及的组件][architecture]

本示例方案介绍了使用 Azure Batch 时的工作流，数据的流动如下所示：

1. 将输入文件和处理这些文件的应用程序上传到 Azure 存储帐户
2. 创建一个包含 Batch 帐户中的计算节点的 Batch 池、一个用于在池中运行工作负荷的作业，以及作业中的任务。
3. 将输入文件和应用程序下载到 Batch
4. 监视任务执行情况
5. 上传任务输出
6. 下载输出文件

若要简化此过程，也可使用 [Maya 和 3ds Max 的 Batch 插件][batch-plugins]

### <a name="components"></a>组件

Azure Batch 基于以下 Azure 技术：

* [资源组][resource-groups]是 Azure 资源的逻辑容器
* [虚拟网络][vnet]用于“头节点”和“计算”资源
* [存储][storage]帐户用于同步和数据保留
* [虚拟机规模集][vmss]由 CycleCloud 用于计算资源

## <a name="considerations"></a>注意事项

### <a name="machine-sizes-available-for-azure-batch"></a>适用于 Azure Batch 的计算机大小
虽然大多数渲染客户会选择 CPU 功率高的资源，但是其他使用 VM 规模集的工作负荷可能会按其他标准来选择 VM，具体取决于许多因素：
  - 所运行的应用程序是否进行内存绑定？
  - 应用程序是否需要使用 GPU？ 
  - 对于紧密耦合的作业，作业类型是否只能采用不得已的并行方式？或者需要 Infiniband 连接？
  - 需要计算节点上有快速的存储 I/O

Azure 有各种 VM 大小，每个都符合上述应用程序要求，某些是针对 HPC 的，但即使是最小的，也可以提供有效的网格实现：

  - [HPC VM 大小][compute-hpc]：考虑到渲染受 CPU 限制，Microsoft 通常会建议使用 Azure H 系列 VM。  这些是专门针对高端计算需求构建的，提供 8 核和 16 核 vCPU 大小，采用 DDR4 内存、SSD 临时存储以及 Haswell E5 Intel 技术。
  - [GPU VM 大小][compute-gpu]：GPU 优化 VM 大小是具有单个或多个 NVIDIA GPU 的专用虚拟机。 这些大小是针对计算密集型、图形密集型和可视化工作负荷设计的。
    - NC、NCv2、NCv3 和 ND 大小针对计算密集型和网络密集型应用程序和算法进行了优化，包括基于 CUDA 和 OpenCL 的应用程序和模拟、AI 以及深度学习。 NV 大小已针对远程可视化效果、流式处理、游戏、编码和 VDI 方案进行了优化和设计，使用 OpenGL 和 DirectX 之类的框架。
  - [内存优化型 VM 大小][compute-memory]：需要更多内存时，内存优化型 VM 大小的内存/CPU 比率更高。
  - [常规用途 VM 大小][compute-general]：常规用途 VM 大小也可使用，提供均衡的 CPU/内存比率。

### <a name="alternatives"></a>备选项

如果需要对 Azure 中的渲染环境进行更多的控制，或者需要混合实现，则可使用 CycleCloud 计算功能来协调云中的 IaaS 网格。 它使用与 Azure Batch 相同的基础 Azure 技术，使生成和维护 IaaS 网格变得很高效。 若要查找更多信息并了解设计原则，请查看以下链接：

有关 Azure 中提供的所有 HPC 解决方案的完整概述，请参阅[使用 Azure VM 的 HPC、Batch 和 Big 计算解决方案][hpc-alt-solutions]一文

### <a name="availability"></a>可用性

可以通过一系列的服务、工具和 API 监视 Azure Batch 组件。 这在[监视 Batch 解决方案][batch-monitor]一文中有进一步的介绍。

### <a name="scalability"></a>可伸缩性

Azure Batch 帐户中的池可以通过手动干预进行缩放，也可以通过基于 Azure Batch 指标的公式进行自动缩放。 有关此方面的详细信息，请参阅[创建用于缩放 Batch 池中的节点的自动缩放公式][batch-scaling]一文。

### <a name="security"></a>“安全”

若需安全解决方案的通用设计指南，请参阅 [Azure 安全性文档][security]。

### <a name="resiliency"></a>复原

虽然 Azure Batch 中目前没有故障转移功能，但建议你执行以下步骤，确保发生计划外中断时的可用性：

* 使用备用的存储帐户在备用的 Azure 位置创建一个 Azure Batch 帐户
* 使用同一名称创建相同的节点池，不分配任何节点
* 确保通过备用存储帐户创建并更新应用程序
* 将输入文件上传到备用的 Azure Batch 帐户，提交作业时也提交到该帐户

## <a name="deploy-this-scenario"></a>部署此方案

### <a name="creating-an-azure-batch-account-and-pools-manually"></a>手动创建 Azure Batch 帐户和池

本示例方案有助于了解 Azure Batch 工作方式，并将 Azure Batch Labs 以示例 SaaS 解决方案（可以为你自己的客户开发）的方式进行了演示：

[Azure Batch Masterclass][batch-labs-masterclass]

### <a name="deploying-the-sample-scenario-using-an-azure-resource-manager-arm-template"></a>使用 Azure 资源管理器 (ARM) 模板部署示例方案

此模板将部署：
  - 一个新的 Azure Batch 帐户
  - 一个存储帐户
  - 一个与 Batch 帐户关联的节点池
  - 节点池将会配置为对 Canonical Ubuntu 映像使用 A2 v2 VM
  - 节点池一开始包含 0 个 VM，需通过手动缩放来添加 VM

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fhpc%2Fbatchcreatewithpools.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

[详细了解 ARM 模板][azure-arm-templates]

## <a name="pricing"></a>定价

Azure Batch 的使用费用取决于用于池的 VM 大小及其分配和运行时长。创建 Azure Batch 帐户没有相关费用。 存储和数据出口也应考虑在内，因为这些需额外付费。

下面以示例方式说明了一个在 8 小时内完成的作业在使用不同数目的服务器时可能会产生的费用：


- 100 个高性能 CPU VM：[费用估算][hpc-est-high]

  100 x H16m（16 核，225GB RAM，高级存储 512GB），2 TB Blob 存储，1 TB 出口

- 50 个高性能 CPU VM：[费用估算][hpc-est-med]

  50 x H16m（16 核，225GB RAM，高级存储 512GB），2 TB Blob 存储，1 TB 出口

- 10 个高性能 CPU VM：[费用估算][hpc-est-low]
  
  10 x H16m（16 核，225GB RAM，高级存储 512GB），2 TB Blob 存储，1 TB 出口

### <a name="low-priority-vm-pricing"></a>低优先级 VM 定价

Azure Batch 也支持在节点池中使用低优先级 VM*，这可能会节省大量费用。 若要在标准 VM 和低优先级 VM 之间进行价格比较，并了解有关低优先级 VM 的详细信息，请参阅 [Batch 定价][batch-pricing]。

\* 请注意，只有某些应用程序和工作负荷适合在低优先级 VM 上运行。

## <a name="related-resources"></a>相关资源

[Azure Batch 概述][batch-overview]

[Azure Batch 文档][batch-doc]

[在 Azure Batch 上使用容器][batch-containers]

<!-- links -->
[architecture]: ./media/native-hpc-ref-arch.png
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[resiliency]: /azure/architecture/resiliency/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[vnet]: /azure/virtual-network/virtual-networks-overview
[storage]: https://azure.microsoft.com/services/storage/
[batch]: https://azure.microsoft.com/services/batch/
[batch-arch]: https://azure.microsoft.com/solutions/architecture/big-compute-with-azure-batch/
[compute-hpc]: /azure/virtual-machines/windows/sizes-hpc
[compute-gpu]: /azure/virtual-machines/windows/sizes-gpu
[compute-compute]: /azure/virtual-machines/windows/sizes-compute
[compute-memory]: /azure/virtual-machines/windows/sizes-memory
[compute-general]: /azure/virtual-machines/windows/sizes-general
[compute-storage]: /azure/virtual-machines/windows/sizes-storage
[compute-acu]: /azure/virtual-machines/windows/acu
[compute=benchmark]: /azure/virtual-machines/windows/compute-benchmark-scores
[hpc-est-high]: https://azure.com/e/9ac25baf44ef49c3a6b156935ee9544c
[hpc-est-med]: https://azure.com/e/0286f1d6f6784310af4dcda5aec8c893
[hpc-est-low]: https://azure.com/e/e39afab4e71949f9bbabed99b428ba4a
[batch-labs-masterclass]: https://github.com/azurebigcompute/BigComputeLabs/tree/master/Azure%20Batch%20Masterclass%20Labs
[batch-scaling]: /azure/batch/batch-automatic-scaling
[hpc-alt-solutions]: /azure/virtual-machines/linux/high-performance-computing?toc=%2fazure%2fbatch%2ftoc.json
[batch-monitor]: /azure/batch/monitoring-overview
[batch-pricing]: https://azure.microsoft.com/en-gb/pricing/details/batch/
[batch-doc]: /azure/batch/
[batch-overview]: https://azure.microsoft.com/services/batch/
[batch-containers]: https://github.com/Azure/batch-shipyard
[azure-arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[batch-plugins]: /azure/batch/batch-rendering-service#options-for-submitting-a-render-job