---
title: "重试"
description: "当应用程序尝试连接到服务或网络资源时，使应用程序能够通过以透明方式重试先前失败的操作来处理预期的临时故障。"
keywords: "设计模式"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: resiliency
ms.openlocfilehash: 6c02b384e71c068ecbc78f3170d28cea406538e2
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="retry-pattern"></a><span data-ttu-id="f5170-104">重试模式</span><span class="sxs-lookup"><span data-stu-id="f5170-104">Retry pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="f5170-105">当应用程序尝试连接到服务或网络资源时，使应用程序能够通过以透明方式重试失败的操作来处理临时故障。</span><span class="sxs-lookup"><span data-stu-id="f5170-105">Enable an application to handle transient failures when it tries to connect to a service or network resource, by transparently retrying a failed operation.</span></span> <span data-ttu-id="f5170-106">这可以提高应用程序的稳定性。</span><span class="sxs-lookup"><span data-stu-id="f5170-106">This can improve the stability of the application.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="f5170-107">上下文和问题</span><span class="sxs-lookup"><span data-stu-id="f5170-107">Context and problem</span></span>

<span data-ttu-id="f5170-108">与在云中运行的元素进行通信的应用程序必须能够敏感地察觉到此环境中可能会出现的暂时性错误。</span><span class="sxs-lookup"><span data-stu-id="f5170-108">An application that communicates with elements running in the cloud has to be sensitive to the transient faults that can occur in this environment.</span></span> <span data-ttu-id="f5170-109">这类故障包括组件和服务瞬间断开网络连接、服务暂时不可用，或者当服务繁忙时出现超时。</span><span class="sxs-lookup"><span data-stu-id="f5170-109">Faults include the momentary loss of network connectivity to components and services, the temporary unavailability of a service, or timeouts that occur when a service is busy.</span></span>

<span data-ttu-id="f5170-110">这些错误通常可以自己修复，如果在延迟合适的时间后重新执行触发了错误的操作，该操作可能会成功。</span><span class="sxs-lookup"><span data-stu-id="f5170-110">These faults are typically self-correcting, and if the action that triggered a fault is repeated after a suitable delay it's likely to be successful.</span></span> <span data-ttu-id="f5170-111">例如，处理大量并发请求的数据库服务可以实现限制策略，该策略会暂时拒绝任何后续请求，直到其工作负荷得以减轻。</span><span class="sxs-lookup"><span data-stu-id="f5170-111">For example, a database service that's processing a large number of concurrent requests can implement a throttling strategy that temporarily rejects any further requests until its workload has eased.</span></span> <span data-ttu-id="f5170-112">尝试访问该数据库的应用程序可能无法连接，但如果它在延迟一段时间后再次尝试，则可能会成功。</span><span class="sxs-lookup"><span data-stu-id="f5170-112">An application trying to access the database might fail to connect, but if it tries again after a delay it might succeed.</span></span>

## <a name="solution"></a><span data-ttu-id="f5170-113">解决方案</span><span class="sxs-lookup"><span data-stu-id="f5170-113">Solution</span></span>

<span data-ttu-id="f5170-114">在云中，暂时性错误很常见，因此应当将应用程序设计为能够优雅地以透明方式处理它们。</span><span class="sxs-lookup"><span data-stu-id="f5170-114">In the cloud, transient faults aren't uncommon and an application should be designed to handle them elegantly and transparently.</span></span> <span data-ttu-id="f5170-115">这可以尽量降低错误可能会给应用程序正在执行的业务任务带来的影响。</span><span class="sxs-lookup"><span data-stu-id="f5170-115">This minimizes the effects faults can have on the business tasks the application is performing.</span></span>

<span data-ttu-id="f5170-116">如果应用程序在尝试将请求发送到远程服务时检测到故障，则它可以使用以下策略来处理故障：</span><span class="sxs-lookup"><span data-stu-id="f5170-116">If an application detects a failure when it tries to send a request to a remote service, it can handle the failure using the following strategies:</span></span>

