---
title: "让使用者竞争"
description: "使多个并发使用者能够处理同一消息通道上收到的消息。"
keywords: "设计模式"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: messaging
ms.openlocfilehash: d72a09ef7613bebe3701634e4eac0716400e471d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="competing-consumers-pattern"></a><span data-ttu-id="a68e8-104">使用者竞争模式</span><span class="sxs-lookup"><span data-stu-id="a68e8-104">Competing Consumers pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="a68e8-105">使多个并发使用者能够处理同一消息通道上收到的消息。</span><span class="sxs-lookup"><span data-stu-id="a68e8-105">Enable multiple concurrent consumers to process messages received on the same messaging channel.</span></span> <span data-ttu-id="a68e8-106">它可让系统同时处理多个消息，以优化吞吐量、改进可扩展性和可用性，以及平衡工作负荷。</span><span class="sxs-lookup"><span data-stu-id="a68e8-106">This enables a system to process multiple messages concurrently to optimize throughput, to improve scalability and availability, and to balance the workload.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="a68e8-107">上下文和问题</span><span class="sxs-lookup"><span data-stu-id="a68e8-107">Context and problem</span></span>

<span data-ttu-id="a68e8-108">在云中运行的应用程序需要处理大量的请求。</span><span class="sxs-lookup"><span data-stu-id="a68e8-108">An application running in the cloud is expected to handle a large number of requests.</span></span> <span data-ttu-id="a68e8-109">常用方法不是同步处理每个请求，而是应用程序通过消息传递系统将它们传送到异步处理它们的另一个服务（使用者服务）。</span><span class="sxs-lookup"><span data-stu-id="a68e8-109">Rather than process each request synchronously, a common technique is for the application to pass them through a messaging system to another service (a consumer service) that handles them asynchronously.</span></span> <span data-ttu-id="a68e8-110">此策略有助于确保在处理请求时应用程序中的业务逻辑不会被阻止。</span><span class="sxs-lookup"><span data-stu-id="a68e8-110">This strategy helps to ensure that the business logic in the application isn't blocked while the requests are being processed.</span></span>

<span data-ttu-id="a68e8-111">在一段时间内，由于多种原因请求的数量会大幅度变化。</span><span class="sxs-lookup"><span data-stu-id="a68e8-111">The number of requests can vary significantly over time for many reasons.</span></span> <span data-ttu-id="a68e8-112">用户活动或来自多个租户的总请求数的突增可能会导致不可预测的工作负荷。</span><span class="sxs-lookup"><span data-stu-id="a68e8-112">A sudden increase in user activity or aggregated requests coming from multiple tenants can cause an unpredictable workload.</span></span> <span data-ttu-id="a68e8-113">在高峰时间，系统可能需要每秒处理数百个请求，而在其他时间，请求的数量可能非常少。</span><span class="sxs-lookup"><span data-stu-id="a68e8-113">At peak hours a system might need to process many hundreds of requests per second, while at other times the number could be very small.</span></span> <span data-ttu-id="a68e8-114">此外，为处理请求而执行的工作的性质可能会有很大变化。</span><span class="sxs-lookup"><span data-stu-id="a68e8-114">Additionally, the nature of the work performed to handle these requests might be highly variable.</span></span> <span data-ttu-id="a68e8-115">运行使用者服务的单个实例可能导致该实例充满请求，或者消息系统可能由于来自应用程序的消息涌入而过载。</span><span class="sxs-lookup"><span data-stu-id="a68e8-115">Using a single instance of the consumer service can cause that instance to become flooded with requests, or the messaging system might be overloaded by an influx of messages coming from the application.</span></span> <span data-ttu-id="a68e8-116">为了处理这种波动的工作负荷，系统可以运行使用者服务的多个实例。</span><span class="sxs-lookup"><span data-stu-id="a68e8-116">To handle this fluctuating workload, the system can run multiple instances of the consumer service.</span></span> <span data-ttu-id="a68e8-117">但是，这些使用者必须进行协调以确保每条消息仅传送给一个使用者。</span><span class="sxs-lookup"><span data-stu-id="a68e8-117">However, these consumers must be coordinated to ensure that each message is only delivered to a single consumer.</span></span> <span data-ttu-id="a68e8-118">工作负荷还需要在使用者之间处于负载均衡状态，以防止实例成为瓶颈。</span><span class="sxs-lookup"><span data-stu-id="a68e8-118">The workload also needs to be load balanced across consumers to prevent an instance from becoming a bottleneck.</span></span>

