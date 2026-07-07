Excellent. This is arguably the **most important chapter** in the entire Fixtures handbook.

If someone truly understands **fixture execution order**, they understand how Playwright's dependency injection engine works internally.

This is also one of the most common sources of confusion:

-   "Why did this fixture execute first?"
    
-   "Why wasn't my fixture created?"
    
-   "Why is cleanup happening in reverse order?"
    
-   "Why didn't Playwright create my fixture at all?"
    

By the end of this chapter, you'll be able to predict exactly how Playwright will execute any fixture graph.

----------

# 📘 Playwright Fixtures Handbook

# Part 11 – Fixture Execution Order (Complete Guide)

> **Playwright doesn't execute fixtures in the order you write them.**
> 
> **It executes them based on the dependency graph.**

This single sentence explains almost all fixture behavior.

----------

# The Three Rules

Every Playwright fixture follows these rules:

### Rule 1

Create only the fixtures that are requested.

----------

### Rule 2

Resolve dependencies before creating a fixture.

----------

### Rule 3

Destroy fixtures in the reverse order of creation.

----------

Everything else follows these rules.

----------

# Simple Example

Test

```ts
test('Login', async ({

loginPage

})=>{

});

```

Dependencies

```text
loginPage

↓

page

↓

context

↓

browser

```

Execution

```text
browser

↓

context

↓

page

↓

loginPage

↓

Test

```

Cleanup

```text
loginPage

↓

page

↓

context

↓

browser

```

Reverse order.

----------

# Why Reverse Cleanup?

Think about building a house.

Setup

```text
Foundation

↓

Walls

↓

Roof

```

Demolition

```text
Roof

↓

Walls

↓

Foundation

```

Not

Foundation first.

Same principle.

----------

# Example

Fixture

```text
CustomerApi

```

depends on

```text
request

```

Execution

```text
request

↓

CustomerApi

↓

Test

```

Cleanup

```text
CustomerApi

↓

request

```

----------

# Dependency Graph

Suppose

```text
CheckoutPage

↓

CartPage

↓

ProductPage

↓

LoginPage

↓

Page

```

Playwright creates

```text
Browser

↓

Context

↓

Page

↓

LoginPage

↓

ProductPage

↓

CartPage

↓

CheckoutPage

```

Not the order written in the file.

----------

# Example Code

```ts
checkoutPage

↓

productPage

↓

page

```

Even if the fixture definitions appear in another order, Playwright resolves them by dependency.

----------

# Lazy Initialization

Suppose framework has:

```text
loginPage

dashboardPage

checkoutPage

logger

database

```

Test requests

```ts
test('Login', async ({

loginPage

})=>{

});

```

Created

```text
loginPage

```

Not created

```text
dashboardPage

database

logger

```

Unused fixtures never execute.

----------

# Multiple Fixtures

Test

```ts
test('Checkout', async ({

checkoutPage,

customerApi

})=>{

});

```

Dependency graph

```text
checkoutPage

↓

page

↓

context

↓

browser



customerApi

↓

request

```

Execution

```text
browser

↓

context

↓

page

↓

checkoutPage



request

↓

customerApi

```

Independent dependency trees are resolved automatically.

----------

# Automatic Fixtures

Suppose

```text
logger

↓

auto:true

```

Execution

```text
Logger

↓

Requested Fixtures

↓

Test

```

Automatic fixtures always participate in execution even when not requested.

----------

# Worker Fixtures

Worker

```text
database

```

Test

```text
loginPage

```

Execution

```text
Worker Starts

↓

Database

↓

Test Starts

↓

Page

↓

LoginPage

↓

Test

```

Database is already available before the test begins.

----------

# Worker Cleanup

```text
All Tests Done

↓

LoginPage Destroyed

↓

Database Destroyed

↓

Worker Ends

```

Worker fixtures outlive test fixtures.

----------

# Complex Enterprise Graph

```text
Logger (Worker)

Database (Worker)

↓

page

↓

loginPage

↓

dashboardPage

↓

checkoutPage

↓

Test

```

Execution order

```text
Logger

↓

Database

↓

Browser

↓

Context

↓

Page

↓

LoginPage

↓

DashboardPage

↓

CheckoutPage

↓

Test

```

Cleanup

```text
CheckoutPage

↓

DashboardPage

↓

LoginPage

↓

Page

↓

Context

↓

Browser

↓

Database

↓

Logger

```

Perfect reverse order.

----------

# Setup vs Cleanup

Setup

```text
Parent

↓

Child

```

Cleanup

```text
Child

↓

Parent

```

Always.

----------

# Diamond Dependency

Suppose

```text
CheckoutPage

↙       ↘

ProductPage   CartPage

      ↓

      Page

```

