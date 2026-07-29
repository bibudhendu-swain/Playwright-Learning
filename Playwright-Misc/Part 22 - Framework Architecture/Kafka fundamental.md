Chapter: Kafka Fundamentals for Automation Testers

Why Should an Automation Tester Learn Kafka?

Traditionally, UI automation only verified what was visible on the screen.

Modern enterprise applications are different. A single button click may trigger multiple backend services that communicate through events instead of direct API calls.

For example:

User clicks "Place Order"

↓

UI sends HTTP Request

↓

Order Service

↓

Publishes Kafka Event

↓

Inventory Service

↓

Payment Service

↓

Shipping Service

↓

Notification Service

The UI may show "Order Created Successfully", but that doesn't guarantee the downstream services have processed the order correctly.


---

What is Kafka?

Kafka is a distributed event streaming platform.

Think of it as a high-speed messaging system where applications exchange events instead of calling each other directly.

Instead of:

Application A

↓

Application B

You have:

Application A

↓

Kafka

↓

Application B
Application C
Application D

One event can be consumed by many services independently.


---

Real-Life Analogy

Imagine a newspaper publisher.

The publisher prints newspapers.

People subscribe to the newspaper.

Every subscriber receives the same edition.


Kafka works similarly.

Producer

↓

Topic (Newspaper)

↓

Consumer A

Consumer B

Consumer C

The producer doesn't know who is reading the messages.


---

Core Components

1. Producer

A producer publishes messages.

Example:

Create Order

↓

Publish Event

The producer only sends data.


---

2. Consumer

A consumer reads messages.

Example:

Order Created

↓

Inventory Service

Consumers process the messages.


---

3. Topic

A topic is like a category or channel.

Examples:

order-created

payment-success

inventory-updated

shipment-created

Applications publish and consume from topics.


---

4. Broker

A broker is a Kafka server.

Broker 1

Broker 2

Broker 3

Together, multiple brokers form a Kafka cluster.

Your Playwright tests connect to the cluster using the bootstrap servers provided by your infrastructure team.


---

5. Cluster

A cluster is a group of brokers.

Broker

Broker

Broker

↓

Kafka Cluster

Clusters provide scalability and fault tolerance.


---

6. Event

An event is simply a message.

Example:

{
  "orderId": 1001,
  "status": "CREATED",
  "customer": "John"
}

This is what your Playwright framework will validate.


---

What Happens When You Click a Button?

Imagine clicking Submit Order.

Playwright

↓

Click Submit

↓

REST API

↓

Order Service

↓

Kafka

↓

Inventory

↓

Payment

↓

Shipping

↓

Email

One UI action may generate multiple events.


---

Producer Flow

Application

↓

Create Order

↓

Kafka Producer

↓

Topic


---

Consumer Flow

Topic

↓

Consumer

↓

Business Logic

Consumers receive events independently.


---

Multiple Consumers

One event can have many consumers.

Topic

↓

Inventory

↓

Payment

↓

Email

↓

Analytics

Each consumer performs a different task.


---

Partitions

A topic can be divided into partitions.

Instead of storing all messages together:

Order Topic

↓

Partition 1

Partition 2

Partition 3

Partitions improve scalability and allow multiple consumers to work in parallel.

For most automation testers, you don't need to manage partitions directly, but it's useful to know they exist.


---

Offset

Every message has an offset.

Offset 0

Offset 1

Offset 2

Offset 3

Think of it as the message number within a partition.

Consumers use offsets to remember what they've already processed.


---

Consumer Group

Consumers can belong to the same group.

Topic

↓

Group A

↓

Consumer 1

Consumer 2

Consumer 3

Kafka distributes messages among consumers in the same group, enabling parallel processing without duplicate work.

Different consumer groups each receive their own copy of the messages.


---

Why Does This Matter for Playwright?

Suppose your application publishes:

{
   "orderId": 1001,
   "status": "CREATED"
}

Your Playwright test may need to verify:

Was the event published?

Was the correct topic used?

Is the orderId correct?

Is the status correct?

Was the event published within an acceptable time?



---

Kafka vs REST API

REST API	Kafka

Request/Response	Publish/Subscribe
Immediate response	Asynchronous processing
One consumer	Multiple consumers
Caller waits	Publisher continues immediately
HTTP	Event Streaming



---

Common Enterprise Topics

order-created

order-updated

payment-success

payment-failed

inventory-updated

shipment-created

email-sent

audit-log


---

Where Does Playwright Fit?

Playwright

↓

Click Button

↓

REST API

↓

Kafka Event

↓

Consumer

↓

Database

↓

Assertions

Playwright doesn't replace Kafka—it verifies that the correct events are produced and, when needed, that downstream processing has completed successfully.


---

What Should an Automation Tester Know?

By the end of this chapter, an automation tester should be comfortable with:

What Kafka is and why event-driven systems use it.

The roles of producers, consumers, topics, brokers, and clusters.

The concepts of partitions, offsets, and consumer groups.

The difference between synchronous REST APIs and asynchronous messaging.

How a UI action can trigger backend events.

Why Playwright tests may need to verify those events.