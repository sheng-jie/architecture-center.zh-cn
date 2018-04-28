---
title: 使用 Key Vault 保护应用程序机密
description: 如何使用 Key Vault 服务来存储应用程序机密
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: client-assertion
ms.openlocfilehash: d49129a38d0413f6006095f03b817885e1ce6c92
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/16/2018
---
# <a name="use-azure-key-vault-to-protect-application-secrets"></a>使用 Azure Key Vault 保护应用程序机密

[![GitHub](../_images/github.png) 示例代码][sample application]

经常会有敏感的应用程序设置，必须受到保护，例如：

* 数据库连接字符串
* 密码
* 加密密钥

安全的最佳做法是切勿将这些机密存储在源代码管理中。 即使源代码存储库是私有的，它们也很容易泄露。 而且这不仅仅是保护机密不被泄露公开。 在较大型的项目上，建议限制有权访问生产机密的开发者和操作员。 （测试或开发环境的设置有所不同。）

更安全的做法是将这些机密存储在 [Azure Key Vault][KeyVault]。 Key Vault 是用于管理加密密钥和其他机密的云托管服务。 本文介绍如何使用 Key Vault 存储应用的配置设置。

在 [Tailspin Surveys][Surveys] 应用程序中，以下设置是机密：

* 数据库连接字符串。
* Redis 连接字符串。
* Web 应用程序的客户端密码。

Surveys 应用程序从以下位置加载配置设置：

* appsettings.json 文件
* [用户机密存储][user-secrets]（仅开发环境；用于测试）
* 宿主环境（Azure Web 应用中的应用设置）
* Key Vault（启用时）

其中的每个设置都会替代上一个设置，因此存储在 Key Vault 中的任何设置都是优先的。

> [!NOTE]
> 默认情况下，Key Vault 配置提供程序处于禁用状态。 无需在本地运行应用程序。 将在生产部署中启用它。

启动时，应用程序从每个注册的配置提供程序读取设置，并使用它们来填充强类型的选项对象。 有关详细信息，请参阅[使用选项和配置对象][options]。

## <a name="setting-up-key-vault-in-the-surveys-app"></a>在 Surveys 应用中设置 Key Vault
先决条件：

* 安装 [Azure 资源管理器 Cmdlet][azure-rm-cmdlets]。
* 按照[运行 Surveys 应用程序][readme]中所述配置 Surveys 应用程序。

高级步骤：

1. 设置租户中的管理员用户。
2. 设置客户端证书。
3. 创建密钥保管库。
4. 将配置设置添加到密钥保管库。
5. 取消评论启动密钥保管库的代码。
6. 更新应用程序的用户机密。

### <a name="set-up-an-admin-user"></a>设置管理员用户
> [!NOTE]
> 若要创建密钥保管库，必须使用可以管理 Azure 订阅的帐户。 此外，授权从密钥保管库中读取的任何应用程序必须在与该帐户相同的租户中注册。
> 
> 

在此步骤中，当你从注册 Surveys 应用的租户以用户身份登录时，需要确保能够创建密钥保管库。

在注册 Surveys 应用程序的 Azure AD 租户中创建管理员用户。

1. 登录到 [Azure 门户][azure-portal]。
2. 选择注册应用程序所在的 Azure AD 租户。
3. 单击“更多服务” > “安全 + 标识” > “Azure Active Directory” > 用户和组” > “所有用户”。
4. 在门户顶部单击“新建用户”。
5. 填充字段并将用户分配到“全局管理员”目录角色。
6. 单击“创建”。

![全局管理员用户](./images/running-the-app/global-admin-user.png)

现在将此用户分配为订阅所有者。

1. 在“中心”菜单上，选择“订阅”。

    ![](./images/running-the-app/subscriptions.png)

2. 选择希望管理员访问的订阅。
3. 在“订阅”边栏选项卡中，选择“访问控制 (IAM)”。
4. 单击 **“添加”**。
4. 在“角色”下，选择“所有者”。
5. 键入要添加为“所有者”的用户的电子邮件地址。
6. 选择用户并单击“保存”。

### <a name="set-up-a-client-certificate"></a>设置客户端证书
1. 如下所示，运行 PowerShell 脚本 [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault]：
   
    ```
    .\Setup-KeyVault.ps1 -Subject <<subject>>
    ```
    对于 `Subject` 参数，键入任意名称，例如“surveysapp”。 此脚本生成自签名证书，并将其存储在“当前用户/个人”证书存储中。 脚本的输出是 JSON 片段。 复制此值。

2. 在 [Azure 门户][azure-portal]的右上角选择帐户，切换到注册 Surveys 应用程序所在的目录。

