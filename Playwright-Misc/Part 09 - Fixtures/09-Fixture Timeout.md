Excellent. This is a topic that many Playwright users don't even know exists until a long-running fixture starts failing.

A common reaction is:

> "My database setup takes 90 seconds, so I'll increase the test timeout."

That's usually the wrong solution.

**Fixture timeouts** allow you to give slow fixtures more time **without slowing down every test**.

----------

# 📘 Playwright Fixtures Handbook

# Part 9 – Fixture Timeout (Complete Guide)

> **Tests have timeouts.**
> 
> **Fixtures can also have their own timeouts.**

This allows expensive setup operations to have a different timeout than the actual test.

----------

# Why Do Fixture Timeouts Exist?

Imagine this test:

```ts
test('Create Customer', async ({

database

}) => {

});

```

The fixture:

```text
Database Connection

```

takes

```text
60 Seconds

```

The test itself takes:

```text
3 Seconds

```

Should you set:

```text
Test Timeout = 60 Seconds?

```

No.

The long-running part is the fixture—not the test.

----------

# Without Fixture Timeout

```text
Test Timeout

↓

60 Seconds

↓

Everything Shares It

```

Every test now has a huge timeout.

----------

# With Fixture Timeout

```text
Fixture Timeout

↓

60 Seconds



Test Timeout

↓

10 Seconds

```

Much better.

----------

# Default Behavior

Normally:

```text
Fixture

↓

Uses

↓

Test Timeout

```

If the test timeout is:

```ts
timeout:30000

```

The fixture also has:

```text
30 Seconds

```

unless overridden.

----------

# Setting Fixture Timeout

Syntax

```ts
database:[

async({}, use)=>{

},

{

timeout:60000

}

]

```

Notice:

Options object.

----------

# Complete Example

```ts
database:[

async({}, use)=>{

const db=

await connect();

await use(db);

await db.close();

},

{

timeout:60000

}

]

```

Now:

Database setup

↓

Can take

```text
60 Seconds

```

----------

# Execution Timeline

```text
Database Setup

↓

45 Seconds

↓

Run Test

↓

Cleanup

```

Valid.

----------

Without fixture timeout

```text
30 Second Test Timeout

↓

Database

↓

Timeout

```

Failure.

----------

# Test Timeout vs Fixture Timeout

## Test

```ts
test.setTimeout(10000);

```

Controls

```text
Entire Test

```

----------

Fixture

```ts
timeout:60000

```

Controls

```text
Fixture Setup

```

Only.

----------

# Example – Slow Authentication

Suppose authentication API takes

```text
40 Seconds

```

Fixture

```ts
login:[

async({}, use)=>{

await login();

await use();

},

{

timeout:45000

}

]

```

Test still finishes in

```text
5 Seconds

```

----------

# Example – Database Restore

```text
Restore Backup

↓

90 Seconds

```

Fixture

```ts
database:[

async({}, use)=>{

await restore();

await use();

},

{

timeout:120000

}

]

```

Only database setup gets extra time.

----------

# Worker Fixture Timeout

Example

```ts
database:[

async({}, use)=>{

...

},

{

scope:'worker',

timeout:120000

}

]

```

Worker startup can take:

```text
2 Minutes

```

Then all tests reuse it.

----------

# Enterprise Example

```text
Worker Starts

↓

Connect Oracle

↓

Connect Kafka

↓

Connect Redis

↓

Run 300 Tests

```

Infrastructure may legitimately require a longer timeout.

----------

# Fixture Timeout Does NOT Affect

```text
Test Logic

```

If your test hangs for

```text
10 Minutes

```

Fixture timeout won't help.

Test timeout still applies.

----------

# Relationship

```text
Fixture Setup

↓

Fixture Timeout



Test

↓

Test Timeout

```

Two separate concepts.

----------

# Multiple Fixtures

```text
Database

60 Seconds



Logger

5 Seconds



API

20 Seconds

```

Each fixture can have its own timeout.

----------

# Enterprise Example

```ts
database:[

async(...){

},

{

timeout:120000

}

],

redis:[

async(...){

},

{

timeout:30000

}

]

```

Different infrastructure.

Different limits.

----------

# Timeout During Cleanup

