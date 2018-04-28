---
title: 将本地 AD 域与 Azure Active Directory 集成
description: 如何使用 Active Directory 实施安全的混合网络体系结构。
author: telmosampaio
pnp.series.title: Identity management
ms.date: 11/28/2016
pnp.series.next: adds-extend-domain
pnp.series.prev: ./index
cardTitle: Integrate on-premises AD with Azure AD
ms.openlocfilehash: 9475d669b2cb8888a7ceabed7e36317fe63681fd
ms.sourcegitcommit: d702b4d27e96e7a5a248dc4f2f0e25cf6e82c134
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2018
---
# <a name="integrate-on-premises-active-directory-domains-with-azure-active-directory"></a>将本地 Active Directory 域与 Azure Active Directory 集成

Azure Active Directory (Azure AD) 是一种基于云的多租户目录和标识服务。 此参考体系结构展示了将本地 Active Directory 域与 Azure AD 集成以提供基于云的标识身份验证的最佳做法。 [**部署此解决方案**。](#deploy-the-solution)

[![0]][0] 

下载此体系结构的 [Visio 文件][visio-download]。

> [!NOTE]
> 为简单起见，此图仅展示了与 Azure AD 直接相关的连接，而未展示在身份验证和联合身份验证过程中可能产生的协议相关流量。 例如，Web 应用程序可能重定向 Web 浏览器，以便通过 Azure AD 对请求进行身份验证。 经过身份验证后，可以将带有相应标识信息的请求传递回 Web 应用程序。
> 

此参考体系结构的典型用途包括：

* 部署在 Azure 中的 Web 应用程序，用于为属于组织的远程用户提供访问权限。
* 为最终用户实现自助服务功能，比如重置其密码，以及委派组管理。 请注意，这需要 Azure AD Premium 版本。
* 本地网络和应用程序的 Azure VNet 不使用 VPN 隧道或 ExpressRoute 线路进行连接的体系结构。

> [!NOTE]
> Azure AD 当前仅支持用户身份验证。 某些应用程序和服务（如 SQL Server）可能需要进行计算机身份验证，此解决方案不适合这种情况。
> 

有关其他注意事项，请参阅[选择用于将本地 Active Directory 与 Azure 相集成的解决方案][considerations]。 

## <a name="architecture"></a>体系结构

该体系结构具有以下组件。

* **Azure AD 租户**。 由组织创建的 [Azure AD][azure-active-directory] 实例。 它通过存储从本地 Active Directory 中复制的对象，充当云应用程序的目录服务，并提供标识服务。
* **Web 层子网**。 此子网包含运行 Web 应用程序的 VM。 Azure AD 可充当此应用程序的标识代理。
* **本地 AD DS 服务器**。 一种本地目录和标识服务。 AD DS 目录可与 Azure AD 同步，从而能够对本地用户进行身份验证。
* **Azure AD Connect 同步服务器**。 运行 [Azure AD Connect][azure-ad-connect] 同步服务的本地计算机。 此服务可将本地 Active Directory 中保存的信息同步到 Azure AD。 例如，如果在本地预配或取消设置组和用户，这些更改会传播到 Azure AD。 
  
  > [!NOTE]
  > 出于安全原因，Azure AD 以哈希形式存储用户的密码。 如果用户需要重置密码，则必须在本地执行，并且必须将新哈希发送到 Azure AD。 Azure AD Premium 版本包含一些可自动执行此任务的功能，让用户能够重置自己的密码。
  > 

* **N 层应用程序所用的 VM**。 该部署包括 N 层应用程序的基础结构。 有关这些资源的详细信息，请参阅[运行 N 层体系结构所用的 VM][implementing-a-multi-tier-architecture-on-Azure]。

## <a name="recommendations"></a>建议

以下建议适用于大多数方案。 除非有优先于这些建议的特定要求，否则请遵循这些建议。 

### <a name="azure-ad-connect-sync-service"></a>Azure AD Connect 同步服务

Azure AD Connect 同步服务可确保存储在云中的标识信息与保存在本地的标识信息保持一致。 可使用 Azure AD Connect 软件安装此服务。 

在实施 Azure AD Connect 同步之前，先确定组织的同步要求。 例如，要从哪些域同步哪些内容，以及频率如何。 有关详细信息，请参阅[确定目录同步要求][aad-sync-requirements]。

可以在本地托管的 VM 或计算机上运行 Azure AD Connect 同步服务。 与 Azure AD 初次同步后，Azure AD Connect 同步服务上的负载一般不会很高，具体取决于 Active Directory 目录中信息的易变性。 如果在 VM 上运行该服务，可以在需要时更轻松地缩放服务器。 如“监视注意事项”部分中所述，监视 VM 上的活动，以确定是否有必要进行缩放。

如果林中有多个本地域，则建议存储整个林的信息并将信息同步到单个 Azure AD 租户。 对出现在多个域中的标识的信息进行筛选，以便每个标识在 Azure AD 中仅出现一次，而不是重复出现。 重复项会导致数据同步时出现不一致。 有关详细信息，请参阅下面的“拓扑”部分。 

使用筛选，以便仅在 Azure AD 中存储必要的数据。 例如，组织可能不想在 Azure AD 中存储有关停用帐户的信息。 可以按组、按域、按组织单位 (OU) 或按属性进行筛选。 可以组合使用多个筛选器，以生成更复杂的规则。 例如，可以同步某个域中保存的、针对所选属性具有特定值的对象。 有关详细信息，请参阅 [Azure AD Connect 同步：配置筛选][aad-filtering]。

若要实现 AD Connect 同步服务的高可用性，请运行辅助临时服务器。 有关详细信息，请参阅“拓扑建议”部分。

### <a name="security-recommendations"></a>安全建议

**用户密码管理。** Azure AD Premium 版本支持密码回写，从而允许本地用户从 Azure 门户执行自助密码重置。 若要启用此功能，必须先查看组织的密码安全策略。 例如，可限制哪些用户可以更改其密码，并定制密码管理体验。 有关详细信息，请参阅[自定义密码管理以满足组织的需求][aad-password-management]。 

**保护可从外部访问的本地应用程序。** 可使用 Azure AD 应用程序代理，通过 Azure AD 为外部用户提供对本地 Web 应用程序的受控访问权限。 只有在你的 Azure 目录中具有有效凭据的用户才有权使用该应用程序。 有关详细信息，请参阅[在 Azure 门户中启用应用程序代理][aad-application-proxy]一文。

**主动监视 Azure AD 中的可疑活动迹象。**    请考虑使用 Azure AD Premium P2 版本，该版本包括 Azure AD Identity Protection。 Identity Protection 使用自适应机器学习算法和启发式学习法来检测异常行为以及可能表示标识已遭入侵的风险事件。 例如，它可以检测到潜在的异常活动，比如异常登录活动，来自未知来源或活动可疑的 IP 地址的登录，或者来自可能受感染的设备的登录。 Identity Protection 使用此数据生成报告和警报，使你能够调查这些风险事件并采取相应的措施。 有关详细信息，请参阅 [Azure Active Directory Identity Protection][aad-identity-protection]。
  
可以在 Azure 门户中使用 Azure AD 的报告功能，来监视系统中发生的与安全相关的活动。 有关使用这些报告的详细信息，请参阅 [Azure Active Directory 报告指南][aad-reporting-guide]。

### <a name="topology-recommendations"></a>拓扑建议

配置 Azure AD Connect，以实现最符合组织要求的拓扑。 Azure AD Connect 支持的拓扑如下所示：

* **单个林，单个 Azure AD 目录**。 在此拓扑中，Azure AD Connect 将单个本地林的一个或多个域中的对象和标识信息同步到单个 Azure AD 租户。 这是通过 Azure AD Connect 的快速安装实现的默认拓扑。
  
  > [!NOTE]
  > 除非在临时模式下运行服务器（如下所述），否则不要使用多台 Azure AD Connect 同步服务器将同一本地林中的不同域连接到同一 Azure AD 租户。
  > 
  > 

* **多个林，单个 Azure AD 目录**。 在此拓扑中，Azure AD Connect 将多个林中的对象和标识信息同步到单个 Azure AD 租户。 如果组织有多个本地林，则使用此拓扑。 你可以合并标识信息，以便每个唯一的用户在 Azure AD 目录中只出现一次，即使同一用户存在于多个林也是如此。 所有林使用同一台 Azure AD Connect 同步服务器。 Azure AD Connect 同步服务器不必属于任何域，但必须可从所有林访问该服务器。
  
  > [!NOTE]
  > 在此拓扑中，请勿使用不同的 Azure AD Connect 同步服务器将每个本地林连接到单个 Azure AD 租户。 如果用户同时存在于多个林，这会导致 Azure AD 中的标识信息重复。
  > 
  > 

* **多个林，不同的拓扑**。 此拓扑将不同林中的标识信息合并到单个 Azure AD 租户，并视所有林为不同的实体。 如果合并来自不同组织的林，并且每个用户的标识信息仅保存在一个林中，则此拓扑非常有用。
  
  > [!NOTE]
  > 如果同步每个林中的全局地址列表 (GAL)，则一个林中的用户可能会以联系人的身份出现在另一个林中。 如果组织已使用 Forefront Identity Manager 2010 或 Microsoft Identity Manager 2016 实现 GALSync，则会出现此问题。 在这种情况下，可以指定应通过 *Mail* 属性识别用户。 也可以使用 *ObjectSID* 和 *msExchMasterAccountSID* 属性匹配标识。 如果有一个或多个包含已禁用帐户的资源林，则此方法很有用。
  > 
  > 

* **临时服务器**。 在此配置中，将 Azure AD Connect 同步服务器的第二个实例与第一个实例并行运行。 此结构支持如下方案：
  
  * 高可用性。
  * 测试和部署 Azure AD Connect 同步服务器的新配置。
  * 引入新服务器，并取消旧配置。 
    
    在这些方案中，第二个实例在*临时模式*下运行。 服务器将导入的对象和同步数据记录到数据库中，但不将数据传递给 Azure AD。 如果禁用临时模式，服务器除了开始将数据写入到 Azure AD，还会在本地目录中的适当位置开始执行密码回写。 有关详细信息，请参阅 [Azure AD Connect 同步：操作任务和注意事项][aad-connect-sync-operational-tasks]。

* **多个 Azure AD 目录**。 建议为组织创建单个 Azure AD 目录，但在某些情况下，可能需要将信息分区到不同的 Azure AD 目录中。 在这种情况下，应确保本地林中的每个对象仅在一个 Azure AD 目录中出现，以免出现同步和密码回写问题。 若要实现此方案，请为每个 Azure AD 目录配置不同的 Azure AD Connect 同步服务器，并使用筛选，以便各 Azure AD Connect 同步服务器在一组互相排斥的对象上运行。 

有关这些拓扑的详细信息，请参阅 [Azure AD Connect 的拓扑][aad-topologies]。

### <a name="user-authentication"></a>用户身份验证

默认情况下，Azure AD Connect 同步服务器会在本地域和 Azure AD 之间配置密码哈希同步，并且 Azure AD 服务假定用户通过提供他们在本地使用的密码进行身份验证。 许多组织支持这种做法，但你应当考虑自己组织现有的策略和基础结构。 例如：

* 组织的安全策略可能禁止将密码哈希同步到云。 在这种情况下，组织应考虑[直通身份验证](/azure/active-directory/connect/active-directory-aadconnect-pass-through-authentication)。
* 你可能要求用户从公司网络访问已加入域的计算机中的云资源时，使用无缝单一登录 (SSO)。
* 贵组织可能已部署 Active Directory 联合身份验证服务 (AD FS) 或第三方联合身份验证提供程序。 你可以将 Azure AD 配置为使用此基础结构实施身份验证和 SSO，而不是使用保存在云中的密码信息。

有关详细信息，请参阅 [Azure AD Connect 用户登录选项][aad-user-sign-in]。

### <a name="azure-ad-application-proxy"></a>Azure AD 应用程序代理 

使用 Azure AD 可提供对本地应用程序的访问。

使用由 Azure AD 应用程序代理组件管理的应用程序代理连接器，公开本地 Web 应用程序。 应用程序代理连接器与 Azure AD 应用程序代理建立出站网络连接，远程用户的请求通过此连接从 Azure AD 路由回 Web 应用。 这无需在本地防火墙中打开入站端口，减少了贵组织暴露的受攻击面。

有关详细信息，请参阅[使用 Azure AD 应用程序代理发布应用程序][aad-application-proxy]。

### <a name="object-synchronization"></a>对象同步 

Azure AD Connect 的默认配置基于 [Azure AD Connect 同步：了解默认配置][aad-connect-sync-default-rules]一文中指定的规则，同步本地 Active Directory 目录中的对象。 将同步满足这些规则的对象，而忽略所有其他对象。 一些示例规则：

* 用户对象必须具有唯一的 *sourceAnchor* 属性，并且必须已填充 *accountEnabled* 属性。
* 用户对象必须具有 *sAMAccountName* 属性，并且不能以文本 *Azure AD_* 或 *MSOL_* 开头。

Azure AD Connect 可向 User、Contact、Group、ForeignSecurityPrincipal 和 Computer 对象应用多种规则。 如果需要修改默认规则集，请使用随 Azure AD Connect 一起安装的同步规则编辑器。 有关详细信息，请参阅 [Azure AD Connect 同步：了解默认配置][aad-connect-sync-default-rules]。

也可以定义自己的筛选器，以限制域或 OU 要同步的对象。 或者，还可以如 [Azure AD Connect 同步：配置筛选][aad-filtering]中所述，实施更复杂的自定义筛选。

### <a name="monitoring"></a>监视 

运行状况监视由安装在本地的以下代理执行：

* Azure AD Connect 安装了一个用于捕获同步操作相关信息的代理。 可使用 Azure 门户中的 Azure AD Connect Health 边栏选项卡监视其运行状况和性能。 有关详细信息，请参阅[使用 Azure AD Connect Health 进行同步][aad-health]。
* 若要从 Azure 监视 AD DS 域和目录的运行状况，请在本地域内的计算机上安装用于 AD DS 代理的 Azure AD Connect Health。 可使用 Azure 门户中的 Azure Active Directory Connect Health 边栏选项卡监视运行状况。 有关详细信息，请参阅[在 AD DS 中使用 Azure AD Connect Health][aad-health-adds] 
* 安装用于 AD FS 代理的 Azure AD Connect Health，以监视本地运行的服务的运行状况，并使用 Azure 门户中的 Azure Active Directory Connect Health 边栏选项卡监视 AD FS。 有关详细信息，请参阅[在 AD FS 中使用 Azure AD Connect Health][aad-health-adfs]

有关安装 AD Connect Health 代理及其要求的详细信息，请参阅 [Azure AD Connect Health 代理安装][aad-agent-installation]。

## <a name="scalability-considerations"></a>可伸缩性注意事项

Azure AD 服务支持基于副本（一个处理写入操作的主要副本，以及多个只读次要副本）的可伸缩性。 Azure AD 将对次要副本尝试进行的写入操作以透明方式重定向到主要副本，并提供最终一致性。 对主要副本进行的所有更改都会传播到次要副本。 此体系结构的可伸缩性良好，因为针对 Azure AD 的大多数操作都是读取操作，而非写入操作。 有关详细信息，请参阅 [Azure AD：揭秘异地冗余且高度可用的分布式云目录][aad-scalability]。

对于 Azure AD Connect 同步服务器，应确定可能从本地目录同步的对象数。 如果对象数少于 100,000 个，则可以使用 Azure AD Connect 随附的默认 SQL Server Express LocalDB 软件。 如果对象数超过此数，则应安装 SQL Server 的生产版本，并执行 Azure AD Connect 的自定义安装，以指定它应使用 SQL Server 的现有实例。

## <a name="availability-considerations"></a>可用性注意事项

Azure AD 服务是一种异地分布式服务，它在分布于世界各地的多个数据中心内运行，并提供自动故障转移功能。 如果某个数据中心变得不可用，Azure AD 可确保分散在不同区域的至少两个数据中心的目录数据可供实例访问。

> [!NOTE]
> Azure AD Basic 和 Premium 服务的服务级别协议 (SLA) 保证至少 99.9% 的可用性。 Azure AD 免费层没有 SLA。 有关详细信息，请参阅 [Azure Active Directory 的 SLA][sla-aad]。
> 
> 

请考虑在临时模式下预配 Azure AD Connect 同步服务器的第二个实例来提高可用性，如“拓扑建议”部分中所述。 

如果未使用 Azure AD Connect 随附的 SQL Server Express LocalDB 实例，请考虑使用 SQL 群集来实现高可用性。 Azure AD Connect 不支持镜像和 Always On 等解决方案。

有关实现 Azure AD Connect 同步服务器高可用性以及如何在发生故障后恢复的其他注意事项，请参阅 [Azure AD Connect 同步：操作任务和注意事项 - 灾难恢复][aad-sync-disaster-recovery]。

## <a name="manageability-considerations"></a>可管理性注意事项

可从两方面管理 Azure AD：

* 在云中管理 Azure AD。
* 维护 Azure AD Connect 同步服务器。

Azure AD 提供以下选项，用于在云中管理域和目录： 

* **Azure Active Directory PowerShell 模块**。 如果需要编写常见 Azure AD 管理任务（比如用户管理、域管理和配置单一登录）的脚本，则使用此[模块][aad-powershell]。
* **Azure 门户中的 Azure AD 管理边栏选项卡**。 此边栏选项卡提供目录的交互式管理视图，并且可用于控制和配置 Azure AD 的大多数方面。 

Azure AD Connect 通过安装以下工具来维护本地计算机中的 Azure AD Connect 同步服务：
  
* **Microsoft Azure Active Directory Connect 控制台**。 此工具可用于修改 Azure AD Sync 服务器的配置、自定义同步方式、启用或禁用临时模式以及切换用户登录模式。 请注意，可以使用本地基础结构启用 Active Directory FS 登录。
* **Synchronization Service Manager**。 此工具中的*操作*选项卡用于管理同步过程并检测该过程的任何环节是否失败。 可以使用此工具手动触发同步。 *连接器*选项卡用于控制同步引擎附加到的域的连接。
* **同步规则编辑器**。 此工具用于自定义在本地目录与 Azure AD 之间复制对象时对象的转换方式。 使用此工具，可指定同步的其他属性和对象，然后执行筛选器以确定应该同步或不应该同步的对象。 有关详细信息，请参阅 [Azure AD Connect 同步：了解默认配置][aad-connect-sync-default-rules]文档中的“同步规则编辑器”部分。

有关管理 Azure AD Connect 的详细信息和提示，请参阅 [Azure AD Connect 同步：有关更改默认配置的最佳实践][aad-sync-best-practices]。

## <a name="security-considerations"></a>安全注意事项

可使用条件访问控制拒绝来自意外来源的身份验证请求：

- 如果用户尝试从不受信任的位置（比如从 Internet，而不是从受信任的网络）进行连接，则触发 [Azure 多重身份验证 (MFA)][azure-multifactor-authentication]。

- 使用用户的设备平台类型（iOS、Android、Windows Mobile、Windows）来确定应用程序和功能的访问策略。

- 记录用户设备的启用/禁用状态，并将此信息合并到访问策略检查中。 例如，如果用户的电话丢失或被盗，应将它记录为已禁用，防止有人用它来获取访问权限。

- 基于组成员资格控制用户对资源的访问权限。 可使用 [Azure AD 动态成员资格规则][aad-dynamic-membership-rules]简化组管理。 有关其工作原理的简要概述，请参阅[组动态成员资格简介][aad-dynamic-memberships]。

- 将条件访问风险策略与 Azure AD Identity Protection 结合使用，基于异常登录活动或其他事件提供高级保护。

有关详细信息，请参阅 [Azure Active Directory 条件访问][aad-conditional-access]。

## <a name="deploy-the-solution"></a>部署解决方案

GitHub 上提供了可实施这些建议和注意事项的参考体系结构部署。 此参考体系结构会在 Azure 中部署一个可用来测试和试验的模拟本地网络。 可以按照以下说明，使用 Windows VM 或 Linux VM 部署该参考体系结构： 

1. 单击下面的按钮：<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fidentity%2Fazure-ad%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. 在 Azure 门户中打开该链接后，必须输入某些设置的值： 
   * 参数文件中已定义**资源组**名称，因此请选择“新建”，并在文本框中输入 `ra-aad-onpremise-rg`。
   * 从“位置”下拉框中选择区域。
   * 不要编辑“模板根 URI”或“参数根 URI”文本框。
   * 从“OS 类型”下拉框中选择“Windows”或“Linux”。
   * 查看条款和条件，并单击“我同意上述条款和条件”复选框。
   * 单击“购买”按钮。
3. 等待部署完成。
4. 参数文件包括硬编码的管理员用户名和密码，强烈建议立即在所有 VM 上更改它们。 在 Azure 门户中单击每个 VM，然后，在“支持 + 故障排除”边栏选项卡中单击“重置密码”。 从“模式”下拉框中选择“重置密码”，然后选择新的**用户名**和**密码**。 单击“更新”按钮保存新用户名和密码。

<!-- links -->

[implementing-a-multi-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[aad-agent-installation]: /azure/active-directory/active-directory-aadconnect-health-agent-install
[aad-application-proxy]: /azure/active-directory/active-directory-application-proxy-enable
[aad-conditional-access]: /azure/active-directory//active-directory-conditional-access
[aad-connect-sync-default-rules]: /azure/active-directory/active-directory-aadconnectsync-understanding-default-configuration
[aad-connect-sync-operational-tasks]: /azure/active-directory/active-directory-aadconnectsync-operations#staging-mode
[aad-dynamic-memberships]: https://youtu.be/Tdiz2JqCl9Q
[aad-dynamic-membership-rules]: /azure/active-directory/active-directory-accessmanagement-groups-with-advanced-rules
[aad-editions]: /azure/active-directory/active-directory-editions
[aad-filtering]: /azure/active-directory/active-directory-aadconnectsync-configure-filtering
[aad-health]: /azure/active-directory/active-directory-aadconnect-health-sync
[aad-health-adds]: /azure/active-directory/active-directory-aadconnect-health-adds
[aad-health-adfs]: /azure/active-directory/active-directory-aadconnect-health-adfs
[aad-identity-protection]: /azure/active-directory/active-directory-identityprotection
[aad-password-management]: /azure/active-directory/active-directory-passwords-customize
[aad-powershell]: https://msdn.microsoft.com/library/azure/mt757189.aspx
[aad-reporting-guide]: /azure/active-directory/active-directory-reporting-guide
[aad-scalability]: https://blogs.technet.microsoft.com/enterprisemobility/2014/09/02/azure-ad-under-the-hood-of-our-geo-redundant-highly-available-distributed-cloud-directory/
[aad-sync-best-practices]: /azure/active-directory/active-directory-aadconnectsync-best-practices-changing-default-configuration
[aad-sync-disaster-recovery]: /azure/active-directory/active-directory-aadconnectsync-operations#disaster-recovery
[aad-sync-requirements]: /azure/active-directory/active-directory-hybrid-identity-design-considerations-directory-sync-requirements
[aad-topologies]: /azure/active-directory/active-directory-aadconnect-topologies
[aad-user-sign-in]: /azure/active-directory/active-directory-aadconnect-user-signin
[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/active-directory-aadconnect
[azure-multifactor-authentication]: /azure/multi-factor-authentication/multi-factor-authentication
[considerations]: ./considerations.md
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[sla-aad]: https://azure.microsoft.com/support/legal/sla/active-directory/v1_0/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx


[0]: ./images/azure-ad.png "使用 Azure Active Directory 的云标识体系结构"
