# Part 1 – Introduction to Playwright

# Chapter 3 – Playwright Architecture

----------

# Introduction

Every software framework has an underlying architecture that defines how its components interact.

Understanding Playwright's architecture helps answer questions such as:

-   Why do we need BrowserContext?
    
-   Why doesn't Playwright use WebDriver?
    
-   Why are tests isolated?
    
-   Why is Playwright faster than many traditional automation tools?
    
-   How does parallel execution work?
    

Rather than memorizing APIs, this chapter builds a mental model of how Playwright operates internally.

----------

# High-Level Architecture

At a high level, Playwright sits between your automation code and the browser.

```text
Your Test

↓

Playwright API

↓

Browser Engine

↓

Web Application

```

Unlike older browser automation solutions, Playwright communicates directly with the browser engine instead of relying on an external browser driver.

----------

# The Complete Architecture

A Playwright test usually follows this hierarchy.

```text
Playwright

↓

Browser

↓

Browser Context

↓

Page

↓

Frame

↓

Locator

↓

Element

```

Every Playwright automation scenario is built upon these core objects.

----------

# Understanding the Hierarchy

Think of it like a real office building.

```text
Building

↓

Floor

↓

Room

↓

Desk

```

Equivalent Playwright objects:

```text
Browser

↓

Browser Context

↓

Page

↓

Locator

```

Each layer has a specific responsibility.

----------

# Browser

The Browser object represents a running browser process.

Examples include:

-   Chromium
    
-   Firefox
    
-   WebKit
    

Architecture

```text
Browser

↓

Chromium.exe

```

or

```text
Browser

↓

Firefox.exe

```

Only one browser process may serve many tests.

----------

# Browser Responsibilities

The Browser object is responsible for:

-   Launching the browser
    
-   Closing the browser
    
-   Creating BrowserContexts
    

It does **not** directly interact with web pages.

----------

# Browser Context

The BrowserContext is one of Playwright's most important concepts.

It represents an isolated browser session.

Each context has its own:

-   Cookies
    
-   Local Storage
    
-   Session Storage
    
-   Cache
    
-   Permissions
    

Example

```text
Browser

├── Context A

├── Context B

├── Context C

```

Each context behaves like a separate browser profile.

----------

# Why BrowserContext Exists

Imagine logging into an application.

Without isolation:

```text
Test A

↓

Login

↓

Cookies

↓

Test B

```

Test B unexpectedly starts already logged in.

With BrowserContexts:

```text
Browser

↓

Context A

↓

Logged In

----------------

Browser

↓

Context B

↓

Anonymous

```

Tests remain independent.

----------

# Browser vs BrowserContext

Browser

BrowserContext

Browser process

Isolated session

Heavyweight

Lightweight

Shared

Independent

Created less frequently

Created per test (typically)

Creating a new context is much faster than launching a new browser.

----------

# Page

A Page represents a browser tab.

```text
Browser

↓

Context

↓

Page

```

Every interaction happens through the Page object.

Examples:

-   Navigate
    
-   Click
    
-   Fill
    
-   Screenshot
    
-   Evaluate JavaScript
    

----------

# Multiple Pages

One BrowserContext may contain multiple tabs.

```text
Context

├── Page 1

├── Page 2

├── Page 3

```

Useful for:

-   Popups
    
-   Multiple tabs
    
-   Payment redirects
    
-   OAuth authentication
    

----------

# Frame

Some pages contain embedded frames.

Architecture

```text
Page

↓

Frame

↓

Frame

↓

Frame

```

Each frame has its own DOM.

Playwright provides dedicated APIs for interacting with frames.

----------

# Locator

Locators are one of Playwright's most powerful concepts.

Instead of immediately finding an element,

a Locator represents **a strategy for finding an element whenever it is needed**.

```text
Locator

↓

Find Element

↓

Interact

```

This design supports auto-waiting and retry logic.

----------

# Lazy Evaluation

Unlike many automation libraries,

Playwright locators are **lazy**.

Example

```typescript
const loginButton =
page.getByRole("button", { name: "Login" });

```

At this point,

Playwright has **not** searched the page.

The element is located only when an action occurs.

```typescript
await loginButton.click();

```

This improves stability when pages update dynamically.

----------

# ElementHandle vs Locator

Older APIs often relied on element handles.

```text
Find Element

↓

Store Reference

↓

DOM Changes

↓

Reference Invalid

```

Locators work differently.

```text
Locator

↓

Action

↓

Locate Again

↓

Success

```

This significantly reduces stale element issues.

----------

# Request and Response

Playwright also provides network objects.

