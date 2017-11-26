---
title: "同步 I/O 对立模式"
description: "在完成 I/O 时阻塞调用线程可能会降低性能并影响纵向可伸缩性。"
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: d5b3635565c6b71ef7716f54ee8cccc76093c3a3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="synchronous-io-antipattern"></a><span data-ttu-id="7f98e-103">同步 I/O 对立模式</span><span class="sxs-lookup"><span data-stu-id="7f98e-103">Synchronous I/O antipattern</span></span>

<span data-ttu-id="7f98e-104">在完成 I/O 时阻塞调用线程可能会降低性能并影响纵向可伸缩性。</span><span class="sxs-lookup"><span data-stu-id="7f98e-104">Blocking the calling thread while I/O completes can reduce performance and affect vertical scalability.</span></span>

## <a name="problem-description"></a><span data-ttu-id="7f98e-105">问题描述</span><span class="sxs-lookup"><span data-stu-id="7f98e-105">Problem description</span></span>

<span data-ttu-id="7f98e-106">完成 I/O 时，同步 I/O 操作会阻塞调用线程。</span><span class="sxs-lookup"><span data-stu-id="7f98e-106">A synchronous I/O operation blocks the calling thread while the I/O completes.</span></span> <span data-ttu-id="7f98e-107">在此时间间隔内，调用线程会进入等待状态，且无法执行有用的工作，因而浪费了处理资源。</span><span class="sxs-lookup"><span data-stu-id="7f98e-107">The calling thread enters a wait state and is unable to perform useful work during this interval, wasting processing resources.</span></span>

<span data-ttu-id="7f98e-108">常见 I/O 示例包括：</span><span class="sxs-lookup"><span data-stu-id="7f98e-108">Common examples of I/O include:</span></span>

- <span data-ttu-id="7f98e-109">在数据库或任何类型的持久性存储中检索或保存数据。</span><span class="sxs-lookup"><span data-stu-id="7f98e-109">Retrieving or persisting data to a database or any type of persistent storage.</span></span>
- <span data-ttu-id="7f98e-110">向 Web 服务发送请求。</span><span class="sxs-lookup"><span data-stu-id="7f98e-110">Sending a request to a web service.</span></span>
- <span data-ttu-id="7f98e-111">发布消息，或者从队列中检索消息。</span><span class="sxs-lookup"><span data-stu-id="7f98e-111">Posting a message or retrieving a message from a queue.</span></span>
- <span data-ttu-id="7f98e-112">写入或读取本地文件。</span><span class="sxs-lookup"><span data-stu-id="7f98e-112">Writing to or reading from a local file.</span></span>

<span data-ttu-id="7f98e-113">出现此对立模式的原因通常是：</span><span class="sxs-lookup"><span data-stu-id="7f98e-113">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="7f98e-114">此模式似乎是执行操作的最直观方式。</span><span class="sxs-lookup"><span data-stu-id="7f98e-114">It appears to be the most intuitive way to perform an operation.</span></span> 
- <span data-ttu-id="7f98e-115">应用程序需要请求返回的响应。</span><span class="sxs-lookup"><span data-stu-id="7f98e-115">The application requires a response from a request.</span></span>
- <span data-ttu-id="7f98e-116">应用程序使用的库只对 I/O 提供同步方法。</span><span class="sxs-lookup"><span data-stu-id="7f98e-116">The application uses a library that only provides synchronous methods for I/O.</span></span> 
- <span data-ttu-id="7f98e-117">外部库在内部执行同步 I/O 操作。</span><span class="sxs-lookup"><span data-stu-id="7f98e-117">An external library performs synchronous I/O operations internally.</span></span> <span data-ttu-id="7f98e-118">单个同步 I/O 调用可以阻塞整个调用链。</span><span class="sxs-lookup"><span data-stu-id="7f98e-118">A single synchronous I/O call can block an entire call chain.</span></span>

<span data-ttu-id="7f98e-119">以下代码将某个文件上传到 Azure Blob 存储。</span><span class="sxs-lookup"><span data-stu-id="7f98e-119">The following code uploads a file to Azure blob storage.</span></span> <span data-ttu-id="7f98e-120">代码块在以下两个位置等待同步 I/O：`CreateIfNotExists` 方法和 `UploadFromStream` 方法。</span><span class="sxs-lookup"><span data-stu-id="7f98e-120">There are two places where the code blocks waiting for synchronous I/O, the `CreateIfNotExists` method and the `UploadFromStream` method.</span></span>

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

