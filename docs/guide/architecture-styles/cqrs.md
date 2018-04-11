---
title: CQRS 体系结构样式
description: 介绍 CQRS 体系结构的优点、挑战和最佳做法
author: MikeWasson
ms.openlocfilehash: dd3da5886587159f57646ff1bfffa2094725f798
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="cqrs-architecture-style"></a><span data-ttu-id="78a74-103">CQRS 体系结构样式</span><span class="sxs-lookup"><span data-stu-id="78a74-103">CQRS architecture style</span></span>

<span data-ttu-id="78a74-104">命令和查询责任分离 (CQRS) 是将读取操作与写入操作分离的体系结构样式。</span><span class="sxs-lookup"><span data-stu-id="78a74-104">Command and Query Responsibility Segregation (CQRS) is an architecture style that separates read operations from write operations.</span></span> 

![](./images/cqrs-logical.svg)

<span data-ttu-id="78a74-105">在传统的体系结构中，使用同一数据模型查询和更新数据库。</span><span class="sxs-lookup"><span data-stu-id="78a74-105">In traditional architectures, the same data model is used to query and update a database.</span></span> <span data-ttu-id="78a74-106">这十分简单，非常适用于基本的 CRUD 操作。</span><span class="sxs-lookup"><span data-stu-id="78a74-106">That's simple and works well for basic CRUD operations.</span></span> <span data-ttu-id="78a74-107">但是，在更复杂的应用程序中，此方法会变得难以操作。</span><span class="sxs-lookup"><span data-stu-id="78a74-107">In more complex applications, however, this approach can become unwieldy.</span></span> <span data-ttu-id="78a74-108">例如，在读取方面，应用程序可能执行大量不同的查询，返回具有不同形状的数据传输对象 (DTO)。</span><span class="sxs-lookup"><span data-stu-id="78a74-108">For example, on the read side, the application may perform many different queries, returning data transfer objects (DTOs) with different shapes.</span></span> <span data-ttu-id="78a74-109">对象映射可能会变得复杂。</span><span class="sxs-lookup"><span data-stu-id="78a74-109">Object mapping can become complicated.</span></span> <span data-ttu-id="78a74-110">在写入方面，模型可能实施复杂验证和业务逻辑。</span><span class="sxs-lookup"><span data-stu-id="78a74-110">On the write side, the model may implement complex validation and business logic.</span></span> <span data-ttu-id="78a74-111">结果，模型执行太多操作，过度复杂。</span><span class="sxs-lookup"><span data-stu-id="78a74-111">As a result, you can end up with an overly complex model that does too much.</span></span>

<span data-ttu-id="78a74-112">另一潜在问题是，读取和写入工作负载通常是非对称的，两者的性能和缩放需求存在显著差异。</span><span class="sxs-lookup"><span data-stu-id="78a74-112">Another potential problem is that read and write workloads are often asymmetrical, with very different performance and scale requirements.</span></span> 

<span data-ttu-id="78a74-113">CQRS 将读取和写入分离到单独的模型中，使用命令更新数据，使用查询读取数据，从而解决上述问题。</span><span class="sxs-lookup"><span data-stu-id="78a74-113">CQRS addresses these problems by separating reads and writes into separate models, using **commands** to update data, and **queries** to read data.</span></span>

- <span data-ttu-id="78a74-114">命令应基于任务，而非以数据为中心。</span><span class="sxs-lookup"><span data-stu-id="78a74-114">Commands should be task based, rather than data centric.</span></span> <span data-ttu-id="78a74-115">（“预订酒店房间”，而不是“将预订状态设置为已预订”。）可将命令放置于队列上供异步处理，而不是同步处理。</span><span class="sxs-lookup"><span data-stu-id="78a74-115">("Book hotel room," not "set ReservationStatus to Reserved.") Commands may be placed on a queue for asynchronous processing, rather than being processed synchronously.</span></span>

- <span data-ttu-id="78a74-116">查询从不修改数据库。</span><span class="sxs-lookup"><span data-stu-id="78a74-116">Queries never modify the database.</span></span> <span data-ttu-id="78a74-117">查询返回的 DTO 不封装任何域知识。</span><span class="sxs-lookup"><span data-stu-id="78a74-117">A query returns a DTO that does not encapsulate any domain knowledge.</span></span>

