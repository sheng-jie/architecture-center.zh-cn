---
title: 用于酒店预订的对话式 Azure 聊天机器人
description: 经验证的解决方案，所生成的对话式聊天机器人适合使用 Azure 机器人服务、认知服务和 LUIS，Azure SQL 数据库以及 Application Insights 的商业应用程序。
author: iainfoulds
ms.date: 07/05/2018
ms.openlocfilehash: 85bdc3194961bbbd8d89db34e5c56e4baa8d8599
ms.sourcegitcommit: 5d99b195388b7cabba383c49a81390ac48f86e8a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/06/2018
ms.locfileid: "37891268"
---
# <a name="conversational-azure-chatbot-for-hotel-reservations"></a>用于酒店预订的对话式 Azure 聊天机器人

本示例方案适用于需将对话式聊天机器人集成到应用程序中的企业。 在本解决方案中，将对连锁酒店使用一个 C# 聊天机器人，以便客户通过 Web 或移动应用程序查看房间可用情况并进行预订。

示例场景包括：允许客户查看酒店房间可用情况并订房、允许客户查看餐馆外卖菜单并下单，或者允许客户搜索并订购照片。 传统上，企业需雇佣并培训客户服务代理来响应此类客户请求，而客户则需等待代理为其提供帮助。

有了机器人服务和语言理解或语音 API 服务等 Azure 服务，公司就可以使用自动化的可缩放机器人帮助客户并处理订单或预订。

## <a name="potential-use-cases"></a>可能的用例

以下用例可以考虑本解决方案：

* 查看餐馆外卖菜单并下单
* 查看酒店房间可用情况并订房
* 搜索并订购提供的照片

## <a name="architecture"></a>体系结构

![从体系结构的角度概要说明对话式聊天机器人中涉及的 Azure 组件][architecture]

本解决方案涉及的对话式聊天机器人充当酒店接待。 数据流经解决方案的情形如下所示：

1. 客户通过移动或 Web 应用访问此聊天机器人。
2. 用户通过 Azure Active Directory B2C（企业对客户）进行身份验证。
3. 用户与机器人服务交互即可了解酒店房间可用情况。
4. 认知服务通过处理自然语言请求来理解客户通信。
5. 用户对结果满意之后，机器人会在 SQL 数据库中添加或更新客户预订项目。
6. 在此过程中，Application Insights 会一直收集运行时遥测数据，以便 DevOps 团队改进机器人性能和使用效果。

### <a name="components"></a>组件

* [Azure Active Directory][aad-docs] 是 Microsoft 提供的多租户、基于云的目录和标识管理服务。 Azure AD 支持使用 B2C 连接器，目的是确定使用外部 ID（例如 Google、Facebook 或 Microsoft 帐户）的用户。
* [应用服务][appservice-docs]允许你采用所选编程语言生成和托管 Web 应用程序，无需管理基础结构。
* [机器人服务][botservice-docs]提供的工具可用于生成、测试、部署和管理智能机器人。
* [认知服务][cognitive-docs]允许你使用智能算法，以自然的沟通方式来观察、倾听、表述、了解和解释用户需求。
* [SQL 数据库][sqldatabase-docs]是一种完全托管的云关系数据库服务，可以与 SQL Server 引擎兼容。
* [Application Insights][appinsights-docs] 是可扩展的应用程序性能管理 (APM) 服务，用于监视应用程序（例如聊天机器人）的性能。

### <a name="alternatives"></a>备选项

* [Microsoft 语音 API][speech-api] 可以用来更改客户与机器人的交互方式。
* [QnA Maker][qna-maker] 可以用来以半结构化内容（例如常见问题解答）的形式为机器人快速添加知识。
* [文本翻译 API][translator] 是一项可以考虑使用的服务，可以轻松地向机器人添加多语音支持。

## <a name="considerations"></a>注意事项

### <a name="availability"></a>可用性

本解决方案使用 Azure SQL 数据库来存储客户预订项目。 SQL 数据库包括区域冗余数据库、故障转移组和异地复制。 有关详细信息，请参阅 [Azure SQL 数据库可用性功能][sqlavailability-docs]。

若要了解其他可伸缩性主题，请参阅 Azure 体系结构中心的[可用性核对清单][availability]。

### <a name="scalability"></a>可伸缩性

本解决方案使用 Azure 应用服务。 可以通过应用服务自动调整运行机器人的实例数。 可以通过此功能时刻了解客户对 Web 应用程序和聊天机器人的需求情况。 有关自动缩放的详细信息，请参阅体系结构中心的[自动缩放最佳做法][autoscaling]。

若要了解其他可伸缩性主题，请参阅 Azure 体系结构中心的[可伸缩性核对清单][scalability]。

### <a name="security"></a>“安全”

本解决方案使用 Azure Active Directory B2C（企业对客户）进行用户身份验证。 使用 AAD B2C 时，聊天机器人不存储任何敏感的客户帐户信息或凭据。 有关详细信息，请参阅 [Azure Active Directory B2C 概述][aadb2c-docs]。

存储在 Azure SQL 数据库中的信息使用透明数据加密 (TDE) 进行静态加密。 SQL 数据库还提供 Always Encrypted，用于在查询和处理过程中加密数据。 有关 SQL 数据库安全性的详细信息，请参阅 [Azure SQL 数据库安全性和符合性][sqlsecurity-docs]。

若需安全解决方案的通用设计指南，请参阅 [Azure 安全性文档][security]。

### <a name="resiliency"></a>复原

本解决方案使用 Azure SQL 数据库来存储客户预订项目。 SQL 数据库包括区域冗余数据库、故障转移组、异地复制和自动备份。 有了这些功能，应用程序就可以在进行维护或出现服务中断的情况下继续运行。 有关详细信息，请参阅 [Azure SQL 数据库可用性功能][sqlavailability-docs]。