3. 选择“Azure Active Directory” > “应用注册”>“Surveys”

4.  单击“清单”，然后单击“编辑”。

5.  将脚本的输出粘贴到 `keyCredentials` 属性。 如下图所示：
        
    ```json
    "keyCredentials": [
        {
        "type": "AsymmetricX509Cert",
        "usage": "Verify",
        "keyId": "29d4f7db-0539-455e-b708-....",
        "customKeyIdentifier": "ZEPpP/+KJe2fVDBNaPNOTDoJMac=",
        "value": "MIIDAjCCAeqgAwIBAgIQFxeRiU59eL.....
        }
    ],
    ```          

6. 单击“ **保存**”。  

7. 重复步骤 3-6，将相同的 JSON 片段添加到 Web API (Surveys.WebAPI) 的应用程序清单。

8. 从 PowerShell 窗口运行以下命令以获取证书指纹。
   
    ```
    certutil -store -user my [subject]
    ```
    
    对于 `[subject]`，请使用为 PowerShell 脚本中的主题指定的值。 指纹在“Cert Hash(sha1)”下列出。 复制此值。 稍后将使用指纹。

### <a name="create-a-key-vault"></a>创建 key vault
1. 如下所示，运行 PowerShell 脚本 [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault]：
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ResourceGroupName <<resource group name>> -Location <<location>>
    ```
   
    提示输入凭证时，请以之前创建的 Azure AD 用户身份登录。 此脚本创建新的资源组，并在该资源组中创建新的密钥保管库。 
   
2. 再次运行 SetupKeyVault.ps，如下所示：
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ApplicationIds @("<<Surveys app id>>", "<<Surveys.WebAPI app ID>>")
    ```
   
    设置以下参数值：
   
       * 密钥保管库名称 = 上一步中赋予密钥保管库的名称。
       * Surveys 应用 ID = Surveys Web 应用程序的应用程序 ID。
       * Surveys.WebApi 应用 ID = Surveys.WebAPI 应用程序的应用程序 ID。
         
    示例：
     
    ```
     .\Setup-KeyVault.ps1 -KeyVaultName tailspinkv -ApplicationIds @("f84df9d1-91cc-4603-b662-302db51f1031", "8871a4c2-2a23-4650-8b46-0625ff3928a6")
    ```
    
    此脚本授权 Web 应用和 Web API 从密钥保管库检索机密。 有关详细信息，请参阅 [Azure Key Vault 入门](/azure/key-vault/key-vault-get-started/)。

