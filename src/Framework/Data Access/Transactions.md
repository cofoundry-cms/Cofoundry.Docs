Cofoundry includes `ITransactionScopeManager` for managing transactions across multiple statements and handling nested scopes.

```csharp
public class AddBlogPostHandler
{
    private readonly MyDbContext _myDbContext;
    private readonly ITransactionScopeManager _transactionScopeManager;

    public AddBlogPostHandler(
        MyDbContext myDbContext,
        ITransactionScopeManager transactionScopeManager
        )
    {
        _myDbContext = myDbContext;
        _transactionScopeManager = transactionScopeManager;
    }

    public async Task ExecuteAsync(AddBlogPostCommand command)
    {
        using (var scope = _transactionScopeManager.Create(_myDbContext))
        {
            // …code to create and add a draft blog post to the context
            await _myDbContext.SaveChangesAsync();

            var publishBlogPostCommand = new PublishBlogPostCommand();
            // …set some variables on the PublishBlogPostCommand
            await PublishBlogPostAsync(command);

            await scope.CompleteAsync();
        }
    }
}
```

## Accessing via Repositories

Instead of using `ITransactionScopeManager` directly, it's often simpler to access this functionality from our [repository classes](/content-management/accessing-data-programmatically) such as [`IDomainRepository`](idomainrepository). In this case, the database connection associated with the ambient Cofoundry DbContext is used.

```csharp
public class TransactionExample
{
    private readonly IAdvancedContentRepository _contentRepository;

    public TransactionExample(
        IAdvancedContentRepository contentRepository
        )
    {
        _contentRepository = contentRepository;
    }

    public async Task RunExample()
    {
        using (var scope = _contentRepository.Transactions().CreateScope())
        {
            var addDirectoryCommand = new AddPageDirectoryCommand();
            // …set some properties on the command
            await _contentRepository
                .PageDirectories()
                .AddAsync(addDirectoryCommand);

            var addPageCommand = new AddPageCommand();
            // …set some properties on the command
            await _contentRepository
                .Pages()
                .AddAsync(addPageCommand);

            await scope.CompleteAsync();
        }
    }
}
```

## Why not use System.Transaction.TransactionScope?

Before the re-introduction of System.Transactions in .NET Core 2.0 `ITransactionScopeManager` was a wrapper around EF transactions which added support for nested scopes. Now that System.Transactions is available the default `ITransactionScopeManager` implementation uses `System.Transaction.TransactionScope` to manage nested scopes, so it might seem pointless to use this abstraction, however there are some additional benefits of using the Cofoundry implementation:

- Deferred execution of "completion tasks" in nested transactions
- Better defaults for scope creation

## Deferred execution of "completion tasks"

`ITransactionScopeManager` adds the ability to defer execution of tasks until after the transaction is complete. Cofoundry uses this internally to ensure that message handlers and cache breaking code does not run until the top-level parent transaction is complete.

Here's an example based on a simplified [`DeletePageCommandHandler`](https://github.com/cofoundry-cms/cofoundry/blob/master/src/Cofoundry.Domain/Domain/Pages/Commands/DeletePageCommandHandler.cs). In the sample  `scope.QueueCompletionTask` is used to ensure the message publication and cache clearing code is only run when the parent transaction is complete. 

```csharp
public class DeletePageCommandHandler
{
    // …constructor removed for brevity
        
    public async Task ExecuteAsync(DeletePageCommand command, IExecutionContext executionContext)
    {
        var page = await _dbContext
            .Pages
            .FilterByPageId(command.PageId)
            .SingleOrDefaultAsync()

        if (page == null) return;

        _dbContext.Pages.Remove(page);
            
        using (var scope = _transactionScopeFactory.Create(_dbContext))
        {
            await _commandExecutor.ExecuteAsync(new DeleteUnstructuredDataDependenciesCommand(PageEntityDefinition.DefinitionCode, command.PageId), executionContext);
            await _dbContext.SaveChangesAsync();
            await _pageStoredProcedures.UpdatePublishStatusQueryLookupAsync(command.PageId);

            scope.QueueCompletionTask(() => OnTransactionComplete(command));

            await scope.CompleteAsync();
        }
    }

    private Task OnTransactionComplete(DeletePageCommand command)
    {
        _pageCache.Clear(command.PageId);

        return _messageAggregator.PublishAsync(new PageDeletedMessage()
        {
            PageId = command.PageId
        });
    }
}
```

It's unlikely that anyone implementing a site using Cofoundry will need to use the task queuing features directly, but if you're running built-in Cofoundry commands from your code in transactions then it's best to manage the scopes using `ITransactionScopeManager` to ensure internal Cofoundry code runs as expected.

## Better defaults for scope creation

`ITransactionManager` uses the following defaults when creating a new `TransactionScope`:

- `TransactionScopeAsyncFlowOption.Enabled`: Since we should always use async for database access, we enable this by default
- `IsolationLevel.ReadCommitted`: EF uses a default isolation level of `Serializable` for backwards compatibility, but this is [rarely the best choice](https://joshthecoder.com/2020/07/27/transactionscope-considered-annoying.html) and doesn't align with the SQL Server default. Instead Cofoundry uses `ReadCommitted` by default.

### Customizing scope creation

#### Per Transaction

To change the scope creation settings there are several extensions you can use. These extensions are specific to our default Cofoundry implementation and so you will need to reference the namespace `Cofoundry.Core.Data.TransactionScopeManager.Default` to use them.

```csharp

using Cofoundry.Domain;
using Cofoundry.Domain.TransactionManager.Default;

public class TransactionExample
{
    private readonly IAdvancedContentRepository _contentRepository;

    public TransactionExample(
        IAdvancedContentRepository contentRepository
        )
    {
        _contentRepository = contentRepository;
    }

    public async Task RunExample()
    {
        using (var scope = _contentRepository
                .Transactions()
                .CreateScope(TransactionScopeOption.Required, IsolationLevel.Serializable))
        {
            // …code removed for brevity

            await scope.CompleteAsync();
        }
        
        using (var scope = _contentRepository
                .Transactions()
                .CreateScope(CreateScope))
        {
            // …code removed for brevity

            await scope.CompleteAsync();
        }
    }
    
    private TransactionScope CreateScope()
    {
        var options = new TransactionOptions()
        {
            IsolationLevel = IsolationLevel.Serializable,
            Timeout = TransactionManager.DefaultTimeout
        };

        return new TransactionScope(
            TransactionScopeOption.Required,
            options,
            TransactionScopeAsyncFlowOption.Suppress
            );
    }
}
```

#### Customizing the defaults

To change the default settings for scope creation, create your own [`ITransactionScopeFactory`](https://github.com/cofoundry-cms/cofoundry/blob/master/src/Cofoundry.Core/Data/Transactions/TransactionScopeFactory.cs) implementation and override it using the [DI System](/framework/dependency-injection).