```text
Browser

↓

Request

↓

Server

↓

Response

```

Useful for:

-   API testing
    
-   Network interception
    
-   Mocking
    
-   Validation
    

----------

# Worker

Workers execute tests in parallel.

```text
Worker 1

↓

Tests

----------------

Worker 2

↓

Tests

----------------

Worker 3

↓

Tests

```

Each worker has isolated execution resources.

----------

# Playwright Test Runner

The Test Runner orchestrates execution.

Responsibilities include:

-   Discovering tests
    
-   Scheduling workers
    
-   Creating fixtures
    
-   Running tests
    
-   Reporting results
    
-   Managing retries
    

Architecture

```text
Playwright Test

↓

Workers

↓

Fixtures

↓

Tests

```

----------

# Complete Execution Lifecycle

A typical test follows this sequence.

```text
Start Test

↓

Launch Browser

↓

Create Browser Context

↓

Create Page

↓

Navigate

↓

Locate Element

↓

Perform Action

↓

Assertion

↓

Close Context

↓

Finish Test

```

Most of this lifecycle is managed automatically.

----------

# Object Relationships

```text
Browser

↓

Context

↓

Page

↓

Frame

↓

Locator

```

Each object depends on the layer above it.

----------

# Why BrowserContext Is So Important

BrowserContext enables:

-   Parallel execution
    
-   Test isolation
    
-   Authentication reuse
    
-   Multi-user testing
    

Example

```text
Admin Context

↓

Admin User

----------------

Customer Context

↓

Customer User

```

Both users can interact simultaneously.

----------

# Memory Model

```text
Browser

↓

Multiple Contexts

↓

Multiple Pages

↓

Locators

```

This architecture is efficient because contexts consume far fewer resources than launching multiple browsers.

----------

# Communication Flow

```text
Test

↓

Playwright

↓

Browser Engine

↓

DOM

```

Commands travel down the stack, while results travel back to the test.

----------

# Playwright Without WebDriver

Unlike Selenium,

Playwright communicates directly with browser engines.

```text
Test

↓

Playwright

↓

Browser

```

There is no external WebDriver executable between the framework and the browser.

We'll compare this in detail in the next chapter.

----------

# Object Creation Order

Objects are typically created in this order.

```text
Browser

↓

Browser Context

↓

Page

↓

Locator

```

Objects are destroyed in the reverse order.

----------

# Enterprise Architecture

```text
Tests

↓

Fixtures

↓

Browser

↓

Context

↓

Page

↓

Components

↓

Services

```

This layered architecture scales well for large automation suites.

----------

# Common Misconceptions

### "BrowserContext is another browser."

No. It is an isolated session inside a browser process.

----------

### "Each test launches a new browser."

Not necessarily. By default, Playwright typically reuses the browser process while creating new contexts for isolation.

----------

### "Locators immediately find elements."

No. Locators are lazy and resolve elements when actions or assertions are performed.

----------

### "A Page represents the whole browser."

No. A Page represents a single tab within a BrowserContext.

----------

# Best Practices

-   Prefer a new BrowserContext for each independent test.
    
-   Use Locators instead of `ElementHandle` whenever possible.
    
-   Keep Page objects focused on a single browser tab or view.
    
-   Let the Playwright Test Runner manage browser lifecycles.
    
-   Understand object relationships before designing framework abstractions.
    

----------

# Interview Questions

### Q1. What is the difference between a Browser and a BrowserContext?

A Browser represents the browser process, while a BrowserContext represents an isolated browser session with its own cookies, storage, and permissions.

----------

### Q2. Why is BrowserContext considered lightweight?

Because it creates isolated sessions within an existing browser process, avoiding the overhead of launching an entirely new browser.

----------

### Q3. What does a Page object represent?

A Page represents a single browser tab where navigation, interactions, and assertions take place.

----------

### Q4. Why are Playwright Locators called lazy?

Locators do not immediately search for elements. They resolve the element only when an action or assertion is executed, enabling automatic retries and better handling of dynamic pages.

----------

### Q5. How does Playwright support parallel execution?

The Playwright Test Runner uses multiple workers, each with isolated browser contexts and resources, allowing independent tests to execute concurrently.

----------

# Summary

Playwright's architecture is built around a hierarchy of Browser, BrowserContext, Page, Frame, and Locator objects. BrowserContexts provide efficient isolation, Pages represent browser tabs, and Locators offer lazy, resilient element interactions. Combined with the Playwright Test Runner and worker-based parallel execution, this architecture enables fast, reliable, and scalable browser automation while keeping tests isolated and maintainable.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1MzE5MDUxMzVdfQ==
-->