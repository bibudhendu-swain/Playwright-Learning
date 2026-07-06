This is another topic that is becoming increasingly important in modern web applications.

Applications like:

-   Microsoft Teams
    
-   Slack
    
-   Gmail
    
-   Outlook
    
-   WhatsApp Web
    
-   Google Calendar
    
-   Jira
    
-   GitHub
    

all use **Browser Notifications** to alert users even when they're not actively looking at the page.

Testing these notifications manually is difficult because they depend on browser permissions and sometimes service workers.

Playwright provides excellent support for testing the permission flow, and you can mock notification creation where appropriate.

----------

# Part 17 – Mock Browser APIs

# Chapter 5 – Notifications API

----------

# Introduction

Many web applications need to notify users about important events.

Examples include:

-   New chat message
    
-   Meeting reminder
    
-   Build completed
    
-   Payment received
    
-   Security alert
    
-   Calendar reminder
    

Typical flow:

```text
Application

↓

Notification Permission

↓

Create Notification

↓

Browser Notification

↓

User Clicks

```

Without permission, notifications cannot be displayed.

----------

# What is the Notifications API?

The Notifications API allows websites to display system notifications outside the browser window.

The browser provides:

```typescript
Notification

```

Example

```typescript
new Notification("Order Shipped");

```

Unlike an alert dialog, the notification appears in the operating system's notification area.

----------

# Notification Flow

```text
Application

↓

Notification API

↓

Operating System

↓

User Sees Notification

```

The notification exists outside the web page.

----------

# Browser Permission

Before creating notifications, the application must obtain permission.

Possible states are:

```text
default

↓

granted

↓

denied

```

----------

# Checking Permission

Application code

```typescript
console.log(Notification.permission);

```

Possible output:

```text
default

```

or

```text
granted

```

or

```text
denied

```

----------

# Requesting Permission

Applications request permission using:

```typescript
await Notification.requestPermission();

```

Normally, the browser displays a permission prompt.

----------

# Grant Notification Permission

In Playwright:

```typescript
const context =
await browser.newContext();

await context.grantPermissions([
    "notifications"
]);

```

The browser automatically grants notification permission.

----------

# Verify Permission

```typescript
const permission =
await page.evaluate(() => {

    return Notification.permission;

});

expect(permission)

.toBe("granted");

```

This confirms the page can create notifications.

----------

# Creating a Notification

Application

```typescript
new Notification(

    "Build Successful"

);

```

The operating system displays:

```text
Build Successful

```

----------

# Mocking Notification Creation

Because operating system notifications are outside the DOM, a common testing strategy is to intercept the browser API rather than verifying the OS notification itself.

Example:

```typescript
await page.addInitScript(() => {

    const OriginalNotification = window.Notification;

    class MockNotification extends OriginalNotification {

        constructor(title: string, options?: NotificationOptions) {

            console.log("Notification:", title);

            super(title, options);

        }

    }

    window.Notification = MockNotification;

});

```

This allows tests to observe notification creation.

> **Note:** In some applications, replacing the constructor entirely with a lightweight mock (instead of extending it) is preferable, depending on browser support and your testing goals.

----------

# Capturing Notification Requests

Another simple approach is to record every notification request.

```typescript
await page.addInitScript(() => {

    const notifications: string[] = [];

    const OriginalNotification = window.Notification;

    // @ts-ignore
    window.Notification = function (
        title: string,
        options?: NotificationOptions
    ) {
        notifications.push(title);
        return new OriginalNotification(title, options);
    } as any;

    // @ts-ignore
    window.__notifications = notifications;

});

```

Later:

```typescript
const notifications =
await page.evaluate(() => {

    // @ts-ignore
    return window.__notifications;

});

expect(notifications)

.toContain("Order Created");

```

----------

# Testing Notification Permission

Positive scenario

```text
Permission

↓

Granted

↓

Notification Created

```

Negative scenario

```text
Permission

↓

Denied

↓

Application Shows Message

```

Both should be tested.

----------

# Testing Permission Denied

Do **not** grant notification permission.

```typescript
const context =
await browser.newContext();

const page =
await context.newPage();

```

Application

```typescript
Notification.permission

```

returns

```text
default

```

or

```text
denied

```

depending on the application's flow and browser behavior.

Verify the application displays an appropriate message such as:

```text
Enable Notifications

```

----------

# Mocking Different Permission States

Sometimes you want to simulate permission values directly.

Example:

```typescript
await page.addInitScript(() => {

    Object.defineProperty(

        Notification,

        "permission",

        {

            get: () => "denied"

        }

    );

});

```

Now

```typescript
Notification.permission

```

always returns

