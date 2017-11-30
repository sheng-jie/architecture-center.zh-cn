---
title: "保护多租户应用程序中的后端 Web API"
description: "如何保护后端 Web API"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: authorize
pnp.series.next: token-cache
ms.openlocfilehash: 65529280c5849e36ed7ff23de08a0b485034d0d8
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="secure-a-backend-web-api"></a><span data-ttu-id="69c1c-103">保护后端 Web API</span><span class="sxs-lookup"><span data-stu-id="69c1c-103">Secure a backend web API</span></span>

<span data-ttu-id="69c1c-104">[![GitHub](../_images/github.png) 示例代码][sample application]</span><span class="sxs-lookup"><span data-stu-id="69c1c-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="69c1c-105">[Tailspin Surveys] 应用程序使用后端 Web API 来管理针对调查表执行的 CRUD 操作。</span><span class="sxs-lookup"><span data-stu-id="69c1c-105">The [Tailspin Surveys] application uses a backend web API to manage CRUD operations on surveys.</span></span> <span data-ttu-id="69c1c-106">例如，当用户单击“我的调查表”时，Web 应用程序会将 HTTP 请求发送到 Web API：</span><span class="sxs-lookup"><span data-stu-id="69c1c-106">For example, when a user clicks "My Surveys", the web application sends an HTTP request to the web API:</span></span>

```
GET /users/{userId}/surveys
```

<span data-ttu-id="69c1c-107">Web API 返回一个 JSON 对象：</span><span class="sxs-lookup"><span data-stu-id="69c1c-107">The web API returns a JSON object:</span></span>

```
{
  "Published":[],
  "Own":[
    {"Id":1,"Title":"Survey 1"},
    {"Id":3,"Title":"Survey 3"},
    ],
  "Contribute": [{"Id":8,"Title":"My survey"}]
}
```

<span data-ttu-id="69c1c-108">Web API 不允许匿名请求，因此，Web 应用必须使用 OAuth 2 持有者令牌对自身进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="69c1c-108">The web API does not allow anonymous requests, so the web app must authenticate itself using OAuth 2 bearer tokens.</span></span>

> [!NOTE]
> <span data-ttu-id="69c1c-109">这是一种服务器到服务器的方案。</span><span class="sxs-lookup"><span data-stu-id="69c1c-109">This is a server-to-server scenario.</span></span> <span data-ttu-id="69c1c-110">应用程序不会通过浏览器客户端对 API 发出任何 AJAX 调用。</span><span class="sxs-lookup"><span data-stu-id="69c1c-110">The application does not make any AJAX calls to the API from the browser client.</span></span>
> 
> 

<span data-ttu-id="69c1c-111">可以采用两种主要方法：</span><span class="sxs-lookup"><span data-stu-id="69c1c-111">There are two main approaches you can take:</span></span>

* <span data-ttu-id="69c1c-112">委托的用户标识。</span><span class="sxs-lookup"><span data-stu-id="69c1c-112">Delegated user identity.</span></span> <span data-ttu-id="69c1c-113">Web 应用程序使用用户标识进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="69c1c-113">The web application authenticates with the user's identity.</span></span>
* <span data-ttu-id="69c1c-114">应用程序标识。</span><span class="sxs-lookup"><span data-stu-id="69c1c-114">Application identity.</span></span> <span data-ttu-id="69c1c-115">Web 应用程序通过 OAuth2 客户端凭据流，使用其客户端 ID 进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="69c1c-115">The web application authenticates with its client ID, using OAuth2 client credential flow.</span></span>

<span data-ttu-id="69c1c-116">Tailspin 应用程序实施委托的用户标识。</span><span class="sxs-lookup"><span data-stu-id="69c1c-116">The Tailspin application implements delegated user identity.</span></span> <span data-ttu-id="69c1c-117">两种方法的主要区别是：</span><span class="sxs-lookup"><span data-stu-id="69c1c-117">Here are the main differences:</span></span>

<span data-ttu-id="69c1c-118">**委托的用户标识**</span><span class="sxs-lookup"><span data-stu-id="69c1c-118">**Delegated user identity**</span></span>

