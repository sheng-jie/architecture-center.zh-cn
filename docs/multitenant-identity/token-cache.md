---
title: "在多租户应用程序中缓存访问令牌"
description: "缓存用于调用后端 Web API 的访问令牌"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: web-api
pnp.series.next: adfs
ms.openlocfilehash: cffc15686ef9d77fafb40982efdbcd4a79f5aaf2
ms.sourcegitcommit: 8ab30776e0c4cdc16ca0dcc881960e3108ad3e94
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/08/2017
---
# <a name="cache-access-tokens"></a><span data-ttu-id="17e6f-103">缓存访问令牌</span><span class="sxs-lookup"><span data-stu-id="17e6f-103">Cache access tokens</span></span>

<span data-ttu-id="17e6f-104">[![GitHub](../_images/github.png) 示例代码][sample application]</span><span class="sxs-lookup"><span data-stu-id="17e6f-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="17e6f-105">获取 OAuth 访问令牌成本较高，因为它需要对令牌终结点进行 HTTP 请求。</span><span class="sxs-lookup"><span data-stu-id="17e6f-105">It's relatively expensive to get an OAuth access token, because it requires an HTTP request to the token endpoint.</span></span> <span data-ttu-id="17e6f-106">因此，最好尽可能缓存令牌。</span><span class="sxs-lookup"><span data-stu-id="17e6f-106">Therefore, it's good to cache tokens whenever possible.</span></span> <span data-ttu-id="17e6f-107">[Azure AD 身份验证库][ADAL] (ADAL) 会自动缓存从 Azure AD 获取的令牌，包括刷新令牌。</span><span class="sxs-lookup"><span data-stu-id="17e6f-107">The [Azure AD Authentication Library][ADAL] (ADAL)  automatically caches tokens obtained from Azure AD, including refresh tokens.</span></span>

<span data-ttu-id="17e6f-108">ADAL 提供默认令牌缓存实现。</span><span class="sxs-lookup"><span data-stu-id="17e6f-108">ADAL provides a default token cache implementation.</span></span> <span data-ttu-id="17e6f-109">但是，此令牌缓存适用于本机客户端应用，不适用于 web 应用：</span><span class="sxs-lookup"><span data-stu-id="17e6f-109">However, this token cache is intended for native client apps, and is **not** suitable for web apps:</span></span>

* <span data-ttu-id="17e6f-110">它是一个静态实例，非线程安全。</span><span class="sxs-lookup"><span data-stu-id="17e6f-110">It is a static instance, and not thread safe.</span></span>
* <span data-ttu-id="17e6f-111">它不会扩展到许多用户，因为所有用户的令牌都进入同一个字典。</span><span class="sxs-lookup"><span data-stu-id="17e6f-111">It doesn't scale to large numbers of users, because tokens from all users go into the same dictionary.</span></span>
* <span data-ttu-id="17e6f-112">它也无法在一个场中的多个 web 服务器间共享。</span><span class="sxs-lookup"><span data-stu-id="17e6f-112">It can't be shared across web servers in a farm.</span></span>

<span data-ttu-id="17e6f-113">应实现自定义令牌缓存（该缓存派生自 ADAL `TokenCache` 类且适用于服务器环境），并为不同用户提供所需的令牌间的隔离级别。</span><span class="sxs-lookup"><span data-stu-id="17e6f-113">Instead, you should implement a custom token cache that derives from the ADAL `TokenCache` class but is suitable for a server environment and provides the desirable level of isolation between tokens for different users.</span></span>

<span data-ttu-id="17e6f-114">`TokenCache` 类存储令牌字典，该字典按照颁发者、资源、客户端 ID 和用户索引。</span><span class="sxs-lookup"><span data-stu-id="17e6f-114">The `TokenCache` class stores a dictionary of tokens, indexed by issuer, resource, client ID, and user.</span></span> <span data-ttu-id="17e6f-115">自定义令牌缓存应将此字典写入一个后备存储，如 Redis 缓存。</span><span class="sxs-lookup"><span data-stu-id="17e6f-115">A custom token cache should write this dictionary to a backing store, such as a Redis cache.</span></span>

<span data-ttu-id="17e6f-116">在 Tailspin Surveys 应用程序中，`DistributedTokenCache` 类实现令牌缓存。</span><span class="sxs-lookup"><span data-stu-id="17e6f-116">In the Tailspin Surveys application, the `DistributedTokenCache` class implements the token cache.</span></span> <span data-ttu-id="17e6f-117">此实现使用 ASP.NET Core 中的 [IDistributedCache][distributed-cache] 抽象。</span><span class="sxs-lookup"><span data-stu-id="17e6f-117">This implementation uses the [IDistributedCache][distributed-cache] abstraction from ASP.NET Core.</span></span> <span data-ttu-id="17e6f-118">这样一来，任何 `IDistributedCache` 实现可用作后备存储。</span><span class="sxs-lookup"><span data-stu-id="17e6f-118">That way, any `IDistributedCache` implementation can be used as a backing store.</span></span>

