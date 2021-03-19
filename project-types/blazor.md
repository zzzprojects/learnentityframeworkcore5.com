<a href="https://entityframework-extensions.net/">**Improve EF Core performance with EF Extensions**</a>

<a href="https://entityframework-extensions.net/">
<img src="https://zzzprojects.github.io/images/logo/entityframework-extensions-pub.jpg" width="600" />
</a>

# Blazor

**Blazor** is a free and open-source web framework that enables developers to create web apps using C\# and HTML. Blazor is a framework for building interactive client-side web UI with [.NET](https://docs.microsoft.com/en-us/dotnet/standard/tour).

* Create rich interactive UIs using [C\#](https://docs.microsoft.com/en-us/dotnet/csharp/) instead of [JavaScript](https://www.javascript.com/).
* Share server-side and client-side app logic written in .NET.
* Render the UI as HTML and CSS for wide browser support, including mobile browsers.
* Integrate with modern hosting platforms, such as [Docker](https://docs.microsoft.com/en-us/dotnet/standard/microservices-architecture/container-docker-introduction/index).



### Create Blazor App

To start, we will create an ASP.NET Core hosted Blazor App project. The project type comes with all template files to create a Blazor application before adding anything. Let's open Visual Studio 2019. If you haven't already installed Visual Studio, go to the [Visual Studio downloads](https://visualstudio.microsoft.com/downloads) page to install it for free.

![](../.gitbook/assets/image%20%2853%29.png)

On the start window, choose **Create a new project**.

![](../.gitbook/assets/image%20%2829%29.png)

On the **Create a new project** window, enter or type _blazor_ in the search box. Next, choose **C\#** from the Language list. Select the **Blazor App** template, and then choose **Next**.

![](../.gitbook/assets/image%20%2860%29.png)

In the **Configure your new project** window, type or enter _**EFCore5InBlazorApp**_ in the **Project name** box and click on the **Create** button.

![](../.gitbook/assets/image%20%2840%29.png)

Now select the **Blazor WebAssembly App** project template and select the **ASP.NET Core hosted** check box. Click on the **Create** button to create an ASP.NET Core-hosted application.

![](../.gitbook/assets/image%20%2861%29.png)

Visual Studio opens your new project and includes the default code files in your project as shown in the **Solution Explorer**. You will see that 3 projects are created inside this solution.

* **EFCore5InBlazorApp.Client**: It has the client-side code and contains the pages that will be rendered on the browser.
* **EFCore5InBlazorApp.Server**: It has the server-side code, such as DB related operations and the web API.
* **EFCore5InBlazorApp.Shared**: It contains the shared code that can be accessed by both client and server.

### Install Entity Framework Core

To use Entity Framework Core we need to install [Microsoft.EntityFrameworkCore](https://www.nuget.org/packages/Z.EntityFramework.Extensions.EFCore/) library in the **EFCore5InBlazorApp.Server** project. [It](https://www.nuget.org/packages/Z.EntityFramework.Extensions.EFCore/) is available as a nuget package, and you can install it using **Nuget Package Manager**.

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

To add a data model to the application, right-click on **EFCore5InBlazorApp.Shared** project and then select **Add &gt; New Folder** and name the folder **Models**. Right-click on the **Models** folder and select **Add &gt; Class...,**  enter a class file name **Author.cs,** and add the following code.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace EFCore5InBlazorApp.Shared.Models
{
    public class Author
    {
        public string Id { get; set; }
        public string Name { get; set; }
        public string Gender { get; set; }
        public string Address { get; set; }
    }
}
```

 So let's create a folder in your project by right-clicking on **EFCore5InBlazorApp.Server** project in **Solution Explorer** and click **Add &gt; New Folder**. Name the folder **DAL** \(Data Access Layer\). In that folder, create a new class file named **AuthorContext.cs**, and replace the following code.

```csharp
using EFCore5InBlazorApp.Shared.Models;
using Microsoft.EntityFrameworkCore;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace EFCore5InBlazorApp.Server.DAL
{
    public class AuthorContext : DbContext
    {
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            optionsBuilder.UseSqlServer(@"Data Source=(localdb)\\ProjectsV13;Initial Catalog=AuthorDb;Trusted_Connection=True;MultipleActiveResultSets=true");
        }

        public DbSet<Author> Authors { get; set; }
    }
}

```

### Register Context Class

To register `AuthoContext` as a service, open `Startup.cs` from **EFCore5InBlazorApp.Server** project, and call the `AddDnContext` in the `ConfigureServices` method.

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
    "DefaultConnection": "Data Source=(localdb)\\ProjectsV13;Initial Catalog=AuthorDb;Trusted_Connection=True;MultipleActiveResultSets=true"
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

The above connection string specifies that the Entity Framework will use a `LocalDB` database named `AuthorDb`.

### Initialize Database

The Entity Framework will create an empty database for you. So we need to write a method that's called after the database is created to populate it with test data.

In the **DAL** folder of **EFCore5InBlazorApp.Server** project, add a new class `AuthorContextInitializer` and replace the following code.

```csharp
using EFCore5InBlazorApp.Shared.Models;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace EFCore5InBlazorApp.Server.DAL
{
    public class AuthorContextInitializer
    {
        public static void Initialize(AuthorContext context)
        {

            context.Database.EnsureCreated();

            // Look for any authors.
            if (context.Authors.Any())
            {
                return;   // DB has been seeded
            }

            var authors = new List<Author>
            {
                new Author { Name="Carson Alexander", Gender="Male", Address = "Skagen 21, Stavanger, Norway"},
                new Author { Name="Meredith Alonso", Gender="Female", Address = "Keskuskatu 45, Helsinki, Finland"},
                new Author { Name="Arturo Anand", Gender="Male", Address = "305 - 14th Ave. S. Suite 3B, Seattle, USA"},
            };

            authors.ForEach(a => context.Authors.Add(a));
            context.SaveChanges();

        }
    }
}
```

* The above code creates a database when needed and loads test data into the new database.
* It also checks if there are any authors in the database, and if not, it assumes the database is new and needs to be seeded with test data.

In `Program.cs` file of **EFCore5InBlazorApp.Server** project, replace the following code in the `Main` method.

```csharp
using EFCore5InBlazorApp.Server.DAL;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace EFCore5InBlazorApp.Server
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
                    var context = services.GetRequiredService<AuthorContext>();
                    AuthorContextInitializer.Initialize(context);
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

### Create Web API Controller

To create a controller, right-click the **Controllers** folder in **EFCore5InBlazorApp.Server**, and select **Add &gt; Controller...** and it will open the **Add Scaffold** dialog box.

![](../.gitbook/assets/image%20%2831%29.png)

Select **API Controller - Empty**, and then click the **Add** button.

![](../.gitbook/assets/image%20%2833%29.png)

Select **API Controller - Empty**, enter **AuthorController** \(not AuthorsController\) as a **Controller name** and click the **Add** button. It will create an empty controller, let's add the following code which contains all the basic CRUD operations.

```csharp
using EFCore5InBlazorApp.Server.DAL;
using EFCore5InBlazorApp.Shared.Models;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace EFCore5InBlazorApp.Server.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class AuthorController : ControllerBase
    {
        AuthorContext context = new AuthorContext();

        [HttpGet]
        [Route("Index")]
        public IEnumerable<Author> Index()
        {
            return context.Authors.ToList();
        }

        [HttpPost]
        [Route("Create")]
        public void Create([FromBody] Author author)
        {
            if (ModelState.IsValid)
            {
                context.Authors.Add(author);
                context.SaveChanges();
            }
        }

        [HttpGet]
        [Route("Details/{id}")]
        public Author Details(int id)
        {
            Author author = context.Authors.Find(id);
            return author;
        }

        [HttpPut]
        [Route("Edit")]
        public void Edit([FromBody] Author author)
        {
            if (ModelState.IsValid)
            {
                context.Entry(author).State = EntityState.Modified;
                context.SaveChanges();
            }
        }

        [HttpDelete]
        [Route("Delete/{id}")]
        public void Delete(int id)
        {
            Author author = context.Authors.Find(id);
            context.Authors.Remove(author);
            context.SaveChanges();
        }
    }
}

```

### Add Views

To display all the authors, we need to create a view by right-clicking on the **Pages** folder in **EFCore5InBlazorApp.Client** project and select **Add &gt; Razor Component...**

![](../.gitbook/assets/image%20%2850%29.png)

Select **Razor Component**, enter **GetAuthors.razor** as a **Name,** and click the **Add** button. It will create an empty view and then add the following code.

```csharp
@page "/getauthors"
@using EFCore5InBlazorApp.Shared.Models
@inject HttpClient Http

<h1>Authors</h1>

<p>This component demonstrates fetching Author data from the server.</p>

<p>
    <a href="/addauthor">Create New</a>
</p>

@if (authors == null)
{
    <p><em>Loading...</em></p>
}
else
{
    <table class='table'>
        <thead>
            <tr>
                <th>Name</th>
                <th>Gender</th>
                <th>Address</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var author in authors)
            {
                <tr>
                    <td>@author.Name</td>
                    <td>@author.Gender</td>
                    <td>@author.Address</td>
                    <td>
                        <a href='/editemployee/@author.Id'>Edit</a>  |
                        <a href='/delete/@author.Id'>Delete</a>
                    </td>
                </tr>
            }
        </tbody>
    </table>
}
@code {

    Author[] authors;

    protected override async Task OnInitializedAsync()
    {
        authors = await Http.GetFromJsonAsync<Author[]>("/api/Author/Index");
    }
}
```

Now let's add one more page that will use to add a new author, call it **AddAuthor.razor,** and add the following code.

```csharp
@page "/addauthor"
@using EFCore5InBlazorApp.Shared.Models
@inject HttpClient Http
@inject NavigationManager urlNavigationManager

<h1>Create Author</h1>
<hr />

<EditForm Model="@author" OnValidSubmit="CreateAuthor">
    <DataAnnotationsValidator />
    <div class="form-group row">
        <label class="control-label col-md-12">Name</label>
        <div class="col-md-4">
            <input class="form-control" @bind="author.Name" />
        </div>
        <ValidationMessage For="@(() => author.Name)" />
    </div>
    <div class="form-group row">
        <label class="control-label col-md-12">Gender</label>
        <div class="col-md-4">
            <select class="form-control" @bind="author.Gender">
                <option value="">-- Select Gender --</option>
                <option value="Male">Male</option>
                <option value="Female">Female</option>
            </select>
        </div>
        <ValidationMessage For="@(() => author.Gender)" />
    </div>
    <div class="form-group row">
        <label class="control-label col-md-12">Address</label>
        <div class="col-md-4">
            <input class="form-control" @bind="author.Address" />
        </div>
        <ValidationMessage For="@(() => author.Address)" />
    </div>
    <div class="form-group">
        <button type="submit" class="btn btn-primary">Save</button>
        <button class="btn btn-light" @onclick="Cancel">Cancel</button>
    </div>
</EditForm>

@code {
    Author author = new Author();

    protected async Task CreateAuthor()
    {
        await Http.PostAsJsonAsync("/api/Author/Create", author);
        urlNavigationManager.NavigateTo("/getauthors");
    }

    void Cancel()
    {
        urlNavigationManager.NavigateTo("/getauthors");
    }
}
```

Let's run your application and click on the Authors menu option.

![](../.gitbook/assets/image%20%2838%29.png)

To add a new author click on the **Create New** link and it will open the **Create Author** Page.

![](../.gitbook/assets/image%20%2826%29.png)

Enter **Name, Gender,** and **Address** and click on the **Save** button and you will see a new author is added. 

![](../.gitbook/assets/image%20%2855%29.png)

