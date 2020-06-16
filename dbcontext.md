# DbContext

The `DbContext` is simply the way for the developers to incorporate Entity Framework based data to the application.

* It allows you to make database connections inside an application model and allows the developer to link the model properties to the database table using a connection string.
* It is the base class to manage all types of database operations, such as establishing a connection with the database, query the database and end the connection.
* In the older version of Entity Framework, there was an `ObjectContext` class, but now in the latest version, we have `DbContext` class.

The `DbContext` in Entity Framework Core consist of the following features and responsibilities:

* Database Management
* Database Connections
* Entity Set
* Querying
* Change Tracking
* Persisting Data
* Caching
* Model Binding
* Materialization
* Configuration
* Validation

All these features are present to perform a dedicated task on the Entity Framework Core.

### Database Management

In Entity Framework Core, the `DbContext` allows the user to manage the complete database. It allows the user to create, delete or check for the existing database connections inside the given project.

### Database Connections

The `DbContext` also allows the user to check, establish and close the connections between the databases as per the requirement of the project.

### Entity Set

The `DbContext` represents all the entities in a project, it also manages these entities throughout their lifetime. It also applies CRUD operations on all the entity types, such as `Add`, `Attach` or `Remove`.

### Querying

The `DbContext` also converts the LINQ queries to the SQL queries using the querying method.

### Change Tracking

In Entity Framework, the `DbContext` includes the Change Tracker API which tracks all the changes when entities are added, updated or deleted.

The state of the entity can be changed manually or automatically depending on the user.

### Persisting Data

Since `DbContext` performs all the CRUD related operations, It persists all the changes made to the database.

### Caching

The `DbContext` keeps all the changes made to the entities throughout their lifetime in the form of first-level cache files.

### Model Binding

The `DbContext` also performs model binding operations in which it automatically read classes and code-based configuration to build an in-memory model, metadata and database.

### Materialization

In materialization, the `DbContext` converts the queries from the database tables into entities.

### Configuration

The `DbContext` configures the behavior of the context of the databases.

### Validations

The `DbContext` checks the validity of the data and performs automatic validation of the data.

**DbContext Query**

Using `DbContext` in Entity Framework Core, there are three types of query operations which can be performed, they are:

* Adding a new entity.
* Changing or modifying the properties of the current entity.
* Deleting or removing the existing Entity.

## Adding new Entity

To add a new entity using `DbContext`, the developer must prepare the relevant code related to the database.

This code is very straightforward and new entities can be added by using these four key methods:

### Add&lt;TEntity&gt;

```csharp
// with type parameter
var Actor = new Actor{ FirstName = "Dwayne", LastName = "Johnson" };
context.Add<Actor>(actor);
context.SaveChanges();

// without type parameter
var Actor = new Actor{ FirstName = "Dwayne", LastName = "Johnson" };
context.Add(actor);
context.SaveChanges();
```

### Add\(object entity\)

```csharp
object actor = new Actor{ FirstName = "Dwayne", LastName = "Johnson" };
context.Add(actor);
context.SaveChanges();
```

### AddRange\(IEnumerable&lt;object&gt; entities\)

```csharp
var context = new SampleContext();
var actor = new Actor()
{
    FirstName = "Dwayne",
    LastName = "Johnson",
    Movies = new List<Movie>()
    {
        new Movie { Title = "Baywatch"},
        new Movie { Title = "Rampage" },
        new Movie { Title = "SkyScraper" }
    }
};

context.Add(actor);
context.SaveChanges();
```

### AddRange\(params object \[\] entities\)

```csharp
var actor = new Actor{ FirstName = "Dwayne", LastName = "Johnson" };
var baywatch = new Movie { Title = "Baywatch", Actor= actor};
var rampage = new Movie { Title = "Rampage", Actor= actor};
var skyscraper = new Movie { Title = "SkyScraper", Actor= actor};
context.Add(actor);
context.SaveChanges();
```

To add multiple records the developer must use the AddRange entity method.

#### Sample 1:

```csharp
var context = new SampleContext();
var actor = new Actor()
{
    FirstName = "Dwayne",
    LastName = "Johnson",
    Movies = new List<Movie>()
    {
        new Movie { Title = "Baywatch"},
        new Movie { Title = "Rampage" },
        new Movie { Title = "SkyScraper" }
    }
};

context.AddRange(Movies);
context.SaveChanges();
```