* <span data-ttu-id="69c1c-119">发送到 Web API 的持有者令牌包含用户标识。</span><span class="sxs-lookup"><span data-stu-id="69c1c-119">The bearer token sent to the web API contains the user identity.</span></span>
* <span data-ttu-id="69c1c-120">Web API 基于用户标识做出授权决策。</span><span class="sxs-lookup"><span data-stu-id="69c1c-120">The web API makes authorization decisions based on the user identity.</span></span>
* <span data-ttu-id="69c1c-121">如果用户无权执行操作，Web 应用程序需要处理来自 Web API 的 403（禁止访问）错误。</span><span class="sxs-lookup"><span data-stu-id="69c1c-121">The web application needs to handle 403 (Forbidden) errors from the web API, if the user is not authorized to perform an action.</span></span>
* <span data-ttu-id="69c1c-122">通常，Web 应用程序仍会做出一些影响 UI 的授权决策（例如，显示或隐藏 UI 元素）。</span><span class="sxs-lookup"><span data-stu-id="69c1c-122">Typically, the web application still makes some authorization decisions that affect UI, such as showing or hiding UI elements).</span></span>
* <span data-ttu-id="69c1c-123">Web API 可能会被不受信任的客户端（例如 JavaScript 应用程序或本机客户端应用程序）使用。</span><span class="sxs-lookup"><span data-stu-id="69c1c-123">The web API can potentially be used by untrusted clients, such as a JavaScript application or a native client application.</span></span>

<span data-ttu-id="69c1c-124">**应用程序标识**</span><span class="sxs-lookup"><span data-stu-id="69c1c-124">**Application identity**</span></span>

* <span data-ttu-id="69c1c-125">Web API 不会获取有关用户的信息。</span><span class="sxs-lookup"><span data-stu-id="69c1c-125">The web API does not get information about the user.</span></span>
* <span data-ttu-id="69c1c-126">Web API 无法基于用户标识执行任何授权。</span><span class="sxs-lookup"><span data-stu-id="69c1c-126">The web API cannot perform any authorization based on the user identity.</span></span> <span data-ttu-id="69c1c-127">所有授权决策由 Web 应用程序做出。</span><span class="sxs-lookup"><span data-stu-id="69c1c-127">All authorization decisions are made by the web application.</span></span>  
* <span data-ttu-id="69c1c-128">不受信任的客户端（JavaScript 或本机客户端应用程序）无法使用 Web API。</span><span class="sxs-lookup"><span data-stu-id="69c1c-128">The web API cannot be used by an untrusted client (JavaScript or native client application).</span></span>
* <span data-ttu-id="69c1c-129">此方法在一定程度上更容易实现，因为 Web API 中不存在授权逻辑。</span><span class="sxs-lookup"><span data-stu-id="69c1c-129">This approach may be somewhat simpler to implement, because there is no authorization logic in the Web API.</span></span>

<span data-ttu-id="69c1c-130">不管使用哪种方法，Web 应用程序都必须获取访问令牌，也就是调用 Web API 时所需的凭据。</span><span class="sxs-lookup"><span data-stu-id="69c1c-130">In either approach, the web application must get an access token, which is the credential needed to call the web API.</span></span>