container.CreateIfNotExists();
var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    blockBlob.UploadFromStream(fileStream);
}
```

<span data-ttu-id="7f98e-121">下面是等待外部服务返回响应的示例。</span><span class="sxs-lookup"><span data-stu-id="7f98e-121">Here's an example of waiting for a response from an external service.</span></span> <span data-ttu-id="7f98e-122">`GetUserProfile` 方法调用返回 `UserProfile` 的远程服务。</span><span class="sxs-lookup"><span data-stu-id="7f98e-122">The `GetUserProfile` method calls a remote service that returns a `UserProfile`.</span></span>

```csharp
public interface IUserProfileService
{
    UserProfile GetUserProfile();
}

public class SyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public SyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is a synchronous method that calls the synchronous GetUserProfile method.
    public UserProfile GetUserProfile()
    {
        return _userProfileService.GetUserProfile();
    }
}
```

<span data-ttu-id="7f98e-123">可在[此处][sample-app]找到这两个示例的完整代码。</span><span class="sxs-lookup"><span data-stu-id="7f98e-123">You can find the complete code for both of these examples [here][sample-app].</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="7f98e-124">如何解决问题</span><span class="sxs-lookup"><span data-stu-id="7f98e-124">How to fix the problem</span></span>

<span data-ttu-id="7f98e-125">将同步 I/O 操作替换为异步操作。</span><span class="sxs-lookup"><span data-stu-id="7f98e-125">Replace synchronous I/O operations with asynchronous operations.</span></span> <span data-ttu-id="7f98e-126">这会释放（而不是阻塞）当前线程用于继续执行有意义的工作，并帮助提高计算资源的利用率。</span><span class="sxs-lookup"><span data-stu-id="7f98e-126">This frees the current thread to continue performing meaningful work rather than blocking, and helps improve the utilization of compute resources.</span></span> <span data-ttu-id="7f98e-127">以异步方式执行 I/O 特别有利于应对客户端应用程序发出的请求出现意外浪涌。</span><span class="sxs-lookup"><span data-stu-id="7f98e-127">Performing I/O asynchronously is particularly efficient for handling an unexpected surge in requests from client applications.</span></span> 

<span data-ttu-id="7f98e-128">许多库提供方法的同步和异步版本。</span><span class="sxs-lookup"><span data-stu-id="7f98e-128">Many libraries provide both synchronous and asynchronous versions of methods.</span></span> <span data-ttu-id="7f98e-129">请尽量使用异步版本。</span><span class="sxs-lookup"><span data-stu-id="7f98e-129">Whenever possible, use the asynchronous versions.</span></span> <span data-ttu-id="7f98e-130">下面是上述将文件上传到 Azure Blob 存储的示例的异步版本。</span><span class="sxs-lookup"><span data-stu-id="7f98e-130">Here is the asynchronous version of the previous example that uploads a file to Azure blob storage.</span></span>

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

await container.CreateIfNotExistsAsync();

var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    await blockBlob.UploadFromStreamAsync(fileStream);
}
```

<span data-ttu-id="7f98e-131">执行异步操作时，`await` 运算符将控制权返回给调用环境。</span><span class="sxs-lookup"><span data-stu-id="7f98e-131">The `await` operator returns control to the calling environment while the asynchronous operation is performed.</span></span> <span data-ttu-id="7f98e-132">此语句后面的代码充当完成异步操作时运行的延续标记。</span><span class="sxs-lookup"><span data-stu-id="7f98e-132">The code after this statement acts as a continuation that runs when the asynchronous operation has completed.</span></span>

<span data-ttu-id="7f98e-133">合理设计的服务也应提供异步操作。</span><span class="sxs-lookup"><span data-stu-id="7f98e-133">A well designed service should also provide asynchronous operations.</span></span> <span data-ttu-id="7f98e-134">下面是返回用户配置文件的 Web 服务的异步版本。</span><span class="sxs-lookup"><span data-stu-id="7f98e-134">Here is an asynchronous version of the web service that returns user profiles.</span></span> <span data-ttu-id="7f98e-135">`GetUserProfileAsync` 方法依赖于使用用户配置文件服务的异步版本。</span><span class="sxs-lookup"><span data-stu-id="7f98e-135">The `GetUserProfileAsync` method depends on having an asynchronous version of the User Profile service.</span></span>

