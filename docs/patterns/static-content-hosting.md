---
title: 静态内容托管
description: 将静态内容部署到基于云的存储服务，再由后者将它们直接传送给客户端。
keywords: 设计模式
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- design-implementation
- performance-scalability
ms.openlocfilehash: deb15001bea2598d56a2793be78bbc3e7473bdf3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="static-content-hosting-pattern"></a><span data-ttu-id="4fd3b-104">静态内容托管模式</span><span class="sxs-lookup"><span data-stu-id="4fd3b-104">Static Content Hosting pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="4fd3b-105">将静态内容部署到基于云的存储服务，再由后者将它们直接传送给客户端。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-105">Deploy static content to a cloud-based storage service that can deliver them directly to the client.</span></span> <span data-ttu-id="4fd3b-106">这样可降低对可能很昂贵的计算实例的需求。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-106">This can reduce the need for potentially expensive compute instances.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="4fd3b-107">上下文和问题</span><span class="sxs-lookup"><span data-stu-id="4fd3b-107">Context and problem</span></span>

<span data-ttu-id="4fd3b-108">Web 应用程序通常包括某些静态内容元素。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-108">Web applications typically include some elements of static content.</span></span> <span data-ttu-id="4fd3b-109">此静态内容可能包括 HTML 页面和其他资源，例如可供客户端使用的图像和文档，以 HTML 页面内容（例如内联图像、样式表、客户端 JavaScript 文件）或独立下载项目（例如 PDF 文档）的形式提供。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-109">This static content might include HTML pages and other resources such as images and documents that are available to the client, either as part of an HTML page (such as inline images, style sheets, and client-side JavaScript files) or as separate downloads (such as PDF documents).</span></span>

<span data-ttu-id="4fd3b-110">Web 服务器已经过优化，可以通过高效且动态地执行页面代码以及对输出进行缓存来优化请求，但仍需处理下载静态内容的请求。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-110">Although web servers are well tuned to optimize requests through efficient dynamic page code execution and output caching, they still have to handle requests to download static content.</span></span> <span data-ttu-id="4fd3b-111">这样会占用那些通常可以有更好用途的处理周期。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-111">This consumes processing cycles that could often be put to better use.</span></span>

## <a name="solution"></a><span data-ttu-id="4fd3b-112">解决方案</span><span class="sxs-lookup"><span data-stu-id="4fd3b-112">Solution</span></span>

<span data-ttu-id="4fd3b-113">在大多数云托管环境中，可以将应用程序的部分资源和静态页面放置到存储服务中，尽量降低对计算实例的需求（例如，使用较小或较少的实例）。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-113">In most cloud hosting environments it's possible to minimize the need for compute instances (for example, use a smaller instance or fewer instances), by locating some of an application’s resources and static pages in a storage service.</span></span> <span data-ttu-id="4fd3b-114">基于云的存储相对于计算实例来说，其成本通常要低得多。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-114">The cost for cloud-hosted storage is typically much less than for compute instances.</span></span>

<span data-ttu-id="4fd3b-115">将应用程序的一些部件托管在存储服务中时，主要考虑因素涉及应用程序的部署，以及保护那些不应提供给匿名用户的资源。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-115">When hosting some parts of an application in a storage service, the main considerations are related to deployment of the application and to securing resources that aren't intended to be available to anonymous users.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="4fd3b-116">问题和注意事项</span><span class="sxs-lookup"><span data-stu-id="4fd3b-116">Issues and considerations</span></span>

<span data-ttu-id="4fd3b-117">在决定如何实现此模式时，请考虑以下几点：</span><span class="sxs-lookup"><span data-stu-id="4fd3b-117">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="4fd3b-118">托管的存储服务必须公开一个 HTTP 终结点，供下载静态资源的用户访问。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-118">The hosted storage service must expose an HTTP endpoint that users can access to download the static resources.</span></span> <span data-ttu-id="4fd3b-119">某些存储服务也支持 HTTPS，因此可以将资源托管在需要 SSL 的存储服务中。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-119">Some storage services also support HTTPS, so it's possible to host resources in storage services that require SSL.</span></span>

- <span data-ttu-id="4fd3b-120">为了尽量提高性能和可用性，可考虑使用内容交付网络 (CDN)，将存储容器的内容缓存在全球的多个数据中心。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-120">For maximum performance and availability, consider using a content delivery network (CDN) to cache the contents of the storage container in multiple datacenters around the world.</span></span> <span data-ttu-id="4fd3b-121">但是，可能需要支付 CDN 使用费用。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-121">However, you'll likely have to pay for using the CDN.</span></span>

