---
title: "事务数据"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: b7fdbb403d2a438ebc59e40ef58ed8067489dddc
ms.sourcegitcommit: 943e671a8d522cef5ddc8c6e04848134b03c2de4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/05/2018
---
# <a name="transactional-data"></a><span data-ttu-id="8259d-102">事务数据</span><span class="sxs-lookup"><span data-stu-id="8259d-102">Transactional data</span></span>

<span data-ttu-id="8259d-103">事务数据是指一种信息，用于跟踪与组织活动相关的交互。</span><span class="sxs-lookup"><span data-stu-id="8259d-103">Transactional data is information that tracks the interactions related to an organization's activities.</span></span> <span data-ttu-id="8259d-104">这些交互通常是业务事务，如从客户收到的付款、对供应商进行的付款、从库存移动的产品、接受的订单或交付的服务。</span><span class="sxs-lookup"><span data-stu-id="8259d-104">These interactions are typically business transactions, such as payments received from customers, payments made to suppliers, products moving through inventory, orders taken, or services delivered.</span></span> <span data-ttu-id="8259d-105">表示事务本身的事务事件通常包含时间维度、一些数值和对其他数据的引用。</span><span class="sxs-lookup"><span data-stu-id="8259d-105">Transactional events, which represent the transactions themselves, typically contain a time dimension, some numerical values, and references to other data.</span></span> 