```csharp
public interface IUserProfileService
{
    Task<UserProfile> GetUserProfileAsync();
}

public class AsyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public AsyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is an synchronous method that calls the Task based GetUserProfileAsync method.
    public Task<UserProfile> GetUserProfileAsync()
    {
        return _userProfileService.GetUserProfileAsync();
    }
}
```

<span data-ttu-id="7f98e-136">对于不提供异步操作版本的库，也许可以围绕选定的同步方法创建异步包装器。</span><span class="sxs-lookup"><span data-stu-id="7f98e-136">For libraries that don't provide asynchronous versions of operations, it may be possible to create asynchronous wrappers around selected synchronous methods.</span></span> <span data-ttu-id="7f98e-137">遵循此方法时请小心。</span><span class="sxs-lookup"><span data-stu-id="7f98e-137">Follow this approach with caution.</span></span> <span data-ttu-id="7f98e-138">尽管它能提高调用异步包装器的线程的响应能力，但实际上会消耗更多资源。</span><span class="sxs-lookup"><span data-stu-id="7f98e-138">While it may improve responsiveness on the thread that invokes the asynchronous wrapper, it actually consumes more resources.</span></span> <span data-ttu-id="7f98e-139">这可能会创建一个额外的线程，另外，同步此线程执行的工作也会产生相应的开销。</span><span class="sxs-lookup"><span data-stu-id="7f98e-139">An extra thread may be created, and there is overhead associated with synchronizing the work done by this thread.</span></span> <span data-ttu-id="7f98e-140">以下博客文章讨论了一些相关的利弊：[Should I expose asynchronous wrappers for synchronous methods?][async-wrappers]（是否应公开同步方法的异步包装器？）</span><span class="sxs-lookup"><span data-stu-id="7f98e-140">Some tradeoffs are discussed in this blog post: [Should I expose asynchronous wrappers for synchronous methods?][async-wrappers]</span></span>

<span data-ttu-id="7f98e-141">下面是围绕同步方法创建的异步包装器示例。</span><span class="sxs-lookup"><span data-stu-id="7f98e-141">Here is an example of an asynchronous wrapper around a synchronous method.</span></span>

```csharp
// Asynchronous wrapper around synchronous library method
private async Task<int> LibraryIOOperationAsync()
{
    return await Task.Run(() => LibraryIOOperation());
}
```

<span data-ttu-id="7f98e-142">现在，调用代码可以在包装器中等待：</span><span class="sxs-lookup"><span data-stu-id="7f98e-142">Now the calling code can await on the wrapper:</span></span>

```csharp
// Invoke the asynchronous wrapper using a task
await LibraryIOOperationAsync();
```

## <a name="considerations"></a><span data-ttu-id="7f98e-143">注意事项</span><span class="sxs-lookup"><span data-stu-id="7f98e-143">Considerations</span></span>

- <span data-ttu-id="7f98e-144">生存期预期很短且不太可能导致资源争用的 I/O 操作在性能上可能与同步操作相当。</span><span class="sxs-lookup"><span data-stu-id="7f98e-144">I/O operations that are expected to be very short lived and are unlikely to cause contention might be more performant as synchronous operations.</span></span> <span data-ttu-id="7f98e-145">读取 SSD 驱动器中的小文件就是这样一个例子。</span><span class="sxs-lookup"><span data-stu-id="7f98e-145">An example might be reading small files on an SSD drive.</span></span> <span data-ttu-id="7f98e-146">尽管异步 I/O 可带来一定的好处，但是，将某个任务调度到另一个线程，并在完成该任务时与该线程同步所产生的开销可能会使这种好处得不偿失。</span><span class="sxs-lookup"><span data-stu-id="7f98e-146">The overhead of dispatching a task to another thread, and synchronizing with that thread when the task completes, might outweigh the benefits of asynchronous I/O.</span></span> <span data-ttu-id="7f98e-147">不过，这种情况相对很少出现，大多数 I/O 操作还是应该以异步方式执行。</span><span class="sxs-lookup"><span data-stu-id="7f98e-147">However, these cases are relatively rare, and most I/O operations should be done asynchronously.</span></span>

