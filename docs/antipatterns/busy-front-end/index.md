---
title: "繁忙前端对立模式"
description: "大量后台线程中的异步工作可能会耗尽其他前台任务的资源。"
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: cedb80ddac5ceb1eb901455df3165993fd28a138
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="busy-front-end-antipattern"></a><span data-ttu-id="da140-103">繁忙前端对立模式</span><span class="sxs-lookup"><span data-stu-id="da140-103">Busy Front End antipattern</span></span>

<span data-ttu-id="da140-104">在大量后台线程中执行异步工作可能耗尽其他并发前台任务的资源，将响应时间降低到不可接受的水平。</span><span class="sxs-lookup"><span data-stu-id="da140-104">Performing asynchronous work on a large number of background threads can starve other concurrent foreground tasks of resources, decreasing response times to unacceptable levels.</span></span>

## <a name="problem-description"></a><span data-ttu-id="da140-105">问题描述</span><span class="sxs-lookup"><span data-stu-id="da140-105">Problem description</span></span>

<span data-ttu-id="da140-106">资源密集型任务可能增大用户请求的响应时间，使延迟提高。</span><span class="sxs-lookup"><span data-stu-id="da140-106">Resource-intensive tasks can increase the response times for user requests and cause high latency.</span></span> <span data-ttu-id="da140-107">缩短响应时间的方法之一是将资源密集型任务卸载到独立的线程。</span><span class="sxs-lookup"><span data-stu-id="da140-107">One way to improve response times is to offload a resource-intensive task to a separate thread.</span></span> <span data-ttu-id="da140-108">此方法可让应用程序保持响应能力，同时可在后台进行处理。</span><span class="sxs-lookup"><span data-stu-id="da140-108">This approach lets the application stay responsive while processing happens in the background.</span></span> <span data-ttu-id="da140-109">但是，在后台线程中运行的任务仍会占用资源。</span><span class="sxs-lookup"><span data-stu-id="da140-109">However, tasks that run on a background thread still consume resources.</span></span> <span data-ttu-id="da140-110">如果这些任务过多，可能会耗尽用于处理请求的线程。</span><span class="sxs-lookup"><span data-stu-id="da140-110">If there are too many of them, they can starve the threads that are handling requests.</span></span>

> [!NOTE]
> <span data-ttu-id="da140-111">术语“资源”涵盖许多指标，例如 CPU 利用率、内存占用率以及网络或磁盘 I/O。</span><span class="sxs-lookup"><span data-stu-id="da140-111">The term *resource* can encompass many things, such as CPU utilization, memory occupancy, and network or disk I/O.</span></span>

<span data-ttu-id="da140-112">如果应用程序是作为整体性的代码段开发的，其中的业务逻辑合并成为一个与呈现层共享的层，则通常就会发生此问题。</span><span class="sxs-lookup"><span data-stu-id="da140-112">This problem typically occurs when an application is developed as monolithic piece of code, with all of the business logic combined into a single tier shared with the presentation layer.</span></span>

<span data-ttu-id="da140-113">以下示例使用 ASP.NET 来演示该问题。</span><span class="sxs-lookup"><span data-stu-id="da140-113">Here’s an example using ASP.NET that demonstrates the problem.</span></span> <span data-ttu-id="da140-114">可在[此处][code-sample]找到完整示例。</span><span class="sxs-lookup"><span data-stu-id="da140-114">You can find the complete sample [here][code-sample].</span></span>

```csharp
public class WorkInFrontEndController : ApiController
{
    [HttpPost]
    [Route("api/workinfrontend")]
    public HttpResponseMessage Post()
    {
        new Thread(() =>
        {
            //Simulate processing
            Thread.SpinWait(Int32.MaxValue / 100);
        }).Start();

        return Request.CreateResponse(HttpStatusCode.Accepted);
    }
}

public class UserProfileController : ApiController
{
    [HttpGet]
    [Route("api/userprofile/{id}")]
    public UserProfile Get(int id)
    {
        //Simulate processing
        return new UserProfile() { FirstName = "Alton", LastName = "Hudgens" };
    }
}
```

