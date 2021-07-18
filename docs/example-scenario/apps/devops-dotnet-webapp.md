---
title: 将 CI/CD 管道与 VSTS 配合使用
description: 举例说明如何生成 .NET 应用并将其发布到 Azure Web 应用
author: christianreddington
ms.date: 07/11/18
ms.openlocfilehash: ae4ac5fc02cc841fc39b3cbef46124fe9da75e9b
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/14/2018
ms.locfileid: "39061008"
---
# <a name="cicd-pipeline-with-vsts"></a>将 CI/CD 管道与 VSTS 配合使用

DevOps 集成了开发、质量保证和 IT 运营。 DevOps 要求通过统一的区域性和一系列严格的流程来交付软件。

本示例方案演示了开发团队如何使用 Visual Studio Team Services 将 .NET 双层 Web 应用程序部署到 Azure 应用服务。 此 Web 应用程序依赖于下游的 Azure 平台即服务 (PaaS) 服务。 本文档还指出了在使用 Azure 平台即服务 (PaaS) 设计此类方案时应考虑的一些注意事项。

采用现代的方法通过持续集成 (CI) 和持续部署 (CD) 进行应用程序开发，你可以通过可靠的生成、测试、部署和监视服务更快地为用户带来价值。 使用 Visual Studio Team Services 之类的平台和应用服务之类的 Azure 服务，组织就可以确保只关注方案的开发，不必管理启用这些方案所需的基础结构。

## <a name="related-use-cases"></a>相关的用例

以下用例可以考虑 DevOps：

* 缩短应用程序开发和部署生命周期
* 将质量和一致性管理内置到自动化的生成和发布过程中

## <a name="architecture"></a>体系结构

![从体系结构的角度概要说明某个使用 Visual Studio Team Services 和 Azure 应用服务的 DevOps 方案中涉及的 Azure 组件][architecture]

本方案介绍的 DevOps 管道针对一个使用 Visual Studio Team Services (VSTS) 的 .NET Web 应用程序。 数据流经方案的情形如下所示：

1. 更改应用程序源代码。
2. 提交应用程序代码和 Web 应用 web.config 文件。
3. 持续集成触发应用程序生成和单元测试。
4. 持续部署触发器使用特定于环境的参数化配置值来协调应用程序项目的部署。
5. 部署到 Azure 应用服务。
6. Azure Application Insights 收集并分析运行状况、性能和使用情况数据。
7. 查看运行状况、性能和使用情况信息。

### <a name="components"></a>组件

* [资源组][resource-groups]是 Azure 资源的逻辑容器，此外还为管理平面提供一个访问控制边界 - 可以将资源组视为“部署单元”的体现。
* [Visual Studio Team Services (VSTS)][vsts] 是一项服务，用于对开发生命周期进行端到端管理，包括规划和项目管理、代码管理以及生成和发布管理。
* [Azure Web 应用][web-apps]是一种平台即服务 (PaaS) 服务，用于托管 Web 应用程序、REST API 和移动后端。 虽然本文着重讨论 .NET，但系统也支持多个其他的开发平台选项。
* [Application Insights][application-insights] 是第一方的可扩展应用程序性能管理 (APM) 服务，适用于多个平台上的 Web 开发人员。

### <a name="alternative-devops-tooling-options"></a>替代 DevOps 工具选项

虽然本文重点介绍 Visual Studio Team Services，但也可在本地使用 [Team Foundation Server][team-foundation-server] 作为替代。 另外，也可将一系列技术一起用于某个利用 [Jenkins][jenkins-on-azure] 的开源开发管道。

从基础结构即代码角度来看，可将 [Azure 资源管理器 (ARM) 模板][arm-templates]包括在 Azure DevOps 项目中，但也可考虑使用 [Terraform][terraform] 或 [Chef][chef]（如果在这里有投资）。 如果首选以基础结构即服务 (IaaS) 为基础的部署并且需要配置管理，则可考虑 [Azure Desired State Configuration][desired-state-configuration]、[Ansible][ansible] 或 [Chef][chef]。