<span data-ttu-id="78a74-118">为更好地实现隔离，可将读取数据与写入数据通过物理方式分离。</span><span class="sxs-lookup"><span data-stu-id="78a74-118">For greater isolation, you can physically separate the read data from the write data.</span></span> <span data-ttu-id="78a74-119">在此情况下，读取数据库可使用自己的已针对查询进行优化的数据架构。</span><span class="sxs-lookup"><span data-stu-id="78a74-119">In that case, the read database can use its own data schema that is optimized for queries.</span></span> <span data-ttu-id="78a74-120">例如，它可以存储数据的[具体化视图][materialized-view]，从而避免复杂联接或复杂 O/RM 映射。</span><span class="sxs-lookup"><span data-stu-id="78a74-120">For example, it can store a [materialized view][materialized-view] of the data, in order to avoid complex joins or complex O/RM mappings.</span></span> <span data-ttu-id="78a74-121">它甚至可能使用不同类型的数据存储。</span><span class="sxs-lookup"><span data-stu-id="78a74-121">It might even use a different type of data store.</span></span> <span data-ttu-id="78a74-122">例如，写入数据库可能是关系数据库，而读取数据库是文档数据库。</span><span class="sxs-lookup"><span data-stu-id="78a74-122">For example, the write database might be relational, while the read database is a document database.</span></span>

<span data-ttu-id="78a74-123">如果使用了单独的读取和写入数据库，必须将它们保持同步。通常情况下，每次更新数据库时写模型会发布一个事件，从而实现同步。</span><span class="sxs-lookup"><span data-stu-id="78a74-123">If separate read and write databases are used, they must be kept in sync. Typically this is accomplished by  having the write model publish an event whenever it updates the database.</span></span> <span data-ttu-id="78a74-124">必须在一个事务中更新数据库和发布事件。</span><span class="sxs-lookup"><span data-stu-id="78a74-124">Updating the database and publishing the event must occur in a single transaction.</span></span> 

<span data-ttu-id="78a74-125">某些 CQRS 实现使用[事件溯源模式][event-sourcing]。</span><span class="sxs-lookup"><span data-stu-id="78a74-125">Some implementations of CQRS use the [Event Sourcing pattern][event-sourcing].</span></span> <span data-ttu-id="78a74-126">在此模式中，应用程序状态存储为事件序列。</span><span class="sxs-lookup"><span data-stu-id="78a74-126">With this pattern, application state is stored as a sequence of events.</span></span> <span data-ttu-id="78a74-127">每个事件表示对数据所作的一系列更改。</span><span class="sxs-lookup"><span data-stu-id="78a74-127">Each event represents a set of changes to the data.</span></span> <span data-ttu-id="78a74-128">通过重播事件构造当前状态。</span><span class="sxs-lookup"><span data-stu-id="78a74-128">The current state is constructed by replaying the events.</span></span> <span data-ttu-id="78a74-129">在 CQRS 上下文中，事件溯源的一个好处是，可以使用相同的事件通知其他组件 &mdash; 特别是通知读模型。</span><span class="sxs-lookup"><span data-stu-id="78a74-129">In a CQRS context, one benefit of Event Sourcing is that the same events can be used to notify other components &mdash; in particular, to notify the read model.</span></span> <span data-ttu-id="78a74-130">读模型使用事件创建当前状态的快照，这对查询而言更高效。</span><span class="sxs-lookup"><span data-stu-id="78a74-130">The read model uses the events to create a snapshot of the current state, which is more efficient for queries.</span></span> <span data-ttu-id="78a74-131">但是，事件溯源增加了设计的复杂度。</span><span class="sxs-lookup"><span data-stu-id="78a74-131">However, Event Sourcing adds complexity to the design.</span></span>

![](./images/cqrs-events.svg)

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="78a74-132">此体系结构适用的情况</span><span class="sxs-lookup"><span data-stu-id="78a74-132">When to use this architecture</span></span>

<span data-ttu-id="78a74-133">请考虑对协作域使用 CQRS，在该域中，多名用户访问相同的数据，尤其当读取和写入工作负载非对称时。</span><span class="sxs-lookup"><span data-stu-id="78a74-133">Consider CQRS for collaborative domains where many users access the same data, especially when the read and write workloads are asymmetrical.</span></span>

