Once you understand the design philosophy, you'll stop writing Playwright like Selenium and start writing **idiomatic Playwright**.

----------

# Part 1 – Introduction to Playwright

# Chapter 5 – Playwright Design Philosophy

----------

# Introduction

When engineers first migrate to Playwright, many continue writing automation using the same patterns they used in Selenium.

Examples include:

-   Creating BaseTest classes
    
-   Wrapping every Playwright API
    
-   Using explicit waits everywhere
    
-   Treating Locators like WebElements
    
-   Sharing browser sessions across tests
    

Although these approaches may work, they ignore many of the architectural decisions that make Playwright powerful.

Playwright was designed around a set of engineering principles that prioritize reliability, isolation, simplicity, and developer productivity.

Understanding these principles helps you write cleaner, faster, and more maintainable automation.

----------

# Design Philosophy Overview

Playwright is built around several core principles.

```text
Reliability

↓

Isolation

↓

Simplicity

↓

Developer Experience

↓

Performance

```

Almost every Playwright feature exists because of one or more of these principles.

----------

# Philosophy 1 – Test Isolation by Default

One of Playwright's most important principles is:

> Every test should be independent.

Instead of sharing browser sessions,

Playwright encourages:

```text
Browser

↓

Context A

↓

Test A

------------------

Browser

↓

Context B

↓

Test B

```

Each test starts with a clean environment.

Benefits:

-   No shared cookies
    
-   No shared local storage
    
-   Parallel-safe execution
    
-   Easier debugging
    
-   Fewer flaky tests
    

----------

# Why BrowserContext Instead of Browser?

Launching a browser is expensive.

Launching a BrowserContext is inexpensive.

Instead of

```text
Test

↓

New Browser

↓

Run

↓

Close

```

Playwright prefers

```text
Browser

↓

Context

↓

Run Test

```

This improves execution speed while preserving isolation.

----------

# Philosophy 2 – Web-First Automation

Older frameworks often automate HTML elements.

Playwright automates the **user experience**.

Instead of

```typescript
page.locator("#btn123")

```

prefer

```typescript
page.getByRole(
"button",
{ name: "Login" }
)

```

This mirrors how users interact with the application.

----------

# Why Semantic Locators?

Consider

```typescript
#submit-btn

```

versus

```typescript
getByRole(
"button",
{ name: "Submit" }
)

```

The second version:

-   Improves readability
    
-   Encourages accessibility
    
-   Survives many UI refactorings
    
-   Better reflects user intent
    

----------

# Philosophy 3 – Lazy Locators

One of Playwright's biggest innovations.

Creating a locator

```typescript
const loginButton =
page.getByRole(
"button",
{ name:"Login" }
);

```

does **not**

search the DOM immediately.

Instead

```text
Locator

↓

Wait

↓

Find

↓

Action

```

The element is resolved only when required.

----------

# Why Not Store Elements?

Older frameworks often store element references.

```text
Find Element

↓

Store

↓

DOM Changes

↓

Reference Invalid

```

This leads to stale element problems.

Playwright stores **the search strategy**, not the element.

----------

# Philosophy 4 – Auto-Waiting

Instead of forcing engineers to write synchronization code,

Playwright automatically waits.

Bad

```typescript
waitForTimeout(5000);

```

Better

```typescript
await loginButton.click();

```

The framework waits until the action is safe.

----------

# Philosophy 5 – Assertions Should Wait

Older frameworks

```text
Check

↓

Fail

```

Playwright

```text
Retry

↓

Retry

↓

Retry

↓

Pass

```

Web-first assertions retry until timeout.

Example

```typescript
await expect(
loginButton
).toBeVisible();

```

No manual polling required.

----------

# Philosophy 6 – Composition over Inheritance

Instead of

```text
BasePage

↓

AdminPage

↓

OrderPage

↓

CheckoutPage

```

Prefer

```text
CheckoutPage

↓

Header

↓

Navigation

↓

Cart

↓

Footer

```

This aligns with modern frontend frameworks like React and Angular.

----------

# Philosophy 7 – Fixtures Instead of Base Classes

Many Selenium frameworks rely on

```text
BaseTest

```

Playwright encourages

```text
Fixtures

↓

Dependency Injection

↓

Tests

```

Benefits:

-   Cleaner tests
    
-   Explicit dependencies
    
-   Automatic lifecycle management
    
-   Better scalability
    

----------

# Philosophy 8 – BrowserContext Instead of ThreadLocal

Selenium frameworks often use

```text
ThreadLocal<WebDriver>

```

Playwright replaces this with

```text
Worker

↓

Browser

↓

Context

↓

Page

```

No manual ThreadLocal management required.

----------

# Philosophy 9 – Native Capabilities over Wrappers

Avoid creating wrappers like

```typescript
ClickUtil.click(locator);

```

when

```typescript
await locator.click();

```

already provides:

-   Waiting
    
-   Retry
    
