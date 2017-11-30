---
title: "多租户应用程序中的授权"
description: "如何在多租户应用程序中执行授权"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: app-roles
pnp.series.next: web-api
ms.openlocfilehash: 86c308d21f19bb3ac2a4a2240a9a03a504de5cf4
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="role-based-and-resource-based-authorization"></a><span data-ttu-id="e84cf-103">基于角色和基于资源的授权</span><span class="sxs-lookup"><span data-stu-id="e84cf-103">Role-based and resource-based authorization</span></span>

<span data-ttu-id="e84cf-104">[![GitHub](../_images/github.png) 示例代码][sample application]</span><span class="sxs-lookup"><span data-stu-id="e84cf-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="e84cf-105">[参考实现]是 ASP.NET Core 应用程序。</span><span class="sxs-lookup"><span data-stu-id="e84cf-105">Our [reference implementation] is an ASP.NET Core application.</span></span> <span data-ttu-id="e84cf-106">在本文中，我们将介绍使用 ASP.NET Core 中提供的授权 API 来进行授权的两种通用方法。</span><span class="sxs-lookup"><span data-stu-id="e84cf-106">In this article we'll look at two general approaches to authorization, using the authorization APIs provided in ASP.NET Core.</span></span>

* <span data-ttu-id="e84cf-107">**基于角色的授权**。</span><span class="sxs-lookup"><span data-stu-id="e84cf-107">**Role-based authorization**.</span></span> <span data-ttu-id="e84cf-108">基于分配给用户的角色授权操作。</span><span class="sxs-lookup"><span data-stu-id="e84cf-108">Authorizing an action based on the roles assigned to a user.</span></span> <span data-ttu-id="e84cf-109">例如，某些操作需要管理员角色。</span><span class="sxs-lookup"><span data-stu-id="e84cf-109">For example, some actions require an administrator role.</span></span>
* <span data-ttu-id="e84cf-110">**基于资源的授权**。</span><span class="sxs-lookup"><span data-stu-id="e84cf-110">**Resource-based authorization**.</span></span> <span data-ttu-id="e84cf-111">基于特定资源授权操作。</span><span class="sxs-lookup"><span data-stu-id="e84cf-111">Authorizing an action based on a particular resource.</span></span> <span data-ttu-id="e84cf-112">例如，每个资源都有一个所有者。</span><span class="sxs-lookup"><span data-stu-id="e84cf-112">For example, every resource has an owner.</span></span> <span data-ttu-id="e84cf-113">所有者可以删除资源；其他用户则不能。</span><span class="sxs-lookup"><span data-stu-id="e84cf-113">The owner can delete the resource; other users cannot.</span></span>

<span data-ttu-id="e84cf-114">一般的应用会结合使用这两种方法。</span><span class="sxs-lookup"><span data-stu-id="e84cf-114">A typical app will employ a mix of both.</span></span> <span data-ttu-id="e84cf-115">例如，若要删除资源，用户必须是资源所有者或管理员。</span><span class="sxs-lookup"><span data-stu-id="e84cf-115">For example, to delete a resource, the user must be the resource owner *or* an admin.</span></span>

## <a name="role-based-authorization"></a><span data-ttu-id="e84cf-116">基于角色的授权</span><span class="sxs-lookup"><span data-stu-id="e84cf-116">Role-Based Authorization</span></span>
<span data-ttu-id="e84cf-117">[Tailspin Surveys][Tailspin] 应用程序可定义以下角色：</span><span class="sxs-lookup"><span data-stu-id="e84cf-117">The [Tailspin Surveys][Tailspin] application defines the following roles:</span></span>

* <span data-ttu-id="e84cf-118">管理员。</span><span class="sxs-lookup"><span data-stu-id="e84cf-118">Administrator.</span></span> <span data-ttu-id="e84cf-119">可在属于该租户的任何调查中执行所有 CRUD 操作。</span><span class="sxs-lookup"><span data-stu-id="e84cf-119">Can perform all CRUD operations on any survey that belongs to that tenant.</span></span>
* <span data-ttu-id="e84cf-120">创建者。</span><span class="sxs-lookup"><span data-stu-id="e84cf-120">Creator.</span></span> <span data-ttu-id="e84cf-121">可创建新调查</span><span class="sxs-lookup"><span data-stu-id="e84cf-121">Can create new surveys</span></span>
* <span data-ttu-id="e84cf-122">读者。</span><span class="sxs-lookup"><span data-stu-id="e84cf-122">Reader.</span></span> <span data-ttu-id="e84cf-123">可读取属于该租户的任何调查</span><span class="sxs-lookup"><span data-stu-id="e84cf-123">Can read any surveys that belong to that tenant</span></span>

