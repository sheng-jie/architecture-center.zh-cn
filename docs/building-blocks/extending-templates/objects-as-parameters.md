---
title: 将对象用作 Azure 资源管理器模板中的参数
description: 介绍如何扩展 Azure 资源管理器模板的功能，以便将对象用作参数
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 76f8b9d459f4ab3147b52762b7c26552ec92c7a3
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/06/2018
ms.locfileid: "30847169"
---
# <a name="use-an-object-as-a-parameter-in-an-azure-resource-manager-template"></a>将对象用作 Azure 资源管理器模板中的参数

[创建 Azure 资源管理器模板][azure-resource-manager-create-template]时，可在模板中直接指定资源属性值，或在部署过程中定义参数并提供值。 可以将每个属性值的参数用于小型部署，但每个部署限 255 个参数。 在遇到更大、更复杂的部署时，可能会用完参数。

解决此问题的方法之一是将对象用作参数，而不是值。 为此，请在模板中定义参数，并在部署过程中指定 JSON 对象，而不是单个值。 然后，在模板中使用 [`parameter()` 函数][azure-resource-manager-functions]和点运算符引用参数的子属性。

我们看看部署虚拟网络资源的示例。 首先，在模板中指定一个 `VNetSettings` 参数并将 `type` 设置为 `object`：

```json
...
"parameters": {
    "VNetSettings":{"type":"object"}
},
```
接下来，为 `VNetSettings` 对象提供值：

> [!NOTE]
> 若要了解如何在部署过程中提供参数值，请参阅[了解 Azure 资源管理器模板的结构和语法][azure-resource-manager-authoring-templates]的“参数”部分。 

```json
"parameters":{
    "VNetSettings":{
        "value":{
            "name":"VNet1",
            "addressPrefixes": [
                {
                    "name": "firstPrefix",
                    "addressPrefix": "10.0.0.0/22"
                }
            ],
            "subnets":[
                {
                    "name": "firstSubnet",
                    "addressPrefix": "10.0.0.0/24"
                },
                {
                    "name":"secondSubnet",
                    "addressPrefix":"10.0.1.0/24"
                }
            ]
        }
    }
}
```

如你所见，单个参数实际上指定了三个子属性：`name`、`addressPrefixes` 和 `subnets`。 每个子属性指定一个值或其他子属性。 因此，单个参数指定了部署虚拟网络所需的所有值。

我们看一下模板的其余部分，了解如何使用 `VNetSettings` 对象：

```json
...
"resources": [
    {
        "apiVersion": "2015-06-15",
        "type": "Microsoft.Network/virtualNetworks",
        "name": "[parameters('VNetSettings').name]",
        "location":"[resourceGroup().location]",
        "properties": {
          "addressSpace":{
              "addressPrefixes": [
                "[parameters('VNetSettings').addressPrefixes[0].addressPrefix]"
              ]
          },
          "subnets":[
              {
                  "name":"[parameters('VNetSettings').subnets[0].name]",
                  "properties": {
                      "addressPrefix": "[parameters('VNetSettings').subnets[0].addressPrefix]"
                  }
              },
              {
                  "name":"[parameters('VNetSettings').subnets[1].name]",
                  "properties": {
                      "addressPrefix": "[parameters('VNetSettings').subnets[1].addressPrefix]"
                  }
              }
          ]
        }
    }
  ]
```
`VNetSettings` 对象的值被应用到虚拟网络资源所需要的属性，该资源使用具有 `[]` 数组索引器和点运算符的 `parameters()` 函数。 如果只是想静态地将参数对象的值应用到资源，那么此方法适用。 但是，如果希望在部署过程中动态分配属性值数组，则可使用[复制循环][azure-resource-manager-create-multiple-instances]。 为了使用复制循环，可提供资源属性值的 JSON 数组，复制循环再将值动态地应用到资源的属性中。 

如果使用动态方法，需注意一个问题。 为演示此问题，让我们看看属性值的典型数组。 在此示例中，属性的值存储在一个变量中。 请注意，此处有两个数组&mdash;一个名为 `firstProperty`，另一个名为 `secondProperty`。 

```json
"variables": {
    "firstProperty": [
        {
            "name": "A",
            "type": "typeA"
        },
        {
            "name": "B",
            "type": "typeB"
        },
        {
            "name": "C",
            "type": "typeC"
        }
    ],
    "secondProperty": [
        "one","two", "three"
    ]
}
```