- <span data-ttu-id="4fd3b-122">通常情况下，存储帐户会默认进行异地复制，确保在发生可能影响某个数据中心的事件时能够复原。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-122">Storage accounts are often geo-replicated by default to provide resiliency against events that might affect a datacenter.</span></span> <span data-ttu-id="4fd3b-123">这意味着，IP 地址可能会变，但 URL 会保持不变。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-123">This means that the IP address might change, but the URL will remain the same.</span></span>

- <span data-ttu-id="4fd3b-124">当某些内容位于存储帐户中，而另一些内容却位于托管计算实例中时，部署和更新应用程序会变得更具挑战性。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-124">When some content is located in a storage account and other content is in a hosted compute instance it becomes more challenging to deploy an application and to update it.</span></span> <span data-ttu-id="4fd3b-125">可能需要执行单独的部署，并对应用程序和内容进行版本控制以方便管理，尤其是在静态内容包含脚本文件或 UI 组件的情况下。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-125">You might have to perform separate deployments, and version the application and content to manage it more easily&mdash;especially when the static content includes script files or UI components.</span></span> <span data-ttu-id="4fd3b-126">但是，如果只需更新静态资源，则可直接将其上传到存储帐户，不需重新部署应用程序包。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-126">However, if only static resources have to be updated, they can simply be uploaded to the storage account without needing to redeploy the application package.</span></span>

- <span data-ttu-id="4fd3b-127">存储服务可能不支持使用自定义域名。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-127">Storage services might not support the use of custom domain names.</span></span> <span data-ttu-id="4fd3b-128">在这种情况下，需在链接中指定资源的完整 URL，因为其所在的域不同于动态生成的包含链接的内容所在的域。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-128">In this case it's necessary to specify the full URL of the resources in links because they'll be in a different domain from the dynamically-generated content containing the links.</span></span>

- <span data-ttu-id="4fd3b-129">必须将存储容器配置为允许公开读取访问，但不得将其配置为允许公开写入访问，防止用户上传内容。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-129">The storage containers must be configured for public read access, but it's vital to ensure that they aren't configured for public write access to prevent users being able to upload content.</span></span> <span data-ttu-id="4fd3b-130">考虑使用辅助密钥或令牌来控制不允许匿名使用的资源的访问。有关详细信息，请参阅[辅助密钥模式](valet-key.md)。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-130">Consider using a valet key or token to control access to resources that shouldn't be available anonymously&mdash;see the [Valet Key pattern](valet-key.md) for more information.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="4fd3b-131">何时使用此模式</span><span class="sxs-lookup"><span data-stu-id="4fd3b-131">When to use this pattern</span></span>

<span data-ttu-id="4fd3b-132">此模式适合用于：</span><span class="sxs-lookup"><span data-stu-id="4fd3b-132">This pattern is useful for:</span></span>

- <span data-ttu-id="4fd3b-133">尽量降低包含了一些静态资源的网站和应用程序的托管成本。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-133">Minimizing the hosting cost for websites and applications that contain some static resources.</span></span>

- <span data-ttu-id="4fd3b-134">尽量降低只包含静态内容和资源的网站的托管成本。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-134">Minimizing the hosting cost for websites that consist of only static content and resources.</span></span> <span data-ttu-id="4fd3b-135">如果托管提供商的存储系统允许，可以将全静态网站整个托管在存储帐户中。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-135">Depending on the capabilities of the hosting provider’s storage system, it might be possible to entirely host a fully static website in a storage account.</span></span>

- <span data-ttu-id="4fd3b-136">公开在其他托管环境或本地服务器上运行的应用程序的静态资源和内容。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-136">Exposing static resources and content for applications running in other hosting environments or on-premises servers.</span></span>

- <span data-ttu-id="4fd3b-137">通过内容交付网络将内容置于多个地理区域，该网络将存储帐户的内容缓存在全球的多个数据中心。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-137">Locating content in more than one geographical area using a content delivery network that caches the contents of the storage account in multiple datacenters around the world.</span></span>

- <span data-ttu-id="4fd3b-138">监视成本和带宽使用情况。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-138">Monitoring costs and bandwidth usage.</span></span> <span data-ttu-id="4fd3b-139">使用单独的存储帐户来存储部分或全部静态内容，这样可以更容易地将此方面的成本与托管和运行时成本区分开来。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-139">Using a separate storage account for some or all of the static content allows the costs to be more easily separated from hosting and runtime costs.</span></span>

<span data-ttu-id="4fd3b-140">此模式在以下情况中可能不起作用：</span><span class="sxs-lookup"><span data-stu-id="4fd3b-140">This pattern might not be useful in the following situations:</span></span>

