# Drop Column from SQLite Database

SQLite is relatively limited in its schema manipulation capabilities as compared to other databases. For example, dropping a column from an existing table requires that the entire table be dropped and re-created. 

In EF Core 5.0,  migrations now support the automatic rebuilding of the table for schema changes that require it.

Let's suppose we have a simple model that contains a `Book` entity as shown below.

```csharp
public class Book
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Category { get; set; }
}

public class EntityContext : DbContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder opBuilder)
    {
        opBuilder.UseSqlite("Data Source=D:\\Blogging.db");
    }
    
    public DbSet<Book> Books { get; set; }
}

```

To create the database, let's run the following migration command in **Package Manager Console**.

```bash
PM> Add-Migration Init
```

You will see that migration creates the following script. 

```csharp
public partial class Init : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "Books",
            columns: table => new
            {
                Id = table.Column<int>(nullable: false)
                    .Annotation("Sqlite:Autoincrement", true),
                Title = table.Column<string>(nullable: true),
                Category = table.Column<string>(nullable: true)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_Books", x => x.Id);
            });
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable(
            name: "Books");
    
```

To apply the above script to the database, run the following command to update the database.

```bash
PM> Update-Database
```

You will see that the database is created that contains a `Books` table.

Now we want to remove the `Category` property so let's remove that property from the `Book` class as shown below.

```csharp
public class Book
{
    public int Id { get; set; }
    public string Title { get; set; }
}
```

Add a new migration using the following migration command.

```csharp
PM> Add-Migration RemoveCategory
```

It will create the following script to update the database.

```csharp
public partial class RemoveCategory : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropColumn(
            name: "Category",
            table: "Books");
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.AddColumn<string>(
            name: "Category",
            table: "Books",
            type: "TEXT",
            nullable: true);
    }
}

```

To update the database with the above script, run the following.

```bash
PM> Update-Database
```

Before EF Core 5.0, this update will fail, because the column cannot be dropped. 

![](../.gitbook/assets/image%20%282%29.png)

In EF Core 5.0, you will see that migrations will instead rebuild the table successfully.

![](../.gitbook/assets/image%20%284%29.png)

### How it Works

* A temporary table is created with the desired schema for the new table.
* Data is copied from the current table into the temporary table.
* Foreign key enforcement is switched off.
* The current table is dropped
* The temporary table is renamed to be the new table.

