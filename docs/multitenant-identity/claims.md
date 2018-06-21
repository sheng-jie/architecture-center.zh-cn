---
title: 在多租户应用程序中使用基于声明的标识
description: 如何使用声明进行颁发者验证和授权
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
ms.locfileid: "24541907"
---
# <a name="work-with-claims-based-identities"></a>使用基于声明的标识

[![GitHub](../_images/github.png) 示例代码][sample application]

## <a name="claims-in-azure-ad"></a>Azure AD 中的声明
当用户登录时，Azure AD 会发送一个 ID 令牌，其中包含有关该用户的声明集。 声明只需是一段以键值对形式表示的信息。 例如，`email`=`bob@contoso.com`。  声明包含一个颁发者 &mdash; 在本例中为 Azure AD &mdash; 即用于验证用户身份和创建声明的实体。 由于你信任该颁发者，因此也就信任这些声明。 （相反，如果你不信任颁发者，则不信任声明！）

在高级别中：

1. 用户进行身份验证。
2. IDP 发送声明集。
3. 应用规范化或补充声明（可选）。
4. 应用使用声明做出授权决策。

在 OpenID Connect 中，收到的声明集由身份验证请求的[范围参数]控制。 但是，Azure AD 会通过 OpenID Connect 颁发受限的声明集；请参阅[支持的令牌和声明类型]。 要了解有关用户的详细信息，需要使用 Azure AD 图形 API。

下面是 AAD 发送的、可能受应用关注的一些声明：

| ID 令牌中的声明类型 | 说明 |
| --- | --- |
| aud |令牌的颁发对象。 此值将是应用程序的客户端 ID。 一般情况下，应该不需要考虑此声明，因为中间件会自动验证它。 示例：`"91464657-d17a-4327-91f3-2ed99386406f"` |
| 组 |用户所属的 AAD 组的列表。 示例：`["93e8f556-8661-4955-87b6-890bc043c30f", "fc781505-18ef-4a31-a7d5-7d931d7b857e"]` |
| iss |OIDC 令牌的[颁发者]。 示例：`https://sts.windows.net/b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4/` |
| 名称 |用户的显示名称。 示例：`"Alice A."` |
| oid |AAD 中用户的对象标识符。 此值是用户的不可变且不可重用的标识符。 请使用此值而不是电子邮件作为用户的唯一标识符；电子邮件地址可能会更改。 如果在应用中使用 Azure AD 图形 API，对象 ID 是用于查询配置文件信息的值。 示例：`"59f9d2dc-995a-4ddf-915e-b3bb314a7fa4"` |
| 角色 |用户的应用角色列表。    示例：`["SurveyCreator"]` |
| tid |租户 ID。 此值是 Azure AD 中租户的唯一标识符。 示例：`"b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4"` |
| unique_name |用户的人工可读显示名称。 示例：`"alice@contoso.com"` |
| upn |用户主体名称。 示例：`"alice@contoso.com"` |

此表列出了 ID 令牌中显示的声明类型。 在 ASP.NET Core 中，OpenID Connect 中间件会在填充用户主体的声明集合时转换某些声明类型：

* oid > `http://schemas.microsoft.com/identity/claims/objectidentifier`
* tid > `http://schemas.microsoft.com/identity/claims/tenantid`
* unique_name > `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`
* upn > `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn`

## <a name="claims-transformations"></a>声明转换
在执行身份验证流期间，可能需要修改从 IDP 获取的声明。 在 ASP.NET Core 中，可以通过 OpenID Connect 中间件在 **AuthenticationValidated** 事件的内部执行声明转换。 （请参阅[身份验证事件]。）

在执行 **AuthenticationValidated** 期间添加的所有声明会存储在会话身份验证 Cookie 中。 这些声明不会推回到 Azure AD。

下面是声明转换的一些示例：

* **声明规范化**，或者在用户间保持声明一致。 这尤其适用于从多个 IDP 获取声明的情况，此时，可能对类似的信息使用了不同的声明类型。
  例如，Azure AD 会发送一个包含用户电子邮件的“upn”声明。 其他 IDP 可能发送“email”声明。 以下代码将“upn”声明转换为“email”声明：
  
  ```csharp
  var email = principal.FindFirst(ClaimTypes.Upn)?.Value;
  if (!string.IsNullOrWhiteSpace(email))
  {
      identity.AddClaim(new Claim(ClaimTypes.Email, email));
  }
  ```
