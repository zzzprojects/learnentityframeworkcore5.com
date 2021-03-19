<a href="https://entityframework-extensions.net/">**Improve EF Core performance with EF Extensions**</a>

<a href="https://entityframework-extensions.net/">
<img src="https://zzzprojects.github.io/images/logo/entityframework-extensions-pub.jpg" width="600" />
</a>

# MVC

### What is MVC?

The Model-View-Controller \(**MVC**\) is an architectural pattern that separates an **application** into three main logical components: the model, the view, and the controller. Each of these components is built to handle specific development aspects of an **application**.

MVC stands for Model, View, and Controller. MVC separates the application into three components

* **Model:** Responsible for maintaining application data and business logic.
* **View:** The user interface of the application, which displays the data.
* **Controller:** Handles users' requests and renders appropriate view with model data.

### Create MVC App

To start, we will create an ASP.NET Core web application project. The project type comes with all template files to create a web application before adding anything. Let's open Visual Studio 2019,  if you haven't already installed Visual Studio, go to the [Visual Studio downloads](https://visualstudio.microsoft.com/downloads) page to install it for free.

![](../.gitbook/assets/image%20%2814%29.png)

On the start window, choose **Create a new project**.

![](../.gitbook/assets/image%20%2819%29.png)

On the **Create a new project** window, enter or type _asp.net_ in the search box. Next, choose **C\#** from the Language list, and then choose **Windows** from the Platform list. Select the  **ASP.NET Core Web Application** template, and then choose **Next**.

![](../.gitbook/assets/image%20%2823%29.png)

In the **Configure your new project** window, type or enter _**EFCore5InMvcApp**_ in the **Project name** box and click on the **Create** button. 

![](../.gitbook/assets/image%20%2821%29.png)

In the **Create a new ASP.NET Core web application** window, verify that **ASP.NET Core 5.0** appears in the top drop-down menu. Then, choose **ASP.NET Core Web App \(Model-View-Controller\)**\, including example ASP.NET Core MVC Views and Controllers, and then click on the **Create** button.

![](../.gitbook/assets/image%20%2820%29.png)

Visual Studio opens your new project and includes the default code files in your project as shown in the **Solution Explorer**.

### Install Entity Framework Core

To use Entity Framework Core we need to install [Microsoft.EntityFrameworkCore](https://www.nuget.org/packages/Z.EntityFramework.Extensions.EFCore/) library. [I](https://www.nuget.org/packages/Z.EntityFramework.Extensions.EFCore/)t is available as a nuget package and you can install it using **Nuget Package Manager**.

In the **Package Manager Console** window, enter the following command.

```bash
PM> Install-Package Microsoft.EntityFrameworkCore
```

For SQL Server LocalDB, which is installed with Visual Studio, we need to install [Microsoft.EntityFrameworkCore.SqlServer](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer) and will get all the packages required for EF Core.

```bash
PM> Install-Package Microsoft.EntityFrameworkCore.SqlServer
```

We also need to install the following NuGet package.

```csharp
PM> Install-Package Microsoft.VisualStudio.Web.CodeGeneration.Design
```

### Create a Data Model and Database Context

In **Solution Explorer**, right-click on the _**Models**_ folder and choose **Add &gt; Class**. Enter a class file name **Author.cs** and add the following code.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace EFCore5InMvcApp.Models
{
    public class Author
    {
        public int AuthorId { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public DateTime BirthDate { get; set; }
        public virtual ICollection<Book> Books { get; set; }
    }
}
```

Now let's add another entity class `Book` and replace the following code.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace EFCore5InMvcApp.Models
{
    public class Book
    {
        public int Id { get; set; }
        public string Title { get; set; }
        public int AuthorId { get; set; }
        public Author Author { get; set; }
    }
}

```

So let's create a folder in your project by right-clicking on your project in Solution Explorer and click **Add &gt; New Folder**. Name the folder **DAL** \(Data Access Layer\). In that folder, create a new class file named **BookStore.cs**, and replace the following code.

```csharp
using EFCore5InMvcApp.Models;
using Microsoft.EntityFrameworkCore;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace EFCore5InMvcApp.DAL
{
    public class BookStore : DbContext
    {
        public BookStore(DbContextOptions<BookStore> options) : base(options)
        {
        }
        public DbSet<Author> Authors { get; set; }
        public DbSet<Book> Books { get; set; }
    }
}

```

### Register Context Class

To register `BookStore` as a service, open `Startup.cs`, and call the `AddDnContext` in the `ConfigureServices` method.

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<BookStore>(options => options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
    services.AddControllersWithViews();
}
```

The name of the connection string is passed into the context by calling a method on a `DbContextOptionsBuilder` object.

### Setup Connection String

For local development, the ASP.NET Core configuration system reads the connection string from the _**appsettings.json**_ file. So let's add the connection to that file as shown below.

```javascript
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=(localdb)\\ProjectsV13;Initial Catalog=BookStoreDb;Trusted_Connection=True;MultipleActiveResultSets=true"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*"
}

```

The above connection string specifies that the Entity Framework will use a `LocalDB` database named `BookStoreDb`.

### Initialize Database

The Entity Framework will create an empty database for you. So we need to write a method that's called after the database is created to populate it with test data.

In the DAL folder, add a new class `BookStoreInitializer` and replace the following code.

```csharp
using EFCore5InMvcApp.Models;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace EFCore5InMvcApp.DAL
{
    public class BookStoreInitializer
    {
        public static void Initialize(BookStore context)
        {

            context.Database.EnsureCreated();

            // Look for any authors.
            if (context.Authors.Any())
            {
                return;   // DB has been seeded
            }

            var authors = new List<Author>
            {
                new Author { FirstName="Carson", LastName="Alexander", BirthDate = DateTime.Parse("1985-09-01")},
                new Author { FirstName="Meredith", LastName="Alonso", BirthDate = DateTime.Parse("1970-09-01")},
                new Author { FirstName="Arturo", LastName="Anand", BirthDate = DateTime.Parse("1963-09-01")},
                new Author { FirstName="Gytis", LastName="Barzdukas", BirthDate = DateTime.Parse("1988-09-01")},
                new Author { FirstName="Yan", LastName="Li", BirthDate = DateTime.Parse("2000-09-01")},
            };

            authors.ForEach(a => context.Authors.Add(a));
            context.SaveChanges();

            var books = new List<Book>
            {
                new Book { Title = "Introduction to Machine Learning", AuthorId = 1 },
                new Book { Title = "Advanced Topics in Machine Learning", AuthorId = 1 },
                new Book { Title = "Introduction to Computing", AuthorId = 1 },
                new Book { Title = "Introduction to Microeconomics", AuthorId = 2 },
                new Book { Title = "Calculus I", AuthorId = 3 },
                new Book { Title = "Calculus II", AuthorId = 3 },
                new Book { Title = "Trigonometry Basics", AuthorId = 4 },
                new Book { Title = "Special Topics in Trigonometry", AuthorId = 4 },
                new Book { Title = "Advanced Topics in Mathematics", AuthorId = 4 },
                new Book { Title = "Introduction to AI", AuthorId = 4 },
            };

            books.ForEach(b => context.Books.Add(b));
            context.SaveChanges();
        }
    }
}

```

* The above code creates a database when needed and loads test data into the new database.
* It also checks if there are any authors in the database, and if not, it assumes the database is new and needs to be seeded with test data.

In `Program.cs` file, replace the following code in the `Main` method.

```csharp
using EFCore5InMvcApp.DAL;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace EFCore5InMvcApp
{
    public class Program
    {
        public static void Main(string[] args)
        {
            var host = CreateHostBuilder(args).Build();

            using (var scope = host.Services.CreateScope())
            {
                var services = scope.ServiceProvider;
                try
                {
                    var context = services.GetRequiredService<BookStore>();
                    BookStoreInitializer.Initialize(context);
                }
                catch (Exception ex)
                {
                    var logger = services.GetRequiredService<ILogger<Program>>();
                    logger.LogError(ex, "An error occurred while seeding the database.");
                }
            }

            host.Run();
        }

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                });
    }
}

