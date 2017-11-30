---
title: "在多租户应用程序中使用基于声明的标识"
description: "如何使用声明进行颁发者验证和授权"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: authenticate
pnp.series.next: signup
ms.openlocfilehash: 61788d9759715b21ef1bdda59c5b54d923fd8f62
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="work-with-claims-based-identities"></a><span data-ttu-id="5a7a5-103">使用基于声明的标识</span><span class="sxs-lookup"><span data-stu-id="5a7a5-103">Work with claims-based identities</span></span>

<span data-ttu-id="5a7a5-104">[![GitHub](../_images/github.png) 示例代码][sample application]</span><span class="sxs-lookup"><span data-stu-id="5a7a5-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

## <a name="claims-in-azure-ad"></a><span data-ttu-id="5a7a5-105">Azure AD 中的声明</span><span class="sxs-lookup"><span data-stu-id="5a7a5-105">Claims in Azure AD</span></span>
<span data-ttu-id="5a7a5-106">当用户登录时，Azure AD 会发送一个 ID 令牌，其中包含有关该用户的声明集。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-106">When a user signs in, Azure AD sends an ID token that contains a set of claims about the user.</span></span> <span data-ttu-id="5a7a5-107">声明只需是一段以键值对形式表示的信息。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-107">A claim is simply a piece of information, expressed as a key/value pair.</span></span> <span data-ttu-id="5a7a5-108">例如，`email`=`bob@contoso.com`。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-108">For example, `email`=`bob@contoso.com`.</span></span>  <span data-ttu-id="5a7a5-109">声明包含一个颁发者 &mdash; 在本例中为 Azure AD &mdash; 即用于验证用户身份和创建声明的实体。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-109">Claims have an issuer &mdash; in this case, Azure AD &mdash; which is the entity that authenticates the user and creates the claims.</span></span> <span data-ttu-id="5a7a5-110">由于你信任该颁发者，因此也就信任这些声明。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-110">You trust the claims because you trust the issuer.</span></span> <span data-ttu-id="5a7a5-111">（相反，如果你不信任颁发者，则不信任声明！）</span><span class="sxs-lookup"><span data-stu-id="5a7a5-111">(Conversely, if you don't trust the issuer, don't trust the claims!)</span></span>

<span data-ttu-id="5a7a5-112">在高级别中：</span><span class="sxs-lookup"><span data-stu-id="5a7a5-112">At a high level:</span></span>

1. <span data-ttu-id="5a7a5-113">用户进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-113">The user authenticates.</span></span>
2. <span data-ttu-id="5a7a5-114">IDP 发送声明集。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-114">The IDP sends a set of claims.</span></span>
3. <span data-ttu-id="5a7a5-115">应用规范化或补充声明（可选）。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-115">The app normalizes or augments the claims (optional).</span></span>
4. <span data-ttu-id="5a7a5-116">应用使用声明做出授权决策。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-116">The app uses the claims to make authorization decisions.</span></span>

<span data-ttu-id="5a7a5-117">在 OpenID Connect 中，收到的声明集由身份验证请求的[范围参数]控制。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-117">In OpenID Connect, the set of claims that you get is controlled by the [scope parameter] of the authentication request.</span></span> <span data-ttu-id="5a7a5-118">但是，Azure AD 会通过 OpenID Connect 颁发受限的声明集；请参阅[支持的令牌和声明类型]。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-118">However, Azure AD issues a limited set of claims through OpenID Connect; see [Supported Token and Claim Types].</span></span> <span data-ttu-id="5a7a5-119">要了解有关用户的详细信息，需要使用 Azure AD 图形 API。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-119">If you want more information about the user, you'll need to use the Azure AD Graph API.</span></span>

<span data-ttu-id="5a7a5-120">下面是 AAD 发送的、可能受应用关注的一些声明：</span><span class="sxs-lookup"><span data-stu-id="5a7a5-120">Here are some of the claims from AAD that an app might typically care about:</span></span>

