<a href="https://entityframework-extensions.net/">**Improve EF Core performance with EF Extensions**</a>

<a href="https://entityframework-extensions.net/">
<img src="https://zzzprojects.github.io/images/logo/entityframework-extensions-pub.jpg" width="600" />
</a>

# Lazy Loading in EF Core

Lazy Loading was introduced in Entity Framework Core with EF Core 2.1 to allow better optimizations, performance, and working of the software.

* Lazy Loading is a method of loading and processing only the required data to run the application, the data which is not required at that moment stays untouched.
* It allows the system to perform better and faster and it has become an essential part of the Entity Framework core.

## Procedures to enable Lazy Loading in EF-Core

To enable Lazy Loading in Entity Framework core, there are 2 methods which can be applied.

### With Proxy Package

The First method is by installing the **Proxy Package** provided by Microsoft. All the developer has to do is install `Microsoft.EntityFrameworkCore.Proxies` package which will add all the required proxies needed to run Lazy Loading.

After installing the package, the system will ask the developer to allow the installed proxies to access the databases and enable lazy loading.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<EFCoreContext>(b => b.UseLazyLoadingProxies()
        .UseSqlServer(ConnectionString));
}

```

```csharp
public class Actor
{
    public int ActorId { get; set; }
    public string FullName { get; set; }
    public virtual List<Movie> Movies{ get; set; }
}

public class Movie
{
    public int MovieId { get; set; }
    public string Title { get; set; }
    public virtual Actor Actor{ get; set; }
}
```

### Without Proxy Package

The second method of enabling Lazy Loading in Entity Framework Core is using the `ILazyLoader` interface.

* The `ILazyLoader` interface represents a component that is responsible for loading navigation properties if they haven't already been loaded.
* It can be embedded directly into the principle entity of the database. 
* The `ILazyLoader` can be found in `Microsoft.EntityFrameworkCore.Abstraction` Package.

```csharp
public class Actor
{
    private List<Movie> _movies;

    public Actor()
    {
    }
    
    private Actor(ILazyLoader lazyLoader)
    {
        LazyLoader = lazyLoader;
    }
    
    private ILazyLoader LazyLoader { get; set; }
    public int ActorId { get; set; }
    public string FullName { get; set; }
 
    public List<Movie> Movies
    {
        get => LazyLoader.Load(this, ref _movies);
        set => _movies = value;
    }
}

public class Movie
{
    private Actor _actor;
    
    public Movie()
    {
    }
    
    private Movie(ILazyLoader lazyLoader)
    {
        LazyLoader = lazyLoader;
    }
    
    private ILazyLoader LazyLoader { get; set; }
    public int MovieId { get; set; }
    public string Title { get; set; }
    
    public Actor Actor
    {
        get => LazyLoader.Load(this, ref _actor);
        set => _actor = value;
    }
}
```

## Is Lazy Loading Useful?

Lazy loading is helpful once the association between entities is a one-to-many relationship and you're certain that associated entities aren't going to be utilized immediately.

It helps in functionality reducing application startup time, less memory utilization, and reduce the load on DBMS due to a small amount of query load on the server.

What can be concluded from Lazy Loading is that this feature is useful in some scenarios, otherwise, it could distract the users and developers if they forget to disable the feature.