-   Visibility checks
    
-   Stability checks
    

Only wrap Playwright APIs when adding meaningful functionality.

----------

# Philosophy 10 – Prefer Intent over Implementation

Bad

```typescript
locator(".btn-green")

```

Better

```typescript
getByRole(
"button",
{
name:"Checkout"
}
)

```

The code describes **what** the user is doing rather than **how** the UI is implemented.

----------

# Philosophy 11 – Framework Should Be Small

Playwright intentionally includes many built-in capabilities:

-   Test Runner
    
-   Assertions
    
-   Mocking
    
-   API Testing
    
-   Reporting
    
-   Tracing
    
-   Screenshots
    

This reduces the need for numerous third-party libraries.

----------

# Philosophy 12 – Parallel by Design

Parallel execution was considered from the beginning.

```text
Worker 1

↓

Context

↓

Tests

----------------

Worker 2

↓

Context

↓

Tests

```

Parallel execution is not an afterthought.

----------

# Philosophy 13 – Debugging Should Be Easy

Playwright includes:

```text
Trace

Inspector

UI Mode

HTML Report

Videos

Screenshots

```

The framework assumes failures will happen and provides tools to investigate them efficiently.

----------

# Philosophy 14 – One Framework for Multiple Needs

Playwright combines:

```text
UI Testing

↓

API Testing

↓

Network Mocking

↓

Browser Mocking

↓

Authentication

```

Teams can often reduce the number of tools they need to maintain.

----------

# Philosophy 15 – Secure by Default

Features such as BrowserContext isolation help prevent accidental sharing of:

-   Cookies
    
-   Authentication state
    
-   Storage
    

This is particularly important for parallel execution.

----------

# Philosophy 16 – Predictable Behavior

Playwright avoids hidden global state.

Resources have well-defined lifecycles.

```text
Browser

↓

Context

↓

Page

↓

Locator

```

Predictability improves debugging and maintenance.

----------

# What This Means for Selenium Users

Many familiar patterns should be reconsidered.

Selenium Habit

Playwright Approach

`Thread.sleep()`

Auto-waiting & web-first assertions

`WebDriverWait` everywhere

Built-in waiting where possible

`WebElement`

`Locator`

`ThreadLocal<WebDriver>`

BrowserContext + Workers

`BaseTest`

Fixtures

Deep inheritance

Composition

PageFactory

Native locators

Migration involves changing architectural thinking, not just syntax.

----------

# Common Mistakes

### ❌ Writing Playwright Like Selenium

Avoid recreating Selenium patterns unnecessarily.

----------

### ❌ Wrapping Every API

Don't introduce wrappers unless they provide additional value such as logging, retry policies, or standardized diagnostics.

----------

### ❌ Ignoring BrowserContext

Sharing state between unrelated tests defeats one of Playwright's biggest strengths.

----------

### ❌ Overusing XPath

Prefer semantic, accessibility-based locators whenever practical.

----------

### ❌ Fighting the Framework

If Playwright already provides a capability, understand it before building a custom alternative.

----------

# Best Practices

-   Trust Playwright's built-in waiting mechanisms before adding explicit waits.
    
-   Design tests to be isolated and parallel-safe.
    
-   Prefer semantic locators that reflect user intent.
    
-   Use fixtures for dependency management instead of base classes.
    
-   Favor composition over inheritance.
    
-   Leverage built-in tooling before introducing external libraries.
    
-   Learn the rationale behind Playwright's APIs rather than treating them as direct replacements for Selenium methods.
    

----------

# Interview Questions

### Q1. Why does Playwright prefer `Locator` over `ElementHandle`?

Because locators are lazy, automatically retry actions and assertions, and reduce issues caused by dynamic DOM updates.

----------

### Q2. Why are BrowserContexts central to Playwright's design?

They provide lightweight, isolated browser sessions that support reliable parallel execution and prevent state leakage between tests.

----------

### Q3. Why does Playwright emphasize semantic locators?

Semantic locators are more readable, align with accessibility best practices, and are generally more resilient to UI implementation changes.

----------

### Q4. Why are fixtures preferred over `BaseTest`?

Fixtures provide explicit dependency injection, automatic lifecycle management, and better scalability than inheritance-based test setup.

----------

### Q5. Should every Playwright API be wrapped in a custom framework method?

No. Wrappers should only be introduced when they add meaningful functionality such as logging, retry logic, diagnostics, or business-specific abstractions.

----------

# Summary

Playwright's design philosophy is centered on reliability, isolation, simplicity, and developer productivity. Features such as BrowserContext isolation, lazy locators, automatic waiting, semantic selectors, dependency injection through fixtures, and built-in tooling are all expressions of these principles. By understanding _why_ Playwright was designed this way, engineers can write automation that is more idiomatic, maintainable, and aligned with the framework's strengths rather than carrying over patterns from older automation frameworks.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIyNDQzNTg1MV19
-->