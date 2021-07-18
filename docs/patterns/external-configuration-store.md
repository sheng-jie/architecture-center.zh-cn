---
title: 外部配置存储
description: 将配置信息从应用程序部署包移出，移到一个集中的位置。
keywords: 设计模式
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- management-monitoring
ms.openlocfilehash: 733ca979903d1526d3a1a6b281a8903893e19fda
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
ms.locfileid: "24542275"
---
# <a name="external-configuration-store-pattern"></a>外部配置存储模式

[!INCLUDE [header](../_includes/header.md)]

将配置信息从应用程序部署包移出，移到一个集中的位置。 这可以提供用于简化管理和控制配置数据，以及用于在应用程序和应用程序实例之间共享配置数据的机会。

## <a name="context-and-problem"></a>上下文和问题

应用程序运行时环境的大部分包括随应用程序部署的文件中保留的配置信息。 在某些情况下，可以编辑这些文件，以在部署应用程序之后更改应用程序行为。 但是，对配置的更改需要重新部署应用程序，常常导致不可接受的停机和其他管理开销。

本地配置文件也将配置限制到单个应用程序，但有时在多个应用程序之间共享配置设置会很有用。 示例包括数据库连接字符串、UI 主题信息或一组相关应用程序使用的队列和存储的 URL。

跨多个正在运行的应用程序实例管理本地配置的更改具有挑战性，特别是在云托管方案中。 它可能导致在部署更新的过程中实例使用不同的配置设置。

此外，应用程序和组件的更新可能需要更改配置架构。 许多配置系统不支持不同版本的配置信息。

## <a name="solution"></a>解决方案

将配置信息存储在外部存储中，并提供可用来快速、高效地读取和更新配置设置的接口。 外部存储的类型取决于应用程序的托管和运行时环境。 在云托管方案中，它通常是一种基于云的存储服务，但可能是托管数据库或其他系统。

为配置信息选择的后备存储应有一个接口，该接口提供一致和易于使用的访问。 它应以正确类型化和结构化的格式公开信息。 实现可能还需要授予用户的访问权限，以便保护配置数据，并有足够的灵活性以允许存储多个版本的配置（如开发、过渡或生产，包括每一个的多个发行版）。