<span data-ttu-id="78a74-134">CQRS 不是适用于整个系统的顶层体系结构。</span><span class="sxs-lookup"><span data-stu-id="78a74-134">CQRS is not a top-level architecture that applies to an entire system.</span></span> <span data-ttu-id="78a74-135">仅将 CQRS 应用于分离读写行为具有显著价值的子系统。</span><span class="sxs-lookup"><span data-stu-id="78a74-135">Apply CQRS only to those subsystems where there is clear value in separating reads and writes.</span></span> <span data-ttu-id="78a74-136">否则，会增加复杂性，却无任何好处。</span><span class="sxs-lookup"><span data-stu-id="78a74-136">Otherwise, you are creating additional complexity for no benefit.</span></span>

## <a name="benefits"></a><span data-ttu-id="78a74-137">优点</span><span class="sxs-lookup"><span data-stu-id="78a74-137">Benefits</span></span>

- <span data-ttu-id="78a74-138">**独立缩放**。</span><span class="sxs-lookup"><span data-stu-id="78a74-138">**Independently scaling**.</span></span> <span data-ttu-id="78a74-139">CQRS 允许读取和写入工作负载独立缩放，这可能会减少锁争用。</span><span class="sxs-lookup"><span data-stu-id="78a74-139">CQRS allows the read and write workloads to scale independently, and may result in fewer lock contentions.</span></span>
- <span data-ttu-id="78a74-140">**优化的数据架构。**</span><span class="sxs-lookup"><span data-stu-id="78a74-140">**Optimized data schemas.**</span></span>  <span data-ttu-id="78a74-141">读取端可使用针对查询优化的架构，写入端可使用针对更新优化的架构。</span><span class="sxs-lookup"><span data-stu-id="78a74-141">The read side can use a schema that is optimized for queries, while the write side uses a schema that is optimized for updates.</span></span>  
- <span data-ttu-id="78a74-142">**安全性**。</span><span class="sxs-lookup"><span data-stu-id="78a74-142">**Security**.</span></span> <span data-ttu-id="78a74-143">更轻松地确保仅正确的域实体对数据执行写入操作。</span><span class="sxs-lookup"><span data-stu-id="78a74-143">It's easier to ensure that only the right domain entities are performing writes on the data.</span></span>
- <span data-ttu-id="78a74-144">**将问题分离**。</span><span class="sxs-lookup"><span data-stu-id="78a74-144">**Separation of concerns**.</span></span> <span data-ttu-id="78a74-145">分离读取和写入端可使模型更易维护且更灵活。</span><span class="sxs-lookup"><span data-stu-id="78a74-145">Segregating the read and write sides can result in models that are more maintainable and flexible.</span></span> <span data-ttu-id="78a74-146">大多数复杂的业务逻辑被分到写模型。</span><span class="sxs-lookup"><span data-stu-id="78a74-146">Most of the complex business logic goes into the write model.</span></span> <span data-ttu-id="78a74-147">读模型会变得相对简单。</span><span class="sxs-lookup"><span data-stu-id="78a74-147">The read model can be relatively simple.</span></span>
- <span data-ttu-id="78a74-148">**查询更简单**。</span><span class="sxs-lookup"><span data-stu-id="78a74-148">**Simpler queries**.</span></span> <span data-ttu-id="78a74-149">通过将具体化视图存储在读取数据库中，应用程序可在查询时避免复杂联接。</span><span class="sxs-lookup"><span data-stu-id="78a74-149">By storing a materialized view in the read database, the application can avoid complex joins when querying.</span></span>

## <a name="challenges"></a><span data-ttu-id="78a74-150">挑战</span><span class="sxs-lookup"><span data-stu-id="78a74-150">Challenges</span></span>

- <span data-ttu-id="78a74-151">**复杂性**。</span><span class="sxs-lookup"><span data-stu-id="78a74-151">**Complexity**.</span></span> <span data-ttu-id="78a74-152">CQRS 的基本理念十分简单。</span><span class="sxs-lookup"><span data-stu-id="78a74-152">The basic idea of CQRS is simple.</span></span> <span data-ttu-id="78a74-153">但它可能导致应用程序的设计更为复杂，尤其在包含事件溯源模式时。</span><span class="sxs-lookup"><span data-stu-id="78a74-153">But it can lead to a more complex application design, especially if they include the Event Sourcing pattern.</span></span>

