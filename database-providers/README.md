# Database Providers

Entity Framework Core can access many different databases through plug-in libraries called database providers.

*  Database providers can extend EF Core to enable functionality unique to specific databases. 
* Some concepts are common to most databases, and are included in the primary EF Core components. 
* Such concepts include expressing queries in LINQ, transactions, and tracking changes to objects once they are loaded from the database. 
* Some concepts are specific to a particular provider, for example, the SQL Server provider allows you to [configure memory-optimized tables](https://docs.microsoft.com/en-us/ef/core/providers/sql-server/memory-optimized-tables) \(a feature specific to SQL Server\). 

The following providers are supported in Entity Framework Core 5.

| Name | NuGet Package |
| :--- | :--- |
| \[SQL Server\]\(/sql-server\) | [Microsoft.EntityFrameworkCore.SqlServer](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer) |
| SQLite | [Microsoft.EntityFrameworkCore.Sqlite](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Sqlite) |
| InMemory | [Microsoft.EntityFrameworkCore.InMemory](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.InMemory) |
| Cosmos | [Microsoft.EntityFrameworkCore.Cosmos](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Cosmos) |
| PostgreSQL   | [Npgsql.EntityFrameworkCore.PostgreSQL](https://www.nuget.org/packages/Npgsql.EntityFrameworkCore.PostgreSQL) |



