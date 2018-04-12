---
title: 运行状况终结点监视
description: 在应用程序中实施可让外部工具通过公开终结点定期访问的功能检查。
keywords: 设计模式
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- availability
- management-monitoring
- resiliency
ms.openlocfilehash: 3b3bce46b460148af17bfe6064cd052a5f9a6458
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/06/2018
---
# <a name="health-endpoint-monitoring-pattern"></a><span data-ttu-id="a39ff-104">运行状况终结点监视模式</span><span class="sxs-lookup"><span data-stu-id="a39ff-104">Health Endpoint Monitoring pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="a39ff-105">在应用程序中实施可让外部工具通过公开终结点定期访问的功能检查。</span><span class="sxs-lookup"><span data-stu-id="a39ff-105">Implement functional checks in an application that external tools can access through exposed endpoints at regular intervals.</span></span> <span data-ttu-id="a39ff-106">这有助于验证应用程序和服务的运行是否正常。</span><span class="sxs-lookup"><span data-stu-id="a39ff-106">This can help to verify that applications and services are performing correctly.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="a39ff-107">上下文和问题</span><span class="sxs-lookup"><span data-stu-id="a39ff-107">Context and problem</span></span>

<span data-ttu-id="a39ff-108">一种良好的做法（通常也是一项业务要求）是监视 Web 应用程序和后端服务，以确保它们保持可用且正常运行。</span><span class="sxs-lookup"><span data-stu-id="a39ff-108">It's a good practice, and often a business requirement, to monitor web applications and back-end services, to ensure they're available and performing correctly.</span></span> <span data-ttu-id="a39ff-109">但是，监视云中运行的服务比监视本地服务更困难。</span><span class="sxs-lookup"><span data-stu-id="a39ff-109">However, it's more difficult to monitor services running in the cloud than it is to monitor on-premises services.</span></span> <span data-ttu-id="a39ff-110">例如，无法完全控制宿主环境，并且服务通常依赖于平台供应商和其他公司提供的其他服务。</span><span class="sxs-lookup"><span data-stu-id="a39ff-110">For example, you don't have full control of the hosting environment, and the services typically depend on other services provided by platform vendors and others.</span></span>

<span data-ttu-id="a39ff-111">有许多因素会影响云托管的应用程序，例如网络延迟、底层计算与存储系统的性能和可用性，以及这些系统之间的网络带宽。</span><span class="sxs-lookup"><span data-stu-id="a39ff-111">There are many factors that affect cloud-hosted applications such as network latency, the performance and availability of the underlying compute and storage systems, and the network bandwidth between them.</span></span> <span data-ttu-id="a39ff-112">由于其中的任何因素，服务可能发生整体性或局部性的故障。</span><span class="sxs-lookup"><span data-stu-id="a39ff-112">The service can fail entirely or partially due to any of these factors.</span></span> <span data-ttu-id="a39ff-113">因此，必须定期验证服务是否正确运行，确保提供所需级别的可用性，这可能是服务级别协议 (SLA) 所涵盖的要求。</span><span class="sxs-lookup"><span data-stu-id="a39ff-113">Therefore, you must verify at regular intervals that the service is performing correctly to ensure the required level of availability, which might be part of your service level agreement (SLA).</span></span>

## <a name="solution"></a><span data-ttu-id="a39ff-114">解决方案</span><span class="sxs-lookup"><span data-stu-id="a39ff-114">Solution</span></span>

<span data-ttu-id="a39ff-115">通过将请求发送到应用程序上的终结点来实施运行状况监视。</span><span class="sxs-lookup"><span data-stu-id="a39ff-115">Implement health monitoring by sending requests to an endpoint on the application.</span></span> <span data-ttu-id="a39ff-116">应用程序应执行必要的检查，并返回其状态指示。</span><span class="sxs-lookup"><span data-stu-id="a39ff-116">The application should perform the necessary checks, and return an indication of its status.</span></span>

<span data-ttu-id="a39ff-117">运行状况监视检查通常结合了两个因素：</span><span class="sxs-lookup"><span data-stu-id="a39ff-117">A health monitoring check typically combines two factors:</span></span>

- <span data-ttu-id="a39ff-118">应用程序或服务在响应针对运行状况验证终结点的请求时执行的检查（如果有）。</span><span class="sxs-lookup"><span data-stu-id="a39ff-118">The checks (if any) performed by the application or service in response to the request to the health verification endpoint.</span></span>
- <span data-ttu-id="a39ff-119">通过执行运行状况验证检查的工具或框架分析结果。</span><span class="sxs-lookup"><span data-stu-id="a39ff-119">Analysis of the results by the tool or framework that performs the health verification check.</span></span>

<span data-ttu-id="a39ff-120">响应代码指示应用程序的状态，并选择性地指示应用程序使用的任何组件或服务的状态。</span><span class="sxs-lookup"><span data-stu-id="a39ff-120">The response code indicates the status of the application and, optionally, any components or services it uses.</span></span> <span data-ttu-id="a39ff-121">延迟或响应时间检查由监视工具或框架执行。</span><span class="sxs-lookup"><span data-stu-id="a39ff-121">The latency or response time check is performed by the monitoring tool or framework.</span></span> <span data-ttu-id="a39ff-122">下图提供了模式概览。</span><span class="sxs-lookup"><span data-stu-id="a39ff-122">The figure provides an overview of the pattern.</span></span>

![模式概览](./_images/health-endpoint-monitoring-pattern.png)

<span data-ttu-id="a39ff-124">应用程序中运行状况监视代码可能执行的其他检查包括：</span><span class="sxs-lookup"><span data-stu-id="a39ff-124">Other checks that might be carried out by the health monitoring code in the application include:</span></span>
- <span data-ttu-id="a39ff-125">检查云存储或数据库的可用性和响应时间。</span><span class="sxs-lookup"><span data-stu-id="a39ff-125">Checking cloud storage or a database for availability and response time.</span></span>
- <span data-ttu-id="a39ff-126">检查位于应用程序中的或者位于其他位置、但由应用程序使用的其他资源或服务。</span><span class="sxs-lookup"><span data-stu-id="a39ff-126">Checking other resources or services located in the application, or located elsewhere but used by the application.</span></span>

<span data-ttu-id="a39ff-127">可以使用某些服务和工具，通过将请求提交到一组可配置的终结点，并根据一组可配置的规则评估结果，来监视 Web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="a39ff-127">Services and tools are available that monitor web applications by submitting a request to a configurable set of endpoints, and evaluating the results against a set of configurable rules.</span></span> <span data-ttu-id="a39ff-128">创建一个只是在系统上执行某些功能测试的服务终结点相对较为容易。</span><span class="sxs-lookup"><span data-stu-id="a39ff-128">It's relatively easy to create a service endpoint whose sole purpose is to perform some functional tests on the system.</span></span>

<span data-ttu-id="a39ff-129">监视工具可执行的典型检查包括：</span><span class="sxs-lookup"><span data-stu-id="a39ff-129">Typical checks that can be performed by the monitoring tools include:</span></span>

- <span data-ttu-id="a39ff-130">验证响应代码。</span><span class="sxs-lookup"><span data-stu-id="a39ff-130">Validating the response code.</span></span> <span data-ttu-id="a39ff-131">例如，HTTP 响应 200（正常）表示应用程序已做出响应且未出错。</span><span class="sxs-lookup"><span data-stu-id="a39ff-131">For example, an HTTP response of 200 (OK) indicates that the application responded without error.</span></span> <span data-ttu-id="a39ff-132">监视系统可能还会检查其他响应代码以提供更全面的结果。</span><span class="sxs-lookup"><span data-stu-id="a39ff-132">The monitoring system might also check for other response codes to give more comprehensive results.</span></span>
- <span data-ttu-id="a39ff-133">即使返回了 200（正常）状态代码，也会检查响应内容以检测错误。</span><span class="sxs-lookup"><span data-stu-id="a39ff-133">Checking the content of the response to detect errors, even when a 200 (OK) status code is returned.</span></span> <span data-ttu-id="a39ff-134">这可以检测仅影响所返回网页的某个部分或服务响应的错误。</span><span class="sxs-lookup"><span data-stu-id="a39ff-134">This can detect errors that affect only a section of the returned web page or service response.</span></span> <span data-ttu-id="a39ff-135">例如，检查页面标题，或查找指示返回了正确页面的特定短语。</span><span class="sxs-lookup"><span data-stu-id="a39ff-135">For example, checking the title of a page or looking for a specific phrase that indicates the correct page was returned.</span></span>
- <span data-ttu-id="a39ff-136">测量响应时间，即网络延迟与应用程序执行请求所花费时间的合计。</span><span class="sxs-lookup"><span data-stu-id="a39ff-136">Measuring the response time, which indicates a combination of the network latency and the time that the application took to execute the request.</span></span> <span data-ttu-id="a39ff-137">如果值增大，则可能表示应用程序或网络正在出现问题。</span><span class="sxs-lookup"><span data-stu-id="a39ff-137">An increasing value can indicate an emerging problem with the application or network.</span></span>
- <span data-ttu-id="a39ff-138">检查位于应用程序外部的资源或服务，例如应用程序用来交付全局缓存中的内容的内容交付网络。</span><span class="sxs-lookup"><span data-stu-id="a39ff-138">Checking resources or services located outside the application, such as a content delivery network used by the application to deliver content from global caches.</span></span>
- <span data-ttu-id="a39ff-139">检查 SSL 证书过期时间。</span><span class="sxs-lookup"><span data-stu-id="a39ff-139">Checking for expiration of SSL certificates.</span></span>
- <span data-ttu-id="a39ff-140">测量针对应用程序 URL 执行 DNS 查找所花费的响应时间，以测量 DNS 延迟和 DNS 故障。</span><span class="sxs-lookup"><span data-stu-id="a39ff-140">Measuring the response time of a DNS lookup for the URL of the application to measure DNS latency and DNS failures.</span></span>
- <span data-ttu-id="a39ff-141">验证 DNS 查找返回的 URL 以确保条目正确。</span><span class="sxs-lookup"><span data-stu-id="a39ff-141">Validating the URL returned by the DNS lookup to ensure correct entries.</span></span> <span data-ttu-id="a39ff-142">这有助于避免恶意用户通过成功攻击 DNS 服务器进行请求重定向。</span><span class="sxs-lookup"><span data-stu-id="a39ff-142">This can help to avoid malicious request redirection through a successful attack on the DNS server.</span></span>

<span data-ttu-id="a39ff-143">在可能的情况下，从不同的本地位置或托管位置运行这些检查来测量和比较响应时间也很有用。</span><span class="sxs-lookup"><span data-stu-id="a39ff-143">It's also useful, where possible, to run these checks from different on-premises or hosted locations to measure and compare response times.</span></span> <span data-ttu-id="a39ff-144">理想情况下，应该从靠近客户的位置监视应用程序，以获取每个位置的准确性能视图。</span><span class="sxs-lookup"><span data-stu-id="a39ff-144">Ideally you should monitor applications from locations that are close to customers to get an accurate view of the performance from each location.</span></span> <span data-ttu-id="a39ff-145">除了提供更可靠的检查机制以外，结果可以帮助确定应用程序的部署位置，以及是否将其部署到多个数据中心。</span><span class="sxs-lookup"><span data-stu-id="a39ff-145">In addition to providing a more robust checking mechanism, the results can help you decide on the deployment location for the application&mdash;and whether to deploy it in more than one datacenter.</span></span>

<span data-ttu-id="a39ff-146">此外，还应针对客户使用的所有服务实例运行测试，确保所有客户可正常运行应用程序。</span><span class="sxs-lookup"><span data-stu-id="a39ff-146">Tests should also be run against all the service instances that customers use to ensure the application is working correctly for all customers.</span></span> <span data-ttu-id="a39ff-147">例如，如果客户存储分散在多个存储帐户中，则监视过程应检查所有这些存储帐户。</span><span class="sxs-lookup"><span data-stu-id="a39ff-147">For example, if customer storage is spread across more than one storage account, the monitoring process should check all of these.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="a39ff-148">问题和注意事项</span><span class="sxs-lookup"><span data-stu-id="a39ff-148">Issues and considerations</span></span>

<span data-ttu-id="a39ff-149">在决定如何实现此模式时，请考虑以下几点：</span><span class="sxs-lookup"><span data-stu-id="a39ff-149">Consider the following points when deciding how to implement this pattern:</span></span>

<span data-ttu-id="a39ff-150">如何验证响应。</span><span class="sxs-lookup"><span data-stu-id="a39ff-150">How to validate the response.</span></span> <span data-ttu-id="a39ff-151">例如，仅凭一个 200（正常）状态代码是否足以确认应用程序在正常运行？</span><span class="sxs-lookup"><span data-stu-id="a39ff-151">For example, is just a single 200 (OK) status code sufficient to verify the application is working correctly?</span></span> <span data-ttu-id="a39ff-152">尽管这是应用程序可用性的最基本测量方法，并且是此模式的最简单实现方式，但是，它在应用程序中的操作、趋势和可能即将出现的问题方面提供的信息极少。</span><span class="sxs-lookup"><span data-stu-id="a39ff-152">While this provides the most basic measure of application availability, and is the minimum implementation of this pattern, it provides little information about the operations, trends, and possible upcoming issues in the application.</span></span>

   >  <span data-ttu-id="a39ff-153">确保应用程序仅在找到并处理目标资源时才正确返回 200（正常）。</span><span class="sxs-lookup"><span data-stu-id="a39ff-153">Make sure that the application correctly returns a 200 (OK) only when the target resource is found and processed.</span></span> <span data-ttu-id="a39ff-154">在某些情况下（例如，使用母版页托管目标网页时），即使未找到目标内容页，服务器也会发回 200（正常）状态代码而不是 404（未找到）代码。</span><span class="sxs-lookup"><span data-stu-id="a39ff-154">In some scenarios, such as when using a master page to host the target web page, the server sends back a 200 (OK) status code instead of a 404 (Not Found) code, even when the target content page was not found.</span></span>

<span data-ttu-id="a39ff-155">要为应用程序公开的终结点数。</span><span class="sxs-lookup"><span data-stu-id="a39ff-155">The number of endpoints to expose for an application.</span></span> <span data-ttu-id="a39ff-156">一种方法是将针对应用程序使用的核心服务至少公开一个终结点，并为低优先级服务公开另一个终结点，以便将不同级别的重要性分配给每项监视结果。</span><span class="sxs-lookup"><span data-stu-id="a39ff-156">One approach is to expose at least one endpoint for the core services that the application uses and another for lower priority services, allowing different levels of importance to be assigned to each monitoring result.</span></span> <span data-ttu-id="a39ff-157">此外，请考虑公开更多的终结点，例如，为每个核心服务公开一个终结点，以获得更高的监视粒度。</span><span class="sxs-lookup"><span data-stu-id="a39ff-157">Also consider exposing more endpoints, such as one for each core service, for additional monitoring granularity.</span></span> <span data-ttu-id="a39ff-158">例如，运行状况验证检查可能会检查应用程序使用的数据库、存储和外部地理编码服务，每个检查项需要不同级别的正常运行时间和响应时间。</span><span class="sxs-lookup"><span data-stu-id="a39ff-158">For example, a health verification check might check the database, storage, and an external geocoding service that an application uses, with each requiring a different level of uptime and response time.</span></span> <span data-ttu-id="a39ff-159">如果地理编码服务或其他某个后台任务有几分钟不可用，应用程序可能仍处于正常状态。</span><span class="sxs-lookup"><span data-stu-id="a39ff-159">The application could still be healthy if the geocoding service, or some other background task, is unavailable for a few minutes.</span></span>

<span data-ttu-id="a39ff-160">确定是否使用进行常规访问时所用的同一个终结点进行监视，不过，要使用专门用于运行状况验证检查的特定路径，例如，在常规访问终结点上使用 /HealthCheck/{GUID}/。</span><span class="sxs-lookup"><span data-stu-id="a39ff-160">Whether to use the same endpoint for monitoring as is used for general access, but to a specific path designed for health verification checks, for example, /HealthCheck/{GUID}/ on the general access endpoint.</span></span> <span data-ttu-id="a39ff-161">这样，监视工具便可运行应用程序中的某些功能测试，例如，添加新用户注册、登录和下达测试工单，同时还可验证常规访问终结点是否可用。</span><span class="sxs-lookup"><span data-stu-id="a39ff-161">This allows some functional tests in the application to be run by the monitoring tools, such as adding a new user registration, signing in, and placing a test order, while also verifying that the general access endpoint is available.</span></span>

<span data-ttu-id="a39ff-162">响应监视请求时要在服务中收集的信息类型，以及如何返回此信息。</span><span class="sxs-lookup"><span data-stu-id="a39ff-162">The type of information to collect in the service in response to monitoring requests, and how to return this information.</span></span> <span data-ttu-id="a39ff-163">大多数现有工具和框架仅查看终结点返回的 HTTP 状态代码。</span><span class="sxs-lookup"><span data-stu-id="a39ff-163">Most existing tools and frameworks look only at the HTTP status code that the endpoint returns.</span></span> <span data-ttu-id="a39ff-164">若要返回并验证其他信息，可能需要创建自定义监视实用工具或服务。</span><span class="sxs-lookup"><span data-stu-id="a39ff-164">To return and validate additional information, you might have to create a custom monitoring utility or service.</span></span>

<span data-ttu-id="a39ff-165">要收集多少信息。</span><span class="sxs-lookup"><span data-stu-id="a39ff-165">How much information to collect.</span></span> <span data-ttu-id="a39ff-166">在检查期间执行过多的处理可能导致应用程序过载并影响其他用户。</span><span class="sxs-lookup"><span data-stu-id="a39ff-166">Performing excessive processing during the check can overload the application and impact other users.</span></span> <span data-ttu-id="a39ff-167">所需的时间可能超过监视系统的超时时间，因此会将应用程序标记为不可用。</span><span class="sxs-lookup"><span data-stu-id="a39ff-167">The time it takes might exceed the timeout of the monitoring system so it marks the application as unavailable.</span></span> <span data-ttu-id="a39ff-168">大多数应用程序包含错误处理程序和性能计数器等检测功能用于记录性能与详细错误信息，这些信息可能已足够，而无需通过运行状况验证检查返回更多信息。</span><span class="sxs-lookup"><span data-stu-id="a39ff-168">Most applications include instrumentation such as error handlers and performance counters that log performance and detailed error information, this might be sufficient instead of returning additional information from a health verification check.</span></span>

<span data-ttu-id="a39ff-169">缓存终结点状态。</span><span class="sxs-lookup"><span data-stu-id="a39ff-169">Caching the endpoint status.</span></span> <span data-ttu-id="a39ff-170">过于频繁地执行运行状况检查可能会产生很高的系统开销。</span><span class="sxs-lookup"><span data-stu-id="a39ff-170">It could be expensive to run the health check too frequently.</span></span> <span data-ttu-id="a39ff-171">例如，如果通过仪表板报告运行状况，则不需要来自仪表板的每个请求都触发运行状况检查。</span><span class="sxs-lookup"><span data-stu-id="a39ff-171">If the health status is reported through a dashboard, for example, you don't want every request from the dashboard to trigger a health check.</span></span> <span data-ttu-id="a39ff-172">应该定期检查系统运行状况并缓存状态。</span><span class="sxs-lookup"><span data-stu-id="a39ff-172">Instead, periodically check the system health and cache the status.</span></span> <span data-ttu-id="a39ff-173">公开返回缓存状态的终结点。</span><span class="sxs-lookup"><span data-stu-id="a39ff-173">Expose an endpoint that returns the cached status.</span></span>

<span data-ttu-id="a39ff-174">如何配置监视终结点的安全性以保护其受到的公开访问，否则，应用程序可能会遭受恶意攻击，出现敏感信息透露或成为拒绝服务 (DoS) 攻击目标的风险。</span><span class="sxs-lookup"><span data-stu-id="a39ff-174">How to configure security for the monitoring endpoints to protect them from public access, which might expose the application to malicious attacks, risk the exposure of sensitive information, or attract denial of service (DoS) attacks.</span></span> <span data-ttu-id="a39ff-175">通常应在应用程序配置中执行此操作，以便无需重启应用程序就能更新配置。</span><span class="sxs-lookup"><span data-stu-id="a39ff-175">Typically this should be done in the application configuration so that it can be updated easily without restarting the application.</span></span> <span data-ttu-id="a39ff-176">考虑使用以下一种或多种技术：</span><span class="sxs-lookup"><span data-stu-id="a39ff-176">Consider using one or more of the following techniques:</span></span>

