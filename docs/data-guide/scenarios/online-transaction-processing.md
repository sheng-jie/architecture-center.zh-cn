---
title: "联机事务处理 (OLTP)"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 07e7f680c8ee5e8589ff7cd2236ff95f6ee84f4c
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/14/2018
---
# <a name="online-transaction-processing-oltp"></a><span data-ttu-id="f4992-102">联机事务处理 (OLTP)</span><span class="sxs-lookup"><span data-stu-id="f4992-102">Online transaction processing (OLTP)</span></span>

<span data-ttu-id="f4992-103">使用计算机系统对[事务数据](../concepts/transactional-data.md)进行管理称为联机事务处理 (OLTP)。</span><span class="sxs-lookup"><span data-stu-id="f4992-103">The management of [transactional data](../concepts/transactional-data.md) using computer systems is referred to as Online Transaction Processing (OLTP).</span></span> <span data-ttu-id="f4992-104">OLTP 系统在组织的日常运营中发生业务交互时对其进行记录，并且支持对该数据进行查询来做出推断。</span><span class="sxs-lookup"><span data-stu-id="f4992-104">OLTP systems record business interactions as they occur in the day-to-day operation of the organization, and support querying of this data to make inferences.</span></span>

![Azure 中的 OLTP](./images/oltp-data-pipeline.png)

## <a name="when-to-use-this-solution"></a><span data-ttu-id="f4992-106">何时使用此解决方案</span><span class="sxs-lookup"><span data-stu-id="f4992-106">When to use this solution</span></span>

<span data-ttu-id="f4992-107">如果需要高效地处理和存储业务事务并立即使其能够以一致的方式供客户端应用程序使用，请选择 OLTP。</span><span class="sxs-lookup"><span data-stu-id="f4992-107">Choose OLTP when you need to efficiently process and store business transactions and immediately make them available to client applications in a consistent way.</span></span> <span data-ttu-id="f4992-108">当处理过程中任何明显的延迟都会对业务的日常运营造成负面影响时，请使用此体系结构。</span><span class="sxs-lookup"><span data-stu-id="f4992-108">Use this architecture when any tangible delay in processing would have a negative impact on the day-to-day operations of the business.</span></span>

<span data-ttu-id="f4992-109">OLTP 系统设计用于高效地处理和存储事务，以及查询事务数据。</span><span class="sxs-lookup"><span data-stu-id="f4992-109">OLTP systems are designed to efficiently process and store transactions, as well as query transactional data.</span></span> <span data-ttu-id="f4992-110">通过 OLTP 系统高效地处理和存储各个事务有一部分是通过数据规范化完成的 &mdash; 也就是说，将数据拆分为冗余较少的块。</span><span class="sxs-lookup"><span data-stu-id="f4992-110">The goal of efficiently processing and storing individual transactions by an OLTP system is partly accomplished by data normalization &mdash; that is, breaking the data up into smaller chunks that are less redundant.</span></span> <span data-ttu-id="f4992-111">这可以提高效率，因为它使得 OLTP 系统能够独立处理大量的事务，并且避免了在存在冗余数据的情况下维护数据完整性所需执行的额外处理。</span><span class="sxs-lookup"><span data-stu-id="f4992-111">This supports efficiency because it enables the OLTP system to process large numbers of transactions independently, and avoids extra processing needed to maintain data integrity in the presence of redundant data.</span></span>

## <a name="challenges"></a><span data-ttu-id="f4992-112">挑战</span><span class="sxs-lookup"><span data-stu-id="f4992-112">Challenges</span></span>
<span data-ttu-id="f4992-113">实现和使用 OLTP 系统可能会带来一些挑战：</span><span class="sxs-lookup"><span data-stu-id="f4992-113">Implementing and using an OLTP system can create a few challenges:</span></span>

- <span data-ttu-id="f4992-114">OLTP 系统并非非常适合用于处理对大量数据的聚合，但是有一些例外，例如，进行了良好规划的基于 SQL Server 的解决方案。</span><span class="sxs-lookup"><span data-stu-id="f4992-114">OLTP systems are not always good for handling aggregates over large amounts of data, although there are exceptions, such as a well-planned SQL Server-based solution.</span></span> <span data-ttu-id="f4992-115">对数据的分析依赖于对数百个单独事务的聚合计算，对于 OLTP 系统而言，这会消耗大量的资源。</span><span class="sxs-lookup"><span data-stu-id="f4992-115">Analytics against the data, that rely on aggregate calculations over millions of individual transactions, are very resource intensive for an OLTP system.</span></span> <span data-ttu-id="f4992-116">它们可能执行较慢，并且可能会因为阻塞数据库中的其他事务在而导致减速。</span><span class="sxs-lookup"><span data-stu-id="f4992-116">They can be slow to execute and can cause a slow-down by blocking other transactions in the database.</span></span>
- <span data-ttu-id="f4992-117">在对高度规范化的数据进行分析和报告时，查询通常比较复杂，因为大多数查询需要使用联接来使数据非规范化。</span><span class="sxs-lookup"><span data-stu-id="f4992-117">When conducting analytics and reporting on data that is highly normalized, the queries tend to be complex, because most queries need to de-normalize the data by using joins.</span></span> <span data-ttu-id="f4992-118">此外，在 OLTP 系统中，数据库对象的命名约定通常简洁而精炼。</span><span class="sxs-lookup"><span data-stu-id="f4992-118">Also, naming conventions for database objects in OLTP systems tend to be terse and succinct.</span></span> <span data-ttu-id="f4992-119">OLTP 系统中增强的规范化与简洁的命名约定共同使得业务用户在没有 DBA 或数据开发者帮助的情况下难以执行查询。</span><span class="sxs-lookup"><span data-stu-id="f4992-119">The increased normalization coupled with terse naming conventions makes OLTP systems difficult for business users to query, without the help of a DBA or data developer.</span></span>
- <span data-ttu-id="f4992-120">无限期地存储事务历史记录以及在任何一个表中存储太多数据都会导致查询性能变慢，具体取决于所存储的事务数。</span><span class="sxs-lookup"><span data-stu-id="f4992-120">Storing the history of transactions indefinitely and storing too much data in any one table can lead to slow query performance, depending on the number of transactions stored.</span></span> <span data-ttu-id="f4992-121">常见的解决方案是在 OLTP 系统中维护一个相关时间范围（例如当前会计年度）并将历史数据卸载到其他系统，例如数据市场或[数据仓库](../technology-choices/data-warehouses.md)。</span><span class="sxs-lookup"><span data-stu-id="f4992-121">The common solution is to maintain a relevant window of time (such as the current fiscal year) in the OLTP system and offload historical data to other systems, such as a data mart or [data warehouse](../technology-choices/data-warehouses.md).</span></span>

