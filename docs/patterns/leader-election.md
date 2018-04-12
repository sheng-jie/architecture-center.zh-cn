---
title: 领导选拔
description: 通过选拔一个实例作为领导来负责管理其他实例，协调分布式应用程序中协作性任务实例集合所执行的操作。
keywords: 设计模式
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- resiliency
ms.openlocfilehash: 3e7d47f70f660f2507f0619e1c41bf9a32a25be4
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/06/2018
---
# <a name="leader-election-pattern"></a><span data-ttu-id="c5f6b-104">领导选拔模式</span><span class="sxs-lookup"><span data-stu-id="c5f6b-104">Leader Election pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="c5f6b-105">通过选拔一个实例作为领导来负责管理其他实例，对分布式应用程序中协作性实例集合所执行的操作进行协调。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-105">Coordinate the actions performed by a collection of collaborating instances in a distributed application by electing one instance as the leader that assumes responsibility for managing the others.</span></span> <span data-ttu-id="c5f6b-106">这有助于确保各个实例互不冲突，不会导致争用共享资源或者意外地干扰其他实例正在执行的工作。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-106">This can help to ensure that instances don't conflict with each other, cause contention for shared resources, or inadvertently interfere with the work that other instances are performing.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="c5f6b-107">上下文和问题</span><span class="sxs-lookup"><span data-stu-id="c5f6b-107">Context and problem</span></span>

<span data-ttu-id="c5f6b-108">典型的云应用程序具有许多以协调方式执行的任务。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-108">A typical cloud application has many tasks acting in a coordinated manner.</span></span> <span data-ttu-id="c5f6b-109">这些任务可能全部是运行相同代码并需要访问相同资源的实例，也可能并行协作来执行复杂计算的各个部分。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-109">These tasks could all be instances running the same code and requiring access to the same resources, or they might be working together in parallel to perform the individual parts of a complex calculation.</span></span>

<span data-ttu-id="c5f6b-110">任务实例可能大多数时间分别运行，但也可能需要协调每个实例的操作来确保它们互不冲突，不会导致争用共享资源或者意外地干扰其他任务实例正在执行的工作。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-110">The task instances might run separately for much of the time, but it might also be necessary to coordinate the actions of each instance to ensure that they don’t conflict, cause contention for shared resources, or accidentally interfere with the work that other task instances are performing.</span></span>

<span data-ttu-id="c5f6b-111">例如：</span><span class="sxs-lookup"><span data-stu-id="c5f6b-111">For example:</span></span>

- <span data-ttu-id="c5f6b-112">在实现水平缩放的基于云的系统中，同一任务的多个实例可以同时运行，并且每个实例为一个不同的用户提供服务。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-112">In a cloud-based system that implements horizontal scaling, multiple instances of the same task could be running at the same time with each instance serving a different user.</span></span> <span data-ttu-id="c5f6b-113">如果这些实例向共享资源进行写入，则需要协调它们的操作以防止各个实例覆盖其他实例所做的更改。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-113">If these instances write to a shared resource, it's necessary to coordinate their actions to prevent each instance from overwriting the changes made by the others.</span></span>
- <span data-ttu-id="c5f6b-114">如果各个任务并行执行复杂计算的各个元素，则需要在它们全部完成时对结果进行聚合。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-114">If the tasks are performing individual elements of a complex calculation in parallel, the results need to be aggregated when they all complete.</span></span>

<span data-ttu-id="c5f6b-115">任务实例全部是对等的，因此没有可以充当协调器或聚合器的天生领导。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-115">The task instances are all peers, so there isn't a natural leader that can act as the coordinator or aggregator.</span></span>

## <a name="solution"></a><span data-ttu-id="c5f6b-116">解决方案</span><span class="sxs-lookup"><span data-stu-id="c5f6b-116">Solution</span></span>

