This chapter is the **foundation of Playwright**. If someone masters Locators, they can automate almost any web application effectively.

----------

# Chapter: Locators (Part 1) – Fundamentals

----------

# Introduction

One of the biggest advantages of Playwright over traditional automation frameworks is its **Locator API**.

A **Locator** represents a way to find one or more elements on a web page. Unlike Selenium's `WebElement`, a Playwright Locator **does not immediately locate the element**. Instead, it stores the strategy to find the element and resolves it only when an action (like `click()`, `fill()`, or an assertion) is performed.

This design provides several benefits:

-   Automatic waiting
    
-   Retry mechanism
    
-   Better stability
    
-   Better readability
    
-   Reduced flaky tests
    

Instead of writing:

```typescript
const element = page.$('#login');
await element.click();

```

Playwright encourages:

```typescript
await page.locator('#login').click();

```

The locator waits automatically until the button becomes actionable.

----------

# Why Playwright Introduced Locators

Traditional automation tools locate an element once and store a reference to it.

Example:

```
Locate Button
       ↓
Store Reference
       ↓
DOM Refresh
       ↓
Reference becomes invalid
       ↓
Stale Element Exception

```

Playwright follows a different model.

```
Locator Created
      ↓
Action Called
      ↓
Locate Element
      ↓
Perform Action

```

Every action re-resolves the element against the current DOM, greatly reducing issues caused by dynamic pages.

----------

# Quick Guide

Below are the most commonly used locator methods.

Method

Best For

Recommended

`getByRole()`

Buttons, links, inputs

⭐⭐⭐⭐⭐

`getByLabel()`

Form fields

⭐⭐⭐⭐⭐

`getByPlaceholder()`

Inputs with placeholders

⭐⭐⭐⭐

`getByText()`

Visible text

⭐⭐⭐⭐

`getByAltText()`

Images

⭐⭐⭐⭐

`getByTitle()`

Tooltip/title attribute

⭐⭐⭐

`getByTestId()`

Test-specific elements

⭐⭐⭐⭐⭐

`locator(css)`

Complex CSS selectors

⭐⭐⭐

`locator(xpath)`

Legacy applications

⭐⭐

----------

# Locating Elements

The Locator API starts from either:

```typescript
page.locator(...)

```

or

```typescript
page.getByRole(...)

```

Both return a Locator object.

Example

```typescript
const loginButton = page.getByRole('button', {
    name: 'Login'
});

await loginButton.click();

```

A locator can be reused multiple times.

```typescript
const username = page.getByLabel('Username');

await username.fill('admin');

await expect(username).toHaveValue('admin');

```

----------

# Locate by Role

This is the **most recommended locator strategy**.

Playwright uses the Accessibility Tree to locate elements based on their semantic role.

Example HTML

```html
<button>Login</button>

```

Locator

```typescript
await page.getByRole('button', {
    name: 'Login'
}).click();

```

----------

Another example

```html
<a href="/home">Home</a>

```

```typescript
await page.getByRole('link', {
    name: 'Home'
}).click();

```

----------

Textbox

```html
<input type="text">

```

```typescript
await page.getByRole('textbox')
    .fill('John');

```

----------

Checkbox

```html
<input type="checkbox">

```

```typescript
await page.getByRole('checkbox')
    .check();

```

----------

Radio Button

```typescript
await page.getByRole('radio', {
    name: 'Male'
}).check();

```

----------

Combobox

```typescript
await page.getByRole('combobox')
    .selectOption('India');

```

----------

Common Roles

Role

HTML Element

button

`<button>`

textbox

`<input>`

checkbox

`<input type="checkbox">`

radio

Radio button

link

`<a>`

heading

`<h1>`–`<h6>`

img

`<img>`

list

`<ul>`

listitem

`<li>`

table

`<table>`

row

`<tr>`

cell

`<td>`

----------

### Why Role is Recommended

Consider

```html
<button id="btn12345">Submit</button>

```

Tomorrow the developer changes

```html
<button id="btn56789">Submit</button>

```

CSS selector breaks.

Role locator still works.

```typescript
await page.getByRole('button', {
    name: 'Submit'
}).click();

```

Role is based on **what the user experiences**, not how the HTML is implemented.

----------

# Locate by Label

Used for forms.

HTML

```html
<label>Email</label>

<input type="email">

```

Playwright

```typescript
await page.getByLabel('Email')
    .fill('abc@gmail.com');

```

----------

