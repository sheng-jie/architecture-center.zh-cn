---
title: "API 设计指南"
description: "有关如何创建设计良好的 API 的指南。"
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 3ffadce1b0c4a4da808e52d61cff0b7f0b27de11
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="api-design"></a><span data-ttu-id="5024b-103">API 设计</span><span class="sxs-lookup"><span data-stu-id="5024b-103">API design</span></span>
[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="5024b-104">许多现代的基于 Web 的解决方案都利用 Web 服务器托管的 Web 服务，来为远程客户端应用程序提供相关功能。</span><span class="sxs-lookup"><span data-stu-id="5024b-104">Many modern web-based solutions make the use of web services, hosted by web servers, to provide functionality for remote client applications.</span></span> <span data-ttu-id="5024b-105">Web 服务公开的操作构成 Web API。</span><span class="sxs-lookup"><span data-stu-id="5024b-105">The operations that a web service exposes constitute a web API.</span></span> <span data-ttu-id="5024b-106">设计良好的 Web API 应旨在支持：</span><span class="sxs-lookup"><span data-stu-id="5024b-106">A well-designed web API should aim to support:</span></span>

* <span data-ttu-id="5024b-107">**平台独立性**。</span><span class="sxs-lookup"><span data-stu-id="5024b-107">**Platform independence**.</span></span> <span data-ttu-id="5024b-108">客户端应用程序应能够利用 Web 服务提供的 API，而不需要了解该 API 公开的数据或操作如何物理实现。</span><span class="sxs-lookup"><span data-stu-id="5024b-108">Client applications should be able to utilize the API that the web service provides without requiring how the data or operations that API exposes are physically implemented.</span></span> <span data-ttu-id="5024b-109">这就要求该 API 遵守通用标准，使客户端应用程序和 Web 服务可以就以下项达成一致：要使用哪些数据格式以及客户端应用程序和 Web 服务之间交换的数据的结构。</span><span class="sxs-lookup"><span data-stu-id="5024b-109">This requires that the API abides by common standards that enable a client application and web service to agree on which data formats to use, and the structure of the data that is exchanged between client applications and the web service.</span></span>
* <span data-ttu-id="5024b-110">**服务演变**。</span><span class="sxs-lookup"><span data-stu-id="5024b-110">**Service evolution**.</span></span> <span data-ttu-id="5024b-111">Web 服务应能够改进和添加（或删除）功能，而不会影响客户端应用程序。</span><span class="sxs-lookup"><span data-stu-id="5024b-111">The web service should be able to evolve and add (or remove) functionality independently from client applications.</span></span> <span data-ttu-id="5024b-112">当 Web 服务所提供的功能发生更改时，现有客户端应用程序应该能够继续不变地运行。</span><span class="sxs-lookup"><span data-stu-id="5024b-112">Existing client applications should be able to continue to operate unmodified as the features provided by the web service change.</span></span> <span data-ttu-id="5024b-113">所有功能还应是可检测到的，以便客户端应用程序能够充分利用它。</span><span class="sxs-lookup"><span data-stu-id="5024b-113">All functionality should also be discoverable, so that client applications can fully utilize it.</span></span>

<span data-ttu-id="5024b-114">本指南旨在说明设计 Web API 时应考虑的问题。</span><span class="sxs-lookup"><span data-stu-id="5024b-114">The purpose of this guidance is to describe the issues that you should consider when designing a web API.</span></span>

## <a name="introduction-to-representational-state-transfer-rest"></a><span data-ttu-id="5024b-115">表述性状态传递 (REST) 简介</span><span class="sxs-lookup"><span data-stu-id="5024b-115">Introduction to Representational State Transfer (REST)</span></span>
<span data-ttu-id="5024b-116">Roy Fielding 在他 2000 年的论文中提出了一种替代架构方法，用于组织 Web 服务公开的操作；REST。</span><span class="sxs-lookup"><span data-stu-id="5024b-116">In his dissertation in 2000, Roy Fielding proposed an alternative architectural approach to structuring the operations exposed by web services; REST.</span></span> <span data-ttu-id="5024b-117">REST 是基于超媒体构建分布式系统的架构样式。</span><span class="sxs-lookup"><span data-stu-id="5024b-117">REST is an architectural style for building distributed systems based on hypermedia.</span></span> <span data-ttu-id="5024b-118">REST 模型的主要优势在于它基于开放标准，并且不将模型的实现或访问它的客户端应用程序绑定到任何特定实现。</span><span class="sxs-lookup"><span data-stu-id="5024b-118">A primary advantage of the REST model is that it is based on open standards and does not bind the implementation of the model or the client applications that access it to any specific implementation.</span></span> <span data-ttu-id="5024b-119">例如，REST Web 服务可以使用 Microsoft ASP.NET Web API 加以实现，而客户端应用程序则可通过使用任何可生成 HTTP 请求并分析 HTTP 响应的语言和工具集进行开发。</span><span class="sxs-lookup"><span data-stu-id="5024b-119">For example, a REST web service could be implemented by using the Microsoft ASP.NET Web API, and client applications could be developed by using any language and toolset that can generate HTTP requests and parse HTTP responses.</span></span>

> [!NOTE]
> <span data-ttu-id="5024b-120">REST 实际上独立于任何基础协议，并且不一定绑定到 HTTP。</span><span class="sxs-lookup"><span data-stu-id="5024b-120">REST is actually independent of any underlying protocol and is not necessarily tied to HTTP.</span></span> <span data-ttu-id="5024b-121">但是，最常见的基于 REST 的系统实现利用 HTTP 作为发送和接收请求的应用程序协议。</span><span class="sxs-lookup"><span data-stu-id="5024b-121">However, most common implementations of systems that are based on REST utilize HTTP as the application protocol for sending and receiving requests.</span></span> <span data-ttu-id="5024b-122">本文档重点介绍如何将 REST 原则映射到设计为使用 HTTP 运行的系统。</span><span class="sxs-lookup"><span data-stu-id="5024b-122">This document focuses on mapping REST principles to systems designed to operate using HTTP.</span></span>
>
>

<span data-ttu-id="5024b-123">REST 模型使用导航方案来表示网络上的对象和服务（称为*资源*）。</span><span class="sxs-lookup"><span data-stu-id="5024b-123">The REST model uses a navigational scheme to represent objects and services over a network (referred to as *resources*).</span></span> <span data-ttu-id="5024b-124">实现 REST 的许多系统通常使用 HTTP 协议来传输要访问这些资源的请求。</span><span class="sxs-lookup"><span data-stu-id="5024b-124">Many systems that implement REST typically use the HTTP protocol to transmit requests to access these resources.</span></span> <span data-ttu-id="5024b-125">在这些系统中，客户端应用程序以 URI 的形式提交请求，其中标识资源和 HTTP 方法（最常见的有 GET、POST、PUT 或 DELETE，用于指示要对该资源执行的操作）。</span><span class="sxs-lookup"><span data-stu-id="5024b-125">In these systems, a client application submits a request in the form of a URI that identifies a resource, and an HTTP method (the most common being GET, POST, PUT, or DELETE) that indicates the operation to be performed on that resource.</span></span>  <span data-ttu-id="5024b-126">HTTP 请求的正文包含执行该操作所需的数据。</span><span class="sxs-lookup"><span data-stu-id="5024b-126">The body of the HTTP request contains the data required to perform the operation.</span></span> <span data-ttu-id="5024b-127">要了解的要点是 REST 定义无状态的请求模型。</span><span class="sxs-lookup"><span data-stu-id="5024b-127">The important point to understand is that REST defines a stateless request model.</span></span> <span data-ttu-id="5024b-128">HTTP 请求应是独立的，并可以任何顺序发出，因此尝试保留请求之间的瞬时状态信息并不可行。</span><span class="sxs-lookup"><span data-stu-id="5024b-128">HTTP requests should be independent and may occur in any order, so attempting to retain transient state information between requests is not feasible.</span></span>  <span data-ttu-id="5024b-129">存储信息的唯一位置是在资源本身中，并且每个请求应是原子操作。</span><span class="sxs-lookup"><span data-stu-id="5024b-129">The only place where information is stored is in the resources themselves, and each request should be an atomic operation.</span></span> <span data-ttu-id="5024b-130">实际上，REST 模型可实现有限状态机，其中请求将资源从一种明确定义的非瞬时状态转换为另一种状态。</span><span class="sxs-lookup"><span data-stu-id="5024b-130">Effectively, a REST model implements a finite state machine where a request transitions a resource from one well-defined non-transient state to another.</span></span>

> [!NOTE]
> <span data-ttu-id="5024b-131">REST 模型中单个请求的无状态性质使得按照这些原则构造的系统高度可缩放。</span><span class="sxs-lookup"><span data-stu-id="5024b-131">The stateless nature of individual requests in the REST model enables a system constructed by following these principles to be highly scalable.</span></span> <span data-ttu-id="5024b-132">发出一系列请求的客户端应用程序与处理这些请求的特定 Web 服务器的之间无需保留任何关联。</span><span class="sxs-lookup"><span data-stu-id="5024b-132">There is no need to retain any affinity between a client application making a series of requests and the specific web servers handling those requests.</span></span>
>
>

<span data-ttu-id="5024b-133">实现有效的 REST 模型的另一个要点是了解该模型提供访问的各种资源之间的关系。</span><span class="sxs-lookup"><span data-stu-id="5024b-133">Another crucial point in implementing an effective REST model is to understand the relationships between the various resources to which the model provides access.</span></span> <span data-ttu-id="5024b-134">这些资源通常被组织为集合和关系。</span><span class="sxs-lookup"><span data-stu-id="5024b-134">These resources are typically organized as collections and relationships.</span></span> <span data-ttu-id="5024b-135">例如，假设一个电子商务系统的快速分析显示有两个集合客户端应用程序可能感兴趣：订单和客户。</span><span class="sxs-lookup"><span data-stu-id="5024b-135">For example, suppose that a quick analysis of an ecommerce system shows that there are two collections in which client applications are likely to be interested: orders and customers.</span></span> <span data-ttu-id="5024b-136">为了进行标识，每个订单和客户都应有自己的唯一键。</span><span class="sxs-lookup"><span data-stu-id="5024b-136">Each order and customer should have its own unique key for identification purposes.</span></span> <span data-ttu-id="5024b-137">用于访问订单集合的 URI 可以是如下简单的形式：*/orders*，同样，用于检索所有客户的 URI 可以是 */customers*。</span><span class="sxs-lookup"><span data-stu-id="5024b-137">The URI to access the collection of orders could be something as simple as */orders*, and similarly the URI for retrieving all customers could be */customers*.</span></span> <span data-ttu-id="5024b-138">向 */orders* URI 发出 HTTP GET 请求应返回表示编码为 HTTP 响应的集合中的所有订单的列表：</span><span class="sxs-lookup"><span data-stu-id="5024b-138">Issuing an HTTP GET request to the */orders* URI should return a list representing all orders in the collection encoded as an HTTP response:</span></span>

```HTTP
GET http://adventure-works.com/orders HTTP/1.1
...
```

<span data-ttu-id="5024b-139">下面所示的响应将订单编码为 JSON 列表结构：</span><span class="sxs-lookup"><span data-stu-id="5024b-139">The response shown below encodes the orders as a JSON list structure:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Date: Fri, 22 Aug 2014 08:49:02 GMT
Content-Length: ...
[{"orderId":1,"orderValue":99.90,"productId":1,"quantity":1},{"orderId":2,"orderValue":10.00,"productId":4,"quantity":2},{"orderId":3,"orderValue":16.60,"productId":2,"quantity":4},{"orderId":4,"orderValue":25.90,"productId":3,"quantity":1},{"orderId":5,"orderValue":99.90,"productId":1,"quantity":1}]
```
<span data-ttu-id="5024b-140">若要提取单个订单，需要指定*订单*资源中的订单标识符，例如 */orders/2*：</span><span class="sxs-lookup"><span data-stu-id="5024b-140">To fetch an individual order requires specifying the identifier for the order from the *orders* resource, such as */orders/2*:</span></span>

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
...
```

```HTTP
HTTP/1.1 200 OK
...
Date: Fri, 22 Aug 2014 08:49:02 GMT
Content-Length: ...
{"orderId":2,"orderValue":10.00,"productId":4,"quantity":2}
```

> [!NOTE]
> <span data-ttu-id="5024b-141">为简单起见，这些示例显示响应中作为 JSON 文本数据返回的信息。</span><span class="sxs-lookup"><span data-stu-id="5024b-141">For simplicity, these examples show the information in responses being returned as JSON text data.</span></span> <span data-ttu-id="5024b-142">但是，没有理由资源不应包含 HTTP 支持的任何其他类型的数据（如二进制或加密信息）；HTTP 响应中的 content-type 应指定类型。</span><span class="sxs-lookup"><span data-stu-id="5024b-142">However, there is no reason why resources should not contain any other type of data supported by HTTP, such as binary or encrypted information; the content-type in the HTTP response should specify the type.</span></span> <span data-ttu-id="5024b-143">此外，REST 模型还能够以不同格式返回相同数据，如 XML 或 JSON。</span><span class="sxs-lookup"><span data-stu-id="5024b-143">Also, a REST model may be able to return the same data in different formats, such as XML or JSON.</span></span> <span data-ttu-id="5024b-144">在这种情况下，Web 服务应能够与发出请求的客户端执行内容协商。</span><span class="sxs-lookup"><span data-stu-id="5024b-144">In this case, the web service should be able to perform content negotiation with the client making the request.</span></span> <span data-ttu-id="5024b-145">请求可包含 *Accept* 标头以指定客户端想要接收的首选格式，Web 服务应尝试尽可能采用这种格式。</span><span class="sxs-lookup"><span data-stu-id="5024b-145">The request can include an *Accept* header which specifies the preferred format that the client would like to receive and the web service should attempt to honor this format if at all possible.</span></span>
>
>

<span data-ttu-id="5024b-146">请注意，REST 请求的响应需使用标准 HTTP 状态代码。</span><span class="sxs-lookup"><span data-stu-id="5024b-146">Notice that the response from a REST request makes use of the standard HTTP status codes.</span></span> <span data-ttu-id="5024b-147">例如，返回有效数据的请求应包括 HTTP 响应代码 200（正常），而无法找到或删除指定资源的请求则应返回包含 HTTP 状态代码 404（未找到）的响应。</span><span class="sxs-lookup"><span data-stu-id="5024b-147">For example, a request that returns valid data should include the HTTP response code 200 (OK), while a request that fails to find or delete a specified resource should return a response that includes the HTTP status code 404 (Not Found).</span></span>

## <a name="design-and-structure-of-a-restful-web-api"></a><span data-ttu-id="5024b-148">设计和组织 RESTful Web API</span><span class="sxs-lookup"><span data-stu-id="5024b-148">Design and structure of a RESTful web API</span></span>
<span data-ttu-id="5024b-149">设计成功的 Web API 的关键是简单和一致性。</span><span class="sxs-lookup"><span data-stu-id="5024b-149">The keys to designing a successful web API are simplicity and consistency.</span></span> <span data-ttu-id="5024b-150">使用表现出这两个因素的 Web API，可更轻松地生成需要使用该 API 的客户端应用程序。</span><span class="sxs-lookup"><span data-stu-id="5024b-150">A Web API that exhibits these two factors makes it easier to build client applications that need to consume the API.</span></span>

<span data-ttu-id="5024b-151">RESTful Web API 着重于公开一组连接资源，并提供核心操作使应用程序可以处理这些资源并在这些资源之间轻松导航。</span><span class="sxs-lookup"><span data-stu-id="5024b-151">A RESTful web API is focused on exposing a set of connected resources, and providing the core operations that enable an application to manipulate these resources and easily navigate between them.</span></span> <span data-ttu-id="5024b-152">为此，构成典型 RESTful Web API 的 URI 应面向它所公开的数据，并使用 HTTP 提供的工具来操作此数据。</span><span class="sxs-lookup"><span data-stu-id="5024b-152">For this reason, the URIs that constitute a typical RESTful web API should be oriented towards the data that it exposes, and use the facilities provided by HTTP to operate on this data.</span></span> <span data-ttu-id="5024b-153">此方法的思维方式需要与在面向对象的 API 中设计一组类时通常采用的思维方式不同，后者往往更多由对象和类的行为驱动。</span><span class="sxs-lookup"><span data-stu-id="5024b-153">This approach requires a different mindset from that typically employed when designing a set of classes in an object-oriented API which tends to be more motivated by the behavior of objects and classes.</span></span> <span data-ttu-id="5024b-154">此外，RESTful Web API 应是无状态的，并且不依赖于按特定顺序调用的操作。</span><span class="sxs-lookup"><span data-stu-id="5024b-154">Additionally, a RESTful web API should be stateless and not depend on operations being invoked in a particular sequence.</span></span> <span data-ttu-id="5024b-155">以下部分总结了在设计 RESTful Web API 时应考虑的点。</span><span class="sxs-lookup"><span data-stu-id="5024b-155">The following sections summarize the points you should consider when designing a RESTful web API.</span></span>

### <a name="organizing-the-web-api-around-resources"></a><span data-ttu-id="5024b-156">围绕资源组织 Web API</span><span class="sxs-lookup"><span data-stu-id="5024b-156">Organizing the web API around resources</span></span>
> [!TIP]
> <span data-ttu-id="5024b-157">REST Web 服务公开的 URI 应基于名词（Web API 提供访问的数据），而不是基于动词（应用程序可以对数据执行什么操作）。</span><span class="sxs-lookup"><span data-stu-id="5024b-157">The URIs exposed by a REST web service should be based on nouns (the data to which the web API provides access) and not verbs (what an application can do with the data).</span></span>
>
>

<span data-ttu-id="5024b-158">侧重于 Web API 公开的业务实体。</span><span class="sxs-lookup"><span data-stu-id="5024b-158">Focus on the business entities that the web API exposes.</span></span> <span data-ttu-id="5024b-159">例如，在前面所述的设计为支持电子商务系统的 Web API 中，主要实体为客户和订单。</span><span class="sxs-lookup"><span data-stu-id="5024b-159">For example, in a web API designed to support the ecommerce system described earlier, the primary entities are customers and orders.</span></span> <span data-ttu-id="5024b-160">下订单操作等过程可以通过提供 HTTP POST 操作完成，该操作接受订单信息并将其添加到客户的订单列表中。</span><span class="sxs-lookup"><span data-stu-id="5024b-160">Processes such as the act of placing an order can be achieved by providing an HTTP POST operation that takes the order information and adds it to the list of orders for the customer.</span></span> <span data-ttu-id="5024b-161">在内部，此 POST 操作可以执行检查库存级别和为客户计费等任务。</span><span class="sxs-lookup"><span data-stu-id="5024b-161">Internally, this POST operation can perform tasks such as checking stock levels, and billing the customer.</span></span> <span data-ttu-id="5024b-162">HTTP 响应可以指示下订单是否成功。</span><span class="sxs-lookup"><span data-stu-id="5024b-162">The HTTP response can indicate whether the order was placed successfully or not.</span></span> <span data-ttu-id="5024b-163">另请注意，资源无需基于单个物理数据项目。</span><span class="sxs-lookup"><span data-stu-id="5024b-163">Also note that a resource does not have to be based on a single physical data item.</span></span> <span data-ttu-id="5024b-164">例如，订单资源可能会通过使用从分布在关系数据库中多个表的多个行聚合的信息在内部实现，但以单个实体的形式提供给客户端。</span><span class="sxs-lookup"><span data-stu-id="5024b-164">As an example, an order resource might be implemented internally by using information aggregated from many rows spread across several tables in a relational database but presented to the client as a single entity.</span></span>

> [!TIP]
> <span data-ttu-id="5024b-165">避免设计这样的 REST 接口：镜像或依赖于它公开的数据的内部结构。</span><span class="sxs-lookup"><span data-stu-id="5024b-165">Avoid designing a REST interface that mirrors or depends on the internal structure of the data that it exposes.</span></span> <span data-ttu-id="5024b-166">REST 不只是关于对关系数据库中的单独表实现简单 CRUD（创建、检索、更新、删除）操作。</span><span class="sxs-lookup"><span data-stu-id="5024b-166">REST is about more than implementing simple CRUD (Create, Retrieve, Update, Delete) operations over separate tables in a relational database.</span></span> <span data-ttu-id="5024b-167">REST 旨在将业务实体和应用程序可以对这些实体上执行的操作映射到这些实体的物理实现，但不应向客户端公开这些物理详细信息。</span><span class="sxs-lookup"><span data-stu-id="5024b-167">The purpose of REST is to map business entities and the operations that an application can perform on these entities to the physical implementation of these entities, but a client should not be exposed to these physical details.</span></span>
>
>

<span data-ttu-id="5024b-168">单个业务实体很少会孤立存在（尽管可能存在某些单一实例对象），而是往往会组合在一起成为集合。</span><span class="sxs-lookup"><span data-stu-id="5024b-168">Individual business entities rarely exist in isolation (although some singleton objects may exist), but instead tend to be grouped together into collections.</span></span> <span data-ttu-id="5024b-169">在 REST 术语中，每个实体和每个集合都是资源。</span><span class="sxs-lookup"><span data-stu-id="5024b-169">In REST terms, each entity and each collection are resources.</span></span> <span data-ttu-id="5024b-170">在 RESTful Web API 中，每个集合都在 Web 服务中有自己的 URI，对集合的 URI 执行 HTTP GET 请求将检索该集合中的项目的列表。</span><span class="sxs-lookup"><span data-stu-id="5024b-170">In a RESTful web API, each collection has its own URI within the web service, and performing an HTTP GET request over a URI for a collection retrieves a list of items in that collection.</span></span> <span data-ttu-id="5024b-171">每个单个项目也具有自己的 URI，应用程序可以使用该 URI 再提交一个 HTTP GET 请求以检索该项目的详细信息。</span><span class="sxs-lookup"><span data-stu-id="5024b-171">Each individual item also has its own URI, and an application can submit another HTTP GET request using that URI to retrieve the details of that item.</span></span> <span data-ttu-id="5024b-172">应以分层方式组织集合和项目的 URI。</span><span class="sxs-lookup"><span data-stu-id="5024b-172">You should organize the URIs for collections and items in a hierarchical manner.</span></span> <span data-ttu-id="5024b-173">在电子商务系统中，URI */customers* 表示客户的集合，*/customers/5* 检索此集合中 ID 为 5 的单个客户的详细信息。</span><span class="sxs-lookup"><span data-stu-id="5024b-173">In the ecommerce system, the URI */customers* denotes the customer’s collection, and */customers/5* retrieves the details for the single customer with the ID 5 from this collection.</span></span> <span data-ttu-id="5024b-174">这种方法有助于使 Web API 保持直观。</span><span class="sxs-lookup"><span data-stu-id="5024b-174">This approach helps to keep the web API intuitive.</span></span>

> [!TIP]
> <span data-ttu-id="5024b-175">在 URI 中采用一致的命名约定；通常，这有助于对引用集合的 URI 中使用复数名词。</span><span class="sxs-lookup"><span data-stu-id="5024b-175">Adopt a consistent naming convention in URIs; in general it helps to use plural nouns for URIs that reference collections.</span></span>
>
>

<span data-ttu-id="5024b-176">还需要考虑不同类型的资源之间的关系，以及你可能会如何公开这些关联。</span><span class="sxs-lookup"><span data-stu-id="5024b-176">You also need to consider the relationships between different types of resources and how you might expose these associations.</span></span> <span data-ttu-id="5024b-177">例如，客户可能会下零个或更多订单。</span><span class="sxs-lookup"><span data-stu-id="5024b-177">For example, customers may place zero or more orders.</span></span> <span data-ttu-id="5024b-178">表示此关系的自然方式可以是通过 URI（如 */customers/5/orders*）查找客户 5 的所有订单。</span><span class="sxs-lookup"><span data-stu-id="5024b-178">A natural way to represent this relationship would be through a URI such as */customers/5/orders* to find all the orders for customer 5.</span></span> <span data-ttu-id="5024b-179">还可以考虑通过 URI 从订单追溯到特定客户来表示此关联，例如 */orders/99/customer* 可查找订单 99 的客户，但扩展此模型太长可能会变得难以实现。</span><span class="sxs-lookup"><span data-stu-id="5024b-179">You might also consider representing the association from an order back to a specific customer through a URI such as */orders/99/customer* to find the customer for order 99, but extending this model too far can become cumbersome to implement.</span></span> <span data-ttu-id="5024b-180">较好的解决方案是在查询订单时返回的 HTTP 响应消息的正文中提供指向关联资源（例如，客户）的可导航链接。</span><span class="sxs-lookup"><span data-stu-id="5024b-180">A better solution is to provide navigable links to associated resources, such as the customer, in the body of the HTTP response message returned when the order is queried.</span></span> <span data-ttu-id="5024b-181">此机制将稍后在本指南的“使用 HATEOAS 方法启用相关资源的导航”一节中详细介绍。</span><span class="sxs-lookup"><span data-stu-id="5024b-181">This mechanism is described in more detail in the section Using the HATEOAS Approach to Enable Navigation To Related Resources later in this guidance.</span></span>

<span data-ttu-id="5024b-182">在更复杂的系统中可能会有更多类型的实体，这容易让人想到提供 URI 使客户端应用程序可以在多级关系中导航，例如 */customers/1/orders/99/products* 用于获取客户 1 下的订单 99 中的产品列表。</span><span class="sxs-lookup"><span data-stu-id="5024b-182">In more complex systems there may be many more types of entity, and it can be tempting to provide URIs that enable a client application to navigate through several levels of relationships, such as */customers/1/orders/99/products* to obtain the list of products in order 99 placed by customer 1.</span></span> <span data-ttu-id="5024b-183">但是，如果资源之间的关系在将来更改，此级别的复杂性可能很难维护并且不够灵活。</span><span class="sxs-lookup"><span data-stu-id="5024b-183">However, this level of complexity can be difficult to maintain and is inflexible if the relationships between resources change in the future.</span></span> <span data-ttu-id="5024b-184">更准确地说，应尽量保持 URI 相对简单。</span><span class="sxs-lookup"><span data-stu-id="5024b-184">Rather, you should seek to keep URIs relatively simple.</span></span> <span data-ttu-id="5024b-185">请记住，应用程序具有对资源的引用后，它应能够使用此引用查找与该资源相关的项目。</span><span class="sxs-lookup"><span data-stu-id="5024b-185">Bear in mind that once an application has a reference to a resource, it should be possible to use this reference to find items related to that resource.</span></span> <span data-ttu-id="5024b-186">前面的查询可以替换为 URI /customers/1/orders 以查找客户 1 的所有订单，然后查询 URI /orders/99/products 以查找此订单中的产品（假定订单 99 由客户 1 放置）。</span><span class="sxs-lookup"><span data-stu-id="5024b-186">The preceding query can be replaced with the URI */customers/1/orders* to find all the orders for customer 1, and then query the URI */orders/99/products* to find the products in this order (assuming order 99 was placed by customer 1).</span></span>

> [!TIP]
> <span data-ttu-id="5024b-187">避免需要复杂度超过*集合/项目/集合*的资源 URI。</span><span class="sxs-lookup"><span data-stu-id="5024b-187">Avoid requiring resource URIs more complex than *collection/item/collection*.</span></span>
>
>

<span data-ttu-id="5024b-188">要考虑的另一点是：所有 Web 请求都在 Web 服务器上施加负载，请求数越多，负载越大。</span><span class="sxs-lookup"><span data-stu-id="5024b-188">Another point to consider is that all web requests impose a load on the web server, and the greater the number of requests the bigger the load.</span></span> <span data-ttu-id="5024b-189">应尝试定义资源以避免使用公开大量小型资源的“细粒度”Web API。</span><span class="sxs-lookup"><span data-stu-id="5024b-189">You should attempt to define your resources to avoid “chatty” web APIs that expose a large number of small resources.</span></span> <span data-ttu-id="5024b-190">此类 API 可能需要客户端应用程序提交多个请求才能查找它所需的所有数据。</span><span class="sxs-lookup"><span data-stu-id="5024b-190">Such an API may require a client application to submit multiple requests to find all the data that it requires.</span></span> <span data-ttu-id="5024b-191">最好使数据非规范化，将相关信息组合在一起成为可通过发出单个请求检索的较大资源。</span><span class="sxs-lookup"><span data-stu-id="5024b-191">It may be beneficial to denormalize data and combine related information together into bigger resources that can be retrieved by issuing a single request.</span></span> <span data-ttu-id="5024b-192">但是，需要根据客户端可能并不经常需要执行的提取数据的开销来权衡此方法。</span><span class="sxs-lookup"><span data-stu-id="5024b-192">However, you need to balance this approach against the overhead of fetching data that might not be frequently required by the client.</span></span> <span data-ttu-id="5024b-193">如果附加的数据不经常使用，检索大型对象可能增加请求的延迟时间并产生额外的带宽成本，相对而言优势则较小。</span><span class="sxs-lookup"><span data-stu-id="5024b-193">Retrieving large objects can increase the latency of a request and incur additional bandwidth costs for little advantage if the additional data is not often used.</span></span>

<span data-ttu-id="5024b-194">避免将 Web API 之间的依赖关系引入到基础数据源的结构、类型或位置。</span><span class="sxs-lookup"><span data-stu-id="5024b-194">Avoid introducing dependencies between the web API to the structure, type, or location of the underlying data sources.</span></span> <span data-ttu-id="5024b-195">例如，如果数据位于关系数据库中，则 Web API 不需要将每个表公开为资源的集合。</span><span class="sxs-lookup"><span data-stu-id="5024b-195">For example, if your data is located in a relational database, the web API does not need to expose each table as a collection of resources.</span></span> <span data-ttu-id="5024b-196">将 Web API 视为数据库的抽象，如有必要引入数据库和 Web API 之间的映射层。</span><span class="sxs-lookup"><span data-stu-id="5024b-196">Think of the web API as an abstraction of the database, and if necessary introduce a mapping layer between the database and the web API.</span></span> <span data-ttu-id="5024b-197">这样，如果数据库的设计或实现发生更改（例如，从包含规范化表集合的关系数据库移到文档数据库等非规范化 NoSQL 存储系统），客户端应用程序将不会受这些更改影响。</span><span class="sxs-lookup"><span data-stu-id="5024b-197">In this way, if the design or implementation of the database changes (for example, you move from a relational database containing a collection of normalized tables to a denormalized NoSQL storage system such as a document database) client applications are insulated from these changes.</span></span>

> [!TIP]
> <span data-ttu-id="5024b-198">支持 Web API 的数据源不必是数据存储；它可以是另一个服务或业务线应用程序，甚至可以是在组织内本地运行的旧版应用程序。</span><span class="sxs-lookup"><span data-stu-id="5024b-198">The source of the data that underpins a web API does not have to be a data store; it could be another service or line-of-business application or even a legacy application running on-premises within an organization.</span></span>
>
>

<span data-ttu-id="5024b-199">最后，可能无法将 Web API 实现的每个操作都映射到特定资源。</span><span class="sxs-lookup"><span data-stu-id="5024b-199">Finally, it might not be possible to map every operation implemented by a web API to a specific resource.</span></span> <span data-ttu-id="5024b-200">可以通过 HTTP GET 请求处理此类*非资源*方案，这些请求调用一项功能并将结果作为 HTTP 响应消息返回。</span><span class="sxs-lookup"><span data-stu-id="5024b-200">You can handle such *non-resource* scenarios through HTTP GET requests that invoke a piece of functionality and return the results as an HTTP response message.</span></span> <span data-ttu-id="5024b-201">实现简单的计算器样式操作（例如，加法和减法）的 Web API 可以提供公开这些操作作为伪资源的 URI，并利用查询字符串来指定所需的参数。</span><span class="sxs-lookup"><span data-stu-id="5024b-201">A web API that implements simple calculator-style operations such as add and subtract could provide URIs that expose these operations as pseudo resources and utilize the query string to specify the parameters required.</span></span> <span data-ttu-id="5024b-202">例如，向 URI */add?operand1=99&operand2=1* 发出 GET 请求会返回正文包含值 100 的响应消息，而向 URI */subtract?operand1=50&operand2=20* 发出 GET 请求则会返回正文包含值 30 的响应消息。</span><span class="sxs-lookup"><span data-stu-id="5024b-202">For example a GET request to the URI */add?operand1=99&operand2=1* could return a response message with the body containing the value 100, and GET request to the URI */subtract?operand1=50&operand2=20* could return a response message with the body containing the value 30.</span></span> <span data-ttu-id="5024b-203">但是，请尽量少使用这些形式的 URI。</span><span class="sxs-lookup"><span data-stu-id="5024b-203">However, only use these forms of URIs sparingly.</span></span>

### <a name="defining-operations-in-terms-of-http-methods"></a><span data-ttu-id="5024b-204">根据 HTTP 方法定义操作</span><span class="sxs-lookup"><span data-stu-id="5024b-204">Defining operations in terms of HTTP methods</span></span>
<span data-ttu-id="5024b-205">HTTP 协议定义了大量为请求赋于语义含义的方法。</span><span class="sxs-lookup"><span data-stu-id="5024b-205">The HTTP protocol defines a number of methods that assign semantic meaning to a request.</span></span> <span data-ttu-id="5024b-206">大多数 RESTful Web API 使用的常见 HTTP 方法是：</span><span class="sxs-lookup"><span data-stu-id="5024b-206">The common HTTP methods used by most RESTful web APIs are:</span></span>

* <span data-ttu-id="5024b-207">**GET**，用于检索指定 URI 处的资源的副本。</span><span class="sxs-lookup"><span data-stu-id="5024b-207">**GET**, to retrieve a copy of the resource at the specified URI.</span></span> <span data-ttu-id="5024b-208">响应消息的正文包含所请求资源的详细信息。</span><span class="sxs-lookup"><span data-stu-id="5024b-208">The body of the response message contains the details of the requested resource.</span></span>
* <span data-ttu-id="5024b-209">**POST**，用于在指定 URI 处创建新资源。</span><span class="sxs-lookup"><span data-stu-id="5024b-209">**POST**, to create a new resource at the specified URI.</span></span> <span data-ttu-id="5024b-210">请求消息的正文将提供新资源的详细信息。</span><span class="sxs-lookup"><span data-stu-id="5024b-210">The body of the request message provides the details of the new resource.</span></span> <span data-ttu-id="5024b-211">请注意，POST 还用于触发不实际创建资源的操作。</span><span class="sxs-lookup"><span data-stu-id="5024b-211">Note that POST can also be used to trigger operations that don't actually create resources.</span></span>
* <span data-ttu-id="5024b-212">**PUT**，用于替换或更新指定 URI 处的资源。</span><span class="sxs-lookup"><span data-stu-id="5024b-212">**PUT**, to replace or update the resource at the specified URI.</span></span> <span data-ttu-id="5024b-213">请求消息的正文指定要修改的资源和要应用的值。</span><span class="sxs-lookup"><span data-stu-id="5024b-213">The body of the request message specifies the resource to be modified and the values to be applied.</span></span>
* <span data-ttu-id="5024b-214">**DELETE**，用于删除指定 URI 处的资源。</span><span class="sxs-lookup"><span data-stu-id="5024b-214">**DELETE**, to remove the resource at the specified URI.</span></span>

> [!NOTE]
> <span data-ttu-id="5024b-215">HTTP 协议还定义了其他不太常用的方法，如 PATCH（用于请求对资源进行选择性更新）、HEAD（用于请求资源的说明）、OPTIONS（使客户端可以获取有关服务器支持的通信选项的信息）和 TRACE（允许客户端请求可用于测试和诊断目的的信息）。</span><span class="sxs-lookup"><span data-stu-id="5024b-215">The HTTP protocol also defines other less commonly-used methods, such as PATCH which is used to request selective updates to a resource, HEAD which is used to request a description of a resource, OPTIONS which enables a client information to obtain information about the communication options supported by the server, and TRACE which allows a client to request information that it can use for testing and diagnostics purposes.</span></span>
>
>

<span data-ttu-id="5024b-216">特定请求的作用应取决于它所应用于的资源是集合还是单个项目。</span><span class="sxs-lookup"><span data-stu-id="5024b-216">The effect of a specific request should depend on whether the resource to which it is applied is a collection or an individual item.</span></span> <span data-ttu-id="5024b-217">下表汇总了使用电子商务示例的大多数 RESTful 实现所采用的常见约定。</span><span class="sxs-lookup"><span data-stu-id="5024b-217">The following table summarizes the common conventions adopted by most RESTful implementations using the ecommerce example.</span></span> <span data-ttu-id="5024b-218">请注意，可能并非实现所有这些请求；这取决于特定方案。</span><span class="sxs-lookup"><span data-stu-id="5024b-218">Note that not all of these requests might be implemented; it depends on the specific scenario.</span></span>

| <span data-ttu-id="5024b-219">**资源**</span><span class="sxs-lookup"><span data-stu-id="5024b-219">**Resource**</span></span> | <span data-ttu-id="5024b-220">**POST**</span><span class="sxs-lookup"><span data-stu-id="5024b-220">**POST**</span></span> | <span data-ttu-id="5024b-221">**GET**</span><span class="sxs-lookup"><span data-stu-id="5024b-221">**GET**</span></span> | <span data-ttu-id="5024b-222">**PUT**</span><span class="sxs-lookup"><span data-stu-id="5024b-222">**PUT**</span></span> | <span data-ttu-id="5024b-223">**DELETE**</span><span class="sxs-lookup"><span data-stu-id="5024b-223">**DELETE**</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="5024b-224">/customers</span><span class="sxs-lookup"><span data-stu-id="5024b-224">/customers</span></span> |<span data-ttu-id="5024b-225">创建新客户</span><span class="sxs-lookup"><span data-stu-id="5024b-225">Create a new customer</span></span> |<span data-ttu-id="5024b-226">检索所有客户</span><span class="sxs-lookup"><span data-stu-id="5024b-226">Retrieve all customers</span></span> |<span data-ttu-id="5024b-227">批量更新客户（*如果已实现*）</span><span class="sxs-lookup"><span data-stu-id="5024b-227">Bulk update of customers (*if implemented*)</span></span> |<span data-ttu-id="5024b-228">删除所有客户</span><span class="sxs-lookup"><span data-stu-id="5024b-228">Remove all customers</span></span> |
| <span data-ttu-id="5024b-229">/customers/1</span><span class="sxs-lookup"><span data-stu-id="5024b-229">/customers/1</span></span> |<span data-ttu-id="5024b-230">错误</span><span class="sxs-lookup"><span data-stu-id="5024b-230">Error</span></span> |<span data-ttu-id="5024b-231">检索客户 1 的详细信息</span><span class="sxs-lookup"><span data-stu-id="5024b-231">Retrieve the details for customer 1</span></span> |<span data-ttu-id="5024b-232">如果客户 1 存在，更新客户 1 的详细信息，否则返回错误</span><span class="sxs-lookup"><span data-stu-id="5024b-232">Update the details of customer 1 if it exists, otherwise return an error</span></span> |<span data-ttu-id="5024b-233">删除客户 1</span><span class="sxs-lookup"><span data-stu-id="5024b-233">Remove customer 1</span></span> |
| <span data-ttu-id="5024b-234">/customers/1/orders</span><span class="sxs-lookup"><span data-stu-id="5024b-234">/customers/1/orders</span></span> |<span data-ttu-id="5024b-235">创建客户 1 的新订单</span><span class="sxs-lookup"><span data-stu-id="5024b-235">Create a new order for customer 1</span></span> |<span data-ttu-id="5024b-236">检索客户 1 的所有订单</span><span class="sxs-lookup"><span data-stu-id="5024b-236">Retrieve all orders for customer 1</span></span> |<span data-ttu-id="5024b-237">批量更新客户 1 的订单（*如果已实现*）</span><span class="sxs-lookup"><span data-stu-id="5024b-237">Bulk update of orders for customer 1 (*if implemented*)</span></span> |<span data-ttu-id="5024b-238">删除客户 1 的所有订单（*如果已实现*）</span><span class="sxs-lookup"><span data-stu-id="5024b-238">Remove all orders for customer 1(*if implemented*)</span></span> |

<span data-ttu-id="5024b-239">GET 和 DELETE 请求的目的相对简单，但就 POST 和 PUT 请求的的目的和作用而言，存在混淆区域。</span><span class="sxs-lookup"><span data-stu-id="5024b-239">The purpose of GET and DELETE requests are relatively straightforward, but there is scope for confusion concerning the purpose and effects of POST and PUT requests.</span></span>

<span data-ttu-id="5024b-240">POST 请求应使用请求正文中提供的数据创建新资源。</span><span class="sxs-lookup"><span data-stu-id="5024b-240">A POST request should create a new resource with data provided in the body of the request.</span></span> <span data-ttu-id="5024b-241">在 REST 模型中，会经常将 POST 请求应用于作为集合的资源；将新资源添加到集合中。</span><span class="sxs-lookup"><span data-stu-id="5024b-241">In the REST model, you frequently apply POST requests to resources that are collections; the new resource is added to the collection.</span></span>

> [!NOTE]
> <span data-ttu-id="5024b-242">还可以定义触发某种功能（以及不必返回数据）的 POST 请求，这些类型的请求可应用于集合。</span><span class="sxs-lookup"><span data-stu-id="5024b-242">You can also define POST requests that trigger some functionality (and that don't necessarily return data), and these types of request can be applied to collections.</span></span> <span data-ttu-id="5024b-243">例如，可以使用 POST 请求将工时单传递给工资单处理服务并返回计算的税款作为响应。</span><span class="sxs-lookup"><span data-stu-id="5024b-243">For example you could use a POST request to pass a timesheet to a payroll processing service and get the calculated taxes back as a response.</span></span>
>
>

<span data-ttu-id="5024b-244">PUT 请求用于修改现有资源。</span><span class="sxs-lookup"><span data-stu-id="5024b-244">A PUT request is intended to modify an existing resource.</span></span> <span data-ttu-id="5024b-245">如果指定的资源不存在，PUT 请求可能返回错误（在某些情况下，它可能会实际创建该资源）。</span><span class="sxs-lookup"><span data-stu-id="5024b-245">If the specified resource does not exist, the PUT request could return an error (in some cases, it might actually create the resource).</span></span> <span data-ttu-id="5024b-246">PUT 请求最常应用于作为单个项目的资源（例如，特定客户或订单），虽然它们可以应用于集合，但这不太常实现。</span><span class="sxs-lookup"><span data-stu-id="5024b-246">PUT requests are most frequently applied to resources that are individual items (such as a specific customer or order), although they can be applied to collections, although this is less-commonly implemented.</span></span> <span data-ttu-id="5024b-247">请注意，PUT 请求是幂等的，而 POST 请求则不是；如果应用程序多次提交同一 PUT 请求，则结果应始终是相同的（将使用相同的值修改同一资源），但如果应用程序重复同一 POST 请求，则结果将是创建多个资源。</span><span class="sxs-lookup"><span data-stu-id="5024b-247">Note that PUT requests are idempotent whereas POST requests are not; if an application submits the same PUT request multiple times the results should always be the same (the same resource will be modified with the same values), but if an application repeats the same POST request the result will be the creation of multiple resources.</span></span>

> [!NOTE]
> <span data-ttu-id="5024b-248">严格地说，HTTP PUT 请求会将现有资源替换为请求正文中指定的资源。</span><span class="sxs-lookup"><span data-stu-id="5024b-248">Strictly speaking, an HTTP PUT request replaces an existing resource with the resource specified in the body of the request.</span></span> <span data-ttu-id="5024b-249">如果目的是要修改资源的所选属性但保留其他属性保持不变，则应使用 HTTP PATCH 请求实现此目的。</span><span class="sxs-lookup"><span data-stu-id="5024b-249">If the intention is to modify a selection of properties in a resource but leave other properties unchanged, then this should be implemented by using an HTTP PATCH request.</span></span> <span data-ttu-id="5024b-250">但是，许多 RESTful 实现放宽了此规则并对这两种情况都使用 PUT。</span><span class="sxs-lookup"><span data-stu-id="5024b-250">However, many RESTful implementations relax this rule and use PUT for both situations.</span></span>
>
>

### <a name="processing-http-requests"></a><span data-ttu-id="5024b-251">处理 HTTP 请求</span><span class="sxs-lookup"><span data-stu-id="5024b-251">Processing HTTP requests</span></span>
<span data-ttu-id="5024b-252">许多 HTTP 请求中客户端应用程序提供的数据和 Web 服务器返回的相应响应消息可以各种格式（或媒体类型）提供。</span><span class="sxs-lookup"><span data-stu-id="5024b-252">The data included by a client application in many HTTP requests, and the corresponding response messages from the web server, could be presented in a variety of formats (or media types).</span></span> <span data-ttu-id="5024b-253">例如，用于指定客户或订单的详细信息的数据可以 XML、JSON 或其他其种编码和压缩格式提供。</span><span class="sxs-lookup"><span data-stu-id="5024b-253">For example, the data that specifies the details for a customer or order could be provided as XML, JSON, or some other encoded and compressed format.</span></span> <span data-ttu-id="5024b-254">RESTful Web API 应支持提交请求的客户端应用程序请求的多种不同的媒体类型。</span><span class="sxs-lookup"><span data-stu-id="5024b-254">A RESTful web API should support different media types as requested by the client application that submits a request.</span></span>

<span data-ttu-id="5024b-255">当客户端应用程序发送需要在消息正文中返回数据的请求时，它可以在请求的 Accept 标头中指定它可以处理的媒体类型。</span><span class="sxs-lookup"><span data-stu-id="5024b-255">When a client application sends a request that returns data in the body of a message, it can specify the media types it can handle in the Accept header of the request.</span></span> <span data-ttu-id="5024b-256">下面的代码说明了一个 HTTP GET 请求，该请求检索订单 2 的详细信息并请求以 JSON 格式返回结果（客户端仍应检查响应中的数据的媒体类型以验证返回的数据格式）：</span><span class="sxs-lookup"><span data-stu-id="5024b-256">The following code illustrates an HTTP GET request that retrieves the details of order 2 and requests the result to be returned as JSON (the client should still examine the media type of the data in the response to verify the format of the data returned):</span></span>

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
...
Accept: application/json
...
```

<span data-ttu-id="5024b-257">如果 Web 服务器支持此媒体类型，则可以使用包含 Content-Type 标头（指定消息正文中的数据的格式）的响应来应答：</span><span class="sxs-lookup"><span data-stu-id="5024b-257">If the web server supports this media type, it can reply with a response that includes Content-Type header that specifies the format of the data in the body of the message:</span></span>

> [!NOTE]
> <span data-ttu-id="5024b-258">为了最大限度地提高互操作性，Accept 和 Content-type 标头中引用的媒体类型应为公认的 MIME 类型而不是某种自定义媒体类型。</span><span class="sxs-lookup"><span data-stu-id="5024b-258">For maximum interoperability, the media types referenced in the Accept and Content-Type headers should be recognized MIME types rather than some custom media type.</span></span>
>
>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
```

<span data-ttu-id="5024b-259">如果 Web 服务器不支持所请求的媒体类型，它可以使用其他格式发送数据。</span><span class="sxs-lookup"><span data-stu-id="5024b-259">If the web server does not support the requested media type, it can send the data in a different format.</span></span> <span data-ttu-id="5024b-260">在所有情况下，它都必须在 Content-Type 标头中指定媒体类型（如 *application/json*）。</span><span class="sxs-lookup"><span data-stu-id="5024b-260">IN all cases it must specify the media type (such as *application/json*) in the Content-Type header.</span></span> <span data-ttu-id="5024b-261">客户端应用程序负责分析响应消息并相应地解释消息正文中的结果。</span><span class="sxs-lookup"><span data-stu-id="5024b-261">It is the responsibility of the client application to parse the response message and interpret the results in the message body appropriately.</span></span>

<span data-ttu-id="5024b-262">请注意，在此示例中，Web 服务器成功检索所请求的数据，并通过在响应标头中传递回状态代码 200 来指示成功。</span><span class="sxs-lookup"><span data-stu-id="5024b-262">Note that in this example, the web server successfully retrieves the requested data and indicates success by passing back a status code of 200 in the response header.</span></span> <span data-ttu-id="5024b-263">如果未找到任何匹配的数据，它应改为返回状态代码 404（找不到），响应消息的正文可以包含附加信息。</span><span class="sxs-lookup"><span data-stu-id="5024b-263">If no matching data is found, it should instead return a status code of 404 (not found) and the body of the response message can contain additional information.</span></span> <span data-ttu-id="5024b-264">此信息的格式由 Content-Type 标头指定，如下面的示例中所示：</span><span class="sxs-lookup"><span data-stu-id="5024b-264">The format of this information is specified by the Content-Type header, as shown in the following example:</span></span>

```HTTP
GET http://adventure-works.com/orders/222 HTTP/1.1
...
Accept: application/json
...
```

<span data-ttu-id="5024b-265">订单 222 不存在，因此响应消息如下所示：</span><span class="sxs-lookup"><span data-stu-id="5024b-265">Order 222 does not exist, so the response message looks like this:</span></span>

```HTTP
HTTP/1.1 404 Not Found
...
Content-Type: application/json; charset=utf-8
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"message":"No such order"}
```

<span data-ttu-id="5024b-266">当应用程序发送 HTTP PUT 请求以更新资源时，它将指定资源的 URI 并在请求消息的正文中提供要修改的数据。</span><span class="sxs-lookup"><span data-stu-id="5024b-266">When an application sends an HTTP PUT request to update a resource, it specifies the URI of the resource and provides the data to be modified in the body of the request message.</span></span> <span data-ttu-id="5024b-267">它还应使用 Content-Type 标头指定此数据的格式。</span><span class="sxs-lookup"><span data-stu-id="5024b-267">It should also specify the format of this data by using the Content-Type header.</span></span> <span data-ttu-id="5024b-268">用于基于文本的信息的常见格式是 *application/x-www-form-urlencoded*，它由用 & 字符分隔的一组名称/值对组成。</span><span class="sxs-lookup"><span data-stu-id="5024b-268">A common format used for text-based information is *application/x-www-form-urlencoded*, which comprises a set of name/value pairs separated by the & character.</span></span> <span data-ttu-id="5024b-269">下一个示例演示了用于修改订单 1 中的信息的 HTTP PUT 请求：</span><span class="sxs-lookup"><span data-stu-id="5024b-269">The next example shows an HTTP PUT request that modifies the information in order 1:</span></span>

```HTTP
PUT http://adventure-works.com/orders/1 HTTP/1.1
...
Content-Type: application/x-www-form-urlencoded
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
ProductID=3&Quantity=5&OrderValue=250
```

<span data-ttu-id="5024b-270">如果修改成功，在理想情况下它应以 HTTP 204 状态代码响应，以指示该过程已成功处理，但响应正文未包含任何进一步的信息。</span><span class="sxs-lookup"><span data-stu-id="5024b-270">If the modification is successful, it should ideally respond with an HTTP 204 status code, indicating that the process has been successfully handled, but that the response body contains no further information.</span></span> <span data-ttu-id="5024b-271">响应中的 Location 标头包含新更新的资源的 URI：</span><span class="sxs-lookup"><span data-stu-id="5024b-271">The Location header in the response contains the URI of the newly updated resource:</span></span>

```HTTP
HTTP/1.1 204 No Content
...
Location: http://adventure-works.com/orders/1
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
```

> [!TIP]
> <span data-ttu-id="5024b-272">如果 HTTP PUT 请求消息中的数据包括日期和时间信息，请确保 Web 服务接受按 ISO 8601 标准设置格式的日期和时间。</span><span class="sxs-lookup"><span data-stu-id="5024b-272">If the data in an HTTP PUT request message includes date and time information, make sure that your web service accepts dates and times formatted following the ISO 8601 standard.</span></span>
>
>

<span data-ttu-id="5024b-273">如果要更新的资源不存在，如前所述，Web 服务器可以使用“未找到”响应进行响应。</span><span class="sxs-lookup"><span data-stu-id="5024b-273">If the resource to be updated does not exist, the web server can respond with a Not Found response as described earlier.</span></span> <span data-ttu-id="5024b-274">或者，如果服务器实际创建了该对象本身，则可以返回状态代码 HTTP 200（正常）或 HTTP 201（已创建），响应正文可以包含新资源的数据。</span><span class="sxs-lookup"><span data-stu-id="5024b-274">Alternatively, if the server actually creates the object itself it could return the status codes HTTP 200 (OK) or HTTP 201 (Created) and the response body could contain the data for the new resource.</span></span> <span data-ttu-id="5024b-275">如果请求的 Content-Type 标头指定了 Web 服务器无法处理的数据格式，则它应以 HTTP 状态代码 415（不支持的媒体类型）进行响应。</span><span class="sxs-lookup"><span data-stu-id="5024b-275">If the Content-Type header of the request specifies a data format that the web server cannot handle, it should respond with HTTP status code 415 (Unsupported Media Type).</span></span>

> [!TIP]
> <span data-ttu-id="5024b-276">请考虑实现可批量更新集合中的多个资源的批量 HTTP PUT 操作。</span><span class="sxs-lookup"><span data-stu-id="5024b-276">Consider implementing bulk HTTP PUT operations that can batch updates to multiple resources in a collection.</span></span> <span data-ttu-id="5024b-277">PUT 请求应指定集合的 URI，而请求正文则应指定要修改的资源的详细信息。</span><span class="sxs-lookup"><span data-stu-id="5024b-277">The PUT request should specify the URI of the collection, and the request body should specify the details of the resources to be modified.</span></span> <span data-ttu-id="5024b-278">此方法可帮助减少交互成本并提高性能。</span><span class="sxs-lookup"><span data-stu-id="5024b-278">This approach can help to reduce chattiness and improve performance.</span></span>
>
>

<span data-ttu-id="5024b-279">用于创建新资源的 HTTP POST 请求的格式与 PUT 请求的格式类似；消息正文包含要添加的新资源的详细信息。</span><span class="sxs-lookup"><span data-stu-id="5024b-279">The format of an HTTP POST requests that create new resources are similar to those of PUT requests; the message body contains the details of the new resource to be added.</span></span> <span data-ttu-id="5024b-280">但是，URI 通常指定资源应添加到的集合。</span><span class="sxs-lookup"><span data-stu-id="5024b-280">However, the URI typically specifies the collection to which the resource should be added.</span></span> <span data-ttu-id="5024b-281">下面的示例将创建新订单并将其添加到订单集合：</span><span class="sxs-lookup"><span data-stu-id="5024b-281">The following example creates a new order and adds it to the orders collection:</span></span>

```HTTP
POST http://adventure-works.com/orders HTTP/1.1
...
Content-Type: application/x-www-form-urlencoded
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
productID=5&quantity=15&orderValue=400
```

<span data-ttu-id="5024b-282">如果该请求成功，Web 服务器应以包含 HTTP 状态代码 201（已创建）的消息代码进行响应。</span><span class="sxs-lookup"><span data-stu-id="5024b-282">If the request is successful, the web server should respond with a message code with HTTP status code 201 (Created).</span></span> <span data-ttu-id="5024b-283">Location 标头应包含新创建的资源的 URI，响应正文应包含新资源的副本；Content-Type 标头指定此数据的格式：</span><span class="sxs-lookup"><span data-stu-id="5024b-283">The Location header should contain the URI of the newly created resource, and the body of the response should contain a copy of the new resource; the Content-Type header specifies the format of this data:</span></span>

```HTTP
HTTP/1.1 201 Created
...
Content-Type: application/json; charset=utf-8
Location: http://adventure-works.com/orders/99
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"orderID":99,"productID":5,"quantity":15,"orderValue":400}
```

> [!TIP]
> <span data-ttu-id="5024b-284">如果 PUT 或 POST 请求提供的数据无效，Web 服务器应以包含 HTTP 状态代码 400（错误请求）的消息进行响应。</span><span class="sxs-lookup"><span data-stu-id="5024b-284">If the data provided by a PUT or POST request is invalid, the web server should respond with a message with HTTP status code 400 (Bad Request).</span></span> <span data-ttu-id="5024b-285">此消息的正文可以包含有关请求存在的问题的其他信息以及所需的格式，也可以包含指向提供更多详细信息的 URL 的链接。</span><span class="sxs-lookup"><span data-stu-id="5024b-285">The body of this message can contain additional information about the problem with the request and the formats expected, or it can contain a link to a URL that provides more details.</span></span>
>
>

<span data-ttu-id="5024b-286">若要删除资源，HTTP DELETE 请求只需提供要删除的资源的 URI。</span><span class="sxs-lookup"><span data-stu-id="5024b-286">To remove a resource, an HTTP DELETE request simply provides the URI of the resource to be deleted.</span></span> <span data-ttu-id="5024b-287">下面的示例将尝试删除订单 99：</span><span class="sxs-lookup"><span data-stu-id="5024b-287">The following example attempts to remove order 99:</span></span>

```HTTP
DELETE http://adventure-works.com/orders/99 HTTP/1.1
...
```

<span data-ttu-id="5024b-288">如果删除操作成功，Web 服务器应以 HTTP 状态代码 204 进行响应，以指示该过程已成功处理，但响应正文未包含任何进一步的信息（这与成功的 PUT 操作返回的响应相同，但没有 Location 标头，因为资源不再存在）。如果异步执行删除操作，DELETE 请求还有可能返回 HTTP 状态代码 200（正常）或 202（已接受）。</span><span class="sxs-lookup"><span data-stu-id="5024b-288">If the delete operation is successful, the web server should respond with HTTP status code 204, indicating that the process has been successfully handled, but that the response body contains no further information (this is the same response returned by a successful PUT operation, but without a Location header as the resource no longer exists.) It is also possible for a DELETE request to return HTTP status code 200 (OK) or 202 (Accepted) if the deletion is performed asynchronously.</span></span>

```HTTP
HTTP/1.1 204 No Content
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
```

<span data-ttu-id="5024b-289">如果未找到资源，Web 服务器应改为返回 404（未找到）消息。</span><span class="sxs-lookup"><span data-stu-id="5024b-289">If the resource is not found, the web server should return a 404 (Not Found) message instead.</span></span>

> [!TIP]
> <span data-ttu-id="5024b-290">如果需要删除集合中的所有资源，请为 HTTP DELETE 请求指定集合的 URI，而不是强制应用程序从集合中依次删除每个资源。</span><span class="sxs-lookup"><span data-stu-id="5024b-290">If all the resources in a collection need to be deleted, enable an HTTP DELETE request to be specified for the URI of the collection rather than forcing an application to remove each resource in turn from the collection.</span></span>
>
>

### <a name="filtering-and-paginating-data"></a><span data-ttu-id="5024b-291">对数据进行筛选和分页</span><span class="sxs-lookup"><span data-stu-id="5024b-291">Filtering and paginating data</span></span>
<span data-ttu-id="5024b-292">应尽力使 URI 保持简单和直观。</span><span class="sxs-lookup"><span data-stu-id="5024b-292">You should endeavor to keep the URIs simple and intuitive.</span></span> <span data-ttu-id="5024b-293">通过单个 URI 公开资源的集合可在此方面有帮助，但这可能会导致应用程序在只需信息的子集时提取大量数据。</span><span class="sxs-lookup"><span data-stu-id="5024b-293">Exposing a collection of resources through a single URI assists in this respect, but it can lead to applications fetching large amounts of data when only a subset of the information is required.</span></span> <span data-ttu-id="5024b-294">生成大量流量不仅会影响 Web 服务器的性能和可伸缩性，而且会反过来影响请求数据的客户端应用程序的响应能力。</span><span class="sxs-lookup"><span data-stu-id="5024b-294">Generating a large volume of traffic impacts not only the performance and scalability of the web server but also adversely affect the responsiveness of client applications requesting the data.</span></span>

<span data-ttu-id="5024b-295">例如，如果订单包含为订单支付的价格，需要检索成本高于特定值的所有订单的客户端应用程序可能需要从 */orders* URI 检索所有订单，并在本地筛选出这些订单。</span><span class="sxs-lookup"><span data-stu-id="5024b-295">For example, if orders contain the price paid for the order, a client application that needs to retrieve all orders that have a cost over a specific value might need to retrieve all orders from the */orders* URI and then filter these orders locally.</span></span> <span data-ttu-id="5024b-296">此过程显然效率非常低；它浪费了托管 Web API 的服务器的网络带宽和处理能力。</span><span class="sxs-lookup"><span data-stu-id="5024b-296">Clearly this process is highly inefficient; it wastes network bandwidth and processing power on the server hosting the web API.</span></span>

<span data-ttu-id="5024b-297">一种解决方案是提供 URI 方案，例如 /orders/ordervalue_greater_than_n，其中 n 是订单价格，但对于所有订单而言只是有限数量的价格，这种方法不切实际。</span><span class="sxs-lookup"><span data-stu-id="5024b-297">One solution may be to provide a URI scheme such as */orders/ordervalue_greater_than_n* where *n* is the order price, but for all but a limited number of prices such an approach is impractical.</span></span> <span data-ttu-id="5024b-298">此外，如果需要基于其他条件查询订单，可能最终面临着需要提供使用可能非直观名称的 URI 的长列表。</span><span class="sxs-lookup"><span data-stu-id="5024b-298">Additionally, if you need to query orders based on other criteria, you can end up being faced with providing with a long list of URIs with possibly non-intuitive names.</span></span>

<span data-ttu-id="5024b-299">筛选数据的较好策略是在传递到 Web API 的查询字符串中提供筛选条件，例如 */orders?ordervaluethreshold=n*。</span><span class="sxs-lookup"><span data-stu-id="5024b-299">A better strategy to filtering data is to provide the filter criteria in the query string that is passed to the web API, such as */orders?ordervaluethreshold=n*.</span></span> <span data-ttu-id="5024b-300">在此示例中，Web API 中的相应操作负责分析和处理查询字符串中的 `ordervaluethreshold` 参数并在 HTTP 响应中返回筛选结果。</span><span class="sxs-lookup"><span data-stu-id="5024b-300">In this example, the corresponding operation in the web API is responsible for parsing and handling the `ordervaluethreshold` parameter in the query string and returning the filtered results in the HTTP response.</span></span>

<span data-ttu-id="5024b-301">对集合资源执行的一些简单 HTTP GET 请求可能会返回大量项目。</span><span class="sxs-lookup"><span data-stu-id="5024b-301">Some simple HTTP GET requests over collection resources could potentially return a large number of items.</span></span> <span data-ttu-id="5024b-302">要防止发生这种现象，应将 Web API 设计为限制任何单个请求返回的数据量。</span><span class="sxs-lookup"><span data-stu-id="5024b-302">To combat the possibility of this occurring you should design the web API to limit the amount of data returned by any single request.</span></span> <span data-ttu-id="5024b-303">可以通过支持这样的查询字符串来达到此目的：允许用户指定要检索的最大项目数（这样它本身就可以受到上限约束有助于防止拒绝服务攻击）和集合中的起始偏移量。</span><span class="sxs-lookup"><span data-stu-id="5024b-303">You can achieve this by supporting query strings that enable the user to specify the maximum number of items to be retrieved (which could itself be subject to an upperbound limit to help prevent Denial of Service attacks), and a starting offset into the collection.</span></span> <span data-ttu-id="5024b-304">例如，URI */orders?limit=25&offset=50* 中的查询字符串应检索从订单集合中找到的第 50 个订单开始的 25 个订单。</span><span class="sxs-lookup"><span data-stu-id="5024b-304">For example, the query string in the URI */orders?limit=25&offset=50* should retrieve 25 orders starting with the 50th order found in the orders collection.</span></span> <span data-ttu-id="5024b-305">与筛选数据一样，在 Web API 中实现 GET 请求的操作负责分析和处理查询字符串中的 `limit` 和 `offset` 参数。</span><span class="sxs-lookup"><span data-stu-id="5024b-305">As with filtering data, the operation that implements the GET request in the web API is responsible for parsing and handling the `limit` and `offset` parameters in the query string.</span></span> <span data-ttu-id="5024b-306">若要帮助客户端应用程序，返回分页数据的 GET 请求还应包含某种形式的元数据，以指示集合中可用的资源总数。</span><span class="sxs-lookup"><span data-stu-id="5024b-306">To assist client applications, GET requests that return paginated data should also include some form of metadata that indicate the total number of resources available in the collection.</span></span> <span data-ttu-id="5024b-307">也许还可以考虑其他智能分页策略；有关详细信息，请参阅 [API Design Notes: Smart Paging](http://bizcoder.com/api-design-notes-smart-paging)（API 设计说明：智能分页）</span><span class="sxs-lookup"><span data-stu-id="5024b-307">You might also consider other intelligent paging strategies; for more information, see [API Design Notes: Smart Paging](http://bizcoder.com/api-design-notes-smart-paging)</span></span>

<span data-ttu-id="5024b-308">在提取数据时可以执行类似的策略对数据进行排序；可以提供一个接受字段名称作为值的排序参数，例如 */orders?sort=ProductID*。</span><span class="sxs-lookup"><span data-stu-id="5024b-308">You can follow a similar strategy for sorting data as it is fetched; you could provide a sort parameter that takes a field name as the value, such as */orders?sort=ProductID*.</span></span> <span data-ttu-id="5024b-309">但请注意，这种方法会对缓存产生负面影响（查询字符串参数构成许多缓存实现用作缓存数据的键的资源标识符的一部分）。</span><span class="sxs-lookup"><span data-stu-id="5024b-309">However, note that this approach can have a deleterious effect on caching (query string parameters form part of the resource identifier used by many cache implementations as the key to cached data).</span></span>

<span data-ttu-id="5024b-310">如果单个资源项目包含大量数据，可以扩展此方法来限制（投影）返回的字段。</span><span class="sxs-lookup"><span data-stu-id="5024b-310">You can extend this approach to limit (project) the fields returned if a single resource item contains a large amount of data.</span></span> <span data-ttu-id="5024b-311">例如，可以使用接受以逗号分隔的字段列表的查询字符串参数，例如 */orders?fields=ProductID,Quantity*。</span><span class="sxs-lookup"><span data-stu-id="5024b-311">For example, you could use a query string parameter that accepts a comma-delimited list of fields, such as */orders?fields=ProductID,Quantity*.</span></span>

> [!TIP]
> <span data-ttu-id="5024b-312">为查询字符串中的所有可选参数提供有意义的默认值。</span><span class="sxs-lookup"><span data-stu-id="5024b-312">Give all optional parameters in query strings meaningful defaults.</span></span> <span data-ttu-id="5024b-313">例如，如果实现分页，将 `limit` 参数设为 10，将 `offset` 参数设为 0；如果实现排序，将排序参数设为资源的键；如果支持投影，将 `fields` 参数设为资源中的所有字段。</span><span class="sxs-lookup"><span data-stu-id="5024b-313">For example, set the `limit` parameter to 10 and the `offset` parameter to 0 if you implement pagination, set the sort parameter to the key of the resource if you implement ordering, and set the `fields` parameter to all fields in the resource if you support projections.</span></span>
>
>

### <a name="handling-large-binary-resources"></a><span data-ttu-id="5024b-314">处理大型二进制资源</span><span class="sxs-lookup"><span data-stu-id="5024b-314">Handling large binary resources</span></span>
<span data-ttu-id="5024b-315">单个资源可能包含大型二进制字段，例如文件或图像。</span><span class="sxs-lookup"><span data-stu-id="5024b-315">A single resource may contain large binary fields, such as files or images.</span></span> <span data-ttu-id="5024b-316">若要解决由不可靠和间歇性连接导致的传输问题并缩短响应时间，请考虑提供使客户端应用程序能够分块检索此类资源的操作。</span><span class="sxs-lookup"><span data-stu-id="5024b-316">To overcome the transmission problems caused by unreliable and intermittent connections and to improve response times, consider providing operations that enable such resources to be retrieved in chunks by the client application.</span></span> <span data-ttu-id="5024b-317">为此，Web API 对于针对大型资源的 GET 请求应支持 Accept-Ranges 标头，并完美地实现对这些资源的 HTTP HEAD 请求。</span><span class="sxs-lookup"><span data-stu-id="5024b-317">To do this, the web API should support the Accept-Ranges header for GET requests for large resources, and ideally implement HTTP HEAD requests for these resources.</span></span> <span data-ttu-id="5024b-318">Accept-Ranges 标头指示 GET 操作支持部分结果并且客户端应用程序可以提交返回指定为字节范围的资源子集的 GET 请求。</span><span class="sxs-lookup"><span data-stu-id="5024b-318">The Accept-Ranges header indicates that the GET operation supports partial results, and that a client application can submit GET requests that return a subset of a resource specified as a range of bytes.</span></span> <span data-ttu-id="5024b-319">HEAD 请求与 GET 请求类似，不同的是它仅返回描述资源的标头和空的消息正文。</span><span class="sxs-lookup"><span data-stu-id="5024b-319">A HEAD request is similar to a GET request except that it only returns a header that describes the resource and an empty message body.</span></span> <span data-ttu-id="5024b-320">客户端应用程序可以发出 HEAD 请求以确定是否要通过使用部分 GET 请求获取某个资源。</span><span class="sxs-lookup"><span data-stu-id="5024b-320">A client application can issue a HEAD request to determine whether to fetch a resource by using partial GET requests.</span></span> <span data-ttu-id="5024b-321">下面的示例演示获取有关产品图像的信息的 HEAD 请求：</span><span class="sxs-lookup"><span data-stu-id="5024b-321">The following example shows a HEAD request that obtains information about a product image:</span></span>

```HTTP
HEAD http://adventure-works.com/products/10?fields=productImage HTTP/1.1
...
```

<span data-ttu-id="5024b-322">响应消息包含一个提供资源大小（4580 字节）的标头和 Accept-Ranges 标头以指示相应的 GET 操作支持部分结果：</span><span class="sxs-lookup"><span data-stu-id="5024b-322">The response message contains a header that includes the size of the resource (4580 bytes), and the Accept-Ranges header that the corresponding GET operation supports partial results:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 4580
...
```

<span data-ttu-id="5024b-323">客户端应用程序可以使用此信息构造一系列 GET 操作以检索较小区块中的图像。</span><span class="sxs-lookup"><span data-stu-id="5024b-323">The client application can use this information to construct a series of GET operations to retrieve the image in smaller chunks.</span></span> <span data-ttu-id="5024b-324">第一个请求通过使用 Range 标头提取前 2500 个字节：</span><span class="sxs-lookup"><span data-stu-id="5024b-324">The first request fetches the first 2500 bytes by using the Range header:</span></span>

```HTTP
GET http://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=0-2499
...
```

<span data-ttu-id="5024b-325">响应消息通过返回 HTTP 状态代码 206 指示这是部分响应。</span><span class="sxs-lookup"><span data-stu-id="5024b-325">The response message indicates that this is a partial response by returning HTTP status code 206.</span></span> <span data-ttu-id="5024b-326">Content-Length 标头指定消息正文中返回的实际字节数（不是资源的大小），Content-Range 标头指示这是该资源的哪一部分（第 0 到 2499 字节，总共 4580 个字节）:</span><span class="sxs-lookup"><span data-stu-id="5024b-326">The Content-Length header specifies the actual number of bytes returned in the message body (not the size of the resource), and the Content-Range header indicates which part of the resource this is (bytes 0-2499 out of 4580):</span></span>

```HTTP
HTTP/1.1 206 Partial Content
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2500
Content-Range: bytes 0-2499/4580
...
_{binary data not shown}_
```

<span data-ttu-id="5024b-327">客户端应用程序的后续请求可以通过使用相应的 Range 标头检索资源的其余部分：</span><span class="sxs-lookup"><span data-stu-id="5024b-327">A subsequent request from the client application can retrieve the remainder of the resource by using an appropriate Range header:</span></span>

```HTTP
GET http://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=2500-
...
```

<span data-ttu-id="5024b-328">相应的结果消息应如下所示：</span><span class="sxs-lookup"><span data-stu-id="5024b-328">The corresponding result message should look like this:</span></span>

```HTTP
HTTP/1.1 206 Partial Content
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2080
Content-Range: bytes 2500-4580/4580
...
```

## <a name="using-the-hateoas-approach-to-enable-navigation-to-related-resources"></a><span data-ttu-id="5024b-329">使用 HATEOAS 方法实现导航到相关资源</span><span class="sxs-lookup"><span data-stu-id="5024b-329">Using the HATEOAS approach to enable navigation to related resources</span></span>
<span data-ttu-id="5024b-330">REST 背后的主要动机之一是它应能够导航整个资源集，而无需事先了解 URI 方案。</span><span class="sxs-lookup"><span data-stu-id="5024b-330">One of the primary motivations behind REST is that it should be possible to navigate the entire set of resources without requiring prior knowledge of the URI scheme.</span></span> <span data-ttu-id="5024b-331">每个 HTTP GET 请求应通过响应中包含的超链接返回查找与所请求的对象直接相关的资源所需的信息，还应为它提供描述其中每个资源提供的操作的信息。</span><span class="sxs-lookup"><span data-stu-id="5024b-331">Each HTTP GET request should return the information necessary to find the resources related directly to the requested object through hyperlinks included in the response, and it should also be provided with information that describes the operations available on each of these resources.</span></span> <span data-ttu-id="5024b-332">此原则称为 HATEOAS 或作为应用程序状态引擎的超文本。</span><span class="sxs-lookup"><span data-stu-id="5024b-332">This principle is known as HATEOAS, or Hypertext as the Engine of Application State.</span></span> <span data-ttu-id="5024b-333">该系统实际上是有限状态机，每个请求的响应包含从一种状态转为另一种状态所需的信息；任何其他信息都不应是必需的。</span><span class="sxs-lookup"><span data-stu-id="5024b-333">The system is effectively a finite state machine, and the response to each request contains the information necessary to move from one state to another; no other information should be necessary.</span></span>

> [!NOTE]
> <span data-ttu-id="5024b-334">当前没有任何标准或规范定义如何为 HATEOAS 原则建模。</span><span class="sxs-lookup"><span data-stu-id="5024b-334">Currently there are no standards or specifications that define how to model the HATEOAS principle.</span></span> <span data-ttu-id="5024b-335">此节中所示的示例说明了一个可能的解决方案。</span><span class="sxs-lookup"><span data-stu-id="5024b-335">The examples shown in this section illustrate one possible solution.</span></span>
>
>

<span data-ttu-id="5024b-336">例如，若要处理客户和订单之间的关系，在特定订单的响应中返回的数据应包含超链接形式的 URI，以标识下订单的客户以及可以对该客户执行的操作。</span><span class="sxs-lookup"><span data-stu-id="5024b-336">As an example, to handle the relationship between customers and orders, the data returned in the response for a specific order should contain URIs in the form of a hyperlink identifying the customer that placed the order, and the operations that can be performed on that customer.</span></span>

```HTTP
GET http://adventure-works.com/orders/3 HTTP/1.1
Accept: application/json
...
```

<span data-ttu-id="5024b-337">响应消息的正文包含一个 `links` 数组（在代码示例中突出显示），用于指定关系的性质（*客户*）、客户的 URI (*http://adventure-works.com/customers/3*)、检索此客户的详细信息的方法 (*GET*)，以及 Web 服务器支持检索此信息的 MIME 类型（*text/xml* 和 *application/json*）。</span><span class="sxs-lookup"><span data-stu-id="5024b-337">The body of the response message contains a `links` array (highlighted in the code example) that specifies the nature of the relationship (*Customer*), the URI of the customer (*http://adventure-works.com/customers/3*), how to retrieve the details of this customer (*GET*), and the MIME types that the web server supports for retrieving this information (*text/xml* and *application/json*).</span></span> <span data-ttu-id="5024b-338">这是客户端应用程序要能够提取客户的详细信息所需的所有信息。</span><span class="sxs-lookup"><span data-stu-id="5024b-338">This is all the information that a client application needs to be able to fetch the details of the customer.</span></span> <span data-ttu-id="5024b-339">此外，链接数组还包括其他可以执行的操作的链接，例如 PUT（用于修改客户以及 Web 服务器需要客户端提供的格式）和 DELETE。</span><span class="sxs-lookup"><span data-stu-id="5024b-339">Additionally, the Links array also includes links for the other operations that can be performed, such as PUT (to modify the customer, together with the format that the web server expects the client to provide), and DELETE.</span></span>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"orderID":3,"productID":2,"quantity":4,"orderValue":16.60,"links":[(some links omitted){"rel":"customer","href":" http://adventure-works.com/customers/3", "action":"GET","types":["text/xml","application/json"]},{"rel":"
customer","href":" http://adventure-works.com /customers/3", "action":"PUT","types":["application/x-www-form-urlencoded"]},{"rel":"customer","href":" http://adventure-works.com /customers/3","action":"DELETE","types":[]}]}
```

<span data-ttu-id="5024b-340">出于完整性考虑，链接数组还应包括有关已检索到的资源的自引用信息。</span><span class="sxs-lookup"><span data-stu-id="5024b-340">For completeness, the Links array should also include self-referencing information pertaining to the resource that has been retrieved.</span></span> <span data-ttu-id="5024b-341">这些链接在前一示例中省略了，但在下面的代码中突出显示。</span><span class="sxs-lookup"><span data-stu-id="5024b-341">These links have been omitted from the previous example, but are highlighted in the following code.</span></span> <span data-ttu-id="5024b-342">请注意，在这些链接中，关系 *self* 已用于指示这是对该操作所返回的资源的引用：</span><span class="sxs-lookup"><span data-stu-id="5024b-342">Notice that in these links, the relationship *self* has been used to indicate that this is a reference to the resource being returned by the operation:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"orderID":3,"productID":2,"quantity":4,"orderValue":16.60,"links":[{"rel":"self","href":" http://adventure-works.com/orders/3", "action":"GET","types":["text/xml","application/json"]},{"rel":" self","href":" http://adventure-works.com /orders/3", "action":"PUT","types":["application/x-www-form-urlencoded"]},{"rel":"self","href":" http://adventure-works.com /orders/3", "action":"DELETE","types":[]},{"rel":"customer",
"href":" http://adventure-works.com /customers/3", "action":"GET","types":["text/xml","application/json"]},{"rel":" customer" (customer links omitted)}]}
```

<span data-ttu-id="5024b-343">要使此方法有效，必须客户端应用程序准备好检索和分析此附加信息。</span><span class="sxs-lookup"><span data-stu-id="5024b-343">For this approach to be effective, client applications must be prepared to retrieve and parse this additional information.</span></span>

## <a name="versioning-a-restful-web-api"></a><span data-ttu-id="5024b-344">对 RESTful Web API 进行版本控制</span><span class="sxs-lookup"><span data-stu-id="5024b-344">Versioning a RESTful web API</span></span>
<span data-ttu-id="5024b-345">在除最简单的情况以外的所有情况下，Web API 将不大可能保持不变。</span><span class="sxs-lookup"><span data-stu-id="5024b-345">It is highly unlikely that in all but the simplest of situations that a web API will remain static.</span></span> <span data-ttu-id="5024b-346">随着业务需求变化，可能会添加新的资源集合，资源之间的关系可能会更改，并且可能会修改资源中的数据结构。</span><span class="sxs-lookup"><span data-stu-id="5024b-346">As business requirements change new collections of resources may be added, the relationships between resources might change, and the structure of the data in resources might be amended.</span></span> <span data-ttu-id="5024b-347">虽然更新 Web API 以处理新的或不同的需求是一个相对简单的过程，但你必须考虑此类更改将对使用该 Web API 的客户端应用程序造成的影响。</span><span class="sxs-lookup"><span data-stu-id="5024b-347">While updating a web API to handle new or differing requirements is a relatively straightforward process, you must consider the effects that such changes will have on client applications consuming the web API.</span></span> <span data-ttu-id="5024b-348">问题在于尽管设计和实现 Web API 的开发人员可以完全控制该 API，但开发人员对客户端应用程序不具有相同程度的控制，因为这些客户端应用程序可能是由远程运营的第三方组织生成的。</span><span class="sxs-lookup"><span data-stu-id="5024b-348">The issue is that although the developer designing and implementing a web API has full control over that API, the developer does not have the same degree of control over client applications which may be built by third party organizations operating remotely.</span></span> <span data-ttu-id="5024b-349">主要规则是要让现有客户端应用程序能够继续不变地正常运行，同时允许新客户端应用程序利用新功能和新资源。</span><span class="sxs-lookup"><span data-stu-id="5024b-349">The primary imperative is to enable existing client applications to continue functioning unchanged while allowing new client applications to take advantage of new features and resources.</span></span>

<span data-ttu-id="5024b-350">版本控制使 Web API 可以指定它所公开的功能和资源，并且客户端应用程序可以提交定向到特定版本的功能或资源的请求。</span><span class="sxs-lookup"><span data-stu-id="5024b-350">Versioning enables a web API to indicate the features and resources that it exposes, and a client application can submit requests that are directed to a specific version of a feature or resource.</span></span> <span data-ttu-id="5024b-351">以下各节介绍几种不同的方法，其中每一种方法都有其自己的优势和不足。</span><span class="sxs-lookup"><span data-stu-id="5024b-351">The following sections describe several different approaches, each of which has its own benefits and trade-offs.</span></span>

### <a name="no-versioning"></a><span data-ttu-id="5024b-352">无版本控制</span><span class="sxs-lookup"><span data-stu-id="5024b-352">No versioning</span></span>
<span data-ttu-id="5024b-353">这是最简单的方法，它对于一些内部 API 来说可能是可以接受的。</span><span class="sxs-lookup"><span data-stu-id="5024b-353">This is the simplest approach, and may be acceptable for some internal APIs.</span></span> <span data-ttu-id="5024b-354">较大的更改可以表示为新资源或新链接。</span><span class="sxs-lookup"><span data-stu-id="5024b-354">Big changes could be represented as new resources or new links.</span></span>  <span data-ttu-id="5024b-355">向现有资源添加内容可能未呈现重大更改，因为不应查看此内容的客户端应用程序将直接忽略它。</span><span class="sxs-lookup"><span data-stu-id="5024b-355">Adding content to existing resources might not present a breaking change as client applications that are not expecting to see this content will simply ignore it.</span></span>

<span data-ttu-id="5024b-356">例如，向 URI *http://adventure-works.com/customers/3* 发出请求应返回包含客户端应用程序所需的 `id`、`name` 和 `address` 字段的单个客户的详细信息：</span><span class="sxs-lookup"><span data-stu-id="5024b-356">For example, a request to the URI *http://adventure-works.com/customers/3* should return the details of a single customer containing `id`, `name`, and `address` fields expected by the client application:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

> [!NOTE]
> <span data-ttu-id="5024b-357">为简单清晰起见，本节中所示的示例响应将不包括 HATEOAS 链接。</span><span class="sxs-lookup"><span data-stu-id="5024b-357">For the purposes of simplicity and clarity, the example responses shown in this section do not include HATEOAS links.</span></span>
>
>

<span data-ttu-id="5024b-358">如果 `DateCreated` 字段已添加到客户资源的架构中，则响应将如下所示：</span><span class="sxs-lookup"><span data-stu-id="5024b-358">If the `DateCreated` field is added to the schema of the customer resource, then the response would look like this:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":"1 Microsoft Way Redmond WA 98053"}
```

<span data-ttu-id="5024b-359">现有的客户端应用程序可能会继续正常工作（如果能够忽略无法识别的字段），而新的客户端应用程序则可以设计为处理该新字段。</span><span class="sxs-lookup"><span data-stu-id="5024b-359">Existing client applications might continue functioning correctly if they are capable of ignoring unrecognized fields, while new client applications can be designed to handle this new field.</span></span> <span data-ttu-id="5024b-360">但是，如果对资源的架构进行了更根本的更改（如删除或重命名字段）或资源之间的关系发生更改，则这些更改可能构成重大更改，从而阻止现有客户端应用程序正常工作。</span><span class="sxs-lookup"><span data-stu-id="5024b-360">However, if more radical changes to the schema of resources occur (such as removing or renaming fields) or the relationships between resources change then these may constitute breaking changes that prevent existing client applications from functioning correctly.</span></span> <span data-ttu-id="5024b-361">在这些情况下应考虑以下方法之一。</span><span class="sxs-lookup"><span data-stu-id="5024b-361">In these situations you should consider one of the following approaches.</span></span>

### <a name="uri-versioning"></a><span data-ttu-id="5024b-362">URI 版本控制</span><span class="sxs-lookup"><span data-stu-id="5024b-362">URI versioning</span></span>
<span data-ttu-id="5024b-363">每次修改 Web API 或更改资源的架构时，向每个资源的 URI 添加版本号。</span><span class="sxs-lookup"><span data-stu-id="5024b-363">Each time you modify the web API or change the schema of resources, you add a version number to the URI for each resource.</span></span> <span data-ttu-id="5024b-364">以前存在的 URI 应像以前一样继续运行，并返回符合原始架构的资源。</span><span class="sxs-lookup"><span data-stu-id="5024b-364">The previously existing URIs should continue to operate as before, returning resources that conform to their original schema.</span></span>

<span data-ttu-id="5024b-365">继续前面的示例，如果将 `address` 字段重构为包含地址的每个构成部分的子字段（例如 `streetAddress`、`city`、`state` 和 `zipCode`），则此版本的资源可通过包含版本号的 URI（如 http://adventure-works.com/v2/customers/3）公开：</span><span class="sxs-lookup"><span data-stu-id="5024b-365">Extending the previous example, if the `address` field is restructured into sub-fields containing each constituent part of the address (such as `streetAddress`, `city`, `state`, and `zipCode`), this version of the resource could be exposed through a URI containing a version number, such as http://adventure-works.com/v2/customers/3:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

<span data-ttu-id="5024b-366">此版本控制机制非常简单，但依赖于将请求路由到相应终结点的服务器。</span><span class="sxs-lookup"><span data-stu-id="5024b-366">This versioning mechanism is very simple but depends on the server routing the request to the appropriate endpoint.</span></span> <span data-ttu-id="5024b-367">但是，随着 Web API 经过多次迭代而变得成熟，服务器必须支持多个不同版本，它可能变得难以处理。</span><span class="sxs-lookup"><span data-stu-id="5024b-367">However, it can become unwieldy as the web API matures through several iterations and the server has to support a number of different versions.</span></span> <span data-ttu-id="5024b-368">此外，从单纯的角度来看，在所有情况下客户端应用程序都要提取相同数据（客户 3），因此 URI 实在不应该因版本而有所不同。</span><span class="sxs-lookup"><span data-stu-id="5024b-368">Also, from a purist’s point of view, in all cases the client applications are fetching the same data (customer 3), so the URI should not really be different depending on the version.</span></span> <span data-ttu-id="5024b-369">此方案也增加了 HATEOAS 实现的复杂性，因为所有链接都需要在其 URI 中包括版本号。</span><span class="sxs-lookup"><span data-stu-id="5024b-369">This scheme also complicates implementation of HATEOAS as all links will need to include the version number in their URIs.</span></span>

### <a name="query-string-versioning"></a><span data-ttu-id="5024b-370">查询字符串版本控制</span><span class="sxs-lookup"><span data-stu-id="5024b-370">Query string versioning</span></span>
<span data-ttu-id="5024b-371">不是提供多个 URI，而是可以通过在追加到 HTTP 请求后面的查询字符串中使用参数来指定资源的版本，例如 *http://adventure-works.com/customers/3?version=2*。</span><span class="sxs-lookup"><span data-stu-id="5024b-371">Rather than providing multiple URIs, you can specify the version of the resource by using a parameter within the query string appended to the HTTP request, such as *http://adventure-works.com/customers/3?version=2*.</span></span> <span data-ttu-id="5024b-372">如果 version 参数被较旧的客户端应用程序省略，则应默认为有意义的值（例如 1）。</span><span class="sxs-lookup"><span data-stu-id="5024b-372">The version parameter should default to a meaningful value such as 1 if it is omitted by older client applications.</span></span>

<span data-ttu-id="5024b-373">此方法具有语义优势（即，同一资源始终从同一 URI 进行检索），但它依赖于代码处理请求以分析查询字符串并发送回相应的 HTTP 响应。</span><span class="sxs-lookup"><span data-stu-id="5024b-373">This approach has the semantic advantage that the same resource is always retrieved from the same URI, but it depends on the code that handles the request to parse the query string and send back the appropriate HTTP response.</span></span> <span data-ttu-id="5024b-374">此方法也与 URI 版本控制机制一样，增加了实现 HATEOAS 的复杂性。</span><span class="sxs-lookup"><span data-stu-id="5024b-374">This approach also suffers from the same complications for implementing HATEOAS as the URI versioning mechanism.</span></span>

> [!NOTE]
> <span data-ttu-id="5024b-375">某些较旧的 Web 浏览器和 Web 代理将不会缓存在 URL 中包括查询字符串的请求的响应。</span><span class="sxs-lookup"><span data-stu-id="5024b-375">Some older web browsers and web proxies will not cache responses for requests that include a query string in the URL.</span></span> <span data-ttu-id="5024b-376">这可能会对使用 Web API 的 Web 应用程序以及从此类 Web 浏览器运行的 Web 应用程序的性能产生不利影响。</span><span class="sxs-lookup"><span data-stu-id="5024b-376">This can have an adverse impact on performance for web applications that use a web API and that run from within such a web browser.</span></span>
>
>

### <a name="header-versioning"></a><span data-ttu-id="5024b-377">标头版本控制</span><span class="sxs-lookup"><span data-stu-id="5024b-377">Header versioning</span></span>
<span data-ttu-id="5024b-378">不是追加版本号作为查询字符串参数，而是可以实现指示资源的版本的自定义标头。</span><span class="sxs-lookup"><span data-stu-id="5024b-378">Rather than appending the version number as a query string parameter, you could implement a custom header that indicates the version of the resource.</span></span> <span data-ttu-id="5024b-379">此方法需要客户端应用程序将相应标头添加到所有请求，虽然如果省略了版本标头，处理客户端请求的代码可以使用默认值（版本 1）。</span><span class="sxs-lookup"><span data-stu-id="5024b-379">This approach requires that the client application adds the appropriate header to any requests, although the code handling the client request could use a default value (version 1) if the version header is omitted.</span></span> <span data-ttu-id="5024b-380">下面的示例利用了名为 *Custom-Header* 的自定义标头。</span><span class="sxs-lookup"><span data-stu-id="5024b-380">The following examples utilize a custom header named *Custom-Header*.</span></span> <span data-ttu-id="5024b-381">此标头的值指示 Web API 的版本。</span><span class="sxs-lookup"><span data-stu-id="5024b-381">The value of this header indicates the version of web API.</span></span>

<span data-ttu-id="5024b-382">版本 1：</span><span class="sxs-lookup"><span data-stu-id="5024b-382">Version 1:</span></span>

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Custom-Header: api-version=1
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

<span data-ttu-id="5024b-383">版本 2：</span><span class="sxs-lookup"><span data-stu-id="5024b-383">Version 2:</span></span>

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Custom-Header: api-version=2
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

<span data-ttu-id="5024b-384">请注意，与前面两个方法一样，实现 HATEOAS 需要在任何链接中包括相应的自定义标头。</span><span class="sxs-lookup"><span data-stu-id="5024b-384">Note that as with the previous two approaches, implementing HATEOAS requires including the appropriate custom header in any links.</span></span>

### <a name="media-type-versioning"></a><span data-ttu-id="5024b-385">媒体类型版本控制</span><span class="sxs-lookup"><span data-stu-id="5024b-385">Media type versioning</span></span>
<span data-ttu-id="5024b-386">如本指南前面所述，当客户端应用程序向 Web 服务器发送 HTTP GET 请求时，它应使用 Accept 标头规定它可以处理的内容的格式。</span><span class="sxs-lookup"><span data-stu-id="5024b-386">When a client application sends an HTTP GET request to a web server it should stipulate the format of the content that it can handle by using an Accept header, as described earlier in this guidance.</span></span> <span data-ttu-id="5024b-387">通常，*Accept* 标头的用途是允许客户端应用程序指定响应的正文应是 XML、JSON 还是客户端可以分析的其他某种常见格式。</span><span class="sxs-lookup"><span data-stu-id="5024b-387">Frequently the purpose of the *Accept* header is to allow the client application to specify whether the body of the response should be XML, JSON, or some other common format that the client can parse.</span></span> <span data-ttu-id="5024b-388">但是，可以定义包括以下信息的自定义媒体类型：该信息使客户端应用程序可以指示它所需的资源版本。</span><span class="sxs-lookup"><span data-stu-id="5024b-388">However, it is possible to define custom media types that include information enabling the client application to indicate which version of a resource it is expecting.</span></span> <span data-ttu-id="5024b-389">下面的示例演示了将 *Accept* 标头指定为值 *application/vnd.adventure-works.v1+json* 的请求。</span><span class="sxs-lookup"><span data-stu-id="5024b-389">The following example shows a request that specifies an *Accept* header with the value *application/vnd.adventure-works.v1+json*.</span></span> <span data-ttu-id="5024b-390">*vnd.adventure-works.v1* 元素向 Web 服务器指示它应返回资源的版本 1，而 *json* 元素则指定响应正文的格式应为 JSON：</span><span class="sxs-lookup"><span data-stu-id="5024b-390">The *vnd.adventure-works.v1* element indicates to the web server that it should return version 1 of the resource, while the *json* element specifies that the format of the response body should be JSON:</span></span>

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Accept: application/vnd.adventure-works.v1+json
...
```

<span data-ttu-id="5024b-391">处理请求的代码负责处理 *Accept* 标头并尽可能采用该值（客户端应用程序可以在 *Accept* 标头中指定多种格式，在这种情况下，Web 服务器可以在其中选择最适合的格式用于响应正文）。</span><span class="sxs-lookup"><span data-stu-id="5024b-391">The code handling the request is responsible for processing the *Accept* header and honoring it as far as possible (the client application may specify multiple formats in the *Accept* header, in which case the web server can choose the most appropriate format for the response body).</span></span> <span data-ttu-id="5024b-392">Web 服务器使用 Content-Type 标头确认响应正文中的数据格式：</span><span class="sxs-lookup"><span data-stu-id="5024b-392">The web server confirms the format of the data in the response body by using the Content-Type header:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/vnd.adventure-works.v1+json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

<span data-ttu-id="5024b-393">如果 Accept 标头未指定任何已知的媒体类型，则 Web 服务器可以生成 HTTP 406（不可接受）响应消息或返回使用默认媒体类型的消息。</span><span class="sxs-lookup"><span data-stu-id="5024b-393">If the Accept header does not specify any known media types, the web server could generate an HTTP 406 (Not Acceptable) response message or return a message with a default media type.</span></span>

<span data-ttu-id="5024b-394">此方法可以说是最纯粹的版本控制机制并自然地适用于 HATEOAS，后者可以在资源链接中包含相关数据的 MIME 类型。</span><span class="sxs-lookup"><span data-stu-id="5024b-394">This approach is arguably the purest of the versioning mechanisms and lends itself naturally to HATEOAS, which can include the MIME type of related data in resource links.</span></span>

> [!NOTE]
> <span data-ttu-id="5024b-395">选择版本控制策略时，还应考虑对性能的影响，尤其是在 Web 服务器上缓存时。</span><span class="sxs-lookup"><span data-stu-id="5024b-395">When you select a versioning strategy, you should also consider the implications on performance, especially caching on the web server.</span></span> <span data-ttu-id="5024b-396">URI 版本控制和查询字符串版本控制方案都是缓存友好的，因为同一 URI/查询字符串组合每次都指向相同的数据。</span><span class="sxs-lookup"><span data-stu-id="5024b-396">The URI versioning and Query String versioning schemes are cache-friendly inasmuch as the same URI/query string combination refers to the same data each time.</span></span>
>
> <span data-ttu-id="5024b-397">标头版本控制和媒体类型版本控制机制通常需要其他逻辑来检查自定义标头或 Accept 标头中的值。</span><span class="sxs-lookup"><span data-stu-id="5024b-397">The Header versioning and Media Type versioning mechanisms typically require additional logic to examine the values in the custom header or the Accept header.</span></span> <span data-ttu-id="5024b-398">在大型环境中，使用不同版本的 Web API 的多个客户端可能会在服务器端缓存中生成大量重复数据。</span><span class="sxs-lookup"><span data-stu-id="5024b-398">In a large-scale environment, many clients using different versions of a web API can result in a significant amount of duplicated data in a server-side cache.</span></span> <span data-ttu-id="5024b-399">如果客户端应用程序通过实现缓存的代理与 Web 服务器进行通信，并且该代理在当前未在其缓存中保留所请求数据的副本时，仅将请求转发到 Web 服务器，则此问题可能会变得很严重。</span><span class="sxs-lookup"><span data-stu-id="5024b-399">This issue can become acute if a client application communicates with a web server through a proxy that implements caching, and that only forwards a request to the web server if it does not currently hold a copy of the requested data in its cache.</span></span>
>
>

## <a name="open-api-initiative"></a><span data-ttu-id="5024b-400">Open API 计划</span><span class="sxs-lookup"><span data-stu-id="5024b-400">Open API Initiative</span></span>
<span data-ttu-id="5024b-401">[Open API 计划](https://www.openapis.org/)由一个行业协会创建，目的是标准化供应商的 REST API 说明。</span><span class="sxs-lookup"><span data-stu-id="5024b-401">The [Open API Initiative](https://www.openapis.org/) was created by an industry consortium to standardize REST API descriptions across vendors.</span></span> <span data-ttu-id="5024b-402">作为该计划的一部分，Swagger 2.0 规范被重新命名为 OpenAPI 规范 (OAS)，并引入 Open API 计划。</span><span class="sxs-lookup"><span data-stu-id="5024b-402">As part of this initiative, the Swagger 2.0 specification was renamed the OpenAPI Specification (OAS) and brought under the Open API Initiative.</span></span>

<span data-ttu-id="5024b-403">建议为 Web API 采用 OpenAPI。</span><span class="sxs-lookup"><span data-stu-id="5024b-403">You may want to adopt OpenAPI for your web APIs.</span></span> <span data-ttu-id="5024b-404">考虑的要点：</span><span class="sxs-lookup"><span data-stu-id="5024b-404">Some points to consider:</span></span>

- <span data-ttu-id="5024b-405">OpenAPI 规范附带关于应该如何设计 REST API 的准则。</span><span class="sxs-lookup"><span data-stu-id="5024b-405">The OpenAPI Specification comes with with a set of opinionated guidelines on how a REST API should be designed.</span></span> <span data-ttu-id="5024b-406">这有益于互操作性，但在设计 API 时需多加注意，以符合规范。</span><span class="sxs-lookup"><span data-stu-id="5024b-406">That has advantages for interoperability, but requires more care when designing your API to conform to the specification.</span></span>
- <span data-ttu-id="5024b-407">OpenAPI 首推协定优先的方法，而不是实现优先的方法。</span><span class="sxs-lookup"><span data-stu-id="5024b-407">OpenAPI promotes a contract-first approach, rather than an implementation-first approach.</span></span> <span data-ttu-id="5024b-408">协定优先意味着首先设计 API 协定（接口），然后写入实现协定的代码。</span><span class="sxs-lookup"><span data-stu-id="5024b-408">Contract-first means you design the API contract (the interface) first and then write code that implements the contract.</span></span> 
- <span data-ttu-id="5024b-409">Swagger 之类的工具可以从 API 协定生成客户端库或文档。</span><span class="sxs-lookup"><span data-stu-id="5024b-409">Tools like Swagger can generate client libraries or documentation from API contracts.</span></span> <span data-ttu-id="5024b-410">有关示例，请参阅[有关使用 Swagger 的 ASP.NET Web API 帮助页](/aspnet/core/tutorials/web-api-help-pages-using-swagger)。</span><span class="sxs-lookup"><span data-stu-id="5024b-410">For example, see [ASP.NET Web API Help Pages using Swagger](/aspnet/core/tutorials/web-api-help-pages-using-swagger).</span></span>

## <a name="more-information"></a><span data-ttu-id="5024b-411">详细信息</span><span class="sxs-lookup"><span data-stu-id="5024b-411">More information</span></span>
* <span data-ttu-id="5024b-412">[Microsoft REST API 指南](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md)包含设计公共 REST API 的详细建议。</span><span class="sxs-lookup"><span data-stu-id="5024b-412">The [Microsoft REST API Guidelines](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md) contain detailed recommendations for designing public REST APIs.</span></span>
* <span data-ttu-id="5024b-413">[RESTful Cookbook](http://restcookbook.com/) 包含构建 REST 样式 API 的简介。</span><span class="sxs-lookup"><span data-stu-id="5024b-413">The [RESTful Cookbook](http://restcookbook.com/) contains an introduction to building RESTful APIs.</span></span>
* <span data-ttu-id="5024b-414">[Web API 清单](https://mathieu.fenniak.net/the-api-checklist/)包含在设计和实现 Web API 时要考虑的有用的项目列表。</span><span class="sxs-lookup"><span data-stu-id="5024b-414">The [Web API Checklist](https://mathieu.fenniak.net/the-api-checklist/) contains a useful list of items to consider when designing and implementing a web API.</span></span>
* <span data-ttu-id="5024b-415">[Open API 计划](https://www.openapis.org/)站点包含 Open API 上所有的 相关文档和实现的详细信息。</span><span class="sxs-lookup"><span data-stu-id="5024b-415">The [Open API Initiative](https://www.openapis.org/) site, contains all related documentation and implementation details on Open API.</span></span>
