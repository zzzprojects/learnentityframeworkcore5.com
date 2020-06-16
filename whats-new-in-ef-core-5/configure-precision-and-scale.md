# Configure Precision and Scale

In EF Core 5.0, you can configure the precision and scale using Fluent API. It tells the database provider how much storage is needed for a given column. It only applies to data types where the provider allows the precision and scale to vary - usually just `decimal` and `DateTime`.

* For `decimal` properties, precision defines the maximum number of digits needed to express any value the column will contain, and scale defines the maximum number of decimal places needed.
* For `DateTime` properties, precision defines the maximum number of digits needed to express fractions of seconds, and scale is not used.

In the following example, configuring the `Score` property to have precision 14 and scale 2 will cause a column of type `decimal(14,2)` to be created on SQL Server, and configuring the `LastUpdated` property to have precision 3 will cause a column of type `datetime2(3)`.

```csharp
class MyContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Blog>()
            .Property(b => b.Score)
            .HasPrecision(14, 2);

        modelBuilder.Entity<Blog>()
            .Property(b => b.LastUpdated)
            .HasPrecision(3);
    }
}

public class Blog
{
    public int BlogId { get; set; }
    public decimal Score { get; set; }
    public DateTime LastUpdated { get; set; }
}
```

> Entity Framework does not do any validation of precision or scale before passing data to the provider. It is up to the provider or data store to validate as appropriate. For example, when targeting SQL Server, a column of data type `datetime` does not allow the precision to be set, whereas a `datetime2` one can have precision between 0 and 7 inclusive.