## <a name="solution"></a><span data-ttu-id="a68e8-119">解决方案</span><span class="sxs-lookup"><span data-stu-id="a68e8-119">Solution</span></span>

<span data-ttu-id="a68e8-120">使用消息队列来实现应用程序和使用者服务实例之间的信道。</span><span class="sxs-lookup"><span data-stu-id="a68e8-120">Use a message queue to implement the communication channel between the application and the instances of the consumer service.</span></span> <span data-ttu-id="a68e8-121">应用程序以消息的形式将请求发送到队列，使用者服务实例从队列接收消息并进行处理。</span><span class="sxs-lookup"><span data-stu-id="a68e8-121">The application posts requests in the form of messages to the queue, and the consumer service instances receive messages from the queue and process them.</span></span> <span data-ttu-id="a68e8-122">此方法可让使用者服务实例的相同池处理来自应用程序实例的消息。</span><span class="sxs-lookup"><span data-stu-id="a68e8-122">This approach enables the same pool of consumer service instances to handle messages from any instance of the application.</span></span> <span data-ttu-id="a68e8-123">该图说明了如何使用消息队列将工作分布到服务实例。</span><span class="sxs-lookup"><span data-stu-id="a68e8-123">The figure illustrates using a message queue to distribute work to instances of a service.</span></span>

![使用消息队列将工作分布到服务实例](./_images/competing-consumers-diagram.png)

<span data-ttu-id="a68e8-125">此解决方案具有以下优点：</span><span class="sxs-lookup"><span data-stu-id="a68e8-125">This solution has the following benefits:</span></span>

- <span data-ttu-id="a68e8-126">它提供了一个负载调节系统，可以处理应用程序实例发送的大幅度变化的请求量。</span><span class="sxs-lookup"><span data-stu-id="a68e8-126">It provides a load-leveled system that can handle wide variations in the volume of requests sent by application instances.</span></span> <span data-ttu-id="a68e8-127">队列充当应用程序实例和使用者服务实例之间的缓冲区。</span><span class="sxs-lookup"><span data-stu-id="a68e8-127">The queue acts as a buffer between the application instances and the consumer service instances.</span></span> <span data-ttu-id="a68e8-128">这有助于尽量减少对应用程序和服务实例的可用性和响应性的影响，如[基于队列的负载调节模式](queue-based-load-leveling.md)中所述。</span><span class="sxs-lookup"><span data-stu-id="a68e8-128">This can help to minimize the impact on availability and responsiveness for both the application and the service instances, as described by the [Queue-based Load Leveling pattern](queue-based-load-leveling.md).</span></span> <span data-ttu-id="a68e8-129">处理需要长时间运行处理的消息时不会阻止使用者服务的其他实例同时处理其他消息。</span><span class="sxs-lookup"><span data-stu-id="a68e8-129">Handling a message that requires some long-running processing doesn't prevent other messages from being handled concurrently by other instances of the consumer service.</span></span>

- <span data-ttu-id="a68e8-130">它提高了可靠性。</span><span class="sxs-lookup"><span data-stu-id="a68e8-130">It improves reliability.</span></span> <span data-ttu-id="a68e8-131">如果生成者直接与使用者通信，而不使用这种模式且不对使用者进行监视，则消息很可能丢失或未能处理（如果使用者失败）。</span><span class="sxs-lookup"><span data-stu-id="a68e8-131">If a producer communicates directly with a consumer instead of using this pattern, but doesn't monitor the consumer, there's a high probability that messages could be lost or fail to be processed if the consumer fails.</span></span> <span data-ttu-id="a68e8-132">在此模式中，消息不会发送到特定服务实例。</span><span class="sxs-lookup"><span data-stu-id="a68e8-132">In this pattern, messages aren't sent to a specific service instance.</span></span> <span data-ttu-id="a68e8-133">失败的服务实例不会阻止生成者，并且任何工作服务实例都可处理消息。</span><span class="sxs-lookup"><span data-stu-id="a68e8-133">A failed service instance won't block a producer, and messages can be processed by any working service instance.</span></span>

