The next topic is **Dialogs**.

Although dialogs are a relatively small Playwright feature, they're extremely common in real applications. You'll encounter them in scenarios such as:

-   Delete confirmation
    
-   Unsaved changes warning
    
-   Password reset confirmation
    
-   Browser alerts
    
-   Print dialogs
    
-   File downloads
    
-   Legacy JavaScript applications
    

One important thing to understand is that **browser dialogs are different from application modals**.

----------

# Part 8 – Dialogs

# Chapter 1 – Handling Browser Dialogs

----------

# Introduction

A **browser dialog** is a native dialog displayed by the browser itself.

Examples include:

-   `alert()`
    
-   `confirm()`
    
-   `prompt()`
    
-   `beforeunload`
    

Unlike HTML popups or modals, browser dialogs are **not part of the DOM**.

For example:

```javascript
alert('Saved Successfully');

```

This creates a browser dialog.

----------

# Browser Dialog vs HTML Modal

Many engineers confuse these two.

## Browser Dialog

Created using JavaScript.

```javascript
alert('Hello');

```

Looks like:

```text
-------------------------
|        Alert          |
|-----------------------|
| Hello                 |
|                       |
|         OK            |
-------------------------

```

Not part of the webpage.

----------

## HTML Modal

Created using HTML.

```html
<div class="modal">

Delete Record?

<button>Yes</button>

<button>No</button>

</div>

```

This **is part of the DOM**.

You interact with it like any other element.

```typescript
await page.getByRole('button', {
    name: 'Yes'
}).click();

```

----------

# How Playwright Handles Dialogs

Playwright exposes browser dialogs through the `dialog` event.

Whenever a dialog appears:

```text
Page

↓

Dialog Event

↓

Dialog Object

```

----------

# Dialog Object

The dialog object provides:

Method

Purpose

`accept()`

Click OK

`dismiss()`

Click Cancel

`message()`

Read dialog text

`type()`

Dialog type

`defaultValue()`

Prompt default value

----------

# Listening for Dialogs

General pattern:

```typescript
page.on('dialog', async dialog => {

    console.log(dialog.message());

    await dialog.accept();

});

```

Once the listener is registered,

any browser dialog is handled automatically.

----------

# alert()

## What is an Alert?

JavaScript:

```javascript
alert('Order Saved');

```

User only has:

```text
OK

```

----------

## Handling an Alert

```typescript
page.on('dialog', async dialog => {

    expect(dialog.type()).toBe('alert');

    expect(dialog.message()).toBe('Order Saved');

    await dialog.accept();

});

await page.getByRole('button', {
    name: 'Save'
}).click();

```

----------

## Real-World Example

Application:

```text
Click Save

↓

alert("Saved Successfully")

```

Playwright:

```typescript
page.on('dialog', async dialog => {

    expect(dialog.message())
        .toContain('Saved');

    await dialog.accept();

});

await page.click('#save');

```

----------

# confirm()

A confirmation dialog provides two choices:

```text
OK

Cancel

```

JavaScript:

```javascript
confirm('Delete this record?');

```

----------

## Accept

```typescript
page.on('dialog', async dialog => {

    await dialog.accept();

});

```

Equivalent to clicking **OK**.

----------

## Dismiss

```typescript
page.on('dialog', async dialog => {

    await dialog.dismiss();

});

```

Equivalent to clicking **Cancel**.

----------

## Example

```typescript
page.on('dialog', async dialog => {

    expect(dialog.type()).toBe('confirm');

    await dialog.dismiss();

});

await page.getByRole('button', {
    name: 'Delete'
}).click();

```

----------

# prompt()

A prompt asks the user to enter text.

JavaScript:

```javascript
prompt('Enter your name');

```

----------

## Accept with Value

```typescript
page.on('dialog', async dialog => {

    await dialog.accept('John Doe');

});

```

Playwright enters the text and clicks **OK**.

----------

## Dismiss

```typescript
page.on('dialog', async dialog => {

    await dialog.dismiss();

});

```

Equivalent to pressing **Cancel**.

----------

## Example

```typescript
page.on('dialog', async dialog => {

    expect(dialog.type()).toBe('prompt');

    await dialog.accept('Playwright');

});

await page.click('#rename');

```

----------

# Reading Dialog Information

## Dialog Type

```typescript
dialog.type();

```

Returns:

```text
alert

confirm

prompt

beforeunload

```

----------

## Message

```typescript
dialog.message();

```

Example:

```text
Delete this user?

```

----------

## Default Prompt Value

```typescript
dialog.defaultValue();

```

Useful only for prompt dialogs.

----------

# beforeunload Dialog

This dialog appears when a page attempts to close or reload while there are unsaved changes.