<span data-ttu-id="e84cf-124">角色应用于该应用程序的用户。</span><span class="sxs-lookup"><span data-stu-id="e84cf-124">Roles apply to *users* of the application.</span></span> <span data-ttu-id="e84cf-125">在 Surveys 应用程序中，用户可以是管理员、创建者或读者。</span><span class="sxs-lookup"><span data-stu-id="e84cf-125">In the Surveys application, a user is either an administrator, creator, or reader.</span></span>

<span data-ttu-id="e84cf-126">有关如何定义和管理角色的讨论，请参阅[应用程序角色]。</span><span class="sxs-lookup"><span data-stu-id="e84cf-126">For a discussion of how to define and manage roles, see [Application roles].</span></span>

<span data-ttu-id="e84cf-127">无论如何管理角色，授权代码看起来都类似。</span><span class="sxs-lookup"><span data-stu-id="e84cf-127">Regardless of how you manage the roles, your authorization code will look similar.</span></span> <span data-ttu-id="e84cf-128">ASP.NET Core 具有名为[授权策略][policies]的抽象。</span><span class="sxs-lookup"><span data-stu-id="e84cf-128">ASP.NET Core has an abstraction called [authorization policies][policies].</span></span> <span data-ttu-id="e84cf-129">使用此功能，可在代码中定义授权策略，然后将其应用于控制器操作。</span><span class="sxs-lookup"><span data-stu-id="e84cf-129">With this feature, you define authorization policies in code, and then apply those policies to controller actions.</span></span> <span data-ttu-id="e84cf-130">策略与控制器分离。</span><span class="sxs-lookup"><span data-stu-id="e84cf-130">The policy is decoupled from the controller.</span></span>

### <a name="create-policies"></a><span data-ttu-id="e84cf-131">创建策略</span><span class="sxs-lookup"><span data-stu-id="e84cf-131">Create policies</span></span>
<span data-ttu-id="e84cf-132">若要定义策略，首先创建实现 `IAuthorizationRequirement` 的类。</span><span class="sxs-lookup"><span data-stu-id="e84cf-132">To define a policy, first create a class that implements `IAuthorizationRequirement`.</span></span> <span data-ttu-id="e84cf-133">从 `AuthorizationHandler` 中派生是最简单的方法。</span><span class="sxs-lookup"><span data-stu-id="e84cf-133">It's easiest to derive from `AuthorizationHandler`.</span></span> <span data-ttu-id="e84cf-134">在 `Handle` 方法中，请检查相关声明。</span><span class="sxs-lookup"><span data-stu-id="e84cf-134">In the `Handle` method, examine the relevant claim(s).</span></span>

<span data-ttu-id="e84cf-135">下面是 Tailspin Surveys 应用程序中的示例：</span><span class="sxs-lookup"><span data-stu-id="e84cf-135">Here is an example from the Tailspin Surveys application:</span></span>

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

<span data-ttu-id="e84cf-136">此类定义用户创建新调查的需求。</span><span class="sxs-lookup"><span data-stu-id="e84cf-136">This class defines the requirement for a user to create a new survey.</span></span> <span data-ttu-id="e84cf-137">用户必须是 SurveyAdmin 或 SurveyCreator 角色。</span><span class="sxs-lookup"><span data-stu-id="e84cf-137">The user must be in the SurveyAdmin or SurveyCreator role.</span></span>

<span data-ttu-id="e84cf-138">在启动类中，定义包含一个或多个需求的命名策略。</span><span class="sxs-lookup"><span data-stu-id="e84cf-138">In your startup class, define a named policy that includes one or more requirements.</span></span> <span data-ttu-id="e84cf-139">如果有多个需求，用户必须满足每个需求才能得到授权。</span><span class="sxs-lookup"><span data-stu-id="e84cf-139">If there are multiple requirements, the user must meet *every* requirement to be authorized.</span></span> <span data-ttu-id="e84cf-140">下面的代码定义了两个策略：</span><span class="sxs-lookup"><span data-stu-id="e84cf-140">The following code defines two policies:</span></span>

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

