---
title: 选择认知服务技术
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 055769188fbd6742b94094ee18766293812849fa
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/06/2018
---
# <a name="choosing-a-microsoft-cognitive-services-technology"></a>选择 Microsoft 认知服务技术

Microsoft 认知服务是基于云的 API，可以在人工智能 (AI) 应用程序和数据流中使用。 它们提供了就绪可在应用程序中使用的预先训练的模型，不需要你提供数据，也不需要你进行模型训练。 认知服务是由 Microsoft 的人工智能和研究团队开发的，利用了最新的深度学习算法。 可通过 HTTP REST 接口使用它们。 此外，对于许多常见的应用程序开发框架，还可以使用 SDK。

认知服务包括：

* 文本分析
* 计算机视觉
* 视频分析
* 语音识别和生成
* 自然语言理解
* 智能搜索

主要优势：

* 最大程度地降低了开发最先进人工智能服务所需的工作量。
* 可以通过 HTTP REST 接口轻松集成到应用。
* 内置了对在 Azure Data Lake Analytics 中使用认知服务的支持。

注意事项：

* 仅可通过 Web 使用。 通常需要 Internet 连接。 自定义影像服务是一个例外，可以导出其训练后的模型，用于在设备上和 IoT Edge 处进行预测。
* 虽然支持相当多的自定义，但是，可用的服务未必满足所有预测分析要求。

## <a name="what-are-your-options-when-choosing-amongst-the-cognitive-services"></a>在各个认知服务之间进行选择时有哪些选项？
在 Azure 中，有许多认知服务可用。 按服务支持的功能区域进行了分类的目录中提供了这些服务的当前列表：
- [影像](https://azure.microsoft.com/services/cognitive-services/directory/vision/)
- [语音](https://azure.microsoft.com/services/cognitive-services/directory/speech/)
- [知识](https://azure.microsoft.com/services/cognitive-services/directory/know/)
- [搜索](https://azure.microsoft.com/services/cognitive-services/directory/search/)
- [语言](https://azure.microsoft.com/services/cognitive-services/directory/lang/)

## <a name="key-selection-criteria"></a>关键选择条件

若要缩小选择范围，请先回答以下问题：

- 要处理什么类型的数据？ 根据你处理的输入数据的类型缩小选项范围。 例如，如果输入是文本，请从输入类型为文本的服务中进行选择。 

- 你是否有用来训练模型的数据？ 如果有，请考虑使用允许使用你提供的数据对其基础模型进行训练的自定义服务，以提高准确度和性能。 

## <a name="capability-matrix"></a>功能矩阵

以下各表汇总了功能上的关键差异。 

### <a name="uses-prebuilt-models"></a>使用预构建的模型

|                                                   |             输入类型              |                                                                                主要优势                                                                                |
|---------------------------------------------------|-------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                文本分析 API                 |                文本                 |                                                       评估情绪和主题以理解用户的需求。                                                        |
|                实体链接 API                 |                文本                 |                                               增强应用的数据链接的功能，为其提供命名实体识别并消除歧义。                                               |
| 语言理解智能服务 (LUIS) |                文本                 |                                                          教会应用理解用户发出的命令。                                                          |
|                 QnA Maker 服务                 |                文本                 |                                             提取常见问题解答格式的信息，并将其转化为一目了然的对话式答案。                                              |
|              语言分析 API              |                文本                 |                                                            简化复杂的语言概念，并分析文本。                                                             |
|           知识探索服务           |                文本                 |                                          通过自然语言输入实现结构化数据的交互式搜索体验。                                          |
|              Web 语言模型 API               |                文本                 |                                                         使用针对 Web 上的数据训练的预测语言模型。                                                         |
|              学术知识 API               |                文本                 |                                        利用 Microsoft Academic Graph 中由必应填充的丰富的学术内容。                                         |
|               必应自动建议 API                |                文本                 |                                                        为你的应用提供用于搜索的智能自动建议选项。                                                        |
|               必应拼写检查 API                |                文本                 |                                                             检测并更正应用中的拼写错误。                                                             |
|                文本翻译 API                |                文本                 |                                                                           机器翻译。                                                                            |
|                建议 API                |                文本                 |                                                             预测和推荐客户所需的商品。                                                              |
|              必应实体搜索 API               |       文本（Web 搜索查询）       |                                                           从 Web 识别并补充实体信息。                                                           |
|               必应图像搜索 API               |       文本（Web 搜索查询）       |                                                                            搜索图像。                                                                             |
|               必应新闻搜索 API                |       文本（Web 搜索查询）       |                                                                             搜索新闻。                                                                              |
|               必应视频搜索 API               |       文本（Web 搜索查询）       |                                                                            搜索视频。                                                                             |
|                必应 Web 搜索 API                |       文本（Web 搜索查询）       |                                                        从数十亿 Web 文档中获取增强的搜索详细信息。                                                        |
|                  必应语音 API                  |           文本或语音            |                                                                  将语音转换为文本并重新转换回来。                                                                   |
|              说话人识别 API              |               语音                |                                                       使用语音来识别各个说话人并对其进行身份验证。                                                        |
|               语音翻译 API               |               语音                |                                                                   执行实时语音翻译。                                                                   |
|                计算机视觉 API                |    图像（或视频中的帧）    | 从图像中提取可操作的信息，自动创建照片说明，派生标记，识别名人，提取文本，以及创建准确的缩略图。 |
|                 内容审查器                 |        文本、图像或视频        |                                                               自动化图像、文本和视频审查。                                                                |
|                    情感 API                    | 图像（包含人物对象的照片） |                                                              识别人物对象的各种情绪。                                                               |
|                     人脸 API                      | 图像（包含人物对象的照片） |                                                       检测、识别、分析、组织和标记照片中的人脸。                                                       |
|                   视频索引器                   |                视频                |                        视频见解，诸如分析情绪，转录语音，翻译语音，识别人脸和情绪，以及提取关键字。                         |

### <a name="trained-with-custom-data-you-provide"></a>使用你提供的自定义数据训练的模型

| | 输入类型 | 主要优势 |
| --- | --- | --- |
| 自定义影像服务 | 图像（或视频中的帧） | 自定义自己的计算机视觉模型。 |
| 自定义语音服务 | 语音 | 克服语音识别障碍，如说话风格、背景噪音和词汇。 | 
| 自定义决策服务 | Web 内容（例如，RSS 源） | 使用机器学习自动为主页选择合适的内容 |
| 必应自定义搜索 API | 文本（Web 搜索查询） | 商业级搜索工具。 |

