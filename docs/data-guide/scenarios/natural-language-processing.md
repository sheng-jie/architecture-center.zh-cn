---
title: "自然语言处理"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: c03e2d017f9b4eb955a0e3494b5bc6c2603d1058
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/14/2018
---
# <a name="natural-language-processing"></a>自然语言处理

自然语言处理 (NLP) 用于诸如情绪分析、主题检测、语言检测、关键短语提取和文档归类之类的任务。

![](./images/nlp-pipeline.png)

## <a name="when-to-use-this-solution"></a>何时使用此解决方案

NLP 可以用来对文档进行分类，例如将文档标记为敏感文档或垃圾文档。 NLP 的输出可用于后续处理或搜索。 NLP 的另一个用途是通过识别文档中存在的实体对文本进行总结。 这些实体还可以用来为文档标记关键字，从而启用基于内容的搜索和检索。 可以将实体以及说明每个文档中存在的重要主题的总结组合到主题中。 可以使用检测到的主题来对文档进行分类以便导航，或者针对选定的主题枚举相关文档。 NLP 的另一个用途是对文本进行情绪评分，以评估文档的正面或负面语气。 这些方法使用许多自然语言处理技术，例如： 

- **分词器**。 将文本拆分为字或短语。
- **词干分解和词元化**。 对字进行规范化，以便将不同的形式映射到具有相同含义的规范字。 例如，“running”和“ran”映射到“run”。 
- **实体提取**。 识别文本中的主题。
- **语音部件检测**。 将文本标识为谓词、名词、分词、谓词短语，等等。
- **句子边界检测**。 检测文本段落中的完整句子。

当使用 NLP 从自由格式文本提取信息和见解时，起点通常是存储在对象存储（例如 Azure 存储或 Azure Data Lake Store）中的原始文档。 

## <a name="challenges"></a>挑战

- 处理自由格式文本文档的集合在计算时通常需要消耗大量资源，而且非常耗时。
- 没有一种标准化的文档格式，可能非常难以使用自由格式文本处理来从文档中提取特定事实以稳定地获得准确结果。 例如，想一想发票的文本表示形式&mdash;难以构建一个流程来准确地从任意数量的供应商那里提取发票编号和发票日期。

## <a name="architecture"></a>体系结构

在 NLP 解决方案中，可以对包含文本段落的文档执行自由格式文本处理。 总体体系结构可以是[批处理](./batch-processing.md)或[实时流处理](./real-time-processing.md)体系结构。

实际处理因所需的输出而异，但是就管道而言，可以采用批处理方式或实时方式来应用 NLP。 例如，可以对文本块使用情绪分析来生成情绪分数。 可以通过对存储中的数据运行批处理来执行此操作，或者使用流经消息传递服务的较小数据块实时执行此操作。

## <a name="technology-choices"></a>技术选择

- [自然语言处理](../technology-choices/natural-language-processing.md)