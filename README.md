# MYeORM
A high performance ORM, for those who prefer SQL to achieve optimal performance.

#### As Micro ORM
* provides extension methods to IDbConnection, so can be used with any database provider
* for details please check [micro orm page](https://github.com/meettaimur/MYeORM/blob/master/Micro%20ORM.md)
````C#
conection.Execute("INSERT INTO Company (CompanyId, Title, Email) Values (@CompanyId, @Title, @Email);", new { CompanyId = companyId, Title = "New Company", Email = "email@company.com" });
````

#### As Hybrid ORM
* built-in support for SQLServer, Oracle, MySQL, PostgreSQL and SQLite without external dependencies
* CRUD operations
* Transactions got better and simple
* Sequential Guid generator
* DB Migrations management simplified (a different approach than Entity Framework)
* DB Migrations supported without external dependencies
* DB Migrations support Provider Data Types
* Data Listing support tables/views, paging, sorting and filtering
* Transfer Data between databases
* Can be used (but not tested yet) with other protocol compatible databases like, Azure SQL Database, MariaDB, Percona Server, Amazon Aurora, Azure Database for MySQL, Google Cloud SQL for MySQL, YugaByte, TimescaleDB, CockroachDB etc.


## Installation
get from [MYeORM nuget package](https://www.nuget.org/packages/MYeORM/)

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

// HINT:
//      use Column Attribute - if table column name is different than property name
//      use IgnoreForUpdate Attribute - to ignore property for update operation
//      use IgnoreForInsert Attribute - to ignore property for insert operation
//      use Ignore Attribute - to ignore property completely
````
#### Validation
````C#
Dictionary<string, string> errors = db.ValidateEntity(company);

// where
//      Key = property name or caption
//      Value = error message (default or custom)
````
###### For details please check [validations page](https://github.com/meettaimur/MYeORM/blob/master/Validations.md)
#####
#### Transactions
````C#
var transactionId = db.NewGuid().ToString();

// insert
db.Insert(company, transactionId);
...do more operations

// at end
db.CommitTransaction(transactionId);
    // OR
db.RollbackTransaction(transactionId);
````
#### Threadsafe Operations
###### Use DB class for threadsafe operations directly instead OrmDbAgent.
````C#
// insert
DB.Insert(company, dbId);

// find
company = DB.GetById<Company>(company.CompanyId, dbId);

// update
DB.Update(company, dbId);

// delete
DB.Delete(company, dbId);
db.DeleteById<Company>(company.CompanyId, dbId);

// save(insert or update)
DB.Save(company, dbId);



// Transactions
var transactionId = db.NewGuid().ToString();

DB.Insert(company, dbId, transactionId);
...do more operations

// at end
DB.CommitTransaction(transactionId);
    // OR
DB.RollbackTransaction(transactionId);

// HINT: OrmDbAgent is using DB class internally
````
#### Extend OrmDbAgent
###### Inherit and extend the agent class to meet your custom requirements
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
###### queries
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
###### query views
````C#
// create view in database e.g. 'CompaniesBySaleTopTen_view'
var companies = db.Query<Company>("SELECT * FROM CompaniesBySaleTopTen_view");

// OR   
// 1) create separate class for view
    [Table("CompaniesBySaleTopTen_view")]
    public class CompanyView
    {
        [Key]
        public Guid CompanyId { get; set; }
        public string Title { get; set; }
        public string Email { get; set; }
    }
    
// 2) now load data from view
var companiesTopTenBySale = db.GetAll<CompanyView>();
    // OR
var companiesTopTenBySale = db.Query<CompanyView>("SELECT * FROM CompaniesBySaleTopTen_view");

````
###### parameterized queries
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
###### cross database parameterized queries with filtering and paging
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

// execute scalar
var value = db.ExecuteScalarStoredProcedure("GenerateInvoiceCode");

// execute with Output parameter
var paramList = new DbxParameters()
{
    new DbxParameter("CompanyId", company.CompanyId) { Direction = ParameterDirection.Input },
    new DbxParameter("OutputParameterName", "") { Direction = ParameterDirection.Output }
};
db.ExecuteStoredProcedure("DeleteCompanyById", paramList);

// now get Output parameter value
var outValue = paramList[1].Value;
````
#### Data Transfer
###### transfer data between databases transparently
````C#
var dbSqlServer = new OrmDbAgent(dbIdSQLServer);
var dbOracle = new OrmDbAgent(dbIdOracle);

// get data from sqlserver
var companies = dbSqlServer.GetAll<Company>();

// transfer to oracle
var transId = Guid.NewGuid().ToString();
foreach(var company in companies)
{
    dbOracle.Insert(company, transId);
    // OR 
    // (insert or update)
    dbOracle.Save(company, transId);
}

// commit changes
dbOracle.CommitTransaction(transId);

````
## DB Migrations
Built-in db migrations supported for these major databases SQL Server, MySQL, Oracle, PostgreSQL.
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

// HINT: 
//      set DbMigrationAlter attribute - when property is changed e.g. MaxLength changed
//      set DbMigrationDrop attribute - when column is removed from table
//      set IgnoreDbSchema attribute - to ignore property for script generation
//                          while Ignore attribute is for client-side validation/manipulation only(not for schema generation)
//      set RequiredDbSchema attribute - to generate NOT NULL for property in script,
//                           while Required attribute is for client-side validation only(not for schema generation)
//      set NumberDbSchema(precision, scale) - to change precision and scale for decimal/double/float/numeric properties
//
//      create separate class for db-migration to manage columns removed from table like below
//              [Table("Company")]
//              public class CompanyDbMigration : Company 
//              { 
//                  [DbMigrationDrop]
//                  public string EmaiAddress { get; set; }
//              }
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
## DB Migrations for Provider Data Types
#### Add property to class
````C#
// property for postgres data type
public NpgsqlTypes.NpgsqlBox BoxProperty { get; set; }
````
#### Register data type
````C#
DbMigrations.Register_DbDataType(typeof(NpgsqlTypes.NpgsqlBox), "box", DbServerType.PostgreSQL);
````
#### Generate script
````C#
var script = DbMigrations.Generate_CreateTable_Script(typeof(YourClassName), "", DbServerType.PostgreSQL);
````
#### Primary Key and Index Script Generation
###### Guid primary key
````C#
[Key]
public Guid CompanyId { get; set; }
````
###### Identity primary key
````C#
[Key(autoIncrement:true)]
public long InvoiceNo { get; set; }
````
###### Guid primary key "non-clustered", while identity column "clustered" (note: generation depends on database type as well)
````C#
[Key, Index(isClustered: false, isUnique: true)]
public Guid InvoiceId { get; set; }

[Index(isClustered: true, isUnique: true, isAutoIncrement: true), Ignore]
public long InvoiceNo { get; set; }
    // OR
[Index(isClustered: true, isUnique: true, isAutoIncrement: true), Ignore]
public ulong InvoiceNo { get; set; }
````
###### Non-Unique index generation
````C#
[Index]
public string Email { get; set;}
````
###### Unique index generation
````C#
[Index(isUnique: true)]
public string Email { get; set;}
````
###### Composite unique index generation
````C#
[CompositeUniqueIndex(group:1, order:1)]
public int? Column1 { get; set; }

[CompositeUniqueIndex(group:1, order:2)]
public int? Column2 { get; set; }
````
##### for details please check [db migrations page](https://github.com/meettaimur/MYeORM/blob/master/DB%20Migrations.md) 
## Data Listing
#### Create View Class
````C#
[Table("Company"), DefaultOrderByClause("Title"), Caption("All Companies")]
public class CompanyViewAll
{
    [Key]
    public Guid CompanyId { get; set; }

    [Filterable]
    public string Title { get; set; }

    public string Email { get; set; }

    [Filterable]
    public DateTime? DateCreated { get; set; }
}
    
// HINT: use CustomViewQuery Attribute to
//      1) provide custom query, (note: complex queries not supported)
            [Table("Company"), DefaultOrderByClause("Title"), Caption("All Companies"), CustomViewQuery("SELECT CompanyId, Title, Email, DateCreated FROM Company")]
            public class CompanyViewAll { ... }

//      2) use view from database
            [Table("Company"), DefaultOrderByClause("Title"), Caption("All Companies"), CustomViewQuery("SELECT * FROM allcompanies_view")]
            public class CompanyViewAll { ... }
````
#### Register View
````C#
var view = db.RegisterView(typeof(CompanyViewAll));

var id = view.Id;
var title = view.Title; // e.g. "All Companies"
var entityType = view.EntityType;

// get all filterable fields
PickComboItems filterableFields = db.GetViewFilterableFields(view.Id);
// where
//      DisplayText = property name or caption
//      Key = property name or column name
````
#### Manipulate View
````C#
var page = db.ExecuteView(view.Id, ViewPageActionType.First, pageSize: 25);

page = db.ExecuteView(view.Id, ViewPageActionType.Next);
page = db.ExecuteView(view.Id, ViewPageActionType.Previous);
page = db.ExecuteView(view.Id, ViewPageActionType.Last);
page = db.ExecuteView(view.Id, ViewPageActionType.Refresh);
page = db.ExecuteView(view.Id, ViewPageActionType.All);

page = db.ExecuteView(view.Id, ViewPageActionType.First);

// page properties:
//      page.ViewId                         = view.Id
//      page.CurrenPage                     = 1;
//      page.FirstItemIndex                 = 1;
//      page.LastItemIndex                  = 25;
//      page.TotalItemsCount                = 300;
//      page.PageActionType                 = ViewPageActionType.First;
//      page.PageSize                       = 25;
//      page.Items                          = List<object>;
````
###### change sort column
````C#
db.SetViewSortColumn(view.Id, "Title", isAscending: true);
page = db.ExecuteView(view.Id, ViewPageActionType.First);
````
###### apply filter
````C#
var fieldName = "Title";            // OR: nameof(CompanyViewAll.Title);
var userInput = "micro";
var filterOperator = (int)ViewFilterComparisonOperator.Contains;

db.SetViewFilter(view.Id, fieldName, userInput, filterOperator);
page = db.ExecuteView(view.Id, ViewPageActionType.First);
````
###### clear filter
````C#
db.ClearViewFilter(view.Id);

// SetViewFilter(...) also auto clears the existing filter
````
###### apply date filter
````C#
fieldName = "DateCreated";
var userDateInput = DateTime.Now.Date;      // for a single day
filterOperator = (int)ViewFilterComparisonOperator.Equal;

db.SetViewFilter(view.Id, fieldName, userDateInput, filterOperator);
page = db.ExecuteView(view.Id, ViewPageActionType.First);

// apply date range filter
var userDateInput = DateTime.Now.Date.AddDays(-3);
var endDateInput = DateTime.Now.Date;
db.SetViewFilter(view.Id, fieldName, userDateInput, filterOperator, endDateInput);
page = db.ExecuteView(view.Id, ViewPageActionType.First);
````
###### apply filter across all filterable fields, excluding date/time fields
````C#
fieldName = "AllFields";
userInput = "micro";
filterOperator = (int)ViewFilterComparisonOperator.Contains;        // 'Contains' operator is supported only for "AllFields"

db.SetViewFilter(view.Id, fieldName, userInput, filterOperator);
page = db.ExecuteView(view.Id, ViewPageActionType.First);
````
###### filter operators supported for each data type
````C#
var operators = db.GetFilterOperatorsForString();
operators = db.GetFilterOperatorsForIntegerDecimalAndDateTime();
operators = db.GetFilterOperatorsForAllFields();

// where
//      DisplayText = caption       (changeable)
//      Key = enum ViewFilterComparisonOperator     e.g. ((int)ViewFilterComparisonOperator.Equal).ToString();
````
###### apply filter - user selected field and operator on UI
````C#
var userSelectedField = viewFilterableFields[1];                    // hint: db.GetViewFilterableFields(view.Id);
var userSelectedOperator = int.Parse(operators[0].Key);

fieldName = userSelectedField.Key;
filterOperator = userSelectedOperator;
userInput = "micro";

db.SetViewFilter(view.Id, fieldName, userInput, filterOperator);
page = db.ExecuteView(view.Id, ViewPageActionType.First);
````
#### Related Table View
###### to list contacts of a given company
````C#
[Table("Contact"), DefaultOrderByClause("FullName"), Caption("Company Contacts"), DefaultWhereClause("CompanyId = @CompanyId")]
public class CompanyContactsView
{
    [Key]
    public Guid ContactId { get; set; }
    public Guid CompanyId { get; set; }

    [Filterable]
    public string FullName { get; set; }
}

// register view
var relatedParameter = new ViewRelatedDynamicParameter()
{
    ParamName = "CompanyId",
    ParamValue = Guid.Empty
};
var relatedView = db.RegisterView(typeof(CompanyContactsView), relatedParameter);

// no record fetched as ParameValue = Guid.Empty
var page = db.ExecuteView(relatedView.Id, ViewPageActionType.First, pageSize: 25);

// now set company id
relatedParameter.ParamValue = selectedCompany.CompanyId;
page = db.ExecuteView(relatedView.Id, ViewPageActionType.First, rp: relatedParameter);
````
#### Child Table View
###### to list invoice-items of given invoice
````C#
[Table("InvoiceItem"), DefaultOrderByClause("SerialNo"), Caption("Invoice Items"), DefaultWhereClause("InvoiceId = @InvoiceId")]
public class InvoiceItemView
{
    [Key]
    public Guid InvoiceItemId { get; set; }
    public Guid InvoiceId { get; set; }

    public int? SerialNo { get; set; }
    
    [Filterable]
    public string Description { get; set; }
}

// register view
var childParameter = new ViewRelatedDynamicParameter()
{
    ParamName = "InvoiceId",
    ParamValue = Guid.Empty
};
var childView = db.RegisterView(typeof(InvoiceItemView), childParameter);

// no record fetched as ParameValue = Guid.Empty
var page = db.ExecuteView(childView.Id, ViewPageActionType.First, pageSize: 25);

// now set invoice id
childParameter.ParamValue = selectedInvoice.InvoiceId;
page = db.ExecuteView(childView.Id, ViewPageActionType.First, rp: childParameter);
````
## Who is using
We are using in our own projects for years
