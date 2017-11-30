---
title: "API 实现指南"
description: "有关如何实现 API 的指南。"
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: b4d197719380bf55033942b3ebcad384170d950d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="api-implementation"></a><span data-ttu-id="f63b8-103">API 实现</span><span class="sxs-lookup"><span data-stu-id="f63b8-103">API implementation</span></span>
[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="f63b8-104">精心设计的 REST 样式 Web API 定义了客户端应用程序可以访问的资源、关系和导航方案。</span><span class="sxs-lookup"><span data-stu-id="f63b8-104">A carefully-designed RESTful web API defines the resources, relationships, and navigation schemes that are accessible to client applications.</span></span> <span data-ttu-id="f63b8-105">实现和部署 Web API 时，应考虑托管该 Web API 的环境的物理要求以及构造该 Web API 的方式，而不是考虑数据的逻辑结构。</span><span class="sxs-lookup"><span data-stu-id="f63b8-105">When you implement and deploy a web API, you should consider the physical requirements of the environment hosting the web API and the way in which the web API is constructed rather than the logical structure of the data.</span></span> <span data-ttu-id="f63b8-106">本指南重点介绍实现 Web API 并发布它，使其可供客户端应用程序使用的最佳实践。</span><span class="sxs-lookup"><span data-stu-id="f63b8-106">This guidance focusses on best practices for implementing a web API and publishing it to make it available to client applications.</span></span> <span data-ttu-id="f63b8-107">有关 Web API 设计的详细信息，请参阅 [API 设计指南](/azure/architecture/best-practices/api-design)。</span><span class="sxs-lookup"><span data-stu-id="f63b8-107">For detailed information about web API design, see [API Design Guidance](/azure/architecture/best-practices/api-design).</span></span>

## <a name="considerations-for-processing-requests"></a><span data-ttu-id="f63b8-108">有关处理请求的注意事项</span><span class="sxs-lookup"><span data-stu-id="f63b8-108">Considerations for processing requests</span></span>

<span data-ttu-id="f63b8-109">在实现处理请求的代码时，请考虑以下几点。</span><span class="sxs-lookup"><span data-stu-id="f63b8-109">Consider the following points when you implement the code to handle requests.</span></span>

### <a name="get-put-delete-head-and-patch-actions-should-be-idempotent"></a><span data-ttu-id="f63b8-110">GET、PUT、DELETE、HEAD 和 PATCH 操作应当是幂等的</span><span class="sxs-lookup"><span data-stu-id="f63b8-110">GET, PUT, DELETE, HEAD, and PATCH actions should be idempotent</span></span>

<span data-ttu-id="f63b8-111">实现这些请求的代码不应施加任何副作用。</span><span class="sxs-lookup"><span data-stu-id="f63b8-111">The code that implements these requests should not impose any side-effects.</span></span> <span data-ttu-id="f63b8-112">对同一资源重复进行同一请求应导致同一状态。</span><span class="sxs-lookup"><span data-stu-id="f63b8-112">The same request repeated over the same resource should result in the same state.</span></span> <span data-ttu-id="f63b8-113">例如，将多个 DELETE 请求发送到同一 URI 应产生相同的效果，尽管响应消息中的 HTTP 状态代码可能有所不同。</span><span class="sxs-lookup"><span data-stu-id="f63b8-113">For example, sending multiple DELETE requests to the same URI should have the same effect, although the HTTP status code in the response messages may be different.</span></span> <span data-ttu-id="f63b8-114">第一个 DELETE 请求可能返回状态代码 204（无内容），而接下来的 DELETE 请求则可能返回状态代码 404（未找到）。</span><span class="sxs-lookup"><span data-stu-id="f63b8-114">The first DELETE request might return status code 204 (No Content), while a subsequent DELETE request might return status code 404 (Not Found).</span></span>

> [!NOTE]
> <span data-ttu-id="f63b8-115">Jonathan Oliver 博客上的 [Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/)（幂等性模式）一文概述了幂等性以及它如何与数据管理操作相关。</span><span class="sxs-lookup"><span data-stu-id="f63b8-115">The article [Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) on Jonathan Oliver’s blog provides an overview of idempotency and how it relates to data management operations.</span></span>
>

### <a name="post-actions-that-create-new-resources-should-not-have-unrelated-side-effects"></a><span data-ttu-id="f63b8-116">创建新资源的 POST 操作不应具有不相关的副作用</span><span class="sxs-lookup"><span data-stu-id="f63b8-116">POST actions that create new resources should not have unrelated side-effects</span></span>

<span data-ttu-id="f63b8-117">如果 POST 请求用于创建新资源，则请求的作用应限制为新资源（以及任何可能直接相关的资源，如果涉及某种关联）。例如，在电子商务系统中，创建新客户订单的 POST 请求可能还修改库存水平并生成计费信息，但它不应修改与订单不直接相关的信息，也不应对系统的整体状态有任何其他副作用。</span><span class="sxs-lookup"><span data-stu-id="f63b8-117">If a POST request is intended to create a new resource, the effects of the request should be limited to the new resource (and possibly any directly related resources if there is some sort of linkage involved) For example, in an ecommerce system, a POST request that creates a new order for a customer might also amend inventory levels and generate billing information, but it should not modify information not directly related to the order or have any other side-effects on the overall state of the system.</span></span>

### <a name="avoid-implementing-chatty-post-put-and-delete-operations"></a><span data-ttu-id="f63b8-118">避免实现琐碎的 POST、PUT 和 DELETE 操作</span><span class="sxs-lookup"><span data-stu-id="f63b8-118">Avoid implementing chatty POST, PUT, and DELETE operations</span></span>

<span data-ttu-id="f63b8-119">支持对资源集合发出 POST、PUT 和 DELETE 请求。</span><span class="sxs-lookup"><span data-stu-id="f63b8-119">Support POST, PUT and DELETE requests over resource collections.</span></span> <span data-ttu-id="f63b8-120">POST 请求可以包含多个新资源的详细信息并将这些资源全都添加到同一个集合，PUT 请求可以替换集合中的整个资源集，DELETE 请求可以删除整个集合。</span><span class="sxs-lookup"><span data-stu-id="f63b8-120">A POST request can contain the details for multiple new resources and add them all to the same collection, a PUT request can replace the entire set of resources in a collection, and a DELETE request can remove an entire collection.</span></span>

<span data-ttu-id="f63b8-121">ASP.NET Web API 2 中包含的 OData 支持提供了成批处理请求的功能。</span><span class="sxs-lookup"><span data-stu-id="f63b8-121">The OData support included in ASP.NET Web API 2 provides the ability to batch requests.</span></span> <span data-ttu-id="f63b8-122">客户端应用程序可以打包多个 Web API 请求并通过单个 HTTP 请求将其发送给服务器，然后接收包含每个请求的答复的单个 HTTP 响应。</span><span class="sxs-lookup"><span data-stu-id="f63b8-122">A client application can package up several web API requests and send them to the server in a single HTTP request, and receive a single HTTP response that contains the replies to each request.</span></span> <span data-ttu-id="f63b8-123">有关详细信息，请参阅 [Introducing Batch Support in Web API and Web API OData](http://blogs.msdn.com/b/webdev/archive/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata.aspx)（Web API 和 Web API OData 中的批处理支持简介）。</span><span class="sxs-lookup"><span data-stu-id="f63b8-123">For more information, [Introducing Batch Support in Web API and Web API OData](http://blogs.msdn.com/b/webdev/archive/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata.aspx).</span></span>

### <a name="follow-the-http-specification-when-sending-a-response"></a><span data-ttu-id="f63b8-124">发送响应时请按照 HTTP 规范操作</span><span class="sxs-lookup"><span data-stu-id="f63b8-124">Follow the HTTP specification when sending a response</span></span> 

<span data-ttu-id="f63b8-125">Web API 必须返回包含正确 HTTP 状态代码的消息（使客户端可以确定如何处理结果）、相应的 HTTP 标头（以便客户端了解结果的性质），以及适当格式化的正文（使客户端可以分析结果）。</span><span class="sxs-lookup"><span data-stu-id="f63b8-125">A web API must return messages that contain the correct HTTP status code to enable the client to determine how to handle the result, the appropriate HTTP headers so that the client understands the nature of the result, and a suitably formatted body to enable the client to parse the result.</span></span> 

<span data-ttu-id="f63b8-126">例如，POST 操作应返回状态代码 201（已创建），并且响应消息应在其 Location 标头中包含新创建的资源的 URI。</span><span class="sxs-lookup"><span data-stu-id="f63b8-126">For example, a POST operation should return status code 201 (Created) and the response message should include the URI of the newly created resource in the Location header of the response message.</span></span>

### <a name="support-content-negotiation"></a><span data-ttu-id="f63b8-127">支持内容协商</span><span class="sxs-lookup"><span data-stu-id="f63b8-127">Support content negotiation</span></span>

<span data-ttu-id="f63b8-128">响应消息的正文可能包含不同格式的数据。</span><span class="sxs-lookup"><span data-stu-id="f63b8-128">The body of a response message may contain data in a variety of formats.</span></span> <span data-ttu-id="f63b8-129">例如，HTTP GET 请求可以返回 JSON 或 XML 格式的数据。</span><span class="sxs-lookup"><span data-stu-id="f63b8-129">For example, an HTTP GET request could return data in JSON, or XML format.</span></span> <span data-ttu-id="f63b8-130">客户端提交请求时，它可以包括指定它可以处理的数据格式的 Accept 标头。</span><span class="sxs-lookup"><span data-stu-id="f63b8-130">When the client submits a request, it can include an Accept header that specifies the data formats that it can handle.</span></span> <span data-ttu-id="f63b8-131">这些格式将指定为媒体类型。</span><span class="sxs-lookup"><span data-stu-id="f63b8-131">These formats are specified as media types.</span></span> <span data-ttu-id="f63b8-132">例如，发出用于检索图像的 GET 请求的客户端可以指定 Accept 标头，其中列出客户端可以处理的媒体类型，如“image/jpeg, image/gif, image/png”。</span><span class="sxs-lookup"><span data-stu-id="f63b8-132">For example, a client that issues a GET request that retrieves an image can specify an Accept header that lists the media types that the client can handle, such as "image/jpeg, image/gif, image/png".</span></span>  <span data-ttu-id="f63b8-133">当 Web API 返回结果时，它应使用其中一种媒体类型设置数据格式，并在响应的 Content-Type 标头中指定该格式。</span><span class="sxs-lookup"><span data-stu-id="f63b8-133">When the web API returns the result, it should format the data by using one of these media types and specify the format in the Content-Type header of the response.</span></span>

<span data-ttu-id="f63b8-134">如果客户端未指定 Accept 标头，则对响应正文使用有意义的默认格式。</span><span class="sxs-lookup"><span data-stu-id="f63b8-134">If the client does not specify an Accept header, then use a sensible default format for the response body.</span></span> <span data-ttu-id="f63b8-135">例如，ASP.NET Web API 框架对基于文本的数据默认使用 JSON 格式。</span><span class="sxs-lookup"><span data-stu-id="f63b8-135">As an example, the ASP.NET Web API framework defaults to JSON for text-based data.</span></span>

### <a name="provide-links-to-support-hateoas-style-navigation-and-discovery-of-resources"></a><span data-ttu-id="f63b8-136">提供支持 HATEOAS 样式导航和资源发现的链接</span><span class="sxs-lookup"><span data-stu-id="f63b8-136">Provide links to support HATEOAS-style navigation and discovery of resources</span></span>

<span data-ttu-id="f63b8-137">HATEOAS 方法使客户端能够从初始的起点导航和发现资源。</span><span class="sxs-lookup"><span data-stu-id="f63b8-137">The HATEOAS approach enables a client to navigate and discover resources from an initial starting point.</span></span> <span data-ttu-id="f63b8-138">这通过使用包含 URI 的链接来实现；当客户端发出 HTTP GET 请求以获取资源时，响应应包含使客户端应用程序能够快速找到任何直接相关的资源的 URI。</span><span class="sxs-lookup"><span data-stu-id="f63b8-138">This is achieved by using links containing URIs; when a client issues an HTTP GET request to obtain a resource, the response should contain URIs that enable a client application to quickly locate any directly related resources.</span></span> <span data-ttu-id="f63b8-139">例如，在支持电子商务解决方案的 Web API 中，客户可能下了多个订单。</span><span class="sxs-lookup"><span data-stu-id="f63b8-139">For example, in a web API that supports an e-commerce solution, a customer may have placed many orders.</span></span> <span data-ttu-id="f63b8-140">当客户端应用程序检索客户的详细信息时，响应中应该包含使客户端应用程序能够发送可检索这些订单的 HTTP GET 请求的链接。</span><span class="sxs-lookup"><span data-stu-id="f63b8-140">When a client application retrieves the details for a customer, the response should include links that enable the client application to send HTTP GET requests that can retrieve these orders.</span></span> <span data-ttu-id="f63b8-141">此外，HATEOAS 样式的链接应描述每个链接的资源支持的其他操作（POST、PUT、DELETE 等）以及用于执行每个请求的相应 URI。</span><span class="sxs-lookup"><span data-stu-id="f63b8-141">Additionally, HATEOAS-style links should describe the other operations (POST, PUT, DELETE, and so on) that each linked resource supports together with the corresponding URI to perform each request.</span></span> <span data-ttu-id="f63b8-142">[API 设计][api-design]中对此方法进行了更详细的介绍。</span><span class="sxs-lookup"><span data-stu-id="f63b8-142">This approach is described in more detail in [API Design][api-design].</span></span>

<span data-ttu-id="f63b8-143">当前没有控制 HATEOAS 的实现的标准，但下面的示例说明了一种可行方法。</span><span class="sxs-lookup"><span data-stu-id="f63b8-143">Currently there are no standards that govern the implementation of HATEOAS, but the following example illustrates one possible approach.</span></span> <span data-ttu-id="f63b8-144">在此示例中，用于查找客户的详细信息的 HTTP GET 请求返回一个响应，其中包括引用该客户的订单的 HATEOAS 链接：</span><span class="sxs-lookup"><span data-stu-id="f63b8-144">In this example, an HTTP GET request that finds the details for a customer returns a response that include HATEOAS links that reference the orders for that customer:</span></span>

```HTTP
GET http://adventure-works.com/customers/2 HTTP/1.1
Accept: text/json
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"CustomerID":2,"CustomerName":"Bert","Links":[
    {"rel":"self",
    "href":"http://adventure-works.com/customers/2",
    "action":"GET",
    "types":["text/xml","application/json"]},
    {"rel":"self",
    "href":"http://adventure-works.com/customers/2",
    "action":"PUT",
    "types":["application/x-www-form-urlencoded"]},
    {"rel":"self",
    "href":"http://adventure-works.com/customers/2",
    "action":"DELETE",
    "types":[]},
    {"rel":"orders",
    "href":"http://adventure-works.com/customers/2/orders",
    "action":"GET",
    "types":["text/xml","application/json"]},
    {"rel":"orders",
    "href":"http://adventure-works.com/customers/2/orders",
    "action":"POST",
    "types":["application/x-www-form-urlencoded"]}
]}
```

<span data-ttu-id="f63b8-145">在此示例中，客户数据由以下代码片段中所示的 `Customer` 类表示。</span><span class="sxs-lookup"><span data-stu-id="f63b8-145">In this example, the customer data is represented by the `Customer` class shown in the following code snippet.</span></span> <span data-ttu-id="f63b8-146">HATEOAS 链接在 `Links` 集合属性中保存：</span><span class="sxs-lookup"><span data-stu-id="f63b8-146">The HATEOAS links are held in the `Links` collection property:</span></span>

```csharp
public class Customer
{
    public int CustomerID { get; set; }
    public string CustomerName { get; set; }
    public List<Link> Links { get; set; }
    ...
}

public class Link
{
    public string Rel { get; set; }
    public string Href { get; set; }
    public string Action { get; set; }
    public string [] Types { get; set; }
}
```

<span data-ttu-id="f63b8-147">HTTP GET 操作从存储中检索客户数据并构造 `Customer` 对象，并填充 `Links` 集合。</span><span class="sxs-lookup"><span data-stu-id="f63b8-147">The HTTP GET operation retrieves the customer data from storage and constructs a `Customer` object, and then populates the `Links` collection.</span></span> <span data-ttu-id="f63b8-148">结果将格式化为 JSON 响应消息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-148">The result is formatted as a JSON response message.</span></span> <span data-ttu-id="f63b8-149">每个链接包含以下字段：</span><span class="sxs-lookup"><span data-stu-id="f63b8-149">Each link comprises the following fields:</span></span>

* <span data-ttu-id="f63b8-150">返回的对象与链接所描述的对象之间的关系。</span><span class="sxs-lookup"><span data-stu-id="f63b8-150">The relationship between the object being returned and the object described by the link.</span></span> <span data-ttu-id="f63b8-151">在此示例中，“self”指示该链接是返回到对象本身的引用（类似于许多面向对象语言中的 `this` 指针），而“orders”则是包含相关订单信息的集合的名称。</span><span class="sxs-lookup"><span data-stu-id="f63b8-151">In this case "self" indicates that the link is a reference back to the object itself (similar to a `this` pointer in many object-oriented languages), and "orders" is the name of a collection containing the related order information.</span></span>
* <span data-ttu-id="f63b8-152">URI 形式的链接所描述的对象的超链接 (`Href`)。</span><span class="sxs-lookup"><span data-stu-id="f63b8-152">The hyperlink (`Href`) for the object being described by the link in the form of a URI.</span></span>
* <span data-ttu-id="f63b8-153">可发送到此 URI 的 HTTP 请求的类型 (`Action`)。</span><span class="sxs-lookup"><span data-stu-id="f63b8-153">The type of HTTP request (`Action`) that can be sent to this URI.</span></span>
* <span data-ttu-id="f63b8-154">应在 HTTP 请求中提供或可在响应中返回的任何数据的格式 (`Types`)，具体取决于请求的类型。</span><span class="sxs-lookup"><span data-stu-id="f63b8-154">The format of any data (`Types`) that should be provided in the HTTP request or that can be returned in the response, depending on the type of the request.</span></span>

<span data-ttu-id="f63b8-155">HTTP 响应的示例中所示的 HATEOAS 链接指示客户端应用程序可以执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="f63b8-155">The HATEOAS links shown in the example HTTP response indicate that a client application can perform the following operations:</span></span>

* <span data-ttu-id="f63b8-156">向 URI `http://adventure-works.com/customers/2` 发出 HTTP GET 请求以提取客户的详细信息（再次）。</span><span class="sxs-lookup"><span data-stu-id="f63b8-156">An HTTP GET request to the URI `http://adventure-works.com/customers/2` to fetch the details of the customer (again).</span></span> <span data-ttu-id="f63b8-157">数据可以 XML 或 JSON 格式返回。</span><span class="sxs-lookup"><span data-stu-id="f63b8-157">The data can be returned as XML or JSON.</span></span>
* <span data-ttu-id="f63b8-158">向 URI `http://adventure-works.com/customers/2` 发出 HTTP PUT 请求以修改客户的详细信息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-158">An HTTP PUT request to the URI `http://adventure-works.com/customers/2` to modify the details of the customer.</span></span> <span data-ttu-id="f63b8-159">必须在请求消息中以 x-www-form-urlencoded 格式提供新数据。</span><span class="sxs-lookup"><span data-stu-id="f63b8-159">The new data must be provided in the request message in x-www-form-urlencoded format.</span></span>
* <span data-ttu-id="f63b8-160">向 URI `http://adventure-works.com/customers/2` 发出 HTTP DELETE 请求以删除客户。</span><span class="sxs-lookup"><span data-stu-id="f63b8-160">An HTTP DELETE request to the URI `http://adventure-works.com/customers/2` to delete the customer.</span></span> <span data-ttu-id="f63b8-161">该请求不需要任何其他信息，也不需要在响应消息正文中返回数据。</span><span class="sxs-lookup"><span data-stu-id="f63b8-161">The request does not expect any additional information or return data in the response message body.</span></span>
* <span data-ttu-id="f63b8-162">向 URI `http://adventure-works.com/customers/2/orders` 发出 HTTP GET 请求以查找客户的所有订单。</span><span class="sxs-lookup"><span data-stu-id="f63b8-162">An HTTP GET request to the URI `http://adventure-works.com/customers/2/orders` to find all the orders for the customer.</span></span> <span data-ttu-id="f63b8-163">数据可以 XML 或 JSON 格式返回。</span><span class="sxs-lookup"><span data-stu-id="f63b8-163">The data can be returned as XML or JSON.</span></span>
* <span data-ttu-id="f63b8-164">向 URI `http://adventure-works.com/customers/2/orders` 发出 HTTP PUT 请求以为此客户创建新订单。</span><span class="sxs-lookup"><span data-stu-id="f63b8-164">An HTTP PUT request to the URI `http://adventure-works.com/customers/2/orders` to create a new order for this customer.</span></span> <span data-ttu-id="f63b8-165">必须在请求消息中以 x-www-form-urlencoded 格式提供数据。</span><span class="sxs-lookup"><span data-stu-id="f63b8-165">The data must be provided in the request message in x-www-form-urlencoded format.</span></span>

## <a name="considerations-for-handling-exceptions"></a><span data-ttu-id="f63b8-166">有关处理异常的注意事项</span><span class="sxs-lookup"><span data-stu-id="f63b8-166">Considerations for handling exceptions</span></span>

<span data-ttu-id="f63b8-167">如果操作引发未捕获的异常，请考虑以下几点。</span><span class="sxs-lookup"><span data-stu-id="f63b8-167">Consider the following points if an operation throws an uncaught exception.</span></span>

### <a name="capture-exceptions-and-return-a-meaningful-response-to-clients"></a><span data-ttu-id="f63b8-168">捕获异常并向客户端返回有意义的响应</span><span class="sxs-lookup"><span data-stu-id="f63b8-168">Capture exceptions and return a meaningful response to clients</span></span>

<span data-ttu-id="f63b8-169">实现 HTTP 操作的代码应提供全面的异常处理，而不是让未捕获的异常传播到框架。</span><span class="sxs-lookup"><span data-stu-id="f63b8-169">The code that implements an HTTP operation should provide comprehensive exception handling rather than letting uncaught exceptions propagate to the framework.</span></span> <span data-ttu-id="f63b8-170">如果异常会导致无法成功完成此操作，则可以在响应消息中传递回此异常，但它应包括导致异常的错误的有意义描述。</span><span class="sxs-lookup"><span data-stu-id="f63b8-170">If an exception makes it impossible to complete the operation successfully, the exception can be passed back in the response message, but it should include a meaningful description of the error that caused the exception.</span></span> <span data-ttu-id="f63b8-171">该异常还应包括相应的 HTTP 状态代码，而不是对于每种情况都只返回状态代码 500。</span><span class="sxs-lookup"><span data-stu-id="f63b8-171">The exception should also include the appropriate HTTP status code rather than simply returning status code 500 for every situation.</span></span> <span data-ttu-id="f63b8-172">例如，如果用户请求导致了违反约束的数据库更新（例如，尝试删除具有未完成订单的客户），则应返回状态代码 409（冲突）和指示冲突原因的消息正文。</span><span class="sxs-lookup"><span data-stu-id="f63b8-172">For example, if a user request causes a database update that violates a constraint (such as attempting to delete a customer that has outstanding orders), you should return status code 409 (Conflict) and a message body indicating the reason for the conflict.</span></span> <span data-ttu-id="f63b8-173">如果某种其他情况显示请求无法完成，则可以返回状态代码 400（错误请求）。</span><span class="sxs-lookup"><span data-stu-id="f63b8-173">If some other condition renders the request unachievable, you can return status code 400 (Bad Request).</span></span> <span data-ttu-id="f63b8-174">可以在 W3C 网站上的 [Status Code Definitions](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)（状态代码定义）页中找到 HTTP 状态代码的完整列表。</span><span class="sxs-lookup"><span data-stu-id="f63b8-174">You can find a full list of HTTP status codes on the [Status Code Definitions](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) page on the W3C website.</span></span>

<span data-ttu-id="f63b8-175">代码示例捕获不同条件并返回相应响应。</span><span class="sxs-lookup"><span data-stu-id="f63b8-175">The code example traps different conditions and returns an appropriate response.</span></span>

```csharp
[HttpDelete]
[Route("customers/{id:int}")]
public IHttpActionResult DeleteCustomer(int id)
{
    try
    {
        // Find the customer to be deleted in the repository
        var customerToDelete = repository.GetCustomer(id);

        // If there is no such customer, return an error response
        // with status code 404 (Not Found)
        if (customerToDelete == null)
        {
                return NotFound();
        }

        // Remove the customer from the repository
        // The DeleteCustomer method returns true if the customer
        // was successfully deleted
        if (repository.DeleteCustomer(id))
        {
            // Return a response message with status code 204 (No Content)
            // To indicate that the operation was successful
            return StatusCode(HttpStatusCode.NoContent);
        }
        else
        {
            // Otherwise return a 400 (Bad Request) error response
            return BadRequest(Strings.CustomerNotDeleted);
        }
    }
    catch
    {
        // If an uncaught exception occurs, return an error response
        // with status code 500 (Internal Server Error)
        return InternalServerError();
    }
}
```

> [!TIP]
> <span data-ttu-id="f63b8-176">请勿包括可能对尝试入侵 API 的攻击者有用的信息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-176">Do not include information that could be useful to an attacker attempting to penetrate your API.</span></span>
  
<span data-ttu-id="f63b8-177">许多 Web 服务器在错误条件到达 Web API 之前，自行捕获错误条件。</span><span class="sxs-lookup"><span data-stu-id="f63b8-177">Many web servers trap error conditions themselves before they reach the web API.</span></span> <span data-ttu-id="f63b8-178">例如，如果为网站配置了身份验证，但用户无法提供正确的身份验证信息，则 Web 服务器应以状态代码 401（未经授权）进行响应。</span><span class="sxs-lookup"><span data-stu-id="f63b8-178">For example, if you configure authentication for a web site and the user fails to provide the correct authentication information, the web server should respond with status code 401 (Unauthorized).</span></span> <span data-ttu-id="f63b8-179">客户端经过身份验证后，代码可以执行自己的检查来验证客户端是否应能够访问所请求的资源。</span><span class="sxs-lookup"><span data-stu-id="f63b8-179">Once a client has been authenticated, your code can perform its own checks to verify that the client should be able access the requested resource.</span></span> <span data-ttu-id="f63b8-180">如果此授权失败，则应返回状态代码 403（禁止访问）。</span><span class="sxs-lookup"><span data-stu-id="f63b8-180">If this authorization fails, you should return status code 403 (Forbidden).</span></span>
 
### <a name="handle-exceptions-consistently-and-log-information-about-errors"></a><span data-ttu-id="f63b8-181">一致地处理异常，并记录有关错误的信息</span><span class="sxs-lookup"><span data-stu-id="f63b8-181">Handle exceptions consistently and log information about errors</span></span>

<span data-ttu-id="f63b8-182">若要以一致方式处理异常，请考虑在整个 Web API 中实现全局错误处理策略。</span><span class="sxs-lookup"><span data-stu-id="f63b8-182">To handle exceptions in a consistent manner, consider implementing a global error handling strategy across the entire web API.</span></span> <span data-ttu-id="f63b8-183">还应将整合捕获每个异常的完整详细信息的错误日志记录；只要客户端无法通过 Web 访问它，此错误日志就会包含详细的信息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-183">You should also incorporate error logging which captures the full details of each exception; this error log can contain detailed information as long as it is not made accessible over the web to clients.</span></span> 

### <a name="distinguish-between-client-side-errors-and-server-side-errors"></a><span data-ttu-id="f63b8-184">区分客户端错误和服务器端错误</span><span class="sxs-lookup"><span data-stu-id="f63b8-184">Distinguish between client-side errors and server-side errors</span></span>

<span data-ttu-id="f63b8-185">HTTP 协议可区分因客户端应用程序发生的错误（HTTP 4xx 状态代码）和因服务器上的事故导致的错误（HTTP 5xx 状态代码）。</span><span class="sxs-lookup"><span data-stu-id="f63b8-185">The HTTP protocol distinguishes between errors that occur due to the client application (the HTTP 4xx status codes), and errors that are caused by a mishap on the server (the HTTP 5xx status codes).</span></span> <span data-ttu-id="f63b8-186">请确保在任何错误响应消息中遵守此约定。</span><span class="sxs-lookup"><span data-stu-id="f63b8-186">Make sure that you respect this convention in any error response messages.</span></span>

## <a name="considerations-for-optimizing-client-side-data-access"></a><span data-ttu-id="f63b8-187">有关优化客户端数据访问的注意事项</span><span class="sxs-lookup"><span data-stu-id="f63b8-187">Considerations for optimizing client-side data access</span></span>
<span data-ttu-id="f63b8-188">例如，在分布式环境（例如，涉及 Web 服务器和客户端应用程序）中，主要问题源之一是网络。</span><span class="sxs-lookup"><span data-stu-id="f63b8-188">In a distributed environment such as that involving a web server and client applications, one of the primary sources of concern is the network.</span></span> <span data-ttu-id="f63b8-189">这可能表现为值得注意的瓶颈问题，尤其当客户端应用程序频繁地将发送请求或接收数据时。</span><span class="sxs-lookup"><span data-stu-id="f63b8-189">This can act as a considerable bottleneck, especially if a client application is frequently sending requests or receiving data.</span></span> <span data-ttu-id="f63b8-190">因此，目标应是将网络间流动的通信量降到最低。</span><span class="sxs-lookup"><span data-stu-id="f63b8-190">Therefore you should aim to minimize the amount of traffic that flows across the network.</span></span> <span data-ttu-id="f63b8-191">在实现检索和维护数据的代码时，请考虑以下几点：</span><span class="sxs-lookup"><span data-stu-id="f63b8-191">Consider the following points when you implement the code to retrieve and maintain data:</span></span>

### <a name="support-client-side-caching"></a><span data-ttu-id="f63b8-192">支持客户端缓存</span><span class="sxs-lookup"><span data-stu-id="f63b8-192">Support client-side caching</span></span>

<span data-ttu-id="f63b8-193">HTTP 1.1 协议支持在客户端和中间服务器中缓存，请求通过这些客户端和服务器使用 Cache-Control 标头进行路由。</span><span class="sxs-lookup"><span data-stu-id="f63b8-193">The HTTP 1.1 protocol supports caching in clients and intermediate servers through which a request is routed by the use of the Cache-Control header.</span></span> <span data-ttu-id="f63b8-194">当客户端应用程序向 Web API 发送 HTTP GET 请求时，响应可以包含 Cache-Control 标头，以指示响应的正文中的数据是否可以由通过其路由该请求的客户端或中间服务器安全地缓存，以及多长时间之后应失效并视为过期。</span><span class="sxs-lookup"><span data-stu-id="f63b8-194">When a client application sends an HTTP GET request to the web API, the response can include a Cache-Control header that indicates whether the data in the body of the response can be safely cached by the client or an intermediate server through which the request has been routed, and for how long before it should expire and be considered out-of-date.</span></span> <span data-ttu-id="f63b8-195">下面的示例演示了 HTTP GET 请求和包含 Cache-Control 标头的相应响应：</span><span class="sxs-lookup"><span data-stu-id="f63b8-195">The following example shows an HTTP GET request and the corresponding response that includes a Cache-Control header:</span></span>

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
```

```HTTP
HTTP/1.1 200 OK
...
Cache-Control: max-age=600, private
Content-Type: text/json; charset=utf-8
Content-Length: ...
{"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
```

<span data-ttu-id="f63b8-196">在此示例中，Cache-Control 标头指定返回的数据应在 600 秒后过期并且只适用于单个客户端，不能在其他客户端使用的共享缓存中存储（它是*专用*的）。</span><span class="sxs-lookup"><span data-stu-id="f63b8-196">In this example, the Cache-Control header specifies that the data returned should be expired after 600 seconds, and is only suitable for a single client and must not be stored in a shared cache used by other clients (it is *private*).</span></span> <span data-ttu-id="f63b8-197">Cache-Control 标头可以指定 *public*（而不是 *private*），在这种情况下，数据可以存储在共享缓存中；它也可以指定 *no-store*，在这种情况下，数据**不**能由客户端缓存。</span><span class="sxs-lookup"><span data-stu-id="f63b8-197">The Cache-Control header could specify *public* rather than *private* in which case the data can be stored in a shared cache, or it could specify *no-store* in which case the data must **not** be cached by the client.</span></span> <span data-ttu-id="f63b8-198">下面的代码示例显示如何在响应消息中构造 Cache-Control 标头：</span><span class="sxs-lookup"><span data-stu-id="f63b8-198">The following code example shows how to construct a Cache-Control header in a response message:</span></span>

```csharp
public class OrdersController : ApiController
{
    ...
    [Route("api/orders/{id:int:min(0)}")]
    [HttpGet]
    public IHttpActionResult FindOrderByID(int id)
    {
        // Find the matching order
        Order order = ...;
        ...
        // Create a Cache-Control header for the response
        var cacheControlHeader = new CacheControlHeaderValue();
        cacheControlHeader.Private = true;
        cacheControlHeader.MaxAge = new TimeSpan(0, 10, 0);
        ...

        // Return a response message containing the order and the cache control header
        OkResultWithCaching<Order> response = new OkResultWithCaching<Order>(order, this)
        {
            CacheControlHeader = cacheControlHeader
        };
        return response;
    }
    ...
}
```

<span data-ttu-id="f63b8-199">此代码使用名为 `OkResultWithCaching` 的自定义 `IHttpActionResult` 类。</span><span class="sxs-lookup"><span data-stu-id="f63b8-199">This code makes use of a custom `IHttpActionResult` class named `OkResultWithCaching`.</span></span> <span data-ttu-id="f63b8-200">此类使控制器可设置缓存标头内容：</span><span class="sxs-lookup"><span data-stu-id="f63b8-200">This class enables the controller to set the cache header contents:</span></span>

```csharp
public class OkResultWithCaching<T> : OkNegotiatedContentResult<T>
{
    public OkResultWithCaching(T content, ApiController controller)
        : base(content, controller) { }

    public OkResultWithCaching(T content, IContentNegotiator contentNegotiator, HttpRequestMessage request, IEnumerable<MediaTypeFormatter> formatters)
        : base(content, contentNegotiator, request, formatters) { }

    public CacheControlHeaderValue CacheControlHeader { get; set; }
    public EntityTagHeaderValue ETag { get; set; }

    public override async Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
    {
        HttpResponseMessage response;
        try
        {
            response = await base.ExecuteAsync(cancellationToken);
            response.Headers.CacheControl = this.CacheControlHeader;
            response.Headers.ETag = ETag;
        }
        catch (OperationCanceledException)
        {
            response = new HttpResponseMessage(HttpStatusCode.Conflict) {ReasonPhrase = "Operation was cancelled"};
        }
        return response;
    }
}
```

> [!NOTE]
> <span data-ttu-id="f63b8-201">HTTP 协议还为 Cache-Control 标头定义了 *no-cache* 指令。</span><span class="sxs-lookup"><span data-stu-id="f63b8-201">The HTTP protocol also defines the *no-cache* directive for the Cache-Control header.</span></span> <span data-ttu-id="f63b8-202">令人困惑的是，此指令并不意味着“不缓存”而是指示“在返回信息之前使用服务器重新验证缓存的信息”；数据仍可以缓存，但在每次使用它时检查以确保它仍是最新的。</span><span class="sxs-lookup"><span data-stu-id="f63b8-202">Rather confusingly, this directive does not mean "do not cache" but rather "revalidate the cached information with the server before returning it"; the data can still be cached, but it is checked each time it is used to ensure that it is still current.</span></span>
>
>

<span data-ttu-id="f63b8-203">缓存管理是客户端应用程序或中间服务器的职责，但如果正确实现它，可以节省带宽和提高性能，因为它可以消除提取最近已检索的数据的需要。</span><span class="sxs-lookup"><span data-stu-id="f63b8-203">Cache management is the responsibility of the client application or intermediate server, but if properly implemented it can save bandwidth and improve performance by removing the need to fetch data that has already been recently retrieved.</span></span>

<span data-ttu-id="f63b8-204">Cache-Control 标头中的 *max-age* 值只是指南，并不保证相应的数据在指定的时间内不会更改。</span><span class="sxs-lookup"><span data-stu-id="f63b8-204">The *max-age* value in the Cache-Control header is only a guide and not a guarantee that the corresponding data won't change during the specified time.</span></span> <span data-ttu-id="f63b8-205">Web API 应将 max-age 设置为适当的值，具体取决于数据的预期波动性。</span><span class="sxs-lookup"><span data-stu-id="f63b8-205">The web API should set the max-age to a suitable value depending on the expected volatility of the data.</span></span> <span data-ttu-id="f63b8-206">此期间到期后，客户端应放弃缓存中的对象。</span><span class="sxs-lookup"><span data-stu-id="f63b8-206">When this period expires, the client should discard the object from the cache.</span></span>

> [!NOTE]
> <span data-ttu-id="f63b8-207">如前所述，大多数现代 Web 浏览器通过向请求添加相应的 Cache-Control 标头并检查结果的标头来支持客户端缓存。</span><span class="sxs-lookup"><span data-stu-id="f63b8-207">Most modern web browsers support client-side caching by adding the appropriate cache-control headers to requests and examining the headers of the results, as described.</span></span> <span data-ttu-id="f63b8-208">但是，某些较旧的浏览器将不缓存从包含查询字符串的 URL 返回的值。</span><span class="sxs-lookup"><span data-stu-id="f63b8-208">However, some older browsers will not cache the values returned from a URL that includes a query string.</span></span> <span data-ttu-id="f63b8-209">这通常不是基于此处讨论的协议实现自己的缓存管理策略的自定义客户端应用程序的问题。</span><span class="sxs-lookup"><span data-stu-id="f63b8-209">This is not usually an issue for custom client applications which implement their own cache management strategy based on the protocol discussed here.</span></span>
>
> <span data-ttu-id="f63b8-210">某些较旧的代理显示相同的行为并可能不会基于包含查询字符串的 URL 缓存请求。</span><span class="sxs-lookup"><span data-stu-id="f63b8-210">Some older proxies exhibit the same behavior and might not cache requests based on URLs with query strings.</span></span> <span data-ttu-id="f63b8-211">这可能是通过此类代理连接到 Web 服务器的自定义客户端应用程序的问题。</span><span class="sxs-lookup"><span data-stu-id="f63b8-211">This could be an issue for custom client applications that connect to a web server through such a proxy.</span></span>
>

### <a name="provide-etags-to-optimize-query-processing"></a><span data-ttu-id="f63b8-212">提供 ETag 以优化查询处理</span><span class="sxs-lookup"><span data-stu-id="f63b8-212">Provide ETags to optimize query processing</span></span>

<span data-ttu-id="f63b8-213">当客户端应用程序检索对象时，响应消息还可以包括 *ETag*（实体标记）。</span><span class="sxs-lookup"><span data-stu-id="f63b8-213">When a client application retrieves an object, the response message can also include an *ETag* (Entity Tag).</span></span> <span data-ttu-id="f63b8-214">ETag 是不透明的字符串，它指示资源的版本，每次资源发生更改时，也会修改 Etag。</span><span class="sxs-lookup"><span data-stu-id="f63b8-214">An ETag is an opaque string that indicates the version of a resource; each time a resource changes the Etag is also modified.</span></span> <span data-ttu-id="f63b8-215">客户端应用程序应将此 ETag 作为数据的一部分缓存。</span><span class="sxs-lookup"><span data-stu-id="f63b8-215">This ETag should be cached as part of the data by the client application.</span></span> <span data-ttu-id="f63b8-216">下面的代码示例演示如何添加 ETag 作为 HTTP GET 请求的响应的一部分。</span><span class="sxs-lookup"><span data-stu-id="f63b8-216">The following code example shows how to add an ETag as part of the response to an HTTP GET request.</span></span> <span data-ttu-id="f63b8-217">此代码使用对象的 `GetHashCode` 方法生成用于标识对象的数字值（如有必要，可以使用 MD5 等算法重写此方法并生成自己的哈希值）：</span><span class="sxs-lookup"><span data-stu-id="f63b8-217">This code uses the `GetHashCode` method of an object to generate a numeric value that identifies the object (you can override this method if necessary and generate your own hash using an algorithm such as MD5) :</span></span>

```csharp
public class OrdersController : ApiController
{
    ...
    public IHttpActionResult FindOrderByID(int id)
    {
        // Find the matching order
        Order order = ...;
        ...

        var hashedOrder = order.GetHashCode();
        string hashedOrderEtag = $"\"{hashedOrder}\"";
        var eTag = new EntityTagHeaderValue(hashedOrderEtag);

        // Return a response message containing the order and the cache control header
        OkResultWithCaching<Order> response = new OkResultWithCaching<Order>(order, this)
        {
            ...,
            ETag = eTag
        };
        return response;
    }
    ...
}
```

<span data-ttu-id="f63b8-218">Web API 发布的响应消息如下所示：</span><span class="sxs-lookup"><span data-stu-id="f63b8-218">The response message posted by the web API looks like this:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Cache-Control: max-age=600, private
Content-Type: text/json; charset=utf-8
ETag: "2147483648"
Content-Length: ...
{"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
```

> [!TIP]
> <span data-ttu-id="f63b8-219">出于安全原因，不允许缓存敏感数据或通过经过身份验证 (HTTPS) 的连接返回的数据。</span><span class="sxs-lookup"><span data-stu-id="f63b8-219">For security reasons, do not allow sensitive data or data returned over an authenticated (HTTPS) connection to be cached.</span></span>
>
>

<span data-ttu-id="f63b8-220">客户端应用程序随时可以发出后续 GET 请求以检索同一资源，并且如果资源已更改（它具有不同的 ETag），则应放弃缓存的版本，并将新版本添加到缓存中。</span><span class="sxs-lookup"><span data-stu-id="f63b8-220">A client application can issue a subsequent GET request to retrieve the same resource at any time, and if the resource has changed (it has a different ETag) the cached version should be discarded and the new version added to the cache.</span></span> <span data-ttu-id="f63b8-221">如果资源很大并且需要大量的带宽才能传输回客户端，则重复执行提取相同数据的请求可能会效率低下。</span><span class="sxs-lookup"><span data-stu-id="f63b8-221">If a resource is large and requires a significant amount of bandwidth to transmit back to the client, repeated requests to fetch the same data can become inefficient.</span></span> <span data-ttu-id="f63b8-222">为了应对这种情况，HTTP 协议定义了以下过程来优化应在 Web API 中支持的 GET 请求：</span><span class="sxs-lookup"><span data-stu-id="f63b8-222">To combat this, the HTTP protocol defines the following process for optimizing GET requests that you should support in a web API:</span></span>

* <span data-ttu-id="f63b8-223">客户端构造 GET 请求，该请求包含 If-None-Match HTTP 标头中引用的资源的当前缓存版本的 ETag：</span><span class="sxs-lookup"><span data-stu-id="f63b8-223">The client constructs a GET request containing the ETag for the currently cached version of the resource referenced in an If-None-Match HTTP header:</span></span>

    ```HTTP
    GET http://adventure-works.com/orders/2 HTTP/1.1
    If-None-Match: "2147483648"
    ```
* <span data-ttu-id="f63b8-224">Web API 中的 GET 操作获取所请求数据的当前 ETag（上面示例中的订单 2），并将其与 If-None-Match 标头中的值进行比较。</span><span class="sxs-lookup"><span data-stu-id="f63b8-224">The GET operation in the web API obtains the current ETag for the requested data (order 2 in the above example), and compares it to the value in the If-None-Match header.</span></span>
* <span data-ttu-id="f63b8-225">如果所请求数据的当前 ETag 与请求提供的 ETag 匹配，则资源尚未更改，Web API 应返回 HTTP 响应，其中包含空的消息正文和状态代码 304（未修改）。</span><span class="sxs-lookup"><span data-stu-id="f63b8-225">If the current ETag for the requested data matches the ETag provided by the request, the resource has not changed and the web API should return an HTTP response with an empty message body and a status code of 304 (Not Modified).</span></span>
* <span data-ttu-id="f63b8-226">如果所请求数据的当前 ETag 与请求提供的 ETag 不匹配，则数据已更改，Web API 应返回 HTTP 响应，其中包含带有新数据的消息正文和状态代码 200（正常）。</span><span class="sxs-lookup"><span data-stu-id="f63b8-226">If the current ETag for the requested data does not match the ETag provided by the request, then the data has changed and the web API should return an HTTP response with the new data in the message body and a status code of 200 (OK).</span></span>
* <span data-ttu-id="f63b8-227">如果所请求的数据不再存在，则 Web API 应返回状态代码为 404（未找到）的 HTTP 响应。</span><span class="sxs-lookup"><span data-stu-id="f63b8-227">If the requested data no longer exists then the web API should return an HTTP response with the status code of 404 (Not Found).</span></span>
* <span data-ttu-id="f63b8-228">客户端使用状态代码来维护缓存。</span><span class="sxs-lookup"><span data-stu-id="f63b8-228">The client uses the status code to maintain the cache.</span></span> <span data-ttu-id="f63b8-229">如果数据尚未更改（状态代码 304），则对象可保持缓存，并且客户端应用程序应继续使用此版本的对象。</span><span class="sxs-lookup"><span data-stu-id="f63b8-229">If the data has not changed (status code 304) then the object can remain cached and the client application should continue to use this version of the object.</span></span> <span data-ttu-id="f63b8-230">如果数据已更改（状态代码 200），则应放弃缓存的对象，并插入新对象。</span><span class="sxs-lookup"><span data-stu-id="f63b8-230">If the data has changed (status code 200) then the cached object should be discarded and the new one inserted.</span></span> <span data-ttu-id="f63b8-231">如果数据不再可用（状态代码 404），则应从缓存中删除该对象。</span><span class="sxs-lookup"><span data-stu-id="f63b8-231">If the data is no longer available (status code 404) then the object should be removed from the cache.</span></span>

> [!NOTE]
> <span data-ttu-id="f63b8-232">如果响应标头包含 Cache-Control 标头 no-store，则应始终从缓存中删除对象，而不考虑 HTTP 状态代码。</span><span class="sxs-lookup"><span data-stu-id="f63b8-232">If the response header contains the Cache-Control header no-store then the object should always be removed from the cache regardless of the HTTP status code.</span></span>
>

<span data-ttu-id="f63b8-233">下面的代码显示了已扩展为支持 If-None-Match 标头的 `FindOrderByID` 方法。</span><span class="sxs-lookup"><span data-stu-id="f63b8-233">The code below shows the `FindOrderByID` method extended to support the If-None-Match header.</span></span> <span data-ttu-id="f63b8-234">请注意，如果省略 If-None-Match 标头，则始终检索指定的订单：</span><span class="sxs-lookup"><span data-stu-id="f63b8-234">Notice that if the If-None-Match header is omitted, the specified order is always retrieved:</span></span>

```csharp
public class OrdersController : ApiController
{
    [Route("api/orders/{id:int:min(0)}")]
    [HttpGet]
    public IHttpActionResult FindOrderByID(int id)
    {
        try
        {
            // Find the matching order
            Order order = ...;

            // If there is no such order then return NotFound
            if (order == null)
            {
                return NotFound();
            }

            // Generate the ETag for the order
            var hashedOrder = order.GetHashCode();
            string hashedOrderEtag = $"\"{hashedOrder}\"";

            // Create the Cache-Control and ETag headers for the response
            IHttpActionResult response;
            var cacheControlHeader = new CacheControlHeaderValue();
            cacheControlHeader.Public = true;
            cacheControlHeader.MaxAge = new TimeSpan(0, 10, 0);
            var eTag = new EntityTagHeaderValue(hashedOrderEtag);

            // Retrieve the If-None-Match header from the request (if it exists)
            var nonMatchEtags = Request.Headers.IfNoneMatch;

            // If there is an ETag in the If-None-Match header and
            // this ETag matches that of the order just retrieved,
            // then create a Not Modified response message
            if (nonMatchEtags.Count > 0 &&
                String.CompareOrdinal(nonMatchEtags.First().Tag, hashedOrderEtag) == 0)
            {
                response = new EmptyResultWithCaching()
                {
                    StatusCode = HttpStatusCode.NotModified,
                    CacheControlHeader = cacheControlHeader,
                    ETag = eTag
                };
            }
            // Otherwise create a response message that contains the order details
            else
            {
                response = new OkResultWithCaching<Order>(order, this)
                {
                    CacheControlHeader = cacheControlHeader,
                    ETag = eTag
                };
            }

            return response;
        }
        catch
        {
            return InternalServerError();
        }
    }
...
}
```

<span data-ttu-id="f63b8-235">此示例引入了名为 `EmptyResultWithCaching` 的附加自定义 `IHttpActionResult` 类。</span><span class="sxs-lookup"><span data-stu-id="f63b8-235">This example incorporates an additional custom `IHttpActionResult` class named `EmptyResultWithCaching`.</span></span> <span data-ttu-id="f63b8-236">此类只充当不包含响应正文的 `HttpResponseMessage` 对象的包装：</span><span class="sxs-lookup"><span data-stu-id="f63b8-236">This class simply acts as a wrapper around an `HttpResponseMessage` object that does not contain a response body:</span></span>

```csharp
public class EmptyResultWithCaching : IHttpActionResult
{
    public CacheControlHeaderValue CacheControlHeader { get; set; }
    public EntityTagHeaderValue ETag { get; set; }
    public HttpStatusCode StatusCode { get; set; }
    public Uri Location { get; set; }

    public async Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
    {
        HttpResponseMessage response = new HttpResponseMessage(StatusCode);
        response.Headers.CacheControl = this.CacheControlHeader;
        response.Headers.ETag = this.ETag;
        response.Headers.Location = this.Location;
        return response;
    }
}
```

> [!TIP]
> <span data-ttu-id="f63b8-237">在此示例中，通过对从基础数据源检索到的数据进行哈希处理生成数据的 ETag。</span><span class="sxs-lookup"><span data-stu-id="f63b8-237">In this example, the ETag for the data is generated by hashing the data retrieved from the underlying data source.</span></span> <span data-ttu-id="f63b8-238">如果 ETag 可以某种其他方式计算，则可以进一步优化此过程，并且仅在数据已更改时，才需要从数据源中提取数据。</span><span class="sxs-lookup"><span data-stu-id="f63b8-238">If the ETag can be computed in some other way, then the process can be optimized further and the data only needs to be fetched from the data source if it has changed.</span></span>  <span data-ttu-id="f63b8-239">如果数据很大或访问数据源可能导致显著延迟（例如，如果数据源是远程数据库），则此方法特别有用。</span><span class="sxs-lookup"><span data-stu-id="f63b8-239">This approach is especially useful if the data is large or accessing the data source can result in significant latency (for example, if the data source is a remote database).</span></span>
>

### <a name="use-etags-to-support-optimistic-concurrency"></a><span data-ttu-id="f63b8-240">使用 ETag 支持乐观并发</span><span class="sxs-lookup"><span data-stu-id="f63b8-240">Use ETags to Support Optimistic Concurrency</span></span>

<span data-ttu-id="f63b8-241">为了启用对以前缓存的数据更新，HTTP 协议支持乐观并发策略。</span><span class="sxs-lookup"><span data-stu-id="f63b8-241">To enable updates over previously cached data, the HTTP protocol supports an optimistic concurrency strategy.</span></span> <span data-ttu-id="f63b8-242">如果在提取和缓存资源后，客户端应用程序随后发送 PUT 或 DELETE 请求以更改或删除资源，则应包含引用 ETag 的 If-match 标头。</span><span class="sxs-lookup"><span data-stu-id="f63b8-242">If, after fetching and caching a resource, the client application subsequently sends a PUT or DELETE request to change or remove the resource, it should include in If-Match header that references the ETag.</span></span> <span data-ttu-id="f63b8-243">然后，Web API 可以使用此信息来确定该资源在检索到后是否已被其他用户更改，并将相应响应发送回客户端应用程序，如下所示：</span><span class="sxs-lookup"><span data-stu-id="f63b8-243">The web API can then use this information to determine whether the resource has already been changed by another user since it was retrieved and send an appropriate response back to the client application as follows:</span></span>

* <span data-ttu-id="f63b8-244">客户端构造 PUT 请求，该请求包含资源的新详细信息，以及 If-Match HTTP 标头中引用的资源的当前缓存版本的 ETag。</span><span class="sxs-lookup"><span data-stu-id="f63b8-244">The client constructs a PUT request containing the new details for the resource and the ETag for the currently cached version of the resource referenced in an If-Match HTTP header.</span></span> <span data-ttu-id="f63b8-245">下面的示例演示了用于更新订单的 PUT 请求：</span><span class="sxs-lookup"><span data-stu-id="f63b8-245">The following example shows a PUT request that updates an order:</span></span>

    ```HTTP
    PUT http://adventure-works.com/orders/1 HTTP/1.1
    If-Match: "2282343857"
    Content-Type: application/x-www-form-urlencoded
    Content-Length: ...
    productID=3&quantity=5&orderValue=250
    ```
* <span data-ttu-id="f63b8-246">Web API 中的 PUT 操作获取所请求数据的当前 ETag（上面示例中的订单 1），并将其与 If-Match 标头中的值进行比较。</span><span class="sxs-lookup"><span data-stu-id="f63b8-246">The PUT operation in the web API obtains the current ETag for the requested data (order 1 in the above example), and compares it to the value in the If-Match header.</span></span>
* <span data-ttu-id="f63b8-247">如果所请求数据的当前 ETag 与请求提供的 ETag 匹配，则资源尚未未更改并且 Web API 应执行更新，并在成功的情况下返回包含 HTTP 状态代码 204（无内容）的消息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-247">If the current ETag for the requested data matches the ETag provided by the request, the resource has not changed and the web API should perform the update, returning a message with HTTP status code 204 (No Content) if it is successful.</span></span> <span data-ttu-id="f63b8-248">响应可以包括资源的更新后版本的 Cache-Control 和 ETag 标头。</span><span class="sxs-lookup"><span data-stu-id="f63b8-248">The response can include Cache-Control and ETag headers for the updated version of the resource.</span></span> <span data-ttu-id="f63b8-249">响应中应始终包含引用新更新资源的 URI 的 Location 标头。</span><span class="sxs-lookup"><span data-stu-id="f63b8-249">The response should always include the Location header that references the URI of the newly updated resource.</span></span>
* <span data-ttu-id="f63b8-250">如果所请求数据的当前 ETag 与请求提供的 ETag 不匹配，则数据在提取后已被其他用户更改，Web API 应返回包含空的消息正文和状态代码 412（不满足前提条件）的 HTTP 响应。</span><span class="sxs-lookup"><span data-stu-id="f63b8-250">If the current ETag for the requested data does not match the ETag provided by the request, then the data has been changed by another user since it was fetched and the web API should return an HTTP response with an empty message body and a status code of 412 (Precondition Failed).</span></span>
* <span data-ttu-id="f63b8-251">如果要更新的资源不再存在，则 Web API 应返回状态代码为 404（未找到）的 HTTP 响应。</span><span class="sxs-lookup"><span data-stu-id="f63b8-251">If the resource to be updated no longer exists then the web API should return an HTTP response with the status code of 404 (Not Found).</span></span>
* <span data-ttu-id="f63b8-252">客户端使用状态代码和响应标头来维护缓存。</span><span class="sxs-lookup"><span data-stu-id="f63b8-252">The client uses the status code and response headers to maintain the cache.</span></span> <span data-ttu-id="f63b8-253">如果数据已更新（状态代码 204），则对象可保持缓存（只要 Cache-Control 标头未指定 no-store），但应更新 ETag。</span><span class="sxs-lookup"><span data-stu-id="f63b8-253">If the data has been updated (status code 204) then the object can remain cached (as long as the Cache-Control header does not specify no-store) but the ETag should be updated.</span></span> <span data-ttu-id="f63b8-254">如果数据已被其他用户更改（状态代码 412）或未找到（状态代码 404），则应放弃缓存的对象。</span><span class="sxs-lookup"><span data-stu-id="f63b8-254">If the data was changed by another user changed (status code 412) or not found (status code 404) then the cached object should be discarded.</span></span>

<span data-ttu-id="f63b8-255">下一个代码示例显示了 Orders 控制器的 PUT 操作的实现：</span><span class="sxs-lookup"><span data-stu-id="f63b8-255">The next code example shows an implementation of the PUT operation for the Orders controller:</span></span>

```csharp
public class OrdersController : ApiController
{
    [HttpPut]
    [Route("api/orders/{id:int}")]
    public IHttpActionResult UpdateExistingOrder(int id, DTOOrder order)
    {
        try
        {
            var baseUri = Constants.GetUriFromConfig();
            var orderToUpdate = this.ordersRepository.GetOrder(id);
            if (orderToUpdate == null)
            {
                return NotFound();
            }

            var hashedOrder = orderToUpdate.GetHashCode();
            string hashedOrderEtag = $"\"{hashedOrder}\"";

            // Retrieve the If-Match header from the request (if it exists)
            var matchEtags = Request.Headers.IfMatch;

            // If there is an Etag in the If-Match header and
            // this etag matches that of the order just retrieved,
            // or if there is no etag, then update the Order
            if (((matchEtags.Count > 0 &&
                String.CompareOrdinal(matchEtags.First().Tag, hashedOrderEtag) == 0)) ||
                matchEtags.Count == 0)
            {
                // Modify the order
                orderToUpdate.OrderValue = order.OrderValue;
                orderToUpdate.ProductID = order.ProductID;
                orderToUpdate.Quantity = order.Quantity;

                // Save the order back to the data store
                // ...

                // Create the No Content response with Cache-Control, ETag, and Location headers
                var cacheControlHeader = new CacheControlHeaderValue();
                cacheControlHeader.Private = true;
                cacheControlHeader.MaxAge = new TimeSpan(0, 10, 0);

                hashedOrder = order.GetHashCode();
                hashedOrderEtag = $"\"{hashedOrder}\"";
                var eTag = new EntityTagHeaderValue(hashedOrderEtag);

                var location = new Uri($"{baseUri}/{Constants.ORDERS}/{id}");
                var response = new EmptyResultWithCaching()
                {
                    StatusCode = HttpStatusCode.NoContent,
                    CacheControlHeader = cacheControlHeader,
                    ETag = eTag,
                    Location = location
                };

                return response;
            }

            // Otherwise return a Precondition Failed response
            return StatusCode(HttpStatusCode.PreconditionFailed);
        }
        catch
        {
            return InternalServerError();
        }
    }
    ...
}
```

> [!TIP]
> <span data-ttu-id="f63b8-256">是否使用 If-match 标头完全是可选的，如果省略它，Web API 将始终尝试更新指定的订单，可能会盲目地覆盖其他用户所做的更新。</span><span class="sxs-lookup"><span data-stu-id="f63b8-256">Use of the If-Match header is entirely optional, and if it is omitted the web API will always attempt to update the specified order, possibly blindly overwriting an update made by another user.</span></span> <span data-ttu-id="f63b8-257">若要避免由于丢失更新出现的问题，请始终提供 If-Match 标头。</span><span class="sxs-lookup"><span data-stu-id="f63b8-257">To avoid problems due to lost updates, always provide an If-Match header.</span></span>
>
>

## <a name="considerations-for-handling-large-requests-and-responses"></a><span data-ttu-id="f63b8-258">有关处理大型请求和响应的注意事项</span><span class="sxs-lookup"><span data-stu-id="f63b8-258">Considerations for handling large requests and responses</span></span>
<span data-ttu-id="f63b8-259">可能会有这样的情况：客户端应用程序需要发出用于发送或接收大小可能为几兆字节（或更大）的数据的请求。</span><span class="sxs-lookup"><span data-stu-id="f63b8-259">There may be occasions when a client application needs to issue requests that send or receive data that may be several megabytes (or bigger) in size.</span></span> <span data-ttu-id="f63b8-260">等待传输如此大量的数据可能会导致客户端应用程序停止响应。</span><span class="sxs-lookup"><span data-stu-id="f63b8-260">Waiting while this amount of data is transmitted could cause the client application to become unresponsive.</span></span> <span data-ttu-id="f63b8-261">在需要处理包含大量数据的请求时，请考虑以下几点：</span><span class="sxs-lookup"><span data-stu-id="f63b8-261">Consider the following points when you need to handle requests that include significant amounts of data:</span></span>

### <a name="optimize-requests-and-responses-that-involve-large-objects"></a><span data-ttu-id="f63b8-262">优化涉及大型对象的请求和响应</span><span class="sxs-lookup"><span data-stu-id="f63b8-262">Optimize requests and responses that involve large objects</span></span>

<span data-ttu-id="f63b8-263">一些资源可能是大型对象或包含大量字段，例如图形图像或其他类型的二进制数据。</span><span class="sxs-lookup"><span data-stu-id="f63b8-263">Some resources may be large objects or include large fields, such as graphics images or other types of binary data.</span></span> <span data-ttu-id="f63b8-264">Web API 应支持流式处理以实现优化这些资源的上传和下载。</span><span class="sxs-lookup"><span data-stu-id="f63b8-264">A web API should support streaming to enable optimized uploading and downloading of these resources.</span></span>

<span data-ttu-id="f63b8-265">HTTP 协议提供了分块传输编码机制，用于将大型数据对象流式传输回客户端。</span><span class="sxs-lookup"><span data-stu-id="f63b8-265">The HTTP protocol provides the chunked transfer encoding mechanism to stream large data objects back to a client.</span></span> <span data-ttu-id="f63b8-266">当客户端为大型对象发送 HTTP GET 请求时，Web API 可以通过 HTTP 连接在段落*区块*中发送回答复。</span><span class="sxs-lookup"><span data-stu-id="f63b8-266">When the client sends an HTTP GET request for a large object, the web API can send the reply back in piecemeal *chunks* over an HTTP connection.</span></span> <span data-ttu-id="f63b8-267">最初答复中数据的长度可能未知（可能会生成它），因此托管 Web API 的服务器应发送包含每个区块的响应消息，这些区块指定 Transfer-Encoding: Chunked 标头而不是 Content-Length 标头。</span><span class="sxs-lookup"><span data-stu-id="f63b8-267">The length of the data in the reply may not be known initially (it might be generated), so the server hosting the web API should send a response message with each chunk that specifies the Transfer-Encoding: Chunked header rather than a Content-Length header.</span></span> <span data-ttu-id="f63b8-268">客户端应用程序可以依次接收每个区块以组合成完整响应。</span><span class="sxs-lookup"><span data-stu-id="f63b8-268">The client application can receive each chunk in turn to build up the complete response.</span></span> <span data-ttu-id="f63b8-269">在数据传输完成后，服务器将发送回最后一个区块（大小为零）。</span><span class="sxs-lookup"><span data-stu-id="f63b8-269">The data transfer completes when the server sends back a final chunk with zero size.</span></span> 

<span data-ttu-id="f63b8-270">可以想象单个请求生成使用大量资源的大型对象。</span><span class="sxs-lookup"><span data-stu-id="f63b8-270">A single request could conceivably result in a massive object that consumes considerable resources.</span></span> <span data-ttu-id="f63b8-271">如果在流式处理过程中，Web API 确定请求中的数据量超过一些可接受的范围，它可以中止操作并返回具有状态代码 413（请求实体太大）的响应消息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-271">If, during the streaming process, the web API determines that the amount of data in a request has exceeded some acceptable bounds, it can abort the operation and return a response message with status code 413 (Request Entity Too Large).</span></span>

<span data-ttu-id="f63b8-272">可以使用 HTTP 压缩将通过网络传输的大型对象的大小降到最低。</span><span class="sxs-lookup"><span data-stu-id="f63b8-272">You can minimize the size of large objects transmitted over the network by using HTTP compression.</span></span> <span data-ttu-id="f63b8-273">此方法可帮助减少网络流量和关联的网络延迟，但代价是需要在客户端和托管 Web API 的服务器上进行额外的处理。</span><span class="sxs-lookup"><span data-stu-id="f63b8-273">This approach helps to reduce the amount of network traffic and the associated network latency, but at the cost of requiring additional processing at the client and the server hosting the web API.</span></span> <span data-ttu-id="f63b8-274">例如，需要接收压缩数据的客户端应用程序可以提供 Accept-Encoding: gzip 请求标头（还可以指定其他数据压缩算法）。</span><span class="sxs-lookup"><span data-stu-id="f63b8-274">For example, a client application that expects to receive compressed data can include an Accept-Encoding: gzip request header (other data compression algorithms can also be specified).</span></span> <span data-ttu-id="f63b8-275">如果服务器支持压缩，则应以消息正文中以 gzip 格式存储的内容以及 Content-Encoding: gzip 响应标头进行响应。</span><span class="sxs-lookup"><span data-stu-id="f63b8-275">If the server supports compression it should respond with the content held in gzip format in the message body and the Content-Encoding: gzip response header.</span></span>

<span data-ttu-id="f63b8-276">可以将编码压缩和流式处理结合使用；在流式处理数据之前先压缩它，并在消息标头中指定 gzip 内容编码和 chunked 传输编码。</span><span class="sxs-lookup"><span data-stu-id="f63b8-276">You can combine encoded compression with streaming; compress the data first before streaming it, and specify the gzip content encoding and chunked transfer encoding in the message headers.</span></span> <span data-ttu-id="f63b8-277">另请注意，某些 Web 服务器（如 Internet Information Server）可以配置为自动压缩 HTTP 响应，而不管 Web API 是否压缩数据。</span><span class="sxs-lookup"><span data-stu-id="f63b8-277">Also note that some web servers (such as Internet Information Server) can be configured to automatically compress HTTP responses regardless of whether the web API compresses the data or not.</span></span>

### <a name="implement-partial-responses-for-clients-that-do-not-support-asynchronous-operations"></a><span data-ttu-id="f63b8-278">为不支持异步操作的客户端实现部分响应</span><span class="sxs-lookup"><span data-stu-id="f63b8-278">Implement partial responses for clients that do not support asynchronous operations</span></span>

<span data-ttu-id="f63b8-279">作为异步流式处理的替代方法，客户端应用程序可以区块方式显式请求大型对象的数据，称为部分响应。</span><span class="sxs-lookup"><span data-stu-id="f63b8-279">As an alternative to asynchronous streaming, a client application can explicitly request data for large objects in chunks, known as partial responses.</span></span> <span data-ttu-id="f63b8-280">客户端应用程序发送 HTTP HEAD 请求以获取有关对象的信息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-280">The client application sends an HTTP HEAD request to obtain information about the object.</span></span> <span data-ttu-id="f63b8-281">如果 Web API 支持部分响应，则应以包含 Accept-Ranges 标头和 Content-Length 标头（指示该对象的总大小），但消息正文应为空的响应消息响应 HEAD 请求。</span><span class="sxs-lookup"><span data-stu-id="f63b8-281">If the web API supports partial responses if should respond to the HEAD request with a response message that contains an Accept-Ranges header and a Content-Length header that indicates the total size of the object, but the body of the message should be empty.</span></span> <span data-ttu-id="f63b8-282">客户端应用程序可以使用此信息来构造一系列指定要接收的字节范围的 GET 请求。</span><span class="sxs-lookup"><span data-stu-id="f63b8-282">The client application can use this information to construct a series of GET requests that specify a range of bytes to receive.</span></span> <span data-ttu-id="f63b8-283">Web API 应返回包含以下内容的响应消息：HTTP 状态 206（部分内容）、指定响应消息正文中包含的实际数据量的 Content-Length 标头，以及指示此数据表示对象的哪一部分（例如 4000 到 8000 个字节）的 Content-Range 标头。</span><span class="sxs-lookup"><span data-stu-id="f63b8-283">The web API should return a response message with HTTP status 206 (Partial Content), a Content-Length header that specifies the actual amount of data included in the body of the response message, and a Content-Range header that indicates which part (such as bytes 4000 to 8000) of the object this data represents.</span></span>

<span data-ttu-id="f63b8-284">HTTP HEAD 请求和部分响应会在 [API 设计][api-design]中详细介绍。</span><span class="sxs-lookup"><span data-stu-id="f63b8-284">HTTP HEAD requests and partial responses are described in more detail in [API Design][api-design].</span></span>

### <a name="avoid-sending-unnecessary-100-continue-status-messages-in-client-applications"></a><span data-ttu-id="f63b8-285">避免在客户端应用程序中发送不必要的“100-Continue”状态消息</span><span class="sxs-lookup"><span data-stu-id="f63b8-285">Avoid sending unnecessary 100-Continue status messages in client applications</span></span>

<span data-ttu-id="f63b8-286">将要发送大量数据到服务器的客户端应用程序可能会先确定服务器是否实际可以接受该请求。</span><span class="sxs-lookup"><span data-stu-id="f63b8-286">A client application that is about to send a large amount of data to a server may determine first whether the server is actually willing to accept the request.</span></span> <span data-ttu-id="f63b8-287">在发送数据之前，客户端应用程序可以提交一个 HTTP 请求，其中包含 Expect: 100-Continue 标头、Content-Length 标头（指示数据的大小），但消息正文为空。</span><span class="sxs-lookup"><span data-stu-id="f63b8-287">Prior to sending the data, the client application can submit an HTTP request with an Expect: 100-Continue header, a Content-Length header that indicates the size of the data, but an empty message body.</span></span> <span data-ttu-id="f63b8-288">如果服务器可以处理该请求，则应以指定 HTTP 状态 100（继续）的消息进行响应。</span><span class="sxs-lookup"><span data-stu-id="f63b8-288">If the server is willing to handle the request, it should respond with a message that specifies the HTTP status 100 (Continue).</span></span> <span data-ttu-id="f63b8-289">然后，客户端应用程序可以继续操作并发送包含消息体中的数据的完整请求。</span><span class="sxs-lookup"><span data-stu-id="f63b8-289">The client application can then proceed and send the complete request including the data in the message body.</span></span>

<span data-ttu-id="f63b8-290">如果使用 IIS 来托管服务，则 HTTP.sys 驱动程序会自动检测并处理 Expect: 100-Continue 标头，然后再将请求传递到 Web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="f63b8-290">If you are hosting a service by using IIS, the HTTP.sys driver automatically detects and handles Expect: 100-Continue headers before passing requests to your web application.</span></span> <span data-ttu-id="f63b8-291">这意味着你很可能在应用程序代码中看不到这些标头，可以假设 IIS 已筛选掉任何它认为不适合或太大的消息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-291">This means that you are unlikely to see these headers in your application code, and you can assume that IIS has already filtered any messages that it deems to be unfit or too large.</span></span>

<span data-ttu-id="f63b8-292">如果使用.NET Framework 生成客户端应用程序，则默认情况下所有 POST 和 PUT 消息都将先发送包含 Expect: 100-Continue 标头的消息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-292">If you are building client applications by using the .NET Framework, then all POST and PUT messages will first send messages with Expect: 100-Continue headers by default.</span></span> <span data-ttu-id="f63b8-293">与服务器端一样，由 .NET Framework 透明地处理该过程。</span><span class="sxs-lookup"><span data-stu-id="f63b8-293">As with the server-side, the process is handled transparently by the .NET Framework.</span></span> <span data-ttu-id="f63b8-294">但是，此过程的结果是，每个 POST 和 PUT 请求会导致对服务器进行 2 次往返，即使是小请求，也是如此。</span><span class="sxs-lookup"><span data-stu-id="f63b8-294">However, this process results in each POST and PUT request causing two round-trips to the server, even for small requests.</span></span> <span data-ttu-id="f63b8-295">如果应用程序不发送包含大量数据的请求，则可以通过在客户端应用程序中使用 `ServicePointManager` 类创建 `ServicePoint` 对象来禁用此功能。</span><span class="sxs-lookup"><span data-stu-id="f63b8-295">If your application is not sending requests with large amounts of data, you can disable this feature by using the `ServicePointManager` class to create `ServicePoint` objects in the client application.</span></span> <span data-ttu-id="f63b8-296">`ServicePoint` 对象将基于标识服务器上的资源的 URI 的方案和主机片段处理客户端建立的与服务器的连接。</span><span class="sxs-lookup"><span data-stu-id="f63b8-296">A `ServicePoint` object handles the connections that the client makes to a server based on the scheme and host fragments of URIs that identify resources on the server.</span></span> <span data-ttu-id="f63b8-297">然后，可以将 `ServicePoint` 对象的 `Expect100Continue` 属性设置为 false。</span><span class="sxs-lookup"><span data-stu-id="f63b8-297">You can then set the `Expect100Continue` property of the `ServicePoint` object to false.</span></span> <span data-ttu-id="f63b8-298">客户端通过与 `ServicePoint` 对象的方案和主机片段匹配的 URI 发出的所有后续 POST 和 PUT 请求在发送时会不包含 Expect: 100-Continue 标头。</span><span class="sxs-lookup"><span data-stu-id="f63b8-298">All subsequent POST and PUT requests made by the client through a URI that matches the scheme and host fragments of the `ServicePoint` object will be sent without Expect: 100-Continue headers.</span></span> <span data-ttu-id="f63b8-299">下面的代码演示如何配置 `ServicePoint` 对象，以便将所有请求都发送到方案为 `http` 且主机为 `www.contoso.com` 的 URI。</span><span class="sxs-lookup"><span data-stu-id="f63b8-299">The following code shows how to configure a `ServicePoint` object that configures all requests sent to URIs with a scheme of `http` and a host of `www.contoso.com`.</span></span>

```csharp
Uri uri = new Uri("http://www.contoso.com/");
ServicePoint sp = ServicePointManager.FindServicePoint(uri);
sp.Expect100Continue = false;
```

<span data-ttu-id="f63b8-300">还可以设置 `ServicePointManager` 类的静态 `Expect100Continue` 属性，以便为所有后续创建的 `ServicePoint` 对象指定此属性的默认值。</span><span class="sxs-lookup"><span data-stu-id="f63b8-300">You can also set the static `Expect100Continue` property of the `ServicePointManager` class to specify the default value of this property for all subsequently created `ServicePoint` objects.</span></span> <span data-ttu-id="f63b8-301">有关详细信息，请参阅 [ServicePoint 类](https://msdn.microsoft.com/library/system.net.servicepoint.aspx)。</span><span class="sxs-lookup"><span data-stu-id="f63b8-301">For more information, see [ServicePoint Class](https://msdn.microsoft.com/library/system.net.servicepoint.aspx).</span></span>

### <a name="support-pagination-for-requests-that-may-return-large-numbers-of-objects"></a><span data-ttu-id="f63b8-302">支持对可能返回大量对象的请求进行分页</span><span class="sxs-lookup"><span data-stu-id="f63b8-302">Support pagination for requests that may return large numbers of objects</span></span>

<span data-ttu-id="f63b8-303">如果集合包含大量资源，则向相应的 URI 发出 GET 请求可能会导致在托管 Web API 的服务器上进行大量处理而影响性能，并生成大量网络流量导致延迟时间增加。</span><span class="sxs-lookup"><span data-stu-id="f63b8-303">If a collection contains a large number of resources, issuing a GET request to the corresponding URI could result in significant processing on the server hosting the web API affecting performance, and generate a significant amount of network traffic resulting in increased latency.</span></span>

<span data-ttu-id="f63b8-304">若要处理这些情况，Web API 应支持这样的查询字符串：使客户端应用程序可以细化请求或在更易于管理的离散块（或页）中提取数据。</span><span class="sxs-lookup"><span data-stu-id="f63b8-304">To handle these cases, the web API should support query strings that enable the client application to refine requests or fetch data in more manageable, discrete blocks (or pages).</span></span> <span data-ttu-id="f63b8-305">下面的代码显示 `Orders` 控制器中的 `GetAllOrders` 方法。</span><span class="sxs-lookup"><span data-stu-id="f63b8-305">The code below shows the `GetAllOrders` method in the `Orders` controller.</span></span> <span data-ttu-id="f63b8-306">此方法检索订单的详细信息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-306">This method retrieves the details of orders.</span></span> <span data-ttu-id="f63b8-307">如果此方法不受约束，可以想像它可能返回大量数据。</span><span class="sxs-lookup"><span data-stu-id="f63b8-307">If this method was unconstrained, it could conceivably return a large amount of data.</span></span> <span data-ttu-id="f63b8-308">`limit` 和 `offset` 参数旨在将数据量减少到较小子集，在本示例中默认情况下为前 10 个订单：</span><span class="sxs-lookup"><span data-stu-id="f63b8-308">The `limit` and `offset` parameters are intended to reduce the volume of data to a smaller subset, in this case only the first 10 orders by default:</span></span>

```csharp
public class OrdersController : ApiController
{
    ...
    [Route("api/orders")]
    [HttpGet]
    public IEnumerable<Order> GetAllOrders(int limit=10, int offset=0)
    {
        // Find the number of orders specified by the limit parameter
        // starting with the order specified by the offset parameter
        var orders = ...
        return orders;
    }
    ...
}
```

<span data-ttu-id="f63b8-309">客户端应用程序可以使用 URI `http://www.adventure-works.com/api/orders?limit=30&offset=50` 发出请求，检索从偏移量 50 开始的 30 个订单。</span><span class="sxs-lookup"><span data-stu-id="f63b8-309">A client application can issue a request to retrieve 30 orders starting at offset 50 by using the URI `http://www.adventure-works.com/api/orders?limit=30&offset=50`.</span></span>

> [!TIP]
> <span data-ttu-id="f63b8-310">请避免让客户端应用程序指定的查询字符串导致 URI 超过 2000 个字符。</span><span class="sxs-lookup"><span data-stu-id="f63b8-310">Avoid enabling client applications to specify query strings that result in a URI that is more than 2000 characters long.</span></span> <span data-ttu-id="f63b8-311">许多 Web 客户端和服务器无法处理这么长的 URI。</span><span class="sxs-lookup"><span data-stu-id="f63b8-311">Many web clients and servers cannot handle URIs that are this long.</span></span>
>
>

## <a name="considerations-for-maintaining-responsiveness-scalability-and-availability"></a><span data-ttu-id="f63b8-312">有关保持响应能力、可伸缩性和可用性的注意事项</span><span class="sxs-lookup"><span data-stu-id="f63b8-312">Considerations for maintaining responsiveness, scalability, and availability</span></span>
<span data-ttu-id="f63b8-313">同一 Web API 可能由世界任何地方运行的许多客户端应用程序利用。</span><span class="sxs-lookup"><span data-stu-id="f63b8-313">The same web API might be utilized by many client applications running anywhere in the world.</span></span> <span data-ttu-id="f63b8-314">请务必确保将 Web API 实现为在重负载下保持响应能力、可扩展以支持高度变化的工作负荷，并保证执行关键业务操作的客户端的可用性。</span><span class="sxs-lookup"><span data-stu-id="f63b8-314">It is important to ensure that the web API is implemented to maintain responsiveness under a heavy load, to be scalable to support a highly varying workload, and to guarantee availability for clients that perform business-critical operations.</span></span> <span data-ttu-id="f63b8-315">确定如何满足这些要求时，请考虑以下几点：</span><span class="sxs-lookup"><span data-stu-id="f63b8-315">Consider the following points when determining how to meet these requirements:</span></span>

### <a name="provide-asynchronous-support-for-long-running-requests"></a><span data-ttu-id="f63b8-316">为长时间运行的请求提供异步支持</span><span class="sxs-lookup"><span data-stu-id="f63b8-316">Provide asynchronous support for long-running requests</span></span>

<span data-ttu-id="f63b8-317">可能需要很长的时间来处理的请求应在执行时确保不会阻塞提交请求的客户端。</span><span class="sxs-lookup"><span data-stu-id="f63b8-317">A request that might take a long time to process should be performed without blocking the client that submitted the request.</span></span> <span data-ttu-id="f63b8-318">Web API 可以执行一些初始检查来验证请求，启动单独的任务来执行工作，并返回包含 HTTP 代码 202（已接受）的响应消息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-318">The web API can perform some initial checking to validate the request, initiate a separate task to perform the work, and then return a response message with HTTP code 202 (Accepted).</span></span> <span data-ttu-id="f63b8-319">任务可作为 Web API 处理的一部分异步运行，或可将其卸载为后台任务。</span><span class="sxs-lookup"><span data-stu-id="f63b8-319">The task could run asynchronously as part of the web API processing, or it could be offloaded to a background task.</span></span>

<span data-ttu-id="f63b8-320">Web API 还应提供一种向客户端应用程序返回处理结果的机制。</span><span class="sxs-lookup"><span data-stu-id="f63b8-320">The web API should also provide a mechanism to return the results of the processing to the client application.</span></span> <span data-ttu-id="f63b8-321">可以通过以下两种方案实现此目的：为客户端应用程序提供轮询机制以定期查询处理是否已完成并获取结果，或者使 Web API 可以在操作完成时发送通知。</span><span class="sxs-lookup"><span data-stu-id="f63b8-321">You can achieve this by providing a polling mechanism for client applications to periodically query whether the processing has finished and obtain the result, or enabling the web API to send a notification when the operation has completed.</span></span>

<span data-ttu-id="f63b8-322">可以使用以下方法，通过提供充当虚拟资源的*轮询* URI 来实现简单的轮询机制：</span><span class="sxs-lookup"><span data-stu-id="f63b8-322">You can implement a simple polling mechanism by providing a *polling* URI that acts as a virtual resource using the following approach:</span></span>

1. <span data-ttu-id="f63b8-323">客户端应用程序将初始请求发送到 Web API。</span><span class="sxs-lookup"><span data-stu-id="f63b8-323">The client application sends the initial request to the web API.</span></span>
2. <span data-ttu-id="f63b8-324">Web API 将有关该请求的信息存储在表存储或 Microsoft Azure 缓存中保存的表中，并可能以 GUID 形式为此条目生成唯一键。</span><span class="sxs-lookup"><span data-stu-id="f63b8-324">The web API stores information about the request in a table held in table storage or Microsoft Azure Cache, and generates a unique key for this entry, possibly in the form of a GUID.</span></span>
3. <span data-ttu-id="f63b8-325">Web API 启动处理作为单独的任务。</span><span class="sxs-lookup"><span data-stu-id="f63b8-325">The web API initiates the processing as a separate task.</span></span> <span data-ttu-id="f63b8-326">Web API 在表中将该任务的状态记录为“正在运行”。</span><span class="sxs-lookup"><span data-stu-id="f63b8-326">The web API records the state of the task in the table as *Running*.</span></span>
4. <span data-ttu-id="f63b8-327">Web API 返回包含 HTTP 状态代码 202（已接受）的响应消息，消息正文中包含该表条目的 GUID。</span><span class="sxs-lookup"><span data-stu-id="f63b8-327">The web API returns a response message with HTTP status code 202 (Accepted), and the GUID of the table entry in the body of the message.</span></span>
5. <span data-ttu-id="f63b8-328">完成该任务后，Web API 将结果存储在表中，并将该任务的状态设置为“完成”。</span><span class="sxs-lookup"><span data-stu-id="f63b8-328">When the task has completed, the web API stores the results in the table, and sets the state of the task to *Complete*.</span></span> <span data-ttu-id="f63b8-329">请注意，如果该任务失败，Web API 还可能存储有关失败的信息并将状态设置为“失败”。</span><span class="sxs-lookup"><span data-stu-id="f63b8-329">Note that if the task fails, the web API could also store information about the failure and set the status to *Failed*.</span></span>
6. <span data-ttu-id="f63b8-330">该任务运行时，客户端可以继续执行其自己的处理。</span><span class="sxs-lookup"><span data-stu-id="f63b8-330">While the task is running, the client can continue performing its own processing.</span></span> <span data-ttu-id="f63b8-331">它可以定期将请求发送到 URI */polling/{guid}*，其中 *{guid}* 是 Web API 在 202 响应消息中返回的 GUID。</span><span class="sxs-lookup"><span data-stu-id="f63b8-331">It can periodically send a request to the URI */polling/{guid}* where *{guid}* is the GUID returned in the 202 response message by the web API.</span></span>
7. <span data-ttu-id="f63b8-332">*/polling/{guid}* URI 中的 Web API 在表中查询相应任务的状态并返回包含 HTTP 状态代码“200 (正常)”的响应消息，该代码包含以下状态：“正在运行”、“完成”或“失败”。</span><span class="sxs-lookup"><span data-stu-id="f63b8-332">The web API at the */polling/{guid}* URI queries the state of the corresponding task in the table and returns a response message with HTTP status code 200 (OK) containing this state (*Running*, *Complete*, or *Failed*).</span></span> <span data-ttu-id="f63b8-333">如果该任务已完成或失败，则响应消息还可以包括处理的结果或获得的有关失败原因的任何信息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-333">If the task has completed or failed, the response message can also include the results of the processing or any information available about the reason for the failure.</span></span>

<span data-ttu-id="f63b8-334">用于实现通知的选项包括：</span><span class="sxs-lookup"><span data-stu-id="f63b8-334">Options for implementing notifications include:</span></span>

- <span data-ttu-id="f63b8-335">使用 Azure 通知中心将异步响应推送到客户端应用程序。</span><span class="sxs-lookup"><span data-stu-id="f63b8-335">Using an Azure Notification Hub to push asynchronous responses to client applications.</span></span> <span data-ttu-id="f63b8-336">有关详细信息，请参阅 [Azure 通知中心通知用户](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/)。</span><span class="sxs-lookup"><span data-stu-id="f63b8-336">For more information, see [Azure Notification Hubs Notify Users](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/).</span></span>
- <span data-ttu-id="f63b8-337">使用 Comet 模型来保持客户端与托管 Web API 的服务器之间的永久网络连接，并使用此连接将消息从服务器推送回客户端。</span><span class="sxs-lookup"><span data-stu-id="f63b8-337">Using the Comet model to retain a persistent network connection between the client and the server hosting the web API, and using this connection to push messages from the server back to the client.</span></span> <span data-ttu-id="f63b8-338">MSDN 杂志文章 [Building a Simple Comet Application in the Microsoft .NET Framework](https://msdn.microsoft.com/magazine/jj891053.aspx)（在 Microsoft .NET Framework 中构建简单的 Comet 应用程序）介绍了一个示例解决方案。</span><span class="sxs-lookup"><span data-stu-id="f63b8-338">The MSDN magazine article [Building a Simple Comet Application in the Microsoft .NET Framework](https://msdn.microsoft.com/magazine/jj891053.aspx) describes an example solution.</span></span>
- <span data-ttu-id="f63b8-339">使用 SignalR 通过永久网络连接将 Web 服务器中的实时数据推送到客户端。</span><span class="sxs-lookup"><span data-stu-id="f63b8-339">Using SignalR to push data in real-time from the web server to the client over a persistent network connection.</span></span> <span data-ttu-id="f63b8-340">SignalR 可作为 NuGet 程序包用于 ASP.NET Web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="f63b8-340">SignalR is available for ASP.NET web applications as a NuGet package.</span></span> <span data-ttu-id="f63b8-341">可以在 [ASP.NET SignalR](http://signalr.net/) 网站上找到详细信息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-341">You can find more information on the [ASP.NET SignalR](http://signalr.net/) website.</span></span>

### <a name="ensure-that-each-request-is-stateless"></a><span data-ttu-id="f63b8-342">确保每个请求都是无状态的</span><span class="sxs-lookup"><span data-stu-id="f63b8-342">Ensure that each request is stateless</span></span>

<span data-ttu-id="f63b8-343">每个请求应视为原子的。</span><span class="sxs-lookup"><span data-stu-id="f63b8-343">Each request should be considered atomic.</span></span> <span data-ttu-id="f63b8-344">客户端应用程序发出的一个请求应与同一客户端提交的任何后续请求之间没有任何依赖关系。</span><span class="sxs-lookup"><span data-stu-id="f63b8-344">There should be no dependencies between one request made by a client application and any subsequent requests submitted by the same client.</span></span> <span data-ttu-id="f63b8-345">此方法可帮助你实现伸缩性；可以在多个服务器上部署 Web 服务的实例。</span><span class="sxs-lookup"><span data-stu-id="f63b8-345">This approach assists in scalability; instances of the web service can be deployed on a number of servers.</span></span> <span data-ttu-id="f63b8-346">客户端请求可以定向到其中任一实例，并且结果应始终相同。</span><span class="sxs-lookup"><span data-stu-id="f63b8-346">Client requests can be directed at any of these instances and the results should always be the same.</span></span> <span data-ttu-id="f63b8-347">由于类似的原因，它还提高了可用性；如果某个 Web 服务器失败，可以将请求路由到其他实例（通过使用 Azure 流量管理器），同时重新启动该服务器，对客户端应用程序没有任何不良影响。</span><span class="sxs-lookup"><span data-stu-id="f63b8-347">It also improves availability for a similar reason; if a web server fails requests can be routed to another instance (by using Azure Traffic Manager) while the server is restarted with no ill effects on client applications.</span></span>

### <a name="track-clients-and-implement-throttling-to-reduce-the-chances-of-dos-attacks"></a><span data-ttu-id="f63b8-348">跟踪客户端并实施限制以减少 DOS 攻击的可能性</span><span class="sxs-lookup"><span data-stu-id="f63b8-348">Track clients and implement throttling to reduce the chances of DOS attacks</span></span>

<span data-ttu-id="f63b8-349">如果特定客户端在给定的时间段内发出了大量请求，则它可能会独占服务并影响其他客户端的性能。</span><span class="sxs-lookup"><span data-stu-id="f63b8-349">If a specific client makes a large number of requests within a given period of time it might monopolize the service and affect the performance of other clients.</span></span> <span data-ttu-id="f63b8-350">若要缓解此问题，Web API 可以通过跟踪所有传入请求的 IP 地址或通过记录每次经过身份验证的访问来监视客户端应用程序的调用。</span><span class="sxs-lookup"><span data-stu-id="f63b8-350">To mitigate this issue, a web API can monitor calls from client applications either by tracking the IP address of all incoming requests or by logging each authenticated access.</span></span> <span data-ttu-id="f63b8-351">可以使用此信息来限制资源访问。</span><span class="sxs-lookup"><span data-stu-id="f63b8-351">You can use this information to limit resource access.</span></span> <span data-ttu-id="f63b8-352">如果客户端超出定义的限制，Web API 可以返回包含状态 503（服务不可用）的响应消息并在其中包括 Retry-After 标头以指定客户端可以发送下一个请求而不会被拒绝的时间。</span><span class="sxs-lookup"><span data-stu-id="f63b8-352">If a client exceeds a defined limit, the web API can return a response message with status 503 (Service Unavailable) and include a Retry-After header that specifies when the client can send the next request without it being declined.</span></span> <span data-ttu-id="f63b8-353">此策略可帮助降低停止系统的一组客户端发起拒绝服务 (DOS) 攻击的可能性。</span><span class="sxs-lookup"><span data-stu-id="f63b8-353">This strategy can help to reduce the chances of a Denial Of Service (DOS) attack from a set of clients stalling the system.</span></span>

### <a name="manage-persistent-http-connections-carefully"></a><span data-ttu-id="f63b8-354">小心管理永久 HTTP 连接</span><span class="sxs-lookup"><span data-stu-id="f63b8-354">Manage persistent HTTP connections carefully</span></span>

<span data-ttu-id="f63b8-355">HTTP 协议支持永久 HTTP 连接（在这些连接可用时）。</span><span class="sxs-lookup"><span data-stu-id="f63b8-355">The HTTP protocol supports persistent HTTP connections where they are available.</span></span> <span data-ttu-id="f63b8-356">HTTP 1.0 规范已添加 Connection:Keep-Alive 标头以允许客户端应用程序向服务器表明它可能使用同一连接发送后续请求而不是打开新连接。</span><span class="sxs-lookup"><span data-stu-id="f63b8-356">The HTTP 1.0 specificiation added the Connection:Keep-Alive header that enables a client application to indicate to the server that it can use the same connection to send subsequent requests rather than opening new ones.</span></span> <span data-ttu-id="f63b8-357">如果客户端在主机定义的时间段内未重用连接，则该连接会自动关闭。</span><span class="sxs-lookup"><span data-stu-id="f63b8-357">The connection closes automatically if the client does not reuse the connection within a period defined by the host.</span></span> <span data-ttu-id="f63b8-358">此行为在 Azure 服务使用的 HTTP 1.1 中是默认设置，因此无需在消息中包括 Keep-Alive 标头。</span><span class="sxs-lookup"><span data-stu-id="f63b8-358">This behavior is the default in HTTP 1.1 as used by Azure services, so there is no need to include Keep-Alive headers in messages.</span></span>

<span data-ttu-id="f63b8-359">让连接保持打开状态可以减少延迟和网络拥塞，从而有助于提高响应能力，但让不必要的连接保持打开状态的时间长于所需时间可能会不利于可扩展性，从而限制了其他并发客户端进行连接的能力。</span><span class="sxs-lookup"><span data-stu-id="f63b8-359">Keeping a connection open can help to improve responsiveness by reducing latency and network congestion, but it can be detrimental to scalability by keeping unnecessary connections open for longer than required, limiting the ability of other concurrent clients to connect.</span></span> <span data-ttu-id="f63b8-360">如果客户端应用程序运行在移动设备上，则还会影响电池使用寿命；如果应用程序只偶尔向服务器发出请求，则维持连接的打开状态可能会导致电池更快耗尽。</span><span class="sxs-lookup"><span data-stu-id="f63b8-360">It can also affect battery life if the client application is running on a mobile device; if the application only makes occasional requests to the server, maintaining an open connection can cause the battery to drain more quickly.</span></span> <span data-ttu-id="f63b8-361">若要确保不使用 HTTP 1.1 建立永久连接，客户端可以在消息中包括 Connection:Close 标头以覆盖默认行为。</span><span class="sxs-lookup"><span data-stu-id="f63b8-361">To ensure that a connection is not made persistent with HTTP 1.1, the client can include a Connection:Close header with messages to override the default behavior.</span></span> <span data-ttu-id="f63b8-362">同样，如果服务器正在处理大量客户端，它可以在响应消息中包括 Connection:Close 标头，这会关闭连接并节省服务器资源。</span><span class="sxs-lookup"><span data-stu-id="f63b8-362">Similarly, if a server is handling a very large number of clients it can include a Connection:Close header in response messages which should close the connection and save server resources.</span></span>

> [!NOTE]
> <span data-ttu-id="f63b8-363">永久 HTTP 连接是纯粹的可选功能，用于减少与反复建立通信通道关联的网络开销。</span><span class="sxs-lookup"><span data-stu-id="f63b8-363">Persistent HTTP connections are a purely optional feature to reduce the network overhead associated with repeatedly establishing a communications channel.</span></span> <span data-ttu-id="f63b8-364">Web API 和客户端应用程序都不应依赖于可用的永久 HTTP 连接。</span><span class="sxs-lookup"><span data-stu-id="f63b8-364">Neither the web API nor the client application should depend on a persistent HTTP connection being available.</span></span> <span data-ttu-id="f63b8-365">不要使用永久 HTTP 连接实现 Comet 样式通知系统，而是应利用 TCP 层的套接字（或 websocket，如果可用）。</span><span class="sxs-lookup"><span data-stu-id="f63b8-365">Do not use persistent HTTP connections to implement Comet-style notification systems; instead you should utilize sockets (or websockets if available) at the TCP layer.</span></span> <span data-ttu-id="f63b8-366">最后，请注意：如果客户端应用程序通过代理与服务器通信，则 Keep-Alive 标头的作用有限；只有与客户端和代理的连接将是持久的。</span><span class="sxs-lookup"><span data-stu-id="f63b8-366">Finally, note Keep-Alive headers are of limited use if a client application communicates with a server via a proxy; only the connection with the client and the proxy will be persistent.</span></span>
>
>

## <a name="considerations-for-publishing-and-managing-a-web-api"></a><span data-ttu-id="f63b8-367">有关发布和管理 Web API 的注意事项</span><span class="sxs-lookup"><span data-stu-id="f63b8-367">Considerations for publishing and managing a web API</span></span>
<span data-ttu-id="f63b8-368">要使 Web API 可供客户端应用程序使用，Web API 必须部署到主机环境中。</span><span class="sxs-lookup"><span data-stu-id="f63b8-368">To make a web API available for client applications, the web API must be deployed to a host environment.</span></span> <span data-ttu-id="f63b8-369">此环境通常是 Web 服务器，尽管它可能是某种其他类型的主机进程。</span><span class="sxs-lookup"><span data-stu-id="f63b8-369">This environment is typically a web server, although it may be some other type of host process.</span></span> <span data-ttu-id="f63b8-370">发布 Web API 时，应考虑以下几点：</span><span class="sxs-lookup"><span data-stu-id="f63b8-370">You should consider the following points when publishing a web API:</span></span>

* <span data-ttu-id="f63b8-371">所有请求都必须经过身份验证和授权，必须强制实施相应的访问控制级别。</span><span class="sxs-lookup"><span data-stu-id="f63b8-371">All requests must be authenticated and authorized, and the appropriate level of access control must be enforced.</span></span>
* <span data-ttu-id="f63b8-372">商业 Web API 可能会受到与响应时间有关的各种质量保证约束。</span><span class="sxs-lookup"><span data-stu-id="f63b8-372">A commercial web API might be subject to various quality guarantees concerning response times.</span></span> <span data-ttu-id="f63b8-373">如果负载随着时间的推移会发生显著变化，请务必确保主机环境是可缩放的。</span><span class="sxs-lookup"><span data-stu-id="f63b8-373">It is important to ensure that host environment is scalable if the load can vary significantly over time.</span></span>
* <span data-ttu-id="f63b8-374">出于盈利目的，可能有必要计量请求。</span><span class="sxs-lookup"><span data-stu-id="f63b8-374">It may be necessary to meter requests for monetization purposes.</span></span>
* <span data-ttu-id="f63b8-375">可能需要使用 Web API 调控通信流量，并对已用完其配额的特定客户端实施限制。</span><span class="sxs-lookup"><span data-stu-id="f63b8-375">It might be necessary to regulate the flow of traffic to the web API, and implement throttling for specific clients that have exhausted their quotas.</span></span>
* <span data-ttu-id="f63b8-376">法规要求可能会强制执行所有请求和响应的记录和审核。</span><span class="sxs-lookup"><span data-stu-id="f63b8-376">Regulatory requirements might mandate logging and auditing of all requests and responses.</span></span>
* <span data-ttu-id="f63b8-377">为了确保可用性，可能有必要监视托管 Web API 的服务器的运行状况并在必要时重新启动它。</span><span class="sxs-lookup"><span data-stu-id="f63b8-377">To ensure availability, it may be necessary to monitor the health of the server hosting the web API and restart it if necessary.</span></span>

<span data-ttu-id="f63b8-378">如果能够将这些问题从有关 Web API 的实现的技术问题中分离出来，会很有用。</span><span class="sxs-lookup"><span data-stu-id="f63b8-378">It is useful to be able to decouple these issues from the technical issues concerning the implementation of the web API.</span></span> <span data-ttu-id="f63b8-379">因此，可考虑创建一个[外观](http://en.wikipedia.org/wiki/Facade_pattern)，作为独立的进程运行，并将请求路由到 Web API。</span><span class="sxs-lookup"><span data-stu-id="f63b8-379">For this reason, consider creating a [façade](http://en.wikipedia.org/wiki/Facade_pattern), running as a separate process and that routes requests to the web API.</span></span> <span data-ttu-id="f63b8-380">外观可用于进行管理操作，并将验证过的请求转发到 Web API。</span><span class="sxs-lookup"><span data-stu-id="f63b8-380">The façade can provide the management operations and forward validated requests to the web API.</span></span> <span data-ttu-id="f63b8-381">使用外观还有许多功能优势，包括：</span><span class="sxs-lookup"><span data-stu-id="f63b8-381">Using a façade can also bring many functional advantages, including:</span></span>

* <span data-ttu-id="f63b8-382">充当多个 Web API 的集成点。</span><span class="sxs-lookup"><span data-stu-id="f63b8-382">Acting as an integration point for multiple web APIs.</span></span>
* <span data-ttu-id="f63b8-383">转换消息并转换使用不同技术生成的客户端的通信协议。</span><span class="sxs-lookup"><span data-stu-id="f63b8-383">Transforming messages and translating communications protocols for clients built by using varying technologies.</span></span>
* <span data-ttu-id="f63b8-384">缓存请求和响应以减少托管 Web API 的服务器上的负载。</span><span class="sxs-lookup"><span data-stu-id="f63b8-384">Caching requests and responses to reduce load on the server hosting the web API.</span></span>

## <a name="considerations-for-testing-a-web-api"></a><span data-ttu-id="f63b8-385">有关测试 Web API 的注意事项</span><span class="sxs-lookup"><span data-stu-id="f63b8-385">Considerations for testing a web API</span></span>
<span data-ttu-id="f63b8-386">Web API 应和软件的任何其他部分一样进行全面测试。</span><span class="sxs-lookup"><span data-stu-id="f63b8-386">A web API should be tested as thoroughly as any other piece of software.</span></span> <span data-ttu-id="f63b8-387">应考虑创建单元测试来验证其功能。Web API 的性质使它在验证操作是否正确方面具有它自有的额外要求。</span><span class="sxs-lookup"><span data-stu-id="f63b8-387">You should consider creating unit tests to validate the functionality of The nature of a web API brings its own additional requirements to verify that it operates correctly.</span></span> <span data-ttu-id="f63b8-388">应特别注意以下几个方面：</span><span class="sxs-lookup"><span data-stu-id="f63b8-388">You should pay particular attention to the following aspects:</span></span>

* <span data-ttu-id="f63b8-389">测试所有路由以验证它们是否调用正确的操作。</span><span class="sxs-lookup"><span data-stu-id="f63b8-389">Test all routes to verify that they invoke the correct operations.</span></span> <span data-ttu-id="f63b8-390">特别要注意意外返回的 HTTP 状态代码 405（不允许的方法），因为这可能指示路由与可分派给该路由的 HTTP 方法（GET、POST、PUT、DELETE）不匹配。</span><span class="sxs-lookup"><span data-stu-id="f63b8-390">Be especially aware of HTTP status code 405 (Method Not Allowed) being returned unexpectedly as this can indicate a mismatch between a route and the HTTP methods (GET, POST, PUT, DELETE) that can be dispatched to that route.</span></span>

    <span data-ttu-id="f63b8-391">将 HTTP 请求发送到不支持这些请求的路由，例如，将 POST 请求提交到特定资源（POST 请求只应发送到资源集合）。</span><span class="sxs-lookup"><span data-stu-id="f63b8-391">Send HTTP requests to routes that do not support them, such as submitting a POST request to a specific resource (POST requests should only be sent to resource collections).</span></span> <span data-ttu-id="f63b8-392">在这些情况下，唯一有效的响应*应*为状态代码“405 (不允许)”。</span><span class="sxs-lookup"><span data-stu-id="f63b8-392">In these cases, the only valid response *should* be status code 405 (Not Allowed).</span></span>
* <span data-ttu-id="f63b8-393">验证所有路由是否都得到正确保护并受相应身份验证和授权检查的制约。</span><span class="sxs-lookup"><span data-stu-id="f63b8-393">Verify that all routes are protected properly and are subject to the appropriate authentication and authorization checks.</span></span>

  > [!NOTE]
  > <span data-ttu-id="f63b8-394">安全性的某些方面（如用户身份验证）最有可能是主机环境（而不是 Web API）的职责，但仍有必要在部署过程中进行安全测试。</span><span class="sxs-lookup"><span data-stu-id="f63b8-394">Some aspects of security such as user authentication are most likely to be the responsibility of the host environment rather than the web API, but it is still necessary to include security tests as part of the deployment process.</span></span>
  >
  >
* <span data-ttu-id="f63b8-395">测试每个操作执行的异常处理，并验证是否将相应的有意义的 HTTP 响应传递回客户端应用程序。</span><span class="sxs-lookup"><span data-stu-id="f63b8-395">Test the exception handling performed by each operation and verify that an appropriate and meaningful HTTP response is passed back to the client application.</span></span>
* <span data-ttu-id="f63b8-396">验证请求和响应消息的格式是否正确。</span><span class="sxs-lookup"><span data-stu-id="f63b8-396">Verify that request and response messages are well-formed.</span></span> <span data-ttu-id="f63b8-397">例如，如果 HTTP POST 请求包含 x-www-form-urlencoded 格式的新资源数据，请确认相应的操作正确分析数据、创建该资源，并返回包含新资源的详细信息的响应，包括正确的 Location 标头。</span><span class="sxs-lookup"><span data-stu-id="f63b8-397">For example, if an HTTP POST request contains the data for a new resource in x-www-form-urlencoded format, confirm that the corresponding operation correctly parses the data, creates the resources, and returns a response containing the details of the new resource, including the correct Location header.</span></span>
* <span data-ttu-id="f63b8-398">验证响应消息中的所有链接和 URI。</span><span class="sxs-lookup"><span data-stu-id="f63b8-398">Verify all links and URIs in response messages.</span></span> <span data-ttu-id="f63b8-399">例如，HTTP POST 消息应返回新创建的资源的 URI。</span><span class="sxs-lookup"><span data-stu-id="f63b8-399">For example, an HTTP POST message should return the URI of the newly-created resource.</span></span> <span data-ttu-id="f63b8-400">所有 HATEOAS 链接都应有效。</span><span class="sxs-lookup"><span data-stu-id="f63b8-400">All HATEOAS links should be valid.</span></span>

* <span data-ttu-id="f63b8-401">确保每个操作针对不同输入组合返回正确的状态代码。</span><span class="sxs-lookup"><span data-stu-id="f63b8-401">Ensure that each operation returns the correct status codes for different combinations of input.</span></span> <span data-ttu-id="f63b8-402">例如：</span><span class="sxs-lookup"><span data-stu-id="f63b8-402">For example:</span></span>

  * <span data-ttu-id="f63b8-403">如果查询成功，则应返回状态代码 200（正常）</span><span class="sxs-lookup"><span data-stu-id="f63b8-403">If a query is successful, it should return status code 200 (OK)</span></span>
  * <span data-ttu-id="f63b8-404">如果未找到资源，则操作应返回 HTTP 状态代码 404（未找到）。</span><span class="sxs-lookup"><span data-stu-id="f63b8-404">If a resource is not found, the operation should return HTTP status code 404 (Not Found).</span></span>
  * <span data-ttu-id="f63b8-405">如果客户端发送的请求成功删除资源，则状态代码应为 204（无内容）。</span><span class="sxs-lookup"><span data-stu-id="f63b8-405">If the client sends a request that successfully deletes a resource, the status code should be 204 (No Content).</span></span>
  * <span data-ttu-id="f63b8-406">如果客户端发送的请求创建了新资源，则状态代码应为 201（已创建）</span><span class="sxs-lookup"><span data-stu-id="f63b8-406">If the client sends a request that creates a new resource, the status code should be 201 (Created)</span></span>

<span data-ttu-id="f63b8-407">密切注意 5xx 范围内的异常响应状态代码。</span><span class="sxs-lookup"><span data-stu-id="f63b8-407">Watch out for unexpected response status codes in the 5xx range.</span></span> <span data-ttu-id="f63b8-408">这些消息通常由主机服务器报告，以指示无法完成有效请求。</span><span class="sxs-lookup"><span data-stu-id="f63b8-408">These messages are usually reported by the host server to indicate that it was unable to fulfill a valid request.</span></span>

* <span data-ttu-id="f63b8-409">测试客户端应用程序可以指定的不同请求标头组合并确保 Web API 在响应消息中返回预期的信息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-409">Test the different request header combinations that a client application can specify and ensure that the web API returns the expected information in response messages.</span></span>
* <span data-ttu-id="f63b8-410">测试查询字符串。</span><span class="sxs-lookup"><span data-stu-id="f63b8-410">Test query strings.</span></span> <span data-ttu-id="f63b8-411">如果操作可以接受可选参数（例如分页请求），则测试参数的不同组合和顺序。</span><span class="sxs-lookup"><span data-stu-id="f63b8-411">If an operation can take optional parameters (such as pagination requests), test the different combinations and order of parameters.</span></span>
* <span data-ttu-id="f63b8-412">验证异步操作是否成功完成。</span><span class="sxs-lookup"><span data-stu-id="f63b8-412">Verify that asynchronous operations complete successfully.</span></span> <span data-ttu-id="f63b8-413">如果 Web API 支持对返回大型二进制对象（如视频或音频）的请求进行流式处理，请确保在流式传输数据时不会阻止客户端请求。</span><span class="sxs-lookup"><span data-stu-id="f63b8-413">If the web API supports streaming for requests that return large binary objects (such as video or audio), ensure that client requests are not blocked while the data is streamed.</span></span> <span data-ttu-id="f63b8-414">如果 Web API 实现了轮询长时间运行的数据修改操作，请验证这些操作在执行时正确报告其状态。</span><span class="sxs-lookup"><span data-stu-id="f63b8-414">If the web API implements polling for long-running data modification operations, verify that that the operations report their status correctly as they proceed.</span></span>

<span data-ttu-id="f63b8-415">还应创建并运行性能测试以检查 Web API 在压力下令人满意地运行。</span><span class="sxs-lookup"><span data-stu-id="f63b8-415">You should also create and run performance tests to check that the web API operates satisfactorily under duress.</span></span> <span data-ttu-id="f63b8-416">可以使用 Visual Studio Ultimate 构建一个 Web 性能和负载测试项目。</span><span class="sxs-lookup"><span data-stu-id="f63b8-416">You can build a web performance and load test project by using Visual Studio Ultimate.</span></span> <span data-ttu-id="f63b8-417">有关详细信息，请参阅 [Run performance tests on an application before a release](https://msdn.microsoft.com/library/dn250793.aspx)（在发布前对应用程序运行性能测试）。</span><span class="sxs-lookup"><span data-stu-id="f63b8-417">For more information, see [Run performance tests on an application before a release](https://msdn.microsoft.com/library/dn250793.aspx).</span></span>

## <a name="publish-and-manage-a-web-api-using-the-azure-api-management-service"></a><span data-ttu-id="f63b8-418">使用 Azure API 管理服务发布和管理 Web API</span><span class="sxs-lookup"><span data-stu-id="f63b8-418">Publish and manage a web API using the Azure API Management Service</span></span>
<span data-ttu-id="f63b8-419">Azure 提供了 [API 管理服务](https://azure.microsoft.com/documentation/services/api-management/)，可以用来发布和管理 Web API。</span><span class="sxs-lookup"><span data-stu-id="f63b8-419">Azure provides the [API Management Service](https://azure.microsoft.com/documentation/services/api-management/) which you can use to publish and manage a web API.</span></span> <span data-ttu-id="f63b8-420">使用此工具，可以生成一个充当一个或多个 Web API 的外观的服务。</span><span class="sxs-lookup"><span data-stu-id="f63b8-420">Using this facility, you can generate a service that acts as a façade for one or more web APIs.</span></span> <span data-ttu-id="f63b8-421">该服务本身是一个可缩放的 Web 服务，可以使用 Azure 管理门户创建和配置它。</span><span class="sxs-lookup"><span data-stu-id="f63b8-421">The service is itself a scalable web service that you can create and configure by using the Azure Management portal.</span></span> <span data-ttu-id="f63b8-422">可以使用此服务发布和管理 Web API，如下所示：</span><span class="sxs-lookup"><span data-stu-id="f63b8-422">You can use this service to publish and manage a web API as follows:</span></span>

1. <span data-ttu-id="f63b8-423">将 Web API 部署到网站、Azure 云服务或 Azure 虚拟机。</span><span class="sxs-lookup"><span data-stu-id="f63b8-423">Deploy the web API to a website, Azure cloud service, or Azure virtual machine.</span></span>
2. <span data-ttu-id="f63b8-424">将 API 管理服务连接到 Web API。</span><span class="sxs-lookup"><span data-stu-id="f63b8-424">Connect the API management service to the web API.</span></span> <span data-ttu-id="f63b8-425">发送到管理 API 的 URL 的请求将映射到 Web API 中的 URI。</span><span class="sxs-lookup"><span data-stu-id="f63b8-425">Requests sent to the URL of the management API are mapped to URIs in the web API.</span></span> <span data-ttu-id="f63b8-426">同一 API 管理服务可以将请求路由到多个 Web API。</span><span class="sxs-lookup"><span data-stu-id="f63b8-426">The same API management service can route requests to more than one web API.</span></span> <span data-ttu-id="f63b8-427">这样便可以将多个 Web API 聚合为单个管理服务。</span><span class="sxs-lookup"><span data-stu-id="f63b8-427">This enables you to aggregate multiple web APIs into a single management service.</span></span> <span data-ttu-id="f63b8-428">同样，如果需要限制或分隔可用于不同应用程序的功能，则可以从多个 API 管理服务引用同一 Web API。</span><span class="sxs-lookup"><span data-stu-id="f63b8-428">Similarly, the same web API can be referenced from more than one API management service if you need to restrict or partition the functionality available to different applications.</span></span>

   > [!NOTE]
   > <span data-ttu-id="f63b8-429">作为 HTTP GET 请求的响应的一部分生成的 HATEOAS 链接中的 URI 应引用 API 管理服务（而不是托管 Web API 的 Web 服务器）的 URL。</span><span class="sxs-lookup"><span data-stu-id="f63b8-429">The URIs in HATEOAS links generated as part of the response for HTTP GET requests should reference the URL of the API management service and not the web server hosting the web API.</span></span>
   >
   >
3. <span data-ttu-id="f63b8-430">对于每个 Web API，指定该 Web API 公开的 HTTP 操作以及操作可以获取为输入的任何可选参数。</span><span class="sxs-lookup"><span data-stu-id="f63b8-430">For each web API, specify the HTTP operations that the web API exposes together with any optional parameters that an operation can take as input.</span></span> <span data-ttu-id="f63b8-431">还可以配置 API 管理服务是否应缓存从 Web API 接收的响应以优化对相同数据的重复请求。</span><span class="sxs-lookup"><span data-stu-id="f63b8-431">You can also configure whether the API management service should cache the response received from the web API to optimize repeated requests for the same data.</span></span> <span data-ttu-id="f63b8-432">记录每个操作可以生成的 HTTP 响应的详细信息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-432">Record the details of the HTTP responses that each operation can generate.</span></span> <span data-ttu-id="f63b8-433">此信息用于为开发人员生成文档，因此它应准确且完整。</span><span class="sxs-lookup"><span data-stu-id="f63b8-433">This information is used to generate documentation for developers, so it is important that it is accurate and complete.</span></span>

   <span data-ttu-id="f63b8-434">可以使用 Azure 管理门户提供的向导手动定义操作，也可以从包含 WADL 或 Swagger 格式的定义的文件中导入操作。</span><span class="sxs-lookup"><span data-stu-id="f63b8-434">You can either define operations manually using the wizards provided by the Azure Management portal, or you can import them from a file containing the definitions in WADL or Swagger format.</span></span>
4. <span data-ttu-id="f63b8-435">为 API 管理服务与托管 Web API 的 Web 服务器之间的通信配置安全设置。</span><span class="sxs-lookup"><span data-stu-id="f63b8-435">Configure the security settings for communications between the API management service and the web server hosting the web API.</span></span> <span data-ttu-id="f63b8-436">API 管理服务目前支持使用证书和 OAuth 2.0 用户授权的基本身份验证和相互身份验证。</span><span class="sxs-lookup"><span data-stu-id="f63b8-436">The API management service currently supports Basic authentication and mutual authentication using certificates, and OAuth 2.0 user authorization.</span></span>
5. <span data-ttu-id="f63b8-437">创建产品。</span><span class="sxs-lookup"><span data-stu-id="f63b8-437">Create a product.</span></span> <span data-ttu-id="f63b8-438">产品是发布的单元；可将先前连接到管理服务的 Web API 添加到产品。</span><span class="sxs-lookup"><span data-stu-id="f63b8-438">A product is the unit of publication; you add the web APIs that you previously connected to the management service to the product.</span></span> <span data-ttu-id="f63b8-439">发布产品后，该 Web API 便可供开发人员使用了。</span><span class="sxs-lookup"><span data-stu-id="f63b8-439">When the product is published, the web APIs become available to developers.</span></span>

   > [!NOTE]
   > <span data-ttu-id="f63b8-440">在发布产品之前，还可以定义可以访问该产品的用户组，并将用户添加到这些组。</span><span class="sxs-lookup"><span data-stu-id="f63b8-440">Prior to publishing a product, you can also define user-groups that can access the product and add users to these groups.</span></span> <span data-ttu-id="f63b8-441">这让可以控制可以使用该 Web API 的开发人员和应用程序。</span><span class="sxs-lookup"><span data-stu-id="f63b8-441">This gives you control over the developers and applications that can use the web API.</span></span> <span data-ttu-id="f63b8-442">如果 Web API 需要批准，则在能够访问它之前，开发人员必须向产品管理员发送请求。</span><span class="sxs-lookup"><span data-stu-id="f63b8-442">If a web API is subject to approval, prior to being able to access it a developer must send a request to the product administrator.</span></span> <span data-ttu-id="f63b8-443">管理员可以授予或拒绝开发人员的访问权限。</span><span class="sxs-lookup"><span data-stu-id="f63b8-443">The administrator can grant or deny access to the developer.</span></span> <span data-ttu-id="f63b8-444">如果情况发生变化，也可以阻止现有开发人员。</span><span class="sxs-lookup"><span data-stu-id="f63b8-444">Existing developers can also be blocked if circumstances change.</span></span>
   >
   >
6. <span data-ttu-id="f63b8-445">为每个 Web API 配置策略。</span><span class="sxs-lookup"><span data-stu-id="f63b8-445">Configure policies for each web API.</span></span> <span data-ttu-id="f63b8-446">策略可以控制以下方面：是否应允许跨域调用、如何对客户端进行身份验证、是否要在 XML 和 JSON 数据格式之间透明地进行转换、是否要限制从给定 IP 范围发起的调用、使用配额，以及是否要限制调用率等。</span><span class="sxs-lookup"><span data-stu-id="f63b8-446">Policies govern aspects such as whether cross-domain calls should be allowed, how to authenticate clients, whether to convert between XML and JSON data formats transparently, whether to restrict calls from a given IP range, usage quotas, and whether to limit the call rate.</span></span> <span data-ttu-id="f63b8-447">策略可以对整个产品全局应用、对产品中的单个 Web API 应用，或者对 Web API 中的单个操作应用。</span><span class="sxs-lookup"><span data-stu-id="f63b8-447">Policies can be applied globally across the entire product, for a single web API in a product, or for individual operations in a web API.</span></span>

<span data-ttu-id="f63b8-448">有关详细信息，请参阅 [API 管理文档](/azure/api-management/)。</span><span class="sxs-lookup"><span data-stu-id="f63b8-448">For more information, see the [API Management Documentation](/azure/api-management/).</span></span> 

> [!TIP]
> <span data-ttu-id="f63b8-449">Azure 提供了使用 Azure 流量管理器，使用它可以实现故障转移和负载均衡，并可以减少在不同地理位置托管的多个网站实例之间的延迟。</span><span class="sxs-lookup"><span data-stu-id="f63b8-449">Azure provides the Azure Traffic Manager which enables you to implement failover and load-balancing, and reduce latency across multiple instances of a web site hosted in different geographic locations.</span></span> <span data-ttu-id="f63b8-450">可以将 Azure 流量管理器与 API 管理服务结合使用；API 管理服务可以通过 Azure 流量管理器将请求路由到网站实例。</span><span class="sxs-lookup"><span data-stu-id="f63b8-450">You can use Azure Traffic Manager in conjunction with the API Management Service; the API Management Service can route requests to instances of a web site through Azure Traffic Manager.</span></span>  <span data-ttu-id="f63b8-451">有关详细信息，请参阅[流量管理器路由方法](/azure/traffic-manager/traffic-manager-routing-methods/)。</span><span class="sxs-lookup"><span data-stu-id="f63b8-451">For more information, see [Traffic Manager routing Methods](/azure/traffic-manager/traffic-manager-routing-methods/).</span></span>
>
> <span data-ttu-id="f63b8-452">在此结构中，如果要对网站使用自定义 DNS 名称，则应将每个网站的相应 CNAME 记录配置为指向 Azure 流量管理器网站的 DNS 名称。</span><span class="sxs-lookup"><span data-stu-id="f63b8-452">In this structure, if you are using custom DNS names for your web sites, you should configure the appropriate CNAME record for each web site to point to the DNS name of the Azure Traffic Manager web site.</span></span>
>

## <a name="support-developers-building-client-applications"></a><span data-ttu-id="f63b8-453">为构建客户端应用程序的开发人员提供支持</span><span class="sxs-lookup"><span data-stu-id="f63b8-453">Support developers building client applications</span></span>
<span data-ttu-id="f63b8-454">构造客户端应用程序的开发人员通常需要了解有关如何访问 Web API 和与参数、数据类型、返回类型和返回代码（描述 Web 服务和客户端应用程序之间的不同请求和响应）相关的文档的信息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-454">Developers constructing client applications typically require information on how to access the web API, and documentation concerning the parameters, data types, return types, and return codes that describe the different requests and responses between the web service and the client application.</span></span>

### <a name="document-the-rest-operations-for-a-web-api"></a><span data-ttu-id="f63b8-455">记录 Web API 的 REST 操作</span><span class="sxs-lookup"><span data-stu-id="f63b8-455">Document the REST operations for a web API</span></span>
<span data-ttu-id="f63b8-456">Azure API 管理服务包括一个开发人员门户，其中描述了由 Web API 公开的 REST 操作。</span><span class="sxs-lookup"><span data-stu-id="f63b8-456">The Azure API Management Service includes a developer portal that describes the REST operations exposed by a web API.</span></span> <span data-ttu-id="f63b8-457">产品发布后，便会显示在此门户上。</span><span class="sxs-lookup"><span data-stu-id="f63b8-457">When a product has been published it appears on this portal.</span></span> <span data-ttu-id="f63b8-458">开发人员可以使用此门户注册访问；然后，管理员可以批准或拒绝该请求。</span><span class="sxs-lookup"><span data-stu-id="f63b8-458">Developers can use this portal to sign up for access; the administrator can then approve or deny the request.</span></span> <span data-ttu-id="f63b8-459">如果开发人员获得批准，则会向其分配一个订阅密钥，用于对所开发的客户端应用程序发出的调用进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="f63b8-459">If the developer is approved, they are assigned a subscription key that is used to authenticate calls from the client applications that they develop.</span></span> <span data-ttu-id="f63b8-460">此密钥必须与每个 Web API 调用一起提供，否则会被拒绝。</span><span class="sxs-lookup"><span data-stu-id="f63b8-460">This key must be provided with each web API call otherwise it will be rejected.</span></span>

<span data-ttu-id="f63b8-461">此门户还提供了：</span><span class="sxs-lookup"><span data-stu-id="f63b8-461">This portal also provides:</span></span>

* <span data-ttu-id="f63b8-462">产品的文档，列出它公开的操作、所需参数和可以返回的不同响应。</span><span class="sxs-lookup"><span data-stu-id="f63b8-462">Documentation for the product, listing the operations that it exposes, the parameters required, and the different responses that can be returned.</span></span> <span data-ttu-id="f63b8-463">请注意，此信息从通过使用 Microsoft Azure API 管理服务发布 Web API 部分的列表中的步骤 3 所提供的详细信息生成。</span><span class="sxs-lookup"><span data-stu-id="f63b8-463">Note that this information is generated from the details provided in step 3 in the list in the Publishing a web API by using the Microsoft Azure API Management Service section.</span></span>
* <span data-ttu-id="f63b8-464">演示如何通过多种语言（包括 JavaScript、C#、Java、Ruby、Python 和 PHP）调用操作的代码片段。</span><span class="sxs-lookup"><span data-stu-id="f63b8-464">Code snippets that show how to invoke operations from several languages, including JavaScript, C#, Java, Ruby, Python, and PHP.</span></span>
* <span data-ttu-id="f63b8-465">开发人员使用开发人员控制台，能够发送 HTTP 请求以测试产品中的每个操作并查看结果。</span><span class="sxs-lookup"><span data-stu-id="f63b8-465">A developers' console that enables a developer to send an HTTP request to test each operation in the product and view the results.</span></span>
* <span data-ttu-id="f63b8-466">在此页中开发人员可以报告发现的任何问题。</span><span class="sxs-lookup"><span data-stu-id="f63b8-466">A page where the developer can report any issues or problems found.</span></span>

<span data-ttu-id="f63b8-467">使用 Azure 管理门户可以自定义开发人员门户，以便更改样式和布局以匹配组织的品牌。</span><span class="sxs-lookup"><span data-stu-id="f63b8-467">The Azure Management portal enables you to customize the developer portal to change the styling and layout to match the branding of your organization.</span></span>

### <a name="implement-a-client-sdk"></a><span data-ttu-id="f63b8-468">实现客户端 SDK</span><span class="sxs-lookup"><span data-stu-id="f63b8-468">Implement a client SDK</span></span>
<span data-ttu-id="f63b8-469">构建调用 REST 请求以访问 Web API 的客户端应用程序需要编写大量代码来构造每个请求并相应地设置其格式，将请求发送到托管 Web 服务的服务器，分析响应以确定请求是成功还是失败并提取返回的任何数据。</span><span class="sxs-lookup"><span data-stu-id="f63b8-469">Building a client application that invokes REST requests to access a web API requires writing a significant amount of code to construct each request and format it appropriately, send the request to the server hosting the web service, and parse the response to work out whether the request succeeded or failed and extract any data returned.</span></span> <span data-ttu-id="f63b8-470">要使客户端应用程序免除这些问题，可以提供这样一个 SDK：包装 REST 接口并在一组功能更强的方法内抽象这些低级别的详细信息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-470">To insulate the client application from these concerns, you can provide an SDK that wraps the REST interface and abstracts these low-level details inside a more functional set of methods.</span></span> <span data-ttu-id="f63b8-471">客户端应用程序使用这些方法，这些方法透明地将调用转换为 REST 请求，然后将响应转换回方法返回值。</span><span class="sxs-lookup"><span data-stu-id="f63b8-471">A client application uses these methods, which transparently convert calls into REST requests and then convert the responses back into method return values.</span></span> <span data-ttu-id="f63b8-472">这是许多服务（包括 Azure SDK）已实现的一种常用技术。</span><span class="sxs-lookup"><span data-stu-id="f63b8-472">This is a common technique that is implemented by many services, including the Azure SDK.</span></span>

<span data-ttu-id="f63b8-473">创建客户端 SDK 是一项要求相当高的任务，因为它必须一致地实现，并经过严格测试。</span><span class="sxs-lookup"><span data-stu-id="f63b8-473">Creating a client-side SDK is a considerable undertaking as it has to be implemented consistently and tested carefully.</span></span> <span data-ttu-id="f63b8-474">但是，此过程的大部分操作可以机械地进行，并且许多供应商提供了可自动执行上述许多任务的工具。</span><span class="sxs-lookup"><span data-stu-id="f63b8-474">However, much of this process can be made mechanical, and many vendors supply tools that can automate many of these tasks.</span></span>

## <a name="monitoring-a-web-api"></a><span data-ttu-id="f63b8-475">监视 Web API</span><span class="sxs-lookup"><span data-stu-id="f63b8-475">Monitoring a web API</span></span>
<span data-ttu-id="f63b8-476">根据你发布和部署 Web API 的方式，可以直接监视 Web API，也可以分析通过 API 管理服务传递的流量来收集使用情况和运行状况信息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-476">Depending on how you have published and deployed your web API you can monitor the web API directly, or you can gather usage and health information by analyzing the traffic that passes through the API Management service.</span></span>

### <a name="monitoring-a-web-api-directly"></a><span data-ttu-id="f63b8-477">直接监视 Web API</span><span class="sxs-lookup"><span data-stu-id="f63b8-477">Monitoring a web API directly</span></span>
<span data-ttu-id="f63b8-478">如果已通过使用 ASP.NET Web API 模板（不管作为 Web API 项目还是作为 Azure 云服务中的 Web 角色）和 Visual Studio 2013 实现 Web API，则可以通过使用 ASP.NET Application Insights 收集可用性、性能和使用情况数据。</span><span class="sxs-lookup"><span data-stu-id="f63b8-478">If you have implemented your web API by using the ASP.NET Web API template (either as a Web API project or as a Web role in an Azure cloud service) and Visual Studio 2013, you can gather availability, performance, and usage data by using ASP.NET Application Insights.</span></span> <span data-ttu-id="f63b8-479">Application Insights 是在 Web API 部署到云中后透明地跟踪和记录有关请求和响应的信息的程序包；安装并配置该包后，无需修改 Web API 中的任何代码即可使用它。</span><span class="sxs-lookup"><span data-stu-id="f63b8-479">Application Insights is a package that transparently tracks and records information about requests and responses when the web API is deployed to the cloud; once the package is installed and configured, you don't need to amend any code in your web API to use it.</span></span> <span data-ttu-id="f63b8-480">将 Web API 部署到 Azure 网站时，会检查所有通信并收集以下统计信息：</span><span class="sxs-lookup"><span data-stu-id="f63b8-480">When you deploy the web API to an Azure web site, all traffic is examined and the following statistics are gathered:</span></span>

* <span data-ttu-id="f63b8-481">服务器响应时间。</span><span class="sxs-lookup"><span data-stu-id="f63b8-481">Server response time.</span></span>
* <span data-ttu-id="f63b8-482">服务器请求数和每个请求的详细信息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-482">Number of server requests and the details of each request.</span></span>
* <span data-ttu-id="f63b8-483">就平均响应时间而言，速度最慢的前几个请求。</span><span class="sxs-lookup"><span data-stu-id="f63b8-483">The top slowest requests in terms of average response time.</span></span>
* <span data-ttu-id="f63b8-484">任何失败的请求的详细信息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-484">The details of any failed requests.</span></span>
* <span data-ttu-id="f63b8-485">由不同浏览器和用户代理启动的会话数。</span><span class="sxs-lookup"><span data-stu-id="f63b8-485">The number of sessions initiated by different browsers and user agents.</span></span>
* <span data-ttu-id="f63b8-486">最经常查看的网页（主要适用于 Web 应用程序而不是 Web API）。</span><span class="sxs-lookup"><span data-stu-id="f63b8-486">The most frequently viewed pages (primarily useful for web applications rather than web APIs).</span></span>
* <span data-ttu-id="f63b8-487">访问 Web API 的不同用户角色。</span><span class="sxs-lookup"><span data-stu-id="f63b8-487">The different user roles accessing the web API.</span></span>

<span data-ttu-id="f63b8-488">可以从 Azure 管理门户实时查看此数据。</span><span class="sxs-lookup"><span data-stu-id="f63b8-488">You can view this data in real time from the Azure Management portal.</span></span> <span data-ttu-id="f63b8-489">此外，还可以创建监视 Web API 的运行状况的 webtest。</span><span class="sxs-lookup"><span data-stu-id="f63b8-489">You can also create webtests that monitor the health of the web API.</span></span> <span data-ttu-id="f63b8-490">Webtest 将定期请求发送到 Web API 中指定的 URI，并捕获响应。</span><span class="sxs-lookup"><span data-stu-id="f63b8-490">A webtest sends a periodic request to a specified URI in the web API and captures the response.</span></span> <span data-ttu-id="f63b8-491">可以指定成功响应的定义（如 HTTP 状态代码 200），如果请求未返回此响应，可以安排向管理员发送警报。</span><span class="sxs-lookup"><span data-stu-id="f63b8-491">You can specify the definition of a successful response (such as HTTP status code 200), and if the request does not return this response you can arrange for an alert to be sent to an administrator.</span></span> <span data-ttu-id="f63b8-492">如有必要，管理员可以重新启动托管 Web API 的服务器（如果它出现故障）。</span><span class="sxs-lookup"><span data-stu-id="f63b8-492">If necessary, the administrator can restart the server hosting the web API if it has failed.</span></span>

<span data-ttu-id="f63b8-493">有关详细信息，请参阅 [Application Insights - ASP.NET 入门](/azure/application-insights/app-insights-asp-net/)。</span><span class="sxs-lookup"><span data-stu-id="f63b8-493">For more information, see [Application Insights - Get started with ASP.NET](/azure/application-insights/app-insights-asp-net/).</span></span>

### <a name="monitoring-a-web-api-through-the-api-management-service"></a><span data-ttu-id="f63b8-494">通过 API 管理服务监视 Web API</span><span class="sxs-lookup"><span data-stu-id="f63b8-494">Monitoring a web API through the API Management Service</span></span>
<span data-ttu-id="f63b8-495">如果已通过使用 API 管理服务发布 Web API，则 Azure 管理门户上的 API 管理页包含一个可用于查看该服务的整体性能的仪表板。</span><span class="sxs-lookup"><span data-stu-id="f63b8-495">If you have published your web API by using the API Management service, the API Management page on the Azure Management portal contains a dashboard that enables you to view the overall performance of the service.</span></span> <span data-ttu-id="f63b8-496">使用“分析”页，可向下钻取到该产品的使用方式的详细信息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-496">The Analytics page enables you to drill down into the details of how the product is being used.</span></span> <span data-ttu-id="f63b8-497">此页包含以下选项卡：</span><span class="sxs-lookup"><span data-stu-id="f63b8-497">This page contains the following tabs:</span></span>

* <span data-ttu-id="f63b8-498">**用法**。</span><span class="sxs-lookup"><span data-stu-id="f63b8-498">**Usage**.</span></span> <span data-ttu-id="f63b8-499">此选项卡提供有关随着时间推移已进行的 API 调用数和用于处理这些调用的带宽的信息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-499">This tab provides information about the number of API calls made and the bandwidth used to handle these calls over time.</span></span> <span data-ttu-id="f63b8-500">可以按产品、API 和操作筛选使用情况详细信息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-500">You can filter usage details by product, API, and operation.</span></span>
* <span data-ttu-id="f63b8-501">**运行状况**。</span><span class="sxs-lookup"><span data-stu-id="f63b8-501">**Health**.</span></span> <span data-ttu-id="f63b8-502">使用此选项卡可查看 API 请求的结果（返回的 HTTP 状态代码）、缓存策略的有效性、API 响应时间和服务响应时间。</span><span class="sxs-lookup"><span data-stu-id="f63b8-502">This tab enables you to view the outcome of API requests (the HTTP status codes returned), the effectiveness of the caching policy, the API response time, and the service response time.</span></span> <span data-ttu-id="f63b8-503">同样，可以按产品、API 和操作筛选运行状况数据。</span><span class="sxs-lookup"><span data-stu-id="f63b8-503">Again, you can filter health data by product, API, and operation.</span></span>
* <span data-ttu-id="f63b8-504">**活动**。</span><span class="sxs-lookup"><span data-stu-id="f63b8-504">**Activity**.</span></span> <span data-ttu-id="f63b8-505">此选项卡提供了以下信息的文本摘要：成功的调用数、失败的调用数、阻止的调用数、平均响应时间以及每个产品、Web API 和操作的响应时间。</span><span class="sxs-lookup"><span data-stu-id="f63b8-505">This tab provides a text summary of the numbers of successful calls, failed calls, blocked calls, average response time, and response times for each product, web API, and operation.</span></span> <span data-ttu-id="f63b8-506">此页还列出了每个开发人员进行的调用数。</span><span class="sxs-lookup"><span data-stu-id="f63b8-506">This page also lists the number of calls made by each developer.</span></span>
* <span data-ttu-id="f63b8-507">**速览**。</span><span class="sxs-lookup"><span data-stu-id="f63b8-507">**At a glance**.</span></span> <span data-ttu-id="f63b8-508">此选项卡显示性能数据的摘要，包括负责进行大多数 API 调用的开发人员，以及接收这些调用的产品、Web API 和操作。</span><span class="sxs-lookup"><span data-stu-id="f63b8-508">This tab displays a summary of the performance data, including the developers responsible for making the most API calls, and the products, web APIs, and operations that received these calls.</span></span>

<span data-ttu-id="f63b8-509">可以使用此信息来确定是否是特定 Web API 或操作导致了瓶颈问题，如有必要，扩展主机环境并添加更多服务器。</span><span class="sxs-lookup"><span data-stu-id="f63b8-509">You can use this information to determine whether a particular web API or operation is causing a bottleneck, and if necessary scale the host environment and add more servers.</span></span> <span data-ttu-id="f63b8-510">还可以确定一个或多个应用程序是否正在使用不相称的资源量，从而应用适当的策略以设置配额并限制调用率。</span><span class="sxs-lookup"><span data-stu-id="f63b8-510">You can also ascertain whether one or more applications are using a disproportionate volume of resources and apply the appropriate policies to set quotas and limit call rates.</span></span>

> [!NOTE]
> <span data-ttu-id="f63b8-511">可以更改已发布产品的详细信息，所做的更改将立即应用。</span><span class="sxs-lookup"><span data-stu-id="f63b8-511">You can change the details for a published product, and the changes are applied immediately.</span></span> <span data-ttu-id="f63b8-512">例如，可以在 Web API 中添加或删除操作，而无需重新发布包含该 Web API 的产品。</span><span class="sxs-lookup"><span data-stu-id="f63b8-512">For example, you can add or remove an operation from a web API without requiring that you republish the product that contains the web API.</span></span>
>
>

## <a name="more-information"></a><span data-ttu-id="f63b8-513">详细信息</span><span class="sxs-lookup"><span data-stu-id="f63b8-513">More information</span></span>
* <span data-ttu-id="f63b8-514">[ASP.NET Web API OData](http://www.asp.net/web-api/overview/odata-support-in-aspnet-web-api) 包含有关如何使用 ASP.NET 实现 OData Web API 的示例和更多信息。</span><span class="sxs-lookup"><span data-stu-id="f63b8-514">[ASP.NET Web API OData](http://www.asp.net/web-api/overview/odata-support-in-aspnet-web-api) contains examples and further information on implementing an OData web API by using ASP.NET.</span></span>
* <span data-ttu-id="f63b8-515">[Introducing Batch Support in Web API and Web API OData](http://blogs.msdn.com/b/webdev/archive/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata.aspx)（Web API 和 Web API OData 中的批处理支持简介）介绍了如何使用 OData 在 Web API 中实现批处理操作。</span><span class="sxs-lookup"><span data-stu-id="f63b8-515">[Introducing Batch Support in Web API and Web API OData](http://blogs.msdn.com/b/webdev/archive/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata.aspx) describes how to implement batch operations in a web API by using OData.</span></span>
* <span data-ttu-id="f63b8-516">Jonathan Oliver 博客上的 [Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/)（幂等性模式）概述了幂等性以及它如何与数据管理操作相关。</span><span class="sxs-lookup"><span data-stu-id="f63b8-516">[Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) on Jonathan Oliver’s blog provides an overview of idempotency and how it relates to data management operations.</span></span>
* <span data-ttu-id="f63b8-517">W3C 网站上的 [Status Code Definitions](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)（状态代码定义）包含 HTTP 状态代码及其说明的完整列表。</span><span class="sxs-lookup"><span data-stu-id="f63b8-517">[Status Code Definitions](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) on the W3C website contains a full list of HTTP status codes and their descriptions.</span></span>
* <span data-ttu-id="f63b8-518">[使用 WebJobs 运行后台任务](/azure/app-service-web/web-sites-create-web-jobs/)提供了有关如何使用 WebJobs 执行后台操作的信息和示例。</span><span class="sxs-lookup"><span data-stu-id="f63b8-518">[Run background tasks with WebJobs](/azure/app-service-web/web-sites-create-web-jobs/) provides information and examples on using WebJobs to perform background operations.</span></span>
* <span data-ttu-id="f63b8-519">[Azure 通知中心通知用户](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/)介绍了如何使用 Azure 通知中心将异步响应推送到客户端应用程序。</span><span class="sxs-lookup"><span data-stu-id="f63b8-519">[Azure Notification Hubs Notify Users](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/) shows how to use an Azure Notification Hub to push asynchronous responses to client applications.</span></span>
* <span data-ttu-id="f63b8-520">[API 管理](https://azure.microsoft.com/services/api-management/)介绍了如何发布可对 Web API 进行受控安全访问的产品。</span><span class="sxs-lookup"><span data-stu-id="f63b8-520">[API Management](https://azure.microsoft.com/services/api-management/) describes how to publish a product that provides controlled and secure access to a web API.</span></span>
* <span data-ttu-id="f63b8-521">[Azure API Management REST API Reference](https://msdn.microsoft.com/library/azure/dn776326.aspx)（Azure API 管理 REST API 参考）介绍了如何使用 API 管理 REST API 生成自定义管理应用程序。</span><span class="sxs-lookup"><span data-stu-id="f63b8-521">[Azure API Management REST API Reference](https://msdn.microsoft.com/library/azure/dn776326.aspx) describes how to use the API Management REST API to build custom management applications.</span></span>
* <span data-ttu-id="f63b8-522">[流量管理器路由方法](/azure/traffic-manager/traffic-manager-routing-methods/)概述了如何使用 Azure 流量管理器对托管 Web API 的多个网站实例上的请求进行负载均衡操作。</span><span class="sxs-lookup"><span data-stu-id="f63b8-522">[Traffic Manager Routing Methods](/azure/traffic-manager/traffic-manager-routing-methods/) summarizes how Azure Traffic Manager can be used to load-balance requests across multiple instances of a website hosting a web API.</span></span>
* <span data-ttu-id="f63b8-523">[Application Insights - ASP.NET 入门](/azure/application-insights/app-insights-asp-net/)详细介绍了如何在 ASP.NET Web API 项目中安装和配置 Application Insights。</span><span class="sxs-lookup"><span data-stu-id="f63b8-523">[Application Insights - Get started with ASP.NET](/azure/application-insights/app-insights-asp-net/) provides detailed information on installing and configuring Application Insights in an ASP.NET Web API project.</span></span>


<!-- links -->

[api-design]: ./api-design.md
