# What's New in EF Core 5

EF Core 5.0 is currently in development, and here is the list of all the interesting changes introduced so far in each preview.

## Preview 1

### Simple logging

The simple logging feature adds functionality similar to `Database.Log` in EF6 by providing a simple way to get logs from EF Core without the need to configure any kind of external logging framework.

### Simple way to get generated SQL

EF Core 5.0 introduces the `ToQueryString` extension method, which will return the SQL that EF Core will generate when executing a LINQ query.

### Use a C\# attribute to indicate that an entity has no key

An entity type can now be configured as having no key using the new `KeylessAttribute`. 

```csharp
[Keyless]
public class Address
{
    public string Street { get; set; }
    public string City { get; set; }
    public int Zip { get; set; }
}
```

### Connection or connection string can be changed on initialized DbContext

* It is now easier to create a `DbContext` instance without any connection or connection string. 
* The connection or connection string can now be mutated on the context instance. 
* This feature allows the same context instance to dynamically connect to different databases.

### Change-tracking proxies

* EF Core can now generate runtime proxies that automatically implement `INotifyPropertyChanging` and `INotifyPropertyChanged`. 
* These then report value changes on entity properties directly to EF Core, avoiding the need to scan for changes. 
* However, proxies come with their own set of limitations, so they are not for everyone.

### Enhanced debug views

* Debug views are an easy way to look at the internals of EF Core when debugging issues. A debug view for the Model was implemented some time ago. For EF Core 5.0, we have made the model view easier to read and added a new debug view for tracked entities in the state manager.

### Improved handling of database null semantics

* Relational databases typically treat NULL as an unknown value and therefore not equal to any other NULL. 
* While C\# treats null as a defined value, which compares equal to any other null. 
* EF Core by default translates queries so that they use C\# null semantics. EF Core 5.0 greatly improves the efficiency of these translations.

### Indexer properties

EF Core 5.0 supports the mapping of C\# indexer properties. These properties allow entities to act as property bags where columns are mapped to named properties in the bag.

### Generation of check constraints for enum mappings

EF Core 5.0 migrations can now generate `CHECK` constraints for `enum` property mappings.

```sql
MyEnumColumn VARCHAR(10) NOT NULL CHECK (MyEnumColumn IN ('Useful', 'Useless', 'Unknown'))
```

### IsRelational

A new `IsRelational` method has been added in addition to the existing `IsSqlServer`, `IsSqlite`, and `IsInMemory`. This method can be used to test if the DbContext is using any relational database provider.

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    if (Database.IsRelational())
    {
        // Do relational-specific model configuration.
    }
}
```

### Cosmos optimistic concurrency with ETags

The Azure Cosmos DB database provider now supports optimistic concurrency using `ETags`. Use the model builder in `OnModelCreating` to configure an `ETag`.

```csharp
builder.Entity<Customer>().Property(c => c.ETag).IsEtagConcurrency();
```

The `SaveChanges` will then throw a `DbUpdateConcurrencyException` on a concurrency conflict, which can be handled to implement retries, etc.

### Query translations for more DateTime constructs

Queries containing new `DateTime` construction are now translated.

In addition, the following SQL Server functions are now mapped:

* DateDiffWeek
* DateFromParts

For example:

```csharp
var count = context.Orders.Count(c => date > EF.Functions.DateFromParts(DateTime.Now.Year, 12, 25));

```

### Query translations for more byte array constructs

Queries using `Contains`, `Length`, `SequenceEqual`, etc. on `byte[]` properties are now translated to SQL.

### Query translation for Reverse

Queries using `Reverse` are now translated.

```csharp
context.Employees.OrderBy(e => e.EmployeeID).Reverse()
```

### Query translation for bitwise operators

Queries using bit-wise operators are now translated in more cases.

```csharp
context.Orders.Where(o => ~o.OrderID == negatedId)
```

### Query translation for strings on Cosmos

Queries that use the string methods such as, `Contains`, `StartsWith`, and `EndsWith` are now translated when using the Azure Cosmos DB provider.

## Preview 2

### Use a C\# attribute to specify a property backing field

A C\# attribute can now be used to specify the backing field for a property. This attribute allows EF Core to still write to and read from the backing field as would normally happen, even when the backing field cannot be found automatically.

```csharp
public class Blog
{
    private string _mainTitle;

    public int Id { get; set; }

