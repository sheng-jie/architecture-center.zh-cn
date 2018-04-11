---
title: 守护程序
description: 通过使用专用的主机实例保护应用程序和服务，该实例用于充当客户端和应用程序或服务之间的中转站、验证和整理请求，并在它们之间传递请求和数据。
keywords: 设计模式
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- security
ms.openlocfilehash: 39f8548bbccb5e19d433f65b2e7e09147d676996
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="gatekeeper-pattern"></a><span data-ttu-id="77054-104">守护程序模式</span><span class="sxs-lookup"><span data-stu-id="77054-104">Gatekeeper pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="77054-105">通过使用专用的主机实例保护应用程序和服务，该实例用于充当客户端和应用程序或服务之间的中转站、验证和整理请求，并在它们之间传递请求和数据。</span><span class="sxs-lookup"><span data-stu-id="77054-105">Protect applications and services by using a dedicated host instance that acts as a broker between clients and the application or service, validates and sanitizes requests, and passes requests and data between them.</span></span> <span data-ttu-id="77054-106">这可以额外提供一层安全，并限制系统的受攻击面。</span><span class="sxs-lookup"><span data-stu-id="77054-106">This can provide an additional layer of security, and limit the attack surface of the system.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="77054-107">上下文和问题</span><span class="sxs-lookup"><span data-stu-id="77054-107">Context and problem</span></span>

<span data-ttu-id="77054-108">应用程序通过接受和处理请求向客户端公开其功能。</span><span class="sxs-lookup"><span data-stu-id="77054-108">Applications expose their functionality to clients by accepting and processing requests.</span></span> <span data-ttu-id="77054-109">在云托管方案中，应用程序公开客户端连接，并通常包括代码以处理来自客户端的请求。</span><span class="sxs-lookup"><span data-stu-id="77054-109">In cloud-hosted scenarios, applications expose endpoints clients connect to, and typically include the code to handle the requests from clients.</span></span> <span data-ttu-id="77054-110">此代码执行身份验证和验证、部分或全部请求处理，并可能代表客户端访问存储和其他服务。</span><span class="sxs-lookup"><span data-stu-id="77054-110">This code performs authentication and validation, some or all request processing, and is likely to accesses storage and other services on behalf of the client.</span></span>

<span data-ttu-id="77054-111">如果恶意用户能够危害系统并访问应用程序的托管环境，那么它所使用的安全机制（例如凭据和存储密钥）以及它可访问的服务和数据访问都会暴露出来。</span><span class="sxs-lookup"><span data-stu-id="77054-111">If a malicious user is able to compromise the system and gain access to the application’s hosting environment, the security mechanisms it uses such as credentials and storage keys, and the services and data it accesses, are exposed.</span></span> <span data-ttu-id="77054-112">因此，恶意用户可以无限制地访问敏感信息和其他服务。</span><span class="sxs-lookup"><span data-stu-id="77054-112">As a result, the malicious user can gain unrestrained access to sensitive information and other services.</span></span>

## <a name="solution"></a><span data-ttu-id="77054-113">解决方案</span><span class="sxs-lookup"><span data-stu-id="77054-113">Solution</span></span>

<span data-ttu-id="77054-114">为了最大限度地减少客户端访问敏感信息和服务的风险，请将公共终结点的主机或任务与处理请求和访问存储的代码分离。</span><span class="sxs-lookup"><span data-stu-id="77054-114">To minimize the risk of clients gaining access to sensitive information and services, decouple hosts or tasks that expose public endpoints from the code that processes requests and accesses storage.</span></span> <span data-ttu-id="77054-115">可通过使用某个门面或专用任务完成此操作，该门面或专用任务与客户端交互，然后将请求&mdash;可能通过一个分离的接口&mdash;提交到将要处理该请求的主机或任务。</span><span class="sxs-lookup"><span data-stu-id="77054-115">You can achieve this by using a façade or a dedicated task that interacts with clients and then hands off the request&mdash;perhaps through a decoupled interface&mdash;to the hosts or tasks that'll handle the request.</span></span> <span data-ttu-id="77054-116">图提供了此模式的综合概述。</span><span class="sxs-lookup"><span data-stu-id="77054-116">The figure provides a high-level overview of this pattern.</span></span>

![此模式的综合概述](./_images/gatekeeper-diagram.png)


<span data-ttu-id="77054-118">守护程序模式可用于简单地保护存储，或者可用作更全面的机制来保护应用程序的所有功能。</span><span class="sxs-lookup"><span data-stu-id="77054-118">The gatekeeper pattern can be used to simply protect storage, or it can be used as a more comprehensive façade to protect all of the functions of the application.</span></span> <span data-ttu-id="77054-119">重要因素是：</span><span class="sxs-lookup"><span data-stu-id="77054-119">The important factors are:</span></span>

