---
layout: post
title: How to Return Random Rows Efficiently in SQL Server
redirect_from:
- "/post/random-rows-efficiently-in-sql-server.aspx.html"
- "/post/random-rows-efficiently-in-sql-server.aspx.aspx/"
tags: [sql, random, rows]
---

### Introduction

When building an application, sometimes you need some random rows from a database table whether it is for testing purpose or some other. There are different ways to select random rows from a table in SQL Server. For example, consider the following SQL statement which returns 20 random orders from the Northwind database

```sql
select  top(20) * from Orders order by newid()
```

Because the previous query scans the whole table, this solution is perfect for small tables. However, for large tables that contain hundred of thousands or even millions of rows, this query will be rather slow.

### TABLESAMPLE Clause

Fortunately, SQL Server 2005 and later include a feature you may have never heard of that fits for this purpose. This unknown feature is a clause called TABLESAMPLE that you specify after table name in a FROM clause. The TABLESAMPLE clause has the following syntax:

```sql
TABLESAMPLE [SYSTEM] (sample_number [ PERCENT | ROWS ] )
[ REPEATABLE (repeat_seed) ]
```
Here is an example that returns random rows from the Orders table using TABLESAMPLE:

```sql
Select * from Orders TABLESAMPLE(20 rows)
```

Note that you might not get exactly 20 rows. Also note that for small tables you probably won’t get any results at all. We will see why and how to overcome this problem.

SYSTEM specifies an ANSI SQL implementation-dependent sampling method. Specifying SYSTEM is optional, but this option is the only sampling method available in SQL Server and is applied by default.

You can use either ROWS or PERCENT to specify how many rows you want back in the results. SQL Server generates a random value for each physical page in that table. Based on that value, the page is either included or excluded. When a page is included, all rows in that page are included. For example, if you choose to select only 5 percent, then all rows from approximately 5 percent of the data pages are included in the result. When you choose the number of rows explicitly (use ROWS option) as in the previous example, this number is actually converted into a percentage of the total number of rows in that table. Because page size can vary, you might not get the exact number of rows you requested. Rather, you will get a result set size close to the number you requested.

To make it more likely you will get the exact number of rows you requested, you should specify a greater number of rows than what you need in the TABLESAMPLE clause and use TOP to limit the result to the actual number of rows you need. For example, if you need 500 rows:

```sql
Select top(500) * from Orders TABLESAMPLE(1000 rows)
```

You may still get a fewer number of rows, but you will never get more. The larger the number of rows you specify in the TABLESAMPLE clause, the more likelihood to get the exact number of rows you really want.

### Using the REPEATABLE Option

If you need to get repeatable results, use the REPEATABLE option. The REPEATABLE option causes a selected sample to be returned again. When REPEATABLE is specified with the same repeat_seed value, SQL Server returns the same subset of rows, as long as no changes have been made to the table. When REPEATABLE is specified with a different repeat_seed value, SQL Server will typically return a different sample of the rows in the table. For example, the following code returns the same set of results even if you run it multiple times:

```sql
select * from Orders TABLESAMPLE(30 rows) repeatable(55)
```

### When to use TABLESAMPLE

Use TABLESAMPLE on large tables and when the resulting rows do not have to be truly random at the level of individual rows. However, TABLESAMPLE cannot be applied to derived tables, tables from linked servers, and tables derived from table-valued functions, rowset functions, or OPENXML. TABLESAMPLE cannot be specified in the definition of a view or an inline table-valued function.