    [BackingField(nameof(_mainTitle))]
    public string Title
    {
        get => _mainTitle;
        set => _mainTitle = value;
    }
}
```

### Complete discriminator mapping

EF Core uses a discriminator column for TPH mapping of an inheritance hierarchy. 

* Some performance enhancements are possible as long as EF Core knows all possible values for the discriminator. 
* EF Core 5.0 now implements these enhancements.

For example, previous versions of EF Core would always generate this SQL for a query returning all types in a hierarchy.

```sql
SELECT [a].[Id], [a].[Discriminator], [a].[Name]
FROM [Animal] AS [a]
WHERE [a].[Discriminator] IN (N'Animal', N'Cat', N'Dog', N'Human')
```

EF Core 5.0 will now generate the following when a complete discriminator mapping is configured

```sql
SELECT [a].[Id], [a].[Discriminator], [a].[Name]
FROM [Animal] AS [a]
```

It will be the default behavior starting with preview 3.

### Performance improvements in Microsoft.Data.Sqlite

The following two performances improvements are made for SQLite:

* Retrieving binary and string data with `GetBytes`, `GetChars`, and `GetTextReader` is now more efficient by making use of **SqliteBlob** and **streams**.
* The initialization of `SqliteConnection` is now lazy.

These improvements are in the ADO.NET `Microsoft.Data.Sqlite` provider and hence also improve performance outside of EF Core.

## Preview 3

### Filtered Include

The `Include` method now supports filtering of the entities included. 

```csharp
var blogs = context.Blogs
    .Include(e => e.Posts.Where(p => p.Title.Contains("Cheese")))
    .ToList();
```

This query will return blogs together with each associated post, but only when the post title contains "Cheese".

Skip and Take can also be used to reduce the number of included entities.

```csharp
var blogs = context.Blogs
    .Include(e => e.Posts.OrderByDescending(post => post.Title).Take(5)))
    .ToList();
```

This query will return blogs with at most five posts included on each blog.

### New ModelBuilder API for navigation properties

Navigation properties are primarily configured when defining relationships. However, the new `Navigation` method can be used in cases where navigation properties need an additional configuration. For example, to set a backing field for the navigation when the field would not be found by convention.

```csharp
modelBuilder.Entity<Blog>().Navigation(e => e.Posts).HasField("_myposts");
```

Note that the `Navigation` API does not replace relationship configuration. Instead, it allows additional configuration of navigation properties in already discovered or defined relationships.

### New command-line parameters for namespaces and connection strings

Migrations and scaffolding now allow namespaces to be specified on the command line. For example, to reverse engineer a database putting the context and model classes in different namespaces.

```bash
dotnet ef dbcontext scaffold "connection string" Microsoft.EntityFrameworkCore.SqlServer --context-namespace "My.Context" --namespace "My.Model"
```

Also, a connection string can now be passed to the `database-update` command.

```bash
dotnet ef database update --connection "connection string"
```

Equivalent parameters have also been added to the **PowerShell** commands used in the VS Package Manager Console.

### EnableDetailedErrors has returned

For performance reasons, EF doesn't do additional null-checks when reading values from the database. This can result in exceptions that are hard to root-cause when an unexpected null is encountered.

Using `EnableDetailedErrors` will add extra null checking to queries such that, for a small performance overhead, these errors are easier to trace back to a root cause.

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder
        .EnableDetailedErrors()
        .EnableSensitiveDataLogging() // Often also useful with EnableDetailedErrors 
        .UseSqlServer(Your.SqlServerConnectionString);
```

### Cosmos partition keys

The partition key to use for a given query can now be specified in the query.

```csharp
await context.Set<Customer>()
             .WithPartitionKey(myPartitionKey)
             .FirstAsync();
```

### Support for the SQL Server DATALENGTH function

This can be accessed using the new `EF.Functions.DataLength` method. 

```csharp
var count = context.Orders.Count(c => 100 < EF.Functions.DataLength(c.OrderDate));
```

## Preview 4

### Configure database precision/scale in model

Precision and scale for a property can now be specified using the model builder. 

```csharp
modelBuilder
    .Entity<Blog>()
    .Property(b => b.Numeric)
    .HasPrecision(16, 4);
```

Precision and scale can still be set via the full database type, such as "decimal\(16,4\)".

### Specify SQL Server index fill factor

The fill factor can now be specified when creating an index on SQL Server. For example:CSharpCopy

```csharp
modelBuilder
    .Entity<Customer>()
    .HasIndex(e => e.Name)
    .HasFillFactor(90);
```

