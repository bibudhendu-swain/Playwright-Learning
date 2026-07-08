Excellent. This is another reporting-focused feature introduced to improve the developer experience.

By default, Playwright displays the **fixture property name** in reports. That works for small projects, but in enterprise frameworks fixture names are often technical:

-   `customerApiV2`
    
-   `dbPool`
    
-   `authStorageFixture`
    
-   `featureFlagManager`
    

These names make sense to framework developers but not necessarily to testers or stakeholders.

**Custom Fixture Titles** solve that problem.

----------

# 📘 Playwright Fixtures Handbook

# Part 14 – Custom Fixture Title (Complete Guide)

> **Fixture names are for developers.**
> 
> **Fixture titles are for humans reading reports.**

----------

# Why Do We Need Custom Titles?

Suppose your fixture is named:

```text
customerApiV2Internal

```

Perfectly fine for code.

But in the HTML report you'll see:

```text
customerApiV2Internal

```

Not very friendly.

Instead you'd rather see:

```text
Customer API

```

----------

# Default Behavior

Fixture

```ts
customerApiV2Internal

```

Report

```text
customerApiV2Internal

```

----------

# With Custom Title

Fixture

```ts
customerApiV2Internal

```

Title

```text
Customer API

```

Report

```text
Customer API

```

Much nicer.

----------

# Syntax

Titles are defined in the fixture options.

```ts
customerApi: [

  async ({ request }, use) => {

    const api = new CustomerApi(request);

    await use(api);

  },

  {
    title: 'Customer API'
  }

]

```

Notice:

The fixture name stays:

```text
customerApi

```

Only the displayed title changes.

----------

# Another Example

Internal fixture

```text
dbPool

```

Title

```text
Database Connection Pool

```

Now reports become self-explanatory.

----------

# Code vs Report

## Code

```ts
test('Create Customer', async ({

customerApi

}) => {

});

```

----------

## Report

```text
Customer API

```

Developers keep short names.

Reports stay readable.

----------

# Multiple Titles

```text
customerApi

↓

Customer API



paymentApi

↓

Payment API



dbPool

↓

Database Pool

```

Every fixture can have its own title.

----------

# Enterprise Example

Internal names

```text
cfg

db

msgBus

secMgr

```

Titles

```text
Configuration

Database

Message Bus

Secret Manager

```

Reports become understandable even for people unfamiliar with the codebase.

----------

# Combining with Box Fixtures

Very common.

Example

```text
Logger

↓

box:true

↓

title:"Execution Logger"

```

Result

```text
Hidden

```

The title exists but won't normally appear because the fixture is boxed.

----------

# Combining with Automatic Fixtures

Example

```text
Execution Timer

↓

auto:true

↓

title:"Performance Timer"

```

Runs automatically.

Shows a friendly name in reports (unless also boxed).

----------

# Example

```ts
database: [

  async ({}, use) => {

    const db = await connect();

    await use(db);

    await db.close();

  },

  {

    title: 'Database Connection'

  }

]

```

The code still requests:

```ts
database

```

The report displays:

```text
Database Connection

```

----------

# Large Framework Example

Without titles

```text
cfg

↓

dbPool

↓

ffMgr

↓

authSvc

↓

checkoutPg

```

Report isn't very expressive.

----------

With titles

```text
Configuration

↓

Database Pool

↓

Feature Flags

↓

Authentication

↓

Checkout Page

```

Immediately understandable.

----------

# Naming Strategy

## Code

Keep names:

-   Short
    
-   Consistent
    
-   Developer-friendly
    

Example

```text
db

logger

cfg

cartPage

```

----------

## Report

Use:

-   Full words
    
-   Business language
    
-   Clear descriptions
    

Example

```text
Database

Execution Logger

Configuration

Shopping Cart Page

```

----------

# Typical Enterprise Mapping

Code Name

Report Title

`cfg`

Configuration

`dbPool`

Database Connection Pool

`logger`

Execution Logger

`customerApi`

Customer API

`cartPage`

Shopping Cart Page

