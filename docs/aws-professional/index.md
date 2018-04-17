---
title: 面向 AWS 专业人员的 Azure
description: 了解 Microsoft Azure 帐户、平台和服务的基础知识。 另外，了解 AWS 与 Azure 平台之间的重要相似之处和差别。 在 Azure 中利用 AWS 方面的经验。
keywords: AWS 专家, Azure 比较, AWS 比较, azure 与 aws 之间的差别, azure 与 aws
author: lbrader
ms.date: 03/24/2017
pnp.series.title: Azure for AWS Professionals
ms.openlocfilehash: 0af0890d383d22db0ed9d3b445cdd5b561b498ae
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/16/2018
---
# <a name="azure-for-aws-professionals"></a>面向 AWS 专业人员的 Azure

本文可帮助 Amazon Web Services (AWS) 专家了解 Microsoft Azure 帐户、平台和服务的基础知识。 其中介绍了 AWS 与 Azure 平台之间的重要相似之处和差别。

学习内容：

* 帐户和资源在 Azure 中的组织方式。
* 如何在 Azure 中构建可用的解决方案。
* 如何 Azure 服务与 AWS 服务之间有哪些重要差别。

长久以来，Azure 和 AWS 各自建立了自身的功能，因此，两者的实施方式和设计具有重要差别。

## <a name="overview"></a>概述

像 AWS 一样，Microsoft Azure 是围绕一组核心计算、存储、数据库和网络服务构建的。 在许多情况下，这两个平台在所提供的产品和服务方面具有基本的相似性。 AWS 和 Azure 都允许基于 Windows 或 Linux 主机构建高可用性解决方案。 因此，如果你习惯于使用 Linux 和 OSS 技术进行开发，这两个平台都可满足要求。

尽管这两个平台的功能类似，但提供这些功能的资源通常以不同的方式进行组织。 用于构建解决方案的服务之间的具体一对一关系有时并不明确。 另外，在某些情况下，特定的服务可能已在其中一个平台上提供，但没有在另一个平台上提供。 请参阅[类似的 Azure 和 AWS 服务图表](services.md)。

## <a name="accounts-and-subscriptions"></a>帐户和订阅