<span data-ttu-id="e84cf-141">该代码还设置了身份验证方案，该方案告诉 ASP.NET 在授权失败时，应该运行哪些身份验证中间件。</span><span class="sxs-lookup"><span data-stu-id="e84cf-141">This code also sets the authentication scheme, which tells ASP.NET which authentication middleware should run if authorization fails.</span></span> <span data-ttu-id="e84cf-142">在这种情况下，我们指定 cookie 身份验证中间件，因为 cookie 身份验证中间件可将用户重定向到“禁止访问”页。</span><span class="sxs-lookup"><span data-stu-id="e84cf-142">In this case, we specify the cookie authentication middleware, because the cookie authentication middleware can redirect the user to a "Forbidden" page.</span></span> <span data-ttu-id="e84cf-143">在 `AccessDeniedPath` 选项中为 cookie 中间件设置“禁止访问”页的位置；请参阅[配置身份验证中间件]。</span><span class="sxs-lookup"><span data-stu-id="e84cf-143">The location of the Forbidden page is set in the `AccessDeniedPath` option for the cookie middleware; see [Configuring the authentication middleware].</span></span>

### <a name="authorize-controller-actions"></a><span data-ttu-id="e84cf-144">授权控制器操作</span><span class="sxs-lookup"><span data-stu-id="e84cf-144">Authorize controller actions</span></span>
<span data-ttu-id="e84cf-145">最后，为了授权 MVC 控制器中的操作，请在 `Authorize` 特性中设置策略：</span><span class="sxs-lookup"><span data-stu-id="e84cf-145">Finally, to authorize an action in an MVC controller, set the policy in the `Authorize` attribute:</span></span>

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
public IActionResult Create()
{
    var survey = new SurveyDTO();
    return View(survey);
}
```

<span data-ttu-id="e84cf-146">在早期版本的 ASP.NET 中，需要对该特性设置角色属性：</span><span class="sxs-lookup"><span data-stu-id="e84cf-146">In earlier versions of ASP.NET, you would set the **Roles** property on the attribute:</span></span>

```csharp
// old way
[Authorize(Roles = "SurveyCreator")]

```

<span data-ttu-id="e84cf-147">这在 ASP.NET Core 中仍受支持，但与授权策略相比，它有一些缺点：</span><span class="sxs-lookup"><span data-stu-id="e84cf-147">This is still supported in ASP.NET Core, but it has some drawbacks compared with authorization policies:</span></span>

* <span data-ttu-id="e84cf-148">它假定特定声明类型。</span><span class="sxs-lookup"><span data-stu-id="e84cf-148">It assumes a particular claim type.</span></span> <span data-ttu-id="e84cf-149">策略可以检查任何声明类型。</span><span class="sxs-lookup"><span data-stu-id="e84cf-149">Policies can check for any claim type.</span></span> <span data-ttu-id="e84cf-150">角色只是一种声明类型。</span><span class="sxs-lookup"><span data-stu-id="e84cf-150">Roles are just a type of claim.</span></span>
* <span data-ttu-id="e84cf-151">角色名称被硬编码到该特性。</span><span class="sxs-lookup"><span data-stu-id="e84cf-151">The role name is hard-coded into the attribute.</span></span> <span data-ttu-id="e84cf-152">通过使用策略，授权逻辑全在同一位置，更容易更新或从配置设置中加载。</span><span class="sxs-lookup"><span data-stu-id="e84cf-152">With policies, the authorization logic is all in one place, making it easier to update or even load from configuration settings.</span></span>
* <span data-ttu-id="e84cf-153">策略支持简单角色成员资格无法表示的更复杂的授权决策（例如，年龄 >= 21）。</span><span class="sxs-lookup"><span data-stu-id="e84cf-153">Policies enable more complex authorization decisions (e.g., age >= 21) that can't be expressed by simple role membership.</span></span>

## <a name="resource-based-authorization"></a><span data-ttu-id="e84cf-154">基于资源的授权</span><span class="sxs-lookup"><span data-stu-id="e84cf-154">Resource based authorization</span></span>
<span data-ttu-id="e84cf-155">每当授权依赖于受操作影响的特定资源时采用基于资源的授权。</span><span class="sxs-lookup"><span data-stu-id="e84cf-155">*Resource based authorization* occurs whenever the authorization depends on a specific resource that will be affected by an operation.</span></span> <span data-ttu-id="e84cf-156">在 Tailspin Surveys 应用程序中，每个 Surveys 都有一个所有者，以及零个或许多个参与者。</span><span class="sxs-lookup"><span data-stu-id="e84cf-156">In the Tailspin Surveys application, every survey has an owner and zero-to-many contributors.</span></span>

* <span data-ttu-id="e84cf-157">所有者可以读取、更新、删除、发布和取消发布调查。</span><span class="sxs-lookup"><span data-stu-id="e84cf-157">The owner can read, update, delete, publish, and unpublish the survey.</span></span>
* <span data-ttu-id="e84cf-158">所有者可以为调查分配参与者。</span><span class="sxs-lookup"><span data-stu-id="e84cf-158">The owner can assign contributors to the survey.</span></span>
* <span data-ttu-id="e84cf-159">参与者可以读取和更新调查。</span><span class="sxs-lookup"><span data-stu-id="e84cf-159">Contributors can read and update the survey.</span></span>

<span data-ttu-id="e84cf-160">请注意，“所有者”和“参与者”不是应用程序角色；他们按照根据调查存储在应用程序数据库中。</span><span class="sxs-lookup"><span data-stu-id="e84cf-160">Note that "owner" and "contributor" are not application roles; they are stored per survey, in the application database.</span></span> <span data-ttu-id="e84cf-161">例如，若要检查用户是否可以删除调查，应用会检查用户是否为该调查的所有者。</span><span class="sxs-lookup"><span data-stu-id="e84cf-161">To check whether a user can delete a survey, for example, the app checks whether the user is the owner for that survey.</span></span>

<span data-ttu-id="e84cf-162">在 ASP.NET Core 中，通过从 AuthorizationHandler 中派生并覆盖 Handle 方法来实现基于资源的授权。</span><span class="sxs-lookup"><span data-stu-id="e84cf-162">In ASP.NET Core, implement resource-based authorization by deriving from **AuthorizationHandler** and overriding the **Handle** method.</span></span>

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override void HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement operation, Survey resource)
    {
    }
}
```