| <span data-ttu-id="5a7a5-121">ID 令牌中的声明类型</span><span class="sxs-lookup"><span data-stu-id="5a7a5-121">Claim type in ID token</span></span> | <span data-ttu-id="5a7a5-122">说明</span><span class="sxs-lookup"><span data-stu-id="5a7a5-122">Description</span></span> |
| --- | --- |
| <span data-ttu-id="5a7a5-123">aud</span><span class="sxs-lookup"><span data-stu-id="5a7a5-123">aud</span></span> |<span data-ttu-id="5a7a5-124">令牌的颁发对象。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-124">Who the token was issued for.</span></span> <span data-ttu-id="5a7a5-125">此值将是应用程序的客户端 ID。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-125">This will be the application's client ID.</span></span> <span data-ttu-id="5a7a5-126">一般情况下，应该不需要考虑此声明，因为中间件会自动验证它。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-126">Generally, you shouldn't need to worry about this claim, because the middleware automatically validates it.</span></span> <span data-ttu-id="5a7a5-127">示例：`"91464657-d17a-4327-91f3-2ed99386406f"`</span><span class="sxs-lookup"><span data-stu-id="5a7a5-127">Example:  `"91464657-d17a-4327-91f3-2ed99386406f"`</span></span> |
| <span data-ttu-id="5a7a5-128">组</span><span class="sxs-lookup"><span data-stu-id="5a7a5-128">groups</span></span> |<span data-ttu-id="5a7a5-129">用户所属的 AAD 组的列表。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-129">A list of AAD groups of which the user is a member.</span></span> <span data-ttu-id="5a7a5-130">示例：`["93e8f556-8661-4955-87b6-890bc043c30f", "fc781505-18ef-4a31-a7d5-7d931d7b857e"]`</span><span class="sxs-lookup"><span data-stu-id="5a7a5-130">Example: `["93e8f556-8661-4955-87b6-890bc043c30f", "fc781505-18ef-4a31-a7d5-7d931d7b857e"]`</span></span> |
| <span data-ttu-id="5a7a5-131">iss</span><span class="sxs-lookup"><span data-stu-id="5a7a5-131">iss</span></span> |<span data-ttu-id="5a7a5-132">OIDC 令牌的[颁发者]。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-132">The [issuer] of the OIDC token.</span></span> <span data-ttu-id="5a7a5-133">示例：`https://sts.windows.net/b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4/`</span><span class="sxs-lookup"><span data-stu-id="5a7a5-133">Example: `https://sts.windows.net/b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4/`</span></span> |
| <span data-ttu-id="5a7a5-134">名称</span><span class="sxs-lookup"><span data-stu-id="5a7a5-134">name</span></span> |<span data-ttu-id="5a7a5-135">用户的显示名称。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-135">The user's display name.</span></span> <span data-ttu-id="5a7a5-136">示例：`"Alice A."`</span><span class="sxs-lookup"><span data-stu-id="5a7a5-136">Example: `"Alice A."`</span></span> |
| <span data-ttu-id="5a7a5-137">oid</span><span class="sxs-lookup"><span data-stu-id="5a7a5-137">oid</span></span> |<span data-ttu-id="5a7a5-138">AAD 中用户的对象标识符。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-138">The object identifier for the user in AAD.</span></span> <span data-ttu-id="5a7a5-139">此值是用户的不可变且不可重用的标识符。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-139">This value is the immutable and non-reusable identifier of the user.</span></span> <span data-ttu-id="5a7a5-140">请使用此值而不是电子邮件作为用户的唯一标识符；电子邮件地址可能会更改。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-140">Use this value, not email, as a unique identifier for users; email addresses can change.</span></span> <span data-ttu-id="5a7a5-141">如果在应用中使用 Azure AD 图形 API，对象 ID 是用于查询配置文件信息的值。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-141">If you use the Azure AD Graph API in your app, object ID is that value used to query profile information.</span></span> <span data-ttu-id="5a7a5-142">示例：`"59f9d2dc-995a-4ddf-915e-b3bb314a7fa4"`</span><span class="sxs-lookup"><span data-stu-id="5a7a5-142">Example: `"59f9d2dc-995a-4ddf-915e-b3bb314a7fa4"`</span></span> |
| <span data-ttu-id="5a7a5-143">角色</span><span class="sxs-lookup"><span data-stu-id="5a7a5-143">roles</span></span> |<span data-ttu-id="5a7a5-144">用户的应用角色列表。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-144">A list of app roles for the user.</span></span>    <span data-ttu-id="5a7a5-145">示例：`["SurveyCreator"]`</span><span class="sxs-lookup"><span data-stu-id="5a7a5-145">Example: `["SurveyCreator"]`</span></span> |
| <span data-ttu-id="5a7a5-146">tid</span><span class="sxs-lookup"><span data-stu-id="5a7a5-146">tid</span></span> |<span data-ttu-id="5a7a5-147">租户 ID。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-147">Tenant ID.</span></span> <span data-ttu-id="5a7a5-148">此值是 Azure AD 中租户的唯一标识符。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-148">This value is a unique identifier for the tenant in Azure AD.</span></span> <span data-ttu-id="5a7a5-149">示例：`"b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4"`</span><span class="sxs-lookup"><span data-stu-id="5a7a5-149">Example: `"b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4"`</span></span> |
| <span data-ttu-id="5a7a5-150">unique_name</span><span class="sxs-lookup"><span data-stu-id="5a7a5-150">unique_name</span></span> |<span data-ttu-id="5a7a5-151">用户的人工可读显示名称。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-151">A human readable display name of the user.</span></span> <span data-ttu-id="5a7a5-152">示例：`"alice@contoso.com"`</span><span class="sxs-lookup"><span data-stu-id="5a7a5-152">Example: `"alice@contoso.com"`</span></span> |
| <span data-ttu-id="5a7a5-153">upn</span><span class="sxs-lookup"><span data-stu-id="5a7a5-153">upn</span></span> |<span data-ttu-id="5a7a5-154">用户主体名称。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-154">User principal name.</span></span> <span data-ttu-id="5a7a5-155">示例：`"alice@contoso.com"`</span><span class="sxs-lookup"><span data-stu-id="5a7a5-155">Example: `"alice@contoso.com"`</span></span> |

