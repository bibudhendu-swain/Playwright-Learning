----------
# Appendix A – Playwright Assertion Reference

> This appendix provides a quick reference for all commonly used Playwright assertions. Use it as a cheat sheet while writing tests.

----------

# Locator Assertions

## toBeAttached()

### Purpose

Verifies that an element is attached to the DOM.

### Syntax

```typescript
await expect(locator).toBeAttached();

```

### Example

```typescript
await expect(page.locator('#profile')).toBeAttached();

```

### Auto Retry

✅ Yes

### Common Use Cases

-   Dynamic elements
    
-   SPA applications
    
-   Lazy-loaded components
    

----------

## toBeChecked()

### Purpose

Verifies that a checkbox or radio button is selected.

### Syntax

```typescript
await expect(locator).toBeChecked();

```

### Example

```typescript
await expect(page.getByRole('checkbox')).toBeChecked();

```

### Auto Retry

✅ Yes

### Works With

-   Checkbox
    
-   Radio Button
    

----------

## toBeDisabled()

### Purpose

Checks whether an element is disabled.

```typescript
await expect(locator).toBeDisabled();

```

Commonly used for:

-   Submit buttons
    
-   Permission checks
    
-   Read-only forms
    

----------

## toBeEditable()

### Purpose

Verifies that a user can edit the field.

```typescript
await expect(locator).toBeEditable();

```

Useful for

-   Forms
    
-   Role-based access testing
    

----------

## toBeEmpty()

### Purpose

Checks that an element or input has no content.

```typescript
await expect(locator).toBeEmpty();

```

----------

## toBeEnabled()

### Purpose

Checks that an element is enabled.

```typescript
await expect(locator).toBeEnabled();

```

----------

## toBeFocused()

### Purpose

Checks keyboard focus.

```typescript
await expect(locator).toBeFocused();

```

Useful for

-   Accessibility
    
-   Keyboard navigation
    

----------

## toBeHidden()

### Purpose

Checks that an element is hidden.

```typescript
await expect(locator).toBeHidden();

```

Better than checking CSS properties like `display: none`.

----------

## toBeInViewport()

### Purpose

Checks that an element is currently visible within the browser viewport.

```typescript
await expect(locator).toBeInViewport();

```

Useful for:

-   Infinite scrolling
    
-   Lazy loading
    
-   Responsive layouts
    

----------

## toBeVisible()

### Purpose

Checks that an element is visible to the user.

```typescript
await expect(locator).toBeVisible();

```

This is the **most frequently used Playwright assertion**.

----------

# Text Assertions

----------

## toHaveText()

### Purpose

Checks exact text.

```typescript
await expect(locator)
.toHaveText("Success");

```

Supports:

-   String
    
-   Regular Expression
    
-   Array
    

----------

## toContainText()

### Purpose

Checks partial text.

```typescript
await expect(locator)
.toContainText("Success");

```

Use when the full text is dynamic.

----------

# Input Assertions

----------

## toHaveValue()

```typescript
await expect(locator)
.toHaveValue("admin");

```

For input fields.

----------

## toHaveValues()

```typescript
await expect(locator)
.toHaveValues([
"Java",
"TypeScript"
]);

```

Works with multi-select controls.

----------

# Collection Assertions

----------

## toHaveCount()

```typescript
await expect(locator)
.toHaveCount(5);

```

Preferred over:

```typescript
expect(await locator.count()).toBe(5);

```

because it auto-retries.

----------

# Attribute Assertions

----------

## toHaveAttribute()

```typescript
await expect(locator)
.toHaveAttribute(
'type',
'email'
);

```

----------

## toHaveId()

```typescript
await expect(locator)
.toHaveId('username');

```

----------

## toHaveRole()

```typescript
await expect(locator)
.toHaveRole('button');

```

Accessibility testing.

----------

# CSS Assertions

----------

## toHaveClass()

```typescript
await expect(locator)
.toHaveClass('active');

```

