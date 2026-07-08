**Frames** are one of the most important Playwright topics because many enterprise applications use them extensively for payment gateways, legacy applications, document viewers, embedded reports, advertisements, chat widgets, and third-party integrations.

Many automation engineers struggle with iframes because they don't fully understand **what a frame is**, **why it exists**, and **how Playwright handles it**.

----------

# Part 7 – Frames

# Chapter 1 – Working with Frames

----------

# Introduction

Modern web applications often embed one webpage inside another webpage.

Examples include:

-   Payment gateways (Stripe, PayPal)
    
-   Google Maps
    
-   YouTube videos
    
-   Chat widgets
    
-   Power BI dashboards
    
-   PDF viewers
    
-   Legacy applications
    

These embedded webpages are called **Frames (iframes)**.

----------

# What is a Frame?

A Frame (or iframe) is an HTML element that loads another webpage inside the current webpage.

Example:

```html
<iframe
    src="https://example.com/payment">
</iframe>

```

Visually:

```text
+--------------------------------------+
|              Main Page               |
|                                      |
|  Username                            |
|  Password                            |
|                                      |
|  +------------------------------+    |
|  |       Payment Page           |    |
|  |                              |    |
|  | Card Number                  |    |
|  | CVV                          |    |
|  | Expiry                       |    |
|  +------------------------------+    |
|                                      |
+--------------------------------------+

```

The payment section is actually another webpage.

----------

# Why Do Websites Use Frames?

Common reasons include:

-   Security
    
-   Third-party integrations
    
-   Legacy systems
    
-   Isolated applications
    
-   Sandboxing
    
-   Independent loading
    

Example:

```text
Main Website

↓

Stripe Payment

↓

Stripe controls everything

```

The payment page is isolated from the main application.

----------

# Main Frame vs Child Frame

Every webpage has one **Main Frame**.

Example:

```text
Main Frame
│
├── Header
├── Login Form
├── Footer

```

If an iframe exists:

```text
Main Frame
│
├── Header
├── Login Form
├── Payment Frame
│      ├── Card Number
│      ├── CVV
│      └── Expiry
└── Footer

```

----------

# How Playwright Represents Frames

Playwright provides two primary ways to work with frames:

1.  **frameLocator()** ✅ (Recommended)
    
2.  **Frame object**
    

----------

# frameLocator() (Recommended)

This is the preferred way to interact with iframes.

Example HTML

```html
<iframe id="payment-frame"></iframe>

```

Playwright

```typescript
await page
    .frameLocator('#payment-frame')
    .getByLabel('Card Number')
    .fill('4111111111111111');

```

Notice that everything after `frameLocator()` happens inside the iframe.

----------

# Why frameLocator()?

Suppose the iframe reloads.

Traditional automation:

```text
Locate Frame

↓

Store Reference

↓

Frame Reloads

↓

Reference Invalid

```

Playwright:

```text
frameLocator()

↓

Action

↓

Locate Frame Again

↓

Perform Action

```

Just like `Locator`, `frameLocator()` is lazy and automatically resolves the frame when needed.

----------

# Real-World Example – Payment Gateway

```html
<iframe id="payment">

```

```typescript
const payment =
page.frameLocator('#payment');

await payment
.getByLabel('Card Number')
.fill('4111111111111111');

await payment
.getByLabel('Expiry')
.fill('12/30');

await payment
.getByLabel('CVV')
.fill('123');

await payment
.getByRole('button',{
name:'Pay'
})
.click();

```

Very clean.

----------

# Frame Object

Sometimes you need direct access to the underlying Frame.

Example

```typescript
const frame =
page.frame({
    name:'payment'
});

```

or

```typescript
const frame =
page.frame({
    url:/payment/
});

```

Now

```typescript
await frame
.locator('#card')
.fill('4111111111111111');

```

----------

# When Should You Use Frame Objects?

Only when you need:

-   Frame metadata
    
-   Enumerating frames
    
-   Accessing frames dynamically
    
-   Low-level APIs
    

For normal automation,

prefer

```typescript
frameLocator()

```

----------

# Getting All Frames

```typescript
const frames =
page.frames();

console.log(frames.length);

```

Example output

```text
Main Frame

Payment Frame

Advertisement

Chat Widget

```

----------

# Getting the Main Frame

```typescript
const main =
page.mainFrame();

```

Useful for debugging.

----------

# Finding a Frame by Name

Example HTML

```html
<iframe
name="payment">

```

```typescript
const frame =
page.frame({
    name:'payment'
});

```

----------

# Finding a Frame by URL

```typescript
const frame =
page.frame({
url:/payment/
});

```

Very useful when IDs are dynamic.

----------

# Nested Frames

Frames can contain other frames.

