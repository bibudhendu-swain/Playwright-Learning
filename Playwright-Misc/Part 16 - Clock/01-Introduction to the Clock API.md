Now we arrive at one of the **newest and most powerful features in Playwright**.

# Part 16 – Clock

Until recently, testing time-based functionality was one of the hardest problems in UI automation.

Think about features like:

-   OTP expiration after 30 seconds
    
-   Session timeout after 15 minutes
    
-   Countdown timers
    
-   Scheduled notifications
    
-   Auto logout
    
-   Subscription renewal
    
-   Flash sales
    
-   Token expiration
    

Traditionally, teams had two bad choices:

1.  Actually wait 15 minutes 😄
    
2.  Mock time manually using application-specific hacks
    

The **Clock API** solves this elegantly.

----------

# Part 14 – Clock

# Chapter 1 – Introduction to the Clock API

----------

# Introduction

Many applications depend on **time**.

Examples:

```text
Login

↓

Session expires after 15 minutes

↓

User logged out

```

or

```text
OTP Sent

↓

30 Seconds Countdown

↓

OTP Expires

```

Testing these manually is slow.

Playwright allows us to **control time itself**.

----------

# What is the Clock API?

The Clock API allows Playwright to control the browser's perception of time.

Instead of waiting for real time:

```text
Real Time

00:00

↓

00:01

↓

00:02

↓

00:03

```

Playwright can instantly jump forward.

----------

# Why is it Useful?

Without Clock:

```text
Wait 30 seconds

↓

Test OTP Expiration

```

With Clock:

```text
Advance Clock 30 seconds

↓

OTP Expires Immediately

```

Huge time savings.

----------

# How the Clock Works

Internally:

```text
Real System Clock

×

Ignored

------------------

Playwright Clock

↓

Application Reads Mock Time

```

The browser page sees the mocked time instead of the real system time.

----------

# Installing the Clock

Before controlling time, install the clock.

```typescript
await page.clock.install();

```

This replaces browser timing APIs with Playwright-controlled implementations.

> **Important:** Install the clock **before** navigating to or executing code in the page that relies on timers or the current time.

----------

# What Gets Controlled?

The Clock API affects browser-side time APIs such as:

-   `Date`
    
-   `Date.now()`
    
-   `setTimeout()`
    
-   `setInterval()`
    
-   `clearTimeout()`
    
-   `clearInterval()`
    
-   `requestAnimationFrame()` (behavior coordinated with mocked time)
    
-   `performance.now()` (mocked consistently with the clock)
    

This gives deterministic control over time-based behavior.

----------

# Example Application

Suppose the page contains:

```typescript
setTimeout(() => {
    console.log("Expired");
}, 30000);

```

Normally:

```text
Wait 30 seconds

```

With the Clock API:

```text
Advance Clock

↓

Timer Executes Immediately

```

----------

# Setting an Initial Time

Freeze the browser at a specific date.

```typescript
await page.clock.setFixedTime(
    new Date("2026-01-01T10:00:00Z")
);

```

Now:

```typescript
await page.evaluate(() => {
    return new Date().toISOString();
});

```

Returns:

```text
2026-01-01T10:00:00.000Z

```

No matter what the real system time is.

----------

# Real-World Example – Greeting

Application:

```text
Morning

↓

Good Morning

```

Test:

```typescript
await page.clock.setFixedTime(
    new Date("2026-07-01T08:00:00")
);

await page.goto("/");

await expect(
    page.getByText("Good Morning")
).toBeVisible();

```

----------

# Reading the Mocked Time

```typescript
const now = await page.evaluate(() => {
    return Date.now();
});

```

Returns the mocked value.

----------

# Mock Time vs Real Time

Real machine:

```text
21:30

```

Browser sees:

```text
08:00

```

Only the browser context is affected.

Your Node.js test process still uses the real clock.

----------

# Freezing Time

A fixed clock means time does not advance automatically.

```text
08:00

↓

08:00

↓

08:00

```

Until you move it.

This is ideal for deterministic tests.

----------

# Example – Expiry Banner

Application:

```text
Subscription expires today

```

Freeze:

```typescript
await page.clock.setFixedTime(
    new Date("2026-12-31")
);

```

Now every test run sees the same date.

----------

# Example – Age Verification

Application:

```text
Date of Birth

↓

Age >= 18?

```

Instead of changing test data every year:

```typescript
await page.clock.setFixedTime(
    new Date("2030-01-01")
);

```

Your expected behavior remains stable.

----------

# Clock Installation Order

Correct:

```typescript
await page.clock.install();

await page.clock.setFixedTime(
    new Date("2026-01-01")
);

await page.goto("/");

```

This ensures page scripts observe the mocked clock from the beginning.

----------

Incorrect:

```typescript
await page.goto("/");

await page.clock.install();

```

If the application already initialized timers or read the current time, mocking afterwards may not affect that earlier logic.

----------

# Browser Only

The Clock API affects browser JavaScript.

It does **not** change:

-   Node.js time
    
-   Operating system time
    
-   Backend server time
    
-   Database timestamps
    

Example:

```typescript
console.log(new Date());

```

Still prints the real machine time.

----------

# Common Time-Based Features

```text
OTP

↓

Session Timeout

↓

Countdown

↓

Flash Sale

↓

Trial Expiration

↓

Scheduled Reminder

```

All become easy to test.

----------

# Browser APIs Controlled

API

Controlled

`Date`

✅

`Date.now()`

✅

`setTimeout()`

✅

`setInterval()`

✅

`performance.now()`

✅

`requestAnimationFrame()`

✅

----------

# Best Practices

-   Install the clock before the application initializes.
    
-   Freeze time to make tests deterministic.
    
-   Use fixed dates for business rules that depend on calendars.
    
-   Keep in mind that only browser-side code observes the mocked clock.
    
-   Combine the Clock API with assertions to verify business behavior rather than internal implementation details.
    

----------

# Common Mistakes

### ❌ Installing the clock after navigation

If the application has already scheduled timers or read the current time, those operations may not be affected.

----------

### ❌ Assuming the backend uses mocked time

The Clock API affects only the browser. Server-side code continues using the real time unless separately controlled.

----------

### ❌ Mixing browser and Node.js time

Remember:

```typescript
await page.evaluate(() => Date.now());

```

returns the mocked browser time.

Whereas:

```typescript
Date.now();

```

in your test returns the real Node.js time.

----------

### ❌ Using the Clock API when it's unnecessary

If your application has no time-dependent behavior, there's no benefit to mocking time.

----------

# Interview Questions

### Q1. What is the Playwright Clock API?

It allows Playwright to control browser-side time APIs, making time-dependent features deterministic and fast to test.

----------

### Q2. Does the Clock API change the operating system clock?

No. It only affects the browser page's perception of time.

----------

### Q3. Why should the clock be installed before page navigation?

Because many applications read the current time or start timers during initialization. Installing the clock first ensures those operations use the mocked time.

----------

### Q4. What kinds of features benefit from the Clock API?

Examples include OTP expiration, session timeouts, countdown timers, flash sales, subscription expiry, and scheduled UI updates.

----------

### Q5. Does the Clock API affect backend services?

No. Backend servers, databases, and the Node.js test process continue using real time.

----------

# Summary

The Clock API gives Playwright deterministic control over browser-side time, eliminating the need to wait for real timers during testing. By freezing or mocking the current time before the application initializes, you can reliably test time-sensitive features such as session expiration, countdowns, and date-based business rules while keeping your tests fast and repeatable.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTExMDY2OTc5OTNdfQ==
-->