- <span data-ttu-id="7f98e-148">提高 I/O 性能可能导致系统的其他部分成为瓶颈。</span><span class="sxs-lookup"><span data-stu-id="7f98e-148">Improving I/O performance may cause other parts of the system to become bottlenecks.</span></span> <span data-ttu-id="7f98e-149">例如，取消阻塞线程可能导致针对共享资源发出的并发请求数量增加，从而导致资源枯竭或限制。</span><span class="sxs-lookup"><span data-stu-id="7f98e-149">For example, unblocking threads might result in a higher volume of concurrent requests to shared resources, leading in turn to resource starvation or throttling.</span></span> <span data-ttu-id="7f98e-150">如果出现这种问题，可能需要增加 Web 服务器的数量或者将数据存储分区，以减少资源争用。</span><span class="sxs-lookup"><span data-stu-id="7f98e-150">If that becomes a problem, you might need to scale out the number of web servers or partition data stores to reduce contention.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="7f98e-151">如何检测问题</span><span class="sxs-lookup"><span data-stu-id="7f98e-151">How to detect the problem</span></span>

<span data-ttu-id="7f98e-152">在用户那里，应用程序可能呈死机状态，或者定期挂起。</span><span class="sxs-lookup"><span data-stu-id="7f98e-152">For users, the application may seem unresponsive or appear to hang periodically.</span></span> <span data-ttu-id="7f98e-153">应用程序可能会失败并出现超时异常。</span><span class="sxs-lookup"><span data-stu-id="7f98e-153">The application might fail with timeout exceptions.</span></span> <span data-ttu-id="7f98e-154">这些失败还可能返回 HTTP 500（内部服务器）错误。</span><span class="sxs-lookup"><span data-stu-id="7f98e-154">These failures could also return HTTP 500 (Internal Server) errors.</span></span> <span data-ttu-id="7f98e-155">在服务器上，传入的客户端请求可能会阻塞到某个线程可用为止，导致请求队列变得过长，并显示 HTTP 503（服务不可用）错误。</span><span class="sxs-lookup"><span data-stu-id="7f98e-155">On the server, incoming client requests might be blocked until a thread becomes available, resulting in excessive request queue lengths, manifested as HTTP 503 (Service Unavailable) errors.</span></span>

<span data-ttu-id="7f98e-156">可执行以下步骤来帮助识别问题：</span><span class="sxs-lookup"><span data-stu-id="7f98e-156">You can perform the following steps to help identify the problem:</span></span>

1. <span data-ttu-id="7f98e-157">监视生产系统，并确定已阻塞的工作线程是否约束了吞吐量。</span><span class="sxs-lookup"><span data-stu-id="7f98e-157">Monitor the production system and determine whether blocked worker threads are constraining throughput.</span></span>

2. <span data-ttu-id="7f98e-158">如果请求由于缺少线程而阻塞，请检查应用程序，确定哪些操作可能在以异步方式执行 I/O。</span><span class="sxs-lookup"><span data-stu-id="7f98e-158">If requests are being blocked due to lack of threads, review the application to determine which operations may be performing I/O synchronously.</span></span>

3. <span data-ttu-id="7f98e-159">针对执行同步 I/O 的每个操作运行受控的负载测试，确定这些操作是否影响了系统性能。</span><span class="sxs-lookup"><span data-stu-id="7f98e-159">Perform controlled load testing of each operation that is performing synchronous I/O, to find out whether those operations are affecting system performance.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="7f98e-160">示例诊断</span><span class="sxs-lookup"><span data-stu-id="7f98e-160">Example diagnosis</span></span>

<span data-ttu-id="7f98e-161">以下部分将这些步骤应用到前面所述的示例应用程序。</span><span class="sxs-lookup"><span data-stu-id="7f98e-161">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="monitor-web-server-performance"></a><span data-ttu-id="7f98e-162">监视 Web 服务器性能</span><span class="sxs-lookup"><span data-stu-id="7f98e-162">Monitor web server performance</span></span>

