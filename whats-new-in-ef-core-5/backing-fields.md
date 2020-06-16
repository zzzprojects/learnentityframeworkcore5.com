# Backing Fields

Backing fields allow EF to read and/or write to a field rather than a property. This can be useful when encapsulation in the class is being used to restrict the use of and/or enhance the semantics around access to the data by application code, but the value should be read from and/or written to the database without using those restrictions/enhancements.

## Basic configuration

By convention, the following fields will be discovered as backing fields for a given property \(listed in precedence order\).

* `_<camel-cased property name>`
* `_<property name>`
* `m_<camel-cased property name>`
* `m_<property name>`

In the following sample, the `Url` property is configured to have `_url` as its backing field.

```csharp
class MyContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
}

public class Blog
{
    private string _url;

    public int BlogId { get; set; }

    public string Url
    {
        get { return _url; }
        set { _url = value; }
    }
}
```

The backing fields are only discovered for properties that are included in the model. 

You can also configure backing fields by using a Data Annotation \(available in EFCore 5.0\) or the Fluent API, e.g. if the field name doesn't correspond to the above conventions:

### Data Annotations

```csharp
class MyContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
}

public class Blog
{
    private string _validatedUrl;

    public int BlogId { get; set; }

    [BackingField(nameof(_validatedUrl))]
    public string Url
    {
        get { return _validatedUrl; }
    }

    public void SetUrl(string url)
    {
        // put your validation code here

        _validatedUrl = url;
    }
}
```

### Fluent API

```csharp
class MyContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Blog>()
            .Property(b => b.Url)
            .HasField("_validatedUrl");
    }
}

public class Blog
{
    private string _validatedUrl;

    public int BlogId { get; set; }

    public string Url
    {
        get { return _validatedUrl; }
    }

    public void SetUrl(string url)
    {
        using (var client = new HttpClient())
        {
            var response = client.GetAsync(url).Result;
            response.EnsureSuccessStatusCode();
        }

        _validatedUrl = url;
    }
}
```

## Field and property access

By default, EF will always read and write to the backing field. It will assume that one has been properly configured and will never use the property. However, EF also supports other access patterns. For example, the following sample instructs EF to write to the backing field only while materializing and to use the property in all other cases.

```csharp
class MyContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Blog>()
            .Property(b => b.Url)
            .HasField("_validatedUrl")
            .UsePropertyAccessMode(PropertyAccessMode.PreferFieldDuringConstruction);
    }
}

public class Blog
{
    private string _validatedUrl;

    public int BlogId { get; set; }

    public string Url
    {
        get { return _validatedUrl; }
    }

    public void SetUrl(string url)
    {
        using (var client = new HttpClient())
        {
            var response = client.GetAsync(url).Result;
            response.EnsureSuccessStatusCode();
        }

        _validatedUrl = url;
    }
}
```

## Field-only properties

You can also create a conceptual property in your model that does not have a corresponding CLR property in the entity class but instead uses a field to store the data in the entity. This is different from Shadow Properties, where the data is stored in the change tracker, rather than in the entity's CLR type. Field-only properties are commonly used when the entity class uses methods instead of properties to get/set values, or in cases where fields shouldn't be exposed at all in the domain model \(e.g. primary keys\).

You can configure a field-only property by providing a name in the `Property(...)` API.

```csharp
class MyContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Blog>()
            .Property("_validatedUrl");
    }
}

public class Blog
{
    private string _validatedUrl;

    public int BlogId { get; set; }

    public string GetUrl()
    {
        return _validatedUrl;
    }

    public void SetUrl(string url)
    {
        using (var client = new HttpClient())
        {
            var response = client.GetAsync(url).Result;
            response.EnsureSuccessStatusCode();
        }

        _validatedUrl = url;
    }
}
```

EF will attempt to find a CLR property with the given name, or a field if a property isn't found. If neither a property nor a field is found, a shadow property will be set up instead.

You may need to refer to a field-only property from LINQ queries, but such fields are typically private. You can use `EF.Property(...)` method in a LINQ query to refer to the field.

```csharp
var blogs = db.blogs.OrderBy(b => EF.Property<string>(b, "_validatedUrl"));
```