Even if HTML changes

```html
<label for="email">Email</label>

<input id="email">

```

or

```html
<label>

Email

<input>

</label>

```

The locator still works.

----------

Real Login Example

```typescript
await page.getByLabel('Username')
    .fill('admin');

await page.getByLabel('Password')
    .fill('admin123');

```

----------

Why use Label?

Because this is how screen readers identify form controls.

----------

# Locate by Placeholder

Many modern applications don't use labels.

Example

```html
<input placeholder="Enter Username">

```

Locator

```typescript
await page
    .getByPlaceholder('Enter Username')
    .fill('admin');

```

----------

Another example

```html
<input placeholder="Search Products">

```

```typescript
await page
    .getByPlaceholder('Search Products')
    .fill('Laptop');

```

----------

Use placeholder only when no label is available.

Labels are generally more stable.

----------

# Locate by Text

One of the easiest ways to locate elements.

HTML

```html
<button>Continue</button>

```

```typescript
await page.getByText('Continue')
    .click();

```

----------

Partial Text

```typescript
await page.getByText('Continue');

```

Matches

```
Continue

Continue Shopping

Continue to Payment

```

----------

Exact Match

```typescript
await page.getByText(
    'Continue',
    { exact: true }
).click();

```

----------

Real Example

```html
<div>

Order Successful

</div>

```

```typescript
await expect(
    page.getByText('Order Successful')
).toBeVisible();

```

----------

When to Use Text

Excellent for

-   Success messages
    
-   Error messages
    
-   Links
    
-   Buttons
    
-   Headings
    
-   Notifications
    

Avoid using text when the UI is multilingual or the text changes frequently.

----------

# Locate by Alt Text

Useful for images.

HTML

```html
<img src="logo.png"
     alt="Company Logo">

```

Locator

```typescript
await expect(
    page.getByAltText('Company Logo')
).toBeVisible();

```

----------

Another Example

```html
<img alt="Shopping Cart">

```

```typescript
await page
    .getByAltText('Shopping Cart')
    .click();

```

----------

This is especially useful for accessibility-focused applications.

----------

# Locate by Title

HTML

```html
<button title="Refresh">

```

Locator

```typescript
await page
    .getByTitle('Refresh')
    .click();

```

----------

Another Example

```html
<span title="Premium User">

```

```typescript
await expect(
    page.getByTitle('Premium User')
).toBeVisible();

```

----------

The `title` attribute often provides tooltip information.

Prefer `getByRole()` or `getByLabel()` when possible, as `title` attributes are less common and may change.

----------

# Locate by Test Id

One of the most stable locator strategies.

HTML

```html
<button data-testid="login-btn">

Login

</button>

```

Locator

```typescript
await page
    .getByTestId('login-btn')
    .click();

```

----------

Another Example

```html
<div data-testid="total-price">

₹4500

</div>

```

```typescript
await expect(
    page.getByTestId('total-price')
).toContainText('4500');

```

----------

### Why Test IDs?

Suppose developers redesign the page.

Before

```html
<button class="blue-btn">

```

After

```html
<button class="primary">

```

CSS selector breaks.

The test ID remains the same.

```html
data-testid="login-btn"

```

Therefore,

```typescript
page.getByTestId('login-btn')

```

continues to work.

----------

### Configuring a Custom Test ID Attribute

By default, Playwright looks for the `data-testid` attribute. If your application uses a different attribute, you can configure it globally.

Example using `data-test`:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    testIdAttribute: 'data-test'
  }
});

```

Then the following locator works:

```typescript
await page.getByTestId('login-button').click();

```

against HTML like:

```html
<button data-test="login-button">Login</button>

```

----------

# Locate by CSS or XPath

Playwright fully supports CSS selectors and XPath.

CSS

```typescript
await page.locator('#username')
    .fill('admin');

```

Class selector

```typescript
await page.locator('.login-btn')
    .click();

```

Attribute selector

```typescript
await page.locator(
    '[data-id="123"]'
).click();

```

Descendant selector

```typescript
await page.locator(
    '.card .title'
);

```

----------

XPath

```typescript
await page.locator(
    '//button[text()="Login"]'
).click();

```

----------

Relative XPath

```typescript
await page.locator(
    '//table//tr[2]/td[3]'
);