- <span data-ttu-id="a39ff-177">要求身份验证，以保护终结点。</span><span class="sxs-lookup"><span data-stu-id="a39ff-177">Secure the endpoint by requiring authentication.</span></span> <span data-ttu-id="a39ff-178">为此，可以在请求标头中使用身份验证安全密钥，或者连同请求一起传递凭据，前提是监视服务或工具支持身份验证。</span><span class="sxs-lookup"><span data-stu-id="a39ff-178">You can do this by using an authentication security key in the request header or by passing credentials with the request, provided that the monitoring service or tool supports authentication.</span></span>

  - <span data-ttu-id="a39ff-179">使用模糊化或隐藏的终结点。</span><span class="sxs-lookup"><span data-stu-id="a39ff-179">Use an obscure or hidden endpoint.</span></span> <span data-ttu-id="a39ff-180">例如，在不是由默认应用程序 URL 使用的 IP 地址上公开终结点，在非标准 HTTP 端口上配置终结点，和/或使用测试页的复杂路径。</span><span class="sxs-lookup"><span data-stu-id="a39ff-180">For example, expose the endpoint on a different IP address to that used by the default application URL, configure the endpoint on a nonstandard HTTP port, and/or use a complex path to the test page.</span></span> <span data-ttu-id="a39ff-181">通常可以在应用程序配置中指定其他终结点地址和端口，并可以根据需要将这些终结点的条目添加到 DNS 服务器，以免直接指定 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="a39ff-181">You can usually specify additional endpoint addresses and ports in the application configuration, and add entries for these endpoints to the DNS server if required to avoid having to specify the IP address directly.</span></span>

  - <span data-ttu-id="a39ff-182">在终结点上公开一个可接受键值或操作模式值等参数的方法。</span><span class="sxs-lookup"><span data-stu-id="a39ff-182">Expose a method on an endpoint that accepts a parameter such as a key value or an operation mode value.</span></span> <span data-ttu-id="a39ff-183">根据为此参数提供的值，在收到到请求时，代码可以执行一个特定的测试或一组测试；如果参数值未被识别，则代码会返回 404（未找到）错误。</span><span class="sxs-lookup"><span data-stu-id="a39ff-183">Depending on the value supplied for this parameter, when a request is received the code can perform a specific test or set of tests, or return a 404 (Not Found) error if the parameter value isn't recognized.</span></span> <span data-ttu-id="a39ff-184">可以在应用程序配置中设置识别的参数值。</span><span class="sxs-lookup"><span data-stu-id="a39ff-184">The recognized parameter values could be set in the application configuration.</span></span>

     >  <span data-ttu-id="a39ff-185">DoS 攻击可能只对执行基本功能测试的独立终结点产生较小的影响，而不会影响应用程序的运行。</span><span class="sxs-lookup"><span data-stu-id="a39ff-185">DoS attacks are likely to have less impact on a separate endpoint that performs basic functional tests without compromising the operation of the application.</span></span> <span data-ttu-id="a39ff-186">理想情况下，请避免使用可能透露敏感信息的测试。</span><span class="sxs-lookup"><span data-stu-id="a39ff-186">Ideally, avoid using a test that might expose sensitive information.</span></span> <span data-ttu-id="a39ff-187">如果必须返回可能对攻击者有用的信息，请考虑如何防范终结点和数据受到未经授权的访问。</span><span class="sxs-lookup"><span data-stu-id="a39ff-187">If you must return information that might be useful to an attacker, consider how you'll protect the endpoint and the data from unauthorized access.</span></span> <span data-ttu-id="a39ff-188">在这种情况下，只是依靠模糊处理并不足够。</span><span class="sxs-lookup"><span data-stu-id="a39ff-188">In this case just relying on obscurity isn't enough.</span></span> <span data-ttu-id="a39ff-189">还应考虑使用 HTTPS 连接并加密所有敏感数据，尽管这会增大服务器的负载。</span><span class="sxs-lookup"><span data-stu-id="a39ff-189">You should also consider using an HTTPS connection and encrypting any sensitive data, although this will increase the load on the server.</span></span>

- <span data-ttu-id="a39ff-190">如何访问使用身份验证保护的终结点。</span><span class="sxs-lookup"><span data-stu-id="a39ff-190">How to access an endpoint that's secured using authentication.</span></span> <span data-ttu-id="a39ff-191">并非所有工具和框架都可以配置为在运行状况验证请求中包含凭据。</span><span class="sxs-lookup"><span data-stu-id="a39ff-191">Not all tools and frameworks can be configured to include credentials with the health verification request.</span></span> <span data-ttu-id="a39ff-192">例如，Microsoft Azure 内置的运行状况验证功能无法提供身份验证凭据。</span><span class="sxs-lookup"><span data-stu-id="a39ff-192">For example, Microsoft Azure built-in health verification features can't provide authentication credentials.</span></span> <span data-ttu-id="a39ff-193">某些第三方替代方案包括 [Pingdom](https://www.pingdom.com/)、[Panopta](http://www.panopta.com/)、[NewRelic](https://newrelic.com/) 和 [Statuscake](https://www.statuscake.com/)。</span><span class="sxs-lookup"><span data-stu-id="a39ff-193">Some third-party alternatives are [Pingdom](https://www.pingdom.com/), [Panopta](http://www.panopta.com/), [NewRelic](https://newrelic.com/), and [Statuscake](https://www.statuscake.com/).</span></span>

- <span data-ttu-id="a39ff-194">如何确保监视代理正常运行。</span><span class="sxs-lookup"><span data-stu-id="a39ff-194">How to ensure that the monitoring agent is performing correctly.</span></span> <span data-ttu-id="a39ff-195">一种方法是公开一个只返回应用程序配置中的值，或者返回可用于测试代理的随机值的终结点。</span><span class="sxs-lookup"><span data-stu-id="a39ff-195">One approach is to expose an endpoint that simply returns a value from the application configuration or a random value that can be used to test the agent.</span></span>

   >  <span data-ttu-id="a39ff-196">另外，请确保监视系统对自身进行检查，例如自测试和内置测试，以免发出误报结果。</span><span class="sxs-lookup"><span data-stu-id="a39ff-196">Also ensure that the monitoring system performs checks on itself, such as a self-test and built-in test, to avoid it issuing false positive results.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="a39ff-197">何时使用此模式</span><span class="sxs-lookup"><span data-stu-id="a39ff-197">When to use this pattern</span></span>

<span data-ttu-id="a39ff-198">此模式适合用于：</span><span class="sxs-lookup"><span data-stu-id="a39ff-198">This pattern is useful for:</span></span>
- <span data-ttu-id="a39ff-199">监视网站和 Web 应用程序，以验证可用性。</span><span class="sxs-lookup"><span data-stu-id="a39ff-199">Monitoring websites and web applications to verify availability.</span></span>
- <span data-ttu-id="a39ff-200">监视网站和 Web 应用程序以检查是否正常运行。</span><span class="sxs-lookup"><span data-stu-id="a39ff-200">Monitoring websites and web applications to check for correct operation.</span></span>
- <span data-ttu-id="a39ff-201">监视中间层或共享服务，以检测并隔离可能会中断其他应用程序的故障。</span><span class="sxs-lookup"><span data-stu-id="a39ff-201">Monitoring middle-tier or shared services to detect and isolate a failure that could disrupt other applications.</span></span>
- <span data-ttu-id="a39ff-202">补充应用程序中的现有检测，例如性能计数器和错误处理程序。</span><span class="sxs-lookup"><span data-stu-id="a39ff-202">Complementing existing instrumentation in the application, such as performance counters and error handlers.</span></span> <span data-ttu-id="a39ff-203">运行状况验证检查不能取代应用程序中的日志记录和审核要求。</span><span class="sxs-lookup"><span data-stu-id="a39ff-203">Health verification checking doesn't replace the requirement for logging and auditing in the application.</span></span> <span data-ttu-id="a39ff-204">检测可为现有框架提供有用的信息，使它们能够监视计数器和错误日志，以检测故障或其他问题。</span><span class="sxs-lookup"><span data-stu-id="a39ff-204">Instrumentation can provide valuable information for an existing framework that monitors counters and error logs to detect failures or other issues.</span></span> <span data-ttu-id="a39ff-205">但是，如果应用程序不可用，则它无法提供信息。</span><span class="sxs-lookup"><span data-stu-id="a39ff-205">However, it can't provide information if the application is unavailable.</span></span>

## <a name="example"></a><span data-ttu-id="a39ff-206">示例</span><span class="sxs-lookup"><span data-stu-id="a39ff-206">Example</span></span>

<span data-ttu-id="a39ff-207">以下代码示例摘自 `HealthCheckController` 类（[GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/health-endpoint-monitoring) 上提供的一个用于演示此模式的示例），演示如何公开一个可执行一系列运行状况检查的终结点。</span><span class="sxs-lookup"><span data-stu-id="a39ff-207">The following code examples, taken from the `HealthCheckController` class (a sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/health-endpoint-monitoring)), demonstrates exposing an endpoint for performing a range of health checks.</span></span>

