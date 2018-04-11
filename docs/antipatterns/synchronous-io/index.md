---
title: 同步 I/O 对立模式
description: 在完成 I/O 时阻塞调用线程可能会降低性能并影响纵向可伸缩性。
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: d5b3635565c6b71ef7716f54ee8cccc76093c3a3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="synchronous-io-antipattern"></a>同步 I/O 对立模式

在完成 I/O 时阻塞调用线程可能会降低性能并影响纵向可伸缩性。

## <a name="problem-description"></a>问题描述

完成 I/O 时，同步 I/O 操作会阻塞调用线程。 在此时间间隔内，调用线程会进入等待状态，且无法执行有用的工作，因而浪费了处理资源。

常见 I/O 示例包括：

- 在数据库或任何类型的持久性存储中检索或保存数据。
- 向 Web 服务发送请求。
- 发布消息，或者从队列中检索消息。
- 写入或读取本地文件。

出现此对立模式的原因通常是：

- 此模式似乎是执行操作的最直观方式。 
- 应用程序需要请求返回的响应。
- 应用程序使用的库只对 I/O 提供同步方法。 
- 外部库在内部执行同步 I/O 操作。 单个同步 I/O 调用可以阻塞整个调用链。

以下代码将某个文件上传到 Azure Blob 存储。 代码块在以下两个位置等待同步 I/O：`CreateIfNotExists` 方法和 `UploadFromStream` 方法。

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

下面是等待外部服务返回响应的示例。 `GetUserProfile` 方法调用返回 `UserProfile` 的远程服务。

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

可在[此处][sample-app]找到这两个示例的完整代码。

## <a name="how-to-fix-the-problem"></a>如何解决问题

将同步 I/O 操作替换为异步操作。 这会释放（而不是阻塞）当前线程用于继续执行有意义的工作，并帮助提高计算资源的利用率。 以异步方式执行 I/O 特别有利于应对客户端应用程序发出的请求出现意外浪涌。 

许多库提供方法的同步和异步版本。 请尽量使用异步版本。 下面是上述将文件上传到 Azure Blob 存储的示例的异步版本。

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

执行异步操作时，`await` 运算符将控制权返回给调用环境。 此语句后面的代码充当完成异步操作时运行的延续标记。

合理设计的服务也应提供异步操作。 下面是返回用户配置文件的 Web 服务的异步版本。 `GetUserProfileAsync` 方法依赖于使用用户配置文件服务的异步版本。

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

对于不提供异步操作版本的库，也许可以围绕选定的同步方法创建异步包装器。 遵循此方法时请小心。 尽管它能提高调用异步包装器的线程的响应能力，但实际上会消耗更多资源。 这可能会创建一个额外的线程，另外，同步此线程执行的工作也会产生相应的开销。 以下博客文章讨论了一些相关的利弊：[Should I expose asynchronous wrappers for synchronous methods?][async-wrappers]（是否应公开同步方法的异步包装器？）

下面是围绕同步方法创建的异步包装器示例。

```csharp
// Asynchronous wrapper around synchronous library method
private async Task<int> LibraryIOOperationAsync()
{
    return await Task.Run(() => LibraryIOOperation());
}
```

现在，调用代码可以在包装器中等待：

```csharp
// Invoke the asynchronous wrapper using a task
await LibraryIOOperationAsync();
```

## <a name="considerations"></a>注意事项

- 生存期预期很短且不太可能导致资源争用的 I/O 操作在性能上可能与同步操作相当。 读取 SSD 驱动器中的小文件就是这样一个例子。 尽管异步 I/O 可带来一定的好处，但是，将某个任务调度到另一个线程，并在完成该任务时与该线程同步所产生的开销可能会使这种好处得不偿失。 不过，这种情况相对很少出现，大多数 I/O 操作还是应该以异步方式执行。

- 提高 I/O 性能可能导致系统的其他部分成为瓶颈。 例如，取消阻塞线程可能导致针对共享资源发出的并发请求数量增加，从而导致资源枯竭或限制。 如果出现这种问题，可能需要增加 Web 服务器的数量或者将数据存储分区，以减少资源争用。