```text
denied

```

Useful for negative testing.

----------

# Notification with Body

Application

```typescript
new Notification(

    "Order Ready",

    {

        body:

        "Pickup within 30 minutes"

    }

);

```

Test

Verify:

-   Title
    
-   Body
    
-   Timing
    

----------

# Notification with Icon

Application

```typescript
new Notification(

    "Payment",

    {

        icon:

        "/payment.png"

    }

);

```

Verify notification creation and, if you're mocking the constructor, that the expected options were passed.

----------

# Enterprise Example – Chat

```text
New Message

↓

Notification

↓

User Opens Chat

```

Verify

-   Notification requested
    
-   Correct title
    
-   Correct body
    

----------

# Enterprise Example – CI/CD

```text
Build Completed

↓

Notification

↓

Developer Opens Report

```

Notification title

```text
Pipeline Successful

```

----------

# Enterprise Example – Banking

```text
Money Debited

↓

Notification

↓

Customer Opens Statement

```

Verify

-   Notification content
    
-   Amount
    
-   Account
    

----------

# Enterprise Example – Calendar

```text
Meeting

↓

Notification

↓

Join Meeting

```

Application

```text
Standup

Starts in 5 Minutes

```

----------

# Notification Test Flow

```text
Grant Permission

↓

Open Application

↓

Trigger Event

↓

Notification Requested

↓

Verify

```

----------

# Notification Helper

Instead of repeating verification logic:

```typescript
class NotificationHelper {

    static async permission(page){

        return await page.evaluate(

            () => Notification.permission

        );

    }

}

```

Usage

```typescript
expect(

await NotificationHelper.permission(page)

)

.toBe("granted");

```

----------

# Suggested Folder Structure

```text
helpers/

├── NotificationHelper.ts

├── ClipboardHelper.ts

├── PermissionManager.ts

└── BrowserContextFactory.ts

```

----------

# Notifications vs Alerts

Alert

Notification

Blocks the page

Non-blocking

Browser dialog

OS/browser notification

Requires user interaction

Can appear in the background

Handled via dialog events

Controlled via Notifications API

----------

# Best Practices

-   Grant notification permission only for tests that require it.
    
-   Verify permission state before triggering notifications.
    
-   Prefer observing notification requests rather than relying on operating system UI.
    
-   Keep notification verification reusable through helper utilities.
    
-   Test both permission granted and permission denied scenarios.
    

----------

# Common Mistakes

### ❌ Trying to automate the operating system notification

Playwright automates the browser, not the operating system notification center.

Instead,

verify:

-   Notification permission
    
-   Notification request
    
-   Notification payload
    

----------

### ❌ Ignoring Permission State

Always verify

```typescript
Notification.permission

```

before testing notification behavior.

----------

### ❌ Testing Only Success

Applications should also handle

```text
denied

```

gracefully.

----------

### ❌ Assuming Notifications Work Identically Everywhere

Notification behavior can vary depending on:

-   Browser
    
-   Operating system
    
-   Service Worker implementation
    
-   User settings
    

Design tests around your application's behavior rather than OS-specific rendering.

----------

# Service Workers and Push Notifications

Many production applications use **Push Notifications** delivered through **Service Workers**.

The flow typically looks like:

```text
Push Service

↓

Service Worker

↓

Notification

↓

User

```

While Playwright can test permission handling and browser-side notification logic, full end-to-end push notification delivery usually requires integration with your application's push infrastructure and is often validated through integration or end-to-end environment testing rather than isolated UI tests.

----------

# Interview Questions

### Q1. What is the Notifications API?

It allows web applications to display system notifications outside the browser window after the user grants permission.

----------

### Q2. How do you grant notification permission in Playwright?

```typescript
await context.grantPermissions([
    "notifications"
]);

```

----------

### Q3. What are the possible values of `Notification.permission`?

-   `"default"`
    
-   `"granted"`
    
-   `"denied"`
    

----------

### Q4. Should UI tests verify operating system notifications directly?

Generally no. It's more reliable to verify notification permission, notification creation, and the data passed to the Notifications API rather than interacting with the OS notification center.

----------

### Q5. Why are Service Workers relevant to notifications?

Many web applications use Service Workers to receive push messages and create notifications even when the page is not actively open.

----------

# Summary

The Notifications API enables web applications to display browser and system notifications after obtaining user permission. Playwright makes notification testing reliable by allowing tests to grant permissions programmatically and inspect notification creation within the browser. Rather than relying on operating system UI, enterprise tests should focus on permission state, notification requests, and the payload passed to the Notifications API, while treating end-to-end push delivery as an integration concern.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTA0NDA2MDMwXX0=
-->