- <span data-ttu-id="f5170-117">**取消**。</span><span class="sxs-lookup"><span data-stu-id="f5170-117">**Cancel**.</span></span> <span data-ttu-id="f5170-118">如果错误表明故障不是暂时性的或者在重新执行的情况下不可能成功，则应用程序应当取消操作并报告异常。</span><span class="sxs-lookup"><span data-stu-id="f5170-118">If the fault indicates that the failure isn't transient or is unlikely to be successful if repeated, the application should cancel the operation and report an exception.</span></span> <span data-ttu-id="f5170-119">例如，对于因为提供了无效的凭据而导致的身份验证失败，无论尝试多少次，身份验证都不可能成功。</span><span class="sxs-lookup"><span data-stu-id="f5170-119">For example, an authentication failure caused by providing invalid credentials is not likely to succeed no matter how many times it's attempted.</span></span>

- <span data-ttu-id="f5170-120">**重试**。</span><span class="sxs-lookup"><span data-stu-id="f5170-120">**Retry**.</span></span> <span data-ttu-id="f5170-121">如果所报告的具体错误不常见或极少见，则它可能是由不常见的情况（例如网络包在传输过程中损坏）导致的。</span><span class="sxs-lookup"><span data-stu-id="f5170-121">If the specific fault reported is unusual or rare, it might have been caused by unusual circumstances such as a network packet becoming corrupted while it was being transmitted.</span></span> <span data-ttu-id="f5170-122">在这种情况下，应用程序可以立即再次重试失败的请求，因为不大可能会重复出现同一故障并且请求可能会成功。</span><span class="sxs-lookup"><span data-stu-id="f5170-122">In this case, the application could retry the failing request again immediately because the same failure is unlikely to be repeated and the request will probably be successful.</span></span>

- <span data-ttu-id="f5170-123">**在延迟一段时间后重试。**</span><span class="sxs-lookup"><span data-stu-id="f5170-123">**Retry after delay.**</span></span> <span data-ttu-id="f5170-124">如果错误是由更普遍的连接或繁忙故障之一引起的，则网络或服务可能需要很短的一段时间来等待连接问题得以修复或积压的工作得以清除。</span><span class="sxs-lookup"><span data-stu-id="f5170-124">If the fault is caused by one of the more commonplace connectivity or busy failures, the network or service might need a short period while the connectivity issues are corrected or the backlog of work is cleared.</span></span> <span data-ttu-id="f5170-125">应用程序应当等待合适的时间，然后重试请求。</span><span class="sxs-lookup"><span data-stu-id="f5170-125">The application should wait for a suitable time before retrying the request.</span></span>

<span data-ttu-id="f5170-126">对于更常见的暂时性故障，在选择重试之间的时长时应当考虑使来自应用程序的多个实例的请求尽可能均匀地分布。</span><span class="sxs-lookup"><span data-stu-id="f5170-126">For the more common transient failures, the period between retries should be chosen to spread requests from multiple instances of the application as evenly as possible.</span></span> <span data-ttu-id="f5170-127">这可以降低繁忙的服务持续过载的可能性。</span><span class="sxs-lookup"><span data-stu-id="f5170-127">This reduces the chance of a busy service continuing to be overloaded.</span></span> <span data-ttu-id="f5170-128">如果应用程序的许多实例由于重试请求而导致某个服务持续过载，则该服务将需要更长的时间才能恢复。</span><span class="sxs-lookup"><span data-stu-id="f5170-128">If many instances of an application are continually overwhelming a service with retry requests, it'll take the service longer to recover.</span></span>

