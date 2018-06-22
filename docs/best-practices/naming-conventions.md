---
title: Azure 资源的命名约定
description: Azure 资源的命名约定。 如何命名虚拟机、存储帐户、网络、虚拟网络、子网和其他 Azure 实体
author: telmosampaio
ms.date: 05/18/2017
pnp.series.title: Best Practices
ms.openlocfilehash: a92b6a1a23b35e7379f586d477b6f7cc6ccfc7e1
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206372"
---
# <a name="naming-conventions"></a>命名约定

[!INCLUDE [header](../_includes/header.md)]

本文包含有关 Azure 资源的命名规则和限制的总结，以及命名约定的一系列基本建议。  可以将这些建议作为符合自己特定需求的约定的起点。

Microsoft Azure 中任何资源的名称选择都很重要，因为：

* 之后很难更改名称。
* 名称必须满足它们特定资源类型的要求。

一致的命名约定使得资源更易于查找。 它们还可以指示解决方案中资源的角色。

命名约定成功的关键是在应用程序和组织中创建并遵循它们。

## <a name="naming-subscriptions"></a>命名订阅
命名 Azure 订阅时，详细的名称有助于清楚地了解每个订阅的上下文和目的。  在具有许多订阅的环境中工作时，遵循共享命名约定可以提高简明性。

建议的命名订阅模式是：

`<Company> <Department (optional)> <Product Line (optional)> <Environment>`

* 公司通常对每个订阅都是一样的。 但是，一些公司可能存在组织结构内的子公司。 这些公司可能由中心 IT 组管理。 在这些情况下，可能会用母公司名称（Contoso）加上子公司名称（Northwind）来加以区分。
* 部门是组织内的名称，包含一组员工。 命名空间中的此项为可选项。
* 产品线是部门中执行的产品或功能的特定名称。 对于面向内部的服务和应用程序，这通常是可选的。 但是强烈建议将此用于需要轻松分离和识别的面向公众的服务（例如，清晰分离账单记录）。
* 环境是描述应用程序或服务（如开发、QA 或生产）的部署生命周期的名称。

| 公司 | 部门 | 产品线或服务 | 环境 | 全名 |
| --- | --- | --- | --- | --- |
| Contoso |SocialGaming |AwesomeService |生产 |Contoso SocialGaming AwesomeService Production |
| Contoso |SocialGaming |AwesomeService |Dev |Contoso SocialGaming AwesomeService Dev |
| Contoso |IT |InternalApps |生产 |Contoso IT InternalApps Production |
| Contoso |IT |InternalApps |Dev |Contoso IT InternalApps Dev |

有关如何为较大型企业组织订阅的详细信息，请参阅[订阅管理说明性指南][scaffold]。

## <a name="use-affixes-to-avoid-ambiguity"></a>使用词缀以避免多义性

命名 Azure 中的资源时，建议使用常用前缀或后缀来标识资源的类型和上下文。  虽然关于类型、元数据和上下文的所有信息可通过编程方式获得，但应用常用词缀可简化视觉识别。  将词缀引入命名约定时，请务必明确指定词缀位于名称的开头（前缀）还是结尾（后缀）。

例如，下面是托管计算引擎的服务的两个可能名称：

* SvcCalculationEngine（前缀）
* CalculationEngineSvc（后缀）

词缀可以引用描述特定资源的不同方面。 下表显示了通常使用的一些示例。

| 方面 | 示例 | 说明 |
| --- | --- | --- |
| 环境 |dev，prod，QA |标识资源的环境 |
| 位置 |uw（美国西部），ue（美国东部） |标识要部署资源的区域 |
| 实例 |01，02 |适用于具有多个命名实例的资源（Web 服务器等）。 |
| 产品或服务 |服务 |标识资源支持的产品、应用程序或服务 |
| 角色 |sql，web，messaging |标识关联的资源的角色 |

为公司或项目制定特定的命名约定时，请务必选择一组常用的词缀和位置（后缀或前缀）。

## <a name="naming-rules-and-restrictions"></a>命名规则和限制

Azure 中的每个资源或服务类型强制实施一组命名限制和范围；任何命名约定或模式必须符合必需的命名规则和范围。  例如，虽然 VM 的名称映射到 DNS 名称（因此其需要在整个 Azure 中保持唯一），但 VNET 的范围设置为创建它的资源组。

通常，应避免将任何特殊字符（`-` 或 `_`）作为任何名称的第一个或最后一个字符。 这些字符将导致大多数验证规则失败。

### <a name="general"></a>常规