* <span data-ttu-id="69c1c-131">使用委托的用户标识时，令牌必须来自可代表用户颁发令牌的 IDP。</span><span class="sxs-lookup"><span data-stu-id="69c1c-131">For delegated user identity, the token has to come from the IDP, which can issue a token on behalf of the user.</span></span>
* <span data-ttu-id="69c1c-132">对于客户端凭据，应用程序可以从 IDP 获取令牌，或者托管其自身的令牌服务器。</span><span class="sxs-lookup"><span data-stu-id="69c1c-132">For client credentials, an application might get the token from the IDP or host its own token server.</span></span> <span data-ttu-id="69c1c-133">（但是，不需要从头开始编写令牌服务器；使用类似于 [IdentityServer3] 的经全面测试的框架即可。）如果通过 Azure AD 进行身份验证，则即使使用了客户端凭据流，我们也强烈建议从 Azure AD 获取访问令牌。</span><span class="sxs-lookup"><span data-stu-id="69c1c-133">(But don't write a token server from scratch; use a well-tested framework like [IdentityServer3].) If you authenticate with Azure AD, it's strongly recommended to get the access token from Azure AD, even with client credential flow.</span></span>

<span data-ttu-id="69c1c-134">本文的余下部分假设应用程序使用 Azure AD 进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="69c1c-134">The rest of this article assumes the application is authenticating with Azure AD.</span></span>

![获取访问令牌](./images/access-token.png)

## <a name="register-the-web-api-in-azure-ad"></a><span data-ttu-id="69c1c-136">在 Azure AD 中注册 Web API</span><span class="sxs-lookup"><span data-stu-id="69c1c-136">Register the web API in Azure AD</span></span>
<span data-ttu-id="69c1c-137">要使 Azure AD 颁发 Web API 的持有者令牌，需要在 Azure AD 中配置一些设置。</span><span class="sxs-lookup"><span data-stu-id="69c1c-137">In order for Azure AD to issue a bearer token for the web API, you need to configure some things in Azure AD.</span></span>

1. <span data-ttu-id="69c1c-138">在 Azure AD 中注册 Web API。</span><span class="sxs-lookup"><span data-stu-id="69c1c-138">Register the web API in Azure AD.</span></span>

2. <span data-ttu-id="69c1c-139">将 Web 应用的客户端 ID 添加到 Web API 应用程序清单中的 `knownClientApplications` 属性。</span><span class="sxs-lookup"><span data-stu-id="69c1c-139">Add the client ID of the web app to the web API application manifest, in the `knownClientApplications` property.</span></span> <span data-ttu-id="69c1c-140">请参阅[更新应用程序清单]。</span><span class="sxs-lookup"><span data-stu-id="69c1c-140">See [Update the application manifests].</span></span>

3. <span data-ttu-id="69c1c-141">授予 Web 应用程序调用 Web API 的权限。</span><span class="sxs-lookup"><span data-stu-id="69c1c-141">Give the web application permission to call the web API.</span></span> <span data-ttu-id="69c1c-142">在 Azure 管理门户中，可以设置两种类型的权限：针对应用程序标识（客户端凭据流）的“应用程序权限”，或针对委托用户标识的“委托的权限”。</span><span class="sxs-lookup"><span data-stu-id="69c1c-142">In the Azure Management Portal, you can set two types of permissions: "Application Permissions" for application identity (client credential flow), or "Delegated Permissions" for delegated user identity.</span></span>
   
   ![委托的权限](./images/delegated-permissions.png)

## <a name="getting-an-access-token"></a><span data-ttu-id="69c1c-144">获取访问令牌</span><span class="sxs-lookup"><span data-stu-id="69c1c-144">Getting an access token</span></span>
<span data-ttu-id="69c1c-145">在调用 Web API 之前，Web 应用程序会从 Azure AD 获取访问令牌。</span><span class="sxs-lookup"><span data-stu-id="69c1c-145">Before calling the web API, the web application gets an access token from Azure AD.</span></span> <span data-ttu-id="69c1c-146">在 .NET 应用程序，需使用[适用于 .NET 的 Azure AD 身份验证库 (ADAL)][ADAL]。</span><span class="sxs-lookup"><span data-stu-id="69c1c-146">In a .NET application, use the [Azure AD Authentication Library (ADAL) for .NET][ADAL].</span></span>

<span data-ttu-id="69c1c-147">在 OAuth 2 授权代码流中，应用程序会使用授权代码交换访问令牌。</span><span class="sxs-lookup"><span data-stu-id="69c1c-147">In the OAuth 2 authorization code flow, the application exchanges an authorization code for an access token.</span></span> <span data-ttu-id="69c1c-148">以下代码使用 ADAL 获取访问令牌。</span><span class="sxs-lookup"><span data-stu-id="69c1c-148">The following code uses ADAL to get the access token.</span></span> <span data-ttu-id="69c1c-149">此代码是在 `AuthorizationCodeReceived` 事件期间调用的。</span><span class="sxs-lookup"><span data-stu-id="69c1c-149">This code is called during the `AuthorizationCodeReceived` event.</span></span>

```csharp
// The OpenID Connect middleware sends this event when it gets the authorization code.   
public override async Task AuthorizationCodeReceived(AuthorizationCodeReceivedContext context)
{
    string authorizationCode = context.ProtocolMessage.Code;
    string authority = "https://login.microsoftonline.com/" + tenantID
    string resourceID = "https://tailspin.onmicrosoft.com/surveys.webapi" // App ID URI
    ClientCredential credential = new ClientCredential(clientId, clientSecret);

    AuthenticationContext authContext = new AuthenticationContext(authority, tokenCache);
    AuthenticationResult authResult = await authContext.AcquireTokenByAuthorizationCodeAsync(
        authorizationCode, new Uri(redirectUri), credential, resourceID);

    // If successful, the token is in authResult.AccessToken
}
```

<span data-ttu-id="69c1c-150">下面是需要的各个参数：</span><span class="sxs-lookup"><span data-stu-id="69c1c-150">Here are the various parameters that are needed:</span></span>

* <span data-ttu-id="69c1c-151">`authority`。</span><span class="sxs-lookup"><span data-stu-id="69c1c-151">`authority`.</span></span> <span data-ttu-id="69c1c-152">派生自登录用户的租户 ID。</span><span class="sxs-lookup"><span data-stu-id="69c1c-152">Derived from the tenant ID of the signed in user.</span></span> <span data-ttu-id="69c1c-153">（不是 SaaS 提供程序的租户 ID）</span><span class="sxs-lookup"><span data-stu-id="69c1c-153">(Not the tenant ID of the SaaS provider)</span></span>  
* <span data-ttu-id="69c1c-154">`authorizationCode`。</span><span class="sxs-lookup"><span data-stu-id="69c1c-154">`authorizationCode`.</span></span> <span data-ttu-id="69c1c-155">从 IDP 取回的身份验证代码。</span><span class="sxs-lookup"><span data-stu-id="69c1c-155">the auth code that you got back from the IDP.</span></span>
* <span data-ttu-id="69c1c-156">`clientId`。</span><span class="sxs-lookup"><span data-stu-id="69c1c-156">`clientId`.</span></span> <span data-ttu-id="69c1c-157">Web 应用程序的客户端 ID。</span><span class="sxs-lookup"><span data-stu-id="69c1c-157">The web application's client ID.</span></span>
* <span data-ttu-id="69c1c-158">`clientSecret`。</span><span class="sxs-lookup"><span data-stu-id="69c1c-158">`clientSecret`.</span></span> <span data-ttu-id="69c1c-159">Web 应用程序的客户端机密。</span><span class="sxs-lookup"><span data-stu-id="69c1c-159">The web application's client secret.</span></span>
* <span data-ttu-id="69c1c-160">`redirectUri`。</span><span class="sxs-lookup"><span data-stu-id="69c1c-160">`redirectUri`.</span></span> <span data-ttu-id="69c1c-161">为 OpenID Connect 设置的重定向 URI。</span><span class="sxs-lookup"><span data-stu-id="69c1c-161">The redirect URI that you set for OpenID connect.</span></span> <span data-ttu-id="69c1c-162">IDP 在此 URI 中使用令牌发出回调。</span><span class="sxs-lookup"><span data-stu-id="69c1c-162">This is where the IDP calls back with the token.</span></span>
* <span data-ttu-id="69c1c-163">`resourceID`。</span><span class="sxs-lookup"><span data-stu-id="69c1c-163">`resourceID`.</span></span> <span data-ttu-id="69c1c-164">Web API 的应用 ID URI，这是在 Azure AD 中注册 Web API 时创建的 URI</span><span class="sxs-lookup"><span data-stu-id="69c1c-164">The App ID URI of the web API, which you created when you registered the web API in Azure AD</span></span>
* <span data-ttu-id="69c1c-165">`tokenCache`。</span><span class="sxs-lookup"><span data-stu-id="69c1c-165">`tokenCache`.</span></span> <span data-ttu-id="69c1c-166">用于缓存访问令牌的对象。</span><span class="sxs-lookup"><span data-stu-id="69c1c-166">An object that caches the access tokens.</span></span> <span data-ttu-id="69c1c-167">请参阅[令牌缓存]。</span><span class="sxs-lookup"><span data-stu-id="69c1c-167">See [Token caching].</span></span>

<span data-ttu-id="69c1c-168">如果 `AcquireTokenByAuthorizationCodeAsync` 成功，则 ADAL 会缓存令牌。</span><span class="sxs-lookup"><span data-stu-id="69c1c-168">If `AcquireTokenByAuthorizationCodeAsync` succeeds, ADAL caches the token.</span></span> <span data-ttu-id="69c1c-169">以后，可以通过调用 AcquireTokenSilentAsync 从缓存中获取令牌：</span><span class="sxs-lookup"><span data-stu-id="69c1c-169">Later, you can get the token from the cache by calling AcquireTokenSilentAsync:</span></span>

```csharp
AuthenticationContext authContext = new AuthenticationContext(authority, tokenCache);
var result = await authContext.AcquireTokenSilentAsync(resourceID, credential, new UserIdentifier(userId, UserIdentifierType.UniqueId));
```

<span data-ttu-id="69c1c-170">其中，`userId` 是 `http://schemas.microsoft.com/identity/claims/objectidentifier` 声明中的用户对象 ID。</span><span class="sxs-lookup"><span data-stu-id="69c1c-170">where `userId` is the user's object ID, which is found in the `http://schemas.microsoft.com/identity/claims/objectidentifier` claim.</span></span>

## <a name="using-the-access-token-to-call-the-web-api"></a><span data-ttu-id="69c1c-171">使用访问令牌调用 Web API</span><span class="sxs-lookup"><span data-stu-id="69c1c-171">Using the access token to call the web API</span></span>
<span data-ttu-id="69c1c-172">获取令牌后，请在发往 Web API 的 HTTP 请求的授权标头中发送该令牌。</span><span class="sxs-lookup"><span data-stu-id="69c1c-172">Once you have the token, send it in the Authorization header of the HTTP requests to the web API.</span></span>

```
Authorization: Bearer xxxxxxxxxx
```

<span data-ttu-id="69c1c-173">Surveys 应用程序中的以下扩展方法使用 **HttpClient** 类在 HTTP 请求中设置授权标头。</span><span class="sxs-lookup"><span data-stu-id="69c1c-173">The following extension method from the Surveys application sets the Authorization header on an HTTP request, using the **HttpClient** class.</span></span>

```csharp
public static async Task<HttpResponseMessage> SendRequestWithBearerTokenAsync(this HttpClient httpClient, HttpMethod method, string path, object requestBody, string accessToken, CancellationToken ct)
{
    var request = new HttpRequestMessage(method, path);
    if (requestBody != null)
    {
        var json = JsonConvert.SerializeObject(requestBody, Formatting.None);
        var content = new StringContent(json, Encoding.UTF8, "application/json");
        request.Content = content;
    }

    request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
    request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));

    var response = await httpClient.SendAsync(request, ct);
    return response;
}
```

## <a name="authenticating-in-the-web-api"></a><span data-ttu-id="69c1c-174">在 Web API 中进行身份验证</span><span class="sxs-lookup"><span data-stu-id="69c1c-174">Authenticating in the web API</span></span>
<span data-ttu-id="69c1c-175">Web API 必须对持有者令牌进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="69c1c-175">The web API has to authenticate the bearer token.</span></span> <span data-ttu-id="69c1c-176">在 ASP.NET Core 中，可以使用 [Microsoft.AspNet.Authentication.JwtBearer][JwtBearer] 包。</span><span class="sxs-lookup"><span data-stu-id="69c1c-176">In ASP.NET Core, you can use the [Microsoft.AspNet.Authentication.JwtBearer][JwtBearer] package.</span></span> <span data-ttu-id="69c1c-177">此包提供可让应用程序接收 OpenID Connect 持有者令牌的中间件。</span><span class="sxs-lookup"><span data-stu-id="69c1c-177">This package provides middleware that enables the application to receive OpenID Connect bearer tokens.</span></span>

<span data-ttu-id="69c1c-178">在 Web API `Startup` 类中注册该中间件。</span><span class="sxs-lookup"><span data-stu-id="69c1c-178">Register the middleware in your web API `Startup` class.</span></span>

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ApplicationDbContext dbContext, ILoggerFactory loggerFactory)
{
    // ...

    app.UseJwtBearerAuthentication(new JwtBearerOptions {
        Audience = configOptions.AzureAd.WebApiResourceId,
        Authority = Constants.AuthEndpointPrefix,
        TokenValidationParameters = new TokenValidationParameters {
            ValidateIssuer = false
        },
        Events= new SurveysJwtBearerEvents(loggerFactory.CreateLogger<SurveysJwtBearerEvents>())
    });
    
    // ...
}
```

* <span data-ttu-id="69c1c-179">**Audience**。</span><span class="sxs-lookup"><span data-stu-id="69c1c-179">**Audience**.</span></span> <span data-ttu-id="69c1c-180">请将此类设置为 Web API 的应用 ID URL，即在 Azure AD 中注册 Web API 时创建的 URL。</span><span class="sxs-lookup"><span data-stu-id="69c1c-180">Set this to the App ID URL for the web API, which you created when you registered the web API with Azure AD.</span></span>
* <span data-ttu-id="69c1c-181">**Authority**。</span><span class="sxs-lookup"><span data-stu-id="69c1c-181">**Authority**.</span></span> <span data-ttu-id="69c1c-182">对于多租户应用程序，请将此类设置为 `https://login.microsoftonline.com/common/`。</span><span class="sxs-lookup"><span data-stu-id="69c1c-182">For a multitenant application, set this to `https://login.microsoftonline.com/common/`.</span></span>
* <span data-ttu-id="69c1c-183">**TokenValidationParameters**。</span><span class="sxs-lookup"><span data-stu-id="69c1c-183">**TokenValidationParameters**.</span></span> <span data-ttu-id="69c1c-184">对于多租户应用程序，请将 **ValidateIssuer** 设置为 false。</span><span class="sxs-lookup"><span data-stu-id="69c1c-184">For a multitenant application, set **ValidateIssuer** to false.</span></span> <span data-ttu-id="69c1c-185">这意味着应用程序将会验证颁发者。</span><span class="sxs-lookup"><span data-stu-id="69c1c-185">That means the application will validate the issuer.</span></span>
* <span data-ttu-id="69c1c-186">**Events** 是派生自 **JwtBearerEvents** 的类。</span><span class="sxs-lookup"><span data-stu-id="69c1c-186">**Events** is a class that derives from **JwtBearerEvents**.</span></span>

