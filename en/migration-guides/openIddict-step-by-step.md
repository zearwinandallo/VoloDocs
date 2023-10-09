# Migrating from IdentityServer to OpenIddict Step by Step Guide

This guide provides layer-by-layer guidance for migrating your existing application to OpenIddict. Since OpenIddict is only available with ABP v6.0, you will need to update your existing application in order to apply OpenIddict changes.

## History
We are not removing the Identity Server packages and we will continue to release new versions of IdentityServer-related NuGet/NPM packages. That means you won't have an issue while upgrading to v6.0 when the stable version is released. We will continue to fix bugs in our packages for a while. ABP 7.0 will be based on .NET 7. If Identity Server continues to work with .NET 7, we will also continue to ship NuGet packages for our IDS integration.

On the other hand, Identity Server ends support for the open-source Identity Server at the end of 2022. The Identity Server team has decided to move to Duende IDS and ABP will not be migrated to the commercial Duende IDS. You can see the Duende Identity Server announcement from [this link](https://blog.duendesoftware.com/posts/20220111_fair_trade). 

## OpenIddict Migration Steps

Use the `abp update` command to update your existing application. See the [Upgrading docs](../Upgrading.md) for more info. Apply the required migrations by following the [Migration Guides](Index.md) based on your application version.

### Domain.Shared Layer

- In **MyApplication.Domain.Shared.csproj** replace the **project reference**:
  ```csharp
  <PackageReference Include="Volo.Abp.IdentityServer.Domain.Shared" Version="6.0.*" />
  ```
  with   
  ```csharp
  <PackageReference Include="Volo.Abp.OpenIddict.Domain.Shared" Version="6.0.*" />
  ```

- In **MyApplicationDomainSharedModule.cs** replace the usings and **module dependencies:**

  ```csharp
  using Volo.Abp.IdentityServer;
  ...
  typeof(AbpIdentityServerDomainSharedModule)
  ```
  with 
  ```csharp
  using Volo.Abp.OpenIddict;
  ...
  typeof(AbpOpenIddictDomainSharedModule)

### Domain Layer

- In **MyApplication.Domain.csproj** replace the **project references**:

  ```csharp
  <PackageReference Include="Volo.Abp.IdentityServer.Domain" Version="6.0.*" />
  <PackageReference Include="Volo.Abp.PermissionManagement.Domain.IdentityServer" Version="6.0.*" />
  ```

  with   

  ```csharp
  <PackageReference Include="Volo.Abp.OpenIddict.Domain" Version="6.0.*" />
  <PackageReference Include="Volo.Abp.PermissionManagement.Domain.OpenIddict" Version="6.0.*" />
  ```

- In **MyApplicationDomainModule.cs** replace the usings and **module dependencies**:

  ```csharp
  using Volo.Abp.IdentityServer;
  using Volo.Abp.PermissionManagement.IdentityServer;
  ...
  typeof(AbpIdentityServerDomainModule),
  typeof(AbpPermissionManagementDomainIdentityServerModule),
  ```

  with 

  ```csharp
  using Volo.Abp.OpenIddict;
  using Volo.Abp.PermissionManagement.OpenIddict;
  ...
  typeof(AbpOpenIddictDomainModule),
  typeof(AbpPermissionManagementDomainOpenIddictModule),
  ```

#### OpenIddictDataSeedContributor

- Create a folder named *OpenIddict* under the Domain project and copy the [OpenIddictDataSeedContributor.cs](https://github.com/abpframework/abp-samples/blob/master/Ids2OpenId/src/Ids2OpenId.Domain/OpenIddict/OpenIddictDataSeedContributor.cs) under this folder. **Rename** all the `Ids2OpenId` with your project name.
- Delete the *IdentityServer* folder that contains `IdentityServerDataSeedContributor.cs` which is no longer needed.

You can also create a project with the same name and copy the `OpenIddict` folder of the new project into your project.

### EntityFrameworkCore Layer

If you are using MongoDB, skip this step and check the *MongoDB* layer section.

- In **MyApplication.EntityFrameworkCore.csproj** replace the **project reference**:

  ```csharp
  <PackageReference Include="Volo.Abp.IdentityServer.EntityFrameworkCore" Version="6.0.*" />
  ```

  with   

  ```csharp
  <PackageReference Include="Volo.Abp.OpenIddict.EntityFrameworkCore" Version="6.0.*" />
  ```

- In **MyApplicationEntityFrameworkCoreModule.cs** replace the usings and **module dependencies**:

  ```csharp
  using Volo.Abp.IdentityServer.EntityFrameworkCore;
  ...
  typeof(AbpIdentityServerEntityFrameworkCoreModule),
  ```

  with 

  ```csharp
  using Volo.Abp.OpenIddict.EntityFrameworkCore;
  ...
  typeof(AbpOpenIddictEntityFrameworkCoreModule),
  ```

- In **MyApplicationDbContext.cs** replace the usings and **fluent api configurations**:

  ```csharp
  using Volo.Abp.IdentityServer.EntityFrameworkCore;
  ...
  using Volo.Abp.OpenIddict.EntityFrameworkCore;
  ...
  protected override void OnModelCreating(ModelBuilder builder)
  {
      base.OnModelCreating(builder);
  
      /* Include modules to your migration db context */
  
      ...
      builder.ConfigureIdentityServer();
  ```

  with 

  ```csharp
  using Volo.Abp.OpenIddict.EntityFrameworkCore;
  ...
  protected override void OnModelCreating(ModelBuilder builder)
  {
      base.OnModelCreating(builder);
  
      /* Include modules to your migration db context */
  
      ...
      builder.ConfigureOpenIddict();
  ```

### MongoDB Layer

If you are using EntityFrameworkCore, skip this step and check the *EntityFrameworkCore* layer section.

- In **MyApplication.MongoDB.csproj** replace the **project reference**:

  ```csharp
  <PackageReference Include="Volo.Abp.IdentityServer.MongoDB" Version="6.0.*" />
  ```

  with   

  ```csharp
  <PackageReference Include="Volo.Abp.OpenIddict.MongoDB" Version="6.0.*" />
  ```

- In **MyApplicationMongoDbModule.cs** replace the usings and **module dependencies**:

  ```csharp
  using Volo.Abp.IdentityServer.MongoDB;
  ...
  typeof(AbpIdentityServerMongoDbModule),
  ```

  with 

  ```csharp
  using Volo.Abp.OpenIddict.MongoDB;
  ...
  typeof(AbpOpenIddictMongoDbModule),
  ```

### DbMigrator Project

- In **MyApplication.DbMigrator.csproj** **add the project reference**:

  ```csharp
  <PackageReference Include="Microsoft.Extensions.Hosting" Version="6.0.1" />
  ```

for creating the host builder.

- In `appsettings.json` **replace the IdentityServer section with OpenIddict:**

  ```json
  "OpenIddict": {
      "Applications": {
        "MyApplication_Web": {
          "ClientId": "MyApplication_Web",
          "ClientSecret": "1q2w3e*",
          "RootUrl": "https://localhost:44384"
        },
        "MyApplication_App": {
          "ClientId": "MyApplication_App",
          "RootUrl": "http://localhost:4200"
        },
        "MyApplication_BlazorServerTiered": {
          "ClientId": "MyApplication_BlazorServerTiered",
          "ClientSecret": "1q2w3e*",
          "RootUrl": "https://localhost:44346"
        },
        "MyApplication_Swagger": {
          "ClientId": "MyApplication_Swagger",
          "RootUrl": "https://localhost:44391"
        }
      }
    }
  ```

  Replace **MyApplication** with your application name.

### Application Contracts Layer

- In **MyApplication.Application.Contracts.csproj** replace the **project reference**:

  ```csharp
  <PackageReference Include="Volo.Abp.IdentityServer.Application.Contracts" Version="6.0.*" />
  ```

  with   

  ```csharp
  <PackageReference Include="Volo.Abp.OpenIddict.Pro.Application.Contracts" Version="6.0.*" />
  ```

- In **MyApplicationApplicationContractsModule.cs** replace the usings and **module dependencies**:

  ```csharp
  using Volo.Abp.IdentityServer;
  ...
  typeof(AbpIdentityServerApplicationContractsModule),
  ```

  with 

  ```csharp
  using Volo.Abp.OpenIddict;
  ...
  typeof(AbpOpenIddictProApplicationContractsModule),
  ```

### Application Layer

- In **MyApplication.Application.csproj** replace the **project reference**:

  ```csharp
  <PackageReference Include="Volo.Abp.IdentityServer.Application" Version="6.0.*" />
  ```

  with   

  ```csharp
  <PackageReference Include="Volo.Abp.OpenIddict.Pro.Application" Version="6.0.*" />
  ```

- In **MyApplicationApplicationModule.cs** replace the usings and **module dependencies**:

  ```csharp
  using Volo.Abp.IdentityServer;
  ...
  typeof(AbpIdentityServerApplicationModule),
  ```

  with 

  ```csharp
  using Volo.Abp.OpenIddict;
  ...
  typeof(AbpOpenIddictProApplicationModule),
  ```

### HttpApi Layer

- In **MyApplication.HttpApi.csproj** replace the **project reference**:

  ```csharp
  <PackageReference Include="Volo.Abp.IdentityServer.HttpApi" Version="6.0.*" />
  ```

  with   

  ```csharp
  <PackageReference Include="Volo.Abp.OpenIddict.Pro.HttpApi" Version="6.0.*" />
  ```

- In **MyApplicationHttpApiModule.cs** replace the usings and **module dependencies**:

  ```csharp
  using Volo.Abp.IdentityServer;
  ...
  typeof(AbpIdentityServerHttpApiModule),
  ```

  with 

  ```csharp
  using Volo.Abp.OpenIddict;
  ...
  typeof(AbpOpenIddictProHttpApiModule),
  ```

### HttpApi.Client Layer

- In **MyApplication.HttpApi.Client.csproj** replace the **project reference**:

  ```csharp
  <PackageReference Include="Volo.Abp.IdentityServer.HttpApi.Client" Version="6.0.*" />
  ```

  with   

  ```csharp
  <PackageReference Include="Volo.Abp.OpenIddict.Pro.HttpApi.Client" Version="6.0.*" />
  ```

- In **MyApplicationHttpApiModule.cs** replace the usings and **module dependencies**:

  ```csharp
  using Volo.Abp.IdentityServer;
  ...
  typeof(AbpIdentityServerHttpApiClientModule),
  ```

  with 

  ```csharp
  using Volo.Abp.OpenIddict;
  ...
  typeof(AbpOpenIddictProHttpApiClientModule),
  ```

### Test Project

- In **MyApplicationTestBaseModule.cs** **remove** the IdentityServer related using and PreConfigurations:

  ```csharp
  using Volo.Abp.IdentityServer;
  ```

  and

  ```csharp
  PreConfigure<AbpIdentityServerBuilderOptions>(options =>
  {
      options.AddDeveloperSigningCredential = false;
  });
  
  PreConfigure<IIdentityServerBuilder>(identityServerBuilder =>
  {
      identityServerBuilder.AddDeveloperSigningCredential(false, System.Guid.NewGuid().ToString());
  });
  ```

  from `PreConfigureServices`.
  
### Migration
Add a new migration that reflects the underlying changes via 
dotnet ef migrations add AbpOpenIddictMigration

### UI Layer

- [Angular UI Migration](openIddict-angular.md)
- [MVC/Razor UI Migration](openIddict-mvc.md)
- [Blazor-Server UI Migration](openIddict-blazor-server.md)
- [Blazor-Wasm UI Migration](openIddict-blazor.md)

## Source code of the samples and module

* [Open source tiered & separate auth server application migrate Identity Server to OpenIddct](https://github.com/abpframework/abp-samples/tree/master/Ids2OpenId)
* [Commercial tiered & separate auth server application migrate Identity Server to OpenIddct](https://abp.io/Account/Login?returnUrl=/api/download/samples/Ids2OpenId)
* [OpenIddict module document](https://docs.abp.io/en/abp/6.0/Modules/OpenIddict)
* [OpenIddict module source code](https://github.com/abpframework/abp/tree/rel-6.0/modules/openiddict)

## See Also

* [ABP Version 6.0 Migration Guide](Abp-6_0.md)