现在我们来看看如何使用复制循环访问变量中的属性。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    ...
    "copy": {
        "name": "copyLoop1",
        "count": "[length(variables('firstProperty'))]"
    },
    ...
    "properties": {
        "name": { "value": "[variables('firstProperty')[copyIndex()].name]" },
        "type": { "value": "[variables('firstProperty')[copyIndex()].type]" },
        "number": { "value": "[variables('secondProperty')[copyIndex()]]" }
    }
}
```

`copyIndex()` 函数返回复制循环的当前迭代，我们将它同时用作两个数组中的索引。

当这两个数组长度相同时，这种方法没有任何问题。 如果出错并且两个数组的长度不同，就会出现问题&mdash;在这种情况下，模板将在部署过程中验证失败。 通过将所有属性包含在单个对象中可以避免此问题，因为这样更容易了解什么时候丢失了一个值。 例如，我们来看看另一个参数对象，其中 `propertyObject` 数组的每个元素都是前面 `firstProperty` 和 `secondProperty` 数组的联合。

```json
"variables": {
    "propertyObject": [
        {
            "name": "A",
            "type": "typeA",
            "number": "one"
        },
        {
            "name": "B",
            "type": "typeB",
            "number": "two"
        },
        {
            "name": "C",
            "type": "typeC"
        }
    ]
}
```

注意到数组中的第三个元素吗？ 它缺少 `number` 属性，但是当以这种方式创建参数值时，更容易注意到该属性丢失。

## <a name="using-a-property-object-in-a-copy-loop"></a>在复制循环中使用属性对象

此方法在与[串行复制循环][azure-resource-manager-create-multiple] 结合使用时更为有用，特别是在部署子资源时。 

为了说明这一点，我们看一下使用两个安全规则部署[网络安全组 (NSG)][nsg] 的模板。 

首先，我们看看参数。 在查看模板时会看到，我们已经定义了一个名为 `networkSecurityGroupsSettings` 的参数，其中包括一个名为 `securityRules` 的数组。 该数组包含两个为安全规则指定多个设置的 JSON 对象。

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

现在我们看看模板。 名为 `NSG1` 的第一个资源部署 NSG。 名为 `loop-0` 的第二个资源执行两个函数：首先，对 NSG 执行 `dependsOn`，使其部署只有在完成 `NSG1` 后才开始，这是顺序循环的第一次迭代。 第三个资源是嵌套模板，如最后一个示例中所示，它使用某个对象作为参数值部署安全规则。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "networkSecurityGroupsSettings": {"type":"object"}
  },
  "variables": {},
  "resources": [
    {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/networkSecurityGroups",
      "name": "NSG1",
      "location":"[resourceGroup().location]",
      "properties": {
          "securityRules":[]
      }
    },
    {
        "apiVersion": "2015-01-01",
        "type": "Microsoft.Resources/deployments",
        "name": "loop-0",
        "dependsOn": [
            "NSG1"
        ],
        "properties": {
            "mode":"Incremental",
            "parameters":{},
            "template": {
                "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                "contentVersion": "1.0.0.0",
                "parameters": {},
                "variables": {},
                "resources": [],
                "outputs": {}
            }
        }       
    },
    {
        "apiVersion": "2015-01-01",
        "type": "Microsoft.Resources/deployments",
        "name": "[concat('loop-', copyIndex(1))]",
        "dependsOn": [
          "[concat('loop-', copyIndex())]"
        ],
        "copy": {
          "name": "iterator",
          "count": "[length(parameters('networkSecurityGroupsSettings').securityRules)]"
        },
        "properties": {
          "mode": "Incremental",
          "template": {
            "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
           "parameters": {},
            "variables": {},
            "resources": [
                {
                    "name": "[concat('NSG1/' , parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].name)]",
                    "type": "Microsoft.Network/networkSecurityGroups/securityRules",
                    "apiVersion": "2016-09-01",
                    "location":"[resourceGroup().location]",
                    "properties":{
                        "description": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].description]",
                        "priority":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].priority]",
                        "protocol":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].protocol]",
                        "sourcePortRange": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].sourcePortRange]",
                        "destinationPortRange": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].destinationPortRange]",
                        "sourceAddressPrefix": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].sourceAddressPrefix]",
                        "destinationAddressPrefix": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].destinationAddressPrefix]",
                        "access":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].access]",
                        "direction":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].direction]"
                        }
                  }
            ],
            "outputs": {}
          }
        }
    }
  ],          
  "outputs": {}
}
```

让我们详细了解如何在 `securityRules` 子资源中指定属性值。 所有的属性都使用 `parameter()` 函数进行引用，然后我们使用点运算符引用根据迭代的当前值建立索引的 `securityRules` 数组。 最后，使用另一个点运算符引用对象名称。 

## <a name="try-the-template"></a>尝试模板

如果要试验此模板，请按照下列步骤进行操作： 

1.  转到 Azure 门户，选择“+”图标，搜索“模板部署”
2.  导航到“模板部署”页，选择“创建”按钮。 此按钮会打开“自定义部署”边栏选项卡。
3.  选择“编辑模板”按钮。
4.  选择空模板。 
5.  将示例模板复制粘贴到右侧窗格中。
6.  选择“保存”按钮。
7.  返回到“自定义部署”窗格后，请选择“编辑参数”按钮。
8.  在“编辑参数”边栏选项卡中，删除现有的模板。
9.  复制并粘贴上述示例参数模板。
10. 选择“保存”按钮，返回到“自定义部署”边栏选项卡。
11. 在“自定义部署”边栏选项卡中选择订阅，新建资源组或使用现有资源组，然后选择位置。 查看条款和条件，并选中“我同意”复选框。
12. 选择“购买”按钮。

## <a name="next-steps"></a>后续步骤

* 可以扩展这些技术来实现[属性对象转换器和收集器](./collector.md)。 转换器和收集器技术更常规，可从模板链接。
* 此技术也可在[模板构建块项目](https://github.com/mspnp/template-building-blocks)和 [Azure 参考体系结构](/azure/architecture/reference-architectures/)中实现。 请查看我们的模板，了解如何实现此技术。

<!-- links -->
[azure-resource-manager-authoring-templates]: /azure/azure-resource-manager/resource-group-authoring-templates
[azure-resource-manager-create-template]: /azure/azure-resource-manager/resource-manager-create-first-template
[azure-resource-manager-create-multiple-instances]: /azure/azure-resource-manager/resource-group-create-multiple
[azure-resource-manager-functions]: /azure/azure-resource-manager/resource-group-template-functions-resource
[nsg]: /azure/virtual-network/virtual-networks-nsg