<span data-ttu-id="a39ff-208">如以下 C# 代码所示，`CoreServices` 方法对应用程序中使用的服务执行一系列检查。</span><span class="sxs-lookup"><span data-stu-id="a39ff-208">The `CoreServices` method, shown below in C#, performs a series of checks on services used in the application.</span></span> <span data-ttu-id="a39ff-209">如果所有测试已完成运行且未出错，该方法会返回 200（正常）状态代码。</span><span class="sxs-lookup"><span data-stu-id="a39ff-209">If all of the tests run without error, the method returns a 200 (OK) status code.</span></span> <span data-ttu-id="a39ff-210">如果任何测试引发异常，该方法会返回 500（内部错误）状态代码。</span><span class="sxs-lookup"><span data-stu-id="a39ff-210">If any of the tests raises an exception, the method returns a 500 (Internal Error) status code.</span></span> <span data-ttu-id="a39ff-211">如果监视工具或框架能够利用其他信息，则发生错误时，该方法可能会选择性地返回这些信息。</span><span class="sxs-lookup"><span data-stu-id="a39ff-211">The method could optionally return additional information when an error occurs, if the monitoring tool or framework is able to make use of it.</span></span>

```csharp
public ActionResult CoreServices()
{
  try
  {
    // Run a simple check to ensure the database is available.
    DataStore.Instance.CoreHealthCheck();

    // Run a simple check on our external service.
    MyExternalService.Instance.CoreHealthCheck();
  }
  catch (Exception ex)
  {
    Trace.TraceError("Exception in basic health check: {0}", ex.Message);

    // This can optionally return different status codes based on the exception.
    // Optionally it could return more details about the exception.
    // The additional information could be used by administrators who access the
    // endpoint with a browser, or using a ping utility that can display the
    // additional information.
    return new HttpStatusCodeResult((int)HttpStatusCode.InternalServerError);
  }
  return new HttpStatusCodeResult((int)HttpStatusCode.OK);
}
```
<span data-ttu-id="a39ff-212">`ObscurePath` 方法演示如何从应用程序配置中读取某个路径，并将其用作测试终结点。</span><span class="sxs-lookup"><span data-stu-id="a39ff-212">The `ObscurePath` method shows how you can read a path from the application configuration and use it as the endpoint for tests.</span></span> <span data-ttu-id="a39ff-213">此 C# 示例还演示如何接受某个 ID 作为参数，并使用它来检查有效请求。</span><span class="sxs-lookup"><span data-stu-id="a39ff-213">This example, in C#, also shows how you can accept an ID as a parameter and use it to check for valid requests.</span></span>

```csharp
public ActionResult ObscurePath(string id)
{
  // The id could be used as a simple way to obscure or hide the endpoint.
  // The id to match could be retrieved from configuration and, if matched,
  // perform a specific set of tests and return the result. If not matched it
  // could return a 404 (Not Found) status.

  // The obscure path can be set through configuration to hide the endpoint.
  var hiddenPathKey = CloudConfigurationManager.GetSetting("Test.ObscurePath");

  // If the value passed does not match that in configuration, return 404 (Not Found).
  if (!string.Equals(id, hiddenPathKey))
  {
    return new HttpStatusCodeResult((int)HttpStatusCode.NotFound);
  }

  // Else continue and run the tests...
  // Return results from the core services test.
  return this.CoreServices();
}
```

