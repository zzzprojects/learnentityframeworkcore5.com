# WinForm

A **Windows Forms application** is an event-driven **application** supported by Microsoft's .NET Framework. Unlike a batch program, it spends most of its time simply waiting for the user to do something, such as fill in a text box or click a button.

* Windows Forms provides access to native Windows User Interface Common Controls by wrapping the existent Windows API in managed code. 
* With the help of Windows Forms, the .NET Framework provides a more comprehensive abstraction above the Win32 API than Visual Basic or MFC did.

### Create a WinForm App

The project type comes with all the template files you will need before adding anything. Let's open Visual Studio 2019,  if you haven't already installed Visual Studio, go to the [Visual Studio downloads](https://visualstudio.microsoft.com/downloads) page to install it for free.

![](../.gitbook/assets/image%20%2834%29.png)

On the start window, choose **Create a new project**.

![](../.gitbook/assets/image%20%2832%29.png)

On the **Create a new project** window, enter or type _windows forms_ in the search box. Next, choose **C\#** from the Language list, and then choose **Windows** from the Platform list. Select the **Windows Forms App \(.NET\)** template, and then choose **Next**.

![](../.gitbook/assets/image%20%2835%29.png)

In the **Configure your new project** window, type or enter _**EFCore5InWinFormApp**_ in the **Project name** box and click on the **Create** button, and you will see Visual Studio opens your new project.

### Install Entity Framework Core

To use Entity Framework Core we need to install [Microsoft.EntityFrameworkCore](https://www.nuget.org/packages/Z.EntityFramework.Extensions.EFCore/) library.  [I](https://www.nuget.org/packages/Z.EntityFramework.Extensions.EFCore/)t is available as a nuget package and you can install it using **Nuget Package Manager**.

In the **Package Manager Console** window, enter the following command.

```bash
PM> Install-Package Microsoft.EntityFrameworkCore
```

For SQL Server LocalDB, which is installed with Visual Studio, we need to install [Microsoft.EntityFrameworkCore.SqlServer](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer) and will get all the packages required for EF Core.

```bash
PM> Install-Package Microsoft.EntityFrameworkCore.SqlServer
```

### Create a Data Model and Database Context

In **Solution Explorer**, right-click on your project and choose **Add &gt; Class**. Enter a class file name **Author.cs** and add the following code.

```csharp
using System;
using System.Collections.Generic;
using System.Text;

namespace EFCore5InWinFormApp
{
    public class Author
    {
        public int AuthorId { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public DateTime BirthDate { get; set; }
    }
}
```

So let's create a new class file named **BookStore.cs**, and replace the following code.

```csharp
using Microsoft.EntityFrameworkCore;
using System;
using System.Collections.Generic;
using System.Text;

namespace EFCore5InWinFormApp
{
    public class AuthorContext : DbContext
    {
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            optionsBuilder.UseSqlServer(@"Data Source=(localdb)\ProjectsV13;Initial Catalog=AuthorDb;");            
        }

        public DbSet<Author> Authors { get; set; }
    }
}
```

We need to add **DataGridView** to display the data from the database, so let's select the **DataGridView** from the Toolbox and drag it to the Form1 as shown below. 

![](../.gitbook/assets/image%20%2841%29.png)

Now in the `Form1_Load` method, we will add some authors' data to the database first, and then we will retrieve the data and display it on the **DataGridView** as shown below.

```csharp
private void Form1_Load(object sender, EventArgs e)
{
    using (var context = new AuthorContext())
    {
        context.Database.EnsureCreated();
        var authors = new List<Author>
        {
            new Author { FirstName="Carson", LastName="Alexander", BirthDate = DateTime.Parse("1985-09-01")},
            new Author { FirstName="Meredith", LastName="Alonso", BirthDate = DateTime.Parse("1970-09-01")},
            new Author { FirstName="Arturo", LastName="Anand", BirthDate = DateTime.Parse("1963-09-01")},
            new Author { FirstName="Gytis", LastName="Barzdukas", BirthDate = DateTime.Parse("1988-09-01")},
            new Author { FirstName="Yan", LastName="Li", BirthDate = DateTime.Parse("2000-09-01")},
        };

        context.Authors.AddRange(authors);
        context.SaveChanges();

        dataGridView1.DataSource = authors;
    }
}

```

If you run the application, you will see that authors are successfully inserted into the database and display on the **DataGridView**.

![](../.gitbook/assets/image%20%2840%29.png)

