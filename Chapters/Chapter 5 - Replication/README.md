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

