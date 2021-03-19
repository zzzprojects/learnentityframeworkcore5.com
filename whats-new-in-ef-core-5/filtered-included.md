<a href="https://entityframework-extensions.net/">**Improve EF Core performance with EF Extensions**</a>

<a href="https://entityframework-extensions.net/">
<img src="https://zzzprojects.github.io/images/logo/entityframework-extensions-pub.jpg" width="600" />
</a>

# Filtered Included

The Include method now supports filtering of the entities included. When applying Include to load related data, you can apply certain enumerable operations on the included collection navigation, which allows for filtering and sorting of the results.

The supported operations are `Where`, `OrderBy`, `OrderByDescending`, `ThenBy`, `ThenByDescending`, `Skip`, and `Take` which should be applied on the collection navigation in the lambda passed to the `Include` method, as shown in the below example. 

```csharp
var blogs = context.Blogs
    .Include(e => e.Posts.Where(p => p.Title.Contains("Cheese")))
    .ToList();
```

The above query will return blogs together with each associated post, but only when the post title contains "Cheese".

You can also use `Skip` and `Take` methods to reduce the number of included entities.

```csharp
var blogs = context.Blogs
    .Include(e => e.Posts.OrderByDescending(post => post.Title).Take(5)))
    .ToList();
```

This query will return blogs with at most five posts included on each blog.



