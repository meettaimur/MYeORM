# MYeORM
A high performance ORM, for those who prefer SQL to achieve optimal performance.

#### As Micro ORM
* provides extension methods to IDbConnection, so can be used with any database provider
* for details check [micro orm page](https://github.com/meettaimur/MYeORM/blob/master/Micro%20ORM.md)
````C#
conection.Execute("INSERT INTO Company (CompanyId, Title, Email) Values (@CompanyId, @Title, @Email);", new { CompanyId = companyId, Title = "New Company", Email = "email@company.com" });
````

#### As Hybrid ORM
* built-in support for SQLServer, Oracle, MySQL, PostgreSQL and SQLite without external dependencies
* CRUD operations
* transactions but in a way better than traditional approach
* validation attributes
* sequential Guid generator
* DB Migrations supported without external depdencies
* DB Migrations management simplified (a different approach than Entity Framework)
* Data Listing support tables/views, filtering, sorting and paging
* Can be used (not tested yet) with other protocol compatible databases like MariaDB, Percona Server, Amazon Aurora, Azure Database for MySQL, Google Cloud SQL for MySQL, YugaByte, TimescaleDB, CockroachDB etc.


## Installation
get [MYeORM nuget package](https://www.nuget.org/packages/MYeORM/)

## Overview

#### Register Connection
````c#
using MYeORM;
using MYeORM.Client;
using MYeORM.Client.OrmAttributes;
using MYeORM.ViewsQuery;
````
````c#
// to register MySQL connection
string dbId = DB.RegisterConnectionType<MySql.Data.MySqlClient.MySqlConnection>("connection string");

// to register SQL Server connection
string dbId = DB.RegisterConnectionType<System.Data.SqlClient.SqlConnection>("connection string");

// to register Oracle connection
string dbId = DB.RegisterConnectionType<Oracle.ManagedDataAccess.Client.OracleConnection>("connection string");

// to register PostgreSQL connection
string dbId = DB.RegisterConnectionType<Npgsql.NpgsqlConnection>("connection string");

// to register SQLite connection (Microsoft.Data.SQlite)
string dbId = DB.RegisterConnectionType<Microsoft.Data.Sqlite.SqliteConnection>("Data Source=ormsample.db;");

// to register SQLite connection (System.Data.SQLite)
string dbId = DB.RegisterConnectionType<System.Data.SQLite.SQLiteConnection>("Data Source=ormsample.sqlitedb;Version=3;New=True;Compress=True;");

// create class
public class Company
{
    [Key]
    public Guid CompanyId { get; set; }

    [MaxLength(100), Required]
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
#### Basic Operations
````C#
// create db agent
var db = new OrmDbAgent(dbId);

// new company
var company = new Company() { CompanyId = db.NewGuid(), Title = "Microsoft", Phone = "", Email = "testemail@microsoft.com" };

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
#### Validation
````C#
Dictionary<string, string> errors = db.ValidateEntity(company);

// where
//      Key = property name or caption
//      Value = error message (default or custom)
````
###### For details check [validations page](https://github.com/meettaimur/MYeORM/blob/master/Validations.md)
#### Transactions
````C#
var transactionId = db.NewGuid().ToString();

db.Insert(company, transactionId);
...excute more operations

db.CommitTransaction(transactionId);
    // OR
db.RollbackTransaction(transactionId);
````
#### Threadsafe Operations
###### Use DB class for threadsafe operations directly instead OrmDbAgent.
````C#
DB.Insert(company, dbId);

company = DB.GetById<Company>(company.CompanyId, dbId);

DB.Update(company, dbId);
DB.Delete(company, dbId);
DB.Save(company, dbId);

// HINT: OrmDbAgent is also using DB class internally
````
#### Extend OrmDbAgent
Inherit and extend the agent class to meet your custom requirements
````C#
public class LoggedUser : OrmDbAgent
{
    public LoggedUser(string dbId) : base(dbId) { }

    public Guid UserId { get; set; }
    public string UserName { get; set; }
    
    // add more properties or methods as per your requirement
};
````
#### Query Data
##### queries
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
##### parameterized queries
````C#
// using anonymous class
companies = db.Query<Company>("SELECT * FROM Company WHERE CompanyId = @CompanyId ORDER BY Title", new { CompanyId = company.CompanyId });

// using dictionary
companies = db.Query<Company>("SELECT * FROM Company WHERE CompanyId = @CompanyId ORDER BY Title", new Dictionary<string, object>() { { "CompanyId", company.CompanyId } });

// using DbxParameter class
companies = db.Query<Company>("SELECT * FROM Company WHERE CompanyId = @CompanyId ORDER BY Title", new DbxParameter("CompanyId", company.CompanyId));

// standard cross database query
companies = db.Query<Company>($"SELECT * FROM Company WHERE CompanyId = {db.ParamChar}CompanyId ORDER BY Title", new { CompanyId = company.CompanyId });
````
##### cross database parameterized queries with filtering and paging
````C#
var userInput = "micro";

// create filter for current database-type
var filter = db.CreateFilter(() => company.Title, ViewFilterComparisonOperator.Like, userInput, out var filterParam);
    // OR
filter = db.CreateFilter(typeof(string), "Title", ViewFilterComparisonOperator.Like, userInput, out filterParam);
    // OR
filter = db.CreateFilter(typeof(string), nameof(company.Title), ViewFilterComparisonOperator.Like, userInput, out filterParam);

// get from table
companies = db.GetAll<Company>(filter, filterParam, orderByClause: "Title, DateCreated DESC");

// get using custom query(table or view)
companies = db.Query<Company>($"SELECT * FROM Company WHERE {filter} ORDER BY Title, DateCreated DESC", filterParam);

// get items paged
int pageSize = 25;
var firstPage = db.Query<Company>(db.ToPagedQuery(0, pageSize, $"SELECT * FROM Company WHERE {filter} ORDER BY Title"), filterParam);
var secondPage = db.Query<Company>(db.ToPagedQuery(25, pageSize, $"SELECT * FROM Company WHERE {filter} ORDER BY Title"), filterParam);
````
#### Stored Procedures
````C#
// execute
companies = db.QueryStoredProcedure<Company>("GetAllCompanies");
company = db.QueryStoredProcedure<Company>("GetCompanyById", new { CompanyId = company.CompanyId }).FirstOrDefault();
company = db.QueryStoredProcedure<Company>("GetCompanyById", new DbxParameter("CompanyId", company.CompanyId)).FirstOrDefault();
company = db.QueryStoredProcedure<Company>("GetCompanyById", new DbxParameter("CompanyId", company.CompanyId) { Direction = ParameterDirection.Input }).FirstOrDefault();

// execute with Output parameter
var paramList = new DbxParameters()
{
    new DbxParameter("CompanyId", company.CompanyId) { Direction = ParameterDirection.Input },
    new DbxParameter("OutputParameterName", "") { Direction = ParameterDirection.Output }
};
db.ExecuteStoredProcedure("DeleteCompanyById", paramList);

// now get OUT parameter value
var outValue = paramList[1].Value;

// execute scalar
var value = db.ExecuteScalarStoredProcedure("GenerateInvoiceCode");
````
####
## DB Migrations
Built-in db migrations is supported for these major databases only SQL Server, MySQL, Oracle, PostgreSQL.
#### Create class
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
}
````
#### Generate Schema Script
````C#
// generate script for mysql
var script =  DbMigrations.Generate_CreateTable_Script(typeof(Company), "", DbServerType.MySQL);
// OR
var script = DbMigrations.Generate_CreateTable_Script(new List<Type>() { typeof(Company) }, "", DbServerType.MySQL);
````
###### script generated
````SQL
CREATE TABLE IF NOT EXISTS Company (CompanyId CHAR(36) BINARY DEFAULT '' NOT NULL,Title nvarchar(100),Phone nvarchar(25),Email nvarchar(150) , primary key (CompanyId)) engine=InnoDb auto_increment=0;
````
#### Add Index to property
````C#
[MaxLength(150), EmailAddress, Index(isUnique:true)]
public string Email { get; set; }
````
###### script generated
````SQL
CREATE TABLE IF NOT EXISTS Company (CompanyId CHAR(36) BINARY DEFAULT '' NOT NULL,Title nvarchar(100),Phone nvarchar(25),Email nvarchar(150) , primary key (CompanyId)) engine=InnoDb auto_increment=0;
CREATE UNIQUE INDEX IX_Email ON Company(Email DESC) using HASH;
````
#### Add more Properties to class
````C#
[DbMigrationAdd, IgnoreForUpdate]
public DateTime? DateCreated { get; set; } = DateTime.Now;

[DbMigrationAdd, IgnoreForInsert]
public DateTime? DateModified { get; set; }
````
###### script generated for MySQL
````SQL
CREATE TABLE IF NOT EXISTS Company (CompanyId CHAR(36) BINARY DEFAULT '' NOT NULL,Title nvarchar(100),Phone nvarchar(25),Email nvarchar(150),DateCreated datetime,DateModified datetime , primary key (CompanyId)) engine=InnoDb auto_increment=0;
CREATE UNIQUE INDEX IX_Email ON Company(Email DESC) using HASH;
ALTER TABLE Company ADD COLUMN DateCreated datetime;
ALTER TABLE Company ADD COLUMN DateModified datetime;
````
###### script generated for SQL Server
````SQL
CREATE TABLE [dbo].[Company] ([CompanyId] uniqueidentifier NOT NULL,[Title] nvarchar(100),[Phone] nvarchar(25),[Email] nvarchar(150),[DateCreated] datetime,[DateModified] datetime , CONSTRAINT [PK_Company] PRIMARY KEY ([CompanyId]));
GO
CREATE UNIQUE INDEX [IX_Email] ON [dbo].[Company]([Email]);
GO
ALTER TABLE [dbo].[Company] ADD [DateCreated] datetime;
GO
ALTER TABLE [dbo].[Company] ADD [DateModified] datetime;
GO
````
###### script generated for Oracle
````SQL
CREATE TABLE Company (CompanyId raw(16) NOT NULL,Title nvarchar2(100) NULL,Phone nvarchar2(25) NULL,Email nvarchar2(150) NULL,DateCreated date NULL,DateModified date NULL , CONSTRAINT PK_Company PRIMARY KEY (CompanyId));
/
CREATE UNIQUE INDEX IX_Email ON Company(Email);
/
ALTER TABLE Company ADD DateCreated date NULL;
/
ALTER TABLE Company ADD DateModified date NULL;
/
````
###### script generated for PostgreSQL
````SQL
CREATE TABLE IF NOT EXISTS public.Company (CompanyId uuid NOT NULL DEFAULT '00000000-0000-0000-0000-000000000000',Title varchar(100),Phone varchar(25),Email varchar(150),DateCreated timestamp,DateModified timestamp , CONSTRAINT PK_Company PRIMARY KEY (CompanyId));
CREATE UNIQUE INDEX IF NOT EXISTS IX_Email ON public.Company(Email);
ALTER TABLE public.Company ADD COLUMN IF NOT EXISTS DateCreated timestamp;
ALTER TABLE public.Company ADD COLUMN IF NOT EXISTS DateModified timestamp;
````
#### for details check [db migrations page](https://github.com/meettaimur/MYeORM/blob/master/DB%20Migrations.md) 
####
## Data Listing
####
## Limitations

## Who is using
We are using in our own projects for years
