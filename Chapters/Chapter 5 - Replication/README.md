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

For the above reason, **the best practice you make one follow synchronous and the other is asynchronous** (this approach is sometimes called semi-synchronous).

## Setting Up New Followers

How do you ensure that the new follower has an accurate copy of the leader’s data? copying data files from one node to another will not work as there is constantly writing to DB.

To set up new follower without downtime

- Take a consistent snapshot of the leader’s database at some point in time.
- Copy the snapshot to the new follower node.
- The follower **connects** to the leader and **requests** all the data changes that have
happened since the snapshot was taken.

## Handling Node Outages

### Follower failure: Catch-up recovery

On its local disk, each follower keeps a log or snapshot of the data changes it has received from the leader. 

The follower can recover quite easily: from its log, it knows the last transaction that was processed before the fault occurred. Thus, the follower can connect to the leader and request all the data changes that occurred during the time when the follower was disconnected.

### Leader failure: Failover

If the leader crashed, one of the following needs to be promoted to be the new leader, and clients  need to be reconfigured to send their writes to the new leader, and the other followers need to start consuming data changes from the new leader. This process is called **failover**.

Failover can be happened manually (by administrator) or automatically by the following steps:

1. Determining that the leader has failed (based on node respond timeout)
2. Choosing a new leader through election process or by most updated data node.
3. Reconfiguring the system to use the new leader

Automatic failover may be go wrong

- If asynchronous replication is used, the new leader may not have received all the writes from the old leader before it failed.
- Promoted out of update follower can cause a conflicts and discording some data

Thus, some operations team prefer to perform failovers manually.

