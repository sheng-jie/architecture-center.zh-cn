---
title: 语义建模
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 343d17af0d933d515c724a062237c8d5df3a9e31
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/31/2018
---
# <a name="semantic-modeling"></a><span data-ttu-id="01411-102">语义建模</span><span class="sxs-lookup"><span data-stu-id="01411-102">Semantic modeling</span></span>

<span data-ttu-id="01411-103">语义数据模型是一个概念模型，描述了它包含的数据元素的含义。</span><span class="sxs-lookup"><span data-stu-id="01411-103">A semantic data model is a conceptual model that describes the meaning of the data elements it contains.</span></span> <span data-ttu-id="01411-104">各个组织经常使用其自己的术语来表示事物，有时使用同义词，甚至同一术语具有不同的含义。</span><span class="sxs-lookup"><span data-stu-id="01411-104">Organizations often have their own terms for things, sometimes with synonyms, or even different meanings for the same term.</span></span> <span data-ttu-id="01411-105">例如，库存数据库可能使用资产 ID 和序列号来跟踪设备部件，但销售数据库可能将序列号称为资产 ID。</span><span class="sxs-lookup"><span data-stu-id="01411-105">For example, an inventory database might track a piece of equipment with an asset ID and a serial number, but a sales database might refer to the serial number as the asset ID.</span></span> <span data-ttu-id="01411-106">如果没有对关系进行描述的模型，则没有简单的方法来将这些值关联起来。</span><span class="sxs-lookup"><span data-stu-id="01411-106">There is no simple way to relate these values without a model that describes the relationship.</span></span> 

<span data-ttu-id="01411-107">语义建模基于数据库架构提供了一定级别的抽象，因此，用户不需要知道基础数据结构。</span><span class="sxs-lookup"><span data-stu-id="01411-107">Semantic modeling provides a level of abstraction over the database schema, so that users don't need to know the underlying data structures.</span></span> <span data-ttu-id="01411-108">这使得最终用户可以更容易地查询数据，不需要基于基础架构执行聚合和联接。</span><span class="sxs-lookup"><span data-stu-id="01411-108">This makes it easier for end users to query data without performing aggregates and joins over the underlying schema.</span></span> <span data-ttu-id="01411-109">此外，通常会将列重命名为对用户更友好的名称，使数据的上下文和含义更加明确。</span><span class="sxs-lookup"><span data-stu-id="01411-109">Also, usually columns are renamed to more user-friendly names, so that the context and meaning of the data are more obvious.</span></span>

<span data-ttu-id="01411-110">语义建模主要用于高频读取应用场景中，例如分析和商业智能 (OLAP)，与之相对的是高频写入事务数据处理 (OLTP)。</span><span class="sxs-lookup"><span data-stu-id="01411-110">Semantic modeling is predominately used for read-heavy scenarios, such as analytics and business intelligence (OLAP), as opposed to more write-heavy transactional data processing (OLTP).</span></span> <span data-ttu-id="01411-111">这主要是由典型语义层的性质导致的：</span><span class="sxs-lookup"><span data-stu-id="01411-111">This is mostly due to the nature of a typical semantic layer:</span></span>

- <span data-ttu-id="01411-112">设置了聚合行为，以便报告工具可以正确显示它们。</span><span class="sxs-lookup"><span data-stu-id="01411-112">Aggregation behaviors are set so that reporting tools display them properly.</span></span>
- <span data-ttu-id="01411-113">定义了业务逻辑和计算。</span><span class="sxs-lookup"><span data-stu-id="01411-113">Business logic and calculations are defined.</span></span>
- <span data-ttu-id="01411-114">包括了面向时间的计算。</span><span class="sxs-lookup"><span data-stu-id="01411-114">Time-oriented calculations are included.</span></span>
- <span data-ttu-id="01411-115">数据通常是从多个源集成的。</span><span class="sxs-lookup"><span data-stu-id="01411-115">Data is often integrated from multiple sources.</span></span> 

<span data-ttu-id="01411-116">传统上，由于这些原因，语义层放置在数据仓库之上。</span><span class="sxs-lookup"><span data-stu-id="01411-116">Traditionally, the semantic layer is placed over a data warehouse for these reasons.</span></span>

![位于数据仓库与报告工具之间的语义层的示例关系图](./images/semantic-modeling.png)

<span data-ttu-id="01411-118">有两种主要类型的语义模型：</span><span class="sxs-lookup"><span data-stu-id="01411-118">There are two primary types of semantic models:</span></span>

