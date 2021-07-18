---
title: 处理 CSV 和 JSON 文件
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 02e684d562cfe555f9e3596ad0a2f1a00d05c7a7
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/03/2018
ms.locfileid: "30298605"
---
# <a name="working-with-csv-and-json-files-for-data-solutions"></a>处理数据解决方案的 CSV 和 JSON 文件

CSV 和 JSON 可能是用于引入、交换和存储非结构化或半结构化数据的最常用格式。 

## <a name="about-csv-format"></a>关于 CSV 格式

CSV（逗号分隔值）文件通常用于在系统之间以纯文本形式交换表格数据。 它们通常包含一个提供数据列名的标头行，否则会将数据视为半结构化。 这是因为 CSV 无法自然表示分层数据或关系数据。 数据关系通常使用多个 CSV 文件来处理，其中的外键存储在一个或多个文件的列中，但这些文件之间的关系不会由格式本身表示。 CSV 格式的文件可以使用除逗号以外的其他分隔符，例如制表符或空格。

尽管存在一些限制，但 CSV 文件仍是数据交换的热门选项，因为它们受众多业务、消费型和科技型应用程序的支持。 例如，数据库和电子表格程序可以导入和导出 CSV 文件。 同样，大多数批处理和流数据处理引擎（例如 Spark 和 Hadoop）原生支持序列化和反序列化 CSV 格式的文件，并提供相应的方式用于在读取时应用架构。 由于可以使用相应的选项查询数据并以更有效的数据格式存储信息以加速处理，因此可以更轻松地处理数据。

## <a name="about-json-format"></a>关于 JSON 格式

JSON（JavaScript 对象表示法）数据表示为半结构化格式的键值对。 人们经常将 JSON 与 XML 相对比，因为两者都能以分层格式存储数据，并且表示的子数据内联在其父级中。 两者都是自述性的且可供用户阅读，但 JSON 文档往往要小得多，因此在联机数据交换中非常流行，尤其是在基于 REST 的 Web 服务应用场合中。 

与 CSV 相比，JSON 格式的文件具有以下几种优点：

* JSON 保留分层结构，使我们能够更轻松地在一个文档中保存相关数据，并表示复杂的关系。
* 大多数编程语言原生支持将 JSON 反序列化为对象，或提供轻量 JSON 序列化库。
* JSON 支持对象列表，可帮助避免杂乱无章地将列表转换为关系数据模型。
* JSON 是适用于 MongoDB、Couchbase 和 Azure Cosmos DB 等 NoSQL 数据库的常用文件格式。

由于来自网络的大量数据已采用 JSON 格式，因此大多数基于 Web 的编程语言都支持本机使用 JSON，或者通过外部库序列化和反序列化 JSON 数据。 由于 JSON 受到普遍支持，因此可以通过数据结构表示法、热数据交换格式和冷数据的数据存储，将它用作逻辑格式。

许多批处理和流数据处理引擎原生支持 JSON 序列化和反序列化。 尽管最终可以使用性能得到进一步优化的格式（例如 Parquet 或 Avro）存储 JSON 文档中包含的数据，但这些数据充当真实源的原始数据，在按需重新处理数据时，这种性质非常关键。

## <a name="when-to-use-csv-or-json-formats"></a>何时使用 CSV 或 JSON 格式

CSV 更常用于导出和导入数据，或者处理数据以供分析和机器学习。 JSON 格式的文件具有相同的优点，但更常用于热数据交换解决方案。 JSON 文档通常由执行联机事务的 Web 和移动设备、单向或双向通信的 IoT（物联网）设备，或者与 SaaS 和 PaaS 服务或无服务器体系结构通信的客户端应用程序发送。 

使用 CSV 和 JSON 文件格式都能在不同的系统或设备之间交换数据。 借助它们的半结构化格式可以灵活传输几乎任何类型的数据，这些格式受到的普遍支持使它们易于使用。 如果处理的数据以二进制格式存储，则可以将两者用作原始事实源，以提高查询效率。 

## <a name="working-with-csv-and-json-data-in-azure"></a>在 Azure 中处理 CSV 和 JSON 数据

Azure 提供多种解决方案用于根据需要处理 CSV 和 JSON 文件。 这些文件的主要存储位置是 Azure 存储或 Azure Data Lake Store。 处理这些文件和其他基于文本的文件的大多数 Azure 服务都与任一对象存储服务相集成。 但是，在某些情况下，可以选择将数据直接导入 Azure SQL 或其他某个数据存储。 SQL Server 原生支持存储和处理 JSON 文档，因此，可以轻松[导入和处理这些类型的文件](/sql/relational-databases/json/import-json-documents-into-sql-server)。 可以使用 SQL 批量导入等实用工具轻松[导入 CSV 文件](/sql/relational-databases/json/import-json-documents-into-sql-server)。

可以根据情况对数据执行[批处理](../big-data/batch-processing.md)或[实时处理](../big-data/real-time-processing.md)。

## <a name="challenges"></a>挑战

使用这些格式时，需要考虑到一些难题：

* 如果数据模型没有任何约束，则 CSV 和 JSON 文件中很容易出现数据损坏（“胡乱输入输出”）。 例如，这两种文件都不提供日期/时间对象的表示，因此，该文件格式不会阻止在日期字段中插入“ABC123”。

* 在处理大数据时，使用 CSV 和 JSON 文件作为冷存储解决方案无法正常缩放。 在大多数情况下，这些文件将拆分成分区供并行处理，同时无法压缩为二进制格式。 这通常导致将这些数据处理和存储成 Parquet 和 ORC（优化行纵栏表）等读取优化的格式，这些格式还提供索引以及有关包含的数据的内联统计信息。

* 可能需要对半结构化数据应用架构才能更轻松地执行查询和分析。 通常，这需要以符合环境数据存储需求的另一种格式存储数据，例如，在数据库中存储。

