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
