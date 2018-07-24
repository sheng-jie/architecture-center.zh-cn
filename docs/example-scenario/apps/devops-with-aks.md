---
title: 适用于基于容器的工作负荷的 CI/CD 管道
description: 经验证的方案，所生成的 DevOps 管道适合使用 Jenkins、Azure 容器注册表、Azure Kubernetes 服务、Cosmos DB 和 Grafana 的 Node.js Web 应用。
author: iainfoulds
ms.date: 07/05/2018
ms.openlocfilehash: d9f6571234a0c3e67a233cfda1a37f6fb32929a3
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060755"
---
# <a name="cicd-pipeline-for-container-based-workloads"></a>适用于基于容器的工作负荷的 CI/CD 管道

本示例方案适用于需要通过容器和 DevOps 工作流实现应用程序开发现代化的企业。 在本方案中，将会通过 Jenkins 生成一个 Node.js Web 应用并将其部署到 Azure 容器注册表和 Azure Kubernetes 服务中。 为了实现全局分布式数据库层，将使用 Azure Cosmos DB。 为了监视应用程序性能并排查其问题，Azure Monitor 会与 Grafana 实例及仪表板集成。

示例应用程序方案包括提供自动化开发环境、验证新的代码提交，以及将新部署推送到过渡环境或生产环境中。 传统上，企业必须手动生成和编译应用程序和更新，维持一个大型代码库。 现代的应用程序开发方法使用持续集成 (CI) 和持续部署 (CD)，可以更快速地生成、测试和部署服务。 此现代方法可以更快速地将应用程序和更新发布给客户，更灵活地响应不断变化的业务需求。

有了 Azure Kubernetes 服务、容器注册表和 Cosmos DB 之类的 Azure 服务，公司就可以使用最新的应用程序开发技术和工具来简化高可用性的实现过程。

## <a name="related-use-cases"></a>相关的用例

以下用例可以考虑本方案：

* 改革应用程序开发做法，采用微服务式的基于容器的方法。
* 缩短应用程序开发和部署生命周期。
* 自动部署到测试或验收环境进行验证。

## <a name="architecture"></a>体系结构

![从体系结构的角度概要说明某个使用 Jenkins、Azure 容器注册表和 Azure Kubernetes 服务的 DevOps 方案中涉及的 Azure 组件][architecture]

本方案涉及一个用于 Node.js Web 应用程序和数据库后端的 DevOps 管道。 数据流经方案的情形如下所示：

1. 开发人员更改 Node.js Web 应用程序源代码。
2. 所做的代码更改提交到某个源代码管理存储库，例如 GitHub。
3. 为了启动持续集成 (CI) 过程，某个 GitHub Webhook 会触发一个 Jenkins 项目生成。
4. Jenkins 生成作业使用 Azure Kubernetes 服务中的动态生成代理来执行容器生成过程。
5. 系统会根据源代码管理中的代码创建容器映像并将其推送到 Azure 容器注册表。
6. Jenkins 通过持续部署 (CD) 将这个更新的容器映像部署到 Kubernetes 群集。
7. Node.js Web 应用程序将 Azure Cosmos DB 用作其后端。 Cosmos DB 和 Azure Kubernetes 服务都会将指标报告给 Azure Monitor。
8. Grafana 实例根据 Azure Monitor 提供的数据在仪表板上直观显示应用程序性能。

### <a name="components"></a>组件

