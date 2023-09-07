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

### Linearizability VS Serializability

- **Serializability** is an **isolation guarantee property** of transactions, where every transaction may read and write multiple objects concurrent. Executed in **some** serial order.
- **Linearizability** is a **recency guarantee** on reads and writes of a register.

A database may provide both serializability and linearizability, and this combination is known as **strict serializability**.

Implementations of serializability based on **two-phase locking** or **actual serial execution** are **typically linearizable**. While serializable **snapshot isolation** is **not linearizable**.

### When need to rely on Linearizability?

- **Locking and leader election**: Make sure that only one leader is elected by using a lock. To make system linearizability, ZooKeeper used to implement distributed locks and leader election by using consensus algorithms.
- **Constraints and uniqueness guarantees**: Uniqueness constraints in database (such as username or email, if two people try to concurrently create a user or a file with the same name, one of them will be returned an error) should be linearizability. To implement it **add lock** on their chosen username or by compare-and-set **CAS(username, empty, newUserName)**.
    - Another examples that make sure that a bank account balance never goes negative, or that you don’t sell more items than you have in stock in the warehouse, or that two people don’t concurrently book the same seat on a flight or in a theater.
- **Cross-channel timing dependencie**s: if the system uses two different communication channels depending on each other. The system should be Linearizability to prevent race conditions.
    - For the following examples shows two different communication channels (web server and image resizer) If the file storage service is linearizable, then this system should work fine. If it is not linearizable, there is the risk of a race condition: the message queue (steps 3 and 4) might be faster than the internal replication inside the storage service. In this case, when the resizer fetches the image (step 5), it might see an old version of the image, or nothing at all.
    
    <p align="center" width="100%">
      <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%209%20-%20Consistency%20and%20Consensus/images/cross-channel-example.png" width="700" hight="500"/>
    </p>

## The CAP theorem

https://www.ibm.com/topics/cap-theorem

This CAP theorem show that you cannot achieve the 3 principles together into the distributed systems. You can achieve only two of them. 

- CA ⇒ has strong/strict consistency
- AP ⇒ has eventual consistency

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%209%20-%20Consistency%20and%20Consensus/images/cap-theorem.jpeg" width="700" hight="500"/>
</p>

- CA ⇒ If your application **requires linearizability**, and some replicas are disconnected from the other replicas due to a network problem, then some replicas cannot process requests while they are disconnected: they must either wait until the net‐ work problem is fixed.
- AP ⇒ If your application does **not require linearizability**, then it can be written in a way that each replica can process requests independently, even if it is disconnected from other replicas (e.g., multi-leader). In this case, the application can remain available in the face of a network problem

Thus, According to CAP theorem, applications that don't require linearizability, can be more tolerant of network problems, because it can remain available in the face of them.

The CAP theorem doesn’t say thing about network delays, dead nodes, or other trade-offs. Thus, although CAP has been historically influential, it has little practical value for designing systems.

### Linearizability and Performance

Although linearizability is a useful guarantee, few systems are actually linearizable in practice, as it is always slow even when there is no network fault, which decreases the performance significantly.
