Cofoundry uses a fairly simple permissions system. At the most basic level a *permission* represents a single action and a *role* represents a collection of many permitted permissions. Roles are assigned to users, or if you are not logged in a special *Anonymous* role is assigned to you.

## Managing Roles & Permissions

Roles & permissions can be managed either through the admin panel or in code.

#### Managing Roles in the Admin Panel

Roles can be managed in the admin panel in the *Roles* section, where you can add, remove and update permissions on a role. Note that the two built-in roles (*Anonymous* and *Super Administrator*) cannot be deleted from the admin panel because they are used internally by Cofoundry. You can manage the permissions assigned to the anonymous role.

#### Managing Roles in Code

Managing roles in code has the advantage of automatically being added to the system on startup. Additionally you also define a unique string code for the role that makes it easier and more reliable to query the role programmatically.

To define a role in code, create a class that implements `IRoleDefinition`:

```csharp
using Cofoundry.Domain;

public class MarketingRole : IRoleDefinition
{
    public const string MarketingRoleCode = "MKT";

    /// <summary>
    /// The role title is used to identify the role and select it in the admin 
    /// UI and therefore must be unique. Max 50 characters.
    /// </summary>
    public string Title =>  "Marketing";

    /// <summary>
    /// The role code is a unique three letter code that can be used to reference the
    /// role programmatically. The code must be unique and convention is to use upper 
    /// case, although code matching is case insensitive.
    /// </summary>
    public string RoleCode => MarketingRoleCode;

    /// <summary>
    /// A role must be assigned to a user area, in this case we are adding it to
    /// the Cofoundry user area so they will have access to the Cofoundry admin panel.
    /// </summary>
    public string UserAreaCode => CofoundryAdminUserArea.AreaCode;
}

```

To add permissions to this role you define a class that implements `IRoleInitializer<TRoleDefinition>`

```csharp
using Cofoundry.Domain;

public class MarketingRoleInitializer : IRoleInitializer<MarketingRole>
{
    /// <summary>
    /// This method determins what permissions the role has. To help do this
    /// you are provided with a collection of all permissions in the system and
    /// you can query the collection to either filter out permissions you dont want
    /// or create a new collection and mix permissions in that you do want. There are
    /// a number of extension method on the collection that make this easier.
    /// </summary>
    public IEnumerable<IPermission> GetPermissions(IEnumerable<IPermission> allPermissions)
    {
        return allPermissions
            .ExceptDeletePermissions()
            .ExceptUserManagementPermissions()
            .ExceptEntityPermissions<RoleEntityDefinition>()
            .ExceptPermission<SettingsAdminModulePermission>()
            ;
    }
}
```

The initializer runs when the role is first created, and will only run again when new permissions are added to the system, initializing only the new permissions for the role. 

After a role has been initialized it can be managed in the admin panel.

#### Customizing the Anonymous Role

By default the anonymous role only includes read permissions to entities (excluding users). If you want to customize this you can create a class that inherits from `IRoleInitializer<AnonymousRole>` in the same way as described above for custom roles.

## Creating Permissions

Permissions already exists for Cofoundry entities and are dynamically generated for any custom entities you create, so you'll not often need to create your own permissions. 

Permissions are created in the domain layer and come in two flavors:

- *IPermission*: A basic permission for an action not associated with a specific entity e.g. 'View Error Log'
- *IEntityPermission*: A permission that relates to an entity type, e.g. 'Add Page' or 'Delete Image Asset'

Typically when developing a new area of functionality, it will be based around a specific entity type.

Permissions should sit in the **Domain Layer** because they are principally applied when executing queries and commands. By defining permissions at the domain level we ensure that permissions are always enforced at the data access level, irrespective of where the request comes from (e.g. api/page/console app).

Cofoundry defines it's permissions using the same system, so check out the [source code](https://github.com/cofoundry-cms/cofoundry/tree/master/src/Cofoundry.Domain/Domain/) for examples. 

