## IDomainRepository

`IDomainRepository` is the base repository that `IContentRepository` and `IAdvancedContentRepository` inherit from. It provides easy access to components such as generic command and query execution, permission escalation and transaction management. Typically you'd use this if your only interested in using Cofoundry as a framework without any of the CMS content APIs.

## Additional Features

### Executing queries and commands

Under the covers Cofoundry uses a lightweight [CQS](CQS) framework to structure data access. You can structure your data access any way you want, but this framework is there for you to use if you want to benefit from:

- A best-practice and structured methodology
- Integrated validation
- Integrated permission handling
- Integrated logging

Detailed information on using the CQS framework itself is [here](CQS).

Cofoundry repositories have the following methods to facilitate working with queries and commands:

- `ExecuteQueryAsync`: Directly executes an `IQuery` instance and returns the results.
- `ExecuteCommandAsync`: Directly executes an `ICommand` instance.
- `WithQuery`: Allows you to chain mutator functions to run after execution of a query such as `Map` or `MapItem`.

**Example:**

```csharp
var command = new ExampleCommand()
{
    ExampleId = 1
};
await _domainRepository.ExecuteCommandAsync(command);
```

Executing the queries in this way allows you to benefit from many of the additional features described here such as mapping, permission elevation and transaction management.


### Mapping

Queries executed using `WithQuery` can benefit from the same mapping features found in [content repository queries](/content-management/accessing-data-programmatically):

```csharp
var results = await _domainRepository
    .WithQuery(new SearchCustomEntityRenderSummariesQuery()
    {
        CustomEntityDefinitionCode = BlogPostCustomEntityDefinition.DefinitionCode
    })
    .MapItem(b => new { b.Title })
    .ExecuteAsync();
```

### Elevating Permissions

Execution of queries and commands is restricted by [permissions](/framework/roles-and-Permissions). If you want to run a query or command that currently signed in user doesn't have permissions for, you'll need to elevate your permissions before executing it.

An example of when this might be useful would be registering a new user from a public sign-up form. The anonymous user role does not typically have permissions to create a user, so we'd need to elevate permissions:

```csharp
public async Task RegisterUser(string email, string password)
{
    await _advancedContentRepository
        .WithElevatedPermissions()
        .Users()
        .AddAsync(new AddUserCommand()
        {
            Email = email,
            Password = password,
            UserAreaCode = MemberUserArea.Code,
            RoleCode = MemberRole.Code
        });
}
```

### Transactions

The Cofoundry transaction manager can be accessed via the `Transactions()` extension method:

```csharp
using (var scope = _domainRepository
    .Transactions()
    .CreateScope())
{
    // Do stuff
    await scope.CompleteAsync();
}
```

Read more about the Cofoundry transaction manager in the [transactions docs](transactions).