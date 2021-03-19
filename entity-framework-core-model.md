<a href="https://entityframework-extensions.net/">**Improve EF Core performance with EF Extensions**</a>

<a href="https://entityframework-extensions.net/">
<img src="https://zzzprojects.github.io/images/logo/entityframework-extensions-pub.jpg" width="600" />
</a>

# Entity Framework Core Model

In Entity Framework, the model is prepared according to the requirement of the user. It depends upon the number of classes and categories that will be embedded into the database.

* To perform various CRUD operations on the applications, model can also be configured manually, and it can be further modified to suit the required database.
* Creating a model in Entity Framework Core has become easy for developers.
* To design a model, it can either be done by coding it manually or by using the previous database model and tweaking it as per the requirement.
* But with latest updates in the Entity Framework Core, there is a slight change in the approach for generating a model from an existing database.
* Earlier the developers used the Database-First approach but now in the Entity Framework Core, the Code-First approach is used to generate Model using the existing database.

## Code-First Approach to Generate Model using Existing Database

### Command Line Interface

For creating a model using the existing database, the developer can use the **Command Line Interface \(CLI\)** tools, these tools help in generating SQL statements for model based on the existing database.

Using the CLI method, the developers can create as well as apply migrations and generate code for the model based on the existing model database. Another advantage of using the CLI approach is that the commands generated using this can be used for .NET core projects as well.

Before starting, don’t forget to create a folder.

```bash
mkdir EFCoreExample
```

Once the folder is created, navigate to this folder

```bash
cd EFCoreExample
```

Now, create another project

```bash
dotnet new console
```

After doing all this, add these Entity framework Core Tool and Packages, if you don’t have them.

```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Design
```

In the above two packages, the first one is the EF-Core provider for SQL Server,

The second package have the EF-Core commands, without these packages it is not possible to execute the SQL Server statements,

You use the Db Scaffold command to generate the model. The command has two required arguments - a connection string and a provider.

```bash
dotnet ef dbcontext scaffold "Server=.\;Database=DemoEFCore;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -o Model
```

The `DbContext` class will take the name of the database plus context, You can override this using the -c or --context option e.g.

```bash
dotnet ef dbcontext scaffold "Server=.\;Database=DemoEFCore;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -o Model -c "EFCoreContext"
```

Model Configuration

```bash
dotnet ef dbcontext scaffold "Server=.\;Database=DemoEFCore;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -d
```

Updating the model

```bash
dotnet ef dbcontext scaffold "Server=.\;Database=DemoEFCore;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -force
```

###  Visual Studio

Another great approach for generating the model using the existing database is through Visual Studio.

* In Visual Studio the developer can use the Package Manager Console \(PMC\) to create the required model using the existing database.
* Using PMC the user can create migrations, apply migrations and generate the relevant code for the model based on the existing model.
* To keep the new changes in the database in sync with the generated model in Entity Framework Core, we use migrations.
* With Migrations, changes will get updated within the model and the application will get in sync with the model resulting in better performance.

```bash
PM> Scaffold-DbContext "Server=.\;Database=DemoEFCore;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Model -Context "EFCoreContext" -DataAnnotations
```

## Shadow Properties for Generating Model in EF Core

Shadow properties are introduced with the release of the EF Core, they are not present in .NET entity class but rather situated or defined in the entity type of the Entity Framework Core model.

* Shadow properties are dependent on the **Change Tracker** as their value and state can be completely maintained using Change Tracker.
* So, whenever the user wants to change or need the values of the shadow properties, they will use the Change Tracker API.
* To configure the shadow properties the developers can use the **Fluent API** which will enable them to tweak their values and state.
* Most places where shadow properties are preferred are for the use of **foreign key properties**, to represent the relationship between two entities in the Database.
* To represent shadow property into the database in a relationship between two entities when no foreign key is found, EF Core will use the **Convention** method.

Below example shows the `LastUpdated` shadow property which was configured on the Contact entity:

```csharp
public class SampleContext : DbContext
{
    public DbSet<Account> Accounts{ get; set; }
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Account>()
            .Property<DateTime>("LastUpdated");
    }
}

public class Account
{
    public int AccountId { get; set; }
    public string FName { get; set; }
    public string LName { get; set; }
    public string EmailID { get; set; }
}
```



Now, the below code shows a `Version` of shadow property which is added to the actor entity in the `OnModelCreating` method, and then it is configured to be a part in concurrency management:

```csharp
public class SampleContext : DbContext
{
    public DbSet<Actor> Actors { get; set; }
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Actor>()
            .Property<byte[]>("Version")
            .IsRowVersion();
    }
}

public class Actor
{
    public int ActorId { get; set; }
    public string FName { get; set; }
    public string LName { get; set; }
    public ICollection<Movie> Movies{ get; set; }
}
```

#### Setting the value of shadow properties:

To access shadow property, you can use the `DbContext.Entry` property and then set the value through the `CurrentValue` property:

```csharp
var context = new SampleContext();
var account = new Account{ FName = "John", LName = "Cena" };
context.Add(account);
context.Entry(account).Property("LastUpdated").CurrentValue = DateTime.UtcNow;
context.SaveChanges();
```

Another way to set the values is the ChangeTracker API by its `Entries()` method. This method will provide more logical way to a `LastUpdated` value by overriding the `SaveChanges` method:

```csharp
public class SampleContext: DbContext
{
    protected override void OnModelBuilding(ModelBuider modelBuilder)
    {
        modelBuilder.Entity<Account>()
            .Property<DateTime>("LastUpdated");
    }
    
    public override int SaveChanges()
    {
        ChangeTracker.DetectChanges();
        foreach (var enty in ChangeTracker.Entries())
        {
            if(enty.State == EntityState.Added 
                || enty.State == EntityState.Modified)
            {
                enty.Property("LastUpdated").CurrentValue = DateTime.UtcNow;
            }
        }
        
        return base.SaveChanges();
    }
    
    public DbSet<Account> Accounts{ get; set; }
}

public class Account
{
    public int AccountId { get; set; }
    public string FName { get; set; }
    public string LName { get; set; }
    public string EmailID { get; set; }
}
```

#### Querying with shadow properties:

The shadow properties can be linked to LINQ queries by the static Property method in the EF utility class:

```csharp
var accounts = context.Accounts
    .OrderBy(account=> EF.Property<DateTime>(account, "LastUpdated"));
```

You can use the C\# 6 using static directive:

```csharp
using static Microsoft.EntityFrameworkCore.EF;
using static System.Console;
...
var accounts = context.Accounts
    .OrderBy(account=> Property<DateTime>(account, "LastUpdated"));
```

