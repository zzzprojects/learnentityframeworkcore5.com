# Console

A **Console application** is a program designed to be used via a text-only computer interface, such as a text terminal, the command line interface, etc.

* A user typically interacts with a console application using only a keyboard and display screen, as opposed to GUI applications, which normally require the use of a mouse or other pointing device.
* Many console applications such as command-line interpreters are command-line tools, but numerous text-based user interface \(TUI\) programs also exist.

### Create a Console App

The project type comes with all the template files you will need before adding anything. Let's open Visual Studio 2019,  if you haven't already installed Visual Studio, go to the [Visual Studio downloads](https://visualstudio.microsoft.com/downloads) page to install it for free.

![](../.gitbook/assets/image%20%289%29.png)

On the start window, choose **Create a new project**.

![](../.gitbook/assets/image%20%2812%29.png)

On the **Create a new project** window, enter or type _console_ in the search box. Next, choose **C\#** from the Language list, and then choose **Windows** from the Platform list. Select the **Console App \(.NET Core\)** template, and then choose **Next**.

![](../.gitbook/assets/image%20%288%29.png)

 In the **Configure your new project** window, type or enter _**EFCore5InConsoleApp**_ in the **Project name** box and click on the **Create** button. 

![](../.gitbook/assets/image%20%2810%29.png)

Visual Studio opens your new project and includes the default "Hello World" code in your project.

### Install Entity Framework Core

Let's create a new application using the **Console App \(.NET Core\)** template and install [Microsoft.EntityFrameworkCore](https://www.nuget.org/packages/Z.EntityFramework.Extensions.EFCore/). [I](https://www.nuget.org/packages/Z.EntityFramework.Extensions.EFCore/)t is available as a nuget package and you can install it using **Nuget Package Manager**.

In the **Package Manager Console** window, enter the following command.

```bash
PM> Install-Package Microsoft.EntityFrameworkCore
```

For SQL Server LocalDB, which is installed with Visual Studio, we need to install [Microsoft.EntityFrameworkCore.SqlServer](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer) and will get all the packages required for EF Core.

```bash
PM> Install-Package Microsoft.EntityFrameworkCore.SqlServer
```

### Create a Data Model and Database Context

To create a data model for our application, we will start with the following two entities.

```csharp
public class Author
{
    public int AuthorId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public DateTime BirthDate { get; set; }
    public List<Book> Books { get; set; }
}

public class Book
{
    public int BookId { get; set; }
    public string Title { get; set; }
    public Author Author { get; set; }
}
```

The database context class provides the main functionality to coordinate Entity Framework with a given data model. So, let's add a new `BookStore` class which will inherit the `DbContext` class.

```csharp
public class BookStore : DbContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer(@"Data Source=(localdb)\ProjectsV13;Initial Catalog=BookStoreDb;");
    }
        
    public DbSet<Author> Authors { get; set; }
    public DbSet<Book> Books { get; set; }
}
```

Now, we are done with the required classes and database creation, let's add some authors and book records to the database and then retrieve them as shown below.

```csharp
static void Main(string[] args)
{
    using (var context = new BookStore())
    {
        context.Database.EnsureCreated();

        var authors = new List<Author>
        {
            new Author
            {
                FirstName ="Carson",
                LastName ="Alexander",
                BirthDate = DateTime.Parse("1985-09-01"),
                Books = new List<Book>()
                {
                    new Book { Title = "Introduction to Machine Learning"},
                    new Book { Title = "Advanced Topics on Machine Learning"},
                    new Book { Title = "Introduction to Computing"}
                }
            },
            new Author
            {
                FirstName ="Meredith",
                LastName ="Alonso",
                BirthDate = DateTime.Parse("1970-09-01"),
                Books = new List<Book>()
                {
                    new Book { Title = "Introduction to Microeconomics"}
                }
            },
            new Author
            {
                FirstName ="Arturo",
                LastName ="Anand",
                BirthDate = DateTime.Parse("1963-09-01"),
                Books = new List<Book>()
                {
                    new Book { Title = "Calculus I"},
                    new Book { Title = "Calculus II"}
                }
            }
        };

        context.Authors.AddRange(authors);
        context.SaveChanges();
    }

    using (var context = new BookStore())
    {
        var list = context.Authors
            .Include(a => a.Books)
            .ToList();

        foreach (var author in list)
        {
            Console.WriteLine(author.FirstName + " " + author.LastName);

            foreach (var book in author.Books)
            {
                Console.WriteLine("\t" + book.Title);
            }
        }
    }
}

```

If you run the application, you will see that authors and books are successfully inserted into the database and also print on the console window.

![](../.gitbook/assets/image%20%2811%29.png)

