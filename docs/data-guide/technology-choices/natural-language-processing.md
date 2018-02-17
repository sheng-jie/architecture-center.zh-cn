---
title: "选择自然语言处理技术"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: dacf7bf9cf3e9efed212f34da93c1470954965cf
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-natural-language-processing-technology-in-azure"></a><span data-ttu-id="90d65-102">在 Azure 中选择自然语言处理技术</span><span class="sxs-lookup"><span data-stu-id="90d65-102">Choosing a natural language processing technology in Azure</span></span>

<span data-ttu-id="90d65-103">自由格式文本处理针对包含文本段落的文档执行，通常是为了支持搜索，但也用于执行其他自然语言处理 (NLP) 任务，如情绪分析、主题检测、语言检测、关键短语提取和文档分类。</span><span class="sxs-lookup"><span data-stu-id="90d65-103">Free-form text processing is performed against documents containing paragraphs of text, typically for the purpose of supporting search, but is also used to perform other natural language processing (NLP) tasks such as sentiment analysis, topic detection, language detection, key phrase extraction, and document categorization.</span></span> <span data-ttu-id="90d65-104">本文重点介绍支持 NLP 任务的技术选项。</span><span class="sxs-lookup"><span data-stu-id="90d65-104">This article focuses on the technology choices that act in support of the NLP tasks.</span></span>

## <a name="what-are-your-options-when-choosing-an-nlp-service"></a><span data-ttu-id="90d65-105">选择 NLP 服务时有哪些选项？</span><span class="sxs-lookup"><span data-stu-id="90d65-105">What are your options when choosing an NLP service?</span></span>

<span data-ttu-id="90d65-106">在 Azure 中，以下服务提供自然语言处理 (NLP) 功能：</span><span class="sxs-lookup"><span data-stu-id="90d65-106">In Azure, the following services provide natural language processing (NLP) capabilities:</span></span>