- <span data-ttu-id="da140-115">`WorkInFrontEnd` 控制器中的 `Post` 方法实现 HTTP POST 操作。</span><span class="sxs-lookup"><span data-stu-id="da140-115">The `Post` method in the `WorkInFrontEnd` controller implements an HTTP POST operation.</span></span> <span data-ttu-id="da140-116">此操作模拟一个长时间运行的 CPU 密集型任务。</span><span class="sxs-lookup"><span data-stu-id="da140-116">This operation simulates a long-running, CPU-intensive task.</span></span> <span data-ttu-id="da140-117">工作在独立的线程中执行，以尝试使 POST 操作快速完成。</span><span class="sxs-lookup"><span data-stu-id="da140-117">The work is performed on a separate thread, in an attempt to enable the POST operation to complete quickly.</span></span>

- <span data-ttu-id="da140-118">`UserProfile` 控制器中的 `Get` 方法实现 HTTP GET 操作。</span><span class="sxs-lookup"><span data-stu-id="da140-118">The `Get` method in the `UserProfile` controller implements an HTTP GET operation.</span></span> <span data-ttu-id="da140-119">此方法消耗的 CPU 资源要少得多。</span><span class="sxs-lookup"><span data-stu-id="da140-119">This method is much less CPU intensive.</span></span>

<span data-ttu-id="da140-120">`Post` 方法的资源要求是主要考虑因素。</span><span class="sxs-lookup"><span data-stu-id="da140-120">The primary concern is the resource requirements of the `Post` method.</span></span> <span data-ttu-id="da140-121">尽管此方法会将工作放入后台线程，但该工作仍会占用相当多的 CPU 资源。</span><span class="sxs-lookup"><span data-stu-id="da140-121">Although it puts the work onto a background thread, the work can still consume considerable CPU resources.</span></span> <span data-ttu-id="da140-122">这些资源与其他并发用户执行的其他操作共享。</span><span class="sxs-lookup"><span data-stu-id="da140-122">These resources are shared with other operations being performed by other concurrent users.</span></span> <span data-ttu-id="da140-123">如果适度数量的用户同时发送此请求，整体性能可能会受到影响，使所有操作的速度变慢。</span><span class="sxs-lookup"><span data-stu-id="da140-123">If a moderate number of users send this request at the same time, overall performance is likely to suffer, slowing down all operations.</span></span> <span data-ttu-id="da140-124">例如，用户在运行 `Get` 方法时可能会遇到明显的延迟。</span><span class="sxs-lookup"><span data-stu-id="da140-124">Users might experience significant latency in the `Get` method, for example.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="da140-125">如何解决问题</span><span class="sxs-lookup"><span data-stu-id="da140-125">How to fix the problem</span></span>

<span data-ttu-id="da140-126">将消耗大量资源的进程移到独立的后端。</span><span class="sxs-lookup"><span data-stu-id="da140-126">Move processes that consume significant resources to a separate back end.</span></span> 

<span data-ttu-id="da140-127">使用这种方法，前端会将资源密集型任务放入某个消息队列。</span><span class="sxs-lookup"><span data-stu-id="da140-127">With this approach, the front end puts resource-intensive tasks onto a message queue.</span></span> <span data-ttu-id="da140-128">后端会拾取这些任务进行异步处理。</span><span class="sxs-lookup"><span data-stu-id="da140-128">The back end picks up the tasks for asynchronous processing.</span></span> <span data-ttu-id="da140-129">该队列还充当负载调控器，可缓冲后端的请求。</span><span class="sxs-lookup"><span data-stu-id="da140-129">The queue also acts as a load leveler, buffering requests for the back end.</span></span> <span data-ttu-id="da140-130">如果队列长度过长，你可以配置自动缩放，以横向扩展后端。</span><span class="sxs-lookup"><span data-stu-id="da140-130">If the queue length becomes too long, you can configure autoscaling to scale out the back end.</span></span>

<span data-ttu-id="da140-131">下面是上述代码的修改版本。</span><span class="sxs-lookup"><span data-stu-id="da140-131">Here is a revised version of the previous code.</span></span> <span data-ttu-id="da140-132">在此版本中，`Post` 方法将消息放在服务总线队列中。</span><span class="sxs-lookup"><span data-stu-id="da140-132">In this version, the `Post` method puts a message on a Service Bus queue.</span></span> 

