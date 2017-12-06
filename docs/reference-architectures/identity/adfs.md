---
title: "在 Azure 中实现 Active Directory 联合身份验证服务 (AD FS)"
description: "如何在 Azure 中实现采用 Active Directory 联合身份验证服务授权的安全混合网络体系结构。\n指南,vpn 网关,ExpressRoute,负载均衡器,虚拟网络,active-directory"
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Identity management
pnp.series.prev: adds-forest
cardTitle: Extend AD FS to Azure
ms.openlocfilehash: b24f4e72b13331437d92f20a228e3ba8121db90a
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="extend-active-directory-federation-services-ad-fs-to-azure"></a>将 Active Directory 联合身份验证服务 (AD FS) 扩展到 Azure

此参考体系结构实现一个安全混合网络，该网络将本地网络扩展到 Azure，并使用 [Active Directory 联合身份验证服务 (AD FS)][active-directory-federation-services] 为 Azure 中运行的组件执行联合身份验证和授权。 [**部署此解决方案**。](#deploy-the-solution)

[![0]][0]

*下载此体系结构的 [Visio 文件][visio-download]。*

AD FS 可以在本地进行承载，但是如果应用程序是其中某些部分在 Azure 中实现的混合体，则在云中复制 AD FS 可能会更加高效。 

该图显示以下方案：

* 来自合作伙伴组织的应用程序代码访问在 Azure VNet 中承载的 Web 应用程序。
* 凭据存储在 Active Directory 域服务 (DS) 中的外部已注册用户访问在 Azure VNet 中承载的 Web 应用程序。
* 使用授权设备连接到 VNet 的用户执行在 Azure VNet 中承载的 Web 应用程序。

此体系结构的典型用途包括：

* 在本地运行一部分工作负荷，在 Azure 中运行一部分工作负荷的混合应用程序。
* 使用联合授权向合作伙伴组织公开 Web 应用程序的解决方案。
* 支持从组织防火墙外部运行的 Web 浏览器进行访问的系统。
* 使用户可以通过从授权外部设备（如远程计算机、笔记本电脑和其他移动设备）进行连接来访问 Web 应用程序的系统。 

此参考体系结构侧重于被动联合，其中由联合服务器决定如何以及何时对用户进行身份验证。 用户在启动应用程序时提供登录信息。 此机制最常由 Web 浏览器使用，涉及将浏览器重定向到对用户进行身份验证的站点的协议。 AD FS 还支持主动联合，其中应用程序负责提供凭据，无需进一步的用户交互，但是该方案不在此体系结构的范围内。

有关其他注意事项，请参阅[选择用于将本地 Active Directory 与 Azure 相集成的解决方案][considerations]。 

## <a name="architecture"></a>体系结构

此体系结构扩展在[将 AD DS 扩展到 Azure][extending-ad-to-azure] 中介绍的实现。 它包含以下组件。

* **AD DS 子网**。 AD DS 服务器包含在其自己的子网中，具有充当防火墙的网络安全组 (NSG) 规则。

* **AD DS 服务器**。 作为 VM 在 Azure 中运行的域控制器。 这些服务器提供域中的本地标识的身份验证。

* **AD FS 子网**。 AD FS 服务器位于其自己的子网中，具有充当防火墙的 NSG 规则。

* **AD FS 服务器**。 AD FS 服务器提供联合授权和身份验证。 在此体系结构中，它们执行以下任务：
  
  * 代表合作伙伴用户接收包含由合作伙伴联合服务器发出的声明的安全令牌。 AD FS 先验证令牌是否有效，然后将声明传递给在 Azure 中运行的 Web 应用程序以对请求进行授权。 
  
    在 Azure 中运行的 Web 应用程序是信赖方。 合作伙伴联合服务器必须发出由 Web 应用程序理解的声明。 合作伙伴联合服务器称为帐户伙伴，因为它们代表合作伙伴组织中经过身份验证的帐户提交访问请求。 AD FS 服务器称为资源伙伴，因为它们提供对资源（Web 应用程序）的访问。

  * 使用 AD DS 和 [Active Directory 设备注册服务][ADDRS]，对运行需要访问 Web 应用程序的 Web 浏览器或设备的外部用户发出的传入请求进行身份验证和授权。
    
  AD FS 服务器配置为通过 Azure 负载均衡器访问的场。 此实现可提高可用性和可伸缩性。 AD FS 服务器不直接向 Internet 公开。 所有 Internet 流量都通过 AD FS Web 应用程序代理服务器和 DMZ（也称为外围网络）进行筛选。

  有关 AD FS 工作原理的详细信息，请参阅 [Active Directory 联合身份验证服务概述][active-directory-federation-services-overview]。 此外，[Azure 中的 AD FS 部署][adfs-intro]一文包含实现的详细分步介绍。

* **AD FS 代理子网**。 AD FS 代理服务器可以包含在其自己的子网中，并具有提供保护的 NSG 规则。 此子网中的服务器通过在 Azure 虚拟网络与 Internet 之间提供防火墙的一组网络虚拟设备向 Internet 公开。

* **AD FS Web 应用程序代理 (WAP) 服务器**。 这些 VM 充当用于来自合作伙伴组织和外部设备的传入请求的 AD FS 服务器。 WAP 服务器充当筛选器，使得无法从 Internet 直接访问 AD FS 服务器。 与 AD FS 服务器一样，在具有负载均衡的场中部署 WAP 服务器可提供比部署独立服务器集合更高的可用性和可伸缩性。
  
  > [!NOTE]
  > 有关安装 WAP 服务器的详细信息，请参阅[安装和配置 Web 应用程序代理服务器][install_and_configure_the_web_application_proxy_server]
  > 
  > 

* **合作伙伴组织**。 运行的 Web 应用程序请求访问在 Azure 中运行的 Web 应用程序的合作伙伴组织。 合作伙伴组织中的联合服务器在本地对请求进行身份验证，并将包含声明的安全令牌提交到在 Azure 中运行的 AD FS。 Azure 中的 AD FS 会验证安全令牌，如果有效，则可以将声明传递给在 Azure 中运行的 Web 应用程序以对它们进行授权。
  
  > [!NOTE]
  > 还可以使用 Azure 网关配置 VPN 隧道，以便为受信任合作伙伴提供对 AD FS 的直接访问。 从这些合作伙伴接收的请求不会经过 WAP 服务器。
  > 
  > 

有关不与 AD FS 相关的体系结构部分的详细信息，请参阅以下内容：
- [在 Azure 中实现安全混合网络体系结构][implementing-a-secure-hybrid-network-architecture]
- [在 Azure 中实现提供 Internet 访问方式的安全混合网络体系结构][implementing-a-secure-hybrid-network-architecture-with-internet-access]
- [在 Azure 中实现采用 Active Directory 标识的安全混合网络体系结构][extending-ad-to-azure]。


## <a name="recommendations"></a>建议

以下建议适用于大多数方案。 除非有优先于这些建议的特定要求，否则请遵循这些建议。 

### <a name="vm-recommendations"></a>VM 建议

创建具有足够资源来处理预期流量的 VM。 使用在本地承载 AD FS 的现有计算机的大小作为起点。 监视资源利用率。 可以调整 VM 的大小，在它们太大时进行缩减。

遵循[在 Azure 上运行 Windows VM][vm-recommendations] 中列出的建议。

### <a name="networking-recommendations"></a>网络建议

使用静态专用 IP 地址为承载 AD FS 和 WAP 服务器的每个 VM 配置网络接口。

请勿向 AD FS VM 提供公共 IP 地址。 有关详细信息，请参阅“安全注意事项”部分。

为每个 AD FS 和 WAP VM 的网络接口设置首选和辅助域名服务 (DNS) 服务器的 IP 地址，以引用 Active Directory DS VM。 Active Directory DS VM 应运行 DNS。 使每个 VM 可以加入域需要此步骤。

### <a name="ad-fs-availability"></a>AD FS 可用性 

创建具有至少两台服务器的 AD FS 场以提高服务的可用性。 为场中的每个 AD FS VM 使用不同的存储帐户。 此方法可帮助确保单个存储帐户中的故障不会使整个场不可访问。

> [!IMPORTANT]
> 建议使用[托管磁盘](/azure/storage/storage-managed-disks-overview)。 托管磁盘不需要存储帐户。 只需指定磁盘的大小和类型，它就会以高度可用的方式部署。 我们的[参考体系结构](/azure/architecture/reference-architectures/)当前未部署托管磁盘，但是[模板构建基块](https://github.com/mspnp/template-building-blocks/wiki)会在版本 2 中更新为部署托管磁盘。

为 AD FS 和 WAP VM 创建单独的 Azure 可用性集。 确保每个集中至少有两个 VM。 每个可用性集必须具有至少两个更新域和两个容错域。

按如下所示为 AD FS VM 和 WAP VM 配置负载均衡器：

* 使用 Azure 负载均衡器提供对 WAP VM 的外部访问，并使用内部负载均衡器在场中的 AD FS 服务器之间分布负载。
* 仅将在端口 443 (HTTPS) 上出现的流量传递给 AD FS/WAP 服务器。
* 为负载均衡器提供静态 IP 地址。
* 使用 TCP 协议而不是 HTTPS 创建运行状况探测。 可以对端口 443 执行 ping 操作，以验证 AD FS 服务器是否在正常运行。
  
  > [!NOTE]
  > AD FS 服务器使用服务器名称指示 (SNI) 协议，因此尝试从负载均衡器使用 HTTPS 终结点进行探测会失败。
  > 
  > 
* 将 DNS A 记录添加到 AD FS 负载均衡器的域。 指定负载均衡器的 IP 地址，并为它提供域中的名称（例如 adfs.contoso.com）。 这是客户端和 WAP 服务器用于访问 AD FS 服务器场的名称。

### <a name="ad-fs-security"></a>AD FS 安全性 

阻止向 Internet 直接公开 AD FS 服务器。 AD FS 服务器是具有完整授权，可授予安全令牌的已加入域的计算机。 如果服务器受到损害，则恶意用户可以向所有 Web 应用程序并向 AD FS 保护的所有联合服务器颁发完全访问令牌。 如果系统必须处理不是从受信任合作伙伴站点连接的外部用户发出的请求，请使用 WAP 服务器处理这些请求。 有关详细信息，请参阅[放置联合服务器代理的位置][where-to-place-an-fs-proxy]。

将 AD FS 服务器和 WAP 服务器放置在具有自己的防火墙的单独子网中。 可以使用 NSG 规则定义防火墙规则。 如果需要更全面的保护，则可以使用一对子网和网络虚拟设备 (NVA) 在服务器周围实现附加安全外围，如文档[在 Azure 中实现提供 Internet 访问方式的安全混合网络体系结构][implementing-a-secure-hybrid-network-architecture-with-internet-access]中所述。 所有防火墙都应允许端口 443 (HTTPS) 上的流量。

限制对 AD FS 和 WAP 服务器的直接登录访问。 只有 DevOps 员工才应能够连接。

请勿将 WAP 服务器加入域。

### <a name="ad-fs-installation"></a>AD FS 安装 

文章[部署联合服务器场][Deploying_a_federation_server_farm]提供了有关安装和配置 AD FS 的详细说明。 在场中配置第一台 AD FS 服务器之前执行以下任务：

1. 获取用于执行服务器身份验证的公开受信任证书。 使用者名称必须包含客户端用于访问联合身份验证服务的名称。 这可以是为负载均衡器注册的 DNS 名称，例如 adfs.contoso.com（出于安全原因，请避免使用通配符名称，如 *.contoso.com）。 在所有 AD FS 服务器 VM 上使用相同证书。 可以从受信任证书颁发机构购买证书，但如果组织使用 Active Directory 证书服务，则可以创建自己的证书。 
   
    使用者可选名称由设备注册服务 (DRS) 用于启用从外部设备进行的访问。 这应采用 enterpriseregistration.contoso.com 的形式。
   
    有关详细信息，请参阅[为 AD FS 获取并配置安全套接字层 (SSL) 证书][adfs_certificates]。

2. 在域控制器上，为密钥分发服务生成新的根密钥。 将有效时间设置为当前时间减 10 小时（此配置会减少在域中分发和同步密钥时可能发生的延迟）。 支持创建用于运行 AD FS 服务的组服务帐户需要此步骤。 以下 PowerShell 命令演示有关如何执行此操作的示例：
   
    ```powershell
    Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
    ```

3. 将每个 AD FS 服务器 VM 添加到域。

> [!NOTE]
> 若要安装 AD FS，为域运行主域控制器 (PDC) 仿真器灵活单主机操作 (FSMO) 角色的域控制器必须正在运行并且可从 AD FS VM 进行访问。 <<RBC：是否可通过某种方法减少此内容的重复性？>>
> 
> 

### <a name="ad-fs-trust"></a>AD FS 信任 

在 AD FS 安装与任何合作伙伴组织的联合服务器之间建立联合身份验证信任。 配置所需的任何声明筛选和映射。 

* 每个合作伙伴组织的 DevOps 员工必须添加信赖方信任，以便可通过 AD FS 服务器访问 Web 应用程序。
* 组织中的 DevOps 员工必须配置声明提供方信任，以便使 AD FS 服务器可以信任合作伙伴组织提供的声明。
* 组织中的 DevOps 员工还必须配置 AD FS 以将声明传递给组织的 Web 应用程序。
  
有关详细信息，请参阅[建立联合身份验证信任][establishing-federation-trust]。

通过 WAP 服务器使用预身份验证发布组织的 Web 应用程序并将它们提供给外部合作伙伴。 有关详细信息，请参阅[使用 AD FS 预身份验证发布应用程序][publish_applications_using_AD_FS_preauthentication]

AD FS 支持令牌转换和扩大。 Azure Active Directory 不提供此功能。 借助 AD FS，在设置信任关系时可以：

* 为授权规则配置声明转换。 例如，可以将组安全性从非 Microsoft 合作伙伴组织使用的表示形式映射到 Active Directory DS 可以在组织中授权的某种内容。
* 将声明从一种格式转换为另一种格式。 例如，如果应用程序仅支持 SAML 1.1 声明，则可以从 SAML 2.0 映射到 SAML 1.1。

### <a name="ad-fs-monitoring"></a>AD FS 监视 

[适用于 Active Directory 联合身份验证服务 2012 R2 的 Microsoft System Center 管理包][oms-adfs-pack]为联合服务器提供 AD FS 部署的主动和被动监视。 此管理包监视：

* AD FS 服务在其事件日志中记录的事件。
* AD FS 性能计数器收集的性能数据。 
* AD FS 系统和 Web 应用程序（信赖方）的总体运行状况，提供针对关键问题和警告的警报。 

## <a name="scalability-considerations"></a>可伸缩性注意事项

从文章[规划 AD FS 部署][plan-your-adfs-deployment]中汇总的以下注意事项为调整 AD FS 场规模提供了起点：

* 如果用户数少于 1000，请勿创建专用服务器，而是改为在云中的每台 Active Directory DS 服务器上安装 AD FS。 确保具有至少两台 Active Directory DS 服务器以保持可用性。 创建单台 WAP 服务器。
* 如果用户数介于 1000 与 15000 之间，请创建两台专用 AD FS 服务器和两台专用 WAP 服务器。
* 如果用户数介于 15000 与 60000 之间，请创建三到五台专用 AD FS 服务器和至少两台专用 WAP 服务器。

这些注意事项假设在 Azure 中使用双四核 VM（标准 D4_v2 或更好）大小。

如果使用 Windows 内部数据库存储 AD FS 配置数据，则场中仅限八台 AD FS 服务器。 如果预计在将来需要更多服务器，请使用 SQL Server。 有关详细信息，请参阅 [AD FS 配置数据库的角色][adfs-configuration-database]。

## <a name="availability-considerations"></a>可用性注意事项

可以使用 SQL Server 或 Windows 内部数据库保存 AD FS 配置信息。 Windows 内部数据库提供基本冗余。 更改仅仅直接写入 AD FS 群集中的一个 AD FS 数据库，而其他服务器使用请求复制使其数据库保持最新状态。 使用 SQL Server 可以提供完整数据库冗余和高可用性（使用故障转移群集或镜像）。

## <a name="manageability-considerations"></a>可管理性注意事项

DevOps 员工应准备好执行以下任务：

* 管理联合服务器，包括管理 AD FS 场、管理联合服务器上的信任策略以及管理联合身份验证服务使用的证书。
* 管理 WAP 服务器，包括管理 WAP 场和证书。
* 管理 Web 应用程序，包括配置信赖方、身份验证方法和声明映射。
* 备份 AD FS 组件。

## <a name="security-considerations"></a>安全注意事项

AD FS 使用 HTTPS 协议，因此请确保包含 Web 层 VM 的子网的 NSG 规则允许 HTTPS 请求。 这些请求可能源自本地网络、包含 Web 层、业务层、数据层、专用 DMZ、公共 DMZ 的子网以及包含 AD FS 服务器的子网。

请考虑使用一组网络虚拟设备，它们记录有关遍历虚拟网络边缘的流量的详细信息以用于审核。

## <a name="deploy-the-solution"></a>部署解决方案

[Github][github] 上提供了一个用于部署此参考体系结构的解决方案。 若要运行部署此解决方案的 Powershell 脚本，需要具有 [Azure CLI][azure-cli] 的最新版本。 若要部署此参考体系结构，请执行以下步骤：

1. 将解决方案文件夹从 [Github][github] 克隆到本地计算机。

2. 打开 Azure CLI 并导航到本地解决方案文件夹。

3. 运行以下命令：
   
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```
   
    将 `<subscription id>` 替换为你的 Azure 订阅 ID。
   
    对于 `<location>`，请指定一个 Azure 区域，例如 `eastus` 或 `westus`。
   
    `<mode>` 参数控制部署粒度，可以是下列值之一：
   
   * `Onpremise`：部署模拟的本地环境。 如果没有现有本地网络，或如果要测试此参考体系结构而不更改现有本地网络的配置，则可以使用此部署进行测试和试验。
   * `Infrastructure`：部署 VNet 基础结构和 jumpbox。
   * `CreateVpn`：部署 Azure 虚拟网络网关并将其连接到模拟的本地网络。
   * `AzureADDS`：部署充当 Active Directory DS 服务器的 VM，将 Active Directory 部署到这些 VM，并在 Azure 中创建域。
   * `AdfsVm`：在 Azure 中部署 AD FS VM 并将它们加入域。
   * `PublicDMZ`：在 Azure 中部署公共 DMZ。
   * `ProxyVm`：在 Azure 中部署 AD FS 代理 VM 并将它们加入域。
   * `Prepare`：部署上述所有部署。 **如果在构建全新部署并且没有现有本地基础结构，则这是推荐选项。** 
   * `Workload`：（可选）部署 Web、业务和数据层 VM 以及支持网络。 不包括在 `Prepare` 部署模式中。
   * `PrivateDMZ`：（可选）在 Azure 中，在上面部署的 `Workload` VM 前部署专用 DMZ。 不包括在 `Prepare` 部署模式中。

4. 等待部署完成。 如果使用 `Prepare` 选项，则部署需要几个小时才能完成，完成时会显示消息 `Preparation is completed. Please install certificate to all AD FS and proxy VMs.`

5. 重新启动 jumpbox（ra-adfs-security-rg 组中的 ra-adfs-mgmt-vm1）以允许其 DNS 设置生效。

6. [为 AD FS 获取 SSL 证书][adfs_certificates]并在 AD FS VM 上安装此证书。 请注意，可以通过 jumpbox 连接到它们。 IP 地址是 10.0.5.4 和 10.0.5.5。 默认用户名为 contoso\testuser，密码为 AweSome@PW。
   
   > [!NOTE]
   > Deploy-ReferenceArchitecture.ps1 脚本中的注释此时提供有关使用 `makecert` 命令创建自签名测试证书和颁发机构的详细说明。 但是，仅将这些步骤作为**测试**来执行，请勿在生产环境中使用 makecert 生成的证书。
   > 
   > 

7. 运行以下 PowerShell 命令以部署 AD FS 服务器场：
   
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Adfs
    ``` 

8. 在 jumpbox 上，浏览到 `https://adfs.contoso.com/adfs/ls/idpinitiatedsignon.htm` 以测试 AD FS 安装（可能会收到证书警告，对于此测试，可以忽略它）。 验证 Contoso Corporation 登录页是否出现。 使用密码 AweS0me@PW，以 contoso\testuser 身份登录。

9. 在 AD FS 代理 VM 上安装 SSL 证书。 IP 地址是 10.0.6.4 和 10.0.6.5。

10. 运行以下 PowerShell 命令以部署第一台 AD FS 代理服务器：
   
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Proxy1
    ```

11. 按照脚本显示的说明测试第一台代理服务器的安装。

12. 运行以下 PowerShell 命令以部署第二台代理服务器：
    
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Proxy2
    ```

13. 按照脚本显示的说明测试完整代理配置。

## <a name="next-steps"></a>后续步骤

* 了解 [Azure Active Directory][aad]。
* 了解 [Azure Active Directory B2C][aadb2c]。

<!-- links -->
[extending-ad-to-azure]: adds-extend-domain.md

[vm-recommendations]: ../virtual-machines-windows/single-vm.md
[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md
[hybrid-azure-on-prem-vpn]: ../hybrid-networking/vpn.md

[azure-cli]: /azure/azure-resource-manager/xplat-cli-azure-resource-manager
[DRS]: https://technet.microsoft.com/library/dn280945.aspx
[where-to-place-an-fs-proxy]: https://technet.microsoft.com/library/dd807048.aspx
[ADDRS]: https://technet.microsoft.com/library/dn486831.aspx
[plan-your-adfs-deployment]: https://msdn.microsoft.com/library/azure/dn151324.aspx
[ad_network_recommendations]: #network_configuration_recommendations_for_AD_DS_VMs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[create_service_account_for_adfs_farm]: https://technet.microsoft.com/library/dd807078.aspx
[adfs-configuration-database]: https://technet.microsoft.com/library/ee913581(v=ws.11).aspx
[active-directory-federation-services]: https://technet.microsoft.com/windowsserver/dd448613.aspx
[security-considerations]: #security-considerations
[recommendations]: #recommendations
[active-directory-federation-services-overview]: https://technet.microsoft.com/library/hh831502(v=ws.11).aspx
[establishing-federation-trust]: https://blogs.msdn.microsoft.com/alextch/2011/06/27/establishing-federation-trust/
[Deploying_a_federation_server_farm]:  https://azure.microsoft.com/documentation/articles/active-directory-aadconnect-azure-adfs/
[install_and_configure_the_web_application_proxy_server]: https://technet.microsoft.com/library/dn383662.aspx
[publish_applications_using_AD_FS_preauthentication]: https://technet.microsoft.com/library/dn383640.aspx
[managing-adfs-components]: https://technet.microsoft.com/library/cc759026.aspx
[oms-adfs-pack]: https://www.microsoft.com/download/details.aspx?id=41184
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[aad]: https://azure.microsoft.com/documentation/services/active-directory/
[aadb2c]: https://azure.microsoft.com/documentation/services/active-directory-b2c/
[adfs-intro]: /azure/active-directory/active-directory-aadconnect-azure-adfs
[github]: https://github.com/mspnp/reference-architectures/tree/master/identity/adfs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[considerations]: ./considerations.md
[visio-download]: https://archcenter.azureedge.net/cdn/identity-architectures.vsdx
[0]: ./images/adfs.png "使用 Active Directory 保护混合网络体系结构的安全"