<span data-ttu-id="5a7a5-156">此表列出了 ID 令牌中显示的声明类型。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-156">This table lists the claim types as they appear in the ID token.</span></span> <span data-ttu-id="5a7a5-157">在 ASP.NET Core 中，OpenID Connect 中间件会在填充用户主体的声明集合时转换某些声明类型：</span><span class="sxs-lookup"><span data-stu-id="5a7a5-157">In ASP.NET Core, the OpenID Connect middleware converts some of the claim types when it populates the Claims collection for the user principal:</span></span>

* <span data-ttu-id="5a7a5-158">oid > `http://schemas.microsoft.com/identity/claims/objectidentifier`</span><span class="sxs-lookup"><span data-stu-id="5a7a5-158">oid > `http://schemas.microsoft.com/identity/claims/objectidentifier`</span></span>
* <span data-ttu-id="5a7a5-159">tid > `http://schemas.microsoft.com/identity/claims/tenantid`</span><span class="sxs-lookup"><span data-stu-id="5a7a5-159">tid > `http://schemas.microsoft.com/identity/claims/tenantid`</span></span>
* <span data-ttu-id="5a7a5-160">unique_name > `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`</span><span class="sxs-lookup"><span data-stu-id="5a7a5-160">unique_name > `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`</span></span>
* <span data-ttu-id="5a7a5-161">upn > `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn`</span><span class="sxs-lookup"><span data-stu-id="5a7a5-161">upn > `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn`</span></span>

## <a name="claims-transformations"></a><span data-ttu-id="5a7a5-162">声明转换</span><span class="sxs-lookup"><span data-stu-id="5a7a5-162">Claims transformations</span></span>
<span data-ttu-id="5a7a5-163">在执行身份验证流期间，可能需要修改从 IDP 获取的声明。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-163">During the authentication flow, you might want to modify the claims that you get from the IDP.</span></span> <span data-ttu-id="5a7a5-164">在 ASP.NET Core 中，可以通过 OpenID Connect 中间件在 **AuthenticationValidated** 事件的内部执行声明转换。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-164">In ASP.NET Core, you can perform claims transformation inside of the **AuthenticationValidated** event from the OpenID Connect middleware.</span></span> <span data-ttu-id="5a7a5-165">（请参阅[身份验证事件]。）</span><span class="sxs-lookup"><span data-stu-id="5a7a5-165">(See [Authentication events].)</span></span>

