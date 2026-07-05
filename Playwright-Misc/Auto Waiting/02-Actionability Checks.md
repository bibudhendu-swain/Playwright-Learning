This chapter covers **how Playwright decides whether an element is actually ready for interaction**. Many engineers know that Playwright "auto waits," but few understand **what it is waiting for**. Understanding these actionability checks will help you diagnose almost every interaction failure.

----------

# Chapter: Auto Waiting (Part 2) – Actionability Checks

----------

# Introduction

When you perform an action such as:

```typescript
await page.getByRole('button', {
    name: 'Login'
}).click();

```

Playwright **does not immediately click** the button.

Instead, it verifies that the element is **actionable**.

An element is actionable only if it satisfies several conditions.

----------

# What are Actionability Checks?

Actionability checks are internal validations Playwright performs before executing an action.

Think of them as answering the question:

> **"Can a real user successfully interact with this element right now?"**

If the answer is **No**, Playwright waits.

If the answer is **Yes**, Playwright performs the action.

----------

# Actionability Flow

Internally, Playwright follows a sequence similar to:

```text
Locate Element
      │
      ▼
Attached?
      │
      ▼
Visible?
      │
      ▼
Stable?
      │
      ▼
Enabled?
      │
      ▼
Editable? (only for text input actions)
      │
      ▼
Receives Pointer Events?
      │
      ▼
Perform Action

```

If any check fails:

```text
Retry
     ↓
Retry
     ↓
Retry
     ↓
Timeout

```

----------

# Which Actions Perform Actionability Checks?

Most user interaction APIs.

Examples:

```typescript
click()

dblclick()

fill()

check()

uncheck()

hover()

dragTo()

tap()

selectOption()

pressSequentially()

```

Each action performs only the checks that are relevant.

For example:

Action

Visible

Stable

Enabled

Editable

Receives Events

click()

✅

✅

✅

❌

✅

fill()

✅

❌

✅

✅

❌

hover()

✅

✅

❌

❌

✅

check()

✅

✅

✅

❌

✅

----------

# Attached to the DOM

Before interacting with an element, Playwright verifies that it still exists.

Example:

```html
<button id="save">Save</button>

```

```typescript
await page.locator('#save').click();

```

If JavaScript removes the button:

```javascript
button.remove();

```

Playwright retries until:

-   The element reappears
    
-   Timeout occurs
    

----------

## Why This Matters

Dynamic frameworks such as:

-   React
    
-   Angular
    
-   Vue
    
-   Svelte
    

frequently recreate DOM elements during rendering.

Unlike Selenium, Playwright automatically resolves the locator again.

----------

# Visible

An element must be visible before most user interactions.

Playwright considers an element visible when it:

-   Exists in the DOM
    
-   Has a rendered size greater than zero
    
-   Is not hidden via CSS (`display: none` or `visibility: hidden`)
    
-   Is not fully collapsed
    

Example:

```typescript
await page.getByRole('button', {
    name: 'Submit'
}).click();

```

If the button is hidden:

```css
display: none;

```

Playwright waits until it becomes visible.

----------

## Example

```typescript
await page.getByRole('button', {
    name: 'Load'
}).click();

await expect(
    page.locator('.results')
).toBeVisible();

```

The click waits for the button.

The assertion waits for the results.

----------

# Stable

An element should not be moving while Playwright interacts with it.

Example:

A button slides into view.

```text
Frame 1

────────►

Frame 2

────────►

Frame 3

────────►

```

Playwright waits until the animation completes.

----------

## Example

Suppose a menu fades in.

```typescript
await page.getByRole('button', {
    name: 'Menu'
}).click();

```

If the animation lasts 300 ms:

Playwright waits until the element reaches a stable position before clicking.

----------

## Why Stability Matters

Without this check:

```text
Mouse Click

↓

Button moves

↓

Click misses target

```

----------

# Enabled

Playwright verifies that the control is enabled.

HTML:

```html
<button disabled>
    Submit
</button>

```

This action waits:

```typescript
await page.getByRole('button', {
    name: 'Submit'
}).click();

```

until:

```html
<button>
    Submit
</button>

```

----------

## Real-World Example

```typescript
await page.getByLabel('Username')
.fill('admin');

await page.getByLabel('Password')
.fill('password');

await page.getByRole('button', {
    name: 'Login'
}).click();

```

If the Login button becomes enabled only after both fields are valid, Playwright waits automatically.

----------

# Editable

Only relevant for actions that modify input values.

Example:

```typescript
await page.getByLabel('Email')
.fill('admin@example.com');

```

Playwright verifies:

-   Visible
    
-   Enabled
    
-   Editable
    

----------

## Read-only Example

```html
<input readonly>

```

This fails:

```typescript
await page.locator('input')
.fill('abc');

```

because the field is not editable.

----------

## Editable vs Enabled

Many engineers confuse these concepts.

Enabled

Editable

User can interact

User can modify text

Applies to buttons and inputs

Mainly applies to text-entry controls

Example:

```html
<input disabled>

```

Not enabled.

----------

```html
<input readonly>

```

Enabled, but not editable.

----------

# Receives Pointer Events

This is one of the most misunderstood checks.

Playwright verifies that the click can actually reach the element.