Suppose

```text
Database Close

```

takes time.

Remember:

Cleanup is also part of the fixture lifecycle.

```text
Setup

↓

Test

↓

Cleanup

```

The fixture timeout covers the fixture's execution, including teardown.

----------

# Slow External Systems

Good candidates

```text
Oracle

SQL Server

SAP

Salesforce

Kafka

Redis

SFTP

```

They often justify longer fixture timeouts.

----------

# Not Needed For

```text
Page Object

Logger

Simple Helper

Utility Class

```

These initialize almost instantly.

----------

# Enterprise Example

```text
Environment Validation

↓

30 Seconds

↓

Database

↓

60 Seconds

↓

API Token

↓

15 Seconds

↓

Run Tests

```

Each fixture independently configured.

----------

# Common Mistakes

## ❌ Increasing Test Timeout

Instead of

```ts
timeout:120000

```

for everything,

set a timeout on the slow fixture.

----------

## ❌ Huge Fixture Timeout Everywhere

Don't give every fixture:

```text
10 Minutes

```

Only slow fixtures need custom values.

----------

## ❌ Ignoring Slow Fixtures

If setup consistently takes:

```text
90 Seconds

```

Investigate the root cause instead of only increasing the timeout.

----------

## ❌ Forgetting Worker Fixtures

Worker fixtures often initialize infrastructure.

They're common candidates for custom timeouts.

----------

# Decision Guide

Resource

Custom Timeout?

Page Object

❌

Logger

❌

API Client

Usually No

Authentication Setup

Sometimes

Database Connection

Often

Database Restore

Yes

Docker Startup

Yes

External Services

Usually

----------

# Timeout Hierarchy

```text
Global Timeout

↓

Test Timeout

↓

Fixture Timeout

↓

Action Timeout

↓

Expect Timeout

```

Each serves a different purpose.

----------

# Real Example

Suppose

```text
Test

↓

5 Seconds

```

Database

↓

45 Seconds

Configuration

```ts
test.setTimeout(10000);

database:{

timeout:60000

}

```

Perfect.

----------

# Interview Questions

### Q1. What is a fixture timeout?

A fixture timeout specifies how long Playwright will allow a fixture to complete its setup (and lifecycle) before considering it timed out.

----------

### Q2. Why not just increase the test timeout?

Because the delay may be caused only by fixture initialization. Increasing the test timeout unnecessarily slows failure detection for all test logic.

----------

### Q3. Can worker fixtures have timeouts?

Yes.

This is common for infrastructure initialization such as database connections or service startup.

----------

### Q4. Can every fixture have a different timeout?

Yes.

Each fixture can define its own timeout independently.

----------

### Q5. Should page objects have custom timeouts?

Usually not.

They initialize quickly.

----------

### Q6. When should you use fixture timeouts?

When fixture setup or teardown involves slow external resources, such as databases, authentication, or infrastructure services.

----------

# Best Practices

-   Keep test timeouts focused on the business scenario.
    
-   Increase fixture timeouts only for genuinely slow setup or teardown.
    
-   Use worker-scoped fixtures for expensive infrastructure and combine them with appropriate fixture timeouts.
    
-   Investigate slow fixtures before simply extending their timeout.
    
-   Don't assign large timeouts to lightweight fixtures.
    
-   Treat fixture timeout tuning as a performance optimization exercise, not just a workaround.
    

----------

# ⭐ Enterprise Example

```ts
import { test as base } from '@playwright/test';

export const test = base.extend({

  database: [

    async ({}, use) => {

      const db = await connectToOracle();

      await use(db);

      await db.close();

    },

    {
      scope: 'worker',
      timeout: 120000
    }

  ],

  authToken: [

    async ({ request }, use) => {

      const token = await generateToken();

      await use(token);

    },

    {
      timeout: 30000
    }

  ]

});

```

Execution:

```text
Worker Starts
        │
        ▼
Connect Oracle (up to 120s)
        │
        ▼
Test Starts
        │
        ▼
Generate Auth Token (up to 30s)
        │
        ▼
Execute Test (normal test timeout)
        │
        ▼
Cleanup

```

Each phase has an appropriate timeout based on its responsibility.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTI3MzI4NDU3N119
-->