# Chapter 12: The Future of Data Systems

## Data Integration

The most appropriate choice of software tool also depends on the circumstan‐ ces. Every piece of software, even a so-called “general-purpose” database, is designed for a particular usage pattern.

You need to know how figure out the mapping between the software products and the circumstances in which they are a good fit.

## Combining Specialized Tools by Deriving Data

For example, it is common to need to integrate an OLTP database with a full-text search index in order to handle queries for arbitrary keywords. Although some databases (such as PostgreSQL) include a full-text indexing feature, which can be suffi‐ cient for simple applications.

As the number of different representations of the data increases, the integration problem becomes harder. Besides the database and the search index, perhaps you need to keep copies of the data in analytics systems (data warehouses, or batch and stream processing systems); maintain caches or denormalized versions of objects that were derived from the original data; pass the data through machine learning, classification, ranking, or recommendation systems; or send notifications based on changes to the data.

Distributed transactions decide on an ordering of writes by using locks for mutual exclusion (Two-Phase Locking (2PL) but this always comes with some limitations and overheads (like XA has poor fault tolerance and performance), while CDC and event sourcing use a log for ordering. Distributed transactions use atomic commit to ensure that changes take effect exactly once, while log-based systems are often based on deterministic retry and idempotence. 

In the absence of widespread support for a good distributed transaction protocol, I believe that log-based derived data is the most promising approach for integrating different data systems. However, guarantees such as reading your own writes are useful.

### Batch and Stream Processing

Both batch processing and streaming processing has a quite strong functional flavor which is good for fault tolerance, and for reasoning about the dataflows inside an organization.it is helpful to think in terms of data pipelines that derive one thing from another, pushing state changes in one system through functional application code and applying the effects to derived systems.

The outputs of batch and stream processes are derived datasets such as search indexes, materialized views, recommendations to show to users, aggregate metrics, and so on.

Spark performs stream processing on top of a batch processing engine by breaking the stream into microbatches, whereas Apache Flink performs batch processing on top of a stream processing engine.

Derived views allow *gradual evolution*, which means we can maintain two (old and new) schemas side by side independently. The beauty of this is that we always have a working system to go back to.

Some systems which need a quickly approximated data through stream processing, as well as correct and reliable version of the data later through batch processing, usually use **lambda architecture**, which records incoming data as immutable events to an always-growing dataset, and runs both systems in parallel, where each uses a derived view. The only down side is the operational complexity of debugging and maintaining two different systems.

In the lambda approach, the stream processor consumes the events and quickly pro‐ duces an approximate update to the view; the batch processor later consumes the same set of events and produces a corrected version of the derived view.

## Unbundling Databases

So far there are many various features provided by databases and how they work, including:

- Secondary indexes, which allow you to efficiently search for records based on the value of a field
- Materialized views, which are a kind of precomputed cache of query results
- Replication logs, which keep copies of the data on other nodes up to date
- Full-text search indexes, which allow keyword search in text

Database is consisted of different interacting components that we usually take for granted to work synchronously to achieve the desired storage role. This traditional synchronous actions require distributed transactions with all its overheads, so an asynchronous event-log (with Idempotence) might be a much more robust and practical approach, which leads us to the concept of *unbundling the database*.

Unbundling the database means building systems that abstractly acts like a database, but it in fact consists of a loosely coupled components. This has the advantages of making the system more robust to outages or performance degradation of individual components.

The goal of unbundling is to allow to combine several different databases in order to achieve good performance for much wider range of workloads that no single piece of software can satisfy them all.

### Designing Applications Around Dataflow

deployment and cluster management tools such as Mesos, YARN, Docker, Kubernetes, and others are designed specifically for the purpose of running application code. By focusing on doing one thing well, they are able to do it much better than a database that provides execution of user-defined functions as one of its many features.

It might make sense to have some parts of a system that specialize in durable data storage, and other parts that specialize in running application code. The two can interact while still remaining independent.

Most web applications today are deployed as **stateless services**, in which any user request can be routed to any application server, and the server forgets everything about the request once it has sent the response. The trend has been to keep stateless application logic separate from state management (databases): not putting application logic in the database and not putting persistent state in the application. As people in the functional programming community like to joke, “We believe in the separation of Church and state”

The advantage of such a service-oriented architecture over a single monolithic application is primarily organizational scalability through loose coupling: different teams can work on different services, which reduces coordination effort between teams (as long as the services can be deployed and updated independently).

The difference between dataflow systems compared to microservices is that it has a one-directional, asynchronous communication mechanism, rather than synchronous request/response interaction, so instead of RPC we have a stream join between events.

dataflow systems can also achieve better performance. For example, say a customer is purchasing an item that is priced in one currency but paid for in another currency. In order to perform the currency conversion, you need to know the current exchange rate. This operation could be implemented in two ways:

- ***In the microservices approach,*** the code that processes the purchase would probably query an **exchange-rate service** or database in order to obtain the current rate for a particular currency
- .***In the dataflow approach***, the code that processes purchases would subscribe to a **stream of exchange rate updates** ahead of time, and record the current rate in a local database whenever it changes. When it comes to processing the purchase, it only needs to query the local database.

Not only is the dataflow approach faster, but it is also more robust to the failure of another service. The fastest and most reliable network request is no network request at all! Instead of RPC, we now have a stream join between purchase events and exchange rate update events.


### Observing Derived State

The following diagram shows an example of updating a search index when write and read.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%2012%20-%20The%20Future%20of%20Data%20Systems/images/update-search-index-example.png" width="700" hight="500"/>
</p>

HTTP-based feed subscription protocols like RSS are really just a basic form of polling.

More recent protocols have moved beyond the basic request/response pattern of HTTP: server-sent events (the EventSource API) and **WebSockets** provide communication channels by which a web browser can keep an open **TCP connection to a server**, and the server can actively **push messages** to the browser as long as it remains connected. This provides an opportunity for the server to actively inform the enduser client about any changes to the state it has stored locally, reducing the staleness of the client-side state

Recent tools for developing stateful clients and user interfaces, such as the Elm language and Facebook’s toolchain of React, Flux, and Redux, already manage internal client-side state by subscribing to a stream of events representing user input or responses from a server, structured similarly to event sourcing. Some applications, such as instant messaging and online games, already have such a “real-time” architecture

The ideas of stream processing and messaging and not restricted to datacenters, but we can extend them all the way to the end-user devices.

### **Aiming for Correctness**

Transactions have been the choice for building correct applications for more than four decades by now, and while in some areas they have been completely abandoned for their overheads, they are not going away, but also correctness can be achieved in the context of dataflow.

Data systems that provide strong safety properties (eg. serializable transactions) are not guaranteed to be free from data loss or corruption. However, it would be easier to recover from such mistakes by preventing faulty code from destroying good (immutable) data. One of the most effective approaches to achieve this is to make all operations *idempotent.*

Duplicate suppression can be happened TCP connection for a client’s connection to a database and it is currently executing the following transaction

```sql
# This transaction is non-idempotent
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance + 11.00 WHERE account_id = 1234;
UPDATE accounts SET balance = balance - 11.00 WHERE account_id = 4321;
COMMIT;
```

In many databases, a transaction is tied to a client connection (if the client sends several queries, the database knows that they belong to the same transaction because they are sent on the same TCP connection). If the client suffers a network interruption and connection timeout after sending the COMMIT, but before hearing back from the database server, it does not know whether the transaction has been committed or aborted.

Two-phase commits are not sufficient to ensure that the transaction will be executed once, so to make an operation idempotent, we need to consider *end-to-end flow* of the whole operation. For example, you could generate a unique identifier for an operation (such as a UUID) and include it as a hidden form field in the client application, or calculate a hash of all the relevant form fields to derive the operation ID. If the web browser submits the POST request twice, the two requests will have the same operation ID.

```sql
ALTER TABLE requests ADD UNIQUE (request_id); # if the request has the same request_id, the insert will fail

BEGIN TRANSACTION;
INSERT INTO requests
	(request_id, from_account, to_account, amount)
	VALUES('0286FDB8-D7E1-423F-B40B-792B3608036C', 4321, 1234, 11.00);

UPDATE accounts SET balance = balance + 11.00 WHERE account_id = 1234;
UPDATE accounts SET balance = balance - 11.00 WHERE account_id = 4321;
COMMIT;
```
