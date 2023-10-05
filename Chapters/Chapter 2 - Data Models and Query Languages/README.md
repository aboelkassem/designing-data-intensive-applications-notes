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

JSON representation has better locality than the multi-table schema (Instead of doing multiple joins or queries in tables, in JSON all the relevant information is in one place, and one query is sufficient)

Document-oriented databases on the other hand supports **one-to-many** relationship natively, and provides better *locality* for the data object, thanks to the self-contained nature of JSON.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%202%20-%20Data%20Models%20and%20Query%20Languages/images/document-modal-tree.png" width="700" hight="500"/>
</p>

### Many-to-One and Many-to-Many Relationships

Removing such duplication is the key idea behind **normalization** in databases. 

Relational databases deal with **many-to-one** relationship by referring to rows in tables by ID, as joins are easy. However, Document databases doesn't nicely support **many-to-one** relationships. Instead, the application code would need to go through the overhead of simulating the join itself, which can cache it all in memory if it's small and slow-changing.

Document database worked well for one-to-many relationships, but it made many-to-many relationships difficult, and it didn’t support joins. Developers had to decide whether to duplicate (denormalize) data or to manually resolve references from one record to another.

In a relational database, the query optimizer automatically decides which parts of the query to execute in which order, and which indexes to use. The relational model thus made it much easier to add new features to applications.

When comparing document model to relational model, arguments in favor of document model are schema flexibility, better performance, and more-matching data structure with the application, while Relational model provides better support for joins, and **many-to-one** and **many-to-many** relationships.

**Which data model leads to simpler application code?**

- If the data in your application has a document-like structure (i.e., a tree of **one-to-many relationships**, where typically the entire tree is loaded at once or no relationships between records) then it’s probably a good idea to use a document model.
- If your application does use **many-to-many relationships and joins**, the relational model is more suitable.

Document databases are not *schemaless*, but rather have *schema on read* (the structure of the data is implicit, and only interpreted when the data is read) similar to dynamic (runtime) in oppose to relational model's *schema on write (*where the schema is explicit and the database ensures all written data conforms to it*)*. It is not enforced by the database, but more easier to change and modify similar to static (compile-time).

Example of storing each user’s full name in one field, and you instead want to store the first name and last name separately.

- In a document database, you would just start writing new documents with the new fields and have code in the application that handles the case when old documents are read. For example:

```jsx
if (user && user.name && !user.first_name) {
	// Documents written before Dec 8, 2013 don't have first_name
	user.first_name = user.name.split(" ")[0];
}
```

- On the other hand, in a “statically typed” database schema, you would typically perform a migration along the lines of:

```sql
ALTER TABLE users ADD COLUMN first_name text;
UPDATE users SET first_name = split_part(name, ' ', 1); -- PostgreSQL
UPDATE users SET first_name = substring_index(name, ' ', 1); -- MySQL
```

The schema-on-read approach is advantageous if the items in the collection don’t all

have the same structure for some reason (i.e., the data is heterogeneous)—for example, because:

- There are many different types of objects, and it is not practical to put each type of object in its own table.
- The structure of the data is determined by external systems over which you have no control and which may change at any time.

In situations like these, a schema may hurt more than it helps, and schemaless documents can be a much more natural data model. But in cases where all records are expected to have the same structure, schemas are a useful mechanism for documenting and enforcing that structure.

For document databases to benefit from *locality* (retrieve all document data and no need to joins lookups), documents have to be relatively small in size. It is generally recommended that you keep documents fairly small and avoid writes that increase the size of a document

Google’s Spanner database offers the same locality properties in a relational data model, by allowing the schema to declare that a table’s rows should be nested within a parent table. Oracle allows the same, using a feature called multi-table index cluster tables. The column-family concept in the Bigtable data model (used in Cassandra and HBase) has a similar purpose of managing locality.

It seems that relational and document databases are becoming more similar overtime. PostgreSQL and MySQL become support JSON documents.  RethinkDB supports relational-like joins, and some MongoDB drivers automatically resolve database references (effectively performing a client-side join, although this is likely to be slower than a join performed in the database).

A hybrid of relational and document models might be the future of databases, as they are becoming more similar over time. If a database is able to handle document-like data and also perform relational queries on it, applications can use the combination of features that best fits their needs.