- <span data-ttu-id="77054-120">**受控的验证。**</span><span class="sxs-lookup"><span data-stu-id="77054-120">**Controlled validation.**</span></span> <span data-ttu-id="77054-121">守护程序验证所有请求，并拒绝不满足验证的要求。</span><span class="sxs-lookup"><span data-stu-id="77054-121">The gatekeeper validates all requests, and rejects those that don't meet validation requirements.</span></span>
- <span data-ttu-id="77054-122">**有限的风险和公开。**</span><span class="sxs-lookup"><span data-stu-id="77054-122">**Limited risk and exposure.**</span></span> <span data-ttu-id="77054-123">守护程序不能访问受信任主机访问存储和服务所用的凭据或密钥。</span><span class="sxs-lookup"><span data-stu-id="77054-123">The gatekeeper doesn't have access to the credentials or keys used by the trusted host to access storage and services.</span></span> <span data-ttu-id="77054-124">如果守护程序受到攻击，攻击者无权访问这些凭据或密钥。</span><span class="sxs-lookup"><span data-stu-id="77054-124">If the gatekeeper is compromised, the attacker doesn't get access to these credentials or keys.</span></span>
- <span data-ttu-id="77054-125">**适当的安全。**</span><span class="sxs-lookup"><span data-stu-id="77054-125">**Appropriate security.**</span></span> <span data-ttu-id="77054-126">守护程序在有限特权模式下运行，而其他应用程序在访问存储和服务所需的全信任模式下运行。</span><span class="sxs-lookup"><span data-stu-id="77054-126">The gatekeeper runs in a limited privilege mode, while the rest of the application runs in the full trust mode required to access storage and services.</span></span> <span data-ttu-id="77054-127">如果守护程序受到攻击，那么它不能直接访问应用程序服务或数据。</span><span class="sxs-lookup"><span data-stu-id="77054-127">If the gatekeeper is compromised, it can't directly access the application services or data.</span></span>

<span data-ttu-id="77054-128">此模式就像典型网络拓扑中防火墙。</span><span class="sxs-lookup"><span data-stu-id="77054-128">This pattern acts like a firewall in a typical network topography.</span></span> <span data-ttu-id="77054-129">它允许守护程序检查请求，并决定是否将请求传递到执行所需任务的受信任主机（有时称为 keymaster）。</span><span class="sxs-lookup"><span data-stu-id="77054-129">It allows the gatekeeper to examine requests and make a decision about whether to pass the request on to the trusted host (sometimes called the keymaster) that performs the required tasks.</span></span> <span data-ttu-id="77054-130">此决策通常需要守护程序验证和整理请求内容，然后再将它传递到受信任主机。</span><span class="sxs-lookup"><span data-stu-id="77054-130">This decision typically requires the gatekeeper to validate and sanitize the request content before passing it on to the trusted host.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="77054-131">问题和注意事项</span><span class="sxs-lookup"><span data-stu-id="77054-131">Issues and considerations</span></span>

<span data-ttu-id="77054-132">在决定如何实现此模式时，请考虑以下几点：</span><span class="sxs-lookup"><span data-stu-id="77054-132">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="77054-133">请确保接受守护程序传递请求的受信任主机只公开到内部或受保护的终结点，并只连接守护程序。</span><span class="sxs-lookup"><span data-stu-id="77054-133">Ensure that the trusted hosts the gatekeeper passes requests to expose only internal or protected endpoints, and connect only to the gatekeeper.</span></span> <span data-ttu-id="77054-134">受信任的主机不应公开任何外部终结点或接口。</span><span class="sxs-lookup"><span data-stu-id="77054-134">The trusted hosts shouldn't expose any external endpoints or interfaces.</span></span>
- <span data-ttu-id="77054-135">守护程序必须在有限特权模式下运行。</span><span class="sxs-lookup"><span data-stu-id="77054-135">The gatekeeper must run in a limited privilege mode.</span></span> <span data-ttu-id="77054-136">通常这意味着在分离托管的服务或虚拟机中运行守护程序和信任的主机。</span><span class="sxs-lookup"><span data-stu-id="77054-136">Typically this means running the gatekeeper and the trusted host in separate hosted services or virtual machines.</span></span>
- <span data-ttu-id="77054-137">守护程序不应执行与应用程序或服务有关的任何处理或访问任何数据。</span><span class="sxs-lookup"><span data-stu-id="77054-137">The gatekeeper shouldn't perform any processing related to the application or services, or access any data.</span></span> <span data-ttu-id="77054-138">其作用仅是用于验证并整理请求。</span><span class="sxs-lookup"><span data-stu-id="77054-138">Its function is purely to validate and sanitize requests.</span></span> <span data-ttu-id="77054-139">受信任的主机可能需要执行请求的其他验证，但应通过守护程序执行核心验证。</span><span class="sxs-lookup"><span data-stu-id="77054-139">The trusted hosts might need to perform additional validation of requests, but the core validation should be performed by the gatekeeper.</span></span>
- <span data-ttu-id="77054-140">在守护程序和受信任的主机或任务（在可能的情况下）之间使用安全通信通道（HTTPS、SSL或TLS）。</span><span class="sxs-lookup"><span data-stu-id="77054-140">Use a secure communication channel (HTTPS, SSL, or TLS) between the gatekeeper and the trusted hosts or tasks where this is possible.</span></span> <span data-ttu-id="77054-141">但是，某些宿主环境在内部终结点上不支持 HTTPS。</span><span class="sxs-lookup"><span data-stu-id="77054-141">However, some hosting environments don't support HTTPS on internal endpoints.</span></span>
- <span data-ttu-id="77054-142">为实现实现守护程序，需要将此附加层添加到应用程序，这可能会对性能造成影响（因为它需要额外的处理和网络通信）。</span><span class="sxs-lookup"><span data-stu-id="77054-142">Adding the extra layer to the application to implement the gatekeeper pattern is likely to have some impact on performance due to the additional processing and network communication it requires.</span></span>
- <span data-ttu-id="77054-143">守护程序实例可能是单点故障。</span><span class="sxs-lookup"><span data-stu-id="77054-143">The gatekeeper instance could be a single point of failure.</span></span> <span data-ttu-id="77054-144">若要尽量减少失败的影响，请考虑部署其他实例并使用自动缩放机制以确保维护可用性的能力。</span><span class="sxs-lookup"><span data-stu-id="77054-144">To minimize the impact of a failure, consider deploying additional instances and using an autoscaling mechanism to ensure capacity to maintain availability.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="77054-145">何时使用此模式</span><span class="sxs-lookup"><span data-stu-id="77054-145">When to use this pattern</span></span>