### <a name="issuer-validation"></a><span data-ttu-id="69c1c-187">颁发者验证</span><span class="sxs-lookup"><span data-stu-id="69c1c-187">Issuer validation</span></span>
<span data-ttu-id="69c1c-188">在 **JwtBearerEvents.TokenValidated** 事件中验证令牌颁发者。</span><span class="sxs-lookup"><span data-stu-id="69c1c-188">Validate the token issuer in the **JwtBearerEvents.TokenValidated** event.</span></span> <span data-ttu-id="69c1c-189">该颁发者在“iss”声明中发送。</span><span class="sxs-lookup"><span data-stu-id="69c1c-189">The issuer is sent in the "iss" claim.</span></span>

<span data-ttu-id="69c1c-190">在 Surveys 应用程序中，Web API 不会处理[租户注册]。</span><span class="sxs-lookup"><span data-stu-id="69c1c-190">In the Surveys application, the web API doesn't handle [tenant sign-up].</span></span> <span data-ttu-id="69c1c-191">因此，它只会检查应用程序数据库中是否已包含该颁发者。</span><span class="sxs-lookup"><span data-stu-id="69c1c-191">Therefore, it just checks if the issuer is already in the application database.</span></span> <span data-ttu-id="69c1c-192">如果不包含，则引发异常，从而导致身份验证失败。</span><span class="sxs-lookup"><span data-stu-id="69c1c-192">If not, it throws an exception, which causes authentication to fail.</span></span>