## <a name="how-to-detect-the-problem"></a>如何检测问题

在用户那里，应用程序可能呈死机状态，或者定期挂起。 应用程序可能会失败并出现超时异常。 这些失败还可能返回 HTTP 500（内部服务器）错误。 在服务器上，传入的客户端请求可能会阻塞到某个线程可用为止，导致请求队列变得过长，并显示 HTTP 503（服务不可用）错误。

可执行以下步骤来帮助识别问题：

1. 监视生产系统，并确定已阻塞的工作线程是否约束了吞吐量。

2. 如果请求由于缺少线程而阻塞，请检查应用程序，确定哪些操作可能在以异步方式执行 I/O。

3. 针对执行同步 I/O 的每个操作运行受控的负载测试，确定这些操作是否影响了系统性能。

## <a name="example-diagnosis"></a>示例诊断

以下部分将这些步骤应用到前面所述的示例应用程序。

### <a name="monitor-web-server-performance"></a>监视 Web 服务器性能

对于 Azure Web 应用程序和 Web 角色，有必要监视 IIS Web 服务器的性能。 具体而言，应注意请求队列的长度，以判定在高峰期间请求是否处于阻塞状态，在等待可用线程。 可通过启用 Azure 诊断来收集此信息。 有关详细信息，请参阅：

- [监视 Azure 应用服务中的应用][web-sites-monitor]
- [在 Azure 应用程序中创建和使用性能计数器][performance-counters]

检测应用程序，了解在接受请求后如何对其进行处理。 跟踪某个请求的流有助于确定该请求是否在执行缓慢运行的调用并阻塞了当前线程。 线程分析也能突出显示正在阻塞的请求。

### <a name="load-test-the-application"></a>对应用程序进行负载测试

下图显示了在承受不同负载（最大负载为 4000 个并发用户）的情况下，前面所示同步 `GetUserProfile` 方法的性能。 该应用程序是在 Azure 云服务 Web 角色中运行的 ASP.NET 应用程序。

![执行同步 I/O 操作的示例应用程序的性能图表][sync-performance]

该同步操作硬编码为休眠 2 秒以模拟同步 I/O，因此最小响应时间略微超过 2 秒。 当负载达到大约 2500 个并发用户时，平均响应时间达到平稳状态，不过，每秒请求数量持续增加。 请注意，这些两个度量值的刻度为对数。 此时间点与测试结束时的每秒请求数相差两倍。

在隔离状态下，此测试不一定能够清楚地反映同步 I/O 是否造成了问题。 承受更重的负载时，应用程序可能会进入一个临界点，此时，Web 服务器可能不再及时处理请求，从而导致客户端应用程序收到超时异常。

IIS Web 服务器会将传入的请求排队，并将其转发到 ASP.NET 线程池中运行的线程。 由于每个操作同步执行 I/O，因此，线程会阻塞到该操作完成为止。 随着工作负荷的增大，最终会分配并阻塞线程池中的所有 ASP.NET 线程。 此时，传入的其他任何请求必须在队列中等待有线程可用。 随着队列长度不断增大，请求开始超时。

### <a name="implement-the-solution-and-verify-the-result"></a>实施解决方案并验证结果

下图显示了针对异步代码版本执行的负载测试结果。

![执行异步 I/O 操作的示例应用程序的性能图表][async-performance]

吞吐量要高得多。 在与前次测试相同的持续时间内，系统已成功处理将近十倍的吞吐量增长（以每秒请求数为单位）。 此外，平均响应时间相对恒定，并且比前次测试大约小 25 倍。


[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/SynchronousIO


[async-wrappers]: http://blogs.msdn.com/b/pfxteam/archive/2012/03/24/10287244.aspx
[performance-counters]: /azure/cloud-services/cloud-services-dotnet-diagnostics-performance-counters
[web-sites-monitor]: /azure/app-service-web/web-sites-monitor

[sync-performance]: _images/SyncPerformance.jpg
[async-performance]: _images/AsyncPerformance.jpg



