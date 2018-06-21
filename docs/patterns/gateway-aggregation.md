---
title: 网关聚合模式
description: 使用网关可将多个单独请求聚合成一个请求。
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: f59c8b8b02c6db28024d13621b782997e63a4e9e
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
ms.locfileid: "24541267"
---
# <a name="gateway-aggregation-pattern"></a>网关聚合模式

使用网关可将多个单独请求聚合成一个请求。 当客户端必须向不同的后端系统发出多个调用来执行某项操作时，此模式非常有用。

## <a name="context-and-problem"></a>上下文和问题

若要执行单个任务，客户端可能需要向不同的后端服务发出多个调用。 依赖使用许多服务执行某项任务的应用程序必须扩展每个请求的资源。 将任何新的功能或服务添加到应用程序时，需要额外的请求，从而进一步提高了资源要求并增加了网络调用。 客户端与后端之间的这种频繁通信可能会对应用程序的性能和规模产生不利影响。  此问题在微服务体系结构中更常见，因为围绕许多小型服务构建的应用程序原生就包含更多的跨服务调用。 

在下图中，客户端向每个服务发送请求 (1,2,3)。 每个服务处理该请求，然后向应用程序返回响应 (4,5,6)。 通过延迟通常较高的移动电话网络，以这种方式使用各个请求并不高效，可能导致连接断开或请求不完整。 尽管可以并行执行每个请求，但应用程序必须发送、等待并处理每个请求的数据，而所有这些操作都要通过单独的连接完成，因此增大了故障可能性。

![](./_images/gateway-aggregation-problem.png) 

## <a name="solution"></a>解决方案

使用网关减少客户端与服务之间的通信频率。 网关会接收客户端请求，将请求分派到不同的后端系统，然后聚合结果并将其返回给请求客户端。

此模式可以减少应用程序向后端服务发出的请求数，并通过高延迟网络改进应用程序的性能。

在下图中，应用程序网关发送一个请求 (1)。 该请求包含其他一些请求。 网关分解其他这些请求，并通过将每个请求发送到相关的服务来处理每个请求 (2)。 每个服务向网关返回响应 (3)。 网关组合来自每个服务的响应，并向应用程序发送响应 (4)。 应用程序发出单个请求，并仅接收来自网关的单个响应。

![](./_images/gateway-aggregation.png)

## <a name="issues-and-considerations"></a>问题和注意事项

- 网关不应在后端服务之间造成服务耦合。
- 网关应靠近后端服务，以尽量降低延迟。
- 网关服务可能会造成单一故障点。 请确保网关设计合理，符合应用程序的可用性要求。
- 网关可能造成瓶颈。 请确保网关可提供足够的性能来处理负载，并可根据预期的发展进行缩放。
- 对网关执行负载测试，确保不会对服务造成连锁故障。
- 使用[隔舱][bulkhead]、[断路][circuit-breaker]、[重试][retry]和超时等技术实施弹性设计。
- 如果一个或多个服务调用花费的时间过长，可接受超时并返回一部分数据。 请考虑应用程序处理这种情况的方式。
- 使用异步 I/O 来确保后端延迟不会导致应用程序中出现性能问题。
- 使用关联 ID 实施分布式跟踪，以跟踪每个调用。
- 监视请求指标和响应大小。
- 考虑返回缓存的数据（作为故障转移策略）来处理故障。
- 不要在网关中内置聚合，而应考虑将聚合服务放在网关后面。 请求聚合的资源要求可能与网关中其他服务不同，并可能影响网关的路由和卸载功能。

## <a name="when-to-use-this-pattern"></a>何时使用此模式

在以下情况下使用此模式：

- 客户端需与多个后端服务通信来执行某个操作。
- 客户端可能要使用延迟很高的网络，例如移动电话网络。

此模式可能不适用于以下情况：

- 在执行多个不同的操作时，希望减少客户端与单个服务之间的调用次数。 在这种情况下，将批处理操作添加到服务可能更有利。
- 客户端或应用程序靠近后端服务，延迟并不是一个重要考虑因素。

## <a name="example"></a>示例

以下示例演示如何使用 Lua 创建一个简单的网关聚合 NGINX 服务。

```lua
worker_processes  4;

events {
  worker_connections 1024;
}

http {
  server {
    listen 80;

    location = /batch {
      content_by_lua '
        ngx.req.read_body()

        -- read json body content
        local cjson = require "cjson"
        local batch = cjson.decode(ngx.req.get_body_data())["batch"]

        -- create capture_multi table
        local requests = {}
        for i, item in ipairs(batch) do
          table.insert(requests, {item.relative_url, { method = ngx.HTTP_GET}})
        end

        -- execute batch requests in parallel
        local results = {}
        local resps = { ngx.location.capture_multi(requests) }
        for i, res in ipairs(resps) do
          table.insert(results, {status = res.status, body = cjson.decode(res.body), header = res.header})
        end

        ngx.say(cjson.encode({results = results}))
      ';
    }

    location = /service1 {
      default_type application/json;
      echo '{"attr1":"val1"}';
    }

    location = /service2 {
      default_type application/json;
      echo '{"attr2":"val2"}';
    }
  }
}
```

## <a name="related-guidance"></a>相关指南

- [用于前端的后端模式](./backends-for-frontends.md)
- [网关卸载模式](./gateway-offloading.md)
- [网关路由模式](./gateway-routing.md)

[bulkhead]: ./bulkhead.md
[circuit-breaker]: ./circuit-breaker.md
[retry]: ./retry.md