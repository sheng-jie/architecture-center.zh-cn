---
title: 选择 Azure 计算服务的条件
description: 跨多个轴比较 Azure 计算服务
author: MikeWasson
layout: LandingPage
ms.date: 06/13/2018
ms.openlocfilehash: 29c21c44bdf3a3bfa29f17015565eecf5f86163b
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206531"
---
# <a name="criteria-for-choosing-an-azure-compute-service"></a>选择 Azure 计算服务的条件

术语“计算”指的是计算资源（应用程序在这些资源上运行）的承载模型。 下表在多个轴之间比较 Azure 计算服务。 为应用程序选择计算选项时，请参阅这些表。

## <a name="hosting-model"></a>托管模型

| 条件 | 虚拟机 | 应用服务 | Service Fabric | Azure Functions | Azure 容器服务 | 容器实例 | Azure 批处理 |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| 应用程序组合 | 不可知 | 应用程序、容器 | 服务、来宾可执行文件、容器 | 函数 | 容器 | 容器 | 计划的作业  |
| 密度 | 不可知 | 通过应用计划，每个实例多个应用 | 每个 VM 多个服务 | 没有专用实例 <a href="#note1"><sup>1</sup></a> | 每个 VM 多个容器 |无专用实例 | 每个 VM 多个应用 |
| 最小节点数 | 1 <a href="#note2"><sup>2</sup></a>  | 1 | 5 <a href="#note3"><sup>3</sup></a> | 没有专用节点 <a href="#note1"><sup>1</sup></a> | 3 | 无专用节点 | 1 <a href="#note4"><sup>4</sup></a> |
| 状态管理 | 无状态或有状态 | 无状态 | 无状态或有状态 | 无状态 | 无状态或有状态 | 无状态 | 无状态 |
| Web 托管 | 不可知 | 内置 | 不可知 | 不适用 | 不可知 | 不可知 | 否 |
| 操作系统 | Windows、Linux | Windows、Linux  | Windows、Linux | 不适用 | Windows（预览版）、Linux | Windows、Linux | Windows、Linux |
| 是否可部署到专用 VNet？ | 支持 | 受支持 <a href="#note5"><sup>5</sup></a> | 支持 | 不支持 | 支持 | 不支持 | 支持 |
| 混合连接 | 支持 | 受支持 <a href="#note1"><sup>6</sup></a>  | 支持 | 不支持 | 支持 | 不支持 | 支持 |

说明

1. <span id="note1">如果使用消耗计划。如果使用应用服务计划，Functions 在为应用服务计划分配的 VM 上运行。请参阅[为 Azure Functions 选择正确的服务计划][function-plans]</span>
2. <span id="note2">具有两个或多个实例的更高版本 SLA。</span>
3. <span id="note3">对于生产环境。</span>
4. <span id="note4">作业完成后可以缩小至零。</span>
5. <span id="note5">需要应用服务环境 (ASE)。</span>
6. <span id="note7">需要 ASE 或 BizTalk 混合连接</span>

## <a name="devops"></a>DevOps

| 条件 | 虚拟机 | 应用服务 | Service Fabric | Azure Functions | Azure 容器服务 | 容器实例 | Azure 批处理 |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| 本地调试 | 不可知 | IIS Express，其他<a href="#note1b"><sup>1</sup></a> | 本地节点群集 | Azure Functions CLI | 本地容器运行时 | 本地容器运行时 | 不支持 |
| 编程模型 | 不可知 | Web 应用程序、后台任务的 Web 作业 | 来宾可执行文件、服务模型、参与者模型、容器 | 具有触发器的 Functions | 不可知 | 不可知 | 命令行应用程序 |
| 应用程序更新 | 没有内置支持 | 部署槽 | 滚动升级（每个服务） | 没有内置支持 | 取决于业务流程协调程序。 大多数支持滚动更新 | 更新容器映像 | 不适用 |

说明

1. <span id="note1b">选项包括 IIS Express for ASP.NET 或 node.js (iisnode)；PHP web 服务器；Azure Toolkit for IntelliJ，Azure Toolkit for Eclipse。应用服务还支持对已部署的 web 应用进行远程调试。</span>
2. <span id="note2b">请参阅[资源管理器提供程序、区域、API 版本和架构][resource-manager-supported-services]。</span> 


## <a name="scalability"></a>可伸缩性

