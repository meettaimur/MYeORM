# MYeORM
A high performance ORM, for those who prefer SQL for optimal performance.

###### As Micro ORM
* provides extension methods to IDbConnection, so can be used with any database provider
* use it like any other micro orm like Dapper
* provides validation attributes like Required, EmailAddress, MaxLength, MinLength, ValueRange, Required(groupId)

###### As Hybrid ORM
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

###### Register Connection
````c#
using MYeORM;
using MYeORM.Client;
using MYeORM.Client.OrmAttributes;

string conString = "Server=192.168.75.150;Port=3306;Database=OrmSampleDb;User Id=userid;Password=pass;SslMode=None;";
string dbId = OrmDbAgent.RegisterConnectionType<MySql.Data.MySqlClient.MySqlConnection>(conString);
````
###### CRUD
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
##### Transactions
````C#
var transactionId = db.NewGuid().ToString();

db.Insert(company, transactionId);
...do other operations

db.CommitTransaction(transactionId);
    // OR
db.RollbackTransaction(transactionId);
````
###### Threadsafe CRUD
````C#
DB.Insert(company, dbId);

company = DB.GetById<Company>(company.CompanyId, dbId);

DB.Update(company, dbId);
DB.Delete(company, dbId);
DB.Save(company, dbId);
````
###### Company class
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
##### Validation
````C#
Dictionary<string, string> errors = db.ValidateEntity(company);
````
where 
Key = property name or caption and 
Value = error message (default or custom)
````C#
// set validation Attributes to properties
public class Company
{
    [Key]
    public Guid CompanyId { get; set; }

    [MaxLength(100), Required(errorMessage: "Title is required")]
    public string Title { get; set; }

    [MaxLength(25), MinLength(7)]
    public string Phone { get; set; }

    [MaxLength(125), EmailAddress(errorMessage: "invalid email address")]
    public string Email { get; set; }

    [Required(groupId: 1, errorMessage: "email2 or email3, atleast one of them is required"), EmailAddress]
    public string Email2 { get; set; }

    [Required(groupId: 1), EmailAddress]
    public string Email3 { get; set; }

    [ValueRange(1, 10, errorMessage: "given value is out of range")]
    public int? IntegerColumn { get; set; }

    [ValueRange(1.5, 10.5, errorMessage: "given value is out of range")]
    public double? DoubleColumn { get; set; }
}
````

## Data Listing

## DB Migrations

## Limitations

## Who is using
We are using this in our own projects for years