### <a name="alternatives-to-web-app-hosting"></a>Web 应用托管的替代方式

以下方式可以替代在 Azure Web 应用中进行的托管：

* [VM][compare-vm-hosting] - 适用于需要进行严格控制的工作负荷，或者所适用的工作负荷依赖于 OS 组件/服务，而后者 Web 应用无法提供（例如 Windows GAC 或 COM）
* [容器托管][azure-containers] - 适用于依赖 OS 以及对托管可移植性或托管密度也有要求的情况。
* [Service Fabric][service-fabric] - 如果工作负荷体系结构侧重于分布式组件，而此类组件适合在严格控制的群集上部署和运行，则可使用此选项。 Service Fabric 也可用于托管容器。
* [无服务器 Azure Functions][azure-functions] - 如果工作负荷体系结构侧重于精细的分布式组件，几乎不需要依赖，只在需要的情况下运行单个组件（不需持续运行），且不需对组件的运行进行协调，则可使用此选项。

### <a name="devops"></a>DevOps

**[持续集成 (CI)][continuous-integration]** 的目标应该是体现稳定生成的优势，即允许多个单独的开发人员或团队持续向共享的代码库提交小的频繁进行的更改。
在进行持续集成管道操作时，应遵循以下要求：

* 频繁签入少量代码（避免批量签入较大的或较复杂的更改，因为这样会更难合并成功）
* 对应用程序组件进行单元测试时，需涵盖足够多的代码（包括异常路径）
* 确保针对共享的主分库运行生成。 此分库应该稳定且始终处于“准备部署”状态。 不完整的或者正在进行的更改应该置于单独的可以频繁进行“前向集成”式合并的分库中，避免以后发生冲突。

**[持续交付 (CD)][continuous-delivery]** 的目标应该是体现稳定生成和稳定部署的优势。 这样会使得 CD 的实现更为困难一些，既需进行特定于环境的配置，又需通过某个机制来确保这些值设置正确。

另外，集成测试还需要有足够的涵盖面，确保以端到端的方式正确配置和运行各种组件。

这可能还需要设置和重置特定于环境的数据，以及管理数据库架构版本。

持续交付可能还会扩展到负载测试和用户验收测试环境。

持续交付得益于理想情况下对所有环境进行的持续监视。
可以针对创建和配置基础结构或托管基础结构来编写脚本，这样就可以更容易地确保跨环境进行的部署和集成测试的一致性和可靠性（对于基于云的工作负荷来说，这要更容易得多，详见“Azure 基础结构即代码”）- 这也称为[“基础结构即代码”][infra-as-code]。

* 在项目生命周期中尽早启动持续交付。 越晚越困难。
* 作为项目功能，集成和单元测试的优先级应该是一样的。
* 使用与环境无关的部署包，在发布过程中管理特定于环境的配置。
* 在发布管理工具中保护敏感配置，或者通过在发布过程中调用硬件安全模块 (HSM) 或 [Key Vault][azure-key-vault] 来这样做。 请勿在源代码管理中存储敏感配置。

**持续学习** - 对 CD 环境进行的最有效监视由应用程序性能监视（简称 APM）工具（例如 Microsoft 的 [Application Insights][application-insights]）提供。 若要了解 Bug 和负载下的性能，必须针对应用程序工作负荷进行够深入的监视。 [可以将 App Insights 集成到 VSTS 中以持续监视 CD 管道][app-insights-cd-monitoring]。 可以通过这种方式启用自动进入下一阶段的功能（不需人工干预），或者启用在检测到警报的情况下自动回退的功能。

## <a name="considerations"></a>注意事项

### <a name="availability"></a>可用性

考虑在生成云应用程序时，利用[针对可用性的典型设计模式][design-patterns-availability]。

在适当的[应用服务 Web 应用程序参考体系结构][app-service-reference-architecture]中审核可用性注意事项