| 实体 | 范围 | Length | 大小写 | 有效的字符 | 建议的模式 | 示例 |
| --- | --- | --- | --- | --- | --- | --- |
|资源组 |订阅 |1-90 |不区分大小写 |字母数字、下划线、括号、连字符、句点（位于末尾的除外） |`<service short name>-<environment>-rg` |`profx-prod-rg` |
|可用性集 |资源组 |1-80 |不区分大小写 |字母数字、下划线和连字符 |`<service-short-name>-<context>-as` |`profx-sql-as` |
|标记 |关联的实体 |512（名称）、256（值） |不区分大小写 |字母数字 |`"key" : "value"` |`"department" : "Central IT"` |

### <a name="compute"></a>计算

| 实体 | 范围 | Length | 大小写 | 有效的字符 | 建议的模式 | 示例 |
| --- | --- | --- | --- | --- | --- | --- |
|虚拟机 |资源组 |1-15 (Windows)、1-64 (Linux) |不区分大小写 |字母数字和连字符 |`<name>-<role>-vm<number>` |`profx-sql-vm1` |
|Function App | 全局 |1-60 |不区分大小写 |字母数字和连字符 |`<name>-func` |`calcprofit-func` |

> [!NOTE]
> Azure 中的虚拟机具有两个不同的名称：虚拟机名称和主机名。 在门户中创建 VM 时，主机名和虚拟机资源名称使用相同的名称。 以上限制适用于主机名。 实际资源名称最多可包含 64 个字符。

### <a name="storage"></a>存储

| 实体 | 范围 | Length | 大小写 | 有效的字符 | 建议的模式 | 示例 |
| --- | --- | --- | --- | --- | --- | --- |
|存储帐户名称（数据） |全局 |3-24 |小写 |字母数字 |`<globally unique name><number>`（使用函数计算命名存储帐户的唯一 Guid） |`profxdata001` |
|存储帐户名称（磁盘） |全局 |3-24 |小写 |字母数字 |`<vm name without hyphens>st<number>` |`profxsql001st0` |
| 容器名称 |存储帐户 |3-63 |小写 |字母数字和连字符 |`<context>` |`logs` |
|Blob 名称 | 容器 |1-1024 |区分大小写 |任何 URL 字符 |`<variable based on blob usage>` |`<variable based on blob usage>` |
|队列名称 |存储帐户 |3-63 |小写 |字母数字和连字符 |`<service short name>-<context>-<num>` |`awesomeservice-messages-001` |
|表名称 | 存储帐户 |3-63 |不区分大小写 |字母数字 |`<service short name><context>` |`awesomeservicelogs` |
|文件名 | 存储帐户 |3-63 |小写 | 字母数字 |`<variable based on blob usage>` |`<variable based on blob usage>` |
|Data Lake Store | 全局 |3-24 |小写 | 字母数字 |`<name>dls` |`telemetrydls` |

### <a name="networking"></a>网络

| 实体 | 范围 | Length | 大小写 | 有效的字符 | 建议的模式 | 示例 |
| --- | --- | --- | --- | --- | --- | --- |
|虚拟网络 (VNet) |资源组 |2-64 |不区分大小写 |字母数字、连字符、下划线和句点 |`<service short name>-vnet` |`profx-vnet` |
|子网 |父级 VNet |2-80 |不区分大小写 |字母数字、连字符、下划线和句点 |`<descriptive context>` |`web` |
|网络接口 |资源组 |1-80 |不区分大小写 |字母数字、连字符、下划线和句点 |`<vmname>-nic<num>` |`profx-sql1-nic1` |
|网络安全组 |资源组 |1-80 |不区分大小写 |字母数字、连字符、下划线和句点 |`<service short name>-<context>-nsg` |`profx-app-nsg` |
|网络安全组规则 |资源组 |1-80 |不区分大小写 |字母数字、连字符、下划线和句点 |`<descriptive context>` |`sql-allow` |
|公共 IP 地址 |资源组 |1-80 |不区分大小写 |字母数字、连字符、下划线和句点 |`<vm or service name>-pip` |`profx-sql1-pip` |
|负载均衡器 |资源组 |1-80 |不区分大小写 |字母数字、连字符、下划线和句点 |`<service or role>-lb` |`profx-lb` |
|负载均衡规则配置 |负载均衡器 |1-80 |不区分大小写 |字母数字、连字符、下划线和句点 |`<descriptive context>` |`http` |
|Azure 应用程序网关 |资源组 |1-80 |不区分大小写 |字母数字、连字符、下划线和句点 |`<service or role>-agw` |`profx-agw` |
|流量管理器配置文件 |资源组 |1-63 |不区分大小写 |字母数字、连字符和句点 |`<descriptive context>` |`app1` |

## <a name="organize-resources-with-tags"></a>使用标记组织资源

