# MYeORM
A high performance ORM, for those who prefer SQL to achieve optimal performance.

##### As Micro ORM
* provides extension methods to IDbConnection, so can be used with any database provider
* use it like any other micro orm like Dapper
* provides validation attributes like Required, EmailAddress, MaxLength, MinLength, ValueRange, Required(groupId)

##### As Hybrid ORM
* has built-in support for SQLServer, Oracle, MySQL, PostgreSQL and SQLite without external dependency
* support CRUD operations
* support transactions but in a way better than traditional approach
* provides validation attributes
* built-in Sequential Guid generator
* DB Migrations supported without external depdency
* DB Migrations management simplified (a different approach than Entity Framework)
* Data Listing support tables/views, filtering, sorting and paging
* Can be used but not yet tested with MariaDB, Percona Server, Amazon Aurora, Azure Database for MySQL, Google Cloud SQL for MySQL, YugaByte, TimescaleDB, CockroachDB or with other protocol compatible databases


## Installation
install from [MYeORM nuget package](https://www.nuget.org/packages/MYeORM/)

## Quick Overview

##### Register Connection
````c#
using MYeORM;
using MYeORM.Client;
using MYeORM.Client.OrmAttributes;
using MYeORM.ViewsQuery;

string conString = "Server=192.168.75.150;Port=3306;Database=OrmSampleDb;User Id=userid;Password=pass;SslMode=None;";
string dbId = OrmDbAgent.RegisterConnectionType<MySql.Data.MySqlClient.MySqlConnection>(conString);
````
##### CRUD Operations
````C#
// create db agent, hint: OrmDbAgent
var db = new OrmDbAgent(dbId);

// new company
var company = new Company() { CompanyId = db.NewGuid(), Title = "Microsoft", Phone = "", Email = "example@microsoft.com" };

// insert
db.Insert(company);

// find
company = db.GetById<Company>(company.CompanyId);

// update
company.Title = "Microsoft Corporation";
db.Update(company);

// delete
db.Delete(company);
    // OR
db.DeleteById<Company>(company.CompanyId);

// save(insert or update)
db.Save(company);
````
##### Validation
````C#
Dictionary<string, string> errors = db.ValidateEntity(company);

// where
//      Key = property name or caption
//      Value = error message (default or custom)

````
##### Transactions
````C#
var transactionId = db.NewGuid().ToString();

db.Insert(company, transactionId);
...do other operations

db.CommitTransaction(transactionId);
    // OR
db.RollbackTransaction(transactionId);
````
##### Threadsafe CRUD Operations
Use DB class for threadsafe operations directly instead OrmDbAgent.
````C#
DB.Insert(company, dbId);

company = DB.GetById<Company>(company.CompanyId, dbId);

DB.Update(company, dbId);
DB.Delete(company, dbId);
DB.Save(company, dbId);
````
##### Query Data
###### get items
````C#
// all
var companies = db.GetAll<Company>();

// ordered
companies = db.GetAll<Company>(orderByClause: "Title");
companies = db.GetAll<Company>(orderByClause: nameof(company.Title));
companies = db.GetAll<Company>(orderByClause: "DateCreated DESC");

// filtered and ordered
companies = db.GetAll<Company>("Title LIKE '%micro%'", orderByClause: "Title");

// to entity
companies = db.Query<Company>("SELECT * FROM Company");

// to DataTable
var table = db.QueryToDataTable("SELECT * FROM Company");

// query few column(s)
companies = db.Query<Company>("SELECT CompanyId FROM Company");
companies = db.Query<Company>("SELECT CompanyId, Title FROM Company ORDER BY Title");

// query few column(s) to another class
var comboList = db.Query<CompanyComboItem>("SELECT CompanyId, Title FROM Company ORDER BY Title");

````
###### cross rdbms parameterized filtering and paging
````C#
var userInput = "micro";

// create filter for current database-type
var filter = db.CreateFilter(() => company.Title, ViewFilterComparisonOperator.Like, userInput, out var filterParam);
    // OR
filter = db.CreateFilter(typeof(string), "Title", ViewFilterComparisonOperator.Like, userInput, out filterParam);
    // OR
filter = db.CreateFilter(typeof(string), nameof(company.Title), ViewFilterComparisonOperator.Like, userInput, out filterParam);

// get items from table
companies = db.GetAll<Company>(filter, filterParam, orderByClause: "Title, DateCreated DESC");

// get items using custom query
companies = db.Query<Company>($"SELECT * FROM Company WHERE {filter} ORDER BY Title, DateCreated DESC", filterParam);

// get items paged
int pageSize = 25;
var firstPage = db.Query<Company>(db.ToPagedQuery(0, pageSize, $"SELECT * FROM Company WHERE {filter} ORDER BY Title"), filterParam);
var secondPage = db.Query<Company>(db.ToPagedQuery(25, pageSize, $"SELECT * FROM Company WHERE {filter} ORDER BY Title"), filterParam);
````
###### queries with parameters
````C#
// anonymous class
companies = db.Query<Company>("SELECT * FROM Company WHERE CompanyId = @CompanyId ORDER BY Title", new { CompanyId = company.CompanyId });

// dictionary
companies = db.Query<Company>("SELECT * FROM Company WHERE CompanyId = @CompanyId ORDER BY Title", new Dictionary<string, object>() { { "CompanyId", company.CompanyId } });

// DbxParameter class
companies = db.Query<Company>("SELECT * FROM Company WHERE CompanyId = @CompanyId ORDER BY Title", new DbxParameter("CompanyId", company.CompanyId));

// standard cross database query
companies = db.Query<Company>($"SELECT * FROM Company WHERE CompanyId = {db.ParamChar}CompanyId ORDER BY Title", new { CompanyId = company.CompanyId });
````
##### Company class
````C#
public class Company
{
    [Key]
    public Guid CompanyId { get; set; }

    [MaxLength(100)]
    public string Title { get; set; }

    [MaxLength(25)]
    public string Phone { get; set; }

    [MaxLength(150), EmailAddress]
    public string Email { get; set; }

    [IgnoreForUpdate]
    public DateTime? DateCreated { get; set; } = DateTime.Now;

    [IgnoreForInsert]
    public DateTime? DateModified { get; set; }
}
````
## Data Listing

## DB Migrations

## Limitations

## Who is using
We are using this in our own projects for years