<span data-ttu-id="c5f6b-117">应当选拨单个任务实例来充当领导，并且此实例应当对其他下属任务实例的操作进行协调。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-117">A single task instance should be elected to act as the leader, and this instance should coordinate the actions of the other subordinate task instances.</span></span> <span data-ttu-id="c5f6b-118">如果所有任务实例都运行相同的代码，则它们每个都能够充当领导。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-118">If all of the task instances are running the same code, they are each capable of acting as the leader.</span></span> <span data-ttu-id="c5f6b-119">因此，必须仔细管理选拨流程以防止两个或更多实例同时接管领导角色。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-119">Therefore, the election process must be managed carefully to prevent two or more instances taking over the leader role at the same time.</span></span>

<span data-ttu-id="c5f6b-120">系统必须提供用于选拨领导的可靠机制。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-120">The system must provide a robust mechanism for selecting the leader.</span></span> <span data-ttu-id="c5f6b-121">此方法必须能够应对网络中断或进程失败等事件。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-121">This method has to cope with events such as network outages or process failures.</span></span> <span data-ttu-id="c5f6b-122">在许多解决方案中，下属任务实例通过某种类型的检测信号方法或通过轮询来监视领导。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-122">In many solutions, the subordinate task instances monitor the leader through some type of heartbeat method, or by polling.</span></span> <span data-ttu-id="c5f6b-123">如果指定的领导意外终止，或者网络故障导致领导不可供下属任务实例使用，则它们需要选拨一个新领导。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-123">If the designated leader terminates unexpectedly, or a network failure makes the leader unavailable to the subordinate task instances, it's necessary for them to elect a new leader.</span></span>

<span data-ttu-id="c5f6b-124">有几种策略可用于从分布式环境中的一组任务中选拨领导，这些策略包括：</span><span class="sxs-lookup"><span data-stu-id="c5f6b-124">There are several strategies for electing a leader among a set of tasks in a distributed environment, including:</span></span>
- <span data-ttu-id="c5f6b-125">选择具有排名最低的实例或进程 ID 的任务实例。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-125">Selecting the task instance with the lowest-ranked instance or process ID.</span></span>
- <span data-ttu-id="c5f6b-126">争夺一个共享的分布式互斥体。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-126">Racing to acquire a shared, distributed mutex.</span></span> <span data-ttu-id="c5f6b-127">第一个获得该互斥体的任务实例成为领导。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-127">The first task instance that acquires the mutex is the leader.</span></span> <span data-ttu-id="c5f6b-128">但是，系统必须确保，当领导终止或者与系统的其余部分断开了连接时，必须释放该互斥体以允许其他任务实例成为领导。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-128">However, the system must ensure that, if the leader terminates or becomes disconnected from the rest of the system, the mutex is released to allow another task instance to become the leader.</span></span>
- <span data-ttu-id="c5f6b-129">实现常用的领导选拨算法之一，例如[欺负算法](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html)或[环型算法](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html)。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-129">Implementing one of the common leader election algorithms such as the [Bully Algorithm](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html) or the [Ring Algorithm](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html).</span></span> <span data-ttu-id="c5f6b-130">这些算法假设选举中的每个候选者都具有唯一的 ID，并且它可以可靠地与其他候选者进行通信。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-130">These algorithms assume that each candidate in the election has a unique ID, and that it can communicate with the other candidates reliably.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="c5f6b-131">问题和注意事项</span><span class="sxs-lookup"><span data-stu-id="c5f6b-131">Issues and considerations</span></span>