- [<span data-ttu-id="90d65-107">Azure HDInsight with Spark and Spark MLlib</span><span class="sxs-lookup"><span data-stu-id="90d65-107">Azure HDInsight with Spark and Spark MLlib</span></span>](/azure/hdinsight/spark/apache-spark-overview)
- [<span data-ttu-id="90d65-108">Microsoft 认知服务</span><span class="sxs-lookup"><span data-stu-id="90d65-108">Microsoft Cognitive Services</span></span>](/azure/#pivot=products&panel=cognitive)

## <a name="key-selection-criteria"></a><span data-ttu-id="90d65-109">关键选择条件</span><span class="sxs-lookup"><span data-stu-id="90d65-109">Key selection criteria</span></span>

<span data-ttu-id="90d65-110">若要缩小选择范围，请先回答以下问题：</span><span class="sxs-lookup"><span data-stu-id="90d65-110">To narrow the choices, start by answering these questions:</span></span>

- <span data-ttu-id="90d65-111">是否要使用预构建的模型？</span><span class="sxs-lookup"><span data-stu-id="90d65-111">Do you want to use prebuilt models?</span></span> <span data-ttu-id="90d65-112">如果是，请考虑使用 Microsoft 认知服务提供的 API。</span><span class="sxs-lookup"><span data-stu-id="90d65-112">If yes, consider using the APIs offered by Microsoft Cognitive Services.</span></span>

- <span data-ttu-id="90d65-113">是否需要针对大型文本数据语料库训练自定义模型？</span><span class="sxs-lookup"><span data-stu-id="90d65-113">Do you need to train custom models against a large corpus of text data?</span></span> <span data-ttu-id="90d65-114">如果是，请考虑使用 Azure HDInsight with Spark MLlib and Spark NLP。</span><span class="sxs-lookup"><span data-stu-id="90d65-114">If yes, consider using Azure HDInsight with Spark MLlib and Spark NLP.</span></span>

- <span data-ttu-id="90d65-115">是否需要低级别 NLP 功能，如词汇切分、词干分解、词形还原和词频/逆文档频率 (TF/IDF)？</span><span class="sxs-lookup"><span data-stu-id="90d65-115">Do you need low-level NLP capabilities like tokenization, stemming, lemmatization, and term frequency/inverse document frequency (TF/IDF)?</span></span> <span data-ttu-id="90d65-116">如果是，请考虑使用 Azure HDInsight with Spark MLlib and Spark NLP。</span><span class="sxs-lookup"><span data-stu-id="90d65-116">If yes, consider using Azure HDInsight with Spark MLlib and Spark NLP.</span></span>

- <span data-ttu-id="90d65-117">是否需要简单的高级别 NLP 功能，如实体和意图识别、主题检测、拼写检查或情绪分析？</span><span class="sxs-lookup"><span data-stu-id="90d65-117">Do you need simple, high-level NLP capabilities like entity and intent identification, topic detection, spell check, or sentiment analysis?</span></span> <span data-ttu-id="90d65-118">如果是，请考虑使用 Microsoft 认知服务提供的 API。</span><span class="sxs-lookup"><span data-stu-id="90d65-118">If yes, consider using the APIs offered by Microsoft Cognitive Services.</span></span>

## <a name="capability-matrix"></a><span data-ttu-id="90d65-119">功能矩阵</span><span class="sxs-lookup"><span data-stu-id="90d65-119">Capability matrix</span></span>

<span data-ttu-id="90d65-120">下表汇总了功能上的关键差异。</span><span class="sxs-lookup"><span data-stu-id="90d65-120">The following tables summarize the key differences in capabilities.</span></span>  

### <a name="general-capabilities"></a><span data-ttu-id="90d65-121">常规功能</span><span class="sxs-lookup"><span data-stu-id="90d65-121">General capabilities</span></span>

| | <span data-ttu-id="90d65-122">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="90d65-122">Azure HDInsight</span></span> | <span data-ttu-id="90d65-123">Microsoft 认知服务</span><span class="sxs-lookup"><span data-stu-id="90d65-123">Microsoft Cognitive Services</span></span> |
| --- | --- | --- |
| <span data-ttu-id="90d65-124">提供预先训练的模型作为服务</span><span class="sxs-lookup"><span data-stu-id="90d65-124">Provides pretrained models as a service</span></span> | <span data-ttu-id="90d65-125">否</span><span class="sxs-lookup"><span data-stu-id="90d65-125">No</span></span> | <span data-ttu-id="90d65-126">是</span><span class="sxs-lookup"><span data-stu-id="90d65-126">Yes</span></span> |
| <span data-ttu-id="90d65-127">REST API</span><span class="sxs-lookup"><span data-stu-id="90d65-127">REST API</span></span> | <span data-ttu-id="90d65-128">是</span><span class="sxs-lookup"><span data-stu-id="90d65-128">Yes</span></span> | <span data-ttu-id="90d65-129">是</span><span class="sxs-lookup"><span data-stu-id="90d65-129">Yes</span></span> |
| <span data-ttu-id="90d65-130">可编程性</span><span class="sxs-lookup"><span data-stu-id="90d65-130">Programmability</span></span> | <span data-ttu-id="90d65-131">Python、Scala、Java</span><span class="sxs-lookup"><span data-stu-id="90d65-131">Python, Scala, Java</span></span> | <span data-ttu-id="90d65-132">C#、Java、Node.js、Python、PHP、Ruby</span><span class="sxs-lookup"><span data-stu-id="90d65-132">C#, Java, Node.js, Python, PHP, Ruby</span></span> |
| <span data-ttu-id="90d65-133">支持大数据集和大型文档的处理</span><span class="sxs-lookup"><span data-stu-id="90d65-133">Support processing of big data sets and large documents</span></span> | <span data-ttu-id="90d65-134">是</span><span class="sxs-lookup"><span data-stu-id="90d65-134">Yes</span></span> | <span data-ttu-id="90d65-135">否</span><span class="sxs-lookup"><span data-stu-id="90d65-135">No</span></span> |

### <a name="low-level-natural-language-processing-capabilities"></a><span data-ttu-id="90d65-136">低级别的自然语言处理功能</span><span class="sxs-lookup"><span data-stu-id="90d65-136">Low-level natural language processing capabilities</span></span>

| | <span data-ttu-id="90d65-137">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="90d65-137">Azure HDInsight</span></span> | <span data-ttu-id="90d65-138">Microsoft 认知服务</span><span class="sxs-lookup"><span data-stu-id="90d65-138">Microsoft Cognitive Services</span></span> |  
| --- | --- | --- | 
| <span data-ttu-id="90d65-139">分词器</span><span class="sxs-lookup"><span data-stu-id="90d65-139">Tokenizer</span></span> | <span data-ttu-id="90d65-140">是 (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="90d65-140">Yes (Spark NLP)</span></span> | <span data-ttu-id="90d65-141">是（语言分析 API）</span><span class="sxs-lookup"><span data-stu-id="90d65-141">Yes (Linguistic Analysis API)</span></span> |
| <span data-ttu-id="90d65-142">词干分析器</span><span class="sxs-lookup"><span data-stu-id="90d65-142">Stemmer</span></span> | <span data-ttu-id="90d65-143">是 (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="90d65-143">Yes (Spark NLP)</span></span> | <span data-ttu-id="90d65-144">否</span><span class="sxs-lookup"><span data-stu-id="90d65-144">No</span></span> |
| <span data-ttu-id="90d65-145">词形还原工具</span><span class="sxs-lookup"><span data-stu-id="90d65-145">Lemmatizer</span></span> | <span data-ttu-id="90d65-146">是 (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="90d65-146">Yes (Spark NLP)</span></span> | <span data-ttu-id="90d65-147">否</span><span class="sxs-lookup"><span data-stu-id="90d65-147">No</span></span> |
| <span data-ttu-id="90d65-148">词性标记</span><span class="sxs-lookup"><span data-stu-id="90d65-148">Part of speech tagging</span></span> | <span data-ttu-id="90d65-149">是 (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="90d65-149">Yes (Spark NLP)</span></span> | <span data-ttu-id="90d65-150">是（语言分析 API）</span><span class="sxs-lookup"><span data-stu-id="90d65-150">Yes (Linguistic Analysis API)</span></span> |
| <span data-ttu-id="90d65-151">词频/逆向文档频率 (TF/IDF)</span><span class="sxs-lookup"><span data-stu-id="90d65-151">Term frequency/inverse-document frequency (TF/IDF)</span></span> | <span data-ttu-id="90d65-152">是 (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="90d65-152">Yes (Spark MLlib)</span></span> | <span data-ttu-id="90d65-153">否</span><span class="sxs-lookup"><span data-stu-id="90d65-153">No</span></span> |
| <span data-ttu-id="90d65-154">字符串相似性&mdash;编辑距离计算</span><span class="sxs-lookup"><span data-stu-id="90d65-154">String similarity&mdash;edit distance calculation</span></span> | <span data-ttu-id="90d65-155">是 (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="90d65-155">Yes (Spark MLlib)</span></span> | <span data-ttu-id="90d65-156">否</span><span class="sxs-lookup"><span data-stu-id="90d65-156">No</span></span> |
| <span data-ttu-id="90d65-157">N 元语法计算</span><span class="sxs-lookup"><span data-stu-id="90d65-157">N-gram calculation</span></span> | <span data-ttu-id="90d65-158">是 (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="90d65-158">Yes (Spark MLlib)</span></span> | <span data-ttu-id="90d65-159">否</span><span class="sxs-lookup"><span data-stu-id="90d65-159">No</span></span> |
| <span data-ttu-id="90d65-160">停止词删除</span><span class="sxs-lookup"><span data-stu-id="90d65-160">Stop word removal</span></span> | <span data-ttu-id="90d65-161">是 (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="90d65-161">Yes (Spark MLlib)</span></span> | <span data-ttu-id="90d65-162">否</span><span class="sxs-lookup"><span data-stu-id="90d65-162">No</span></span> |

### <a name="high-level-natural-language-processing-capabilities"></a><span data-ttu-id="90d65-163">高级别的自然语言处理功能</span><span class="sxs-lookup"><span data-stu-id="90d65-163">High-level natural language processing capabilities</span></span>

| | <span data-ttu-id="90d65-164">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="90d65-164">Azure HDInsight</span></span> | <span data-ttu-id="90d65-165">Microsoft 认知服务</span><span class="sxs-lookup"><span data-stu-id="90d65-165">Microsoft Cognitive Services</span></span> |
| --- | --- | --- | 
| <span data-ttu-id="90d65-166">实体/意图识别和提取</span><span class="sxs-lookup"><span data-stu-id="90d65-166">Entity/intent identification & extraction</span></span> | <span data-ttu-id="90d65-167">否</span><span class="sxs-lookup"><span data-stu-id="90d65-167">No</span></span> | <span data-ttu-id="90d65-168">是（语言理解智能服务 (LUIS) API）</span><span class="sxs-lookup"><span data-stu-id="90d65-168">Yes (Language Understanding Intelligent Service (LUIS) API)</span></span> |    
| <span data-ttu-id="90d65-169">主题检测</span><span class="sxs-lookup"><span data-stu-id="90d65-169">Topic detection</span></span> | <span data-ttu-id="90d65-170">是 (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="90d65-170">Yes (Spark NLP)</span></span> | <span data-ttu-id="90d65-171">是（文本分析 API）</span><span class="sxs-lookup"><span data-stu-id="90d65-171">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="90d65-172">拼写检查</span><span class="sxs-lookup"><span data-stu-id="90d65-172">Spell checking</span></span> | <span data-ttu-id="90d65-173">是 (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="90d65-173">Yes (Spark NLP)</span></span> | <span data-ttu-id="90d65-174">是（必应拼写检查 API）</span><span class="sxs-lookup"><span data-stu-id="90d65-174">Yes (Bing Spell Check API)</span></span> |
| <span data-ttu-id="90d65-175">情绪分析</span><span class="sxs-lookup"><span data-stu-id="90d65-175">Sentiment analysis</span></span> | <span data-ttu-id="90d65-176">是 (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="90d65-176">Yes (Spark NLP)</span></span> | <span data-ttu-id="90d65-177">是（文本分析 API）</span><span class="sxs-lookup"><span data-stu-id="90d65-177">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="90d65-178">语言检测</span><span class="sxs-lookup"><span data-stu-id="90d65-178">Language detection</span></span> | <span data-ttu-id="90d65-179">否</span><span class="sxs-lookup"><span data-stu-id="90d65-179">No</span></span> | <span data-ttu-id="90d65-180">是（文本分析 API）</span><span class="sxs-lookup"><span data-stu-id="90d65-180">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="90d65-181">支持除英语以外的多种语言</span><span class="sxs-lookup"><span data-stu-id="90d65-181">Supports multiple languages besides English</span></span> | <span data-ttu-id="90d65-182">否</span><span class="sxs-lookup"><span data-stu-id="90d65-182">No</span></span> | <span data-ttu-id="90d65-183">是（因 API 而异）</span><span class="sxs-lookup"><span data-stu-id="90d65-183">Yes (varies by API)</span></span> |

## <a name="see-also"></a><span data-ttu-id="90d65-184">另请参阅</span><span class="sxs-lookup"><span data-stu-id="90d65-184">See also</span></span>

[<span data-ttu-id="90d65-185">自然语言处理</span><span class="sxs-lookup"><span data-stu-id="90d65-185">Natural language processing</span></span>](../scenarios/natural-language-processing.md)