<span data-ttu-id="7f98e-163">对于 Azure Web 应用程序和 Web 角色，有必要监视 IIS Web 服务器的性能。</span><span class="sxs-lookup"><span data-stu-id="7f98e-163">For Azure web applications and web roles, it's worth monitoring the performance of the IIS web server.</span></span> <span data-ttu-id="7f98e-164">具体而言，应注意请求队列的长度，以判定在高峰期间请求是否处于阻塞状态，在等待可用线程。</span><span class="sxs-lookup"><span data-stu-id="7f98e-164">In particular, pay attention to the request queue length to establish whether requests are being blocked waiting for available threads during periods of high activity.</span></span> <span data-ttu-id="7f98e-165">可通过启用 Azure 诊断来收集此信息。</span><span class="sxs-lookup"><span data-stu-id="7f98e-165">You can gather this information by enabling Azure diagnostics.</span></span> <span data-ttu-id="7f98e-166">有关详细信息，请参阅：</span><span class="sxs-lookup"><span data-stu-id="7f98e-166">For more information, see:</span></span>

- <span data-ttu-id="7f98e-167">[监视 Azure 应用服务中的应用][web-sites-monitor]</span><span class="sxs-lookup"><span data-stu-id="7f98e-167">[Monitor Apps in Azure App Service][web-sites-monitor]</span></span>
- <span data-ttu-id="7f98e-168">[在 Azure 应用程序中创建和使用性能计数器][performance-counters]</span><span class="sxs-lookup"><span data-stu-id="7f98e-168">[Create and use performance counters in an Azure application][performance-counters]</span></span>

<span data-ttu-id="7f98e-169">检测应用程序，了解在接受请求后如何对其进行处理。</span><span class="sxs-lookup"><span data-stu-id="7f98e-169">Instrument the application to see how requests are handled once they have been accepted.</span></span> <span data-ttu-id="7f98e-170">跟踪某个请求的流有助于确定该请求是否在执行缓慢运行的调用并阻塞了当前线程。</span><span class="sxs-lookup"><span data-stu-id="7f98e-170">Tracing the flow of a request can help to identify whether it is performing slow-running calls and blocking the current thread.</span></span> <span data-ttu-id="7f98e-171">线程分析也能突出显示正在阻塞的请求。</span><span class="sxs-lookup"><span data-stu-id="7f98e-171">Thread profiling can also highlight requests that are being blocked.</span></span>

### <a name="load-test-the-application"></a><span data-ttu-id="7f98e-172">对应用程序进行负载测试</span><span class="sxs-lookup"><span data-stu-id="7f98e-172">Load test the application</span></span>

<span data-ttu-id="7f98e-173">下图显示了在承受不同负载（最大负载为 4000 个并发用户）的情况下，前面所示同步 `GetUserProfile` 方法的性能。</span><span class="sxs-lookup"><span data-stu-id="7f98e-173">The following graph shows the performance of the synchronous `GetUserProfile` method shown earlier, under varying loads of up to 4000 concurrent users.</span></span> <span data-ttu-id="7f98e-174">该应用程序是在 Azure 云服务 Web 角色中运行的 ASP.NET 应用程序。</span><span class="sxs-lookup"><span data-stu-id="7f98e-174">The application is an ASP.NET application running in an Azure Cloud Service web role.</span></span>

![执行同步 I/O 操作的示例应用程序的性能图表][sync-performance]

<span data-ttu-id="7f98e-176">该同步操作硬编码为休眠 2 秒以模拟同步 I/O，因此最小响应时间略微超过 2 秒。</span><span class="sxs-lookup"><span data-stu-id="7f98e-176">The synchronous operation is hard-coded to sleep for 2 seconds, to simulate synchronous I/O, so the minimum response time is slightly over 2 seconds.</span></span> <span data-ttu-id="7f98e-177">当负载达到大约 2500 个并发用户时，平均响应时间达到平稳状态，不过，每秒请求数量持续增加。</span><span class="sxs-lookup"><span data-stu-id="7f98e-177">When the load reaches approximately 2500 concurrent users, the average response time reaches a plateau, although the volume of requests per second continues to increase.</span></span> <span data-ttu-id="7f98e-178">请注意，这些两个度量值的刻度为对数。</span><span class="sxs-lookup"><span data-stu-id="7f98e-178">Note that the scale for these two measures is logarithmic.</span></span> <span data-ttu-id="7f98e-179">此时间点与测试结束时的每秒请求数相差两倍。</span><span class="sxs-lookup"><span data-stu-id="7f98e-179">The number of requests per second doubles between this point and the end of the test.</span></span>

