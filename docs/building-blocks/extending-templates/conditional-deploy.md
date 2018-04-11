---
title: 在 Azure 资源管理器模板中有条件地部署资源
description: 介绍如何扩展 Azure 资源管理器模板的功能，根据参数的值有条件地部署资源
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: e911e7dc41b4f71ebfaf13a00f8cdbb5b4e2578b
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="conditionally-deploy-a-resource-in-an-azure-resource-manager-template"></a><span data-ttu-id="47143-103">在 Azure 资源管理器模板中有条件地部署资源</span><span class="sxs-lookup"><span data-stu-id="47143-103">Conditionally deploy a resource in an Azure Resource Manager template</span></span>

<span data-ttu-id="47143-104">在一些方案中，需要根据某一条件（例如是否存在参数值）设计模板以部署资源。</span><span class="sxs-lookup"><span data-stu-id="47143-104">There are some scenarios in which you need to design your template to deploy a resource based on a condition, such as whether or not a parameter value is present.</span></span> <span data-ttu-id="47143-105">例如，模板可能部署虚拟网络并包括参数以指定对等互连的其他虚拟网络。</span><span class="sxs-lookup"><span data-stu-id="47143-105">For example, your template may deploy a virtual network and include parameters to specify other virtual networks for peering.</span></span> <span data-ttu-id="47143-106">如果未指定对等互连的参数值，则不希望资源管理器部署对等互连资源。</span><span class="sxs-lookup"><span data-stu-id="47143-106">If you've not specified any parameter values for peering, you don't want Resource Manager to deploy the peering resource.</span></span>

<span data-ttu-id="47143-107">要完成此操作，使用资源中的 [`condition` 元素][azure-resource-manager-condition]以测试参数数组的长度。</span><span class="sxs-lookup"><span data-stu-id="47143-107">To accomplish this, use the [`condition` element][azure-resource-manager-condition] in the resource to test the length of your parameter array.</span></span> <span data-ttu-id="47143-108">若长度为零，则返回 `false` 以阻止部署，但是所有大于零的值将返回 `true` 以允许部署。</span><span class="sxs-lookup"><span data-stu-id="47143-108">If the length is zero, return `false` to prevent deployment, but for all values greater than zero return `true` to allow deployment.</span></span>

## <a name="example-template"></a><span data-ttu-id="47143-109">示例模板</span><span class="sxs-lookup"><span data-stu-id="47143-109">Example template</span></span>

<span data-ttu-id="47143-110">让我们看一下演示此操作的示例模板。</span><span class="sxs-lookup"><span data-stu-id="47143-110">Let's look at an example template that demonstrates this.</span></span> <span data-ttu-id="47143-111">我们的模板使用 [`condition` 元素][azure-resource-manager-condition] 以控制 `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` 资源的部署。</span><span class="sxs-lookup"><span data-stu-id="47143-111">Our template uses the [`condition` element][azure-resource-manager-condition] to control deployment of the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource.</span></span> <span data-ttu-id="47143-112">此资源创建同一区域中的两个 Azure 虚拟网络之间的对等互连。</span><span class="sxs-lookup"><span data-stu-id="47143-112">This resource creates a peering between two Azure Virtual Networks in the same region.</span></span>

<span data-ttu-id="47143-113">让我们看看模板的每个部分。</span><span class="sxs-lookup"><span data-stu-id="47143-113">Let's take a look at each section of the template.</span></span>

<span data-ttu-id="47143-114">`parameters` 元素定义名为 `virtualNetworkPeerings` 的单个参数：</span><span class="sxs-lookup"><span data-stu-id="47143-114">The `parameters` element defines a single parameter named `virtualNetworkPeerings`:</span></span> 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "virtualNetworkPeerings": {
      "type": "array",
      "defaultValue": []
    }
  },