<span data-ttu-id="f5170-129">如果请求仍然失败，则应用程序可以等待并进行另一尝试。</span><span class="sxs-lookup"><span data-stu-id="f5170-129">If the request still fails, the application can wait and make another attempt.</span></span> <span data-ttu-id="f5170-130">如果需要，可以在增大重试尝试之间的延迟时间的情况下不断重复此过程，直到已尝试的请求数目达到某个最大数目。</span><span class="sxs-lookup"><span data-stu-id="f5170-130">If necessary, this process can be repeated with increasing delays between retry attempts, until some maximum number of requests have been attempted.</span></span> <span data-ttu-id="f5170-131">可以采用递增方式或指数方式增大延迟时间，具体取决于故障的类型和它在此时间段内被更正的可能性。</span><span class="sxs-lookup"><span data-stu-id="f5170-131">The delay can be increased incrementally or exponentially, depending on the type of failure and the probability that it'll be corrected during this time.</span></span>

<span data-ttu-id="f5170-132">下图展示了使用此模式调用托管服务中的某个操作。</span><span class="sxs-lookup"><span data-stu-id="f5170-132">The following diagram illustrates invoking an operation in a hosted service using this pattern.</span></span> <span data-ttu-id="f5170-133">如果请求在经历预定义的尝试次数后没有成功，则应用程序应当将该错误视为异常并相应地对其进行处理。</span><span class="sxs-lookup"><span data-stu-id="f5170-133">If the request is unsuccessful after a predefined number of attempts, the application should treat the fault as an exception and handle it accordingly.</span></span>

![图 1 - 使用重试模式调用托管服务中的某个操作](./_images/retry-pattern.png)

<span data-ttu-id="f5170-135">应用程序应当将访问远程服务的所有尝试包装在代码中并在代码中实现与上面列出的策略之一匹配的重试策略。</span><span class="sxs-lookup"><span data-stu-id="f5170-135">The application should wrap all attempts to access a remote service in code that implements a retry policy matching one of the strategies listed above.</span></span> <span data-ttu-id="f5170-136">发送到不同服务的请求遵守不同的策略。</span><span class="sxs-lookup"><span data-stu-id="f5170-136">Requests sent to different services can be subject to different policies.</span></span> <span data-ttu-id="f5170-137">某些供应商提供了实现了重试策略的库，应用程序可以在这些重试策略中指定最大重试次数、重试尝试之间的间隔时间以及其他参数。</span><span class="sxs-lookup"><span data-stu-id="f5170-137">Some vendors provide libraries that implement retry policies, where the application can specify the maximum number of retries, the time between retry attempts, and other parameters.</span></span>

<span data-ttu-id="f5170-138">应用程序应当记录错误和失败操作的详细信息。</span><span class="sxs-lookup"><span data-stu-id="f5170-138">An application should log the details of faults and failing operations.</span></span> <span data-ttu-id="f5170-139">此信息对操作员比较有用。</span><span class="sxs-lookup"><span data-stu-id="f5170-139">This information is useful to operators.</span></span> <span data-ttu-id="f5170-140">如果某个服务频繁不可用或繁忙，通常是由于该服务已耗尽了其资源。</span><span class="sxs-lookup"><span data-stu-id="f5170-140">If a service is frequently unavailable or busy, it's often because the service has exhausted its resources.</span></span> <span data-ttu-id="f5170-141">可以通过横向扩展该服务来降低出现这些错误的频率。</span><span class="sxs-lookup"><span data-stu-id="f5170-141">You can reduce the frequency of these faults by scaling out the service.</span></span> <span data-ttu-id="f5170-142">例如，如果某个数据库服务持续过载，则对数据库进行分区并将负载分布到多个服务器中可能有助于解决问题。</span><span class="sxs-lookup"><span data-stu-id="f5170-142">For example, if a database service is continually overloaded, it might be beneficial to partition the database and spread the load across multiple servers.</span></span>

