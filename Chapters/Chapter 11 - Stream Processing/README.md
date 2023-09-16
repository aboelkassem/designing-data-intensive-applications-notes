# Chapter 11: Stream Processing

What is Steam?

Stream refers to data that is incrementally made available over time. So the dataset never ends and we need to process the data received up till now. On the other hand, in batch processing we know when the database has finished, we can start computation after that.

In streaming, the input is an event which immutable object containing the details of something that happened at some point in time. It may be encoded as a text string, or JSON, or perhaps in some binary form.

A database is sufficient to connect producers and consumers. However, continuous polling is expensive, so its better for consumers to be notified when new events appear, the behavior that usually requires specialized tools such as messaging systems.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%2011%20-%20Stream%20Processing/images/event.png" width="700" hight="500"/>
</p>

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%2011%20-%20Stream%20Processing/images/transmission.png" width="700" hight="500"/>
</p>

## Messaging System

A common approach for notifying consumers about new events is to use **Messaging systems**: allows multiple producers nodes to send messages to the same topic and allows multiple consumer nodes to receive messages in a topic.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%2011%20-%20Stream%20Processing/images/pub-sub-modal.png" width="700" hight="500"/>
</p>

Within this publish/subscribe model, it might be helpful to ask the following two questions:

- **What happens if the producers send messages faster than the consumers can process them**? the system can drop messages, buffer messages in a queue, or apply backpressure (block producer to send new events).
- **What happens if nodes crash or temporarily go offline—are any messages lost**? it depends on the application, like if with sensor streaming which missing data point perhaps not important. And for durability, a combination of **writing messages to disk and having replication** might be used, but with a cost of lower throughput and higher latency.

One option for a messaging system is **direct network communication**, such as UDP multi-cast, broker-less messaging libraries like ZeroMQ, or direct HTTP or RPC requests. However, their biggest drawback is that they require applications to be aware of loss possibility.

Another more **widely** used option is communication via a ***message broker* or *message queue***, which acts as a server that both producers and consumers connect to, it automatically deletes a message after delivery, it supports some way of subscribing to a subset of **topics**, and **it notifies clients when data changes**. Consumers are generally **asynchronous**: when a producer sends a message, it normally only waits for the broker to confirm that it has buffered the message and does not wait for the message to be processed by consumers

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%2011%20-%20Stream%20Processing/images/broker.png" width="700" hight="500"/>
</p>

This is the traditional view of message brokers, which is encapsulated in **standards** like JMS and AMQP and **implemented** in software like RabbitMQ, ActiveMQ, HornetQ, Qpid, TIBCO Enterprise Message Service, IBM MQ, Azure Service Bus, and Google Cloud Pub/Sub.

Message Broker can decide to distribute the event load among consumers (**Load balancing**), or deliver all messages to all consumers (**Fan-out**), or a combination of both.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%2011%20-%20Stream%20Processing/images/message-broker-modals.png" width="700" hight="500"/>
</p>

**Acknowledgments and redelivery**

Message brokers use **acknowledgments**: a client must explicitly tell the broker when it has finished processing a message so that the broker can remove it from the queue. If the connection to a client is closed or times out without the broker receiving an acknowledgment, it assumes that the message was not processed, and therefore it delivers the message again to another consumer.

In load balancing approach, The following example, consumer 2 crashes while processing m3, so it is redelivered to consumer 1 at a later time.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%2011%20-%20Stream%20Processing/images/message-broker-acknowledgment.png" width="700" hight="500"/>
</p>

But this leads to inconsistency with the order that were sent by producer 1. Message broker tries to preserve the order of messages and use a separate queue per consumer to solve this issue. 

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%2011%20-%20Stream%20Processing/images/messagning-systems.png" width="700" hight="500"/>
</p>

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%2011%20-%20Stream%20Processing/images/broker-vs-database.png" width="700" hight="500"/>
</p>

## Partitioned Logs (Kafka)

**Log-based message brokers** is simply an **append-only** sequence of records on disk. A producer sends a message by appending it to the end of the log, and a consumer receives messages by reading the log sequentially. If a consumer reaches the end of the log, it waits for a notification that a new message has been appended. The Unix tool `tail -f`, which watches a file for data being appended, essentially works like this.

For scaling out, the log is partitioned on different machines, making each partition is independently for other partitions. A topic is defined to group some partitions. Each consumer has a **read offset** per partition and be grouped also.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%2011%20-%20Stream%20Processing/images/partitioned-logs.png" width="700" hight="500"/>
</p>

Apache Kafka, Amazon Kinesis Streams, and Twitter’s DistributedLog are log-based message brokers that work like this. Google Cloud Pub/Sub is architecturally similar but exposes a JMS-style API rather than a log abstraction

## Partitioned Log Broker vs Message Queue/Passing Broker

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%2011%20-%20Stream%20Processing/images/partitioned-logs-vs-message-queue-1.png" width="700" hight="500"/>
</p>


<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%2011%20-%20Stream%20Processing/images/partitioned-logs-vs-message-queue-2.png" width="700" hight="500"/>
</p>


<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%2011%20-%20Stream%20Processing/images/partitioned-logs-vs-message-queue-3.png" width="700" hight="500"/>
</p>
