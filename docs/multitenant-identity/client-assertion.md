---
title: "使用客户端断言从 Azure AD 获取访问令牌"
description: "如何使用客户端断言从 Azure AD 获取访问令牌。"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: adfs
pnp.series.next: key-vault
ms.openlocfilehash: 9fe1ee2ec5a540edc41c3a310476507f8d862f0c
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="use-client-assertion-to-get-access-tokens-from-azure-ad"></a><span data-ttu-id="bc080-103">使用客户端断言从 Azure AD 获取访问令牌</span><span class="sxs-lookup"><span data-stu-id="bc080-103">Use client assertion to get access tokens from Azure AD</span></span>

<span data-ttu-id="bc080-104">[![GitHub](../_images/github.png) 示例代码][sample application]</span><span class="sxs-lookup"><span data-stu-id="bc080-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

## <a name="background"></a><span data-ttu-id="bc080-105">背景</span><span class="sxs-lookup"><span data-stu-id="bc080-105">Background</span></span>
<span data-ttu-id="bc080-106">在 OpenID Connect 中使用授权代码流或混合流时，客户端交换授权代码来获取访问令牌。</span><span class="sxs-lookup"><span data-stu-id="bc080-106">When using authorization code flow or hybrid flow in OpenID Connect, the client exchanges an authorization code for an access token.</span></span> <span data-ttu-id="bc080-107">在此步骤中，客户端必须向服务器进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="bc080-107">During this step, the client has to authenticate itself to the server.</span></span>

![客户端机密](./images/client-secret.png)

<span data-ttu-id="bc080-109">客户端进行身份验证的一种方法是使用客户端密码。</span><span class="sxs-lookup"><span data-stu-id="bc080-109">One way to authenticate the client is by using a client secret.</span></span> <span data-ttu-id="bc080-110">默认情况下，配置 [Tailspin Surveys][Surveys] 应用程序也是采用相同方式。</span><span class="sxs-lookup"><span data-stu-id="bc080-110">That's how the [Tailspin Surveys][Surveys] application is configured by default.</span></span>

<span data-ttu-id="bc080-111">下面是从客户端到 IDP 请求访问令牌的示例请求。</span><span class="sxs-lookup"><span data-stu-id="bc080-111">Here is an example request from the client to the IDP, requesting an access token.</span></span> <span data-ttu-id="bc080-112">请注意 `client_secret` 参数。</span><span class="sxs-lookup"><span data-stu-id="bc080-112">Note the `client_secret` parameter.</span></span>

```
POST https://login.microsoftonline.com/b9bd2162xxx/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

resource=https://tailspin.onmicrosoft.com/surveys.webapi
  &client_id=87df91dc-63de-4765-8701-b59cc8bd9e11
  &client_secret=i3Bf12Dn...
  &grant_type=authorization_code
  &code=PG8wJG6Y...
```

<span data-ttu-id="bc080-113">密码只是一个字符串，所以必须确保不会泄漏该值。</span><span class="sxs-lookup"><span data-stu-id="bc080-113">The secret is just a string, so you have to make sure not to leak the value.</span></span> <span data-ttu-id="bc080-114">最佳做法是将客户端密码排除在源代码管理之外。</span><span class="sxs-lookup"><span data-stu-id="bc080-114">The best practice is to keep the client secret out of source control.</span></span> <span data-ttu-id="bc080-115">在部署到 Azure 时，请将密码存储在[应用设置][configure-web-app]中。</span><span class="sxs-lookup"><span data-stu-id="bc080-115">When you deploy to Azure, store the secret in an [app setting][configure-web-app].</span></span>

<span data-ttu-id="bc080-116">但任何能够访问 Azure 订阅的人都可以查看应用设置。</span><span class="sxs-lookup"><span data-stu-id="bc080-116">However, anyone with access to the Azure subscription can view the app settings.</span></span> <span data-ttu-id="bc080-117">此外，密码始终倾向于签入到源代码管理中（例如，在部署脚本中），并通过电子邮件等进行共享。</span><span class="sxs-lookup"><span data-stu-id="bc080-117">Further, there is always a temptation to check secrets into source control (e.g., in deployment scripts), share them by email, and so on.</span></span>

<span data-ttu-id="bc080-118">为提高安全性，可使用[客户端断言]取代客户端密码。</span><span class="sxs-lookup"><span data-stu-id="bc080-118">For additional security, you can use [client assertion] instead of a client secret.</span></span> <span data-ttu-id="bc080-119">通过使用客户端断言，客户端可以使用 X.509 证书来证明令牌请求来自客户端。</span><span class="sxs-lookup"><span data-stu-id="bc080-119">With client assertion, the client uses an X.509 certificate to prove the token request came from the client.</span></span> <span data-ttu-id="bc080-120">客户端证书安装在 Web 服务器上。</span><span class="sxs-lookup"><span data-stu-id="bc080-120">The client certificate is installed on the web server.</span></span> <span data-ttu-id="bc080-121">通常情况下，限制对证书的访问比确保人们不会无意透露客户端密码更容易。</span><span class="sxs-lookup"><span data-stu-id="bc080-121">Generally, it will be easier to restrict access to the certificate, than to ensure that nobody inadvertently reveals a client secret.</span></span> <span data-ttu-id="bc080-122">有关在 Web 应用程序中配置证书的详细信息，请参阅[在 Azure 网站应用程序中使用证书][using-certs-in-websites]</span><span class="sxs-lookup"><span data-stu-id="bc080-122">For more information about configuring certificates in a web app, see [Using Certificates in Azure Websites Applications][using-certs-in-websites]</span></span>

<span data-ttu-id="bc080-123">下面是使用客户端断言的令牌请求：</span><span class="sxs-lookup"><span data-stu-id="bc080-123">Here is a token request using client assertion:</span></span>