```csharp
public override async Task TokenValidated(TokenValidatedContext context)
{
    var principal = context.Ticket.Principal;
    var tenantManager = context.HttpContext.RequestServices.GetService<TenantManager>();
    var userManager = context.HttpContext.RequestServices.GetService<UserManager>();
    var issuerValue = principal.GetIssuerValue();
    var tenant = await tenantManager.FindByIssuerValueAsync(issuerValue);

    if (tenant == null)
    {
        // The caller was not from a trusted issuer. Throw to block the authentication flow.
        throw new SecurityTokenValidationException();
    }

    var identity = principal.Identities.First();

    // Add new claim for survey_userid
    var registeredUser = await userManager.FindByObjectIdentifier(principal.GetObjectIdentifierValue());
    identity.AddClaim(new Claim(SurveyClaimTypes.SurveyUserIdClaimType, registeredUser.Id.ToString()));
    identity.AddClaim(new Claim(SurveyClaimTypes.SurveyTenantIdClaimType, registeredUser.TenantId.ToString()));

    // Add new claim for Email
    var email = principal.FindFirst(ClaimTypes.Upn)?.Value;
    if (!string.IsNullOrWhiteSpace(email))
    {
        identity.AddClaim(new Claim(ClaimTypes.Email, email));
    }
}
```

