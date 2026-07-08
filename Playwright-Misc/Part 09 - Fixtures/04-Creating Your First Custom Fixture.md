Excellent. This is the chapter where everything starts to click.

After this chapter, you'll understand the syntax behind every Playwright fixture. Most people memorize `test.extend()`, but once you understand the **`await use()` pattern**, fixtures become very intuitive.

----------

# 📘 Playwright Fixtures Handbook

# Part 4 – Creating Your First Custom Fixture (Complete Guide)

> **Every Playwright fixture follows the same pattern:**
> 
> **Setup → `await use()` → Teardown**
> 
> If you understand this lifecycle, you understand 90% of Playwright fixtures.

----------

# What is a Custom Fixture?

A custom fixture is simply a reusable dependency that **you create** and Playwright manages.

Instead of using only built-in fixtures like:

```ts
page

context

request

```

you can create your own:

```text
loginPage

dashboardPage

customerApi

database

logger

```

They behave exactly like Playwright's built-in fixtures.

----------

# Creating a Fixture

All custom fixtures are created using:

```ts
test.extend()

```

Think of it as:

```text
Built-in Test

+

My Fixtures

↓

New Test Object

```

----------

# Basic Syntax

```ts
import { test as base } from '@playwright/test';

export const test = base.extend({

});

```

`base` is the original Playwright `test`.

`extend()` adds your custom fixtures.

----------

# Your First Fixture

Let's create a simple logger.

```ts
import { test as base } from '@playwright/test';

export const test = base.extend({

  logger: async ({}, use) => {

    console.log('Setup Logger');

    await use(console);

    console.log('Cleanup Logger');

  }

});

```

Congratulations!

You have created your first custom fixture.

----------

# Understanding the Parameters

Every fixture receives:

```ts
async (

    dependencies,

    use

)

```

There are two parameters.

----------

## First Parameter

Dependencies.

Example

```ts
async (

{ page },

use

)

```

Means:

"My fixture needs `page`."

----------

## Second Parameter

```ts
use

```

This is the most important function.

It hands the created object to the test.

----------

# What Does `await use()` Mean?

This is where most beginners get confused.

Imagine this fixture:

```ts
logger: async ({}, use) => {

    console.log('Setup');

    await use(console);

    console.log('Cleanup');

}

```

Execution order:

```text
Setup

↓

Run Test

↓

Cleanup

```

`await use()` pauses the fixture until the test completes.

----------

# Internal Timeline

```text
Fixture Starts

↓

Setup

↓

await use()

↓

Test Executes

↓

Fixture Resumes

↓

Cleanup

```

This is why teardown code goes **after** `await use()`.

----------

# Visualizing `await use()`

Think of it like opening and closing a door.

```text
Open Door

↓

Guest Enters

↓

Guest Uses Room

↓

Guest Leaves

↓

Clean Room

```

The room is the fixture.

The guest is the test.

----------

# Creating a Page Object Fixture

Suppose we have:

```ts
class LoginPage {

    constructor(page){}

}

```

Fixture

```ts
import { LoginPage } from '../pages/LoginPage';

export const test = base.extend({

  loginPage: async ({ page }, use) => {

    const login = new LoginPage(page);

    await use(login);

  }

});

```

----------

# Using the Fixture

Instead of

```ts
const login =
    new LoginPage(page);

```

Simply write:

```ts
test('Login', async ({

loginPage

}) => {

});

```

Playwright injects it.

----------

# What Happens Internally?

You request:

```text
loginPage

```

Playwright executes:

```text
Create LoginPage

↓

Pass To Test

↓

Destroy Fixture

```

Automatically.

----------

# Setup and Teardown

Suppose we connect to a database.

```ts
database: async ({}, use)=>{

    const db = await connect();

    await use(db);

    await db.close();

}

```

Execution

```text
Connect

↓

Test

↓

Disconnect

```

Beautiful.

----------

# Why Teardown Goes After `use`

Many beginners try:

```ts
await db.close();

await use(db);

```

Wrong.

The database is already closed before the test runs.

Correct

```ts
Connect

↓

await use()

↓

Close

```

----------

# Returning Objects

You don't return fixtures.

Wrong

```ts
return loginPage;

```

Correct

```ts
await use(loginPage);

```

`use()` is how Playwright provides the fixture to the test.

----------

# Multiple Fixtures

```ts
export const test = base.extend({

loginPage:async(...){},

dashboardPage:async(...){},

customerApi:async(...){}

});

```

Tests request whichever fixtures they need.

----------

# Fixture Depends on Another Fixture

Example

```ts
dashboardPage:

async (

{ loginPage },

use

)=>{

```

