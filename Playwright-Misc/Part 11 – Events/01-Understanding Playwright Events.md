**Events are the foundation of how Playwright works internally**.

Most automation engineers use Playwright like this:

```ts
await page.click();
await page.fill();
await expect(...);

```

But internally, Playwright is **event-driven**.

Once you understand events, many APIs suddenly make much more sense:

-   Downloads
    
-   Dialogs
    
-   Popups
    
-   Network monitoring
    
-   Console logs
    
-   File chooser
    
-   WebSocket
    
-   Worker
    
-   Page crash
    
-   New tabs
    

Everything is an event.

----------

# Part 11 – Events

# Chapter 1 – Understanding Playwright Events

----------

# Introduction

Modern browsers constantly generate events.

Imagine you're browsing Amazon.

```text
Open Website
      │
      ▼
Page Loaded
      │
      ▼
API Request Sent
      │
      ▼
API Response Received
      │
      ▼
Console Log
      │
      ▼
Button Click
      │
      ▼
Popup Opens
      │
      ▼
Download Starts

```

Every one of these actions produces an **event**.

Playwright allows us to listen for these events.

----------

# What is an Event?

An event is simply a **notification that something happened**.

Examples:

-   User clicked a button
    
-   Browser opened a popup
    
-   Download started
    
-   Network request completed
    
-   JavaScript error occurred
    
-   Dialog opened
    

Instead of continuously checking whether something happened (polling), Playwright notifies your test.

----------

# Event-Driven Programming

Most Playwright automation follows this model:

```text
Something Happens
        │
        ▼
Playwright Fires Event
        │
        ▼
Your Code Responds

```

Example

```typescript
page.on('dialog', async dialog => {

    await dialog.accept();

});

```

Your code waits until the dialog appears.

----------

# Event Sources

Events can originate from different objects.

```text
Browser
   │
   ├── Browser Events
   │
BrowserContext
   │
   ├── Context Events
   │
Page
   │
   ├── Page Events
   │
Network
   │
   ├── Request Events
   └── Response Events

```

Each object exposes its own events.

----------

# Three Ways to Listen for Events

Playwright provides three primary approaches.

## 1. page.on()

Permanent listener.

```typescript
page.on('dialog', async dialog => {

    await dialog.accept();

});

```

The listener remains active until removed.

----------

## 2. page.once()

One-time listener.

```typescript
page.once('dialog', async dialog => {

    await dialog.accept();

});

```

Runs once.

Automatically removed afterwards.

----------

## 3. waitForEvent()

Waits for a single occurrence.

```typescript
const downloadPromise =
page.waitForEvent('download');

```

Usually the preferred approach inside tests.

----------

# page.on()

## Purpose

Register a listener that remains active.

Syntax

```typescript
page.on(eventName, callback);

```

----------

## Example

```typescript
page.on('console', message => {

    console.log(message.text());

});

```

Every console log from the browser is printed.

----------

## Another Example

```typescript
page.on('dialog', async dialog => {

    console.log(dialog.message());

    await dialog.accept();

});

```

Every dialog is handled automatically.

----------

# page.once()

Only handles the first event.

Example

```typescript
page.once('dialog', async dialog => {

    await dialog.accept();

});

```

Suppose:

```text
Dialog 1

↓

Handled

---------

Dialog 2

↓

Ignored

```

The listener removes itself after the first dialog.

----------

# waitForEvent()

Instead of creating a permanent listener,

wait only when needed.

Example

```typescript
const popupPromise =
page.waitForEvent('popup');

await page.click('#login');

const popup =
await popupPromise;

```

Very common.

----------

# Event Order

Consider downloading a file.

```text
Click Download
      │
      ▼
Request Sent
      │
      ▼
Response Received
      │
      ▼
Download Event
      │
      ▼
Download Completed

```

Understanding event order helps debug timing issues.

----------

# Removing Event Listeners

Sometimes listeners are temporary.