#### Sample 2:

```csharp
var context = new SampleContext();
var actor = new Actor{ FirstName = "Dwayne", LastName = "Johnson" };
var movie = new Movie { Title = "Baywatch", Actor= actor};
context.AddRange(actor, movie);
context.SaveChanges();
```

The `SaveChanges` method insert all entities to the database.

## Changing/Modifying Entity

To change or modify an entity, the DbContext should track the current modifications of the entities, so whenever there will be a change in the entity, the DbContext will register it as Modified, and the change tracker API will record the current modifications and will also keep the previous changes.

There are various method for Modifying Entities, which are:

### Disconnected Scenario.

```csharp
var actor = context.Actors.First(a => a.ActorId == 1);
actor.FirstName = "Dwayne";
context.SaveChanges();
```

### Setting Entitystate.

```csharp
public void Save(Actor actor)
{
    context.Entry(actor).State = EntityState.Modified;
    context.SaveChanges();
}
```

### DbContext Update.

```csharp
public void Save(Actor actor)
{
    context.Update(actor);
    context.SaveChanges();
}
```

### Attach.

```csharp
var context = new TestContext();
var actor = new Actor()
{
    ActorId = 1,
    FirstName = "Dwayne",
    LastName = "Johnson"
};

actor.Movies.Add(new Movie {MovieId = 1, Title = "Baywatch" });
context.Attach(actor);
context.Entry(actor).Property("FirstName").IsModified = true;
context.SaveChanges();
```

### TrackGraph.

#### Sample 1:

```csharp
var actor = new Actor() 
{
    ActorId = 1,
    FirstName = "Dwayne",
    LastName = "Johnson"
};

actor.Movies.Add(new Movie { ActorId = 1, MovieId = 1, Title = "Baywatch", Isbn = "0123" });
actor.Movies.Add(new Movie { ActorId = 1, MovieId = 2, Title = "Rampage", Isbn = "0123" });
actor.Movies.Add(new Movie { ActorId = 1, MovieId = 3, Title = "SkyScraper", Isbn = "0123" });

var context = new TestContext();
context.ChangeTracker.TrackGraph(actor, e => 
{
    if((e.Entry.Entity as Actor) != null)
    {
        e.Entry.State = EntityState.Unchanged;
    }
    else
    {
        e.Entry.State = EntityState.Modified;
    }
});

context.SaveChanges();
```

#### Sample 2:

```csharp
var actor = new Actor() 
{
    ActorId = 1,
    FirstName = "Dwayne",
    LastName = "Johnson"
};

actor.Movies.Add(new Movie { MovieId = 1, Title = "Baywatch", Isbn = "0123" });
actor.Movies.Add(new Movie { MovieId = 2, Title = "Rampage", Isbn = "0123" });
actor.Movies.Add(new Movie { MovieId = 3, Title = "SkyScraper", Isbn = "0123" });

var context = new TestContext();

context.ChangeTracker.TrackGraph(actor, e => 
{
    e.Entry.State = EntityState.Unchanged; //starts tracking
    if((e.Entry.Entity as Movie) != null)
    {
        context.Entry(e.Entry.Entity as Movie).Property("Isbn").IsModified = true;
    }
});
```

## Deleting/Removing Entity

To delete an existing entity, the current entity should be continuously tracked by the `DbContext` in order to check its state and change it to Deleted.

To apply this approach, we use `DbContext.Remove` method, after applying this code, the `DbContext` usually executes two SQL Statements.

The first one to retrieve the entity from the database and the other one to delete it permanently from the Database.

To remove entities there are usually two properties:

### Setting Entity State.

```csharp
var context = new SampleContext();
var actor = new Actor { ActorId = 1 };
context.Entry(actor).State = EntityState.Deleted;
context.SaveChanges();
```

### Related Data.

```csharp
var context = new SampleContext();
var actor = context.Actors.Single(a => a.ActorId == 1);
var movies = context.Movies.Where(b => EF.Property<int>(b, "ActorId") == 1);

foreach (var movie in movies)
{
    actor.Movies.Remove(movie);
}

context.Remove(actor);
context.SaveChanges();
```