```csharp
public class WorkInBackgroundController : ApiController
{
    private static readonly QueueClient QueueClient;
    private static readonly string QueueName;
    private static readonly ServiceBusQueueHandler ServiceBusQueueHandler;

    public WorkInBackgroundController()
    {
        var serviceBusConnectionString = ...;
        QueueName = ...;
        ServiceBusQueueHandler = new ServiceBusQueueHandler(serviceBusConnectionString);
        QueueClient = ServiceBusQueueHandler.GetQueueClientAsync(QueueName).Result;
    }

    [HttpPost]
    [Route("api/workinbackground")]
    public async Task<long> Post()
    {
        return await ServiceBusQueuehandler.AddWorkLoadToQueueAsync(QueueClient, QueueName, 0);
    }
}
```

<span data-ttu-id="da140-133">后端从服务总线队列中提取消息并执行处理。</span><span class="sxs-lookup"><span data-stu-id="da140-133">The back end pulls messages from the Service Bus queue and does the processing.</span></span>

```csharp
public async Task RunAsync(CancellationToken cancellationToken)
{
    this._queueClient.OnMessageAsync(
        // This lambda is invoked for each message received.
        async (receivedMessage) =>
        {
            try
            {
                // Simulate processing of message
                Thread.SpinWait(Int32.Maxvalue / 1000);

                await receivedMessage.CompleteAsync();
            }
            catch
            {
                receivedMessage.Abandon();
            }
        });
}
```

## <a name="considerations"></a><span data-ttu-id="da140-134">注意事项</span><span class="sxs-lookup"><span data-stu-id="da140-134">Considerations</span></span>

- <span data-ttu-id="da140-135">此方法在一定程度上增大了应用程序的复杂性。</span><span class="sxs-lookup"><span data-stu-id="da140-135">This approach adds some additional complexity to the application.</span></span> <span data-ttu-id="da140-136">必须安全处理排队和取消排队，以免在发生故障时丢失请求。</span><span class="sxs-lookup"><span data-stu-id="da140-136">You must handle queuing and dequeuing safely to avoid losing requests in the event of a failure.</span></span>
- <span data-ttu-id="da140-137">应用程序依赖于其他服务处理消息队列。</span><span class="sxs-lookup"><span data-stu-id="da140-137">The application takes a dependency on an additional service for the message queue.</span></span>
- <span data-ttu-id="da140-138">处理环境必须有足够高的可伸缩性，可处理预期的工作负荷并满足所需的吞吐量目标。</span><span class="sxs-lookup"><span data-stu-id="da140-138">The processing environment must be sufficiently scalable to handle the expected workload and meet the required throughput targets.</span></span>
- <span data-ttu-id="da140-139">尽管此方法可提高总体响应能力，但移到后端的任务可能需要更长的时间才能完成。</span><span class="sxs-lookup"><span data-stu-id="da140-139">While this approach should improve overall responsiveness, the tasks that are moved to the back end may take longer to complete.</span></span> 

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="da140-140">如何检测问题</span><span class="sxs-lookup"><span data-stu-id="da140-140">How to detect the problem</span></span>

<span data-ttu-id="da140-141">繁忙前端的症状包括执行资源密集型任务时延迟偏高。</span><span class="sxs-lookup"><span data-stu-id="da140-141">Symptoms of a busy front end include high latency when resource-intensive tasks are being performed.</span></span> <span data-ttu-id="da140-142">最终用户可能会反映响应时间延长，或服务超时导致失败。这些失败还可能返回 HTTP 500（内部服务器）错误或 HTTP 503（服务不可用）错误。</span><span class="sxs-lookup"><span data-stu-id="da140-142">End users are likely to report extended response times or failures caused by services timing out. These failures could also return HTTP 500 (Internal Server) errors or HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="da140-143">检查 Web 服务器的事件日志，其中可能包含有关错误原因和情况的更详细信息。</span><span class="sxs-lookup"><span data-stu-id="da140-143">Examine the event logs for the web server, which are likely to contain more detailed information about the causes and circumstances of the errors.</span></span>