----------

# Does It Change Execution?

No.

Titles only affect presentation.

They do **not** change:

-   Setup
    
-   Cleanup
    
-   Scope
    
-   Timeout
    
-   Dependencies
    

----------

# Code Example

```ts
logger: [

  async ({}, use) => {

    const logger = new Logger();

    await use(logger);

  },

  {

    title: 'Execution Logger'

  }

]

```

Execution remains exactly the same.

----------

# Reports Before

```text
logger

↓

cfg

↓

dbPool

```

----------

# Reports After

```text
Execution Logger

↓

Configuration

↓

Database Pool

```

Much cleaner.

----------

# Enterprise Reporting

Imagine an automation report sent to:

-   Developers
    
-   QA
    
-   Product Owner
    
-   Business Analyst
    

Which is easier?

Option A

```text
custApi

cfg

dbPool

```

Option B

```text
Customer API

Configuration

Database Connection Pool

```

The second is much more accessible.

----------

# Good Titles

✅ Authentication

✅ Customer API

✅ Database Connection

✅ Feature Flag Manager

✅ Test Logger

----------

# Poor Titles

❌ API1

❌ Temp

❌ Helper

❌ Fixture123

Titles should describe purpose, not implementation.

----------

# Combining Features

One fixture can use:

```text
title

+

timeout

+

scope

+

auto

+

box

```

Example

```ts
logger: [

  async ({}, use) => {

    await use(new Logger());

  },

  {

    scope: 'worker',

    auto: true,

    box: true,

    title: 'Execution Logger'

  }

]

```

Very common in enterprise frameworks.

----------

# Common Mistakes

## ❌ Long Technical Titles

Bad

```text
CustomerApiVersion2InternalImplementation

```

Prefer

```text
Customer API

```

----------

## ❌ Different Naming Styles

Avoid

```text
Customer API

database pool

LOGGER

```

Be consistent.

----------

## ❌ Using Titles as Variable Names

Code should still use:

```ts
customerApi

```

Titles are only for reporting.

----------

## ❌ Expecting Functional Changes

Changing the title does not affect execution.

----------

# Recommended Naming Style

Infrastructure

```text
Execution Logger

Configuration

Database Pool

Secrets Manager

```

Business

```text
Login Page

Checkout Page

Customer API

Payment API

```

----------

# Interview Questions

### Q1. What is a custom fixture title?

A human-readable name shown in Playwright reports instead of the fixture's property name.

----------

### Q2. Does changing the title affect execution?

No.

It only changes how the fixture is displayed in reports.

----------

### Q3. Should code use the title?

No.

Tests still use the fixture's property name.

----------

### Q4. Can a fixture have both a title and be boxed?

Yes.

The title exists, but if the fixture is boxed it is generally hidden from reports.

----------

### Q5. Can titles be combined with worker fixtures?

Yes.

Titles work independently of fixture scope.

----------

### Q6. Why use titles?

To improve report readability and make reports understandable for both technical and non-technical audiences.

----------

# Best Practices

-   Keep fixture variable names concise and developer-friendly.
    
-   Use descriptive titles for reports.
    
-   Maintain consistent naming across all fixtures.
    
-   Combine titles with `box: true` thoughtfully.
    
-   Don't duplicate unnecessary implementation details in titles.
    
-   Treat titles as documentation for report readers.
    

----------

# ⭐ Enterprise Example

Imagine a large enterprise framework.

```text
                    Framework
                         │
        ┌────────────────┼────────────────┐
        │                │                │
      cfg             dbPool         customerApi
        │                │                │
        ▼                ▼                ▼
 Configuration   Database Connection   Customer API
        │                │                │
        └────────────────┼────────────────┘
                         │
                    HTML Report

```

Developers continue writing:

```ts
test('Checkout', async ({
  customerApi,
  checkoutPage
}) => {

  await customerApi.createCustomer();

  await checkoutPage.placeOrder();

});

```

But the report presents friendly labels that are much easier to interpret.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTI0MjI4MzMyMF19
-->