<span data-ttu-id="c5f6b-132">在决定如何实现此模式时，请考虑以下几点：</span><span class="sxs-lookup"><span data-stu-id="c5f6b-132">Consider the following points when deciding how to implement this pattern:</span></span>
- <span data-ttu-id="c5f6b-133">选拨领导的流程在发生暂时性和永久性故障后应该能够复原。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-133">The process of electing a leader should be resilient to transient and persistent failures.</span></span>
- <span data-ttu-id="c5f6b-134">必须能够检测领导何时发生故障或变得不可用（例如由于通信故障）。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-134">It must be possible to detect when the leader has failed or has become otherwise unavailable (such as due to a communications failure).</span></span> <span data-ttu-id="c5f6b-135">需要多快的检测速度取决于系统。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-135">How quickly detection is needed is system dependent.</span></span> <span data-ttu-id="c5f6b-136">某些系统也许能够在没有领导的情况下短时行使职责，在这期间，暂时性错误也许能够得到修复。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-136">Some systems might be able to function for a short time without a leader, during which a transient fault might be fixed.</span></span> <span data-ttu-id="c5f6b-137">在其他情况下，可能需要立即检测到领导故障并触发新的选举。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-137">In other cases, it might be necessary to detect leader failure immediately and trigger a new election.</span></span>
- <span data-ttu-id="c5f6b-138">在实现水平自动缩放的系统中，如果系统收缩并关闭一些计算资源，则领导可能会终止。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-138">In a system that implements horizontal autoscaling, the leader could be terminated if the system scales back and shuts down some of the computing resources.</span></span>
- <span data-ttu-id="c5f6b-139">使用共享的分布式互斥体将依赖于提供该互斥体的外部服务。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-139">Using a shared, distributed mutex introduces a dependency on the external service that provides the mutex.</span></span> <span data-ttu-id="c5f6b-140">该服务成为单一故障点。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-140">The service constitutes a single point of failure.</span></span> <span data-ttu-id="c5f6b-141">如果它由于任何原因而变得不可用，则系统将无法选拨领导。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-141">If it becomes unavailable for any reason, the system won't be able to elect a leader.</span></span>
- <span data-ttu-id="c5f6b-142">使用单个专用进程作为领导是简单明了的方法。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-142">Using a single dedicated process as the leader is a straightforward approach.</span></span> <span data-ttu-id="c5f6b-143">但是，如果该进程发生故障，则在它重新启动时可能会导致明显的延迟。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-143">However, if the process fails there could be a significant delay while it's restarted.</span></span> <span data-ttu-id="c5f6b-144">如果其他进程在等待领导来协调某个操作，则导致的延迟可能会影响其他进程的性能和响应时间。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-144">The resulting latency can affect the performance and response times of other processes if they're waiting for the leader to coordinate an operation.</span></span>
- <span data-ttu-id="c5f6b-145">手动实现领导选拨算法之一可以在调整和优化代码方面提供最大的灵活性。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-145">Implementing one of the leader election algorithms manually provides the greatest flexibility for tuning and optimizing the code.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="c5f6b-146">何时使用此模式</span><span class="sxs-lookup"><span data-stu-id="c5f6b-146">When to use this pattern</span></span>

<span data-ttu-id="c5f6b-147">当分布式应用程序（例如云托管解决方案）中的任务需要仔细协调并且没有天生的领导时，请使用此模式。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-147">Use this pattern when the tasks in a distributed application, such as a cloud-hosted solution, need careful coordination and there's no natural leader.</span></span>

>  <span data-ttu-id="c5f6b-148">请避免使领导成为系统中的瓶颈。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-148">Avoid making the leader a bottleneck in the system.</span></span> <span data-ttu-id="c5f6b-149">领导的用途是协调下属任务的工作，并且它并非必须参与此工作本身 &mdash; 尽管当该任务未被选举为领导时应该能够参与此工作。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-149">The purpose of the leader is to coordinate the work of the subordinate tasks, and it doesn't necessarily have to participate in this work itself&mdash;although it should be able to do so if the task isn't elected as the leader.</span></span>