Example:

```text
Button

──────────

Loading Overlay

██████████

```

Although the button is visible,

the overlay intercepts mouse clicks.

Playwright waits.

----------

## Example

```typescript
await page.getByRole('button', {
    name: 'Checkout'
}).click();

```

If a loading overlay is covering the button,

Playwright waits until the overlay disappears.

----------

## Common Scenario

```css
.loading-overlay {
    position: absolute;
}

```

Until the overlay is removed,

clicking would fail.

Playwright prevents this.

----------

# Actionability Sequence by Action

## click()

Checks:

```text
Attached

↓

Visible

↓

Stable

↓

Enabled

↓

Receives Events

```

----------

## fill()

Checks:

```text
Attached

↓

Visible

↓

Enabled

↓

Editable

```

Notice that `fill()` does **not** require pointer event checks because it sets the input value rather than clicking it.

----------

## hover()

Checks:

```text
Attached

↓

Visible

↓

Stable

↓

Receives Events

```

----------

## check()

Checks:

```text
Attached

↓

Visible

↓

Stable

↓

Enabled

↓

Receives Events

```

----------

# Force Actions

Sometimes you intentionally want to bypass actionability checks.

Example:

```typescript
await page.getByRole('button', {
    name: 'Delete'
}).click({
    force: true
});

```

This skips several actionability checks and attempts the action immediately.

----------

## When Should `force: true` Be Used?

Very rarely.

Acceptable scenarios:

-   Testing intentionally hidden controls.
    
-   Interacting with custom UI libraries where you've confirmed the application behavior.
    
-   Working around a known application bug during investigation.
    

It should **not** be used to "fix" flaky tests.

----------

## Bad Practice

```typescript
await page.click('#save', {
    force: true
});

```

without understanding **why** the normal click failed.

Instead, investigate the underlying issue.

----------

# Debugging Actionability Failures

Suppose Playwright reports:

```text
Timeout 30000ms exceeded

waiting for locator(...)

```

Don't immediately increase the timeout.

Ask these questions:

### Is the element attached?

```typescript
await expect(locator)
.toBeAttached();

```

----------

### Is it visible?

```typescript
await expect(locator)
.toBeVisible();

```

----------

### Is it enabled?

```typescript
await expect(locator)
.toBeEnabled();

```

----------

### Is another element covering it?

Use the Playwright Inspector:

```bash
npx playwright test --debug

```

or

```bash
npx playwright codegen

```

Inspect the page to identify overlays, animations, or unexpected UI states.

----------

# Real-World Example – Loading Overlay

Application flow:

```text
Click Save

↓

Loading Overlay Appears

↓

API Call

↓

Overlay Disappears

↓

Button Clickable

```

Playwright:

```typescript
await page.getByRole('button', {
    name: 'Save'
}).click();

await expect(
    page.getByText('Saved Successfully')
).toBeVisible();

```

No manual waiting is required.

----------

# Best Practices

-   Trust Playwright's actionability checks before adding custom waits.
    
-   Treat timeouts as a sign to investigate the application's state rather than simply increasing the timeout.
    
-   Use assertions to verify business outcomes after actions.
    
-   Avoid `force: true` unless you've confirmed it's the correct solution.
    
-   Keep locators precise so Playwright can consistently identify the intended element.
    

----------

# Common Mistakes

### ❌ Adding fixed delays before every click

```typescript
await page.waitForTimeout(3000);

await page.click('#login');

```

Usually unnecessary.

----------

### ❌ Using `force: true` to hide synchronization issues

```typescript
await page.click('#submit', {
    force: true
});

```

This bypasses Playwright's safety checks and may create flaky tests.

----------

### ❌ Assuming visible means clickable

An element may be visible but still covered by another element.

Visibility and pointer-event availability are different concepts.

----------

### ❌ Increasing timeouts without investigation

Longer timeouts rarely solve the root cause. Determine **which actionability check is failing**.

----------

# Interview Questions

### Q1. What are Playwright's actionability checks?

Before performing most actions, Playwright verifies that the element is attached, visible, stable, enabled, editable (when applicable), and able to receive pointer events.

----------

### Q2. Why does Playwright wait for an element to become stable?

To avoid interacting with elements that are still moving due to animations or layout changes, which could result in missed or incorrect interactions.

----------

### Q3. What is the difference between `enabled` and `editable`?

-   **Enabled** means the control accepts user interaction.
    
-   **Editable** means its value can be modified, which mainly applies to text-entry controls.
    

----------

### Q4. Why might a visible element still not be clickable?

Another element (such as a loading overlay or modal backdrop) may be intercepting pointer events, preventing the click from reaching the target.

----------

### Q5. When should `force: true` be used?

Only in exceptional situations where bypassing actionability checks is intentional and understood. It should not be used as a general solution for flaky tests.

----------

# Summary

Actionability checks are the foundation of Playwright's reliability. Before interacting with an element, Playwright validates that it is attached, visible, stable, enabled, editable when appropriate, and capable of receiving user input. These checks eliminate many timing issues that previously required custom waits in other automation frameworks. Understanding each check helps you diagnose failures quickly and write automation that behaves much more like a real user.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5MDQwMDY5NjddfQ==
-->