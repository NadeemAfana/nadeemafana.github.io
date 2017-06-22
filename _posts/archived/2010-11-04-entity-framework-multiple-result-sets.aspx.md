---
layout: post
title: Entity Framework Advanced Tips - part 1
redirect_from:
- "/post/entity-framework-multiple-result-sets.aspx.html"
- "/post/entity-framework-multiple-result-sets.aspx/"
tags: [aspnetmvc, aspnet, mvc, ef]
---
**[Download Code](/attachments/posts/archived/session-less-controllers-and-tempdata-aspnet-mvc.zip)**

### Introduction
In this post, I am going to show you how to leverage multiple result sets, which can be used to boost performance. 

### Why performance matters

Performance is one of the most important things to consider when building any software application. For small applications, poor code may not be noticeable, but for large scale applications, poor code can be really a bottleneck. But we all know that performance comes at a cost, so it’s always good to balance between performance, security, time, and effort. Dealing with data efficiently can lead to a very scalable website, since websites rely heavily on databases, and a very large amount of data has to be retrieved and sent to the end data store frequently. The rule of thumb is to retrieve only what is needed. For example, if you need an address of some employee, you would only retrieve the address fields from the table employee. In plain ADO.NET datareader, this can be achieved easily. However, in Entity Framework, things get quite complicated since EF does not support partial entity filling yet. If the employee record’s total size is relatively small, then it does not hurt. But if the size is large, it can degrade performance. As you might know, using plain ADO.NET datareaders is the most flexible and efficient approach – provided it is well-implemented - , but it is the most tedious. So it does not matter to drop a little performance in order to reduce a large amount of effort and time needed to build the software. 

### Stored Procedures

Some software applications model is simple and plug in easily with Entity Framework, but others are complicated and cannot be plugged in easily. It is true that stored procedures are not normally necessary to retrieve and update information in a database. However, there are many things that can be achieved using stored procedures, but cannot be achieved using plain SQL. For example, in a library case, some client wants to rent a book, before doing the reserve operation, you will need to check if the book has already been reserved. This can be done by using some status code like output variables or return values in a stored procedure. Also, you may need to check the client’s status and make sure the number of books already reserved by him or her does not exceed the maximum allowed limit, or even if he or she has not returned any previously reserved book. There are many other cases too. Anyway, this type of conditional checking is crucial to this type of applications. A stored procedure can do all this in only one roundtrip. 

### Multiple Result Sets

An efficient way to retrieve data occurs in as few roundtrips as possible, and ideally in only one. EF 4 does not support multiple result sets. It relies on lazy loading or eager loading for related entities. The more related entities you eager-load, the more joins are made and the more superfluous columns are repeated due to the join. Instead of performing multiple joins as EF does, it would be more efficient to load every entity type in a separate result set. Lazy loading, which loads related entities on demand using extra roundtrips to the database, can be good or bad depending on what you need. I will blog about these two in the upcoming posts. For now, let’s see a working example. 

### Try It Out

In this example, I use the pubs database. You can download it from the attachment Download code. Let’s say we need to load three things together in only one roundtrip to the database:

1.    Top 10 titles (by sales count)
2.    All authors from California state
3.    Top 10 stores (by sales total)

Without multiple result sets, EF will have to make 3 separate roundtrips to the database in order to achieve the same result. You can watch this behavior using SQL Profiler. Now let’s create the stored procedure that will perform the queries: 

```sql
CREATE proc dbo.spGetTitleAuthorStore
AS
BEGIN
    -- SET NOCOUNT ON added to prevent extra result sets from
    -- interfering with SELECT statements.
    SET NOCOUNT ON;

   -- 1st set: Top 10 titles (by sales count)
   select top 10 * from titles order by ytd_sales desc
   
   -- 2nd set: All authors from California state.
   select * from authors where state='CA' 
   
   -- 3rd set: Top 10 stores (by sales total)
   select top 10 st.*, SUM(s.qty*t.price) as total from stores st inner join sales s on st.stor_id = s.stor_id
                                                     inner join titles t ON t.title_id=s.title_id 
                                                     group by st.stor_id, stor_name, stor_address, city, state ,zip
                                                     order by total desc 

   
END
GO
```

And here is a console application that executes this sp using EF 4:

```csharp
static void Main(string[] args)
        {
            using (var context = new pubsEntities())
            {

                var cs = @"Data Source=.;Initial Catalog=pubs;Integrated Security=True";
                var conn = new SqlConnection(cs);
                var cmd = conn.CreateCommand();

                

                cmd.CommandType = System.Data.CommandType.StoredProcedure;
                cmd.CommandText = "dbo.spGetTitleAuthorStore";

                cmd.Connection.Open();

                var reader = cmd.ExecuteReader(System.Data.CommandBehavior.CloseConnection);
                var titles = context.Translate<title>(reader).ToList();
                reader.NextResult();
                var authors = context.Translate<author>(reader).ToList();
                reader.NextResult();
                var stores = context.Translate<store>(reader).ToList();


                Console.WriteLine("1st set: Top 10 titles (by sales count):");
                Console.WriteLine(new string('=', 50)); // line separator

                int i = 1;
                foreach (var t in titles)
                    Console.WriteLine("{0}. {1}", i++, t.Title);

                Console.WriteLine(new string('-', 50)); // line separator


                Console.WriteLine("2nd set: All authors from California state):");
                Console.WriteLine(new string('=', 50)); // line separator
                i = 1;
                foreach (var a in authors)
                    Console.WriteLine("{0}. {1}", i++, a.au_fname + " " + a.au_lname);


                Console.WriteLine(new string('-', 50)); // line separator

                Console.WriteLine("3rd set: Top 10 stores (by sales total amount):");
                Console.WriteLine(new string('=', 50)); // line separator
                i = 1;
                foreach (var s in stores)
                    Console.WriteLine("{0}. {1} in {2},{3}", i++, s.stor_name, s.city, s.state);


            }

        }
```

![](/images/posts/archived/entity-framework-multiple-result-sets-1.png)

The [Translate](http://msdn.microsoft.com/en-us/library/dd466384.aspx) method translates a DbDataReader that contains rows of entity data to objects of the requested entity type. It is the nicest part that saves you a lot of work. Without it, you will need to create the objects yourself and then initialize them with the result data. Anyway, all the entities returned from this method are disconnected, not tracked by ObjectContext, so any changes to any of these entities won’t be persisted. In this example, we did not need that feature, so for performance reasons we didn't use it. If you want ObjectContext to track entities, there is another overload of the `Translate` method which takes 3 parameters: reader, entity set, and merge option. You can use this overload also to fix associations between your entities. For example, if you load all publishers and 10 titles, you can expect title.publisher to be populated automatically without any extra roundtrips to the database. It is something amazing, too. Note that the .translate method has its limitations. All columns returned from data reader must have matching properties with the same name, otherwise, the method fails. 

### When I should use Multiple Result Set

Use multiple result sets when you need to load data from database in one roundtrip. If the entities you would like to return are related together (e.g. authors and their titles), then EF might be able to load them in one roundtrip. So to figure it out, always use SQL Profiler to determine if EF is making more than one roundtrip to the database. 