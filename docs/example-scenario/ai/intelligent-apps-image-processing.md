---
title: 智能应用 - Azure 上的图像处理
description: 经验证的解决方案，用于将图像处理内置到 Azure 应用程序中。
author: david-stanford
ms.date: 07/05/2018
ms.openlocfilehash: c5bfb9a929ddddda4336e1cbc8665a0b4d3bbe2c
ms.sourcegitcommit: 5d99b195388b7cabba383c49a81390ac48f86e8a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/06/2018
ms.locfileid: "37891269"
---
# <a name="insurance-claim-image-classification-on-azure"></a>Azure 上的保险索赔图像分类

本示例方案适用于需处理图像的企业。

可能的应用包括：对时尚网站的图像分类、分析保险索赔的文本和图像，或者理解游戏屏幕截图中的遥测数据。 传统上，公司需开发机器学习模型方面的专业技术，训练模型，最后再通过自定义过程运行图像，以便从这些图像中获取数据。

使用 Azure 服务（例如计算机视觉 API 和 Azure Functions），公司不需管理各个服务器，既减少了成本，又可充分利用 Microsoft 围绕认知服务图像处理开发的专业技术。 本方案专门解决图像处理方案问题。 如果有各种不同的的 AI 需求，可以考虑全套[认知服务][cognitive-docs]。

## <a name="potential-use-cases"></a>可能的用例

以下用例可以考虑本解决方案：

* 对时尚网站上的图像分类。
* 对保险索赔的图像分类。
* 对游戏屏幕截图中的遥测数据分类。

## <a name="architecture"></a>体系结构

![智能应用体系结构 - 计算机视觉][architecture-computer-vision]

本解决方案涉及 Web 或移动应用程序的后端组件。 数据流经解决方案的情形如下所示：

1. Azure Functions 充当 API 层。 应用程序可以通过这些 API 上传图像以及检索 Cosmos DB 中的数据。

2. 通过 API 调用上传图像时，该图像会存储在 Blob 存储中。

3. 向 Blob 存储添加新文件会触发系统将 EventGrid 通知发送给某个 Azure 函数。

4. Azure Functions 会向计算机视觉 API 发送新上传文件的链接，供其分析。

5. 数据从计算机视觉 API 返回以后，Azure Functions 会在 Cosmos DB 中生成一个条目，以便持久保存分析结果和图像元数据。

### <a name="components"></a>组件

* [计算机视觉 API][computer-vision-docs] 是认知服务套件的一部分，用于检索每个图像的信息。

* [Azure Functions][functions-docs] 为 Web 应用程序提供后端 API，并为已上传的图像提供事件处理。

* [事件网格][eventgrid-docs]在新图像上传到 Blob 存储时触发一个事件。 该图像然后就会通过 Azure Functions 进行处理。

* [Blob 存储][storage-docs]存储上传到 Web 应用程序中的所有图像文件，以及 Web 应用程序使用的任何静态文件。

* [Cosmos DB][cosmos-docs] 存储每个已上传图像的元数据，包括计算机视觉 API 的处理结果。

## <a name="alternatives"></a>备选项

* [自定义视觉服务][custom-vision-docs]。 计算机视觉 API 返回一组[基于分类的类别][cv-categories]。 若需处理不是由计算机视觉 API 返回的信息，则可考虑使用自定义视觉服务，以便生成自定义图像分类器。

* [Azure 搜索][azure-search-docs]。 如果用例需要查询元数据来查找符合特定条件的图像，则可考虑使用 Azure 搜索。 [认知搜索][cognitive-search]目前为预览版，可以无缝集成此工作流。

## <a name="considerations"></a>注意事项

### <a name="scalability"></a>可伸缩性

大多数情况下，本解决方案的所有组件是托管服务，可以自动进行缩放。 一些需注意的例外：Azure Functions 存在限制，最多可以使用 200 个实例。 如果所需规模超出限制，可以考虑使用多个区域或应用计划。

Cosmos DB 不会按照预配的请求单位 (RU) 自动缩放。  有关如何估算需求的指南，请参阅文档中的[请求单位][request-units]。 若要充分利用 Cosmos DB 中的缩放功能，还应参阅[分区键][partition-key]的内容。

很多情况下，为了确保可用性、可伸缩性和分区，NoSQL 数据库会牺牲一致性（这就是所谓的 CAP 法则）。  但就键-值数据模型（在本方案中使用）来说，很少需要事务一致性，因为大多数操作就其定义来说属于原子操作。 有关如何[选择正确的数据存储](../../guide/technology-choices/data-store-overview.md)的其他指南，请访问体系结构中心。

有关如何设计可缩放解决方案的通用指南，请参阅 Azure 体系结构中心的[可伸缩性核对清单][scalability]。

### <a name="security"></a>“安全”

[托管服务标识][msi] (MSI) 用于访问帐户的其他内部资源，然后会被系统分配给你的 Azure Functions。 只允许通过这些标识访问必要的资源，这样可确保不将额外的内容公开给函数（可能还包括客户）。  

若需安全解决方案的通用设计指南，请参阅 [Azure 安全性文档][security]。

### <a name="resiliency"></a>复原

本解决方案中的所有组件都是托管的，因此均可在区域级别自动复原。 

若需可复原解决方案的通用设计指南，请参阅[设计适用于 Azure 的可复原应用程序][resiliency]。

## <a name="pricing"></a>定价

为了方便用户了解运行本解决方案的成本，我们已在成本计算器中预配置了所有服务。 若要了解自己的特定用例的定价变化情况，请按预期的流量更改相应的变量。

我们已根据流量（假定所有图像的大小均为 100kb）提供了三个示例性的成本配置文件：

* [小][pricing]：对应于每月处理的图像数 &lt; 5000 的情况。
* [中][medium-pricing]：对应于每月处理的图像数为 500,000 的情况。
* [大][large-pricing]：对应于每月处理的图像数为 5 千万的情况。

## <a name="related-resources"></a>相关资源

如需此解决方案的引导式学习路径，请参阅[在 Azure 中生成无服务器 Web 应用][serverless]。  

将此解决方案放到生产环境中之前，请参阅 Azure Functions [最佳做法][functions-best-practices]。

<!-- links -->
[pricing]: https://azure.com/e/f9b59d238b43423683db73f4a31dc380
[medium-pricing]: https://azure.com/e/7c7fc474db344b87aae93bc29ae27108
[large-pricing]: https://azure.com/e/cbadbca30f8640d6a061f8457a74ba7d
[functions-docs]: /azure/azure-functions/
[computer-vision-docs]: /azure/cognitive-services/computer-vision/home
[storage-docs]: /azure/storage/
[azure-search-docs]: /azure/search/
[cognitive-search]: /azure/search/cognitive-search-concept-intro
[architecture-computer-vision]: ./media/architecture-computer-vision.png
[serverless]: /azure/functions/tutorial-static-website-serverless-api-with-database
[cosmos-docs]: /azure/cosmos-db/
[eventgrid-docs]: /azure/event-grid/
[cognitive-docs]: /azure/#pivot=products&panel=ai
[custom-vision-docs]: /azure/cognitive-services/Custom-Vision-Service/home
[cv-categories]: /azure/cognitive-services/computer-vision/home#the-86-category-concept
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[functions-best-practices]: /azure/azure-functions/functions-best-practices
[msi]: /azure/app-service/app-service-managed-service-identity
[request-units]: /azure/cosmos-db/request-units
[partition-key]: /azure/cosmos-db/partition-data