```
<span data-ttu-id="47143-115">`virtualNetworkPeerings` 参数为 `array` 且包含以下方案：</span><span class="sxs-lookup"><span data-stu-id="47143-115">Our `virtualNetworkPeerings` parameter is an `array` and has the following schema:</span></span>

```json
"virtualNetworkPeerings": [
    {
        "remoteVirtualNetwork": {
            "name": "my-other-virtual-network"
        },
        "allowForwardedTraffic": true,
        "allowGatewayTransit": true,
        "useRemoteGateways": false
    }
]
```

<span data-ttu-id="47143-116">参数中的属性指定[与对等互连虚拟网络相关的设置][vnet-peering-resource-schema]。</span><span class="sxs-lookup"><span data-stu-id="47143-116">The properties in our parameter specify the [settings related to peering virtual networks][vnet-peering-resource-schema].</span></span> <span data-ttu-id="47143-117">指定 `resources` 部分中的 `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` 资源时，将为这些属性提供值：</span><span class="sxs-lookup"><span data-stu-id="47143-117">We'll provide the values for these properties when we specify the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource in the `resources` section:</span></span>

```json
"resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2017-05-10",
      "name": "[concat('vnp-', copyIndex())]",
      "condition": "[greater(length(parameters('virtualNetworkPeerings')), 0)]",
      "dependsOn": [
        "virtualNetworks"
      ],
      "copy": {
          "name": "iterator",
          "count": "[length(variables('peerings'))]",
          "mode": "serial"
      },
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
          },
          "variables": {
          },
          "resources": [
            {
              "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings",
              "apiVersion": "2016-06-01",
              "location": "[resourceGroup().location]",
              "name": "[variables('peerings')[copyIndex()].name]",
              "properties": "[variables('peerings')[copyIndex()].properties]"
            }
          ],
          "outputs": {
          }
        }
      }
    }
]
```
<span data-ttu-id="47143-118">模板的此部分需要注意几点。</span><span class="sxs-lookup"><span data-stu-id="47143-118">There are a couple of things going on in this part of our template.</span></span> <span data-ttu-id="47143-119">首先，正在部署的实际资源是包括部署 `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` 的它自己模板的类型 `Microsoft.Resources/deployments` 的内联模板。</span><span class="sxs-lookup"><span data-stu-id="47143-119">First, the actual resource being deployed is an inline template of type `Microsoft.Resources/deployments` that includes its own template that actually deploys the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.</span></span>

<span data-ttu-id="47143-120">内联模板的 `name` 是唯一的，方法是将 `copyIndex()` 的当前迭代与前缀 `vnp-` 连接。</span><span class="sxs-lookup"><span data-stu-id="47143-120">Our `name` for the inline template is made unique by concatenating the current iteration of the `copyIndex()` with the prefix `vnp-`.</span></span> 

<span data-ttu-id="47143-121">`condition` 元素指定，当 `greater()` 函数计算结果为 `true` 时，应处理资源。</span><span class="sxs-lookup"><span data-stu-id="47143-121">The `condition` element specifies that our resource should be processed when the `greater()` function evaluates to `true`.</span></span> <span data-ttu-id="47143-122">我们要测试 `virtualNetworkPeerings` 参数数组是否 `greater()` 零。</span><span class="sxs-lookup"><span data-stu-id="47143-122">Here, we're testing if the `virtualNetworkPeerings` parameter array is `greater()` than zero.</span></span> <span data-ttu-id="47143-123">如果大于零，计算结果为 `true` 且满足 `condition`。</span><span class="sxs-lookup"><span data-stu-id="47143-123">If it is, it evaluates to `true` and the `condition` is satisfied.</span></span> <span data-ttu-id="47143-124">否则为 `false`。</span><span class="sxs-lookup"><span data-stu-id="47143-124">Otherwise, it's `false`.</span></span>

<span data-ttu-id="47143-125">接下来，指定 `copy` 循环。</span><span class="sxs-lookup"><span data-stu-id="47143-125">Next, we specify our `copy` loop.</span></span> <span data-ttu-id="47143-126">它是 `serial` 循环，这意味着此循环按顺序执行，上一个资源部署完成前，下一个资源处于等待状态。</span><span class="sxs-lookup"><span data-stu-id="47143-126">It's a `serial` loop that means the loop is done in sequence, with each resource waiting until the last resource has been deployed.</span></span> <span data-ttu-id="47143-127">`count` 属性指定循环的循环访问次数。</span><span class="sxs-lookup"><span data-stu-id="47143-127">The `count` property specifies the number of times the loop iterates.</span></span> <span data-ttu-id="47143-128">我们通常将其设置为 `virtualNetworkPeerings` 数组的长度，因为它包含指定所需部署的资源的参数对象。</span><span class="sxs-lookup"><span data-stu-id="47143-128">Here, normally we'd set it to the length of the `virtualNetworkPeerings` array because it contains the parameter objects specifying the resource we want to deploy.</span></span> <span data-ttu-id="47143-129">但是，如果我们进行此操作，在数组为空的情况下验证将失败，因为资源管理器会通知我们正尝试访问的属性不存在。</span><span class="sxs-lookup"><span data-stu-id="47143-129">However, if we do that, validation will fail if the array is empty because Resource Manager notices that we are attempting to access properties that do not exist.</span></span> <span data-ttu-id="47143-130">但是我们可以解决这一问题。</span><span class="sxs-lookup"><span data-stu-id="47143-130">We can work around this, however.</span></span> <span data-ttu-id="47143-131">让我们看看需要的变量：</span><span class="sxs-lookup"><span data-stu-id="47143-131">Let's take a look at the variables we'll need:</span></span>

```json
  "variables": {
    "workaround": {
       "true": "[parameters('virtualNetworkPeerings')]",
       "false": [{
           "name": "workaround",
           "properties": {}
       }]
     },
     "peerings": "[variables('workaround')[string(greater(length(parameters('virtualNetworkPeerings')), 0))]]"
  },
