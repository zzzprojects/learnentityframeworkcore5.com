<a href="https://entityframework-extensions.net/">**Improve EF Core performance with EF Extensions**</a>

<a href="https://entityframework-extensions.net/">
<img src="https://zzzprojects.github.io/images/logo/entityframework-extensions-pub.jpg" width="600" />
</a>

# Migrations in EF-Core

In Entity Framework Core, when there is a change in the model, the database tables are also needed to be updated to keep everything in sync for the proper working of the application.

* To update or generate the change in the ongoing model, the **Migration** method is used, it allows the developer to update the data without losing it.
* Also, the migration method can be used to update and generate change in the Database tables, based on the Model.
* Migrations make it easy for the developer to change and update the data of the application, otherwise, all the codes had to be re-written manually.

In Entity Framework Core, migration method can help in multiple numbers of tasks, such as:

* Creating Migration.
* Update or create Database
* Modify current migration code.
* Remove Migration from Model.
* Revert the effect of Migration.
* Create SQL Scripts.
* Apply Migration at Runtime.

All these tasks can be applied with the help of the Migration method.

## Creating a Migration

To add migration to the model, the developers can use the `Add-Migration` command.

**Command Line - CLI**

`dotnet ef migrations add <name of migration>`

**Package Manager console**

`add-migration <name of migration>`

When you create a migration, the framework checks if there is any change from the previous migration if one exists and generates a file containing a class inheriting from `Microsoft.EntityFrameworkCore.Migrations.Migration` featuring an Up and a Down method.

If it finds any change in the model then EF Core updates the model as well as the Database according to the current changes that are required for running the application.

## Update Database

After updating the model, migration does not automatically update the database, for updating the database, we need to use an additional command.

To update the database in order to keep it in sync with the current model, `Update-Database` command is used to bring the database in sync with the current migration updates.

**Command Line - CLI**

`dotnet ef database update`

**Package Manager console**

`Update-Database`

## Modify Current Migration code

The Entity Framework Core allows the developer to modify the migration code in order to make everything work in sync with each other.

This feature comes into play every time the developer needs to make changes to the model and the database tables and make everything up to date.

To modify the migration code simply add or modify the code with the required changes for the model.

**Command Line - CLI**

`dotnet ef migrations add AddActor`

**Package Manager console**

`Add-Migration AddActor`

Rewrite the database schema in the following way:

```csharp
migrationBuilder.AddColumn<string>(
    name: "FullName",
    table: "Actor",
    nullable: true);
 
migrationBuilder.Sql(
    @"UPDATE Actor SET FullName = FName + ' ' + LName;");

migrationBuilder.DropColumn(
    name: "FName",
    table: "Actor");
 
migrationBuilder.DropColumn(
    name: "LName",
    table: "Actor");
```

## Removing Migration from EF-Core Model

It is easy to remove migration from your Entity Framework model, sometimes adding a migration can cause issues in the application.

In that case, removing that migration helps a lot, the developer can easily remove that migration and add the relevant migration later when required.

To simply remove the last migration, the `Remove-Migration` command can be used.

**Command Line - CLI**

`dotnet ef migrations remove`

**Package Manager console**

`Remove-Migration`

## Revert Migration Effect

After applying the migration to the database, if the developer wants to revert the effect of the changes made, then they can easily revert the model or the database to its previous form by passing the name of a target migration to the update command.

**Command Line - CLI**

`dotnet ef database update LastGoodMigration`

**Package Manager console**

`Update-Database LastGoodMigration`

## Create SQL Scripts

Sometimes if the migration process is not working for your model due to some fault or incompatibility from the database provider, then creating custom SQL scripts can help in resolving the issue, the good thing is that the migration process in EF-Core allows the developers to create custom SQL scripts to be embedded in the Model.

**Command Line - CLI**

`dotnet ef migrations script`

**Package Manager console**

`Script-Migration`

## Apply Migration at Runtime

Different applications prefer different modes in EF-Core, some prefer start-up migration and some prefer runtime migration, this is based on the type of application.

To apply migration during runtime, developers can simply use the `Migrate()` method and achieve the migration process during Runtime.

```csharp
EFCoreDbContext.Database.Migrate();
```

## Additional Features of Migration in EF – Core

### Targeting Multiple Database Providers Simultaneously

Entity Framework Core allows the migrations to be database friendly, it allows migration settings from one Model to be embedded in other similar types of models.

But there can be situations where a migration which is working on some model might not work on another, in this situation, the developer can set these types of migrations as ignored.

### Custom Migration Timeout

In some cases, the migration might not work due to application taking more time than the default setting for which the migration was configured.

1. The developer can set customized active time for the migrations so that they can properly sync with the application.
2. To set the migration timeout, the developer must set the required timeout through the `DbContext` ****Level, but it comes with a trade-off.
3. If you set timeout using `DbContext`, then this setting will be applied to all the operations that are coming under the `DbContext` Level.

## Summary

This completes everything about the migrations in Microsoft Entity Framework Core, over-all migration is a very useful feature and it helps lots of developers while using EF Core.

