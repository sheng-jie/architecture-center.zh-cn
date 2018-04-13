---
title: API 设计指南
description: 有关如何创建合理设计的 Web API 的指南。
author: dragon119
ms.date: 01/12/2018
pnp.series.title: Best Practices
ms.openlocfilehash: a8c4a81835ebd3ebdba2fd2cec624a9a9d5646f5
ms.sourcegitcommit: ea7108f71dab09175ff69322874d1bcba800a37a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/17/2018
---
# <a name="api-design"></a><span data-ttu-id="3fbbd-103">API 设计</span><span class="sxs-lookup"><span data-stu-id="3fbbd-103">API design</span></span>

<span data-ttu-id="3fbbd-104">大多数新式 Web 应用程序都会公开 API，客户端可以使用这些 API 来与该应用程序交互。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-104">Most modern web applications expose APIs that clients can use to interact with the application.</span></span> <span data-ttu-id="3fbbd-105">设计良好的 Web API 应旨在支持：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-105">A well-designed web API should aim to support:</span></span>

* <span data-ttu-id="3fbbd-106">**平台独立性**。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-106">**Platform independence**.</span></span> <span data-ttu-id="3fbbd-107">不管 API 的内部实现方式如何，任何客户端都应该能够调用该 API。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-107">Any client should be able to call the API, regardless of how the API is implemented internally.</span></span> <span data-ttu-id="3fbbd-108">这就需要使用标准协议并创建一种机制，使客户端和 Web 服务能够就交换数据的格式达成一致。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-108">This requires using standard protocols, and having a mechanism whereby the client and the web service can agree on the format of the data to exchange.</span></span>

* <span data-ttu-id="3fbbd-109">**服务演变**。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-109">**Service evolution**.</span></span> <span data-ttu-id="3fbbd-110">Web API 应能在不影响客户端应用程序的情况下改进和添加功能。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-110">The web API should be able to evolve and add functionality independently from client applications.</span></span> <span data-ttu-id="3fbbd-111">随着 API 的发展，现有客户端应用程序应可继续运行而无需进行任何修改。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-111">As the API evolves, existing client applications should continue to function without modification.</span></span> <span data-ttu-id="3fbbd-112">所有功能应可发现，使客户端应用程序能够充分利用它们。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-112">All functionality should be discoverable, so that client applications can fully utilize it.</span></span>

<span data-ttu-id="3fbbd-113">本指南阐述在设计 Web API 时应考虑的问题。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-113">This guidance describes issues that you should consider when designing a web API.</span></span>

## <a name="introduction-to-rest"></a><span data-ttu-id="3fbbd-114">REST 简介</span><span class="sxs-lookup"><span data-stu-id="3fbbd-114">Introduction to REST</span></span>

<span data-ttu-id="3fbbd-115">在 2000 年，Roy Fielding 提议使用表述性状态转移 (REST) 作为设计 Web 服务的体系性方法。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-115">In 2000, Roy Fielding proposed Representational State Transfer (REST) as an architectural approach to designing web services.</span></span> <span data-ttu-id="3fbbd-116">REST 是基于超媒体构建分布式系统的架构样式。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-116">REST is an architectural style for building distributed systems based on hypermedia.</span></span> <span data-ttu-id="3fbbd-117">REST 独立于任何基础协议，并且不一定绑定到 HTTP。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-117">REST is independent of any underlying protocol and is not necessarily tied to HTTP.</span></span> <span data-ttu-id="3fbbd-118">但是，最常见的 REST 实现使用 HTTP 作为应用程序协议，本指南重点介绍如何设计适用于 HTTP 的 REST API。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-118">However, most common REST implementations use HTTP as the application protocol, and this guide focuses on designing REST APIs for HTTP.</span></span>

<span data-ttu-id="3fbbd-119">基于 HTTP 的 REST 的主要优势在于它使用开放标准，并且不会 API 或客户端应用程序的实现绑定到任何特定实现。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-119">A primary advantage of REST over HTTP is that it uses open standards, and does not bind the implementation of the API or the client applications any specific implementation.</span></span> <span data-ttu-id="3fbbd-120">例如，可以使用 ASP.NET 编写 REST Web 服务，而客户端应用程序能够使用可生成 HTTP 请求和分析 HTTP 响应的任何语言或工具集。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-120">For example, a REST web service could be written in ASP.NET, and client applications can use any language or toolset that can generate HTTP requests and parse HTTP responses.</span></span>

<span data-ttu-id="3fbbd-121">下面是使用 HTTP 设计 RESTful API 时的一些主要原则：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-121">Here are some of the main design principles of RESTful APIs using HTTP:</span></span>

- <span data-ttu-id="3fbbd-122">REST API 围绕资源设计，资源是可由客户端访问的任何类型的对象、数据或服务。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-122">REST APIs are designed around *resources*, which are any kind of object, data, or service that can be accessed by the client.</span></span> 

- <span data-ttu-id="3fbbd-123">每个资源有一个标识符，即，唯一标识该资源的 URI。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-123">A resource has an *identifier*, which is a URI that uniquely identifies that resource.</span></span> <span data-ttu-id="3fbbd-124">例如，特定客户订单的 URI 可能是：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-124">For example, the URI for a particular customer order might be:</span></span> 
 
    ```http
    http://adventure-works.com/orders/1
    ```
 
- <span data-ttu-id="3fbbd-125">客户端通过交换资源的表示形式来与服务交互。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-125">Clients interact with a service by exchanging *representations* of resources.</span></span> <span data-ttu-id="3fbbd-126">许多 Web API 使用 JSON 作为交换格式。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-126">Many web APIs use JSON as the exchange format.</span></span> <span data-ttu-id="3fbbd-127">例如，对上面所列的 URI 发出 GET 请求可能返回以下响应正文：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-127">For example, a GET request to the URI listed above might return this response body:</span></span>

    ```json
    {"orderId":1,"orderValue":99.90,"productId":1,"quantity":1}
    ```

- <span data-ttu-id="3fbbd-128">REST API 使用统一接口，这有助于分离客户端和服务实现。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-128">REST APIs use a uniform interface, which helps to decouple the client and service implementations.</span></span> <span data-ttu-id="3fbbd-129">对于基于 HTTP 构建的 REST API，统一接口包括使用标准 HTTP 谓词对资源执行操作。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-129">For REST APIs built on HTTP, the uniform interface includes using standard HTTP verbs to perform operations on resources.</span></span> <span data-ttu-id="3fbbd-130">最常见的操作是 GET、POST、PUT、PATCH 和 DELETE。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-130">The most common operations are GET, POST, PUT, PATCH, and DELETE.</span></span> 

- <span data-ttu-id="3fbbd-131">REST API 使用无状态请求模型。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-131">REST APIs use a stateless request model.</span></span> <span data-ttu-id="3fbbd-132">HTTP 请求应是独立的并可按任意顺序发生，因此保留请求之间的瞬时状态信息并不可行。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-132">HTTP requests should be independent and may occur in any order, so keeping transient state information between requests is not feasible.</span></span> <span data-ttu-id="3fbbd-133">存储信息的唯一位置是在资源本身中，并且每个请求应是原子操作。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-133">The only place where information is stored is in the resources themselves, and each request should be an atomic operation.</span></span> <span data-ttu-id="3fbbd-134">此约束可让 Web 服务获得高度可伸缩性，因为无需在客户端与特定服务器之间保留任何关联。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-134">This constraint enables web services to be highly scalable, because there is no need to retain any affinity between clients and specific servers.</span></span> <span data-ttu-id="3fbbd-135">任何服务器可以处理来自任何客户端的任何请求。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-135">Any server can handle any request from any client.</span></span> <span data-ttu-id="3fbbd-136">也就是说，其他因素可能会限制可伸缩性。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-136">That said, other factors can limit scalability.</span></span> <span data-ttu-id="3fbbd-137">例如，许多 Web 服务向后端数据存储写入数据，因此可能难以横向扩展。（[数据分区](./data-partitioning.md)一文介绍了横向扩展数据存储的策略。）</span><span class="sxs-lookup"><span data-stu-id="3fbbd-137">For example, many web services write to a backend data store, which may be hard to scale out. (The article [Data Partitioning](./data-partitioning.md) describes strategies to scale out a data store.)</span></span>

- <span data-ttu-id="3fbbd-138">REST API 由表示形式中包含的超媒体链接驱动。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-138">REST APIs are driven by hypermedia links that are contained in the representation.</span></span> <span data-ttu-id="3fbbd-139">例如，下面显示了某个订单的 JSON 表示形式。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-139">For example, the following shows a JSON representation of an order.</span></span> <span data-ttu-id="3fbbd-140">该表示形式包含用于获取或更新与该订单关联的客户的链接。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-140">It contains links to get or update the customer associated with the order.</span></span> 
 
    ```json
    {
        "orderID":3,
        "productID":2,
        "quantity":4,
        "orderValue":16.60,
        "links": [
            {"rel":"product","href":"http://adventure-works.com/customers/3", "action":"GET" },
            {"rel":"product","href":"http://adventure-works.com/customers/3", "action":"PUT" } 
        ]
    } 
    ```


