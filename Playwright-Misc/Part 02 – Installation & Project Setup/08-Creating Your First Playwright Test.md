# Part 2 – Installation & Project Setup

# Chapter 8 – Creating Your First Playwright Test

----------

# Introduction

Every Playwright journey begins with a simple test.

At first glance, a Playwright test appears surprisingly small.

```typescript
import { test, expect } from '@playwright/test';

test('Homepage loads', async ({ page }) => {
    await page.goto('https://playwright.dev');
    await expect(page).toHaveTitle(/Playwright/);
});

```

Although this example contains only a few lines, it introduces many important concepts:

-   Test Runner
    
-   Test Discovery
    
-   Fixtures
    
-   Browser Lifecycle
    
-   Page Object
    
-   Async Programming
    
-   Auto Waiting
    
-   Assertions
    

Understanding these concepts is far more valuable than memorizing the syntax.

----------

# What Happens When You Run a Test?

Suppose you execute:

```bash
npx playwright test

```

Playwright performs the following steps.

```text
Read Configuration

↓

Discover Tests

↓

Create Worker

↓

Launch Browser

↓

Create Browser Context

↓

Open Page

↓

Execute Test

↓

Generate Report

↓

Close Resources

```

All of this happens automatically.

----------

# Our First Test

Let's begin with a simple example.

```typescript
import { test, expect } from '@playwright/test';

test('Playwright homepage loads', async ({ page }) => {

    await page.goto('https://playwright.dev');

    await expect(page).toHaveTitle(/Playwright/);

});

```

We'll now examine every part of this test.

----------

# Line 1 – Import Statement

```typescript
import { test, expect } from '@playwright/test';

```

This imports two important functions.

Import

Purpose

test

Defines a Playwright test

expect

Performs assertions

Without this import,

Playwright wouldn't recognize your test.

----------

# Why Import?

Think of imports as bringing tools into your file.

```text
Playwright Library

↓

Import

↓

Your Test File

```

Only the imported functionality becomes available.

----------

# Understanding `test()`

```typescript
test('Playwright homepage loads', async ({ page }) => {

});

```

The `test()` function defines a single executable test.

Syntax

```typescript
test(

name,

implementation

);

```

Every Playwright test has:

-   A descriptive name
    
-   A function containing the test steps
    

----------

# Test Name

```typescript
'Playwright homepage loads'

```

This name appears in:

-   Reports
    
-   Console output
    
-   Trace Viewer
    
-   CI pipelines
    

Use names that describe business behavior rather than implementation details.

Good

```text
Customer can log in

```

Poor

```text
Test1

```

----------

# The Test Function

```typescript
async ({ page }) => {

}

```

This is where the automation logic lives.

Everything inside this function executes when the test runs.

----------

# Why `async`?

Playwright performs many operations that take time.

Examples:

-   Opening browsers
    
-   Navigating pages
    
-   Clicking buttons
    
-   Waiting for responses
    

These operations are asynchronous.

```text
Click

↓

Wait

↓

Continue

```

The `async` keyword allows the function to use `await`.

----------

# Understanding `await`

Example

```typescript
await page.goto(...);

```

Meaning:

```text
Start Navigation

↓

Wait Until Complete

↓

Continue

```

Without `await`,

the next statement could execute before navigation finishes.

----------

# What is the `page` Object?

```typescript
({ page })

```

`page` is a built-in Playwright fixture.

It represents a single browser tab.

```text
Browser

↓

Browser Context

↓

Page

```

Every interaction happens through this object.

----------

# Where Does `page` Come From?

Many beginners assume:

```typescript
const page = ...

```

should exist.

It doesn't.

Playwright automatically creates the page fixture.

```text
Playwright

↓

Fixture

↓

Page

↓

Your Test

```

You simply receive it as a function parameter.

----------

# Understanding Fixtures

Fixtures provide ready-to-use objects.

Common fixtures include:

```text
page

browser

context

request

```

We'll explore fixtures in detail later.

----------

# Navigating to a Website

```typescript
await page.goto(
'https://playwright.dev'
);

```

This instructs the browser to navigate.

Workflow

```text
Browser

↓

Request

↓

Website

↓

Load

↓

Continue

```

Playwright waits until navigation reaches the configured load state before proceeding.

----------

# What Happens During `goto()`?

Internally,

Playwright:

```text
Create Request

↓

Navigate

↓

Wait

↓

Return Control

```

This synchronization happens automatically.

