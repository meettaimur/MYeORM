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

````c#
// register the connection once
string dbId = OrmDbAgent.RegisterConnectionType<MySql.Data.MySqlClient.MySqlConnection>("Server=192.168.75.150;Port=3306;Database=OrmSampleDb;User Id=root;Password=****;SslMode=None;");

// create db agent
var db = new OrmDbAgent(dbId);

var company = new Company() { CompanyId = db.NewGuid(), Title = "Microsoft", Phone = "", Email = "example@microsoft.com" };

// insert
db.Insert(company);

// find
company = db.GetById<Company>(company.CompanyId);

// update
company.Title = "Microsoft Corporation";
db.Update(company);
// or
db.DeleteById<Company>(company.CompanyId);

// delete
db.Delete(company);

// save(insert or update)
db.Save(company);


    public class Company
    {
        [Key]
        public Guid CompanyId { get; set; }

        [MaxLength(100)]
        public string Title { get; set; }

        [MaxLength(25)]
        public string Phone { get; set; }

        [MaxLength(125), EmailAddress]
        public string Email { get; set; }

        [IgnoreForUpdate]
        public DateTime? DateCreated { get; set; } = DateTime.Now;

        [IgnoreForInsert]
        public DateTime? DateModified { get; set; }
    }
````
##### Transactions
````C#
var transactionId = db.NewGuid().ToString();

db.Insert(company, transactionId);

db.CommitTransaction(transactionId);
    // OR
db.RollbackTransaction(transactionId);
````


## Data Listing

## DB Migrations

## Limitations

## Who is using
We are using this in our own projects for years
