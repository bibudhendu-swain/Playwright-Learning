For an enterprise Playwright framework, do not put Kafka code directly inside the tests. Instead,create a reusable messaging layer, just like we do for APIs.

Recommended Project Structure
```
src/
├── pages/
├── services/
├── api/
├── messaging/
│   ├── kafkaConsumer.ts
│   ├── kafkaProducer.ts
│   ├── kafkaClient.ts
│   ├── eventValidator.ts
│   └── topicResolver.ts
├── fixtures/
├── utils/
└── tests/

```
---

Step 1: Kafka Client

Install:

npm install kafkajs

// kafkaClient.ts

import { Kafka } from 'kafkajs';

export const kafka = new Kafka({
    clientId: 'playwright-tests',
    brokers: ['localhost:9092']
});


---

Step 2: Kafka Producer

// kafkaProducer.ts

import { kafka } from './kafkaClient';

export class KafkaProducer {

    async publish(topic: string, message: any) {

        const producer = kafka.producer();

        await producer.connect();

        await producer.send({
            topic,
            messages: [
                {
                    value: JSON.stringify(message)
                }
            ]
        });

        await producer.disconnect();
    }

}

Usage

await producer.publish("order-created", {

    orderId: 1001,

    customer: "John"

});


---

Step 3: Kafka Consumer

// kafkaConsumer.ts

import { kafka } from "./kafkaClient";

export class KafkaConsumer {

    async consume(topic: string) {

        const consumer = kafka.consumer({
            groupId: 'playwright-group'
        });

        await consumer.connect();

        await consumer.subscribe({
            topic,
            fromBeginning: false
        });

        return new Promise((resolve) => {

            consumer.run({

                eachMessage: async ({ message }) => {

                    resolve(JSON.parse(message.value!.toString()));

                    await consumer.disconnect();

                }

            });

        });

    }

}


---

Step 4: Event Validator

// eventValidator.ts

import { expect } from "@playwright/test";

export class EventValidator {

    validateOrderCreated(event: any, orderId: number) {

        expect(event.orderId).toBe(orderId);

        expect(event.status).toBe("CREATED");

        expect(event.customer).toBeTruthy();

    }

}


---

Step 5: Business Service

Rather than calling Kafka directly from the test:

await orderPage.createOrder();

const event = await consumer.consume("order-created");

validator.validateOrderCreated(event, orderId);

Wrap it:

export class OrderService {

    async verifyOrderCreated(orderId: number) {

        const event = await consumer.consume("order-created");

        validator.validateOrderCreated(event, orderId);

    }

}

Then your test becomes:

await orderPage.createOrder();

await orderService.verifyOrderCreated(orderId);

Notice the test has no Kafka-specific code.


---

Advanced Pattern – Wait for a Specific Event

In shared environments, many events may arrive. Filter by a unique identifier (such as a correlation ID) instead of taking the first message.

export async function waitForEvent(
    topic: string,
    correlationId: string
) {

    return new Promise((resolve) => {

        consumer.run({

            eachMessage: async ({ message }) => {

                const event = JSON.parse(
                    message.value!.toString()
                );

                if (event.correlationId === correlationId) {

                    resolve(event);

                    await consumer.disconnect();

                }

            }

        });

    });

}

Then:

const correlationId = crypto.randomUUID();

await orderPage.createOrder(correlationId);

const event =
    await waitForEvent("order-created", correlationId);

This avoids picking up events from other tests.


---

Fixture Pattern

You can also expose the messaging layer through a Playwright fixture.

export const test = base.extend({

    kafkaConsumer: async ({}, use) => {

        const consumer = new KafkaConsumer();

        await use(consumer);

    }

});

Then in a test:

test("Verify Order Event", async ({

    kafkaConsumer

}) => {

    const event = await kafkaConsumer.consume(
        "order-created"
    );

});


---

Enterprise Architecture

Playwright Test
       │
       ▼
Page Objects
       │
       ▼
Business Services
       │
       ├───────────────┐
       ▼               ▼
REST APIs       Messaging Layer
                        │
          ┌─────────────┴─────────────┐
          ▼                           ▼
      Kafka                   RabbitMQ/SQS
          │
          ▼
   Event Validator
          │
          ▼
     Playwright Assertions


