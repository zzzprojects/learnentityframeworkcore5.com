# Index Attribute

Entity Framework 6 provides the `Index` attribute to create an index on a particular column in the database.

```csharp
public class Book
{
    public int Id { get; set; }
    [Index]
    public string Title { get; set; }
    public string Category { get; set; }
    public int AuthorId { get; set; }
    public Author Author { get; set; }
}
```

But in EF Core, indexes cannot be created using data annotations. You have to use the Fluent API to specify an index on a column as shown below.

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Book>()
        .HasIndex(b => b.Title);
}
```

Now in EF Core, the new `Index` attribute can be placed on an entity type to specify an index for one or more columns.  

```csharp
[Index(nameof(Title), IsUnique = true)]
public class Book
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Category { get; set; }
    public int AuthorId { get; set; }
    public Author Author { get; set; }
}

```

Now when you add a migration, you will see that the index is created for the `Title` column.

```sql
CREATE TABLE [dbo].[Books] (
    [Id]       INT            IDENTITY (1, 1) NOT NULL,
    [Title]    NVARCHAR (450) NULL,
    [Category] NVARCHAR (MAX) NULL,
    [AuthorId] INT            NOT NULL,
    CONSTRAINT [PK_Books] PRIMARY KEY CLUSTERED ([Id] ASC),
    CONSTRAINT [FK_Books_Authors_AuthorId] FOREIGN KEY ([AuthorId]) REFERENCES [dbo].[Authors] ([Id]) ON DELETE CASCADE
);

GO
CREATE NONCLUSTERED INDEX [IX_Books_AuthorId]
    ON [dbo].[Books]([AuthorId] ASC);

GO
CREATE UNIQUE NONCLUSTERED INDEX [IX_Books_Title]
    ON [dbo].[Books]([Title] ASC) WHERE ([Title] IS NOT NULL);


```

You can also use the `Index` attribute to specify an index spanning multiple columns.

```csharp
[Index(nameof(FirstName), nameof(LastName), IsUnique = true)]
public class Author
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public ICollection<Book> Books { get; set; }
}

```

For SQL Server, Migrations will then generate the following SQL.

```sql
CREATE TABLE [dbo].[Authors] (
    [Id]        INT            IDENTITY (1, 1) NOT NULL,
    [FirstName] NVARCHAR (450) NULL,
    [LastName]  NVARCHAR (450) NULL,
    CONSTRAINT [PK_Authors] PRIMARY KEY CLUSTERED ([Id] ASC)
);

GO
CREATE UNIQUE NONCLUSTERED INDEX [IX_Authors_FirstName_LastName]
    ON [dbo].[Authors]([FirstName] ASC, [LastName] ASC) 
    WHERE ([FirstName] IS NOT NULL AND [LastName] IS NOT NULL);


```