<span data-ttu-id="69c1c-193">如此示例中所示，也可以使用 **TokenValidated** 事件修改声明。</span><span class="sxs-lookup"><span data-stu-id="69c1c-193">As this example shows, you can also use the **TokenValidated** event to modify the claims.</span></span> <span data-ttu-id="69c1c-194">请记住，这些声明直接来自 Azure AD。</span><span class="sxs-lookup"><span data-stu-id="69c1c-194">Remember that the claims come directly from Azure AD.</span></span> <span data-ttu-id="69c1c-195">如果 Web 应用程序修改了它所获取的声明，这些更改不会显示在 Web API 收到的持有者令牌中。</span><span class="sxs-lookup"><span data-stu-id="69c1c-195">If the web application modifies the claims that it gets, those changes won't show up in the bearer token that the web API receives.</span></span> <span data-ttu-id="69c1c-196">有关详细信息，请参阅[声明转换][claims-transformation]。</span><span class="sxs-lookup"><span data-stu-id="69c1c-196">For more information, see [Claims transformations][claims-transformation].</span></span>

## <a name="authorization"></a><span data-ttu-id="69c1c-197">授权</span><span class="sxs-lookup"><span data-stu-id="69c1c-197">Authorization</span></span>
<span data-ttu-id="69c1c-198">有关授权的一般性介绍，请参阅[基于角色和基于资源的授权][Authorization]。</span><span class="sxs-lookup"><span data-stu-id="69c1c-198">For a general discussion of authorization, see [Role-based and resource-based authorization][Authorization].</span></span> 

