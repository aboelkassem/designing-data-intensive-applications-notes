## Chapter 7: Transactions

## Things that will go wrong with your application!

- The application or database will fail in the middle of a write operation
- Interruptions in the network will cut off the application from the database
- Several clients will write/read to the database at the same time, overwriting each other’s changes
- A client will read data that doesn’t make sense because it has only partially been updated
- Race conditions will keep happening.

To solve these issues we will use Transactions.
<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%207%20-%20Transactions/images/transactions-intro-1.png" width="700" hight="500"/>
</p>

A transaction is a way for an application to group several reads and writes together into a logical unit. Conceptually, all the reads and writes in a transaction are executed as one operation: either the entire transaction succeeds (commit) or it fails (abort, rollback)

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%207%20-%20Transactions/images/transactions-intro-2.png" width="700" hight="500"/>
</p>

By using transactions, the application is free to ignore certain potential error scenarios and concurrency issues, because the database takes care of them instead (we call these **safety guarantees (which treat them as synchronized operations)**).

This chapter discusses area of concurrency **control, various kinds of race conditions, and how databases implement isolation levels such as read committed, snapshot isolation, and serializability.**

Transactions support most relational databases like MySQL, PostgreSQL, Oracle, SQL Server, and some non-relational databases.

## The Slippery Concept of a Transaction

There are wrong misconceptions that any large system should abandon transactions in order to maintain good performance and high availability, and that transactions are essential to database vendors, but these viewpoints are pure hyperbole.

There are two mechanisms to achieve this ACID (Atomicity + Consistency + Isolation + Durability ) and its inverse called BASE (Basically Available + Soft state + Eventual Consistency)

## ACID

To achieve safety guarantees, this is described by ACID which stands to **Atomicity, Consistency, Isolation, and durability**.

Systems that do not meet the ACID criteria are sometimes called BASE, which stands for Basically Available, Soft state, and Eventual consistency. This is even more vague than the definition of ACID. It seems that the only sensible definition of BASE is “not ACID”; i.e., it can mean almost anything you want

### Atomicity

Refers to something that cannot be broken down into smaller parts, so when a client wants to make several writes, but a fault occurs, the whole transaction aborts/is discarded by the database, and the client can safely retry.

- **ALL or None**: The system can only be in the state it was during the operation or after the operation. not something in between.
- **Failure guarantee**

The following diagram illustrates that Atomicity ensures that if an error occurs any prior writes from that transaction are undone, to avoid an **inconsistent state**.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%207%20-%20Transactions/images/atomicity-example.png" width="700" hight="500"/>
</p>

### Consistency

Insert only valid data and that the database is being in a "good-state", which is not something the application should guarantee, not the database.

- **Data** should make sense (**domain-specific**: twitter vs banking system)
- **Semantic guarantee**

### Isolation

- Each transaction can not see what is happening now inside any other concurrent transaction till they finish.
- Implemented using locks
- **Concurrency guarantee**

The following example showing a **race condition problem** that they are reading the same data at the same time.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%207%20-%20Transactions/images/isolation-example-1.png" width="700" hight="500"/>
</p>

Another example shows violating isolation: one transaction reads another transaction’s uncommitted writes (a “dirty read”).

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%207%20-%20Transactions/images/isolation-example-2.png" width="700" hight="500"/>
</p>

To Fix this: we use Isolation

### Durability

- Any data it has written **successfully** will not be forgotten, even if there is a hardware fault or database crashes.
- Implemented using write-ahead-log or replication
- **Reliability guarantee** (perfect durability does not exist)
