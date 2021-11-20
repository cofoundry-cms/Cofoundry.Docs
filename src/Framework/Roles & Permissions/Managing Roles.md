
Roles and permissions can be managed either through the admin panel or defined and initialized in code.

## Managing Roles in the Admin Panel

Roles can be managed in the admin panel in the *Roles* section. Here you can add new roles that are managed within the interface, and you can also update the permissions of roles that have been defined in code. 

There are two built-in roles that cannot be removed:

- **Anonymous:** The role assigned to users who are not logged in i.e. browsing your site without logging in. The permissions for this role can be managed in the admin panel.
- **Super Administrator:** This is the default administrator role which is assigned all permissions. This role is not editable.

## Managing Roles in Code

Managing a role in code has two advantages: the first is that it is automatically added to the system on startup, the second is that code-based roles use a unique string code that makes it easier and more reliable to query the role programmatically.

To define a role in code, create a class that implements `IRoleDefinition`:

```csharp
using Cofoundry.Domain;

public class MarketingRole : IRoleDefinition
{
    /// <summary>
    /// By convention we add a constant for the role code
    /// to make it easier to reference.
    /// </summary>
    public const string Code = "MKT";

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
    public string RoleCode => Code;

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

The initializer runs when the role is first created, and will only run again when new permissions are added to the system. This ensures that you don't get any nasty surprises when new permissions are added to the system because any newly defined permissions are added using the rules defined in the role initializer.

After a role has been initialized it can be managed in the admin panel.

### Customizing the Anonymous Role

By default the anonymous role only includes read permissions to entities (excluding users). If you want to customize this you can create a class that inherits from `IRoleInitializer<AnonymousRole>` in the same way as described above for custom roles:

```csharp
public class ExampleAnonymousRoleInitializer : IRoleInitializer<AnonymousRole>
{
    public IEnumerable<IPermission> GetPermissions(IEnumerable<IPermission> allPermissions)
    {
        var additionalPermissions = allPermissions.FilterByType<ExamplePermission>();
        
        return allPermissions
            .FilterToAnonymousRoleDefaults()
            .Union(additionalPermissions);
    }
}
```



