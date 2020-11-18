# Required one-to-one Dependents

 EF Core allows you to model entity types that can only ever appear on navigation properties of other entity types. These are called _owned entity types_. The entity containing an owned entity type is its _owner_.

* In EF Core 3.1, the dependent end of a one-to-one relationship was always considered optional.
* This was most apparent when using owned entities. 

Let's consider the following model.

```sql
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }

    public Address HomeAddress { get; set; }
    public Address BillingAddress { get; set; }
}

public class Address
{
    public string Street { get; set; }
    public string City { get; set; }
    public string State { get; set; }
    public string Country { get; set; }
}
```

Here is the implementation of the context class which contains the configuration.

```sql
public class EntityContext : DbContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder opBuilder)
    {
        opBuilder.UseSqlServer("Data Source=(localdb)\\ProjectsV13;Initial Catalog=PeopleContextDb1;");
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Customer>(b =>
        {
            b.OwnsOne(e => e.HomeAddress);

            b.OwnsOne(e => e.BillingAddress,
                b =>
                {
                    b.Property(e => e.Street).IsRequired();
                    b.Property(e => e.City).IsRequired();
                    b.Property(e => e.State).IsRequired();
                });

        });
    }

    public DbSet<Customer> Customers { get; set; }
}
```

When migrations or `EnsureCreated` are used to create the database. On SQL Server, it will translate to the following SQL.

```sql
CREATE TABLE [dbo].[Customers] (
    [Id]                     INT            IDENTITY (1, 1) NOT NULL,
    [Name]                   NVARCHAR (MAX) NULL,
    [HomeAddress_Street]     NVARCHAR (MAX) NULL,
    [HomeAddress_City]       NVARCHAR (MAX) NULL,
    [HomeAddress_State]      NVARCHAR (MAX) NULL,
    [HomeAddress_Country]    NVARCHAR (MAX) NULL,
    [BillingAddress_Street]  NVARCHAR (MAX) NULL,
    [BillingAddress_City]    NVARCHAR (MAX) NULL,
    [BillingAddress_State]   NVARCHAR (MAX) NULL,
    [BillingAddress_Country] NVARCHAR (MAX) NULL,
    CONSTRAINT [PK_Customers] PRIMARY KEY CLUSTERED ([Id] ASC)
);
```

As you can see all the columns are nullable, even though some of the `BillingAddress` properties have been configured as required. 

* When you query for a `Customer`, and all the columns for any of the addresses are `null`. 
* EF Core will leave all the properties of that address `null`, instead of setting an empty instance of address.

In EF Core 5.0, the `BillingAddress` navigation can now be configured as a required dependent. 

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Customer>(b =>
    {                
        b.OwnsOne(e => e.HomeAddress);

        b.OwnsOne(e => e.BillingAddress,
            b =>
            {
                b.Property(e => e.Street).IsRequired();
                b.Property(e => e.City).IsRequired();
                b.Property(e => e.State).IsRequired();
            });
        b.Navigation(e => e.BillingAddress).IsRequired();
    });
}

```

Now when you create migration or call the `EnsureCreated` method, you will see that it will now include non-nullable columns for the required properties of the required dependent.

```sql
CREATE TABLE [dbo].[Customers] (
    [Id]                     INT            IDENTITY (1, 1) NOT NULL,
    [Name]                   NVARCHAR (MAX) NULL,
    [HomeAddress_Street]     NVARCHAR (MAX) NULL,
    [HomeAddress_City]       NVARCHAR (MAX) NULL,
    [HomeAddress_State]      NVARCHAR (MAX) NULL,
    [HomeAddress_Country]    NVARCHAR (MAX) NULL,
    [BillingAddress_Street]  NVARCHAR (MAX) NOT NULL,
    [BillingAddress_City]    NVARCHAR (MAX) NOT NULL,
    [BillingAddress_State]   NVARCHAR (MAX) NOT NULL,
    [BillingAddress_Country] NVARCHAR (MAX) NULL,
    CONSTRAINT [PK_Customers] PRIMARY KEY CLUSTERED ([Id] ASC)
);
```

 EF Core will now throw an exception if an attempt is made to save a customer with a null`BillingAddress`.

