# Many-to-many Relationship

Earlier in Entity Framework, the many-to-many relationship was classified as two one-to-many relationships. To make it work the developer must create a joining entity class.

Now in Entity Framework Core 5.0, it will have full support for many-to-many relations without explicitly mapping the join table. 

* The navigation properties skip the join table and directly point to the other entity. 
* It will result in writing cleaner queries and simplify the use of the query result.

Let's consider the following model.

```csharp
public class Movie
{
    public int MovieId { get; set; }
    public string Name{ get; set; }
    public Actor Actor { get; set; }
    public List<Genre> Genres { get; set; }
}

public class Genre
{
    public int GenreId { get; set; }
    public string GenreName { get; set; }
    public List<Movie> Movies{ get; set; }
}
```

As you can see that `Movie` class contains a collection of `Genres`, and `Genre` class contains a collection of `Movies`. EF Core 5.0 recognizes this as a many-to-many relationship by convention and there is no need for configuration in `OnModelCreating`.

```csharp
public class MyEntityContext : DbContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder opBuilder)
    {
        opBuilder.UseSqlServer("Data Source=(localdb)\\ProjectsV13;Initial Catalog=MyContextDB;");
    }

    public DbSet<Movie> Movies { get; set; }
    public DbSet<Genre> Genres { get; set; }

}

```

When you create migration or call the `EnsureCreated`method, it will create the following tables including the join table. 

```sql
CREATE TABLE [dbo].[Movies] (
    [MovieId] INT            IDENTITY (1, 1) NOT NULL,
    [Name]    NVARCHAR (MAX) NULL,
    CONSTRAINT [PK_Movies] PRIMARY KEY CLUSTERED ([MovieId] ASC)
);

CREATE TABLE [dbo].[Genres] (
    [GenreId]   INT            IDENTITY (1, 1) NOT NULL,
    [GenreName] NVARCHAR (MAX) NULL,
    CONSTRAINT [PK_Genres] PRIMARY KEY CLUSTERED ([GenreId] ASC)
);

CREATE TABLE [dbo].[GenreMovie] (
    [GenresGenreId] INT NOT NULL,
    [MoviesMovieId] INT NOT NULL,
    CONSTRAINT [PK_GenreMovie] PRIMARY KEY CLUSTERED ([GenresGenreId] ASC, [MoviesMovieId] ASC),
    CONSTRAINT [FK_GenreMovie_Genres_GenresGenreId] FOREIGN KEY ([GenresGenreId]) REFERENCES [dbo].[Genres] ([GenreId]) ON DELETE CASCADE,
    CONSTRAINT [FK_GenreMovie_Movies_MoviesMovieId] FOREIGN KEY ([MoviesMovieId]) REFERENCES [dbo].[Movies] ([MovieId]) ON DELETE CASCADE
);
```

Let's insert some movies and genres.

```sql
using (var context = new MyEntityContext())
{
    context.Database.EnsureCreated();

    var comedy = new Genre() { GenreName = "Comedy" };
    var action = new Genre() { GenreName = "Action" };
    var horror = new Genre() { GenreName = "Horror" };
    var scifi = new Genre() { GenreName = "Sci-fi" };

    context.AddRange(
        new Movie() { Name = "Avengers", Genres = new List<Genre>() { action, scifi } },
        new Movie() { Name = "Satanic Panic", Genres = new List<Genre>() { comedy, horror } });

    context.SaveChanges();

}
```

EF will then automatically create rows in the join table. 

![](../.gitbook/assets/image.png)

