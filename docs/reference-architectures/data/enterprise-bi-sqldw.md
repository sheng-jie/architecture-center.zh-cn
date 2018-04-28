---
title: 将 Enterprise BI 与 SQL 数据仓库配合使用
description: 使用 Azure 从本地存储的关系数据获取业务见解
author: alexbuckgit
ms.date: 04/13/2018
ms.openlocfilehash: b5e5aa32fc9cc8c7b8b5a42c9a4fc3e0216b2f72
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/16/2018
---
# <a name="enterprise-bi-with-sql-data-warehouse"></a><span data-ttu-id="a23ee-103">将 Enterprise BI 与 SQL 数据仓库配合使用</span><span class="sxs-lookup"><span data-stu-id="a23ee-103">Enterprise BI with SQL Data Warehouse</span></span>
 
<span data-ttu-id="a23ee-104">此参考体系结构实现 [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt)（提取-加载-转换）管道，该管道可将数据从本地 SQL Server 数据库移到 SQL 数据仓库，并转换数据以进行分析。</span><span class="sxs-lookup"><span data-stu-id="a23ee-104">This reference architecture implements an [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) (extract-load-transform) pipeline that moves data from an on-premises SQL Server database into SQL Data Warehouse and transforms the data for analysis.</span></span> [<span data-ttu-id="a23ee-105">**部署此解决方案**。</span><span class="sxs-lookup"><span data-stu-id="a23ee-105">**Deploy this solution**.</span></span>](#deploy-the-solution)

![](./images/enterprise-bi-sqldw.png)

<span data-ttu-id="a23ee-106">**场景**：某个组织在本地的 SQL Server 数据库中存储了大型 OLTP 数据集。</span><span class="sxs-lookup"><span data-stu-id="a23ee-106">**Scenario**: An organization has a large OLTP data set stored in a SQL Server database on premises.</span></span> <span data-ttu-id="a23ee-107">该组织想要使用 SQL 数据仓库通过 Power BI 执行分析。</span><span class="sxs-lookup"><span data-stu-id="a23ee-107">The organization wants to use SQL Data Warehouse to perform analysis using Power BI.</span></span> 

<span data-ttu-id="a23ee-108">此参考体系结构是针对一次性或按需作业设计的。</span><span class="sxs-lookup"><span data-stu-id="a23ee-108">This reference architecture is designed for one-time or on-demand jobs.</span></span> <span data-ttu-id="a23ee-109">如果需要持续移动数据（每小时或每日），我们建议使用 Azure 数据工厂来定义自动化工作流。</span><span class="sxs-lookup"><span data-stu-id="a23ee-109">If you need to move data on a continuing basis (hourly or daily), we recommend using Azure Data Factory to define an automated workflow.</span></span>

## <a name="architecture"></a><span data-ttu-id="a23ee-110">体系结构</span><span class="sxs-lookup"><span data-stu-id="a23ee-110">Architecture</span></span>

<span data-ttu-id="a23ee-111">该体系结构包括以下组件。</span><span class="sxs-lookup"><span data-stu-id="a23ee-111">The architecture consists of the following components.</span></span>

<span data-ttu-id="a23ee-112">**SQL Server**。</span><span class="sxs-lookup"><span data-stu-id="a23ee-112">**SQL Server**.</span></span> <span data-ttu-id="a23ee-113">源数据位于本地的 SQL Server 数据库中。</span><span class="sxs-lookup"><span data-stu-id="a23ee-113">The source data is located in a SQL Server database on premises.</span></span> <span data-ttu-id="a23ee-114">为了模拟本地环境，此体系结构的部署脚本将在 Azure 中预配一个装有 SQL Server 的虚拟机。</span><span class="sxs-lookup"><span data-stu-id="a23ee-114">To simulate the on-premises environment, the deployment scripts for this architecture provision a virtual machine in Azure with SQL Server installed.</span></span> 

<span data-ttu-id="a23ee-115">**Blob 存储**。</span><span class="sxs-lookup"><span data-stu-id="a23ee-115">**Blob Storage**.</span></span> <span data-ttu-id="a23ee-116">Blob 存储用作临时区域，在将数据载入 SQL 数据仓库之前，会先将数据复制到该区域。</span><span class="sxs-lookup"><span data-stu-id="a23ee-116">Blob storage is used as a staging area to copy the data before loading it into SQL Data Warehouse.</span></span>

<span data-ttu-id="a23ee-117">**Azure SQL 数据仓库**。</span><span class="sxs-lookup"><span data-stu-id="a23ee-117">**Azure SQL Data Warehouse**.</span></span> <span data-ttu-id="a23ee-118">[SQL 数据仓库](/azure/sql-data-warehouse/)是分布式系统，旨在对大型数据执行分析。</span><span class="sxs-lookup"><span data-stu-id="a23ee-118">[SQL Data Warehouse](/azure/sql-data-warehouse/) is a distributed system designed to perform analytics on large data.</span></span> <span data-ttu-id="a23ee-119">它支持大规模并行处理 (MPP)，因此很适合用于运行高性能分析。</span><span class="sxs-lookup"><span data-stu-id="a23ee-119">It supports massive parallel processing (MPP), which makes it suitable for running high-performance analytics.</span></span> 

<span data-ttu-id="a23ee-120">**Azure Analysis Services**。</span><span class="sxs-lookup"><span data-stu-id="a23ee-120">**Azure Analysis Services**.</span></span> <span data-ttu-id="a23ee-121">[Analysis Services](/azure/analysis-services/) 是提供数据建模功能的完全托管服务。</span><span class="sxs-lookup"><span data-stu-id="a23ee-121">[Analysis Services](/azure/analysis-services/) is a fully managed service that provides data modeling capabilities.</span></span> <span data-ttu-id="a23ee-122">使用 Analysis Services 能够创建用户可查询的语义模型。</span><span class="sxs-lookup"><span data-stu-id="a23ee-122">Use Analysis Services to create a semantic model that users can query.</span></span> <span data-ttu-id="a23ee-123">Analysis Services 在 BI 仪表板场景中尤其有用。</span><span class="sxs-lookup"><span data-stu-id="a23ee-123">Analysis Services is especially useful in a BI dashboard scenario.</span></span> <span data-ttu-id="a23ee-124">在此体系结构中，Analysis Services 从数据仓库读取数据以处理语义模型，并有效地为仪表板查询提供服务。</span><span class="sxs-lookup"><span data-stu-id="a23ee-124">In this architecture, Analysis Services reads data from the data warehouse to process the semantic model, and efficiently serves dashboard queries.</span></span> <span data-ttu-id="a23ee-125">它还通过横向扩展副本来加快查询处理的速度，以支持弹性并发性。</span><span class="sxs-lookup"><span data-stu-id="a23ee-125">It also supports elastic concurrency, by scaling out replicas for faster query processing.</span></span>

<span data-ttu-id="a23ee-126">目前，Azure Analysis Services 支持表格模型，但不支持多维模型。</span><span class="sxs-lookup"><span data-stu-id="a23ee-126">Currently, Azure Analysis Services supports tabular models but not multidimensional models.</span></span> <span data-ttu-id="a23ee-127">表格模型使用关系建模构造（表和列），而多维模型使用 OLAP 建模构造（多维数据集、维度和度量值）。</span><span class="sxs-lookup"><span data-stu-id="a23ee-127">Tabular models use relational modeling constructs (tables and columns), whereas multidimensional models use OLAP modeling constructs (cubes, dimensions, and measures).</span></span> <span data-ttu-id="a23ee-128">如果需要多维模型，请使用 SQL Server Analysis Services (SSAS)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-128">If you require multidimensional models, use SQL Server Analysis Services (SSAS).</span></span> <span data-ttu-id="a23ee-129">有关详细信息，请参阅[表格和多维解决方案的比较](/sql/analysis-services/comparing-tabular-and-multidimensional-solutions-ssas)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-129">For more information, see [Comparing tabular and multidimensional solutions](/sql/analysis-services/comparing-tabular-and-multidimensional-solutions-ssas).</span></span>

<span data-ttu-id="a23ee-130">**Power BI**。</span><span class="sxs-lookup"><span data-stu-id="a23ee-130">**Power BI**.</span></span> <span data-ttu-id="a23ee-131">Power BI 是一套商业分析工具，用于分析数据以获取商业见解。</span><span class="sxs-lookup"><span data-stu-id="a23ee-131">Power BI is a suite of business analytics tools to analyze data for business insights.</span></span> <span data-ttu-id="a23ee-132">在此体系结构中，Power BI 查询 Analysis Services 中存储的语义模型。</span><span class="sxs-lookup"><span data-stu-id="a23ee-132">In this architecture, it queries the semantic model stored in Analysis Services.</span></span>

<span data-ttu-id="a23ee-133">**Azure Active Directory** (Azure AD) 通过 Power BI 对连接到 Analysis Services 服务器的用户进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="a23ee-133">**Azure Active Directory** (Azure AD) authenticates users who connect to the Analysis Services server through Power BI.</span></span>

## <a name="data-pipeline"></a><span data-ttu-id="a23ee-134">Data Pipeline</span><span class="sxs-lookup"><span data-stu-id="a23ee-134">Data Pipeline</span></span>
 
<span data-ttu-id="a23ee-135">此参考体系结构使用 [WorldWideImporters](/sql/sample/world-wide-importers/wide-world-importers-oltp-database) 示例数据库作为数据源。</span><span class="sxs-lookup"><span data-stu-id="a23ee-135">This reference architecture uses the [WorldWideImporters](/sql/sample/world-wide-importers/wide-world-importers-oltp-database) sample database as data source.</span></span> <span data-ttu-id="a23ee-136">数据管道具有以下阶段：</span><span class="sxs-lookup"><span data-stu-id="a23ee-136">The data pipeline has the following stages:</span></span>

1. <span data-ttu-id="a23ee-137">将数据从 SQL Server 导出到平面文件（bcp 实用工具）。</span><span class="sxs-lookup"><span data-stu-id="a23ee-137">Export the data from SQL Server to flat files (bcp utility).</span></span>
2. <span data-ttu-id="a23ee-138">将平面文件复制到 Azure Blob 存储 (AzCopy)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-138">Copy the flat files to Azure Blob Storage (AzCopy).</span></span>
3. <span data-ttu-id="a23ee-139">将数据载入 SQL 数据仓库 (PolyBase)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-139">Load the data into SQL Data Warehouse (PolyBase).</span></span>
4. <span data-ttu-id="a23ee-140">将数据转换为星型架构 (T-SQL)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-140">Transform the data into a star schema (T-SQL).</span></span>
5. <span data-ttu-id="a23ee-141">将语义模型载入 Analysis Services (SQL Server Data Tools)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-141">Load a semantic model into Analysis Services (SQL Server Data Tools).</span></span>

![](./images/enterprise-bi-sqldw-pipeline.png)
 
> [!NOTE]
> <span data-ttu-id="a23ee-142">对于步骤 1 &ndash; 3，请考虑使用 Redgate Data Platform Studio。</span><span class="sxs-lookup"><span data-stu-id="a23ee-142">For steps 1 &ndash; 3, consider using Redgate Data Platform Studio.</span></span> <span data-ttu-id="a23ee-143">Data Platform Studio 应用了适当的兼容性修补程序和优化措施，可以快速启动 SQL 数据仓库操作。</span><span class="sxs-lookup"><span data-stu-id="a23ee-143">Data Platform Studio applies the most appropriate compatibility fixes and optimizations, so it's the quickest way to get started with SQL Data Warehouse.</span></span> <span data-ttu-id="a23ee-144">有关详细信息，请参阅[使用 Redgate Data Platform Studio 加载数据](/azure/sql-data-warehouse/sql-data-warehouse-load-with-redgate)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-144">For more information, see [Load data with Redgate Data Platform Studio](/azure/sql-data-warehouse/sql-data-warehouse-load-with-redgate).</span></span> 

<span data-ttu-id="a23ee-145">后续部分将会更详细地介绍这些阶段。</span><span class="sxs-lookup"><span data-stu-id="a23ee-145">The next sections describe these stages in more detail.</span></span>

### <a name="export-data-from-sql-server"></a><span data-ttu-id="a23ee-146">从 SQL Server 导出数据</span><span class="sxs-lookup"><span data-stu-id="a23ee-146">Export data from SQL Server</span></span>

<span data-ttu-id="a23ee-147">使用 [bcp](/sql/tools/bcp-utility)（批量复制程序）实用工具可以从 SQL 表快速创建平面文本文件。</span><span class="sxs-lookup"><span data-stu-id="a23ee-147">The [bcp](/sql/tools/bcp-utility) (bulk copy program) utility is a fast way to create flat text files from SQL tables.</span></span> <span data-ttu-id="a23ee-148">在此步骤中，请选择想要导出的列，但不要转换数据。</span><span class="sxs-lookup"><span data-stu-id="a23ee-148">In this step, you select the columns that you want to export, but don't transform the data.</span></span> <span data-ttu-id="a23ee-149">所有数据转换应在 SQL 数据仓库中进行。</span><span class="sxs-lookup"><span data-stu-id="a23ee-149">Any data transformations should happen in SQL Data Warehouse.</span></span>

<span data-ttu-id="a23ee-150">**建议**</span><span class="sxs-lookup"><span data-stu-id="a23ee-150">**Recommendations**</span></span>

<span data-ttu-id="a23ee-151">请尽量将数据提取安排在非高峰期，以最大程度地减少生产环境中的资源争用情况。</span><span class="sxs-lookup"><span data-stu-id="a23ee-151">If possible, schedule data extraction during off-peak hours, to minimize resource contention in the production environment.</span></span> 

<span data-ttu-id="a23ee-152">避免在数据库服务器上运行 bcp，</span><span class="sxs-lookup"><span data-stu-id="a23ee-152">Avoid running bcp on the database server.</span></span> <span data-ttu-id="a23ee-153">应该从另一台计算机运行 bcp。</span><span class="sxs-lookup"><span data-stu-id="a23ee-153">Instead, run it from another machine.</span></span> <span data-ttu-id="a23ee-154">将文件写入本地驱动器。</span><span class="sxs-lookup"><span data-stu-id="a23ee-154">Write the files to a local drive.</span></span> <span data-ttu-id="a23ee-155">确保有足够的 I/O 资源用于处理并发写入。</span><span class="sxs-lookup"><span data-stu-id="a23ee-155">Ensure that you have sufficient I/O resources to handle the concurrent writes.</span></span> <span data-ttu-id="a23ee-156">为获得最佳性能，请将文件导出到专用的高速存储驱动器。</span><span class="sxs-lookup"><span data-stu-id="a23ee-156">For best performance, export the files to dedicated fast storage drives.</span></span>

<span data-ttu-id="a23ee-157">以 Gzip 压缩格式保存导出的数据可以加速网络传输。</span><span class="sxs-lookup"><span data-stu-id="a23ee-157">You can speed up the network transfer by saving the exported data in Gzip compressed format.</span></span> <span data-ttu-id="a23ee-158">但是，将压缩文件载入仓库的速度比加载非压缩文件要慢，因此，加快网络传输速度与加快加载速度之间各有利弊。</span><span class="sxs-lookup"><span data-stu-id="a23ee-158">However, loading compressed files into the warehouse is slower than loading uncompressed files, so there is a tradeoff between faster network transfer versus faster loading.</span></span> <span data-ttu-id="a23ee-159">如果决定使用 Gzip 压缩，请不要创建单个 Gzip 文件，</span><span class="sxs-lookup"><span data-stu-id="a23ee-159">If you decide to use Gzip compression, don't create a single Gzip file.</span></span> <span data-ttu-id="a23ee-160">而是将数据拆分为多个压缩文件。</span><span class="sxs-lookup"><span data-stu-id="a23ee-160">Instead, split the data into multiple compressed files.</span></span>

### <a name="copy-flat-files-into-blob-storage"></a><span data-ttu-id="a23ee-161">将平面文件复制到 Blob 存储中</span><span class="sxs-lookup"><span data-stu-id="a23ee-161">Copy flat files into blob storage</span></span>

<span data-ttu-id="a23ee-162">[AzCopy](/azure/storage/common/storage-use-azcopy) 实用工具旨在以较高的性能将数据复制到 Azure Blob 存储。</span><span class="sxs-lookup"><span data-stu-id="a23ee-162">The [AzCopy](/azure/storage/common/storage-use-azcopy) utility is designed for high-performance copying of data into Azure blob storage.</span></span>

<span data-ttu-id="a23ee-163">**建议**</span><span class="sxs-lookup"><span data-stu-id="a23ee-163">**Recommendations**</span></span>

<span data-ttu-id="a23ee-164">在靠近源数据位置的区域中创建存储帐户。</span><span class="sxs-lookup"><span data-stu-id="a23ee-164">Create the storage account in a region near the location of the source data.</span></span> <span data-ttu-id="a23ee-165">在同一区域中部署存储帐户和 SQL 数据仓库实例。</span><span class="sxs-lookup"><span data-stu-id="a23ee-165">Deploy the storage account and the SQL Data Warehouse instance in the same region.</span></span> 

<span data-ttu-id="a23ee-166">不要在运行生产工作负荷的同一台计算机上运行 AzCopy，因为 CPU 和 I/O 消耗可能会干扰生产工作负荷。</span><span class="sxs-lookup"><span data-stu-id="a23ee-166">Don't run AzCopy on the same machine that runs your production workloads, because the CPU and I/O consumption can interfere with the production workload.</span></span> 

<span data-ttu-id="a23ee-167">首先测试上传，以确定大致的上传速度。</span><span class="sxs-lookup"><span data-stu-id="a23ee-167">Test the upload first to see what the upload speed is like.</span></span> <span data-ttu-id="a23ee-168">可以在 AzCopy 中使用 /NC 选项来指定并发复制操作的数目。</span><span class="sxs-lookup"><span data-stu-id="a23ee-168">You can use the /NC option in AzCopy to specify the number of concurrent copy operations.</span></span> <span data-ttu-id="a23ee-169">使用默认值启动，然后使用此设置进行试验，以优化性能。</span><span class="sxs-lookup"><span data-stu-id="a23ee-169">Start with the default value, then experiment with this setting to tune the performance.</span></span> <span data-ttu-id="a23ee-170">在低带宽环境中，过多的并发操作可能会使网络连接瘫痪，并导致操作无法彻底完成。</span><span class="sxs-lookup"><span data-stu-id="a23ee-170">In a low-bandwidth environment, too many concurrent operations can overwhelm the network connection and prevent the operations from completing successfully.</span></span>  

<span data-ttu-id="a23ee-171">AzCopy 通过公共 Internet 将数据移到存储中。</span><span class="sxs-lookup"><span data-stu-id="a23ee-171">AzCopy moves data to storage over the public internet.</span></span> <span data-ttu-id="a23ee-172">如果速度不够快，请考虑设置 [ExpressRoute](/azure/expressroute/) 线路。</span><span class="sxs-lookup"><span data-stu-id="a23ee-172">If this isn't fast enough, consider setting up an [ExpressRoute](/azure/expressroute/) circuit.</span></span> <span data-ttu-id="a23ee-173">ExpressRoute 是通过专用连接将数据路由到 Azure 的服务。</span><span class="sxs-lookup"><span data-stu-id="a23ee-173">ExpressRoute is a service that routes your data through a dedicated private connection to Azure.</span></span> <span data-ttu-id="a23ee-174">如果网络连接速度太慢，可以采用另一种做法，即以物理方式将磁盘上的数据传送到 Azure 数据中心。</span><span class="sxs-lookup"><span data-stu-id="a23ee-174">Another option, if your network connection is too slow, is to physically ship the data on disk to an Azure datacenter.</span></span> <span data-ttu-id="a23ee-175">有关详细信息，请参阅[将数据传入和传出 Azure](/azure/architecture/data-guide/scenarios/data-transfer)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-175">For more information, see [Transferring data to and from Azure](/azure/architecture/data-guide/scenarios/data-transfer).</span></span>

<span data-ttu-id="a23ee-176">在执行复制操作期间，AzCopy 将创建一个临时日记文件，使 AzCopy 在操作中断（例如，由于网络错误）时重启操作。</span><span class="sxs-lookup"><span data-stu-id="a23ee-176">During a copy operation, AzCopy creates a temporary journal file, which enables AzCopy to restart the operation if it gets interrupted (for example, due to a network error).</span></span> <span data-ttu-id="a23ee-177">确保有足够的磁盘空间用于存储日记文件。</span><span class="sxs-lookup"><span data-stu-id="a23ee-177">Make sure there is enough disk space to store the journal files.</span></span> <span data-ttu-id="a23ee-178">可以使用 /Z 选项指定日记文件的写入位置。</span><span class="sxs-lookup"><span data-stu-id="a23ee-178">You can use the /Z option to specify where the journal files are written.</span></span>

### <a name="load-data-into-sql-data-warehouse"></a><span data-ttu-id="a23ee-179">将数据载入 SQL 数据仓库</span><span class="sxs-lookup"><span data-stu-id="a23ee-179">Load data into SQL Data Warehouse</span></span>

<span data-ttu-id="a23ee-180">使用 [PolyBase](/sql/relational-databases/polybase/polybase-guide) 将文件从 Blob 存储载入数据仓库。</span><span class="sxs-lookup"><span data-stu-id="a23ee-180">Use [PolyBase](/sql/relational-databases/polybase/polybase-guide) to load the files from blob storage into the data warehouse.</span></span> <span data-ttu-id="a23ee-181">PolyBase 利用 SQL 数据仓库的 MPP（大规模并行处理）体系结构，因此是将数据载入 Azure SQL 数据仓库的最快方式。</span><span class="sxs-lookup"><span data-stu-id="a23ee-181">PolyBase is designed to leverage the MPP (Massively Parallel Processing) architecture of SQL Data Warehouse, which makes it the fastest way to load data into SQL Data Warehouse.</span></span> 

<span data-ttu-id="a23ee-182">加载数据的过程包括两个步骤：</span><span class="sxs-lookup"><span data-stu-id="a23ee-182">Loading the data is a two-step process:</span></span>

1. <span data-ttu-id="a23ee-183">为数据创建一组外部表。</span><span class="sxs-lookup"><span data-stu-id="a23ee-183">Create a set of external tables for the data.</span></span> <span data-ttu-id="a23ee-184">外部表是指向仓库外部存储的数据（在本例中，为 Blob 存储中的平面文件）的表定义。</span><span class="sxs-lookup"><span data-stu-id="a23ee-184">An external table is a table definition that points to data stored outside of the warehouse &mdash; in this case, the flat files in blob storage.</span></span> <span data-ttu-id="a23ee-185">此步骤不会将任何数据移入仓库。</span><span class="sxs-lookup"><span data-stu-id="a23ee-185">This step does not move any data into the warehouse.</span></span>
2. <span data-ttu-id="a23ee-186">创建临时表，并将数据载入临时表。</span><span class="sxs-lookup"><span data-stu-id="a23ee-186">Create staging tables, and load the data into the staging tables.</span></span> <span data-ttu-id="a23ee-187">此步骤将数据复制到仓库中。</span><span class="sxs-lookup"><span data-stu-id="a23ee-187">This step copies the data into the warehouse.</span></span>

<span data-ttu-id="a23ee-188">**建议**</span><span class="sxs-lookup"><span data-stu-id="a23ee-188">**Recommendations**</span></span>

<span data-ttu-id="a23ee-189">如果有大量的数据（超过 1 TB），并在运行受益于并行度的分析工作负荷，请考虑使用 SQL 数据仓库。</span><span class="sxs-lookup"><span data-stu-id="a23ee-189">Consider SQL Data Warehouse when you have large amounts of data (more than 1 TB) and are running an analytics workload that will benefit from parallelism.</span></span> <span data-ttu-id="a23ee-190">SQL 数据仓库并不很适合 OLTP 工作负荷或较小的数据集（小于 250 GB）。</span><span class="sxs-lookup"><span data-stu-id="a23ee-190">SQL Data Warehouse is not a good fit for OLTP workloads or smaller data sets (< 250GB).</span></span> <span data-ttu-id="a23ee-191">对于小于 250 GB 的数据集，请考虑使用 Azure SQL 数据库或 SQL Server。</span><span class="sxs-lookup"><span data-stu-id="a23ee-191">For data sets less than 250GB, consider Azure SQL Database or SQL Server.</span></span> <span data-ttu-id="a23ee-192">有关详细信息，请参阅[数据仓库](../../data-guide/relational-data/data-warehousing.md)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-192">For more information, see [Data warehousing](../../data-guide/relational-data/data-warehousing.md).</span></span>

<span data-ttu-id="a23ee-193">将临时表创建为未编制索引的堆表。</span><span class="sxs-lookup"><span data-stu-id="a23ee-193">Create the staging tables as heap tables, which are not indexed.</span></span> <span data-ttu-id="a23ee-194">创建生产表的查询将导致全表扫描，因此没有理由为临时表编制索引。</span><span class="sxs-lookup"><span data-stu-id="a23ee-194">The queries that create the production tables will result in a full table scan, so there is no reason to index the staging tables.</span></span>

<span data-ttu-id="a23ee-195">PolyBase 自动利用仓库中的并行度。</span><span class="sxs-lookup"><span data-stu-id="a23ee-195">PolyBase automatically takes advantage of parallelism in the warehouse.</span></span> <span data-ttu-id="a23ee-196">负载性能会随着 DWU 的增加而扩展。</span><span class="sxs-lookup"><span data-stu-id="a23ee-196">The load performance scales as you increase DWUs.</span></span> <span data-ttu-id="a23ee-197">为获得最佳性能，请使用单个加载操作。</span><span class="sxs-lookup"><span data-stu-id="a23ee-197">For best performance, use a single load operation.</span></span> <span data-ttu-id="a23ee-198">将输入数据分解为区块和运行多个并发负载不会带来任何性能优势。</span><span class="sxs-lookup"><span data-stu-id="a23ee-198">There is no performance benefit to breaking the input data into chunks and running multiple concurrent loads.</span></span>

<span data-ttu-id="a23ee-199">PolyBase 可以读取 Gzip 压缩文件。</span><span class="sxs-lookup"><span data-stu-id="a23ee-199">PolyBase can read Gzip compressed files.</span></span> <span data-ttu-id="a23ee-200">但是，只会对每个压缩文件使用单个读取器，因为解压缩文件是单线程操作。</span><span class="sxs-lookup"><span data-stu-id="a23ee-200">However, only a single reader is used per compressed file, because uncompressing the file is a single-threaded operation.</span></span> <span data-ttu-id="a23ee-201">因此，请避免加载单个大型压缩文件。</span><span class="sxs-lookup"><span data-stu-id="a23ee-201">Therefore, avoid loading a single large compressed file.</span></span> <span data-ttu-id="a23ee-202">应该将数据拆分为多个压缩文件，以利用并行度。</span><span class="sxs-lookup"><span data-stu-id="a23ee-202">Instead, split the data into multiple compressed files, in order to take advantage of parallelism.</span></span> 

<span data-ttu-id="a23ee-203">注意以下限制：</span><span class="sxs-lookup"><span data-stu-id="a23ee-203">Be aware of the following limitations:</span></span>

- <span data-ttu-id="a23ee-204">PolyBase 支持的最大列大小为 `varchar(8000)`、`nvarchar(4000)` 或 `varbinary(8000)`。</span><span class="sxs-lookup"><span data-stu-id="a23ee-204">PolyBase supports a maximum column size of `varchar(8000)`, `nvarchar(4000)`, or `varbinary(8000)`.</span></span> <span data-ttu-id="a23ee-205">如果数据超过这些限制，一种做法是在导出数据时将数据分解为区块，然后在导入后重新汇编区块。</span><span class="sxs-lookup"><span data-stu-id="a23ee-205">If you have data that exceeds these limits, one option is to break the data up into chunks when you export it, and then reassemble the chunks after import.</span></span> 

- <span data-ttu-id="a23ee-206">PolyBase 使用固定行终止符 \n 或换行符。</span><span class="sxs-lookup"><span data-stu-id="a23ee-206">PolyBase uses a fixed row terminator of \n or newline.</span></span> <span data-ttu-id="a23ee-207">如果源数据中显示换行符，则可能会导致问题。</span><span class="sxs-lookup"><span data-stu-id="a23ee-207">This can cause problems if newline characters appear in the source data.</span></span>

- <span data-ttu-id="a23ee-208">源数据架构可能包含 SQL 数据仓库中不支持的数据类型。</span><span class="sxs-lookup"><span data-stu-id="a23ee-208">Your source data schema might contain data types that are not supported in SQL Data Warehouse.</span></span>

<span data-ttu-id="a23ee-209">若要解决这些限制，可以创建一个执行所需转换的存储过程。</span><span class="sxs-lookup"><span data-stu-id="a23ee-209">To work around these limitations, you can create a stored procedure that performs the necessary conversions.</span></span> <span data-ttu-id="a23ee-210">运行 bcp 时引用此存储过程。</span><span class="sxs-lookup"><span data-stu-id="a23ee-210">Reference this stored procedure when you run bcp.</span></span> <span data-ttu-id="a23ee-211">或者，使用 [Redgate Data Platform Studio](/azure/sql-data-warehouse/sql-data-warehouse-load-with-redgate) 自动转换 SQL 数据仓库中不支持的数据类型。</span><span class="sxs-lookup"><span data-stu-id="a23ee-211">Alternatively, [Redgate Data Platform Studio](/azure/sql-data-warehouse/sql-data-warehouse-load-with-redgate) automatically converts data types that aren’t supported in SQL Data Warehouse.</span></span>

<span data-ttu-id="a23ee-212">有关详细信息，请参阅以下文章：</span><span class="sxs-lookup"><span data-stu-id="a23ee-212">For more information, see the following articles:</span></span>

- <span data-ttu-id="a23ee-213">[将数据载入 Azure SQL 数据仓库的最佳做法](/azure/sql-data-warehouse/guidance-for-loading-data)</span><span class="sxs-lookup"><span data-stu-id="a23ee-213">[Best practices for loading data into Azure SQL Data Warehouse](/azure/sql-data-warehouse/guidance-for-loading-data).</span></span>
- [<span data-ttu-id="a23ee-214">将架构迁移到 SQL 数据仓库</span><span class="sxs-lookup"><span data-stu-id="a23ee-214">Migrate your schemas to SQL Data Warehouse</span></span>](/azure/sql-data-warehouse/sql-data-warehouse-migrate-schema)
- [<span data-ttu-id="a23ee-215">有关为 SQL 数据仓库中的表定义数据类型的指南</span><span class="sxs-lookup"><span data-stu-id="a23ee-215">Guidance for defining data types for tables in SQL Data Warehouse</span></span>](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types)

### <a name="transform-the-data"></a><span data-ttu-id="a23ee-216">转换数据</span><span class="sxs-lookup"><span data-stu-id="a23ee-216">Transform the data</span></span>

<span data-ttu-id="a23ee-217">转换数据，并将其移入生产表。</span><span class="sxs-lookup"><span data-stu-id="a23ee-217">Transform the data and move it into production tables.</span></span> <span data-ttu-id="a23ee-218">在此步骤中，数据将转换为包含维度表和事实数据表且适用于语义建模的星型架构。</span><span class="sxs-lookup"><span data-stu-id="a23ee-218">In this step, the data is transformed into a star schema with dimension tables and fact tables, suitable for semantic modeling.</span></span>

<span data-ttu-id="a23ee-219">创建具有聚集列存储索引并提供最佳整体查询性能的生产表。</span><span class="sxs-lookup"><span data-stu-id="a23ee-219">Create the production tables with clustered columnstore indexes, which offer the best overall query performance.</span></span> <span data-ttu-id="a23ee-220">列存储索引已针对扫描大量记录的查询进行优化。</span><span class="sxs-lookup"><span data-stu-id="a23ee-220">Columnstore indexes are optimized for queries that scan many records.</span></span> <span data-ttu-id="a23ee-221">列存储索引并不是很适合用于单一实例查找（即，查找单个行）。</span><span class="sxs-lookup"><span data-stu-id="a23ee-221">Columnstore indexes don't perform as well for singleton lookups (that is, looking up a single row).</span></span> <span data-ttu-id="a23ee-222">如果需要执行频繁的单一实例查找，可以在表中添加非聚集索引。</span><span class="sxs-lookup"><span data-stu-id="a23ee-222">If you need to perform frequent singleton lookups, you can add a non-clustered index to a table.</span></span> <span data-ttu-id="a23ee-223">使用非聚集索引能够明显加快单一实例查找的运行速度。</span><span class="sxs-lookup"><span data-stu-id="a23ee-223">Singleton lookups can run significantly faster using a non-clustered index.</span></span> <span data-ttu-id="a23ee-224">但是，与在 OLTP 工作负荷中相比，单一实例查找在数据仓库场景中通常不太常见。</span><span class="sxs-lookup"><span data-stu-id="a23ee-224">However, singleton lookups are typically less common in data warehouse scenarios than OLTP workloads.</span></span> <span data-ttu-id="a23ee-225">有关详细信息，请参阅[为 SQL 数据仓库中的表编制索引](/azure/sql-data-warehouse/sql-data-warehouse-tables-index)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-225">For more information, see [Indexing tables in SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-tables-index).</span></span>

> [!NOTE]
> <span data-ttu-id="a23ee-226">聚集列存储表不支持 `varchar(max)`、`nvarchar(max)` 或 `varbinary(max)` 数据类型。</span><span class="sxs-lookup"><span data-stu-id="a23ee-226">Clustered columnstore tables do not support `varchar(max)`, `nvarchar(max)`, or `varbinary(max)` data types.</span></span> <span data-ttu-id="a23ee-227">在这种情况下，请考虑堆或聚集索引。</span><span class="sxs-lookup"><span data-stu-id="a23ee-227">In that case, consider a heap or clustered index.</span></span> <span data-ttu-id="a23ee-228">可将这些列放入单独的表中。</span><span class="sxs-lookup"><span data-stu-id="a23ee-228">You might put those columns into a separate table.</span></span>

<span data-ttu-id="a23ee-229">由于示例数据库不是很大，因此我们创建了不带分区的复制表。</span><span class="sxs-lookup"><span data-stu-id="a23ee-229">Because the sample database is not very large, we created replicated tables with no partitions.</span></span> <span data-ttu-id="a23ee-230">对于生产工作负荷，使用分布式表可能会提高查询性能。</span><span class="sxs-lookup"><span data-stu-id="a23ee-230">For production workloads, using distributed tables is likely to improve query performance.</span></span> <span data-ttu-id="a23ee-231">请参阅[有关在 Azure SQL 数据仓库中设计分布式表的指南](/azure/sql-data-warehouse/sql-data-warehouse-tables-distribute)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-231">See [Guidance for designing distributed tables in Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-tables-distribute).</span></span> <span data-ttu-id="a23ee-232">示例脚本使用静态[资源类](/azure/sql-data-warehouse/resource-classes-for-workload-management)运行查询。</span><span class="sxs-lookup"><span data-stu-id="a23ee-232">Our example scripts run the queries using a static [resource class](/azure/sql-data-warehouse/resource-classes-for-workload-management).</span></span>

### <a name="load-the-semantic-model"></a><span data-ttu-id="a23ee-233">加载语义模型</span><span class="sxs-lookup"><span data-stu-id="a23ee-233">Load the semantic model</span></span>

<span data-ttu-id="a23ee-234">将数据载入 Azure Analysis Services 中的表格模型。</span><span class="sxs-lookup"><span data-stu-id="a23ee-234">Load the data into a tabular model in Azure Analysis Services.</span></span> <span data-ttu-id="a23ee-235">在此步骤中，我们将使用 SQL Server Data Tools (SSDT) 创建语义数据模型。</span><span class="sxs-lookup"><span data-stu-id="a23ee-235">In this step, you create a semantic data model by using SQL Server Data Tools (SSDT).</span></span> <span data-ttu-id="a23ee-236">也可以通过从 Power BI Desktop 文件导入一个模型来创建该模型。</span><span class="sxs-lookup"><span data-stu-id="a23ee-236">You can also create a model by importing it from a Power BI Desktop file.</span></span> <span data-ttu-id="a23ee-237">由于 SQL 数据仓库不支持外键，因为必须将关系添加到语义模型，以便可以跨表联接。</span><span class="sxs-lookup"><span data-stu-id="a23ee-237">Because SQL Data Warehouse does not support foreign keys, you must add the relationships to the semantic model, so that you can join across tables.</span></span>

### <a name="use-power-bi-to-visualize-the-data"></a><span data-ttu-id="a23ee-238">使用 Power BI 将数据可视化</span><span class="sxs-lookup"><span data-stu-id="a23ee-238">Use Power BI to visualize the data</span></span>

<span data-ttu-id="a23ee-239">Power BI 支持使用两个选项连接到 Azure Analysis Services：</span><span class="sxs-lookup"><span data-stu-id="a23ee-239">Power BI supports two options for connecting to Azure Analysis Services:</span></span>

- <span data-ttu-id="a23ee-240">导入。</span><span class="sxs-lookup"><span data-stu-id="a23ee-240">Import.</span></span> <span data-ttu-id="a23ee-241">将数据导入 Power BI 模型。</span><span class="sxs-lookup"><span data-stu-id="a23ee-241">The data is imported into the Power BI model.</span></span>
- <span data-ttu-id="a23ee-242">实时连接。</span><span class="sxs-lookup"><span data-stu-id="a23ee-242">Live Connection.</span></span> <span data-ttu-id="a23ee-243">直接从 Analysis Services 提取数据。</span><span class="sxs-lookup"><span data-stu-id="a23ee-243">Data is pulled directly from Analysis Services.</span></span>

<span data-ttu-id="a23ee-244">我们建议使用“实时连接”，因为此选项不需要将数据复制到 Power BI 模型中。</span><span class="sxs-lookup"><span data-stu-id="a23ee-244">We recommend Live Connection because it doesn't require copying data into the Power BI model.</span></span> <span data-ttu-id="a23ee-245">此外，使用 DirectQuery 可确保结果始终与最新源数据保持一致。</span><span class="sxs-lookup"><span data-stu-id="a23ee-245">Also, using DirectQuery ensures that results are always consistent with the latest source data.</span></span> <span data-ttu-id="a23ee-246">有关详细信息，请参阅[使用 Power BI 进行连接](/azure/analysis-services/analysis-services-connect-pbi)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-246">For more information, see [Connect with Power BI](/azure/analysis-services/analysis-services-connect-pbi).</span></span>

<span data-ttu-id="a23ee-247">**建议**</span><span class="sxs-lookup"><span data-stu-id="a23ee-247">**Recommendations**</span></span>

<span data-ttu-id="a23ee-248">避免直接对数据仓库运行 BI 仪表板查询。</span><span class="sxs-lookup"><span data-stu-id="a23ee-248">Avoid running BI dashboard queries directly against the data warehouse.</span></span> <span data-ttu-id="a23ee-249">BI 仪表板要求的响应时间很短，针对仓库直接运行查询可能无法满足该要求。</span><span class="sxs-lookup"><span data-stu-id="a23ee-249">BI dashboards require very low response times, which direct queries against the warehouse may be unable to satisfy.</span></span> <span data-ttu-id="a23ee-250">此外，刷新仪表板的操作将计入并发查询次数，这可能会影响性能。</span><span class="sxs-lookup"><span data-stu-id="a23ee-250">Also, refreshing the dashboard will count against the number of concurrent queries, which could impact performance.</span></span> 

<span data-ttu-id="a23ee-251">Azure Analysis Services 旨在处理 BI 仪表板的查询要求，因此建议的做法是从 Power BI 查询 Analysis Services。</span><span class="sxs-lookup"><span data-stu-id="a23ee-251">Azure Analysis Services is designed to handle the query requirements of a BI dashboard, so the recommended practice is to query Analysis Services from Power BI.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="a23ee-252">可伸缩性注意事项</span><span class="sxs-lookup"><span data-stu-id="a23ee-252">Scalability Considerations</span></span>

### <a name="sql-data-warehouse"></a><span data-ttu-id="a23ee-253">SQL 数据仓库</span><span class="sxs-lookup"><span data-stu-id="a23ee-253">SQL Data Warehouse</span></span>

<span data-ttu-id="a23ee-254">使用 SQL 数据仓库可以按需横向扩展计算资源。</span><span class="sxs-lookup"><span data-stu-id="a23ee-254">With SQL Data Warehouse, you can scale out your compute resources on demand.</span></span> <span data-ttu-id="a23ee-255">查询引擎基于计算节点数目优化并行处理的查询，并按需在节点之间移动数据。</span><span class="sxs-lookup"><span data-stu-id="a23ee-255">The query engine optimizes queries for parallel processing based on the number of compute nodes, and moves data between nodes as necessary.</span></span> <span data-ttu-id="a23ee-256">有关详细信息，请参阅[管理 Azure SQL 数据仓库中的计算资源](/azure/sql-data-warehouse/sql-data-warehouse-manage-compute-overview)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-256">For more information, see [Manage compute in Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-manage-compute-overview).</span></span>

### <a name="analysis-services"></a><span data-ttu-id="a23ee-257">Analysis Services</span><span class="sxs-lookup"><span data-stu-id="a23ee-257">Analysis Services</span></span>

<span data-ttu-id="a23ee-258">对于生产工作负荷，我们建议使用 Azure Analysis Services 的标准层，因为该层支持分区和 DirectQuery。</span><span class="sxs-lookup"><span data-stu-id="a23ee-258">For production workloads, we recommend the Standard Tier for Azure Analysis Services, because it supports partitioning and DirectQuery.</span></span> <span data-ttu-id="a23ee-259">在层中，实例大小确定内存和处理能力。</span><span class="sxs-lookup"><span data-stu-id="a23ee-259">Within a tier, the instance size determines the memory and processing power.</span></span> <span data-ttu-id="a23ee-260">处理能力以查询处理单位 (QPU) 来计量。</span><span class="sxs-lookup"><span data-stu-id="a23ee-260">Processing power is measured in Query Processing Units (QPUs).</span></span> <span data-ttu-id="a23ee-261">请监视 QPU 使用率以选择适当的大小。</span><span class="sxs-lookup"><span data-stu-id="a23ee-261">Monitor your QPU usage to select the appropriate size.</span></span> <span data-ttu-id="a23ee-262">有关详细信息，请参阅[监视服务器指标](/azure/analysis-services/analysis-services-monitor)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-262">For more information, see [Monitor server metrics](/azure/analysis-services/analysis-services-monitor).</span></span>

<span data-ttu-id="a23ee-263">在负载较高的情况下，查询性能可能因查询并发性而下降。</span><span class="sxs-lookup"><span data-stu-id="a23ee-263">Under high load, query performance can become degraded due to query concurrency.</span></span> <span data-ttu-id="a23ee-264">可以通过创建副本池来处理查询，以横向扩展 Analysis Services，从而能够并行执行更多的查询。</span><span class="sxs-lookup"><span data-stu-id="a23ee-264">You can scale out Analysis Services by creating a pool of replicas to process queries, so that more queries can be performed concurrently.</span></span> <span data-ttu-id="a23ee-265">处理数据模型的工作始终在主服务器上进行。</span><span class="sxs-lookup"><span data-stu-id="a23ee-265">The work of processing the data model always happens on the primary server.</span></span> <span data-ttu-id="a23ee-266">默认情况下，主服务器也会处理查询。</span><span class="sxs-lookup"><span data-stu-id="a23ee-266">By default, the primary server also handles queries.</span></span> <span data-ttu-id="a23ee-267">（可选）可以指定以独占方式运行处理的主服务器，让查询池处理所有查询。</span><span class="sxs-lookup"><span data-stu-id="a23ee-267">Optionally, you can designate the primary server to run processing exclusively, so that the query pool handles all queries.</span></span> <span data-ttu-id="a23ee-268">如果处理要求较高，应将处理负载与查询池隔离开来。</span><span class="sxs-lookup"><span data-stu-id="a23ee-268">If you have high processing requirements, you should separate the processing from the query pool.</span></span> <span data-ttu-id="a23ee-269">如果查询负载较高，而处理负载相对较低，则可以在查询池中包含主服务器。</span><span class="sxs-lookup"><span data-stu-id="a23ee-269">If you have high query loads, and relatively light processing, you can include the primary server in the query pool.</span></span> <span data-ttu-id="a23ee-270">有关详细信息，请参阅 [Azure Analysis Services 横向扩展](/azure/analysis-services/analysis-services-scale-out)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-270">For more information, see [Azure Analysis Services scale-out](/azure/analysis-services/analysis-services-scale-out).</span></span> 

<span data-ttu-id="a23ee-271">若要减少不必要的处理量，请考虑使用分区将表格模型划分为逻辑部分。</span><span class="sxs-lookup"><span data-stu-id="a23ee-271">To reduce the amount of unnecessary processing, consider using partitions to divide the tabular model into logical parts.</span></span> <span data-ttu-id="a23ee-272">可以单独处理每个分区。</span><span class="sxs-lookup"><span data-stu-id="a23ee-272">Each partition can be processed separately.</span></span> <span data-ttu-id="a23ee-273">有关详细信息，请参阅[分区](/sql/analysis-services/tabular-models/partitions-ssas-tabular)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-273">For more information, see [Partitions](/sql/analysis-services/tabular-models/partitions-ssas-tabular).</span></span>

## <a name="security-considerations"></a><span data-ttu-id="a23ee-274">安全注意事项</span><span class="sxs-lookup"><span data-stu-id="a23ee-274">Security Considerations</span></span>

### <a name="ip-whitelisting-of-analysis-services-clients"></a><span data-ttu-id="a23ee-275">Analysis Services 客户端的 IP 白名单</span><span class="sxs-lookup"><span data-stu-id="a23ee-275">IP whitelisting of Analysis Services clients</span></span>

<span data-ttu-id="a23ee-276">考虑使用 Analysis Services 防火墙功能将客户端 IP 地址加入白名单。</span><span class="sxs-lookup"><span data-stu-id="a23ee-276">Consider using the Analysis Services firewall feature to whitelist client IP addresses.</span></span> <span data-ttu-id="a23ee-277">如果已启用防火墙，防火墙会阻止其规则中指定的连接以外的所有客户端连接。</span><span class="sxs-lookup"><span data-stu-id="a23ee-277">If enabled, the firewall blocks all client connections other than those specified in the firewall rules.</span></span> <span data-ttu-id="a23ee-278">默认规则会将 Power BI 服务加入白名单，但你可以根据需要禁用此规则。</span><span class="sxs-lookup"><span data-stu-id="a23ee-278">The default rules whitelist the Power BI service, but you can disable this rule if desired.</span></span> <span data-ttu-id="a23ee-279">有关详细信息，请参阅[使用新防火墙功能强化 Azure Analysis Services](https://azure.microsoft.com/blog/hardening-azure-analysis-services-with-the-new-firewall-capability/)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-279">For more information, see [Hardening Azure Analysis Services with the new firewall capability](https://azure.microsoft.com/blog/hardening-azure-analysis-services-with-the-new-firewall-capability/).</span></span>

### <a name="authorization"></a><span data-ttu-id="a23ee-280">授权</span><span class="sxs-lookup"><span data-stu-id="a23ee-280">Authorization</span></span>

<span data-ttu-id="a23ee-281">Azure Analysis Services 使用 Azure Active Directory (Azure AD) 对连接到 Analysis Services 服务器的用户进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="a23ee-281">Azure Analysis Services uses Azure Active Directory (Azure AD) to authenticate users who connect to an Analysis Services server.</span></span> <span data-ttu-id="a23ee-282">可以通过创建角色，然后将 Azure AD 用户或组分配到这些角色，来限制特定的用户可以查看哪些数据。</span><span class="sxs-lookup"><span data-stu-id="a23ee-282">You can restrict what data a particular user is able to view, by creating roles and then assigning Azure AD users or groups to those roles.</span></span> <span data-ttu-id="a23ee-283">对于每个角色，可以：</span><span class="sxs-lookup"><span data-stu-id="a23ee-283">For each role, you can:</span></span> 

- <span data-ttu-id="a23ee-284">保护表或单个列。</span><span class="sxs-lookup"><span data-stu-id="a23ee-284">Protect tables or individual columns.</span></span> 
- <span data-ttu-id="a23ee-285">基于筛选表达式保护单个行。</span><span class="sxs-lookup"><span data-stu-id="a23ee-285">Protect individual rows based on filter expressions.</span></span> 

<span data-ttu-id="a23ee-286">有关详细信息，请参阅[管理数据库角色和用户](/azure/analysis-services/analysis-services-database-users)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-286">For more information, see [Manage database roles and users](/azure/analysis-services/analysis-services-database-users).</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="a23ee-287">部署解决方案</span><span class="sxs-lookup"><span data-stu-id="a23ee-287">Deploy the solution</span></span>

<span data-ttu-id="a23ee-288">[GitHub][ref-arch-repo-folder] 中提供了此参考体系结构的部署。</span><span class="sxs-lookup"><span data-stu-id="a23ee-288">A deployment for this reference architecture is available on [GitHub][ref-arch-repo-folder].</span></span> <span data-ttu-id="a23ee-289">它将部署以下部分：</span><span class="sxs-lookup"><span data-stu-id="a23ee-289">It deploys the following:</span></span>

  * <span data-ttu-id="a23ee-290">一个用于模拟本地数据库服务器的 Windows VM。</span><span class="sxs-lookup"><span data-stu-id="a23ee-290">A Windows VM to simulate an on-premises database server.</span></span> <span data-ttu-id="a23ee-291">该 VM 包含 SQL Server 2017 和相关工具以及 Power BI Desktop。</span><span class="sxs-lookup"><span data-stu-id="a23ee-291">It includes SQL Server 2017 and related tools, along with Power BI Desktop.</span></span>
  * <span data-ttu-id="a23ee-292">一个 Azure 存储帐户。该帐户提供 Blob 存储用于保存从 SQL Server 数据库导出的数据。</span><span class="sxs-lookup"><span data-stu-id="a23ee-292">An Azure storage account that provides Blob storage to hold data exported from the SQL Server database.</span></span>
  * <span data-ttu-id="a23ee-293">一个 Azure SQL 数据仓库实例。</span><span class="sxs-lookup"><span data-stu-id="a23ee-293">An Azure SQL Data Warehouse instance.</span></span>
  * <span data-ttu-id="a23ee-294">一个 Azure Analysis Services 实例。</span><span class="sxs-lookup"><span data-stu-id="a23ee-294">An Azure Analysis Services instance.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="a23ee-295">先决条件</span><span class="sxs-lookup"><span data-stu-id="a23ee-295">Prerequisites</span></span>

1. <span data-ttu-id="a23ee-296">克隆、分叉或下载 [Azure 参考体系结构][ref-arch-repo] GitHub 存储库的 zip 文件。</span><span class="sxs-lookup"><span data-stu-id="a23ee-296">Clone, fork, or download the zip file for the [Azure reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="a23ee-297">安装 [Azure 构建基块][azbb-wiki] (azbb)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-297">Install the [Azure Building Blocks][azbb-wiki] (azbb).</span></span>

3. <span data-ttu-id="a23ee-298">在命令提示符、bash 提示符或 PowerShell 提示符下使用以下命令并遵照说明登录到 Azure 帐户。</span><span class="sxs-lookup"><span data-stu-id="a23ee-298">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using the command below and following the instructions.</span></span>

  ```bash
  az login  
  ```

### <a name="deploy-the-simulated-on-premises-server"></a><span data-ttu-id="a23ee-299">部署模拟的本地服务器</span><span class="sxs-lookup"><span data-stu-id="a23ee-299">Deploy the simulated on-premises server</span></span>

<span data-ttu-id="a23ee-300">首先，将 VM 部署为包含 SQL Server 2017 和相关工具的模拟本地服务器。</span><span class="sxs-lookup"><span data-stu-id="a23ee-300">First you'll deploy a VM as a simulated on-premises server, which includes SQL Server 2017 and related tools.</span></span> <span data-ttu-id="a23ee-301">此步骤还会将 [Wide World Importers OLTP 数据库](/sql/sample/world-wide-importers/wide-world-importers-oltp-database)载入 SQL Server。</span><span class="sxs-lookup"><span data-stu-id="a23ee-301">This step also loads the sample [Wide World Importers OLTP database](/sql/sample/world-wide-importers/wide-world-importers-oltp-database) into SQL Server.</span></span>

1. <span data-ttu-id="a23ee-302">导航到前面先决条件部分中下载的存储库的 `data\enterprise-bi-sqldw\onprem\templates` 文件夹。</span><span class="sxs-lookup"><span data-stu-id="a23ee-302">Navigate to the `data\enterprise-bi-sqldw\onprem\templates` folder of the repository you downloaded in the prerequisites above.</span></span>

2. <span data-ttu-id="a23ee-303">在 `onprem.parameters.json` 文件中，替换 `adminUsername` 和 `adminPassword` 的值。</span><span class="sxs-lookup"><span data-stu-id="a23ee-303">In the `onprem.parameters.json` file, replace the values for `adminUsername` and `adminPassword`.</span></span> <span data-ttu-id="a23ee-304">另外，请更改 `SqlUserCredentials` 节中的值，使之与用户名和密码匹配。</span><span class="sxs-lookup"><span data-stu-id="a23ee-304">Also change the values in the `SqlUserCredentials` section to match the user name and password.</span></span> <span data-ttu-id="a23ee-305">记下 userName 属性中的 `.\\` 前缀。</span><span class="sxs-lookup"><span data-stu-id="a23ee-305">Note the `.\\` prefix in the userName property.</span></span>
    
    ```bash
    "SqlUserCredentials": {
      "userName": ".\\username",
      "password": "password"
    }
    ```

3. <span data-ttu-id="a23ee-306">运行如下所示的 `azbb` 以部署本地服务器。</span><span class="sxs-lookup"><span data-stu-id="a23ee-306">Run `azbb` as shown below to deploy the on-premises server.</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <location> -p onprem.parameters.json --deploy
    ```

4. <span data-ttu-id="a23ee-307">部署过程可能需要 20 到 30 分钟才能完成，其中包括运行 [DSC](/powershell/dsc/overview) 脚本来安装工具和还原数据库。</span><span class="sxs-lookup"><span data-stu-id="a23ee-307">The deployment may take 20 to 30 minutes to complete, which includes running the [DSC](/powershell/dsc/overview) script to install the tools and restore the database.</span></span> <span data-ttu-id="a23ee-308">在 Azure 门户中通过查看资源组中的资源来验证部署。</span><span class="sxs-lookup"><span data-stu-id="a23ee-308">Verify the deployment in the Azure portal by reviewing the resources in the resource group.</span></span> <span data-ttu-id="a23ee-309">应会看到 `sql-vm1` 虚拟机及其关联的资源。</span><span class="sxs-lookup"><span data-stu-id="a23ee-309">You should see the `sql-vm1` virtual machine and its associated resources.</span></span>

### <a name="deploy-the-azure-resources"></a><span data-ttu-id="a23ee-310">部署 Azure 资源</span><span class="sxs-lookup"><span data-stu-id="a23ee-310">Deploy the Azure resources</span></span>

<span data-ttu-id="a23ee-311">此步骤预配 Azure SQL 数据仓库和 Azure Analysis Services 以及存储帐户。</span><span class="sxs-lookup"><span data-stu-id="a23ee-311">This step provisions Azure SQL Data Warehouse and Azure Analysis Services, along with a Storage account.</span></span> <span data-ttu-id="a23ee-312">如果需要，可与上一步骤一同运行此步骤。</span><span class="sxs-lookup"><span data-stu-id="a23ee-312">If you want, you can run this step in parallel with the previous step.</span></span>

1. <span data-ttu-id="a23ee-313">导航到前面先决条件部分中下载的存储库的 `data\enterprise-bi-sqldw\azure\templates` 文件夹。</span><span class="sxs-lookup"><span data-stu-id="a23ee-313">Navigate to the `data\enterprise-bi-sqldw\azure\templates` folder of the repository you downloaded in the prerequisites above.</span></span>

2. <span data-ttu-id="a23ee-314">运行以下 Azure CLI 命令创建资源组（请替换带括号的指定参数）。</span><span class="sxs-lookup"><span data-stu-id="a23ee-314">Run the following Azure CLI command to create a resource group, replacing the bracketed parameters specified.</span></span> <span data-ttu-id="a23ee-315">请注意，可以部署到其他资源组，而不是上一步骤中所述本地服务器使用的资源组。</span><span class="sxs-lookup"><span data-stu-id="a23ee-315">Note that you can deploy to a different resource group than you used for the on-premises server in the previous step.</span></span> 

    ```bash
    az group create --name <resource_group_name> --location <location>  
    ```

3. <span data-ttu-id="a23ee-316">运行以下 Azure CLI 命令部署 Azure 资源（请替换带括号的指定参数）。</span><span class="sxs-lookup"><span data-stu-id="a23ee-316">Run the following Azure CLI command to deploy the Azure resources, replacing the bracketed parameters specified.</span></span> <span data-ttu-id="a23ee-317">`storageAccountName` 参数必须后接存储帐户的[命名规则](../../best-practices/naming-conventions.md#naming-rules-and-restrictions)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-317">The `storageAccountName` parameter must follow the [naming rules](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) for Storage accounts.</span></span> <span data-ttu-id="a23ee-318">对于 `analysisServerAdmin` 参数，请使用 Azure Active Directory 用户主体名称 (UPN)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-318">For the `analysisServerAdmin` parameter, use your Azure Active Directory user principal name (UPN).</span></span>

    ```bash
    az group deployment create --resource-group <resource_group_name> --template-file azure-resources-deploy.json --parameters "dwServerName"="<server_name>" "dwAdminLogin"="<admin_username>" "dwAdminPassword"="<password>" "storageAccountName"="<storage_account_name>" "analysisServerName"="<analysis_server_name>" "analysisServerAdmin"="user@contoso.com"
    ```

4. <span data-ttu-id="a23ee-319">在 Azure 门户中通过查看资源组中的资源来验证部署。</span><span class="sxs-lookup"><span data-stu-id="a23ee-319">Verify the deployment in the Azure portal by reviewing the resources in the resource group.</span></span> <span data-ttu-id="a23ee-320">应会看到一个存储帐户、Azure SQL 数据仓库实例和 Analysis Services 实例。</span><span class="sxs-lookup"><span data-stu-id="a23ee-320">You should see a storage account, Azure SQL Data Warehouse instance, and Analysis Services instance.</span></span>

5. <span data-ttu-id="a23ee-321">使用 Azure 门户获取存储帐户的访问密钥。</span><span class="sxs-lookup"><span data-stu-id="a23ee-321">Use the Azure portal to get the access key for the storage account.</span></span> <span data-ttu-id="a23ee-322">选择存储帐户以将其打开。</span><span class="sxs-lookup"><span data-stu-id="a23ee-322">Select the storage account to open it.</span></span> <span data-ttu-id="a23ee-323">在“设置”下，选择“访问密钥”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-323">Under **Settings**, select **Access keys**.</span></span> <span data-ttu-id="a23ee-324">复制主密钥值。</span><span class="sxs-lookup"><span data-stu-id="a23ee-324">Copy the primary key value.</span></span> <span data-ttu-id="a23ee-325">在下一步骤中将要使用该值。</span><span class="sxs-lookup"><span data-stu-id="a23ee-325">You will use it in the next step.</span></span>

### <a name="export-the-source-data-to-azure-blob-storage"></a><span data-ttu-id="a23ee-326">将源数据导出到 Azure Blob 存储</span><span class="sxs-lookup"><span data-stu-id="a23ee-326">Export the source data to Azure Blob storage</span></span> 

<span data-ttu-id="a23ee-327">此步骤将运行一个 PowerShell 脚本，该脚本使用 bcp 将 SQL 数据库导出到 VM 上的平面文件，然后使用 AzCopy 将这些文件复制到 Azure Blob 存储中。</span><span class="sxs-lookup"><span data-stu-id="a23ee-327">In this step, you will run a PowerShell script that uses bcp to export the SQL database to flat files on the VM, and then uses AzCopy to copy those files into Azure Blob Storage.</span></span>

1. <span data-ttu-id="a23ee-328">使用远程桌面连接到模拟的本地 VM。</span><span class="sxs-lookup"><span data-stu-id="a23ee-328">Use Remote Desktop to connect to the simulated on-premises VM.</span></span>

2. <span data-ttu-id="a23ee-329">登录到 VM 后，从 PowerShell 窗口运行以下命令。</span><span class="sxs-lookup"><span data-stu-id="a23ee-329">While logged into the VM, run the following commands from a PowerShell window.</span></span>  

    ```powershell
    cd 'C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw\onprem'

    .\Load_SourceData_To_Blob.ps1 -File .\sql_scripts\db_objects.txt -Destination 'https://<storage_account_name>.blob.core.windows.net/wwi' -StorageAccountKey '<storage_account_key>'
    ```

    <span data-ttu-id="a23ee-330">对于 `Destination` 参数，请将 `<storage_account_name>` 替换为前面创建的存储帐户的名称。</span><span class="sxs-lookup"><span data-stu-id="a23ee-330">For the `Destination` parameter, replace `<storage_account_name>` with the name the Storage account that you created previously.</span></span> <span data-ttu-id="a23ee-331">对于 `StorageAccountKey` 参数，请使用该存储帐户的访问密钥。</span><span class="sxs-lookup"><span data-stu-id="a23ee-331">For the `StorageAccountKey` parameter, use the access key for that Storage account.</span></span>

3. <span data-ttu-id="a23ee-332">在 Azure 门户中导航到存储帐户，选择 Blob 服务并打开 `wwi` 容器，以验证源数据是否已复制到 Blob 存储。</span><span class="sxs-lookup"><span data-stu-id="a23ee-332">In the Azure portal, verify that the source data was copied to Blob storage by navigating to the storage account, selecting the Blob service, and opening the `wwi` container.</span></span> <span data-ttu-id="a23ee-333">应会看到以 `WorldWideImporters_Application_*` 开头的表列表。</span><span class="sxs-lookup"><span data-stu-id="a23ee-333">You should see a list of tables prefaced with `WorldWideImporters_Application_*`.</span></span>

### <a name="execute-the-data-warehouse-scripts"></a><span data-ttu-id="a23ee-334">执行数据仓库脚本</span><span class="sxs-lookup"><span data-stu-id="a23ee-334">Execute the data warehouse scripts</span></span>

1. <span data-ttu-id="a23ee-335">通过远程桌面会话启动 SQL Server Management Studio (SSMS)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-335">From your Remote Desktop session, launch SQL Server Management Studio (SSMS).</span></span> 

2. <span data-ttu-id="a23ee-336">连接到 SQL 数据仓库</span><span class="sxs-lookup"><span data-stu-id="a23ee-336">Connect to SQL Data Warehouse</span></span>

    - <span data-ttu-id="a23ee-337">服务器类型：数据库引擎</span><span class="sxs-lookup"><span data-stu-id="a23ee-337">Server type: Database Engine</span></span>
    
    - <span data-ttu-id="a23ee-338">服务器名称：`<dwServerName>.database.windows.net`，其中，`<dwServerName>` 是部署 Azure 资源时指定的名称。</span><span class="sxs-lookup"><span data-stu-id="a23ee-338">Server name: `<dwServerName>.database.windows.net`, where `<dwServerName>` is the name that you specified when you deployed the Azure resources.</span></span> <span data-ttu-id="a23ee-339">可以从 Azure 门户获取此名称。</span><span class="sxs-lookup"><span data-stu-id="a23ee-339">You can get this name from the Azure portal.</span></span>
    
    - <span data-ttu-id="a23ee-340">身份验证：SQL Server 身份验证。</span><span class="sxs-lookup"><span data-stu-id="a23ee-340">Authentication: SQL Server Authentication.</span></span> <span data-ttu-id="a23ee-341">在 `dwAdminLogin` 和 `dwAdminPassword` 参数中使用部署 Azure 资源时指定的凭据。</span><span class="sxs-lookup"><span data-stu-id="a23ee-341">Use the credentials that you specified when you deployed the Azure resources, in the `dwAdminLogin` and `dwAdminPassword` parameters.</span></span>

2. <span data-ttu-id="a23ee-342">导航到 VM 上的 `C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw\azure\sqldw_scripts` 文件夹。</span><span class="sxs-lookup"><span data-stu-id="a23ee-342">Navigate to the `C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw\azure\sqldw_scripts` folder on the VM.</span></span> <span data-ttu-id="a23ee-343">将在此文件夹中按数字顺序 `STEP_1` 至 `STEP_7` 执行脚本。</span><span class="sxs-lookup"><span data-stu-id="a23ee-343">You will execute the scripts in this folder in numerical order, `STEP_1` through `STEP_7`.</span></span>

3. <span data-ttu-id="a23ee-344">在 SSMS 中选择 `master` 数据库并打开 `STEP_1` 脚本。</span><span class="sxs-lookup"><span data-stu-id="a23ee-344">Select the `master` database in SSMS and open the `STEP_1` script.</span></span> <span data-ttu-id="a23ee-345">更改以下行中的密码值，然后执行脚本。</span><span class="sxs-lookup"><span data-stu-id="a23ee-345">Change the value of the password in the following line, then execute the script.</span></span>

    ```sql
    CREATE LOGIN LoaderRC20 WITH PASSWORD = '<change this value>';
    ```

4. <span data-ttu-id="a23ee-346">在 SSMS 中选择 `wwi` 数据库。</span><span class="sxs-lookup"><span data-stu-id="a23ee-346">Select the `wwi` database in SSMS.</span></span> <span data-ttu-id="a23ee-347">打开 `STEP_2` 脚本并执行该脚本。</span><span class="sxs-lookup"><span data-stu-id="a23ee-347">Open the `STEP_2` script and execute the script.</span></span> <span data-ttu-id="a23ee-348">如果遇到错误，请确保针对 `wwi` 数据库而不是 `master` 运行脚本。</span><span class="sxs-lookup"><span data-stu-id="a23ee-348">If you get an error, make sure you are running the script against the `wwi` database and not `master`.</span></span>

5. <span data-ttu-id="a23ee-349">使用 `STEP_1` 脚本中指定的 `LoaderRC20` 用户和密码来与 SQL 数据仓库建立新连接。</span><span class="sxs-lookup"><span data-stu-id="a23ee-349">Open a new connection to SQL Data Warehouse, using the `LoaderRC20` user and the password indicated in the `STEP_1` script.</span></span>

6. <span data-ttu-id="a23ee-350">使用此连接打开 `STEP_3` 脚本。</span><span class="sxs-lookup"><span data-stu-id="a23ee-350">Using this connection, open the `STEP_3` script.</span></span> <span data-ttu-id="a23ee-351">在脚本中设置以下值：</span><span class="sxs-lookup"><span data-stu-id="a23ee-351">Set the following values in the script:</span></span>

    - <span data-ttu-id="a23ee-352">SECRET：使用存储帐户的访问密钥。</span><span class="sxs-lookup"><span data-stu-id="a23ee-352">SECRET: Use the access key for your storage account.</span></span>
    - <span data-ttu-id="a23ee-353">LOCATION：使用存储帐户的名称，如下所示：`wasbs://wwi@<storage_account_name>.blob.core.windows.net`。</span><span class="sxs-lookup"><span data-stu-id="a23ee-353">LOCATION: Use the name of the storage account as follows: `wasbs://wwi@<storage_account_name>.blob.core.windows.net`.</span></span>

7. <span data-ttu-id="a23ee-354">使用相同的连接按顺序执行脚本 `STEP_4` 至 `STEP_7`。</span><span class="sxs-lookup"><span data-stu-id="a23ee-354">Using the same connection, execute scripts `STEP_4` through `STEP_7` sequentially.</span></span> <span data-ttu-id="a23ee-355">验证每个脚本是否成功完成，然后再运行下一个脚本。</span><span class="sxs-lookup"><span data-stu-id="a23ee-355">Verify that each script completes successfully before running the next.</span></span>

<span data-ttu-id="a23ee-356">在 SMSS 中，应会看到 `wwi` 数据库中的一组 `prd.*` 表。</span><span class="sxs-lookup"><span data-stu-id="a23ee-356">In SMSS, you should see a set of `prd.*` tables in the `wwi` database.</span></span> <span data-ttu-id="a23ee-357">若要验证是否已生成数据，请运行以下查询：</span><span class="sxs-lookup"><span data-stu-id="a23ee-357">To verify that the data was generated, run the following query:</span></span> 

```sql
SELECT TOP 10 * FROM prd.CityDimensions
```

### <a name="build-the-azure-analysis-services-model"></a><span data-ttu-id="a23ee-358">生成 Azure Analysis Services 模型</span><span class="sxs-lookup"><span data-stu-id="a23ee-358">Build the Azure Analysis Services model</span></span>

<span data-ttu-id="a23ee-359">此步骤将创建一个用于从数据仓库导入数据的表格模型。</span><span class="sxs-lookup"><span data-stu-id="a23ee-359">In this step, you will create a tabular model that imports data from the data warehouse.</span></span> <span data-ttu-id="a23ee-360">然后，将该模型部署到 Azure Analysis Services。</span><span class="sxs-lookup"><span data-stu-id="a23ee-360">Then you will deploy the model to Azure Analysis Services.</span></span>

1. <span data-ttu-id="a23ee-361">通过远程桌面会话启动 SQL Server Data Tools 2015。</span><span class="sxs-lookup"><span data-stu-id="a23ee-361">From your Remote Desktop session, launch SQL Server Data Tools 2015.</span></span>

2. <span data-ttu-id="a23ee-362">选择“文件” > “新建” > “项目”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-362">Select **File** > **New** > **Project**.</span></span>

3. <span data-ttu-id="a23ee-363">在“新建项目”对话框中的“模板”下，选择“商业智能” > “Analysis Services” > “Analysis Services 表格项目”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-363">In the **New Project** dialog, under **Templates**, select  **Business Intelligence** > **Analysis Services** > **Analysis Services Tabular Project**.</span></span> 

4. <span data-ttu-id="a23ee-364">为项目命名，并单击“确定”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-364">Name the project and click **OK**.</span></span>

5. <span data-ttu-id="a23ee-365">在“表格模型设计器”对话框中，选择“集成的工作区”，并将“兼容性级别”设置为 `SQL Server 2017 / Azure Analysis Services (1400)`。</span><span class="sxs-lookup"><span data-stu-id="a23ee-365">In the **Tabular model designer** dialog, select **Integrated workspace**  and set **Compatibility level** to `SQL Server 2017 / Azure Analysis Services (1400)`.</span></span> <span data-ttu-id="a23ee-366">单击“确定”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-366">Click **OK**.</span></span>

6. <span data-ttu-id="a23ee-367">在“表格模型资源管理器”窗口中，右键单击该项目并选择“从数据源导入”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-367">In the **Tabular Model Explorer** window, right-click the project and select **Import from Data Source**.</span></span>

7. <span data-ttu-id="a23ee-368">选择“Azure SQL 数据仓库”并单击“连接”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-368">Select **Azure SQL Data Warehouse** and click **Connect**.</span></span>

8. <span data-ttu-id="a23ee-369">对于“服务器”，请输入 Azure SQL 数据仓库服务器的完全限定名称。</span><span class="sxs-lookup"><span data-stu-id="a23ee-369">For **Server**, enter the fully qualified name of your Azure SQL Data Warehouse server.</span></span> <span data-ttu-id="a23ee-370">对于“数据库”，请输入 `wwi`。</span><span class="sxs-lookup"><span data-stu-id="a23ee-370">For **Database**, enter `wwi`.</span></span> <span data-ttu-id="a23ee-371">单击“确定”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-371">Click **OK**.</span></span>

9. <span data-ttu-id="a23ee-372">在下一个对话框中，选择“数据库”身份验证并输入 Azure SQL 数据仓库用户名和密码，然后单击“确定”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-372">In the next dialog, choose **Database** authentication and enter your Azure SQL Data Warehouse user name and password, and click **OK**.</span></span>

10. <span data-ttu-id="a23ee-373">在“导航器”对话框中，选中“prd.CityDimensions”、“prd.DateDimensions”和“prd.SalesFact”对应的复选框。</span><span class="sxs-lookup"><span data-stu-id="a23ee-373">In the **Navigator** dialog, select the checkboxes for **prd.CityDimensions**, **prd.DateDimensions**, and **prd.SalesFact**.</span></span> 

    ![](./images/analysis-services-import.png)

11. <span data-ttu-id="a23ee-374">单击“加载”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-374">Click **Load**.</span></span> <span data-ttu-id="a23ee-375">处理完成后，单击“关闭”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-375">When processing is complete, click **Close**.</span></span> <span data-ttu-id="a23ee-376">现在应会看到数据的表格视图。</span><span class="sxs-lookup"><span data-stu-id="a23ee-376">You should now see a tabular view of the data.</span></span>

12. <span data-ttu-id="a23ee-377">在“表格模型资源管理器”窗口中，右键单击该项目并选择“模型视图” > “图示视图”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-377">In the **Tabular Model Explorer** window, right-click the project and select **Model View** > **Diagram View**.</span></span>

13. <span data-ttu-id="a23ee-378">将“[prd.SalesFact].[WWI City ID]”字段拖到“[prd.CityDimensions].[WWI City ID]”字段，以创建关系。</span><span class="sxs-lookup"><span data-stu-id="a23ee-378">Drag the **[prd.SalesFact].[WWI City ID]** field to the **[prd.CityDimensions].[WWI City ID]** field to create a relationship.</span></span>  

14. <span data-ttu-id="a23ee-379">将“[prd.SalesFact].[Invoice Date Key]”字段拖到“[prd.DateDimensions].[Date]”字段。</span><span class="sxs-lookup"><span data-stu-id="a23ee-379">Drag the **[prd.SalesFact].[Invoice Date Key]** field to the **[prd.DateDimensions].[Date]** field.</span></span>  
    ![](./images/analysis-services-relations.png)

15. <span data-ttu-id="a23ee-380">在“文件”菜单中，选择“全部保存”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-380">From the **File** menu, choose **Save All**.</span></span>  

16. <span data-ttu-id="a23ee-381">在“解决方案资源管理器”中，右键单击该项目并选择“属性”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-381">In **Solution Explorer**, right-click the project and select **Properties**.</span></span> 

17. <span data-ttu-id="a23ee-382">在“服务器”下，输入 Azure Analysis Services 实例的 URL。</span><span class="sxs-lookup"><span data-stu-id="a23ee-382">Under **Server**, enter the URL of your Azure Analysis Services instance.</span></span> <span data-ttu-id="a23ee-383">可从 Azure 门户获取此值。</span><span class="sxs-lookup"><span data-stu-id="a23ee-383">You can get this value from the Azure Portal.</span></span> <span data-ttu-id="a23ee-384">在门户中选择 Analysis Services 资源，单击“概述”窗格，并找到“服务器名称”属性。</span><span class="sxs-lookup"><span data-stu-id="a23ee-384">In the portal, select the Analysis Services resource, click the Overview pane, and look for the **Server Name** property.</span></span> <span data-ttu-id="a23ee-385">该属性类似于 `asazure://westus.asazure.windows.net/contoso`。</span><span class="sxs-lookup"><span data-stu-id="a23ee-385">It will be similar to `asazure://westus.asazure.windows.net/contoso`.</span></span> <span data-ttu-id="a23ee-386">单击“确定”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-386">Click **OK**.</span></span>

    ![](./images/analysis-services-properties.png)

18. <span data-ttu-id="a23ee-387">在“解决方案资源管理器”中，右键单击该项目并选择“部署”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-387">In **Solution Explorer**, right-click the project and select **Deploy**.</span></span> <span data-ttu-id="a23ee-388">根据提示登录到 Azure。</span><span class="sxs-lookup"><span data-stu-id="a23ee-388">Sign into Azure if prompted.</span></span> <span data-ttu-id="a23ee-389">处理完成后，单击“关闭”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-389">When processing is complete, click **Close**.</span></span>

19. <span data-ttu-id="a23ee-390">在 Azure 门户中，查看 Azure Analysis Services 实例的详细信息。</span><span class="sxs-lookup"><span data-stu-id="a23ee-390">In the Azure portal, view the details for your Azure Analysis Services instance.</span></span> <span data-ttu-id="a23ee-391">检查上述模型是否出现在模型列表中。</span><span class="sxs-lookup"><span data-stu-id="a23ee-391">Verify that your model appears in the list of models.</span></span>

    ![](./images/analysis-services-models.png)

### <a name="analyze-the-data-in-power-bi-desktop"></a><span data-ttu-id="a23ee-392">在 Power BI Desktop 中分析数据</span><span class="sxs-lookup"><span data-stu-id="a23ee-392">Analyze the data in Power BI Desktop</span></span>

<span data-ttu-id="a23ee-393">此步骤使用 Power BI 基于 Analysis Services 中的数据创建报告。</span><span class="sxs-lookup"><span data-stu-id="a23ee-393">In this step, you will use Power BI to create a report from the data in Analysis Services.</span></span>

1. <span data-ttu-id="a23ee-394">通过远程桌面会话启动 Power BI Desktop。</span><span class="sxs-lookup"><span data-stu-id="a23ee-394">From your Remote Desktop session, launch Power BI Desktop.</span></span>

2. <span data-ttu-id="a23ee-395">在“欢迎”屏幕中单击“获取数据”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-395">In the Welcome Scren, click **Get Data**.</span></span>

3. <span data-ttu-id="a23ee-396">选择“Azure” > “Azure Analysis Services 数据库”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-396">Select **Azure** > **Azure Analysis Services database**.</span></span> <span data-ttu-id="a23ee-397">单击“连接”</span><span class="sxs-lookup"><span data-stu-id="a23ee-397">Click **Connect**</span></span>

    ![](./images/power-bi-get-data.png)

4. <span data-ttu-id="a23ee-398">输入 Analysis Services 实例的 URL，并单击“确定”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-398">Enter the URL of your Analysis Services instance, then click **OK**.</span></span> <span data-ttu-id="a23ee-399">根据提示登录到 Azure。</span><span class="sxs-lookup"><span data-stu-id="a23ee-399">Sign into Azure if prompted.</span></span>

5. <span data-ttu-id="a23ee-400">在“导航器”对话框中展开部署的表格项目，选择创建的模型，并单击“确定”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-400">In the **Navigator** dialog, expand the tabular project that you deployed, select the model that you created, and click **OK**.</span></span>

2. <span data-ttu-id="a23ee-401">在“可视化效果”窗格中，选择“堆积条形图”图标。</span><span class="sxs-lookup"><span data-stu-id="a23ee-401">In the **Visualizations** pane, select the **Stacked Bar Chart** icon.</span></span> <span data-ttu-id="a23ee-402">在“报告”视图中，调整可视化效果的大小以将其放大。</span><span class="sxs-lookup"><span data-stu-id="a23ee-402">In the Report view, resize the visualization to make it larger.</span></span>

6. <span data-ttu-id="a23ee-403">在“字段”窗格中，展开“prd.CityDimensions”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-403">In the **Fields** pane, expand **prd.CityDimensions**.</span></span>

7. <span data-ttu-id="a23ee-404">将“prd.CityDimensions” > “WWI 城市 ID”适当地拖到“Axis”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-404">Drag **prd.CityDimensions** > **WWI City ID** to the **Axis well**.</span></span>

8. <span data-ttu-id="a23ee-405">将“prd.CityDimensions” > “城市”适当地拖到“图例”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-405">Drag **prd.CityDimensions** > **City** to the **Legend** well.</span></span>

9. <span data-ttu-id="a23ee-406">在“字段”窗格中，展开“prd.SalesFact”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-406">In the Fields pane, expand **prd.SalesFact**.</span></span>

10. <span data-ttu-id="a23ee-407">将“prd.SalesFact” > “总计(不含税)”适当地拖到“值”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-407">Drag **prd.SalesFact** > **Total Excluding Tax** to the **Value** well.</span></span>

    ![](./images/power-bi-visualization.png)

11. <span data-ttu-id="a23ee-408">在“视觉级筛选器”下，选择“WWI 城市 ID”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-408">Under **Visual Level Filters**, select **WWI City ID**.</span></span>

12. <span data-ttu-id="a23ee-409">将“筛选器类型”设置为 `Top N`，将“显示项”设置为 `Top 10`。</span><span class="sxs-lookup"><span data-stu-id="a23ee-409">Set the **Filter Type** to `Top N`, and set **Show Items** to `Top 10`.</span></span>

13. <span data-ttu-id="a23ee-410">将“prd.SalesFact” > “总计(不含税)”适当地拖到“依据值”</span><span class="sxs-lookup"><span data-stu-id="a23ee-410">Drag **prd.SalesFact** > **Total Excluding Tax** to the **By Value** well</span></span>

    ![](./images/power-bi-visualization2.png)

14. <span data-ttu-id="a23ee-411">单击“应用筛选器”。</span><span class="sxs-lookup"><span data-stu-id="a23ee-411">Click **Apply Filter**.</span></span> <span data-ttu-id="a23ee-412">可视化效果将按城市显示排名靠前的 10 项总销售额。</span><span class="sxs-lookup"><span data-stu-id="a23ee-412">The visualization shows the top 10 total sales by city.</span></span>

    ![](./images/power-bi-report.png)

<span data-ttu-id="a23ee-413">若要详细了解 Power BI Desktop，请参阅 [Power BI Desktop 入门](/power-bi/desktop-getting-started)。</span><span class="sxs-lookup"><span data-stu-id="a23ee-413">To learn more about Power BI Desktop, see [Getting started with Power BI Desktop](/power-bi/desktop-getting-started).</span></span>

## <a name="next-steps"></a><span data-ttu-id="a23ee-414">后续步骤</span><span class="sxs-lookup"><span data-stu-id="a23ee-414">Next steps</span></span>

- <span data-ttu-id="a23ee-415">有关此参考体系结构的详细信息，请访问 [GitHub 存储库][ref-arch-repo-folder]。</span><span class="sxs-lookup"><span data-stu-id="a23ee-415">For more information about this reference architecture, visit our [GitHub repository][ref-arch-repo-folder].</span></span>
- <span data-ttu-id="a23ee-416">了解 [Azure 构建基块][azbb-repo]。</span><span class="sxs-lookup"><span data-stu-id="a23ee-416">Learn about the [Azure Building Blocks][azbb-repo].</span></span>

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb-repo]: https://github.com/mspnp/template-building-blocks
[azbb-wiki]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[ref-arch-repo-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw

