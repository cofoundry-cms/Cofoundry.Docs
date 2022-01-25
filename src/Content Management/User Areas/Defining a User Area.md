To get started with a new user area, you first need to create a definition class that inherits from `IUserAreaDefinition`. This definition class will be registered and set up automatically. 

Below is an example for a members area:

```csharp
using Cofoundry.Domain;

public class MemberUserArea : IUserAreaDefinition
{
    /// <summary>
    /// By convention we add a constant for the user area code
    /// to make it easier to reference.
    /// </summary>
    public const string Code = "MEM";

    /// <summary>
    /// A unique 3 letter code identifying this user area. The cofoundry 
    /// user are uses the code "COF" so you can pick anything else!
    /// </summary>
    public string UserAreaCode { get; } = Code;

    /// <summary>
    /// Display name of the area, used in the Cofoundry admin panel
    /// as the navigation link to manage your users
    /// </summary>
    public string Name { get; } = "Member";

    /// <summary>
    /// Indicates if users in this area can login using a password. If this is false
    /// the password field will be null and login will typically be via SSO or some 
    /// other method.
    /// </summary>
    public bool AllowPasswordLogin { get; } = true;

    /// <summary>
    /// Indicates whether the user should login using thier email address as the username.
    /// Some SSO systems might provide only a username and not an email address so in
    /// this case the email address is allowed to be null.
    /// </summary>
    public bool UseEmailAsUsername { get; } = true;
    
    /// <summary>
    /// The path to a login page to use when a user does not have permission to 
    /// access a resource. The path to the denied resource is appended to the query
    /// string of the LoginPath using the parameter name "ReturnUrl".
    /// </summary>
    public string LoginPath { get; } = "/members/auth/login";
    
    /// <summary>
    /// Cofoundry creates an auth scheme for each user area. Use this property to set this
    /// user area as the default auth scheme, which means the HttpContext.User property will
    /// be set to this identity.
    /// </summary>
    public bool IsDefaultAuthScheme { get; } = true;
    
    /// <summary>
    /// Optionally implement this method to configure any additional settings.
    /// </summary>
    /// <param name="options">
    /// The current configuration with any values from configuration providers (e.g. appsettings.json) applied.
    /// </param>
    public void ConfigureOptions(UserAreaOptions options)
    {
        // Default: No additional config
    }
}
```

## Roles

Roles are scoped to a user area, and so your user area will require one or more roles to add users to. Roles can be created manually via the admin panel, but you can also define roles in code using `IRoleDefinition`, which is particularly useful if you need to refer to specific roles programmatically.

For more information on defining roles take a look at the [roles documentation](/framework/roles-and-permissions).

## User Areas in the Admin Panel

A new section for managing your users will be added automatically to the admin panel. From here you can create and manage new users and perform various other management tasks such as resetting passwords (if applicable).

## Multiple User Areas

It's rare that you would want to define more than one custom user area, but it is possible. In cases where you have different levels of membership you would typically use roles or permissions to describe that relationship instead. User areas are separate and cannot share users or roles; each area exist in isolation and a user must be signed up to each area independently. However, in rare cases this level of isolation is exactly what you're after! 