<span data-ttu-id="69c1c-199">JwtBearer 中间件处理授权响应。</span><span class="sxs-lookup"><span data-stu-id="69c1c-199">The JwtBearer middleware handles the authorization responses.</span></span> <span data-ttu-id="69c1c-200">例如，若要限制为只有经过身份验证的用户才能执行控制器操作，请使用 **[Authorize]** 属性，并指定 **JwtBearerDefaults.AuthenticationScheme** 作为身份验证方案：</span><span class="sxs-lookup"><span data-stu-id="69c1c-200">For example, to restrict a controller action to authenticated users, use the **[Authorize]** atrribute and specify **JwtBearerDefaults.AuthenticationScheme** as the authentication scheme:</span></span>

```csharp
[Authorize(ActiveAuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme)]
```

<span data-ttu-id="69c1c-201">如果用户未经身份验证，此代码会返回 401 状态代码。</span><span class="sxs-lookup"><span data-stu-id="69c1c-201">This returns a 401 status code if the user is not authenticated.</span></span>

<span data-ttu-id="69c1c-202">若要通过授权策略限制控制器操作，请在 **[Authorize]** 属性中指定策略名称：</span><span class="sxs-lookup"><span data-stu-id="69c1c-202">To restrict a controller action by authorizaton policy, specify the policy name in the **[Authorize]** attribute:</span></span>

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
```

<span data-ttu-id="69c1c-203">如果用户未经身份验证，此代码会返回 401 状态代码；如果用户已经过身份验证但未获授权，此代码会返回 403。</span><span class="sxs-lookup"><span data-stu-id="69c1c-203">This returns a 401 status code if the user is not authenticated, and 403 if the user is authenticated but not authorized.</span></span> <span data-ttu-id="69c1c-204">启动时注册策略：</span><span class="sxs-lookup"><span data-stu-id="69c1c-204">Register the policy on startup:</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthorization(options =>
    {
        options.AddPolicy(PolicyNames.RequireSurveyCreator,
            policy =>
            {
                policy.AddRequirements(new SurveyCreatorRequirement());
                policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement 
                policy.AddAuthenticationSchemes(JwtBearerDefaults.AuthenticationScheme);
            });
        options.AddPolicy(PolicyNames.RequireSurveyAdmin,
            policy =>
            {
                policy.AddRequirements(new SurveyAdminRequirement());
                policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement 
                policy.AddAuthenticationSchemes(JwtBearerDefaults.AuthenticationScheme);
            });
    });
    
    // ...
}
```

<span data-ttu-id="69c1c-205">[**下一篇**][token cache]</span><span class="sxs-lookup"><span data-stu-id="69c1c-205">[**Next**][token cache]</span></span>

<!-- links -->
[ADAL]: https://msdn.microsoft.com/library/azure/jj573266.aspx
[JwtBearer]: https://www.nuget.org/packages/Microsoft.AspNet.Authentication.JwtBearer

[Tailspin Surveys]: tailspin.md
[IdentityServer3]: https://github.com/IdentityServer/IdentityServer3
[更新应用程序清单]: ./run-the-app.md#update-the-application-manifests
[令牌缓存]: token-cache.md
[租户注册]: signup.md
[claims-transformation]: claims.md#claims-transformations
[Authorization]: authorize.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[token cache]: token-cache.md
