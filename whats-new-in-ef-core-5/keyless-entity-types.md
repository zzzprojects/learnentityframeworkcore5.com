# Keyless Entity Types

In addition to regular entity types, an EF Core model can contain _keyless entity types_, which can be used to carry out database queries against data that doesn't contain key values.

## Defining Keyless entity types

Keyless entity types can be defined using either the Data Annotation or the Fluent API.

### Data Annotations

```csharp
class BlogsContext : DbContext
{
    public DbSet<BlogPostsCount> BlogPostCounts { get; set; }
}

[Keyless]
public class BlogPostsCount
{
    public string BlogName { get; set; }
    public int PostCount { get; set; }
}
```

### Fluent API

```csharp
class BlogsContext : DbContext
{
    public DbSet<BlogPostsCount> BlogPostCounts { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<BlogPostsCount>()
            .HasNoKey();
    }
}

public class BlogPostsCount
{
    public string BlogName { get; set; }
    public int PostCount { get; set; }
}
```

### Keyless entity types characteristics

Keyless entity types support many of the same mapping capabilities as regular entity types, like inheritance mapping and navigation properties. On relational stores, they can configure the target database objects and columns via fluent API methods or data annotations.

However, they are different from regular entity types, such as:

* It cannot have a key defined.
* Are never tracked for changes in the _DbContext_ and therefore are never inserted, updated or deleted on the database.
* Are never discovered by convention.
* Only support a subset of navigation mapping capabilities, specifically:
  * They may never act as the principal end of a relationship.
  * They may not have navigations to owned entities
  * They can only contain reference navigation properties pointing to regular entities.
  * Entities cannot contain navigation properties to keyless entity types.
* Need to be configured with a `[Keyless]` data annotation or a `.HasNoKey()` method call.
* May be mapped to a _defining query_. A defining query is a query declared in the model that acts as a data source for a keyless entity type.

### Usage scenarios

Some of the main usage scenarios for keyless entity types are:

* Serving as the return type for raw SQL queries.
* Mapping to database views that do not contain a primary key.
* Mapping to tables that do not have a primary key defined.
* Mapping to queries defined in the model.

### Mapping to database objects

Mapping a keyless entity type to a database object is achieved using the `ToTable` or `ToView` fluent API. From the perspective of EF Core, the database object specified in this method is a _view_, meaning that it is treated as a read-only query source and cannot be the target of the update, insert or delete operations. However, this does not mean that the database object is actually required to be a database view. It can alternatively be a database table that will be treated as read-only. Conversely, for regular entity types, EF Core assumes that a database object specified in the `ToTable` method can be treated as a _table_, meaning that it can be used as a query source but also targeted by the update, delete and insert operations. In fact, you can specify the name of a database view in `ToTable` and everything should work fine as long as the view is configured to be updatable on the database.

> \[!NOTE\] `ToView` assumes that the object already exists in the database and it won't be created by migrations.

### Example

The following example shows how to use keyless entity types to query a database view.

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder
        .Entity<BlogPostsCount>(eb =>
        {
            eb.HasNoKey();
            eb.ToView("View_BlogPostCounts");
            eb.Property(v => v.BlogName).HasColumnName("Name");
        });
}
```