<span data-ttu-id="da140-144">可执行以下步骤来帮助识别此问题：</span><span class="sxs-lookup"><span data-stu-id="da140-144">You can perform the following steps to help identify this problem:</span></span>

1. <span data-ttu-id="da140-145">对生产系统执行进程监视，识别响应时间变长的位置。</span><span class="sxs-lookup"><span data-stu-id="da140-145">Perform process monitoring of the production system, to identify points when response times slow down.</span></span>
2. <span data-ttu-id="da140-146">检查在这些位置捕获到的遥测数据，确定所执行的混合操作以及所用的资源。</span><span class="sxs-lookup"><span data-stu-id="da140-146">Examine the telemetry data captured at these points to determine the mix of operations being performed and the resources being used.</span></span> 
3. <span data-ttu-id="da140-147">找出不佳响应时间与在这些时间发生的操作的数量与组合之间的任何关联。</span><span class="sxs-lookup"><span data-stu-id="da140-147">Find any correlations between poor response times and the volumes and combinations of operations that were happening at those times.</span></span>
4. <span data-ttu-id="da140-148">针对每个可疑的操作执行负载测试，确定哪些操作消耗了资源并在耗尽其他操作的操作。</span><span class="sxs-lookup"><span data-stu-id="da140-148">Load test each suspected operation to identify which operations are consuming resources and starving other operations.</span></span> 
5. <span data-ttu-id="da140-149">检查这些操作的源代码，确定它们导致资源消耗过度的原因。</span><span class="sxs-lookup"><span data-stu-id="da140-149">Review the source code for those operations to determine why they might cause excessive resource consumption.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="da140-150">示例诊断</span><span class="sxs-lookup"><span data-stu-id="da140-150">Example diagnosis</span></span> 

<span data-ttu-id="da140-151">以下部分将这些步骤应用到前面所述的示例应用程序。</span><span class="sxs-lookup"><span data-stu-id="da140-151">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="identify-points-of-slowdown"></a><span data-ttu-id="da140-152">识别速度变慢的位置</span><span class="sxs-lookup"><span data-stu-id="da140-152">Identify points of slowdown</span></span>

<span data-ttu-id="da140-153">检测每个方法，跟踪每个请求的持续时间和消耗的资源。</span><span class="sxs-lookup"><span data-stu-id="da140-153">Instrument each method to track the duration and resources consumed by each request.</span></span> <span data-ttu-id="da140-154">然后在生产环境中监视应用程序。</span><span class="sxs-lookup"><span data-stu-id="da140-154">Then monitor the application in production.</span></span> <span data-ttu-id="da140-155">这可以提供请求相互争用资源的总体视图。</span><span class="sxs-lookup"><span data-stu-id="da140-155">This can provide an overall view of how requests compete with each other.</span></span> <span data-ttu-id="da140-156">在受压过程中，运行速度缓慢且大量消耗资源的请求可能会影响其他操作。可以通过监视系统并关注性能下降位置来观察此行为。</span><span class="sxs-lookup"><span data-stu-id="da140-156">During periods of stress, slow-running resource-hungry requests will likely impact other operations, and this behavior can be observed by monitoring the system and noting the drop off in performance.</span></span>

<span data-ttu-id="da140-157">下图显示了监视仪表板。</span><span class="sxs-lookup"><span data-stu-id="da140-157">The following image shows a monitoring dashboard.</span></span> <span data-ttu-id="da140-158">（我们在测试中使用了 [AppDynamics]。）系统最初的负载较轻。</span><span class="sxs-lookup"><span data-stu-id="da140-158">(We used [AppDynamics] for our tests.) Initially, the system has light load.</span></span> <span data-ttu-id="da140-159">然后，用户开始请求 `UserProfile` GET 方法。</span><span class="sxs-lookup"><span data-stu-id="da140-159">Then users start requesting the `UserProfile` GET method.</span></span> <span data-ttu-id="da140-160">在其他用户开始对 `WorkInFrontEnd` POST 方法发出请求之前，性能一直相当良好。</span><span class="sxs-lookup"><span data-stu-id="da140-160">The performance is reasonably good until other users start issuing requests to the `WorkInFrontEnd` POST method.</span></span> <span data-ttu-id="da140-161">此后，响应时间急剧增大（第一个箭头）。</span><span class="sxs-lookup"><span data-stu-id="da140-161">At that point, response times increase dramatically (first arrow).</span></span> <span data-ttu-id="da140-162">只有在对 `WorkInFrontEnd` 控制器的请求量下降之后，响应时间才有改善（第二个箭头）。</span><span class="sxs-lookup"><span data-stu-id="da140-162">Response times only improve after the volume of requests to the `WorkInFrontEnd` controller diminishes (second arrow).</span></span>

