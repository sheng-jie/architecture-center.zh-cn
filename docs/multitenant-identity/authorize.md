---
title: 多租户应用程序中的授权
description: 如何在多租户应用程序中执行授权
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: app-roles
pnp.series.next: web-api
ms.openlocfilehash: 03c4d5fa10c75437a7b066534619ba9a123c350c
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/06/2018
---
# <a name="role-based-and-resource-based-authorization"></a>基于角色和基于资源的授权

[![GitHub](../_images/github.png) 示例代码][sample application]

[参考实现]是 ASP.NET Core 应用程序。 在本文中，我们将介绍使用 ASP.NET Core 中提供的授权 API 来进行授权的两种通用方法。

* **基于角色的授权**。 基于分配给用户的角色授权操作。 例如，某些操作需要管理员角色。
* **基于资源的授权**。 基于特定资源授权操作。 例如，每个资源都有一个所有者。 所有者可以删除资源；其他用户则不能。

一般的应用会结合使用这两种方法。 例如，若要删除资源，用户必须是资源所有者或管理员。

## <a name="role-based-authorization"></a>基于角色的授权
[Tailspin Surveys][Tailspin] 应用程序可定义以下角色：

* 管理员。 可在属于该租户的任何调查中执行所有 CRUD 操作。
* 创建者。 可创建新调查
* 读者。 可读取属于该租户的任何调查

角色应用于该应用程序的用户。 在 Surveys 应用程序中，用户可以是管理员、创建者或读者。

有关如何定义和管理角色的讨论，请参阅[应用程序角色]。

无论如何管理角色，授权代码看起来都类似。 ASP.NET Core 具有名为[授权策略][policies]的抽象。 使用此功能，可在代码中定义授权策略，然后将其应用于控制器操作。 策略与控制器分离。

### <a name="create-policies"></a>创建策略
若要定义策略，首先创建实现 `IAuthorizationRequirement` 的类。 从 `AuthorizationHandler` 中派生是最简单的方法。 在 `Handle` 方法中，请检查相关声明。

下面是 Tailspin Surveys 应用程序中的示例：

```csharp
public class SurveyCreatorRequirement : AuthorizationHandler<SurveyCreatorRequirement>, IAuthorizationRequirement
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, SurveyCreatorRequirement requirement)
    {
        if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyAdmin) || 
            context.User.HasClaim(ClaimTypes.Role, Roles.SurveyCreator))
        {
            context.Succeed(requirement);
        }
        return Task.FromResult(0);
    }
}
```

此类定义用户创建新调查的需求。 用户必须是 SurveyAdmin 或 SurveyCreator 角色。

在启动类中，定义包含一个或多个需求的命名策略。 如果有多个需求，用户必须满足每个需求才能得到授权。 下面的代码定义了两个策略：

```csharp
services.AddAuthorization(options =>
{
    options.AddPolicy(PolicyNames.RequireSurveyCreator,
        policy =>
        {
            policy.AddRequirements(new SurveyCreatorRequirement());
            policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement 
            // By adding the CookieAuthenticationDefaults.AuthenticationScheme, if an authenticated
            // user is not in the appropriate role, they will be redirected to a "forbidden" page.
            policy.AddAuthenticationSchemes(CookieAuthenticationDefaults.AuthenticationScheme);
        });

    options.AddPolicy(PolicyNames.RequireSurveyAdmin,
        policy =>
        {
            policy.AddRequirements(new SurveyAdminRequirement());
            policy.RequireAuthenticatedUser();  
            policy.AddAuthenticationSchemes(CookieAuthenticationDefaults.AuthenticationScheme);
        });
});
```

该代码还设置了身份验证方案，该方案告诉 ASP.NET 在授权失败时，应该运行哪些身份验证中间件。 在这种情况下，我们指定 cookie 身份验证中间件，因为 cookie 身份验证中间件可将用户重定向到“禁止访问”页。 在 `AccessDeniedPath` 选项中为 cookie 中间件设置“禁止访问”页的位置；请参阅[配置身份验证中间件]。

### <a name="authorize-controller-actions"></a>授权控制器操作
最后，为了授权 MVC 控制器中的操作，请在 `Authorize` 特性中设置策略：

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
public IActionResult Create()
{
    var survey = new SurveyDTO();
    return View(survey);
}
```

在早期版本的 ASP.NET 中，需要对该特性设置角色属性：

```csharp
// old way
[Authorize(Roles = "SurveyCreator")]
```

这在 ASP.NET Core 中仍受支持，但与授权策略相比，它有一些缺点：

* 它假定特定声明类型。 策略可以检查任何声明类型。 角色只是一种声明类型。
* 角色名称被硬编码到该特性。 通过使用策略，授权逻辑全在同一位置，更容易更新或从配置设置中加载。
* 策略支持简单角色成员资格无法表示的更复杂的授权决策（例如，年龄 >= 21）。

## <a name="resource-based-authorization"></a>基于资源的授权
每当授权依赖于受操作影响的特定资源时采用基于资源的授权。 在 Tailspin Surveys 应用程序中，每个 Surveys 都有一个所有者，以及零个或许多个参与者。

* 所有者可以读取、更新、删除、发布和取消发布调查。
* 所有者可以为调查分配参与者。
* 参与者可以读取和更新调查。

请注意，“所有者”和“参与者”不是应用程序角色；他们按照根据调查存储在应用程序数据库中。 例如，若要检查用户是否可以删除调查，应用会检查用户是否为该调查的所有者。

在 ASP.NET Core 中，通过从 AuthorizationHandler 中派生并覆盖 Handle 方法来实现基于资源的授权。

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override void HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement operation, Survey resource)
    {
    }
}
```

