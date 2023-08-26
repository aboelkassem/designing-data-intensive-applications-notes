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

By using transactions, the application is free to ignore certain potential error scenarios and concurrency issues, because the database takes care of them instead (we call these **safety guarantees (which tread them as synchronized operations)**).

This chapter discusses area of concurrency **control, various kinds of race conditions and how databases implement isolation levels such as read committed, snapshot isolation, and serializability.**

Transactions supports on most relational databases like MySQL, PostgreSQL, Oracle, Sql Server and some non relational databases.