![AppDynamics“业务事务”窗格，其中显示了使用 WorkInFrontEnd 控制器时对所有请求的响应时间造成的影响][AppDynamics-Transactions-Front-End-Requests]

### <a name="examine-telemetry-data-and-find-correlations"></a><span data-ttu-id="da140-164">检查遥测数据并找出关联</span><span class="sxs-lookup"><span data-stu-id="da140-164">Examine telemetry data and find correlations</span></span>

<span data-ttu-id="da140-165">下图显示了所收集的、用于监视同一时间间隔内资源利用率的指标。</span><span class="sxs-lookup"><span data-stu-id="da140-165">The next image shows some of the metrics gathered to monitor resource utilization during the same interval.</span></span> <span data-ttu-id="da140-166">一开始，只有少量的用户访问系统。</span><span class="sxs-lookup"><span data-stu-id="da140-166">At first, few users are accessing the system.</span></span> <span data-ttu-id="da140-167">随着越来越多的用户建立连接，CPU 利用率变得极高 (100%)。</span><span class="sxs-lookup"><span data-stu-id="da140-167">As more users connect, CPU utilization becomes very high (100%).</span></span> <span data-ttu-id="da140-168">另请注意，网络 I/O 速率最初是根据 CPU 使用率的提高而上升。</span><span class="sxs-lookup"><span data-stu-id="da140-168">Also notice that the network I/O rate initially goes up as CPU usage rises.</span></span> <span data-ttu-id="da140-169">但是，一旦 CPU 使用率达到峰值，网络 I/O 实际上会下降。</span><span class="sxs-lookup"><span data-stu-id="da140-169">But once CPU usage peaks, network I/O actually goes down.</span></span> <span data-ttu-id="da140-170">这是因为，一旦 CPU 达到容量限制，系统就只能处理相对较小数量的请求。</span><span class="sxs-lookup"><span data-stu-id="da140-170">That's because the system can only handle a relatively small number of requests once the CPU is at capacity.</span></span> <span data-ttu-id="da140-171">随着用户断开连接，CPU 负载逐渐降低。</span><span class="sxs-lookup"><span data-stu-id="da140-171">As users disconnect, the CPU load tails off.</span></span>

![显示 CPU 和网络利用率的 AppDynamics 指标][AppDynamics-Metrics-Front-End-Requests]

<span data-ttu-id="da140-173">此时，`WorkInFrontEnd` 控制器中的 `Post` 方法似乎就成了可供密切检查的首要候选项。</span><span class="sxs-lookup"><span data-stu-id="da140-173">At this point, it appears the `Post` method in the `WorkInFrontEnd` controller is a prime candidate for closer examination.</span></span> <span data-ttu-id="da140-174">需要在受控环境中开展更多的工作，以确认假设条件。</span><span class="sxs-lookup"><span data-stu-id="da140-174">Further work in a controlled environment is needed to confirm the hypothesis.</span></span>

### <a name="perform-load-testing"></a><span data-ttu-id="da140-175">执行负载测试</span><span class="sxs-lookup"><span data-stu-id="da140-175">Perform load testing</span></span> 

<span data-ttu-id="da140-176">下一步是在受控环境中执行测试。</span><span class="sxs-lookup"><span data-stu-id="da140-176">The next step is to perform tests in a controlled environment.</span></span> <span data-ttu-id="da140-177">例如，轮流运行一系列包含然后省略每个请求的负载测试，以查看影响。</span><span class="sxs-lookup"><span data-stu-id="da140-177">For example, run a series of load tests that include and then omit each request in turn to see the effects.</span></span>