<span data-ttu-id="8259d-106">事务通常需要*原子性*和*一致性*。</span><span class="sxs-lookup"><span data-stu-id="8259d-106">Transactions typically need to be *atomic* and *consistent*.</span></span> <span data-ttu-id="8259d-107">原子性意味着整个事务始终作为一个工作单元成功或失败，永远不会处于半完成状态。</span><span class="sxs-lookup"><span data-stu-id="8259d-107">Atomicity means that an entire transaction always succeeds or fails as one unit of work, and is never left in a half-completed state.</span></span> <span data-ttu-id="8259d-108">如果无法完成某个事务，数据库系统必须回退任何已作为该事务的一部分完成的步骤。</span><span class="sxs-lookup"><span data-stu-id="8259d-108">If a transaction cannot be completed, the database system must roll back any steps that were already done as part of that transaction.</span></span> <span data-ttu-id="8259d-109">在传统 RDBMS 中，如果某个事务无法完成，这种回退会自动发生。</span><span class="sxs-lookup"><span data-stu-id="8259d-109">In a traditional RDBMS, this rollback happens automatically if a transaction cannot be completed.</span></span> <span data-ttu-id="8259d-110">一致性意味着事务始终让数据处于有效状态。</span><span class="sxs-lookup"><span data-stu-id="8259d-110">Consistency means that transactions always leave the data in a valid state.</span></span> <span data-ttu-id="8259d-111">（这些是原子性和一致性的非常不正式的说明。</span><span class="sxs-lookup"><span data-stu-id="8259d-111">(These are very informal descriptions of atomicity and consistency.</span></span> <span data-ttu-id="8259d-112">这些属性有更正式的定义，如 [ACID](https://en.wikipedia.org/wiki/ACID)。）</span><span class="sxs-lookup"><span data-stu-id="8259d-112">There are more formal definitions of these properties, such as [ACID](https://en.wikipedia.org/wiki/ACID).)</span></span>

<span data-ttu-id="8259d-113">事务数据库可以使用各种锁定策略（如悲观锁定）支持事务的强一致性，以确保所有用户和进程的所有数据在企业的上下文中具有强一致性。</span><span class="sxs-lookup"><span data-stu-id="8259d-113">Transactional databases can support strong consistency for transactions using various locking strategies, such as pessimistic locking, to ensure that all data is strongly consistent within the context of the enterprise, for all users and processes.</span></span> 

<span data-ttu-id="8259d-114">使用事务数据的最常见部署体系结构是 3 层体系结构中的数据存储层。</span><span class="sxs-lookup"><span data-stu-id="8259d-114">The most common deployment architecture that uses transactional data is the data store tier in a 3-tier architecture.</span></span> <span data-ttu-id="8259d-115">3 层体系结构通常包含表示层、业务逻辑层和数据存储层。</span><span class="sxs-lookup"><span data-stu-id="8259d-115">A 3-tier architecture typically consists of a presentation tier, business logic tier, and data store tier.</span></span> <span data-ttu-id="8259d-116">相关的部署体系结构是 [N 层](/azure/architecture/guide/architecture-styles/n-tier)体系结构，它可能具有多个处理业务逻辑的中间层。</span><span class="sxs-lookup"><span data-stu-id="8259d-116">A related deployment architecture is the [N-tier](/azure/architecture/guide/architecture-styles/n-tier) architecture, which may have multiple middle-tiers handling business logic.</span></span>

![3 层应用程序的示例](./images/three-tier-application.png)

## <a name="typical-traits-of-transactional-data"></a><span data-ttu-id="8259d-118">事务数据的典型特征</span><span class="sxs-lookup"><span data-stu-id="8259d-118">Typical traits of transactional data</span></span>

<span data-ttu-id="8259d-119">事务数据往往具有以下特征：</span><span class="sxs-lookup"><span data-stu-id="8259d-119">Transactional data tends to have the following traits:</span></span>

| <span data-ttu-id="8259d-120">要求</span><span class="sxs-lookup"><span data-stu-id="8259d-120">Requirement</span></span> | <span data-ttu-id="8259d-121">说明</span><span class="sxs-lookup"><span data-stu-id="8259d-121">Description</span></span> |
| --- | --- |
| <span data-ttu-id="8259d-122">规范化</span><span class="sxs-lookup"><span data-stu-id="8259d-122">Normalization</span></span> | <span data-ttu-id="8259d-123">高度规范化</span><span class="sxs-lookup"><span data-stu-id="8259d-123">Highly normalized</span></span> |
| <span data-ttu-id="8259d-124">架构</span><span class="sxs-lookup"><span data-stu-id="8259d-124">Schema</span></span> | <span data-ttu-id="8259d-125">写入时架构，强制实施</span><span class="sxs-lookup"><span data-stu-id="8259d-125">Schema on write, strongly enforced</span></span>|
| <span data-ttu-id="8259d-126">一致性</span><span class="sxs-lookup"><span data-stu-id="8259d-126">Consistency</span></span> | <span data-ttu-id="8259d-127">强一致性，保证 ACID</span><span class="sxs-lookup"><span data-stu-id="8259d-127">Strong consistency, ACID guarantees</span></span> |
| <span data-ttu-id="8259d-128">完整性</span><span class="sxs-lookup"><span data-stu-id="8259d-128">Integrity</span></span> | <span data-ttu-id="8259d-129">高完整性</span><span class="sxs-lookup"><span data-stu-id="8259d-129">High integrity</span></span> |
| <span data-ttu-id="8259d-130">使用事务</span><span class="sxs-lookup"><span data-stu-id="8259d-130">Uses transactions</span></span> | <span data-ttu-id="8259d-131">是</span><span class="sxs-lookup"><span data-stu-id="8259d-131">Yes</span></span> |
| <span data-ttu-id="8259d-132">锁定策略</span><span class="sxs-lookup"><span data-stu-id="8259d-132">Locking strategy</span></span> | <span data-ttu-id="8259d-133">悲观或乐观</span><span class="sxs-lookup"><span data-stu-id="8259d-133">Pessimistic or optimistic</span></span>|
| <span data-ttu-id="8259d-134">可更新</span><span class="sxs-lookup"><span data-stu-id="8259d-134">Updateable</span></span> | <span data-ttu-id="8259d-135">是</span><span class="sxs-lookup"><span data-stu-id="8259d-135">Yes</span></span> |
| <span data-ttu-id="8259d-136">可追加</span><span class="sxs-lookup"><span data-stu-id="8259d-136">Appendable</span></span> | <span data-ttu-id="8259d-137">是</span><span class="sxs-lookup"><span data-stu-id="8259d-137">Yes</span></span> |
| <span data-ttu-id="8259d-138">工作负载</span><span class="sxs-lookup"><span data-stu-id="8259d-138">Workload</span></span> | <span data-ttu-id="8259d-139">大量写入，适度读取</span><span class="sxs-lookup"><span data-stu-id="8259d-139">Heavy writes, moderate reads</span></span> |
| <span data-ttu-id="8259d-140">索引</span><span class="sxs-lookup"><span data-stu-id="8259d-140">Indexing</span></span> | <span data-ttu-id="8259d-141">主要和辅助索引</span><span class="sxs-lookup"><span data-stu-id="8259d-141">Primary and secondary indexes</span></span> |
| <span data-ttu-id="8259d-142">基准大小</span><span class="sxs-lookup"><span data-stu-id="8259d-142">Datum size</span></span> | <span data-ttu-id="8259d-143">中小型</span><span class="sxs-lookup"><span data-stu-id="8259d-143">Small to medium sized</span></span> |
| <span data-ttu-id="8259d-144">模型</span><span class="sxs-lookup"><span data-stu-id="8259d-144">Model</span></span> | <span data-ttu-id="8259d-145">关系</span><span class="sxs-lookup"><span data-stu-id="8259d-145">Relational</span></span> |
| <span data-ttu-id="8259d-146">数据形状</span><span class="sxs-lookup"><span data-stu-id="8259d-146">Data shape</span></span> | <span data-ttu-id="8259d-147">表格</span><span class="sxs-lookup"><span data-stu-id="8259d-147">Tabular</span></span> |
| <span data-ttu-id="8259d-148">查询灵活性</span><span class="sxs-lookup"><span data-stu-id="8259d-148">Query flexibility</span></span> | <span data-ttu-id="8259d-149">高度灵活</span><span class="sxs-lookup"><span data-stu-id="8259d-149">Highly flexible</span></span> |
| <span data-ttu-id="8259d-150">缩放</span><span class="sxs-lookup"><span data-stu-id="8259d-150">Scale</span></span> | <span data-ttu-id="8259d-151">小型 (MB) 到大型（几 TB）</span><span class="sxs-lookup"><span data-stu-id="8259d-151">Small (MBs) to Large (a few TBs)</span></span> | 

## <a name="see-also"></a><span data-ttu-id="8259d-152">另请参阅</span><span class="sxs-lookup"><span data-stu-id="8259d-152">See Also</span></span>

[<span data-ttu-id="8259d-153">联机事务处理</span><span class="sxs-lookup"><span data-stu-id="8259d-153">Online Transaction Processing</span></span>](../scenarios/online-transaction-processing.md)