<span data-ttu-id="3fbbd-141">2008 年，Leonard Richardson 提议对 Web API 使用以下[成熟度模型](https://martinfowler.com/articles/richardsonMaturityModel.html)：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-141">In 2008, Leonard Richardson proposed the following [maturity model](https://martinfowler.com/articles/richardsonMaturityModel.html) for web APIs:</span></span>

- <span data-ttu-id="3fbbd-142">级别 0：定义一个 URI，所有操作是对此 URI 发出的 POST 请求。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-142">Level 0: Define one URI, and all operations are POST requests to this URI.</span></span>
- <span data-ttu-id="3fbbd-143">级别 1：为各个资源单独创建 URI。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-143">Level 1: Create separate URIs for individual resources.</span></span>
- <span data-ttu-id="3fbbd-144">级别 2：使用 HTTP 方法来定义对资源执行的操作。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-144">Level 2: Use HTTP methods to define operations on resources.</span></span>
- <span data-ttu-id="3fbbd-145">级别 3：使用超媒体（HATEOAS，如下所述）。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-145">Level 3: Use hypermedia (HATEOAS, described below).</span></span>

<span data-ttu-id="3fbbd-146">根据 Fielding 的定义，级别 3 对应于某个真正的 RESTful API。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-146">Level 3 corresponds to a truly RESTful API according to Fielding's definition.</span></span> <span data-ttu-id="3fbbd-147">在实践中，许多发布的 Web API 大致都处于级别 2。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-147">In practice, many  published web APIs fall somewhere around level 2.</span></span>  

## <a name="organize-the-api-around-resources"></a><span data-ttu-id="3fbbd-148">围绕资源组织 API</span><span class="sxs-lookup"><span data-stu-id="3fbbd-148">Organize the API around resources</span></span>

<span data-ttu-id="3fbbd-149">侧重于 Web API 公开的业务实体。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-149">Focus on the business entities that the web API exposes.</span></span> <span data-ttu-id="3fbbd-150">例如，在电子商务系统中，主实体可能是客户和订单。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-150">For example, in an e-commerce system, the primary entities might be customers and orders.</span></span> <span data-ttu-id="3fbbd-151">可以通过发送包含订单信息的 HTTP POST 请求来创建订单。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-151">Creating an order can be achieved by sending an HTTP POST request that contains the order information.</span></span> <span data-ttu-id="3fbbd-152">HTTP 响应指示下单是否成功。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-152">The HTTP response indicates whether the order was placed successfully or not.</span></span> <span data-ttu-id="3fbbd-153">如果可能，资源 URI 应基于名词（资源）而不是动词（对资源执行的操作）。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-153">When possible, resource URIs should be based on nouns (the resource) and not verbs (the operations on the resource).</span></span> 

```HTTP
http://adventure-works.com/orders // Good

http://adventure-works.com/create-order // Avoid
```

<span data-ttu-id="3fbbd-154">资源无需基于单个物理数据项。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-154">A resource does not have to be based on a single physical data item.</span></span> <span data-ttu-id="3fbbd-155">例如，订单资源可以在内部实现为关系数据库中的多个表，但以单个实体的形式提供给客户端。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-155">For example, an order resource might be implemented internally as several tables in a relational database, but presented to the client as a single entity.</span></span> <span data-ttu-id="3fbbd-156">避免创建只是镜像数据库内部结构的 API。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-156">Avoid creating APIs that simply mirror the internal structure of a database.</span></span> <span data-ttu-id="3fbbd-157">REST 旨在为实体建模，以及为应用程序可对这些实体执行的操作建模。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-157">The purpose of REST is to model entities and the operations that an application can perform on those entities.</span></span> <span data-ttu-id="3fbbd-158">不应向内部实现公开客户端。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-158">A client should not be exposed to the internal implementation.</span></span>

<span data-ttu-id="3fbbd-159">实体通常分组成集合（订单、客户）。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-159">Entities are often grouped together into collections (orders, customers).</span></span> <span data-ttu-id="3fbbd-160">集合是不同于集合中的项的资源，应具有自身的 URI。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-160">A collection is a separate resource from the item within the collection, and should have its own URI.</span></span> <span data-ttu-id="3fbbd-161">例如，以下 URI 可以表示订单集合：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-161">For example, the following URI might represent the collection of orders:</span></span> 

```HTTP
http://adventure-works.com/orders
```

<span data-ttu-id="3fbbd-162">向集合 URI 发送 HTTP GET 请求可检索集合中的项列表。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-162">Sending an HTTP GET request to the collection URI retrieves a list of items in the collection.</span></span> <span data-ttu-id="3fbbd-163">集合中的每个项也有自身的唯一 URI。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-163">Each item in the collection also has its own unique URI.</span></span> <span data-ttu-id="3fbbd-164">对项的 URI 发出 HTTP GET 请求会返回该项的详细信息。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-164">An HTTP GET request to the item's URI returns the details of that item.</span></span> 

<span data-ttu-id="3fbbd-165">在 URI 中采用一致的命名约定。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-165">Adopt a consistent naming convention in URIs.</span></span> <span data-ttu-id="3fbbd-166">一般而言，有效的做法是对引用集合的 URI 使用复数名词。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-166">In general, it helps to use plural nouns for URIs that reference collections.</span></span> <span data-ttu-id="3fbbd-167">最好是将集合和项的 URI 组织成层次结构。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-167">It's a good practice to organize URIs for collections and items into a hierarchy.</span></span> <span data-ttu-id="3fbbd-168">例如，`/customers` 是客户集合的路径，`/customers/5` 是 ID 为 5 的客户的路径。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-168">For example, `/customers` is the path to the customers collection, and `/customers/5` is the path to the customer with ID equal to 5.</span></span> <span data-ttu-id="3fbbd-169">这种方法有助于使 Web API 保持直观。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-169">This approach helps to keep the web API intuitive.</span></span> <span data-ttu-id="3fbbd-170">此外，有许多 Web API 框架可以基于参数化 URI 路径来路由请求，因此，你可以对路径 `/customers/{id}` 定义路由。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-170">Also, many web API frameworks can route requests based on parameterized URI paths, so you could define a route for the path `/customers/{id}`.</span></span>

<span data-ttu-id="3fbbd-171">还需要考虑不同类型的资源之间的关系，以及如何公开这些关联。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-171">Also consider the relationships between different types of resources and how you might expose these associations.</span></span> <span data-ttu-id="3fbbd-172">例如，`/customers/5/orders` 可以表示客户 5 的所有订单。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-172">For example, the `/customers/5/orders` might represent all of the orders for customer 5.</span></span> <span data-ttu-id="3fbbd-173">我们还可以改变思维方向，使用类似于 `/orders/99/customer` 的 URI 来表示从订单到客户的关联。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-173">You could also go in the other direction, and represent the association from an order back to a customer with a URI such as `/orders/99/customer`.</span></span> <span data-ttu-id="3fbbd-174">但是，过度扩展此模型可能会变得难以实现。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-174">However, extending this model too far can become cumbersome to implement.</span></span> <span data-ttu-id="3fbbd-175">更好的解决方案是在 HTTP 响应消息的正文中提供指向关联资源的可导航链接。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-175">A better solution is to provide navigable links to associated resources in the body of the HTTP response message.</span></span> <span data-ttu-id="3fbbd-176">稍后的[使用 HATEOAS 方法启用相关资源的导航](#using-the-hateoas-approach-to-enable-navigation-to-related-resources)部分详细介绍了此机制。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-176">This mechanism is described in more detail in the section [Using the HATEOAS Approach to Enable Navigation To Related Resources later](#using-the-hateoas-approach-to-enable-navigation-to-related-resources).</span></span>

<span data-ttu-id="3fbbd-177">在更复杂的系统中，我们往往提供 URI（例如 `/customers/1/orders/99/products`），使客户端能够通过多个关系级别进行导航。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-177">In more complex systems, it can be tempting to provide URIs that enable a client to navigate through several levels of relationships, such as `/customers/1/orders/99/products`.</span></span> <span data-ttu-id="3fbbd-178">但是，如果资源之间的关系在将来更改，此级别的复杂性可能很难维护并且不够灵活。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-178">However, this level of complexity can be difficult to maintain and is inflexible if the relationships between resources change in the future.</span></span> <span data-ttu-id="3fbbd-179">相反，请尽量让 URI 相对简单。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-179">Instead, try to keep URIs relatively simple.</span></span> <span data-ttu-id="3fbbd-180">应用程序获取对某个资源的引用后，应可使用此引用查找与该资源相关的项目。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-180">Once an application has a reference to a resource, it should be possible to use this reference to find items related to that resource.</span></span> <span data-ttu-id="3fbbd-181">可将前面的查询替换为 URI `/customers/1/orders` 以查找客户 1 的所有订单，然后替换为 `/orders/99/products` 以查找此订单中的产品。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-181">The preceding query can be replaced with the URI `/customers/1/orders` to find all the orders for customer 1, and then `/orders/99/products` to find the products in this order.</span></span>

> [!TIP]
> <span data-ttu-id="3fbbd-182">避免需要复杂度超过*集合/项目/集合*的资源 URI。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-182">Avoid requiring resource URIs more complex than *collection/item/collection*.</span></span>

<span data-ttu-id="3fbbd-183">另一个因素是所有 Web 请求都会在 Web 服务器上施加负载。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-183">Another factor is that all web requests impose a load on the web server.</span></span> <span data-ttu-id="3fbbd-184">请求越多，负载就越大。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-184">The more requests, the bigger the load.</span></span> <span data-ttu-id="3fbbd-185">因此，请尽量避免使用公开大量小型资源的“琐碎”Web API。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-185">Therefore, try to avoid "chatty" web APIs that expose a large number of small resources.</span></span> <span data-ttu-id="3fbbd-186">此类 API 可能需要客户端应用程序发送多个请求才能找到它需要的所有数据。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-186">Such an API may require a client application to send multiple requests to find all of the data that it requires.</span></span> <span data-ttu-id="3fbbd-187">我们建议将数据非规范化，并将相关信息合并成可通过单个请求检索的较大资源。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-187">Instead, you might want to denormalize the data and combine related information into bigger resources that can be retrieved with a single request.</span></span> <span data-ttu-id="3fbbd-188">但是，需要根据提取客户端并不需要的数据所产生的开销来权衡此方法。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-188">However, you need to balance this approach against the overhead of fetching data that the client doesn't need.</span></span> <span data-ttu-id="3fbbd-189">检索大型对象可能增大请求的延迟时间，并产生额外的带宽成本。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-189">Retrieving large objects can increase the latency of a request and incur additional bandwidth costs.</span></span> <span data-ttu-id="3fbbd-190">有关这些性能对立模式的详细信息，请参阅[琐碎 I/O](../antipatterns/chatty-io/index.md) 和[超量提取](../antipatterns/extraneous-fetching/index.md)。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-190">For more information about these performance antipatterns, see [Chatty I/O](../antipatterns/chatty-io/index.md) and [Extraneous Fetching](../antipatterns/extraneous-fetching/index.md).</span></span>

<span data-ttu-id="3fbbd-191">避免在 Web API 与底层数据源之间引入依赖关系。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-191">Avoid introducing dependencies between the web API and the underlying data sources.</span></span> <span data-ttu-id="3fbbd-192">例如，如果数据存储在关系数据库中，则 Web API 不需要将每个表公开为资源集合。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-192">For example, if your data is stored in a relational database, the web API doesn't need to expose each table as a collection of resources.</span></span> <span data-ttu-id="3fbbd-193">事实上，这可以算作一种粗劣的设计。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-193">In fact, that's probably a poor design.</span></span> <span data-ttu-id="3fbbd-194">请考虑将 Web API 视为数据库的抽象。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-194">Instead, think of the web API as an abstraction of the database.</span></span> <span data-ttu-id="3fbbd-195">如有必要，可在数据库与 Web API 之间引入映射层。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-195">If necessary, introduce a mapping layer between the database and the web API.</span></span> <span data-ttu-id="3fbbd-196">这样，对于底层数据库方案所做的更改不会影响客户端应用程序。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-196">That way, client applications are isolated from changes to the underlying database scheme.</span></span>

<span data-ttu-id="3fbbd-197">最后，可能无法将 Web API 实现的每个操作都映射到特定资源。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-197">Finally, it might not be possible to map every operation implemented by a web API to a specific resource.</span></span> <span data-ttu-id="3fbbd-198">可以通过 HTTP 请求处理此类非资源方案，这些请求调用某个函数并将结果作为 HTTP 响应消息返回。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-198">You can handle such *non-resource* scenarios through HTTP requests that invoke a function and return the results as an HTTP response message.</span></span> <span data-ttu-id="3fbbd-199">例如，实现简单计算器操作（例如，加法和减法）的 Web API 可以提供公开这些操作作为伪资源的 URI，并使用查询字符串来指定所需的参数。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-199">For example, a web API that implements simple calculator operations such as add and subtract could provide URIs that expose these operations as pseudo resources and use the query string to specify the parameters required.</span></span> <span data-ttu-id="3fbbd-200">例如，向 URI */add?operand1=99&operand2=1* 发出 GET 请求会返回正文包含值 100 的响应消息。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-200">For example a GET request to the URI */add?operand1=99&operand2=1* would return a response message with the body containing the value 100.</span></span> <span data-ttu-id="3fbbd-201">但是，请尽量少使用这些形式的 URI。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-201">However, only use these forms of URIs sparingly.</span></span>

## <a name="define-operations-in-terms-of-http-methods"></a><span data-ttu-id="3fbbd-202">根据 HTTP 方法定义操作</span><span class="sxs-lookup"><span data-stu-id="3fbbd-202">Define operations in terms of HTTP methods</span></span>

<span data-ttu-id="3fbbd-203">HTTP 协议定义了大量为请求赋于语义含义的方法。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-203">The HTTP protocol defines a number of methods that assign semantic meaning to a request.</span></span> <span data-ttu-id="3fbbd-204">大多数 RESTful Web API 使用的常见 HTTP 方法是：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-204">The common HTTP methods used by most RESTful web APIs are:</span></span>

* <span data-ttu-id="3fbbd-205">**GET** 检索位于指定 URI 处的资源的表示形式。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-205">**GET** retrieves a representation of the resource at the specified URI.</span></span> <span data-ttu-id="3fbbd-206">响应消息的正文包含所请求资源的详细信息。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-206">The body of the response message contains the details of the requested resource.</span></span>
* <span data-ttu-id="3fbbd-207">**POST** 在指定的 URI 处创建新资源。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-207">**POST** creates a new resource at the specified URI.</span></span> <span data-ttu-id="3fbbd-208">请求消息的正文将提供新资源的详细信息。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-208">The body of the request message provides the details of the new resource.</span></span> <span data-ttu-id="3fbbd-209">请注意，POST 还用于触发不实际创建资源的操作。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-209">Note that POST can also be used to trigger operations that don't actually create resources.</span></span>
* <span data-ttu-id="3fbbd-210">**PUT** 在指定的 URI 处创建或替换资源。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-210">**PUT** either creates or replaces the resource at the specified URI.</span></span> <span data-ttu-id="3fbbd-211">请求消息的正文指定要创建或更新的资源。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-211">The body of the request message specifies the resource to be created or updated.</span></span>
* <span data-ttu-id="3fbbd-212">**PATCH** 对资源执行部分更新。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-212">**PATCH** performs a partial update of a resource.</span></span> <span data-ttu-id="3fbbd-213">请求正文指定要应用到资源的更改集。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-213">The request body specifies the set of changes to apply to the resource.</span></span>
* <span data-ttu-id="3fbbd-214">**DELETE** 删除位于指定 URI 处的资源。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-214">**DELETE** removes the resource at the specified URI.</span></span>

<span data-ttu-id="3fbbd-215">特定请求的影响应取决于资源是集合还是单个项。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-215">The effect of a specific request should depend on whether the resource is a collection or an individual item.</span></span> <span data-ttu-id="3fbbd-216">下表汇总了使用电子商务示例的大多数 RESTful 实现所采用的常见约定。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-216">The following table summarizes the common conventions adopted by most RESTful implementations using the ecommerce example.</span></span> <span data-ttu-id="3fbbd-217">请注意，可能并非实现所有这些请求；这取决于特定方案。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-217">Note that not all of these requests might be implemented; it depends on the specific scenario.</span></span>

| <span data-ttu-id="3fbbd-218">**资源**</span><span class="sxs-lookup"><span data-stu-id="3fbbd-218">**Resource**</span></span> | <span data-ttu-id="3fbbd-219">**POST**</span><span class="sxs-lookup"><span data-stu-id="3fbbd-219">**POST**</span></span> | <span data-ttu-id="3fbbd-220">**GET**</span><span class="sxs-lookup"><span data-stu-id="3fbbd-220">**GET**</span></span> | <span data-ttu-id="3fbbd-221">**PUT**</span><span class="sxs-lookup"><span data-stu-id="3fbbd-221">**PUT**</span></span> | <span data-ttu-id="3fbbd-222">**DELETE**</span><span class="sxs-lookup"><span data-stu-id="3fbbd-222">**DELETE**</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="3fbbd-223">/customers</span><span class="sxs-lookup"><span data-stu-id="3fbbd-223">/customers</span></span> |<span data-ttu-id="3fbbd-224">创建新客户</span><span class="sxs-lookup"><span data-stu-id="3fbbd-224">Create a new customer</span></span> |<span data-ttu-id="3fbbd-225">检索所有客户</span><span class="sxs-lookup"><span data-stu-id="3fbbd-225">Retrieve all customers</span></span> |<span data-ttu-id="3fbbd-226">批量更新客户</span><span class="sxs-lookup"><span data-stu-id="3fbbd-226">Bulk update of customers</span></span> |<span data-ttu-id="3fbbd-227">删除所有客户</span><span class="sxs-lookup"><span data-stu-id="3fbbd-227">Remove all customers</span></span> |
| <span data-ttu-id="3fbbd-228">/customers/1</span><span class="sxs-lookup"><span data-stu-id="3fbbd-228">/customers/1</span></span> |<span data-ttu-id="3fbbd-229">错误</span><span class="sxs-lookup"><span data-stu-id="3fbbd-229">Error</span></span> |<span data-ttu-id="3fbbd-230">检索客户 1 的详细信息</span><span class="sxs-lookup"><span data-stu-id="3fbbd-230">Retrieve the details for customer 1</span></span> |<span data-ttu-id="3fbbd-231">如果客户 1 存在，则更新其详细信息</span><span class="sxs-lookup"><span data-stu-id="3fbbd-231">Update the details of customer 1 if it exists</span></span> |<span data-ttu-id="3fbbd-232">删除客户 1</span><span class="sxs-lookup"><span data-stu-id="3fbbd-232">Remove customer 1</span></span> |
| <span data-ttu-id="3fbbd-233">/customers/1/orders</span><span class="sxs-lookup"><span data-stu-id="3fbbd-233">/customers/1/orders</span></span> |<span data-ttu-id="3fbbd-234">创建客户 1 的新订单</span><span class="sxs-lookup"><span data-stu-id="3fbbd-234">Create a new order for customer 1</span></span> |<span data-ttu-id="3fbbd-235">检索客户 1 的所有订单</span><span class="sxs-lookup"><span data-stu-id="3fbbd-235">Retrieve all orders for customer 1</span></span> |<span data-ttu-id="3fbbd-236">批量更新客户 1 的订单</span><span class="sxs-lookup"><span data-stu-id="3fbbd-236">Bulk update of orders for customer 1</span></span> |<span data-ttu-id="3fbbd-237">删除客户 1 的所有订单</span><span class="sxs-lookup"><span data-stu-id="3fbbd-237">Remove all orders for customer 1</span></span> |

<span data-ttu-id="3fbbd-238">POST、PUT 和 PATCH 之间的差异可能会引起混淆。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-238">The differences between POST, PUT, and PATCH can be confusing.</span></span>

- <span data-ttu-id="3fbbd-239">POST 请求创建资源。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-239">A POST request creates a resource.</span></span> <span data-ttu-id="3fbbd-240">服务器为新资源分配 URI，并将该 URI 返回给客户端。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-240">The server assigns a URI for the new resource, and returns that URI to the client.</span></span> <span data-ttu-id="3fbbd-241">在 REST 模型中，我们经常向集合应用 POST 请求。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-241">In the REST model, you frequently apply POST requests to collections.</span></span> <span data-ttu-id="3fbbd-242">新资源将添加到集合中。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-242">The new resource is added to the collection.</span></span> <span data-ttu-id="3fbbd-243">还可以使用 POST 请求将待处理数据提交到现有资源，且不创建任何新资源。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-243">A POST request can also be used to submit data for processing to an existing resource, without any new resource being created.</span></span>

- <span data-ttu-id="3fbbd-244">PUT 请求创建资源或更新现有资源。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-244">A PUT request creates a resource *or* updates an existing resource.</span></span> <span data-ttu-id="3fbbd-245">客户端指定资源的 URI。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-245">The client specifies the URI for the resource.</span></span> <span data-ttu-id="3fbbd-246">请求正文包含资源的完整表示形式。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-246">The request body contains a complete representation of the resource.</span></span> <span data-ttu-id="3fbbd-247">如果已存在具有此 URI 的资源，则替换该资源。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-247">If a resource with this URI already exists, it is replaced.</span></span> <span data-ttu-id="3fbbd-248">否则创建新资源（如果服务器支持此操作）。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-248">Otherwise a new resource is created, if the server supports doing so.</span></span> <span data-ttu-id="3fbbd-249">PUT 请求往往应用到充当单个项的资源（例如特定的客户）而不是集合。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-249">PUT requests are most frequently applied to resources that are individual items, such as a specific customer, rather than collections.</span></span> <span data-ttu-id="3fbbd-250">服务器可能支持通过 PUT 更新，但不支持通过 PUT 执行创建。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-250">A server might support updates but not creation via PUT.</span></span> <span data-ttu-id="3fbbd-251">是否支持通过 PUT 执行创建取决于在创建某个资源之前，客户端能否以有意义的方式向该资源分配 URI。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-251">Whether to support creation via PUT depends on whether the client can meaningfully assign a URI to a resource before it exists.</span></span> <span data-ttu-id="3fbbd-252">如果不能，则可以使用 POST 来创建资源，并使用 PUT 或 PATCH 来执行更新。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-252">If not, then use POST to create resources and PUT or PATCH to update.</span></span>

- <span data-ttu-id="3fbbd-253">PATCH 请求对现有资源执行部分更新。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-253">A PATCH request performs a *partial update* to an existing resource.</span></span> <span data-ttu-id="3fbbd-254">客户端指定资源的 URI。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-254">The client specifies the URI for the resource.</span></span> <span data-ttu-id="3fbbd-255">请求正文指定要应用到资源的更改集。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-255">The request body specifies a set of *changes* to apply to the resource.</span></span> <span data-ttu-id="3fbbd-256">这比使用 PUT 更高效，因为客户端只发送更改，而无需发送资源的整个表示形式。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-256">This can be more efficient than using PUT, because the client only sends the changes, not the entire representation of the resource.</span></span> <span data-ttu-id="3fbbd-257">从技术上讲，如果受服务器的支持，PATCH 也可以创建新资源（通过对一个“null”资源指定一组更新）。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-257">Technically PATCH can also create a new resource (by specifying a set of updates to a "null" resource), if the server supports this.</span></span> 

<span data-ttu-id="3fbbd-258">PUT 请求必须是幂等的。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-258">PUT requests must be idempotent.</span></span> <span data-ttu-id="3fbbd-259">如果客户端多次提交同一个 PUT 请求，结果应始终相同（使用相同的值修改相同的资源）。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-259">If a client submits the same PUT request multiple times, the results should always be the same (the same resource will be modified with the same values).</span></span> <span data-ttu-id="3fbbd-260">无法保证 POST 和 PATCH 请求的幂等性。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-260">POST and PATCH requests are not guaranteed to be idempotent.</span></span>

## <a name="conform-to-http-semantics"></a><span data-ttu-id="3fbbd-261">符合 HTTP 语义</span><span class="sxs-lookup"><span data-stu-id="3fbbd-261">Conform to HTTP semantics</span></span>

<span data-ttu-id="3fbbd-262">本部分阐述在设计符合 HTTP 规范的 API 时的一些典型注意事项。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-262">This section describes some typical considerations for designing an API that conforms to the HTTP specification.</span></span> <span data-ttu-id="3fbbd-263">但是，本部分不会介绍每个可能的细节或方案。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-263">However, it doesn't cover every possible detail or scenario.</span></span> <span data-ttu-id="3fbbd-264">如有疑问，请查阅 HTTP 规范。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-264">When in doubt, consult the HTTP specifications.</span></span>

### <a name="media-types"></a><span data-ttu-id="3fbbd-265">媒体类型</span><span class="sxs-lookup"><span data-stu-id="3fbbd-265">Media types</span></span>

<span data-ttu-id="3fbbd-266">如前所述，客户端和服务器交换资源的表示形式。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-266">As mentioned earlier, clients and servers exchange representations of resources.</span></span> <span data-ttu-id="3fbbd-267">例如，在 POST 请求中，请求正文包含要创建的资源的表示形式。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-267">For example, in a POST request, the request body contains a representation of the resource to create.</span></span> <span data-ttu-id="3fbbd-268">在 GET 请求中，响应正文包含已提取的资源的表示形式。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-268">In a GET request, the response body contains a representation of the fetched resource.</span></span>

<span data-ttu-id="3fbbd-269">在 HTTP 协议中，格式是使用媒体类型（也称为 MIME 类型）指定的。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-269">In the HTTP protocol, formats are specified through the use of *media types*, also called MIME types.</span></span> <span data-ttu-id="3fbbd-270">对于非二进制数据，大多数 Web API 支持 JSON（媒体类型 = application/json），可能还支持 XML（媒体类型 = application/xml）。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-270">For non-binary data, most web APIs support JSON (media type = application/json) and possibly XML (media type = application/xml).</span></span> 

<span data-ttu-id="3fbbd-271">请求或响应中的 Content-Type 标头指定表示形式的格式。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-271">The Content-Type header in a request or response specifies the format of the representation.</span></span> <span data-ttu-id="3fbbd-272">下面是包含 JSON 数据的 POST 请求示例：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-272">Here is an example of a POST request that includes JSON data:</span></span>

```HTTP
POST http://adventure-works.com/orders HTTP/1.1
Content-Type: application/json; charset=utf-8
Content-Length: 57

{"Id":1,"Name":"Gizmo","Category":"Widgets","Price":1.99}
```

<span data-ttu-id="3fbbd-273">如果服务器不支持媒体类型，则应返回 HTTP 状态代码 415（不支持的媒体类型）。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-273">If the server doesn't support the media type, it should return HTTP status code 415 (Unsupported Media Type).</span></span>

<span data-ttu-id="3fbbd-274">客户端请求可以包含一个 Accept 标头，该标头包含客户端可以接受的、来自服务器的响应消息中的媒体类型列表。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-274">A client request can include an Accept header that contains a list of media types the client will accept from the server in the response message.</span></span> <span data-ttu-id="3fbbd-275">例如：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-275">For example:</span></span>

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
Accept: application/json
```

<span data-ttu-id="3fbbd-276">如果服务器无法匹配所列的任何媒体类型，应返回 HTTP 状态代码 406（不可接受）。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-276">If the server cannot match any of the media type(s) listed, it should return HTTP status code 406 (Not Acceptable).</span></span> 

### <a name="get-methods"></a><span data-ttu-id="3fbbd-277">GET 方法</span><span class="sxs-lookup"><span data-stu-id="3fbbd-277">GET methods</span></span>

<span data-ttu-id="3fbbd-278">成功的 GET 方法通常返回 HTTP 状态代码 200（正常）。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-278">A successful GET method typically returns HTTP status code 200 (OK).</span></span> <span data-ttu-id="3fbbd-279">如果找不到资源，该方法应返回 404（未找到）。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-279">If the resource cannot be found, the method should return 404 (Not Found).</span></span>

### <a name="post-methods"></a><span data-ttu-id="3fbbd-280">POST 方法</span><span class="sxs-lookup"><span data-stu-id="3fbbd-280">POST methods</span></span>

<span data-ttu-id="3fbbd-281">如果 POST 方法创建了新资源，则会返回 HTTP 状态代码 201（已创建）。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-281">If a POST method creates a new resource, it returns HTTP status code 201 (Created).</span></span> <span data-ttu-id="3fbbd-282">新资源的 URI 包含在响应的 Location 标头中。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-282">The URI of the new resource is included in the Location header of the response.</span></span> <span data-ttu-id="3fbbd-283">响应正文包含资源的表示形式。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-283">The response body contains a representation of the resource.</span></span>

<span data-ttu-id="3fbbd-284">如果该方法执行了一些处理但未创建新资源，则可以返回 HTTP 状态代码 200，并在响应正文中包含操作结果。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-284">If the method does some processing but does not create a new resource, the method can return HTTP status code 200 and include the result of the operation in the response body.</span></span> <span data-ttu-id="3fbbd-285">或者，如果没有可返回的结果，该方法可以返回 HTTP 状态代码 204（无内容）但不返回任何响应正文。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-285">Alternatively, if there is no result to return, the method can return HTTP status code 204 (No Content) with no response body.</span></span>

<span data-ttu-id="3fbbd-286">如果客户端将无效数据放入请求，服务器应返回 HTTP 状态代码 400（错误的请求）。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-286">If the client puts invalid data into the request, the server should return HTTP status code 400 (Bad Request).</span></span> <span data-ttu-id="3fbbd-287">响应正文可以包含有关错误的其他信息，或包含可提供更多详细信息的 URI 链接。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-287">The response body can contain additional information about the error or a link to a URI that provides more details.</span></span>

### <a name="put-methods"></a><span data-ttu-id="3fbbd-288">PUT 方法</span><span class="sxs-lookup"><span data-stu-id="3fbbd-288">PUT methods</span></span>

<span data-ttu-id="3fbbd-289">与 POST 方法一样，如果 PUT 方法创建了新资源，则会返回 HTTP 状态代码 201（已创建）。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-289">If a PUT method creates a new resource, it returns HTTP status code 201 (Created), as with a POST method.</span></span> <span data-ttu-id="3fbbd-290">如果该方法更新了现有资源，则会返回 200（正常）或 204（无内容）。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-290">If the method updates an existing resource, it returns either 200 (OK) or 204 (No Content).</span></span> <span data-ttu-id="3fbbd-291">在某些情况下，可能无法更新现有资源。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-291">In some cases, it might not be possible to update an existing resource.</span></span> <span data-ttu-id="3fbbd-292">在这种情况下，可考虑返回 HTTP 状态代码 409（冲突）。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-292">In that case, consider returning HTTP status code 409 (Conflict).</span></span> 

<span data-ttu-id="3fbbd-293">请考虑实现可批量更新集合中的多个资源的批量 HTTP PUT 操作。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-293">Consider implementing bulk HTTP PUT operations that can batch updates to multiple resources in a collection.</span></span> <span data-ttu-id="3fbbd-294">PUT 请求应指定集合的 URI，而请求正文则应指定要修改的资源的详细信息。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-294">The PUT request should specify the URI of the collection, and the request body should specify the details of the resources to be modified.</span></span> <span data-ttu-id="3fbbd-295">此方法可帮助减少交互成本并提高性能。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-295">This approach can help to reduce chattiness and improve performance.</span></span>

### <a name="patch-methods"></a><span data-ttu-id="3fbbd-296">PATCH 方法</span><span class="sxs-lookup"><span data-stu-id="3fbbd-296">PATCH methods</span></span>

<span data-ttu-id="3fbbd-297">客户端可以使用 PATCH 请求向现有资源发送一组修补文档形式的更新。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-297">With a PATCH request, the client sends a set of updates to an existing resource, in the form of a *patch document*.</span></span> <span data-ttu-id="3fbbd-298">服务器将处理该修补文档以执行更新。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-298">The server processes the patch document to perform the update.</span></span> <span data-ttu-id="3fbbd-299">修补文档不会描述整个资源，而只描述一组要应用的更改。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-299">The patch document doesn't describe the whole resource, only a set of changes to apply.</span></span> <span data-ttu-id="3fbbd-300">PATCH 方法的规范 ([RFC 5789](https://tools.ietf.org/html/rfc5789)) 未定义修补文档的特定格式。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-300">The specification for the PATCH method ([RFC 5789](https://tools.ietf.org/html/rfc5789)) doesn't define a particular format for patch documents.</span></span> <span data-ttu-id="3fbbd-301">必须从请求中的媒体类型推断格式。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-301">The format must be inferred from the media type in the request.</span></span>

<span data-ttu-id="3fbbd-302">JSON 也许是 Web API 的最常用数据格式。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-302">JSON is probably the most common data format for web APIs.</span></span> <span data-ttu-id="3fbbd-303">有两种基于 JSON 的主要修补格式，分别称作“JSON 修补”和“JSON 合并修补”。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-303">There are two main JSON-based patch formats, called *JSON patch* and *JSON merge patch*.</span></span>

<span data-ttu-id="3fbbd-304">JSON 合并修补更简单一些。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-304">JSON merge patch is somewhat simpler.</span></span> <span data-ttu-id="3fbbd-305">修补文档的结构与原始 JSON 资源相同，但只包含要更改或添加的字段的子集。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-305">The patch document has the same structure as the original JSON resource, but includes just the subset of fields that should be changed or added.</span></span> <span data-ttu-id="3fbbd-306">此外，可以通过在修补文档中为字段值指定 `null`，来删除该字段。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-306">In addition, a field can be deleted by specifying `null` for the field value in the patch document.</span></span> <span data-ttu-id="3fbbd-307">（这意味着，如果原始资源包含显式 null 值，则不适合使用合并修补。）</span><span class="sxs-lookup"><span data-stu-id="3fbbd-307">(That means merge patch is not suitable if the original resource can have explicit null values.)</span></span>

<span data-ttu-id="3fbbd-308">例如，假设原始资源采用以下 JSON 表示形式：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-308">For example, suppose the original resource has the following JSON representation:</span></span>

```json
{ 
    "name":"gizmo",
    "category":"widgets",
    "color":"blue",
    "price":10
}
```

<span data-ttu-id="3fbbd-309">下面是此资源的可能 JSON 合并修补代码：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-309">Here is a possible JSON merge patch for this resource:</span></span>

```json
{ 
    "price":12,
    "color":null,
    "size":"small"
}
```

<span data-ttu-id="3fbbd-310">此代码告知服务器要更新“price”，删除“color”并添加“size”。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-310">This tells the server to update "price", delete "color", and add "size".</span></span> <span data-ttu-id="3fbbd-311">不修改“name”和“category”。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-311">"Name" and "category" are not modified.</span></span> <span data-ttu-id="3fbbd-312">有关 JSON 合并修补的具体详细信息，请参阅 [RFC 7396](https://tools.ietf.org/html/rfc7396)。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-312">For the exact details of JSON merge patch, see [RFC 7396](https://tools.ietf.org/html/rfc7396).</span></span> <span data-ttu-id="3fbbd-313">JSON 合并修补的媒体类型是“application/merge-patch+json”。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-313">The media type for JSON merge patch is "application/merge-patch+json".</span></span>

<span data-ttu-id="3fbbd-314">由于修补文档中的 `null` 具有特殊的含义，如果原始资源包含显式 null 值，则不适合使用合并修补。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-314">Merge patch is not suitable if the original resource can contain explicit null values, due to the special meaning of `null` in the patch document.</span></span> <span data-ttu-id="3fbbd-315">此外，修补文档不会指定服务器应用更新的顺序。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-315">Also, the patch document doesn't specify the order that the server should apply the updates.</span></span> <span data-ttu-id="3fbbd-316">此限制是否造成影响具体取决于数据和域。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-316">That may or may not matter, depending on the data and the domain.</span></span> <span data-ttu-id="3fbbd-317">[RFC 6902](https://tools.ietf.org/html/rfc6902) 中定义的 JSON 修补更灵活。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-317">JSON patch, defined in [RFC 6902](https://tools.ietf.org/html/rfc6902), is more flexible.</span></span> <span data-ttu-id="3fbbd-318">它以操作序列的形式指定要应用的更改。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-318">It specifies the changes as a sequence of operations to apply.</span></span> <span data-ttu-id="3fbbd-319">操作包括添加、删除、替换、复制和测试（以验证值）。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-319">Operations include add, remove, replace, copy, and test (to validate values).</span></span> <span data-ttu-id="3fbbd-320">JSON 修补的媒体类型是“application/json-patch+json”。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-320">The media type for JSON patch is "application/json-patch+json".</span></span>

<span data-ttu-id="3fbbd-321">下面是在处理 PATCH 请求时可能遇到的典型错误状态，以及相应的 HTTP 状态代码。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-321">Here are some typical error conditions that might be encountered when processing a PATCH request, along with the appropriate HTTP status code.</span></span>

| <span data-ttu-id="3fbbd-322">添加状态</span><span class="sxs-lookup"><span data-stu-id="3fbbd-322">Error condition</span></span> | <span data-ttu-id="3fbbd-323">HTTP 状态代码</span><span class="sxs-lookup"><span data-stu-id="3fbbd-323">HTTP status code</span></span> |
|-----------|------------|
| <span data-ttu-id="3fbbd-324">修补文档格式不受支持。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-324">The patch document format isn't supported.</span></span> | <span data-ttu-id="3fbbd-325">415（媒体类型不受支持）</span><span class="sxs-lookup"><span data-stu-id="3fbbd-325">415 (Unsupported Media Type)</span></span> |
| <span data-ttu-id="3fbbd-326">修补文档的格式不正确。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-326">Malformed patch document.</span></span> | <span data-ttu-id="3fbbd-327">400（错误的请求）</span><span class="sxs-lookup"><span data-stu-id="3fbbd-327">400 (Bad Request)</span></span> |
| <span data-ttu-id="3fbbd-328">修补文档有效，但无法将更改应用到处于当前状态的资源。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-328">The patch document is valid, but the changes can't be applied to the resource in its current state.</span></span> | <span data-ttu-id="3fbbd-329">409（冲突）</span><span class="sxs-lookup"><span data-stu-id="3fbbd-329">409 (Conflict)</span></span>

### <a name="delete-methods"></a><span data-ttu-id="3fbbd-330">DELETE 方法</span><span class="sxs-lookup"><span data-stu-id="3fbbd-330">DELETE methods</span></span>

<span data-ttu-id="3fbbd-331">如果删除操作成功，Web 服务器应以 HTTP 状态代码 204 做出响应，指示已成功处理该过程，但响应正文不包含其他信息。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-331">If the delete operation is successful, the web server should respond with HTTP status code 204, indicating that the process has been successfully handled, but that the response body contains no further information.</span></span> <span data-ttu-id="3fbbd-332">如果资源不存在，Web 服务器可以返回 HTTP 404（未找到）。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-332">If the resource doesn't exist, the web server can return HTTP 404 (Not Found).</span></span>

### <a name="asynchronous-operations"></a><span data-ttu-id="3fbbd-333">异步操作</span><span class="sxs-lookup"><span data-stu-id="3fbbd-333">Asynchronous operations</span></span>

<span data-ttu-id="3fbbd-334">有时，POST、PUT、PATCH 或 DELETE 操作可能需要经过一段时间的处理才能完成。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-334">Sometimes a POST, PUT, PATCH, or DELETE operation might require processing that takes awhile to complete.</span></span> <span data-ttu-id="3fbbd-335">如果需要等待该操作完成后才能向客户端发送响应，可能会造成不可接受的延迟。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-335">If you wait for completion before sending a response to the client, it may cause unacceptable latency.</span></span> <span data-ttu-id="3fbbd-336">在这种情况下，请考虑将该操作设置为异步操作。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-336">If so, consider making the operation asynchronous.</span></span> <span data-ttu-id="3fbbd-337">返回 HTTP 状态代码 202（已接受），指示该请求已接受进行处理，但尚未完成。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-337">Return HTTP status code 202 (Accepted) to indicate the request was accepted for processing but is not completed.</span></span> 

<span data-ttu-id="3fbbd-338">应公开一个可返回异步请求状态的终结点，使客户端能够通过轮询状态终结点来监视状态。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-338">You should expose an endpoint that returns the status of an asynchronous request, so the client can monitor the status by polling the status endpoint.</span></span> <span data-ttu-id="3fbbd-339">在 202 响应的 Location 标头中包含状态终结点的 URI。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-339">Include the URI of the status endpoint in the Location header of the 202 response.</span></span> <span data-ttu-id="3fbbd-340">例如：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-340">For example:</span></span>

```http
HTTP/1.1 202 Accepted
Location: /api/status/12345
```

<span data-ttu-id="3fbbd-341">如果客户端向此终结点发送 GET 请求，响应中应包含该请求的当前状态。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-341">If the client sends a GET request to this endpoint, the response should contain the current status of the request.</span></span> <span data-ttu-id="3fbbd-342">（可选）响应中还可以包含预计完成时间，或者用于取消操作的链接。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-342">Optionally, it could also include an estimated time to completion or a link to cancel the operation.</span></span> 

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "status":"In progress",
    "link": { "rel":"cancel", "method":"delete", "href":"/api/status/12345"
}
```

<span data-ttu-id="3fbbd-343">如果异步操作创建了新资源，则该操作完成后，状态终结点应返回状态代码 303（查看其他）。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-343">If the asynchronous operation creates a new resource, the status endpoint should return status code 303 (See Other) after the operation completes.</span></span> <span data-ttu-id="3fbbd-344">在 303 响应中，包含一个 Location 标头用于提供新资源的 URI：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-344">In the 303 response, include a Location header that gives the URI of the new resource:</span></span>

```http
HTTP/1.1 303 See Other
Location: /api/orders/12345
```

<span data-ttu-id="3fbbd-345">有关详细信息，请参阅 [REST 中的异步操作](https://www.adayinthelifeof.nl/2011/06/02/asynchronous-operations-in-rest/)。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-345">For more information, see [Asynchronous operations in REST](https://www.adayinthelifeof.nl/2011/06/02/asynchronous-operations-in-rest/).</span></span>

## <a name="filter-and-paginate-data"></a><span data-ttu-id="3fbbd-346">数据筛选和分页</span><span class="sxs-lookup"><span data-stu-id="3fbbd-346">Filter and paginate data</span></span>

<span data-ttu-id="3fbbd-347">通过单个 URI 公开资源的集合可能会导致应用程序在只需一部分信息时提取大量数据。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-347">Exposing a collection of resources through a single URI can lead to applications fetching large amounts of data when only a subset of the information is required.</span></span> <span data-ttu-id="3fbbd-348">例如，假设某个客户端应用程序需要查找成本超过特定值的所有订单。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-348">For example, suppose a client application needs to find all orders with a cost over a specific value.</span></span> <span data-ttu-id="3fbbd-349">它可以从 */orders* URI 检索所有订单，然后在客户端筛选这些订单。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-349">It might retrieve all orders from the */orders* URI and then filter these orders on the client side.</span></span> <span data-ttu-id="3fbbd-350">显然，此过程的效率非常低下。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-350">Clearly this process is highly inefficient.</span></span> <span data-ttu-id="3fbbd-351">它浪费了托管 Web API 的服务器的网络带宽和处理能力。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-351">It wastes network bandwidth and processing power on the server hosting the web API.</span></span>

<span data-ttu-id="3fbbd-352">API 可以允许在 URI 的查询字符串中传递筛选器，例如 */orders?minCost=n*。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-352">Instead, the API can allow passing a filter in the query string of the URI, such as */orders?minCost=n*.</span></span> <span data-ttu-id="3fbbd-353">然后，Web API 负责分析和处理查询字符串中的 `minCost` 参数并在服务器端返回筛选后的结果。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-353">The web API is then responsible for parsing and handling the `minCost` parameter in the query string and returning the filtered results on the sever side.</span></span> 

<span data-ttu-id="3fbbd-354">对集合资源执行的 GET 请求可能返回大量的项。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-354">GET requests over collection resources can potentially return a large number of items.</span></span> <span data-ttu-id="3fbbd-355">应将 Web API 设计为限制任何单个请求返回的数据量。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-355">You should design a web API to limit the amount of data returned by any single request.</span></span> <span data-ttu-id="3fbbd-356">请考虑支持查询字符串指定要检索的最大项数和集合中的起始偏移量。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-356">Consider supporting query strings that specify the maximum number of items to retrieve and a starting offset into the collection.</span></span> <span data-ttu-id="3fbbd-357">例如：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-357">For example:</span></span>

```
/orders?limit=25&offset=50
```

<span data-ttu-id="3fbbd-358">此外，请考虑对返回的项数实施上限，以帮助防止拒绝服务攻击。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-358">Also consider imposing an upper limit on the number of items returned, to help prevent Denial of Service attacks.</span></span> <span data-ttu-id="3fbbd-359">若要帮助客户端应用程序，返回分页数据的 GET 请求还应包含某种形式的元数据，以指示集合中可用的资源总数。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-359">To assist client applications, GET requests that return paginated data should also include some form of metadata that indicate the total number of resources available in the collection.</span></span> <span data-ttu-id="3fbbd-360">也许还可以考虑其他智能分页策略；有关详细信息，请参阅 [API Design Notes: Smart Paging](http://bizcoder.com/api-design-notes-smart-paging)（API 设计说明：智能分页）</span><span class="sxs-lookup"><span data-stu-id="3fbbd-360">You might also consider other intelligent paging strategies; for more information, see [API Design Notes: Smart Paging](http://bizcoder.com/api-design-notes-smart-paging)</span></span>

<span data-ttu-id="3fbbd-361">可以通过提供一个将字段名称用作值的 soft 参数（例如 */orders?sort=ProductID*），使用类似的策略对提取的数据排序。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-361">You can use a similar strategy to sort data as it is fetched, by providing a sort parameter that takes a field name as the value, such as */orders?sort=ProductID*.</span></span> <span data-ttu-id="3fbbd-362">但是，此方法会对缓存产生负面影响，因为查询字符串参数构成许多缓存实现用作缓存数据的键的资源标识符的一部分。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-362">However, this approach can have a negative effect on caching, because query string parameters form part of the resource identifier used by many cache implementations as the key to cached data.</span></span>

<span data-ttu-id="3fbbd-363">如果每个项包含大量数据，可以扩展此方法来限制针对每个项返回的字段。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-363">You can extend this approach to limit the fields returned for each item, if each item contains a large amount of data.</span></span> <span data-ttu-id="3fbbd-364">例如，可以使用接受以逗号分隔的字段列表的查询字符串参数，例如 */orders?fields=ProductID,Quantity*。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-364">For example, you could use a query string parameter that accepts a comma-delimited list of fields, such as */orders?fields=ProductID,Quantity*.</span></span> 

<span data-ttu-id="3fbbd-365">为查询字符串中的所有可选参数提供有意义的默认值。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-365">Give all optional parameters in query strings meaningful defaults.</span></span> <span data-ttu-id="3fbbd-366">例如，如果实现分页，将 `limit` 参数设为 10，将 `offset` 参数设为 0；如果实现排序，将排序参数设为资源的键；如果支持投影，将 `fields` 参数设为资源中的所有字段。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-366">For example, set the `limit` parameter to 10 and the `offset` parameter to 0 if you implement pagination, set the sort parameter to the key of the resource if you implement ordering, and set the `fields` parameter to all fields in the resource if you support projections.</span></span>

## <a name="support-partial-responses-for-large-binary-resources"></a><span data-ttu-id="3fbbd-367">支持大型二进制资源的部分响应</span><span class="sxs-lookup"><span data-stu-id="3fbbd-367">Support partial responses for large binary resources</span></span>

<span data-ttu-id="3fbbd-368">资源可能包含大型二进制字段，例如文件或图像。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-368">A resource may contain large binary fields, such as files or images.</span></span> <span data-ttu-id="3fbbd-369">若要解决不可靠和间歇性连接导致的问题并缩短响应时间，请考虑分块检索此类资源。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-369">To overcome problems caused by unreliable and intermittent connections and to improve response times, consider enabling such resources to be retrieved in chunks.</span></span> <span data-ttu-id="3fbbd-370">为此，对于针对大型资源发出的 GET 请求，Web API 应支持 Accept-Ranges 标头。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-370">To do this, the web API should support the Accept-Ranges header for GET requests for large resources.</span></span> <span data-ttu-id="3fbbd-371">此标头指示 GET 操作支持“部分”请求。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-371">This header indicates that the GET operation supports partial requests.</span></span> <span data-ttu-id="3fbbd-372">客户端应用程序可以提交返回指定为字节范围的资源子集的 GET 请求。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-372">The client application can submit GET requests that return a subset of a resource, specified as a range of bytes.</span></span> 

<span data-ttu-id="3fbbd-373">此外，请考虑对这些资源实现 HTTP HEAD 请求。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-373">Also, consider implementing HTTP HEAD requests for these resources.</span></span> <span data-ttu-id="3fbbd-374">HEAD 请求与 GET 请求类似，不过，前者只返回描述资源的 HTTP 标头和空消息正文。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-374">A HEAD request is similar to a GET request, except that it only returns the HTTP headers that describe the resource, with an empty message body.</span></span> <span data-ttu-id="3fbbd-375">客户端应用程序可以发出 HEAD 请求以确定是否要通过使用部分 GET 请求获取某个资源。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-375">A client application can issue a HEAD request to determine whether to fetch a resource by using partial GET requests.</span></span> <span data-ttu-id="3fbbd-376">例如：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-376">For example:</span></span>

```HTTP
HEAD http://adventure-works.com/products/10?fields=productImage HTTP/1.1
```

<span data-ttu-id="3fbbd-377">下面是响应消息的示例：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-377">Here is an example response message:</span></span> 

```HTTP
HTTP/1.1 200 OK

Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 4580
```

<span data-ttu-id="3fbbd-378">Content-Length 标头指定资源的总大小，Accept-Ranges 标头指示相应的 GET 操作支持部分结果。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-378">The Content-Length header gives the total size of the resource, and the Accept-Ranges header indicates that the corresponding GET operation supports partial results.</span></span> <span data-ttu-id="3fbbd-379">客户端应用程序可以使用此信息以较小的区块检索图像。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-379">The client application can use this information to retrieve the image in smaller chunks.</span></span> <span data-ttu-id="3fbbd-380">第一个请求通过使用 Range 标头提取前 2500 个字节：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-380">The first request fetches the first 2500 bytes by using the Range header:</span></span>

```HTTP
GET http://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=0-2499
```

<span data-ttu-id="3fbbd-381">响应消息通过返回 HTTP 状态代码 206 指示这是部分响应。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-381">The response message indicates that this is a partial response by returning HTTP status code 206.</span></span> <span data-ttu-id="3fbbd-382">Content-Length 标头指定消息正文中返回的实际字节数（不是资源的大小），Content-Range 标头指示这是该资源的哪一部分（第 0 到 2499 字节，总共 4580 个字节）:</span><span class="sxs-lookup"><span data-stu-id="3fbbd-382">The Content-Length header specifies the actual number of bytes returned in the message body (not the size of the resource), and the Content-Range header indicates which part of the resource this is (bytes 0-2499 out of 4580):</span></span>

```HTTP
HTTP/1.1 206 Partial Content

Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2500
Content-Range: bytes 0-2499/4580

[...]
```

<span data-ttu-id="3fbbd-383">来自客户端应用程序的后续请求可以检索资源的剩余部分。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-383">A subsequent request from the client application can retrieve the remainder of the resource.</span></span>

## <a name="use-hateoas-to-enable-navigation-to-related-resources"></a><span data-ttu-id="3fbbd-384">使用 HATEOAS 导航到相关资源</span><span class="sxs-lookup"><span data-stu-id="3fbbd-384">Use HATEOAS to enable navigation to related resources</span></span>

<span data-ttu-id="3fbbd-385">REST 背后的主要动机之一是它应能够导航整个资源集，而无需事先了解 URI 方案。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-385">One of the primary motivations behind REST is that it should be possible to navigate the entire set of resources without requiring prior knowledge of the URI scheme.</span></span> <span data-ttu-id="3fbbd-386">每个 HTTP GET 请求应通过响应中包含的超链接返回查找与所请求的对象直接相关的资源所需的信息，还应为它提供描述其中每个资源提供的操作的信息。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-386">Each HTTP GET request should return the information necessary to find the resources related directly to the requested object through hyperlinks included in the response, and it should also be provided with information that describes the operations available on each of these resources.</span></span> <span data-ttu-id="3fbbd-387">此原则称为 HATEOAS 或作为应用程序状态引擎的超文本。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-387">This principle is known as HATEOAS, or Hypertext as the Engine of Application State.</span></span> <span data-ttu-id="3fbbd-388">该系统实际上是有限状态机，每个请求的响应包含从一种状态转为另一种状态所需的信息；任何其他信息都不应是必需的。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-388">The system is effectively a finite state machine, and the response to each request contains the information necessary to move from one state to another; no other information should be necessary.</span></span>

> [!NOTE]
> <span data-ttu-id="3fbbd-389">当前没有任何标准或规范定义如何为 HATEOAS 原则建模。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-389">Currently there are no standards or specifications that define how to model the HATEOAS principle.</span></span> <span data-ttu-id="3fbbd-390">此节中所示的示例说明了一个可能的解决方案。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-390">The examples shown in this section illustrate one possible solution.</span></span>
>
>

<span data-ttu-id="3fbbd-391">例如，若要处理订单与客户之间的关系，可以在订单的表示形式中包含链接，用于指定下单客户可以执行的操作。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-391">For example, to handle the relationship between an order and a customer, the representation of an order could include links that identify the available operations for the customer of the order.</span></span> <span data-ttu-id="3fbbd-392">下面是可能的表示形式：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-392">Here is a possible representation:</span></span> 

```json
{
  "orderID":3,
  "productID":2,
  "quantity":4,
  "orderValue":16.60,
  "links":[
    {
      "rel":"customer",
      "href":"http://adventure-works.com/customers/3", 
      "action":"GET",
      "types":["text/xml","application/json"] 
    },
    {
      "rel":"customer",
      "href":"http://adventure-works.com/customers/3", 
      "action":"PUT",
      "types":["application/x-www-form-urlencoded"]
    },
    {
      "rel":"customer",
      "href":"http://adventure-works.com/customers/3",
      "action":"DELETE",
      "types":[]
    },
    {
      "rel":"self",
      "href":"http://adventure-works.com/orders/3", 
      "action":"GET",
      "types":["text/xml","application/json"]
    },
    {
      "rel":"self",
      "href":"http://adventure-works.com/orders/3", 
      "action":"PUT",
      "types":["application/x-www-form-urlencoded"]
    },
    {
      "rel":"self",
      "href":"http://adventure-works.com/orders/3", 
      "action":"DELETE",
      "types":[]
    }]
}
```

<span data-ttu-id="3fbbd-393">在此示例中，`links` 数组包含一组链接。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-393">In this example, the `links` array has a set of links.</span></span> <span data-ttu-id="3fbbd-394">每个链接表示可对相关实体执行的操作。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-394">Each link represents an operation on a related entity.</span></span> <span data-ttu-id="3fbbd-395">每个链接的数据包含关系 ("customer")、URI (`http://adventure-works.com/customers/3`)、HTTP 方法和支持的 MIME 类型。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-395">The data for each link includes the relationship ("customer"), the URI (`http://adventure-works.com/customers/3`), the HTTP method, and the supported MIME types.</span></span> <span data-ttu-id="3fbbd-396">这是客户端应用程序在调用操作时所需的全部信息。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-396">This is all the information that a client application needs to be able to invoke the operation.</span></span> 

<span data-ttu-id="3fbbd-397">`links` 数组还包含有关已检索的资源本身的自引用信息。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-397">The `links` array also includes self-referencing information about the resource itself that has been retrieved.</span></span> <span data-ttu-id="3fbbd-398">这些链接包含关系 *self*。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-398">These have the relationship *self*.</span></span>

<span data-ttu-id="3fbbd-399">返回的链接集可能会根据资源的状态发生更改。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-399">The set of links that are returned may change, depending on the state of the resource.</span></span> <span data-ttu-id="3fbbd-400">这就是为何将超文本称作“应用程序状态引擎”的原因。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-400">This is what is meant by hypertext being the "engine of application state."</span></span>

## <a name="versioning-a-restful-web-api"></a><span data-ttu-id="3fbbd-401">对 RESTful Web API 进行版本控制</span><span class="sxs-lookup"><span data-stu-id="3fbbd-401">Versioning a RESTful web API</span></span>

<span data-ttu-id="3fbbd-402">Web API 一直保持静态的可能性很小。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-402">It is highly unlikely that a web API will remain static.</span></span> <span data-ttu-id="3fbbd-403">随着业务需求变化，可能会添加新的资源集合，资源之间的关系可能会更改，并且可能会修改资源中的数据结构。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-403">As business requirements change new collections of resources may be added, the relationships between resources might change, and the structure of the data in resources might be amended.</span></span> <span data-ttu-id="3fbbd-404">虽然更新 Web API 以处理新的或不同的需求是一个相对简单的过程，但你必须考虑此类更改将对使用该 Web API 的客户端应用程序造成的影响。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-404">While updating a web API to handle new or differing requirements is a relatively straightforward process, you must consider the effects that such changes will have on client applications consuming the web API.</span></span> <span data-ttu-id="3fbbd-405">问题在于尽管设计和实现 Web API 的开发人员可以完全控制该 API，但开发人员对客户端应用程序不具有相同程度的控制，因为这些客户端应用程序可能是由远程运营的第三方组织生成的。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-405">The issue is that although the developer designing and implementing a web API has full control over that API, the developer does not have the same degree of control over client applications which may be built by third party organizations operating remotely.</span></span> <span data-ttu-id="3fbbd-406">主要规则是要让现有客户端应用程序能够继续不变地正常运行，同时允许新客户端应用程序利用新功能和新资源。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-406">The primary imperative is to enable existing client applications to continue functioning unchanged while allowing new client applications to take advantage of new features and resources.</span></span>

<span data-ttu-id="3fbbd-407">版本控制使 Web API 可以指定它所公开的功能和资源，并且客户端应用程序可以提交定向到特定版本的功能或资源的请求。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-407">Versioning enables a web API to indicate the features and resources that it exposes, and a client application can submit requests that are directed to a specific version of a feature or resource.</span></span> <span data-ttu-id="3fbbd-408">以下各节介绍几种不同的方法，其中每一种方法都有其自己的优势和不足。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-408">The following sections describe several different approaches, each of which has its own benefits and trade-offs.</span></span>

### <a name="no-versioning"></a><span data-ttu-id="3fbbd-409">无版本控制</span><span class="sxs-lookup"><span data-stu-id="3fbbd-409">No versioning</span></span>
<span data-ttu-id="3fbbd-410">这是最简单的方法，它对于一些内部 API 来说可能是可以接受的。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-410">This is the simplest approach, and may be acceptable for some internal APIs.</span></span> <span data-ttu-id="3fbbd-411">较大的更改可以表示为新资源或新链接。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-411">Big changes could be represented as new resources or new links.</span></span>  <span data-ttu-id="3fbbd-412">向现有资源添加内容可能未呈现重大更改，因为不应查看此内容的客户端应用程序将直接忽略它。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-412">Adding content to existing resources might not present a breaking change as client applications that are not expecting to see this content will simply ignore it.</span></span>

<span data-ttu-id="3fbbd-413">例如，向 URI *http://adventure-works.com/customers/3* 发出请求应返回包含客户端应用程序所需的 `id`、`name` 和 `address` 字段的单个客户的详细信息：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-413">For example, a request to the URI *http://adventure-works.com/customers/3* should return the details of a single customer containing `id`, `name`, and `address` fields expected by the client application:</span></span>

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

> [!NOTE]
> <span data-ttu-id="3fbbd-414">为简单起见，本部分中所示的示例响应不包含 HATEOAS 链接。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-414">For simplicity, the example responses shown in this section do not include HATEOAS links.</span></span>
>
>

<span data-ttu-id="3fbbd-415">如果 `DateCreated` 字段已添加到客户资源的架构中，则响应将如下所示：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-415">If the `DateCreated` field is added to the schema of the customer resource, then the response would look like this:</span></span>

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":"1 Microsoft Way Redmond WA 98053"}
```

<span data-ttu-id="3fbbd-416">现有的客户端应用程序可能会继续正常工作（如果能够忽略无法识别的字段），而新的客户端应用程序则可以设计为处理该新字段。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-416">Existing client applications might continue functioning correctly if they are capable of ignoring unrecognized fields, while new client applications can be designed to handle this new field.</span></span> <span data-ttu-id="3fbbd-417">但是，如果对资源的架构进行了更根本的更改（如删除或重命名字段）或资源之间的关系发生更改，则这些更改可能构成重大更改，从而阻止现有客户端应用程序正常工作。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-417">However, if more radical changes to the schema of resources occur (such as removing or renaming fields) or the relationships between resources change then these may constitute breaking changes that prevent existing client applications from functioning correctly.</span></span> <span data-ttu-id="3fbbd-418">在这些情况下应考虑以下方法之一。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-418">In these situations you should consider one of the following approaches.</span></span>

### <a name="uri-versioning"></a><span data-ttu-id="3fbbd-419">URI 版本控制</span><span class="sxs-lookup"><span data-stu-id="3fbbd-419">URI versioning</span></span>
<span data-ttu-id="3fbbd-420">每次修改 Web API 或更改资源的架构时，向每个资源的 URI 添加版本号。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-420">Each time you modify the web API or change the schema of resources, you add a version number to the URI for each resource.</span></span> <span data-ttu-id="3fbbd-421">以前存在的 URI 应像以前一样继续运行，并返回符合原始架构的资源。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-421">The previously existing URIs should continue to operate as before, returning resources that conform to their original schema.</span></span>

<span data-ttu-id="3fbbd-422">继续前面的示例，如果将 `address` 字段重构为包含地址的每个构成部分的子字段（例如 `streetAddress`、`city`、`state` 和 `zipCode`），则此版本的资源可通过包含版本号的 URI（如 http://adventure-works.com/v2/customers/3:）公开：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-422">Extending the previous example, if the `address` field is restructured into sub-fields containing each constituent part of the address (such as `streetAddress`, `city`, `state`, and `zipCode`), this version of the resource could be exposed through a URI containing a version number, such as http://adventure-works.com/v2/customers/3:</span></span>

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

<span data-ttu-id="3fbbd-423">此版本控制机制非常简单，但依赖于将请求路由到相应终结点的服务器。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-423">This versioning mechanism is very simple but depends on the server routing the request to the appropriate endpoint.</span></span> <span data-ttu-id="3fbbd-424">但是，随着 Web API 经过多次迭代而变得成熟，服务器必须支持多个不同版本，它可能变得难以处理。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-424">However, it can become unwieldy as the web API matures through several iterations and the server has to support a number of different versions.</span></span> <span data-ttu-id="3fbbd-425">此外，从单纯的角度来看，在所有情况下客户端应用程序都要提取相同数据（客户 3），因此 URI 实在不应该因版本而有所不同。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-425">Also, from a purist’s point of view, in all cases the client applications are fetching the same data (customer 3), so the URI should not really be different depending on the version.</span></span> <span data-ttu-id="3fbbd-426">此方案也增加了 HATEOAS 实现的复杂性，因为所有链接都需要在其 URI 中包括版本号。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-426">This scheme also complicates implementation of HATEOAS as all links will need to include the version number in their URIs.</span></span>

### <a name="query-string-versioning"></a><span data-ttu-id="3fbbd-427">查询字符串版本控制</span><span class="sxs-lookup"><span data-stu-id="3fbbd-427">Query string versioning</span></span>
<span data-ttu-id="3fbbd-428">不是提供多个 URI，而是可以通过在追加到 HTTP 请求后面的查询字符串中使用参数来指定资源的版本，例如 *http://adventure-works.com/customers/3?version=2*。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-428">Rather than providing multiple URIs, you can specify the version of the resource by using a parameter within the query string appended to the HTTP request, such as *http://adventure-works.com/customers/3?version=2*.</span></span> <span data-ttu-id="3fbbd-429">如果 version 参数被较旧的客户端应用程序省略，则应默认为有意义的值（例如 1）。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-429">The version parameter should default to a meaningful value such as 1 if it is omitted by older client applications.</span></span>

<span data-ttu-id="3fbbd-430">此方法具有语义优势（即，同一资源始终从同一 URI 进行检索），但它依赖于代码处理请求以分析查询字符串并发送回相应的 HTTP 响应。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-430">This approach has the semantic advantage that the same resource is always retrieved from the same URI, but it depends on the code that handles the request to parse the query string and send back the appropriate HTTP response.</span></span> <span data-ttu-id="3fbbd-431">此方法也与 URI 版本控制机制一样，增加了实现 HATEOAS 的复杂性。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-431">This approach also suffers from the same complications for implementing HATEOAS as the URI versioning mechanism.</span></span>

> [!NOTE]
> <span data-ttu-id="3fbbd-432">某些较旧的 Web 浏览器和 Web 代理不会缓存在 URI 中包含查询字符串的请求的响应。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-432">Some older web browsers and web proxies will not cache responses for requests that include a query string in the URI.</span></span> <span data-ttu-id="3fbbd-433">这可能会对使用 Web API 的 Web 应用程序以及从此类 Web 浏览器运行的 Web 应用程序的性能产生不利影响。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-433">This can have an adverse impact on performance for web applications that use a web API and that run from within such a web browser.</span></span>
>
>

### <a name="header-versioning"></a><span data-ttu-id="3fbbd-434">标头版本控制</span><span class="sxs-lookup"><span data-stu-id="3fbbd-434">Header versioning</span></span>
<span data-ttu-id="3fbbd-435">不是追加版本号作为查询字符串参数，而是可以实现指示资源的版本的自定义标头。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-435">Rather than appending the version number as a query string parameter, you could implement a custom header that indicates the version of the resource.</span></span> <span data-ttu-id="3fbbd-436">此方法需要客户端应用程序将相应标头添加到所有请求，虽然如果省略了版本标头，处理客户端请求的代码可以使用默认值（版本 1）。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-436">This approach requires that the client application adds the appropriate header to any requests, although the code handling the client request could use a default value (version 1) if the version header is omitted.</span></span> <span data-ttu-id="3fbbd-437">下面的示例利用了名为 *Custom-Header* 的自定义标头。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-437">The following examples utilize a custom header named *Custom-Header*.</span></span> <span data-ttu-id="3fbbd-438">此标头的值指示 Web API 的版本。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-438">The value of this header indicates the version of web API.</span></span>

<span data-ttu-id="3fbbd-439">版本 1：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-439">Version 1:</span></span>

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
Custom-Header: api-version=1
```

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

<span data-ttu-id="3fbbd-440">版本 2：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-440">Version 2:</span></span>

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
Custom-Header: api-version=2
```

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

<span data-ttu-id="3fbbd-441">请注意，与前面两个方法一样，实现 HATEOAS 需要在任何链接中包括相应的自定义标头。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-441">Note that as with the previous two approaches, implementing HATEOAS requires including the appropriate custom header in any links.</span></span>

### <a name="media-type-versioning"></a><span data-ttu-id="3fbbd-442">媒体类型版本控制</span><span class="sxs-lookup"><span data-stu-id="3fbbd-442">Media type versioning</span></span>
<span data-ttu-id="3fbbd-443">如本指南前面所述，当客户端应用程序向 Web 服务器发送 HTTP GET 请求时，它应使用 Accept 标头规定它可以处理的内容的格式。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-443">When a client application sends an HTTP GET request to a web server it should stipulate the format of the content that it can handle by using an Accept header, as described earlier in this guidance.</span></span> <span data-ttu-id="3fbbd-444">通常，*Accept* 标头的用途是允许客户端应用程序指定响应的正文应是 XML、JSON 还是客户端可以分析的其他某种常见格式。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-444">Frequently the purpose of the *Accept* header is to allow the client application to specify whether the body of the response should be XML, JSON, or some other common format that the client can parse.</span></span> <span data-ttu-id="3fbbd-445">但是，可以定义包括以下信息的自定义媒体类型：该信息使客户端应用程序可以指示它所需的资源版本。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-445">However, it is possible to define custom media types that include information enabling the client application to indicate which version of a resource it is expecting.</span></span> <span data-ttu-id="3fbbd-446">下面的示例演示了将 *Accept* 标头指定为值 *application/vnd.adventure-works.v1+json* 的请求。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-446">The following example shows a request that specifies an *Accept* header with the value *application/vnd.adventure-works.v1+json*.</span></span> <span data-ttu-id="3fbbd-447">*vnd.adventure-works.v1* 元素向 Web 服务器指示它应返回资源的版本 1，而 *json* 元素则指定响应正文的格式应为 JSON：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-447">The *vnd.adventure-works.v1* element indicates to the web server that it should return version 1 of the resource, while the *json* element specifies that the format of the response body should be JSON:</span></span>

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
Accept: application/vnd.adventure-works.v1+json
```

<span data-ttu-id="3fbbd-448">处理请求的代码负责处理 *Accept* 标头并尽可能采用该值（客户端应用程序可以在 *Accept* 标头中指定多种格式，在这种情况下，Web 服务器可以在其中选择最适合的格式用于响应正文）。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-448">The code handling the request is responsible for processing the *Accept* header and honoring it as far as possible (the client application may specify multiple formats in the *Accept* header, in which case the web server can choose the most appropriate format for the response body).</span></span> <span data-ttu-id="3fbbd-449">Web 服务器使用 Content-Type 标头确认响应正文中的数据格式：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-449">The web server confirms the format of the data in the response body by using the Content-Type header:</span></span>

```HTTP
HTTP/1.1 200 OK
Content-Type: application/vnd.adventure-works.v1+json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

<span data-ttu-id="3fbbd-450">如果 Accept 标头未指定任何已知的媒体类型，则 Web 服务器可以生成 HTTP 406（不可接受）响应消息或返回使用默认媒体类型的消息。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-450">If the Accept header does not specify any known media types, the web server could generate an HTTP 406 (Not Acceptable) response message or return a message with a default media type.</span></span>

<span data-ttu-id="3fbbd-451">此方法可以说是最纯粹的版本控制机制并自然地适用于 HATEOAS，后者可以在资源链接中包含相关数据的 MIME 类型。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-451">This approach is arguably the purest of the versioning mechanisms and lends itself naturally to HATEOAS, which can include the MIME type of related data in resource links.</span></span>

> [!NOTE]
> <span data-ttu-id="3fbbd-452">选择版本控制策略时，还应考虑对性能的影响，尤其是在 Web 服务器上缓存时。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-452">When you select a versioning strategy, you should also consider the implications on performance, especially caching on the web server.</span></span> <span data-ttu-id="3fbbd-453">URI 版本控制和查询字符串版本控制方案都是缓存友好的，因为同一 URI/查询字符串组合每次都指向相同的数据。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-453">The URI versioning and Query String versioning schemes are cache-friendly inasmuch as the same URI/query string combination refers to the same data each time.</span></span>
>
> <span data-ttu-id="3fbbd-454">标头版本控制和媒体类型版本控制机制通常需要其他逻辑来检查自定义标头或 Accept 标头中的值。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-454">The Header versioning and Media Type versioning mechanisms typically require additional logic to examine the values in the custom header or the Accept header.</span></span> <span data-ttu-id="3fbbd-455">在大型环境中，使用不同版本的 Web API 的多个客户端可能会在服务器端缓存中生成大量重复数据。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-455">In a large-scale environment, many clients using different versions of a web API can result in a significant amount of duplicated data in a server-side cache.</span></span> <span data-ttu-id="3fbbd-456">如果客户端应用程序通过实现缓存的代理与 Web 服务器进行通信，并且该代理在当前未在其缓存中保留所请求数据的副本时，仅将请求转发到 Web 服务器，则此问题可能会变得很严重。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-456">This issue can become acute if a client application communicates with a web server through a proxy that implements caching, and that only forwards a request to the web server if it does not currently hold a copy of the requested data in its cache.</span></span>
>
>

## <a name="open-api-initiative"></a><span data-ttu-id="3fbbd-457">Open API 计划</span><span class="sxs-lookup"><span data-stu-id="3fbbd-457">Open API Initiative</span></span>
<span data-ttu-id="3fbbd-458">[Open API 计划](https://www.openapis.org/)由一个行业协会创建，目的是标准化供应商的 REST API 说明。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-458">The [Open API Initiative](https://www.openapis.org/) was created by an industry consortium to standardize REST API descriptions across vendors.</span></span> <span data-ttu-id="3fbbd-459">作为该计划的一部分，Swagger 2.0 规范被重新命名为 OpenAPI 规范 (OAS)，并引入 Open API 计划。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-459">As part of this initiative, the Swagger 2.0 specification was renamed the OpenAPI Specification (OAS) and brought under the Open API Initiative.</span></span>

<span data-ttu-id="3fbbd-460">建议为 Web API 采用 OpenAPI。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-460">You may want to adopt OpenAPI for your web APIs.</span></span> <span data-ttu-id="3fbbd-461">考虑的要点：</span><span class="sxs-lookup"><span data-stu-id="3fbbd-461">Some points to consider:</span></span>

- <span data-ttu-id="3fbbd-462">OpenAPI 规范随附了一组有关如何设计 REST API 的强制性准则。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-462">The OpenAPI Specification comes with a set of opinionated guidelines on how a REST API should be designed.</span></span> <span data-ttu-id="3fbbd-463">这有益于互操作性，但在设计 API 时需多加注意，以符合规范。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-463">That has advantages for interoperability, but requires more care when designing your API to conform to the specification.</span></span>
- <span data-ttu-id="3fbbd-464">OpenAPI 首推协定优先的方法，而不是实现优先的方法。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-464">OpenAPI promotes a contract-first approach, rather than an implementation-first approach.</span></span> <span data-ttu-id="3fbbd-465">协定优先意味着首先设计 API 协定（接口），然后写入实现协定的代码。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-465">Contract-first means you design the API contract (the interface) first and then write code that implements the contract.</span></span> 
- <span data-ttu-id="3fbbd-466">Swagger 之类的工具可以从 API 协定生成客户端库或文档。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-466">Tools like Swagger can generate client libraries or documentation from API contracts.</span></span> <span data-ttu-id="3fbbd-467">有关示例，请参阅[有关使用 Swagger 的 ASP.NET Web API 帮助页](/aspnet/core/tutorials/web-api-help-pages-using-swagger)。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-467">For example, see [ASP.NET Web API Help Pages using Swagger](/aspnet/core/tutorials/web-api-help-pages-using-swagger).</span></span>

## <a name="more-information"></a><span data-ttu-id="3fbbd-468">详细信息</span><span class="sxs-lookup"><span data-stu-id="3fbbd-468">More information</span></span>
* <span data-ttu-id="3fbbd-469">[Microsoft REST API 准则](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md)。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-469">[Microsoft REST API Guidelines](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md).</span></span> <span data-ttu-id="3fbbd-470">有关设计公共 REST API 的详细建议。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-470">Detailed recommendations for designing public REST APIs.</span></span>
* <span data-ttu-id="3fbbd-471">[REST 指南](http://restcookbook.com/)。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-471">[The REST Cookbook](http://restcookbook.com/).</span></span> <span data-ttu-id="3fbbd-472">有关构建 RESTful API 的介绍。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-472">Introduction to building RESTful APIs.</span></span>
* <span data-ttu-id="3fbbd-473">[Web API 核对清单](https://mathieu.fenniak.net/the-api-checklist/)。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-473">[Web API Checklist](https://mathieu.fenniak.net/the-api-checklist/).</span></span> <span data-ttu-id="3fbbd-474">设计和实现 Web API 时要考虑的有用事项列表。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-474">A useful list of items to consider when designing and implementing a web API.</span></span>
* <span data-ttu-id="3fbbd-475">[开放式 API 计划](https://www.openapis.org/)。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-475">[Open API Initiative](https://www.openapis.org/).</span></span> <span data-ttu-id="3fbbd-476">有关开放式 API 的文档和实施详细信息。</span><span class="sxs-lookup"><span data-stu-id="3fbbd-476">Documentation and implementation details on Open API.</span></span>