请注意，对于 Survey 对象此类为强类型。  在启动时为 DI 注册类：

```csharp
services.AddSingleton<IAuthorizationHandler>(factory =>
{
    return new SurveyAuthorizationHandler();
});
```

若要执行授权检查，请使用可注入控制器的 IAuthorizationService 接口。 以下代码用于检查用户是否可以读取调查：

```csharp
if (await _authorizationService.AuthorizeAsync(User, survey, Operations.Read) == false)
{
    return StatusCode(403);
}
```

由于传入 `Survey` 对象，此调用将调用 `SurveyAuthorizationHandler`。

在授权代码中，最好聚合用户的基于角色和基于资源的所有权限，然后针对所需操作检查聚合集。
下面是 Surveys 应用的示例。 应用程序定义了几种权限类型：

* 管理员
* 参与者
* 创建者
* 所有者
* 读取器

该应用程序还定义了一组可能的调查操作：

* 创建
* 读取
* 更新
* 删除
* 发布
* 取消发布

下面的代码为特定用户和调查创建了权限列表。 请注意，该代码会查看调查中用户的应用角色和所有者/参与者字段。

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement requirement, Survey resource)
    {
        var permissions = new List<UserPermissionType>();
        int surveyTenantId = context.User.GetSurveyTenantIdValue();
        int userId = context.User.GetSurveyUserIdValue();
        string user = context.User.GetUserName();

        if (resource.TenantId == surveyTenantId)
        {
            // Admin can do anything, as long as the resource belongs to the admin's tenant.
            if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyAdmin))
            {
                context.Succeed(requirement);
                return Task.FromResult(0);
            }

            if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyCreator))
            {
                permissions.Add(UserPermissionType.Creator);
            }
            else
            {
                permissions.Add(UserPermissionType.Reader);
            }

            if (resource.OwnerId == userId)
            {
                permissions.Add(UserPermissionType.Owner);
            }
        }
        if (resource.Contributors != null && resource.Contributors.Any(x => x.UserId == userId))
        {
            permissions.Add(UserPermissionType.Contributor);
        }

        if (ValidateUserPermissions[requirement](permissions))
        {
            context.Succeed(requirement);
        }
        return Task.FromResult(0);
    }
}
```

在多租户应用程序中，必须确保权限不会“泄漏”到另一租户的数据中。 在 Surveys 应用中，参与者权限允许跨租户 &mdash; 可将另一租户中的某人指定为参与者。 其他权限类型仅限于属于该用户租户的资源。 为了强制执行此要求，代码在授予权限之前会检查租户 ID。 （在创建调查时所分配的 `TenantId` 字段。）

下一步是对照权限检查操作（读取、更新、删除等等）。 Surveys 应用使用函数的查找表来实现此步骤：

```csharp
static readonly Dictionary<OperationAuthorizationRequirement, Func<List<UserPermissionType>, bool>> ValidateUserPermissions
    = new Dictionary<OperationAuthorizationRequirement, Func<List<UserPermissionType>, bool>>

    {
        { Operations.Create, x => x.Contains(UserPermissionType.Creator) },

        { Operations.Read, x => x.Contains(UserPermissionType.Creator) ||
                                x.Contains(UserPermissionType.Reader) ||
                                x.Contains(UserPermissionType.Contributor) ||
                                x.Contains(UserPermissionType.Owner) },

        { Operations.Update, x => x.Contains(UserPermissionType.Contributor) ||
                                x.Contains(UserPermissionType.Owner) },

        { Operations.Delete, x => x.Contains(UserPermissionType.Owner) },

        { Operations.Publish, x => x.Contains(UserPermissionType.Owner) },

        { Operations.UnPublish, x => x.Contains(UserPermissionType.Owner) }
    };
```

[**下一篇**][web-api]

<!-- Links -->
[Tailspin]: tailspin.md

[应用程序角色]: app-roles.md
[policies]: /aspnet/core/security/authorization/policies
[参考实现]: tailspin.md
[配置身份验证中间件]: authenticate.md#configure-the-auth-middleware
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[web-api]: web-api.md
