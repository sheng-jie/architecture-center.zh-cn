---
title: 在 Azure 资源管理器模板中实现属性转换器和收集器
description: 描述如何在 Azure 资源管理器模板中实现属性转换器和收集器
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 893779e652b845b3d936d11936dc767ef632fa43
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="implement-a-property-transformer-and-collector-in-an-azure-resource-manager-template"></a>在 Azure 资源管理器模板中实现属性转换器和收集器

在[将对象用作 Azure 资源管理器模板中的参数][objects-as-parameters]中，了解如何在对象中存储资源属性值并在部署时将其应用到资源。 虽然这样管理参数非常有用，但它仍然要求每次在模板中使用时，将对象的属性映射到资源属性。

为了解决此问题，可实现属性转换和收集器模板，循环访问对象数组并将其转换为资源所需的 JSON 架构。

> [!IMPORTANT]
> 此方法要求对资源管理器模板和功能有深入的了解。

让我们通过一个部署[网络安全组 (NSG)][nsg]的示例来看看如何实现属性收集器和转换器。 下图显示了模板和模板中的资源之间的关系：

![属性收集器和转换器体系结构](../_images/collector-transformer.png)

调用模板包含两个资源：
* 调用收集器模板的模板链接。
* 要部署的 NSG 资源。

收集器模板包含两个资源：
* 定位点资源。
* 在复制循环中调用转换模板的模板链接。

转换模板包含一个资源：带有变量的空模板，该模板能将 `source` JSON 转换为主模板中 NSG 资源所需要的 JSON 架构。

## <a name="parameter-object"></a>参数对象

我们使用[对象即参数][objects-as-parameters]中的 `securityRules` 参数对象。 转换模板会将 `securityRules` 数组中的每个对象转换为调用模板中 NSG 资源所需要的 JSON 架构。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters":{ 
      "networkSecurityGroupsSettings": {
      "value": {
          "securityRules": [
            {
              "name": "RDPAllow",
              "description": "allow RDP connections",
              "direction": "Inbound",
              "priority": 100,
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "10.0.0.0/24",
              "sourcePortRange": "*",
              "destinationPortRange": "3389",
              "access": "Allow",
              "protocol": "Tcp"
            },
            {
              "name": "HTTPAllow",
              "description": "allow HTTP connections",
              "direction": "Inbound",
              "priority": 200,
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "10.0.1.0/24",
              "sourcePortRange": "*",
              "destinationPortRange": "80",
              "access": "Allow",
              "protocol": "Tcp"
            }
          ]
        }
      }
    }
  }
```

我们首先看一下转换模板。

## <a name="transform-template"></a>转换模板

转换模板包含从收集器模板传递的两个参数： 
* `source` 是接收属性数组的属性值对象的对象。 在本示例中，`"securityRules"` 数组的每个对象会以一次一个的方式传入。
* `state` 是接收所有以前转换的连接结果的数组。 这是转换后的 JSON 的集合。

参数如下所示：

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "source": { "type": "object" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
  },
```

模板还会定义一个名为 `instance` 的变量。 它执行 `source` 对象到所需的 JSON 架构的实际转换：

```json
  "variables": {
    "instance": [
      {
        "name": "[parameters('source').name]",
        "properties":{
            "description": "[parameters('source').description]",
            "protocol": "[parameters('source').protocol]",
            "sourcePortRange": "[parameters('source').sourcePortRange]",
            "destinationPortRange": "[parameters('source').destinationPortRange]",
            "sourceAddressPrefix": "[parameters('source').sourceAddressPrefix]",
            "destinationAddressPrefix": "[parameters('source').destinationAddressPrefix]",
            "access": "[parameters('source').access]",
            "priority": "[parameters('source').priority]",
            "direction": "[parameters('source').direction]"            
        }
      }
    ]

  },
```

最后，模板的 `output` 将收集的 `state` 参数的转换与当前由 `instance` 变量执行的转换相连接：

```json
  "outputs": {
    "collection": {
      "type": "array",
      "value": "[concat(parameters('state'), variables('instance'))]"
    }
```

接下来，我们介绍收集器模板，了解它如何传入参数值。

## <a name="collector-template"></a>收集器模板

收集器模板包含三个参数：
* `source` 是完整的参数对象数组。 它通过调用模板传入。 它的名称与转换模板中的 `source` 参数相同，但你可能已经注意到了有一个重要差异：它是完整的数组，但我们一次只能将该数组中的一个元素传递给转换模板。
* `transformTemplateUri` 是转换模板的 URI。 为了让模板可以重复使用，我们在此将其定义为参数。
* `state` 刚开始是空数组，我们将其传递给转换模板。 当复制循环完成时，它存储转换后的参数对象的集合。

参数如下所示：

```json
  "parameters": {
    "source": { "type": "array" },
    "transformTemplateUri": { "type": "string" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
``` 