| 条件 | 虚拟机 | 应用服务 | Service Fabric | Azure Functions | Azure 容器服务 | 容器实例 | Azure 批处理 |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| 自动缩放 | VM 规模集 | 内置服务 | VM 规模集 | 内置服务 | 不支持 | 不支持 | 不适用 |
| 负载均衡 | Azure 负载均衡器 | 集成 | Azure 负载均衡器 | 集成 | Azure 负载均衡器 |  没有内置支持 | Azure 负载均衡器 |
| 缩放限制 | 平台映像：每个 VMSS 1000 个节点，自定义映像：每个 VMSS 100 个节点 | 20 个实例，应用服务环境为 50 | 每个 VMSS 100 个节点 | 无限 <a href="#note1c"><sup>1</sup></a> | 100 <a href="#note2c"><sup>2</sup></a> |每个订阅 20 个容器组（默认）。 若要增加数目，请联系客户服务。 <a href="#note3c"><sup>3</sup></a> | 默认的核心限制为 20。 若要增加数目，请联系客户服务。 |

说明

1. <span id="note1c">如果使用消耗计划。如果使用应用服务计划，则应用服务缩放限制适用。请参阅[为 Azure Functions 选择正确的服务计划][function-plans]</span>
2. <span id="note2c">请参阅[在容器服务群集中缩放代理节点][scale-acs]</span>。
3. <span id="note3c">请参阅 [Azure 容器实例的配额和区域可用性](/azure/container-instances/container-instances-quotas)。</span>


## <a name="availability"></a>可用性

| 条件 | 虚拟机 | 应用服务 | Service Fabric | Azure Functions | Azure 容器服务 | 容器实例 | Azure 批处理 |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SLA | [虚拟机的 SLA][sla-vm] | [应用服务的 SLA][sla-app-service] | [Service Fabric 的 SLA][sla-sf] | [Functions 的 SLA][sla-functions] | [Azure 容器服务的 SLA][sla-acs] | [容器实例的 SLA](https://azure.microsoft.com/support/legal/sla/container-instances/) | [Azure 批处理的 SLA][sla-batch] |
| 多区域故障转移 | 流量管理器 | 流量管理器 | 流量管理器，多区域群集 | 不支持  | 流量管理器 | 不支持 | 不支持 |

## <a name="other"></a>其他

| 条件 | 虚拟机 | 应用服务 | Service Fabric | Azure Functions | Azure 容器服务 | 容器实例 | Azure 批处理 |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SSL | 已在 VM 中配置 | 支持 | 支持  | 支持 | 已在 VM 中配置 | 支持与挎斗容器一起使用 | 支持 |
| 成本 | [Windows][cost-windows-vm][Linux][cost-linux-vm] | [应用服务定价][cost-app-service] | [Service Fabric 定价][cost-service-fabric] | [Azure Functions 定价][cost-functions] | [Azure 容器服务定价][cost-acs] | [容器实例定价](https://azure.microsoft.com/pricing/details/container-instances/) | [Azure 批处理定价][cost-batch]
| 合适的体系结构样式 | [N 层][n-tier]、[大计算][big-compute] (HPC) | [Web-队列-辅助角色][w-q-w] | [微服务][microservices]、[事件驱动型体系结构][event-driven] | [微服务][microservices]、[事件驱动型体系结构][event-driven] | [微服务][microservices]、[事件驱动型体系结构][event-driven] | [微服务][microservices]、任务自动化、批处理作业  | [大计算][big-compute] (HPC) |

[cost-linux-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/linux/
[cost-windows-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/windows/
[cost-app-service]: https://azure.microsoft.com/pricing/details/app-service/
[cost-service-fabric]: https://azure.microsoft.com/pricing/details/service-fabric/
[cost-functions]: https://azure.microsoft.com/pricing/details/functions/
[cost-acs]: https://azure.microsoft.com/pricing/details/container-service/
[cost-batch]: https://azure.microsoft.com/pricing/details/batch/

[function-plans]: /azure/azure-functions/functions-scale
[sla-acs]: https://azure.microsoft.com/support/legal/sla/container-service/
[sla-app-service]: https://azure.microsoft.com/support/legal/sla/app-service/
[sla-batch]: https://azure.microsoft.com/support/legal/sla/batch/
[sla-functions]: https://azure.microsoft.com/support/legal/sla/functions/
[sla-sf]: https://azure.microsoft.com/support/legal/sla/service-fabric/
[sla-vm]: https://azure.microsoft.com/support/legal/sla/virtual-machines/

[resource-manager-supported-services]: /azure/azure-resource-manager/resource-manager-supported-services
[scale-acs]: /azure/container-service/kubernetes/container-service-scale#scaling-considerations

[n-tier]: ../architecture-styles/n-tier.md
[w-q-w]: ../architecture-styles/web-queue-worker.md
[microservices]: ../architecture-styles/microservices.md
[event-driven]: ../architecture-styles/event-driven.md
[big-date]: ../architecture-styles/big-data.md
[big-compute]: ../architecture-styles/big-compute.md