```

----------

### CSS vs XPath

CSS

XPath

Faster

Slightly slower

Easier to read

More verbose

Better supported

Useful for complex XML-like relationships

Recommended

Mainly for legacy applications or when CSS cannot express the relationship

In Playwright, prefer **user-facing locators** (`getByRole`, `getByLabel`, `getByTestId`) first, then CSS, and use XPath only when necessary.

----------

# Locate in Shadow DOM

Many modern frameworks use **Shadow DOM** to encapsulate component markup.

Example:

```html
<custom-search>
  #shadow-root
      <input placeholder="Search">
</custom-search>

```

One of Playwright's strengths is that it **automatically pierces open Shadow DOM**. You don't need special APIs.

```typescript
await page
    .getByPlaceholder('Search')
    .fill('Laptop');

```

or

```typescript
await page.locator('custom-search input')
    .fill('Laptop');

```

Playwright automatically traverses **open shadow roots**.

> **Limitation:** Like browsers and other automation tools, Playwright cannot directly access **closed Shadow DOM** because it is intentionally hidden by the component author.

----------

# Choosing the Right Locator

Priority

Locator

⭐⭐⭐⭐⭐

`getByRole()`

⭐⭐⭐⭐⭐

`getByLabel()`

⭐⭐⭐⭐⭐

`getByTestId()`

⭐⭐⭐⭐

`getByPlaceholder()`

⭐⭐⭐⭐

`getByText()`

⭐⭐⭐

`getByAltText()`

⭐⭐⭐

`getByTitle()`

⭐⭐

CSS

⭐

XPath

A good rule of thumb is:

1.  **Can a user identify the element by its role or accessible name?** Use `getByRole()`.
    
2.  **Is it a form control with a label?** Use `getByLabel()`.
    
3.  **Does your team provide stable test IDs?** Use `getByTestId()`.
    
4.  **Only fall back to CSS or XPath when a user-facing locator isn't practical.**
    

----------

# Best Practices

-   Prefer **user-facing locators** over implementation-based selectors.
    
-   Reuse locators instead of recreating them multiple times.
    
-   Use `getByRole()` for buttons, links, headings, tables, and menus whenever possible.
    
-   Use `getByLabel()` for form fields to improve readability and accessibility.
    
-   Encourage developers to add `data-testid` attributes for complex components.
    
-   Avoid fragile selectors based on generated CSS classes or deeply nested DOM structures.
    
-   Use XPath sparingly, primarily when working with legacy applications or complex ancestor relationships.
    

----------

# Common Mistakes

### ❌ Using long CSS chains

```typescript
page.locator(
'.container > div:nth-child(2) > table > tbody > tr:nth-child(3) > td:nth-child(2) button'
);

```

Instead:

```typescript
page.getByRole('button', {
    name: 'Edit'
});

```

----------

### ❌ Depending on dynamic IDs

```typescript
page.locator('#btn_17463892');

```

Dynamic IDs often change between builds.

----------

### ❌ Using XPath for everything

```typescript
page.locator('//div[3]/table/tr[2]/td[1]');

```

Prefer semantic locators unless XPath provides a clear advantage.

----------

# Interview Questions

### Q1. What is a Locator in Playwright?

A Locator is a lazy, retryable object that stores a strategy for finding elements. It resolves the element only when an action or assertion is executed, making tests more resilient to dynamic page updates.

----------

### Q2. Why is `getByRole()` the recommended locator?

Because it reflects how users and assistive technologies interact with the application. It is generally more stable than CSS classes or IDs and encourages accessible applications.

----------

### Q3. What is the difference between `locator()` and `getByRole()`?

-   `locator()` uses implementation details such as CSS selectors or XPath.
    
-   `getByRole()` uses accessibility semantics (role and accessible name), making tests more user-centric and easier to maintain.
    

----------

### Q4. Does Playwright automatically support Shadow DOM?

Yes, for **open Shadow DOM**. Playwright automatically pierces open shadow roots without requiring additional APIs. Closed Shadow DOM is not directly accessible.

----------

### Q5. When should `getByTestId()` be used?

When your application exposes stable test IDs and user-facing locators are insufficient or ambiguous. Test IDs provide a reliable contract between developers and test automation.

----------

## Summary

The Locator API is the cornerstone of Playwright's reliability. By favoring semantic, user-facing locators such as `getByRole()`, `getByLabel()`, and `getByTestId()`, you create tests that are more readable, resilient to UI changes, and aligned with accessibility best practices. CSS selectors and XPath remain valuable tools but should generally be treated as fallback options rather than the default approach.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1NDUyNDM1NzJdfQ==
-->