Playwright creates:

```text
Page

```

Only once.

Dependencies are shared within the test.

----------

# Visual Graph

```text
Browser
   │
Context
   │
  Page
 ┌─┴──────────────┐
 │                │
LoginPage    CustomerApi
 │
CheckoutPage

```

Playwright resolves the graph automatically.

----------

# Multiple Tests

Test 1

```text
Page

↓

LoginPage

```

Destroyed.

----------

Test 2

Fresh

```text
Page

↓

LoginPage

```

Nothing shared.

----------

# Worker Timeline

```text
Worker Starts

↓

Logger

↓

Test 1

↓

Test 2

↓

Test 3

↓

Logger Destroyed

```

One logger.

Many tests.

----------

# What Happens If One Fixture Fails?

Suppose

```text
Database

```

fails.

Execution

```text
Database

↓

Exception

↓

Dependent Fixtures Not Created

↓

Test Does Not Run

```

Playwright stops building that dependency chain.

----------

# Partial Cleanup

Suppose

```text
Logger

↓

Database

↓

LoginPage

```

Database fails.

Cleanup

```text
Logger Cleanup

```

Only fixtures that were successfully created are cleaned up.

----------

# Common Misconceptions

## ❌ Fixtures Execute Top to Bottom

Wrong.

Dependency graph determines order.

----------

## ❌ Cleanup Happens Top to Bottom

Wrong.

Cleanup is always reverse dependency order.

----------

## ❌ Every Fixture Executes

Wrong.

Unused fixtures are never created (unless `auto: true`).

----------

## ❌ Worker Fixtures Execute Per Test

Wrong.

They execute once per worker.

----------

# Dependency Resolution Algorithm (Conceptually)

```text
Read Requested Fixtures

↓

Resolve Dependencies

↓

Remove Duplicates

↓

Create Parents

↓

Create Children

↓

Execute Test

↓

Destroy Children

↓

Destroy Parents

```

This is effectively what Playwright does internally.

----------

# Execution Example

Test

```ts
test('Example', async ({

checkoutPage,

logger

})=>{

});

```

Execution

```text
Logger

↓

Browser

↓

Context

↓

Page

↓

CheckoutPage

↓

Test

```

Cleanup

```text
CheckoutPage

↓

Page

↓

Context

↓

Browser

↓

Logger

```

----------

# Interview Questions

### Q1. Does Playwright execute fixtures in declaration order?

No.

It executes them according to the dependency graph.

----------

### Q2. What is the fixture execution order?

Dependencies first.

Children later.

Cleanup happens in reverse.

----------

### Q3. When are unused fixtures executed?

They aren't, unless configured with:

```text
auto:true

```

----------

### Q4. What happens if a dependency fails?

Dependent fixtures are not created, the test does not execute, and Playwright cleans up any fixtures that were already initialized.

----------

### Q5. Why does cleanup happen in reverse order?

To ensure child resources are released before their parent resources, preventing invalid references and resource leaks.

----------

### Q6. Does Playwright create duplicate dependencies?

No.

If multiple fixtures depend on the same fixture (such as `page`), Playwright creates it only once per scope.

----------

# Best Practices

-   Think in terms of dependency graphs, not file order.
    
-   Keep fixture dependencies simple and acyclic.
    
-   Design parent fixtures to outlive their children naturally.
    
-   Avoid deep dependency chains unless they add real value.
    
-   Use `auto: true` only for true cross-cutting concerns.
    
-   Remember that setup flows from parent → child, while teardown flows from child → parent.
    

----------

# ⭐ Enterprise Execution Diagram

Imagine this enterprise framework:

```text
                    Worker Starts
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
    Logger (Worker)   Config (Worker)   DB Pool (Worker)
                          │
                     Test Starts
                          │
                       Browser
                          │
                       Context
                          │
                         Page
                  ┌───────┼────────┐
                  │                │
             LoginPage      CustomerApi
                  │
            CheckoutPage
                  │
                Execute Test
                  │
            CheckoutPage Cleanup
                  │
            LoginPage Cleanup
                  │
               Page Cleanup
                  │
            Context Cleanup
                  │
            Browser Cleanup
                  │
         DB Pool Cleanup (Worker End)
                  │
         Config Cleanup (Worker End)
                  │
         Logger Cleanup (Worker End)

```

This diagram captures the fundamental principles of Playwright's fixture engine:

-   Worker fixtures are initialized once and survive multiple tests.
    
-   Test fixtures are created only when needed.
    
-   Dependencies are resolved automatically.
    
-   Shared dependencies are created only once.
    
-   Cleanup always occurs in the reverse dependency order.
    

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTEyNzk0NDUwOF19
-->