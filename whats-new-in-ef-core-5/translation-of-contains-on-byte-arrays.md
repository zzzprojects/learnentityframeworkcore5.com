# Translation of Contains on byte arrays

Queries using Contains on byte\[\] properties are now translated to SQL. 

```csharp
var blogs = context.Blogs.Where(e => e.Picture.Contains((byte)127)).ToList();
```

Translates to the following on SQL Server:

```sql
info: 12/5/2019 11:42:42.022 RelationalEventId.CommandExecuted[20101] (Microsoft.EntityFrameworkCore.Database.Command)
      Executed DbCommand (1ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      SELECT [b].[Id], [b].[Picture], [b].[Title]
      FROM [Blogs] AS [b]
      WHERE CHARINDEX(0x7F, [b].[Picture]) > 0
```

