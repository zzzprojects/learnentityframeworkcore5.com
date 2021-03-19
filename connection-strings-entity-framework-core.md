<a href="https://entityframework-extensions.net/">**Improve EF Core performance with EF Extensions**</a>

<a href="https://entityframework-extensions.net/">
<img src="https://zzzprojects.github.io/images/logo/entityframework-extensions-pub.jpg" width="600" />
</a>

# Connection Strings: Entity Framework Core

In Entity Framework Core, there could be multiple numbers of databases that needed to be connected or if any database provider needs to connect with the database.

* The connection string is used to establish a connection between the database and database providers.
* The connection string could have sensitive data from the database, which is required to be protected and can be done by using the Secret Manager tool.
* The connection string is needed to be configured based on the environment, such as testing, production, and Development.

## Configuring Connection Strings in EF Core

After creating the connection string between database and database providers, we need to make it available to the **`DbContext`**for processing the data for the application.

There are a few numbers of methods for configuring the connection strings for `DbContext`.

### OnConfiguring Method

On `DbContext`their data needs to be updated regularly for providing better results, to maintain this procedure the data needs to be overridden onto `DbContext`.

* To override the data every time on `DbConetxt`using the connection string, the `OnConfiguring`method is used to achieve the overridden of the data.
* The only downside of the `OnConfiguring`method is that if it is used on the connection string, it will override all other configurations for that database.

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder opBuilder)
{
    opBuilder.UseSqlServer("server=.;database=myEFCoreDB;trusted_connection=true;");
}
```

### Configuring Connection String as Service

In .NET Core Applications using Entity Framework Core, the connection string can be configured using the `AddDbContext`extension method which can be used in the `Startup` class using the `IServiceCollection`.

In Entity Framework Core, connection string can also be configured to `DbContext` using **ASP.NET Core MVC** applications and **.NET Core Console** application.

```csharp
public void ConfigureServices(IServiceCollection servicescol)
{
    servicescol.AddDbContext<MyEFCoreDbContext>(options => 
    {
        options.UseSqlServer("server=.;database=myEFCoreDb;trusted_connection=true;"));
    });
}
```

#### ASP.NET Core MVC Application

Earlier in ASP.NET, the connection string was stored on the web.config file, but now ASP.NET core can extract and read connection strings from different locations such as `appsettings.json`, command-line arguments, and the environment variable, etc. 

ASP.NET Core uses the **Model-View-Architecture** \(MVC\) Pattern, this model separates the application into three main groups, model, view and controller.

These groups work together to provide the required results from the model.

In any of the MVC Applications using the Entity Framework Core, the `DbContext` is injected using dependency injection in the `ConfigureServices` method.

Configure Services method comes in startup class, which means that **connection strings are also required in the Startup Class**.

To read from the Startup Class, an `IConfiguration` object is required, which can be injected from Dependency injection.

To inject into a `Startup` class, the developers can use a constructor or a **`GetConnectionString`** method.

```csharp
public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public IConfiguration Configuration { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        services.AddDbContext<EFCoreDBContext>(options =>
            options.UseSqlServer(
                Configuration.GetConnectionString("EFCoreDBContext")));

        services.AddIdentity<ApplicationUser, IdentityRole>()
            .AddEntityFrameworkStores<EFCoreDBContext>()
            .AddDefaultTokenProviders();

        services.AddRazorPages();
     }
}
```

After this, the connection string is needed to be passed onto the `DbContext`.

For this to happen a separate context class ****needs to be created derived from the `DbContext` itself.

```csharp
public partial class EFCoreDBContext : DbContext
{
    public EFCoreDBContext ()
    {
    }

    public EFCoreDBContext(DbContextOptions<EFCoreDBContext> options) : base(options)
    {
    }

    public virtual DbSet<Actors> Actors{ get; set; }
    public virtual DbSet<Movies> Movies{ get; set; }
    public virtual DbSet<Biographies> Biographies{ get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        if (!optionsBuilder.IsConfigured)
        {
            //warning You can move this code to protect potentially senstive information
            //in connection string.

            optionsBuilder.UseSqlServer("Data Source= .;Initial Catalog=EFCoreDB;User ID=test;Password=test123");
        }
    }
}
```

The startup service is availed again for using the `ConfigureService` method to register the DbContext for dependency injection.

The Entity Framework Core provides the `AddDbContext` extension method, which can be used to register our context class.

The `DbContext` options are configured using the `DbContextOptionBuilder`, the SQL server is configured as the database provider through `UseSQLServer` method.

#### Console App\(.Net Core\)

In .NET Core application, dependency injection is set up manually unlike the ASP.NET core, which use to do this automatically.

* The `appsettings.json` file is not created in .NET Core, hence it has to be created ourselves.
* To read connection string in the .NET Core console application, the developer has to initialize `IConfigurations`.

To do that, the following packages are required to be installed If you miss these packages install them now.

* Microsoft.Extensions.Configuration
* Microsoft.Extensions.Configuration.FileExtensions
* Microsoft.Extensions.Configuration.Json

After successful installation, connection strings can be read using the relevant codes.

```csharp
class Program
{
    static void Main(string[] args)
    {
        var newbuilder = new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("appsettings.json");

        IConfiguration iconfig = new ConfigurationBuilder()
            .AddJsonFile("appsettings.json", true, true)
            .Build();

        Console.WriteLine($" Hello { iconfig["fullname"] } !");
    }
}
```

To further pass the connection string `DbContext` create a context class. For the reference of context class see the above example in **ASP.NET Core MVC** application.

This is how connection string is provided to `DbContext` using **ASP.NET Core** **and Console app \(.NET Core\)**.

