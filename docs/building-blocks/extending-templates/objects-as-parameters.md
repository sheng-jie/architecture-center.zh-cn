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
---
# <a name="use-an-object-as-a-parameter-in-an-azure-resource-manager-template"></a><span data-ttu-id="78d79-103">将对象用作 Azure 资源管理器模板中的参数</span><span class="sxs-lookup"><span data-stu-id="78d79-103">Use an object as a parameter in an Azure Resource Manager template</span></span>

<span data-ttu-id="78d79-104">[创建 Azure 资源管理器模板][azure-resource-manager-create-template]时，可在模板中直接指定资源属性值，或在部署过程中定义参数并提供值。</span><span class="sxs-lookup"><span data-stu-id="78d79-104">When you [author Azure Resource Manager templates][azure-resource-manager-create-template], you can either specify resource property values directly in the template or define a parameter and provide values during deployment.</span></span> <span data-ttu-id="78d79-105">可以将每个属性值的参数用于小型部署，但每个部署限 255 个参数。</span><span class="sxs-lookup"><span data-stu-id="78d79-105">It's fine to use a parameter for each property value for small deployments, but there is a limit of 255 parameters per deployment.</span></span> <span data-ttu-id="78d79-106">在遇到更大、更复杂的部署时，可能会用完参数。</span><span class="sxs-lookup"><span data-stu-id="78d79-106">Once you get to larger and more complex deployments you may run out of parameters.</span></span>

<span data-ttu-id="78d79-107">解决此问题的方法之一是将对象用作参数，而不是值。</span><span class="sxs-lookup"><span data-stu-id="78d79-107">One way to solve this problem is to use an object as a parameter instead of a value.</span></span> <span data-ttu-id="78d79-108">为此，请在模板中定义参数，并在部署过程中指定 JSON 对象，而不是单个值。</span><span class="sxs-lookup"><span data-stu-id="78d79-108">To do this, define the parameter in your template and specify a JSON object instead of a single value during deployment.</span></span> <span data-ttu-id="78d79-109">然后，在模板中使用 [`parameter()` 函数][azure-resource-manager-functions]和点运算符引用参数的子属性。</span><span class="sxs-lookup"><span data-stu-id="78d79-109">Then, reference the subproperties of the parameter using the [`parameter()` function][azure-resource-manager-functions] and dot operator in your template.</span></span>

<span data-ttu-id="78d79-110">我们看看部署虚拟网络资源的示例。</span><span class="sxs-lookup"><span data-stu-id="78d79-110">Let's take a look at an example that deploys a virtual network resource.</span></span> <span data-ttu-id="78d79-111">首先，在模板中指定一个 `VNetSettings` 参数并将 `type` 设置为 `object`：</span><span class="sxs-lookup"><span data-stu-id="78d79-111">First, let's specify a `VNetSettings` parameter in our template and set the `type` to `object`:</span></span>

```json
...
"parameters": {
    "VNetSettings":{"type":"object"}
},
```
<span data-ttu-id="78d79-112">接下来，为 `VNetSettings` 对象提供值：</span><span class="sxs-lookup"><span data-stu-id="78d79-112">Next, let's provide values for the `VNetSettings` object:</span></span>

> [!NOTE]
> <span data-ttu-id="78d79-113">若要了解如何在部署过程中提供参数值，请参阅[了解 Azure 资源管理器模板的结构和语法][azure-resource-manager-authoring-templates]的“参数”部分。</span><span class="sxs-lookup"><span data-stu-id="78d79-113">To learn how to provide parameter values during deploment, see the **parameters** section of [understand the structure and syntax of Azure Resource Manager templates][azure-resource-manager-authoring-templates].</span></span> 

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

<span data-ttu-id="78d79-114">如你所见，单个参数实际上指定了三个子属性：`name`、`addressPrefixes` 和 `subnets`。</span><span class="sxs-lookup"><span data-stu-id="78d79-114">As you can see, our single parameter actually specifies three subproperties: `name`, `addressPrefixes`, and `subnets`.</span></span> <span data-ttu-id="78d79-115">每个子属性指定一个值或其他子属性。</span><span class="sxs-lookup"><span data-stu-id="78d79-115">Each of these subproperties either specifies a value or other subproperties.</span></span> <span data-ttu-id="78d79-116">因此，单个参数指定了部署虚拟网络所需的所有值。</span><span class="sxs-lookup"><span data-stu-id="78d79-116">The result is that our single parameter specifies all the values necessary to deploy our virtual network.</span></span>

