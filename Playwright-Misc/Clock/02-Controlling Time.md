This is where the **Clock API becomes a game changer**.

The previous chapter introduced the concept of mocking time.

Now we'll actually **control time**.

This eliminates one of the biggest causes of slow UI automation:

> Waiting for timers.

Instead of waiting:

-   30 seconds
    
-   2 minutes
    
-   15 minutes
    
-   24 hours
    

Playwright can advance the application's clock instantly.

----------

# Part 14 – Clock

# Chapter 2 – Controlling Time

----------

# Introduction

Suppose your application has:

-   Session timeout after **15 minutes**
    
-   OTP expires after **30 seconds**
    
-   Countdown timer
    
-   Auto refresh every **60 seconds**
    
-   Toast disappears after **5 seconds**
    

Without Clock API:

```text
Wait

↓

Wait

↓

Wait

↓

Verify

```

With Clock API:

```text
Advance Time

↓

Verify

↓

Done

```

Tests become dramatically faster.

----------

# Clock Operations

Playwright provides several ways to control time.

Method

Purpose

`setFixedTime()`

Freeze browser time

`fastForward()`

Move time forward instantly

`runFor()`

Run timers for a specific duration

`pauseAt()`

Pause automatic progression at a specific time

`resume()`

Resume the mocked clock

Each serves a different purpose.

----------

# fastForward()

The most commonly used operation.

Example:

```typescript
await page.clock.fastForward("30s");

```

Application time immediately advances by:

```text
30 Seconds

```

No real waiting occurs.

----------

# Example – OTP Expiration

Application:

```typescript
setTimeout(() => {

    otpExpired();

}, 30000);

```

Test:

```typescript
await page.clock.install();

await page.goto("/otp");

await page.clock.fastForward("30s");

await expect(
    page.getByText("OTP Expired")
).toBeVisible();

```

The timeout executes immediately.

----------

# Fast Forward One Minute

```typescript
await page.clock.fastForward("1m");

```

----------

# Fast Forward One Hour

```typescript
await page.clock.fastForward("1h");

```

----------

# Fast Forward One Day

```typescript
await page.clock.fastForward("24h");

```

Useful for testing:

-   Trial expiration
    
-   Daily reports
    
-   Subscription renewal
    
-   Calendar reminders
    

----------

# Timeline Example

Without Clock

```text
10:00

↓

10:30

↓

11:00

```

Thirty real minutes pass.

----------

With Clock

```text
10:00

↓

fastForward(30m)

↓

10:30

```

Instant.

----------

# runFor()

Sometimes you want timers to execute naturally while advancing mocked time.

Example:

```typescript
await page.clock.runFor("10s");

```

This advances the mocked clock while allowing scheduled callbacks to execute in the correct order.

----------

# Example – Progress Bar

Application

```typescript
setInterval(updateProgress, 1000);

```

Test

```typescript
await page.clock.install();

await page.goto("/progress");

await page.clock.runFor("10s");

await expect(
    page.getByText("100%")
).toBeVisible();

```

----------

# fastForward() vs runFor()

fastForward()

runFor()

Instantly advances the mocked clock

Advances mocked time while running scheduled timers in sequence

Ideal for expiration scenarios

Ideal for repeated timer updates and animations

Commonly used

Useful for progressive behavior

----------

# pauseAt()

Sometimes time should stop at a specific moment.

Example

```typescript
await page.clock.pauseAt(
    new Date("2026-12-31T23:59:59")
);

```

Once the mocked clock reaches this time, it stops advancing automatically.

Useful for:

-   Year-end processing
    
-   Midnight rollover
    
-   Billing cycles
    

----------

# resume()

Continue advancing the mocked clock after a pause.

```typescript
await page.clock.resume();

```

----------

# Example – Session Timeout

Application

```typescript
setTimeout(() => {

    logoutUser();

}, 15 * 60 * 1000);

```

Test

```typescript
await page.clock.install();

await page.goto("/dashboard");

await page.clock.fastForward("15m");

await expect(
    page.getByText("Session Expired")
).toBeVisible();

```

No 15-minute wait.

----------

# Example – Auto Refresh

Application

```typescript
setInterval(refreshData, 60000);

```

Test

```typescript
await page.clock.install();

await page.goto("/reports");

await page.clock.runFor("5m");

```

Five refresh cycles execute immediately.

----------

