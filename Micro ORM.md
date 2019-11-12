## Micro ORM Overview
Use the extension methods provided for IDbConnection interface

#### Insert
````C#
// using anonymous class
var affectedRowsCount = connection.Execute("INSERT INTO Company (CompanyId, Title, Email) Values (@CompanyId, @Title, @Email);", new { CompanyId = companyId, Title = "New Company", Email = "email@company.com" });

// using dictionary object
var affectedRowsCount = connection.Execute("INSERT INTO Company (CompanyId, Title, Email) Values (@CompanyId, @Title, @Email);", new Dictionary<string, object>() { { "CompanyId", companyId }, { "Title", "New Company" }, { "Email", "email@company.com" } });

// using DbxParameter
var affectedRowsCount = connection.Execute("INSERT INTO Company (CompanyId, Title, Email) Values (@CompanyId, @Title, @Email);", new DbxParameters() { new DbxParameter("CompanyId", companyId), new DbxParameter("Title", "New Company"), new DbxParameter("Email", "email@company.com") });

// using DbxParameter with DbType
var affectedRowsCount = connection.Execute("INSERT INTO Company (CompanyId, Title, Email) Values (@CompanyId, @Title, @Email);", new DbxParameters() { new DbxParameter("CompanyId", companyId, DbType.Guid), new DbxParameter("Title", "New Company", DbType.String), new DbxParameter("Email", "email@company.com", DbType.String) });
````
#### Update
````C#
connection.Execute("UPDATE Company SET Email = @Email WHERE CompanyId = @CompanyId);", new { CompanyId = companyId, Email = "newemail@company.com" });
````
#### Delete
````C#
connection.Execute("DELETE FROM Company WHERE CompanyId = @CompanyId);", new { CompanyId = companyId });
````
#### Query Data
````C#
// as entity
var companiesList = connection.Query<Company>("SELECT * FROM Company ORDER BY Title");
var company = connection.Query<Company>("SELECT * FROM Company WHERE CompanyId = @CompanyId", new { CompanyId = companyId }).FirstOrDefault();

// as datatable
var dataTable = connection.QueryToDataTable("SELECT * FROM Company ORDER BY Title");
````
#### Stored Procedures
````C#
// execute
connection.ExecuteStoredProcedure("InsertCompany", new DbxParameters() { new DbxParameter("CompanyId", companyId), new DbxParameter("Title", "New Company"), new DbxParameter("Email", "email@companyltd.com") });

// execute scalar
var objValue = connection.ExecuteScalarStoredProcedure("GetNextInvoiceCode");

// execute with OUT parameter
var param = new DbxParameters() 
{
    new DbxParameter("CompanyId", companyId), 
    new DbxParameter("StatusMsg", "", DbType.String) { Direction = ParameterDirection.InputOutput } 
};
connection.ExecuteStoredProcedure("RegisterCompanyById", param);

// now get OUT parameter value for further processing
var email = param[1].Value;

````
