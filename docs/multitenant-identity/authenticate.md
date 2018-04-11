---
title: 多租户应用程序中的身份验证
description: 多租户应用程序如何从 Azure AD 验证用户身份
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: tailspin
pnp.series.next: claims
ms.openlocfilehash: e85817626675cec4d126921c19a31a0983ecd62d
ms.sourcegitcommit: 8ab30776e0c4cdc16ca0dcc881960e3108ad3e94
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/08/2017
---
# <a name="authenticate-using-azure-ad-and-openid-connect"></a><span data-ttu-id="69ce8-103">使用 Azure AD 和 OpenID Connect 进行身份验证</span><span class="sxs-lookup"><span data-stu-id="69ce8-103">Authenticate using Azure AD and OpenID Connect</span></span>

<span data-ttu-id="69ce8-104">[![GitHub](../_images/github.png) 示例代码][sample application]</span><span class="sxs-lookup"><span data-stu-id="69ce8-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="69ce8-105">Surveys 应用程序使用 OpenID Connect (OIDC) 协议向 Azure Active Directory (Azure AD) 验证用户身份。</span><span class="sxs-lookup"><span data-stu-id="69ce8-105">The Surveys application uses the OpenID Connect (OIDC) protocol to authenticate users with Azure Active Directory (Azure AD).</span></span> <span data-ttu-id="69ce8-106">Surveys 应用程序使用 ASP.NET Core，后者具有用于 OIDC 的内置中间件。</span><span class="sxs-lookup"><span data-stu-id="69ce8-106">The Surveys application uses ASP.NET Core, which has built-in middleware for OIDC.</span></span> <span data-ttu-id="69ce8-107">下图从较高层面展示了用户登录时发生的情况。</span><span class="sxs-lookup"><span data-stu-id="69ce8-107">The following diagram shows what happens when the user signs in, at a high level.</span></span>

![身份验证流](./images/auth-flow.png)

1. <span data-ttu-id="69ce8-109">用户单击应用中的“登录”按钮。</span><span class="sxs-lookup"><span data-stu-id="69ce8-109">The user clicks the "sign in" button in the app.</span></span> <span data-ttu-id="69ce8-110">此操作由 MVC 控制器处理。</span><span class="sxs-lookup"><span data-stu-id="69ce8-110">This action is handled by an MVC controller.</span></span>
2. <span data-ttu-id="69ce8-111">MVC 控制器返回 **ChallengeResult** 操作。</span><span class="sxs-lookup"><span data-stu-id="69ce8-111">The MVC controller returns a **ChallengeResult** action.</span></span>
3. <span data-ttu-id="69ce8-112">中间件截获 **ChallengeResult**，并创建 302 响应，以将用户重定向到 Azure AD 登录页面。</span><span class="sxs-lookup"><span data-stu-id="69ce8-112">The middleware intercepts the **ChallengeResult** and creates a 302 response, which redirects the user to the Azure AD sign-in page.</span></span>
4. <span data-ttu-id="69ce8-113">用户向 Azure AD 验证身份。</span><span class="sxs-lookup"><span data-stu-id="69ce8-113">The user authenticates with Azure AD.</span></span>
5. <span data-ttu-id="69ce8-114">Azure AD 向应用程序发送一个 ID 令牌。</span><span class="sxs-lookup"><span data-stu-id="69ce8-114">Azure AD sends an ID token to the application.</span></span>
6. <span data-ttu-id="69ce8-115">中间件验证该 ID 令牌。</span><span class="sxs-lookup"><span data-stu-id="69ce8-115">The middleware validates the ID token.</span></span> <span data-ttu-id="69ce8-116">此时，用户已在应用程序内通过身份验证。</span><span class="sxs-lookup"><span data-stu-id="69ce8-116">At this point, the user is now authenticated inside the application.</span></span>
7. <span data-ttu-id="69ce8-117">中间件将用户重定向回应用程序。</span><span class="sxs-lookup"><span data-stu-id="69ce8-117">The middleware redirects the user back to application.</span></span>

## <a name="register-the-app-with-azure-ad"></a><span data-ttu-id="69ce8-118">将应用注册到 Azure AD</span><span class="sxs-lookup"><span data-stu-id="69ce8-118">Register the app with Azure AD</span></span>
<span data-ttu-id="69ce8-119">若要启用 OpenID Connect，SaaS 提供程序可在自己的 Azure AD 租户内注册应用程序。</span><span class="sxs-lookup"><span data-stu-id="69ce8-119">To enable OpenID Connect, the SaaS provider registers the application inside their own Azure AD tenant.</span></span>