接下来，我们定义一个名为 `count` 的变量。 其值为 `source` 参数对象数组的长度：

```json
  "variables": {
    "count": "[length(parameters('source'))]"
  },
```

正如你所料，我们在复制循环中将其用于迭代数。

现在我们来看看资源。 我们定义两种资源：
* `loop-0` 是复制循环的基于零的资源。
* `loop-` 与 `copyIndex(1)` 函数结果连接，以生成基于迭代且以 `1` 开头的唯一资源名称。

资源如下所示：

```json
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2015-01-01",
      "name": "loop-0",
      "properties": {
        "mode": "Incremental",
        "parameters": { },
        "template": {
          "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": { },
          "variables": { },
          "resources": [ ],
          "outputs": {
            "collection": {
              "type": "array",
              "value": "[parameters('state')]"
            }
          }
        }
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2015-01-01",
      "name": "[concat('loop-', copyindex(1))]",
      "copy": {
        "name": "iterator",
        "count": "[variables('count')]",
        "mode": "serial"
      },
      "dependsOn": [
        "loop-0"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": { "uri": "[parameters('transformTemplateUri')]" },
        "parameters": {
          "source": { "value": "[parameters('source')[copyindex()]]" },
          "state": { "value": "[reference(concat('loop-', copyindex())).outputs.collection.value]" }
        }
      }
    }
  ],
```

我们仔细看一下在嵌套模板中传递给转换模板的参数。 回想一下，`source` 参数传递 `source` 参数对象数组中的当前对象。 `state` 参数发生收集所在的位置，因为它接收复制循环的前一次迭代的输出（请注意，`reference()` 函数使用无参数的 `copyIndex()` 函数来引用以前链接的模板对象的 `name`）并将其传递给当前迭代。

最后，模板的 `output` 返回转换模板的最后一次迭代的 `output`：

```json
  "outputs": {
    "result": {
      "type": "array",
      "value": "[reference(concat('loop-', variables('count'))).outputs.collection.value]"
    }
  }
```
将转换模板的最后一次迭代的 `output` 返回到调用模板似乎有悖常理，因为我们好像是将其存储在 `source` 参数中的。 但请记住，这是转换模板的最后一次迭代，将保存转换后的属性对象的完整数组，并且这就是我们想要返回的内容。

最后，我们看看如何从调用模板中调用收集器模板。

## <a name="calling-template"></a>调用模板

调用模板定义一个名为 `networkSecurityGroupsSettings` 的单个参数：

```json
...
"parameters": {
    "networkSecurityGroupsSettings": {
        "type": "object"
    }
```

接下来，模板定义一个名为 `collectorTemplateUri` 的单个变量：

```json
"variables": {
    "collectorTemplateUri": "[uri(deployment().properties.templateLink.uri, 'collector.template.json')]"
  }
```

如你所料，这是收集器模板的 URI，将由链接模板资源使用：

```json
{
    "apiVersion": "2015-01-01",
    "name": "collector",
    "type": "Microsoft.Resources/deployments",
    "properties": {
        "mode": "Incremental",
        "templateLink": {
            "uri": "[variables('linkedTemplateUri')]",
            "contentVersion": "1.0.0.0"
        },
        "parameters": {
            "source" : {"value": "[parameters('networkSecurityGroupsSettings').securityRules]"},
            "transformTemplateUri": { "value": "[uri(deployment().properties.templateLink.uri, 'transform.json')]"}
        }
    }
}
```

我们向收集器模板传递两个参数：
* `source` 是属性对象数组。 在本示例中是 `networkSecurityGroupsSettings` 参数。
* `transformTemplateUri` 是刚才通过收集器模板的 URI 定义的变量。

最后，`Microsoft.Network/networkSecurityGroups` 资源直接将 `collector` 链接模板资源的 `output` 分配给 `securityRules` 属性：

```json
    {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/networkSecurityGroups",
      "name": "networkSecurityGroup1",
      "location": "[resourceGroup().location]",
      "properties": {
        "securityRules": "[reference('firstResource').outputs.result.value]"
      }
    }
  ],
  "outputs": {
      "instance":{
          "type": "array",
          "value": "[reference('firstResource').outputs.result.value]"
      }

  }
```

## <a name="next-steps"></a>后续步骤

* 此技术可在[模板构建块项目](https://github.com/mspnp/template-building-blocks)和 [Azure 参考体系结构](/azure/architecture/reference-architectures/)中实现。 可以使用这些来创建自己的体系结构或部署一个参考体系结构。

<!-- links -->
[objects-as-parameters]: ./objects-as-parameters.md
[resource-manager-linked-template]: /azure/azure-resource-manager/resource-group-linked-templates
[resource-manager-variables]: /azure/azure-resource-manager/resource-group-template-functions-deployment
[nsg]: /azure/virtual-network/virtual-networks-nsg