若要了解其他可用性主题，请参阅 Azure 体系结构中心的[可用性核对清单][availability]。

### <a name="scalability"></a>可伸缩性

生成云应用程序时，请弄清楚[针对可伸缩性的典型设计模式][design-patterns-scalability]。

在适当的[应用服务 Web 应用程序参考体系结构][app-service-reference-architecture]中审核可伸缩性注意事项

若要了解其他可伸缩性主题，请参阅 Azure 体系结构中心的[可伸缩性核对清单][scalability]。

### <a name="security"></a>“安全”

考虑在适当情况下利用[针对安全性的典型设计模式][design-patterns-security]。

在适当的[应用服务 Web 应用程序参考体系结构][app-service-reference-architecture]中审核安全性注意事项。

若需安全解决方案的通用设计指南，请参阅 [Azure 安全性文档][security]。

### <a name="resiliency"></a>复原

审核[针对复原的典型设计模式][design-patterns-resiliency]，考虑在适当情况下实施这些模式。

可以在体系结构中心找到许多[建议用于应用服务的复原做法][resiliency-app-service]。

若需可复原解决方案的通用设计指南，请参阅[设计适用于 Azure 的可复原应用程序][resiliency]。

## <a name="deploy-the-scenario"></a>部署方案

### <a name="prerequisites"></a>先决条件

* 必须已经有 Azure 帐户。 如果还没有 Azure 订阅，可以在开始前创建一个[免费帐户][azure-free-account]。
* 必须有一个现成的 Visual Studio Team Services (VSTS) 帐户。 请了解有关如何[创建 Visual Studio Team Services (VSTS) 帐户][vsts-account-create]的更多详细信息。

### <a name="walk-through"></a>演练

在本方案中，需使用 Azure DevOps 项目创建 CI/CD 管道。

DevOps 项目会部署应用服务计划、应用服务以及 App Insights 资源，并配置 Visual Studio Team Services 项目。

有了 DevOps 项目并完成生成以后，请查看相关联的代码更改、工作项和测试结果。 你会注意到系统未显示测试结果，因为代码不包含任何需要运行的测试。

查看发布定义。 请注意，发布管道已设置好，可以将应用程序发布到开发环境中。 请注意，有一个从 **Drop** 生成项目设置的**持续部署触发器**，可以将内容自动发布到开发环境中。 在持续部署过程中，可能会看到跨多个环境的发布。 发布可以跨基础结构（使用基础结构即代码之类的技术），还可以部署所需的应用程序包以及任何配置后任务。

**其他注意事项。**

* 考虑利用在 VSTS 市场中提供的一个[词汇切分任务][vsts-tokenization]。
* 考虑通过[部署：Azure Key Vault][download-keyvault-secrets] VSTS 任务将机密从 Azure KeyVault 下载到发布中。 然后即可将这些机密用作发布定义中的变量，但不应将其存储在源代码管理中。
* 考虑在发布定义中使用[发布变量][vsts-release-variables]来驱动对环境的配置更改。 发布变量的作用域可以是整个发布，也可以是给定的环境。 如果将变量用于机密信息，请确保选择挂锁图标。
* 考虑在发布管道中使用[部署入口][vsts-deployment-gates]。 这样就可以利用与外部系统（例如，事件管理系统或其他定制系统）关联的监视数据，以便确定是否应提升某个发布。
* 如果需要在发布管道中进行手动干预，请考虑使用[审批][vsts-approvals]功能。
* 考虑在发布管道中尽早使用 [Application Insights][application-insights] 和其他监视工具。 大多数组织只在生产环境中开启监视功能，不过你可以早一点在这个过程中确定可能的 Bug，避免对生产环境中的用户造成影响。

## <a name="pricing"></a>定价

Visual Studio Team Services 成本计算取决于组织中需要进行访问的用户数，以及所需的并发生成/发布数、测试用户数等因素。 这些在 [VSTS 定价页][vsts-pricing-page]中有更详细的说明。