<span data-ttu-id="77054-146">此模式适合用于：</span><span class="sxs-lookup"><span data-stu-id="77054-146">This pattern is useful for:</span></span>

- <span data-ttu-id="77054-147">处理敏感信息、公开服务（此类服务必须具有很高级别保护以防止恶意攻击）或执行不应中断的任务关键型操作的应用程序。</span><span class="sxs-lookup"><span data-stu-id="77054-147">Applications that handle sensitive information, expose services that must have a high degree of protection from malicious attacks, or perform mission-critical operations that shouldn't be disrupted.</span></span>
- <span data-ttu-id="77054-148">分布式应用程序，它需要与主要任务分开执行请求验证，或集中该验证以简化维护和管理。</span><span class="sxs-lookup"><span data-stu-id="77054-148">Distributed applications where it's necessary to perform request validation separately from the main tasks, or to centralize this validation to simplify maintenance and administration.</span></span>

## <a name="example"></a><span data-ttu-id="77054-149">示例</span><span class="sxs-lookup"><span data-stu-id="77054-149">Example</span></span>

<span data-ttu-id="77054-150">在云托管方案中，可以通过将守护程序角色或虚拟机与应用程序中的受信任角色和服务分离来实现此模式。</span><span class="sxs-lookup"><span data-stu-id="77054-150">In a cloud-hosted scenario, this pattern can be implemented by decoupling the gatekeeper role or virtual machine from the trusted roles and services in an application.</span></span> <span data-ttu-id="77054-151">将内部终结点、队列或存储用作中间通信机制执行此操。</span><span class="sxs-lookup"><span data-stu-id="77054-151">Do this by using an internal endpoint, a queue, or storage as an intermediate communication mechanism.</span></span> <span data-ttu-id="77054-152">本图演示了如何使用内部终结点。</span><span class="sxs-lookup"><span data-stu-id="77054-152">The figure illustrates using an internal endpoint.</span></span>

![使用云服务 web 和辅助角色的模式示例](./_images/gatekeeper-endpoint.png)


## <a name="related-patterns"></a><span data-ttu-id="77054-154">相关模式</span><span class="sxs-lookup"><span data-stu-id="77054-154">Related patterns</span></span>

<span data-ttu-id="77054-155">当执行守护程序模式时，[Valet 密钥模式](valet-key.md)可能也相关。</span><span class="sxs-lookup"><span data-stu-id="77054-155">The [Valet Key pattern](valet-key.md) might also be relevant when implementing the Gatekeeper pattern.</span></span> <span data-ttu-id="77054-156">在守护程序和受信任的角色之间通信时，可通过限制访问资源权限的密钥或令牌增强安全。</span><span class="sxs-lookup"><span data-stu-id="77054-156">When communicating between the Gatekeeper and trusted roles it's good practice to enhance security by using keys or tokens that limit permissions for accessing resources.</span></span> <span data-ttu-id="77054-157">描述如何使用令牌或密钥让客户端以有限方式访问特定资源或服务。</span><span class="sxs-lookup"><span data-stu-id="77054-157">Describes how to use a token or key that provides clients with restricted direct access to a specific resource or service.</span></span>