### <a name="add-configuration-settings-to-your-key-vault"></a>将配置设置添加到密钥保管库
1. 运行 SetupKeyVault.ps，如下所示：
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Redis--Configuration -KeyValue "<<Redis DNS name>>.redis.cache.windows.net,password=<<Redis access key>>,ssl=true" 
    ```
    其中
   
   * 密钥保管库名称 = 上一步中赋予密钥保管库的名称。
   * Redis DNS 名称 = Redis 缓存实例的 DNS 名称。
   * Redis 访问密钥 = Redis 缓存实例的访问密钥。
     
2. 此时，最好测试是否已成功将机密存储到密钥保管库。 运行以下 PowerShell 命令：
   
    ```
    Get-AzureKeyVaultSecret <<key vault name>> Redis--Configuration | Select-Object *
    ```

3. 再次运行 SetupKeyVault.ps 以添加数据库连接字符串：
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Data--SurveysConnectionString -KeyValue <<DB connection string>> -ConfigName "Data:SurveysConnectionString"
    ```
   
    其中，`<<DB connection string>>` 是数据库连接字符串的值。
   
    要使用本地数据库进行测试，请从 Tailspin.Surveys.Web/appsettings.json 文件复制连接字符串。 如果进行该操作，请确保将双反斜杠（“\\\\”）更改为单反斜杠。 双反斜杠是 JSON 文件中的转义字符。
   
    示例：
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName mykeyvault -KeyName Data--SurveysConnectionString -KeyValue "Server=(localdb)\MSSQLLocalDB;Database=Tailspin.SurveysDB;Trusted_Connection=True;MultipleActiveResultSets=true" 
    ```

### <a name="uncomment-the-code-that-enables-key-vault"></a>取消评论启动 Key Vault 的代码
1. 打开 Tailspin.Surveys 解决方案。
2. 在 Tailspin.Surveys.Web/Startup.cs 中，查找以下代码块并对其取消评论。
   
    ```csharp
    //var config = builder.Build();
    //builder.AddAzureKeyVault(
    //    $"https://{config["KeyVault:Name"]}.vault.azure.net/",
    //    config["AzureAd:ClientId"],
    //    config["AzureAd:ClientSecret"]);
    ```
3. 在 Tailspin.Surveys.Web/Startup.cs 中，查找注册 `ICredentialService` 的代码。 取消评论使用 `CertificateCredentialService` 的行，并注释掉使用 `ClientCredentialService` 的行：
   
    ```csharp
    // Uncomment this:
    services.AddSingleton<ICredentialService, CertificateCredentialService>();
    // Comment out this:
    //services.AddSingleton<ICredentialService, ClientCredentialService>();
    ```
   
    通过此项更改，Web 应用可以使用[客户端断言][client-assertion]获取 OAuth 访问令牌。 使用客户端断言，就无需 OAuth 客户端密码。 或者，可以在密钥保管库中存储客户端密码。 但密钥保管库和客户端断言都使用客户端证书，因此，如果启用密钥保管库，最好也启用客户端断言。

### <a name="update-the-user-secrets"></a>更新用户机密
在解决方案资源管理器中，右键单击 Tailspin.Surveys.Web 项目并选“管理用户机密”。 在 secrets.json 文件中，删除现有的 JSON 并粘贴以下代码：

    ```
    {
      "AzureAd": {
        "ClientId": "[Surveys web app client ID]",
        "ClientSecret": "[Surveys web app client secret]",
        "PostLogoutRedirectUri": "https://localhost:44300/",
        "WebApiResourceId": "[App ID URI of your Surveys.WebAPI application]",
        "Asymmetric": {
          "CertificateThumbprint": "[certificate thumbprint. Example: 105b2ff3bc842c53582661716db1b7cdc6b43ec9]",
          "StoreName": "My",
          "StoreLocation": "CurrentUser",
          "ValidationRequired": "false"
        }
      },
      "KeyVault": {
        "Name": "[key vault name]"
      }
    }
    ```

用正确的值替换[方括号]中的项。

* `AzureAd:ClientId`：Surveys 应用的客户端 ID。
* `AzureAd:ClientSecret`：在 Azure AD 中注册 Surveys 应用程序时生成的键。
* `AzureAd:WebApiResourceId`：在 Azure AD 中创建 Surveys.WebAPI 应用程序时指定的应用 ID URI。
* `Asymmetric:CertificateThumbprint`：之前在创建客户端证书时获取到的证书指纹。
* `KeyVault:Name`：密钥保管库的名称。

> [!NOTE]
> `Asymmetric:ValidationRequired` 为 false，因为之前创建的证书不是由根证书颁发机构 (CA) 签名的。 在生产中，使用由根 CA 签名的证书并将 `ValidationRequired` 设置为 true。
> 
> 

保存已更新的 secrets.json 文件。

接下来，在解决方案资源管理器中，右键单击 Tailspin.Surveys.WebApi 项目并选择“管理用户机密”。 删除现有的 JSON 并粘贴以下代码：

```
{
  "AzureAd": {
    "ClientId": "[Surveys.WebAPI client ID]",
    "WebApiResourceId": "https://tailspin5.onmicrosoft.com/surveys.webapi",
    "Asymmetric": {
      "CertificateThumbprint": "[certificate thumbprint]",
      "StoreName": "My",
      "StoreLocation": "CurrentUser",
      "ValidationRequired": "false"
    }
  },
  "KeyVault": {
    "Name": "[key vault name]"
  }
}
```

替换[方括号]中的项，并保存 secrets.json 文件。

> [!NOTE]
> 对于 Web API，请确保使用 Surveys.WebAPI 应用程序的客户端 ID，而不是 Surveys 应用程序的客户端 ID。
> 
> 

[下一篇][adfs]

<!-- Links -->
[adfs]: ./adfs.md
[authorize-app]: /azure/key-vault/key-vault-get-started//#authorize
[azure-portal]: https://portal.azure.com
[azure-rm-cmdlets]: https://msdn.microsoft.com/library/mt125356.aspx
[client-assertion]: client-assertion.md
[configuration]: /aspnet/core/fundamentals/configuration
[KeyVault]: https://azure.microsoft.com/services/key-vault/
[key-tags]: https://msdn.microsoft.com/library/azure/dn903623.aspx#BKMK_Keytags
[Microsoft.Azure.KeyVault]: https://www.nuget.org/packages/Microsoft.Azure.KeyVault/
[options]: /aspnet/core/fundamentals/configuration#using-options-and-configuration-objects
[readme]: ./run-the-app.md
[Setup-KeyVault]: https://github.com/mspnp/multitenant-saas-guidance/blob/master/scripts/Setup-KeyVault.ps1
[Surveys]: tailspin.md
[user-secrets]: http://go.microsoft.com/fwlink/?LinkID=532709
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
