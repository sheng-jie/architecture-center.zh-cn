---
title: "在 Azure 上运行 Jenkins 服务器"
description: "本参考体系结构演示如何在 Azure 上部署和运行使用单一登录 (SSO) 保护的可伸缩企业级 Jenkins 服务器。"
author: njray
ms.date: 01/21/18
ms.openlocfilehash: 724185e43ed743013f52ded04b779552dd8e48c1
ms.sourcegitcommit: 29fbcb1eec44802d2c01b6d3bcf7d7bd0bae65fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/27/2018
---
# <a name="run-a-jenkins-server-on-azure"></a>在 Azure 上运行 Jenkins 服务器

本参考体系结构演示如何在 Azure 上部署和运行使用单一登录 (SSO) 保护的可伸缩企业级 Jenkins 服务器。 本体系结构还使用 Azure Monitor 来监视 Jenkins 服务器的状态。 [**部署此解决方案**。](#deploy-the-solution)

![Azure 上运行的 Jenkins 服务器][0]

下载包含本体系结构关系图的 [Visio 文件](https://arch-center.azureedge.net/cdn/Jenkins-architecture.vsdx)。

本体系结构支持对 Azure 服务实现灾难恢复，但不包括涉及到多个主节点或高可用性 (HA) 功能，且可避免停机的其他高级横向扩展方案。 有关各种 Azure 组件的一般见解，包括有关在 Azure 上构建 CI/CD 管道的分步教程，请参阅 [Azure 上的 Jenkins][jenkins-on-azure]。

本文档重点介绍支持 Jenkins 所要执行的核心 Azure 操作，包括使用 Azure 存储来维护生成项目、实现 SSO 所需的安全项、可集成的其他服务，以及管道的可伸缩性。 本体系结构在设计上使用现有的源代码管理存储库。 例如，一种常见方案是基于 GitHub 提交内容启动 Jenkins 作业。

## <a name="architecture"></a>体系结构

该体系结构包括以下组件：

-   **资源组。** [资源组][rg]用于对 Azure 资产进行分组，以便可以根据生存期、所有者和其他条件对其进行管理。 使用资源组能够以组的形式部署和监视 Azure 资产，并按资源组跟踪计费成本。 还可以删除作为集的资源，这对于测试部署非常有用。

-   **Jenkins 服务器**。 部署一个用于运行 [Jenkins][azure-market] 的虚拟机。该虚拟机用作自动化服务器并充当 Jenkins 主节点。 本参考体系结构使用 [Azure 上的 Jenkins 解决方案模板][solution]，该模板安装在 Azure 上的 Linux (Ubuntu 16.04 LTS) 虚拟机中。 Azure Marketplace 中提供了其他 Jenkins 产品。

    > [!NOTE]
    > 已在 VM 上安装充当 Jenkins 的反向代理的 Nginx。 可将 Nginx 配置为对 Jenkins 服务器启用 SSL。
    > 
    > 

-   虚拟网络。 [虚拟网络][vnet]可将 Azure 资源相互连接，并提供逻辑隔离。 在本体系结构中，Jenkins 服务器在虚拟网络中运行。

-   **子网**。 Jenkins 服务器在[子网][subnet]中隔离，因此我们可以更轻松地管理和隔离网络流量，且不影响性能。

-   **NSG**。 使用[网络安全组][nsg] (NSG) 可以限制从 Internet 发往虚拟网络子网的网络流量。

-   **托管磁盘**。 [托管磁盘][managed-disk]是一种持久性虚拟硬盘 (VHD)，可用作应用程序存储，同时还可维护 Jenkins 服务器的状态并提供灾难恢复。 数据磁盘存储在 Azure 存储中。 为实现高性能，我们建议使用[高级存储][premium]。

-   **Azure Blob 存储**。 [Windows Azure 存储插件][configure-storage]使用 Azure Blob 存储来存储所创建的并与其他 Jenkins 生成组件共享的生成项目。

-   **Azure Active Directory (Azure AD)**。 [Azure AD][azure-ad] 支持用户身份验证，并允许设置 SSO。 Azure AD [服务主体][service-principal]使用[基于角色的访问控制][rbac] (RBAC) 为工作流中的每个角色授权定义策略和权限。 每个服务主体与某个 Jenkins 作业相关联。

-   **Azure Key Vault**。 为了在需要机密时管理用于预配 Azure 资源的机密和加密密钥，本体系结构使用了 [Key Vault][key-vault]。 有关存储与管道中应用程序关联的机密的其他帮助，请参阅适用于 Jenkins 的 [Azure 凭据][configure-credential]插件。

-   **Azure 监视服务**。 此服务[监视][monitor]托管 Jenkins 的 Azure 虚拟机。 此部署监视虚拟机状态和 CPU 利用率，并发送警报。

## <a name="recommendations"></a>建议

以下建议适用于大多数方案。 除非有优先于这些建议的特定要求，否则请遵循这些建议。

### <a name="azure-ad"></a>Azure AD

Azure 订阅的 [Azure AD][azure-ad] 租户用于对 Jenkins 用户启用 SSO，并设置可让 Jenkins 作业访问 Azure 资源的[服务主体][service-principal]。

SSO 身份验证和授权由 Jenkins 服务器上安装的 Azure AD 插件实现。 登录到 Jenkins 服务器时，可以借助 SSO 使用 Azure AD 中的组织凭据进行身份验证。 配置 Azure AD 插件时，可以指定用户对 Jenkin 服务器的授权访问级别。

若要提供有权访问 Azure 资源的 Jenkins 作业，Azure AD 管理员可以创建服务主体。 这些授权应用程序（在本例中为 Jenkins 作业）[在经过身份验证且获授权的情况下可以访问][ad-sp] Azure 资源。

[RBAC][rbac] 进一步通过用户或服务主体的角色来定义和控制他们（它们）对 Azure 资源的访问。 支持内置角色和自定义角色。 角色还有助于保护管道，并确保正确分配和授权用户或代理的责任。 此外，可以设置 RBAC 来限制对 Azure 资产的访问。 例如，可将用户限制为只能使用特定资源组中的资产。

### <a name="storage"></a>存储

使用从 Azure Marketplace 安装的 Jenkins [Windows Azure 存储插件][storage-plugin]能够存储可与其他生成和测试组件共享的生成项目。 必须先配置一个 Azure 存储帐户，然后 Jenkins 作业才能使用此插件。

### <a name="jenkins-azure-plugins"></a>Jenkins Azure 插件

Azure 上的 Jenkins 解决方案模板会安装多个 Azure 插件。 Azure DevOps 团队会生成并维护该解决方案模板和以下插件，这些插件使用 Azure Marketplace 中的其他 Jenkins 产品，以及本地安装的 Jenkins 主节点：

-   [Azure AD 插件][configure-azure-ad]可让 Jenkins 服务器基于 Azure AD 支持用户的 SSO。

-   [Azure VM 代理][configure-agent]插件使用 Azure 资源管理器 (ARM) 模板在 Azure 虚拟机中创建 Jenkins 代理。

-   [Azure 凭据][configure-credential]插件可用于在 Jenkins 中存储 Azure 服务主体凭据。

-   [Windows Azure 存储插件][configure-storage]可将生成项目上传到 [Azure Blob 存储][blob]并从中下载生成依赖项。

我们还建议查看使用 Azure 资源的所有可用 Azure 插件列表，该列表不断扩充。 若要查看最新列表，请访问 [Jenkins 插件索引][index]并搜索 Azure。 例如，可将以下插件用于部署：

-   借助 [Azure 容器代理][container-agents]可在 Jenkins 中以代理的形式运行容器。

-   [Kubernetes 持续部署](https://aka.ms/azjenkinsk8s)可将资源配置部署到 Kubernetes 群集。

-   [Azure 容器服务][acs]可以使用 Kubernetes、包含 Marathon 的 DC/OS 或 Docker Swarm 将配置部署到 Azure 容器服务。

-   [Azure Functions][functions] 可将项目部署到 Azure 函数。

-   [Azure 应用服务][app-service]可部署到 Azure 应用服务。

## <a name="scalability-considerations"></a>可伸缩性注意事项

可以缩放 Jenkins 以支持极大型工作负荷。 对于弹性生成，请不要在 Jenkins 主服务器上运行生成。 应将生成任务卸载到 Jenkins 代理，以便按需弹性缩放。 缩放代理时，请考虑两个选项：

- 使用 [Azure VM 代理][vm-agent]插件创建可在 Azure VM 中运行的 Jenkins 代理。 此插件可让代理实现弹性扩展，并可使用不同类型的虚拟机。 可以从 Azure Marketplace 中选择不同的基础映像，也可以使用自定义映像。 有关 Jenkins 代理如何缩放的详细信息，请参阅 Jenkins 文档中的 [Architecting for Scale][scale]（构建可缩放的解决方案）。

- 使用 [Azure 容器代理][container-agents]插件在[包含 Kubernetes 的 Azure 容器服务](/azure/container-service/kubernetes/)或 [Azure 容器实例](/azure/container-instances/)中以代理形式运行容器。

虚拟机的缩放成本通常比容器更高。 但是，若要使用容器进行缩放，则必须配合容器运行生成过程。

此外，可以使用 Azure 存储来共享生成项目，供其他生成代理在管道的下一阶段中使用。

### <a name="scaling-the-jenkins-server"></a>缩放 Jenkins 服务器 

可以通过更改 VM 大小来扩展或缩减 Jenkins 服务器 VM。 [Azure 上的 Jenkins 解决方案模板][azure-market]默认指定了 DS2 v2 大小（两个 CPU，7 GB RAM）。 此大小可以处理中小型团队工作负荷。 构建服务器时，可以通过选择不同的选项来更改 VM 大小。 

哪个服务器大小合适取决于预期工作负荷的大小。 Jenkins 社区维护了一套[选择指南][selection-guide]，以帮助确定最符合要求的配置。 Azure 提供许多 [Linux VM 大小][sizes-linux]，可满足任何要求。 有关缩放 Jenkins 主节点的详细信息，请参阅 Jenkins 社区编写的[最佳做法][best-practices]。


## <a name="availability-considerations"></a>可用性注意事项

评估工作流的可用性要求，以及在 Jenkin 服务器发生故障时如何恢复 Jenkin 的状态。 若要评估可用性要求，请考虑两个常见指标：

-   恢复时间目标 (RTO) 指定允许 Jenkins 发生故障多长时间。

-   恢复点目标 (RPO) 表示服务中断影响到 Jenkins 时，允许丢失多少数据。

在实践中，RTO 和 RPO 涉及到冗余和备份。 可用性对于硬件恢复而言不是一个问题（这是 Azure 的部分责任），它旨在确保维持 Jenkins 服务器的状态。 本参考体系结构使用保证单个虚拟机运行时间到达 99.9% 的 [Azure 服务级别协议][sla] (SLA)。 如果此 SLA 无法满足你的运行时间要求，请确保做好灾难恢复的规划，或考虑使用[多主节点 Jenkins 服务器][multi-master]部署（本文档未介绍）。

考虑使用部署过程步骤 7 中的灾难恢复[脚本][disaster]，来创建包含托管磁盘的 Azure 存储帐户用于存储 Jenkins 服务器状态。 如果 Jenkins 发生故障，可将它还原到此单独存储帐户中存储的状态。

## <a name="security-considerations"></a>安全注意事项

使用以下方法有助于在基本 Jenkins 服务器上锁定安全状态，因为它的基本状态是不安全的。

-   设置某种方法来保护 Jenkin 服务器上的登录。 默认情况下，HTTP 是不安全的方法，本体系结构使用 HTTP 和公共 IP。 考虑在 [Nginx 服务器上设置 HTTPS][nginx] 用于安全登录。

    > [!NOTE]
    > 将 SSL 添加到服务器时，请针对 Jenkins 子网创建一条 NSG 规则，以打开端口 443。 有关详细信息，请参阅[如何使用 Azure 门户向虚拟机开放端口][port443]。
    > 

-   确保 Jenkins 配置可防止跨站点请求伪造（“管理 Jenkins”\>“配置全局安全性”）。 这是 Microsoft Jenkins 服务器的默认设置。

-   使用[矩阵授权策略插件][matrix]配置对 Jenkins 仪表板的只读访问。

-   安装 [Azure 凭据][configure-credential]插件，使用 Key Vault 来处理 Azure 资产、管道中的代理以及第三方组件的机密。

-   使用 RBAC 将服务主体的访问权限限制为运行作业的最低必需权限。 这样可以限制恶意作业造成的损害的范围。

Jenkins 作业通常需要使用机密来访问需要授权的 Azure 服务，例如 Azure 容器服务。 结合 [Azure 凭据插件][configure-credential]使用 [Key Vault][key-vault] 来安全管理这些机密。 使用 Key Vault 存储服务主体凭据、密码、令牌和其他机密。

若要在一个中心视图中查看 Azure 资源的安全状态，请使用 [Azure 安全中心][security-center]。 安全中心监视潜在的安全问题，并全面描述了部署的安全运行状况。 安全中心针对每个 Azure 订阅进行配置。 启用安全数据收集，如 [Azure 安全中心快速入门指南][quick-start]中所述。 启用数据收集后，安全中心会自动扫描该订阅下创建的所有虚拟机。

Jenkins 服务器具有自身的用户管理系统，Jenkins 社区提供了有关[在 Azure 上保护 Jenkins 实例][secure-jenkins]的最佳做法。 Azure 上的 Jenkins 解决方案模板实施这些最佳做法。

## <a name="manageability-considerations"></a>可管理性注意事项

使用资源组来组织已部署的 Azure 资源。 在单独的资源组中部署生产环境和开发/测试环境，以便可以监视每个环境的资源，并按资源组汇总计费成本。 还可以删除作为集的资源，这对于测试部署非常有用。

Azure 提供多种功能用于[监视和诊断][monitoring-diag]整个基础结构。 为了监视 CPU 使用率，本体系结构部署了 Azure Monitor。 例如，可以使用 Azure Monitor 来监视 CPU 使用率，并在 CPU 使用率超过 80% 时发送通知。 （CPU 使用率较高意味着可能需要扩展 Jenkins 服务器 VM。）此外，当 VM 发生故障或不可用时，可向指定的用户发送通知。

## <a name="communities"></a>社区

社区可以解答问题，并帮助设置成功的部署。 请注意以下几点：

-   [Jenkins 社区博客](https://jenkins.io/node/)
-   [Azure 论坛](https://azure.microsoft.com/support/forums/)
-   [Stack Overflow Jenkins](https://stackoverflow.com/tags/jenkins/info)

有关 Jenkins 社区编写的其他最佳做法，请访问 [Jenkins 最佳做法][jenkins-best]。

## <a name="deploy-the-solution"></a>部署解决方案

若要部署本体系结构，请执行以下步骤安装 [Azure 上的 Jenkins 解决方案模板][azure-market]，然后安装所需的脚本，用于设置后续步骤中所述的监视和灾难恢复。

### <a name="prerequisites"></a>先决条件

- 本参考体系结构需要 Azure 订阅。 
- 若要创建 Azure 服务主体，必须对与部署的 Jenkins 服务器相关联的 Azure AD 租户拥有管理权限。
- 本文档中的说明假设 Jenkins 管理员也是拥有最低“参与者”特权的 Azure 用户。

### <a name="step-1-deploy-the-jenkins-server"></a>步骤 1：部署 Jenkins 服务器

1.  在 Web 浏览器中打开 [Azure Jenkins 的 Marketplace 映像][azure-market]，然后从页面左侧选择“立即获取”。

2.  查看定价详细信息并选择“继续”，然后选择“创建”，在 Azure 门户中配置 Jenkins 服务器。

有关详细说明，请参阅[通过 Azure 门户在 Azure Linux VM 上创建 Jenkins 服务器][create-jenkins]。 对于本参考体系结构，使用管理员登录名便足以启动和运行服务器。 然后，可将服务器预配为使用其他各种服务。

### <a name="step-2-set-up-sso"></a>步骤 2：设置 SSO

该步骤由 Jenkins 管理员运行。该管理员还必须在订阅的 Azure AD 目录中有一个用户帐户，并且必须分配有“参与者”角色。

使用 Jenkin 服务器中 Jenkins 更新中心内的 [Azure AD 插件][configure-azure-ad]，然后遵照说明设置 SSO。

### <a name="step-3-provision-jenkins-server-with-azure-vm-agent-plugin"></a>步骤 3：使用 Azure VM 代理插件预配 Jenkins 服务器

Jenkins 管理员运行此步骤来设置已安装的 Azure VM 代理插件。

[遵循以下步骤配置插件][configure-agent]。 有关为插件设置服务主体的教程，请参阅[使用 Azure VM 代理根据需求缩放 Jenkins 部署][scale-agent]。

### <a name="step-4-provision-jenkins-server-with-azure-storage"></a>步骤 4：使用 Azure 存储预配 Jenkins 服务器

Jenkins 管理员运行此步骤来设置已安装的 Windows Azure 存储插件。

[遵循这些步骤配置插件][configure-storage]。

### <a name="step-5-provision-jenkins-server-with-azure-credential-plugin"></a>步骤 5：使用 Azure 凭据插件预配 Jenkins 服务器

Jenkins 管理员运行此步骤来设置已安装的 Azure 凭据插件。

[遵循这些步骤配置插件][configure-credential]。

### <a name="step-6-provision-jenkins-server-for-monitoring-by-the-azure-monitor-service"></a>步骤 6：预配 Jenkins 服务器以通过 Azure Monitor 服务进行监视

若要设置 Jenkins 服务器监视，请遵照[在 Azure Monitor 中针对 Azure 服务创建指标警报][create-metric]中的说明。

### <a name="step-7-provision-jenkins-server-with-managed-disks-for-disaster-recovery"></a>步骤 7：使用托管磁盘预配 Jenkins 服务器以实现灾难恢复

Microsoft Jenkins 产品组已创建灾难恢复脚本，这些脚本可以生成用于保存 Jenkins 状态的托管磁盘。 如果服务器发生故障，可将它还原到最新状态。

从 [GitHub][disaster] 下载并运行灾难恢复脚本。

[acs]: https://aka.ms/azjenkinsacs
[ad-sp]: /azure/active-directory/develop/active-directory-integrating-applications
[app-service]: https://plugins.jenkins.io/azure-app-service
[azure-ad]: /azure/active-directory/
[azure-market]: https://azuremarketplace.microsoft.com/marketplace/apps/azure-oss.jenkins?tab=Overview
[best-practices]: https://jenkins.io/doc/book/architecting-for-scale/
[blob]: /azure/storage/common/storage-java-jenkins-continuous-integration-solution
[configure-azure-ad]: https://plugins.jenkins.io/azure-ad
[configure-agent]: https://plugins.jenkins.io/azure-vm-agents
[configure-credential]: https://plugins.jenkins.io/azure-credentials
[configure-storage]: https://plugins.jenkins.io/windows-azure-storage
[container-agents]: https://aka.ms/azcontaineragent
[create-jenkins]: /azure/jenkins/install-jenkins-solution-template
[create-metric]: /azure/monitoring-and-diagnostics/insights-alerts-portal
[disaster]: https://github.com/Azure/jenkins/tree/master/disaster_recovery
[functions]: https://aka.ms/azjenkinsfunctions
[index]: https://plugins.jenkins.io
[jenkins-best]: https://wiki.jenkins.io/display/JENKINS/Jenkins+Best+Practices
[jenkins-on-azure]: /azure/jenkins/
[key-vault]: /azure/key-vault/
[managed-disk]: /azure/virtual-machines/linux/managed-disks-overview
[matrix]: https://plugins.jenkins.io/matrix-auth
[monitor]: /azure/monitoring-and-diagnostics/
[monitoring-diag]: /azure/architecture/best-practices/monitoring
[multi-master]: https://jenkins.io/doc/book/architecting-for-scale/
[nginx]: https://www.digitalocean.com/community/tutorials/how-to-create-an-ssl-certificate-on-nginx-for-ubuntu-14-04
[nsg]: /azure/virtual-network/virtual-networks-nsg
[quick-start]: /azure/security-center/security-center-get-started
[port443]: /azure/virtual-machines/windows/nsg-quickstart-portal
[premium]: /azure/virtual-machines/linux/premium-storage
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rg]: /azure/azure-resource-manager/resource-group-overview
[scale]: https://jenkins.io/doc/book/architecting-for-scale/
[scale-agent]: /azure/jenkins/jenkins-azure-vm-agents
[selection-guide]: https://jenkins.io/doc/book/hardware-recommendations/
[service-principal]: /azure/active-directory/develop/active-directory-application-objects
[secure-jenkins]: https://jenkins.io/blog/2017/04/20/secure-jenkins-on-azure/
[security-center]: /azure/security-center/security-center-intro
[sizes-linux]: /azure/virtual-machines/linux/sizes?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json
[solution]: https://azure.microsoft.com/blog/announcing-the-solution-template-for-jenkins-on-azure/
[sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/
[storage-plugin]: https://wiki.jenkins.io/display/JENKINS/Windows+Azure+Storage+Plugin
[subnet]: /azure/virtual-network/virtual-network-manage-subnet
[vm-agent]: https://wiki.jenkins.io/display/JENKINS/Azure+VM+Agents+plugin
[vnet]: /azure/virtual-network/virtual-networks-overview
[0]: ./images/jenkins-server.png 
