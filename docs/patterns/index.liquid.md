---
title: 云设计模式
description: Microsoft Azure 的云设计模式
keywords: Azure
ms.openlocfilehash: 4747c896fc6fc5866be782d76c5290d6b49ad451
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/06/2018
---
# <a name="cloud-design-patterns"></a>云设计模式

[!INCLUDE [header](../../_includes/header.md)]

这些设计模式可用于在云中构建可靠且可缩放的安全应用程序。

每种模式描述了该模式解决的问题、有关应用该模式的注意事项，以及基于 Microsoft Azure 的示例。 大多数模式都包含了代码示例或代码片段，演示如何在 Azure 中实现该模式。 但是，无论托管在 Azure 还是其他云平台中，大多数模式都与任一分布式系统相关。

## <a name="problem-areas-in-the-cloud"></a>云中的问题区域

<ul id="categories" class="panel">
{%- for category in categories %}
    <li>
    {% include 'pattern-category-card' %}
    </li>
{%- endfor %}
</ul>

## <a name="catalog-of-patterns"></a>模式目录

| 模式 | 摘要 |
|---------|---------|
|         |         |

{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}