<span data-ttu-id="a39ff-214">`TestResponseFromConfig` 方法演示如何公开一个可以针对指定配置设置值执行检查的终结点。</span><span class="sxs-lookup"><span data-stu-id="a39ff-214">The `TestResponseFromConfig` method shows how you can expose an endpoint that performs a check for a specified configuration setting value.</span></span>

```csharp
public ActionResult TestResponseFromConfig()
{
  // Health check that returns a response code set in configuration for testing.
  var returnStatusCodeSetting = CloudConfigurationManager.GetSetting(
                                                          "Test.ReturnStatusCode");

  int returnStatusCode;

  if (!int.TryParse(returnStatusCodeSetting, out returnStatusCode))
  {
    returnStatusCode = (int)HttpStatusCode.OK;
  }

  return new HttpStatusCodeResult(returnStatusCode);
}
```
## <a name="monitoring-endpoints-in-azure-hosted-applications"></a><span data-ttu-id="a39ff-215">监视 Azure 托管应用程序中的终结点</span><span class="sxs-lookup"><span data-stu-id="a39ff-215">Monitoring endpoints in Azure hosted applications</span></span>

<span data-ttu-id="a39ff-216">用于监视 Azure 应用程序中的终结点的一些选项包括：</span><span class="sxs-lookup"><span data-stu-id="a39ff-216">Some options for monitoring endpoints in Azure applications are:</span></span>

- <span data-ttu-id="a39ff-217">使用 Azure 的内置监视功能。</span><span class="sxs-lookup"><span data-stu-id="a39ff-217">Use the built-in monitoring features of Azure.</span></span>

- <span data-ttu-id="a39ff-218">使用第三方服务或框架，例 Microsoft System Center Operations Manager。</span><span class="sxs-lookup"><span data-stu-id="a39ff-218">Use a third-party service or a framework such as Microsoft System Center Operations Manager.</span></span>

- <span data-ttu-id="a39ff-219">创建自定义实用工具，或者创建可在自己的服务器或托管服务器上运行的服务。</span><span class="sxs-lookup"><span data-stu-id="a39ff-219">Create a custom utility or a service that runs on your own or on a hosted server.</span></span>

   >  <span data-ttu-id="a39ff-220">尽管 Azure 提供了相当全面的监视选项，但你也可以使用其他服务和工具来提供附加的信息。</span><span class="sxs-lookup"><span data-stu-id="a39ff-220">Even though Azure provides a reasonably comprehensive set of monitoring options, you can use additional services and tools to provide extra information.</span></span> <span data-ttu-id="a39ff-221">Azure 管理服务为警报规则提供内置监视机制。</span><span class="sxs-lookup"><span data-stu-id="a39ff-221">Azure Management Services provides a built-in monitoring mechanism for alert rules.</span></span> <span data-ttu-id="a39ff-222">在 Azure 门户中“管理服务”页的“警报”部分，可为服务的每个订阅最多配置 10 条警报规则。</span><span class="sxs-lookup"><span data-stu-id="a39ff-222">The alerts section of the management services page in the Azure portal allows you to configure up to ten alert rules per subscription for your services.</span></span> <span data-ttu-id="a39ff-223">这些规则指定服务的条件和阈值（例如 CPU 负载），或每秒的请求数或错误数，服务可以自动向每条规则中定义的地址发送电子邮件通知。</span><span class="sxs-lookup"><span data-stu-id="a39ff-223">These rules specify a condition and a threshold value for a service such as CPU load, or the number of requests or errors per second, and the service can automatically send email notifications to addresses you define in each rule.</span></span>

<span data-ttu-id="a39ff-224">可监视的条件根据你为应用程序选择的托管机制（例如网站、云服务、虚拟机或移动服务）而异，但所有这些机制都能创建警报规则，该规则使用你在服务设置中指定的 Web 终结点。</span><span class="sxs-lookup"><span data-stu-id="a39ff-224">The conditions you can monitor vary depending on the hosting mechanism you choose for your application (such as Web Sites, Cloud Services, Virtual Machines, or Mobile Services), but all of these include the ability to create an alert rule that uses a web endpoint you specify in the settings for your service.</span></span> <span data-ttu-id="a39ff-225">该终结点应及时做出响应，使警报系统可以检测到应用程序在正常运行。</span><span class="sxs-lookup"><span data-stu-id="a39ff-225">This endpoint should respond in a timely way so that the alert system can detect that the application is operating correctly.</span></span>

>  <span data-ttu-id="a39ff-226">阅读有关[创建警报通知][portal-alerts]的详细信息。</span><span class="sxs-lookup"><span data-stu-id="a39ff-226">Read more information about [creating alert notifications][portal-alerts].</span></span>

<span data-ttu-id="a39ff-227">如果在 Azure 云服务 Web 角色和辅助角色或虚拟机中托管应用程序，则可以利用 Azure 中的一个内置服务，即流量管理器。</span><span class="sxs-lookup"><span data-stu-id="a39ff-227">If you host your application in Azure Cloud Services web and worker roles or Virtual Machines, you can take advantage of one of the built-in services in Azure called Traffic Manager.</span></span> <span data-ttu-id="a39ff-228">流量管理器是一个路由和负载均衡服务，可以根据一系列规则和设置，将请求分发到云服务托管的应用程序的特定实例。</span><span class="sxs-lookup"><span data-stu-id="a39ff-228">Traffic Manager is a routing and load-balancing service that can distribute requests to specific instances of your Cloud Services hosted application based on a range of rules and settings.</span></span>