Example

```text
Main Page
│
├── Frame A
│      │
│      └── Frame B
│              │
│              └── Button

```

Playwright handles nested frames elegantly.

Example

```typescript
await page
.frameLocator('#frameA')
.frameLocator('#frameB')
.getByRole('button',{
name:'Submit'
})
.click();

```

----------

# Dynamic Frames

Some applications load frames only after an action.

Example

```text
Click Checkout

↓

Payment Frame Appears

```

Playwright waits automatically.

```typescript
await page
.getByRole('button',{
name:'Checkout'
})
.click();

await page
.frameLocator('#payment')
.getByLabel('Card Number')
.fill('4111111111111111');

```

No explicit wait needed.

----------

# Cross-Origin Frames

Example

```text
Main Page

https://shop.com

↓

Frame

https://stripe.com

```

Different domains.

Playwright can still automate them.

Example

```typescript
await page
.frameLocator('iframe')
.getByLabel('Card Number')
.fill('4111111111111111');

```

Unlike direct JavaScript access, Playwright communicates with the browser rather than relying on the page's same-origin JavaScript permissions.

----------

# Waiting for Frames

Normally unnecessary.

Playwright waits automatically.

If required,

```typescript
await expect(
page
.frameLocator('#payment')
.getByRole('button',{
name:'Pay'
})
)
.toBeVisible();

```

----------

# Real-World Example – Stripe

```typescript
const stripe =
page.frameLocator(
'iframe[name="stripe"]'
);

await stripe
.getByPlaceholder('Card number')
.fill('4242424242424242');

await stripe
.getByPlaceholder('MM / YY')
.fill('12/30');

await stripe
.getByPlaceholder('CVC')
.fill('123');

```

----------

# Real-World Example – Embedded Report

```typescript
const report =
page.frameLocator('#report');

await expect(
report
.getByText('Revenue')
)
.toBeVisible();

```

----------

# Frame vs Locator

Locator

FrameLocator

Finds elements

Finds iframes

Works in current document

Works inside an iframe

Auto waits

Auto waits

Lazy

Lazy

----------

# FrameLocator vs Frame

FrameLocator

Frame

Recommended

Advanced API

Lazy

Immediate lookup

Better for tests

Better for framework utilities

Auto re-resolves

Represents a specific frame object

----------

# Best Practices

✅ Prefer `frameLocator()` over `Frame`.

✅ Use user-facing locators inside frames (`getByRole()`, `getByLabel()`, etc.).

✅ Avoid XPath inside iframes unless necessary.

✅ Don't add manual waits before interacting with frames.

✅ Use frame URL or name only when locating the iframe itself.

----------

# Common Mistakes

## ❌ Treating iframe elements like normal page elements

Bad

```typescript
await page
.getByLabel('Card Number')
.fill(...);

```

The field is inside an iframe.

Correct

```typescript
await page
.frameLocator('#payment')
.getByLabel('Card Number')
.fill(...);

```

----------

## ❌ Using Frame when FrameLocator is sufficient

```typescript
const frame =
page.frame(...);

```

Usually unnecessary.

Prefer

```typescript
page.frameLocator(...)

```

----------

## ❌ Waiting manually

```typescript
await page.waitForTimeout(5000);

```

Playwright already waits for the frame and the target element.

----------

# Interview Questions

### Q1. What is an iframe?

An iframe is an HTML element that embeds another webpage within the current page. It has its own document and DOM.

----------

### Q2. What is the difference between `frameLocator()` and `page.frame()`?

-   `frameLocator()` is lazy, auto-waiting, and recommended for writing tests.
    
-   `page.frame()` returns a `Frame` object and is typically used for advanced or framework-level scenarios.
    

----------

### Q3. Can Playwright automate cross-origin iframes?

Yes. Playwright can interact with elements inside cross-origin iframes using `frameLocator()` or `Frame`, even though browser JavaScript is subject to same-origin restrictions.

----------

### Q4. How do you locate a frame?

Common options include:

```typescript
page.frameLocator('#payment');

```

or

```typescript
page.frame({
    name: 'payment'
});

```

or

```typescript
page.frame({
    url: /payment/
});

```

----------

### Q5. Can frames be nested?

Yes. `frameLocator()` calls can be chained to access elements inside nested iframes.

----------

# Summary

Frames are isolated documents embedded within a web page. Playwright's `frameLocator()` API makes interacting with them as simple as working with regular locators while preserving automatic waiting and lazy resolution. By understanding the distinction between `frameLocator()` and `Frame`, and knowing how to handle nested and cross-origin frames, you can confidently automate common enterprise scenarios such as payment gateways, embedded reports, and third-party integrations.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTgxMDcwOTIwXX0=
-->