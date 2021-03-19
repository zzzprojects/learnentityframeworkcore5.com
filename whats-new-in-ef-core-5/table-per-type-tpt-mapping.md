<a href="https://entityframework-extensions.net/">**Improve EF Core performance with EF Extensions**</a>

<a href="https://entityframework-extensions.net/">
<img src="https://zzzprojects.github.io/images/logo/entityframework-extensions-pub.jpg" width="600" />
</a>

# Table-per-type \(TPT\) mapping

Table-per-type inheritance uses a separate table in the database to maintain data for non-inherited properties and key properties for each type in the inheritance hierarchy.

* Table per Type is about representing inheritance relationships as relational foreign key associations.
* Every class and subclass including abstract classes has its own table.
* The table for subclasses contains columns only for each non-inherited property along with a primary key that is also a foreign key of the base class table.

By default, EF Core maps an inheritance hierarchy of .NET types to a single database table. This is known as table-per-hierarchy \(TPH\) mapping. EF Core 5.0 also allows mapping each .NET type in an inheritance hierarchy to a different database table; known as table-per-type \(TPT\) mapping.

Let's consider the following simple model with a mapped hierarchy.

```csharp
public class Person
{
    public int Id { get; set; }
    public string FullName { get; set; }
}

public class Student : Person
{
    public DateTime EnrollmentDate { get; set; }
}

public class Teacher : Person
{
    public DateTime HireDate { get; set; }
}
```

Here is the context class implementation without any additional configuration.

```csharp
public class EntityContext : DbContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder opBuilder)
    {
        opBuilder.UseSqlServer("Data Source=(localdb)\\ProjectsV13;Initial Catalog=PeopleContextDb;");
    }
    public DbSet<Person> People { get; set; }
    public DbSet<Student> Students { get; set; }
    public DbSet<Teacher> Teachers { get; set; }
}
```

By default, EF Core will map this to a single table.

```sql
CREATE TABLE [dbo].[People] (
    [Id]             INT            IDENTITY (1, 1) NOT NULL,
    [FullName]       NVARCHAR (MAX) NULL,
    [Discriminator]  NVARCHAR (MAX) NOT NULL,
    [EnrollmentDate] DATETIME2 (7)  NULL,
    [HireDate]       DATETIME2 (7)  NULL,
    CONSTRAINT [PK_People] PRIMARY KEY CLUSTERED ([Id] ASC)
);
```

In the TPT mapping pattern, all the types are mapped to individual tables. Properties that belong solely to a base type or derived type are stored in a table that maps to that type. 

Entity types can be mapped to different tables using mapping attributes.

```csharp
[Table("People")]
public class Person
{
    public int Id { get; set; }
    public string FullName { get; set; }
}

[Table("Students")]
public class Student : Person
{
    public DateTime EnrollmentDate { get; set; }
}

[Table("Teachers")]
public class Teacher : Person
{
    public DateTime HireDate { get; set; }
}
```

However, mapping each entity type to a different table will instead result in one table per type.

```sql
CREATE TABLE [dbo].[People] (
    [Id]       INT            IDENTITY (1, 1) NOT NULL,
    [FullName] NVARCHAR (MAX) NULL,
    CONSTRAINT [PK_People] PRIMARY KEY CLUSTERED ([Id] ASC)
);

CREATE TABLE [dbo].[Students] (
    [Id]             INT           NOT NULL,
    [EnrollmentDate] DATETIME2 (7) NOT NULL,
    CONSTRAINT [PK_Students] PRIMARY KEY CLUSTERED ([Id] ASC),
    CONSTRAINT [FK_Students_People_Id] FOREIGN KEY ([Id]) REFERENCES [dbo].[People] ([Id])
);

CREATE TABLE [dbo].[Teachers] (
    [Id]       INT           NOT NULL,
    [HireDate] DATETIME2 (7) NOT NULL,
    CONSTRAINT [PK_Teachers] PRIMARY KEY CLUSTERED ([Id] ASC),
    CONSTRAINT [FK_Teachers_People_Id] FOREIGN KEY ([Id]) REFERENCES [dbo].[People] ([Id])
);
```

Tables that map to derived types also store a foreign key that joins the derived table with the base table.