> <span data-ttu-id="f5170-143">[Microsoft Entity Framework](https://docs.microsoft.com/ef/) 提供了用于重试数据库操作的设施。</span><span class="sxs-lookup"><span data-stu-id="f5170-143">[Microsoft Entity Framework](https://docs.microsoft.com/ef/) provides facilities for retrying database operations.</span></span> <span data-ttu-id="f5170-144">另外，大多数 Azure 服务和客户端 SDK 都提供了重试机制。</span><span class="sxs-lookup"><span data-stu-id="f5170-144">Also, most Azure services and client SDKs include a retry mechanism.</span></span> <span data-ttu-id="f5170-145">有关详细信息，请参阅[特定服务的重试指南](https://docs.microsoft.com/en-us/azure/architecture/best-practices/retry-service-specific)。</span><span class="sxs-lookup"><span data-stu-id="f5170-145">For more information, see [Retry guidance for specific services](https://docs.microsoft.com/en-us/azure/architecture/best-practices/retry-service-specific).</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="f5170-146">问题和注意事项</span><span class="sxs-lookup"><span data-stu-id="f5170-146">Issues and considerations</span></span>

<span data-ttu-id="f5170-147">在决定如何实现此模式时，应考虑以下几点。</span><span class="sxs-lookup"><span data-stu-id="f5170-147">You should consider the following points when deciding how to implement this pattern.</span></span>

<span data-ttu-id="f5170-148">应当对重试策略进行调整以匹配应用程序的业务要求和故障性质。</span><span class="sxs-lookup"><span data-stu-id="f5170-148">The retry policy should be tuned to match the business requirements of the application and the nature of the failure.</span></span> <span data-ttu-id="f5170-149">对于某些非关键操作，最好是快速失败而不是重试多次并影响应用程序的吞吐量。</span><span class="sxs-lookup"><span data-stu-id="f5170-149">For some noncritical operations, it's better to fail fast rather than retry several times and impact the throughput of the application.</span></span> <span data-ttu-id="f5170-150">例如，在访问远程服务的交互式 Web 应用程序中，最好是在重试较少次数后失败并且重试尝试之间的延迟时间应当很短，而且最好向用户显示合适的消息（例如“请稍后重试”）。</span><span class="sxs-lookup"><span data-stu-id="f5170-150">For example, in an interactive web application accessing a remote service, it's better to fail after a smaller number of retries with only a short delay between retry attempts, and display a suitable message to the user (for example, “please try again later”).</span></span> <span data-ttu-id="f5170-151">对于批处理应用程序，增加重试尝试次数并且在尝试之间采用呈指数级增长的延迟时间可能更为合适。</span><span class="sxs-lookup"><span data-stu-id="f5170-151">For a batch application, it might be more appropriate to increase the number of retry attempts with an exponentially increasing delay between attempts.</span></span>

<span data-ttu-id="f5170-152">对于运行状况已接近或处于其容量上限的繁忙服务，如果采用尝试延迟时间间隔最小且尝试次数较多的积极重试策略，则可能会进一步降低性能。</span><span class="sxs-lookup"><span data-stu-id="f5170-152">An aggressive retry policy with minimal delay between attempts, and a large number of retries, could further degrade a busy service that's running close to or at capacity.</span></span> <span data-ttu-id="f5170-153">如果此重试策略不断尝试执行失败的操作，则它还可能会影响应用程序的响应能力。</span><span class="sxs-lookup"><span data-stu-id="f5170-153">This retry policy could also affect the responsiveness of the application if it's continually trying to perform a failing operation.</span></span>

<span data-ttu-id="f5170-154">如果某个请求在进行大量的重试后失败，则应用程序最好是阻止发往同一资源的后续请求并立即报告失败。</span><span class="sxs-lookup"><span data-stu-id="f5170-154">If a request still fails after a significant number of retries, it's better for the application to prevent further requests going to the same resource and simply report a failure immediately.</span></span> <span data-ttu-id="f5170-155">当期限过期后，应用程序可以试探性地允许一个或多个请求通过以查看它们是否成功。</span><span class="sxs-lookup"><span data-stu-id="f5170-155">When the period expires, the application can tentatively allow one or more requests through to see whether they're successful.</span></span> <span data-ttu-id="f5170-156">有关此策略的详细信息，请参阅[断路器模式](circuit-breaker.md)。</span><span class="sxs-lookup"><span data-stu-id="f5170-156">For more details of this strategy, see the [Circuit Breaker pattern](circuit-breaker.md).</span></span>

<span data-ttu-id="f5170-157">请考虑操作是否是幂等的。</span><span class="sxs-lookup"><span data-stu-id="f5170-157">Consider whether the operation is idempotent.</span></span> <span data-ttu-id="f5170-158">如果是，则可以放心地进行重试。</span><span class="sxs-lookup"><span data-stu-id="f5170-158">If so, it's inherently safe to retry.</span></span> <span data-ttu-id="f5170-159">否则，重试可能会导致操作执行多次并产生意外的副作用。</span><span class="sxs-lookup"><span data-stu-id="f5170-159">Otherwise, retries could cause the operation to be executed more than once, with unintended side effects.</span></span> <span data-ttu-id="f5170-160">例如，某个服务可以收到请求，成功处理该请求，但无法发送响应。</span><span class="sxs-lookup"><span data-stu-id="f5170-160">For example, a service might receive the request, process the request successfully, but fail to send a response.</span></span> <span data-ttu-id="f5170-161">此时，重试逻辑可能会认为第一个请求没有收到并重新发送请求。</span><span class="sxs-lookup"><span data-stu-id="f5170-161">At that point, the retry logic might re-send the request, assuming that the first request wasn't received.</span></span>

<span data-ttu-id="f5170-162">对服务的请求可能会因各种原因而失败并引发不同的异常，具体取决于故障性质。</span><span class="sxs-lookup"><span data-stu-id="f5170-162">A request to a service can fail for a variety of reasons raising different exceptions depending on the nature of the failure.</span></span> <span data-ttu-id="f5170-163">某些异常表明故障可以快速解决，而另一些异常表明故障会持续较长时间。</span><span class="sxs-lookup"><span data-stu-id="f5170-163">Some exceptions indicate a failure that can be resolved quickly, while others indicate that the failure is longer lasting.</span></span> <span data-ttu-id="f5170-164">根据异常类型为重试策略调整重试尝试之间的时间间隔会起作用。</span><span class="sxs-lookup"><span data-stu-id="f5170-164">It's useful for the retry policy to adjust the time between retry attempts based on the type of the exception.</span></span>

<span data-ttu-id="f5170-165">请考虑属于事务一部分的操作将如何影响总体的事务一致性。</span><span class="sxs-lookup"><span data-stu-id="f5170-165">Consider how retrying an operation that's part of a transaction will affect the overall transaction consistency.</span></span> <span data-ttu-id="f5170-166">请优调事务操作的重试策略以尽量提高成功几率并降低撤消所有事务步骤的需求。</span><span class="sxs-lookup"><span data-stu-id="f5170-166">Fine tune the retry policy for transactional operations to maximize the chance of success and reduce the need to undo all the transaction steps.</span></span>

<span data-ttu-id="f5170-167">请确保针对各种故障状况充分测试重试代码。</span><span class="sxs-lookup"><span data-stu-id="f5170-167">Ensure that all retry code is fully tested against a variety of failure conditions.</span></span> <span data-ttu-id="f5170-168">请检查并确保它不会严重影响应用程序的性能或可靠性、不会导致服务和资源过载，不会导致争用状况或瓶颈。</span><span class="sxs-lookup"><span data-stu-id="f5170-168">Check that it doesn't severely impact the performance or reliability of the application, cause excessive load on services and resources, or generate race conditions or bottlenecks.</span></span>

<span data-ttu-id="f5170-169">只有充分了解失败操作的完整上下文后才应实现重试逻辑。</span><span class="sxs-lookup"><span data-stu-id="f5170-169">Implement retry logic only where the full context of a failing operation is understood.</span></span> <span data-ttu-id="f5170-170">例如，如果某个任务包含的重试策略会调用也包含重试策略的另一任务，则这一层额外的重试可能会给处理增加很长的延迟。</span><span class="sxs-lookup"><span data-stu-id="f5170-170">For example, if a task that contains a retry policy invokes another task that also contains a retry policy, this extra layer of retries can add long delays to the processing.</span></span> <span data-ttu-id="f5170-171">更好的解决方案可能是将较低级别的任务配置为快速失败并将失败原因报告给调用它的任务。</span><span class="sxs-lookup"><span data-stu-id="f5170-171">It might be better to configure the lower-level task to fail fast and report the reason for the failure back to the task that invoked it.</span></span> <span data-ttu-id="f5170-172">然后，此较高级别的任务可以根据自己的策略处理失败。</span><span class="sxs-lookup"><span data-stu-id="f5170-172">This higher-level task can then handle the failure based on its own policy.</span></span>

<span data-ttu-id="f5170-173">请务必记录导致重试的所有连接故障，以便可以查明应用程序、服务或资源的底层问题。</span><span class="sxs-lookup"><span data-stu-id="f5170-173">It's important to log all connectivity failures that cause a retry so that underlying problems with the application, services, or resources can be identified.</span></span>

<span data-ttu-id="f5170-174">请调查服务或资源最有可能发生的错误以查明它们可能持续很长时间还是已处于末期。</span><span class="sxs-lookup"><span data-stu-id="f5170-174">Investigate the faults that are most likely to occur for a service or a resource to discover if they're likely to be long lasting or terminal.</span></span> <span data-ttu-id="f5170-175">如果可能持续很长时间，则最好将错误作为异常进行处理。</span><span class="sxs-lookup"><span data-stu-id="f5170-175">If they are, it's better to handle the fault as an exception.</span></span> <span data-ttu-id="f5170-176">应用程序可以报告或记录异常，然后尝试通过调用备用服务（如果有）或通过提供降级的功能来继续运行。</span><span class="sxs-lookup"><span data-stu-id="f5170-176">The application can report or log the exception, and then try to continue either by invoking an alternative service (if one is available), or by offering degraded functionality.</span></span> <span data-ttu-id="f5170-177">有关如何检测和处理持续时间很长的错误的详细信息，请参阅[断路器模式](circuit-breaker.md)。</span><span class="sxs-lookup"><span data-stu-id="f5170-177">For more information on how to detect and handle long-lasting faults, see the [Circuit Breaker pattern](circuit-breaker.md).</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="f5170-178">何时使用此模式</span><span class="sxs-lookup"><span data-stu-id="f5170-178">When to use this pattern</span></span>

<span data-ttu-id="f5170-179">当应用程序与远程服务进行交互或者访问远程资源时可能会遇到暂时性错误时，请使用此模式。</span><span class="sxs-lookup"><span data-stu-id="f5170-179">Use this pattern when an application could experience transient faults as it interacts with a remote service or accesses a remote resource.</span></span> <span data-ttu-id="f5170-180">这些错误预计只会短时存在，并且通过后续尝试重复执行之前失败的请求可能会成功。</span><span class="sxs-lookup"><span data-stu-id="f5170-180">These faults are expected to be short lived, and repeating a request that has previously failed could succeed on a subsequent attempt.</span></span>

<span data-ttu-id="f5170-181">在下列情况下，此模式可能不适用：</span><span class="sxs-lookup"><span data-stu-id="f5170-181">This pattern might not be useful:</span></span>

- <span data-ttu-id="f5170-182">当错误可能会持续很长时间时，因为此模式可能会影响应用程序的响应能力。</span><span class="sxs-lookup"><span data-stu-id="f5170-182">When a fault is likely to be long lasting, because this can affect the responsiveness of an application.</span></span> <span data-ttu-id="f5170-183">如果应用程序尝试重复执行可能会失败的请求，可能会浪费时间和资源。</span><span class="sxs-lookup"><span data-stu-id="f5170-183">The application might be wasting time and resources trying to repeat a request that's likely to fail.</span></span>
- <span data-ttu-id="f5170-184">处理不是由于出现暂时性错误而导致的故障，例如，由应用程序的业务逻辑中的错误导致的内部异常。</span><span class="sxs-lookup"><span data-stu-id="f5170-184">For handling failures that aren't due to transient faults, such as internal exceptions caused by errors in the business logic of an application.</span></span>
- <span data-ttu-id="f5170-185">作为替代方法来解决系统中的可伸缩性问题。</span><span class="sxs-lookup"><span data-stu-id="f5170-185">As an alternative to addressing scalability issues in a system.</span></span> <span data-ttu-id="f5170-186">如果应用程序遇到频繁的繁忙错误，则这通常表明应当纵向扩展正在访问的服务或资源。</span><span class="sxs-lookup"><span data-stu-id="f5170-186">If an application experiences frequent busy faults, it's often a sign that the service or resource being accessed should be scaled up.</span></span>

## <a name="example"></a><span data-ttu-id="f5170-187">示例</span><span class="sxs-lookup"><span data-stu-id="f5170-187">Example</span></span>

<span data-ttu-id="f5170-188">以 C# 编写的此示例展示了重试模式的实现。</span><span class="sxs-lookup"><span data-stu-id="f5170-188">This example in C# illustrates an implementation of the Retry pattern.</span></span> <span data-ttu-id="f5170-189">下面显示的 `OperationWithBasicRetryAsync` 方法通过 `TransientOperationAsync` 方法以异步方式调用一个外部服务。</span><span class="sxs-lookup"><span data-stu-id="f5170-189">The `OperationWithBasicRetryAsync` method, shown below, invokes an external service asynchronously through the `TransientOperationAsync` method.</span></span> <span data-ttu-id="f5170-190">`TransientOperationAsync` 方法的细节特定于该服务，示例代码中省略了其细节。</span><span class="sxs-lookup"><span data-stu-id="f5170-190">The details of the `TransientOperationAsync` method will be specific to the service and are omitted from the sample code.</span></span>

```csharp
private int retryCount = 3;
private readonly TimeSpan delay = TimeSpan.FromSeconds(5);

public async Task OperationWithBasicRetryAsync()
{
  int currentRetry = 0;

  for (;;)
  {
    try
    {
      // Call external service.
      await TransientOperationAsync();

      // Return or break.
      break;
    }
    catch (Exception ex)
    {
      Trace.TraceError("Operation Exception");

      currentRetry++;

      // Check if the exception thrown was a transient exception
      // based on the logic in the error detection strategy.
      // Determine whether to retry the operation, as well as how
      // long to wait, based on the retry strategy.
      if (currentRetry > this.retryCount || !IsTransient(ex))
      {
        // If this isn't a transient error or we shouldn't retry, 
        // rethrow the exception.
        throw;
      }
    }

    // Wait to retry the operation.
    // Consider calculating an exponential delay here and
    // using a strategy best suited for the operation and fault.
    await Task.Delay(delay);
  }
}

// Async method that wraps a call to a remote service (details not shown).
private async Task TransientOperationAsync()
{
  ...
}
```

<span data-ttu-id="f5170-191">包装在 for 循环中的一个 try/catch 块包含了调用此方法的语句。</span><span class="sxs-lookup"><span data-stu-id="f5170-191">The statement that invokes this method is contained in a try/catch block wrapped in a for loop.</span></span> <span data-ttu-id="f5170-192">如果对 `TransientOperationAsync` 方法的调用成功且没有引发异常，则 for 循环会退出。</span><span class="sxs-lookup"><span data-stu-id="f5170-192">The for loop exits if the call to the `TransientOperationAsync` method succeeds without throwing an exception.</span></span> <span data-ttu-id="f5170-193">如果 `TransientOperationAsync` 方法失败，则 catch 块会检查失败原因。</span><span class="sxs-lookup"><span data-stu-id="f5170-193">If the `TransientOperationAsync` method fails, the catch block examines the reason for the failure.</span></span> <span data-ttu-id="f5170-194">如果认为失败原因是某个暂时性错误，则代码将等待很短的延迟时间，然后重试该操作。</span><span class="sxs-lookup"><span data-stu-id="f5170-194">If it's believed to be a transient error the code waits for a short delay before retrying the operation.</span></span>

<span data-ttu-id="f5170-195">For 循环还会跟踪已尝试的操作次数，并且如果代码失败三次，则会认为异常将持续更长的时间。</span><span class="sxs-lookup"><span data-stu-id="f5170-195">The for loop also tracks the number of times that the operation has been attempted, and if the code fails three times the exception is assumed to be more long lasting.</span></span> <span data-ttu-id="f5170-196">如果异常不是暂时性的或者它持续很长时间，则 catch 处理程序会引发异常。</span><span class="sxs-lookup"><span data-stu-id="f5170-196">If the exception isn't transient or it's long lasting, the catch handler throws an exception.</span></span> <span data-ttu-id="f5170-197">此异常会使 for 循环退出，应当由调用 `OperationWithBasicRetryAsync` 方法的代码捕获此异常。</span><span class="sxs-lookup"><span data-stu-id="f5170-197">This exception exits the for loop and should be caught by the code that invokes the `OperationWithBasicRetryAsync` method.</span></span>

<span data-ttu-id="f5170-198">下面显示的 `IsTransient` 方法检查是否存在与在其中运行代码的环境相关的一组特定异常。</span><span class="sxs-lookup"><span data-stu-id="f5170-198">The `IsTransient` method, shown below, checks for a specific set of exceptions that are relevant to the environment the code is run in.</span></span> <span data-ttu-id="f5170-199">暂时性异常的定义根据正在访问的资源以及在其中执行操作的环境而异。</span><span class="sxs-lookup"><span data-stu-id="f5170-199">The definition of a transient exception will vary according to the resources being accessed and the environment the operation is being performed in.</span></span>

```csharp
private bool IsTransient(Exception ex)
{
  // Determine if the exception is transient.
  // In some cases this is as simple as checking the exception type, in other
  // cases it might be necessary to inspect other properties of the exception.
  if (ex is OperationTransientException)
    return true;

  var webException = ex as WebException;
  if (webException != null)
  {
    // If the web exception contains one of the following status values
    // it might be transient.
    return new[] {WebExceptionStatus.ConnectionClosed,
                  WebExceptionStatus.Timeout,
                  WebExceptionStatus.RequestCanceled }.
            Contains(webException.Status);
  }

  // Additional exception checking logic goes here.
  return false;
}
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="f5170-200">相关模式和指南</span><span class="sxs-lookup"><span data-stu-id="f5170-200">Related patterns and guidance</span></span>

- <span data-ttu-id="f5170-201">[断路器模式](circuit-breaker.md)。</span><span class="sxs-lookup"><span data-stu-id="f5170-201">[Circuit Breaker pattern](circuit-breaker.md).</span></span> <span data-ttu-id="f5170-202">重试模式可用于处理暂时性错误。</span><span class="sxs-lookup"><span data-stu-id="f5170-202">The Retry pattern is useful for handling transient faults.</span></span> <span data-ttu-id="f5170-203">如果预计故障会持续较长时间，则实现断路器模式可能更为合适。</span><span class="sxs-lookup"><span data-stu-id="f5170-203">If a failure is expected to be more long lasting, it might be more appropriate to implement the Circuit Breaker pattern.</span></span> <span data-ttu-id="f5170-204">还可以将重试模式与断路器配合使用来提供用于处理错误的综合方法。</span><span class="sxs-lookup"><span data-stu-id="f5170-204">The Retry pattern can also be used in conjunction with a circuit breaker to provide a comprehensive approach to handling faults.</span></span>
- [<span data-ttu-id="f5170-205">特定服务的重试指南</span><span class="sxs-lookup"><span data-stu-id="f5170-205">Retry guidance for specific services</span></span>](https://docs.microsoft.com/en-us/azure/architecture/best-practices/retry-service-specific)
- [<span data-ttu-id="f5170-206">连接复原</span><span class="sxs-lookup"><span data-stu-id="f5170-206">Connection Resiliency</span></span>](https://docs.microsoft.com/en-us/ef/core/miscellaneous/connection-resiliency)