- <span data-ttu-id="a68e8-134">它不需要使用者之间或生成者与使用者实例之间的复杂协调。</span><span class="sxs-lookup"><span data-stu-id="a68e8-134">It doesn't require complex coordination between the consumers, or between the producer and the consumer instances.</span></span> <span data-ttu-id="a68e8-135">消息队列可确保每条消息至少传送一次。</span><span class="sxs-lookup"><span data-stu-id="a68e8-135">The message queue ensures that each message is delivered at least once.</span></span>

- <span data-ttu-id="a68e8-136">可缩放。</span><span class="sxs-lookup"><span data-stu-id="a68e8-136">It's scalable.</span></span> <span data-ttu-id="a68e8-137">消息数量波动时，系统可以动态地增加或减少使用者服务实例的数量。</span><span class="sxs-lookup"><span data-stu-id="a68e8-137">The system can dynamically increase or decrease the number of instances of the consumer service as the volume of messages fluctuates.</span></span>

- <span data-ttu-id="a68e8-138">如果消息队列提供事务读取操作，则可以提高复原能力。</span><span class="sxs-lookup"><span data-stu-id="a68e8-138">It can improve resiliency if the message queue provides transactional read operations.</span></span> <span data-ttu-id="a68e8-139">如果使用者服务实例读取和处理消息（作为事务操作的一部分），并且使用者服务实例失败，则该模式可以确保消息将返回到队列由另一使用者服务实例进行选取并处理。</span><span class="sxs-lookup"><span data-stu-id="a68e8-139">If a consumer service instance reads and processes the message as part of a transactional operation, and the consumer service instance fails, this pattern can ensure that the message will be returned to the queue to be picked up and handled by another instance of the consumer service.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="a68e8-140">问题和注意事项</span><span class="sxs-lookup"><span data-stu-id="a68e8-140">Issues and considerations</span></span>