> 许多内置配置系统在应用程序启动时读取数据并在内存中缓存数据，以提供快速访问并最大程度减少对应用程序性能的影响。 根据所使用的后备存储的类型以及此存储的延迟，在外部配置存储中实现一种缓存机制可能会有用。 有关详细信息，请参阅[缓存指南](https://msdn.microsoft.com/library/dn589802.aspx)。 该图说明了具有可选本地存储的外部配置存储模式的概述。

![具有可选本地存储的外部配置存储模式的概述](./_images/external-configuration-store-overview.png)


## <a name="issues-and-considerations"></a>问题和注意事项

在决定如何实现此模式时，请考虑以下几点：

选择提供可接受的性能、高可用性、可靠性，并可以作为应用程序维护和管理过程的一部分进行备份的后备存储。 在云托管应用程序中，使用云存储机制通常是满足这些需求的不错选择。

设计后备存储的架构，以使它可保留的信息类型具有灵活性。 确保它提供所有配置要求，如类型化的数据、设置的集合、多个版本的设置以及应用程序使用它需要的任何其他功能。 架构应该易于扩展，以在需求发生变化时支持其他设置。

请考虑后备存储的物理容量、它与存储配置信息的方式的关系，以及对性能的影响。 例如，存储包含配置信息的 XML 文档将需要配置界面或应用程序分析文档，以便读取各个设置。 它将使更新设置变得更复杂，尽管缓存设置可有助于抵消较慢的读取性能。

请考虑配置界面将如何允许控制配置设置的作用域和继承。 例如，可能要求将配置设置限定在组织、应用程序和计算机级别。 它可能需要对不同作用域的访问支持控制委派，并阻止或允许各个应用程序替代设置。

确保配置界面可以所需的格式（如类型化的值、集合、键/值对或属性包）公开配置数据。

请考虑当设置包含错误，或不存在于后备存储中时，配置存储接口的行为方式。 可能返回默认设置并记录错误比较合适。 此外，考虑配置设置密钥或名称的区分大小写、二进制数据的存储和处理以及处理 null 值或空值的方法等方面。

考虑如何保护配置数据以允许仅访问相应的用户和应用程序。 这可能是配置存储接口的一项功能，但还需要确保在没有适当权限的情况下不能直接访问后备存储中的数据。 确保严格分离读取配置数据和写入配置数据所需的权限。 还要考虑是否需要加密部分或全部配置设置，以及将如何在配置存储接口中实现这一操作。

在运行过程中会更改应用程序行为的集中存储的配置至关重要，应该使用与部署应用程序代码相同的机制部署、更新和管理它们。 例如，可能会影响多个应用程序的更改必须使用完整的测试和暂存部署方法执行，以确保更改适合使用此配置的所有应用程序。 如果管理员编辑某项设置以更新一个应用程序，则它可能对使用该同一设置的应用程序产生负面影响。

如果应用程序缓存配置信息，则需要在配置更改时警告应用程序。 可以对缓存的配置数据实现过期策略，以便定期自动刷新此信息和选取（以及操作）任何更改。

## <a name="when-to-use-this-pattern"></a>何时使用此模式

此模式适合用于：

- 在多个应用程序和应用程序实例之间共享的配置设置，或必须在多个应用程序和应用程序实例之间实施标准配置的情况。

- 不支持所有所需配置设置的标准配置系统，如存储图像或复杂数据类型。

- 作为应用程序的某些设置的互补存储，可能允许应用程序重写部分或全部集中存储的设置。

- 记录配置存储的部分或全部访问类型，可作为简化管理多个应用程序的一种方法，也可选用于监视配置设置的使用情况。

## <a name="example"></a>示例

在 Microsoft Azure 托管应用程序中，用于从外部存储配置信息的一个典型选择是使用 Azure 存储。 这是弹性的、提供高性能，并通过自动故障转移复制三次，以提供高可用性。 Azure 表存储提供键/值存储以及对值使用灵活架构的功能。 Azure Blob 存储提供分层的、基于容器的存储，可在单独命名的 blob 中存储任何类型的数据。

下面的示例演示可如何对 Blob 存储实现配置存储，以存储和公开配置信息。 `BlobSettingsStore` 类提取用于保存配置信息的 Blob 存储，并实现以下代码中所示的 `ISettingsStore` 接口。

> 此代码在 _ExternalConfigurationStore_ 解决方案中的 _ExternalConfigurationStore.Cloud_ 项目中提供，可从 [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store) 获取。

```csharp
public interface ISettingsStore
{
    Task<string> GetVersionAsync();

    Task<Dictionary<string, string>> FindAllAsync();
}
```

此接口定义用于检索和更新保存在配置存储中的配置设置的方法，并包括可用于检测最近是否修改任何配置设置的版本号。 `BlobSettingsStore` 类使用 Blob 的 `ETag` 属性实施版本控制。 每次写入 blob 时会自动更新 `ETag` 属性。

> 按照设计，这种简单的解决方案将所有配置设置公开为字符串值，而不是类型化的值。

`ExternalConfigurationManager` 类提供 `BlobSettingsStore` 对象的包装。 应用程序可以使用此类存储和检索配置信息。 此类使用 Microsoft [Reactive Extensions](https://msdn.microsoft.com/library/hh242985.aspx)（重新激活扩展）库公开通过实现 `IObservable` 接口对配置进行的任何更改。 如果通过调用 `SetAppSetting` 方法修改某项设置，则会引发 `Changed` 事件，并且会通知此事件的所有订阅服务器。

请注意，所有设置也会缓存在 `ExternalConfigurationManager` 类中的 `Dictionary` 对象内，以实现快速访问。 用于检索配置设置的 `GetSetting` 方法会从缓存中读取数据。 如果在缓存中找不到设置，可改为从 `BlobSettingsStore` 对象检索。

`GetSettings` 方法调用 `CheckForConfigurationChanges` 方法来检测 Blob 存储中的配置信息是否已发生更改。 它通过检查版本号并将其与 `ExternalConfigurationManager` 对象持有的当前版本号进行比较来执行此操作。 如果发生了一个或多个更改，则会引发 `Changed` 事件，并刷新 `Dictionary` 对象中缓存的配置设置。 这是[缓存端模式](cache-aside.md)的一个应用程序。

下面的代码示例演示如何实现 `Changed` 事件、`GetSettings` 方法和 `CheckForConfigurationChanges` 方法：

```csharp
public class ExternalConfigurationManager : IDisposable
{
  // An abstraction of the configuration store.
  private readonly ISettingsStore settings;
  private readonly ISubject<KeyValuePair<string, string>> changed;
  ...
  private readonly ReaderWriterLockSlim settingsCacheLock = new ReaderWriterLockSlim();
  private readonly SemaphoreSlim syncCacheSemaphore = new SemaphoreSlim(1);  
  ...
  private Dictionary<string, string> settingsCache;
  private string currentVersion;
  ...
  public ExternalConfigurationManager(ISettingsStore settings, ...)
  {
    this.settings = settings;
    ...
  }
  ...
  public IObservable<KeyValuePair<string, string>> Changed => this.changed.AsObservable();
  ...

  public string GetAppSetting(string key)
  {
    ...
    // Try to get the value from the settings cache. 
    // If there's a cache miss, get the setting from the settings store and refresh the settings cache.

    string value;
    try
    {
        this.settingsCacheLock.EnterReadLock();

        this.settingsCache.TryGetValue(key, out value);
    }
    finally
    {
        this.settingsCacheLock.ExitReadLock();
    }

    return value;
  }
  ...
  private void CheckForConfigurationChanges()
  {
    try
    {
        // It is assumed that updates are infrequent.
        // To avoid race conditions in refreshing the cache, synchronize access to the in-memory cache.
        await this.syncCacheSemaphore.WaitAsync();

        var latestVersion = await this.settings.GetVersionAsync();

        // If the versions are the same, nothing has changed in the configuration.
        if (this.currentVersion == latestVersion) return;

        // Get the latest settings from the settings store and publish changes.
        var latestSettings = await this.settings.FindAllAsync();

        // Refresh the settings cache.
        try
        {
            this.settingsCacheLock.EnterWriteLock();

            if (this.settingsCache != null)
            {
                //Notify settings changed
                latestSettings.Except(this.settingsCache).ToList().ForEach(kv => this.changed.OnNext(kv));
            }
            this.settingsCache = latestSettings;
        }
        finally
        {
            this.settingsCacheLock.ExitWriteLock();
        }

        // Update the current version.
        this.currentVersion = latestVersion;
    }
    catch (Exception ex)
    {
        this.changed.OnError(ex);
    }
    finally
    {
        this.syncCacheSemaphore.Release();
    }
  }
}
```

> `ExternalConfigurationManager` 类还提供了一个名为 `Environment` 的属性。 此属性支持在不同环境（如过渡和生产）中运行的应用程序的不同配置。

`ExternalConfigurationManager` 对象还可以定期查询 `BlobSettingsStore` 对象是否有任何更改。 在下面的代码中，`StartMonitor` 方法每隔一段时间就会调用 `CheckForConfigurationChanges` 以检测任何更改并引发 `Changed` 事件，如前所述。

```csharp
public class ExternalConfigurationManager : IDisposable
{
  ...
  private readonly ISubject<KeyValuePair<string, string>> changed;
  private Dictionary<string, string> settingsCache;
  private readonly CancellationTokenSource cts = new CancellationTokenSource();
  private Task monitoringTask;
  private readonly TimeSpan interval;

  private readonly SemaphoreSlim timerSemaphore = new SemaphoreSlim(1);
  ...
  public ExternalConfigurationManager(string environment) : this(new BlobSettingsStore(environment), TimeSpan.FromSeconds(15), environment)
  {
  }
  
  public ExternalConfigurationManager(ISettingsStore settings, TimeSpan interval, string environment)
  {
      this.settings = settings;
      this.interval = interval;
      this.CheckForConfigurationChangesAsync().Wait();
      this.changed = new Subject<KeyValuePair<string, string>>();
      this.Environment = environment;
  }
  ...
  /// <summary>
  /// Check to see if the current instance is monitoring for changes
  /// </summary>
  public bool IsMonitoring => this.monitoringTask != null && !this.monitoringTask.IsCompleted;

  /// <summary>
  /// Start the background monitoring for configuration changes in the central store
  /// </summary>
  public void StartMonitor()
  {
      if (this.IsMonitoring)
          return;

      try
      {
          this.timerSemaphore.Wait();

          // Check again to make sure we are not already running.
          if (this.IsMonitoring)
              return;

          // Start running our task loop.
          this.monitoringTask = ConfigChangeMonitor();
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  /// <summary>
  /// Loop that monitors for configuration changes
  /// </summary>
  /// <returns></returns>
  public async Task ConfigChangeMonitor()
  {
      while (!cts.Token.IsCancellationRequested)
      {
          await this.CheckForConfigurationChangesAsync();
          await Task.Delay(this.interval, cts.Token);
      }
  }

  /// <summary>
  /// Stop monitoring for configuration changes
  /// </summary>
  public void StopMonitor()
  {
      try
      {
          this.timerSemaphore.Wait();

          // Signal the task to stop.
          this.cts.Cancel();

          // Wait for the loop to stop.
          this.monitoringTask.Wait();

          this.monitoringTask = null;
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  public void Dispose()
  {
      this.cts.Cancel();
  }
  ...
}
```

`ExternalConfigurationManager` 类被如下所示的 `ExternalConfiguration` 类实例化为单一实例。

```csharp
public static class ExternalConfiguration
{
    private static readonly Lazy<ExternalConfigurationManager> configuredInstance = new Lazy<ExternalConfigurationManager>(
        () =>
        {
            var environment = CloudConfigurationManager.GetSetting("environment");
            return new ExternalConfigurationManager(environment);
        });

    public static ExternalConfigurationManager Instance => configuredInstance.Value;
}
```

以下代码摘自 _ExternalConfigurationStore.Cloud_ 项目中的 `WorkerRole` 类。 它显示应用程序如何使用 `ExternalConfiguration` 类读取设置。

```csharp
public override void Run()
{
  // Start monitoring configuration changes.
  ExternalConfiguration.Instance.StartMonitor();

  // Get a setting.
  var setting = ExternalConfiguration.Instance.GetAppSetting("setting1");
  Trace.TraceInformation("Worker Role: Get setting1, value: " + setting);

  this.completeEvent.WaitOne();
}
```

同样来自于 `WorkerRole` 类的以下代码显示应用程序如何订阅配置事件。

```csharp
public override bool OnStart()
{
  ...
  // Subscribe to the event.
  ExternalConfiguration.Instance.Changed.Subscribe(
     m => Trace.TraceInformation("Configuration has changed. Key:{0} Value:{1}",
          m.Key, m.Value),
     ex => Trace.TraceError("Error detected: " + ex.Message));
  ...
}
```

## <a name="related-patterns-and-guidance"></a>相关模式和指南

- 演示此模式的示例可在 [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store) 上找到。