<span data-ttu-id="e84cf-163">请注意，对于 Survey 对象此类为强类型。</span><span class="sxs-lookup"><span data-stu-id="e84cf-163">Notice that this class is strongly typed for Survey objects.</span></span>  <span data-ttu-id="e84cf-164">在启动时为 DI 注册类：</span><span class="sxs-lookup"><span data-stu-id="e84cf-164">Register the class for DI on startup:</span></span>

```csharp
services.AddSingleton<IAuthorizationHandler>(factory =>
{
    return new SurveyAuthorizationHandler();
});
```

<span data-ttu-id="e84cf-165">若要执行授权检查，请使用可注入控制器的 IAuthorizationService 接口。</span><span class="sxs-lookup"><span data-stu-id="e84cf-165">To perform authorization checks, use the **IAuthorizationService** interface, which you can inject into your controllers.</span></span> <span data-ttu-id="e84cf-166">以下代码用于检查用户是否可以读取调查：</span><span class="sxs-lookup"><span data-stu-id="e84cf-166">The following code checks whether a user can read a survey:</span></span>

```csharp
if (await _authorizationService.AuthorizeAsync(User, survey, Operations.Read) == false)
{
    return StatusCode(403);
}
```

<span data-ttu-id="e84cf-167">由于传入 `Survey` 对象，此调用将调用 `SurveyAuthorizationHandler`。</span><span class="sxs-lookup"><span data-stu-id="e84cf-167">Because we pass in a `Survey` object, this call will invoke the `SurveyAuthorizationHandler`.</span></span>

<span data-ttu-id="e84cf-168">在授权代码中，最好聚合用户的基于角色和基于资源的所有权限，然后针对所需操作检查聚合集。</span><span class="sxs-lookup"><span data-stu-id="e84cf-168">In your authorization code, a good approach is to aggregate all of the user's role-based and resource-based permissions, then check the aggregate set against the desired operation.</span></span>
<span data-ttu-id="e84cf-169">下面是 Surveys 应用的示例。</span><span class="sxs-lookup"><span data-stu-id="e84cf-169">Here is an example from the Surveys app.</span></span> <span data-ttu-id="e84cf-170">应用程序定义了几种权限类型：</span><span class="sxs-lookup"><span data-stu-id="e84cf-170">The application defines several permission types:</span></span>