<span data-ttu-id="78d79-117">我们看一下模板的其余部分，了解如何使用 `VNetSettings` 对象：</span><span class="sxs-lookup"><span data-stu-id="78d79-117">Now let's have a look at the rest of our template to see how the `VNetSettings` object is used:</span></span>

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
<span data-ttu-id="78d79-118">`VNetSettings` 对象的值被应用到虚拟网络资源所需要的属性，该资源使用具有 `[]` 数组索引器和点运算符的 `parameters()` 函数。</span><span class="sxs-lookup"><span data-stu-id="78d79-118">The values of our `VNetSettings` object are applied to the properties required by our virtual network resource using the `parameters()` function with both the `[]` array indexer and the dot operator.</span></span> <span data-ttu-id="78d79-119">如果只是想静态地将参数对象的值应用到资源，那么此方法适用。</span><span class="sxs-lookup"><span data-stu-id="78d79-119">This approach works if you just want to statically apply the values of the parameter object to the resource.</span></span> <span data-ttu-id="78d79-120">但是，如果希望在部署过程中动态分配属性值数组，则可使用[复制循环][azure-resource-manager-create-multiple-instances]。</span><span class="sxs-lookup"><span data-stu-id="78d79-120">However, if you want to dynamically assign an array of property values during deployment you can use a [copy loop][azure-resource-manager-create-multiple-instances].</span></span> <span data-ttu-id="78d79-121">为了使用复制循环，可提供资源属性值的 JSON 数组，复制循环再将值动态地应用到资源的属性中。</span><span class="sxs-lookup"><span data-stu-id="78d79-121">To use a copy loop, you provide a JSON array of resource property values and the copy loop dynamically applies the values to the resource's properties.</span></span> 

<span data-ttu-id="78d79-122">如果使用动态方法，需注意一个问题。</span><span class="sxs-lookup"><span data-stu-id="78d79-122">There is one issue to be aware of if you use the dynamic approach.</span></span> <span data-ttu-id="78d79-123">为演示此问题，让我们看看属性值的典型数组。</span><span class="sxs-lookup"><span data-stu-id="78d79-123">To demonstrate the issue, let's take a look at a typical array of property values.</span></span> <span data-ttu-id="78d79-124">在此示例中，属性的值存储在一个变量中。</span><span class="sxs-lookup"><span data-stu-id="78d79-124">In this example the values for our properties are stored in a variable.</span></span> <span data-ttu-id="78d79-125">请注意，此处有两个数组&mdash;一个名为 `firstProperty`，另一个名为 `secondProperty`。</span><span class="sxs-lookup"><span data-stu-id="78d79-125">Notice we have two arrays here&mdash;one named `firstProperty` and one named `secondProperty`.</span></span> 

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

<span data-ttu-id="78d79-126">现在我们来看看如何使用复制循环访问变量中的属性。</span><span class="sxs-lookup"><span data-stu-id="78d79-126">Now let's take a look at the way we access the properties in the variable using a copy loop.</span></span>

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

<span data-ttu-id="78d79-127">`copyIndex()` 函数返回复制循环的当前迭代，我们将它同时用作两个数组中的索引。</span><span class="sxs-lookup"><span data-stu-id="78d79-127">The `copyIndex()` function returns the current iteration of the copy loop, and we use that as an index into each of the two arrays simultaneously.</span></span>

<span data-ttu-id="78d79-128">当这两个数组长度相同时，这种方法没有任何问题。</span><span class="sxs-lookup"><span data-stu-id="78d79-128">This works fine when the two arrays are the same length.</span></span> <span data-ttu-id="78d79-129">如果出错并且两个数组的长度不同，就会出现问题&mdash;在这种情况下，模板将在部署过程中验证失败。</span><span class="sxs-lookup"><span data-stu-id="78d79-129">The issue arises if you've made a mistake and the two arrays are different lengths&mdash;in this case your template will fail validation during deployment.</span></span> <span data-ttu-id="78d79-130">通过将所有属性包含在单个对象中可以避免此问题，因为这样更容易了解什么时候丢失了一个值。</span><span class="sxs-lookup"><span data-stu-id="78d79-130">You can avoid this issue by including all your properties in a single object, because it is much easier to see when a value is missing.</span></span> <span data-ttu-id="78d79-131">例如，我们来看看另一个参数对象，其中 `propertyObject` 数组的每个元素都是前面 `firstProperty` 和 `secondProperty` 数组的联合。</span><span class="sxs-lookup"><span data-stu-id="78d79-131">For example, let's take a look another parameter object in which each element of the `propertyObject` array is the union of the `firstProperty` and `secondProperty` arrays from earlier.</span></span>

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

<span data-ttu-id="78d79-132">注意到数组中的第三个元素吗？</span><span class="sxs-lookup"><span data-stu-id="78d79-132">Notice the third element in the array?</span></span> <span data-ttu-id="78d79-133">它缺少 `number` 属性，但是当以这种方式创建参数值时，更容易注意到该属性丢失。</span><span class="sxs-lookup"><span data-stu-id="78d79-133">It's missing the `number` property, but it's much easier to notice that you've missed it when you're authoring the parameter values this way.</span></span>

