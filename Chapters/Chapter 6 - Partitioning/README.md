## Chapter 6: Partitioning

Partitions are defined in such a way that each pie of data (record, document) belongs to **one partition**.

Each partition is a small database of its own, however we need to support operations that touch multiple partitions at the same time. The main reason for Replication is **high availability** and **reduce latency** while the main reason for partitioning **scalability.**

Usually we combine replication and partitioning: each node acts as leader for some partitions and follower for other partitions.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%206%20-%20Partitioning/images/partitioning-with-replicaiton.png" width="700" hight="500"/>
</p>

For 10 replicas theoretically speaking, we expect to get
- 10x Storage
- 10x Read and write throughput

## How do you partitioning data ?

### Partitioning of Key-Value Data

The goal of partitioning is to spread data evenly across nodes, and more importantly to avoid having *skewed* partitions with most of the load (called *hot spots*).

There are two main ways of partitioning keys:

**Partitioning by key range**

By sorting the keys we have and assigning a continuous range of keys (Like encyclopedia, partition is from A to B, other from D to F)

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%206%20-%20Partitioning/images/partitioning-by-key-value.png" width="700" hight="500"/>
</p>

The problem with this approach is the partitioning is not always fair and this lead that some partitions will have much bigger data than others (this called partition boundaries). This approach is used in HBase and RethinkDB, and MongoDB before version 2.4.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%206%20-%20Partitioning/images/partitioning-skewed.png" width="700" hight="500"/>
</p>

**Partitioning by Hash of Key**

To fix the issue of skewed data and makes it uniformly distributed. Thtats by using a non-cryptographic hash function (eg. MD5 used in Cassandra and MongoDB and Voldemort uses the Fowler-Noll-Vo hash function).

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%206%20-%20Partitioning/images/partitioning-by-hash-key.png" width="700" hight="500"/>
</p>

The issue with this approach is cannot do **range query (from - to)** from the same partition.

In MongoDB, if you have enabled hash-based sharding mode, any range query has to be sent to **all partitions**. Range queries on the primary key are not supported by Riak, Couchbase, or Voldemort

Cassandra find a compromise approach to achieve rang-query on hash key by using **compound/composite primary key**. also called “one-to-many actions” we will partition based on one which has many action/updates

Only the first part of that key is hashed to determine the partition (on the example: zipcode or username), but the other columns are used as a concatenated index for sorting the data.

for example the following query key = zipcode, user_name, activity_timestamp

```protobuf
zipcode = "2011201" AND
user_name like "M%" AND
activity_timestamp > "2023-01-01"
```

Another example, on a social media site, one user may post many updates. If the primary key for updates is chosen to be (user_id, update_timestamp), then you can efficiently retrieve all updates made by a particular user within some time interval, sorted by timestamp. Different users may be stored on different partitions, but within each user, the updates are stored ordered by timestamp on a single partition.

One problem still is when most reads and writes are for the **same key (causing hotspot and skewed partitions)**. for example on a social media site, a celebrity user with millions of followers may cause a storm of activity when they do something.

This usually left for the application to handle this skew, typically by assigning **random bytes** at the beginning/end of this key to scatter it across all the replicas, however, this requires extra bookkeeping/work, as well as requests to all the replicas when reading.
