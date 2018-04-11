---
title: 选择 Azure 计算选项的条件
description: 跨多个轴比较 Azure 计算服务。
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 36b57d1fb674b5a1452a0e8208de836963b2b01b
ms.sourcegitcommit: c53adf50d3a787956fc4ebc951b163a10eeb5d20
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/23/2017
---
# <a name="criteria-for-choosing-an-azure-compute-option"></a>选择 Azure 计算选项的条件

术语“计算”指的是计算资源（应用程序在这些资源上运行）的承载模型。 下表在多个轴之间比较 Azure 计算服务。 为应用程序选择计算选项时，请参阅这些表。

## <a name="hosting-model"></a>托管模型

| 条件 | 虚拟机 | 应用服务 | Service Fabric | Azure Functions | Azure 容器服务 | 云服务 | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| 应用程序组合 | 不可知 | 应用程序 | 服务、来宾可执行文件、容器 | 函数 | 容器 | 角色 | 计划的作业  |
| 密度 | 不可知 | 通过应用计划，每个实例多个应用 | 每个 VM 多个服务 | 无专用实例 <a href="#note1"><sup>1</sup></a> | 每个 VM 多个容器 | 每个 VM 一个角色实例 | 每个 VM 多个应用 |
| 最小节点数 | 1 <a href="#note2"><sup>2</sup></a>  | 1 | 5 <a href="#note3"><sup>3</sup></a> | 无专用节点 <a href="#note1"><sup>1</sup></a> | 3 | #N/A | 1 <a href="#note4"><sup>4</sup></a> |
| 状态管理 | 无状态或有状态 | 无状态 | 无状态或有状态 | 无状态 | 无状态或有状态 | 无状态 | 无状态 |
| Web 托管 | 不可知 | 内置 | 不可知 | 不适用 | 不可知 | 内置 (IIS) | 否 |
| 操作系统 | Windows、Linux | Windows、Linux  | Windows、Linux | 不适用 | Windows（预览版）、Linux | Windows | Windows、Linux |
| 是否可部署到专用 VNet？ | 支持 | 支持 <a href="#note5"><sup>5</sup></a> | 支持 | 不支持 | 支持 | 支持 <a href="#note6"><sup>6</sup></a> | 支持 |
| 混合连接 | 支持 | 支持 <a href="#note1"><sup>7</sup></a>  | 支持 | 不支持 | 支持 | 支持 <a href="#note8"><sup>8</sup></a> | 支持 |

说明

1. <span id="note1">如果使用消耗计划。 如果使用应用服务计划，Functions 在为应用服务计划分配的 VM 上运行。 请参阅[为 Azure Functions 选择正确的服务计划][function-plans]</a>
2. <span id="note2">具有两个或多个实例的更高版本 SLA。</a>
3. <span id="note3">对于生产环境。</a>
4. <span id="note4">作业完成后可以缩小至零。</a>
5. <span id="note5">需要应用服务环境 (ASE)。</a>
6. <span id="note6">仅经典 VNet。</a>
7. <span id="note7">需要 ASE 或 BizTalk 混合连接</a>
8. <span id="note8">经典 VNet 或通过 VNet 对等互连的资源管理器 VNet</a>

## <a name="devops"></a>DevOps

| 条件 | 虚拟机 | 应用服务 | Service Fabric | Azure Functions | Azure 容器服务 | 云服务 | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| 本地调试 | 不可知 | IIS Express、其他 <a href="#note1b"><sup>1</sup></a> | 本地节点群集 | Azure Functions CLI | 本地容器运行时 | 本地仿真器 | 不支持 |
| 编程模型 | 不可知 | Web 应用程序、后台任务的 Web 作业 | 来宾可执行文件、服务模型、参与者模型、容器 | 具有触发器的 Functions | 不可知 | Web 角色，辅助角色 | 命令行应用程序 |
| 资源管理器 | 支持 | 支持 | 支持 | 支持 | 支持 | 受限制 <a href="#note2b"><sup>2</sup></a> | 支持 |  
| 应用程序更新 | 没有内置支持 | 部署槽 | 滚动升级（每个服务） | 没有内置支持 | 取决于业务流程协调程序。 大多数支持滚动更新 | VIP 交换或滚动更新 | 不适用 |

说明

1. <span id="note1b">选项包括 IIS Express for ASP.NET 或 node.js (iisnode)；PHP web 服务器；Azure Toolkit for IntelliJ，Azure Toolkit for Eclipse。 应用服务还支持对已部署的 web 应用进行远程调试。</a>
2. <span id="note2b">请参阅[资源管理器提供程序、区域、API 版本和架构][resource-manager-supported-services]。 