## <a name="using-a-property-object-in-a-copy-loop"></a><span data-ttu-id="78d79-134">在复制循环中使用属性对象</span><span class="sxs-lookup"><span data-stu-id="78d79-134">Using a property object in a copy loop</span></span>

<span data-ttu-id="78d79-135">此方法在与[串行复制循环][azure-resource-manager-create-multiple] 结合使用时更为有用，特别是在部署子资源时。</span><span class="sxs-lookup"><span data-stu-id="78d79-135">This approach becomes even more useful when combined with the [serial copy loop][azure-resource-manager-create-multiple], particularly for deploying child resources.</span></span> 

<span data-ttu-id="78d79-136">为了说明这一点，我们看一下使用两个安全规则部署[网络安全组 (NSG)][nsg] 的模板。</span><span class="sxs-lookup"><span data-stu-id="78d79-136">To demonstrate this, let's look at a template that deploys a [network security group (NSG)][nsg] with two security rules.</span></span> 

<span data-ttu-id="78d79-137">首先，我们看看参数。</span><span class="sxs-lookup"><span data-stu-id="78d79-137">First, let's take a look at our parameters.</span></span> <span data-ttu-id="78d79-138">在查看模板时会看到，我们已经定义了一个名为 `networkSecurityGroupsSettings` 的参数，其中包括一个名为 `securityRules` 的数组。</span><span class="sxs-lookup"><span data-stu-id="78d79-138">When we look at our template we'll see that we've defined one parameter named `networkSecurityGroupsSettings` that includes an array named `securityRules`.</span></span> <span data-ttu-id="78d79-139">该数组包含两个为安全规则指定多个设置的 JSON 对象。</span><span class="sxs-lookup"><span data-stu-id="78d79-139">This array contains two JSON objects that specify a number of settings for a security rule.</span></span>

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

<span data-ttu-id="78d79-140">现在我们看看模板。</span><span class="sxs-lookup"><span data-stu-id="78d79-140">Now let's take a look at our template.</span></span> <span data-ttu-id="78d79-141">名为 `NSG1` 的第一个资源部署 NSG。</span><span class="sxs-lookup"><span data-stu-id="78d79-141">Our first resource named `NSG1` deploys the NSG.</span></span> <span data-ttu-id="78d79-142">名为 `loop-0` 的第二个资源执行两个函数：首先，对 NSG 执行 `dependsOn`，使其部署只有在完成 `NSG1` 后才开始，这是顺序循环的第一次迭代。</span><span class="sxs-lookup"><span data-stu-id="78d79-142">Our second resource named `loop-0` performs two functions: first, it `dependsOn` the NSG so its deployment doesn't begin until `NSG1` is completed, and it is the first iteration of the sequential loop.</span></span> <span data-ttu-id="78d79-143">第三个资源是嵌套模板，如最后一个示例中所示，它使用某个对象作为参数值部署安全规则。</span><span class="sxs-lookup"><span data-stu-id="78d79-143">Our third resource is a nested template that deploys our security rules using an object for its parameter values as in the last example.</span></span>

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

<span data-ttu-id="78d79-144">让我们详细了解如何在 `securityRules` 子资源中指定属性值。</span><span class="sxs-lookup"><span data-stu-id="78d79-144">Let's take a closer look at how we specify our property values in the `securityRules` child resource.</span></span> <span data-ttu-id="78d79-145">所有的属性都使用 `parameter()` 函数进行引用，然后我们使用点运算符引用根据迭代的当前值建立索引的 `securityRules` 数组。</span><span class="sxs-lookup"><span data-stu-id="78d79-145">All of our properties are referenced using the `parameter()` function, and then we use the dot operator to reference our `securityRules` array, indexed by the current value of the iteration.</span></span> <span data-ttu-id="78d79-146">最后，使用另一个点运算符引用对象名称。</span><span class="sxs-lookup"><span data-stu-id="78d79-146">Finally, we use another dot operator to reference the name of the object.</span></span> 

## <a name="try-the-template"></a><span data-ttu-id="78d79-147">尝试模板</span><span class="sxs-lookup"><span data-stu-id="78d79-147">Try the template</span></span>

<span data-ttu-id="78d79-148">如果要试验此模板，请按照下列步骤进行操作：</span><span class="sxs-lookup"><span data-stu-id="78d79-148">If you would like to experiment with this template, follow these steps:</span></span> 

