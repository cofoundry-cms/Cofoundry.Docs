## CQS, IQuery & ICommand 

Under the covers Cofoundry uses a lightweight [CQS](CQS) framework to structure our data access. You can structure your data access any way you want, but this framework is there for you to use if you want to benefit from:

- A best-practice and structured methodology
- Integrated validation
- Integrated permission handling
- Integrated logging

Detailed information available [here](CQS) 

## Repositories

The one drawback of using a CQS framework is discoverability. To remedy this we've implemented some lightweight repositories that you can take advantage of to get easy to find method with intellisense support. The following repositories are registered with the DI container:

- `IImageAssetRepository`
- `IDocumentAssetRepository`
- `ICustomEntityRepository`
- `IPageRepository`
- `IPageBlockTypeRepository`
- `IPageDirectoryRepository`
- `IUserRepository`
- `IRoleRepository`
- `IRewriteRuleRepository`

## Entity Framework

Cofoundry uses Entity Framework Code First as its ORM strategy. You are free to use other tools and methodologies for your data access but using EF will allow you to re-use some of the existing Cofoundry features.

See the section on  [Entity Framework & DbContext Tools](Entity-Framework-&-DbContext-Tools) which details tools to help you:

- Set up your DbContext
- Run stored procedures
- Manage transactions

## Database Migrations

Cofoundry does not use EF Database Migrations and instead we use our own framework for managing database updates via sql scripts. This framework is also available to you as well as plugin developers.

For more information see the [Auto Update Documentation](Auto-Update)

