---
title: "保护数据解决方案"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 57897c31a8abdcd801874bf92d60360f7a80d1fa
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/14/2018
---
# <a name="securing-data-solutions"></a>保护数据解决方案

对于许多人来说，在云中启用数据的访问，尤其是将专门在本地数据存储中运行的数据过渡到云中，可能会在提高数据访问程度以及保护数据的新方法方面带来某种忧虑。

## <a name="challenges"></a>挑战

* 集中监视和分析大量日志中存储的安全事件。
* 跨应用程序和服务实施加密与授权管理。
* 确保集中式标识管理可以针对所有解决方案组件（不管在本地还是云中）进行。

## <a name="data-protection"></a>数据保护

保护信息的第一步是确定要保护的内容。 制定明确、简单且能顺利沟通的准则，来标识、保护和监视位于任何位置的最重要数据资产。 对于会给组织的使命或盈利造成不相称影响的资产建立最强的保护。 这些资产称为高价值资产 (HVA)。 执行严格的 HVA 生命周期与安全依赖项的分析，并建立适当的安全控制和条件。 同样，标识并分类敏感资产，并定义技术和过程来自动应用安全控制。

确定需要保护的数据后，考虑如何保护静态数据和传输中数据。

* **静态数据**：本地物理媒体（磁盘或光盘）或云中静态存在的数据。
* **传输中数据**：通过网络、服务总线（从本地到云，反之亦然），或者在输入/输出过程中在组件、位置或程序之间传输的数据。

若要详细了解如何保护静态数据或传输中数据，请参阅 [Azure 数据安全与加密最佳做法](/azure/security/azure-security-data-encryption-best-practices)。

## <a name="access-control"></a>Access Control

将标识管理和访问控制相结合，可在一个中心位置保护云中的数据。 随着各种云服务的出现以及[混合云](../scenarios/hybrid-on-premises-and-cloud.md)的不断流行，在建立标识和访问控制时，可以遵循几项关键做法：

* 集中化标识管理
* 启用单一登录 (SSO)。
* 部署密码管理。
* 对用户实施多重身份验证 (MFA)。
* 使用基于角色的访问控制 (RBAC)。
* 应该配置条件访问策略，以使用用户位置、设备类型、修补级别等相关的附加属性来增强用户标识的典型概念。
* 使用资源管理器控制创建资源的位置。
* 主动监视可疑活动

有关详细信息，请参阅 [Azure 标识管理和访问控制安全最佳做法](/azure/security/azure-security-identity-management-best-practices)。

## <a name="auditing"></a>审核

除了前面所述的标识和访问监视以外，在云中使用的服务和应用程序应会生成可监视的安全相关事件。 监视这些事件的主要难题在于如何处理日志的数量，以避免潜在问题或排查发生的问题。 基于云的应用程序往往包含许多移动部件，其中的大多数部件生成某种级别的日志和遥测数据。 使用集中式监视和分析可以帮助管理和利用大量信息。

有关详细信息，请参阅 [Azure 日志记录和审核](/azure/security/azure-log-audit)。



## <a name="securing-data-solutions-in-azure"></a>在 Azure 中保护数据解决方案

### <a name="encryption"></a>加密

**虚拟机**。 使用 [Azure 磁盘加密](/azure/security/azure-security-disk-encryption)可以加密 Windows 或 Linux VM 上附加的磁盘。 此解决方案与 [Azure Key Vault](/azure/key-vault/) 集成，可以控制和管理磁盘加密密钥与机密。 

**Azure 存储**。 使用 [Azure 存储服务加密](/azure/storage/common/storage-service-encryption)可以自动加密 Azure 存储中的静态数据。 加密、解密和密钥管理对于用户而言是完全透明的。 还可以结合使用客户端加密和 Azure Key Vault 来保护传输中数据。 有关详细信息，请参阅 [Microsoft Azure 存储的客户端加密和 Azure Key Vault](/azure/storage/common/storage-client-side-encryption)。

