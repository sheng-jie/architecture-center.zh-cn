---
title: 云应用程序的性能反模式
description: 可能导致可伸缩性问题的常见做法。
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 423fe6533e57268610f625f523714cd1bce89546
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
ms.locfileid: "24538707"
---
# <a name="performance-antipatterns-for-cloud-applications"></a>云应用程序的性能反模式

性能反模式是指在应用程序承受压力时，可能导致可伸缩性问题的某种常见做法。 

下面是一个常见的情景：在测试期间，某个应用程序的行为正常。 然后，该应用程序被发布到生产环境，并开始处理实际工作负荷。 在这时起，它开始表现异常 &mdash; 拒绝用户请求、停滞或引发异常。 此时，开发团队面临两个问题：

- 在测试期间为何未发现此行为？
- 如何解决？

第一个问题的答案非常简单。 在测试环境中很难模拟真实的用户、他们的行为模式，以及他们可能执行的工作量。 了解系统在承受负载时的行为方式的唯一有把握方法是在生产环境中对它进行观察。 要澄清的一点是，我们并不建议跳过性能测试。 性能测试对于获得基准性能指标至关重要。 但是，必须准备好观察并纠正应用程序在实时系统中出现的性能问题。

至于第二个问题 - 如何解决的答案，就不是那么直接。 问题可能由或多或少的因素造成，有时，问题只会在特定的条件下出现。 检测和日志记录对于查找根本原因非常关键，但是，还必须知道要查看哪些信息。 

根据与 Microsoft Azure 客户的互动，我们已识别了客户在生产环境中遇到的最常见性能问题。 对于每个反模式，我们介绍了通常出现该反模式的原因、该反模式的症状，以及解决问题的方法。 此外，我们还提供了示例代码用于演示对立模式和建议的解决方法。 

阅读介绍时，其中一些反模式似乎很明显，但它们出现的次数比想象的更频繁。  有时，应用程序继承了本地工作的设计，但无法在云中缩放。 应用程序的设计一开始可能非常干净，但随着新功能的添加，其中一种或多种反模式就会渗入。 不管怎样，本指南都可帮助你识别并解决这些反模式。

下面是我们已识别的反模式列表： 

| 反模式 | 说明 |
|-------------|-------------|
| [繁忙数据库][BusyDatabase] | 将过多的处理工作附加到数据存储。 |
| [繁忙前端][BusyFrontEnd] | 将资源密集型任务转移到后台线程。 |
| [琐碎 I/O][ChattyIO] | 持续发送许多小型网络请求。 |
| [超量提取][ExtraneousFetching] | 检索的数据量超过需要，导致不必要的 I/O。 |
| [不当实例化][ImproperInstantiation] | 反复创建和销毁原本应该共享并重复使用的对象。 |
| [整体持久性][MonolithicPersistence] | 对采用截然不同的使用模式的数据使用相同的数据存储。 |
| [无缓存][NoCaching] | 无法缓存数据。 |
| [同步 I/O][SynchronousIO] | I/O 完成时阻塞调用线程。 | 

[BusyDatabase]: ./busy-database/index.md
[BusyFrontEnd]: ./busy-front-end/index.md
[ChattyIO]: ./chatty-io/index.md
[ExtraneousFetching]: ./extraneous-fetching/index.md
[ImproperInstantiation]: ./improper-instantiation/index.md
[MonolithicPersistence]: ./monolithic-persistence/index.md
[NoCaching]: ./no-caching/index.md
[SynchronousIO]: ./synchronous-io/index.md