- <span data-ttu-id="4fd3b-141">应用程序需对静态内容进行某些处理，然后才能将其交付给客户端。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-141">The application needs to perform some processing on the static content before delivering it to the client.</span></span> <span data-ttu-id="4fd3b-142">例如，可能需要向文档添加时间戳。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-142">For example, it might be necessary to add a timestamp to a document.</span></span>

- <span data-ttu-id="4fd3b-143">静态内容的量很小。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-143">The volume of static content is very small.</span></span> <span data-ttu-id="4fd3b-144">从单独的存储检索此内容的开销可能超过将其与计算资源分开获得的成本优势。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-144">The overhead of retrieving this content from separate storage can outweigh the cost benefit of separating it out from the compute resource.</span></span>

## <a name="example"></a><span data-ttu-id="4fd3b-145">示例</span><span class="sxs-lookup"><span data-stu-id="4fd3b-145">Example</span></span>

<span data-ttu-id="4fd3b-146">位于 Azure Blob 存储中的静态内容可以直接通过 Web 浏览器来访问。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-146">Static content located in Azure Blob storage can be accessed directly by a web browser.</span></span> <span data-ttu-id="4fd3b-147">对于可以公开给客户端的存储，Azure 提供基于 HTTP 的界面。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-147">Azure provides an HTTP-based interface over storage that can be publicly exposed to clients.</span></span> <span data-ttu-id="4fd3b-148">例如，Azure Blob 存储容器中的内容使用以下形式的 URL 公开：</span><span class="sxs-lookup"><span data-stu-id="4fd3b-148">For example, content in an Azure Blob storage container is exposed using a URL with the following form:</span></span>

`http://[ storage-account-name ].blob.core.windows.net/[ container-name ]/[ file-name ]`


<span data-ttu-id="4fd3b-149">上传内容时，需创建一个或多个 Blob 容器来存储文件和文档。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-149">When uploading the content it's necessary to create one or more blob containers to hold the files and documents.</span></span> <span data-ttu-id="4fd3b-150">请注意，新容器的默认权限为“专用”，必须将其更改为“公用”，然后客户端才能访问其中的内容。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-150">Note that the default permission for a new container is Private, and you must change this to Public to allow clients to access the contents.</span></span> <span data-ttu-id="4fd3b-151">如果必须防止内容被匿名访问，可以实施[辅助密钥模式](valet-key.md)，使用户必须提供有效的令牌才能下载资源。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-151">If it's necessary to protect the content from anonymous access, you can implement the [Valet Key pattern](valet-key.md) so users must present a valid token to download the resources.</span></span>

> <span data-ttu-id="4fd3b-152">[Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx)（Blob 服务概念）介绍了 Blob 存储及其访问和使用方式。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-152">[Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx) has information about blob storage, and the ways that you can access and use it.</span></span>

<span data-ttu-id="4fd3b-153">每页中的链接会指定资源的 URL，客户端可以直接从存储服务对其进行访问。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-153">The links in each page will specify the URL of the resource and the client will access it directly from the storage service.</span></span> <span data-ttu-id="4fd3b-154">下图说明了如何直接从存储服务交付应用程序静态部分的内容。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-154">The figure illustrates delivering static parts of an application directly from a storage service.</span></span>

![图 1 - 直接从存储服务交付应用程序静态部分的内容](./_images/static-content-hosting-pattern.png)


<span data-ttu-id="4fd3b-156">交付给客户端的页面中的链接必须指定 Blob 容器和资源的完整 URL。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-156">The links in the pages delivered to the client must specify the full URL of the blob container and resource.</span></span> <span data-ttu-id="4fd3b-157">例如，包含公共容器中图像的链接的页面可能包含以下 HTML。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-157">For example, a page that contains a link to an image in a public container might contain the following HTML.</span></span>

```html
<img src="http://mystorageaccount.blob.core.windows.net/myresources/image1.png"
     alt="My image" />
```

> <span data-ttu-id="4fd3b-158">如果通过辅助密钥（例如 Azure 共享访问签名）对资源进行保护，则链接的 URL 中必须包含该签名。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-158">If the resources are protected by using a valet key, such as an Azure shared access signature, this signature must be included in the URLs in the links.</span></span>

<span data-ttu-id="4fd3b-159">[GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting) 中提供了一个名为 StaticContentHosting 的解决方案，演示了如何使用外部存储来存储静态资源。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-159">A solution named StaticContentHosting that demonstrates using external storage for static resources is available from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span></span> <span data-ttu-id="4fd3b-160">StaticContentHosting.Cloud 项目包含的配置文件指定了用于存储静态内容的存储帐户和容器。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-160">The StaticContentHosting.Cloud project contains configuration files that specify the storage account and container that holds the static content.</span></span>

