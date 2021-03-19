<a href="https://entityframework-extensions.net/">**Improve EF Core performance with EF Extensions**</a>

<a href="https://entityframework-extensions.net/">
<img src="https://zzzprojects.github.io/images/logo/entityframework-extensions-pub.jpg" width="600" />
</a>

# Raw SQL Queries in EF-Core

In Entity Framework Core, if your LINQ Query is not able to execute the application properly, then there is an option for embedding custom raw SQL queries into the database according to the requirement of the application.

If the LINQ queries generated automatically by the system are not suitable to the application, then the Raw SQL queries can help in executing the command.

## Using FromSqlRaw Method instead of FromSql

With the release of EF Core 3, Microsoft removed the `FromSql` Method and replaced it with the `FromSqlRaw` Method.

They made this change to make it easy for developers to call the required action. In the earlier versions of EF Core, the `FromSql` method made it confusing for the system to accidentally trigger the raw string method when the developer wanted to call the interpolated string method or vice-versa, which give unsuccessful results.

Now with the release of EF Core 3.0, a parameterized query can be developed using the `FromSqlRaw` method.

## Parameterized Query

In Entity Framework Core, there is also support for parameterized queries, which means the developer can pre-compile queries into the applications.

* This method helps in preventing the SQL injection attack on the application, by pre-compiling the SQL Queries.
* So that when the statement is about to be executed the developer can add the suitable parameters for it to be successful.
* Usually, in parameterized queries, placeholders are used in place of parameters and these placeholders are replaced by the parameter values at the time of execution.

```csharp
// Format string
var actor = db.Actors
    .FromRawSql("SELECT * From Actors Where ActorID = {0}", id).FirstOrDefault();

// String interpolation
var actor = db.Actors
    .FromRawSql($"SELECT * From Actors Where ActorID = {Id}").FirstOrDefault();
```

You can also explicitly create DbParameter objects for the provider. The 1st example shows parameter construction for SQLite, and the 2nd example for SQL Server:

```csharp
var param1 = new SqliteParameter("@Id", id);
var actor = db.Actors
    .FromRawSql($"SELECT * From Actors Where ActorId = @Id", param1)
    .FirstOrDefault();
    
var param1= new SqlParameter("@Id", id);
var actor = db.Actors
    .FromRawSql($"SELECT * From Actors Where ActorId = @Id", param1)
    .FirstOrDefault();
```

## Composing Over Raw SQL

In Entity Framework Core, it is possible to compose over raw SQL queries using the LINQ operators.

Due to which EF Core will treat SQL statements as a sub-query and will put this data up in the database.

But in composing over raw SQL with LINQ operator, the statement can only be executed if the available raw SQL query is composable.

Otherwise, it will reject the composing request, resulting in a failed execution of the application.

```csharp
var searchTerm = "Horror";
var Movies = context.Movies
    .FromSqlInterpolated($"SELECT * FROM dbo.MovieCategories({searchTerm})")
    .Where(b => b.Rating > 4)
    .OrderByDescending(b => b.Rating)
    .ToList();
```

Including related data

```csharp
var searchTerm = "Horror";
var Movies = context.Movies
    .FromSqlInterpolated($"SELECT * FROM dbo.MovieCategories({searchTerm})")
    .Include(b => b.directors)
    .ToList();
```

## Change Tracking \(SQL Server\)

In Entity Framework Core, the SQL Server supports a very essential feature called Change Tracking,

This feature enables the SQL servers to change, update or modify the values in the SQL queries according to the need.

Change Tracking is a very useful feature of SQL Server as it allows us to modify the current data, otherwise, everything has to be initiated from start.

```csharp
var searchTerm = "Horror";
var Movies = context.Movies
    .FromSqlInterpolated($"SELECT * FROM dbo.MovieCategories({searchTerm})")
    .AsNoTracking()
    .ToList();
```

## Limitations of Using Raw SQL

1. All the SQL queries must return the queries of the same type, otherwise, the program would fail to run.
2. The SQL queries must return all the columns in the table, also the returned column names should match the names that are mapped into the database.
3. The SQL queries cannot use the join queries to get the previous data, they should use the Include method instead.

