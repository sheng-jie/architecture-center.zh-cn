---
title: 在多个 Azure 区域中运行 Linux VM 以实现高可用性
description: 如何在 Azure 上的多个区域中部署 VM 以实现高可用性和复原能力。
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Linux VM workloads
pnp.series.prev: n-tier
ms.openlocfilehash: 07ccf44f28203e6d5001475b47adce01437e9600
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/30/2018
---
# <a name="run-linux-vms-in-multiple-regions-for-high-availability"></a>在多个区域中运行 Linux VM 以实现高可用性

此参考体系结构展示了在多个 Azure 区域中运行 N 层应用程序以实现可用性和强健的灾难恢复基础结构的一组经过实践检验的做法。 

![[0]][0]

下载此体系结构的 [Visio 文件][visio-download]。

## <a name="architecture"></a>体系结构 

此体系结构构建于[运行用于 N 层应用程序的 Linux VM](n-tier.md) 中所示的体系结构基础之上。 

* **主要和次要区域**。 使用两个区域来实现更高的可用性。 一个是主区域。另一个区域用于故障转移。
* **Azure DNS**。 [Azure DNS][azure-dns] 是 DNS 域的托管服务，它使用 Microsoft Azure 基础结构提供名称解析。 通过在 Azure 中托管域，可以使用与其他 Azure 服务相同的凭据、API、工具和计费来管理 DNS 记录。
* **Azure 流量管理器**。 [流量管理器][traffic-manager]将传入请求路由到其中一个区域。 在正常运行期间，它将请求路由到主要区域。 如果该区域变得不可用，则流量管理器将故障转移到次要区域。 有关详细信息，请参阅[流量管理器配置](#traffic-manager-configuration)部分。
* **资源组**。 为主要区域、次要区域和流量管理器创建单独的[资源组][resource groups]。 这允许你将每个区域作为单个资源集合灵活进行管理。 例如，可以重新部署一个区域而无需关闭另一个区域。 [链接资源组][resource-group-links]，以便可以运行查询来列出应用程序的所有资源。
* **Vnet**。 为每个区域创建一个单独的 VNet。 请确保地址空间不重叠。
* **Apache Cassandra**。 在跨 Azure 区域的数据中心部署 Cassandra 以实现高可用性。 每个区域中，节点以带故障和升级域的机架感知模式配置，以实现区域内的复原能力。

## <a name="recommendations"></a>建议

与部署到单个区域相比，多区域体系结构可以提供更高的可用性。 如果区域性故障影响了主要区域，则可以使用[流量管理器][traffic-manager]故障转移到次要区域。 当应用程序的单个子系统出现故障时，此体系结构可能也比较有用。

有多种常规方法可跨区域实现高可用性：   

* 主动/被动（采用热备用模式）。 流量将前往一个区域，而另一个区域将以热备用模式等待。 “热备用模式”意味着次要区域中的 VM 已被分配并总是处于运行状态。
* 主动/被动（采用冷备用模式）。 流量将前往一个区域，而另一个区域将以冷备用模式等待。 “冷备用模式”意味着次要区域中的 VM 不会被分配，直到故障转移需要它们。 此方法的运行成本较低，但是当发生故障时通常需要花费更长时间才能联机。
* 主动/主动。 两个区域都处于活动状态，并且会在它们之间对请求进行负载均衡。 如果一个区域变得不可用，则不再使其参与轮换。 

此体系结构侧重于“主动/被动（采用热备用模式）”，使用流量管理器进行故障转移。 注意，可以为热备用模式部署少量 VM，然后根据需要横向扩展。


### <a name="regional-pairing"></a>区域配对

每个 Azure 区域都与同一地域内的另一个区域配对。 通常，请选择同一区域对中的区域（例如“美国东部 2”和“美国中部”）。 这样做的好处包括：

* 如果发生大范围的故障，会优先恢复每个区域对中的至少一个区域。
* 计划内 Azure 系统更新会按顺序提供给配对的区域，以尽可能减少停机时间。
* 区域对位于同一地域内，以满足数据驻留要求。

但请确保两个区域都支持应用程序所需的所有 Azure 服务（请参阅[每个区域的服务][services-by-region]）。 有关区域对的详细信息，请参阅[业务连续性和灾难恢复 (BCDR)：Azure 配对区域][regional-pairs]。

### <a name="traffic-manager-configuration"></a>流量管理器配置

配置流量管理器时，请考虑以下几点：

* **路由**。 流量管理器支持多个[路由算法][tm-routing]。 对于本文中所述的情况，请使用“优先级”路由（以前称为“故障转移”路由）。 使用此设置时，流量管理器将所有请求都发送到主要区域，除非主要区域变得无法访问。 那时，它将自动故障转移到次要区域。 请参阅[配置故障转移路由方法][tm-configure-failover]。
* **运行状况探测**。 流量管理器使用 HTTP（或 HTTPS）[探测][tm-monitoring]来监视每个区域的可用性。 探测检查指定 URL 路径的 HTTP 200 响应。 作为最佳做法，请创建一个用于报告应用程序整体运行状况的终结点，并使用此终结点进行运行状况探测。 否则，探测可能会在应用程序的关键部分实际上已出现故障时报告终结点运行状况正常。 有关详细信息，请参阅[运行状况终结点监视模式][health-endpoint-monitoring-pattern]。

当流量管理器进行故障转移时，一段时间内客户端将无法访问应用程序。 持续时间受以下因素影响：

* 运行状况探测必须检测主要区域是否变得无法访问。
* DNS 服务器必须更新 IP 地址的已缓存 DNS 记录，这取决于 DNS 生存时间 (TTL)。 默认 TTL 为 300 秒（5 分钟），但可以在创建流量管理器配置文件时配置此值。

相关详细信息，请参阅[关于流量管理器监视][tm-monitoring]。

如果流量管理器进行故障转移，我们建议执行手动故障回复，而不是实施自动故障回复。 否则，可能会造成应用程序在区域之间来回转移的情况。 在进行故障回复之前，请验证是否所有应用程序子系统的运行状况都正常。

请注意，默认情况下，流量管理器会自动进行故障回复。 若要禁止此操作，请在发生故障转移事件后手动降低主要区域的优先级。 例如，假设主要区域的优先级为 1，次要区域的优先级为 2。 在故障转移后，请将主要区域的优先级设置为 3，以禁止自动故障回复。 当准备好切换回来时，请将优先级更新为 1。

以下 [Azure CLI][install-azure-cli] 命令更新优先级：

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --priority 3
```    

另一种方法是暂时禁用终结点，直到你准备好进行故障回复：

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --status Disabled
```    

可能需要重新部署某个区域中的资源，具体取决于故障转移原因。 在进行故障回复之前，请执行操作准备情况测试。 测试应当验证的事项如下所示：

* VM 是否已正确配置。 （所有必需的软件是否已安装、IIS 是否正在运行，等等。）
* 应用程序子系统运行状况是否正常。
* 功能测试。 （例如，是否可以从 Web 层访问数据库层。）

### <a name="cassandra-deployment-across-multiple-regions"></a>跨多个区域的 Cassandra 部署

Cassandra 数据中心是一组相关的数据节点，这些节点一起配置在群集中，以实现复制和工作负荷分离。

建议将 [DataStax Enterprise][datastax] 用于生产用途。 有关在 Azure 中运行 DataStax 的详细信息，请参阅[适用于 Azure 的 DataStax Enterprise 部署指南][cassandra-in-azure]。 以下常规建议也适用于任何 Cassandra 版本： 

* 将公共 IP 地址分配给每个节点。 这使群集可以使用 Azure 主干基础结构跨区域进行通信，以较低的成本提供高吞吐量。
* 使用相应的防火墙和网络安全组 (NSG) 配置保护节点，使流量只能在已知主机（包括客户端和其他群集节点）之间传输。 请注意，Cassandra 将不同的端口用于通信、OpsCenter、Spark 等等。 有关 Cassandra 中的端口用法，请参阅[配置防火墙端口访问权限][cassandra-ports]。
* 将 SSL 加密用于所有[客户端到节点][ssl-client-node]和[节点到节点][ssl-node-node]通信。
* 在区域中，请遵循 [Cassandra 建议](n-tier.md#cassandra)中的指导。

## <a name="availability-considerations"></a>可用性注意事项

使用复杂的 N 层应用时，可能不需要在次要区域中复制整个应用程序。 相反，可能只需复制支持业务连续性所需的关键子系统。

流量管理器是系统中的一个潜在故障点。 如果流量管理器服务出现故障，则客户端在停机期间无法访问应用程序。 查看[流量管理器 SLA][tm-sla]，然后确定仅使用流量管理器是否能满足业务对高可用性的需求。 如果不能，请考虑添加另一个流量管理解决方案作为故障回复机制。 如果 Azure 流量管理器服务出现故障，请将 DNS 中的 CNAME 记录更改为指向其他流量管理服务。 （此步骤必须手动执行，并且在 DNS 更改被传播之前，应用程序将不可用。）

对于 Cassandra 群集，要考虑的故障转移方案取决于应用程序使用的一致性级别，以及使用的副本数目。 有关 Cassandra 中的一致性级别和用法，请参阅[配置数据一致性][cassandra-consistency]和 [Cassandra：使用仲裁时要与多少个节点通信？][cassandra-consistency-usage] Cassandra 中的数据可用性取决于应用程序使用的一致性级别和复制机制。 有关 Cassandra 中的复制，请参阅 [NoSQL 数据库的数据复制说明][cassandra-replication]。

## <a name="manageability-considerations"></a>可管理性注意事项

更新部署时，请一次更新一个区域，以减少由于错误配置或应用程序中的错误而导致全局故障的可能性。

测试系统在发生故障时的复原能力。 下面是要测试的一些常见故障方案：

* 关闭 VM 实例。
* 对 CPU 和内存等资源进行压力测试。
* 断开网络连接/使网络传输出现延迟。
* 使进程崩溃。
* 使证书过期。
* 模拟硬件错误。
* 关闭域控制器上的 DNS 服务。

测量恢复时间，并验证它们是否满足你的业务要求。 另外还测试故障模式的组合。


<!-- Links -->
[hybrid-vpn]: ../hybrid-networking/vpn.md
[azure-dns]: /azure/dns/dns-overview
[cassandra-in-azure]: https://academy.datastax.com/resources/deployment-guide-azure
[cassandra-consistency]: http://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html
[cassandra-replication]: http://www.planetcassandra.org/data-replication-in-nosql-databases-explained/
[cassandra-consistency-usage]: https://medium.com/@foundev/cassandra-how-many-nodes-are-talked-to-with-quorum-also-should-i-use-it-98074e75d7d5#.b4pb4alb2
[cassandra-ports]: https://docs.datastax.com/en/datastax_enterprise/5.0/datastax_enterprise/sec/configFirewallPorts.html
[datastax]: https://www.datastax.com/products/datastax-enterprise
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[install-azure-cli]: /azure/xplat-cli-install
[regional-pairs]: /azure/best-practices-availability-paired-regions
[resource groups]: /azure/azure-resource-manager/resource-group-overview
[resource-group-links]: /azure/resource-group-link-resources
[services-by-region]: https://azure.microsoft.com/regions/#services
[ssl-client-node]: http://docs.datastax.com/en/cassandra/2.0/cassandra/security/secureSSLClientToNode_t.html
[ssl-node-node]: http://docs.datastax.com/en/cassandra/2.0/cassandra/security/secureSSLNodeToNode_t.html
[tablediff]: https://msdn.microsoft.com/library/ms162843.aspx
[tm-configure-failover]: /azure/traffic-manager/traffic-manager-configure-failover-routing-method
[tm-monitoring]: /azure/traffic-manager/traffic-manager-monitoring
[tm-routing]: /azure/traffic-manager/traffic-manager-routing-methods
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager/v1_0/
[traffic-manager]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[wsfc]: https://msdn.microsoft.com/library/hh270278.aspx
[0]: ./images/multi-region-application-diagram.png "Azure N 层应用程序的高可用性网络体系结构"
