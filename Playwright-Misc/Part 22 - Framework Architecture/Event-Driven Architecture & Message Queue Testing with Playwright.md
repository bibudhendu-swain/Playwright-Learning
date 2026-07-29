
Chapter: Event-Driven Architecture & Message Queue Testing with Playwright

What is Event-Driven Architecture?

Instead of one service directly calling another synchronously, services communicate by publishing events.

```
Traditional architecture:

User
 |
UI
 |
API
 |
Database

Event-driven architecture:

+----------------+
          |    UI/API      |
          +----------------+
                 |
          Publish Event
                 |
          +-------------+
          |   Kafka     |
          +-------------+
             |       |
             |       |
      Service A   Service B
             |       |
          Database  Email

The publisher doesn't know who consumes the message.

Everything happens asynchronously.

```


---

Why Playwright Engineers Should Know This

Suppose a user clicks Place Order.

The UI immediately returns success.

Behind the scenes:

Order Created

↓

Kafka Topic

↓

Inventory Service

↓

Payment Service

↓

Shipping Service

↓

Email Service

A UI test should verify more than just the success message.

It may also need to verify:

Kafka event published

Inventory updated

Email triggered

Shipment created


This is where event verification comes in.


---
```

Common Enterprise Pattern

Never make Playwright directly talk to Kafka.

Instead:

Playwright Test

↓

Business Layer

↓

Messaging Client

↓

Kafka / RabbitMQ

Example:

tests

↓

pages

↓

services

↓

messaging

↓

Kafka

For example:

tests/
pages/
services/
messaging/
    kafkaConsumer.ts
    kafkaProducer.ts
    kafkaAdmin.ts
utils/

Your page objects stay clean and focused on the UI.
```

---

Layered Architecture

UI Layer
    |
Page Objects
    |
Business Service
    |
Messaging Service
    |
Kafka Client

The business service coordinates UI actions and backend verification.


---

Example Flow

Login

↓

Create Order

↓

Click Submit

↓

Wait for UI Success

↓

Consume Kafka Event

↓

Validate Payload

↓

Finish Test

This is much more robust than adding arbitrary waits.


---

Event Verification Pattern

Avoid:

Click

Sleep 20 seconds

Hope Kafka finished

Instead:

Click

↓

Start Listening

↓

Receive Event

↓

Validate Event

↓

Continue

Always wait for the event itself, not time.


---

Event Listener Pattern

Create a reusable listener:

EventListener
    start()

    waitForMessage()

    validate()

    stop()

Every test can reuse the same listener.


---

Event Assertion Pattern

Assertions should cover:

Event exists

Topic name

Event key

Correlation ID

User ID

Order ID

Timestamp

Payload values


Not just "message received".


---

Correlation ID Pattern

One of the most important enterprise patterns.

UI

↓

Generate Correlation ID

↓

HTTP Header

↓

Kafka Event

↓

Database

↓

Logs

The same ID should flow through every component.

Your Playwright test can generate a unique ID and verify it appears in the event, making it easy to match the correct message.


---

Producer Pattern

Sometimes your test must publish an event to drive the system.

Playwright

↓

Kafka Producer

↓

Topic

↓

Application

↓

UI Updated

Useful for testing background processing or inbound integrations.


---

Consumer Pattern

Sometimes the application publishes the event and the test validates it.

UI

↓

Application

↓

Kafka

↓

Playwright Consumer

↓

Assertions

This is the most common scenario.


---

Hybrid UI + API + Kafka Pattern

UI

↓

REST API

↓

Kafka

↓

Database

↓

Email

↓

Assertions

Enterprise tests frequently span multiple layers rather than relying on UI checks alone.


---

Event Polling Pattern

Instead of sleeping:

Repeat

↓

Check Kafka

↓

Message Found?

↓

Yes → Continue

No → Retry

↓

Timeout

Retry with a sensible timeout to handle asynchronous processing.


---

Retry Strategy

Use exponential backoff:

1 sec

2 sec

4 sec

8 sec

16 sec

This reduces unnecessary load while waiting for events.


---

Test Data Isolation

Every test should use unique identifiers:

Order ID

User ID

Correlation ID

Session ID

Never rely on hard-coded values when consuming from shared topics.


---

Framework Structure

tests/

pages/

api/

services/

messaging/

    kafkaConsumer.ts

    kafkaProducer.ts

    eventValidator.ts

    topicResolver.ts

    correlation.ts

fixtures/

utils/

This keeps messaging concerns isolated from UI and API code.


---

Enterprise Flow

Create Order

↓

UI Success

↓

Kafka Event Published

↓

Inventory Reserved

↓

Payment Completed

↓

Shipping Event

↓

Notification Event

↓

Database Verification

↓

Test Pass


---

Best Practices

Keep messaging code separate from page objects.

Use reusable producer and consumer wrappers.

Validate message payloads, not just their existence.

Prefer correlation IDs to identify the right message.

Avoid fixed sleeps; wait for events with timeouts and retries.

Start consumers before the action when you need to ensure no events are missed.

Clean up consumers and connections after each test.

Make topic names configurable for different environments.



---