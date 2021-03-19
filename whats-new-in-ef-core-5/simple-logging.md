<a href="https://entityframework-extensions.net/">**Improve EF Core performance with EF Extensions**</a>

<a href="https://entityframework-extensions.net/">
<img src="https://zzzprojects.github.io/images/logo/entityframework-extensions-pub.jpg" width="600" />
</a>

# Simple Logging

Simple Logging is the equivalent of `Database.Log` in EF6. It provides a simple way to get logs from EF Core without the need to configure any kind of external logging framework.

EF Core replaces `Database.Log` with a `LogTo` method called on `DbContextOptionsBuilder` in either `AddDbContext` or `OnConfiguring`. 

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder.LogTo(Console.WriteLine);
```

There are multiple overloads available which can be used in different use cases.

* Set the minimum log level
  * Example: `.LogTo(Console.WriteLine, LogLevel.Information)`
* Filter for only specific events:
  * Example: `.LogTo(Console.WriteLine, new[] {CoreEventId.ContextInitialized, RelationalEventId.CommandExecuted})`
* Filter for all events in specific categories:
  * Example: `.LogTo(Console.WriteLine, new[] {DbLoggerCategory.Database.Name}, LogLevel.Information)`
* Use a custom filter over event and level:
  * Example: `.LogTo(Console.WriteLine, (id, level) => id == RelationalEventId.CommandExecuting)`

Output format can be minimally configured \(API is in flux\) but the default output looks something like:

```bash
warn: 12/5/2019 09:57:47.574 CoreEventId.SensitiveDataLoggingEnabledWarning[10400] (Microsoft.EntityFrameworkCore.Infrastructure)
      Sensitive data logging is enabled. Log entries and exception messages may include sensitive application data, this mode should only be enabled during development.
dbug: 12/5/2019 09:57:47.581 CoreEventId.ShadowPropertyCreated[10600] (Microsoft.EntityFrameworkCore.Model.Validation)
      The property 'BlogId' on entity type 'Post' was created in shadow state because there are no eligible CLR members with a matching name.
info: 12/5/2019 09:57:47.618 CoreEventId.ContextInitialized[10403] (Microsoft.EntityFrameworkCore.Infrastructure)
      Entity Framework Core 5.0.0-dev initialized 'BloggingContext' using provider 'Microsoft.EntityFrameworkCore.SqlServer' with options: SensitiveDataLoggingEnabled
dbug: 12/5/2019 09:57:47.644 CoreEventId.ValueGenerated[10808] (Microsoft.EntityFrameworkCore.ChangeTracking)
      'BloggingContext' generated temporary value '-2147482647' for the 'Id' property of new 'Blog' entity.
...
```

#### 