* <span data-ttu-id="17e6f-119">默认情况下，Surveys 应用使用 Redis 缓存。</span><span class="sxs-lookup"><span data-stu-id="17e6f-119">By default, the Surveys app uses a Redis cache.</span></span>
* <span data-ttu-id="17e6f-120">对于单实例 web 服务器，可使用 ASP.NET Core [内存中缓存][in-memory-cache]。</span><span class="sxs-lookup"><span data-stu-id="17e6f-120">For a single-instance web server, you could use the ASP.NET Core [in-memory cache][in-memory-cache].</span></span> <span data-ttu-id="17e6f-121">（这对于开发期间在本地运行应用也是一个不错的选择。）</span><span class="sxs-lookup"><span data-stu-id="17e6f-121">(This is also a good option for running the app locally during development.)</span></span>

<span data-ttu-id="17e6f-122">`DistributedTokenCache` 将缓存数据以键/值对的形式存储在后备存储中。</span><span class="sxs-lookup"><span data-stu-id="17e6f-122">`DistributedTokenCache` stores the cache data as key/value pairs in the backing store.</span></span> <span data-ttu-id="17e6f-123">键是用户 ID 加客户端 ID，所以后备存储可为用户/客户端的每个唯一组合保存单独的缓存数据。</span><span class="sxs-lookup"><span data-stu-id="17e6f-123">The key is the user ID plus client ID, so the backing store holds separate cache data for each unique combination of user/client.</span></span>

![令牌缓存](./images/token-cache.png)

<span data-ttu-id="17e6f-125">后备存储由用户进行分区。</span><span class="sxs-lookup"><span data-stu-id="17e6f-125">The backing store is partitioned by user.</span></span> <span data-ttu-id="17e6f-126">对于每个 HTTP 请求，用户的令牌从后备存储中读取并加载到 `TokenCache` 字典。</span><span class="sxs-lookup"><span data-stu-id="17e6f-126">For each HTTP request, the tokens for that user are read from the backing store and loaded into the `TokenCache` dictionary.</span></span> <span data-ttu-id="17e6f-127">如果将 Redis 用作后备存储，则服务器场中的每个服务器实例会读取/写入到同一个缓存，并且此方法可扩展到许多用户。</span><span class="sxs-lookup"><span data-stu-id="17e6f-127">If Redis is used as the backing store, every server instance in a server farm reads/writes to the same cache, and this approach scales to many users.</span></span>

## <a name="encrypting-cached-tokens"></a><span data-ttu-id="17e6f-128">加密已缓存令牌</span><span class="sxs-lookup"><span data-stu-id="17e6f-128">Encrypting cached tokens</span></span>
<span data-ttu-id="17e6f-129">令牌是敏感数据，因为令牌授予对用户资源的访问权限。</span><span class="sxs-lookup"><span data-stu-id="17e6f-129">Tokens are sensitive data, because they grant access to a user's resources.</span></span> <span data-ttu-id="17e6f-130">（而且，令牌与用户密码不同，不能仅存储令牌的哈希。）因此，保护令牌不被盗用至关重要。</span><span class="sxs-lookup"><span data-stu-id="17e6f-130">(Moreover, unlike a user's password, you can't just store a hash of the token.) Therefore, it's critical to protect tokens from being compromised.</span></span> <span data-ttu-id="17e6f-131">Redis 支持的缓存受密码保护，但如果有人获取了密码，他便可以获取所有已缓存的访问令牌。</span><span class="sxs-lookup"><span data-stu-id="17e6f-131">The Redis-backed cache is protected by a password, but if someone obtains the password, they could get all of the cached access tokens.</span></span> <span data-ttu-id="17e6f-132">为此，`DistributedTokenCache` 对所有写入后备存储的信息进行加密。</span><span class="sxs-lookup"><span data-stu-id="17e6f-132">For that reason, the `DistributedTokenCache` encrypts everything that it writes to the backing store.</span></span> <span data-ttu-id="17e6f-133">加密操作是通过 ASP.NET Core [数据保护][data-protection] API 完成的。</span><span class="sxs-lookup"><span data-stu-id="17e6f-133">Encryption is done using the ASP.NET Core [data protection][data-protection] APIs.</span></span>