<span data-ttu-id="a39ff-229">除了路由请求以外，流量管理器会定期针对指定的 URL、端口和相对路径执行 ping，以确定应用程序规则中定义的哪些应用程序实例处于活动状态并在响应请求。</span><span class="sxs-lookup"><span data-stu-id="a39ff-229">In addition to routing requests, Traffic Manager pings a URL, port, and relative path that you specify on a regular basis to determine which instances of the application defined in its rules are active and are responding to requests.</span></span> <span data-ttu-id="a39ff-230">如果流量管理器检测到状态代码 200（正常），则会将应用程序标记为可用。</span><span class="sxs-lookup"><span data-stu-id="a39ff-230">If it detects a status code 200 (OK), it marks the application as available.</span></span> <span data-ttu-id="a39ff-231">其他任何状态代码会导致流量管理器将应用程序标记为脱机。</span><span class="sxs-lookup"><span data-stu-id="a39ff-231">Any other status code causes Traffic Manager to mark the application as offline.</span></span> <span data-ttu-id="a39ff-232">可以在流量管理器控制台中查看状态，并配置规则用于将请求重新路由到正在做出响应的其他应用程序实例。</span><span class="sxs-lookup"><span data-stu-id="a39ff-232">You can view the status in the Traffic Manager console, and configure the rule to reroute requests to other instances of the application that are responding.</span></span>

<span data-ttu-id="a39ff-233">但是，流量管理器在接收监视 URL 的响应期间只会等待 10 秒。</span><span class="sxs-lookup"><span data-stu-id="a39ff-233">However, Traffic Manager will only wait ten seconds to receive a response from the monitoring URL.</span></span> <span data-ttu-id="a39ff-234">因此，应确保运行状况验证代码在此时限内执行，并考虑到在流量管理器与应用程序之间往返所存在的网络延迟。</span><span class="sxs-lookup"><span data-stu-id="a39ff-234">Therefore, you should ensure that your health verification code executes in this time, allowing for network latency for the round trip from Traffic Manager to your application and back again.</span></span>

>  <span data-ttu-id="a39ff-235">阅读有关使用[流量管理器监视应用程序](https://azure.microsoft.com/documentation/services/traffic-manager/)的详细信息。</span><span class="sxs-lookup"><span data-stu-id="a39ff-235">Read more information about using [Traffic Manager to monitor your applications](https://azure.microsoft.com/documentation/services/traffic-manager/).</span></span> <span data-ttu-id="a39ff-236">[多个数据中心的部署指南](https://msdn.microsoft.com/library/dn589779.aspx)中也介绍了流量管理器。</span><span class="sxs-lookup"><span data-stu-id="a39ff-236">Traffic Manager is also discussed in [Multiple Datacenter Deployment Guidance](https://msdn.microsoft.com/library/dn589779.aspx).</span></span>

## <a name="related-guidance"></a><span data-ttu-id="a39ff-237">相关指南</span><span class="sxs-lookup"><span data-stu-id="a39ff-237">Related guidance</span></span>

<span data-ttu-id="a39ff-238">实施此模式时，可参考以下指南：</span><span class="sxs-lookup"><span data-stu-id="a39ff-238">The following guidance can be useful when implementing this pattern:</span></span>
- <span data-ttu-id="a39ff-239">[检测和遥测指南](https://msdn.microsoft.com/library/dn589775.aspx)。</span><span class="sxs-lookup"><span data-stu-id="a39ff-239">[Instrumentation and Telemetry Guidance](https://msdn.microsoft.com/library/dn589775.aspx).</span></span> <span data-ttu-id="a39ff-240">服务和组件的运行状况检查通常是通过探测来完成的，但是，另一种有用的做法是准备好相关的信息，用于监视应用程序的性能，并检测运行时发生的事件。</span><span class="sxs-lookup"><span data-stu-id="a39ff-240">Checking the health of services and components is typically done by probing, but it's also useful to have information in place to monitor application performance and detect events that occur at runtime.</span></span> <span data-ttu-id="a39ff-241">可将此数据作为附加信息传回到监视工具用于运行状况监视。</span><span class="sxs-lookup"><span data-stu-id="a39ff-241">This data can be transmitted back to monitoring tools as additional information for health monitoring.</span></span> <span data-ttu-id="a39ff-242">《检测和遥测指南》探讨了如何收集检测功能在应用程序中收集的远程诊断信息。</span><span class="sxs-lookup"><span data-stu-id="a39ff-242">Instrumentation and Telemetry Guidance explores gathering remote diagnostics information that's collected by instrumentation in applications.</span></span>
- <span data-ttu-id="a39ff-243">[接收警报通知][portal-alerts]。</span><span class="sxs-lookup"><span data-stu-id="a39ff-243">[Receiving alert notifications][portal-alerts].</span></span>
- <span data-ttu-id="a39ff-244">此模式包含一个可下载的[示例应用程序](https://github.com/mspnp/cloud-design-patterns/tree/master/health-endpoint-monitoring)。</span><span class="sxs-lookup"><span data-stu-id="a39ff-244">This pattern includes a downloadable [sample application](https://github.com/mspnp/cloud-design-patterns/tree/master/health-endpoint-monitoring).</span></span>

[portal-alerts]: https://azure.microsoft.com/documentation/articles/insights-receive-alert-notifications/