<span data-ttu-id="da140-178">下图显示了针对上述测试中使用的相同云服务部署执行的负载测试结果。</span><span class="sxs-lookup"><span data-stu-id="da140-178">The graph below shows the results of a load test performed against an identical deployment of the cloud service used in the previous tests.</span></span> <span data-ttu-id="da140-179">该测试使用了一个常量负载，其中包含 500 个在 `UserProfile` 控制器中执行 `Get` 操作的用户；另外还使用了一个阶跃负载，其中包含在 `WorkInFrontEnd` 控制器中执行 `Post` 操作的用户。</span><span class="sxs-lookup"><span data-stu-id="da140-179">The test used a constant load of 500 users performing the `Get` operation in the `UserProfile` controller, along with a step load of users performing the `Post` operation in the `WorkInFrontEnd` controller.</span></span> 

![WorkInFrontEnd 控制器的初始负载测试结果][Initial-Load-Test-Results-Front-End]

<span data-ttu-id="da140-181">最初，阶跃负载为 0，因此，只有活动的用户在执行 `UserProfile` 请求。</span><span class="sxs-lookup"><span data-stu-id="da140-181">Initially, the step load is 0, so the only active users are performing the `UserProfile` requests.</span></span> <span data-ttu-id="da140-182">系统每秒大约能够响应 500 个请求。</span><span class="sxs-lookup"><span data-stu-id="da140-182">The system is able to respond to approximately 500 requests per second.</span></span> <span data-ttu-id="da140-183">60 秒后，包含 100 个附加用户的负载开始向 `WorkInFrontEnd` 控制器发送 POST 请求。</span><span class="sxs-lookup"><span data-stu-id="da140-183">After 60 seconds, a load of 100 additional users starts sending POST requests to the `WorkInFrontEnd` controller.</span></span> <span data-ttu-id="da140-184">几乎在同一时刻，发送到 `UserProfile` 控制器的工作负荷下降到每秒大约 150 个请求。</span><span class="sxs-lookup"><span data-stu-id="da140-184">Almost immediately, the workload sent to the `UserProfile` controller drops to about 150 requests per second.</span></span> <span data-ttu-id="da140-185">这种情况是负载测试运行程序的运行方式造成的。</span><span class="sxs-lookup"><span data-stu-id="da140-185">This is due to the way the load-test runner functions.</span></span> <span data-ttu-id="da140-186">它会在等到获取响应后再发送下一个请求，因此，它收到响应所花费的时间越长，请求速率就越低。</span><span class="sxs-lookup"><span data-stu-id="da140-186">It waits for a response before sending the next request, so the longer it takes to receive a response, the lower the request rate.</span></span>

<span data-ttu-id="da140-187">随着越来越多的用户向 `WorkInFrontEnd` 控制器发送 POST 请求，`UserProfile` 控制器的响应速率会持续下降。</span><span class="sxs-lookup"><span data-stu-id="da140-187">As more users send POST requests to the `WorkInFrontEnd` controller, the response rate of the `UserProfile` controller continues to drop.</span></span> <span data-ttu-id="da140-188">但请注意，`WorkInFrontEnd` 控制器处理的请求数量保持相对稳定。</span><span class="sxs-lookup"><span data-stu-id="da140-188">But note that the volume of requests handled by the `WorkInFrontEnd`controller remains relatively constant.</span></span> <span data-ttu-id="da140-189">随着两种请求的总速率倾向于稳定但又偏低的限制，系统的饱和度一目了然。</span><span class="sxs-lookup"><span data-stu-id="da140-189">The saturation of the system becomes apparent as the overall rate of both requests tends towards a steady but low limit.</span></span>


### <a name="review-the-source-code"></a><span data-ttu-id="da140-190">检查源代码</span><span class="sxs-lookup"><span data-stu-id="da140-190">Review the source code</span></span>

<span data-ttu-id="da140-191">最后一步是查看源代码。</span><span class="sxs-lookup"><span data-stu-id="da140-191">The final step is to look at the source code.</span></span> <span data-ttu-id="da140-192">开发团队已了解到，`Post` 方法可能需要花费相当长的时间，正因如此，原始实现使用了独立的线程。</span><span class="sxs-lookup"><span data-stu-id="da140-192">The development team was aware that the `Post` method could take a considerable amount of time, which is why the original implementation used a separate thread.</span></span> <span data-ttu-id="da140-193">这种做法解决了眼下的问题，因为 `Post` 方法在等待长时间运行的任务完成期间未阻塞。</span><span class="sxs-lookup"><span data-stu-id="da140-193">That solved the immediate problem, because the `Post` method did not block waiting for a long-running task to complete.</span></span>

