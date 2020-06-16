# DbSet

In Entity Framework Core, the `DbSet` represents the set of entities. In a database, a group of similar entities is called an Entity Set.

The `DbSet` enables the user to perform various operations like add, remove, update, etc. on the entity set.

Each entity type shows some `DbSet` properties to participate in CRUD operations.

In the working model, the `DbContext` represents the `DbSet` property of all the entities, and it keeps the collection of entities in memory.

## Operations of DbSet

The `DbSet` is responsible for performing all the basic CRUD \(Create, Read, Update and Delete\) operations on each of the Entity.

The `DbSet` operations are used to change any property of the entity in the EF Core. The most essential methods of the `DbSet` are:

* Querying Data
* Adding Data
* Modifying Data
* Deleting Data

### Querying of Data

In Entity Framework, querying a data is performed using `DbSet` and the queries are specified using LINQ.

* To write LINQ queries, the .NET developers can use query syntax or method syntax.
* The query syntax is similar to the SQL and EF Core provider is responsible to check if the query is converted to SQL or not so that the database can perform the execution of the code.
* The method syntax is a chained method in which some of the queries are like the SQL query, but most of them are not.

In Entity Framework Core, there are various methods for querying the different type of data such as:

* Retrieving Single Object
* Retrieving Multiple objects
* Filtering and Ordering
* Grouping
* Returning Non-Entity types
* Include Related Data
* NoTracking Queries

#### Retrieving Single object

To retrieve a single entity from a query, you can use `First`, `FirstOrdefault`, `Single`, `SingleOrDefault` and `Find` methods.

* In the `First` and `FirstOrDefault` criteria, there is only one entity returned from a large group of entities.
* Whereas in `Single` and `SingleOrDefault` one record entity is returned but that entity should be relevant to the required field otherwise it will be discarded.
* The `Find` method is familiar to the users of the older version of Entity Framework that use to support the `DbSet` API.
* The `Find` method requires a Key-value parameter in the entity to determine the required entity otherwise it gives null as the result.

```csharp
var actor = context.Actors.First();
var actor = context.Actors.Where(a => a.ActorId == 1).Single();
var actor = context.Actors.Single(a => a.ActorId == 1);
var actor = context.Actors.Find(1);
```

#### Retrieving Multiple Objects

The queries that are retrieving the data which is compatible with multiple entities are executed otherwise those queries are ignored.

Data is iterated when the required query is executed in `foreach` loop, `ToList`, `Sum` or `count`. The query is not executed until the `foreach` loop is utilized.

**Sample 1:**

```csharp
var movies = context.Movies; // define query
foreach(var movie in movies) // query executed and data obtained from database
{
    ...
}
```

**Sample 2:**

```csharp
var movies = context.Movies.ToList(); // define query and force execution
```

#### Filtering and Ordering

Filtering is the process of picking up the correct entity using the where method, which has become the principle of the filtering process.

**Sample 1:**

```csharp
var movies = context.Movies.Where(p => p.CategoryId == 1); // where method
```

**Sample 2:**

```csharp
// lambda as expression that returns boolean
var movies = context.Movies.Where(p => p.CategoryId == 1 && p.Ratings< 5);
```

**Sample 3:**

```csharp
var movies = context.Movies.OrderBy(p => p.MovieName);
var categories = context.Categories.OrderBy(c => c.CategoryName)
    .ThenOrderBy(c => c.CategoryId);
```

#### Grouping

It uses the `GroupBy` method which is used to group all the entity results according to their categories.

```csharp
var groups = context.Movies.GroupBy(p => p.CategoryId);

var groups = context.Movies
    .GroupBy(p => new {Director = p.DirectorId, Country = p.CountryId});
```

#### Returning Non-Entity types

When the developer wants to keep the main data but needs to return the sub-data then Non-Entity types `DbSet` can be used.

The returned data can be stated as a non-entity type or the anonymous type, to perform this operation, query types can also be used as an alternate method.

```csharp
public class MovieHeader
{
    public int MovieId { get; set; }
    public string MovieName { get; set; }
}
```

```csharp
List<MovieHeader> headers = context.Movies.Select(p => new MovieHeader()
{
    MovieId = p.MovieId,
    MovieName = p.MovieName
}).ToList();
```

#### Include Related Data

Just as the name suggests the `Include` property is used to include the related entity type into the database.

**Sample 1:**

```csharp
var actors = context.Actors.Include(a => a.Movies).ToList();
```

**Sample 2:**

```csharp
var actors = context.Actors
    .Include(a => a.Producer)
    .Include(a => a.Movies)
    .ToList();
```