本解决方案使用 Application Insights 来监视应用程序的运行状况。 可以使用 Application Insights 生成警报并响应那些会影响客户体验和聊天机器人可用性的性能问题。 有关详细信息，请参阅[什么是 Application Insights？][appinsights-docs]

若需可复原解决方案的通用设计指南，请参阅[设计适用于 Azure 的可复原应用程序][resiliency]。

## <a name="deploy-the-solution"></a>部署解决方案

本解决方案分为三个组件，方便你了解自己最关注的领域：

* [基础结构组件](#deploy-infrastructure-components)。 使用 Azure 资源管理器模板来部署应用服务、Web 应用、Application Insights、存储帐户以及 SQL Server 和数据库的核心基础结构组件。
* [Web 应用聊天机器人](#deploy-web-app-chatbot)。 使用 Azure CLI 将机器人与机器人服务、语言理解和智能服务 (LUIS) 应用部署在一起。
* [C# 聊天机器人应用程序示例](#deploy-chatbot-c-application-code)。 使用 Visual Studio 复查示例性的酒店预订 C# 应用程序代码并将其部署到 Azure 中的机器人。

**先决条件：** 必须已经有 Azure 帐户。 如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

### <a name="deploy-infrastructure-components"></a>部署基础结构组件

若要通过 Azure 资源管理器模板部署基础结构组件，请执行以下步骤。

1. 单击“部署到 Azure”按钮：<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fcommerce-chatbot.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. 等待模板部署在 Azure 门户中打开，然后完成以下步骤：
   * 选择“新建”资源组，然后在文本框中提供一个名称，例如 *myCommerceChatBotInfrastructure*。
   * 从“位置”下拉框中选择区域。
   * 提供用于 SQL Server 管理员帐户的用户名和安全密码。
   * 查看条款和条件，然后勾选“我同意上述条款和条件”。
   * 选择“购买”按钮。

需要几分钟才能完成部署。

### <a name="deploy-web-app-chatbot"></a>部署 Web 应用聊天机器人

若要创建此聊天机器人，请使用 Azure CLI。 以下示例先安装机器人服务的 CLI 扩展，接着创建资源组，然后部署使用 Application Insights 的机器人。 系统提示时，请验证 Microsoft 帐户，并允许机器人自行注册到机器人服务以及语言理解和智能服务 (LUIS) 应用。

```azurecli-interactive
# Install the Azure CLI extension for the Bot Service
az extension add --name botservice --yes

# Create a resource group
az group create --name myCommerceChatbot --location eastus

# Create a Web App Chatbot that uses Application Insights
az bot create \
    --resource-group myCommerceChatbot \
    --name commerceChatbot \
    --location eastus \
    --kind webapp \
    --sku S1 \
    --insights eastus
```

### <a name="deploy-chatbot-c-application-code"></a>部署聊天机器人 C# 应用程序代码

GitHub 上提供了一个示例 C# 应用程序： 

* [商业机器人 C# 示例](https://github.com/Microsoft/AzureBotServices-scenarios/tree/master/CSharp/Commerce/src)

示例应用程序包括 Azure Active Directory 身份验证组件，并集成了认知服务的语言理解和智能服务 (LUIS) 组件。 此应用程序要求使用 Visual Studio 来生成和部署解决方案。 有关如何配置 AAD B2C 和 LUIS 应用的其他信息，可参阅 GitHub 存储库文档。

## <a name="pricing"></a>定价

为了方便用户了解运行本解决方案的成本，我们已在成本计算器中预配置了所有服务。 若要了解自己的特定用例的定价变化情况，请按预期的流量更改相应的变量。

我们已根据你预期的聊天机器人需要处理的消息数提供了三个示例性的成本配置文件：

* [小][small-pricing]：对应于每月处理的消息数 < 10,000 的情况。
* [中][medium-pricing]：对应于每月处理的消息数 < 500,000 的情况。
* [大][large-pricing]：对应于每月处理的消息数 < 1 千万的情况。

## <a name="related-resources"></a>相关资源

如需一系列介绍如何利用 Azure 机器人服务的引导式教程，请参阅文档的[教程节点][botservice-docs]。

<!-- links -->
[aadb2c-docs]: /azure/active-directory-b2c/active-directory-b2c-overview
[aad-docs]: /azure/active-directory/
[appinsights-docs]: /azure/application-insights/app-insights-overview
[appservice-docs]: /azure/app-service/
[architecture]: ./media/commerce-chatbot/architecture-commerce-chatbot.png
[autoscaling]: ../../best-practices/auto-scaling.md
[availability]: ../../checklist/availability.md
[botservice-docs]: /azure/bot-service/
[cognitive-docs]: /azure/cognitive-services/
[resiliency]: ../../resiliency/index.md
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[scalability]: ../../checklist/scalability.md
[sqlavailability-docs]: /azure/sql-database/sql-database-technical-overview#availability-capabilities
[sqldatabase-docs]: /azure/sql-database/
[sqlsecurity-docs]: /azure/sql-database/sql-database-technical-overview#advanced-security-and-compliance
[qna-maker]: /azure/cognitive-services/QnAMaker/Overview/overview
[speech-api]: /azure/cognitive-services/speech/home
[translator]: /azure/cognitive-services/translator/translator-info-overview

[small-pricing]: https://azure.com/e/dce05b6184904c50b38e1a8654f726b6
[medium-pricing]: https://azure.com/e/304d17106afc480dbc414f9726078a03
[large-pricing]: https://azure.com/e/8319dd5e5e3d4f118f9029e32a80e887