Dependency graph

```text
page

↓

loginPage

↓

dashboardPage

```

Playwright resolves everything automatically.

----------

# Complete Example

```ts
export const test = base.extend({

loginPage:async(

{ page },

use

)=>{

const login =

new LoginPage(page);

await use(login);

}

});

```

Usage

```ts
test('Login', async ({

loginPage

})=>{

await loginPage.login();

});

```

Very clean.

----------

# Sharing Types

TypeScript version

```ts
type Fixtures = {

loginPage:LoginPage;

};

```

Then

```ts
export const test =

base.extend<Fixtures>({

});

```

Now IntelliSense works perfectly.

----------

# Lifecycle Diagram

```text
Test Starts

↓

Fixture Starts

↓

Setup

↓

Create Object

↓

await use()

↓

Test Runs

↓

Cleanup

↓

Fixture Ends

```

Every fixture follows this lifecycle.

----------

# Multiple Fixtures Timeline

```text
Logger Setup

↓

Database Setup

↓

LoginPage Setup

↓

Test

↓

LoginPage Cleanup

↓

Database Cleanup

↓

Logger Cleanup

```

Setup follows dependency order.

Cleanup happens in reverse order.

We'll explore execution order in detail later.

----------

# Enterprise Example

```ts
export const test = base.extend({

customerApi:async(

{ request },

use

)=>{

const api =

new CustomerApi(request);

await use(api);

}

});

```

Test

```ts
test('Customer', async ({

customerApi

})=>{

await customerApi.create();

});

```

No manual construction.

----------

# What Can a Fixture Create?

Anything.

```text
Page Objects

API Clients

Database

Redis

Kafka

Logger

Random Data

Email Client

WebSocket

GraphQL Client

```

If it has setup and teardown, it can be a fixture.

----------

# Common Mistakes

## ❌ Returning Instead of `use`

Wrong

```ts
return logger;

```

Correct

```ts
await use(logger);

```

----------

## ❌ Cleanup Before `use`

Wrong

```ts
close();

await use();

```

----------

Correct

```text
Setup

↓

use

↓

Cleanup

```

----------

## ❌ Forgetting `await`

Wrong

```ts
use(logger);

```

Correct

```ts
await use(logger);

```

----------

## ❌ Creating Objects Inside Every Test

Instead of

```ts
new LoginPage(page);

```

Move object creation into a fixture.

----------

# Interview Questions

### Q1. Why do fixtures use `await use()` instead of `return`?

Because `await use()` allows Playwright to pause the fixture while the test executes, then resume it afterward to perform cleanup.

----------

### Q2. What is the lifecycle of a fixture?

```text
Setup

↓

Create Object

↓

await use()

↓

Test

↓

Cleanup

```

----------

### Q3. Can fixtures depend on other fixtures?

Yes.

Example:

```text
page

↓

loginPage

↓

checkoutPage

```

Playwright resolves the dependency chain automatically.

----------

### Q4. Can a fixture clean up resources?

Yes.

Cleanup code goes after `await use()`.

----------

### Q5. What does the first parameter represent?

Dependencies.

Example:

```ts
{ page }

```

means the fixture depends on the built-in `page` fixture.

----------

### Q6. What does the second parameter represent?

The `use` callback, which hands the created fixture value to the test and pauses until the test completes.

----------

# Best Practices

-   Use `test.extend()` to build reusable dependencies.
    
-   Think of every fixture as **Setup → Test → Cleanup**.
    
-   Put cleanup logic after `await use()`.
    
-   Inject dependencies through the first parameter rather than creating them manually.
    
-   Use TypeScript generics with `base.extend<Fixtures>()` for strong typing and IntelliSense.
    
-   Keep each fixture focused on a single responsibility.
    

----------

# ⭐ Enterprise Fixture Example

```ts
import { test as base } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { CustomerApi } from '../api/CustomerApi';

type Fixtures = {
  loginPage: LoginPage;
  customerApi: CustomerApi;
};

export const test = base.extend<Fixtures>({

  loginPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await use(loginPage);
  },

  customerApi: async ({ request }, use) => {
    const customerApi = new CustomerApi(request);
    await use(customerApi);
  }

});

export { expect } from '@playwright/test';

```

Now every test can simply write:

```ts
import { test, expect } from '../fixtures/base.fixture';

test('Create Customer', async ({ loginPage, customerApi }) => {
  await loginPage.login();
  await customerApi.createCustomer();
});

```

The test contains only business logic, while object creation and lifecycle management remain centralized in the fixture layer.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMzQxNTIzMDcyXX0=
-->