<span data-ttu-id="69ce8-120">若要注册应用程序，请遵循[将应用程序与 Azure Active Directory 集成](/azure/active-directory/active-directory-integrating-applications/)的[添加应用程序](/azure/active-directory/active-directory-integrating-applications/#adding-an-application)部分中的步骤。</span><span class="sxs-lookup"><span data-stu-id="69ce8-120">To register the application, follow the steps in [Integrating Applications with Azure Active Directory](/azure/active-directory/active-directory-integrating-applications/), in the section [Adding an Application](/azure/active-directory/active-directory-integrating-applications/#adding-an-application).</span></span>

<span data-ttu-id="69ce8-121">有关 Surveys 应用程序的具体步骤，请参阅[运行 Surveys 应用程序](./run-the-app.md)。</span><span class="sxs-lookup"><span data-stu-id="69ce8-121">See [Run the Surveys application](./run-the-app.md) for the specific steps for the Surveys application.</span></span> <span data-ttu-id="69ce8-122">注意以下事项：</span><span class="sxs-lookup"><span data-stu-id="69ce8-122">Note the following:</span></span>

- <span data-ttu-id="69ce8-123">对于多租户应用程序，必须显式配置多租户选项。</span><span class="sxs-lookup"><span data-stu-id="69ce8-123">For a multitenant application, you must configure the multi-tenanted option explicitly.</span></span> <span data-ttu-id="69ce8-124">这样，其他组织就能访问该应用程序。</span><span class="sxs-lookup"><span data-stu-id="69ce8-124">This enables other organizations to to access the application.</span></span>

- <span data-ttu-id="69ce8-125">回复 URL 是 Azure AD 将在其中发送 OAuth 2.0 响应的 URL。</span><span class="sxs-lookup"><span data-stu-id="69ce8-125">The reply URL is the URL where Azure AD will send OAuth 2.0 responses.</span></span> <span data-ttu-id="69ce8-126">使用 ASP.NET Core 时，该 URL 必须与在身份验证中间件中配置的路径（请参阅下一部分）匹配。</span><span class="sxs-lookup"><span data-stu-id="69ce8-126">When using the ASP.NET Core, this needs to match the path that you configure in the authentication middleware (see next section),</span></span> 

## <a name="configure-the-auth-middleware"></a><span data-ttu-id="69ce8-127">配置身份验证中间件</span><span class="sxs-lookup"><span data-stu-id="69ce8-127">Configure the auth middleware</span></span>
<span data-ttu-id="69ce8-128">本部分介绍如何在 ASP.NET Core 中为多租户身份验证配置 OpenID Connect 的身份验证中间件。</span><span class="sxs-lookup"><span data-stu-id="69ce8-128">This section describes how to configure the authentication middleware in ASP.NET Core for multitenant authentication with OpenID Connect.</span></span>

<span data-ttu-id="69ce8-129">在 [startup 类](/aspnet/core/fundamentals/startup)中，添加 OpenID Connect 中间件：</span><span class="sxs-lookup"><span data-stu-id="69ce8-129">In your [startup class](/aspnet/core/fundamentals/startup), add the OpenID Connect middleware:</span></span>

```csharp
app.UseOpenIdConnectAuthentication(new OpenIdConnectOptions {
    ClientId = configOptions.AzureAd.ClientId,
    ClientSecret = configOptions.AzureAd.ClientSecret, // for code flow
    Authority = Constants.AuthEndpointPrefix,
    ResponseType = OpenIdConnectResponseType.CodeIdToken,
    PostLogoutRedirectUri = configOptions.AzureAd.PostLogoutRedirectUri,
    SignInScheme = CookieAuthenticationDefaults.AuthenticationScheme,
    TokenValidationParameters = new TokenValidationParameters { ValidateIssuer = false },
    Events = new SurveyAuthenticationEvents(configOptions.AzureAd, loggerFactory),
});
```

<span data-ttu-id="69ce8-130">请注意，某些设置来自运行时配置选项。</span><span class="sxs-lookup"><span data-stu-id="69ce8-130">Notice that some of the settings are taken from runtime configuration options.</span></span> <span data-ttu-id="69ce8-131">下面说明了各中间件选项的含义：</span><span class="sxs-lookup"><span data-stu-id="69ce8-131">Here's what the middleware options mean:</span></span>

* <span data-ttu-id="69ce8-132">**ClientId**。</span><span class="sxs-lookup"><span data-stu-id="69ce8-132">**ClientId**.</span></span> <span data-ttu-id="69ce8-133">应用程序的客户端 ID，在 Azure AD 中注册应用程序时获得。</span><span class="sxs-lookup"><span data-stu-id="69ce8-133">The application's client ID, which you got when you registered the application in Azure AD.</span></span>
* <span data-ttu-id="69ce8-134">**Authority**。</span><span class="sxs-lookup"><span data-stu-id="69ce8-134">**Authority**.</span></span> <span data-ttu-id="69ce8-135">对于多租户应用程序，将此选项设置为 `https://login.microsoftonline.com/common/`。</span><span class="sxs-lookup"><span data-stu-id="69ce8-135">For a multitenant application, set this to `https://login.microsoftonline.com/common/`.</span></span> <span data-ttu-id="69ce8-136">这是 Azure AD common 终结点的 URL，允许用户从任何 Azure AD 租户登录。</span><span class="sxs-lookup"><span data-stu-id="69ce8-136">This is the URL for the Azure AD common endpoint, which enables users from any Azure AD tenant to sign in.</span></span> <span data-ttu-id="69ce8-137">有关 common 终结点的详细信息，请参阅[这篇博客文章](http://www.cloudidentity.com/blog/2014/08/26/the-common-endpoint-walks-like-a-tenant-talks-like-a-tenant-but-is-not-a-tenant/)。</span><span class="sxs-lookup"><span data-stu-id="69ce8-137">For more information about the common endpoint, see [this blog post](http://www.cloudidentity.com/blog/2014/08/26/the-common-endpoint-walks-like-a-tenant-talks-like-a-tenant-but-is-not-a-tenant/).</span></span>
* <span data-ttu-id="69ce8-138">在 **TokenValidationParameters** 中，将 **ValidateIssuer** 设置为 false。</span><span class="sxs-lookup"><span data-stu-id="69ce8-138">In **TokenValidationParameters**, set **ValidateIssuer** to false.</span></span> <span data-ttu-id="69ce8-139">这意味着应用将负责验证 ID 令牌中的颁发者值。</span><span class="sxs-lookup"><span data-stu-id="69ce8-139">That means the app will be responsible for validating the issuer value in the ID token.</span></span> <span data-ttu-id="69ce8-140">（中间件仍然验证令牌本身。）有关验证颁发者的详细信息，请参阅[颁发者验证](claims.md#issuer-validation)。</span><span class="sxs-lookup"><span data-stu-id="69ce8-140">(The middleware still validates the token itself.) For more information about validating the issuer, see [Issuer validation](claims.md#issuer-validation).</span></span>
* <span data-ttu-id="69ce8-141">**PostLogoutRedirectUri**。</span><span class="sxs-lookup"><span data-stu-id="69ce8-141">**PostLogoutRedirectUri**.</span></span> <span data-ttu-id="69ce8-142">指定用于在注销后重定向用户的 URL。这应当是允许匿名请求的页面 &mdash; 通常是主页。</span><span class="sxs-lookup"><span data-stu-id="69ce8-142">Specify a URL to redirect users after the sign out. This should be a page that allows anonymous requests &mdash; typically the home page.</span></span>
* <span data-ttu-id="69ce8-143">**SignInScheme**。</span><span class="sxs-lookup"><span data-stu-id="69ce8-143">**SignInScheme**.</span></span> <span data-ttu-id="69ce8-144">将其设置为 `CookieAuthenticationDefaults.AuthenticationScheme`。</span><span class="sxs-lookup"><span data-stu-id="69ce8-144">Set this to `CookieAuthenticationDefaults.AuthenticationScheme`.</span></span> <span data-ttu-id="69ce8-145">此设置意味着用户验证身份后，用户声明存储在本地 cookie 中。</span><span class="sxs-lookup"><span data-stu-id="69ce8-145">This setting means that after the user is authenticated, the user claims are stored locally in a cookie.</span></span> <span data-ttu-id="69ce8-146">使用此 cookie，用户可以在浏览器会话期间保持登录状态。</span><span class="sxs-lookup"><span data-stu-id="69ce8-146">This cookie is how the user stays logged in during the browser session.</span></span>
* <span data-ttu-id="69ce8-147">**Events**。</span><span class="sxs-lookup"><span data-stu-id="69ce8-147">**Events.**</span></span> <span data-ttu-id="69ce8-148">事件回叫；请参阅[身份验证事件](#authentication-events)。</span><span class="sxs-lookup"><span data-stu-id="69ce8-148">Event callbacks; see [Authentication events](#authentication-events).</span></span>

<span data-ttu-id="69ce8-149">同时将 Cookie 身份验证中间件添加到管道中。</span><span class="sxs-lookup"><span data-stu-id="69ce8-149">Also add the Cookie Authentication middleware to the pipeline.</span></span> <span data-ttu-id="69ce8-150">此中间件负责将用户声明写入 cookie，然后在后续页面加载期间读取该 cookie。</span><span class="sxs-lookup"><span data-stu-id="69ce8-150">This middleware is responsible for writing the user claims to a cookie, and then reading the cookie during subsequent page loads.</span></span>

```csharp
app.UseCookieAuthentication(new CookieAuthenticationOptions {
    AutomaticAuthenticate = true,
    AutomaticChallenge = true,
    AccessDeniedPath = "/Home/Forbidden",
    CookieSecure = CookieSecurePolicy.Always,

    // The default setting for cookie expiration is 14 days. SlidingExpiration is set to true by default
    ExpireTimeSpan = TimeSpan.FromHours(1),
    SlidingExpiration = true
});
```

## <a name="initiate-the-authentication-flow"></a><span data-ttu-id="69ce8-151">启动身份验证流</span><span class="sxs-lookup"><span data-stu-id="69ce8-151">Initiate the authentication flow</span></span>
<span data-ttu-id="69ce8-152">若要在 ASP.NET MVC 中启动身份验证流，请从控制器返回 **ChallengeResult**：</span><span class="sxs-lookup"><span data-stu-id="69ce8-152">To start the authentication flow in ASP.NET MVC, return a **ChallengeResult** from the contoller:</span></span>

```csharp
[AllowAnonymous]
public IActionResult SignIn()
{
    return new ChallengeResult(
        OpenIdConnectDefaults.AuthenticationScheme,
        new AuthenticationProperties
        {
            IsPersistent = true,
            RedirectUri = Url.Action("SignInCallback", "Account")
        });
}
```

<span data-ttu-id="69ce8-153">这会使中间件返回 302（已找到）响应，以重定向到身份验证终结点。</span><span class="sxs-lookup"><span data-stu-id="69ce8-153">This causes the middleware to return a 302 (Found) response that redirects to the authentication endpoint.</span></span>

## <a name="user-login-sessions"></a><span data-ttu-id="69ce8-154">用户登录会话</span><span class="sxs-lookup"><span data-stu-id="69ce8-154">User login sessions</span></span>
<span data-ttu-id="69ce8-155">如前所述，用户首次登录时，Cookie 身份验证中间件会将用户声明写入 cookie。</span><span class="sxs-lookup"><span data-stu-id="69ce8-155">As mentioned, when the user first signs in, the Cookie Authentication middleware writes the user claims to a cookie.</span></span> <span data-ttu-id="69ce8-156">之后，通过读取该 cookie 对 HTTP 请求进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="69ce8-156">After that, HTTP requests are authenticated by reading the cookie.</span></span>

<span data-ttu-id="69ce8-157">默认情况下，cookie 中间件写入一个[会话 cookie][session-cookie]，该 cookie 在用户关闭浏览器后即被删除。</span><span class="sxs-lookup"><span data-stu-id="69ce8-157">By default, the cookie middleware writes a [session cookie][session-cookie], which gets deleted once the user closes the browser.</span></span> <span data-ttu-id="69ce8-158">用户下次访问此站点时必须再次登录。</span><span class="sxs-lookup"><span data-stu-id="69ce8-158">The next time the user next visits the site, they will have to sign in again.</span></span> <span data-ttu-id="69ce8-159">但是，如果在 **ChallengeResult** 中将 **IsPersistent** 设置为 true，中间件会写入一个永久性 cookie，以便关闭浏览器后用户仍保持登录。</span><span class="sxs-lookup"><span data-stu-id="69ce8-159">However, if you set **IsPersistent** to true in the **ChallengeResult**, the middleware writes a persistent cookie, so the user stays logged in after closing the browser.</span></span> <span data-ttu-id="69ce8-160">可以配置 cookie 到期时间；请参阅[控制 cookie 选项][cookie-options]。</span><span class="sxs-lookup"><span data-stu-id="69ce8-160">You can configure the cookie expiration; see [Controlling cookie options][cookie-options].</span></span> <span data-ttu-id="69ce8-161">永久性 cookie 更方便用户，但可能不适合某些你希望用户每次都登录的应用程序（比如银行应用程序）。</span><span class="sxs-lookup"><span data-stu-id="69ce8-161">Persistent cookies are more convenient for the user, but may be inappropriate for some applications (say, a banking application) where you want the user to sign in every time.</span></span>

## <a name="about-the-openid-connect-middleware"></a><span data-ttu-id="69ce8-162">关于 OpenID Connect 中间件</span><span class="sxs-lookup"><span data-stu-id="69ce8-162">About the OpenID Connect middleware</span></span>
<span data-ttu-id="69ce8-163">ASP.NET 中的 OpenID Connect 中间件隐藏了大多数协议细节。</span><span class="sxs-lookup"><span data-stu-id="69ce8-163">The OpenID Connect middleware in ASP.NET hides most of the protocol details.</span></span> <span data-ttu-id="69ce8-164">本部分包含有关实现的一些说明，可帮助了解协议流。</span><span class="sxs-lookup"><span data-stu-id="69ce8-164">This section contains some notes about the implementation, that may be useful for understanding the protocol flow.</span></span>

<span data-ttu-id="69ce8-165">首先，根据 ASP.NET 检查身份验证流（忽略应用与 Azure AD 之间 OIDC 协议流的细节）。</span><span class="sxs-lookup"><span data-stu-id="69ce8-165">First, let's examine the authentication flow in terms of ASP.NET (ignoring the details of the OIDC protocol flow between the app and Azure AD).</span></span> <span data-ttu-id="69ce8-166">下图展示了此过程。</span><span class="sxs-lookup"><span data-stu-id="69ce8-166">The following diagram shows the process.</span></span>

![登录流](./images/sign-in-flow.png)

<span data-ttu-id="69ce8-168">在此图中，有两个 MVC 控制器。</span><span class="sxs-lookup"><span data-stu-id="69ce8-168">In this diagram, there are two MVC controllers.</span></span> <span data-ttu-id="69ce8-169">帐户控制器处理登录请求，主页控制器处理主页。</span><span class="sxs-lookup"><span data-stu-id="69ce8-169">The Account controller handles sign-in requests, and the Home controller serves up the home page.</span></span>

<span data-ttu-id="69ce8-170">下面是身份验证过程：</span><span class="sxs-lookup"><span data-stu-id="69ce8-170">Here is the authentication process:</span></span>

1. <span data-ttu-id="69ce8-171">用户单击“登录”按钮，浏览器发送 GET 请求。</span><span class="sxs-lookup"><span data-stu-id="69ce8-171">The user clicks the "Sign in" button, and the browser sends a GET request.</span></span> <span data-ttu-id="69ce8-172">例如：`GET /Account/SignIn/`。</span><span class="sxs-lookup"><span data-stu-id="69ce8-172">For example: `GET /Account/SignIn/`.</span></span>
2. <span data-ttu-id="69ce8-173">帐户控制器返回 `ChallengeResult`。</span><span class="sxs-lookup"><span data-stu-id="69ce8-173">The account controller returns a `ChallengeResult`.</span></span>
3. <span data-ttu-id="69ce8-174">OIDC 中间件返回 HTTP 302 响应，以重定向到 Azure AD。</span><span class="sxs-lookup"><span data-stu-id="69ce8-174">The OIDC middleware returns an HTTP 302 response, redirecting to Azure AD.</span></span>
4. <span data-ttu-id="69ce8-175">浏览器将身份验证请求发送到 Azure AD</span><span class="sxs-lookup"><span data-stu-id="69ce8-175">The browser sends the authentication request to Azure AD</span></span>
5. <span data-ttu-id="69ce8-176">用户登录 Azure AD，Azure AD 发回身份验证响应。</span><span class="sxs-lookup"><span data-stu-id="69ce8-176">The user signs in to Azure AD, and Azure AD sends back an authentication response.</span></span>
6. <span data-ttu-id="69ce8-177">OIDC 中间件创建声明主体，并将其传递给 Cookie 身份验证中间件。</span><span class="sxs-lookup"><span data-stu-id="69ce8-177">The OIDC middleware creates a claims principal and passes it to the Cookie Authentication middleware.</span></span>
7. <span data-ttu-id="69ce8-178">cookie 中间件串行化声明主体，并设置 cookie。</span><span class="sxs-lookup"><span data-stu-id="69ce8-178">The cookie middleware serializes the claims principal and sets a cookie.</span></span>
8. <span data-ttu-id="69ce8-179">OIDC 中间件重定向到应用程序的回叫 URL。</span><span class="sxs-lookup"><span data-stu-id="69ce8-179">The OIDC middleware redirects to the application's callback URL.</span></span>
9. <span data-ttu-id="69ce8-180">浏览器遵循重定向操作，发送请求中的 cookie。</span><span class="sxs-lookup"><span data-stu-id="69ce8-180">The browser follows the redirect, sending the cookie in the request.</span></span>
10. <span data-ttu-id="69ce8-181">cookie 中间件将 cookie 反序列化为声明主体，并将 `HttpContext.User` 设置为声明主体。</span><span class="sxs-lookup"><span data-stu-id="69ce8-181">The cookie middleware deserializes the cookie to a claims principal and sets `HttpContext.User` equal to the claims principal.</span></span> <span data-ttu-id="69ce8-182">请求路由到 MVC 控制器中。</span><span class="sxs-lookup"><span data-stu-id="69ce8-182">The request is routed to an MVC controller.</span></span>

### <a name="authentication-ticket"></a><span data-ttu-id="69ce8-183">身份验证票证</span><span class="sxs-lookup"><span data-stu-id="69ce8-183">Authentication ticket</span></span>
<span data-ttu-id="69ce8-184">如果身份验证成功，OIDC 中间件会创建一个身份验证票证，其中包含保存用户声明的声明主体。</span><span class="sxs-lookup"><span data-stu-id="69ce8-184">If authentication succeeds, the OIDC middleware creates an authentication ticket, which contains a claims principal that holds the user's claims.</span></span> <span data-ttu-id="69ce8-185">可在 **AuthenticationValidated** 或 **TicketReceived** 事件内部访问该票证。</span><span class="sxs-lookup"><span data-stu-id="69ce8-185">You can access the ticket inside the **AuthenticationValidated** or **TicketReceived** event.</span></span>

> [!NOTE]
> <span data-ttu-id="69ce8-186">在整个身份验证流完成之前，`HttpContext.User` 仍保存一个匿名主体，而**不是**经过身份验证的用户。</span><span class="sxs-lookup"><span data-stu-id="69ce8-186">Until the entire authentication flow is completed, `HttpContext.User` still holds an anonymous principal, **not** the authenticated user.</span></span> <span data-ttu-id="69ce8-187">该匿名主体具有一个空的声明集合。</span><span class="sxs-lookup"><span data-stu-id="69ce8-187">The anonymous principal has an empty claims collection.</span></span> <span data-ttu-id="69ce8-188">身份验证完成且应用重定向后，cookie 中间件反序列化身份验证 cookie，并将 `HttpContext.User` 设置为表示经过身份验证的用户的声明主体。</span><span class="sxs-lookup"><span data-stu-id="69ce8-188">After authentication completes and the app redirects, the cookie middleware deserializes the authentication cookie and sets `HttpContext.User` to a claims principal that represents the authenticated user.</span></span>
> 
> 

### <a name="authentication-events"></a><span data-ttu-id="69ce8-189">身份验证事件</span><span class="sxs-lookup"><span data-stu-id="69ce8-189">Authentication events</span></span>
<span data-ttu-id="69ce8-190">在身份验证过程中，OpenID Connect 中间件会引发一系列事件：</span><span class="sxs-lookup"><span data-stu-id="69ce8-190">During the authentication process, the OpenID Connect middleware raises a series of events:</span></span>

* <span data-ttu-id="69ce8-191">**RedirectToIdentityProvider**。</span><span class="sxs-lookup"><span data-stu-id="69ce8-191">**RedirectToIdentityProvider**.</span></span> <span data-ttu-id="69ce8-192">就在中间件重定向到身份验证终结点之前调用。</span><span class="sxs-lookup"><span data-stu-id="69ce8-192">Called right before the middleware redirects to the authentication endpoint.</span></span> <span data-ttu-id="69ce8-193">此事件可用于修改重定向 URL；例如，添加请求参数。</span><span class="sxs-lookup"><span data-stu-id="69ce8-193">You can use this event to modify the redirect URL; for example, to add request parameters.</span></span> <span data-ttu-id="69ce8-194">有关示例，请参阅[添加管理员许可提示](signup.md#adding-the-admin-consent-prompt)。</span><span class="sxs-lookup"><span data-stu-id="69ce8-194">See [Adding the admin consent prompt](signup.md#adding-the-admin-consent-prompt) for an example.</span></span>
* <span data-ttu-id="69ce8-195">**AuthorizationCodeReceived**。</span><span class="sxs-lookup"><span data-stu-id="69ce8-195">**AuthorizationCodeReceived**.</span></span> <span data-ttu-id="69ce8-196">使用授权代码调用。</span><span class="sxs-lookup"><span data-stu-id="69ce8-196">Called with the authorization code.</span></span>
* <span data-ttu-id="69ce8-197">**TokenResponseReceived**。</span><span class="sxs-lookup"><span data-stu-id="69ce8-197">**TokenResponseReceived**.</span></span> <span data-ttu-id="69ce8-198">在中间件从 IDP 获取访问令牌之后且验证该令牌之前调用。</span><span class="sxs-lookup"><span data-stu-id="69ce8-198">Called after the middleware gets an access token from the IDP, but before it is validated.</span></span> <span data-ttu-id="69ce8-199">仅适用于授权代码流。</span><span class="sxs-lookup"><span data-stu-id="69ce8-199">Applies only to authorization code flow.</span></span>
* <span data-ttu-id="69ce8-200">**TokenValidated**。</span><span class="sxs-lookup"><span data-stu-id="69ce8-200">**TokenValidated**.</span></span> <span data-ttu-id="69ce8-201">在中间件验证 ID 令牌后调用。</span><span class="sxs-lookup"><span data-stu-id="69ce8-201">Called after the middleware validates the ID token.</span></span> <span data-ttu-id="69ce8-202">此时，应用程序具有一组有关用户的已验证声明。</span><span class="sxs-lookup"><span data-stu-id="69ce8-202">At this point, the application has a set of validated claims about the user.</span></span> <span data-ttu-id="69ce8-203">此事件可用于对声明执行附加验证或转换声明。</span><span class="sxs-lookup"><span data-stu-id="69ce8-203">You can use this event to perform additional validation on the claims, or to transform claims.</span></span> <span data-ttu-id="69ce8-204">请参阅[使用声明](claims.md)。</span><span class="sxs-lookup"><span data-stu-id="69ce8-204">See [Working with claims](claims.md).</span></span>
* <span data-ttu-id="69ce8-205">**UserInformationReceived**。</span><span class="sxs-lookup"><span data-stu-id="69ce8-205">**UserInformationReceived**.</span></span> <span data-ttu-id="69ce8-206">中间件从用户信息终结点中获取用户配置文件时调用。</span><span class="sxs-lookup"><span data-stu-id="69ce8-206">Called if the middleware gets the user profile from the user info endpoint.</span></span> <span data-ttu-id="69ce8-207">仅适用于授权代码流，并且仅当中间件选项中的 `GetClaimsFromUserInfoEndpoint = true` 时可用。</span><span class="sxs-lookup"><span data-stu-id="69ce8-207">Applies only to authorization code flow, and only when `GetClaimsFromUserInfoEndpoint = true` in the middleware options.</span></span>
* <span data-ttu-id="69ce8-208">**TicketReceived**。</span><span class="sxs-lookup"><span data-stu-id="69ce8-208">**TicketReceived**.</span></span> <span data-ttu-id="69ce8-209">身份验证完成时调用。</span><span class="sxs-lookup"><span data-stu-id="69ce8-209">Called when authentication is completed.</span></span> <span data-ttu-id="69ce8-210">如果身份验证成功，这就是最后一个事件。</span><span class="sxs-lookup"><span data-stu-id="69ce8-210">This is the last event, assuming that authentication succeeds.</span></span> <span data-ttu-id="69ce8-211">处理此事件后，用户即登录到应用。</span><span class="sxs-lookup"><span data-stu-id="69ce8-211">After this event is handled, the user is signed into the app.</span></span>
* <span data-ttu-id="69ce8-212">**AuthenticationFailed**。</span><span class="sxs-lookup"><span data-stu-id="69ce8-212">**AuthenticationFailed**.</span></span> <span data-ttu-id="69ce8-213">身份验证失败时调用。</span><span class="sxs-lookup"><span data-stu-id="69ce8-213">Called if authentication fails.</span></span> <span data-ttu-id="69ce8-214">此事件用于处理身份验证失败 &mdash; 例如，通过重定向到错误页。</span><span class="sxs-lookup"><span data-stu-id="69ce8-214">Use this event to handle authentication failures &mdash; for example, by redirecting to an error page.</span></span>

<span data-ttu-id="69ce8-215">若要提供这些事件的回叫，请设置中间件上的 **Events** 选项。</span><span class="sxs-lookup"><span data-stu-id="69ce8-215">To provide callbacks for these events, set the **Events** option on the middleware.</span></span> <span data-ttu-id="69ce8-216">可通过两种不同的方式来声明事件处理程序：使用 lambdas 内联，或在派生自 **OpenIdConnectEvents** 的类中。</span><span class="sxs-lookup"><span data-stu-id="69ce8-216">There are two different ways to declare the event handlers: Inline with lambdas, or in a class that derives from **OpenIdConnectEvents**.</span></span> <span data-ttu-id="69ce8-217">如果事件回叫包含大量逻辑，则建议使用第二种方法，这样就不会影响 startup 类。</span><span class="sxs-lookup"><span data-stu-id="69ce8-217">The second approach is recommended if your event callbacks have any substantial logic, so they don't clutter your startup class.</span></span> <span data-ttu-id="69ce8-218">我们的引用实现使用此方法。</span><span class="sxs-lookup"><span data-stu-id="69ce8-218">Our reference implementation uses this approach.</span></span>

### <a name="openid-connect-endpoints"></a><span data-ttu-id="69ce8-219">OpenID Connect 终结点</span><span class="sxs-lookup"><span data-stu-id="69ce8-219">OpenID connect endpoints</span></span>
<span data-ttu-id="69ce8-220">Azure AD 支持 [OpenID Connect 发现](https://openid.net/specs/openid-connect-discovery-1_0.html)，标识提供者 (IDP) 可在其中返回来自某个[已知终结点](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig)的 JSON 元数据文档。</span><span class="sxs-lookup"><span data-stu-id="69ce8-220">Azure AD supports [OpenID Connect Discovery](https://openid.net/specs/openid-connect-discovery-1_0.html), wherein the identity provider (IDP) returns a JSON metadata document from a [well-known endpoint](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig).</span></span> <span data-ttu-id="69ce8-221">元数据文档包含以下信息：</span><span class="sxs-lookup"><span data-stu-id="69ce8-221">The metadata document contains information such as:</span></span>

* <span data-ttu-id="69ce8-222">授权终结点的 URL。</span><span class="sxs-lookup"><span data-stu-id="69ce8-222">The URL of the authorization endpoint.</span></span> <span data-ttu-id="69ce8-223">应用通过重定向到该 URL 来验证用户身份。</span><span class="sxs-lookup"><span data-stu-id="69ce8-223">This is where the app redirects to authenticate the user.</span></span>
* <span data-ttu-id="69ce8-224">“结束会话”终结点的 URL，应用通过转到该 URL 来注销用户。</span><span class="sxs-lookup"><span data-stu-id="69ce8-224">The URL of the "end session" endpoint, where the app goes to log out the user.</span></span>
* <span data-ttu-id="69ce8-225">用于获取签名密钥的 URL，客户端用来验证从 IDP 获取的 OIDC 令牌。</span><span class="sxs-lookup"><span data-stu-id="69ce8-225">The URL to get the signing keys, which the client uses to validate the OIDC tokens that it gets from the IDP.</span></span>

<span data-ttu-id="69ce8-226">默认情况下，OIDC 中间件知道如何提取此元数据。</span><span class="sxs-lookup"><span data-stu-id="69ce8-226">By default, the OIDC middleware knows how to fetch this metadata.</span></span> <span data-ttu-id="69ce8-227">设置中间件中的 **Authority** 选项，中间件会为元数据构造 URL。</span><span class="sxs-lookup"><span data-stu-id="69ce8-227">Set the **Authority** option in the middleware, and the middleware constructs the URL for the metadata.</span></span> <span data-ttu-id="69ce8-228">（可通过设置 **MetadataAddress** 选项重写元数据 URL。）</span><span class="sxs-lookup"><span data-stu-id="69ce8-228">(You can override the metadata URL by setting the **MetadataAddress** option.)</span></span>

### <a name="openid-connect-flows"></a><span data-ttu-id="69ce8-229">OpenID Connect 流</span><span class="sxs-lookup"><span data-stu-id="69ce8-229">OpenID connect flows</span></span>
<span data-ttu-id="69ce8-230">默认情况下，OIDC 中间件将混合流与窗体发布响应模式结合使用。</span><span class="sxs-lookup"><span data-stu-id="69ce8-230">By default, the OIDC middleware uses hybrid flow with form post response mode.</span></span>

* <span data-ttu-id="69ce8-231">*混合流*意味着客户端可以在同一个授权服务器往返中获取 ID 令牌和授权代码。</span><span class="sxs-lookup"><span data-stu-id="69ce8-231">*Hybrid flow* means the client can get an ID token and an authorization code in the same round-trip to the authorization server.</span></span>
* <span data-ttu-id="69ce8-232">*窗体发布响应模式*意味着授权服务器使用 HTTP POST 请求将 ID 令牌和授权代码发送到应用。</span><span class="sxs-lookup"><span data-stu-id="69ce8-232">*Form post reponse mode* means the authorization server uses an HTTP POST request to send the ID token and authorization code to the app.</span></span> <span data-ttu-id="69ce8-233">其值采用 form-urlencoded 格式（内容类型 =“application/x-www-form-urlencoded”）。</span><span class="sxs-lookup"><span data-stu-id="69ce8-233">The values are form-urlencoded (content type = "application/x-www-form-urlencoded").</span></span>

<span data-ttu-id="69ce8-234">当 OIDC 中间件重定向到授权终结点时，重定向 URL 包含 OIDC 所需的所有查询字符串参数。</span><span class="sxs-lookup"><span data-stu-id="69ce8-234">When the OIDC middleware redirects to the authorization endpoint, the redirect URL includes all of the query string parameters needed by OIDC.</span></span> <span data-ttu-id="69ce8-235">对于混合流：</span><span class="sxs-lookup"><span data-stu-id="69ce8-235">For hybrid flow:</span></span>

* <span data-ttu-id="69ce8-236">client_id。</span><span class="sxs-lookup"><span data-stu-id="69ce8-236">client_id.</span></span> <span data-ttu-id="69ce8-237">此值在 **ClientId** 选项中设置</span><span class="sxs-lookup"><span data-stu-id="69ce8-237">This value is set in the **ClientId** option</span></span>
* <span data-ttu-id="69ce8-238">scope = "openid profile"，这表示它是一个 OIDC 请求，我们需要用户的配置文件。</span><span class="sxs-lookup"><span data-stu-id="69ce8-238">scope = "openid profile", which means it's an OIDC request and we want the user's profile.</span></span>
* <span data-ttu-id="69ce8-239">response_type  = "code id_token"。</span><span class="sxs-lookup"><span data-stu-id="69ce8-239">response_type  = "code id_token".</span></span> <span data-ttu-id="69ce8-240">此值指定混合流。</span><span class="sxs-lookup"><span data-stu-id="69ce8-240">This specifies hybrid flow.</span></span>
* <span data-ttu-id="69ce8-241">response_mode = "form_post"。</span><span class="sxs-lookup"><span data-stu-id="69ce8-241">response_mode = "form_post".</span></span> <span data-ttu-id="69ce8-242">此值指定窗体发布响应。</span><span class="sxs-lookup"><span data-stu-id="69ce8-242">This specifies form post response.</span></span>

<span data-ttu-id="69ce8-243">若要指定其他流，请设置选项上的 **ResponseType** 属性。</span><span class="sxs-lookup"><span data-stu-id="69ce8-243">To specify a different flow, set the **ResponseType** property on the options.</span></span> <span data-ttu-id="69ce8-244">例如：</span><span class="sxs-lookup"><span data-stu-id="69ce8-244">For example:</span></span>

```csharp
app.UseOpenIdConnectAuthentication(options =>
{
    options.ResponseType = "code"; // Authorization code flow

    // Other options
}
```

<span data-ttu-id="69ce8-245">[**下一篇**][claims]</span><span class="sxs-lookup"><span data-stu-id="69ce8-245">[**Next**][claims]</span></span>

[claims]: claims.md
[cookie-options]: /aspnet/core/security/authentication/cookie#controlling-cookie-options
[session-cookie]: https://en.wikipedia.org/wiki/HTTP_cookie#Session_cookie
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