<span data-ttu-id="c5f6b-150">在下列情况下，此模式可能不适用：</span><span class="sxs-lookup"><span data-stu-id="c5f6b-150">This pattern might not be useful if:</span></span>
- <span data-ttu-id="c5f6b-151">有一个天生的领导或有一个能够始终充当领导的专用进程。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-151">There's a natural leader or dedicated process that can always act as the leader.</span></span> <span data-ttu-id="c5f6b-152">例如，可以实现一个对任务实例进行协调的单一实例进程。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-152">For example, it might be possible to implement a singleton process that coordinates the task instances.</span></span> <span data-ttu-id="c5f6b-153">如果此进程发生故障或变得不正常，则系统可以将其关闭并重新启动它。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-153">If this process fails or becomes unhealthy, the system can shut it down and restart it.</span></span>
- <span data-ttu-id="c5f6b-154">任务之间的协调可以使用更轻量的方法来实现。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-154">The coordination between tasks can be achieved using a more lightweight method.</span></span> <span data-ttu-id="c5f6b-155">例如，如果多个任务实例只是需要实现对共享资源的协调访问，则更好的解决方案是使用乐观或悲观锁定来控制访问。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-155">For example, if several task instances simply need coordinated access to a shared resource, a better solution is to use optimistic or pessimistic locking to control access.</span></span>
- <span data-ttu-id="c5f6b-156">有更合适的第三方解决方案。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-156">A third-party solution is more appropriate.</span></span> <span data-ttu-id="c5f6b-157">例如，Microsoft Azure HDInsight 服务（基于 Apache Hadoop）使用 Apache Zookeeper 提供的服务来协调用于收集和汇总数据的 map 和 reduce 任务。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-157">For example, the Microsoft Azure HDInsight service (based on Apache Hadoop) uses the services provided by Apache Zookeeper to coordinate the map and reduce tasks that collect and summarize data.</span></span>

## <a name="example"></a><span data-ttu-id="c5f6b-158">示例</span><span class="sxs-lookup"><span data-stu-id="c5f6b-158">Example</span></span>

<span data-ttu-id="c5f6b-159">LeaderElection 解决方案中的 DistributedMutex 项目（[GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election) 上提供的演示了此模式的示例）展示了如何在 Azure 存储 blob 上使用租约来提供用于实现共享的分布式互斥体的机制。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-159">The DistributedMutex project in the LeaderElection solution (a sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election)) shows how to use a lease on an Azure Storage blob to provide a mechanism for implementing a shared, distributed mutex.</span></span> <span data-ttu-id="c5f6b-160">可以使用此互斥体从 Azure 云服务中的一组角色实例中选拨领导。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-160">This mutex can be used to elect a leader among a group of role instances in an Azure cloud service.</span></span> <span data-ttu-id="c5f6b-161">第一个获得租约的角色实例被选拨为领导，并且它一直保持为领导，直到它释放租约或者无法续订租约。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-161">The first role instance to acquire the lease is elected the leader, and remains the leader until it releases the lease or isn't able to renew the lease.</span></span> <span data-ttu-id="c5f6b-162">其他角色实例可以继续监视 blob 租约，以防领导不再可用。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-162">Other role instances can continue to monitor the blob lease in case the leader is no longer available.</span></span>