> [!NOTE]
> <span data-ttu-id="17e6f-134">如果部署到 Azure 网站，会将加密密钥备份到网络存储，并同步到所有计算机（请参阅[密钥管理和生存期][key-management]）。</span><span class="sxs-lookup"><span data-stu-id="17e6f-134">If you deploy to Azure Web Sites, the encryption keys are backed up to network storage and synchronized across all machines (see [Key management and lifetime][key-management]).</span></span> <span data-ttu-id="17e6f-135">默认情况下，密钥在 Azure 网站中运行时未加密，但可[使用 X.509 证书][x509-cert-encryption]启用加密。</span><span class="sxs-lookup"><span data-stu-id="17e6f-135">By default, keys are not encrypted when running in Azure Web Sites, but you can [enable encryption using an X.509 certificate][x509-cert-encryption].</span></span>
> 
> 

## <a name="distributedtokencache-implementation"></a><span data-ttu-id="17e6f-136">DistributedTokenCache 实现</span><span class="sxs-lookup"><span data-stu-id="17e6f-136">DistributedTokenCache implementation</span></span>
<span data-ttu-id="17e6f-137">`DistributedTokenCache` 类派生自 ADAL [TokenCache][tokencache-class] 类。</span><span class="sxs-lookup"><span data-stu-id="17e6f-137">The `DistributedTokenCache` class derives from the ADAL [TokenCache][tokencache-class] class.</span></span>

<span data-ttu-id="17e6f-138">在构造函数中，`DistributedTokenCache` 类创建当前用户的密钥，并从后备存储中加载缓存：</span><span class="sxs-lookup"><span data-stu-id="17e6f-138">In the constructor, the `DistributedTokenCache` class creates a key for the current user and loads the cache from the backing store:</span></span>

```csharp
public DistributedTokenCache(
    ClaimsPrincipal claimsPrincipal,
    IDistributedCache distributedCache,
    ILoggerFactory loggerFactory,
    IDataProtectionProvider dataProtectionProvider)
    : base()
{
    _claimsPrincipal = claimsPrincipal;
    _cacheKey = BuildCacheKey(_claimsPrincipal);
    _distributedCache = distributedCache;
    _logger = loggerFactory.CreateLogger<DistributedTokenCache>();
    _protector = dataProtectionProvider.CreateProtector(typeof(DistributedTokenCache).FullName);
    AfterAccess = AfterAccessNotification;
    LoadFromCache();
}
```

<span data-ttu-id="17e6f-139">通过连接用户 ID 和客户端 ID 创建密钥。</span><span class="sxs-lookup"><span data-stu-id="17e6f-139">The key is created by concatenating the user ID and client ID.</span></span> <span data-ttu-id="17e6f-140">两者都来自用户的 `ClaimsPrincipal` 中的声明：</span><span class="sxs-lookup"><span data-stu-id="17e6f-140">Both of these are taken from claims found in the user's `ClaimsPrincipal`:</span></span>

```csharp
private static string BuildCacheKey(ClaimsPrincipal claimsPrincipal)
{
    string clientId = claimsPrincipal.FindFirstValue("aud", true);
    return string.Format(
        "UserId:{0}::ClientId:{1}",
        claimsPrincipal.GetObjectIdentifierValue(),
        clientId);
}
```

<span data-ttu-id="17e6f-141">若要加载缓存数据，请从后备存储中读取序列化 blob，并调用 `TokenCache.Deserialize`，将 blob 转换为缓存数据。</span><span class="sxs-lookup"><span data-stu-id="17e6f-141">To load the cache data, read the serialized blob from the backing store, and call `TokenCache.Deserialize` to convert the blob into cache data.</span></span>

```csharp
private void LoadFromCache()
{
    byte[] cacheData = _distributedCache.Get(_cacheKey);
    if (cacheData != null)
    {
        this.Deserialize(_protector.Unprotect(cacheData));
    }
}
```

<span data-ttu-id="17e6f-142">每当 ADAL 访问此缓存，会引发 `AfterAccess` 事件。</span><span class="sxs-lookup"><span data-stu-id="17e6f-142">Whenever ADAL access the cache, it fires an `AfterAccess` event.</span></span> <span data-ttu-id="17e6f-143">如果缓存数据已更改，则 `HasStateChanged` 属性为 true。</span><span class="sxs-lookup"><span data-stu-id="17e6f-143">If the cache data has changed, the `HasStateChanged` property is true.</span></span> <span data-ttu-id="17e6f-144">在这种情况下，更新后备存储以反映更改，并将 `HasStateChanged` 设置为 false。</span><span class="sxs-lookup"><span data-stu-id="17e6f-144">In that case, update the backing store to reflect the change, and then set `HasStateChanged` to false.</span></span>

