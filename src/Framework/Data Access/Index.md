## CQS, IQuery & ICommand 

Under the covers Cofoundry uses a lightweight [CQS](CQS) framework to structure data access. You can structure your data access any way you want, but this framework is there for you to use if you want to benefit from:

- A best-practice and structured methodology
- Integrated validation
- Integrated permission handling
- Integrated logging

[Go to section](CQS) 

## IDomainRepository

`IDomainRepository` is the base repository that `IContentRepository` and `IAdvancedContentRpository` inherit from. It provides easy access to components such as generic command and query execution, permission escalation and transaction management. 

[Go to section](idomainrepository)

## Transactions

Cofoundry has a transaction manager abstraction that provides a few benefits over `System.Transactions`.

[Go to section](Transactions) 

## Entity Framework

Cofoundry uses Entity Framework Core for data access and includes a few tools you may wish to use if you are also using EF Core for your own data.

[Go to section](Entity-Framework-and-DbContext-Tools) 

## Database Migrations

Cofoundry does not use EF Database Migrations and instead we use our own framework for managing database updates via SQL scripts. This framework is also available to you as well as plugin developers.

For more information see the [Auto Update Documentation](Auto-Update)

