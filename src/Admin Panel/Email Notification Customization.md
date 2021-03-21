Cofoundry allows you to customize the email notifications sent out by the admin panel. This can be useful to add your own branding or to add your own messaging to the templates.

This feature is demonstrated in the [Cofoundry.Samples.Mail](https://github.com/cofoundry-cms/Cofoundry.Samples.Mail) sample project.

## Creating an admin mail template builder

The first step is to override the default mail template builder by creating a class that implements `IUserMailTemplateBuilder<CofoundryAdminUserArea>`.

Once this interface is defined, the Cofoundry DI system will automatically find it and use it to create mail templates for the Cofoundry admin user area.

Each email notificaton has it's own builder function with a context parameter that contains all the key data to build that template:

- **BuildNewUserWithTemporaryPasswordTemplateAsync:** The email template that is used when a new user is created with a temporary password. The context contains their temporary password.
- **BuildPasswordChangedTemplateAsync:** The email template that is used when a user has their password reset by an administrator. The context contains data contains their new temporary password.
- **BuildPasswordResetByAdminTemplateAsync:** The email template that is used when a user requests to reset their password e.g. via a forgot password page. The context contains the parameters required to build a password reset url.  
- **BuildPasswordResetRequestedByUserTemplateAsync:** The email template that is used to notify a user that their password has been changed.

## Making use of ICofoundryAdminMailTemplateBuilder

Each builder method only needs to return an `IMailTemplate` instance, so you are free to use any template class and build it in any way you want. However, it is more likely that you will want to make adaptations to the existing templates, such as changing the layout, subjects or view files.

You can take advantage of the the default builder functions by making use of `ICofoundryAdminMailTemplateBuilder`, simply inject it into your builder and use the equivalent methods.

Here is an baseline implementation without any customization:

```csharp
using Cofoundry.Core.Mail;
using Cofoundry.Domain;
using Cofoundry.Domain.MailTemplates;
using Cofoundry.Domain.MailTemplates.AdminMailTemplates;

public class AdminMailTemplateBuilder : IUserMailTemplateBuilder<CofoundryAdminUserArea>
{
    private readonly ICofoundryAdminMailTemplateBuilder _cofoundryAdminMailTemplateBuilder;

    public AdminMailTemplateBuilder(
        ICofoundryAdminMailTemplateBuilder cofoundryAdminMailTemplateBuilder
        )
    {
        _cofoundryAdminMailTemplateBuilder = cofoundryAdminMailTemplateBuilder;
    }

    public async Task<IMailTemplate> BuildNewUserWithTemporaryPasswordTemplateAsync(NewUserWithTemporaryPasswordTemplateBuilderContext context)
    {
        var template = await _cofoundryAdminMailTemplateBuilder.BuildNewUserWithTemporaryPasswordTemplateAsync(context);
        return template;
    }

    public async Task<IMailTemplate> BuildPasswordChangedTemplateAsync(PasswordChangedTemplateBuilderContext context)
    {
        var template = await _cofoundryAdminMailTemplateBuilder.BuildPasswordChangedTemplateAsync(context);
        return template;
    }

    public async Task<IMailTemplate> BuildPasswordResetByAdminTemplateAsync(PasswordResetByAdminTemplateBuilderContext context)
    {
        var template = await _cofoundryAdminMailTemplateBuilder.BuildPasswordResetByAdminTemplateAsync(context);
        return template;
    }

    public async Task<IMailTemplate> BuildPasswordResetRequestedByUserTemplateAsync(PasswordResetRequestedByUserTemplateBuilderContext context)
    {
        var template = await _cofoundryAdminMailTemplateBuilder.BuildPasswordResetRequestedByUserTemplateAsync(context);
        return template;
    }
}
```

### Example Customization

Full examples can be found in the [Cofoundry.Samples.Mail sample project](https://github.com/cofoundry-cms/Cofoundry.Samples.Mail).

#### Changing the layout

This example simply customizes the layout file using the `LayoutFile` property, which can be useful for wrapping the default content with your own branding.

```csharp
public async Task<IMailTemplate> BuildNewUserWithTemporaryPasswordTemplateAsync(NewUserWithTemporaryPasswordTemplateBuilderContext context)
{
    // build the default template so we can modify any properties we want to customize
    var template = await _cofoundryAdminMailTemplateBuilder.BuildNewUserWithTemporaryPasswordTemplateAsync(context);

    // Change the layout file
    template.LayoutFile = "~/MailTemplates/_ExampleAdminMailLayout";

    return template;
}
```

### Changing the subject

This example shows you how to change the email subject:

```csharp
public async Task<IMailTemplate> BuildPasswordChangedTemplateAsync(PasswordChangedTemplateBuilderContext context)
{
    // build the default template
    var template = await _cofoundryAdminMailTemplateBuilder.BuildPasswordChangedTemplateAsync(context);

    // customize the subject, the optional {0} token is replaced with the application name
    template.SubjectFormat = "{0}: You've changed your password!";

    return template;
}
```

### Changing the view

In this example, the view file is customized, which is useful if you want to change the wording of the email, but don't need any additional properties in the template model.

```csharp
public async Task<IMailTemplate> BuildPasswordResetByAdminTemplateAsync(PasswordResetByAdminTemplateBuilderContext context)
{
    // build the default template
    var template = await _cofoundryAdminMailTemplateBuilder.BuildPasswordResetByAdminTemplateAsync(context);

    // customize the view file
    template.ViewFile = "~/MailTemplates/ExampleAdminPasswordResetByAdminMailTemplate";

    return template;
}
```