<span data-ttu-id="5a7a5-166">在执行 **AuthenticationValidated** 期间添加的所有声明会存储在会话身份验证 Cookie 中。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-166">Any claims that you add during **AuthenticationValidated** are stored in the session authentication cookie.</span></span> <span data-ttu-id="5a7a5-167">这些声明不会推回到 Azure AD。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-167">They don't get pushed back to Azure AD.</span></span>

<span data-ttu-id="5a7a5-168">下面是声明转换的一些示例：</span><span class="sxs-lookup"><span data-stu-id="5a7a5-168">Here are some examples of claims transformation:</span></span>

* <span data-ttu-id="5a7a5-169">**声明规范化**，或者在用户间保持声明一致。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-169">**Claims normalization**, or making claims consistent across users.</span></span> <span data-ttu-id="5a7a5-170">这尤其适用于从多个 IDP 获取声明的情况，此时，可能对类似的信息使用了不同的声明类型。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-170">This is particularly relevant if you are getting claims from multiple IDPs, which might use different claim types for similar information.</span></span>
  <span data-ttu-id="5a7a5-171">例如，Azure AD 会发送一个包含用户电子邮件的“upn”声明。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-171">For example, Azure AD sends a "upn" claim that contains the user's email.</span></span> <span data-ttu-id="5a7a5-172">其他 IDP 可能发送“email”声明。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-172">Other IDPs might send an "email" claim.</span></span> <span data-ttu-id="5a7a5-173">以下代码将“upn”声明转换为“email”声明：</span><span class="sxs-lookup"><span data-stu-id="5a7a5-173">The following code converts the "upn" claim into an "email" claim:</span></span>
  
  ```csharp
  var email = principal.FindFirst(ClaimTypes.Upn)?.Value;
  if (!string.IsNullOrWhiteSpace(email))
  {
      identity.AddClaim(new Claim(ClaimTypes.Email, email));
  }
  ```
* <span data-ttu-id="5a7a5-174">为不存在的声明添加**默认声明值** &mdash; 例如，将用户分配到默认角色。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-174">Add **default claim values** for claims that aren't present &mdash; for example, assigning a user to a default role.</span></span> <span data-ttu-id="5a7a5-175">在某些情况下，这可以简化授权逻辑。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-175">In some cases this can simplify authorization logic.</span></span>
* <span data-ttu-id="5a7a5-176">添加包含有关用户的应用程序特定信息的**自定义声明类型**。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-176">Add **custom claim types** with application-specific information about the user.</span></span> <span data-ttu-id="5a7a5-177">例如，可以在数据库中存储有关用户的某些信息。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-177">For example, you might store some information about the user in a database.</span></span> <span data-ttu-id="5a7a5-178">可将包含此信息的自定义声明添加到身份验证票证。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-178">You could add a custom claim with this information to the authentication ticket.</span></span> <span data-ttu-id="5a7a5-179">声明存储在 Cookie 中，因此，只需在每个登录会话中从数据库获取该声明一次。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-179">The claim is stored in a cookie, so you only need to get it from the database once per login session.</span></span> <span data-ttu-id="5a7a5-180">另一方面，我们还希望避免创建过大的 Cookie，因此，需考虑在 Cookie 大小与数据库查找量之间做出取舍。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-180">On the other hand, you also want to avoid creating excessively large cookies, so you need to consider the trade-off between cookie size versus database lookups.</span></span>   

<span data-ttu-id="5a7a5-181">完成身份验证流之后，`HttpContext.User` 中会提供声明。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-181">After the authentication flow is complete, the claims are available in `HttpContext.User`.</span></span> <span data-ttu-id="5a7a5-182">此时，应将这些声明视为只读的集合 &mdash; 例如，使用它们做出授权决策。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-182">At that point, you should treat them as a read-only collection &mdash; e.g., use them to make authorization decisions.</span></span>