----------

# Assertions

```typescript
await expect(page)
.toHaveTitle(/Playwright/);

```

Assertions verify application behavior.

Without assertions,

the test only performs actions.

It never checks whether the application behaved correctly.

----------

# Why Assertions Matter

Bad Test

```text
Click

↓

Finish

```

Good Test

```text
Click

↓

Verify Result

↓

Pass

```

Verification is what turns browser automation into a meaningful automated test.

----------

# Understanding `expect()`

```typescript
expect(page)

```

This creates an expectation about the current state.

The expectation becomes complete only after a matcher is applied.

----------

# Understanding `toHaveTitle()`

```typescript
toHaveTitle()

```

This matcher verifies the browser title.

Playwright automatically retries until:

-   The title matches
    
-   The timeout expires
    

No explicit wait is required.

----------

# Why Use Regular Expressions?

Example

```typescript
/Playwright/

```

Instead of requiring an exact title,

the test checks whether the title contains the word "Playwright."

This makes the assertion more flexible while still validating the expected behavior.

----------

# Complete Execution Flow

Let's put everything together.

```text
Test Runner

↓

Launch Browser

↓

Create Context

↓

Create Page

↓

Goto Website

↓

Wait

↓

Verify Title

↓

Pass

↓

Close Resources

```

This entire lifecycle is managed automatically.

----------

# Test Lifecycle

Every Playwright test follows the same pattern.

```text
Setup

↓

Execute

↓

Validate

↓

Cleanup

```

Understanding this lifecycle helps when writing larger tests.

----------

# What Makes This Test Reliable?

Several Playwright features contribute to reliability:

-   Auto-waiting
    
-   Browser isolation
    
-   Web-first assertions
    
-   Built-in retries (if configured)
    
-   Fixture management
    

Most of these features work without additional code.

----------

# Running the Test

Execute:

```bash
npx playwright test

```

Console output:

```text
Running 1 test

✓ Playwright homepage loads

1 passed

```

If configured,

Playwright also generates an HTML report.

----------

# Viewing the Report

Open:

```bash
npx playwright show-report

```

The report displays:

-   Test name
    
-   Duration
    
-   Status
    
-   Attachments
    
-   Screenshots (if configured)
    

----------

# Common Beginner Mistakes

## ❌ Forgetting `await`

Wrong

```typescript
page.goto('https://playwright.dev');

```

Correct

```typescript
await page.goto('https://playwright.dev');

```

----------

## ❌ Writing Tests Without Assertions

Automation without validation is not testing.

Always verify expected outcomes.

----------

## ❌ Using Generic Test Names

Prefer descriptive business-focused names.

----------

## ❌ Trying to Create the `page` Object Manually

Playwright provides it automatically through fixtures.

----------

## ❌ Adding Manual Waits

Avoid

```typescript
waitForTimeout(5000);

```

Trust Playwright's auto-waiting whenever possible.

----------

# Best Practices

-   Write descriptive test names.
    
-   Include at least one meaningful assertion in every test.
    
-   Use the `page` fixture instead of creating browser instances manually.
    
-   Always use `await` for asynchronous Playwright operations.
    
-   Let Playwright manage browser lifecycle and synchronization.
    
-   Keep your first tests simple and focused on one behavior.
    

----------

# Interview Questions

### Q1. What is the purpose of the `test()` function?

The `test()` function defines a single executable Playwright test, including its name and implementation.

----------

### Q2. Why is `async` required in Playwright tests?

Most Playwright operations are asynchronous. The `async` keyword allows the use of `await` to pause execution until those operations complete.

----------

### Q3. Where does the `page` object come from?

`page` is a built-in Playwright fixture. The Playwright Test Runner automatically creates and injects it into the test function.

----------

### Q4. Why should every test include assertions?

Assertions verify that the application behaved as expected. Without assertions, a test performs actions but does not validate outcomes.

----------

### Q5. Why is `await` important in Playwright?

`await` ensures that asynchronous operations such as navigation, clicking, and assertions complete before the next statement executes, preventing race conditions and unreliable tests.

----------

# Summary

A Playwright test may appear simple, but it introduces many foundational concepts: test discovery, fixtures, browser lifecycle management, asynchronous programming, navigation, and web-first assertions. By understanding each line of your first test rather than simply copying it, you build the conceptual foundation needed to write reliable, maintainable automation as your framework grows.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTc4NzU3MTg2Nl19
-->