```

On application startup, the `Main` method does the following operations.

* Get a database context instance from the dependency injection container.
* Call the seed method, passing to it the context.
* Dispose of the context when the seed method is done.

### Create Controller and Views

MVC controllers are responsible for responding to requests made against your website. Each browser request is mapped to a particular controller.

To create a controller, right-click the **Controllers** folder in Solution Explorer, and select **Add &gt; Controller...**

![](../.gitbook/assets/image%20%2816%29.png)

It will open the **Add Scaffold** dialog box.

![](../.gitbook/assets/image%20%2813%29.png)

 Select **MVC Controller with views, using Entity Framework**, and then click the **Add** button.

![](../.gitbook/assets/image%20%2817%29.png)

In the **Add MVC Controller with views, using Entity Framework** dialog box, select **Author \(EFCore5InMvcApp.Models\)** from the **Model class** and **BookStore \(EFCore5InMvcApp.DAL\)** from the **Data context class** dropdown.

Enter **AuthorController** \(not AuthorsController\) as a **Controller name** and click the **Add** button. 

![](../.gitbook/assets/image%20%2818%29.png)

The scaffolder creates an `AuthorController.cs` file and a set of views \(`.cshtml` files\) that work with the controller.

### Setup Menu Options

Open _**Views\Shared\\_Layout.cshtml**_, and add a menu entry for **Authors** after the **Home** menu option as shown below.

```csharp
<header>
    <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
        <div class="container">
            <a class="navbar-brand" asp-area="" asp-controller="Home" asp-action="Index">EFCore5InMvcApp</a>
            <button class="navbar-toggler" type="button" data-toggle="collapse" data-target=".navbar-collapse" aria-controls="navbarSupportedContent"
                    aria-expanded="false" aria-label="Toggle navigation">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="navbar-collapse collapse d-sm-inline-flex justify-content-between">
                <ul class="navbar-nav flex-grow-1">
                    <li class="nav-item">
                        <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Index">Home</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link text-dark" asp-area="" asp-controller="Author" asp-action="Index">Authors</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Privacy">Privacy</a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>
</header>

```

 Press Ctrl+F5 to run the project, click the **Authors** tab to see the test data.

![](../.gitbook/assets/image%20%2815%29.png)



