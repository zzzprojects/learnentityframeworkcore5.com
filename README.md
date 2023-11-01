<a href="https://entityframework-extensions.net/">**Improve EF Core performance with EF Extensions**</a>

<a href="https://entityframework-extensions.net/">
<img src="https://zzzprojects.github.io/images/logo/entityframework-extensions-pub.jpg" width="600" />
</a>

# Introduction of Entity Framework Core

EF Core (Entity Framework Core) is an ORM (Object Relational Mapper) developed by Microsoft to simplify the database operations for .NET developers. Entity Framework Core is a lightweight, extensible, and open-source software like all the previous versions of the Entity Framework. It is a cross-platform software, making it easily to work on different operating systems like Windows, Mac OS, and Linux OS.

Some advantage of EF Core include:

* Entity Framework Core execute create, read, update, and delete operation automatically without the need to write all the SQL queries manually every time.
* Entity Framework Core works on the principle of ORM, which allows it to drop the need for writing all the access code that the developers usually had to spend so much time writing.

## Entity Framework Core Approaches

EF Core supports two development approaches:

Code First
Database First

### Code First Approach

The code-first approach allows you to create your model through [Data Annotations](https://www.learnentityframeworkcore.com/configuration/data-annotation-attributes) and [Fluent API](https://www.learnentityframeworkcore.com/configuration/fluent-api)

It also allow you to specify some [Migrations](https://www.learnentityframeworkcore.com/migrations) and execute command like [Add-Migration](https://www.learnentityframeworkcore.com/migrations/add-migration)

### Database First Approach

For the database first approach, EF Core creates the required classes by using the available database or database tables, it creates these classes using the EF Core commands. 

But, the drawback of the database first approach is that this method can be applied to only limited numbers of classes as the EF Core does not support visual designer or wizard.

## Features

If you are familiar with Entity Framework 6, the EF Core consists of all the features from EF 6, Some of the basic features that are included in the Entity Framework Core are:

* [DbContext](/dbcontext)
* [DbSet](/dbset)
* Data Model
* Querying using Linq-to-Entities
* [Change Tracking](https://www.learnentityframeworkcore.com/dbcontext/change-tracker)
* SaveChanges
* [Migrations](https://www.learnentityframeworkcore.com/migrations)
* Easy relationship configuration
* In-memory provider for testing
* Support for IoC \(Inversion of Control\)
* Unique constraints
* Shadow properties
* Alternate keys
* Global query filter
* Field mapping
* DbContext pooling
* Better patterns for handling disconnected entity graphs

Besides these features, Entity Framework Core has been recently updated to have the support for the following features as well.

## LINQ Overhaul

LINQ allows developers to write an unlimited number of different .NET queries of their choices. It helps in having rich type information that offers IntelliSense and compile-time type checking, but the real challenge is to handle these combinations for LINQ providers.

* The newer EF Core 8 allows LINQ providers to translate more numbers of queries into SQL, giving the user more efficient queries in SQL and letting in-efficient queries remain undetected. 
* It also allows developers to create a single SQL statement per LINQ query. Although it can have further improvements which will bring more performance upgrades in the future.

## Cosmos Database Support

For developers who are familiar with Entity Framework, Cosmos DB support enables them to target the Azure Cosmos DB as an Application database.

This allows the .NET developers to have access to features like global distribution, always-on availability, elastic scalability, and low latency.

## Lazy Loading

Lazy loading allows the EF Core to take the required data without writing extra queries for the same.

* To apply this, it uses the proxies but since proxy logic isn’t the core feature of the EF Core, it keeps the data in its own package of the project.
* With lazy loading enabled on the context, any virtual-navigation property is overridden under the covers by the proxy at run time, the developer has to manually change the declaration to virtual.

## Reverse Engineering of Database Views

Those query type which represents data are readable from the database, but they cannot be changed or updated, these queries have been renamed to key-less entity types.

EF Core now automatically creates key-less entity types when reverse engineering of database views.

Entity Framework Core is continuously getting new and better firmware updates which are adding more useful features for developers, some of the upcoming features for the Future are:

* Ability to ignore parts of a model in migrations.
* Property bag entities tracked as two separate issues. 

## Example of The Entity Framework Core

In a typical situation to read, write, update, and delete from the database table, the developers must write different code to generate the SQL operations.

When the data is read from the database, in order to map the data to the relevant classes, the developers must generate another custom code to map the data to their respective classes.

All these actions must be performed for each individual project making it complicated and time-consuming for the developers.

With the help of an Object Relation Mapper like the EF Core, all these tasks can be done automatically making it simple and time saver for all the .NET Developers.

EF Core sits between application code and database and it eliminates the need for the custom data access code that usually had to be written when proper ORM solutions were absent.

**Example:** In case we have to develop an application to manage the employees in a company, we will have different classes like employees, departments, positions, etc. These classes will be called as domain classes.

To arrange all these classes in a systematic and structured order, any developer would have to invest a lot of time and effort to create suitable codes for the following situation.

In this case, an ORM like the EF Core comes really handy as it does all the work automatically, resulting in saving lots of resources.

## Conclusion

Entity Framework Core is very useful yet agile software which sits between different category of classes and the databases or database tables to automatically configure the SQL statements needed to restructure and manage the given databases.

It simplifies the work of the developers by developing the data on its own, which allows the user to do their work more efficiently and effectively.

Since its a cross-platform software, any .NET developer can have the benefits of the Entity Framework Core.

With the on-going development happening to the Entity Framework Core, developers can expect useful and amazing features making their way to future firmware releases.

