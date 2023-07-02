---
title: "Setup ASP.NET Project With PostgreSQL"
date: 2023-07-02T00:00:00-07:00
draft: true
---

When developing a new ASP.NET project, we have two main choices when it comes to communicating with the database servers. We can take a database-first approach or a code-first approach. 

With a database-first approach, we take full advantage of the database features and optimize our queries by communicating with the servers using stored procedures. With a code-first approach, we really more heavily on an ORM, such as Entity Framework Core, to communicate with the servers. With a code-first approach, we write our queries not in SQL but in the language of the ORM.

As you may have guessed, with a database-approach, we are locked into the database we have chosen, and swapping our database in the feature will take considerable effort. The upside, however, is that with a database-first approach, querying the database can be up to 10X in some cases. With a code-first approach, since our queries are in the language of the ORM, swapping between databases is considerably easier.

In this post we will setup a C# ASP.NET application with PostgreSQL.

There are numerous tutorials online about 