---
title: 选择搜索数据存储
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: ead07e307e96696faa5ddf48505eee378027523c
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/06/2018
ms.locfileid: "30848608"
---
# <a name="choosing-a-search-data-store-in-azure"></a>在 Azure 中选择搜索数据存储

本文对 Azure 中用于搜索数据存储的技术选择进行了比较。 搜索数据存储用来创建和存储用于对自由格式文本执行查询的专用索引。 编制了索引的文本可以驻留在单独的数据存储中，例如 blob 存储。 应用程序将查询提交到搜索数据存储，结果是匹配的文档的列表。 有关此方案的详细信息，请参阅[为搜索处理自由格式文本](../scenarios/search.md)。 

## <a name="what-are-your-options-when-choosing-a-search-data-store"></a>在选择搜索数据存储时有哪些选项？
在 Azure 中，以下所有数据存储都将通过提供搜索索引来满足对自由格式文本数据的核心搜索要求：
- [Azure 搜索](/azure/search/search-what-is-azure-search)
- [Elasticsearch](https://azuremarketplace.microsoft.com/marketplace/apps/elastic.elasticsearch?tab=Overview)
- [基于 Solr 的 HDInsight](/azure/hdinsight/hdinsight-hadoop-solr-install-linux)
- [具有全文搜索功能的 Azure SQL 数据库](/sql/relational-databases/search/full-text-search)


## <a name="key-selection-criteria"></a>关键选择条件

针对搜索方案，通过回答以下问题开始选择合适的满足需求的搜索数据存储：

- 你希望使用托管服务还是由你管理自己的服务器？

- 是否可以在设计时指定索引架构？ 如果不可以，请选择一个支持可更新架构的选项。

- 只需要为全文搜索使用索引，还是也需要快速聚合数字数据和其他分析？ 如果需要超出全文搜索的功能，请考虑使用支持其他分析的选项。

- 是否需要一个支持日志收集、聚合及已索引数据可视化且用于日志分析的搜索索引？ 如果需要，请考虑使用 Elasticsearch，它是日志分析堆栈的一部分。

- 是否需要为常见文档格式（例如 PDF、Word、PowerPoint 和 Excel）的数据编制索引？ 如果是，请选择一个提供文档索引器的选项。

- 数据库是否有特定的安全需求？ 如果是，请考虑使用下面列出的安全功能。

## <a name="capability-matrix"></a>功能矩阵

以下各表汇总了功能上的关键差异。

### <a name="general-capabilities"></a>常规功能

| | Azure 搜索 | Elasticsearch | 基于 Solr 的 HDInsight | SQL 数据库 | 
| --- | --- | --- | --- | --- | 
| 是托管服务 | 是 | 否 | 是 | 是 |  
| REST API | 是 | 是 | 是 | 否 |
| 可编程性 | .NET | Java | Java | T-SQL | 
| 常见文件类型（PDF、DOCX、TXT、等等）的文档索引器 | 是 | 否 | 是 | 否 |

### <a name="manageability-capabilities"></a>可管理性功能

| | Azure 搜索 | Elasticsearch | 基于 Solr 的 HDInsight | SQL 数据库 | 
| --- | --- | --- | --- | --- |
| 可更新架构 | 否 | 是 | 是 | 是 |
| 支持横向扩展  | 是 | 是 | 是 | 否 |

### <a name="analytic-workload-capabilities"></a>分析工作负荷功能

| | Azure 搜索 | Elasticsearch | 基于 Solr 的 HDInsight | SQL 数据库 | 
| --- | --- | --- | --- | --- | 
| 支持超出全文搜索的分析 | 否 | 是 | 是 | 是 |
| 日志分析堆栈的一部分 | 否 | 是 (ELK) |  否 | 否 |
| 支持语义搜索 | 是（仅限查找类似文档） | 是 | 是 | 是 | 

### <a name="security-capabilities"></a>安全功能

| | Azure 搜索 | Elasticsearch | 基于 Solr 的 HDInsight | SQL 数据库 | 
| --- | --- | --- | --- | --- | 
| 行级别安全性 | 部分（要求应用程序查询按组 id 进行筛选） | 部分（要求应用程序查询按组 id 进行筛选） | 是 | 是 | 
| 透明数据加密 | 否 | 否 | 否 | 是 |  
| 限制访问，仅限特定 IP 地址进行访问 | 否 | 是 | 是 | 是 |   
| 限制访问，仅允许访问虚拟网络 | 否 | 是 | 是 | 是 |  
| Active Directory 身份验证（集成身份验证） | 否 | 否 | 否 | 是 | 

## <a name="see-also"></a>另请参阅

[为搜索处理自由格式文本](../scenarios/search.md)