```xml
<Setting name="StaticContent.StorageConnectionString"
         value="UseDevelopmentStorage=true" />
<Setting name="StaticContent.Container" value="static-content" />
```

<span data-ttu-id="4fd3b-161">StaticContentHosting.Web 项目的 Settings.cs 文件中的 `Settings` 类包含用于提取这些值并生成一个字符串值（其中包含云存储帐户容器 URL）的方法。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-161">The `Settings` class in the file Settings.cs of the StaticContentHosting.Web project contains methods to extract these values and build a string value containing the cloud storage account container URL.</span></span>

```csharp
public class Settings
{
  public static string StaticContentStorageConnectionString {
    get
    {
      return RoleEnvironment.GetConfigurationSettingValue(
                              "StaticContent.StorageConnectionString");
    }
  }

  public static string StaticContentContainer
  {
    get
    {
      return RoleEnvironment.GetConfigurationSettingValue("StaticContent.Container");
    }
  }

  public static string StaticContentBaseUrl
  {
    get
    {
      var account = CloudStorageAccount.Parse(StaticContentStorageConnectionString);

      return string.Format("{0}/{1}", account.BlobEndpoint.ToString().TrimEnd('/'),
                                      StaticContentContainer.TrimStart('/'));
    }
  }
}
```

<span data-ttu-id="4fd3b-162">StaticContentUrlHtmlHelper.cs 文件中的 `StaticContentUrlHtmlHelper` 类公开了一个名为 `StaticContentUrl` 的方法，该方法生成的 URL 包含云存储帐户的路径，但前提是传递给它的 URL 以 ASP.NET 根路径字符 (~) 开头。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-162">The `StaticContentUrlHtmlHelper` class in the file StaticContentUrlHtmlHelper.cs exposes a method named `StaticContentUrl` that generates a URL containing the path to the cloud storage account if the URL passed to it starts with the ASP.NET root path character (~).</span></span>

```csharp
public static class StaticContentUrlHtmlHelper
{
  public static string StaticContentUrl(this HtmlHelper helper, string contentPath)
  {
    if (contentPath.StartsWith("~"))
    {
      contentPath = contentPath.Substring(1);
    }

    contentPath = string.Format("{0}/{1}", Settings.StaticContentBaseUrl.TrimEnd('/'),
                                contentPath.TrimStart('/'));

    var url = new UrlHelper(helper.ViewContext.RequestContext);

    return url.Content(contentPath);
  }
}
```

<span data-ttu-id="4fd3b-163">Views\Home 文件夹中的 Index.cshtml 文件包含的图像元素使用 `StaticContentUrl` 方法为其 `src` 属性创建 URL。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-163">The file Index.cshtml in the Views\Home folder contains an image element that uses the `StaticContentUrl` method to create the URL for its `src` attribute.</span></span>

```html
<img src="@Html.StaticContentUrl("~/media/orderedList1.png")" alt="Test Image" />
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="4fd3b-164">相关模式和指南</span><span class="sxs-lookup"><span data-stu-id="4fd3b-164">Related patterns and guidance</span></span>

- <span data-ttu-id="4fd3b-165">演示此模式的示例可在 [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting) 上找到。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-165">A sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span></span>
- <span data-ttu-id="4fd3b-166">[辅助密钥模式](valet-key.md)。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-166">[Valet Key pattern](valet-key.md).</span></span> <span data-ttu-id="4fd3b-167">如果不应将目标资源提供给匿名用户，则需对静态内容所在的存储实施安全措施。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-167">If the target resources aren't supposed to be available to anonymous users it's necessary to implement security over the store that holds the static content.</span></span> <span data-ttu-id="4fd3b-168">描述如何使用令牌或密钥，让客户端以受限直接访问方式访问特定资源或服务（例如云托管存储服务）。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-168">Describes how to use a token or key that provides clients with restricted direct access to a specific resource or service such as a cloud-hosted storage service.</span></span>
- <span data-ttu-id="4fd3b-169">Infosys 博客：[An efficient way of deploying a static web site on Azure](http://www.infosysblogs.com/microsoft/2010/06/an_efficient_way_of_deploying.html)（在 Azure 中高效部署静态网站）。</span><span class="sxs-lookup"><span data-stu-id="4fd3b-169">[An efficient way of deploying a static web site on Azure](http://www.infosysblogs.com/microsoft/2010/06/an_efficient_way_of_deploying.html) on the Infosys blog.</span></span>
- <span data-ttu-id="4fd3b-170">[Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx)（Blob 服务概念）</span><span class="sxs-lookup"><span data-stu-id="4fd3b-170">[Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx)</span></span>