Azure 资源管理器支持使用任意文本字符串标记实体，以标识上下文和简化自动化。  例如，`"sqlVersion: "sql2014ee"` 标记可以标识运行 SQL Server 2014 Enterprise Edition 的部署中的 VM，以针对其运行自动化脚本。  应将标记与所选的命名约定结合使用，以便增加和增强上下文。

> [!TIP]
> 标记的另一个优点是标记跨资源组，这样用户可跨不同的部署链接和关联实体。

每个资源或资源组最多可以有 15 个标记。 标记名称不能超过 512 个字符，标记值不能超过 256 个字符。

有关详细信息，请参阅[使用标记来组织 Azure 资源](/azure/azure-resource-manager/resource-group-using-tags/)。

下面是一些常见的标记用例：

* **账单**；将资源分组并将其与账单或退款代码关联。
* **服务上下文标识**；跨资源组进行标识，用于常见操作和分组。
* **访问控制和安全性上下文**；基于项目组合、系统、服务、应用和实例等的管理角色标识。

> [!TIP]
> 尽可能早作标记、勤作标记。  最好有一个基线标记方案，并不断进行调整，而不是在事后进行改进。

一些常见的标记方法的示例：

| 标记名称 | 密钥 | 示例 | 注释 |
| --- | --- | --- | --- |
| 付款人/内部退款 ID |付款人 |`IT-Chargeback-1234` |内部 I/O 或账单代码 |
| 操作员或直接负责人 (DRI) |managedBy |`joe@contoso.com` |别名或电子邮件地址 |
| 项目名称 |projectName |`myproject` |项目或产品系列的名称 |
| 项目版本 |projectVersion |`3.4` |项目或产品系列的版本 |
| 环境 |环境 |`<Production, Staging, QA >` |环境标识符 |
| 层 |层 |`Front End, Back End, Data` |层或角色/上下文标识 |
| 数据配置文件 |dataProfile |`Public, Confidential, Restricted, Internal` |存储在资源中的数据的敏感性 |

## <a name="tips-and-tricks"></a>提示和技巧

某些资源类型的命名和约定可能需要额外关注。

### <a name="virtual-machines"></a>虚拟机

尤其是在大型拓扑中，通过仔细命名虚拟机可简化对每台虚拟机的角色和用途的标识，并实现更具预测性的脚本编写。

### <a name="storage-accounts-and-storage-entities"></a>存储帐户和存储实体

存储帐户有两个主要用例 - 备份 VM 的磁盘以及将数据存储在 Blob、队列和表中。  用于 VM 磁盘的存储帐户应遵循以下命名约定：将其与父级 VM 名称相关联（并且由于可能需要为高端 VM SKU 使用多个存储帐户，还应应用数字后缀）。

> [!TIP]
> 存储帐户（无论用于数据还是磁盘）应遵循允许采用多个存储帐户的命名约定（即始终使用数字后缀）。

可以配置自定义域名以便访问 Azure 存储帐户中的 Blob 数据。 BLOB 服务的默认终结点是 https://\<name\>.blob.core.windows.net。

但是如果将自定义域（如 www.contoso.com）映射到存储帐户的 Blob 终结点，则用户也可以使用该域访问存储帐户中的 Blob 数据。 例如，对于自定义域名，`http://mystorage.blob.core.windows.net/mycontainer/myblob` 可以作为 `http://www.contoso.com/mycontainer/myblob` 访问。

有关配置此功能的详细信息，请参阅[为 Blob 存储终结点配置自定义域名](/azure/storage/storage-custom-domain-name/)。

有关命名 Blob、容器和表的详细信息，请参阅以下列表：

* [命名和引用容器、Blob 与元数据](https://msdn.microsoft.com/library/dd135715.aspx)
* [命名队列和元数据](https://msdn.microsoft.com/library/dd179349.aspx)
* [命名表](https://msdn.microsoft.com/library/azure/dd179338.aspx)

Blob 名称可以包含任何字符组合，但必须正确转义保留的 URL 字符。 Blob 名称应避免使用句点 (.)、正斜杠 (/) 或两者的组合结尾。 根据约定，正斜杠是虚拟目录分隔符。 请勿在 Blob 名称中使用反斜杠 (\\)。 客户端 API 可能会允许使用反斜杠，但无法正确哈希，并且签名将不匹配。

存储帐户或容器的名称创建后，无法再对其进行修改。 如果想要使用新的名称，必须将其删除并创建一个新名称。

> [!TIP]
> 我们建议在开始开发新服务或应用程序前，为所有存储帐户和类型创建命名约定。

<!-- links -->

[scaffold]: /azure/azure-resource-manager/resource-manager-subscription-governance