1.  <span data-ttu-id="78d79-149">转到 Azure 门户，选择“+”图标，搜索“模板部署”</span><span class="sxs-lookup"><span data-stu-id="78d79-149">Go to the Azure portal, select the **+** icon, and search for the **template deployment** resource type, and select it.</span></span>
2.  <span data-ttu-id="78d79-150">导航到“模板部署”页，选择“创建”按钮。</span><span class="sxs-lookup"><span data-stu-id="78d79-150">Navigate to the **template deployment** page, select the **create** button.</span></span> <span data-ttu-id="78d79-151">此按钮会打开“自定义部署”边栏选项卡。</span><span class="sxs-lookup"><span data-stu-id="78d79-151">This button opens the **custom deployment** blade.</span></span>
3.  <span data-ttu-id="78d79-152">选择“编辑模板”按钮。</span><span class="sxs-lookup"><span data-stu-id="78d79-152">Select the **edit template** button.</span></span>
4.  <span data-ttu-id="78d79-153">选择空模板。</span><span class="sxs-lookup"><span data-stu-id="78d79-153">Delete the empty template.</span></span> 
5.  <span data-ttu-id="78d79-154">将示例模板复制粘贴到右侧窗格中。</span><span class="sxs-lookup"><span data-stu-id="78d79-154">Copy and paste the sample template into the right pane.</span></span>
6.  <span data-ttu-id="78d79-155">选择“保存”按钮。</span><span class="sxs-lookup"><span data-stu-id="78d79-155">Select the **save** button.</span></span>
7.  <span data-ttu-id="78d79-156">返回到“自定义部署”窗格后，请选择“编辑参数”按钮。</span><span class="sxs-lookup"><span data-stu-id="78d79-156">When you are returned to the **custom deployment** pane, select the **edit parameters** button.</span></span>
8.  <span data-ttu-id="78d79-157">在“编辑参数”边栏选项卡中，删除现有的模板。</span><span class="sxs-lookup"><span data-stu-id="78d79-157">On the **edit parameters** blade, delete the existing template.</span></span>
9.  <span data-ttu-id="78d79-158">复制并粘贴上述示例参数模板。</span><span class="sxs-lookup"><span data-stu-id="78d79-158">Copy and paste the sample parameter template from above.</span></span>
10. <span data-ttu-id="78d79-159">选择“保存”按钮，返回到“自定义部署”边栏选项卡。</span><span class="sxs-lookup"><span data-stu-id="78d79-159">Select the **save** button, which returns you to the **custom deployment** blade.</span></span>
11. <span data-ttu-id="78d79-160">在“自定义部署”边栏选项卡中选择订阅，新建资源组或使用现有资源组，然后选择位置。</span><span class="sxs-lookup"><span data-stu-id="78d79-160">On the **custom deployment** blade, select your subscription, either create new or use existing resource group, and select a location.</span></span> <span data-ttu-id="78d79-161">查看条款和条件，并选中“我同意”复选框。</span><span class="sxs-lookup"><span data-stu-id="78d79-161">Review the terms and conditions, and select the **I agree** checkbox.</span></span>
12. <span data-ttu-id="78d79-162">选择“购买”按钮。</span><span class="sxs-lookup"><span data-stu-id="78d79-162">Select the **purchase** button.</span></span>

## <a name="next-steps"></a><span data-ttu-id="78d79-163">后续步骤</span><span class="sxs-lookup"><span data-stu-id="78d79-163">Next steps</span></span>

* <span data-ttu-id="78d79-164">可以扩展这些技术来实现[属性对象转换器和收集器](./collector.md)。</span><span class="sxs-lookup"><span data-stu-id="78d79-164">You can expand upon these techniques to implement a [property object transformer and collector](./collector.md).</span></span> <span data-ttu-id="78d79-165">转换器和收集器技术更常规，可从模板链接。</span><span class="sxs-lookup"><span data-stu-id="78d79-165">The transformer and collector techniques are more general and can be linked from your templates.</span></span>
* <span data-ttu-id="78d79-166">此技术也可在[模板构建块项目](https://github.com/mspnp/template-building-blocks)和 [Azure 参考体系结构](/azure/architecture/reference-architectures/)中实现。</span><span class="sxs-lookup"><span data-stu-id="78d79-166">This technique is also implemented in the [template building blocks project](https://github.com/mspnp/template-building-blocks) and the [Azure reference architectures](/azure/architecture/reference-architectures/).</span></span> <span data-ttu-id="78d79-167">请查看我们的模板，了解如何实现此技术。</span><span class="sxs-lookup"><span data-stu-id="78d79-167">You can review our templates to see how we've implemented this technique.</span></span>

<!-- links -->
[azure-resource-manager-authoring-templates]: /azure/azure-resource-manager/resource-group-authoring-templates
[azure-resource-manager-create-template]: /azure/azure-resource-manager/resource-manager-create-first-template
[azure-resource-manager-create-multiple-instances]: /azure/azure-resource-manager/resource-group-create-multiple
[azure-resource-manager-functions]: /azure/azure-resource-manager/resource-group-template-functions-resource
[nsg]: /azure/virtual-network/virtual-networks-nsg