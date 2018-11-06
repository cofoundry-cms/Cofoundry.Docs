Cofoundry includes a service for cleaning and sanitizing html which helps you prevent XSS attacks when rendering user input. 

To use this you can simply request `IHtmlSanitizer` from the DI container, or more commonly you might use the [Cofoundry View Helper](/content-management/cofoundry-view-helper) directly from a view:

```
@using Cofoundry.Domain

@model MyTestViewModel
@inject ICofoundryHelper<MyTestViewModel> Cofoundry

<div>
     @Cofoundry.Sanitizer.Sanitize("<h1>My Heading</h1><script>alert('uh oh')</script>")
</div>

```

## Customizing the sanitization ruleset

The Cofoundry HtmlSanitizer relies on the excellent [mganss/HtmlSanitizer](https://github.com/mganss/HtmlSanitizer) package to perform sanitization and uses it's default ruleset with a couple of modifications – adding the `class` attribute and the `mailto` url scheme to the whitelist. This means the default ruleset is quite liberal, so you may wish to create your own ruleset or modify the default if you want to be more restrictive. 

Here's an example that shows how to create a ruleset, using the mganss sanitizer defaults.

```csharp
var ruleSet = new HtmlSanitizationRuleSet();

// Add defaults
ruleSet.PermittedAttributes = Ganss.XSS.HtmlSanitizer.DefaultAllowedAttributes;
ruleSet.PermittedCssProperties = Ganss.XSS.HtmlSanitizer.DefaultAllowedCssProperties;
ruleSet.PermittedSchemes = Ganss.XSS.HtmlSanitizer.DefaultAllowedSchemes;
ruleSet.PermittedTags = Ganss.XSS.HtmlSanitizer.DefaultAllowedTags;
ruleSet.PermittedUriAttributes = Ganss.XSS.HtmlSanitizer.DefaultUriAttributes;

// modify rules
ruleSet.PermittedAttributes.Add("class");

```

To use the ruleset you can pass it in as a parameter to the sanitize method:

```
var html = "<h1>My Heading</h1><script>alert('uh oh')</script>";
_htmlSanitizer.Sanitize(html, ruleSet)

```

## Changing the default ruleset

If you need to change the default ruleset, you can do so by replacing the default `IDefaultHtmlSanitizationRuleSetFactory` implementation using the [DI override system](dependency-injection#overriding-registrations). 

Here's an example `IDefaultHtmlSanitizationRuleSetFactory` implementation.

```csharp
using Cofoundry.Core.Web;

public class ExampleHtmlSanitizationRuleSetFactory : IDefaultHtmlSanitizationRuleSetFactory
{
    private Lazy<HtmlSanitizationRuleSet> _defaultRulset = new Lazy<HtmlSanitizationRuleSet>(Initizalize);
    
    public IHtmlSanitizationRuleSet Create()
    {
        return _defaultRulset.Value;
    }

    private static HtmlSanitizationRuleSet Initizalize()
    {
        var ruleSet = new HtmlSanitizationRuleSet();

        ruleSet.PermittedAttributes = Ganss.XSS.HtmlSanitizer.DefaultAllowedAttributes;
        ruleSet.PermittedCssProperties = Ganss.XSS.HtmlSanitizer.DefaultAllowedCssProperties;
        ruleSet.PermittedSchemes = Ganss.XSS.HtmlSanitizer.DefaultAllowedSchemes;
        ruleSet.PermittedTags = Ganss.XSS.HtmlSanitizer.DefaultAllowedTags;
        ruleSet.PermittedUriAttributes = Ganss.XSS.HtmlSanitizer.DefaultUriAttributes;
        ruleSet.PermittedAttributes.Add("class");

        return ruleSet;
    }
}
```