* <span data-ttu-id="01411-119">**表格**。</span><span class="sxs-lookup"><span data-stu-id="01411-119">**Tabular**.</span></span> <span data-ttu-id="01411-120">使用关系建模构造（模型、表、列）。</span><span class="sxs-lookup"><span data-stu-id="01411-120">Uses relational modeling constructs (model, tables, columns).</span></span> <span data-ttu-id="01411-121">在内部，元数据是从 OLAP 建模构造（多维数据集、维、度量值）继承的。</span><span class="sxs-lookup"><span data-stu-id="01411-121">Internally, metadata is inherited from OLAP modeling constructs (cubes, dimensions, measures).</span></span> <span data-ttu-id="01411-122">代码和脚本使用 OLAP 元数据。</span><span class="sxs-lookup"><span data-stu-id="01411-122">Code and script use OLAP metadata.</span></span>
* <span data-ttu-id="01411-123">**多维**。</span><span class="sxs-lookup"><span data-stu-id="01411-123">**Multidimensional**.</span></span> <span data-ttu-id="01411-124">使用传统的 OLAP 建模构造（多维数据集、维、度量值）。</span><span class="sxs-lookup"><span data-stu-id="01411-124">Uses traditional OLAP modeling constructs (cubes, dimensions, measures).</span></span>