## <a name="oltp-in-azure"></a><span data-ttu-id="f4992-122">Azure 中的 OLTP</span><span class="sxs-lookup"><span data-stu-id="f4992-122">OLTP in Azure</span></span>

<span data-ttu-id="f4992-123">[应用服务 Web 应用](/azure/app-service/app-service-web-overview)中托管的应用程序（例如网站）、在应用服务中运行的 REST API、移动或桌面应用程序通常通过 REST API 中介与 OLTP 系统进行通信。</span><span class="sxs-lookup"><span data-stu-id="f4992-123">Applications such as websites hosted in [App Service Web Apps](/azure/app-service/app-service-web-overview), REST APIs running in App Service, or mobile or desktop applications communicate with the OLTP system, typically via a REST API intermediary.</span></span>

<span data-ttu-id="f4992-124">实际上，大多数工作负荷不单纯是 OLTP。</span><span class="sxs-lookup"><span data-stu-id="f4992-124">In practice, most workloads are not purely OLTP.</span></span> <span data-ttu-id="f4992-125">通常还存在[分析组件](../scenarios/online-analytical-processing.md)。</span><span class="sxs-lookup"><span data-stu-id="f4992-125">There tends to be an [analytical component](../scenarios/online-analytical-processing.md) as well.</span></span> <span data-ttu-id="f4992-126">此外，对实时报告的需求也在增加，例如针对运营系统运行报表。</span><span class="sxs-lookup"><span data-stu-id="f4992-126">In addition, there is an increasing demand for real-time reporting, such as running reports against the operational system.</span></span> <span data-ttu-id="f4992-127">这也称为 HTAP（混合事务性和分析处理）。</span><span class="sxs-lookup"><span data-stu-id="f4992-127">This is also referred to as HTAP (Hybrid Transactional and Analytical Processing).</span></span> <span data-ttu-id="f4992-128">有关详细信息，请参阅[联机分析处理 (OLAP) 数据存储](../technology-choices/olap-data-stores.md)。</span><span class="sxs-lookup"><span data-stu-id="f4992-128">For more information, see [Online Analytical Processing (OLAP) data stores](../technology-choices/olap-data-stores.md).</span></span>

## <a name="technology-choices"></a><span data-ttu-id="f4992-129">技术选择</span><span class="sxs-lookup"><span data-stu-id="f4992-129">Technology choices</span></span>

<span data-ttu-id="f4992-130">数据存储：</span><span class="sxs-lookup"><span data-stu-id="f4992-130">Data storage:</span></span>

- [<span data-ttu-id="f4992-131">Azure SQL 数据库</span><span class="sxs-lookup"><span data-stu-id="f4992-131">Azure SQL Database</span></span>](/azure/sql-database/)
- [<span data-ttu-id="f4992-132">Azure VM 中的 SQL Server</span><span class="sxs-lookup"><span data-stu-id="f4992-132">SQL Server in an Azure VM</span></span>](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json)
- [<span data-ttu-id="f4992-133">Azure Database for MySQL</span><span class="sxs-lookup"><span data-stu-id="f4992-133">Azure Database for MySQL</span></span>](/azure/mysql/)
- [<span data-ttu-id="f4992-134">Azure Database for PostgreSQL</span><span class="sxs-lookup"><span data-stu-id="f4992-134">Azure Database for PostgreSQL</span></span>](/azure/postgresql/)

<span data-ttu-id="f4992-135">有关详细信息，请参阅[选择 OLTP 数据存储](../technology-choices/oltp-data-stores.md)</span><span class="sxs-lookup"><span data-stu-id="f4992-135">For more information, see [Choosing an OLTP data store](../technology-choices/oltp-data-stores.md)</span></span>

<span data-ttu-id="f4992-136">数据源：</span><span class="sxs-lookup"><span data-stu-id="f4992-136">Data sources:</span></span>

- [<span data-ttu-id="f4992-137">应用服务</span><span class="sxs-lookup"><span data-stu-id="f4992-137">App service</span></span>](/azure/app-service/)
- [<span data-ttu-id="f4992-138">移动应用</span><span class="sxs-lookup"><span data-stu-id="f4992-138">Mobile Apps</span></span>](/azure/app-service-mobile/)