Regex supported.

----------

## toHaveCSS()

```typescript
await expect(locator)
.toHaveCSS(
'display',
'flex'
);

```

Useful for

-   Themes
    
-   Responsive testing
    
-   Dark mode
    

----------

# JavaScript Assertions

----------

## toHaveJSProperty()

```typescript
await expect(locator)
.toHaveJSProperty(
'checked',
true
);

```

Checks DOM properties.

----------

# Page Assertions

----------

## toHaveTitle()

```typescript
await expect(page)
.toHaveTitle(
'Dashboard'
);

```

----------

## toHaveURL()

```typescript
await expect(page)
.toHaveURL(
/dashboard/
);

```

Regex supported.

----------

# API Assertions

```typescript
expect(response.status())
.toBe(200);

expect(body.id)
.toBe(10);

expect(body.name)
.toContain("John");

```

These are standard value assertions and do **not** auto-retry.

----------

# Generic Assertions

----------

## Equality

```typescript
expect(value).toBe(5);

expect(value).toEqual(object);

```

----------

## Truthiness

```typescript
expect(value).toBeTruthy();

expect(value).toBeFalsy();

expect(value).toBeNull();

expect(value).toBeUndefined();

```

----------

## Numbers

```typescript
expect(price)
.toBeGreaterThan(100);

expect(price)
.toBeLessThan(500);

```

----------

## Arrays

```typescript
expect(array)
.toContain('Java');

```

----------

## Strings

```typescript
expect(name)
.toContain('Play');

```

----------

# Auto-Retry Support Matrix

Assertion Type

Auto Retry

Locator Assertions

✅

Page Assertions

✅

API Response Values

❌

Generic JavaScript Values

❌

----------

# Most Frequently Used Assertions

In day-to-day Playwright automation, you'll use these assertions far more than the others:

Assertion

Usage Frequency

`toBeVisible()`

⭐⭐⭐⭐⭐

`toHaveText()`

⭐⭐⭐⭐⭐

`toContainText()`

⭐⭐⭐⭐⭐

`toHaveValue()`

⭐⭐⭐⭐⭐

`toBeEnabled()`

⭐⭐⭐⭐⭐

`toHaveCount()`

⭐⭐⭐⭐

`toBeChecked()`

⭐⭐⭐⭐

`toBeHidden()`

⭐⭐⭐⭐

`toHaveURL()`

⭐⭐⭐⭐

`toHaveTitle()`

⭐⭐⭐

----------

# Assertion Selection Guide

Scenario

Recommended Assertion

Verify an element is displayed

`toBeVisible()`

Verify an element disappears

`toBeHidden()`

Verify exact text

`toHaveText()`

Verify partial text

`toContainText()`

Verify an input field

`toHaveValue()`

Verify a checkbox/radio

`toBeChecked()`

Verify a button is clickable

`toBeEnabled()`

Verify a button is disabled

`toBeDisabled()`

Verify the number of elements

`toHaveCount()`

Verify a page title

`toHaveTitle()`

Verify navigation

`toHaveURL()`

Verify an attribute

`toHaveAttribute()`

Verify a CSS property

`toHaveCSS()`

Verify a DOM property

`toHaveJSProperty()`

----------

# Comparison Cheat Sheet

Instead of

Prefer

`expect(await locator.textContent()).toBe(...)`

`await expect(locator).toHaveText(...)`

`expect(await locator.inputValue()).toBe(...)`

`await expect(locator).toHaveValue(...)`

`expect(await locator.count()).toBe(...)`

`await expect(locator).toHaveCount(...)`

`expect(await locator.isVisible()).toBe(true)`

`await expect(locator).toBeVisible()`

`expect(await locator.isEnabled()).toBe(true)`

`await expect(locator).toBeEnabled()`

These Playwright assertions automatically retry and are generally more reliable than asserting on values retrieved from the locator.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTUyMDk3MjI1N119
-->