* <span data-ttu-id="e84cf-171">管理员</span><span class="sxs-lookup"><span data-stu-id="e84cf-171">Admin</span></span>
* <span data-ttu-id="e84cf-172">参与者</span><span class="sxs-lookup"><span data-stu-id="e84cf-172">Contributor</span></span>
* <span data-ttu-id="e84cf-173">创建者</span><span class="sxs-lookup"><span data-stu-id="e84cf-173">Creator</span></span>
* <span data-ttu-id="e84cf-174">所有者</span><span class="sxs-lookup"><span data-stu-id="e84cf-174">Owner</span></span>
* <span data-ttu-id="e84cf-175">读者</span><span class="sxs-lookup"><span data-stu-id="e84cf-175">Reader</span></span>

<span data-ttu-id="e84cf-176">该应用程序还定义了一组可能的调查操作：</span><span class="sxs-lookup"><span data-stu-id="e84cf-176">The application also defines a set of possible operations on surveys:</span></span>

* <span data-ttu-id="e84cf-177">创建</span><span class="sxs-lookup"><span data-stu-id="e84cf-177">Create</span></span>
* <span data-ttu-id="e84cf-178">读取</span><span class="sxs-lookup"><span data-stu-id="e84cf-178">Read</span></span>
* <span data-ttu-id="e84cf-179">更新</span><span class="sxs-lookup"><span data-stu-id="e84cf-179">Update</span></span>
* <span data-ttu-id="e84cf-180">删除</span><span class="sxs-lookup"><span data-stu-id="e84cf-180">Delete</span></span>
* <span data-ttu-id="e84cf-181">发布</span><span class="sxs-lookup"><span data-stu-id="e84cf-181">Publish</span></span>
* <span data-ttu-id="e84cf-182">取消发布</span><span class="sxs-lookup"><span data-stu-id="e84cf-182">Unpublsh</span></span>

<span data-ttu-id="e84cf-183">下面的代码为特定用户和调查创建了权限列表。</span><span class="sxs-lookup"><span data-stu-id="e84cf-183">The following code creates a list of permissions for a particular user and survey.</span></span> <span data-ttu-id="e84cf-184">请注意，该代码会查看调查中用户的应用角色和所有者/参与者字段。</span><span class="sxs-lookup"><span data-stu-id="e84cf-184">Notice that this code looks at both the user's app roles, and the owner/contributor fields in the survey.</span></span>

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

<span data-ttu-id="e84cf-185">在多租户应用程序中，必须确保权限不会“泄漏”到另一租户的数据中。</span><span class="sxs-lookup"><span data-stu-id="e84cf-185">In a multi-tenant application, you must ensure that permissions don't "leak" to another tenant's data.</span></span> <span data-ttu-id="e84cf-186">在 Surveys 应用中，参与者权限允许跨租户 &mdash; 可将另一租户中的某人指定为参与者。</span><span class="sxs-lookup"><span data-stu-id="e84cf-186">In the Surveys app, the Contributor permission is allowed across tenants &mdash; you can assign someone from another tenant as a contriubutor.</span></span> <span data-ttu-id="e84cf-187">其他权限类型仅限于属于该用户租户的资源。</span><span class="sxs-lookup"><span data-stu-id="e84cf-187">The other permission types are restricted to resources that belong to that user's tenant.</span></span> <span data-ttu-id="e84cf-188">为了强制执行此要求，代码在授予权限之前会检查租户 ID。</span><span class="sxs-lookup"><span data-stu-id="e84cf-188">To enforce this requirement, the code checks the tenant ID before granting the permission.</span></span> <span data-ttu-id="e84cf-189">（在创建调查时所分配的 `TenantId` 字段。）</span><span class="sxs-lookup"><span data-stu-id="e84cf-189">(The `TenantId` field as assigned when the survey is created.)</span></span>

<span data-ttu-id="e84cf-190">下一步是对照权限检查操作（读取、更新、删除等等）。</span><span class="sxs-lookup"><span data-stu-id="e84cf-190">The next step is to check the operation (read, update, delete, etc) against the permissions.</span></span> <span data-ttu-id="e84cf-191">Surveys 应用使用函数的查找表来实现此步骤：</span><span class="sxs-lookup"><span data-stu-id="e84cf-191">The Surveys app implements this step by using a lookup table of functions:</span></span>

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

<span data-ttu-id="e84cf-192">[下一篇][web-api]</span><span class="sxs-lookup"><span data-stu-id="e84cf-192">[**Next**][web-api]</span></span>

<!-- Links -->
[Tailspin]: tailspin.md

[应用程序角色]: app-roles.md
[policies]: /aspnet/core/security/authorization/policies
[参考实现]: tailspin.md
[配置身份验证中间件]: authenticate.md#configure-the-auth-middleware
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[web-api]: web-api.md
