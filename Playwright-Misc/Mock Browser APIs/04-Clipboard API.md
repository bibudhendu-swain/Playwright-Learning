The Clipboard API is used much more often than people realize.

Applications like:

-   Banking portals (Copy Account Number)
    
-   AWS Console (Copy Access Key)
    
-   Azure Portal (Copy Connection String)
    
-   GitHub (Clone URL)
    
-   Docker Hub (Pull Command)
    
-   Kubernetes Dashboard (Copy YAML)
    
-   OTP Applications
    
-   Coupon Websites
    

all depend on the Clipboard API.

Fortunately, Playwright can test these scenarios reliably.

----------

# Part 17 – Mock Browser APIs

# Chapter 4 – Clipboard API

----------

# Introduction

Modern web applications frequently allow users to copy information instead of typing it manually.

For example:

```text
User Clicks Copy

↓

Application Writes

↓

Clipboard

↓

User Pastes

```

Examples include:

-   Copy Coupon Code
    
-   Copy Invite Link
    
-   Copy API Token
    
-   Copy Account Number
    
-   Copy Git Command
    
-   Copy SQL Query
    

Testing these features manually is tedious.

Playwright enables us to verify clipboard operations automatically.

----------

# What is the Clipboard API?

The Clipboard API allows web applications to:

-   Read data from the clipboard
    
-   Write data to the clipboard
    

The browser exposes this functionality through:

```typescript
navigator.clipboard

```

----------

# Clipboard Flow

Writing

```text
Application

↓

Clipboard API

↓

Clipboard

```

Reading

```text
Clipboard

↓

Clipboard API

↓

Application

```

----------

# Browser Security

For security reasons,

clipboard access usually requires:

-   Secure Context (HTTPS or localhost)
    
-   User interaction
    
-   Browser Permission
    

Without permission,

the browser may reject clipboard access.

----------

# Grant Clipboard Permission

```typescript
const context =
await browser.newContext();

await context.grantPermissions([

    "clipboard-read",

    "clipboard-write"

]);

```

Now clipboard operations are allowed.

----------

# Writing to Clipboard

Application code

```typescript
await navigator.clipboard.writeText(

"Playwright Handbook"

);

```

The browser stores:

```text
Playwright Handbook

```

inside the clipboard.

----------

# Reading Clipboard

Application

```typescript
const value =

await navigator.clipboard.readText();

```

Returns

```text
Playwright Handbook

```

----------

# Testing Copy Button

Suppose the application has

```text
[ Copy Coupon ]

```

Workflow

```text
Click Button

↓

Write Clipboard

↓

Read Clipboard

↓

Verify

```

----------

## Example

```typescript
await context.grantPermissions([

    "clipboard-read",

    "clipboard-write"

]);

await page.goto("/coupon");

await page.getByRole("button", {

    name: "Copy"

}).click();

const copied =

await page.evaluate(async () => {

    return navigator.clipboard.readText();

});

expect(copied)

.toBe("SAVE20");

```

----------

# Testing API Key Copy

Application

```text
Generate API Key

↓

Copy

↓

Clipboard

```

Test

```typescript
await page.getByRole("button", {

    name: "Copy API Key"

}).click();

const apiKey =

await page.evaluate(async () => {

    return navigator.clipboard.readText();

});

expect(apiKey)

.toContain("sk-");

```

----------

# Testing Invite Link

Application

```text
Invite User

↓

Copy Link

↓

Clipboard

```

Example

```typescript
const link =

await page.evaluate(async () => {

    return navigator.clipboard.readText();

});

expect(link)

.toContain("/invite/");

```

----------

# Testing Copy Coupon

Application

```text
SAVE20

↓

Copy

↓

Clipboard

```

Verify

```typescript
expect(copied)

.toBe("SAVE20");

```

----------

# Writing Clipboard During Test

Sometimes you want to pre-populate the clipboard.

Example

```typescript
await page.evaluate(async () => {

    await navigator.clipboard.writeText(

        "Hello Playwright"

    );

});

```

Now

```text
Clipboard

↓

Hello Playwright

```

----------

# Testing Paste Operation

Suppose the application pastes clipboard content into an input.

Workflow

```text
Clipboard

↓

Paste

↓

Textbox

```

Setup

```typescript
await page.evaluate(async () => {

    await navigator.clipboard.writeText(

        "Playwright"

    );

});

```

Paste using the browser:

```typescript
await page.getByRole("textbox").click();

await page.keyboard.press(
    "Control+V"
);

```

Verify

```typescript
await expect(

page.getByRole("textbox")

).toHaveValue("Playwright");

```

> **Cross-platform note:** On macOS, use `Meta+V` instead of `Control+V`.

----------

# Testing Copy Multiple Times

Application

```text
Copy

↓

Clipboard Updated

↓

Copy Again

↓

Clipboard Updated

```

Example