* [Jenkins][jenkins] 是一种开源的自动化服务器，与 Azure 服务集成后即可进行持续集成 (CI) 和持续部署 (CD)。 在本方案中，Jenkins 会根据提交到源代码管理中的内容协调新容器映像的创建过程，接着将这些映像推送到 Azure 容器注册表，然后更新 Azure Kubernetes 服务中的应用程序实例。
* [Azure Linux 虚拟机][azurevm-docs]用于运行 Jenkins 和 Grafana 实例。
* [Azure 容器注册表][azureacr-docs]存储和管理 Azure Kubernetes 服务群集使用的容器映像。 映像会以安全的方式进行存储，并可通过 Azure 平台复制到其他区域以加快部署速度。
* [Azure Kubernetes 服务][azureaks-docs]是一种托管的 Kubernetes 平台，可以让用户在没有容器业务流程专业知识的情况下部署和管理容器化的应用程序。 作为一个托管 Kubernetes 服务，Azure 可以自动处理运行状况监视和维护等关键任务。
* [Azure Cosmos DB][azurecosmosdb-docs] 是一种全局分布式多模型数据库，允许用户根据需要选择不同的数据库和一致性模型。 Cosmos DB 允许全局复制数据，且不需部署和配置群集管理或复制组件。
* [Azure Monitor][azuremonitor-docs] 可以跟踪性能、维护安全和确定趋势。 Monitor 获得的指标可供其他资源和工具（例如 Grafana）使用。
* [Grafana][grafana] 是一种用于查询、可视化、警示和了解指标的开源解决方案。 Grafana 可以通过 Azure Monitor 的数据源插件创建直观的仪表板，以便监视在 Azure Kubernetes 服务中运行并使用 Cosmos DB 的应用程序的性能。

### <a name="alternatives"></a>备选项

* [Visual Studio Team Services][vsts] 和 Team Foundation Server 可以为任何应用实现一个持续集成 (CI)、测试和部署 (CD) 管道。
* 若要对群集进行更多的控制，可以直接在 Azure VM 上运行 [Kubernetes][kubernetes]，不必通过托管服务来运行。
* [Service Fabric][service-fabric] 是另一个可以替代 AKS 的容器业务流程协调程序。

## <a name="considerations"></a>注意事项

### <a name="availability"></a>可用性

为了监视应用程序性能并报告问题，本方案将 Azure Monitor 和 Grafana 组合在一起，以便创建直观的仪表板。 可以通过这些工具监视和排查性能问题，这些问题可能需要代码更新，而这些代码更新均可通过 CI/CD 管道进行部署。

作为 Azure Kubernetes 服务群集的一部分，负载均衡器可以将应用程序流量分发到一个或多个运行应用程序的容器 (Pod)。 可以通过这种在 Kubernetes 中运行容器化应用程序的方法，为客户提供高度可用的基础结构。

若要了解其他可用性主题，请参阅体系结构中心提供的[可用性核对清单][availability]。

### <a name="scalability"></a>可伸缩性

Azure Kubernetes 服务允许按照应用程序的需求调整群集节点的数目。 应用程序增加时，可以扩大运行服务的 Kubernetes 节点数。

应用程序数据存储在 Azure Cosmos DB 中，后者是一种全局分布式多模型数据库，可以进行全局缩放。 Cosmos DB 可以使用传统的数据库组件按需求来缩放基础结构，你可以根据客户的需求选择对 Cosmos DB 进行全局复制。

若要了解其他可伸缩性主题，请参阅体系结构中心提供的[可伸缩性核对清单][scalability]。

### <a name="security"></a>“安全”

为了尽量减少受攻击面，本方案不通过 HTTP 公开 Jenkins VM 实例。 若要执行需要与 Jenkins 交互的管理任务，请使用本地计算机的 SSH 隧道创建安全的远程连接。 Jenkins 和 Grafana VM 实例仅允许 SSH 公钥身份验证。 基于密码的登录已禁用。 有关详细信息，请参阅[在 Azure 上运行 Jenkins 服务器](../../reference-architectures/jenkins/index.md)。

为了将凭据和权限隔离，本方案使用专用的 Azure Active Directory (AD) 服务主体。 此服务主体的凭据以安全的凭据对象形式存储在 Jenkins 中，因此不会在脚本或生成管道中直接公开和可见。

若需安全解决方案的通用设计指南，请参阅 [Azure 安全性文档][security]。

### <a name="resiliency"></a>复原

本方案使用适合应用程序的 Azure Kubernetes 服务。 Kubernetes 内置了复原组件，用于监视容器 (Pod) 并在出现问题时重启容器。 应用程序可以运行多个 Kubernetes 节点，可以容忍一个 Pod 或节点不可用的情况。

