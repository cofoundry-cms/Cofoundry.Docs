## Complete Sample Sites

The complete sample sites are the best way to get a feel for how a Cofoundry site works and what the code looks like. 

### [Cofoundry.Samples.SimpleSite](https://github.com/cofoundry-cms/Cofoundry.Samples.SimpleSite)

A simple website implementing content management and some framework features including:

- Startup registration
- Page templates
- Custom page block types
- Image content
- Two custom entities - blog post and category
- Querying and display a list of blog posts
- A blog post custom entity details page
- A simple contact form
- Email notifications & email templating
- Custom error pages
- Configuration settings

[View on GitHub](https://github.com/cofoundry-cms/Cofoundry.Samples.SimpleSite)


### [Cofoundry.Samples.SPASite](https://github.com/cofoundry-cms/Cofoundry.Samples.SPASite)

An example demonstrating how to use Cofoundry to create a SPA (Single Page Application) with WebApi endpoints as well as demonstrating some advanced Cofoundry features.

The application is also separated into two projects demonstrating an example of how you might structure your domain layer to keep this layer separate from your web layer.

- Startup registration
- Web Api and use of `IApiResponseHelper`
- Structuring commands and queries using CQS 
- Multiple related custom entities - Cats, Breeds and Features
- A member area with a sign-up and login process
- Using an entity framework DbContext to represent custom database tables
- Executing stored procedures using `IEntityFrameworkSqlExecutor`
- Integrating custom entity data with entity framework data access
- Using the auto-updater to run sql scripts
- Email notifications & email templating
- Registering services with the DI container

[View on GitHub](https://github.com/cofoundry-cms/Cofoundry.Samples.SPASite)

## Individual Samples

These samples focus on specific areas of functionality in Cofoundry.

### [Cofoundry.Samples.Menus](https://github.com/cofoundry-cms/Cofoundry.Samples.Menus)

A bare website showing various examples of creating content editable menus:

- Simple Menu
- Nested Menu
- Multi-level Menu

[View on GitHub](https://github.com/cofoundry-cms/Cofoundry.Samples.Menus)

### [Cofoundry.Samples.PageBlockTypes](https://github.com/cofoundry-cms/Cofoundry.Samples.PageBlockTypes)

A bare website showing various examples of how to implement page block types including

- Using data model attributes
- Querying for data
- Managing related entities and versions

[View on GitHub](https://github.com/cofoundry-cms/Cofoundry.Samples.PageBlockTypes)
