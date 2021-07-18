---
title: 选择自然语言处理技术
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: dacf7bf9cf3e9efed212f34da93c1470954965cf
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/14/2018
ms.locfileid: "29288849"
---
# <a name="choosing-a-natural-language-processing-technology-in-azure"></a>在 Azure 中选择自然语言处理技术

自由格式文本处理针对包含文本段落的文档执行，通常是为了支持搜索，但也用于执行其他自然语言处理 (NLP) 任务，如情绪分析、主题检测、语言检测、关键短语提取和文档分类。 本文重点介绍支持 NLP 任务的技术选项。

## <a name="what-are-your-options-when-choosing-an-nlp-service"></a>选择 NLP 服务时有哪些选项？

在 Azure 中，以下服务提供自然语言处理 (NLP) 功能：

- [Azure HDInsight with Spark and Spark MLlib](/azure/hdinsight/spark/apache-spark-overview)
- [Microsoft 认知服务](/azure/#pivot=products&panel=cognitive)

## <a name="key-selection-criteria"></a>关键选择条件

若要缩小选择范围，请先回答以下问题：

- 是否要使用预构建的模型？ 如果是，请考虑使用 Microsoft 认知服务提供的 API。

- 是否需要针对大型文本数据语料库训练自定义模型？ 如果是，请考虑使用 Azure HDInsight with Spark MLlib and Spark NLP。

- 是否需要低级别 NLP 功能，如词汇切分、词干分解、词形还原和词频/逆文档频率 (TF/IDF)？ 如果是，请考虑使用 Azure HDInsight with Spark MLlib and Spark NLP。

- 是否需要简单的高级别 NLP 功能，如实体和意图识别、主题检测、拼写检查或情绪分析？ 如果是，请考虑使用 Microsoft 认知服务提供的 API。

## <a name="capability-matrix"></a>功能矩阵

下表汇总了功能上的关键差异。  

### <a name="general-capabilities"></a>常规功能

| | Azure HDInsight | Microsoft 认知服务 |
| --- | --- | --- |
| 提供预先训练的模型作为服务 | 否 | 是 |
| REST API | 是 | 是 |
| 可编程性 | Python、Scala、Java | C#、Java、Node.js、Python、PHP、Ruby |
| 支持大数据集和大型文档的处理 | 是 | 否 |

### <a name="low-level-natural-language-processing-capabilities"></a>低级别的自然语言处理功能

| | Azure HDInsight | Microsoft 认知服务 |  
| --- | --- | --- | 
| 分词器 | 是 (Spark NLP) | 是（语言分析 API） |
| 词干分析器 | 是 (Spark NLP) | 否 |
| 词形还原工具 | 是 (Spark NLP) | 否 |
| 词性标记 | 是 (Spark NLP) | 是（语言分析 API） |
| 词频/逆向文档频率 (TF/IDF) | 是 (Spark MLlib) | 否 |
| 字符串相似性&mdash;编辑距离计算 | 是 (Spark MLlib) | 否 |
| N 元语法计算 | 是 (Spark MLlib) | 否 |
| 停止词删除 | 是 (Spark MLlib) | 否 |

### <a name="high-level-natural-language-processing-capabilities"></a>高级别的自然语言处理功能

| | Azure HDInsight | Microsoft 认知服务 |
| --- | --- | --- | 
| 实体/意图识别和提取 | 否 | 是（语言理解智能服务 (LUIS) API） |    
| 主题检测 | 是 (Spark NLP) | 是（文本分析 API） |
| 拼写检查 | 是 (Spark NLP) | 是（必应拼写检查 API） |
| 情绪分析 | 是 (Spark NLP) | 是（文本分析 API） |
| 语言检测 | 否 | 是（文本分析 API） |
| 支持除英语以外的多种语言 | 否 | 是（因 API 而异） |

## <a name="see-also"></a>另请参阅

[自然语言处理](../scenarios/natural-language-processing.md)