```typescript
await page.evaluate(async () => {

    await navigator.clipboard.writeText(

        "First"

    );

});

await page.evaluate(async () => {

    await navigator.clipboard.writeText(

        "Second"

    );

});

const value =

await page.evaluate(async () => {

    return navigator.clipboard.readText();

});

expect(value)

.toBe("Second");

```

----------

# Reading Clipboard After User Action

Instead of manually writing,

verify application behavior.

```typescript
await page.getByRole("button", {

    name: "Copy"

}).click();

const copied =

await page.evaluate(async () =>

    navigator.clipboard.readText()

);

```

This confirms the application—not the test—placed the expected value on the clipboard.

----------

# Clipboard Permission Denied

Suppose permissions are not granted.

```typescript
const context =

await browser.newContext();

```

Application

```typescript
navigator.clipboard.writeText(...)

```

May result in:

```text
Permission Denied

```

Verify the application handles this gracefully.

----------

# Enterprise Example – Banking

```text
Copy Account Number

↓

Clipboard

↓

Paste Payment Form

```

Verify

-   Correct value copied
    
-   No extra spaces
    
-   Correct formatting
    

----------

# Enterprise Example – Developer Portal

```text
Copy Connection String

↓

Clipboard

↓

Paste IDE

```

Verify

-   Entire string copied
    
-   No truncation
    

----------

# Enterprise Example – Coupon

```text
SAVE20

↓

Copy

↓

Checkout

↓

Paste

↓

Discount Applied

```

One end-to-end workflow.

----------

# Enterprise Example – GitHub

```text
Copy Clone URL

↓

Clipboard

↓

Paste Terminal

```

Verify

```text
git clone

```

is present.

----------

# Enterprise Clipboard Helper

Instead of

```typescript
await page.evaluate(...);

await page.evaluate(...);

await page.evaluate(...);

```

Create helper methods.

```typescript
class ClipboardHelper {

    static async read(page){

        return await page.evaluate(

            ()=>

            navigator.clipboard.readText()

        );

    }

}

```

Usage

```typescript
const value =

await ClipboardHelper.read(page);

```

Much cleaner.

----------

# Suggested Folder Structure

```text
helpers/

├── ClipboardHelper.ts

├── PermissionManager.ts

└── BrowserContextFactory.ts

```

----------

# Clipboard vs Keyboard

Copy

```text
Clipboard

↓

Ctrl+C

↓

Stored

```

Paste

```text
Clipboard

↓

Ctrl+V

↓

Textbox

```

Playwright can verify both the clipboard contents and the resulting UI state.

----------

# Best Practices

-   Grant clipboard permissions before testing clipboard features.
    
-   Verify clipboard contents instead of assuming the copy action succeeded.
    
-   Test both copy and paste workflows when applicable.
    
-   Use helper methods to reduce repeated clipboard code.
    
-   Keep tests platform-aware when simulating keyboard shortcuts.
    

----------

# Common Mistakes

### ❌ Forgetting Clipboard Permission

Without

```typescript
await context.grantPermissions([

    "clipboard-read",

    "clipboard-write"

]);

```

clipboard access may fail.

----------

### ❌ Verifying Only Toast Messages

Application

```text
Copied Successfully

```

does **not** guarantee the correct value was copied.

Always verify clipboard content.

----------

### ❌ Hardcoding Platform Shortcuts

Windows/Linux

```text
Control+V

```

macOS

```text
Meta+V

```

Choose the correct shortcut for the environment under test.

----------

### ❌ Ignoring Browser Security

Clipboard operations generally require a secure context (HTTPS or localhost) and appropriate permissions.

----------

# Interview Questions

### Q1. Which browser object provides Clipboard API access?

```typescript
navigator.clipboard

```

----------

### Q2. How do you verify that a Copy button actually copied the expected value?

Click the button, then read the clipboard using:

```typescript
await page.evaluate(async () => {
    return navigator.clipboard.readText();
});

```

and compare it with the expected value.

----------

### Q3. Why should clipboard permissions be granted before testing clipboard functionality?

Because browsers typically restrict clipboard access for security reasons. Without permission, read or write operations may be rejected.

----------

### Q4. Why is checking the clipboard better than checking a "Copied Successfully" toast?

A success message only indicates that the application attempted the copy. Reading the clipboard verifies that the correct data was actually copied.

----------

### Q5. What kinds of enterprise applications commonly use the Clipboard API?

Developer portals, banking applications, e-commerce sites, cloud platforms, CI/CD dashboards, collaboration tools, and any application that allows users to copy tokens, URLs, commands, or codes.

----------

# Summary

The Clipboard API enables applications to copy and paste data efficiently, and Playwright makes it straightforward to validate these interactions. By granting the appropriate permissions, verifying actual clipboard contents, and testing both copy and paste workflows, you can confidently automate features such as coupon codes, API keys, invite links, account numbers, and developer commands. Reusable clipboard helpers further improve the maintainability of enterprise automation frameworks.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMzA5MDg3NTI2XX0=
-->