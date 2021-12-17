
## Customizing Email Formatting

When adding or updating a user, their email address goes through a two-phase formatting processes:

- **Normalization:** The process of tidying up an email into a consistent format which can still be used to contact the user. By default we trim the value and lowercase the domain, but make no other alterations to ensure that the email is preserved as the user intended. For Example "Eric@EXAMPLE.COM" becomes "Eric@example.com".
- **Uniquification:** The process of formatting an email into a format that can be used to prevent duplicate registrations via a uniqueness check, but is not used for contacting a user. By default we normalize and lowercase the email, for example "Eric@EXAMPLE.COM" becomes "eric@example.com".

By default these processes are very similar because there's not a 100% effective method for preventing duplicate registrations and any implementation may have side-effects that are undesirable in some scenarios. However you can customize both these processes if you want to alter the behavior. To override the process for a given user area, implement one of these interfaces:

- `IEmailAddressNormalizer<TUserArea>`
- `IEmailAddressUniquifier<TUserArea>`

Once defined, your custom formatter will automatically be registered by the Cofoundry DI system and will be used in place of the default for your user area. Here's an example `IEmailAddressUniquifier` implementation that prevents multiple registrations from the same gmail account:

```csharp
using Cofoundry.Core;
using Cofoundry.Domain;

public class CustomerEmailAddressUniquifier : IEmailAddressUniquifier<CustomerUserArea>
{
    private readonly IEmailAddressNormalizer _emailAddressNormalizer;

    public CustomerEmailAddressUniquifier(IEmailAddressNormalizer emailAddressNormalizer)
    {
        _emailAddressNormalizer = emailAddressNormalizer;
    }

    public NormalizedEmailAddress UniquifyAsParts(string emailAddress)
    {
        var normalized = _emailAddressNormalizer.NormalizeAsParts(emailAddress);
        return UniquifyAsParts(normalized);
    }

    public NormalizedEmailAddress UniquifyAsParts(NormalizedEmailAddress emailAddressParts)
    {
        const string GMAIL_DOMAIN = "gmail.com";

        if (emailAddressParts == null) return null;

        // merge both gmail domains as they point to the same inbox
        // ignore any plus addressing and remove superflous dots for gmail addresses only
        var uniqueEmail = emailAddressParts
            .MergeDomains(GMAIL_DOMAIN, "googlemail.com")
            .AlterIf(email => email.HasDomain(GMAIL_DOMAIN), email =>
            {
                return email
                    .WithoutPlusAddressing()
                    .AlterLocal(local => local.Replace(".", string.Empty));
            });

        return uniqueEmail;
    }
}
```

With the above example in place, a customer trying to register two variations of the same gmail account will receive a validation error.

## Customizing Username Formatting

If your user area is not configured to use an email address as the username, you may want to customize the username formatting which uses a similar two-step process:

- **Normalization:** The process of tidying up an username into a consistent format. This value may be used for display and so it's rare that you'd want to change it dramatically, therefore the default implementation simply trims the input e.g. " E.Example" becomes "E.Example".
- **Uniquification:** The process of formatting a username into a format that can be used for comparing usernames, e.g. to prevent duplicate registrations via a uniqueness check and to lookup a user during login. By default we normalize and lowercase the username, for example "E.Example" becomes "e.example".

To override the process for a given user area, implement one of these interfaces:

- `IUsernameNormalizer<TUserArea>`
- `IUsernameUniquifier<TUserArea>`

Once defined, your custom formatter will automatically be registered by the Cofoundry DI system and will be used in place of the default for your user area. Be careful when customizing this behavior on an existing user area, because any changes can break compatibility with existing users defined using the default formatters.