<span data-ttu-id="01411-125">相关的 Azure 服务：</span><span class="sxs-lookup"><span data-stu-id="01411-125">Relevant Azure service:</span></span>
- [<span data-ttu-id="01411-126">Azure Analysis Services</span><span class="sxs-lookup"><span data-stu-id="01411-126">Azure Analysis Services</span></span>](https://azure.microsoft.com/services/analysis-services/)

## <a name="example-use-case"></a><span data-ttu-id="01411-127">示例用例</span><span class="sxs-lookup"><span data-stu-id="01411-127">Example use case</span></span>

<span data-ttu-id="01411-128">某个组织将数据存储在大型数据库中。</span><span class="sxs-lookup"><span data-stu-id="01411-128">An organization has data stored in a large database.</span></span> <span data-ttu-id="01411-129">它希望使此数据可供业务用户和客户用来创建其自己的报表以及执行某些分析。</span><span class="sxs-lookup"><span data-stu-id="01411-129">It wants to make this data available to business users and customers to create their own reports and do some analysis.</span></span> <span data-ttu-id="01411-130">一种选择是向那些用户授予对数据库的直接访问权限。</span><span class="sxs-lookup"><span data-stu-id="01411-130">One option is just to give those users direct access to the database.</span></span> <span data-ttu-id="01411-131">但是，这样做有几个缺点，包括安全性管理和访问控制。</span><span class="sxs-lookup"><span data-stu-id="01411-131">However, there are several drawbacks to doing this, including managing security and controlling access.</span></span> <span data-ttu-id="01411-132">此外，数据库的设计（包括表和列的名称）对用户而言可能难以理解。</span><span class="sxs-lookup"><span data-stu-id="01411-132">Also, the design of the database, including the names of tables and columns, may be hard for a user to understand.</span></span> <span data-ttu-id="01411-133">用户将需要知道要查询哪些表，应当如何联接那些表，还需要知道为获得正确结果而必须应用的其他业务逻辑。</span><span class="sxs-lookup"><span data-stu-id="01411-133">Users would need to know which tables to query, how those tables should be joined, and other business logic that must be applied to get the correct results.</span></span> <span data-ttu-id="01411-134">用户甚至还需要了解 SQL 之类的查询语言才能入门。</span><span class="sxs-lookup"><span data-stu-id="01411-134">Users would also need to know a query language like SQL even to get started.</span></span> <span data-ttu-id="01411-135">通常，这将导致多个用户在报告相同的指标时出现不同的结果。</span><span class="sxs-lookup"><span data-stu-id="01411-135">Typically this leads to multiple users reporting the same metrics but with different results.</span></span>

<span data-ttu-id="01411-136">另一种选择是将用户需要的所有信息封装到一个语义模型中。</span><span class="sxs-lookup"><span data-stu-id="01411-136">Another option is to encapsulate all of the information that users need into a semantic model.</span></span> <span data-ttu-id="01411-137">用户可以使用他们选择的报告工具更轻松地查询语义模型。</span><span class="sxs-lookup"><span data-stu-id="01411-137">The semantic model can be more easily queried by users with a reporting tool of their choice.</span></span> <span data-ttu-id="01411-138">语义模型提供的数据是从数据仓库中拉取的，这确保了所有用户看到的是单一版本的真实数据。</span><span class="sxs-lookup"><span data-stu-id="01411-138">The data provided by the semantic model is pulled from a data warehouse, ensuring that all users see a single version of the truth.</span></span> <span data-ttu-id="01411-139">语义模型还提供了友好的表名和列名、表之间的关系、说明、计算以及行级安全性。</span><span class="sxs-lookup"><span data-stu-id="01411-139">The semantic model also provides friendly table and column names, relationships between tables, descriptions, calculations, and row-level security.</span></span>

## <a name="typical-traits-of-semantic-modeling"></a><span data-ttu-id="01411-140">语义建模的典型特征</span><span class="sxs-lookup"><span data-stu-id="01411-140">Typical traits of semantic modeling</span></span>

<span data-ttu-id="01411-141">语义建模和分析处理通常具有以下特征：</span><span class="sxs-lookup"><span data-stu-id="01411-141">Semantic modeling and analytical processing tends to have the following traits:</span></span>

| <span data-ttu-id="01411-142">要求</span><span class="sxs-lookup"><span data-stu-id="01411-142">Requirement</span></span> | <span data-ttu-id="01411-143">说明</span><span class="sxs-lookup"><span data-stu-id="01411-143">Description</span></span> |
| --- | --- |
| <span data-ttu-id="01411-144">架构</span><span class="sxs-lookup"><span data-stu-id="01411-144">Schema</span></span> | <span data-ttu-id="01411-145">写入时架构，强制实施</span><span class="sxs-lookup"><span data-stu-id="01411-145">Schema on write, strongly enforced</span></span>|
| <span data-ttu-id="01411-146">使用事务</span><span class="sxs-lookup"><span data-stu-id="01411-146">Uses Transactions</span></span> | <span data-ttu-id="01411-147">否</span><span class="sxs-lookup"><span data-stu-id="01411-147">No</span></span> |
| <span data-ttu-id="01411-148">锁定策略</span><span class="sxs-lookup"><span data-stu-id="01411-148">Locking Strategy</span></span> | <span data-ttu-id="01411-149">无</span><span class="sxs-lookup"><span data-stu-id="01411-149">None</span></span> |
| <span data-ttu-id="01411-150">可更新</span><span class="sxs-lookup"><span data-stu-id="01411-150">Updateable</span></span> | <span data-ttu-id="01411-151">否（通常需要重新计算多维数据集）</span><span class="sxs-lookup"><span data-stu-id="01411-151">No (typically requires recomputing cube)</span></span> |
| <span data-ttu-id="01411-152">可追加</span><span class="sxs-lookup"><span data-stu-id="01411-152">Appendable</span></span> | <span data-ttu-id="01411-153">否（通常需要重新计算多维数据集）</span><span class="sxs-lookup"><span data-stu-id="01411-153">No (typically requires recomputing cube)</span></span> |
| <span data-ttu-id="01411-154">工作负载</span><span class="sxs-lookup"><span data-stu-id="01411-154">Workload</span></span> | <span data-ttu-id="01411-155">高频读取，只读的</span><span class="sxs-lookup"><span data-stu-id="01411-155">Heavy reads, read-only</span></span> |
| <span data-ttu-id="01411-156">索引</span><span class="sxs-lookup"><span data-stu-id="01411-156">Indexing</span></span> | <span data-ttu-id="01411-157">多维索引</span><span class="sxs-lookup"><span data-stu-id="01411-157">Multidimensional indexing</span></span> |
| <span data-ttu-id="01411-158">基准大小</span><span class="sxs-lookup"><span data-stu-id="01411-158">Datum size</span></span> | <span data-ttu-id="01411-159">中小型</span><span class="sxs-lookup"><span data-stu-id="01411-159">Small to medium sized</span></span> |
| <span data-ttu-id="01411-160">模型</span><span class="sxs-lookup"><span data-stu-id="01411-160">Model</span></span> | <span data-ttu-id="01411-161">多维</span><span class="sxs-lookup"><span data-stu-id="01411-161">Multidimensional</span></span> |
| <span data-ttu-id="01411-162">数据形状：</span><span class="sxs-lookup"><span data-stu-id="01411-162">Data shape:</span></span>| <span data-ttu-id="01411-163">多维数据集或星型/雪花型架构</span><span class="sxs-lookup"><span data-stu-id="01411-163">Cube or star/snowflake schema</span></span> |
| <span data-ttu-id="01411-164">查询灵活性</span><span class="sxs-lookup"><span data-stu-id="01411-164">Query flexibility</span></span> | <span data-ttu-id="01411-165">高度灵活</span><span class="sxs-lookup"><span data-stu-id="01411-165">Highly flexible</span></span> |
| <span data-ttu-id="01411-166">规模：</span><span class="sxs-lookup"><span data-stu-id="01411-166">Scale:</span></span> | <span data-ttu-id="01411-167">大型（数十到数百 GB）</span><span class="sxs-lookup"><span data-stu-id="01411-167">Large (10s-100s GBs)</span></span> |

## <a name="see-also"></a><span data-ttu-id="01411-168">另请参阅</span><span class="sxs-lookup"><span data-stu-id="01411-168">See also</span></span>

- [<span data-ttu-id="01411-169">数据仓库</span><span class="sxs-lookup"><span data-stu-id="01411-169">Data warehousing</span></span>](../scenarios/data-warehousing.md)
- [<span data-ttu-id="01411-170">联机分析处理 (OLAP)</span><span class="sxs-lookup"><span data-stu-id="01411-170">Online analytical processing (OLAP)</span></span>](../scenarios/online-analytical-processing.md)