>  <span data-ttu-id="c5f6b-163">Blob 租约是一个针对 blob 的排他写入锁。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-163">A blob lease is an exclusive write lock over a blob.</span></span> <span data-ttu-id="c5f6b-164">单个 blob 在任一时刻只能是一个租约的主题。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-164">A single blob can be the subject of only one lease at any point in time.</span></span> <span data-ttu-id="c5f6b-165">角色实例可以请求针对指定 blob 的租约，并且如果没有其他角色实例持有针对同一 blob 的租约，则会向其授予租约。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-165">A role instance can request a lease over a specified blob, and it'll be granted the lease if no other role instance holds a lease over the same blob.</span></span> <span data-ttu-id="c5f6b-166">否则，请求将引发异常。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-166">Otherwise the request will throw an exception.</span></span>
> 
> <span data-ttu-id="c5f6b-167">为避免出现错误的角色实例无限期地保留租约，请指定租约的生存期。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-167">To avoid a faulted role instance retaining the lease indefinitely, specify a lifetime for the lease.</span></span> <span data-ttu-id="c5f6b-168">当生存期过期时，租约将变得可用。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-168">When this expires, the lease becomes available.</span></span> <span data-ttu-id="c5f6b-169">不过，当角色实例持有租约时，它可以请求续订租约，并且将会授权它在更长的一段时间内使用该租约。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-169">However, while a role instance holds the lease it can request that the lease is renewed, and it'll be granted the lease for a further period of time.</span></span> <span data-ttu-id="c5f6b-170">如果角色实例希望保留租约，则它可以不断重复此过程。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-170">The role instance can continually repeat this process if it wants to retain the lease.</span></span>
> <span data-ttu-id="c5f6b-171">有关如何租用 blob 的详细信息，请参阅 [Lease Blob (REST API)](https://msdn.microsoft.com/library/azure/ee691972.aspx)（租用 Blob (REST API)）。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-171">For more information on how to lease a blob, see [Lease Blob (REST API)](https://msdn.microsoft.com/library/azure/ee691972.aspx).</span></span>

<span data-ttu-id="c5f6b-172">下面的 C# 示例中的 `BlobDistributedMutex` 类包含 `RunTaskWhenMutexAquired` 方法，该方法使得角色实例能够尝试获得针对指定 blob 的租约。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-172">The `BlobDistributedMutex` class in the C# example below contains the `RunTaskWhenMutexAquired` method that enables a role instance to attempt to acquire a lease over a specified blob.</span></span> <span data-ttu-id="c5f6b-173">当创建 `BlobDistributedMutex` 对象时（此对象是示例代码中包括的一个简单结构），会将该 blob 的详细信息（名称、容器和存储帐户）传递给 `BlobSettings` 对象中的构造函数。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-173">The details of the blob (the name, container, and storage account) are passed to the constructor in a `BlobSettings` object when the `BlobDistributedMutex` object is created (this object is a simple struct that is included in the sample code).</span></span> <span data-ttu-id="c5f6b-174">该构造函数还接受一个 `Task`，后者引用了角色实例在成功获得针对 blob 的租约并被选拨为领导后应当运行的代码。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-174">The constructor also accepts a `Task` that references the code that the role instance should run if it successfully acquires the lease over the blob and is elected the leader.</span></span> <span data-ttu-id="c5f6b-175">请注意，名为 `BlobLeaseManager` 的一个单独的帮助程序类中实现了用于处理有关获得租约的低层详细信息的代码。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-175">Note that the code that handles the low-level details of acquiring the lease is implemented in a separate helper class named `BlobLeaseManager`.</span></span>

```csharp
public class BlobDistributedMutex
{
  ...
  private readonly BlobSettings blobSettings;
  private readonly Func<CancellationToken, Task> taskToRunWhenLeaseAcquired;
  ...

  public BlobDistributedMutex(BlobSettings blobSettings,
           Func<CancellationToken, Task> taskToRunWhenLeaseAquired)
  {
    this.blobSettings = blobSettings;
    this.taskToRunWhenLeaseAquired = taskToRunWhenLeaseAquired;
  }

  public async Task RunTaskWhenMutexAcquired(CancellationToken token)
  {
    var leaseManager = new BlobLeaseManager(blobSettings);
    await this.RunTaskWhenBlobLeaseAcquired(leaseManager, token);
  }
  ...
```

<span data-ttu-id="c5f6b-176">上面的代码示例中的 `RunTaskWhenMutexAquired` 方法调用下面的代码示例中显示的 `RunTaskWhenBlobLeaseAcquired` 方法来实际获得租约。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-176">The `RunTaskWhenMutexAquired` method in the code sample above invokes the `RunTaskWhenBlobLeaseAcquired` method shown in the following code sample to actually acquire the lease.</span></span> <span data-ttu-id="c5f6b-177">`RunTaskWhenBlobLeaseAcquired` 方法以异步方式运行。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-177">The `RunTaskWhenBlobLeaseAcquired` method runs asynchronously.</span></span> <span data-ttu-id="c5f6b-178">如果成功获得租约，则表明该角色实例已被选拨为领导。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-178">If the lease is successfully acquired, the role instance has been elected the leader.</span></span> <span data-ttu-id="c5f6b-179">`taskToRunWhenLeaseAcquired` 委托的用途是执行对其他角色实例进行协调的工作。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-179">The purpose of the `taskToRunWhenLeaseAcquired` delegate is to perform the work that coordinates the other role instances.</span></span> <span data-ttu-id="c5f6b-180">如果没有获得租约，则表明另一角色实例已被选拨为领导并且当前角色实例保持为下属。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-180">If the lease isn't acquired, another role instance has been elected as the leader and the current role instance remains a subordinate.</span></span> <span data-ttu-id="c5f6b-181">请注意，`TryAcquireLeaseOrWait` 方法是一个帮助程序方法，它使用 `BlobLeaseManager` 对象来获得租约。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-181">Note that the `TryAcquireLeaseOrWait` method is a helper method that uses the `BlobLeaseManager` object to acquire the lease.</span></span>

```csharp
  private async Task RunTaskWhenBlobLeaseAcquired(
    BlobLeaseManager leaseManager, CancellationToken token)
  {
    while (!token.IsCancellationRequested)
    {
      // Try to acquire the blob lease.
      // Otherwise wait for a short time before trying again.
      string leaseId = await this.TryAquireLeaseOrWait(leaseManager, token);

      if (!string.IsNullOrEmpty(leaseId))
      {
        // Create a new linked cancellation token source so that if either the
        // original token is canceled or the lease can't be renewed, the
        // leader task can be canceled.
        using (var leaseCts =
          CancellationTokenSource.CreateLinkedTokenSource(new[] { token }))
        {
          // Run the leader task.
          var leaderTask = this.taskToRunWhenLeaseAquired.Invoke(leaseCts.Token);
          ...
        }
      }
    }
    ...
  }
```

<span data-ttu-id="c5f6b-182">领导启动的任务也是以异步方式运行的。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-182">The task started by the leader also runs asynchronously.</span></span> <span data-ttu-id="c5f6b-183">当此任务在运行时，下面的代码示例中显示的 `RunTaskWhenBlobLeaseAquired` 方法会定期尝试续订租约。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-183">While this task is running, the `RunTaskWhenBlobLeaseAquired` method shown in the following code sample periodically attempts to renew the lease.</span></span> <span data-ttu-id="c5f6b-184">这有助于确保该角色实例保持为领导。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-184">This helps to ensure that the role instance remains the leader.</span></span> <span data-ttu-id="c5f6b-185">在示例解决方案中，续订请求之间的延迟小于为租约的持续时间指定的时间，以防止其他角色实例被选拨为领导。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-185">In the sample solution, the delay between renewal requests is less than the time specified for the duration of the lease in order to prevent another role instance from being elected the leader.</span></span> <span data-ttu-id="c5f6b-186">如果续订由于任何原因而失败，则任务会被取消。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-186">If the renewal fails for any reason, the task is canceled.</span></span>

<span data-ttu-id="c5f6b-187">如果租约未能续订或者任务被取消（可能是由于角色实例关闭），则会释放租约。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-187">If the lease fails to be renewed or the task is canceled (possibly as a result of the role instance shutting down), the lease is released.</span></span> <span data-ttu-id="c5f6b-188">此时，可能会将此角色实例或另一角色实例选拨为领导。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-188">At this point, this or another role instance might be elected as the leader.</span></span> <span data-ttu-id="c5f6b-189">下面的代码摘录显示了过程的这一部分。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-189">The code extract below shows this part of the process.</span></span>

```csharp
  private async Task RunTaskWhenBlobLeaseAcquired(
    BlobLeaseManager leaseManager, CancellationToken token)
  {
    while (...)
    {
      ...
      if (...)
      {
        ...
        using (var leaseCts = ...)
        {
          ...
          // Keep renewing the lease in regular intervals.
          // If the lease can't be renewed, then the task completes.
          var renewLeaseTask =
            this.KeepRenewingLease(leaseManager, leaseId, leaseCts.Token);

          // When any task completes (either the leader task itself or when it
          // couldn't renew the lease) then cancel the other task.
          await CancelAllWhenAnyCompletes(leaderTask, renewLeaseTask, leaseCts);
        }
      }
    }
  }
  ...
}
```

<span data-ttu-id="c5f6b-190">`KeepRenewingLease` 方法是一个帮助程序方法，它使用 `BlobLeaseManager` 对象来续订租约。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-190">The `KeepRenewingLease` method is another helper method that uses the `BlobLeaseManager` object to renew the lease.</span></span> <span data-ttu-id="c5f6b-191">`CancelAllWhenAnyCompletes` 方法取消作为前两个参数指定的任务。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-191">The `CancelAllWhenAnyCompletes` method cancels the tasks specified as the first two parameters.</span></span> <span data-ttu-id="c5f6b-192">下图展示了使用 `BlobDistributedMutex` 类来选拨领导并运行对操作进行协调的任务。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-192">The following diagram illustrates using the `BlobDistributedMutex` class to elect a leader and run a task that coordinates operations.</span></span>

![图 1 展示了 BlobDistributedMutex 类的功能](./_images/leader-election-diagram.png)


<span data-ttu-id="c5f6b-194">下面的代码示例展示了如何以辅助角色使用 `BlobDistributedMutex` 类。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-194">The following code example shows how to use the `BlobDistributedMutex` class in a worker role.</span></span> <span data-ttu-id="c5f6b-195">此代码在开发存储的租约容器中获得对名为 `MyLeaderCoordinatorTask` 的 blob 的租约，并指定当角色实例被选拨为领导时应当运行 `MyLeaderCoordinatorTask` 方法中定义的代码。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-195">This code acquires a lease over a blob named `MyLeaderCoordinatorTask` in the lease's container in development storage, and specifies that the code defined in the `MyLeaderCoordinatorTask` method should run if the role instance is elected the leader.</span></span>

```csharp
var settings = new BlobSettings(CloudStorageAccount.DevelopmentStorageAccount,
  "leases", "MyLeaderCoordinatorTask");
var cts = new CancellationTokenSource();
var mutex = new BlobDistributedMutex(settings, MyLeaderCoordinatorTask);
mutex.RunTaskWhenMutexAcquired(this.cts.Token);
...

// Method that runs if the role instance is elected the leader
private static async Task MyLeaderCoordinatorTask(CancellationToken token)
{
  ...
}
```

<span data-ttu-id="c5f6b-196">请注意关于示例解决方案的以下要点：</span><span class="sxs-lookup"><span data-stu-id="c5f6b-196">Note the following points about the sample solution:</span></span>
- <span data-ttu-id="c5f6b-197">Blob 是潜在的单一故障点。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-197">The blob is a potential single point of failure.</span></span> <span data-ttu-id="c5f6b-198">如果 blob 服务变得不可用，或者无法访问，则领导将无法续订租约，并且没有任何其他角色实例能够获得租约。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-198">If the blob service becomes unavailable, or is inaccessible, the leader won't be able to renew the lease and no other role instance will be able to acquire the lease.</span></span> <span data-ttu-id="c5f6b-199">在这种情况下，将没有任何角色实例能够充当领导。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-199">In this case, no role instance will be able to act as the leader.</span></span> <span data-ttu-id="c5f6b-200">不过，blob 服务设计为具有复原能力，因此，blob 服务完全失败的情况几乎不可能出现。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-200">However, the blob service is designed to be resilient, so complete failure of the blob service is considered to be extremely unlikely.</span></span>
- <span data-ttu-id="c5f6b-201">如果领导正在执行的任务停滞，则领导可能会继续续订租约，从而阻止其他角色实例获得租约并接管领导角色来协调任务。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-201">If the task being performed by the leader stalls, the leader might continue to renew the lease, preventing any other role instance from acquiring the lease and taking over the leader role in order to coordinate tasks.</span></span> <span data-ttu-id="c5f6b-202">在现实世界中，应当频繁检查领导的运行状况。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-202">In the real world, the health of the leader should be checked at frequent intervals.</span></span>
- <span data-ttu-id="c5f6b-203">选举流程具有不确定性。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-203">The election process is nondeterministic.</span></span> <span data-ttu-id="c5f6b-204">无法就哪个角色实例将获得 blob 租约并成为领导做出任何假设。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-204">You can't make any assumptions about which role instance will acquire the blob lease and become the leader.</span></span>
- <span data-ttu-id="c5f6b-205">用作 blob 租约的目标的 blob 不应当用于任何其他用途。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-205">The blob used as the target of the blob lease shouldn't be used for any other purpose.</span></span> <span data-ttu-id="c5f6b-206">如果某个角色实例尝试将数据存储在此 blob 中，则此数据将无法访问，除非该角色实例是领导并持有 blob 租约。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-206">If a role instance attempts to store data in this blob, this data won't be accessible unless the role instance is the leader and holds the blob lease.</span></span>

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="c5f6b-207">相关模式和指南</span><span class="sxs-lookup"><span data-stu-id="c5f6b-207">Related patterns and guidance</span></span>

<span data-ttu-id="c5f6b-208">实现此模式时，以下指南可能也比较有用：</span><span class="sxs-lookup"><span data-stu-id="c5f6b-208">The following guidance might also be relevant when implementing this pattern:</span></span>
- <span data-ttu-id="c5f6b-209">此模式具有可下载的[示例应用程序](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election)。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-209">This pattern has a downloadable [sample application](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election).</span></span>
- <span data-ttu-id="c5f6b-210">[Autoscaling Guidance](https://msdn.microsoft.com/library/dn589774.aspx)（自动缩放指南）。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-210">[Autoscaling Guidance](https://msdn.microsoft.com/library/dn589774.aspx).</span></span> <span data-ttu-id="c5f6b-211">可以根据应用程序上负载的变化来启动和停止任务主机的实例。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-211">It's possible to start and stop instances of the task hosts as the load on the application varies.</span></span> <span data-ttu-id="c5f6b-212">自动缩放有助于在峰值处理期间保持吞吐量和性能。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-212">Autoscaling can help to maintain throughput and performance during times of peak processing.</span></span>
- <span data-ttu-id="c5f6b-213">[计算分区指南](https://msdn.microsoft.com/library/dn589773.aspx)。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-213">[Compute Partitioning Guidance](https://msdn.microsoft.com/library/dn589773.aspx).</span></span> <span data-ttu-id="c5f6b-214">本指南介绍了如何将任务分配给云服务中的主机，以便最大程度地降低运行成本，同时保持服务的可伸缩性、性能、可用性和安全性。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-214">This guidance describes how to allocate tasks to hosts in a cloud service in a way that helps to minimize running costs while maintaining the scalability, performance, availability, and security of the service.</span></span>
- <span data-ttu-id="c5f6b-215">[基于任务的异步模式](https://msdn.microsoft.com/library/hh873175.aspx)。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-215">The [Task-based Asynchronous Pattern](https://msdn.microsoft.com/library/hh873175.aspx).</span></span>
- <span data-ttu-id="c5f6b-216">展示了[欺负算法](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html)的示例。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-216">An example illustrating the [Bully Algorithm](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html).</span></span>
- <span data-ttu-id="c5f6b-217">展示了[环型算法](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html)的示例。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-217">An example illustrating the [Ring Algorithm](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html).</span></span>
- <span data-ttu-id="c5f6b-218">Microsoft Open Technologies 网站上的 [Apache Zookeeper on Microsoft Azure](https://msopentech.com/opentech-projects/apache-zookeeper-on-windows-azure-2/)（Microsoft Azure 上的 Apache Zookeeper）一文。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-218">The article [Apache Zookeeper on Microsoft Azure](https://msopentech.com/opentech-projects/apache-zookeeper-on-windows-azure-2/) on the Microsoft Open Technologies website.</span></span>
- <span data-ttu-id="c5f6b-219">[Apache Curator](http://curator.apache.org/) - Apache ZooKeeper 的一个客户端库。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-219">[Apache Curator](http://curator.apache.org/) a client library for Apache ZooKeeper.</span></span>
- <span data-ttu-id="c5f6b-220">MSDN 上的 [Lease Blob (REST API)](https://msdn.microsoft.com/library/azure/ee691972.aspx)（租用 Blob (REST API)）一文。</span><span class="sxs-lookup"><span data-stu-id="c5f6b-220">The article [Lease Blob (REST API)](https://msdn.microsoft.com/library/azure/ee691972.aspx) on MSDN.</span></span>
