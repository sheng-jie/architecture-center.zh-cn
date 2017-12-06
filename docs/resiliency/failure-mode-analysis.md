---
title: "故障模式分析"
description: "基于 Azure 为云解决方案执行故障模式分析的相关准则。"
author: MikeWasson
ms.date: 03/24/2017
ms.custom: resiliency
pnp.series.title: Design for Resiliency
ms.openlocfilehash: 09d09468eebe5c6fe1c9cdab14e142ff46cf0b25
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="failure-mode-analysis"></a><span data-ttu-id="1ae68-103">故障模式分析</span><span class="sxs-lookup"><span data-stu-id="1ae68-103">Failure mode analysis</span></span>
[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="1ae68-104">故障模式分析 (FMA) 是一个过程，用于通过标识系统中可能的故障点将复原能力内置到系统中。</span><span class="sxs-lookup"><span data-stu-id="1ae68-104">Failure mode analysis (FMA) is a process for building resiliency into a system, by identifying possible failure points in the system.</span></span> <span data-ttu-id="1ae68-105">FMA 应该是体系结构和设计阶段的一部分，以便从一开始就将故障恢复能力内置到系统中。</span><span class="sxs-lookup"><span data-stu-id="1ae68-105">The FMA should be part of the architecture and design phases, so that you can build failure recovery into the system from the beginning.</span></span>

<span data-ttu-id="1ae68-106">下面是执行 FMA 的常规过程：</span><span class="sxs-lookup"><span data-stu-id="1ae68-106">Here is the general process to conduct an FMA:</span></span>

1. <span data-ttu-id="1ae68-107">标识系统中的所有组件。</span><span class="sxs-lookup"><span data-stu-id="1ae68-107">Identify all of the components in the system.</span></span> <span data-ttu-id="1ae68-108">包括外部依赖项，比如标识提供者、第三方服务等等。</span><span class="sxs-lookup"><span data-stu-id="1ae68-108">Include external dependencies, such as as identity providers, third-party services, and so on.</span></span>   
2. <span data-ttu-id="1ae68-109">对于每个组件，标识可能发生的潜在故障。</span><span class="sxs-lookup"><span data-stu-id="1ae68-109">For each component, identify potential failures that could occur.</span></span> <span data-ttu-id="1ae68-110">单个组件可能具有多种故障模式。</span><span class="sxs-lookup"><span data-stu-id="1ae68-110">A single component may have more than one failure mode.</span></span> <span data-ttu-id="1ae68-111">例如，应分开考虑读取故障和写入故障，因为其影响和可行的缓解措施有所不同。</span><span class="sxs-lookup"><span data-stu-id="1ae68-111">For example, you should consider read failures and write failures separately, because the impact and possible mitigations will be different.</span></span>
3. <span data-ttu-id="1ae68-112">根据总体风险对每种故障模式评分。</span><span class="sxs-lookup"><span data-stu-id="1ae68-112">Rate each failure mode according to its overall risk.</span></span> <span data-ttu-id="1ae68-113">请考虑以下因素：</span><span class="sxs-lookup"><span data-stu-id="1ae68-113">Consider these factors:</span></span>  

   * <span data-ttu-id="1ae68-114">发生故障的几率有多大。</span><span class="sxs-lookup"><span data-stu-id="1ae68-114">What is the likelihood of the failure.</span></span> <span data-ttu-id="1ae68-115">是相对常见？</span><span class="sxs-lookup"><span data-stu-id="1ae68-115">Is it relatively common?</span></span> <span data-ttu-id="1ae68-116">还是极其少见？</span><span class="sxs-lookup"><span data-stu-id="1ae68-116">Extrememly rare?</span></span> <span data-ttu-id="1ae68-117">无需准确的数字；其目的是帮助进行优先级排名。</span><span class="sxs-lookup"><span data-stu-id="1ae68-117">You don't need exact numbers; the purpose is to help rank the priority.</span></span>
   * <span data-ttu-id="1ae68-118">对应用程序的可用性、数据丢失、货币成本和业务中断有什么影响？</span><span class="sxs-lookup"><span data-stu-id="1ae68-118">What is the impact on the application, in terms of availability, data loss, monetary cost, and business disruption?</span></span>
4. <span data-ttu-id="1ae68-119">对于每种故障模式，确定应用程序将如何响应和恢复。</span><span class="sxs-lookup"><span data-stu-id="1ae68-119">For each failure mode, determine how the application will respond and recover.</span></span> <span data-ttu-id="1ae68-120">考虑在成本与应用程序复杂程度之间作出折衷。</span><span class="sxs-lookup"><span data-stu-id="1ae68-120">Consider tradeoffs in cost and application complexity.</span></span>   

<span data-ttu-id="1ae68-121">作为 FMA 过程的起点，本文包含了潜在故障模式及其缓解措施的目录。</span><span class="sxs-lookup"><span data-stu-id="1ae68-121">As a starting point for your FMA process, this article contains a catalog of potential failure modes and their mitigations.</span></span> <span data-ttu-id="1ae68-122">该目录按技术或 Azure 服务分类，另外还有一个针对应用程序级设计的常规目录。</span><span class="sxs-lookup"><span data-stu-id="1ae68-122">The catalog is organized by technology or Azure service, plus a general category for application-level design.</span></span> <span data-ttu-id="1ae68-123">该目录并不详尽，但涵盖许多核心 Azure 服务。</span><span class="sxs-lookup"><span data-stu-id="1ae68-123">The catalog is not exhaustive, but covers many of the core Azure services.</span></span>

## <a name="app-service"></a><span data-ttu-id="1ae68-124">应用服务</span><span class="sxs-lookup"><span data-stu-id="1ae68-124">App Service</span></span>
### <a name="app-service-app-shuts-down"></a><span data-ttu-id="1ae68-125">应用服务应用关闭。</span><span class="sxs-lookup"><span data-stu-id="1ae68-125">App Service app shuts down.</span></span>
<span data-ttu-id="1ae68-126">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-126">**Detection**.</span></span> <span data-ttu-id="1ae68-127">可能的原因：</span><span class="sxs-lookup"><span data-stu-id="1ae68-127">Possible causes:</span></span>

* <span data-ttu-id="1ae68-128">意料中的关闭</span><span class="sxs-lookup"><span data-stu-id="1ae68-128">Expected shutdown</span></span>

  * <span data-ttu-id="1ae68-129">操作员关闭该应用程序；例如，使用 Azure 门户。</span><span class="sxs-lookup"><span data-stu-id="1ae68-129">An operator shuts down the application; for example, using the Azure portal.</span></span>
  * <span data-ttu-id="1ae68-130">应用已被卸载，因为它处于闲置状态。</span><span class="sxs-lookup"><span data-stu-id="1ae68-130">The app was unloaded because it was idle.</span></span> <span data-ttu-id="1ae68-131">（仅当禁用 `Always On` 设置时。）</span><span class="sxs-lookup"><span data-stu-id="1ae68-131">(Only if the `Always On` setting is disabled.)</span></span>
* <span data-ttu-id="1ae68-132">意料外的关闭</span><span class="sxs-lookup"><span data-stu-id="1ae68-132">Unexpected shutdown</span></span>

  * <span data-ttu-id="1ae68-133">应用崩溃。</span><span class="sxs-lookup"><span data-stu-id="1ae68-133">The app crashes.</span></span>
  * <span data-ttu-id="1ae68-134">应用服务 VM 实例变为不可用。</span><span class="sxs-lookup"><span data-stu-id="1ae68-134">An App Service VM instance becomes unavailable.</span></span>

<span data-ttu-id="1ae68-135">Application_End 日志记录将捕获应用域关闭（软进程崩溃），它是唯一捕获应用程序域关闭的方法。</span><span class="sxs-lookup"><span data-stu-id="1ae68-135">Application_End logging will catch the app domain shutdown (soft process crash) and is the only way to catch the application domain shutdowns.</span></span>

<span data-ttu-id="1ae68-136">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-136">**Recovery**</span></span>

* <span data-ttu-id="1ae68-137">如果是意料中的关闭，请使用应用程序的关闭事件正常关闭。</span><span class="sxs-lookup"><span data-stu-id="1ae68-137">If the shutdown was expected, use the application's shutdown event to shut down gracefully.</span></span> <span data-ttu-id="1ae68-138">例如，在 ASP.NET 中，使用 `Application_End` 方法。</span><span class="sxs-lookup"><span data-stu-id="1ae68-138">For example, in ASP.NET, use the `Application_End` method.</span></span>
* <span data-ttu-id="1ae68-139">如果应用程序在闲置时被卸载，它将在下一次请求时自动重启。</span><span class="sxs-lookup"><span data-stu-id="1ae68-139">If the application was unloaded while idle, it is automatically restarted on the next request.</span></span> <span data-ttu-id="1ae68-140">但是，这会产生“冷启动”成本。</span><span class="sxs-lookup"><span data-stu-id="1ae68-140">However, you will incur the "cold start" cost.</span></span>
* <span data-ttu-id="1ae68-141">若要防止应用程序在闲置时被卸载，请启用 Web 应用中的 `Always On` 设置。</span><span class="sxs-lookup"><span data-stu-id="1ae68-141">To prevent the application from being unloaded while idle, enable the `Always On` setting in the web app.</span></span> <span data-ttu-id="1ae68-142">请参阅[在 Azure 应用服务中配置 Web 应用][app-service-configure]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-142">See [Configure web apps in Azure App Service][app-service-configure].</span></span>
* <span data-ttu-id="1ae68-143">若要防止操作员关闭应用，请设置 `ReadOnly` 级别的资源锁。</span><span class="sxs-lookup"><span data-stu-id="1ae68-143">To prevent an operator from shutting down the app, set a resource lock with `ReadOnly` level.</span></span> <span data-ttu-id="1ae68-144">请参阅[使用 Azure 资源管理器锁定资源][rm-locks]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-144">See [Lock resources with Azure Resource Manager][rm-locks].</span></span>
* <span data-ttu-id="1ae68-145">如果应用崩溃或应用服务 VM 变得不可用，应用服务会自动重启应用。</span><span class="sxs-lookup"><span data-stu-id="1ae68-145">If the app crashes or an App Service VM becomes unavailable, App Service automatically restarts the app.</span></span>

<span data-ttu-id="1ae68-146">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-146">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-147">应用程序日志和 Web 服务器日志。</span><span class="sxs-lookup"><span data-stu-id="1ae68-147">Application logs and web server logs.</span></span> <span data-ttu-id="1ae68-148">请参阅[在 Azure 应用服务中为 Web 应用启用诊断日志记录][app-service-logging]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-148">See [Enable diagnostics logging for web apps in Azure App Service][app-service-logging].</span></span>

### <a name="a-particular-user-repeatedly-makes-bad-requests-or-overloads-the-system"></a><span data-ttu-id="1ae68-149">特定用户反复发出错误请求或重载系统。</span><span class="sxs-lookup"><span data-stu-id="1ae68-149">A particular user repeatedly makes bad requests or overloads the system.</span></span>
<span data-ttu-id="1ae68-150">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-150">**Detection**.</span></span> <span data-ttu-id="1ae68-151">验证用户身份并将用户 ID 添加到应用程序日志中。</span><span class="sxs-lookup"><span data-stu-id="1ae68-151">Authenticate users and include user ID in application logs.</span></span>

<span data-ttu-id="1ae68-152">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-152">**Recovery**</span></span>

* <span data-ttu-id="1ae68-153">使用 [Azure API 管理][api-management]限制该用户的请求。</span><span class="sxs-lookup"><span data-stu-id="1ae68-153">Use [Azure API Management][api-management] to throttle requests from the user.</span></span> <span data-ttu-id="1ae68-154">请参阅[使用 Azure API 管理进行高级请求限制][api-management-throttling]</span><span class="sxs-lookup"><span data-stu-id="1ae68-154">See [Advanced request throttling with Azure API Management][api-management-throttling]</span></span>
* <span data-ttu-id="1ae68-155">阻止该用户。</span><span class="sxs-lookup"><span data-stu-id="1ae68-155">Block the user.</span></span>

<span data-ttu-id="1ae68-156">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-156">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-157">记录所有身份验证请求。</span><span class="sxs-lookup"><span data-stu-id="1ae68-157">Log all authentication requests.</span></span>

### <a name="a-bad-update-was-deployed"></a><span data-ttu-id="1ae68-158">部署了错误的更新。</span><span class="sxs-lookup"><span data-stu-id="1ae68-158">A bad update was deployed.</span></span>
<span data-ttu-id="1ae68-159">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-159">**Detection**.</span></span> <span data-ttu-id="1ae68-160">通过 Azure 门户监视应用程序运行状况（请参阅[监视 Azure Web 应用性能][app-insights-web-apps]）或实施[运行状况终结点监视模式][health-endpoint-monitoring-pattern]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-160">Monitor the application health through the Azure Portal (see [Monitor Azure web app performance][app-insights-web-apps]) or implement the [health endpoint monitoring pattern][health-endpoint-monitoring-pattern].</span></span>

<span data-ttu-id="1ae68-161">**恢复**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-161">**Recovery**.</span></span> <span data-ttu-id="1ae68-162">使用多个[部署槽][app-service-slots]并回滚到上次已知正常的部署。</span><span class="sxs-lookup"><span data-stu-id="1ae68-162">Use multiple [deployment slots][app-service-slots] and roll back to the last-known-good deployment.</span></span> <span data-ttu-id="1ae68-163">有关详细信息，请参阅[基本 Web 应用程序][ra-web-apps-basic]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-163">For more information, see [Basic web application][ra-web-apps-basic].</span></span>

## <a name="azure-active-directory"></a><span data-ttu-id="1ae68-164">Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="1ae68-164">Azure Active Directory</span></span>
### <a name="openid-connect-oidc-authentication-fails"></a><span data-ttu-id="1ae68-165">OpenID Connect (OIDC) 身份验证失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-165">OpenID Connect (OIDC) authentication fails.</span></span>
<span data-ttu-id="1ae68-166">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-166">**Detection**.</span></span> <span data-ttu-id="1ae68-167">可能的故障模式包括：</span><span class="sxs-lookup"><span data-stu-id="1ae68-167">Possible failure modes include:</span></span>

1. <span data-ttu-id="1ae68-168">Azure AD 不可用，或由于网络问题而不可访问。</span><span class="sxs-lookup"><span data-stu-id="1ae68-168">Azure AD is not available, or cannot be reached due to a network problem.</span></span> <span data-ttu-id="1ae68-169">重定向到身份验证终结点失败，OIDC 中间件引发异常。</span><span class="sxs-lookup"><span data-stu-id="1ae68-169">Redirection to the authentication endpoint fails, and the OIDC middleware throws an exception.</span></span>
2. <span data-ttu-id="1ae68-170">Azure AD 租户不存在。</span><span class="sxs-lookup"><span data-stu-id="1ae68-170">Azure AD tenant does not exist.</span></span> <span data-ttu-id="1ae68-171">重定向到身份验证终结点返回 HTTP 错误代码，OIDC 中间件引发异常。</span><span class="sxs-lookup"><span data-stu-id="1ae68-171">Redirection to the authentication endpoint returns an HTTP error code, and the OIDC middleware throws an exception.</span></span>
3. <span data-ttu-id="1ae68-172">用户无法进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="1ae68-172">User cannot authenticate.</span></span> <span data-ttu-id="1ae68-173">无需实施任何检测策略；Azure AD 会处理登录失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-173">No detection strategy is necessary; Azure AD handles login failures.</span></span>

<span data-ttu-id="1ae68-174">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-174">**Recovery**</span></span>

1. <span data-ttu-id="1ae68-175">捕获中间件引发的未经处理的异常。</span><span class="sxs-lookup"><span data-stu-id="1ae68-175">Catch unhandled exceptions from the middleware.</span></span>
2. <span data-ttu-id="1ae68-176">处理 `AuthenticationFailed` 事件。</span><span class="sxs-lookup"><span data-stu-id="1ae68-176">Handle `AuthenticationFailed` events.</span></span>
3. <span data-ttu-id="1ae68-177">将用户重定向到错误页。</span><span class="sxs-lookup"><span data-stu-id="1ae68-177">Redirect the user to an error page.</span></span>
4. <span data-ttu-id="1ae68-178">用户重试。</span><span class="sxs-lookup"><span data-stu-id="1ae68-178">User retries.</span></span>

## <a name="azure-search"></a><span data-ttu-id="1ae68-179">Azure 搜索</span><span class="sxs-lookup"><span data-stu-id="1ae68-179">Azure Search</span></span>
### <a name="writing-data-to-azure-search-fails"></a><span data-ttu-id="1ae68-180">将数据写入 Azure 搜索失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-180">Writing data to Azure Search fails.</span></span>
<span data-ttu-id="1ae68-181">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-181">**Detection**.</span></span> <span data-ttu-id="1ae68-182">捕获 `Microsoft.Rest.Azure.CloudException` 错误。</span><span class="sxs-lookup"><span data-stu-id="1ae68-182">Catch `Microsoft.Rest.Azure.CloudException` errors.</span></span>

<span data-ttu-id="1ae68-183">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-183">**Recovery**</span></span>

<span data-ttu-id="1ae68-184">[搜索 .NET SDK][search-sdk] 在发生暂时性故障后会自动重试。</span><span class="sxs-lookup"><span data-stu-id="1ae68-184">The [Search .NET SDK][search-sdk] automatically retries after transient failures.</span></span> <span data-ttu-id="1ae68-185">由客户端 SDK 引发的任何异常均应视为非暂时性错误。</span><span class="sxs-lookup"><span data-stu-id="1ae68-185">Any exceptions thrown by the client SDK should be treated as non-transient errors.</span></span>

<span data-ttu-id="1ae68-186">默认重试策略使用指数回退。</span><span class="sxs-lookup"><span data-stu-id="1ae68-186">The default retry policy uses exponential back-off.</span></span> <span data-ttu-id="1ae68-187">若要使用其他重试策略，请调用 `SearchIndexClient` 或 `SearchServiceClient` 类上的 `SetRetryPolicy`。</span><span class="sxs-lookup"><span data-stu-id="1ae68-187">To use a different retry policy, call `SetRetryPolicy` on the `SearchIndexClient` or `SearchServiceClient` class.</span></span> <span data-ttu-id="1ae68-188">有关详细信息，请参阅[自动重试][auto-rest-client-retry]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-188">For more information, see [Automatic Retries][auto-rest-client-retry].</span></span>

<span data-ttu-id="1ae68-189">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-189">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-190">使用[搜索流量分析][search-analytics]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-190">Use [Search Traffic Analytics][search-analytics].</span></span>

### <a name="reading-data-from-azure-search-fails"></a><span data-ttu-id="1ae68-191">从 Azure 搜索中读取数据失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-191">Reading data from Azure Search fails.</span></span>
<span data-ttu-id="1ae68-192">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-192">**Detection**.</span></span> <span data-ttu-id="1ae68-193">捕获 `Microsoft.Rest.Azure.CloudException` 错误。</span><span class="sxs-lookup"><span data-stu-id="1ae68-193">Catch `Microsoft.Rest.Azure.CloudException` errors.</span></span>

<span data-ttu-id="1ae68-194">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-194">**Recovery**</span></span>

<span data-ttu-id="1ae68-195">[搜索 .NET SDK][search-sdk] 在发生暂时性故障后会自动重试。</span><span class="sxs-lookup"><span data-stu-id="1ae68-195">The [Search .NET SDK][search-sdk]  automatically retries after transient failures.</span></span> <span data-ttu-id="1ae68-196">由客户端 SDK 引发的任何异常均应视为非暂时性错误。</span><span class="sxs-lookup"><span data-stu-id="1ae68-196">Any exceptions thrown by the client SDK should be treated as non-transient errors.</span></span>

<span data-ttu-id="1ae68-197">默认重试策略使用指数回退。</span><span class="sxs-lookup"><span data-stu-id="1ae68-197">The default retry policy uses exponential back-off.</span></span> <span data-ttu-id="1ae68-198">若要使用其他重试策略，请调用 `SearchIndexClient` 或 `SearchServiceClient` 类上的 `SetRetryPolicy`。</span><span class="sxs-lookup"><span data-stu-id="1ae68-198">To use a different retry policy, call `SetRetryPolicy` on the `SearchIndexClient` or `SearchServiceClient` class.</span></span> <span data-ttu-id="1ae68-199">有关详细信息，请参阅[自动重试][auto-rest-client-retry]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-199">For more information, see [Automatic Retries][auto-rest-client-retry].</span></span>

<span data-ttu-id="1ae68-200">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-200">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-201">使用[搜索流量分析][search-analytics]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-201">Use [Search Traffic Analytics][search-analytics].</span></span>

## <a name="cassandra"></a><span data-ttu-id="1ae68-202">Cassandra</span><span class="sxs-lookup"><span data-stu-id="1ae68-202">Cassandra</span></span>
### <a name="reading-or-writing-to-a-node-fails"></a><span data-ttu-id="1ae68-203">读取或写入节点失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-203">Reading or writing to a node fails.</span></span>
<span data-ttu-id="1ae68-204">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-204">**Detection**.</span></span> <span data-ttu-id="1ae68-205">捕获异常。</span><span class="sxs-lookup"><span data-stu-id="1ae68-205">Catch the exception.</span></span> <span data-ttu-id="1ae68-206">对于 .NET 客户端，通常为 `System.Web.HttpException`。</span><span class="sxs-lookup"><span data-stu-id="1ae68-206">For .NET clients, this will typically be `System.Web.HttpException`.</span></span> <span data-ttu-id="1ae68-207">其他客户端可能有其他异常类型。</span><span class="sxs-lookup"><span data-stu-id="1ae68-207">Other client may have other exception types.</span></span>  <span data-ttu-id="1ae68-208">有关详细信息，请参阅 [Cassandra error handling done right](http://www.datastax.com/dev/blog/cassandra-error-handling-done-right)（正确的 Cassandra 错误处理）。</span><span class="sxs-lookup"><span data-stu-id="1ae68-208">For more information, see [Cassandra error handling done right](http://www.datastax.com/dev/blog/cassandra-error-handling-done-right).</span></span>

<span data-ttu-id="1ae68-209">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-209">**Recovery**</span></span>

* <span data-ttu-id="1ae68-210">每个 [Cassandra 客户端](https://wiki.apache.org/cassandra/ClientOptions)都有自己的重试策略和功能。</span><span class="sxs-lookup"><span data-stu-id="1ae68-210">Each [Cassandra client](https://wiki.apache.org/cassandra/ClientOptions) has its own retry policies and capabilities.</span></span> <span data-ttu-id="1ae68-211">有关详细信息，请参阅 [Cassandra error handling done right][cassandra-error-handling]（正确的 Cassandra 错误处理）。</span><span class="sxs-lookup"><span data-stu-id="1ae68-211">For more information, see [Cassandra error handling done right][cassandra-error-handling].</span></span>
* <span data-ttu-id="1ae68-212">使用机架感知部署，将数据节点分布在各个容错域中。</span><span class="sxs-lookup"><span data-stu-id="1ae68-212">Use a rack-aware deployment, with data nodes distributed across the fault domains.</span></span>
* <span data-ttu-id="1ae68-213">部署到具有本地仲裁一致性的多个区域。</span><span class="sxs-lookup"><span data-stu-id="1ae68-213">Deploy to multiple regions with local quorum consistency.</span></span> <span data-ttu-id="1ae68-214">如果发生非暂时性故障，则故障转移到另一个区域。</span><span class="sxs-lookup"><span data-stu-id="1ae68-214">If a non-transient failure occurs, fail over to another region.</span></span>

<span data-ttu-id="1ae68-215">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-215">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-216">应用程序日志</span><span class="sxs-lookup"><span data-stu-id="1ae68-216">Application logs</span></span>

## <a name="cloud-service"></a><span data-ttu-id="1ae68-217">云服务</span><span class="sxs-lookup"><span data-stu-id="1ae68-217">Cloud Service</span></span>
### <a name="web-or-worker-roles-are-unexpectedlybeing-shut-down"></a><span data-ttu-id="1ae68-218">Web 角色或辅助角色意外关闭。</span><span class="sxs-lookup"><span data-stu-id="1ae68-218">Web or worker roles are unexpectedly being shut down.</span></span>
<span data-ttu-id="1ae68-219">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-219">**Detection**.</span></span> <span data-ttu-id="1ae68-220">触发了 [RoleEnvironment.Stopping][RoleEnvironment.Stopping] 事件。</span><span class="sxs-lookup"><span data-stu-id="1ae68-220">The [RoleEnvironment.Stopping][RoleEnvironment.Stopping] event is fired.</span></span>

<span data-ttu-id="1ae68-221">**恢复**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-221">**Recovery**.</span></span> <span data-ttu-id="1ae68-222">重写 [RoleEntryPoint.OnStop][RoleEntryPoint.OnStop] 方法以便正常清理。</span><span class="sxs-lookup"><span data-stu-id="1ae68-222">Override the [RoleEntryPoint.OnStop][RoleEntryPoint.OnStop] method to gracefully clean up.</span></span> <span data-ttu-id="1ae68-223">有关详细信息，请参阅[处理 Azure OnStop 事件的正确方式][onstop-events]（博客）。</span><span class="sxs-lookup"><span data-stu-id="1ae68-223">For more information, see [The Right Way to Handle Azure OnStop Events][onstop-events] (blog).</span></span>

## <a name="cosmos-db"></a><span data-ttu-id="1ae68-224">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="1ae68-224">Cosmos DB</span></span> 
### <a name="reading-data-fails"></a><span data-ttu-id="1ae68-225">读取数据失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-225">Reading data fails.</span></span>
<span data-ttu-id="1ae68-226">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-226">**Detection**.</span></span> <span data-ttu-id="1ae68-227">捕获 `System.Net.Http.HttpRequestException` 或 `Microsoft.Azure.Documents.DocumentClientException`。</span><span class="sxs-lookup"><span data-stu-id="1ae68-227">Catch `System.Net.Http.HttpRequestException` or `Microsoft.Azure.Documents.DocumentClientException`.</span></span>

<span data-ttu-id="1ae68-228">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-228">**Recovery**</span></span>

* <span data-ttu-id="1ae68-229">SDK 自动重试失败的尝试。</span><span class="sxs-lookup"><span data-stu-id="1ae68-229">The SDK automatically retries failed attempts.</span></span> <span data-ttu-id="1ae68-230">若要设置重试次数和最长等待时间，请配置 `ConnectionPolicy.RetryOptions`。</span><span class="sxs-lookup"><span data-stu-id="1ae68-230">To set the number of retries and the maximum wait time, configure `ConnectionPolicy.RetryOptions`.</span></span> <span data-ttu-id="1ae68-231">客户端引发的异常会超出重试策略，或不是暂时性错误。</span><span class="sxs-lookup"><span data-stu-id="1ae68-231">Exceptions that the client raises are either beyond the retry policy or are not transient errors.</span></span>
* <span data-ttu-id="1ae68-232">如果 Cosmos DB 限制客户端，它会返回 HTTP 429 错误。</span><span class="sxs-lookup"><span data-stu-id="1ae68-232">If Cosmos DB throttles the client, it returns an HTTP 429 error.</span></span> <span data-ttu-id="1ae68-233">请检查 `DocumentClientException` 中的状态代码。</span><span class="sxs-lookup"><span data-stu-id="1ae68-233">Check the status code in the `DocumentClientException`.</span></span> <span data-ttu-id="1ae68-234">如果一直收到错误 429，请考虑增加集合的吞吐量值。</span><span class="sxs-lookup"><span data-stu-id="1ae68-234">If you are getting error 429 consistently, consider increasing the throughput value of the collection.</span></span>
    * <span data-ttu-id="1ae68-235">如果使用的是 MongoDB API，该服务会在进行限制时返回错误代码 16500。</span><span class="sxs-lookup"><span data-stu-id="1ae68-235">If you are using the MongoDB API, the service returns error code 16500 when throttling.</span></span>
* <span data-ttu-id="1ae68-236">跨两个或更多区域复制 Cosmos DB 数据库。</span><span class="sxs-lookup"><span data-stu-id="1ae68-236">Replicate the Cosmos DB database across two or more regions.</span></span> <span data-ttu-id="1ae68-237">所有副本都是可读的。</span><span class="sxs-lookup"><span data-stu-id="1ae68-237">All replicas are readable.</span></span> <span data-ttu-id="1ae68-238">使用客户端 SDK 指定 `PreferredLocations` 参数。</span><span class="sxs-lookup"><span data-stu-id="1ae68-238">Using the client SDKs, specify the `PreferredLocations` parameter.</span></span> <span data-ttu-id="1ae68-239">这是一个已排序的 Azure 区域列表。</span><span class="sxs-lookup"><span data-stu-id="1ae68-239">This is an ordered list of Azure regions.</span></span> <span data-ttu-id="1ae68-240">所有读取请求将发送到列表中的第一个可用区域。</span><span class="sxs-lookup"><span data-stu-id="1ae68-240">All reads will be sent to the first available region in the list.</span></span> <span data-ttu-id="1ae68-241">如果请求失败，客户端会按顺序尝试发送到列表中的其他区域。</span><span class="sxs-lookup"><span data-stu-id="1ae68-241">If the request fails, the client will try the other regions in the list, in order.</span></span> <span data-ttu-id="1ae68-242">有关详细信息，请参阅[如何使用 DocumentDB API 设置 Azure Cosmos DB 全局分发][docdb-multi-region]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-242">For more information, see [How to setup Azure Cosmos DB global distribution using the DocumentDB API][docdb-multi-region].</span></span>

<span data-ttu-id="1ae68-243">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-243">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-244">记录客户端上的所有错误。</span><span class="sxs-lookup"><span data-stu-id="1ae68-244">Log all errors on the client side.</span></span>

### <a name="writing-data-fails"></a><span data-ttu-id="1ae68-245">写入数据失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-245">Writing data fails.</span></span>
<span data-ttu-id="1ae68-246">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-246">**Detection**.</span></span> <span data-ttu-id="1ae68-247">捕获 `System.Net.Http.HttpRequestException` 或 `Microsoft.Azure.Documents.DocumentClientException`。</span><span class="sxs-lookup"><span data-stu-id="1ae68-247">Catch `System.Net.Http.HttpRequestException` or `Microsoft.Azure.Documents.DocumentClientException`.</span></span>

<span data-ttu-id="1ae68-248">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-248">**Recovery**</span></span>

* <span data-ttu-id="1ae68-249">SDK 自动重试失败的尝试。</span><span class="sxs-lookup"><span data-stu-id="1ae68-249">The SDK automatically retries failed attempts.</span></span> <span data-ttu-id="1ae68-250">若要设置重试次数和最长等待时间，请配置 `ConnectionPolicy.RetryOptions`。</span><span class="sxs-lookup"><span data-stu-id="1ae68-250">To set the number of retries and the maximum wait time, configure `ConnectionPolicy.RetryOptions`.</span></span> <span data-ttu-id="1ae68-251">客户端引发的异常会超出重试策略，或不是暂时性错误。</span><span class="sxs-lookup"><span data-stu-id="1ae68-251">Exceptions that the client raises are either beyond the retry policy or are not transient errors.</span></span>
* <span data-ttu-id="1ae68-252">如果 Cosmos DB 限制客户端，它会返回 HTTP 429 错误。</span><span class="sxs-lookup"><span data-stu-id="1ae68-252">If Cosmos DB throttles the client, it returns an HTTP 429 error.</span></span> <span data-ttu-id="1ae68-253">请检查 `DocumentClientException` 中的状态代码。</span><span class="sxs-lookup"><span data-stu-id="1ae68-253">Check the status code in the `DocumentClientException`.</span></span> <span data-ttu-id="1ae68-254">如果一直收到错误 429，请考虑增加集合的吞吐量值。</span><span class="sxs-lookup"><span data-stu-id="1ae68-254">If you are getting error 429 consistently, consider increasing the throughput value of the collection.</span></span>
* <span data-ttu-id="1ae68-255">跨两个或更多区域复制 Cosmos DB 数据库。</span><span class="sxs-lookup"><span data-stu-id="1ae68-255">Replicate the Cosmos DB database across two or more regions.</span></span> <span data-ttu-id="1ae68-256">如果主要区域发生故障，将提升另一个区域用于写入。</span><span class="sxs-lookup"><span data-stu-id="1ae68-256">If the primary region fails, another region will be promoted to write.</span></span> <span data-ttu-id="1ae68-257">也可以手动触发故障转移。</span><span class="sxs-lookup"><span data-stu-id="1ae68-257">You can also trigger a failover manually.</span></span> <span data-ttu-id="1ae68-258">SDK 执行自动发现和路由，以便应用程序代码在故障转移后继续工作。</span><span class="sxs-lookup"><span data-stu-id="1ae68-258">The SDK does automatic discovery and routing, so application code continues to work after a failover.</span></span> <span data-ttu-id="1ae68-259">在故障转移期间（通常按分钟计），写入操作将有更高的延迟，因为 SDK 会查找新的写入区域。</span><span class="sxs-lookup"><span data-stu-id="1ae68-259">During the failover period (typically minutes), write operations will have higher latency, as the SDK finds the new write region.</span></span>
  <span data-ttu-id="1ae68-260">有关详细信息，请参阅[如何使用 DocumentDB API 设置 Azure Cosmos DB 全局分发][docdb-multi-region]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-260">For more information, see [How to setup Azure Cosmos DB global distribution using the DocumentDB API][docdb-multi-region].</span></span>
* <span data-ttu-id="1ae68-261">作为回退策略，可将文档暂留到备份队列中，稍后再处理该队列。</span><span class="sxs-lookup"><span data-stu-id="1ae68-261">As a fallback, persist the document to a backup queue, and process the queue later.</span></span>

<span data-ttu-id="1ae68-262">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-262">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-263">记录客户端上的所有错误。</span><span class="sxs-lookup"><span data-stu-id="1ae68-263">Log all errors on the client side.</span></span>

## <a name="elasticsearch"></a><span data-ttu-id="1ae68-264">Elasticsearch</span><span class="sxs-lookup"><span data-stu-id="1ae68-264">Elasticsearch</span></span>
### <a name="reading-data-from-elasticsearch-fails"></a><span data-ttu-id="1ae68-265">从 Elasticsearch 读取数据失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-265">Reading data from Elasticsearch fails.</span></span>
<span data-ttu-id="1ae68-266">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-266">**Detection**.</span></span> <span data-ttu-id="1ae68-267">捕获正在使用的特定 [Elasticsearch 客户端][elasticsearch-client]的相应异常。</span><span class="sxs-lookup"><span data-stu-id="1ae68-267">Catch the appropriate exception for the particular [Elasticsearch client][elasticsearch-client] being used.</span></span>

<span data-ttu-id="1ae68-268">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-268">**Recovery**</span></span>

* <span data-ttu-id="1ae68-269">使用重试机制。</span><span class="sxs-lookup"><span data-stu-id="1ae68-269">Use a retry mechanism.</span></span> <span data-ttu-id="1ae68-270">每个客户端都有自己的重试策略。</span><span class="sxs-lookup"><span data-stu-id="1ae68-270">Each client has its own retry policies.</span></span>
* <span data-ttu-id="1ae68-271">部署多个 Elasticsearch 节点并使用复制来实现高可用性。</span><span class="sxs-lookup"><span data-stu-id="1ae68-271">Deploy multiple Elasticsearch nodes and use replication for high availability.</span></span>

<span data-ttu-id="1ae68-272">有关详细信息，请参阅[在 Azure 上运行 Elasticsearch][elasticsearch-azure]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-272">For more information, see [Running Elasticsearch on Azure][elasticsearch-azure].</span></span>

<span data-ttu-id="1ae68-273">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-273">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-274">可以对 Elasticsearch 使用监视工具，或记录具有有效负载的客户端上的所有错误。</span><span class="sxs-lookup"><span data-stu-id="1ae68-274">You can use monitoring tools for Elasticsearch, or log all errors on the client side with the payload.</span></span> <span data-ttu-id="1ae68-275">请参阅[在 Azure 上运行 Elasticsearch][elasticsearch-azure] 中的“监视”部分。</span><span class="sxs-lookup"><span data-stu-id="1ae68-275">See the 'Monitoring' section in [Running Elasticsearch on Azure][elasticsearch-azure].</span></span>

### <a name="writing-data-to-elasticsearch-fails"></a><span data-ttu-id="1ae68-276">将数据写入 Elasticsearch 失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-276">Writing data to Elasticsearch fails.</span></span>
<span data-ttu-id="1ae68-277">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-277">**Detection**.</span></span> <span data-ttu-id="1ae68-278">捕获正在使用的特定 [Elasticsearch 客户端][elasticsearch-client]的相应异常。</span><span class="sxs-lookup"><span data-stu-id="1ae68-278">Catch the appropriate exception for the particular [Elasticsearch client][elasticsearch-client] being used.</span></span>  

<span data-ttu-id="1ae68-279">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-279">**Recovery**</span></span>

* <span data-ttu-id="1ae68-280">使用重试机制。</span><span class="sxs-lookup"><span data-stu-id="1ae68-280">Use a retry mechanism.</span></span> <span data-ttu-id="1ae68-281">每个客户端都有自己的重试策略。</span><span class="sxs-lookup"><span data-stu-id="1ae68-281">Each client has its own retry policies.</span></span>
* <span data-ttu-id="1ae68-282">如果应用程序可承受降低的一致性级别，则考虑在写入时将 `write_consistency` 设置为 `quorum`。</span><span class="sxs-lookup"><span data-stu-id="1ae68-282">If the application can tolerate a reduced consistency level, consider writing with `write_consistency` setting of `quorum`.</span></span>

<span data-ttu-id="1ae68-283">有关详细信息，请参阅[在 Azure 上运行 Elasticsearch][elasticsearch-azure]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-283">For more information, see [Running Elasticsearch on Azure][elasticsearch-azure].</span></span>

<span data-ttu-id="1ae68-284">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-284">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-285">可以对 Elasticsearch 使用监视工具，或记录具有有效负载的客户端上的所有错误。</span><span class="sxs-lookup"><span data-stu-id="1ae68-285">You can use monitoring tools for Elasticsearch, or log all errors on the client side with the payload.</span></span> <span data-ttu-id="1ae68-286">请参阅[在 Azure 上运行 Elasticsearch][elasticsearch-azure] 中的“监视”部分。</span><span class="sxs-lookup"><span data-stu-id="1ae68-286">See the 'Monitoring' section in [Running Elasticsearch on Azure][elasticsearch-azure].</span></span>

## <a name="queue-storage"></a><span data-ttu-id="1ae68-287">队列存储</span><span class="sxs-lookup"><span data-stu-id="1ae68-287">Queue storage</span></span>
### <a name="writing-a-message-to-azure-queue-storage-fails-consistently"></a><span data-ttu-id="1ae68-288">将消息写入 Azure 队列存储一直失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-288">Writing a message to Azure Queue storage fails consistently.</span></span>
<span data-ttu-id="1ae68-289">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-289">**Detection**.</span></span> <span data-ttu-id="1ae68-290">在尝试 *N* 次重试后，写入操作仍然失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-290">After *N* retry attempts, the write operation still fails.</span></span>

<span data-ttu-id="1ae68-291">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-291">**Recovery**</span></span>

* <span data-ttu-id="1ae68-292">将数据存储在本地缓存中，等服务变为可用后，再将写入数据转发到存储。</span><span class="sxs-lookup"><span data-stu-id="1ae68-292">Store the data in a local cache, and forward the writes to storage later, when the service becomes available.</span></span>
* <span data-ttu-id="1ae68-293">如果主队列不可用，则创建辅助队列并写入该队列。</span><span class="sxs-lookup"><span data-stu-id="1ae68-293">Create a secondary queue, and write to that queue if the primary queue is unavailable.</span></span>

<span data-ttu-id="1ae68-294">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-294">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-295">使用[存储度量值][storage-metrics]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-295">Use [storage metrics][storage-metrics].</span></span>

### <a name="the-application-cannot-process-a-particular-message-from-the-queue"></a><span data-ttu-id="1ae68-296">应用程序无法处理来自队列的某条特定消息。</span><span class="sxs-lookup"><span data-stu-id="1ae68-296">The application cannot process a particular message from the queue.</span></span>
<span data-ttu-id="1ae68-297">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-297">**Detection**.</span></span> <span data-ttu-id="1ae68-298">特定于应用程序。</span><span class="sxs-lookup"><span data-stu-id="1ae68-298">Application specific.</span></span> <span data-ttu-id="1ae68-299">例如，该消息包含无效数据，或业务逻辑因某些原因而失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-299">For example, the message contains invalid data, or the business logic fails for some reason.</span></span>

<span data-ttu-id="1ae68-300">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-300">**Recovery**</span></span>

<span data-ttu-id="1ae68-301">将该消息移到单独的队列中。</span><span class="sxs-lookup"><span data-stu-id="1ae68-301">Move the message to a separate queue.</span></span> <span data-ttu-id="1ae68-302">运行单独的进程来检查该队列中的消息。</span><span class="sxs-lookup"><span data-stu-id="1ae68-302">Run a separate process to examine the messages in that queue.</span></span>

<span data-ttu-id="1ae68-303">考虑使用 Azure 服务总线消息传送队列，它会为此提供[死信队列][sb-dead-letter-queue]功能。</span><span class="sxs-lookup"><span data-stu-id="1ae68-303">Consider using Azure Service Bus Messaging queues, which provides a [dead-letter queue][sb-dead-letter-queue] functionality for this purpose.</span></span>

> [!NOTE]
> <span data-ttu-id="1ae68-304">如果通过 WebJobs 使用存储队列，WebJobs SDK 会提供内置的有害消息处理功能。</span><span class="sxs-lookup"><span data-stu-id="1ae68-304">If you are using Storage queues with WebJobs, the WebJobs SDK provides built-in poison message handling.</span></span> <span data-ttu-id="1ae68-305">请参阅[如何通过 WebJobs SDK 使用 Azure 队列存储][sb-poison-message]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-305">See [How to use Azure queue storage with the WebJobs SDK][sb-poison-message].</span></span>

<span data-ttu-id="1ae68-306">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-306">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-307">使用应用程序日志记录。</span><span class="sxs-lookup"><span data-stu-id="1ae68-307">Use application logging.</span></span>

## <a name="redis-cache"></a><span data-ttu-id="1ae68-308">Redis 缓存</span><span class="sxs-lookup"><span data-stu-id="1ae68-308">Redis Cache</span></span>
### <a name="reading-from-the-cache-fails"></a><span data-ttu-id="1ae68-309">从缓存读取失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-309">Reading from the cache fails.</span></span>
<span data-ttu-id="1ae68-310">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-310">**Detection**.</span></span> <span data-ttu-id="1ae68-311">捕获 `StackExchange.Redis.RedisConnectionException`。</span><span class="sxs-lookup"><span data-stu-id="1ae68-311">Catch `StackExchange.Redis.RedisConnectionException`.</span></span>

<span data-ttu-id="1ae68-312">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-312">**Recovery**</span></span>

1. <span data-ttu-id="1ae68-313">对暂时性故障进行重试。</span><span class="sxs-lookup"><span data-stu-id="1ae68-313">Retry on transient failures.</span></span> <span data-ttu-id="1ae68-314">Azure Redis 缓存支持内置的重试，请参阅 [Redis 缓存重试准则][redis-retry]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-314">Azure Redis cache supports built-in retry through See [Redis Cache retry guidelines][redis-retry].</span></span>
2. <span data-ttu-id="1ae68-315">将非暂时性故障视为缓存失误，并回退到原始数据源。</span><span class="sxs-lookup"><span data-stu-id="1ae68-315">Treat non-transient failures as a cache miss, and fall back to the original data source.</span></span>

<span data-ttu-id="1ae68-316">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-316">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-317">使用 [Redis 缓存诊断][redis-monitor]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-317">Use [Redis Cache diagnostics][redis-monitor].</span></span>

### <a name="writing-to-the-cache-fails"></a><span data-ttu-id="1ae68-318">写入缓存失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-318">Writing to the cache fails.</span></span>
<span data-ttu-id="1ae68-319">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-319">**Detection**.</span></span> <span data-ttu-id="1ae68-320">捕获 `StackExchange.Redis.RedisConnectionException`。</span><span class="sxs-lookup"><span data-stu-id="1ae68-320">Catch `StackExchange.Redis.RedisConnectionException`.</span></span>

<span data-ttu-id="1ae68-321">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-321">**Recovery**</span></span>

1. <span data-ttu-id="1ae68-322">对暂时性故障进行重试。</span><span class="sxs-lookup"><span data-stu-id="1ae68-322">Retry on transient failures.</span></span> <span data-ttu-id="1ae68-323">Azure Redis 缓存支持内置的重试，请参阅 [Redis 缓存重试准则][redis-retry]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-323">Azure Redis cache supports built-in retry through See [Redis Cache retry guidelines][redis-retry].</span></span>
2. <span data-ttu-id="1ae68-324">如果是非暂时性错误，则忽略它，稍后让其他事务写入缓存。</span><span class="sxs-lookup"><span data-stu-id="1ae68-324">If the error is non-transient, ignore it and let other transactions write to the cache later.</span></span>

<span data-ttu-id="1ae68-325">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-325">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-326">使用 [Redis 缓存诊断][redis-monitor]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-326">Use [Redis Cache diagnostics][redis-monitor].</span></span>

## <a name="sql-database"></a><span data-ttu-id="1ae68-327">SQL 数据库</span><span class="sxs-lookup"><span data-stu-id="1ae68-327">SQL Database</span></span>
### <a name="cannot-connect-to-the-database-in-the-primary-region"></a><span data-ttu-id="1ae68-328">无法连接到主要区域中的数据库。</span><span class="sxs-lookup"><span data-stu-id="1ae68-328">Cannot connect to the database in the primary region.</span></span>
<span data-ttu-id="1ae68-329">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-329">**Detection**.</span></span> <span data-ttu-id="1ae68-330">连接失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-330">Connection fails.</span></span>

<span data-ttu-id="1ae68-331">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-331">**Recovery**</span></span>

<span data-ttu-id="1ae68-332">先决条件：必须为数据库配置活动异地复制。</span><span class="sxs-lookup"><span data-stu-id="1ae68-332">Prerequisite: The database must be configured for active geo-replication.</span></span> <span data-ttu-id="1ae68-333">请参阅 [SQL 数据库活动异地复制][sql-db-replication]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-333">See [SQL Database Active Geo-Replication][sql-db-replication].</span></span>

* <span data-ttu-id="1ae68-334">对于查询，请从次要副本中读取。</span><span class="sxs-lookup"><span data-stu-id="1ae68-334">For queries, read from a secondary replica.</span></span>
* <span data-ttu-id="1ae68-335">对于插入和更新，请手动故障转移到次要副本。</span><span class="sxs-lookup"><span data-stu-id="1ae68-335">For inserts and updates, manually fail over to a secondary replica.</span></span> <span data-ttu-id="1ae68-336">请参阅[为 Azure SQL 数据库启动计划内或计划外故障转移][sql-db-failover]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-336">See [Initiate a planned or unplanned failover for Azure SQL Database][sql-db-failover].</span></span>

<span data-ttu-id="1ae68-337">该副本使用不同的连接字符串，因此需要更新应用程序中的连接字符串。</span><span class="sxs-lookup"><span data-stu-id="1ae68-337">The replica uses a different connection string, so you will need to update the connection string in your application.</span></span>

### <a name="client-runs-out-of-connections-in-the-connection-pool"></a><span data-ttu-id="1ae68-338">客户端用尽了连接池中的连接。</span><span class="sxs-lookup"><span data-stu-id="1ae68-338">Client runs out of connections in the connection pool.</span></span>
<span data-ttu-id="1ae68-339">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-339">**Detection**.</span></span> <span data-ttu-id="1ae68-340">捕获 `System.InvalidOperationException` 错误。</span><span class="sxs-lookup"><span data-stu-id="1ae68-340">Catch `System.InvalidOperationException` errors.</span></span>

<span data-ttu-id="1ae68-341">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-341">**Recovery**</span></span>

* <span data-ttu-id="1ae68-342">请重试操作即可。</span><span class="sxs-lookup"><span data-stu-id="1ae68-342">Retry the operation.</span></span>
* <span data-ttu-id="1ae68-343">作为缓解计划，可针对每个用例将连接池隔离开来，使一个用例无法支配所有连接。</span><span class="sxs-lookup"><span data-stu-id="1ae68-343">As a mitigation plan, isolate the connection pools for each use case, so that one use case can't dominate all the connections.</span></span>
* <span data-ttu-id="1ae68-344">增加连接池数量上限。</span><span class="sxs-lookup"><span data-stu-id="1ae68-344">Increase the maximum connection pools.</span></span>

<span data-ttu-id="1ae68-345">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-345">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-346">应用程序日志。</span><span class="sxs-lookup"><span data-stu-id="1ae68-346">Application logs.</span></span>

### <a name="database-connection-limit-is-reached"></a><span data-ttu-id="1ae68-347">达到数据库连接限制。</span><span class="sxs-lookup"><span data-stu-id="1ae68-347">Database connection limit is reached.</span></span>
<span data-ttu-id="1ae68-348">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-348">**Detection**.</span></span> <span data-ttu-id="1ae68-349">Azure SQL 数据库限制并发辅助角色、登录名和会话的数目。</span><span class="sxs-lookup"><span data-stu-id="1ae68-349">Azure SQL Database limits the number of concurrent workers, logins, and sessions.</span></span> <span data-ttu-id="1ae68-350">具体限制视服务层而定。</span><span class="sxs-lookup"><span data-stu-id="1ae68-350">The limits depend on the service tier.</span></span> <span data-ttu-id="1ae68-351">有关详细信息，请参阅 [Azure SQL 数据库资源限制][sql-db-limits]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-351">For more information, see [Azure SQL Database resource limits][sql-db-limits].</span></span>

<span data-ttu-id="1ae68-352">若要检测这些错误，请捕获 `System.Data.SqlClient.SqlException` 并检查 SQL 错误代码中 `SqlException.Number` 的值。</span><span class="sxs-lookup"><span data-stu-id="1ae68-352">To detect these errors, catch `System.Data.SqlClient.SqlException` and check the value of `SqlException.Number` for the SQL error code.</span></span> <span data-ttu-id="1ae68-353">有关相关错误代码的列表，请参阅 [SQL 数据库客户端应用程序的 SQL 错误代码：数据库连接错误和其他问题][sql-db-errors]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-353">For a list of relevant error codes, see [SQL error codes for SQL Database client applications: Database connection error and other issues][sql-db-errors].</span></span>

<span data-ttu-id="1ae68-354">**恢复**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-354">**Recovery**.</span></span> <span data-ttu-id="1ae68-355">这些错误被视为暂时性错误，因此重试可能解决此问题。</span><span class="sxs-lookup"><span data-stu-id="1ae68-355">These errors are considered transient, so retrying may resolve the issue.</span></span> <span data-ttu-id="1ae68-356">如果持续遇到这些错误，请考虑缩放数据库。</span><span class="sxs-lookup"><span data-stu-id="1ae68-356">If you consistently hit these errors, consider scaling the database.</span></span>

<span data-ttu-id="1ae68-357">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-357">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-358">- [sys.event_log][sys.event_log] 查询返回成功的数据库连接、失败的连接和死锁。</span><span class="sxs-lookup"><span data-stu-id="1ae68-358">- The [sys.event_log][sys.event_log] query returns successful database connections, connection failures, and deadlocks.</span></span>

* <span data-ttu-id="1ae68-359">为失败的连接创建[警报规则][azure-alerts]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-359">Create an [alert rule][azure-alerts] for failed connections.</span></span>
* <span data-ttu-id="1ae68-360">启用 [SQL 数据库审核][sql-db-audit]并检查失败的登录。</span><span class="sxs-lookup"><span data-stu-id="1ae68-360">Enable [SQL Database auditing][sql-db-audit] and check for failed logins.</span></span>

## <a name="service-bus-messaging"></a><span data-ttu-id="1ae68-361">服务总线消息传送</span><span class="sxs-lookup"><span data-stu-id="1ae68-361">Service Bus Messaging</span></span>
### <a name="reading-a-message-from-a-service-bus-queue-fails"></a><span data-ttu-id="1ae68-362">从服务总线队列读取消息失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-362">Reading a message from a Service Bus queue fails.</span></span>
<span data-ttu-id="1ae68-363">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-363">**Detection**.</span></span> <span data-ttu-id="1ae68-364">捕获客户端 SDK 中的异常。</span><span class="sxs-lookup"><span data-stu-id="1ae68-364">Catch exceptions from the client SDK.</span></span> <span data-ttu-id="1ae68-365">服务总线异常的基类是 [MessagingException][sb-messagingexception-class]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-365">The base class for Service Bus exceptions is [MessagingException][sb-messagingexception-class].</span></span> <span data-ttu-id="1ae68-366">如果是暂时性错误，则 `IsTransient` 属性为 true。</span><span class="sxs-lookup"><span data-stu-id="1ae68-366">If the error is transient, the `IsTransient` property is true.</span></span>

<span data-ttu-id="1ae68-367">有关详细信息，请参阅[服务总线消息传送异常][sb-messaging-exceptions]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-367">For more information, see [Service Bus messaging exceptions][sb-messaging-exceptions].</span></span>

<span data-ttu-id="1ae68-368">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-368">**Recovery**</span></span>

1. <span data-ttu-id="1ae68-369">对暂时性故障进行重试。</span><span class="sxs-lookup"><span data-stu-id="1ae68-369">Retry on transient failures.</span></span> <span data-ttu-id="1ae68-370">请参阅[服务总线重试准则][sb-retry]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-370">See [Service Bus retry guidelines][sb-retry].</span></span>
2. <span data-ttu-id="1ae68-371">无法传递给任何接收方的消息位于*死信队列*中。</span><span class="sxs-lookup"><span data-stu-id="1ae68-371">Messages that cannot be delivered to any receiver are placed in a *dead-letter queue*.</span></span> <span data-ttu-id="1ae68-372">可使用此队列查看无法接收哪些消息。</span><span class="sxs-lookup"><span data-stu-id="1ae68-372">Use this queue to see which messages could not be received.</span></span> <span data-ttu-id="1ae68-373">死信队列不自动执行清理。</span><span class="sxs-lookup"><span data-stu-id="1ae68-373">There is no automatic cleanup of the dead-letter queue.</span></span> <span data-ttu-id="1ae68-374">在显式检索这些消息前，它们会一直留在其中。</span><span class="sxs-lookup"><span data-stu-id="1ae68-374">Messages remain there until you explicitly retrieve them.</span></span> <span data-ttu-id="1ae68-375">请参阅[服务总线死信队列概述][sb-dead-letter-queue]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-375">See [Overview of Service Bus dead-letter queues][sb-dead-letter-queue].</span></span>

### <a name="writing-a-message-to-a-service-bus-queue-fails"></a><span data-ttu-id="1ae68-376">向服务总线队列写入消息失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-376">Writing a message to a Service Bus queue fails.</span></span>
<span data-ttu-id="1ae68-377">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-377">**Detection**.</span></span> <span data-ttu-id="1ae68-378">捕获客户端 SDK 中的异常。</span><span class="sxs-lookup"><span data-stu-id="1ae68-378">Catch exceptions from the client SDK.</span></span> <span data-ttu-id="1ae68-379">服务总线异常的基类是 [MessagingException][sb-messagingexception-class]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-379">The base class for Service Bus exceptions is [MessagingException][sb-messagingexception-class].</span></span> <span data-ttu-id="1ae68-380">如果是暂时性错误，则 `IsTransient` 属性为 true。</span><span class="sxs-lookup"><span data-stu-id="1ae68-380">If the error is transient, the `IsTransient` property is true.</span></span>

<span data-ttu-id="1ae68-381">有关详细信息，请参阅[服务总线消息传送异常][sb-messaging-exceptions]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-381">For more information, see [Service Bus messaging exceptions][sb-messaging-exceptions].</span></span>

<span data-ttu-id="1ae68-382">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-382">**Recovery**</span></span>

1. <span data-ttu-id="1ae68-383">服务总线客户端在发生暂时性错误后会自动重试。</span><span class="sxs-lookup"><span data-stu-id="1ae68-383">The Service Bus client automatically retries after transient errors.</span></span> <span data-ttu-id="1ae68-384">默认情况下，它使用指数回退。</span><span class="sxs-lookup"><span data-stu-id="1ae68-384">By default, it uses exponential back-off.</span></span> <span data-ttu-id="1ae68-385">在达到最大重试计数或最长超时期限后，客户端会引发异常。</span><span class="sxs-lookup"><span data-stu-id="1ae68-385">After the maximum retry count or maximum timeout period, the client throws an exception.</span></span> <span data-ttu-id="1ae68-386">有关详细信息，请参阅[服务总线重试准则][sb-retry]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-386">For more information, see [Service Bus retry guidelines][sb-retry].</span></span>
2. <span data-ttu-id="1ae68-387">如果超过队列配额，客户端会引发 [QuotaExceededException][QuotaExceededException]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-387">If the queue quota is exceeded, the client throws [QuotaExceededException][QuotaExceededException].</span></span> <span data-ttu-id="1ae68-388">该异常消息会提供更多详细信息。</span><span class="sxs-lookup"><span data-stu-id="1ae68-388">The exception message gives more details.</span></span> <span data-ttu-id="1ae68-389">重试前清空队列中的某些消息，并考虑使用断路器模式，以免在超过配额时继续重试。</span><span class="sxs-lookup"><span data-stu-id="1ae68-389">Drain some messages from the queue before retrying, and consider using the Circuit Breaker pattern to avoid continued retries while the quota is exceeded.</span></span> <span data-ttu-id="1ae68-390">另外，确保 [BrokeredMessage.TimeToLive] 属性不要设置得过高。</span><span class="sxs-lookup"><span data-stu-id="1ae68-390">Also, make sure the [BrokeredMessage.TimeToLive] property is not set too high.</span></span>
3. <span data-ttu-id="1ae68-391">在一个区域内，可以使用[分区的队列和主题][sb-partition]提高复原能力。</span><span class="sxs-lookup"><span data-stu-id="1ae68-391">Within a region, resiliency can be improved by using [partitioned queues or topics][sb-partition].</span></span> <span data-ttu-id="1ae68-392">将未分区的队列或主题分配到一个消息存储空间。</span><span class="sxs-lookup"><span data-stu-id="1ae68-392">A non-partitioned queue or topic is assigned to one messaging store.</span></span> <span data-ttu-id="1ae68-393">如果此消息存储空间不可用，则针对该队列或主题的所有操作将都失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-393">If this messaging store is unavailable, all operations on that queue or topic will fail.</span></span> <span data-ttu-id="1ae68-394">将分区的队列或主题跨多个消息传送存储进行分区。</span><span class="sxs-lookup"><span data-stu-id="1ae68-394">A partitioned queue or topic is partitioned across multiple messaging stores.</span></span>
4. <span data-ttu-id="1ae68-395">若要提高复原能力，可在不同的区域中创建两个服务总线命名空间，并复制消息。</span><span class="sxs-lookup"><span data-stu-id="1ae68-395">For additional resiliency, create two Service Bus namespaces in different regions, and replicate the messages.</span></span> <span data-ttu-id="1ae68-396">可使用主动复制或被动复制。</span><span class="sxs-lookup"><span data-stu-id="1ae68-396">You can use either active replication or passive replication.</span></span>

   * <span data-ttu-id="1ae68-397">主动复制：客户端将每条消息同时发送到两个队列。</span><span class="sxs-lookup"><span data-stu-id="1ae68-397">Active replication: The client sends every message to both queues.</span></span> <span data-ttu-id="1ae68-398">接收方对两个队列都进行侦听。</span><span class="sxs-lookup"><span data-stu-id="1ae68-398">The receiver listens on both queues.</span></span> <span data-ttu-id="1ae68-399">使用唯一标识符标记消息，以便客户端放弃重复的消息。</span><span class="sxs-lookup"><span data-stu-id="1ae68-399">Tag messages with a unique identifier, so the client can discard duplicate messages.</span></span>
   * <span data-ttu-id="1ae68-400">被动复制：客户端将消息发送到一个队列。</span><span class="sxs-lookup"><span data-stu-id="1ae68-400">Passive replication: The client sends the message to one queue.</span></span> <span data-ttu-id="1ae68-401">如果出错，客户端回退到另一个队列。</span><span class="sxs-lookup"><span data-stu-id="1ae68-401">If there is an error, the client falls back to the other queue.</span></span> <span data-ttu-id="1ae68-402">接收方对两个队列都进行侦听。</span><span class="sxs-lookup"><span data-stu-id="1ae68-402">The receiver listens on both queues.</span></span> <span data-ttu-id="1ae68-403">此方法可以减少发送的重复消息数。</span><span class="sxs-lookup"><span data-stu-id="1ae68-403">This approach reduces the number of duplicate messages that are sent.</span></span> <span data-ttu-id="1ae68-404">但是，接收方仍须处理重复消息。</span><span class="sxs-lookup"><span data-stu-id="1ae68-404">However, the receiver must still handle duplicate messages.</span></span>

     <span data-ttu-id="1ae68-405">有关详细信息，请参阅[异地复制示例][sb-georeplication-sample]和[使应用程序免受服务总线中断和灾难影响的最佳实践](/azure/service-bus-messaging/service-bus-outages-disasters/)。</span><span class="sxs-lookup"><span data-stu-id="1ae68-405">For more information, see [GeoReplication sample][sb-georeplication-sample] and [Best practices for insulating applications against Service Bus outages and disasters](/azure/service-bus-messaging/service-bus-outages-disasters/).</span></span>

### <a name="duplicate-message"></a><span data-ttu-id="1ae68-406">消息重复。</span><span class="sxs-lookup"><span data-stu-id="1ae68-406">Duplicate message.</span></span>
<span data-ttu-id="1ae68-407">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-407">**Detection**.</span></span> <span data-ttu-id="1ae68-408">检查消息的 `MessageId` 和 `DeliveryCount` 属性。</span><span class="sxs-lookup"><span data-stu-id="1ae68-408">Examine the `MessageId` and `DeliveryCount` properties of the message.</span></span>

<span data-ttu-id="1ae68-409">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-409">**Recovery**</span></span>

* <span data-ttu-id="1ae68-410">如果可能，将消息处理操作设计为幂等。</span><span class="sxs-lookup"><span data-stu-id="1ae68-410">If possible, design your message processing operations to be idempotent.</span></span> <span data-ttu-id="1ae68-411">否则，存储已处理消息的消息 ID，并在处理消息之前检查 ID。</span><span class="sxs-lookup"><span data-stu-id="1ae68-411">Otherwise, store message IDs of messages that are already processed, and check the ID before processing a message.</span></span>
* <span data-ttu-id="1ae68-412">通过创建 `RequiresDuplicateDetection` 设置为 true 的队列，启用重复检测。</span><span class="sxs-lookup"><span data-stu-id="1ae68-412">Enable duplicate detection, by creating the queue with `RequiresDuplicateDetection` set to true.</span></span> <span data-ttu-id="1ae68-413">使用此设置时，服务总线会自动删除发送的与以前消息 `MessageId` 相同的所有消息。</span><span class="sxs-lookup"><span data-stu-id="1ae68-413">With this setting, Service Bus automatically deletes any message that is sent with the same `MessageId` as a previous message.</span></span>  <span data-ttu-id="1ae68-414">注意以下事项：</span><span class="sxs-lookup"><span data-stu-id="1ae68-414">Note the following:</span></span>

  * <span data-ttu-id="1ae68-415">此设置可防止将重复的消息放入队列中，</span><span class="sxs-lookup"><span data-stu-id="1ae68-415">This setting prevents duplicate messages from being put into the queue.</span></span> <span data-ttu-id="1ae68-416">但不能防止接收方多次处理同一消息。</span><span class="sxs-lookup"><span data-stu-id="1ae68-416">It doesn't prevent a receiver from processing the same message more than once.</span></span>
  * <span data-ttu-id="1ae68-417">重复检测具有时间范围。</span><span class="sxs-lookup"><span data-stu-id="1ae68-417">Duplicate detection has a time window.</span></span> <span data-ttu-id="1ae68-418">如果超过此范围发送重复项，则检测不到该重复项。</span><span class="sxs-lookup"><span data-stu-id="1ae68-418">If a duplicate is sent beyond this window, it won't be detected.</span></span>

<span data-ttu-id="1ae68-419">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-419">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-420">记录重复消息。</span><span class="sxs-lookup"><span data-stu-id="1ae68-420">Log duplicated messages.</span></span>

### <a name="the-application-cannot-process-a-particular-message-from-the-queue"></a><span data-ttu-id="1ae68-421">应用程序无法处理来自队列的某条特定消息。</span><span class="sxs-lookup"><span data-stu-id="1ae68-421">The application cannot process a particular message from the queue.</span></span>
<span data-ttu-id="1ae68-422">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-422">**Detection**.</span></span> <span data-ttu-id="1ae68-423">特定于应用程序。</span><span class="sxs-lookup"><span data-stu-id="1ae68-423">Application specific.</span></span> <span data-ttu-id="1ae68-424">例如，该消息包含无效数据，或业务逻辑因某些原因而失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-424">For example, the message contains invalid data, or the business logic fails for some reason.</span></span>

<span data-ttu-id="1ae68-425">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-425">**Recovery**</span></span>

<span data-ttu-id="1ae68-426">考虑以下两种故障模式。</span><span class="sxs-lookup"><span data-stu-id="1ae68-426">There are two failure modes to consider.</span></span>

* <span data-ttu-id="1ae68-427">接收方检测到故障。</span><span class="sxs-lookup"><span data-stu-id="1ae68-427">The receiver detects the failure.</span></span> <span data-ttu-id="1ae68-428">在此情况下，将该消息移至死信队列。</span><span class="sxs-lookup"><span data-stu-id="1ae68-428">In this case, move the message to the dead-letter queue.</span></span> <span data-ttu-id="1ae68-429">然后，运行单独的进程来检查死信队列中的消息。</span><span class="sxs-lookup"><span data-stu-id="1ae68-429">Later, run a separate process to examine the messages in the dead-letter queue.</span></span>
* <span data-ttu-id="1ae68-430">接收方在处理该消息时失败 &mdash; 例如，由于未经处理的异常。</span><span class="sxs-lookup"><span data-stu-id="1ae68-430">The receiver fails in the middle of processing the message &mdash; for example, due to an unhandled exception.</span></span> <span data-ttu-id="1ae68-431">若要处理这种情况，可使用 `PeekLock` 模式。</span><span class="sxs-lookup"><span data-stu-id="1ae68-431">To handle this case, use `PeekLock` mode.</span></span> <span data-ttu-id="1ae68-432">在此模式下，如果锁定到期，该消息会变为可供其他接收方处理。</span><span class="sxs-lookup"><span data-stu-id="1ae68-432">In this mode, if the lock expires, the message becomes available to other receivers.</span></span> <span data-ttu-id="1ae68-433">如果该消息超过最大传递计数或生存时间，则会自动移至死信队列。</span><span class="sxs-lookup"><span data-stu-id="1ae68-433">If the message exceeds the maximum delivery count or the time-to-live, the message is automatically moved to the dead-letter queue.</span></span>

<span data-ttu-id="1ae68-434">有关详细信息，请参阅[服务总线死信队列概述][sb-dead-letter-queue]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-434">For more information, see [Overview of Service Bus dead-letter queues][sb-dead-letter-queue].</span></span>

<span data-ttu-id="1ae68-435">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-435">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-436">应用程序每次将消息移至死信队列时，都会向应用程序日志写入一个事件。</span><span class="sxs-lookup"><span data-stu-id="1ae68-436">Whenever the application moves a message to the dead-letter queue, write an event to the application logs.</span></span>

## <a name="service-fabric"></a><span data-ttu-id="1ae68-437">Service Fabric</span><span class="sxs-lookup"><span data-stu-id="1ae68-437">Service Fabric</span></span>
### <a name="a-request-to-a-service-fails"></a><span data-ttu-id="1ae68-438">服务请求失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-438">A request to a service fails.</span></span>
<span data-ttu-id="1ae68-439">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-439">**Detection**.</span></span> <span data-ttu-id="1ae68-440">服务返回一个错误。</span><span class="sxs-lookup"><span data-stu-id="1ae68-440">The service returns an error.</span></span>

<span data-ttu-id="1ae68-441">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-441">**Recovery**</span></span>

* <span data-ttu-id="1ae68-442">重新找一个代理（`ServiceProxy` 或 `ActorProxy`），并重新调用该服务/参与者方法。</span><span class="sxs-lookup"><span data-stu-id="1ae68-442">Locate a proxy again (`ServiceProxy` or `ActorProxy`) and call the service/actor method again.</span></span>
* <span data-ttu-id="1ae68-443">**有状态服务**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-443">**Stateful service**.</span></span> <span data-ttu-id="1ae68-444">将针对可靠集合的操作包装在一个事务中。</span><span class="sxs-lookup"><span data-stu-id="1ae68-444">Wrap operations on reliable collections in a transaction.</span></span> <span data-ttu-id="1ae68-445">如果出错，将回滚该事务。</span><span class="sxs-lookup"><span data-stu-id="1ae68-445">If there is an error, the transaction will be rolled back.</span></span> <span data-ttu-id="1ae68-446">将重新处理该请求（如果已从队列中拉取）。</span><span class="sxs-lookup"><span data-stu-id="1ae68-446">The request, if pulled from a queue, will be processed again.</span></span>
* <span data-ttu-id="1ae68-447">**无状态服务**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-447">**Stateless service**.</span></span> <span data-ttu-id="1ae68-448">如果服务将数据暂留在外部存储中，则所有操作必须幂等。</span><span class="sxs-lookup"><span data-stu-id="1ae68-448">If the service persists data to an external store, all operations need to be idempotent.</span></span>

<span data-ttu-id="1ae68-449">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-449">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-450">应用程序日志</span><span class="sxs-lookup"><span data-stu-id="1ae68-450">Application log</span></span>

### <a name="service-fabric-node-is-shut-down"></a><span data-ttu-id="1ae68-451">Service Fabric 节点关闭。</span><span class="sxs-lookup"><span data-stu-id="1ae68-451">Service Fabric node is shut down.</span></span>
<span data-ttu-id="1ae68-452">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-452">**Detection**.</span></span> <span data-ttu-id="1ae68-453">向服务的 `RunAsync` 方法传递一个取消标记。</span><span class="sxs-lookup"><span data-stu-id="1ae68-453">A cancellation token is passed to the service's `RunAsync` method.</span></span> <span data-ttu-id="1ae68-454">Service Fabric 会在关闭节点前取消任务。</span><span class="sxs-lookup"><span data-stu-id="1ae68-454">Service Fabric cancels the task before shutting down the node.</span></span>

<span data-ttu-id="1ae68-455">**恢复**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-455">**Recovery**.</span></span> <span data-ttu-id="1ae68-456">使用该取消标记来检测关闭。</span><span class="sxs-lookup"><span data-stu-id="1ae68-456">Use the cancellation token to detect shutdown.</span></span> <span data-ttu-id="1ae68-457">当 Service Fabric 请求取消时，请完成所有工作并尽快退出  `RunAsync`。</span><span class="sxs-lookup"><span data-stu-id="1ae68-457">When Service Fabric requests cancellation, finish any work and exit `RunAsync` as quickly as possible.</span></span>

<span data-ttu-id="1ae68-458">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-458">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-459">应用程序日志</span><span class="sxs-lookup"><span data-stu-id="1ae68-459">Application logs</span></span>

## <a name="storage"></a><span data-ttu-id="1ae68-460">存储</span><span class="sxs-lookup"><span data-stu-id="1ae68-460">Storage</span></span>
### <a name="writing-data-to-azure-storage-fails"></a><span data-ttu-id="1ae68-461">向 Azure 存储写入数据失败</span><span class="sxs-lookup"><span data-stu-id="1ae68-461">Writing data to Azure Storage fails</span></span>
<span data-ttu-id="1ae68-462">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-462">**Detection**.</span></span> <span data-ttu-id="1ae68-463">写入时，客户端收到错误。</span><span class="sxs-lookup"><span data-stu-id="1ae68-463">The client receives errors when writing.</span></span>

<span data-ttu-id="1ae68-464">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-464">**Recovery**</span></span>

1. <span data-ttu-id="1ae68-465">重试该操作，以从暂时性故障中恢复。</span><span class="sxs-lookup"><span data-stu-id="1ae68-465">Retry the operation, to recover from transient failures.</span></span> <span data-ttu-id="1ae68-466">客户端 SDK 中的[重试策略][Storage.RetryPolicies]会自动处理此任务。</span><span class="sxs-lookup"><span data-stu-id="1ae68-466">The [retry policy][Storage.RetryPolicies] in the client SDK handles this automatically.</span></span>
2. <span data-ttu-id="1ae68-467">实施断路器模式，以免存储瘫痪。</span><span class="sxs-lookup"><span data-stu-id="1ae68-467">Implement the Circuit Breaker pattern to avoid overwhelming storage.</span></span>
3. <span data-ttu-id="1ae68-468">如果尝试 N 次重试均失败，则执行正常回退。</span><span class="sxs-lookup"><span data-stu-id="1ae68-468">If N retry attempts fail, perform a graceful fallback.</span></span> <span data-ttu-id="1ae68-469">例如：</span><span class="sxs-lookup"><span data-stu-id="1ae68-469">For example:</span></span>

   * <span data-ttu-id="1ae68-470">将数据存储在本地缓存中，等服务变为可用后，再将写入数据转发到存储。</span><span class="sxs-lookup"><span data-stu-id="1ae68-470">Store the data in a local cache, and forward the writes to storage later, when the service becomes available.</span></span>
   * <span data-ttu-id="1ae68-471">如果写入操作在事务范围内，则补偿事务。</span><span class="sxs-lookup"><span data-stu-id="1ae68-471">If the write action was in a transactional scope, compensate the transaction.</span></span>

<span data-ttu-id="1ae68-472">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-472">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-473">使用[存储度量值][storage-metrics]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-473">Use [storage metrics][storage-metrics].</span></span>

### <a name="reading-data-from-azure-storage-fails"></a><span data-ttu-id="1ae68-474">从 Azure 存储读取数据失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-474">Reading data from Azure Storage fails.</span></span>
<span data-ttu-id="1ae68-475">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-475">**Detection**.</span></span> <span data-ttu-id="1ae68-476">读取时，客户端收到错误。</span><span class="sxs-lookup"><span data-stu-id="1ae68-476">The client receives errors when reading.</span></span>

<span data-ttu-id="1ae68-477">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-477">**Recovery**</span></span>

1. <span data-ttu-id="1ae68-478">重试该操作，以从暂时性故障中恢复。</span><span class="sxs-lookup"><span data-stu-id="1ae68-478">Retry the operation, to recover from transient failures.</span></span> <span data-ttu-id="1ae68-479">客户端 SDK 中的[重试策略][Storage.RetryPolicies]会自动处理此任务。</span><span class="sxs-lookup"><span data-stu-id="1ae68-479">The [retry policy][Storage.RetryPolicies] in the client SDK handles this automatically.</span></span>
2. <span data-ttu-id="1ae68-480">对于 RA-GRS 存储，如果从主终结点读取失败，则尝试从辅助终结点读取。</span><span class="sxs-lookup"><span data-stu-id="1ae68-480">For RA-GRS storage, if reading from the primary endpoint fails, try reading from the secondary endpoint.</span></span> <span data-ttu-id="1ae68-481">客户端 SDK 可以自动处理此任务。</span><span class="sxs-lookup"><span data-stu-id="1ae68-481">The client SDK can handle this automatically.</span></span> <span data-ttu-id="1ae68-482">请参阅 [Azure 存储复制][storage-replication]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-482">See [Azure Storage replication][storage-replication].</span></span>
3. <span data-ttu-id="1ae68-483">如果尝试 *N* 次重试均失败，则执行回退操作，以便正常降级。</span><span class="sxs-lookup"><span data-stu-id="1ae68-483">If *N* retry attempts fail, take a fallback action to degrade gracefully.</span></span> <span data-ttu-id="1ae68-484">例如，如果从存储中检索不到产品图像，则显示一般的占位符图像。</span><span class="sxs-lookup"><span data-stu-id="1ae68-484">For example, if a product image can't be retrieved from storage, show a generic placeholder image.</span></span>

<span data-ttu-id="1ae68-485">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-485">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-486">使用[存储度量值][storage-metrics]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-486">Use [storage metrics][storage-metrics].</span></span>

## <a name="virtual-machine"></a><span data-ttu-id="1ae68-487">虚拟机</span><span class="sxs-lookup"><span data-stu-id="1ae68-487">Virtual Machine</span></span>
### <a name="connection-to-a-backend-vm-fails"></a><span data-ttu-id="1ae68-488">连接到后端 VM 失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-488">Connection to a backend VM fails.</span></span>
<span data-ttu-id="1ae68-489">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-489">**Detection**.</span></span> <span data-ttu-id="1ae68-490">网络连接错误。</span><span class="sxs-lookup"><span data-stu-id="1ae68-490">Network connection errors.</span></span>

<span data-ttu-id="1ae68-491">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-491">**Recovery**</span></span>

* <span data-ttu-id="1ae68-492">将至少两个后端 VM 部署在可用性集中，并放在负载均衡器后面。</span><span class="sxs-lookup"><span data-stu-id="1ae68-492">Deploy at least two backend VMs in an availability set, behind a load balancer.</span></span>
* <span data-ttu-id="1ae68-493">如果连接错误是暂时性的，TCP 有时会成功重试发送消息。</span><span class="sxs-lookup"><span data-stu-id="1ae68-493">If the connection error is transient, sometimes TCP will successfully retry sending the message.</span></span>
* <span data-ttu-id="1ae68-494">在应用程序中实施重试策略。</span><span class="sxs-lookup"><span data-stu-id="1ae68-494">Implement a retry policy in the application.</span></span>
* <span data-ttu-id="1ae68-495">对于持久性或非暂时性错误，实施[断路器][circuit-breaker]模式。</span><span class="sxs-lookup"><span data-stu-id="1ae68-495">For persistent or non-transient errors, implement the [Circuit Breaker][circuit-breaker] pattern.</span></span>
* <span data-ttu-id="1ae68-496">如果调用 VM 超过其网络出口限制，出站队列将填满。</span><span class="sxs-lookup"><span data-stu-id="1ae68-496">If the calling VM exceeds its network egress limit, the outbound queue will fill up.</span></span> <span data-ttu-id="1ae68-497">如果出站队列一直很满，请考虑横向扩展。</span><span class="sxs-lookup"><span data-stu-id="1ae68-497">If the outbound queue is consistently full, consider scaling out.</span></span>

<span data-ttu-id="1ae68-498">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-498">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-499">记录服务边界的事件。</span><span class="sxs-lookup"><span data-stu-id="1ae68-499">Log events at service boundaries.</span></span>

### <a name="vm-instance-becomes-unavailable-or-unhealthy"></a><span data-ttu-id="1ae68-500">VM 实例变得不可用或不正常。</span><span class="sxs-lookup"><span data-stu-id="1ae68-500">VM instance becomes unavailable or unhealthy.</span></span>
<span data-ttu-id="1ae68-501">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-501">**Detection**.</span></span> <span data-ttu-id="1ae68-502">配置负载均衡器[运行状况探测][lb-probe]，以便发信号指示 VM 实例是否正常。</span><span class="sxs-lookup"><span data-stu-id="1ae68-502">Configure a Load Balancer [health probe][lb-probe] that signals whether the VM instance is healthy.</span></span> <span data-ttu-id="1ae68-503">探测应检查关键功能能否正确响应。</span><span class="sxs-lookup"><span data-stu-id="1ae68-503">The probe should check whether critical functions are responding correctly.</span></span>

<span data-ttu-id="1ae68-504">**恢复**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-504">**Recovery**.</span></span> <span data-ttu-id="1ae68-505">对于每个应用程序层，将多个 VM 实例放入同一可用性集中，并在 VM 前面放一个负载均衡器。</span><span class="sxs-lookup"><span data-stu-id="1ae68-505">For each application tier, put multiple VM instances into the same availability set, and place a load balancer in front of the VMs.</span></span> <span data-ttu-id="1ae68-506">如果运行状况探测失败，负载均衡器会停止向不正常的实例发送新连接。</span><span class="sxs-lookup"><span data-stu-id="1ae68-506">If the health probe fails, the Load Balancer stops sending new connections to the unhealthy instance.</span></span>

<span data-ttu-id="1ae68-507">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-507">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-508">- 使用负载均衡器 [Log Analytics][lb-monitor]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-508">- Use Load Balancer [log analytics][lb-monitor].</span></span>

* <span data-ttu-id="1ae68-509">配置监视系统，以监视所有运行状况监视终结点。</span><span class="sxs-lookup"><span data-stu-id="1ae68-509">Configure your monitoring system to monitor all of the health monitoring endpoints.</span></span>

### <a name="operator-accidentally-shuts-down-a-vm"></a><span data-ttu-id="1ae68-510">操作员意外关闭 VM。</span><span class="sxs-lookup"><span data-stu-id="1ae68-510">Operator accidentally shuts down a VM.</span></span>
<span data-ttu-id="1ae68-511">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-511">**Detection**.</span></span> <span data-ttu-id="1ae68-512">不适用</span><span class="sxs-lookup"><span data-stu-id="1ae68-512">N/A</span></span>

<span data-ttu-id="1ae68-513">**恢复**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-513">**Recovery**.</span></span> <span data-ttu-id="1ae68-514">设置 `ReadOnly` 级别的资源锁。</span><span class="sxs-lookup"><span data-stu-id="1ae68-514">Set a resource lock with `ReadOnly` level.</span></span> <span data-ttu-id="1ae68-515">请参阅[使用 Azure 资源管理器锁定资源][rm-locks]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-515">See [Lock resources with Azure Resource Manager][rm-locks].</span></span>

<span data-ttu-id="1ae68-516">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-516">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-517">使用 [Azure 活动日志][azure-activity-logs]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-517">Use [Azure Activity Logs][azure-activity-logs].</span></span>

## <a name="webjobs"></a><span data-ttu-id="1ae68-518">Web 作业</span><span class="sxs-lookup"><span data-stu-id="1ae68-518">WebJobs</span></span>
### <a name="continuous-job-stops-running-when-the-scm-host-is-idle"></a><span data-ttu-id="1ae68-519">SCM 主机闲置时，连续作业停止运行。</span><span class="sxs-lookup"><span data-stu-id="1ae68-519">Continuous job stops running when the SCM host is idle.</span></span>
<span data-ttu-id="1ae68-520">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-520">**Detection**.</span></span> <span data-ttu-id="1ae68-521">向 WebJob 函数传递一个取消标记。</span><span class="sxs-lookup"><span data-stu-id="1ae68-521">Pass a cancellation token to the WebJob function.</span></span> <span data-ttu-id="1ae68-522">有关详细信息，请参阅[正常关闭][web-jobs-shutdown]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-522">For more information, see [Graceful shutdown][web-jobs-shutdown].</span></span>

<span data-ttu-id="1ae68-523">**恢复**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-523">**Recovery**.</span></span> <span data-ttu-id="1ae68-524">在 Web 应用中启用 `Always On` 设置。</span><span class="sxs-lookup"><span data-stu-id="1ae68-524">Enable the `Always On` setting in the web app.</span></span> <span data-ttu-id="1ae68-525">有关详细信息，请参阅[使用 WebJobs 运行后台任务][web-jobs]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-525">For more information, see [Run Background tasks with WebJobs][web-jobs].</span></span>

## <a name="application-design"></a><span data-ttu-id="1ae68-526">应用程序设计</span><span class="sxs-lookup"><span data-stu-id="1ae68-526">Application design</span></span>
### <a name="application-cant-handle-a-spike-in-incoming-requests"></a><span data-ttu-id="1ae68-527">应用程序无法处理陡增的传入请求。</span><span class="sxs-lookup"><span data-stu-id="1ae68-527">Application can't handle a spike in incoming requests.</span></span>
<span data-ttu-id="1ae68-528">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-528">**Detection**.</span></span> <span data-ttu-id="1ae68-529">取决于应用程序。</span><span class="sxs-lookup"><span data-stu-id="1ae68-529">Depends on the application.</span></span> <span data-ttu-id="1ae68-530">典型症状：</span><span class="sxs-lookup"><span data-stu-id="1ae68-530">Typical symptoms:</span></span>

* <span data-ttu-id="1ae68-531">网站开始返回 HTTP 5xx 错误代码。</span><span class="sxs-lookup"><span data-stu-id="1ae68-531">The website starts returning HTTP 5xx error codes.</span></span>
* <span data-ttu-id="1ae68-532">从属服务（比如数据库或存储）开始限制请求。</span><span class="sxs-lookup"><span data-stu-id="1ae68-532">Dependent services, such as database or storage, start to throttle requests.</span></span> <span data-ttu-id="1ae68-533">请查找 HTTP 错误，比如 HTTP 429（请求过多），具体视服务而定。</span><span class="sxs-lookup"><span data-stu-id="1ae68-533">Look for HTTP errors such as HTTP 429 (Too Many Requests), depending on the service.</span></span>
* <span data-ttu-id="1ae68-534">HTTP 队列长度变长。</span><span class="sxs-lookup"><span data-stu-id="1ae68-534">HTTP queue length grows.</span></span>

<span data-ttu-id="1ae68-535">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-535">**Recovery**</span></span>

* <span data-ttu-id="1ae68-536">横向扩展以处理增加的负载。</span><span class="sxs-lookup"><span data-stu-id="1ae68-536">Scale out to handle increased load.</span></span>
* <span data-ttu-id="1ae68-537">缓解故障影响，避免让级联故障中断整个应用程序。</span><span class="sxs-lookup"><span data-stu-id="1ae68-537">Mitigate failures to avoid having cascading failures disrupt the entire application.</span></span> <span data-ttu-id="1ae68-538">缓解策略包括：</span><span class="sxs-lookup"><span data-stu-id="1ae68-538">Mitigation strategies include:</span></span>

  * <span data-ttu-id="1ae68-539">实施[限制模式][throttling-pattern]，以免后端系统瘫痪。</span><span class="sxs-lookup"><span data-stu-id="1ae68-539">Implement the [Throttling Pattern][throttling-pattern] to avoid overwhelming backend systems.</span></span>
  * <span data-ttu-id="1ae68-540">使用[基于队列的负载调节][queue-based-load-leveling]缓冲请求，并以适当的节奏处理它们。</span><span class="sxs-lookup"><span data-stu-id="1ae68-540">Use [queue-based load leveling][queue-based-load-leveling] to buffer requests and process them at an appropriate pace.</span></span>
  * <span data-ttu-id="1ae68-541">优先某些客户端。</span><span class="sxs-lookup"><span data-stu-id="1ae68-541">Prioritize certain clients.</span></span> <span data-ttu-id="1ae68-542">例如，如果应用程序具有免费层和付费层，则限制免费层的客户，而不是付费客户。</span><span class="sxs-lookup"><span data-stu-id="1ae68-542">For example, if the application has free and paid tiers, throttle customers on the free tier, but not paid customers.</span></span> <span data-ttu-id="1ae68-543">请参阅[优先级队列模式][priority-queue-pattern]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-543">See [Priority queue pattern][priority-queue-pattern].</span></span>

<span data-ttu-id="1ae68-544">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-544">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-545">使用[应用服务诊断日志记录][app-service-logging]。</span><span class="sxs-lookup"><span data-stu-id="1ae68-545">Use [App Service diagnostic logging][app-service-logging].</span></span> <span data-ttu-id="1ae68-546">使用 [Azure Log Analytics][azure-log-analytics]、[Application Insights][app-insights] 或 [New Relic][new-relic] 等服务帮助了解诊断日志。</span><span class="sxs-lookup"><span data-stu-id="1ae68-546">Use a service such as [Azure Log Analytics][azure-log-analytics], [Application Insights][app-insights], or [New Relic][new-relic] to help understand the diagnostic logs.</span></span>

### <a name="one-of-the-operations-in-a-workflow-or-distributed-transaction-fails"></a><span data-ttu-id="1ae68-547">工作流或分布式事务中的某个操作失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-547">One of the operations in a workflow or distributed transaction fails.</span></span>
<span data-ttu-id="1ae68-548">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-548">**Detection**.</span></span> <span data-ttu-id="1ae68-549">尝试 *N* 次重试后仍然失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-549">After *N* retry attempts, it still fails.</span></span>

<span data-ttu-id="1ae68-550">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-550">**Recovery**</span></span>

* <span data-ttu-id="1ae68-551">作为缓解计划，可实施[计划程序代理监督程序][scheduler-agent-supervisor]模式来管理整个工作流。</span><span class="sxs-lookup"><span data-stu-id="1ae68-551">As a mitigation plan, implement the [Scheduler Agent Supervisor][scheduler-agent-supervisor] pattern to manage the entire workflow.</span></span>
* <span data-ttu-id="1ae68-552">对于超时错误，不要重试。</span><span class="sxs-lookup"><span data-stu-id="1ae68-552">Don't retry on timeouts.</span></span> <span data-ttu-id="1ae68-553">此错误的成功率较低。</span><span class="sxs-lookup"><span data-stu-id="1ae68-553">There is a low success rate for this error.</span></span>
* <span data-ttu-id="1ae68-554">将工作排队，以便稍后重试。</span><span class="sxs-lookup"><span data-stu-id="1ae68-554">Queue work, in order to retry later.</span></span>

<span data-ttu-id="1ae68-555">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-555">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-556">记录所有操作（成功的和失败的），包括补偿操作。</span><span class="sxs-lookup"><span data-stu-id="1ae68-556">Log all operations (successful and failed), including compensating actions.</span></span> <span data-ttu-id="1ae68-557">使用关联 ID，以便跟踪同一事务内的所有操作。</span><span class="sxs-lookup"><span data-stu-id="1ae68-557">Use correlation IDs, so that you can track all operations within the same transaction.</span></span>

### <a name="a-call-to-a-remote-service-fails"></a><span data-ttu-id="1ae68-558">对远程服务的调用失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-558">A call to a remote service fails.</span></span>
<span data-ttu-id="1ae68-559">**检测**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-559">**Detection**.</span></span> <span data-ttu-id="1ae68-560">HTTP 错误代码。</span><span class="sxs-lookup"><span data-stu-id="1ae68-560">HTTP error code.</span></span>

<span data-ttu-id="1ae68-561">**恢复**</span><span class="sxs-lookup"><span data-stu-id="1ae68-561">**Recovery**</span></span>

1. <span data-ttu-id="1ae68-562">对暂时性故障进行重试。</span><span class="sxs-lookup"><span data-stu-id="1ae68-562">Retry on transient failures.</span></span>
2. <span data-ttu-id="1ae68-563">如果尝试 *N* 次后，调用仍失败，则执行回退操作。</span><span class="sxs-lookup"><span data-stu-id="1ae68-563">If the call fails after *N* attempts, take a fallback action.</span></span> <span data-ttu-id="1ae68-564">（特定于应用程序。）</span><span class="sxs-lookup"><span data-stu-id="1ae68-564">(Application specific.)</span></span>
3. <span data-ttu-id="1ae68-565">实施[断路器模式][circuit-breaker]以免出现级联故障。</span><span class="sxs-lookup"><span data-stu-id="1ae68-565">Implement the [Circuit Breaker pattern][circuit-breaker] to avoid cascading failures.</span></span>

<span data-ttu-id="1ae68-566">**诊断**。</span><span class="sxs-lookup"><span data-stu-id="1ae68-566">**Diagnostics**.</span></span> <span data-ttu-id="1ae68-567">记录所有远程调用失败。</span><span class="sxs-lookup"><span data-stu-id="1ae68-567">Log all remote call failures.</span></span>

## <a name="next-steps"></a><span data-ttu-id="1ae68-568">后续步骤</span><span class="sxs-lookup"><span data-stu-id="1ae68-568">Next steps</span></span>
<span data-ttu-id="1ae68-569">有关 FMA 过程的详细信息，请参阅[专为云服务设计的复原能力][resilience-by-design-pdf]（PDF 下载）。</span><span class="sxs-lookup"><span data-stu-id="1ae68-569">For more information about the FMA process, see [Resilience by design for cloud services][resilience-by-design-pdf] (PDF download).</span></span>

<!-- links -->

[api-management]: https://azure.microsoft.com/documentation/services/api-management/
[api-management-throttling]: /azure/api-management/api-management-sample-flexible-throttling/
[app-insights]: /azure/application-insights/app-insights-overview/
[app-insights-web-apps]: /azure/application-insights/app-insights-azure-web-apps/
[app-service-configure]: /azure/app-service-web/web-sites-configure/
[app-service-logging]: /azure/app-service-web/web-sites-enable-diagnostic-log/
[app-service-slots]: /azure/app-service-web/web-sites-staged-publishing/
[auto-rest-client-retry]: https://github.com/Azure/autorest/tree/master/docs
[azure-activity-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-activity-logs/
[azure-alerts]: /azure/monitoring-and-diagnostics/insights-alerts-portal/
[azure-log-analytics]: /azure/log-analytics/log-analytics-overview/
[BrokeredMessage.TimeToLive]: https://msdn.microsoft.com/library/microsoft.servicebus.messaging.brokeredmessage.timetolive.aspx
[cassandra-error-handling]: http://www.datastax.com/dev/blog/cassandra-error-handling-done-right
[circuit-breaker]: https://msdn.microsoft.com/library/dn589784.aspx
[docdb-multi-region]: /azure/documentdb/documentdb-developing-with-multiple-regions/
[elasticsearch-azure]: ../elasticsearch/index.md
[elasticsearch-client]: https://www.elastic.co/guide/en/elasticsearch/client/index.html
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[onstop-events]: https://azure.microsoft.com/blog/the-right-way-to-handle-azure-onstop-events/
[lb-monitor]: /azure/load-balancer/load-balancer-monitor-log/
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview/#learn-about-the-types-of-probes
[new-relic]: https://newrelic.com/
[priority-queue-pattern]: https://msdn.microsoft.com/library/dn589794.aspx
[queue-based-load-leveling]: https://msdn.microsoft.com/library/dn589783.aspx
[QuotaExceededException]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.quotaexceededexception.aspx
[ra-web-apps-basic]: ../reference-architectures/app-service-web-app/basic-web-app.md
[redis-monitor]: /azure/redis-cache/cache-how-to-monitor/
[redis-retry]: ../best-practices/retry-service-specific.md#azure-redis-cache-retry-guidelines
[resilience-by-design-pdf]: http://download.microsoft.com/download/D/8/C/D8C599A4-4E8A-49BF-80EE-FE35F49B914D/Resilience_by_Design_for_Cloud_Services_White_Paper.pdf
[RoleEntryPoint.OnStop]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.onstop.aspx
[RoleEnvironment.Stopping]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.stopping.aspx
[rm-locks]: /azure/azure-resource-manager/resource-group-lock-resources/
[sb-dead-letter-queue]: /azure/service-bus-messaging/service-bus-dead-letter-queues/
[sb-georeplication-sample]: https://github.com/Azure-Samples/azure-servicebus-messaging-samples/tree/master/GeoReplication
[sb-messagingexception-class]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.messagingexception.aspx
[sb-messaging-exceptions]: /azure/service-bus-messaging/service-bus-messaging-exceptions/
[sb-outages]: /azure/service-bus-messaging/service-bus-outages-disasters/#protecting-queues-and-topics-against-datacenter-outages-or-disasters
[sb-partition]: /azure/service-bus-messaging/service-bus-partitioning/
[sb-poison-message]: /azure/app-service-web/websites-dotnet-webjobs-sdk-storage-queues-how-to/#poison
[sb-retry]: ../best-practices/retry-service-specific.md#service-bus-retry-guidelines
[search-sdk]: https://msdn.microsoft.com/library/dn951165.aspx
[scheduler-agent-supervisor]: https://msdn.microsoft.com/library/dn589780.aspx
[search-analytics]: /azure/search/search-traffic-analytics/
[sql-db-audit]: /azure/sql-database/sql-database-auditing-get-started/
[sql-db-errors]: /azure/sql-database/sql-database-develop-error-messages/#resource-governance-errors
[sql-db-failover]: /azure/sql-database/sql-database-geo-replication-failover-portal/
[sql-db-limits]: /azure/sql-database/sql-database-resource-limits/
[sql-db-replication]: /azure/sql-database/sql-database-geo-replication-overview/
[storage-metrics]: https://msdn.microsoft.com/library/dn782843.aspx
[storage-replication]: /azure/storage/storage-redundancy/
[Storage.RetryPolicies]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.aspx
[sys.event_log]: https://msdn.microsoft.com/library/dn270018.aspx
[throttling-pattern]: https://msdn.microsoft.com/library/dn589798.aspx
[web-jobs]: /azure/app-service-web/web-sites-create-web-jobs/
[web-jobs-shutdown]: /azure/app-service-web/websites-dotnet-webjobs-sdk-storage-queues-how-to/#graceful