```typescript
function onDialog(dialog) {

    dialog.accept();

}

page.on('dialog', onDialog);

page.off('dialog', onDialog);

```

The listener no longer receives dialog events.

----------

# Why Remove Listeners?

Suppose:

```text
Test 1

↓

Listener Added

↓

Test Ends

↓

Test 2

↓

Same Listener Still Active

```

Unexpected behavior may occur if listeners aren't cleaned up in long-running utilities or shared objects.

----------

# Event Parameters

Every event supplies an object.

Examples:

Event

Parameter

dialog

Dialog

download

Download

popup

Page

request

Request

response

Response

console

ConsoleMessage

Example

```typescript
page.on('response', response => {

    console.log(response.url());

});

```

----------

# Real-World Example

Capture every failed request.

```typescript
page.on('requestfailed', request => {

    console.log(

        request.url()

    );

});

```

Very useful while debugging.

----------

# Another Example

Capture all console errors.

```typescript
page.on('console', message => {

    if (message.type() === 'error') {

        console.log(message.text());

    }

});

```

Useful for UI health monitoring.

----------

# Combining Multiple Events

Example

```typescript
page.on('request', request => {

    console.log(

        request.method(),

        request.url()

    );

});

page.on('response', response => {

    console.log(

        response.status(),

        response.url()

    );

});

```

Produces

```text
GET

/api/products

↓

200

/api/products

```

----------

# Event Timing

One of the most important concepts.

Wrong

```typescript
await page.click('#download');

await page.waitForEvent('download');

```

Correct

```typescript
const downloadPromise =
page.waitForEvent('download');

await page.click('#download');

const download =
await downloadPromise;

```

Always register before triggering.

----------

# Event Flow Example

Suppose

```text
Click Login

```

Sequence

```text
Button Click

↓

Request

↓

Response

↓

Page Navigation

↓

Load Event

↓

Console Log

↓

Success Message

```

Each can be observed independently.

----------

# Event Categories

We'll cover each category in detail in the following chapters.

Category

Examples

Page

dialog, popup, download

Network

request, response

Browser

disconnected

Context

page, close

Console

console

Workers

worker

Crash

crash

WebSocket

websocket

----------

# Best Practices

✅ Register listeners before triggering events.

✅ Use `waitForEvent()` for one-time events.

✅ Use `page.on()` for continuous monitoring.

✅ Remove long-lived listeners when they are no longer needed.

✅ Keep event handlers lightweight to avoid slowing down tests.

----------

# Common Mistakes

## ❌ Waiting after the action

```typescript
await page.click();

await page.waitForEvent();

```

The event may already have occurred.

----------

## ❌ Using `page.on()` everywhere

If the event only occurs once,

prefer

```typescript
waitForEvent()

```

----------

## ❌ Forgetting to remove listeners

Especially inside reusable utilities.

----------

# Interview Questions

### Q1. What is an event in Playwright?

An event is a notification that something happened in the browser or page, such as a network request, popup, download, or dialog.

----------

### Q2. What is the difference between `page.on()` and `page.once()`?

-   `page.on()` registers a persistent listener.
    
-   `page.once()` automatically removes the listener after handling the first occurrence.
    

----------

### Q3. When should you use `waitForEvent()`?

When you're expecting a single occurrence of an event as part of a test flow, such as a download, popup, or dialog.

----------

### Q4. Why should event listeners be registered before triggering actions?

Because many browser events occur immediately. Registering first ensures the event isn't missed.

----------

### Q5. Can multiple listeners be attached to the same event?

Yes. Multiple callbacks can listen to the same event, and each will be invoked when the event occurs.

----------

# Summary

Playwright is fundamentally event-driven. Rather than repeatedly checking for changes, it notifies your code when meaningful browser events occur. Understanding when to use `page.on()`, `page.once()`, and `waitForEvent()` allows you to build cleaner, more reliable automation for downloads, dialogs, popups, network activity, and many other browser interactions.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTkyMTUxMTI2OV19
-->