# Example – Toast Notification

Application

```typescript
setTimeout(() => {

    hideToast();

}, 5000);

```

Test

```typescript
await page.clock.install();

await page.goto("/");

await expect(
    page.getByRole("status")
).toBeVisible();

await page.clock.fastForward("5s");

await expect(
    page.getByRole("status")
).toBeHidden();

```

----------

# Example – Countdown Timer

Application

```text
30

↓

29

↓

28

↓

...

↓

0

```

Test

```typescript
await page.clock.install();

await page.goto("/countdown");

await page.clock.runFor("30s");

await expect(
    page.getByText("0")
).toBeVisible();

```

----------

# Example – Subscription Expiry

```typescript
await page.clock.install();

await page.clock.setFixedTime(
    new Date("2026-01-01")
);

await page.goto("/subscription");

await page.clock.fastForward("30d");

await expect(
    page.getByText("Expired")
).toBeVisible();

```

----------

# Example – Flash Sale

Application

```text
Sale Ends In

02:00

```

Test

```typescript
await page.clock.install();

await page.goto("/sale");

await page.clock.fastForward("2m");

await expect(
    page.getByText("Sale Ended")
).toBeVisible();

```

----------

# Timer Execution Order

Suppose:

```typescript
setTimeout(A, 1000);

setTimeout(B, 2000);

setTimeout(C, 3000);

```

After:

```typescript
await page.clock.runFor("3s");

```

Execution order remains:

```text
A

↓

B

↓

C

```

The mocked clock preserves timer ordering.

----------

# Browser APIs Affected

When time advances, these browser APIs stay consistent:

-   `Date`
    
-   `Date.now()`
    
-   `performance.now()`
    
-   `setTimeout()`
    
-   `setInterval()`
    
-   `requestAnimationFrame()`
    

Your application behaves as though real time has passed.

----------

# Enterprise Use Cases

The Clock API is especially valuable for testing:

-   Session timeout after inactivity
    
-   OTP expiration
    
-   Password reset links
    
-   Shopping cart expiration
    
-   Auction countdowns
    
-   Subscription renewals
    
-   Promotional banners
    
-   Auto logout
    
-   Scheduled notifications
    
-   Dashboard auto-refresh
    

----------

# Best Practices

-   Install the clock before the page initializes.
    
-   Prefer `fastForward()` for expiration-based scenarios.
    
-   Use `runFor()` when multiple timers should execute naturally over simulated time.
    
-   Use fixed dates when business rules depend on the calendar.
    
-   Verify user-visible outcomes rather than internal timer implementation.
    

----------

# Common Mistakes

### ❌ Waiting with `waitForTimeout()`

```typescript
await page.waitForTimeout(30000);

```

Prefer:

```typescript
await page.clock.fastForward("30s");

```

----------

### ❌ Installing the clock too late

Timers created before clock installation may continue using the original timing behavior.

----------

### ❌ Assuming server-side jobs advance

The Clock API only affects browser-side JavaScript.

----------

### ❌ Using mocked time for every test

Only use the Clock API for features that genuinely depend on time.

----------

# Interview Questions

### Q1. What is the difference between `fastForward()` and `runFor()`?

-   `fastForward()` is ideal for jumping directly to a future point in mocked time, such as expiring a session or OTP.
    
-   `runFor()` advances mocked time while executing scheduled timers in sequence, making it suitable for countdowns, progress bars, and repeated intervals.
    

----------

### Q2. Can you test a 15-minute session timeout instantly?

Yes.

```typescript
await page.clock.fastForward("15m");

```

----------

### Q3. Does advancing the mocked clock affect backend timers?

No. Only browser-side JavaScript is affected.

----------

### Q4. Why is the Clock API better than `waitForTimeout()`?

It makes tests dramatically faster, deterministic, and less flaky by eliminating unnecessary real-time delays.

----------

### Q5. What kinds of applications benefit most from the Clock API?

Applications with time-sensitive behavior such as authentication, finance, e-commerce, dashboards, booking systems, subscriptions, and notification services.

----------

# Summary

The Clock API transforms how time-based features are tested. Instead of waiting for real time to pass, you can advance the browser's clock instantly, allowing timers, intervals, countdowns, and expiration logic to execute in seconds. Used correctly, it produces faster, more reliable, and deterministic tests while preserving the application's expected timing behavior.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTgwODU4NDg4M119
-->