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

## BASE

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%207%20-%20Transactions/images/acid-vs-base.png" width="700" hight="500"/>
</p>

good reference: https://neo4j.com/blog/acid-vs-base-consistency-models-explained/

- **Basically Available:** The system **appears** to work most of the time.
- **Soft State:**
    - We don’t have to be write-consistent (eg. read your own writes) or mutually consistent (between replicas) all time.
    - The state the user put into the system that will go away (expire) if user doesn’t maintain (refresh) it. your system depends on the external user factor by using refresh
- **Eventual Consistency**: the system will become consistent over time, given that the system doesn’t receive input during that time.

Storage engines almost universally aim to provide *atomicity* and *isolation* on the level of single object. Atomicity can be implemented using log for crash recovery, and *isolation* using a lock on each object.

Multi-object transactions might be needed when a foreign key references to a row in another table, or when de-normalized information and secondary indexes needs to be updated. However, many 
distributed datastores abandoned multi-object transactions because they are difficult to implement across partitions.

Leaderless replicated datastores won't undo something it has already done, so it's the application's responsibility to recover from errors.

Retrying aborted transactions isn't perfect 

- because If the transaction actually succeeded but the network failed while the server is trying to acknowledge the successful commit (so the client thinks it failed), the transaction might get performed twice.
- If the error is due to overload, retrying the transaction will make the problem worse, not better. To avoid such feedback cycles, you can limit the number of retries, use exponential backoff, and handle overload-related errors differently from other errors (if possible)
- It is only worth retrying after transient errors (for example due to deadlock, isolation violation, temporary network interruptions, and failover); when the error is **permanent** (eg. constraint violation), retrying would be pointless.

## Weak Isolation Levels

Concurrency issues (race conditions) only come into play when one transaction reads data that is concurrently modified by another transaction, or when two transactions try to simultaneously modify the same data.

**Serializable** isolation means that the database guarantees that transactions have the same effect as if they ran **serially** (i.e., one at a time, without any concurrency).

Concurrency bugs are hard to find by testing, for that, databases have long tried to hide it by providing *transaction isolation*, especially *serializable isolation*. However, due to its performance cost, systems use weaker levels of isolations more commonly, which protect against ***some* concurrency** issues, but not all. Many popular relational databases that are considered "ACID" use **weak isolation** themselves.

### Read Committed

The most basic level of transaction isolation is read committed.v It makes two guarantees:

- When reading from the database, you will only see data that has been **committed (no dirty reads).**
- When writing to the database, you will only **overwrite** data that has been **committed** **(no dirty writes)**

**No dirty reads**

Imagine a transaction has written some data to the database, but the transaction has not yet committed or aborted. Can another transaction see that uncommitted data? If yes, that is called a **dirty read**

The following example prevents dirty reads

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%207%20-%20Transactions/images/read-committed-example.png" width="700" hight="500"/>
</p>

**No dirty writes**

**Dirty writes** is when a transaction write overwrites another transaction's uncommitted write, this is useful because if the transaction updates multiple objects, dirty writes can lead to bad outcome, however, preventing it still doesn't prevent some other race conditions. Most databases prevents dirty writes by using row-level locks.

The following example shows dirty writes

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%207%20-%20Transactions/images/dirty-writes-example-1.png" width="700" hight="500"/>
</p>

The following example shows it still doesn’t prevent race conditions which that the second transaction happened after first one has committed (which means that there are no dirty writes) but still incorrect behavior. in “Preventing Lost Updates” we will discuss how to make such counter increments safe.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%207%20-%20Transactions/images/dirty-writes-example-2.png" width="700" hight="500"/>
</p>

**How to implement it?**

Read committed is a very popular isolation level. It is the default setting in Oracle 11g, PostgreSQL, SQL Server 2012, MemSQL, and many other databases.

- **To Prevent Dirty Writes**
    - Transaction acquire write lock (exclusive)
- **To Prevent Dirty Reads**
    - Acquire write lock and immediately release it. This would ensure that a read couldn’t happen while an object has a dirty, uncommitted value (because during that time the lock would be held by the transaction that has made the write)
        - The issue with this approach that may write transaction can take long time and hold the other reads to wait the writers to finish.
    - Another approach that most databases do is keeps 2 versions of each object value (**committed** and **overwritteen-but-not-yet-committed**) like the image in preventing dirty reads.

### Snapshot Isolation (Repeatable Read)

Read committed isolation doesn't protect against **read skew**, where a transaction reads different parts from the database in different points of time.

The following example explain **Read skew (non repeatable read)** problem which shows that Alice do transfer transaction between her two accounts. but when she see the two lists at the same time, she seen inconsistent state (one still with 500$ and the other account with 400$ so the total is $900 in her accounts—it seems that $100 has vanished into thin air)

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%207%20-%20Transactions/images/snapshot-isolation-example-1.png" width="700" hight="500"/>
</p>

Other examples for Read skew (non repeatable read) problem:

- **Taking Backups**: Taking a backup requires making a copy of the entire database, which may take hours on a large database. During the time that the backup process is running, writes will continue to be made to the database. Thus, you could end up with some parts of the backup containing an older version of the data, and other parts containing a newer version. If you need to restore from such a backup, the inconsistencies (such as disappearing money) become permanent
- **Executing Analytics query**
- **Doing integrity checks**

Snapshot isolation is the most common solution to this problem, where each transaction reads from a consistent snapshot of the database (only committed data at the beginning of transaction), as if it was frozen at a particular point in time.

Snapshot isolation is useful for read only queries like backups and analytics. this popular features is supported by PostgreSQL, MySQL with the InnoDB storage engine, Oracle, SQL Server, and others.

### Implementing snapshot isolation

By using write locks and reads do not require any locks.  From a performance point of view, a key principle of snapshot isolation is **readers never block writers, and writers never block readers**.

Implemented by **Multi-version concurrency control (MVCC)** meaning that each object in the database has multi version, because various in-progress transactions may need to see the state of the database at different points in time. When another transaction need to read it will return the data of with version number at this timestamp.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%207%20-%20Transactions/images/snapshot-isolation-example-2.png" width="700" hight="500"/>
</p>

In the above diagram there are two transactions initiated by previous transactions 5 and 3. In any writes the database keep multiple versions of the object according to the transactions. When transaction 12 try again to read the value it will find there two version (one caused by previous transaction 5 and one caused by transaction 13 which happened after me, so it will take any transaction happened before me) 

At some later time, when it is certain that no transaction can any longer access the deleted data, a garbage collection process in the database **removes** any rows marked for deletion and frees their space.

Indexing might sound like a problem for snapshot isolation, but one solution is to have the index point to *all* versions of an object, while another approach (used in CouchDB) is to use an append-only/copy-on-write variant that doesn't overwrite pages in the underlying tree.