* 为不存在的声明添加**默认声明值** &mdash; 例如，将用户分配到默认角色。 在某些情况下，这可以简化授权逻辑。
* 添加包含有关用户的应用程序特定信息的**自定义声明类型**。 例如，可以在数据库中存储有关用户的某些信息。 可将包含此信息的自定义声明添加到身份验证票证。 声明存储在 Cookie 中，因此，只需在每个登录会话中从数据库获取该声明一次。 另一方面，我们还希望避免创建过大的 Cookie，因此，需考虑在 Cookie 大小与数据库查找量之间做出取舍。   

完成身份验证流之后，`HttpContext.User` 中会提供声明。 此时，应将这些声明视为只读的集合 &mdash; 例如，使用它们做出授权决策。

## <a name="issuer-validation"></a>颁发者验证
在 OpenID Connect 中，颁发者声明（“iss”）标识颁发 ID 令牌的 IDP。 OIDC 身份验证流的部分工作是验证颁发者声明是否与实际颁发者匹配。 OIDC 中间件会自动处理此验证。

在 Azure AD 中，颁发者值是每个 AD 租户的唯一值 (`https://sts.windows.net/<tenantID>`)。 因此，应用程序应该执行额外的检查，确保颁发者代表有权登录到应用的租户。

对于单租户应用程序，只需检查颁发者是否为你自己的租户。 事实上，OIDC 中间件默认会自动执行此检查。 在多租户应用中，需要允许多个对应于不同租户的颁发者。 下面是一种普通用法：

* 在 OIDC 中间件选项中，将 **ValidateIssuer** 设置为 false。 这会关闭自动检查。
* 当某个租户注册时，会在用户数据库中存储该租户和颁发者。
* 每当用户登录时，会数据库中查找颁发者。 如果找不到该颁发者，则表示租户尚未注册。 可将他们重定向到注册页。
* 还可以将某些租户（例如，未支付订阅费的客户）加入方块列表。

有关更多详细介绍，请参阅[在多租户应用程序中注册和加入租户][signup]。

## <a name="using-claims-for-authorization"></a>使用声明进行授权
使用声明时，用户的标识不再是单一实体。 例如，用户可能具有电子邮件地址、电话号码、生日、性别等信息。所有这些信息可能存储在用户的 IDP 中。 但是，对用户进行身份验证时，通常会以声明的形式收到其中的一部分信息。 在此模型中，用户的标识只是一堆声明。 在针对用户做出授权决策时，会查找特定的声明集。 换而言之，问题“用户 X 是否可以执行操作 Y”最终变成了“用户 X 是否具有声明 Z”。

下面是检查声明的一些基本模式。

* 若要检查用户是否具有包含特定值的特定声明：
  
   ```csharp
   if (User.HasClaim(ClaimTypes.Role, "Admin")) { ... }
   ```
   此代码检查用户是否具有包含值“Admin”的 Role 声明。 如果用户没有 Role 声明或者具有多个 Role 声明，此代码不会做出正确的处理。
  
   **ClaimTypes** 类定义常用声明类型的常量。 但是，可对声明类型使用任何字符串值。
* 当你认为某个声明类型最多只有一个值时，若要获取该声明类型的单个值，请使用以下代码：
  
  ```csharp
  string email = User.FindFirst(ClaimTypes.Email)?.Value;
  ```
* 若要获取某个声明类型的所有值，请使用以下代码：
  
  ```csharp
  IEnumerable<Claim> groups = User.FindAll("groups");
  ```

有关详细信息，请参阅[多租户应用程序中基于角色和基于资源的授权][authorization]。

[**下一篇**][signup]


<!-- Links -->

[范围参数]: http://nat.sakimura.org/2012/01/26/scopes-and-claims-in-openid-connect/
[支持的令牌和声明类型]: /azure/active-directory/active-directory-token-and-claims/
[颁发者]: http://openid.net/specs/openid-connect-core-1_0.html#IDToken
[身份验证事件]: authenticate.md#authentication-events
[signup]: signup.md
[Claims-Based Authorization]: /aspnet/core/security/authorization/claims
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[authorization]: authorize.md
