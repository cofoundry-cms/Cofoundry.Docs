You create a new user area by creating a class that inherits from `IUserAreaDefinition`:

```csharp
using Cofoundry.Domain;

public class MemberUserArea : IUserAreaDefinition
{
    public static string AreaCode = "MEM";

    /// <summary>
    /// Indicates if users in this area can login using a password. If this is false
    /// the password field will be null and login will typically be via SSO or some 
    /// other method.
    /// </summary>
    public bool AllowPasswordLogin { get; } = true;

    /// <summary>
    /// Display name of the area, used in the Cofoundry admin panel
    /// as the navigation link to manage your users
    /// </summary>
    public string Name { get; } = "Site Members";

    /// <summary>
    /// Indicates whether the user should login using thier email address as the username.
    /// Some SSO systems might provide only a username and not an email address so in
    /// this case the email address is allowed to be null.
    /// </summary>
    public bool UseEmailAsUsername { get; } = true;

    /// <summary>
    /// A unique 3 letter code identifying this user area. The cofoundry 
    /// user are uses the code "COF" so you can pick anything else!
    /// </summary>
    public string UserAreaCode { get; } = AreaCode;
    
    /// <summary>
    /// The url to redirect to to when not authenticated.
    /// </summary>
    public string LoginPath { get; } = "/admin/auth/login";
    
    /// <summary>
    /// Cofoundry creates an auth schema for each user area. Use this property to set this
    /// user area as the default auth schema, which means the HttpContext.User property will 
    /// be set to this identity.
    /// </summary>
    public bool IsDefaultAuthSchema { get; } = false;
}
```

This definition class will be registered by the DI container and set up automatically. A new section for managing the users in your user area will be added to the admin panel.


