<a href="https://entityframework-extensions.net/">**Improve EF Core performance with EF Extensions**</a>

<a href="https://entityframework-extensions.net/">
<img src="https://zzzprojects.github.io/images/logo/entityframework-extensions-pub.jpg" width="600" />
</a>

# Support for Fields using Lambda

EF Core 5 allows you to use the lambda methods in the ModelBuilder for fields as well as properties. Let's suppose you don't want to use properties for some reason and decide to use public fields.

```csharp
public class Book
{
    public int Id;
    public string Title;
    public string Category;
    public int AuthorId;
    public Author Author;
}

public class Author
{
    public int Id;
    public string Name;
    public ICollection<Book> Books;
}
```

In EF Core 5, you can map these fields using the lambda builders as shown below. 

```csharp
public class EntityContext : DbContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder opBuilder)
    {
        opBuilder.UseSqlServer("Data Source=(localdb)\\ProjectsV13;Initial Catalog=BookContextDb;");
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Author>(b =>
        {
            b.Property(e => e.Id);
            b.Property(e => e.Name);
        });

        modelBuilder.Entity<Book>(b =>
        {
            b.Property(e => e.Id);
            b.Property(e => e.Title);
            b.Property(e => e.Category);
            b.Property(e => e.AuthorId);
            b.HasOne(e => e.Author).WithMany(e => e.Books);
        });
    }

    public DbSet<Author> Blogs { get; set; }
    public DbSet<Book> Posgs { get; set; }
}
```

It will create the following tables on the SQL Server.

```sql
CREATE TABLE [dbo].[Authors] (
    [Id]   INT            IDENTITY (1, 1) NOT NULL,
    [Name] NVARCHAR (MAX) NULL,
    CONSTRAINT [PK_Authors] PRIMARY KEY CLUSTERED ([Id] ASC)
);

CREATE TABLE [dbo].[Books] (
    [Id]       INT            IDENTITY (1, 1) NOT NULL,
    [AuthorId] INT            NOT NULL,
    [Category] NVARCHAR (MAX) NULL,
    [Title]    NVARCHAR (MAX) NULL,
    CONSTRAINT [PK_Books] PRIMARY KEY CLUSTERED ([Id] ASC),
    CONSTRAINT [FK_Books_Authors_AuthorId] FOREIGN KEY ([AuthorId]) REFERENCES [dbo].[Authors] ([Id]) ON DELETE CASCADE
);
```

Before EF Core 5, if you try to map these fields using the model builder, It will throw the following exception.

```csharp
System.ArgumentException: 'The expression 'e => e.Id' is not a valid property expression. The expression should represent a simple property access: 't => t.MyProperty'. (Parameter 'propertyAccessExpression')'
```



