## Dynamic Page Routing

Cofoundry has content management features that let you create and manage your website pages dynamically using templates.

For more information about dynamic page routing see the [Pages documentation](Pages)

## Rule Base Routing

Cofoundry also supports traditional rule based routing so you can use regular MVC controllers and actions as normal, however to support dynamic pages Cofoundry has to control the registration of routes. To register your rule based routes you should implement an `IRouteRegistration` class, which will automatically be picked up by the Dependency Injector system and injected into the Cofoundry route registration at the right time.

#### Example IRouteRegistration

Here is an example `IRouteRegistration` class:

```csharp
public class RouteRegistration : IRouteRegistration
{
    public void RegisterRoutes(IRouteBuilder routeBuilder)
    {
        routeBuilder.MapRoute(
            "Default", 
            "{controller}/{action}/{id?}",
            new { controller = "Home", action = "Index" }
        );
    }
}
```

You can have multiple registration classes (in advanced scenarios you may wish to modularize your application).

#### Ordering route registration

For most scenarios you won't need to worry about the ordering of your route registration classes, but in some advanced scenarios you may wish to take advantage of these additional interfaces which help you control the ordering that route registrations apply:

- **`IOrderedRouteRegistration`:** Implement this interface to define a custom ordering value for your route registration. Although this value is an integer, the `RouteRegistrationOrdering` enum defines some predefined values which you can use.
- **`IRunBeforeRouteRegistration`:** This interface is used to indicate that the registration should be run before one or more other registrations (indicated by type) are executed.
- **`IRunAfterRouteRegistration`:** This interface is used to indicate that the registration should be run after one or more other registrations (indicated by type) are executed.

## Attribute Routing

ASP.NET Core attribute routing is supported so if you prefer to use attribute routing then just use that as you normal would.