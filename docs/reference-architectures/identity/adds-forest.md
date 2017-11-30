---
title: "在 Azure 中创建 AD DS 资源林"
description: "如何在 Azure 中创建受信任的 Active Directory 域。\n指南,vpn 网关,ExpressRoute,负载均衡器,虚拟网络,active-directory"
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Identity management
pnp.series.prev: adds-extend-domain
pnp.series.next: adfs
cardTitle: Create an AD DS forest in Azure
ms.openlocfilehash: bb7e57af2afacf1faa7679c854bf49217918eba8
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="create-an-active-directory-domain-services-ad-ds-resource-forest-in-azure"></a>在 Azure 中创建 Active Directory 域服务 (AD DS) 资源林

此参考体系结构展示了如何在 Azure 中创建本地 AD 林中的域信任的一个单独 Active Directory 域。 [**部署此解决方案**。](#deploy-the-solution)

[![0]][0] 

*下载此体系结构的 [Visio 文件][visio-download]。*

Active Directory 域服务 (AD DS) 以分层结构存储标识信息。 分层结构中的顶层节点称为林。 林包含域，域包含其他类型的对象。 此参考体系结构在 Azure 中创建与本地域之间具有单向传出信任关系的 AD DS 林。 Azure 中的林包含本地不存在的一个域。 由于信任关系，将信任针对本地域的登录，允许其访问该单独 Azure 域中的资源。 

此体系结构的典型用途包括为云中拥有的对象和标识维护安全隔离，以及将各个域从本地迁移到云。 

有关其他注意事项，请参阅 [Choose a solution for integrating on-premises Active Directory with Azure][considerations]（选择用于将本地 Active Directory 与 Azure 进行集成的解决方案）。 

## <a name="architecture"></a>体系结构

该体系结构具有以下组件。

* **本地网络**。 本地网络包含其自己的 Active Directory 林和域。
* **Active Directory 服务器**。 它们是域控制器，用于实现在云中作为 VM 运行的域服务。 这些服务器托管的林中包含独立于本地域的一个或多个域。
* **单向信任关系**。 关系图中的示例显示了从 Azure 中的域到本地域的单向信任。 此关系允许本地用户访问 Azure 中的域的资源，但无法反向访问。 如果云用户也需要访问本地资源，则可以创建双向信任。
* **Active Directory 子网**。 AD DS 服务器托管在一个单独的子网中。 网络安全组 (NSG) 规则对 AD DS 服务器进行保护，并提供防火墙以阻止来自意外来源的流量。
* **Azure 网关**。 Azure 网关在本地网络与 Azure VNet 之间提供连接。 这可以是 [VPN 连接][azure-vpn-gateway]，也可以是 [Azure ExpressRoute][azure-expressroute]。 有关详细信息，请参阅[在 Azure 中实现安全的混合网络体系结构][implementing-a-secure-hybrid-network-architecture]。

## <a name="recommendations"></a>建议

有关在 Azure 中实现 Active Directory 的具体建议，请参阅以下文章：

- [将 Active Directory 域服务 (AD DS) 扩展到 Azure][adds-extend-domain]。 
- [在 Azure 虚拟机上部署 Windows Server Active Directory 的指南][ad-azure-guidelines]。

### <a name="trust"></a>信任

本地域包含在与云中的域不同的林中。 若要在云中启用对本地用户的身份验证，Azure 中的域必须信任本地林中的登录域。 同样，如果云为外部用户提供了登录域，则本地林可能需要信任云域。

可以通过[创建林信任][creating-forest-trusts]在林级别建立信任，或者通过[创建外部信任][creating-external-trusts]在域级别建立信任。 林级别信任在两个林中的所有域之间创建关系。 外部域级别信任仅在两个指定的域之间创建关系。 只应当在不同林中的域之间创建外部域级别信任。

信任可以是单向的，也可以是双向的：

* 单向信任允许一个域或林（称为*传入*域或林）中的用户访问另一个域或林（*传出*域或林）中拥有的资源。
* 双向信任允许任一域或林中的用户访问另一个域或林中拥有的资源。

下表总结了一些简单方案的信任配置：

| 方案 | 本地信任 | 云信任 |
| --- | --- | --- |
| 本地用户需要访问云中的资源，但云中的用户不需要访问本地资源 |单向、传入 |单向、传出 |
| 云中的用户需要访问本地资源，本地用户不需要访问云中的资源 |单向、传出 |单向、传入 |
| 云中的和本地用户都需要访问云中和本地拥有的资源。 |双向、传入和传出 |双向、传入和传出 |

## <a name="scalability-considerations"></a>可伸缩性注意事项

Active Directory 能够针对属于同一域的域控制器自动进行缩放。 请求被分布到域中的所有控制器中。 可以添加另一个域控制器，并且它将与该域自动同步。 不要配置单独的负载均衡器来将流量定向到该域中的控制器。 确保所有域控制器都有足够的内存和存储资源来处理域数据库。 使所有域控制器 VM 具有相同的大小。

## <a name="availability-considerations"></a>可用性注意事项

请至少为每个域预配两个域控制器。 这可以在服务器之间实现自动复制。 为充当 Active Directory 服务器（用于处理每个域）的 VM 创建一个可用性集。 至少在此可用性集中放置两个服务器。

另外，请考虑将每个域中的一个或多个服务器指定为[备用操作主机][standby-operations-masters]，以应对作为灵活单一主机操作 (FSMO) 角色连接到服务器时失败的情况。

## <a name="manageability-considerations"></a>可管理性注意事项

有关管理和监视注意事项的详细信息，请参阅[将 Active Directory 扩展到 Azure][adds-extend-domain]。 
 
有关详细信息，请参阅[监视 Active Directory][monitoring_ad]。 可以在管理子网中的监视服务器上安装 [Microsoft Systems Center][microsoft_systems_center] 之类的工具来帮助执行这些任务。

## <a name="security-considerations"></a>安全注意事项

林级别信任是可传递的。 如果在一个本地林与云中的一个林之间建立了林级别信任，则此信任将扩展到在任一林中创建的其他新域。 如果出于安全目的而使用域提供隔离，请考虑仅在域级别创建信任。 域级别信任是不可传递的。

有关特定于 Active Directory 的安全注意事项，请参阅[将 Active Directory 扩展到 Azure][adds-extend-domain] 中的安全注意事项部分。

## <a name="deploy-the-solution"></a>部署解决方案

[Github][github] 上提供了一个用于部署此参考体系结构的解决方案。 若要运行部署此解决方案的 Powershell 脚本，需要具有 Azure CLI 的最新版本。 若要部署此参考体系结构，请执行以下步骤：

1. 将解决方案文件夹从 [Github][github] 克隆到本地计算机。

2. 打开 Azure CLI 并导航到本地解决方案文件夹。

3. 运行以下命令：
   
    ```Powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```
   
    将 `<subscription id>` 替换为你的 Azure 订阅 ID。
   
    对于 `<location>`，请指定一个 Azure 区域，例如 `eastus` 或 `westus`。
   
    `<mode>` 参数控制部署粒度，可以是下列值之一：
   
   * `Onpremise`：部署模拟的本地环境。
   * `Infrastructure`：在 Azure 中部署 VNet 基础结构和 jumpbox。
   * `CreateVpn`：部署 Azure 虚拟网络网关并将其连接到模拟的本地网络。
   * `AzureADDS`：部署充当 Active Directory DS 服务器的 VM，将 Active Directory 部署到这些 VM，并在 Azure 中部署域。
   * `WebTier`：部署 Web 层 VM 和负载均衡器。
   * `Prepare`：部署上述所有部署。 **如果没有现有的本地网络，但是希望如上所述部署完整的参考体系结构以用于测试或评估，则这是建议使用的选项。** 
   * `Workload`：部署业务和数据层 VM 和负载均衡器。 注意，`Prepare` 部署中未包括这些 VM。

4. 等待部署完成。 如果要部署 `Prepare` 部署，则将需要花费几个小时。
     
5. 如果使用模拟的本地配置，请配置传入信任关系：
   
   1. 连接到 jumpbox（*ra-adtrust-security-rg* 资源组中的 *ra-adtrust-mgmt-vm1*）。 以 *testuser* 身份和密码 *AweS0me@PW* 登录。
   2. 在 jumpbox 上，在 *contoso.com* 域（本地域）中的第一个 VM 上打开一个 RDP 会话。 此 VM 具有 IP 地址 192.168.0.4。 用户名为 *contoso\testuser*，密码为 *AweS0me@PW*。
   3. 下载 [incoming-trust.ps1][incoming-trust] 脚本并运行它来创建来自 *treyresearch.com* 域的传入信任。

6. 如果你使用自己的本地基础结构：
   
   1. 则下载 [incoming-trust.ps1][incoming-trust] 脚本。
   2. 编辑脚本并将 `$TrustedDomainName` 变量的值替换为你自己的域的值。
   3. 运行该脚本。

7. 从 jumpbox 中，连接到 *treyresearch.com* 域（云中的域）中的第一个 VM。 此 VM 具有 IP 地址 10.0.4.4。 用户名为 *treyresearch\testuser*，密码为 *AweS0me@PW*。

8. 下载 [outgoing-trust.ps1][outgoing-trust] 脚本并运行它来创建来自 *treyresearch.com* 域的传入信任。 如果你在使用自己的本地计算机，请先编辑该脚本。 将 `$TrustedDomainName` 变量设置为你的本地域的名称，在 `$TrustedDomainDnsIpAddresses` 变量中指定此域的 Active Directory DS 服务器的 IP 地址。

9. 等待几分钟时间，直到前面的步骤完成，然后连接到本地 VM 并执行[验证信任][verify-a-trust]一文中列出的步骤来确定是否正确配置了 *contoso.com* 与 *treyresearch.com* 域之间的信任关系。

## <a name="next-steps"></a>后续步骤

* 了解[将本地 AD DS 域扩展到 Azure][adds-extend-domain] 的最佳做法
* 了解在 Azure 中[创建 AD FS 基础结构][adfs]的最佳做法。

<!-- links -->
[adds-extend-domain]: adds-extend-domain.md
[adfs]: adfs.md

[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[running-VMs-for-an-N-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[ad-azure-guidelines]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[azure-expressroute]: https://azure.microsoft.com/documentation/articles/expressroute-introduction/
[azure-vpn-gateway]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-vpngateways/
[considerations]: ./considerations.md
[creating-external-trusts]: https://technet.microsoft.com/library/cc816837(v=ws.10).aspx
[creating-forest-trusts]: https://technet.microsoft.com/library/cc816810(v=ws.10).aspx
[github]: https://github.com/mspnp/reference-architectures/tree/master/identity/adds-forest
[incoming-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/incoming-trust.ps1
[microsoft_systems_center]: https://www.microsoft.com/server-cloud/products/system-center-2016/
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[solution-script]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/Deploy-ReferenceArchitecture.ps1
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[outgoing-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/outgoing-trust.ps1
[verify-a-trust]: https://technet.microsoft.com/library/cc753821.aspx
[visio-download]: https://archcenter.azureedge.net/cdn/identity-architectures.vsdx
[0]: ./images/adds-forest.png "使用独立的 Active Directory 域保护混合网络体系结构的安全"