<span data-ttu-id="da140-194">但是，此方法执行的工作仍会消耗 CPU、内存和其他资源。</span><span class="sxs-lookup"><span data-stu-id="da140-194">However, the work performed by this method still consumes CPU, memory, and other resources.</span></span> <span data-ttu-id="da140-195">让此进程以异步方式运行实际上可能会损害性能，因为用户能够以不受控的方式同时大量触发这些操作。</span><span class="sxs-lookup"><span data-stu-id="da140-195">Enabling this process to run asynchronously might actually damage performance, as users can trigger a large number of these operations simultaneously, in an uncontrolled manner.</span></span> <span data-ttu-id="da140-196">服务器可以运行的线程数有限制。</span><span class="sxs-lookup"><span data-stu-id="da140-196">There is a limit to the number of threads that a server can run.</span></span> <span data-ttu-id="da140-197">如果超过此限制，应用程序在尝试启动新线程时可能发生异常。</span><span class="sxs-lookup"><span data-stu-id="da140-197">Past this limit, the application is likely to get an exception when it tries to start a new thread.</span></span>

> [!NOTE]
> <span data-ttu-id="da140-198">这并不意味着应该避免异步操作。</span><span class="sxs-lookup"><span data-stu-id="da140-198">This doesn't mean you should avoid asynchronous operations.</span></span> <span data-ttu-id="da140-199">在网络调用中执行异步等待是建议的做法。</span><span class="sxs-lookup"><span data-stu-id="da140-199">Performing an asynchronous await on a network call is a recommended practice.</span></span> <span data-ttu-id="da140-200">（请参阅[同步 I/O][sync-io] 对立模式。）此处的问题在于，CPU 密集型工作已在另一个线程中衍生。</span><span class="sxs-lookup"><span data-stu-id="da140-200">(See the [Synchronous I/O][sync-io] antipattern.) The problem here is that CPU-intensive work was spawned on another thread.</span></span> 

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="da140-201">实施解决方案并验证结果</span><span class="sxs-lookup"><span data-stu-id="da140-201">Implement the solution and verify the result</span></span>

<span data-ttu-id="da140-202">下图显示了实施解决方案后的性能监视。</span><span class="sxs-lookup"><span data-stu-id="da140-202">The following image shows performance monitoring after the solution was implemented.</span></span> <span data-ttu-id="da140-203">负载与前面所示的类似，但 `UserProfile` 控制器的响应时间现在要快得多。</span><span class="sxs-lookup"><span data-stu-id="da140-203">The load was similar to that shown earlier, but the response times for the `UserProfile` controller are now much faster.</span></span> <span data-ttu-id="da140-204">在相同的持续时间内，请求数量已从 2,759 增大到 23,565。</span><span class="sxs-lookup"><span data-stu-id="da140-204">The volume of requests increased over the same duration, from 2,759 to 23,565.</span></span> 

![AppDynamics“业务事务”窗格，其中显示了使用 WorkInBackground 控制器时对所有请求的响应时间造成的影响][AppDynamics-Transactions-Background-Requests]

<span data-ttu-id="da140-206">请注意，`WorkInBackground` 控制器还处理了多得多的请求。</span><span class="sxs-lookup"><span data-stu-id="da140-206">Note that the `WorkInBackground` controller also handled a much larger volume of requests.</span></span> <span data-ttu-id="da140-207">但是，在此情况下无法进行直接的比较，因为在此控制器中执行的工作与原始代码有很大的差别。</span><span class="sxs-lookup"><span data-stu-id="da140-207">However, you can't make a direct comparison in this case, because the work being performed in this controller is very different from the original code.</span></span> <span data-ttu-id="da140-208">新版本只是将请求排队，而没有执行耗时的计算。</span><span class="sxs-lookup"><span data-stu-id="da140-208">The new version simply queues a request, rather than performing a time consuming calculation.</span></span> <span data-ttu-id="da140-209">要点是，此方法已不再拖累整个承受负载的系统。</span><span class="sxs-lookup"><span data-stu-id="da140-209">The main point is that this method no longer drags down the entire system under load.</span></span>