* [Visual Studio Team Services (VSTS)][vsts-pricing-calculator] 是一项用于管理开发生命周期的服务，按月按用户付款。 可能会有其他的费用，具体取决于所需的并发管道数，以及是否有其他测试用户，或者是否使用了基本的用户许可证。

## <a name="related-resources"></a>相关资源

* [什么是 DevOps？][devops-whatis]
* [Microsoft 的 DevOps - 如何使用 Visual Studio Team Services][devops-microsoft]
* [分步教程：Visual Studio Team Services 的 DevOps][devops-with-vsts]
* [使用 Azure DevOps 项目创建用于 .NET 的 CI/CD 管道][devops-project-create]

<!-- links -->
[ansible]: /azure/ansible/
[application-insights]: /azure/application-insights/app-insights-overview
[app-service-reference-architecture]: /azure/architecture/reference-architectures/app-service-web-app/
[azure-free-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[architecture]: ./media/devops-dotnet-webapp/architecture-devops-dotnet-webapp.png
[availability]: /azure/architecture/checklist/availability
[chef]: /azure/chef/
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[desired-state-configuration]: /azure/automation/automation-dsc-overview
[devops-microsoft]: /azure/devops/devops-at-microsoft/
[devops-with-vsts]: https://almvm.azurewebsites.net/labs/vsts/
[application-insights]: https://azure.microsoft.com/en-gb/services/application-insights/
[cloud-based-load-testing]: https://visualstudio.microsoft.com/team-services/cloud-load-testing/
[cloud-based-load-testing-on-premises]: /vsts/test/load-test/clt-with-private-machines?view=vsts
[jenkins-on-azure]: /azure/jenkins/
[devops-whatis]: /azure/devops/what-is-devops
[download-keyvault-secrets]: /vsts/pipelines/tasks/deploy/azure-key-vault?view=vsts
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency-app-service]: /azure/architecture/checklist/resiliency-per-service#app-service
[resiliency]: /azure/architecture/checklist/resiliency
[scalability]: /azure/architecture/checklist/scalability
[vsts]: /vsts/?view=vsts#pivot=services
[continuous-integration]: /azure/devops/what-is-continuous-integration
[continuous-delivery]: /azure/devops/what-is-continuous-delivery
[web-apps]: /azure/app-service/app-service-web-overview
[terraform]: /azure/terraform/
[vsts-account-create]: /vsts/organizations/accounts/create-account-msa-or-work-student?view=vsts
[vsts-approvals]: /vsts/pipelines/release/approvals/approvals?view=vsts
[devops-project]: https://portal.azure.com/?feature.customportal=false#create/Microsoft.AzureProject
[vsts-deployment-gates]: /vsts/pipelines/release/approvals/gates?view=vsts
[vsts-pricing-calculator]: https://azure.com/e/498aa024454445a8a352e75724f900b1
[vsts-pricing-page]: https://azure.microsoft.com/en-us/pricing/details/visual-studio-team-services/
[vsts-release-variables]: /vsts/pipelines/release/variables?view=vsts&tabs=batch
[vsts-tokenization]: https://marketplace.visualstudio.com/search?term=token&target=VSTS&category=All%20categories&sortBy=Relevance
[azure-key-vault]: /azure/key-vault/key-vault-overview
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[team-foundation-server]: https://visualstudio.microsoft.com/tfs/
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[service-fabric]:/azure/service-fabric/
[azure-functions]:/azure/azure-functions/
[azure-containers]:https://azure.microsoft.com/en-us/overview/containers/
[compare-vm-hosting]:/azure/app-service/choose-web-site-cloud-service-vm
[app-insights-cd-monitoring]:/azure/application-insights/app-insights-vsts-continuous-monitoring
[azure-region-pair-bcdr]:/azure/best-practices-availability-paired-regions
[devops-project-create]: /vsts/pipelines/apps/cd/azure/azure-devops-project-aspnetcore?view=vsts
[security]: /azure/security/