<span data-ttu-id="7f98e-180">在隔离状态下，此测试不一定能够清楚地反映同步 I/O 是否造成了问题。</span><span class="sxs-lookup"><span data-stu-id="7f98e-180">In isolation, it's not necessarily clear from this test whether the synchronous I/O is a problem.</span></span> <span data-ttu-id="7f98e-181">承受更重的负载时，应用程序可能会进入一个临界点，此时，Web 服务器可能不再及时处理请求，从而导致客户端应用程序收到超时异常。</span><span class="sxs-lookup"><span data-stu-id="7f98e-181">Under heavier load, the application may reach a tipping point where the web server can no longer process requests in a timely manner, causing client applications to receive time-out exceptions.</span></span>

<span data-ttu-id="7f98e-182">IIS Web 服务器会将传入的请求排队，并将其转发到 ASP.NET 线程池中运行的线程。</span><span class="sxs-lookup"><span data-stu-id="7f98e-182">Incoming requests are queued by the IIS web server and handed to a thread running in the ASP.NET thread pool.</span></span> <span data-ttu-id="7f98e-183">由于每个操作同步执行 I/O，因此，线程会阻塞到该操作完成为止。</span><span class="sxs-lookup"><span data-stu-id="7f98e-183">Because each operation performs I/O synchronously, the thread is blocked until the operation completes.</span></span> <span data-ttu-id="7f98e-184">随着工作负荷的增大，最终会分配并阻塞线程池中的所有 ASP.NET 线程。</span><span class="sxs-lookup"><span data-stu-id="7f98e-184">As the workload increases, eventually all of the ASP.NET threads in the thread pool are allocated and blocked.</span></span> <span data-ttu-id="7f98e-185">此时，传入的其他任何请求必须在队列中等待有线程可用。</span><span class="sxs-lookup"><span data-stu-id="7f98e-185">At that point, any further incoming requests must wait in the queue for an available thread.</span></span> <span data-ttu-id="7f98e-186">随着队列长度不断增大，请求开始超时。</span><span class="sxs-lookup"><span data-stu-id="7f98e-186">As the queue length grows, requests start to time out.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="7f98e-187">实施解决方案并验证结果</span><span class="sxs-lookup"><span data-stu-id="7f98e-187">Implement the solution and verify the result</span></span>

<span data-ttu-id="7f98e-188">下图显示了针对异步代码版本执行的负载测试结果。</span><span class="sxs-lookup"><span data-stu-id="7f98e-188">The next graph shows the results from load testing the asynchronous version of the code.</span></span>

![执行异步 I/O 操作的示例应用程序的性能图表][async-performance]

<span data-ttu-id="7f98e-190">吞吐量要高得多。</span><span class="sxs-lookup"><span data-stu-id="7f98e-190">Throughput is far higher.</span></span> <span data-ttu-id="7f98e-191">在与前次测试相同的持续时间内，系统已成功处理将近十倍的吞吐量增长（以每秒请求数为单位）。</span><span class="sxs-lookup"><span data-stu-id="7f98e-191">Over the same duration as the previous test, the system successfully handles a nearly tenfold increase in throughput, as measured in requests per second.</span></span> <span data-ttu-id="7f98e-192">此外，平均响应时间相对恒定，并且比前次测试大约小 25 倍。</span><span class="sxs-lookup"><span data-stu-id="7f98e-192">Moreover, the average response time is relatively constant and remains approximately 25 times smaller than the previous test.</span></span>


[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/SynchronousIO


[async-wrappers]: http://blogs.msdn.com/b/pfxteam/archive/2012/03/24/10287244.aspx
[performance-counters]: /azure/cloud-services/cloud-services-dotnet-diagnostics-performance-counters
[web-sites-monitor]: /azure/app-service-web/web-sites-monitor

[sync-performance]: _images/SyncPerformance.jpg
[async-performance]: _images/AsyncPerformance.jpg