## <a name="scalability"></a>可伸缩性

| 条件 | 虚拟机 | 应用服务 | Service Fabric | Azure Functions | Azure 容器服务 | 云服务 | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| 自动缩放 | VM 规模集 | 内置服务 | VM 规模集 | 内置服务 | 不支持 | 内置服务 | 不适用 |
| 负载均衡器 | Azure 负载均衡器 | 集成 | Azure 负载均衡器 | 集成 | Azure 负载均衡器 | 集成 | Azure 负载均衡器 |
| 缩放限制 | 平台映像：每个 VMSS 1000 个节点，自定义映像：每个 VMSS 100 个节点 | 20 个实例，应用服务环境为 50 | 每个 VMSS 100 个节点 | 无限 <a href="#note1c"><sup>1</sup></a> | 100 | 没有明确的限制，建议最大值为 200 | 默认的核心限制为 20。 若要增加数目，请联系客户服务。 |

说明

1. <span id="note1c">如果使用消耗计划。 如果使用应用服务计划，则应用服务缩放限制适用。 请参阅[为 Azure Functions 选择正确的服务计划][function-plans]</a>

## <a name="availability"></a>可用性

| 条件 | 虚拟机 | 应用服务 | Service Fabric | Azure Functions | Azure 容器服务 | 云服务 | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SLA | [虚拟机的 SLA][sla-vm] | [应用服务的 SLA][sla-app-service] | [Service Fabric 的 SLA][sla-sf] | [Functions 的 SLA][sla-functions] | [Azure 容器服务的 SLA][sla-acs] | [云服务的 SLA][sla-cloud-service] | [Azure 批处理的 SLA][sla-batch] |
| 多区域故障转移 | 流量管理器 | 流量管理器 | 流量管理器，多区域群集 | 不支持  | 流量管理器 | 流量管理器 | 不支持 |

## <a name="security"></a>安全

| 条件 | 虚拟机 | 应用服务 | Service Fabric | Azure Functions | Azure 容器服务 | 云服务 | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SSL | 已在 VM 中配置 | 支持 | 支持  | 支持 | 已在 VM 中配置 | 支持 | 支持 |
| RBAC | 支持 | 支持 | 支持 | 支持 | 支持 | 不支持 | 支持 |

## <a name="other"></a>其他

| 条件 | 虚拟机 | 应用服务 | Service Fabric | Azure Functions | Azure 容器服务 | 云服务 | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| 成本 | [Windows][cost-windows-vm][Linux][cost-linux-vm] | [应用服务定价][cost-app-service] | [Service Fabric 定价][cost-service-fabric] | [Azure Functions 定价][cost-functions] | [Azure 容器服务定价][cost-acs] | [云服务定价][cost-cloud-services] | [Azure 批处理定价][cost-batch]
| 合适的体系结构样式 | N 层、大计算 (HPC) | Web 队列辅助角色 | 微服务，事件驱动的体系结构 (EDA) | 微服务，EDA | 微服务，EDA | Web 队列辅助角色 | 大计算 |

[cost-linux-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/linux/
[cost-windows-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/windows/
[cost-app-service]: https://azure.microsoft.com/pricing/details/app-service/
[cost-service-fabric]: https://azure.microsoft.com/pricing/details/service-fabric/
[cost-functions]: https://azure.microsoft.com/pricing/details/functions/
[cost-acs]: https://azure.microsoft.com/pricing/details/container-service/
[cost-cloud-services]: https://azure.microsoft.com/pricing/details/cloud-services/
[cost-batch]: https://azure.microsoft.com/pricing/details/batch/

[function-plans]: /azure/azure-functions/functions-scale
[sla-acs]: https://azure.microsoft.com/support/legal/sla/container-service/
[sla-app-service]: https://azure.microsoft.com/support/legal/sla/app-service/
[sla-batch]: https://azure.microsoft.com/support/legal/sla/batch/
[sla-cloud-service]: https://azure.microsoft.com/support/legal/sla/cloud-services/
[sla-functions]: https://azure.microsoft.com/support/legal/sla/functions/
[sla-sf]: https://azure.microsoft.com/support/legal/sla/service-fabric/
[sla-vm]: https://azure.microsoft.com/support/legal/sla/virtual-machines/

[resource-manager-supported-services]: /azure/azure-resource-manager/resource-manager-supported-services