<span data-ttu-id="da140-210">CPU 和网络利用率也表明性能有了改进。</span><span class="sxs-lookup"><span data-stu-id="da140-210">CPU and network utilization also show the improved performance.</span></span> <span data-ttu-id="da140-211">CPU 利用率从未达到过 100%，处理的网络请求数量远远超过前面的测试，并且在工作负荷降低之前未曾下降。</span><span class="sxs-lookup"><span data-stu-id="da140-211">The CPU utilization never reached 100%, and the volume of handled network requests was far greater than earlier, and did not tail off until the workload dropped.</span></span>

![显示 WorkInBackground 控制器 CPU 和网络利用率的 AppDynamics 指标][AppDynamics-Metrics-Background-Requests]

<span data-ttu-id="da140-213">下图显示了负载测试的结果。</span><span class="sxs-lookup"><span data-stu-id="da140-213">The following graph shows the results of a load test.</span></span> <span data-ttu-id="da140-214">与前面的测试相比，所服务的请求总数量有了极大的改进。</span><span class="sxs-lookup"><span data-stu-id="da140-214">The overall volume of requests serviced is greatly improved compared to the the earlier tests.</span></span>

![BackgroundImageProcessing 控制器的负载测试结果][Load-Test-Results-Background]

## <a name="related-guidance"></a><span data-ttu-id="da140-216">相关指南</span><span class="sxs-lookup"><span data-stu-id="da140-216">Related guidance</span></span>

- <span data-ttu-id="da140-217">[有关自动缩放的最佳做法][autoscaling]</span><span class="sxs-lookup"><span data-stu-id="da140-217">[Autoscaling best practices][autoscaling]</span></span>
- <span data-ttu-id="da140-218">[有关后台作业的最佳做法][background-jobs]</span><span class="sxs-lookup"><span data-stu-id="da140-218">[Background jobs best practices][background-jobs]</span></span>
- <span data-ttu-id="da140-219">[基于队列的负载调控模式][load-leveling]</span><span class="sxs-lookup"><span data-stu-id="da140-219">[Queue-Based Load Leveling pattern][load-leveling]</span></span>
- <span data-ttu-id="da140-220">[Web 队列辅助角色体系结构样式][web-queue-worker]</span><span class="sxs-lookup"><span data-stu-id="da140-220">[Web Queue Worker architecture style][web-queue-worker]</span></span>

[AppDyanamics]: https://www.appdynamics.com/
[autoscaling]: /azure/architecture/best-practices/auto-scaling
[background-jobs]: /azure/architecture/best-practices/background-jobs
[code-sample]: https://github.com/mspnp/performance-optimization/tree/master/BusyFrontEnd
[fullDemonstrationOfSolution]: https://github.com/mspnp/performance-optimization/tree/master/BusyFrontEnd
[load-leveling]: /azure/architecture/patterns/queue-based-load-leveling
[sync-io]: ../synchronous-io/index.md
[web-queue-worker]: /azure/architecture/guide/architecture-styles/web-queue-worker

[WebJobs]: http://www.hanselman.com/blog/IntroducingWindowsAzureWebJobs.aspx
[ComputePartitioning]: https://msdn.microsoft.com/library/dn589773.aspx
[ServiceBusQueues]: https://msdn.microsoft.com/library/azure/hh367516.aspx
[AppDynamics-Transactions-Front-End-Requests]: ./_images/AppDynamicsPerformanceStats.jpg
[AppDynamics-Metrics-Front-End-Requests]: ./_images/AppDynamicsFrontEndMetrics.jpg
[Initial-Load-Test-Results-Front-End]: ./_images/InitialLoadTestResultsFrontEnd.jpg
[AppDynamics-Transactions-Background-Requests]: ./_images/AppDynamicsBackgroundPerformanceStats.jpg
[AppDynamics-Metrics-Background-Requests]: ./_images/AppDynamicsBackgroundMetrics.jpg
[Load-Test-Results-Background]: ./_images/LoadTestResultsBackground.jpg