```csharp
public void AfterAccessNotification(TokenCacheNotificationArgs args)
{
    if (this.HasStateChanged)
    {
        try
        {
            if (this.Count > 0)
            {
                _distributedCache.Set(_cacheKey, _protector.Protect(this.Serialize()));
            }
            else
            {
                // There are no tokens for this user/client, so remove the item from the cache.
                _distributedCache.Remove(_cacheKey);
            }
            this.HasStateChanged = false;
        }
        catch (Exception exp)
        {
            _logger.WriteToCacheFailed(exp);
            throw;
        }
    }
}
```

<span data-ttu-id="17e6f-145">TokenCache 发送其他两个事件：</span><span class="sxs-lookup"><span data-stu-id="17e6f-145">TokenCache sends two other events:</span></span>

* <span data-ttu-id="17e6f-146">`BeforeWrite`。</span><span class="sxs-lookup"><span data-stu-id="17e6f-146">`BeforeWrite`.</span></span> <span data-ttu-id="17e6f-147">ADAL 写入缓存前会立即进行调用。</span><span class="sxs-lookup"><span data-stu-id="17e6f-147">Called immediately before ADAL writes to the cache.</span></span> <span data-ttu-id="17e6f-148">可将其用于实施并发策略</span><span class="sxs-lookup"><span data-stu-id="17e6f-148">You can use this to implement a concurrency strategy</span></span>
* <span data-ttu-id="17e6f-149">`BeforeAccess`。</span><span class="sxs-lookup"><span data-stu-id="17e6f-149">`BeforeAccess`.</span></span> <span data-ttu-id="17e6f-150">ADAL 从缓存中读取前，会立即进行调用。</span><span class="sxs-lookup"><span data-stu-id="17e6f-150">Called immediately before ADAL reads from the cache.</span></span> <span data-ttu-id="17e6f-151">此时可重新加载缓存，以获取最新版本。</span><span class="sxs-lookup"><span data-stu-id="17e6f-151">Here you can reload the cache to get the latest version.</span></span>

<span data-ttu-id="17e6f-152">在本例中，我们不处理这两个事件。</span><span class="sxs-lookup"><span data-stu-id="17e6f-152">In our case, we decided not to handle these two events.</span></span>

* <span data-ttu-id="17e6f-153">对于并发，以最后一次写入为准。</span><span class="sxs-lookup"><span data-stu-id="17e6f-153">For concurrency, last write wins.</span></span> <span data-ttu-id="17e6f-154">这是没问题的，因为令牌是为每个“用户 + 客户端”组合独立存储的，所以只有当同一个用户有两个并发登录会话时才会发生冲突。</span><span class="sxs-lookup"><span data-stu-id="17e6f-154">That's OK, because tokens are stored independently for each user + client, so a conflict would only happen if the same user had two concurrent login sessions.</span></span>
* <span data-ttu-id="17e6f-155">对于读取，我们会针对每个请求加载缓存。</span><span class="sxs-lookup"><span data-stu-id="17e6f-155">For reading, we load the cache on every request.</span></span> <span data-ttu-id="17e6f-156">请求的生存期较短。</span><span class="sxs-lookup"><span data-stu-id="17e6f-156">Requests are short lived.</span></span> <span data-ttu-id="17e6f-157">如果在请求生存期内修改缓存，下一个请求会选取新值。</span><span class="sxs-lookup"><span data-stu-id="17e6f-157">If the cache gets modified in that time, the next request will pick up the new value.</span></span>

<span data-ttu-id="17e6f-158">[**下一篇**][client-assertion]</span><span class="sxs-lookup"><span data-stu-id="17e6f-158">[**Next**][client-assertion]</span></span>

<!-- links -->
[ADAL]: https://msdn.microsoft.com/library/azure/jj573266.aspx
[client-assertion]: ./client-assertion.md
[data-protection]: /aspnet/core/security/data-protection/
[distributed-cache]: /aspnet/core/performance/caching/distributed
[key-management]: /aspnet/core/security/data-protection/configuration/default-settings
[in-memory-cache]: /aspnet/core/performance/caching/memory
[tokencache-class]: https://msdn.microsoft.com/library/azure/microsoft.identitymodel.clients.activedirectory.tokencache.aspx
[x509-cert-encryption]: /aspnet/core/security/data-protection/implementation/key-encryption-at-rest#x509-certificate
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