**Sample 3:**

```csharp
var actors = context.Actors
    .Include(a => a.Movies)
    .ThenInclude(b => b.Producer)
    .ToList();
```

####  No-Tracking Queries

To improve the performance of the application the no-tracking queries are used in the form `AsNoTracking` method.

This will inform the `DbContext` that the entity is read-only and does not need to be tracked by the context, this will eventually decrease the load on the system, hence improving the performance.

```csharp
var movies = context.Movies.AsNoTracking().ToList();
using (var context = new SampleContext())
{
    context.ChangeTracker.QueryTrackingBehavior = QueryTrackingBehavior.NoTracking;
    var movies = context.Movies.ToList();
    var actors = context.Actors.ToList();
    …
}
```

### Add Data

To add a new entity to the database, the `Add` method is used and then after tracking the entity by `DbContext` the state of the entity is changed to Added.

To add multiple records into the database, the `AddRange` method is used, it is like the `Add` method but in  `AddRange` method, all the records are saved in just a single step instead of adding them all individually.

To add an entity using the `DbSet` there are 3 key methods:

#### Add \(Entity\)

```csharp
// with type parameter
var actor = new Actor{ FirstName = "John", LastName = "Dwayne" };
context.Actors.Add<Actor>(actor);

// without type parameter
var actor = new Actor{ FirstName = "John", LastName = "Dwayne" };
context.Actors.Add(actor);
```

#### AddRange &lt;Entity&gt;

```csharp
var context = new SampleContext();

var actor = new Actor() 
{
    FirstName = "John",
    LastName = "Dwayne" };
    var movies = new List<Movie>()
    {
        new Movie { Title = "Baywatch", Actor = actor},
        new Movie { Title = "Rampage", Actor = actor },
        new Movie { Title = "Skyscraper", Actor = actor }
    }
};

context.Movies.AddRange(movies);
Context.SaveChanges();
```

#### AddRange \[Entity\]

```csharp
var actor = new Actor { FirstName = "John", LastName = "Dwayne" };
var baywatch = new Movie { Title = "Baywatch", Actor = actor };
var rampage = new Movie { Title = "Rampage", Actor = actor };
var skyscraper = new Movie { Title = "Skyscraper", Actor = actor };

context.Movies.AddRange(actor, baywatch, rampage, skyscraper);
context.SaveChanges();
```

### Modify Data

To modify the data in `DbSet`, first it checks whether the data which is to be modified is being tracked or not.

After the Change tracker detects a change in the state of the entity, it issues a SQL statement that updates the properties that were changed.

```csharp
var actor = context.Actors.Find(1);
actor.FirstName = "John R";
context.SaveChanges();
```

#### Disconnected Scenario in Modifying Data

If the application such as ASP.NET is disconnected from the `DbContext`, then the modifications must be updated using another method.

To update the state of the entity in disconnected form, the `DbSet<T>.Update` method is used, so whenever the application is loaded, it changes the state of the entity as Updated.

This method is newly introduced in Entity Framework Core.

#### DbSet Update

The `Update` method is using `DbSet<T>` class and provides different methods to work with an individual or multiple entities.

```csharp
public void Save(Actor actor)
{
    context.Actors.Update(actor);
    context.SaveChanges();
}
```

### Delete Data

Deleting the data is similar to modifying the data using `DbSet`. For deleting any data, it will first check whether the data which is to be deleted is being tracked by the context or not.

To determine the state of the entity `DbSet` uses `DbSet<T>.Remove` method which sets the state of the entity as `Deleted`, after this when the changes are saved, the `DELETE` statement is generated and executed by the Database.

**Sample 1:**

```csharp
context.Actors.Remove(context.Actors.Find(1));
context.SaveChanges();
```

**Sample 2:**

```csharp
var context = new SampleContext();
var actor = new Actor { ActorId = 1 };
context.Actors.Remove(actor);
context.SaveChanges();
```

#### Related Data in Deleting the in DbSet

If the entity that user wants to delete has a related data then the approach that you take will depend on how the relationship has been configured.

If the relation is a fully defined relationship, then the resulting output will be either deleted or set to null.

In other cases, EF Core has also introduced a Shadow property to represent the foreign key, this method is a little longer as it requires four action calls from the database.

```csharp
var context = new SampleContext();
var actor = context.Actors.Find(1);

var movies = context.Movies.Where(b => EF.Property<int>(b, "ActorId") == 1);
foreach (var movie in movies)
{
    actor.Movies.Remove(movie);
}

context.Actors.Remove(actor);
context.SaveChanges();
```

