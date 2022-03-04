﻿The process for publishing and deploying a Cofoundry website is the same as [deploying a regular ASP.NET Core website](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/). There are however some things to consider:

## View pre-compilation

[View pre-compilation](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/view-compilation) is enabled by default in ASP.NET Core which prevents your view file source code from being published. This is a problem because Cofoundry relies on these files to initialize page and block templates. 

To work around this issue you'll need to add two settings to your .csproj project file and set them both to false:

- `MvcRazorExcludeViewFilesFromPublish`: Setting this to false makes sure view files are deployed with your website.
- `MvcRazorExcludeRefAssembliesFromPublish`: Setting this to false allows non-compiled views embedded in Cofoundry libraries and plugins to be loaded. 

**Example .csproj file:**

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>netcoreapp3.1</TargetFramework>
    <MvcRazorExcludeViewFilesFromPublish>false</MvcRazorExcludeViewFilesFromPublish>
    <MvcRazorExcludeRefAssembliesFromPublish>false</MvcRazorExcludeRefAssembliesFromPublish>
  </PropertyGroup>
  
  <!-- other nodes removed for clarity -->
  
</Project>
```

This is an area we intended to work on to reduce friction, and it is also something the ASP.NET team are looking to improve. This is tracked in [issue 137](https://github.com/cofoundry-cms/cofoundry/issues/137)

## Multi-instance & web farm deployment

If you're deploying to a multi-instance environment, there's a few issues you need to consider. These considerations are typically the same when deploying any ASP.NET application to a web farm, and it's best to consult the official [ASP.NET hosting docs](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/web-farm) to become familiar with these requirements.

### Caching

You'll need to configure caching to support a multi-server environment, otherwise you'll have issues with stale data. See the [caching docs](/framework/caching) for more details.

### Data Protection

You'll need to configure the ASP.NET Core data protection system to support multiple machines. This is not specific to Cofoundry so please refer to the [ASP.NET Core data protection documentation](https://docs.microsoft.com/en-us/aspnet/core/security/data-protection/configuration/overview) for more information.