```

<span data-ttu-id="47143-132">`workaround` 变量包括两个属性，一个名为 `true`，一个名为 `false`。</span><span class="sxs-lookup"><span data-stu-id="47143-132">Our `workaround` variable includes two properties, one named `true` and one named `false`.</span></span> <span data-ttu-id="47143-133">`true` 属性计算结果为 `virtualNetworkPeerings` 参数数组的值。</span><span class="sxs-lookup"><span data-stu-id="47143-133">The `true` property evaluates to the value of the `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="47143-134">`false` 属性计算结果为包括资源管理器希望看到的已命名属性的空对象&mdash;请注意，`false` 实际上是一个数组，它和将满足验证的 `virtualNetworkPeerings` 参数一样。</span><span class="sxs-lookup"><span data-stu-id="47143-134">The `false` property evaluates to an empty object including the named properties that Resource Manager expects to see&mdash;note that `false` is actually an array, just as our `virtualNetworkPeerings` parameter is, which will satisfy validation.</span></span> 

<span data-ttu-id="47143-135">`peerings` 变量使用 `workaround` 变量，方法是再次测试 `virtualNetworkPeerings` 参数数组的长度是否大于零。</span><span class="sxs-lookup"><span data-stu-id="47143-135">Our `peerings` variable uses our `workaround` variable by once again testing if the length of the `virtualNetworkPeerings` parameter array is greater than zero.</span></span> <span data-ttu-id="47143-136">如果大于零，`string` 计算结果为 `true`，`workaround` 变量计算结果为 `virtualNetworkPeerings` 参数数组。</span><span class="sxs-lookup"><span data-stu-id="47143-136">If it is, the `string` evaluates to `true` and the `workaround` variable evalutes to the `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="47143-137">否则，计算结果为 `false`，`workaround` 变量计算结果为数组的第一个元素中的空对象。</span><span class="sxs-lookup"><span data-stu-id="47143-137">Otherwise, it evaluates to `false` and the `workaround` variable evaluates to our empty object in the first element of the array.</span></span>

<span data-ttu-id="47143-138">解决验证问题之后，可以指定嵌套模板中的 `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` 资源的部署，从 `virtualNetworkPeerings` 参数数组传递 `name` 和 `properties`。</span><span class="sxs-lookup"><span data-stu-id="47143-138">Now that we've worked around the validation issue, we can simply specify the deployment of the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource in the nested template, passing the `name` and `properties` from our `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="47143-139">可以在嵌套在资源的 `properties` 元素的 `template` 元素中看到。</span><span class="sxs-lookup"><span data-stu-id="47143-139">You can see this in the `template` element nested in the `properties` element of our resource.</span></span>

## <a name="next-steps"></a><span data-ttu-id="47143-140">后续步骤</span><span class="sxs-lookup"><span data-stu-id="47143-140">Next steps</span></span>

* <span data-ttu-id="47143-141">此技术可在[模板构建块项目](https://github.com/mspnp/template-building-blocks)和 [Azure 参考体系结构](/azure/architecture/reference-architectures/)中实现。</span><span class="sxs-lookup"><span data-stu-id="47143-141">This technique is implemented in the [template building blocks project](https://github.com/mspnp/template-building-blocks) and the [Azure reference architectures](/azure/architecture/reference-architectures/).</span></span> <span data-ttu-id="47143-142">可以使用这些来创建自己的体系结构或部署一个参考体系结构。</span><span class="sxs-lookup"><span data-stu-id="47143-142">You can use these to create your own architecture or deploy one of our reference architectures.</span></span>

<!-- links -->
[azure-resource-manager-condition]: /azure/azure-resource-manager/resource-group-authoring-templates#resources
[azure-resource-manager-variable]: /azure/azure-resource-manager/resource-group-authoring-templates#variables
[vnet-peering-resource-schema]: /azure/templates/microsoft.network/virtualnetworks/virtualnetworkpeerings