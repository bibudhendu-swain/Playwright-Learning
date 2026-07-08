This chapter is one of the **most important chapters in the entire Playwright handbook** because these are the assertions you'll use in almost every test.

----------

# Chapter: Assertions (Part 2) – Locator Assertions

----------

# Introduction

Locator assertions verify the **state of UI elements**.

Unlike normal JavaScript assertions, locator assertions:

-   Auto-retry
    
-   Auto-wait
    
-   Work directly with Playwright Locators
    
-   Reduce flaky tests
    

Example:

```typescript
await expect(page.getByRole('button')).toBeVisible();

```

Playwright keeps checking until either:

-   Button becomes visible
    
-   Timeout occurs
    

----------

# Locator Assertion Categories

Locator assertions can be grouped into:

Category

Assertions

Visibility

toBeVisible(), toBeHidden(), toBeInViewport()

State

toBeEnabled(), toBeDisabled(), toBeEditable(), toBeEmpty(), toBeFocused(), toBeAttached()

Checkbox

toBeChecked()

Text

toHaveText(), toContainText()

Attributes

toHaveAttribute(), toHaveId(), toHaveRole()

CSS

toHaveCSS(), toHaveClass()

Input

toHaveValue(), toHaveValues()

JavaScript

toHaveJSProperty()

Collections

toHaveCount()

----------

# Visibility Assertions

----------

# toBeVisible()

Verifies that an element is:

-   Attached to DOM
    
-   Visible
    
-   Not hidden
    
-   Has non-zero size
    

Example

```typescript
await expect(
    page.getByRole('button', {
        name: 'Login'
    })
).toBeVisible();

```

----------

Real-world Example

```typescript
await page.getByRole('button', {
    name: 'Submit'
}).click();

await expect(
    page.getByText('Order Successful')
).toBeVisible();

```

Playwright waits automatically.

----------

Use Cases

-   Success messages
    
-   Error messages
    
-   Popups
    
-   Dialogs
    
-   Buttons
    
-   Menus
    

----------

# toBeHidden()

Opposite of visible.

```typescript
await expect(
    page.locator('.spinner')
).toBeHidden();

```

Very common.

Example

```typescript
await page.getByRole('button', {
    name: 'Search'
}).click();

await expect(
    page.locator('.loading')
).toBeHidden();

```

----------

# toBeInViewport()

Checks if an element is currently inside the visible browser viewport.

Example

```typescript
await expect(
    page.locator('#footer')
).toBeInViewport();

```

Useful for

-   Lazy loading
    
-   Infinite scrolling
    
-   Responsive UI
    

----------

# State Assertions

----------

# toBeAttached()

Checks whether the element exists in the DOM.

```typescript
await expect(
    page.locator('#profile')
).toBeAttached();

```

Useful when JavaScript dynamically creates elements.

----------

# toBeEnabled()

```typescript
await expect(
    page.getByRole('button', {
        name: 'Save'
    })
).toBeEnabled();

```

----------

Real Example

```typescript
await page.getByLabel('Username')
.fill('admin');

await page.getByLabel('Password')
.fill('password');

await expect(
page.getByRole('button',{
name:'Login'
})
).toBeEnabled();

```

----------

# toBeDisabled()

```typescript
await expect(
page.getByRole('button',{
name:'Submit'
})
).toBeDisabled();

```

Example

Before entering mandatory fields

Button remains disabled.

----------

# toBeEditable()

Checks that user can edit an input.

```typescript
await expect(
page.getByLabel('Username')
).toBeEditable();

```

Useful for

-   Read-only fields
    
-   Permission testing
    

----------

# toBeEmpty()

Checks that input or element has no content.

```typescript
await expect(
page.getByRole('textbox')
).toBeEmpty();

```

Example

```typescript
await page.goto('/create');

await expect(
page.getByLabel('Comments')
).toBeEmpty();

```

----------

# toBeFocused()

Checks keyboard focus.

```typescript
await page.getByLabel('Email')
.focus();

await expect(
page.getByLabel('Email')
).toBeFocused();

```

Useful for accessibility testing.

----------

# Checkbox Assertions

----------

# toBeChecked()

```typescript
await page.getByRole('checkbox')
.check();

await expect(
page.getByRole('checkbox')
).toBeChecked();

```

Also works with radio buttons.

----------

Real Example

```typescript
await page
.getByRole('radio',{
name:'Male'
})
.check();

await expect(
page.getByRole('radio',{
name:'Male'
})
).toBeChecked();

```

----------

# Text Assertions

----------

# toHaveText()

Exact match.

```typescript
await expect(
page.locator('.status')
).toHaveText(
'Completed'
);

```

----------

Real Example

```typescript
await page
.getByRole('button',{
name:'Pay'
})
.click();

await expect(
page.locator('.payment-status')
)
.toHaveText(
'Payment Successful'
);

```

----------

Supports arrays.

```typescript
await expect(
page.locator('.menu-item')
).toHaveText([
'Home',
'Products',
'About'
]);

```

----------

# toContainText()

Partial text.

```typescript
await expect(
page.locator('.message')
).toContainText(
'Success'
);

```

Matches

```
Payment Success

Order Success

Login Success

```

----------

Use Cases

-   Toast messages
    
-   Notifications
    
-   Dynamic text
    

----------

# Attribute Assertions

----------

# toHaveAttribute()

Example

```typescript
await expect(
page.locator('img')
)
.toHaveAttribute(
'alt',
'Company Logo'
);

```

----------

Another Example

```typescript
await expect(
page.locator('#email')
)
.toHaveAttribute(
'type',
'email'
);

```

----------

# toHaveId()