Example:

```text
Changes you made may not be saved.

```

----------

## Example

```typescript
page.on('dialog', async dialog => {

    expect(dialog.type())
        .toBe('beforeunload');

    await dialog.accept();

});

await page.close({ runBeforeUnload: true });

```

----------

## Real-World Scenario

```text
Edit Profile

↓

Don't Save

↓

Close Tab

↓

beforeunload Dialog

```

Useful for testing data-loss prevention.

----------

# Print Dialog

Browsers display a native print dialog when:

```javascript
window.print();

```

is called.

----------

## Can Playwright Control the Print Dialog?

No.

The print dialog is an operating system/browser UI outside the webpage, so Playwright cannot click its buttons.

Instead, verify that the application invoked `window.print()`.

----------

## Example

```typescript
await page.evaluate(() => {
    window.print = () => {
        console.log('Print invoked');
    };
});

await page.getByRole('button', {
    name: 'Print'
}).click();

```

This confirms that the print function was called without opening the native dialog.

----------

# Only One Dialog at a Time

Browser dialogs are modal.

```text
Page

↓

Dialog

↓

User must respond

↓

Execution continues

```

The page is blocked until the dialog is accepted or dismissed.

----------

# What Happens if You Don't Handle a Dialog?

Suppose:

```javascript
alert('Hello');

```

and your test never accepts it.

The page becomes blocked.

Subsequent actions fail because the browser is waiting for the dialog to be resolved.

----------

# Waiting for a Dialog

Instead of using a permanent listener, you can wait for a single dialog.

```typescript
const dialogPromise =
    page.waitForEvent('dialog');

await page.click('#delete');

const dialog = await dialogPromise;

expect(dialog.message())
    .toContain('Delete');

await dialog.accept();

```

This is useful when only one dialog is expected.

----------

# Global Dialog Listener

Sometimes every dialog should be accepted.

```typescript
page.on('dialog', async dialog => {

    await dialog.accept();

});

```

Useful for legacy applications that display many alerts.

----------

# Real-World Example – Delete Confirmation

```typescript
page.on('dialog', async dialog => {

    expect(dialog.type()).toBe('confirm');

    expect(dialog.message())
        .toBe('Delete Customer?');

    await dialog.accept();

});

await page.getByRole('button', {
    name: 'Delete'
}).click();

await expect(
    page.getByText('Customer Deleted')
).toBeVisible();

```

----------

# Best Practices

-   Register the dialog listener **before** triggering the action.
    
-   Validate the dialog message whenever it represents an important business rule.
    
-   Use `page.waitForEvent('dialog')` for one-time dialogs.
    
-   Use `page.on('dialog')` when the application displays dialogs repeatedly.
    
-   Treat HTML modals and browser dialogs differently—they require different automation approaches.
    

----------

# Common Mistakes

### ❌ Registering the listener too late

```typescript
await page.click('#delete');

page.on('dialog', async dialog => {
    await dialog.accept();
});

```

The dialog may already have appeared.

Correct:

```typescript
page.on('dialog', async dialog => {
    await dialog.accept();
});

await page.click('#delete');

```

----------

### ❌ Trying to locate browser dialogs

```typescript
await page.locator('.dialog').click();

```

Browser dialogs are **not** DOM elements.

----------

### ❌ Treating HTML modals as browser dialogs

If it's built with HTML and CSS, interact with it using locators—not the `dialog` event.

----------

# Interview Questions

### Q1. What is the difference between a browser dialog and an HTML modal?

-   A **browser dialog** (`alert`, `confirm`, `prompt`) is native browser UI and isn't part of the DOM.
    
-   An **HTML modal** is built using HTML/CSS and is part of the page, so it is automated with locators.
    

----------

### Q2. How do you accept an alert in Playwright?

```typescript
page.on('dialog', async dialog => {
    await dialog.accept();
});

```

----------

### Q3. How do you dismiss a confirmation dialog?

```typescript
await dialog.dismiss();

```

----------

### Q4. Can Playwright automate the browser's print dialog?

No. The native print dialog is outside the webpage. Instead, verify that `window.print()` was invoked.

----------

### Q5. Why should the dialog listener be registered before clicking?

Because dialogs appear immediately after the triggering action. Registering the listener first ensures the event is captured.

----------

# Summary

Browser dialogs are native browser features that temporarily block user interaction until they are accepted or dismissed. Playwright provides a simple event-driven API for handling alerts, confirmations, prompts, and `beforeunload` dialogs. By understanding the distinction between browser dialogs and HTML modals, and by registering listeners before triggering actions, you can reliably automate dialog-based workflows while avoiding common synchronization issues.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTc1MTM0MDczN119
-->