## Chapter 5: Replication
Replication means keeping a copy of the same data on multiple machines that are connected via a network. Why we do replication?

- To keep data geographically close to your users (and thus reduce latency)
- Increase availability
- To scale out the number of machines that can serve read queries (Increase **read** throughput)

## **Leaders and Followers**

leader-based replication (or master-slave) is designated the leader when clients want to **write** to the database, they must send their requests to the leader, first writes the new data into its local storage, then sends the data change to all of its **followers**.

When a client wants to read from the database, it can query either the leader or any of the followers. However, writes are only accepted on the leader.

This mode is build-in feature in relational databases such as PostgresSQL, MySQL, Oracle, and SQL Server. Also, for non-relational databases such as MongoDB, RethinkDB, Espresso. Also in message brokers such as Kafka, RabbitMQ.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%205%20-%20Replication/images/leader-based-replicaiton.png" width="700" hight="500"/>
</p>

## Synchronous vs Asynchronous Replication

Synchronous replication mean that we will wait until all followers get updated, then responded to the user.

Asynchronous replication mean that we will response immediately to the user, and after that in the background we update the followers.

The following example shows Leader-based replication with one synchronous and one asynchronous follower.


<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%205%20-%20Replication/images/sync-vs-async-replication.png" width="700" hight="500"/>
</p>

Follower 1 is **synchronous**: the leader waits until follower 1 has confirmed that it received the write before reporting success to the user.

Follower 2 is **asynchronous**: the leader sends the message, but doesn’t wait for a response from the follower.

The advantage of synchronous

- The follower is guaranteed to have an up-to-date copy.
- If the leader suddenly fails, we can be sure that the data is still available on the follower.

The disadvantage of synchronous

- The leader must block all writes and wait until the synchronous replica is available again.

For the above reason, **the best practice you make one follow synchronous and the other are asynchronous** (this approach sometimes called semi-synchronous).
