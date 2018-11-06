Cofoundry uses Entity Framework Core as its ORM strategy. You are free to use other tools and methodologies for your data access but using EF will allow you to re-use some of the existing Cofoundry features.

## DbContext Tools

There are a handful of ways to create a DbContext in EF, but our approach is to hand-code it using a few helpers to cut down on boilerplate. Here's an example:

```csharp
using Cofoundry.Core.EntityFramework;
using Microsoft.EntityFrameworkCore;

public class MySiteDbContext : DbContext
{
    private readonly ICofoundryDbContextInitializer _cofoundryDbContextInitializer;

    public MySiteDbContext(ICofoundryDbContextInitializer cofoundryDbContextInitializer)
    {
        _cofoundryDbContextInitializer = cofoundryDbContextInitializer;
    }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        _cofoundryDbContextInitializer.Configure(this, optionsBuilder);
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder
            .HasAppSchema()
            .ApplyConfiguration(new CatMap())
            .ApplyConfiguration(new DogMap());
    }

    public DbSet<Cat> Cats { get; set; }
    public DbSet<Dog> Dogs { get; set; }
}
```

See the [SPASite Sample](https://github.com/cofoundry-cms/Cofoundry.Samples.SPASite) for an example of this.

#### Constants

- `DbConstants.DefaultAppSchema`: The default/suggested schema for your applications tables "app", but you can alternatively use you own.
- `DbConstants.CofoundrySchema`: The schema for Cofoundry tables "Cofoundry"

#### Mapping

 - `modelBuilder.HasAppSchema()`: Shortcut for setting the default schema to 'app'.

#### CMS Data Class Mapping

You can mix classes from the Cofoundry DbContext into your own DbContext if you want to link to entities like Images, Users or Pages. To pull in the mappings you can use `modelBuilder.MapCofoundryContent()` in the `OnModelCreating` override.

#### Inheriting from CofoundryDbContext

If you're mixing your own data models with Cofoundry data models, you might find it easier to simply extend `CofoundryDbContext` with your own data models:

```csharp
using Cofoundry.Core.EntityFramework;
using Microsoft.EntityFrameworkCore;

public class MySiteDbContext : CofoundryDbContext
{
    public MySiteDbContext(ICofoundryDbContextInitializer cofoundryDbContextInitializer)
        : base(cofoundryDbContextInitializer)
    {
    }

    public DbSet<Cat> Cats { get; set; }
    public DbSet<Dog> Dogs { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        modelBuilder
            .HasAppSchema()
            .ApplyConfiguration(new CatMap())
            .ApplyConfiguration(new DogMap());
    }
}
```

## Executing Stored Procedures & Raw Sql 

`IEntityFrameworkSqlExecutor` helps you to execute raw SQL statements against an EF DbContext, including methods for executing table queries, scalar queries, commands and commands with an output parameter.

```csharp
using Cofoundry.Core.EntityFramework;
using Cofoundry.Domain;
using Cofoundry.Domain.CQS;
using System.Data.SqlClient;
using System.Threading.Tasks;

public class SetCatFavoriteCommandHandler 
    : IAsyncCommandHandler<SetCatFavoriteCommand>
    , ILoggedInPermissionCheckHandler
{
    private readonly IEntityFrameworkSqlExecutor _entityFrameworkSqlExecutor;
    private readonly CofoundryDbContext _dbContext;
    
    public SetCatFavoriteCommandHandler(
        IEntityFrameworkSqlExecutor entityFrameworkSqlExecutor,
        CofoundryDbContext dbContext
        )
    {
        _entityFrameworkSqlExecutor = entityFrameworkSqlExecutor;
        _dbContext = dbContext;
    }

    public Task ExecuteAsync(SetCatFavoriteCommand command, IExecutionContext executionContext)
    {
        return _entityFrameworkSqlExecutor
            .ExecuteCommandAsync(_dbContext, 
                "app.Cat_SetFavorite",
                new SqlParameter("@CatId", command.CatId),
                new SqlParameter("@UserId", executionContext.UserContext.UserId),
                new SqlParameter("@IsLiked", command.IsFavorite),
                new SqlParameter("@CreateDate", executionContext.ExecutionDate)
                );
    }
}
```

## Managing transactions

For the most part transaction management isn't necessary because EF wraps `SaveChanges()` in a transaction for us. For occasions when we need to run a transaction across multiple statements Cofoundry has `ITransactionScopeManager` which abstracts transactions and allows us to use nested transaction scopes.

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

#### Why not use System.Transaction.TransactionScope?

Before the re-introduction of System.Transactions in .NET Core 2.0 `ITransactionScopeManager` was a wrapper around EF transactions which added support for nested scopes. Now that System.Transactions is available the default `ITransactionScopeManager` implementation uses `System.Transaction.TransactionScope` to manage nested scopes, so it might seem pointless to use this abstraction.

The one feature `ITransactionScopeManager` does add is the ability to defer execution of tasks until after the transaction is complete. Cofoundry uses this internally to ensure that message handlers and cache breaking code does not run until the top-level parent transaction is complete.

Here's an example based on a simplified [`DeletePageCommandHandler`](https://github.com/cofoundry-cms/cofoundry/blob/master/src/Cofoundry.Domain/Domain/Pages/Commands/DeletePageCommandHandler.cs). In the sample  `scope.QueueCompletionTask` is used to ensure the message publication and cache clearing code is only run once any parent transaction is complete. 

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