## <a name="issuer-validation"></a><span data-ttu-id="5a7a5-183">颁发者验证</span><span class="sxs-lookup"><span data-stu-id="5a7a5-183">Issuer validation</span></span>
<span data-ttu-id="5a7a5-184">在 OpenID Connect 中，颁发者声明（“iss”）标识颁发 ID 令牌的 IDP。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-184">In OpenID Connect, the issuer claim ("iss") identifies the IDP that issued the ID token.</span></span> <span data-ttu-id="5a7a5-185">OIDC 身份验证流的部分工作是验证颁发者声明是否与实际颁发者匹配。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-185">Part of the OIDC authentication flow is to verify that the issuer claim matches the actual issuer.</span></span> <span data-ttu-id="5a7a5-186">OIDC 中间件会自动处理此验证。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-186">The OIDC middleware handles this for you.</span></span>

<span data-ttu-id="5a7a5-187">在 Azure AD 中，颁发者值是每个 AD 租户的唯一值 (`https://sts.windows.net/<tenantID>`)。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-187">In Azure AD, the issuer value is unique per AD tenant (`https://sts.windows.net/<tenantID>`).</span></span> <span data-ttu-id="5a7a5-188">因此，应用程序应该执行额外的检查，确保颁发者代表有权登录到应用的租户。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-188">Therefore, an application should do an additional check, to make sure the issuer represents a tenant that is allowed to sign in to the app.</span></span>

<span data-ttu-id="5a7a5-189">对于单租户应用程序，只需检查颁发者是否为你自己的租户。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-189">For a single-tenant application, you can just check that the issuer is your own tenant.</span></span> <span data-ttu-id="5a7a5-190">事实上，OIDC 中间件默认会自动执行此检查。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-190">In fact, the OIDC middleware does this automatically by default.</span></span> <span data-ttu-id="5a7a5-191">在多租户应用中，需要允许多个对应于不同租户的颁发者。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-191">In a multi-tenant app, you need to allow for multiple issuers, corresponding to the different tenants.</span></span> <span data-ttu-id="5a7a5-192">下面是一种普通用法：</span><span class="sxs-lookup"><span data-stu-id="5a7a5-192">Here is a general approach to use:</span></span>

* <span data-ttu-id="5a7a5-193">在 OIDC 中间件选项中，将 **ValidateIssuer** 设置为 false。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-193">In the OIDC middleware options, set **ValidateIssuer** to false.</span></span> <span data-ttu-id="5a7a5-194">这会关闭自动检查。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-194">This turns off the automatic check.</span></span>
* <span data-ttu-id="5a7a5-195">当某个租户注册时，会在用户数据库中存储该租户和颁发者。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-195">When a tenant signs up, store the tenant and the issuer in your user DB.</span></span>
* <span data-ttu-id="5a7a5-196">每当用户登录时，会数据库中查找颁发者。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-196">Whenever a user signs in, look up the issuer in the database.</span></span> <span data-ttu-id="5a7a5-197">如果找不到该颁发者，则表示租户尚未注册。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-197">If the issuer isn't found, it means that tenant hasn't signed up.</span></span> <span data-ttu-id="5a7a5-198">可将他们重定向到注册页。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-198">You can redirect them to a sign up page.</span></span>
* <span data-ttu-id="5a7a5-199">还可以将某些租户（例如，未支付订阅费的客户）加入方块列表。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-199">You could also blacklist certain tenants; for example, for customers that didn't pay their subscription.</span></span>

<span data-ttu-id="5a7a5-200">有关更多详细介绍，请参阅[在多租户应用程序中注册和加入租户][signup]。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-200">For a more detailed discussion, see [Sign-up and tenant onboarding in a multitenant application][signup].</span></span>