若需可复原解决方案的通用设计指南，请参阅[设计适用于 Azure 的可复原应用程序][resiliency]。

## <a name="deploy-the-scenario"></a>部署方案

**先决条件：**

* 必须已经有 Azure 帐户。 如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。
* 需要一个 SSH 公钥对。 有关如何创建公钥对的步骤，请参阅[创建和使用适合 Linux VM 的 SSH 公钥对][sshkeydocs]。
* 需要一个 Azure Active Directory (AD) 服务主体，以便对服务和资源进行身份验证。 可以根据需要使用 [az ad sp create-for-rbac][createsp] 创建一个服务主体

    ```azurecli-interactive
    az ad sp create-for-rbac --name myDevOpsScenario
    ```

    请记下此命令输出中的 *appId* 和 *password*。 部署方案时，请将这些值提供给模板。

若要通过 Azure 资源管理器模板部署此方案，请执行以下步骤。

1. 单击“部署到 Azure”按钮：<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fdevops-with-aks%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. 等待模板部署在 Azure 门户中打开，然后完成以下步骤：
   * 选择“新建”资源组，然后在文本框中提供一个名称，例如 *myAKSDevOpsScenario*。
   * 从“位置”下拉框中选择区域。
   * 通过 `az ad sp create-for-rbac` 命令输入服务主体应用 ID 和密码。
   * 提供用于 Jenkins 实例和 Grafana 控制台的用户名和安全密码。
   * 提供 SSH 密钥，确保安全地登录到 Linux VM。
   * 查看条款和条件，然后勾选“我同意上述条款和条件”。
   * 选择“购买”按钮。

部署可能需要 15-20 分钟才能完成。

## <a name="pricing"></a>定价

为了方便用户了解运行本方案的成本，我们已在成本计算器中预配置了所有服务。 若要了解自己的特定用例的定价变化情况，请按预期的流量更改相应的变量。 更改相应的变量。

我们已根据要存储的容器映像数以及运行应用程序所需的 Kubernetes 节点数提供了三个示例性的成本配置文件。

* [小][small-pricing]：对应于每月 1000 个容器生成的情况。
* [中][medium-pricing]：对应于每月 100,000 个容器生成的情况。
* [大][large-pricing]：对应于每月 1,000,000 个容器生成的情况。

## <a name="related-resources"></a>相关资源

本方案使用了 Azure 容器注册表和 Azure Kubernetes 服务来存储和运行基于容器的应用程序。 Azure 容器实例也可用于运行基于容器的应用程序，不需预配任何业务流程组件。 有关详细信息，请参阅 [Azure 容器实例概述][azureaci-docs]。

<!-- links -->
[architecture]: ./media/devops-with-aks/architecture-devops-with-aks.png
[autoscaling]: ../../best-practices/auto-scaling.md
[availability]: ../../checklist/availability.md
[azureaci-docs]: /azure/container-instances/container-instances-overview
[azureacr-docs]: /azure/container-registry/container-registry-intro
[azurecosmosdb-docs]: /azure/cosmos-db/introduction
[azureaks-docs]: /azure/aks/intro-kubernetes
[azuremonitor-docs]: /azure/monitoring-and-diagnostics/monitoring-overview
[azurevm-docs]: /azure/virtual-machines/linux/overview
[createsp]: /cli/azure/ad/sp#az-ad-sp-create
[grafana]: https://grafana.com/
[jenkins]: https://jenkins.io/
[resiliency]: ../../resiliency/index.md
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[scalability]: ../../checklist/scalability.md
[sshkeydocs]: /azure/virtual-machines/linux/mac-create-ssh-keys
[vsts]: /vsts/?view=vsts
[kubernetes]: https://kubernetes.io/
[service-fabric]: /azure/service-fabric/

[small-pricing]: https://azure.com/e/841f0a75b1ea4802ba1ac8f7918a71e7
[medium-pricing]: https://azure.com/e/eea0e6d79b4e45618a96d33383ec77ba
[large-pricing]: https://azure.com/e/3faab662c54c473da55a1e93a27e0e64