<span data-ttu-id="a68e8-141">在决定如何实现此模式时，请考虑以下几点：</span><span class="sxs-lookup"><span data-stu-id="a68e8-141">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="a68e8-142">**消息排序**。</span><span class="sxs-lookup"><span data-stu-id="a68e8-142">**Message ordering**.</span></span> <span data-ttu-id="a68e8-143">不能保证使用者服务实例接收消息的顺序，且不一定反映创建消息的顺序。</span><span class="sxs-lookup"><span data-stu-id="a68e8-143">The order in which consumer service instances receive messages isn't guaranteed, and doesn't necessarily reflect the order in which the messages were created.</span></span> <span data-ttu-id="a68e8-144">设计系统以确保消息处理是幂等的，因为这有助于消除对消息处理顺序的任何依赖。</span><span class="sxs-lookup"><span data-stu-id="a68e8-144">Design the system to ensure that message processing is idempotent because this will help to eliminate any dependency on the order in which messages are handled.</span></span> <span data-ttu-id="a68e8-145">有关详细信息，请参阅 Jonathon Oliver 博客中的 [Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/)（幂等模式）。</span><span class="sxs-lookup"><span data-stu-id="a68e8-145">For more information, see [Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) on Jonathon Oliver’s blog.</span></span>

    > <span data-ttu-id="a68e8-146">Microsoft Azure 服务总线队列可通过消息会话对消息执行保证的先进先出顺序。</span><span class="sxs-lookup"><span data-stu-id="a68e8-146">Microsoft Azure Service Bus Queues can implement guaranteed first-in-first-out ordering of messages by using message sessions.</span></span> <span data-ttu-id="a68e8-147">有关详细信息，请参阅[使用会话的消息传送模式](https://msdn.microsoft.com/magazine/jj863132.aspx)。</span><span class="sxs-lookup"><span data-stu-id="a68e8-147">For more information, see [Messaging Patterns Using Sessions](https://msdn.microsoft.com/magazine/jj863132.aspx).</span></span>

- <span data-ttu-id="a68e8-148">**为复原能力设计服务**。</span><span class="sxs-lookup"><span data-stu-id="a68e8-148">**Designing services for resiliency**.</span></span> <span data-ttu-id="a68e8-149">如果系统专用于检测和重新启动失败的服务实例，则可能需要实现由服务实例作为幂等操作执行的处理，以尽量减少对多次检索和处理的单个消息的影响。</span><span class="sxs-lookup"><span data-stu-id="a68e8-149">If the system is designed to detect and restart failed service instances, it might be necessary to implement the processing performed by the service instances as idempotent operations to minimize the effects of a single message being retrieved and processed more than once.</span></span>

- <span data-ttu-id="a68e8-150">**检测有害消息**。</span><span class="sxs-lookup"><span data-stu-id="a68e8-150">**Detecting poison messages**.</span></span> <span data-ttu-id="a68e8-151">格式不正确的消息或需要访问不可用资源的任务可能会导致服务实例失败。</span><span class="sxs-lookup"><span data-stu-id="a68e8-151">A malformed message, or a task that requires access to resources that aren't available, can cause a service instance to fail.</span></span> <span data-ttu-id="a68e8-152">系统应阻止此类消息返回队列，并在其他位置捕获和存储这些消息的详细信息，以便在必要时对其进行分析。</span><span class="sxs-lookup"><span data-stu-id="a68e8-152">The system should prevent such messages being returned to the queue, and instead capture and store the details of these messages elsewhere so that they can be analyzed if necessary.</span></span>

- <span data-ttu-id="a68e8-153">**处理结果**。</span><span class="sxs-lookup"><span data-stu-id="a68e8-153">**Handling results**.</span></span> <span data-ttu-id="a68e8-154">处理消息的服务实例与生成消息的应用程序逻辑完全分离，它们可能无法直接通信。</span><span class="sxs-lookup"><span data-stu-id="a68e8-154">The service instance handling a message is fully decoupled from the application logic that generates the message, and they might not be able to communicate directly.</span></span> <span data-ttu-id="a68e8-155">如果服务实例生成必须传递回应用程序逻辑的结果，则此信息必须存储在两者都可访问的位置。</span><span class="sxs-lookup"><span data-stu-id="a68e8-155">If the service instance generates results that must be passed back to the application logic, this information must be stored in a location that's accessible to both.</span></span> <span data-ttu-id="a68e8-156">为了防止应用程序逻辑检索不完整的数据，系统必须在处理完成时指示。</span><span class="sxs-lookup"><span data-stu-id="a68e8-156">In order to prevent the application logic from retrieving incomplete data the system must indicate when processing is complete.</span></span>

     > <span data-ttu-id="a68e8-157">如果使用的是 Azure，工作进程可使用专用消息答复队列将结果传回应用程序逻辑。</span><span class="sxs-lookup"><span data-stu-id="a68e8-157">If you're using Azure, a worker process can pass results back to the application logic by using a dedicated message reply queue.</span></span> <span data-ttu-id="a68e8-158">应用程序逻辑必须能够将这些结果与原始消息相关联。</span><span class="sxs-lookup"><span data-stu-id="a68e8-158">The application logic must be able to correlate these results with the original message.</span></span> <span data-ttu-id="a68e8-159">有关此方案的详细信息，请参阅 [Asynchronous Messaging Primer](https://msdn.microsoft.com/library/dn589781.aspx)（异步消息传送入门）。</span><span class="sxs-lookup"><span data-stu-id="a68e8-159">This scenario is described in more detail in the [Asynchronous Messaging Primer](https://msdn.microsoft.com/library/dn589781.aspx).</span></span>

- <span data-ttu-id="a68e8-160">**缩放消息传送系统**。</span><span class="sxs-lookup"><span data-stu-id="a68e8-160">**Scaling the messaging system**.</span></span> <span data-ttu-id="a68e8-161">在大规模解决方案中，单个消息队列可能不堪应付太多的请求，并且在系统中成为瓶颈。</span><span class="sxs-lookup"><span data-stu-id="a68e8-161">In a large-scale solution, a single message queue could be overwhelmed by the number of messages and become a bottleneck in the system.</span></span> <span data-ttu-id="a68e8-162">在这种情况下，请考虑对消息系统进行分区以将消息从特定生成者发送到特定队列，或者使用负载均衡在多个消息队列之间分发消息。</span><span class="sxs-lookup"><span data-stu-id="a68e8-162">In this situation, consider partitioning the messaging system to send messages from specific producers to a particular queue, or use load balancing to distribute messages across multiple message queues.</span></span>

- <span data-ttu-id="a68e8-163">**确保消息传送系统的可靠性**。</span><span class="sxs-lookup"><span data-stu-id="a68e8-163">**Ensuring reliability of the messaging system**.</span></span> <span data-ttu-id="a68e8-164">需要可靠的消息传递系统来保证在应用程序将消息放入队列之后它不会丢失。</span><span class="sxs-lookup"><span data-stu-id="a68e8-164">A reliable messaging system is needed to guarantee that after the application enqueues a message it won't be lost.</span></span> <span data-ttu-id="a68e8-165">这对于确保所有消息至少传送一次至关重要。</span><span class="sxs-lookup"><span data-stu-id="a68e8-165">This is essential for ensuring that all messages are delivered at least once.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="a68e8-166">何时使用此模式</span><span class="sxs-lookup"><span data-stu-id="a68e8-166">When to use this pattern</span></span>

<span data-ttu-id="a68e8-167">在以下情况下使用此模式：</span><span class="sxs-lookup"><span data-stu-id="a68e8-167">Use this pattern when:</span></span>

- <span data-ttu-id="a68e8-168">应用程序的工作负荷分为可以异步运行的任务。</span><span class="sxs-lookup"><span data-stu-id="a68e8-168">The workload for an application is divided into tasks that can run asynchronously.</span></span>
- <span data-ttu-id="a68e8-169">任务是独立的且可并行运行。</span><span class="sxs-lookup"><span data-stu-id="a68e8-169">Tasks are independent and can run in parallel.</span></span>
- <span data-ttu-id="a68e8-170">工作量是多变的，因此需要可缩放的解决方案。</span><span class="sxs-lookup"><span data-stu-id="a68e8-170">The volume of work is highly variable, requiring a scalable solution.</span></span>
- <span data-ttu-id="a68e8-171">该解决方案必须提供高可用性，并且如果任务的处理失败，必须具有复原能力。</span><span class="sxs-lookup"><span data-stu-id="a68e8-171">The solution must provide high availability, and must be resilient if the processing for a task fails.</span></span>

<span data-ttu-id="a68e8-172">在以下情况下，此模式可能不起作用：</span><span class="sxs-lookup"><span data-stu-id="a68e8-172">This pattern might not be useful when:</span></span>

- <span data-ttu-id="a68e8-173">难以将应用程序工作负荷分成离散任务，或任务之间存在高度依赖性。</span><span class="sxs-lookup"><span data-stu-id="a68e8-173">It's not easy to separate the application workload into discrete tasks, or there's a high degree of dependence between tasks.</span></span>
- <span data-ttu-id="a68e8-174">任务必须同步执行，且应用程序逻辑必须等待任务完成后才能继续。</span><span class="sxs-lookup"><span data-stu-id="a68e8-174">Tasks must be performed synchronously, and the application logic must wait for a task to complete before continuing.</span></span>
- <span data-ttu-id="a68e8-175">必须以特定顺序执行任务。</span><span class="sxs-lookup"><span data-stu-id="a68e8-175">Tasks must be performed in a specific sequence.</span></span>

> <span data-ttu-id="a68e8-176">某些消息传递系统支持会话，使生成者能够将消息组合在一起，并确保由相同的使用者进行处理。</span><span class="sxs-lookup"><span data-stu-id="a68e8-176">Some messaging systems support sessions that enable a producer to group messages together and ensure that they're all handled by the same consumer.</span></span> <span data-ttu-id="a68e8-177">此机制可用于按优先级排列的消息（如果支持）以实现消息排序的形式，从生成者到单个使用者按顺序传送消息。</span><span class="sxs-lookup"><span data-stu-id="a68e8-177">This mechanism can be used with prioritized messages (if they are supported) to implement a form of message ordering that delivers messages in sequence from a producer to a single consumer.</span></span>

## <a name="example"></a><span data-ttu-id="a68e8-178">示例</span><span class="sxs-lookup"><span data-stu-id="a68e8-178">Example</span></span>

<span data-ttu-id="a68e8-179">Azure 提供存储队列和服务总线队列，它们可用作实现此模式的机制。</span><span class="sxs-lookup"><span data-stu-id="a68e8-179">Azure provides storage queues and Service Bus queues that can act as a mechanism for implementing this pattern.</span></span> <span data-ttu-id="a68e8-180">应用程序逻辑可将消息发布到队列，并且作为一个或多个角色中的任务实现的使用者可以从该队列检索消息并进行处理。</span><span class="sxs-lookup"><span data-stu-id="a68e8-180">The application logic can post messages to a queue, and consumers implemented as tasks in one or more roles can retrieve messages from this queue and process them.</span></span> <span data-ttu-id="a68e8-181">对于复原能力，使用者可借助服务总线队列在从队列检索消息时使用 `PeekLock` 模式。</span><span class="sxs-lookup"><span data-stu-id="a68e8-181">For resiliency, a Service Bus queue enables a consumer to use `PeekLock` mode when it retrieves a message from the queue.</span></span> <span data-ttu-id="a68e8-182">此模式不会真正删除消息，而只是对其他使用者隐藏消息。</span><span class="sxs-lookup"><span data-stu-id="a68e8-182">This mode doesn't actually remove the message, but simply hides it from other consumers.</span></span> <span data-ttu-id="a68e8-183">原始使用者可在完成处理后删除消息。</span><span class="sxs-lookup"><span data-stu-id="a68e8-183">The original consumer can delete the message when it's finished processing it.</span></span> <span data-ttu-id="a68e8-184">如果使用者失败，扫视锁定将超时，该消息将再次可见且允许其他使用者检索它。</span><span class="sxs-lookup"><span data-stu-id="a68e8-184">If the consumer fails, the peek lock will time out and the message will become visible again, allowing another consumer to retrieve it.</span></span>

> <span data-ttu-id="a68e8-185">有关如何使用 Azure 服务总线队列的详细信息，请参阅 [Service Bus Queues, Topics, and Subscriptions](https://msdn.microsoft.com/library/windowsazure/hh367516.aspx)（服务总线队列、主题和订阅）。</span><span class="sxs-lookup"><span data-stu-id="a68e8-185">For detailed information on using Azure Service Bus queues, see [Service Bus queues, topics, and subscriptions](https://msdn.microsoft.com/library/windowsazure/hh367516.aspx).</span></span>
<span data-ttu-id="a68e8-186">有关如何使用 Azure 存储队列的信息，请参阅[通过 .NET 开始使用 Azure 队列存储](https://azure.microsoft.com/documentation/articles/storage-dotnet-how-to-use-queues/)。</span><span class="sxs-lookup"><span data-stu-id="a68e8-186">For information on using Azure storage queues, see [Get started with Azure Queue storage using .NET](https://azure.microsoft.com/documentation/articles/storage-dotnet-how-to-use-queues/).</span></span>

<span data-ttu-id="a68e8-187">在 [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/competing-consumers) 上可用的 CompetingConsumers 解决方案中 `QueueManager` 类的以下代码演示了如何使用 Web 或 辅助角色中 `Start` 事件处理程序的 `QueueClient` 实例创建队列。</span><span class="sxs-lookup"><span data-stu-id="a68e8-187">The following code from the `QueueManager` class in CompetingConsumers solution available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/competing-consumers) shows how you can create a queue by using a `QueueClient` instance in the `Start` event handler in a web or worker role.</span></span>

```csharp
private string queueName = ...;
private string connectionString = ...;
...

public async Task Start()
{
  // Check if the queue already exists.
  var manager = NamespaceManager.CreateFromConnectionString(this.connectionString);
  if (!manager.QueueExists(this.queueName))
  {
    var queueDescription = new QueueDescription(this.queueName);

    // Set the maximum delivery count for messages in the queue. A message
    // is automatically dead-lettered after this number of deliveries. The
    // default value for dead letter count is 10.
    queueDescription.MaxDeliveryCount = 3;

    await manager.CreateQueueAsync(queueDescription);
  }
  ...

  // Create the queue client. By default the PeekLock method is used.
  this.client = QueueClient.CreateFromConnectionString(
    this.connectionString, this.queueName);
}
```

<span data-ttu-id="a68e8-188">下一个代码片段演示了应用程序如何创建批量消息并将其发送到队列。</span><span class="sxs-lookup"><span data-stu-id="a68e8-188">The next code snippet shows how an application can create and send a batch of messages to the queue.</span></span>

```csharp
public async Task SendMessagesAsync()
{
  // Simulate sending a batch of messages to the queue.
  var messages = new List<BrokeredMessage>();

  for (int i = 0; i < 10; i++)
  {
    var message = new BrokeredMessage() { MessageId = Guid.NewGuid().ToString() };
    messages.Add(message);
  }
  await this.client.SendBatchAsync(messages);
}
```

<span data-ttu-id="a68e8-189">以下代码显示了使用者服务实例如何通过事件驱动方法从队列接收消息。</span><span class="sxs-lookup"><span data-stu-id="a68e8-189">The following code shows how a consumer service instance can receive messages from the queue by following an event-driven approach.</span></span> <span data-ttu-id="a68e8-190">`ReceiveMessages` 方法的 `processMessageTask` 参数是一个委托，其引用要在接收到消息时运行的代码。</span><span class="sxs-lookup"><span data-stu-id="a68e8-190">The `processMessageTask` parameter to the `ReceiveMessages` method is a delegate that references the code to run when a message is received.</span></span> <span data-ttu-id="a68e8-191">以异步方式运行此代码。</span><span class="sxs-lookup"><span data-stu-id="a68e8-191">This code is run asynchronously.</span></span>

```csharp
private ManualResetEvent pauseProcessingEvent;
...

public void ReceiveMessages(Func<BrokeredMessage, Task> processMessageTask)
{
  // Set up the options for the message pump.
  var options = new OnMessageOptions();

  // When AutoComplete is disabled it's necessary to manually
  // complete or abandon the messages and handle any errors.
  options.AutoComplete = false;
  options.MaxConcurrentCalls = 10;
  options.ExceptionReceived += this.OptionsOnExceptionReceived;

  // Use of the Service Bus OnMessage message pump.
  // The OnMessage method must be called once, otherwise an exception will occur.
  this.client.OnMessageAsync(
    async (msg) =>
    {
      // Will block the current thread if Stop is called.
      this.pauseProcessingEvent.WaitOne();

      // Execute processing task here.
      await processMessageTask(msg);
    },
    options);
}
...

private void OptionsOnExceptionReceived(object sender,
  ExceptionReceivedEventArgs exceptionReceivedEventArgs)
{
  ...
}
```

<span data-ttu-id="a68e8-192">请注意，自动缩放功能（例如 Azure 中提供的功能）可用于在队列长度波动时启动和停止角色实例。</span><span class="sxs-lookup"><span data-stu-id="a68e8-192">Note that autoscaling features, such as those available in Azure, can be used to start and stop role instances as the queue length fluctuates.</span></span> <span data-ttu-id="a68e8-193">有关详细信息，请参阅 [Autoscaling Guidance](https://msdn.microsoft.com/library/dn589774.aspx)（自动缩放指南）。</span><span class="sxs-lookup"><span data-stu-id="a68e8-193">For more information, see [Autoscaling Guidance](https://msdn.microsoft.com/library/dn589774.aspx).</span></span> <span data-ttu-id="a68e8-194">此外，没有必要在角色实例和工作进程之间保持一一对应的关系 &mdash; 单个角色实例可执行多个工作进程。</span><span class="sxs-lookup"><span data-stu-id="a68e8-194">Also, it's not necessary to maintain a one-to-one correspondence between role instances and worker processes&mdash;a single role instance can implement multiple worker processes.</span></span> <span data-ttu-id="a68e8-195">有关详细信息，请参阅[计算资源整合模式](compute-resource-consolidation.md)。</span><span class="sxs-lookup"><span data-stu-id="a68e8-195">For more information, see [Compute Resource Consolidation pattern](compute-resource-consolidation.md).</span></span>

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="a68e8-196">相关模式和指南</span><span class="sxs-lookup"><span data-stu-id="a68e8-196">Related patterns and guidance</span></span>

<span data-ttu-id="a68e8-197">实现此模式时，可能会与以下模式和指南相关：</span><span class="sxs-lookup"><span data-stu-id="a68e8-197">The following patterns and guidance might be relevant when implementing this pattern:</span></span>

- <span data-ttu-id="a68e8-198">[异步消息传送入门](https://msdn.microsoft.com/library/dn589781.aspx)。</span><span class="sxs-lookup"><span data-stu-id="a68e8-198">[Asynchronous Messaging Primer](https://msdn.microsoft.com/library/dn589781.aspx).</span></span> <span data-ttu-id="a68e8-199">消息队列是异步通信机制。</span><span class="sxs-lookup"><span data-stu-id="a68e8-199">Message queues are an asynchronous communications mechanism.</span></span> <span data-ttu-id="a68e8-200">如果使用者服务需要向应用程序发送回复，则可能需要实现某种形式的响应消息传送。</span><span class="sxs-lookup"><span data-stu-id="a68e8-200">If a consumer service needs to send a reply to an application, it might be necessary to implement some form of response messaging.</span></span> <span data-ttu-id="a68e8-201">异步消息传送入门提供了有关如何使用消息队列实现请求/回复消息传送的信息。</span><span class="sxs-lookup"><span data-stu-id="a68e8-201">The Asynchronous Messaging Primer provides information on how to implement request/reply messaging using message queues.</span></span>

- <span data-ttu-id="a68e8-202">[Autoscaling Guidance](https://msdn.microsoft.com/library/dn589774.aspx)（自动缩放指南）。</span><span class="sxs-lookup"><span data-stu-id="a68e8-202">[Autoscaling Guidance](https://msdn.microsoft.com/library/dn589774.aspx).</span></span> <span data-ttu-id="a68e8-203">由于队列应用程序发布消息的长度不同，因此可能可以启动和停止使用者服务的实例。</span><span class="sxs-lookup"><span data-stu-id="a68e8-203">It might be possible to start and stop instances of a consumer service since the length of the queue applications post messages on varies.</span></span> <span data-ttu-id="a68e8-204">自动缩放有助于在峰值处理期间保持吞吐量。</span><span class="sxs-lookup"><span data-stu-id="a68e8-204">Autoscaling can help to maintain throughput during times of peak processing.</span></span>

- <span data-ttu-id="a68e8-205">[计算资源整合模式](compute-resource-consolidation.md)。</span><span class="sxs-lookup"><span data-stu-id="a68e8-205">[Compute Resource Consolidation Pattern](compute-resource-consolidation.md).</span></span> <span data-ttu-id="a68e8-206">可以将使用者服务的多个实例整合到单个进程以降低成本和管理开销。</span><span class="sxs-lookup"><span data-stu-id="a68e8-206">It might be possible to consolidate multiple instances of a consumer service into a single process to reduce costs and management overhead.</span></span> <span data-ttu-id="a68e8-207">计算资源整合模式说明了遵循这种方法的利弊。</span><span class="sxs-lookup"><span data-stu-id="a68e8-207">The Compute Resource Consolidation pattern describes the benefits and tradeoffs of following this approach.</span></span>

- <span data-ttu-id="a68e8-208">[基于队列的负载调节模式](queue-based-load-leveling.md)。</span><span class="sxs-lookup"><span data-stu-id="a68e8-208">[Queue-based Load Leveling Pattern](queue-based-load-leveling.md).</span></span> <span data-ttu-id="a68e8-209">引入消息队列可以为系统添加复原能力，使服务实例能够处理应用程序实例发送的大幅度变化的请求量。</span><span class="sxs-lookup"><span data-stu-id="a68e8-209">Introducing a message queue can add resiliency to the system, enabling service instances to handle widely varying volumes of requests from application instances.</span></span> <span data-ttu-id="a68e8-210">消息队列可作为缓冲区，用于调节负载。</span><span class="sxs-lookup"><span data-stu-id="a68e8-210">The message queue acts as a buffer, which levels the load.</span></span> <span data-ttu-id="a68e8-211">有关此方案的详细信息，请参阅基于队列的负载调节模式。</span><span class="sxs-lookup"><span data-stu-id="a68e8-211">The Queue-based Load Leveling pattern describes this scenario in more detail.</span></span>

- <span data-ttu-id="a68e8-212">此模式具有相关联的[示例应用程序](https://github.com/mspnp/cloud-design-patterns/tree/master/competing-consumers)。</span><span class="sxs-lookup"><span data-stu-id="a68e8-212">This pattern has a [sample application](https://github.com/mspnp/cloud-design-patterns/tree/master/competing-consumers) associated with it.</span></span>