## <a name="using-claims-for-authorization"></a><span data-ttu-id="5a7a5-201">使用声明进行授权</span><span class="sxs-lookup"><span data-stu-id="5a7a5-201">Using claims for authorization</span></span>
<span data-ttu-id="5a7a5-202">使用声明时，用户的标识不再是单一实体。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-202">With claims, a user's identity is no longer a monolithic entity.</span></span> <span data-ttu-id="5a7a5-203">例如，用户可能具有电子邮件地址、电话号码、生日、性别等信息。所有这些信息可能存储在用户的 IDP 中。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-203">For example, a user might have an email address, phone number, birthday, gender, etc. Maybe the user's IDP stores all of this information.</span></span> <span data-ttu-id="5a7a5-204">但是，对用户进行身份验证时，通常会以声明的形式收到其中的一部分信息。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-204">But when you authenticate the user, you'll typically get a subset of these as claims.</span></span> <span data-ttu-id="5a7a5-205">在此模型中，用户的标识只是一堆声明。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-205">In this model, the user's identity is simply a bundle of claims.</span></span> <span data-ttu-id="5a7a5-206">在针对用户做出授权决策时，会查找特定的声明集。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-206">When you make authorization decisions about a user, you will look for particular sets of claims.</span></span> <span data-ttu-id="5a7a5-207">换而言之，问题“用户 X 是否可以执行操作 Y”最终变成了“用户 X 是否具有声明 Z”。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-207">In other words, the question "Can user X perform action Y" ultimately becomes "Does user X have claim Z".</span></span>

<span data-ttu-id="5a7a5-208">下面是检查声明的一些基本模式。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-208">Here are some basic patterns for checking claims.</span></span>

* <span data-ttu-id="5a7a5-209">若要检查用户是否具有包含特定值的特定声明：</span><span class="sxs-lookup"><span data-stu-id="5a7a5-209">To check that the user has a particular claim with a particular value:</span></span>
  
   ```csharp
   if (User.HasClaim(ClaimTypes.Role, "Admin")) { ... }
   ```
   <span data-ttu-id="5a7a5-210">此代码检查用户是否具有包含值“Admin”的 Role 声明。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-210">This code checks whether the user has a Role claim with the value "Admin".</span></span> <span data-ttu-id="5a7a5-211">如果用户没有 Role 声明或者具有多个 Role 声明，此代码不会做出正确的处理。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-211">It correctly handles the case where the user has no Role claim or multiple Role claims.</span></span>
  
   <span data-ttu-id="5a7a5-212">**ClaimTypes** 类定义常用声明类型的常量。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-212">The **ClaimTypes** class defines constants for commonly-used claim types.</span></span> <span data-ttu-id="5a7a5-213">但是，可对声明类型使用任何字符串值。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-213">However, you can use any string value for the claim type.</span></span>
* <span data-ttu-id="5a7a5-214">当你认为某个声明类型最多只有一个值时，若要获取该声明类型的单个值，请使用以下代码：</span><span class="sxs-lookup"><span data-stu-id="5a7a5-214">To get a single value for a claim type, when you expect there to be at most one value:</span></span>
  
  ```csharp
  string email = User.FindFirst(ClaimTypes.Email)?.Value;
  ```
* <span data-ttu-id="5a7a5-215">若要获取某个声明类型的所有值，请使用以下代码：</span><span class="sxs-lookup"><span data-stu-id="5a7a5-215">To get all the values for a claim type:</span></span>
  
  ```csharp
  IEnumerable<Claim> groups = User.FindAll("groups");
  ```

<span data-ttu-id="5a7a5-216">有关详细信息，请参阅[多租户应用程序中基于角色和基于资源的授权][authorization]。</span><span class="sxs-lookup"><span data-stu-id="5a7a5-216">For more information, see [Role-based and resource-based authorization in multitenant applications][authorization].</span></span>

<span data-ttu-id="5a7a5-217">[**下一篇**][signup]</span><span class="sxs-lookup"><span data-stu-id="5a7a5-217">[**Next**][signup]</span></span>


<!-- Links -->

[范围参数]: http://nat.sakimura.org/2012/01/26/scopes-and-claims-in-openid-connect/
[支持的令牌和声明类型]: /azure/active-directory/active-directory-token-and-claims/
[颁发者]: http://openid.net/specs/openid-connect-core-1_0.html#IDToken
[身份验证事件]: authenticate.md#authentication-events
[signup]: signup.md
[Claims-Based Authorization]: /aspnet/core/security/authorization/claims
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[authorization]: authorize.md