**SQL 数据库**和 **Azure SQL 数据仓库**。 使用[透明数据加密](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql) (TDE) 可对数据库、关联的备份和事务日志文件执行实时加密和解密，而无需更改应用程序。 SQL 数据库还可以使用 [Always Encrypted](/azure/sql-database/sql-database-always-encrypted-azure-key-vault) 来帮助保护服务器上的敏感静态数据、在客户端和服务器之间移动的敏感数据，以及正在使用的数据。 可以使用 Azure Key Vault 来存储 Always Encrypted 加密密钥。 

### <a name="rights-management"></a>Rights Management

[Azure Rights Management](/information-protection/understand-explore/what-is-azure-rms) 是基于云的服务，它使用加密、标识和授权策略来保护文件和电子邮件。 它可以跨手机、平板电脑和个人电脑等多个设备工作。 可以在组织内部和外部保护信息，因为数据始终附带了保护，即使数据脱离了组织的边界。

### <a name="access-control"></a>访问控制

使用[基于角色的访问控制](/azure/active-directory/role-based-access-control-what-is) (RBAC) 可以根据用户角色限制对 Azure 资源的访问。 如果在本地使用 Active Directory，则可以[与 Azure AD 同步](/azure/active-directory/active-directory-hybrid-identity-design-considerations-directory-sync-requirements)，以便根据用户的本地标识为他们提供云标识。

使用 [Azure Active Directory 中的条件访问](/azure/active-directory/active-directory-conditional-access-azure-portal)可以根据特定的条件针对环境中的应用程序实施访问控制。 例如，策略语句可以采用以下形式：“如果分包商尝试从不受信任的网络访问我们的云应用，则阻止访问。” 

[Azure AD Privileged Identity Management](/azure/active-directory/active-directory-privileged-identity-management-configure) 可以帮助管理、控制和监视用户，以及他们使用管理特权执行的各种任务。 这是用于限制组织中的用户可在 Azure AD、Azure、Office 365 或 SaaS 应用中执行的特权操作，以及监视其活动的重要步骤。

### <a name="network"></a>网络

若要保护传输中的数据，在不同的位置之间交换数据时，请始终使用 SSL/TLS。 有时，需要使用虚拟专用网络 (VPN) 或 [ExpressRoute](/azure/expressroute/) 隔离本地与云基础结构之间的整个通信通道。 有关详细信息，请参阅[将本地数据解决方案扩展到云](../scenarios/hybrid-on-premises-and-cloud.md)。

使用[网络安全组](/azure/virtual-network/virtual-networks-nsg) (NSG) 可以减少潜在攻击途径的数量。 网络安全组包含一个安全规则列表，这些规则可根据源或目标 IP 地址、端口和协议允许或拒绝入站或出站网络流量。 

使用[虚拟网络服务终结点](/azure/virtual-network/virtual-network-service-endpoints-overview)可以保护 Azure SQL 或 Azure 存储资源，以便只有来自虚拟网络的流量能够访问这些资源。

Azure 虚拟网络 (VNet) 中的 VM 可以使用[虚拟网络对等互连](/azure/virtual-network/virtual-network-peering-overview)来与其他 VNet 安全通信。 对等虚拟网络之间的网络流量是专用的。 虚拟网络之间的流量仅限于 Microsoft 主干网络。

有关详细信息，请参阅 [Azure 网络安全](/azure/security/azure-network-security)

### <a name="monitoring"></a>监视

[Azure 安全中心](/azure/security-center/security-center-intro)自动收集、分析以及集成 Azure 资源、网络和连接的合作伙伴解决方案（如防火墙解决方案）的日志数据，检测真正的威胁并减少误报。 

[Log Analytics](/azure/log-analytics/log-analytics-overview) 提供一个集中位置用于访问日志，以及帮助分析该数据和创建自定义警报。

[Azure SQL 数据库威胁检测](/azure/sql-database/sql-database-threat-detection)可检测异常活动，这些活动表示异常和可能有害的数据库访问或使用尝试。 发生可疑数据库活动时，安全监管员或其他指定的管理员可以收到即时通知。 每个通知提供可疑活动的详细信息，以及如何进一步调查和缓解威胁的建议。


