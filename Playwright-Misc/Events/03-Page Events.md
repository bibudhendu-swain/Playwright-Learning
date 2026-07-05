This is another chapter that separates a **good Playwright engineer** from a **great Playwright engineer**.

Most people use Playwright only for clicking and typing.

Senior automation engineers also monitor:

-   JavaScript errors
    
-   Console logs
    
-   File uploads
    
-   WebSockets
    
-   Service Workers
    
-   Page crashes
    

This chapter covers all of those.

----------

# Part 10 – Events

# Chapter 3 – Page Events

----------

# Introduction

Every web page continuously generates events.

For example:

```text
Page Opens
      │
      ▼
DOM Loaded
      │
      ▼
JavaScript Executes
      │
      ▼
Console Logs
      │
      ▼
Network Requests
      │
      ▼
User Uploads File
      │
      ▼
WebSocket Opens
      │
      ▼
Page Closes

```

Playwright allows us to observe each of these events.

----------

# Page Event Categories

The Page object exposes many events.

Category

Events

Lifecycle

load, domcontentloaded

Browser

close, crash

JavaScript

console, pageerror

File Upload

filechooser

Background

worker

Communication

websocket

We'll examine each one.

----------

# load Event

## What is the load event?

The browser fires the **load** event after:

-   HTML is loaded
    
-   CSS is loaded
    
-   Images are loaded
    
-   JavaScript files are loaded
    

Diagram:

```text
HTML
   │
CSS
   │
Images
   │
Scripts
   │
LOAD EVENT

```

----------

## Listening for Load

```typescript
page.on('load', () => {

    console.log('Page Loaded');

});

```

----------

## Example

```typescript
page.on('load', () => {

    console.log('Dashboard Ready');

});

await page.goto('/dashboard');

```

----------

## When to Use

Useful when:

-   Measuring page loading
    
-   Logging navigation
    
-   Framework utilities
    

Most application tests don't need this because `page.goto()` and Playwright's assertions already synchronize appropriately.

----------

# domcontentloaded Event

This occurs earlier than the load event.

Sequence:

```text
HTML Parsed

↓

DOM Created

↓

DOMContentLoaded

↓

Images Continue Loading

↓

Load

```

----------

## Example

```typescript
page.on('domcontentloaded', () => {

    console.log('DOM Ready');

});

```

----------

## Difference

Event

Waits For

domcontentloaded

HTML parsing

load

HTML + CSS + images + scripts

----------

# close Event

Triggered when a page closes.

Example

```typescript
page.on('close', () => {

    console.log('Page Closed');

});

```

----------

Real Example

```typescript
const popup =
await page.waitForEvent('popup');

popup.on('close', () => {

    console.log('Popup Closed');

});

```

Useful for:

-   OAuth
    
-   Payment windows
    
-   Report windows
    

----------

# crash Event

Sometimes a page crashes.

Reasons:

-   Browser crash
    
-   Out of memory
    
-   Renderer failure
    

Playwright exposes this.

```typescript
page.on('crash', () => {

    console.log('Page Crashed');

});

```

----------

Real Example

```typescript
page.on('crash', () => {

    throw new Error(
        'Browser crashed'
    );

});

```

Useful in long-running automation suites.

----------

# console Event

One of the most useful debugging events.

Every browser console message can be captured.

```typescript
page.on('console', message => {

    console.log(
        message.text()
    );

});

```

----------

Example Output

Suppose the application executes:

```javascript
console.log('User Logged In');

```

Playwright prints:

```text
User Logged In

```

----------

# Console Types

Each console message has a type.

```typescript
page.on('console', message => {

    console.log(

        message.type(),

        message.text()

    );

});

```

Possible values:

```text
log

warning

error

info

debug

```

----------

# Monitoring JavaScript Errors

```typescript
page.on('console', message => {

    if (message.type() === 'error') {

        console.log(

            message.text()

        );

    }

});

```

Very useful for identifying frontend issues.

----------

# pageerror Event

This is different from the `console` event.

It captures **uncaught JavaScript exceptions**.

Example

```javascript
throw new Error('Unexpected Error');

```

Playwright

```typescript
page.on('pageerror', error => {

    console.log(error.message);

});

```

----------

Real Example

```typescript
page.on('pageerror', error => {

    expect(error).toBeUndefined();

});

```

Useful for monitoring application stability during tests.

----------

# console vs pageerror

console

pageerror

Captures console output

Captures uncaught exceptions

Includes `console.log()`

Includes thrown errors

Logging

Application failures

----------

# filechooser Event

Suppose clicking a button opens a file picker.

```text
Upload File

↓

File Chooser Opens

```

Playwright exposes this event.

