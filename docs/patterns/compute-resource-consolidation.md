---
title: 计算资源合并
description: 将多个任务或操作合并到单个计算单元
keywords: 设计模式
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
ms.openlocfilehash: 6e05a30245fbf5183a4e50a54650505f5a5f2aa8
ms.sourcegitcommit: 85334ab0ccb072dac80de78aa82bcfa0f0044d3f
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/11/2018
ms.locfileid: "35252918"
---
# <a name="compute-resource-consolidation-pattern"></a>计算资源合并模式

[!INCLUDE [header](../_includes/header.md)]

将多个任务或操作合并到单个计算单元。 这可以提高计算资源利用率，并减少与在云托管应用程序中执行计算处理关联的成本和管理开销。

## <a name="context-and-problem"></a>上下文和问题

云应用程序通常实现各种操作。 在某些解决方案中，合理的做法是最初遵循问题分离的设计原则，将这些操作划分成分别进行托管和部署的单独计算单元（例如，作为单独的应用服务 Web 应用、单独的虚拟机或单独的云服务角色）。 但是，虽然此策略可以帮助简化解决方案的逻辑设计，不过将大量计算单元作为相同应用程序的一部分进行部署可能会增加运行时托管成本并使系统管理更复杂。

作为示例，下图显示使用多个计算单元实现的云托管解决方案的简化结构。 每个计算单元都在其自己的虚拟环境中运行。 每个函数都作为在其自己的计算单元中运行的单独任务（标记为任务 A 到任务 E）来实现。

![使用一组专用计算单元在云环境中运行任务](./_images/compute-resource-consolidation-diagram.png)


每个计算单元都会消耗应计费资源，即使它处于空闲状态或不常使用。 因此，这并不总是最经济高效的解决方案。

在 Azure 中，此问题适用于云服务、应用服务和虚拟机中的角色。 这些项在其自己的虚拟环境中运行。 运行设计为执行一组定义完善的操作，但需要作为单个解决方案的一部分进行通信和协作的单独角色、网站或虚拟机的集合可能对资源的使用较为低效。

## <a name="solution"></a>解决方案

若要帮助降低成本、提高利用率、加快通信速度并减少管理，可以将多个任务或操作合并到单个计算单元。

