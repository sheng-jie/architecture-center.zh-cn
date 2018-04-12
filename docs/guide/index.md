---
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 530844a0d3b1256cec807e7bad509a40dca304f6
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/06/2018
---
# <a name="azure-application-architecture-guide"></a>Azure 应用程序体系结构指南

本指南演示用于在 Azure 上设计可缩放、可复原且高度可用的应用程序的结构化方法。 该方法基于我们从客户互动中掌握的成熟做法。

<img src="./images/guide-steps.svg" style="max-width:800px;"/>

## <a name="introduction"></a>介绍

云正在改变应用程序的设计方式。 应用程序不再是庞大的单体结构，而是会分解成较小的分散式服务。 这些服务通过 API 或者使用异步消息传送或事件传送进行通信。 根据需要添加新的实例即可实现应用程序横向扩展。 

这些趋势带来了新的挑战。 应用程序状态是分布式的。 操作以并行和异步方式完成。 发生故障时，整个系统必须具有复原能力。 部署必须自动化且可预测。 监视和遥测对于洞察系统至关重要。 《Azure 应用程序体系结构指南》旨在帮助读者探索这些变革。 

<table>
<thead>
    <tr><th>传统的本地部署</th><th>现代云</th></tr>
</thead>
<tbody>
<tr><td>单体集中式<br/>
采用可预测的可伸缩性设计<br/>
关系型数据库<br/>
强一致性<br/>
串行和同步处理<br/>
防故障设计 (MTBF)<br/>
偶发性大规模更新<br/>
手动管理<br/>
雪花型服务器</td>
<td>
分解分散<br/>
采用弹性缩放设计<br/>
Polyglot 持久性（存储技术的混合）<br/>
最终一致性<br/>
并行和异步处理<br/>
防故障设计 (MTTR)<br/>
经常性小规模更新<br/>
自动自我管理<br/>
不可变的基础结构<br/>
</td>
</tbody>
</table>

本指南面向应用程序架构师、开发人员和运营团队。 它并不是一份介绍如何使用单个 Azure 服务的操作说明指南。 阅读本指南后，将会了解体系结构模式，以及有关在 Azure 云平台上生成解决方案时的最佳做法。 也可以下载[电子书版本的指南][ebook]。

## <a name="how-this-guide-is-structured"></a>本指南的结构

《Azure 应用程序体系结构指南》组织成一系列步骤：从体系结构和设计到实施。 每个步骤都有支持性的指导，可帮助设计应用程序体系结构。

**[体系结构样式][arch-styles]**。 第一个决策点至关重要。 要生成哪种类型的体系结构？ 它可能是微服务体系结构、更传统的 N 层应用程序，或大数据解决方案。 我们已识别了七种不同的体系结构样式。 这些样式各有利弊。

> &#10148; [Azure 参考体系结构][ref-archs]介绍了 Azure 中的建议部署，以及有关可伸缩性、可用性、可管理性和安全性的注意事项。 大多数体系结构还包含可部署的资源管理器模板。

**[技术选择][technology-choices]**。 应该尽早在两种技术选择中做出决定，因为它们会影响整个体系结构。 这两个选项是计算和存储技术。 术语“计算”指的是应用程序运行所在的计算资源的托管模型。 存储包括数据库，但也包括消息队列、缓存、IoT 数据、非结构化日志数据，以及应用程序可能持久存储的其他任何内容所用的存储。 

> &#10148; [计算选项][compute-options]和[存储选项][storage-options]提供了有关选择计算和存储服务的详细比较准则。

**[设计原则][design-principles]**。 在整个设计过程中，请牢记这十条高级设计原则。 

> &#10148; [最佳做法][best-practices]一文提供了有关自动缩放、缓存、数据分区、API 设计等方面的具体指导。   

**[要点][pillars]**。 一个成功的云应用程序应注重软件质量的五大构成要素：可伸缩性、可用性、复原能力、管理和安全性。 

> &#10148; 使用我们的[设计评审查检表][checklists]根据这些质量要点评审设计。 

**[云设计模式][patterns]**。 这些设计模式可用于在 Azure 中构建可靠且可缩放的安全应用程序。 每种模式描述了一个问题、用于解决该问题的模式，以及基于 Azure 的示例。

> &#10148; 查看完整的[云设计模式目录](../patterns/index.md)。


[arch-styles]: ./architecture-styles/index.md
[best-practices]: ../best-practices/index.md
[checklists]: ../checklist/index.md
[compute-options]: ./technology-choices/compute-comparison.md
[design-principles]: ./design-principles/index.md
[ebook]: https://azure.microsoft.com/campaigns/cloud-application-architecture-guide/
[patterns]: ../patterns/index.md?toc=/azure/architecture/guide/toc.json
[pillars]: ./pillars.md
[ref-archs]: ../reference-architectures/index.md
[storage-options]: ./technology-choices/data-store-comparison.md
[technology-choices]: ./technology-choices/index.md