可以根据组织的规模和需求，以多个定价选项购买 Azure 服务。 有关详细信息，请参阅[定价概述](https://azure.microsoft.com/pricing/)页。

[Azure 订阅](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-infrastructure-subscription-accounts-guidelines/)是一组资源，其中分配的所有者负责进行计费和权限管理。 订阅是独立于其所有者帐户存在的，可根据需要将其重新分配给新的所有者；而 AWS 则与此不同，其中，在 AWS 帐户下创建的所有资源会绑定到该帐户。

![AWS 帐户与 Azure 订阅的结构和所有权比较](./images/azure-aws-account-compare.png "AWS 帐户与 Azure 订阅的结构和所有权比较")
<br/>*AWS 帐户与 Azure 订阅的结构和所有权比较*
<br/><br/>

订阅分配给三种类型的管理员帐户：

-   **帐户管理员** - 针对订阅中所用资源计费的订阅所有者和帐户。 只能通过转让订阅所有权来更改帐户管理员。

-   **服务管理员** - 此帐户有权在订阅中创建和管理资源，但不负责计费。 默认情况下，会将帐户管理员和服务管理员分配到同一个帐户。 帐户管理员可将一个单独的用户分配到服务管理员帐户，用于管理订阅的技术和操作方面。 每个订阅只有一个服务管理员。

-   **协同管理员** - 可将多个协同管理员帐户分配到一个订阅。 协同管理员无法更改服务管理员，但对订阅资源和用户拥有完全控制权。

在订阅级别下，还可将用户角色和单个权限分配到特定的资源，类似于在 AWS 中向 IAM 用户和组授予权限。 在 Azure 中，所有用户帐户都与 Microsoft 帐户或组织帐户（通过 Azure Active Directory 管理的帐户）相关联。

与 AWS 帐户一样，订阅具有默认的服务配额和限制。 有关这些限制的完整列表，请参阅 [Azure 订阅和服务限制、配额与约束](https://azure.microsoft.com/documentation/articles/azure-subscription-service-limits/)。
可以通过[在管理门户中提出支持请求](https://blogs.msdn.microsoft.com/girishp/2015/09/20/increasing-core-quota-limits-in-azure/)，将这些限制提高到最大值。

### <a name="see-also"></a>另请参阅

-   [如何添加或更改 Azure 管理员角色](https://azure.microsoft.com/documentation/articles/billing-add-change-azure-subscription-administrator/)

-   [如何下载 Azure 帐单发票和每日使用数据](https://azure.microsoft.com/documentation/articles/billing-download-azure-invoice-daily-usage-date/)

## <a name="resource-management"></a>资源管理

在 Azure 中，术语“资源”的用法与在 AWS 中一样，表示可在平台中创建或配置的任何计算实例、存储对象、网络设备或其他实体。

可使用以下两种模型之一部署和管理 Azure 资源：[Azure 资源管理器模型](/azure/azure-resource-manager/resource-group-overview)或早期的 Azure [经典部署模型](/azure/azure-resource-manager/resource-manager-deployment-model)。
任何新资源都是使用资源管理器模型创建的。

### <a name="resource-groups"></a>资源组

Azure 和 AWS 中都包含称作“资源组”的实体，这些实体用于组织 VM、存储和虚拟网络设备等资源。 但是，不能直接将 [Azure 资源组](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/)与 AWS 资源组进行比较。

AWS 允许在多个资源组中标记某个资源，而一个 Azure 资源始终与一个资源组相关联。 在一个资源组中创建的资源可以转移到另一个组，但始终只能在一个资源组中。 资源组是 Azure 资源管理器使用的基本分组方式。

也可以使用[标记](https://azure.microsoft.com/documentation/articles/resource-group-using-tags/)来组织资源。
标记是一些键值对，用于将整个订阅中的资源分组，不管资源组的成员身份如何。

### <a name="management-interfaces"></a>管理界面

Azure 提供多种方式用于管理资源：

-   [Web 界面](https://azure.microsoft.com/documentation/articles/resource-group-portal/)。
    Azure 门户针对 Azure 资源提供类似于 AWS 仪表板的基于 Web 的完整管理界面。

-   [REST API](https://azure.microsoft.com/documentation/articles/resource-manager-rest-api/)。
    使用 Azure 资源管理器 REST API 能以编程方式访问 Azure 门户中提供的大部分功能。

-   [命令行](https://azure.microsoft.com/documentation/articles/xplat-cli-azure-resource-manager/)。
    Azure CLI 2.0 工具提供一个可以创建和管理 Azure 资源的命令行接口。 Azure CLI 适用于 [Windows、Linux 和 Mac OS](https://aka.ms/azurecli2)。

-   [PowerShell](https://azure.microsoft.com/documentation/articles/powershell-azure-resource-manager/)。
    借助 PowerShell 的 Azure 模块，可以使用脚本执行自动化管理任务。 PowerShell 适用于 [Windows、Linux 和 Mac OS](https://github.com/PowerShell/PowerShell)。

-   [模板](https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/)。
    Azure 资源管理器模板提供类似于 AWS CloudFormation 服务的基于 JSON 模板的资源管理功能。

在其中每个接口中，都是围绕资源组创建、部署或修改 Azure 资源。 这类似于在执行 CloudFormation 部署过程中，“堆栈”在分组 AWS 资源时所扮演的角色。

这些接口的语法和结构与其 AWS 等效接口不同，但提供类似的功能。 此外，在 AWS 中使用的许多第三方管理工具，例如 [Hashicorp 的 Terraform](https://www.terraform.io/docs/providers/azurerm/) 和 [Netflix Spinnaker](http://www.spinnaker.io/)，在 Azure 中也可用。

### <a name="see-also"></a>另请参阅

-   [Azure 资源组准则](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/)

## <a name="regions-and-zones-high-availability"></a>区域 (Region) 和局部区域 (Zone)（高可用性）

故障的影响范围各不相同。 某些硬件故障（例如磁盘故障）可能影响单个主机。 网络交换机故障可能影响整个服务器机架。 中断整个数据中心的故障（例如数据中心断电）不太常见。 在极少数情况下，整个区域可能不可用。

冗余是让应用程序保持弹性的方法之一。 但是，需要在计划应用程序时规划这种冗余。 此外，所需的冗余级别取决于业务要求 &mdash; 并非每个应用程序都需要跨区域的冗余才能防范区域性服务中断。 一般情况下，提高冗余和可靠性的弊端就是增大成本和复杂性。  

在 AWS 中，一个区域划分为两个或更多个可用性区域。 可用性区域对应于某个地理区域中物理隔离的数据中心。 为了使应用程序可在任何故障级别冗余，Azure 提供了许多功能，包括**可用性集**、**可用性区域**和**配对区域**。 

![](../resiliency/images/redundancy.svg)

下表汇总了各个选项。

| &nbsp; | 可用性集 | 可用性区域 | 配对区域 |
|--------|------------------|-------------------|---------------|
| 故障范围 | 机架 | 数据中心 | 区域 |
| 请求路由 | 负载均衡器 | 跨区域负载均衡器 | 流量管理器 |
| 网络延迟 | 极低 | 低 | 中到高 |
| 虚拟网络  | VNet | VNet | 跨区域 VNet 对等互连 |

### <a name="availability-sets"></a>可用性集 

若要防范局部性硬件故障（例如磁盘或网络交换机故障），请在可用性集中部署两个或更多个 VM。 可用性集包括两个或更多个容错域，它们共用一个电源和网络交换机。 可用性集中的 VM 分布在不同的容错域中，因此，如果硬件故障影响了一个容错域，仍可将网络流量路由到其他容错域中的 VM。 有关可用性集的详细信息，请参阅[在 Azure 中管理 Windows 虚拟机的可用性](/azure/virtual-machines/windows/manage-availability)。

将 VM 实例添加到可用性集后，还会为这些实例分配一个[更新域](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-manage-availability/)。 更新域是针对同时执行的计划内维护事件设置的一组 VM。 在多个更新域之间分配 VM 可以确保在任意给定时间，计划内更新和修补事件只会影响其中的一部分 VM。

应该根据实例在应用程序中的角色来组织可用性集，确保每个角色都有一个正常运行的实例。 例如，在三层 Web 应用程序中，为前端、应用程序和数据层创建单独的可用性集。

![每个应用程序角色的 Azure 可用性集](./images/three-tier-example.png "每个应用程序角色的可用性集")

### <a name="availability-zones"></a>可用性区域

[可用性区域](/azure/availability-zones/az-overview)是 Azure 区域中在物理上独立的区域。 每个可用性区域有独立的电源、网络和散热设备。 跨可用性区域部署 VM 有助于在发生数据中心范围的故障时保护应用程序。 

### <a name="paired-regions"></a>配对区域

若要使应用程序免受区域性服务中断的影响，可以[使用 Azure 流量管理器][traffic-manager]将 Internet 流量分配到不同的区域，从而跨多个区域部署应用程序。 每个 Azure 区域与另一个区域配对。 它们共同构成了[区域对][paired-regions]。 除巴西南部以外，区域对位于同一区域，以符合税务和执法管辖范围方面的数据驻留要求。

配对区域通常至少相隔 300 英里，这与可用性区域不同，后者虽然是在物理上独立的数据中心，但可能位于相对邻近的地理区域中。 保持这种远距离的目的是确保大规模的灾难只会影响配对中的一个区域。 可将邻近的配对设置为同步数据库和存储服务数据，并对其进行适当的配置，以便每次只将平台更新部署到配对中的一个区域。

Azure [异地冗余存储](https://azure.microsoft.com/documentation/articles/storage-redundancy/#geo-redundant-storage)会自动备份到相应的配对区域。 对于其他所有资源而言，使用配对区域创建完全冗余的解决方案意味着需要在两个区域中创建该解决方案的完整副本。


### <a name="see-also"></a>另请参阅

-   [Azure 中虚拟机的区域和可用性](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-regions-and-availability/)

-   [Azure 应用程序的高可用性](../resiliency/high-availability-azure-applications.md)

-   [Azure 应用程序的灾难恢复](../resiliency/disaster-recovery-azure-applications.md)

-   [Azure 中 Linux 虚拟机的计划内维护](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-planned-maintenance/)

## <a name="services"></a>服务

请参阅 [AWS 和 Azure 服务的完整比较对照表](https://aka.ms/azure4aws-services)，了解平台之间所有服务映射方式的完整列表。

并非所有 Azure 产品和服务在所有区域中都可用。 有关详细信息，请参阅[各区域中推出的产品](https://azure.microsoft.com/regions/services/)页。 可在[服务级别协议](https://azure.microsoft.com/support/legal/sla/)页上找到每个 Azure 产品或服务的运行时间保证以及停机时间信用政策。

以下部分提供 AWS 与 Azure 平台中常用功能和服务的差别的简要说明。

### <a name="compute-services"></a>计算服务

#### <a name="ec2-instances-and-azure-virtual-machines"></a>EC2 实例和 Azure 虚拟机

尽管 AWS 实例类型与 Azure 虚拟机大小的划分方式类似，但 RAM、CPU 和存储功能方面存在一些差别。

-   [Amazon EC2 实例类型](https://aws.amazon.com/ec2/instance-types/)

-   [Azure 中虚拟机的大小 (Windows)](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/)

-   [Azure 中虚拟机的大小 (Linux)](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-sizes/)

与 AWS 的按秒计费不同，Azure 按需 VM 是按分钟计费的。

Azure 不提供与 EC2 Spot 实例或专用主机相同的功能。

#### <a name="ebs-and-azure-storage-for-vm-disks"></a>EBS 与用作 VM 磁盘的 Azure 存储

Azure VM 的持久性数据存储是通过驻留在 Blob 存储中的[数据磁盘](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-about-disks-vhds/)提供的。 这类似于 EC2 实例在弹性块存储 (EBS) 中存储磁盘卷。 与 EC2 实例存储（也称为临时性存储）一样，[Azure 临时存储](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/)也为 VM 提供低延迟的临时读写存储。

使用 [Azure 高级存储](https://docs.microsoft.com/azure/storage/storage-premium-storage)可以支持更高性能的磁盘 IO。
这类似于 AWS 提供的预配 IOPS 存储选项。

#### <a name="lambda-azure-functions-azure-web-jobs-and-azure-logic-apps"></a>Lambda、Azure Functions、Azure Web 作业和 Azure 逻辑应用

[Azure Functions](https://azure.microsoft.com/services/functions/) 基本上相当于 AWS Lambda，提供无服务器的按需代码。
但是，Lambda 功能还与其他 Azure 服务重叠：

-   [Web 作业](https://azure.microsoft.com/documentation/articles/web-sites-create-web-jobs/) - 用于创建计划或连续运行后台任务。

-   [逻辑应用](https://azure.microsoft.com/services/logic-apps/) - 提供通信、集成和业务规则管理服务。

#### <a name="autoscaling-azure-vm-scaling-and-azure-app-service-autoscale"></a>自动缩放、Azure VM 缩放和 Azure 应用服务自动缩放

Azure 中的自动缩放由两个服务进行处理：

-   [VM 规模集](https://azure.microsoft.com/documentation/articles/virtual-machine-scale-sets-overview/) - 用于部署和管理一组相同的 VM。 可以根据性能需求自动缩放实例数。

-   [应用服务自动缩放](https://azure.microsoft.com/documentation/articles/web-sites-scale/) - 提供自动缩放 Azure 应用服务解决方案的功能。


#### <a name="container-service"></a>容器服务
[Azure 容器服务](https://docs.microsoft.com/azure/container-service/container-service-intro)支持通过 Docker Swarm、Kubernetes 或 DC/OS 管理的 Docker 容器。

#### <a name="other-compute-services"></a>其他计算服务 


Azure 提供多个计算服务，在 AWS 中没有直接与此类似的服务：

-   [Azure Batch](https://azure.microsoft.com/documentation/articles/batch-technical-overview/) - 用于跨可缩放的虚拟机集合管理计算密集型工作。

-   [Service Fabric](https://azure.microsoft.com/documentation/articles/service-fabric-overview/) - 用于开发和托管可缩放[微服务](https://azure.microsoft.com/documentation/articles/service-fabric-overview-microservices/)解决方案的平台。

#### <a name="see-also"></a>另请参阅

-   [使用门户在 Azure 上创建 Linux VM](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-quick-create-portal/)

-   [Azure 参考体系结构：在 Azure 上运行 Linux VM](https://azure.microsoft.com/documentation/articles/guidance-compute-single-vm-linux/)

-   [Azure 应用服务中的 Node.js Web 应用入门](https://azure.microsoft.com/documentation/articles/app-service-web-nodejs-get-started/)

-   [Azure 参考体系结构：基本 Web 应用程序](https://azure.microsoft.com/documentation/articles/guidance-web-apps-basic/)

-   [创建第一个 Azure 函数](https://azure.microsoft.com/documentation/articles/functions-create-first-azure-function/)

### <a name="storage"></a>存储

#### <a name="s3ebsefs-and-azure-storage"></a>S3/EBS/EFS 与 Azure 存储

在 AWS 平台中，云存储主要划分为以下三个服务：

-   **简单存储服务 (S3)** - 基本对象存储。 借助该服务能够通过可在 Internet 上访问的 API 使用数据。

-   **弹性块存储 (EBS)** - 块级存储，旨在由单个 VM 访问。

-   **弹性文件系统 (EFS)** - 供多达数千个 EC2 实例用作共享存储的文件存储。

在 Azure 存储中，使用已绑定到订阅的[存储帐户](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/)可以创建和管理以下存储服务：

-   [Blob 存储](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/) - 存储任何类型的文本或二进制数据，例如文档、媒体文件或应用程序安装程序。 可以设置 Blob 存储进行私人访问，或者在 Internet 上公开共享内容。 Blob 存储的用途与 AWS S3 和 EBS 相同。
-   [表存储](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/) - 存储结构化数据集。 表存储是一个 NoSQL“键-属性”数据存储，用于实现快速开发以及快速访问大量数据。 类似于 AWS 的 SimpleDB 和 DynamoDB 服务。

-   [队列存储](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) - 为工作流处理和云服务组件之间的通信提供消息传送。

-   [文件存储](https://azure.microsoft.com/documentation/articles/storage-java-how-to-use-file-storage/) - 使用标准服务器消息块 (SMB) 协议为传统应用程序提供共享存储。 文件存储的使用方式与 AWS 平台中的 EFS 类似。
 
#### <a name="glacier-and-azure-storage"></a>Glacier 与 Azure 存储 

[Azure 存档 Blob 存储](/azure/storage/blobs/storage-blob-storage-tiers#archive-access-tier)相当于 AWS Glacier 存储服务。 它适用于已至少存储 180 天且极少访问的数据，并且可以容忍几个小时的检索延迟。 

针对不经常访问，但访问时必须立即提供的数据，[Azure 冷 Blob 存储层](/azure/storage/blobs/storage-blob-storage-tiers#cool-access-tier)提供比标准 Blob 存储更经济节省的存储。 此存储层相当于 AWS S3 - Infrequent Access 存储服务。

#### <a name="see-also"></a>另请参阅

-   [Microsoft Azure 存储性能和可伸缩性清单](https://azure.microsoft.com/documentation/articles/storage-performance-checklist/)

-   [Azure 存储安全指南](https://azure.microsoft.com/documentation/articles/storage-security-guide/)

-   [模式和做法：内容交付网络 (CDN) 指南](https://azure.microsoft.com/documentation/articles/best-practices-cdn/)

### <a name="networking"></a>网络

#### <a name="elastic-load-balancing-azure-load-balancer-and-azure-application-gateway"></a>弹性负载均衡、Azure 负载均衡器和 Azure 应用程序网关

Azure 提供两个类似于弹性负载均衡的服务：

-   [负载均衡器](https://azure.microsoft.com/documentation/articles/load-balancer-overview/) - 提供与 AWS 经典负载均衡器相同的功能，可用于在网络级别分配多个 VM 的流量。 此外，它还提供故障转移功能。

-   [应用程序网关](https://azure.microsoft.com/documentation/articles/application-gateway-introduction/) - 提供应用程序级别的基于规则的路由，类似于 AWS 应用程序负载均衡器。

#### <a name="route-53-azure-dns-and-azure-traffic-manager"></a>Route 53、Azure DNS 和 Azure 流量管理器

在 AWS 中，Route 53 提供 DNS 名称管理，以及 DNS 级别的流量路由和故障转移服务。 在 Azure 中，这些功能通过两个服务进行处理：

-   [Azure DNS](https://azure.microsoft.com/documentation/services/dns/) 提供域和 DNS 管理。

-   [流量管理器][traffic-manager]提供 DNS 级流量路由、负载均衡和故障转移功能。

#### <a name="direct-connect-and-azure-expressroute"></a>直接连接和 Azure ExpressRoute

Azure 通过其 [ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/) 服务提供类似的站点到站点专用连接。 通过 ExpressRoute 可以使用专用网络连接将本地网络直接连接到 Azure 资源。 Azure 还以更低的价格提供更方便的[站点到站点 VPN 连接](https://azure.microsoft.com/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/)。

#### <a name="see-also"></a>另请参阅

-   [使用 Azure 门户创建虚拟网络](https://azure.microsoft.com/documentation/articles/virtual-networks-create-vnet-arm-pportal/)

-   [规划和设计 Azure 虚拟网络](https://azure.microsoft.com/documentation/articles/virtual-network-vnet-plan-design-arm/)

-   [Azure 网络安全最佳做法](https://azure.microsoft.com/documentation/articles/azure-security-network-security-best-practices/)

### <a name="database-services"></a>数据库服务

#### <a name="rds-and-azure-relational-database-services"></a>RDS 和 Azure 关系数据库服务

Azure 提供多个与 AWS 的关系数据库服务 (RDS) 功能相同的不同关系数据库服务。

-   [SQL 数据库](https://docs.microsoft.com/azure/sql-database/sql-database-technical-overview)
-   [Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/overview)
-   [Azure Database for PostgreSQL](https://docs.microsoft.com/azure/postgresql/overview)

可以使用 Azure VM 实例部署其他数据库引擎，例如 [SQL Server](https://azure.microsoft.com/services/virtual-machines/sql-server/)、[Oracle](https://azure.microsoft.com/campaigns/oracle/) 和 [MySQL](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-classic-mysql-2008r2/)。

AWS RDS 的费用根据实例使用的硬件资源确定，例如 CPU、RAM、存储和网络带宽。 在 Azure 数据库服务中，费用取决于数据库大小、并发连接数和吞吐量级别。

#### <a name="see-also"></a>另请参阅

-   [Azure SQL 数据库教程](https://azure.microsoft.com/documentation/articles/sql-database-explore-tutorials/)

-   [使用 Azure 门户为 Azure SQL 数据库配置异地复制](https://azure.microsoft.com/documentation/articles/sql-database-geo-replication-portal/)

-   [DocumentDB 简介：一种 NoSQL JSON 数据库](/azure/cosmos-db/sql-api-introduction)

-   [如何通过 Node.js 使用 Azure 表存储](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/)

### <a name="security-and-identity"></a>安全和标识

#### <a name="directory-service-and-azure-active-directory"></a>目录服务和 Azure Active Directory

Azure 将目录服务划分为以下服务：

-   [Azure Active Directory](https://azure.microsoft.com/documentation/articles/active-directory-whatis/) - 基于云的目录和标识管理服务。

-   [Azure Active Directory B2B](https://azure.microsoft.com/documentation/articles/active-directory-b2b-collaboration-overview/) - 用于从合作伙伴管理的标识访问你的企业应用程序。

-   [Azure Active Directory B2C](https://azure.microsoft.com/documentation/articles/active-directory-b2c-overview/) - 针对消费型应用程序提供单一登录和用户管理支持的服务。

-   [Azure Active Directory 域服务](https://azure.microsoft.com/documentation/articles/active-directory-ds-overview/) - 托管的域控制器服务，可实现与 Active Directory 兼容的域加入和用户管理功能。

#### <a name="web-application-firewall"></a>Web 应用程序防火墙

除了[应用程序网关 Web 应用程序防火墙](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/)以外，还可以使用来自 [Barracuda Networks](https://azure.microsoft.com/marketplace/partners/barracudanetworks/waf/) 等第三方供应商的 [Web 应用程序防火墙](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/)。

#### <a name="see-also"></a>另请参阅

-   [Microsoft Azure 安全入门](https://azure.microsoft.com/documentation/articles/azure-security-getting-started/)

-   [Azure 标识管理和访问控制安全最佳实践](https://azure.microsoft.com/documentation/articles/azure-security-identity-management-best-practices/)

### <a name="application-and-messaging-services"></a>应用程序和消息传送服务

#### <a name="simple-email-service"></a>简单电子邮件服务

AWS 提供简单电子邮件服务 (SES) 用于发送通知、交易或市场营销电子邮件。 在 Azure 中，[Sendgrid](https://sendgrid.com/partners/azure/) 等第三方解决方案可提供电子邮件服务。

#### <a name="simple-queueing-service"></a>简单队列服务

AWS 简单队列服务 (SQS) 提供一个消息传送系统用于连接 AWS 平台中的应用程序、服务和设备。 在 Azure 中，有两个服务提供类似的功能：

-   [队列存储](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) - 一个云消息传送服务，可在 Azure 平台中的应用程序组件之间实现通信。

-   [服务总线](https://azure.microsoft.com/services/service-bus/) - 一个更可靠的消息传送系统，用于连接应用程序、服务和设备。 使用相关的[服务总线中继](https://docs.microsoft.com/azure/service-bus-relay/relay-what-is-it)，服务总线还可以连接到远程托管的应用程序和服务。

#### <a name="device-farm"></a>设备场

AWS 设备场提供跨设备测试服务。 在 Azure 中，[Xamarin Test Cloud](https://www.xamarin.com/test-cloud) 针对移动设备提供类似的跨设备前端测试。

除了前端测试以外，[Azure 开发测试实验室](https://azure.microsoft.com/services/devtest-lab/)还为 Linux 和 Windows 环境提供后端测试资源。

#### <a name="see-also"></a>另请参阅

-   [如何通过 Node.js 使用队列存储](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/)

-   [如何使用 Service Bus 队列](https://azure.microsoft.com/documentation/articles/service-bus-nodejs-how-to-use-queues/)

### <a name="analytics-and-big-data"></a>分析和大数据

[Cortana Intelligence Suite](https://azure.microsoft.com/suites/cortana-intelligence-suite/) 是 Azure 提供的产品和服务包，旨在捕获、组织、分析和可视化大量数据。 Cortana 套件包括以下服务：

-   [HDInsight](https://azure.microsoft.com/documentation/services/hdinsight/) - 包含 Hadoop、Spark、Storm 或 HBase 的托管 Apache 分发版。

-   [数据工厂](https://azure.microsoft.com/documentation/services/data-factory/) - 提供数据业务流程和数据管道功能。

-   [SQL 数据仓库](https://azure.microsoft.com/documentation/services/sql-data-warehouse/) - 大规模关系型数据存储。

-   [Data Lake Store](https://azure.microsoft.com/documentation/services/data-lake-store/) - 针对大数据分析工作负荷优化的大规模存储。

-   [机器学习](https://azure.microsoft.com/documentation/services/machine-learning/) - 用于基于数据生成和应用预测分析。

-   [流分析](https://azure.microsoft.com/documentation/services/stream-analytics/) - 实时数据分析。

-   [Data Lake Analytics](https://azure.microsoft.com/documentation/articles/data-lake-analytics-overview/) - 经过优化的、可与 Data Lake Store 配合使用的大规模分析服务

-   [PowerBI](https://powerbi.microsoft.com/) - 用于 Power 数据可视化。

#### <a name="see-also"></a>另请参阅

-   [Cortana Intelligence 库](https://gallery.cortanaintelligence.com/)

-   [了解 Microsoft 大数据解决方案](https://msdn.microsoft.com/library/dn749804.aspx)

-   [Azure Data Lake 和 Azure HDInsight 博客](https://blogs.msdn.microsoft.com/azuredatalake/)

### <a name="internet-of-things"></a>物联网

#### <a name="see-also"></a>另请参阅

-   [Azure IoT 中心入门](https://azure.microsoft.com/documentation/articles/iot-hub-csharp-csharp-getstarted/)

-   [IoT 中心与事件中心的比较](https://azure.microsoft.com/documentation/articles/iot-hub-compare-event-hubs/)

### <a name="mobile-services"></a>移动服务

#### <a name="notifications"></a>通知

通知中心不支持发送短信或电子邮件，因此，对于这些传送类型，需要使用第三方服务。

#### <a name="see-also"></a>另请参阅

-   [创建 Android 应用](https://azure.microsoft.com/documentation/articles/app-service-mobile-android-get-started/)

-   [Azure 移动应用中的身份验证和授权](https://azure.microsoft.com/documentation/articles/app-service-mobile-auth/)

-   [使用 Azure 通知中心发送推送通知](https://azure.microsoft.com/documentation/articles/notification-hubs-android-push-notification-google-fcm-get-started/)

### <a name="management-and-monitoring"></a>监视和管理

#### <a name="see-also"></a>另请参阅
-   [监视和诊断指南](https://azure.microsoft.com/documentation/articles/best-practices-monitoring/)

-   [有关创建 Azure 资源管理器模板的最佳做法](https://azure.microsoft.com/documentation/articles/resource-manager-template-best-practices/)

-   [Azure 资源管理器快速入门模板](https://azure.microsoft.com/documentation/templates/)


## <a name="next-steps"></a>后续步骤

-   [AWS 和 Azure 服务的完整比较对照表](https://aka.ms/azure4aws-services)

-   [交互式 Azure 平台概观](http://azureplatform.azurewebsites.net/)

-   [Azure 入门](https://azure.microsoft.com/get-started/)

-   [Azure 解决方案体系结构](https://azure.microsoft.com/solutions/architecture/)

-   [Azure 参考体系结构](https://azure.microsoft.com/documentation/articles/guidance-architecture/)

-   [模式和做法：Azure 指南](https://azure.microsoft.com/documentation/articles/guidance/)

-   [免费在线课程：面向 AWS 专家的 Microsoft Azure](http://aka.ms/azureforaws)


<!-- links -->

[paired-regions]: https://azure.microsoft.com/documentation/articles/best-practices-availability-paired-regions/
[traffic-manager]: /azure/traffic-manager/