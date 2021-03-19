<a href="https://entityframework-extensions.net/">**Improve EF Core performance with EF Extensions**</a>

<a href="https://entityframework-extensions.net/">
<img src="https://zzzprojects.github.io/images/logo/entityframework-extensions-pub.jpg" width="600" />
</a>

# Handling Concurrency in EF-Core

Concurrency means the conflicts happening on the data due to access to multiple users, trying to modify the same data at the same time.

Concurrency Control or Management refers to techniques or methods to maintain the consistency of the data when more than one user is accessing it for different purposes.

Concurrency Management helps in obtaining safety, optimization, consistency and preventing the Data.

## Optimistic Concurrency

By default, Entity Framework core offers Optimistic Concurrency control, in this case, it will consider the data that is saved most recently, and before committing, each transaction verifies that no other transaction has modified the data it has read.

So, If multiple users are working on the same database, the data from the last user will be taken into consideration, this way all users can work simultaneously on the same database and the Last user can save the final data.

## Pessimistic Concurrency

In the Pessimistic concurrency method, the system locks the complete data which is being modified concurrently by multiple users.

Due to this, that data stays unchanged and the modifications can be applied later when there is no concurrency is happening.

However, the Entity Framework Core does not support pessimistic concurrency, as when the internet connection is weak or disconnected the data cannot be managed properly which will affect the working of the database.

### Detecting Conflicts in EF-Core Concurrency

For detecting Concurrency in EF Core, there are two methods available to perform concurrency conflict detection in an optimistic concurrency method.

One is to configure the Entities as concurrency tokens and the other one is adding row version property in the entity classes.

#### Using Concurrency tokens

Consider there are multiple users in a database and all of them are working concurrently, So when the EF Core detects data using the `ConcurrencyCheck`attribute, it performs a comparison of the values of that entity.

If the values of the entity match then the operation is performed successfully, but if these values of the same entity differ from each other than that means there are concurrency conflicts in that entity due to multiple concurrent users.

**Data Annotation**

```csharp
public class Actor
{
    public int ActorId { get; set; }
    [ConcurrencyCheck]
    public string LName { get; set; }
    public string FName { get; set; }
}
```

**Fluent API**

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Actor>()
        .Property(p => p.LName)
        .IsConcurrencyToken();
}
```

####  Using RowVersion Property

In the `RowVersion` Property, a new column is added to the database table and it stores the version stamp of the data. A new Row version value is added each time a user updates the data.

* If two users are working on the same database, and the first user updates the data and leave, then the Second user updates the data and leave.
* Then EF Core will compare both the updated Row version properties and if the values match, the operation is performed successfully.
* Otherwise, if both the Row version values differ, the operation gives a `DbUpdateConcurrencyException`.

**Data Annotation**

```csharp
public class Movie
{
    public int MovieId { get; set; }
    public string Title{ get; set; }
    [Timestamp]
    public byte[] Timestamp { get; set; }
}
```

**Fluent API**

```csharp
class MyContext : DbContext
{
    public DbSet<Movie> Movies { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Movie>()
            .Property(p => p.Timestamp)
            .IsRowVersion();
    }
}

public class Blog
{
    public int MovieId { get; set; }
    public string Title{ get; set; }
    public byte[] Timestamp { get; set; }
}
```

#### Resolving the Data Concurrency Conflicts

To resolve the concurrency conflicts in Entity Framework Core, the system traces 3 main values to determine where the problem is coming from:

* **Current values:** The present values that were last updated into the database by the user.
* **Original Values**: the value that was present in the database initially, before concurrency occurred.
* **Database Value**: The values that are currently stored in the database.

To determine where the problem was occurring from inside the database table, the following code can be applied, and the problem will be shown up-front.  


```csharp
using (var DBcontext = new EFCoreContext())
{
    // Get the actor from database and change its contact number
    var actor = DBcontext.Actors.Single(p => p.ActorID == 1);
    actor.ContactNumber = "222-222-2222";

    // Change name of the actor in the database to simulate a concurrency conflict
    DBcontext.Database.ExecuteSqlRaw(
        "UPDATE dbo.Actor SET FName = 'John' WHERE ActorId = 1");

    var savedData = false;

    while (!savedData)
    {
        try
        {
            // Save the changes to the database
            DBcontext.SaveChanges();
            savedData = true;
        }
        catch (DbUpdateConcurrencyException ex)
        {
            foreach (var item in ex.Entries)
            {
                if (item.Entity is Actor)
                {
                    var currentValues = entry.CurrentValues;
                    var dbValues = entry.GetDatabaseValues();

                    foreach (var prop in currentValues.Properties)
                    {
                        var currentValue = currentValues[prop ];
                        var dbValue = dbValues[prop ];
                    }

                    // Refresh the original values to bypass next concurrency check
                    item.OriginalValues.SetValues(dbValues);
                }
                else
                {
                    throw new NotSupportedException( "Don’t know handling of concurrency
                          conflict " + item.Metadata.Name);
                }
            }
        }
    }
}
```

