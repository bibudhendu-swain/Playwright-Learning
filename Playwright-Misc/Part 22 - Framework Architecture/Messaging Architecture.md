In most large organisations, Playwright is just one part of the ecosystem. It also interacts with REST APIs, databases, Kafka, RabbitMQ, Azure Service Bus, and more. The key is to keep your tests independent of the underlying messaging technology.

Chapter: Messaging Architecture for Enterprise Playwright Frameworks

Why Do We Need a Messaging Layer?

A common mistake is to put Kafka code directly into Playwright tests.

❌ Bad design:

Test

↓

Click Button

↓

Kafka Consumer

↓

Assertion

Now every test knows about Kafka. If your company switches to RabbitMQ or Azure Service Bus, hundreds of tests must change.

Instead, create an abstraction.


---

```

Enterprise Architecture

Playwright Test
                      │
                      ▼
              Business Service
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
     REST Services         Messaging Layer
                                    │
                ┌───────────────────┼───────────────────┐
                ▼                   ▼                   ▼
            Kafka             RabbitMQ         Azure Service Bus

The Playwright test only talks to the Business Service.

The Business Service decides whether it needs to call an API, consume an event, or query a database.
```

---

Layer 1 – Playwright Test

The test should describe business behaviour.

test('Create Order', async () => {

    await orderPage.createOrder();

    await orderService.verifyOrderCreated();

});

Notice there is no Kafka code here.


---

Layer 2 – Business Service

The service orchestrates the flow.

Order Service

↓

Create Order

↓

Wait for Event

↓

Validate Event

↓

Return

It knows what should happen, but not how the message broker works internally.


---

Layer 3 – Messaging Layer

This layer is responsible for:

Publishing messages

Receiving messages

Waiting for events

Filtering messages

Handling retries

Managing connections


Everything broker-specific stays here.


---

Messaging Interface

Instead of depending on Kafka directly, define a contract.

export interface MessageBroker {

    publish(topic: string, payload: object): Promise<void>;

    waitForEvent(
        topic: string,
        filter: (event: any) => boolean
    ): Promise<any>;

}

Every messaging system implements this interface.


---

Kafka Implementation

MessageBroker

↓

KafkaBroker

↓

KafkaJS


---

RabbitMQ Implementation

MessageBroker

↓

RabbitMqBroker

↓

amqplib


---

Azure Service Bus

MessageBroker

↓

AzureServiceBusBroker

↓

Azure SDK

Your Playwright framework doesn't care which one is underneath.


---

Recommended Folder Structure

src/

messaging/

    MessageBroker.ts

    KafkaBroker.ts

    RabbitMqBroker.ts

    AzureServiceBusBroker.ts

    EventValidator.ts

    TopicResolver.ts

    CorrelationManager.ts

services/

pages/

tests/

Everything related to messaging lives in one place.


---

Topic Resolver Pattern

Avoid hardcoding topic names.

❌

await broker.waitForEvent("order-created");

Instead:

await broker.waitForEvent(
    TopicResolver.orderCreated()
);

TopicResolver can return different topic names for different environments.

DEV → order-created-dev

QA → qa-order-created

PROD → order-created

The tests never change.


---

Correlation Manager

In shared environments, many events are flowing at the same time.

Generate a unique correlation ID.

Playwright

↓

UUID

↓

REST Request

↓

Kafka

↓

Database

↓

Assertions

When the event arrives:

{
  "correlationId": "1234-5678",
  "orderId": 1001
}

Your framework matches the correct event using the correlation ID.


---

Event Validator

Don't scatter assertions across tests.

Instead:

Event

↓

Event Validator

↓

Playwright Assertions

Example responsibilities:

Validate schema

Validate mandatory fields

Validate business rules

Validate timestamps

Validate IDs



---

Retry Strategy

Instead of:

Click

↓

Sleep 20 seconds

↓

Read Kafka

Use:

Click

↓

Poll Broker

↓

Event Found?

↓

Yes

↓

Continue

Always wait for the event itself, not for time to pass.


---

Connection Manager

Don't reconnect for every message.

Instead:

Playwright Worker

↓

Messaging Connection

↓

Reuse

↓

Close at Worker End

This reduces execution time significantly in large suites.


---

Event Filtering

Large topics may contain thousands of messages.

Filter on:

Correlation ID

Order ID

Customer ID

Session ID

Event type


Never consume the first message blindly.


---

Event Schema Validation

Validate the event structure before checking business values.

For example:

{
  "eventType": "OrderCreated",
  "orderId": 1001,
  "status": "CREATED",
  "timestamp": "2026-07-29T10:30:00Z"
}

First verify the schema, then verify the content.


---

Enterprise Workflow

Playwright Test

↓

Page Object

↓

Business Service

↓

REST API

↓

Application

↓

Kafka Event

↓

Messaging Layer

↓

Event Validator

↓

Assertions

Notice how the test never communicates directly with Kafka.


---

Extending Beyond Kafka

Because everything depends on the MessageBroker interface, switching technologies becomes easy.

Today:

MessageBroker

↓

Kafka

Tomorrow:

MessageBroker

↓

RabbitMQ

No Playwright tests need to change.


---

Best Practices

1. Never put Kafka or RabbitMQ code directly in tests.


2. Keep messaging in its own layer.


3. Use interfaces to abstract broker implementations.


4. Use correlation IDs to identify the correct event.


5. Validate event schemas before business data.


6. Reuse messaging connections where possible.


7. Resolve topic names dynamically instead of hardcoding them.


8. Keep page objects focused on UI interactions only.


9. Let business services orchestrate UI, APIs, messaging, and database checks.


10. Design for multiple messaging systems from the beginning, even if you currently only use Kafka.




---

Where This Fits in Your Enterprise Framework

By this point, your Playwright framework architecture will naturally evolve into:

Playwright Test
        │
        ▼
Page Objects
        │
        ▼
Business Services
        │
 ┌──────┼────────┬────────────┐
 ▼      ▼        ▼            ▼
API   Messaging Database   Utilities
 │        │
 ▼        ▼
REST   Kafka/RabbitMQ/
       Azure Service Bus

This architecture scales well because every responsibility has its own layer. If your organisation later adds another broker or changes infrastructure, you update only the messaging implementation—not the tests, page objects, or business services. This is the same design principle you've been following in your enterprise Playwright framework for APIs, databases, and reusable utilities.