----------

## Example

```typescript
const chooserPromise =
page.waitForEvent(
'filechooser'
);

await page.click('#upload');

const chooser =
await chooserPromise;

```

----------

Selecting a file:

```typescript
await chooser.setFiles(
'test-data/sample.pdf'
);

```

----------

Real Example

```typescript
const chooserPromise =
page.waitForEvent(
'filechooser'
);

await page.getByRole('button',{
name:'Upload'
})
.click();

const chooser =
await chooserPromise;

await chooser.setFiles(
'documents/resume.pdf'
);

```

----------

# worker Event

Modern web applications use Web Workers.

Purpose:

-   Background calculations
    
-   Data processing
    
-   Image processing
    

Playwright exposes worker creation.

```typescript
page.on('worker', worker => {

    console.log(

        worker.url()

    );

});

```

Useful for advanced debugging.

----------

# websocket Event

Modern applications frequently use WebSockets.

Examples:

-   Chat
    
-   Stock market
    
-   Live sports
    
-   Notifications
    

Playwright

```typescript
page.on(
'websocket',
websocket => {

    console.log(

        websocket.url()

    );

});

```

----------

Real Example

```text
Chat Application

↓

WebSocket Connected

↓

Messages Stream

```

Useful for monitoring live communication.

----------

# Combining Multiple Events

```typescript
page.on('console', message => {

    console.log(message.text());

});

page.on('pageerror', error => {

    console.log(error.message);

});

page.on('crash', () => {

    console.log('Crash');

});

```

Now your test monitors:

-   Console output
    
-   JavaScript errors
    
-   Browser crashes
    

----------

# Real-World Example – Application Health Monitor

```typescript
page.on('console', message => {

    if (message.type() === 'error') {

        console.log(

            'Console Error:',

            message.text()

        );

    }

});

page.on('pageerror', error => {

    console.log(

        'Unhandled Error:',

        error.message

    );

});

```

This provides valuable debugging information without changing the application code.

----------

# Event Timeline

```text
Navigation
      │
      ▼
DOMContentLoaded
      │
      ▼
Load
      │
      ▼
Console Messages
      │
      ▼
WebSocket Opens
      │
      ▼
Worker Starts
      │
      ▼
Page Closes

```

Not every application triggers every event, but this illustrates a common flow.

----------

# Best Practices

-   Monitor `pageerror` in critical end-to-end test suites to detect unexpected JavaScript exceptions.
    
-   Capture console errors when investigating flaky UI behavior.
    
-   Use `filechooser` for native file picker interactions instead of attempting to automate the operating system dialog.
    
-   Keep event listeners lightweight and remove them when they're no longer needed in long-lived utilities.
    
-   Log only the information that's useful for debugging to avoid overwhelming test output.
    

----------

# Common Mistakes

### ❌ Assuming console errors fail tests

By default, Playwright does **not** fail a test because the page logs a console error. If console errors are important, add explicit assertions or monitoring logic.

----------

### ❌ Confusing `console` with `pageerror`

-   `console` captures output such as `console.log()` and `console.error()`.
    
-   `pageerror` captures uncaught JavaScript exceptions.
    

----------

### ❌ Trying to automate the OS file picker

Use the `filechooser` event and `setFiles()` instead of interacting with the operating system's file dialog.

----------

### ❌ Ignoring page crashes

A crashed page may leave later test failures looking unrelated. Listen for the `crash` event in long-running or resource-intensive test scenarios.

----------

# Interview Questions

### Q1. What is the difference between `load` and `domcontentloaded`?

-   `domcontentloaded` fires after the HTML has been parsed and the DOM is ready.
    
-   `load` fires after the page's dependent resources (such as images and stylesheets) have finished loading.
    

----------

### Q2. How do you capture browser console messages?

```typescript
page.on('console', message => {

    console.log(message.text());

});

```

----------

### Q3. How do you detect JavaScript exceptions?

```typescript
page.on('pageerror', error => {

    console.log(error.message);

});

```

----------

### Q4. How do you automate file uploads that open a native file picker?

Wait for the `filechooser` event and then call:

```typescript
await chooser.setFiles('sample.pdf');

```

----------

### Q5. What is the `worker` event used for?

It notifies you when a new Web Worker is created, allowing advanced monitoring or debugging of background processing.

----------

# Summary

Page events provide visibility into everything happening inside a browser page—from lifecycle events and console output to JavaScript exceptions, file chooser interactions, Web Workers, WebSockets, and page crashes. While many day-to-day tests don't require all of these events, understanding them gives you powerful debugging capabilities and helps you build more resilient automation for complex web applications.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbNjQ4NTU4OTczXX0=
-->