Using [Pages](https://github.com/cofoundry-cms/cofoundry/tree/master/src/Cofoundry.Domain/Domain/Pages/Permissions) as an example, you can see each permission has its own C# class and the permissions all inherit from `IEntityPermission`, all using the `PageEntityDefinition` for the `EntityDefinition` property, some using the predefined `PermissionType` in the `CommonPermissionTypes` constants library (e.g. CRUD operations) and some using custom `PermissionType` objects (e.g. `PageUpdateUrlPermission`).

#### Example IPermission Implementation

```csharp
public class ErrorLogReadPermission : IPermission
{
    public ErrorLogReadPermission()
    {
        PermissionType = new PermissionType("ERRLOG", "Error logs", "View error logs in the admin panel");
    }

    public PermissionType PermissionType { get; private set; }
}
```

#### Example IEntityPermission Implementation


```csharp
public class PageReadPermission : IEntityPermission
{
    public IEntityDefinition EntityDefinition => new PageEntityDefinition();
    
    public PermissionType PermissionType => CommonPermissionTypes.Read("Pages");
}
```


## Enforcing Permissions

There are a number of ways to enforce a permission:

#### IPermissionRestrictedCommandHandler & IPermissionRestrictedQueryHandler

Permissions enforcement is built right into the [CQS Framework](data-access/CQS) we use to structure data access. 

The most common way to enforce a permission will be implement `IPermissionRestrictedCommandHandler` or `IPermissionRestrictedQueryHandler` on your command/query handler. When implemented this will automatically call the `GetPermissions` method on the handler to enforce any permissions you return. 

```csharp
using Cofoundry.Domain;
using Cofoundry.Domain.CQS;

public class AddPageCommandHandler
    : IAsyncCommandHandler<AddPageCommand>
    , IPermissionRestrictedCommandHandler<AddPageCommand>
{
    public async Task ExecuteAsync(AddPageCommand command, IExecutionContext executionContext)
    {
        // execution logic removed
    }

    public IEnumerable<IPermissionApplication> GetPermissions(AddPageCommand command)
    {
        yield return new PageCreatePermission();

        if (command.Publish)
        {
            yield return new PagePublishPermission();
        }
    }
}
```

In rare cases you might need to permit an action if one of a combination of permissions are present. For this you can use `CompositePermissionApplication`:

```csharp
public IEnumerable<IPermissionApplication> GetPermissions(AddPageCommand command)
{
    var createPermission = new PageCreatePermission();
    var updatePermission = new PageUpdatePermission();
    
    yield return new CompositePermissionApplication(createPermission, updatePermission);
}
```

Note that you never need to check for a *READ* permission as well as other entity permissions because a user is required to have read permissions to an entity before being assigned other permissions for that entity.

#### IPermissionValidationService

Sometimes you need to do more complex permissions checking in a query or command, typically because you need to read from a database before you can work out what permissions are required. In this case you use the `IPermissionValidationService` directly within the `ExecuteAsync` method, an example of this is in `DeleteCustomEntityCommandHandler` where we don't know the custom entity type until we have pulled it out the db:

```csharp
using Cofoundry.Domain.Data;
using Cofoundry.Domain;
using Cofoundry.Domain.CQS;

public class DeleteCustomEntityCommandHandler
    : ICommandHandler<DeleteCustomEntityCommand>
    , IIgnorePermissionCheckHandler
{
    private readonly CofoundryDbContext _dbContext;
    private readonly IPermissionValidationService _permissionValidationService;

    public DeleteCustomEntityCommandHandler(
        CofoundryDbContext dbContext,
        IPermissionValidationService permissionValidationService
        )
    {
        _dbContext = dbContext;
        _permissionValidationService = permissionValidationService;
    }

    public async Task ExecuteAsync(DeleteCustomEntityCommand command, IExecutionContext executionContext)
    {
        var customEntity = await _dbContext
            .CustomEntities
            .SingleOrDefaultAsync(p => p.CustomEntityId == command.CustomEntityId);

        if (customEntity != null)
        {
            _permissionValidationService.EnforceCustomEntityPermission<CustomEntityDeletePermission>(
                customEntity.CustomEntityDefinitionCode, 
                executionContext.UserContext
                );

            // logic code removed
        }
    }
}
```

#### Simple Permission Handling with ILoggedInPermissionCheckHandler and ICofoundryUserPermissionCheckHandler

If your application only requires simple permission checking to make sure a user is logged in you can implement `ICofoundryUserPermissionCheckHandler` or `ILoggedInPermissionCheckHandler` which allows you to simply check that a user is logged in or is a Cofoundry user.

#### IIgnorePermissionCheckHandler

If your query/command does not need permission restrictions then it must implement `IIgnorePermissionCheckHandler`. This is done so that you don't accidentally forget to apply permissions to a query/command.

This might be the case if your handler just wraps calls to other queries or commands that already handle permissions.

## Checking User Permissions

### In C# Back-End Code

To check user permissions in code you can use methods on `IPermissionValidationService`.

### In C# Razor Views

The [Cofoundry View Helper](/content-management/cofoundry-view-helper) has a `CurrentUser` property that can be used to retrieve user information by calling `await Cofoundry.CurrentUser.GetAsync()`. Once fetched the `CurrentUserViewHelperContext` data is cached for the duration of the request, so don't worry about calling this method multiple times in different components.

The `CurrentUserViewHelperContext` provides user and role information and will allow
e.g. `@Cofoundry.CurrentUser.Role`, which has helper methods like `HasPermission(IPermission permission)`.

```html
@inject ICofoundryHelper Cofoundry

@{
    var user = await Cofoundry.CurrentUser.GetAsync();
}

<h1> Permission Example</h1>

@if (user.Role.HasPermission<MyTestPermission>())
{
    <h2>Restricted Content</h2>
    <p>You have MyTestPermission</p>
}
@if (user.Role.IsSuperAdministrator)
{
    <h2>SuperAdmin Content</h2>
    <p>You are a super administrator</p>
}
@if (user.Role.UserArea.UserAreaCode == MyCustomUserArea.UserAreaCode)
{
    <h2>Custom User Area Content</h2>
    <p>Special content for MyCustomUserArea users</p>
}
```

### In Admin JS (Angular)

When developing angular modules for the admin panel you can use the `shared.permissionValidationService` to access helper methods for querying permissions for the currently logged in user.