任务可以按照基于环境提供的功能以及与这些功能关联的成本的条件进行分组。 一种常见方法是查找在其可伸缩性、生存期和处理要求方面具有类似特征的任务。 将它们组合在一起可使它们作为一个单元进行缩放。 借助许多云环境提供的弹性，可以根据工作负载来启动和停止计算单元的附加实例。 例如，Azure 提供自动缩放功能，可以应用于云服务、应用服务和虚拟机中的角色。 有关详细信息，请参阅 [Autoscaling Guidance](https://msdn.microsoft.com/library/dn589774.aspx)（自动缩放指南）。

作为用于演示如何使用可伸缩性确定不应组合在一起的操作的计数器示例，请考虑以下两个任务：

- 任务 1 轮询发送给队列的对时间不敏感的少见消息。
- 任务 2 处理大量网络流量突发。

第二个任务需要可能会涉及到启动和停止大量计算单元实例的弹性。 将相同缩放应用于第一个任务只会导致更多任务在相同队列中侦听少见消息，是一种资源浪费。

在许多云环境中，可以在 CPU 核心数、内存、磁盘空间等方面指定可供计算单元使用的资源。 一般情况下，指定的资源越多，成本便越高。 为了节省资金，请务必最大程度提高昂贵计算单元执行的工作量，不要让它长时间处于非活动状态。

如果有在短暂突发中需要大量 CPU 能力的任务，请考虑将这些任务合并到可提供所需能力的单个计算单元。 但是，请务必平衡此需求以使昂贵资源在面对可能发生的争用（如果它们处于超负荷状态）时保持繁忙状态。 例如，长时间运行的计算密集型任务不应共享相同的计算单元。

## <a name="issues-and-considerations"></a>问题和注意事项

在实现此模式时，请考虑以下几点：

**可伸缩性和弹性**。 许多云解决方案通过启动和停止计算单元实例，在计算单元级别实现可伸缩性和弹性。 应避免将具有冲突可伸缩性要求的任务分组到相同计算单元中。

**生存期**。 云基础结构会定期回收托管计算单元的虚拟环境。 当一个计算单元中存在许多长时间运行的任务时，可能需要配置该单元以防止在这些任务完成之前回收它。 或者，使用检查点方法设计任务，该方法使任务可完全停止，然后在计算单元重新启动时在中断位置处继续执行。

**发布节奏**。 如果任务的实现或配置经常更改，则可能需要停止托管更新的代码的计算单元，重新配置和重新部署该单元，然后重新启动它。 此过程还需要相同计算单元中的所有其他任务停止、重新部署并重新启动。

**安全性**。 相同计算单元中的任务可能会共享相同安全性上下文，并能够访问相同资源。 任务之间必须存在高度信任，并且确信一个任务不会对其他任务造成损坏或产生负面影响。 此外，增加在计算单元中运行的任务数会增大单元的攻击面。 每个任务的安全性只与具有最多漏洞的任务相同。

**容错**。 如果计算单元中的一个任务失败或行为异常，则它可能会影响在相同单元中运行的其他任务。 例如，如果一个任务未能正确启动，则它可能会导致计算单元的整个启动逻辑失败，并阻止相同单元中的其他任务运行。

**争用**。 应避免在相同计算单元中的任务之间出现竞争资源的争用。 理想情况下，共享相同计算单元的任务应表现出不同的资源利用率特征。 例如，两个计算密集型任务不应位于相同计算单元中，两个占用大量内存的任务也是如此。 但是，混合使用计算密集型任务与需要大量内存的任务是可行的组合。

> [!NOTE]
>  可考虑仅对已在一段时间内处于生产环境的系统合并计算资源，以便操作员和开发人员可以监视系统并创建标识每个任务如何利用不同资源的热度地图。 此地图可以用于确定非常适合用于共享计算资源的任务。

**复杂性**。 将多个任务合并到单个计算单元会向单元中的代码增加复杂性，从而更加难以进行测试、调试和维护。

**稳定的逻辑体系结构**。 设计和实现每个任务中的代码，以便即使运行任务的物理环境发生更改也无需更改代码。

**其他策略**。 合并计算资源只是可帮助降低与并发运行多个任务关联的成本的一种方式。 它需要进行仔细规划和监视以确保保持为有效方法。 其他策略可能更为合适，具体取决于工作的性质以及运行这些任务的用户所处的位置。 例如，工作负载的功能分解（如[计算分区指南](https://msdn.microsoft.com/library/dn589773.aspx)中所述）可能是更好的选择。

## <a name="when-to-use-this-pattern"></a>何时使用此模式

对于在其自己的计算单元中运行时不怎么经济高效的任务，可使用此模式。 如果任务长时间处于空闲状态，则在专用单元中运行此任务可能成本高昂。

此模式可能不适合执行关键容错操作的任务，或是处理高度敏感或私有数据并需要其自己的安全性上下文的任务。 这些任务应在其自己的隔离环境、在单独的计算单元中运行。

## <a name="example"></a>示例

在 Azure 上构建云服务时，可以将多个任务执行的处理合并到单个角色。 通常，这是执行后台或异步处理任务的辅助角色。

> 在某些情况下，可以在 Web 角色中包含后台或异步处理任务。 此方法可帮助降低成本和简化部署，尽管它可能会影响 Web 角色提供的面向公众的接口的可伸缩性和响应能力。 [将多个 Azure 辅助角色合并到 Azure Web 角色](http://www.31a2ba2a-b718-11dc-8314-0800200c9a66.com/2012/02/combining-multiple-azure-worker-roles.html)一文包含在 Web 角色中实现后台或异步处理任务的详细说明。

角色负责启动和停止任务。 当 Azure 结构控制器加载角色时，它会对角色引发 `Start` 事件。 可以替代 `WebRole` 或 `WorkerRole` 类的 `OnStart` 方法以处理此事件，这可能是为了初始化此方法中的任务所依赖的数据和其他资源。

当 `OnStart` 方法完成时，角色可以开始响应请求。 可以在模式和做法指南[将应用程序移动到](https://msdn.microsoft.com/library/ff728592.aspx)中的[应用程序启动进程](https://msdn.microsoft.com/library/ff803371.aspx#sec16)部分中找到有关在角色中使用 `OnStart` 和 `Run` 方法的详细信息和指导。

> 使 `OnStart` 方法中的代码尽可能简洁。 Azure 不会对此方法完成所花费的时间施加任何限制，但是角色在此方法完成之前，无法开始响应发送给它的网络请求。

当 `OnStart` 方法完成时，角色会执行 `Run` 方法。 此时，结构控制器可以开始将请求发送给角色。

将实际创建任务的代码置于 `Run` 方法中。 请注意，`Run` 方法定义角色实例的生存期。 当此方法完成时，结构控制器会安排角色关闭。

当角色关闭或回收时，结构控制器会阻止负载均衡器接收任何其他传入请求并引发 `Stop` 事件。 可以通过替代角色的 `OnStop` 方法来捕获此事件，并执行角色终止之前所需的任何整理。

> 在 `OnStop` 方法中执行的任何操作都必须在五分钟（如果在本地计算机上使用 Azure 仿真器，则是 30 秒）内完成。 否则 Azure 结构控制器会假设角色已终止，并强制它停止。

任务由 `Run` 方法启动，该方法会等待任务完成。 任务实现云服务的业务逻辑，并可以响应通过 Azure 负载均衡器发布到角色的消息。 下图显示 Azure 云服务的角色中的任务和资源生命周期。

![Azure 云服务的角色中的任务和资源生命周期](./_images/compute-resource-consolidation-lifecycle.png)


ComputeResourceConsolidation.Worker 项目中的 WorkerRole.cs 文件演示如何在 Azure 云服务中实现此模式的示例。

> ComputeResourceConsolidation.Worker 项目是 ComputeResourceConsolidation 解决方案的一部分，可从 [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation) 进行下载。

`MyWorkerTask1` 和 `MyWorkerTask2` 方法演示如何在相同辅助角色中执行不同任务。 以下代码演示 `MyWorkerTask1`。 这是一个简单任务，它会睡眠 30 秒，然后输出跟踪消息。 它会重复此过程，直到取消任务。 `MyWorkerTask2` 中的代码类似。

```csharp
// A sample worker role task.
private static async Task MyWorkerTask1(CancellationToken ct)
{
  // Fixed interval to wake up and check for work and/or do work.
  var interval = TimeSpan.FromSeconds(30);

  try
  {
    while (!ct.IsCancellationRequested)
    {
      // Wake up and do some background processing if not canceled.
      // TASK PROCESSING CODE HERE
      Trace.TraceInformation("Doing Worker Task 1 Work");

      // Go back to sleep for a period of time unless asked to cancel.
      // Task.Delay will throw an OperationCanceledException when canceled.
      await Task.Delay(interval, ct);
    }
  }
  catch (OperationCanceledException)
  {
    // Expect this exception to be thrown in normal circumstances or check
    // the cancellation token. If the role instances are shutting down, a
    // cancellation request will be signaled.
    Trace.TraceInformation("Stopping service, cancellation requested");

    // Rethrow the exception.
    throw;
  }
}
```

> 该示例代码演示后台进程的常见实现。 在实际应用程序中，可以遵循此相同结构，只不过应在等待取消请求的循环主体中放置自己的处理逻辑。

辅助角色初始化了它使用的资源之后，`Run` 方法会并发启动两个任务，如下所示。

```csharp
/// <summary>
/// The cancellation token source use to cooperatively cancel running tasks
/// </summary>
private readonly CancellationTokenSource cts = new CancellationTokenSource();

/// <summary>
/// List of running tasks on the role instance
/// </summary>
private readonly List<Task> tasks = new List<Task>();

// RoleEntry Run() is called after OnStart().
// Returning from Run() will cause a role instance to recycle.
public override void Run()
{
  // Start worker tasks and add to the task list
  tasks.Add(MyWorkerTask1(cts.Token));
  tasks.Add(MyWorkerTask2(cts.Token));

  foreach (var worker in this.workerTasks)
  {
      this.tasks.Add(worker);
  }

  Trace.TraceInformation("Worker host tasks started");
  // The assumption is that all tasks should remain running and not return,
  // similar to role entry Run() behavior.
  try
  {
    Task.WaitAll(tasks.ToArray());
  }
  catch (AggregateException ex)
  {
    Trace.TraceError(ex.Message);

    // If any of the inner exceptions in the aggregate exception
    // are not cancellation exceptions then re-throw the exception.
    ex.Handle(innerEx => (innerEx is OperationCanceledException));
  }

  // If there wasn't a cancellation request, stop all tasks and return from Run()
  // An alternative to canceling and returning when a task exits would be to
  // restart the task.
  if (!cts.IsCancellationRequested)
  {
    Trace.TraceInformation("Task returned without cancellation request");
    Stop(TimeSpan.FromMinutes(5));
  }
}
...
```

在此示例中，`Run` 方法会等待任务完成。 如果取消任务，则 `Run` 方法会假设角色正在关闭，并在完成之前等待取消剩余任务（最多等待五分钟，然后终止）。 如果任务由于预期异常而失败，则 `Run` 方法会取消任务。

> 可以在 `Run` 方法中实现更全面的监视和异常处理策略，如重新启动失败的任务，或包含使角色可以停止和启动各个任务的代码。

当结构控制器关闭角色实例时，会调用以下代码中所示的 `Stop` 方法（从 `OnStop` 方法进行调用）。 该代码通过取消来正常停止每个任务。 如果任何任务完成所需的时间超过五分钟，则 `Stop` 方法中的取消处理会停止等待并终止角色。

```csharp
// Stop running tasks and wait for tasks to complete before returning
// unless the timeout expires.
private void Stop(TimeSpan timeout)
{
  Trace.TraceInformation("Stop called. Canceling tasks.");
  // Cancel running tasks.
  cts.Cancel();

  Trace.TraceInformation("Waiting for canceled tasks to finish and return");

  // Wait for all the tasks to complete before returning. Note that the
  // emulator currently allows 30 seconds and Azure allows five
  // minutes for processing to complete.
  try
  {
    Task.WaitAll(tasks.ToArray(), timeout);
  }
  catch (AggregateException ex)
  {
    Trace.TraceError(ex.Message);

    // If any of the inner exceptions in the aggregate exception
    // are not cancellation exceptions then rethrow the exception.
    ex.Handle(innerEx => (innerEx is OperationCanceledException));
  }
}
```

## <a name="related-patterns-and-guidance"></a>相关模式和指南

实施此模式时，可能也会与以下模式和指南相关：

- [Autoscaling Guidance](https://msdn.microsoft.com/library/dn589774.aspx)（自动缩放指南）。 自动缩放可以用于根据对处理的预计需求，启动和停止托管计算资源的服务的实例。

- [计算分区指南](https://msdn.microsoft.com/library/dn589773.aspx)。 介绍如何在云服务中分配服务和组件，以便最大程度地降低运行成本，同时保持服务的可伸缩性、性能、可用性和安全性。

- 此模式包含一个可下载的[示例应用程序](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation)。