```
POST https://login.microsoftonline.com/b9bd2162xxx/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

resource=https://tailspin.onmicrosoft.com/surveys.webapi
  &client_id=87df91dc-63de-4765-8701-b59cc8bd9e11
  &client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer
  &client_assertion=eyJhbGci...
  &grant_type=authorization_code
  &code= PG8wJG6Y...
```

<span data-ttu-id="bc080-124">请注意，`client_secret` 参数已不再使用。</span><span class="sxs-lookup"><span data-stu-id="bc080-124">Notice that the `client_secret` parameter is no longer used.</span></span> <span data-ttu-id="bc080-125">取而代之的是 `client_assertion` 参数，它包含一个使用客户端证书签名的 JWT 令牌。</span><span class="sxs-lookup"><span data-stu-id="bc080-125">Instead, the `client_assertion` parameter contains a JWT token that was signed using the client certificate.</span></span> <span data-ttu-id="bc080-126">`client_assertion_type` 参数指定断言类型 &mdash; 在本例中为 JWT 令牌。</span><span class="sxs-lookup"><span data-stu-id="bc080-126">The `client_assertion_type` parameter specifies the type of assertion &mdash; in this case, JWT token.</span></span> <span data-ttu-id="bc080-127">服务器验证 JWT 令牌。</span><span class="sxs-lookup"><span data-stu-id="bc080-127">The server validates the JWT token.</span></span> <span data-ttu-id="bc080-128">如果 JWT 令牌无效，令牌请求将返回一个错误。</span><span class="sxs-lookup"><span data-stu-id="bc080-128">If the JWT token is invalid, the token request returns an error.</span></span>

> [!NOTE]
> <span data-ttu-id="bc080-129">X.509 证书不是客户断言的唯一形式；我们在此关注它，因为它受 Azure AD 支持。</span><span class="sxs-lookup"><span data-stu-id="bc080-129">X.509 certificates are not the only form of client assertion; we focus on it here because it is supported by Azure AD.</span></span>
> 
> 

<span data-ttu-id="bc080-130">在运行时，Web 应用程序从证书存储中读取证书。</span><span class="sxs-lookup"><span data-stu-id="bc080-130">At run time, the web application reads the certificate from the certificate store.</span></span> <span data-ttu-id="bc080-131">必须在 Web 应用所在的同一计算机上安装证书。</span><span class="sxs-lookup"><span data-stu-id="bc080-131">The certificate must be installed on the same machine as the web app.</span></span>

<span data-ttu-id="bc080-132">Surveys 应用程序包括帮助程序类，该类创建了 [ClientAssertionCertificate](/dotnet/api/microsoft.identitymodel.clients.activedirectory.clientassertioncertificate)，可将其传递给 [AuthenticationContext.AcquireTokenSilentAsync](/dotnet/api/microsoft.identitymodel.clients.activedirectory.authenticationcontext.acquiretokensilentasync) 方法以从 Azure AD 获取令牌。</span><span class="sxs-lookup"><span data-stu-id="bc080-132">The Surveys application includes a helper class that creates a [ClientAssertionCertificate](/dotnet/api/microsoft.identitymodel.clients.activedirectory.clientassertioncertificate) that you can pass to the [AuthenticationContext.AcquireTokenSilentAsync](/dotnet/api/microsoft.identitymodel.clients.activedirectory.authenticationcontext.acquiretokensilentasync) method to acquire a token from Azure AD.</span></span>

```csharp
public class CertificateCredentialService : ICredentialService
{
    private Lazy<Task<AdalCredential>> _credential;

    public CertificateCredentialService(IOptions<ConfigurationOptions> options)
    {
        var aadOptions = options.Value?.AzureAd;
        _credential = new Lazy<Task<AdalCredential>>(() =>
        {
            X509Certificate2 cert = CertificateUtility.FindCertificateByThumbprint(
                aadOptions.Asymmetric.StoreName,
                aadOptions.Asymmetric.StoreLocation,
                aadOptions.Asymmetric.CertificateThumbprint,
                aadOptions.Asymmetric.ValidationRequired);
            string password = null;
            var certBytes = CertificateUtility.ExportCertificateWithPrivateKey(cert, out password);
            return Task.FromResult(new AdalCredential(new ClientAssertionCertificate(aadOptions.ClientId, new X509Certificate2(certBytes, password))));
        });
    }

    public async Task<AdalCredential> GetCredentialsAsync()
    {
        return await _credential.Value;
    }
}
```

<span data-ttu-id="bc080-133">有关在 Surveys 应用程序中设置客户端断言的信息，请参阅[使用 Azure Key Vault 保护应用程序机密][key vault]。</span><span class="sxs-lookup"><span data-stu-id="bc080-133">For information about setting up client assertion in the Surveys application, see [Use Azure Key Vault to protect application secrets ][key vault].</span></span>

<span data-ttu-id="bc080-134">[下一篇][key vault]</span><span class="sxs-lookup"><span data-stu-id="bc080-134">[**Next**][key vault]</span></span>

<!-- Links -->
[configure-web-app]: /azure/app-service-web/web-sites-configure/
[azure-management-portal]: https://portal.azure.com
[客户端断言]: https://tools.ietf.org/html/rfc7521
[key vault]: key-vault.md
[Setup-KeyVault]: https://github.com/mspnp/multitenant-saas-guidance/blob/master/scripts/Setup-KeyVault.ps1
[Surveys]: tailspin.md
[using-certs-in-websites]: https://azure.microsoft.com/blog/using-certificates-in-azure-websites-applications/

[sample application]: https://github.com/mspnp/multitenant-saas-guidance
