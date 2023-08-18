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
