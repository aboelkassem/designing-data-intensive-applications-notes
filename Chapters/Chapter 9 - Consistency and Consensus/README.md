# Chapter 9: Consistency and Consensus 
The simplest way of handling system faults is to simply let the entire system fail, and then show an error message. But, the best way is to have a **general-purpose abstraction** with useful **guarantees** that we can implement.

One of the most important abstractions for distributed systems is **consensus**: that is, getting all of the nodes to agree on something (Like in single leader replication , when leader is dies and we need to fail over another node to be come the leader, the implementation of this **consensus** can prevent problems like if two nodes believe that they are the leader: split brain problem).

# Consistency Guarantees

Most replicated databases provide at least *eventual consistency*, with such a weak guarantee we need to be aware of its limitations, and not to assume too much, as these limitations only appear when there is a fault in the system.

Systems with stronger guarantees may have **worse performance** or be **less fault-tolerant** than systems with weaker guarantees. Nevertheless, stronger guarantees can be appealing because they are easier to use correctly.

There is some similarity between consistency models and transaction isolation levels. Isolation levels is primarily about avoiding race conditions due to concurrently executing transactions (**Isolation guarantee**), whereas distributed consistency is mostly about coordinating the state of replicas in the face of delays and faults (**ordering/recency guarantee**).

# Linearizability

Linearizability achieve strong consistency that guarantee **that every client would have the same copy/view of the data and all operations on it are atomic**, and you wouldn’t have to worry about replication lag.

linearizability (also known atomic consistency, strong consistency, immediate consistency) **is recency guarantee.**.

The following examples shows an system is not linearizable, causing football fans to be confused.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%209%20-%20Consistency%20and%20Consensus/images/linearizability-problem.png" width="700" hight="500"/>
</p>

To understand **concurrent operations** (reads and writes), see the following example what shows uncertainty, it may return either the old or the new value.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%209%20-%20Consistency%20and%20Consensus/images/concurrent-problem-1.png" width="700" hight="500"/>
</p>

To achieve linearizable: you consider that once a new value has been written or read by any client, all subsequent reads **must also return the new value**. Even if the write operation has not yet completed.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%209%20-%20Consistency%20and%20Consensus/images/concurrent-problem-2.png" width="700" hight="500"/>
</p>

In a linearizable system we imagine that there must be some point in time (between the start and end of the write operation). the following screen shows an example of three concurrent threads at some point in the time span has the actual read/write the value.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%209%20-%20Consistency%20and%20Consensus/images/linearizability-history.png" width="700" hight="500"/>
</p>

The following example shows the timing diagram to visualize each operation taking effect atomically at some point in time.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%209%20-%20Consistency%20and%20Consensus/images/linearizability-example-error.png" width="700" hight="500"/>
</p>

**cas(x, V*old*, V*new*)**: is hardware/process atomic operation. Like read and write operations

If the current value of the register x equals V*old,* it should be atomically set to V*new*. If x ≠ V*old* then the operation should leave the register unchanged and return an error. r is the database’s response (ok or error)

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%209%20-%20Consistency%20and%20Consensus/images/linearizability-cas.png" width="700" hight="500"/>
</p>

It's possible to test whether a system is linearizable by recording the timings of all requests and responses and check whether they are sequential.
