---
title: 用于开发/测试工作负荷的 SAP
description: 用于开发/测试环境的 SAP 方案
author: AndrewDibbins
ms.date: 7/11/18
ms.openlocfilehash: 675a5cb4b1ee4001ca50d24c145ce1a177f90da4
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060958"
---
# <a name="sap-for-devtest-workloads"></a>用于开发/测试工作负荷的 SAP

本示例介绍了如何运行一个开发/测试实现，以便在 Azure 上的 Windows 或 Linux 环境中实现 SAP NetWeaver。 使用的数据库为 AnyDB，这是一个 SAP 术语，指任何受支持的 DBMS（不是 SAP HANA）。 由于此体系结构设计用于非生产环境，因此在部署时只有一个虚拟机 (VM)，其大小可以根据你组织的需要进行更改。

对于生产用例，请查看下面提供的 SAP 参考体系结构：

* [适用于 AnyDB 的 SAP NetWeaver][sap-netweaver]
* [SAP S/4Hana][sap-hana]
* [Azure SAP 大型实例][sap-large]

## <a name="related-use-cases"></a>相关的用例

以下用例可以考虑本方案：

* 非关键性 SAP 非生产工作负荷（沙盒、开发、测试、质量保证）
* 非关键性 SAP 业务型工作负荷

## <a name="architecture"></a>体系结构

![图表](media/sap-2tier/SAP-Infra-2Tier_finalversion.png)

本方案介绍如何在单个虚拟机上预配单个 SAP 系统数据库和 SAP 应用程序服务器。数据流经方案的情形如下所示：

1. 展示层的客户使用其 SAP GUI 或本地的其他用户界面（Internet Explorer、Excel 或其他 Web 应用程序）来访问基于 Azure 的 SAP 系统。
2. 使用既定的 Express Route 来提供连接。 Express Route 在 Azure 中的 Express Route 网关处终止。 网络流量的路径是：通过 Express Route 网关到达网关子网，再从网关子网到达应用程序层辐射子网（参见[中心辐射][hub-spoke]模式），最后通过网络安全网关到达 SAP 应用程序虚拟机。
3. 标识管理服务器提供身份验证服务。
4. 跳转盒提供本地管理功能。

### <a name="components"></a>组件

* [资源组](/azure/azure-resource-manager/resource-group-overview#resource-groups)是 Azure 资源的逻辑容器
* [虚拟网络](/azure/virtual-network/virtual-networks-overview)是在 Azure 中进行通信的基础
* [虚拟机](/azure/virtual-machines/windows/overview)：Azure 虚拟机使用 Windows 或 Linux Server 按需提供具有高可伸缩性并且十分安全的虚拟化基础结构
* 使用 [Express Route](/azure/expressroute/expressroute-introduction) 可通过连接服务提供商所提供的专用连接，将本地网络扩展到 Microsoft 云。
* [网络安全组](/azure/virtual-network/security-overview)用于限制发往虚拟网络中的资源的网络流量。 网络安全组包含一个安全规则列表，这些规则可根据源或目标 IP 地址、端口和协议允许或拒绝入站或出站网络流量。 

## <a name="considerations"></a>注意事项

### <a name="availability"></a>可用性

 Microsoft 提供了用于单个 VM 实例的服务级别协议 (SLA)。 若要详细了解适用于虚拟机的 Microsoft Azure 服务级别协议，请参阅[虚拟机的 SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines)

### <a name="scalability"></a>可伸缩性

有关如何设计可缩放解决方案的通用指南，请参阅 Azure 体系结构中心的[可伸缩性核对清单][scalability]。

### <a name="security"></a>“安全”

若需安全解决方案的通用设计指南，请参阅 [Azure 安全性文档][security]。

### <a name="resiliency"></a>复原

若需可复原解决方案的通用设计指南，请参阅[设计适用于 Azure 的可复原应用程序][resiliency]。

## <a name="pricing"></a>定价

为了方便用户了解运行本方案的成本，我们已在成本计算器中预配置了所有服务。  若要了解自己的特定用例的定价变化情况，请按预期的流量更改相应的变量。

我们已根据你预期接收的流量提供了四个示例性的成本配置文件：

|大小|SAP|VM 类型|存储|Azure 定价计算器|
|----|----|-------|-------|---------------|
|小型|8000|D8s_v3|2xP20、1xP10|[小型](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1)|
|中型|16000|D16s_v3|3xP20、1xP10|[中型](https://azure.com/e/465bd07047d148baab032b2f461550cd)|
大型|32000|E32s_v3|3xP20、1xP10|[大型](https://azure.com/e/ada2e849d68b41c3839cc976000c6931)|
特大型|64000|M64s|4xP20、1xP10|[特大型](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef)|

注意：定价为指导价，仅表明 VM 和存储成本（不包括网络、备份存储和数据入口/出口费用）。

* [小](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1)：小型系统，其 VM 类型为 D8s_v3，使用 8x vCPU，32GB RAM 和 200GB 临时存储，另外还有两个 512GB 的和一个 128GB 的高级存储磁盘。
* [中](https://azure.com/e/465bd07047d148baab032b2f461550cd)：中型系统，其 VM 类型为 D16s_v3，使用 16x vCPU，64GB RAM 和 400GB 临时存储，另外还有三个 512GB 的和一个 128GB 的高级存储磁盘。
* [大](https://azure.com/e/ada2e849d68b41c3839cc976000c6931)：大型系统，其 VM 类型为 E32s_v3，使用 32x vCPU，256GB RAM 和 512GB 临时存储，另外还有三个 512GB 的和一个 128GB 的高级存储磁盘。
* [特大](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef)：特大型系统，其 VM 类型为 M64s，使用 64x vCPU，1024GB RAM 和 2000GB 临时存储，另外还有四个 512GB 的和一个 128GB 的高级存储磁盘。

## <a name="deployment"></a>部署

若要部署类似于上述方案的底层基础结构，请使用部署按钮

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-2tier%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

\* SAP 不会安装，需在手动生成基础结构后进行安装。

<!-- links -->
[reference architecture]:  /azure/architecture/reference-architectures/sap
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[sap-netweaver]: /azure/architecture/reference-architectures/sap/sap-netweaver
[sap-hana]: /azure/architecture/reference-architectures/sap/sap-s4hana
[sap-large]: /azure/architecture/reference-architectures/sap/hana-large-instances
[hub-spoke]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke