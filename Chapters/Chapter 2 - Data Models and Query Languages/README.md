# Chapter 2: Data Models and Query Languages
Data models are perhaps the most important part of developing software, because they have such a profound effect on the way we think about the problems we're solving.

## Relational Model vs Document Model

The best-known data model today is probably that of SQL, based on the relational model: data is organized into **relations** (called **tables** in SQL), where each relation is an unordered collection of **tuples** (**rows** in SQL).

by the mid-1980s, relational database management systems (RDBMSes) and SQL had become the tools of choice for most people who needed to store and query data with some kind of regular structure.

Relational model today’s use case for **transaction processing** (entering sales or banking transactions, airline reservations, stock-keeping in warehouses) and **batch processing** (customer invoicing, payroll, reporting).

The goal of the relational model was to hide that implementation detail behind a cleaner interface.

In the 2010s, NoSQL is the latest attempt to overthrow the relational model’s dominance. NoSQL name refer to 2009 twitter hashtag for new open source, non relational database meaning “**Not Only SQL**”. 

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%202%20-%20Data%20Models%20and%20Query%20Languages/images/history-of-data-modal.jpeg" width="700" hight="500"/>
</p>

NoSQL databases have been adopted quickly and easily because:

- It provided a better scaling mechanism, including very high write throughput than relational databases
- It was free and open source
- It supported few specific query operations better than relational databases
- It provided more dynamic and expressive data model than relational databases

Relational databases will continue to be used alongside a broad variety of non-relational datastores (Polyglot Persistence).

Many applications today is done in object-oriented programming languages. An translation layer is required between the objects in application code and the database model of tables, rows, and columns (SQL). **Object-relational mapping (ORM)** frameworks like ActiveRecord, EntityFramework, and Hibernate reduce the amount of boilerplate code required for this translation layer, but they can’t completely hide the differences between the two models.

The following image illustrate an example of resume (a LinkedIn profile) expressed in a relational schema.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%202%20-%20Data%20Models%20and%20Query%20Languages/images/example-of-data-relational-modal.png" width="700" hight="500"/>
</p>

Relational databases deal with the **one-to-many** relationship in one of three ways:

- The common normalized way is to put the *many* values in a separate table, with foreign key reference to the *one* (like the above image)
- Later versions of SQL allowed multi-valued data be stored in a single row, with support for querying inside them (XML or JSON datatypes).
- The least favorable option is to store them as encoded JSON or XML, and let the application do the internal query.