```typescript
await expect(
page.locator('#username')
)
.toHaveId(
'username'
);

```

Useful in legacy applications.

----------

# toHaveRole()

```typescript
await expect(
page.getByRole('button')
)
.toHaveRole(
'button'
);

```

Useful for accessibility validation.

----------

# CSS Assertions

----------

# toHaveCSS()

Checks computed CSS.

```typescript
await expect(
page.locator('.header')
)
.toHaveCSS(
'display',
'flex'
);

```

----------

Another Example

```typescript
await expect(
page.locator('.success')
)
.toHaveCSS(
'color',
'rgb(0, 128, 0)'
);

```

----------

Useful for

-   Theme validation
    
-   Responsive UI
    
-   Dark mode
    

----------

# toHaveClass()

Single class

```typescript
await expect(
page.locator('.card')
)
.toHaveClass(
'active'
);

```

----------

Multiple classes

```typescript
await expect(
page.locator('.card')
)
.toHaveClass(
/active/
);

```

----------

Real Example

```typescript
await page
.getByRole('button',{
name:'Select'
})
.click();

await expect(
page.locator('.selected')
)
.toHaveClass(
/selected/
);

```

----------

# Collection Assertions

----------

# toHaveCount()

```typescript
await expect(
page.locator('.product')
)
.toHaveCount(10);

```

Auto retries.

----------

Real Example

```typescript
await page
.getByRole('button',{
name:'Load Products'
})
.click();

await expect(
page.locator('.product')
)
.toHaveCount(20);

```

----------

# Input Assertions

----------

# toHaveValue()

```typescript
await expect(
page.getByLabel('Email')
)
.toHaveValue(
'admin@test.com'
);

```

Better than

```typescript
expect(
await locator.inputValue()
).toBe(...);

```

----------

# toHaveValues()

For multi-select.

```typescript
await expect(
page.locator('#languages')
)
.toHaveValues([
'java',
'typescript'
]);

```

----------

# JavaScript Property Assertions

----------

# toHaveJSProperty()

Checks DOM property.

```typescript
await expect(
page.locator('#checkbox')
)
.toHaveJSProperty(
'checked',
true
);

```

Another Example

```typescript
await expect(
page.locator('video')
)
.toHaveJSProperty(
'paused',
true
);

```

Useful for

-   Video
    
-   Audio
    
-   Canvas
    
-   Custom Components
    

----------

# Real-world Example

```typescript
test('Login Validation', async ({ page }) => {

await page.goto('/login');

await expect(
page.getByLabel('Username')
).toBeEditable();

await expect(
page.getByRole('button',{
name:'Login'
})
).toBeDisabled();

await page.getByLabel('Username')
.fill('admin');

await page.getByLabel('Password')
.fill('password');

await expect(
page.getByRole('button',{
name:'Login'
})
).toBeEnabled();

await page.getByRole('button',{
name:'Login'
})
.click();

await expect(page)
.toHaveURL(/dashboard/);

await expect(
page.getByRole('heading')
)
.toHaveText('Dashboard');

});

```

----------

# Best Practices

✅ Prefer locator assertions over generic assertions for UI validation.

✅ Use `toContainText()` when only part of the content is stable.

✅ Use `toHaveText()` for exact business validations.

✅ Prefer `toBeHidden()` instead of checking CSS like `display: none`.

✅ Use `toHaveCount()` instead of manually calling `count()` for assertions.

✅ Validate user-visible behavior before validating implementation details such as CSS or IDs.

----------

# Common Mistakes

## ❌ Reading values first

```typescript
const text =
await locator.textContent();

expect(text).toBe('Success');

```

Better

```typescript
await expect(locator)
.toHaveText('Success');

```

----------

## ❌ Checking CSS unnecessarily

```typescript
await expect(locator)
.toHaveCSS(
'display',
'block'
);

```

Usually

```typescript
await expect(locator)
.toBeVisible();

```

is more meaningful.

----------

## ❌ Using exact text for dynamic content

Bad

```typescript
await expect(locator)
.toHaveText(
'Total: ₹5,432.45'
);

```

Better

```typescript
await expect(locator)
.toContainText(
'₹'
);

```

or verify the calculated value separately if the exact amount is the business requirement.

----------

# Interview Questions

### Q1. Why should `toHaveText()` be preferred over `textContent()`?

Because `toHaveText()` automatically retries until the expected text appears, whereas `textContent()` captures the value immediately and can lead to flaky tests.

----------

### Q2. What is the difference between `toHaveText()` and `toContainText()`?

-   `toHaveText()` verifies the complete expected text.
    
-   `toContainText()` verifies that the expected text is present as a substring.
    

----------

### Q3. When would you use `toHaveJSProperty()`?

When validating DOM properties rather than HTML attributes, such as the `checked` state of a checkbox, whether a video is paused, or properties on custom web components.

----------

### Q4. What is the difference between `toBeVisible()` and `toBeAttached()`?

-   `toBeAttached()` only verifies that the element exists in the DOM.
    
-   `toBeVisible()` verifies that the element is attached **and** visible to the user.
    

----------

### Q5. Why should `toHaveCount()` be preferred over `await locator.count()`?

`toHaveCount()` automatically waits for the expected number of elements, making it much more reliable for dynamic applications.

----------

# Summary

Locator assertions are one of Playwright's strongest features. They combine **automatic waiting**, **automatic retries**, and **user-focused validation** into a simple API, allowing you to write concise and resilient tests. In practice, assertions such as `toBeVisible()`, `toHaveText()`, `toHaveValue()`, `toBeEnabled()`, and `toHaveCount()` will form the backbone of most UI test suites.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyMDA4NzQ3ODJdfQ==
-->