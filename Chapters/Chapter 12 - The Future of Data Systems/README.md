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