- <span data-ttu-id="78a74-154">**消息**。</span><span class="sxs-lookup"><span data-stu-id="78a74-154">**Messaging**.</span></span> <span data-ttu-id="78a74-155">虽然 CQRS 不需要消息，但它通常会使用消息处理命令和发布更新事件。</span><span class="sxs-lookup"><span data-stu-id="78a74-155">Although CQRS does not require messaging, it's common to use messaging to process commands and publish update events.</span></span> <span data-ttu-id="78a74-156">在此情况下，应用程序必须处理消息失败或重复的消息。</span><span class="sxs-lookup"><span data-stu-id="78a74-156">In that case, the application must handle message failures or duplicate messages.</span></span> 

- <span data-ttu-id="78a74-157">**最终一致性**。</span><span class="sxs-lookup"><span data-stu-id="78a74-157">**Eventual consistency**.</span></span> <span data-ttu-id="78a74-158">如果分离读取和写入数据库，读取数据可能会过时。</span><span class="sxs-lookup"><span data-stu-id="78a74-158">If you separate the read and write databases, the read data may be stale.</span></span> 

## <a name="best-practices"></a><span data-ttu-id="78a74-159">最佳实践</span><span class="sxs-lookup"><span data-stu-id="78a74-159">Best practices</span></span>

- <span data-ttu-id="78a74-160">有关实现 CQRS 的详细信息，请参阅 [CQRS 模式][cqrs-pattern]。</span><span class="sxs-lookup"><span data-stu-id="78a74-160">For more information about implementing CQRS, see [CQRS Pattern][cqrs-pattern].</span></span>

- <span data-ttu-id="78a74-161">请考虑使用[事件溯源][event-sourcing]模式，避免出现更新冲突。</span><span class="sxs-lookup"><span data-stu-id="78a74-161">Consider using the [Event Sourcing][event-sourcing] pattern to avoid update conflicts.</span></span>

- <span data-ttu-id="78a74-162">请考虑对读模型使用[具体化视图模式][materialized-view]，以便优化查询架构。</span><span class="sxs-lookup"><span data-stu-id="78a74-162">Consider using the [Materialized View pattern][materialized-view] for the read model, to optimize the schema for queries.</span></span>

## <a name="cqrs-in-microservices"></a><span data-ttu-id="78a74-163">微服务中的 CQRS</span><span class="sxs-lookup"><span data-stu-id="78a74-163">CQRS in microservices</span></span>

<span data-ttu-id="78a74-164">CQRS 尤其适用于[微服务体系结构][microservices]。</span><span class="sxs-lookup"><span data-stu-id="78a74-164">CQRS can be especially useful in a [microservices architecture][microservices].</span></span> <span data-ttu-id="78a74-165">微服务的一项原则是服务不能直接访问另一服务的数据存储。</span><span class="sxs-lookup"><span data-stu-id="78a74-165">One of the principles of microservices is that a service cannot directly access another service's data store.</span></span>

![](./images/cqrs-microservices-wrong.png)

<span data-ttu-id="78a74-166">在下图中，服务 A 写入到数据存储，服务 B 保存数据的具体化视图。</span><span class="sxs-lookup"><span data-stu-id="78a74-166">In the following diagram, Service A writes to a data store, and Service B keeps a materialized view of the data.</span></span> <span data-ttu-id="78a74-167">服务 A 在每次写入到数据存储时发布事件。</span><span class="sxs-lookup"><span data-stu-id="78a74-167">Service A publishes an event whenever it writes to the data store.</span></span> <span data-ttu-id="78a74-168">服务 B 订阅该事件。</span><span class="sxs-lookup"><span data-stu-id="78a74-168">Service B subscribes to the event.</span></span>

![](./images/cqrs-microservices-right.png)


<!-- links -->

[cqrs-pattern]: ../../patterns/cqrs.md
[event-sourcing]: ../../patterns/event-sourcing.md
[materialized-view]: ../../patterns/materialized-view.md
[microservices]: ./microservices.md
