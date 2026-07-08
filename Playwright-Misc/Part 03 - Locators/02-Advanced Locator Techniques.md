# Chapter: Locators (Part 2) – Advanced Locator Techniques

In the previous chapter, we learned how to locate elements using Playwright's built-in locator strategies like `getByRole()`, `getByLabel()`, and `getByTestId()`.

However, in real-world applications, simply locating an element is often not enough. Modern applications contain repeated components, dynamic tables, nested cards, dialogs, menus, and lists where multiple elements may match the same locator.

This is where **Locator Filtering** and **Locator Operators** become extremely powerful.

----------

# Why Advanced Locators?

Consider the following page.

```html
<div class="product">
    <h3>iPhone 16</h3>
    <button>Add to Cart</button>
</div>

<div class="product">
    <h3>MacBook Pro</h3>
    <button>Add to Cart</button>
</div>

<div class="product">
    <h3>AirPods Pro</h3>
    <button>Add to Cart</button>
</div>

```

This locator is ambiguous:

```typescript
await page.getByRole('button', {
    name: 'Add to Cart'
}).click();

```

There are **three** matching buttons.

Instead, we first locate the correct product and then find the button inside it.

----------

# Filtering Locators

Playwright provides the `filter()` API to narrow down a locator.

General syntax:

```typescript
page.locator(...)
    .filter(...)

```

Filtering happens **after** Playwright identifies the initial set of matching elements.

----------

# Filter by Text

The most common filter.

Example HTML

```html
<div class="product">
    <h3>MacBook Pro</h3>
    <button>Add to Cart</button>
</div>

<div class="product">
    <h3>iPhone 16</h3>
    <button>Add to Cart</button>
</div>

```

Locator

```typescript
await page.locator('.product')
    .filter({
        hasText: 'MacBook Pro'
    })
    .getByRole('button', {
        name: 'Add to Cart'
    })
    .click();

```

Playwright first finds all `.product` elements.

Then it keeps only the one containing **MacBook Pro**.

----------

## Another Example

```html
<tr>

<td>John</td>

<td>Edit</td>

</tr>

<tr>

<td>David</td>

<td>Edit</td>

</tr>

```

```typescript
await page.locator('tr')
    .filter({
        hasText: 'David'
    })
    .getByRole('button', {
        name: 'Edit'
    })
    .click();

```

Very common in web tables.

----------

# hasText vs getByText

These two are often confused.

### getByText()

Finds an element whose own visible text matches.

```typescript
page.getByText('MacBook Pro');

```

Returns

```html
<h3>MacBook Pro</h3>

```

----------

### hasText

Filters a parent element that contains the text somewhere inside.

```typescript
page.locator('.product')
    .filter({
        hasText: 'MacBook Pro'
    });

```

Returns

```html
<div class="product">

...

</div>

```

This distinction is extremely important.

----------

# Filter by Not Having Text

Sometimes you want the opposite.

Example

```html
<div class="notification">

Draft

</div>

<div class="notification">

Published

</div>

```

Locator

```typescript
await page.locator('.notification')
    .filter({
        hasNotText: 'Draft'
    })
    .click();

```

Only the "Published" notification remains.

----------

Another example

```typescript
const activeUsers =
    page.locator('.user')
        .filter({
            hasNotText: 'Inactive'
        });

```

Useful for dashboards.

----------

# Filter by Child/Descendant

One of Playwright's most powerful features.

Option:

```typescript
has

```

Suppose

```html
<div class="card">

<h3>Laptop</h3>

<button>Buy</button>

</div>

<div class="card">

<h3>Phone</h3>

</div>

```

Locator

```typescript
await page.locator('.card')
    .filter({
        has: page.getByRole('button')
    })
    .click();

```

Only the first card is matched.

----------

## Real Example

```html
<tr>

<td>John</td>

<button>Edit</button>

</tr>

<tr>

<td>David</td>

</tr>

```

```typescript
await page.locator('tr')
    .filter({
        has: page.getByRole('button')
    });

```

Only rows containing buttons are selected.

----------

# Filter by Not Having Child

Opposite of `has`.

```typescript
await page.locator('.card')
    .filter({
        hasNot:
            page.getByRole('button')
    });

```

Matches only cards **without** buttons.

----------

Useful when verifying disabled products, archived records, empty states, or read-only items.

----------

# Combining Filters

Multiple filters can be combined.

```typescript
await page.locator('.product')
    .filter({
        hasText: 'Laptop'
    })
    .filter({
        has:
        page.getByRole('button')
    });

```

This locator means

-   Product contains Laptop
    
-   Product contains button
    

Both conditions must be true.

----------

# Locator Operators

Playwright supports logical operations between locators.

They make locator composition much easier.

Available operators include:

-   `locator.and()`
    
-   `locator.or()`
    

----------

# Matching Inside a Locator (and)

Suppose

```html
<button
class="primary">

Submit

</button>

```

One locator

```typescript
const role =
page.getByRole('button');

```

Second locator

```typescript
const title =
page.getByTitle('Submit Form');

```

Combine them

```typescript
const button =
role.and(title);

await button.click();

```

The element must satisfy **both** conditions.

Think of it as SQL:

```sql
WHERE role='button'
AND title='Submit Form'

```

----------

## Practical Example

```typescript
const saveButton =
page.getByRole('button')
.and(
page.getByText('Save')
);

await saveButton.click();

```

----------

# Matching Two Locators Simultaneously

Consider

```html
<button>

Save

</button>

```

```typescript
const locator =
page.getByRole('button')
.and(
page.getByText('Save')
);

await expect(locator)
.toBeVisible();

```

The element must be

-   button
    
-   text = Save
    

----------

# Matching One of Two Alternative Locators (or)

Sometimes an application behaves differently.

Example

Scenario A

```html
<button>

Continue

</button>

```

Scenario B

```html
<button>

Next

</button>

```

Instead of writing conditions

Use

```typescript
const button =
page.getByRole('button',{
name:'Continue'
})
.or(
page.getByRole('button',{
name:'Next'
})
);

await button.click();

```

Whichever appears first is used.

----------

## Another Example

```typescript
const login =
page.getByText('Login');

const signin =
page.getByText('Sign In');

await login
.or(signin)
.click();

```

Very useful for

-   A/B Testing
    
-   Feature Flags
    
-   Multi-language UI (when limited variations are known)
    
-   Legacy UI migration
    

----------

# Matching Only Visible Elements

Sometimes multiple elements exist in the DOM, but only one is visible.

Example

```html
<button style="display:none">

Save

</button>

<button>

Save

</button>

```

Wrong

```typescript
page.getByText('Save');

```

Better

```typescript
page
.getByText('Save')
.filter({
visible:true
});

```

Or

```typescript
page.locator('button')
.filter({
hasText:'Save',
visible:true
});

```

This ensures hidden elements are ignored.

> **Note:** The `visible` filter is available in modern Playwright versions. In many situations, Playwright's actionability checks already ensure that actions such as `click()` target a visible, actionable element.

----------

# Nested Filtering Example

Consider

```html
<div class="order">

<h3>Order #123</h3>

<span>Delivered</span>

<button>Details</button>

</div>

<div class="order">

<h3>Order #124</h3>

<span>Pending</span>

<button>Details</button>

</div>

```

Suppose you want

> Click Details for Order #124

```typescript
await page.locator('.order')
.filter({
hasText:'Order #124'
})
.getByRole('button',{
name:'Details'
})
.click();

```

Elegant.

Readable.

Stable.

----------

# Real-World Example – Product Grid

```html
<div class="product">

<h3>MacBook Air</h3>

<p>$999</p>

<button>Add</button>

</div>

<div class="product">

<h3>iPhone 16</h3>

<p>$799</p>

<button>Add</button>

</div>

```

Test

```typescript
await page.locator('.product')
.filter({
hasText:'iPhone 16'
})
.getByRole('button',{
name:'Add'
})
.click();

```

This is much more maintainable than relying on indexes like `.nth(1)`.

----------

# Best Practices

-   Prefer `filter()` over `nth()` whenever you can identify an element by meaningful content.
    
-   Use `hasText` to narrow parent containers rather than searching globally.
    
-   Use `has` to locate parents that contain specific child elements.
    
-   Use `and()` when an element should satisfy multiple independent conditions.
    
-   Use `or()` only when your application legitimately presents alternative UI paths.
    
-   Keep filters close to the business meaning (for example, product name, order number, or customer name) instead of DOM structure.
    

----------

# Common Mistakes

### ❌ Using `nth()` unnecessarily

```typescript
page.locator('.product').nth(2);

```

If the order changes, the test breaks.

Better:

```typescript
page.locator('.product')
.filter({
hasText:'MacBook Pro'
});

```

----------

### ❌ Searching globally

```typescript
page.getByRole('button',{
name:'Delete'
});

```

If several Delete buttons exist, this becomes ambiguous.

Instead:

```typescript
page.locator('.user')
.filter({
hasText:'John'
})
.getByRole('button',{
name:'Delete'
});

```

----------

### ❌ Confusing `getByText()` with `hasText`

Remember:

-   `getByText()` finds the element **containing the text**.
    
-   `hasText` filters an **already located parent element**.
    

----------

# Interview Questions

### Q1. What is the purpose of `filter()`?

`filter()` narrows an existing locator by applying additional conditions, such as matching text, child elements, or excluding certain content.

----------

### Q2. What is the difference between `hasText` and `getByText()`?

-   `getByText()` returns the element whose visible text matches.
    
-   `hasText` filters a parent locator based on text found anywhere within its descendants.
    

----------

### Q3. What does `has` do?

The `has` option filters elements by requiring that they contain a descendant matching another locator.

----------

### Q4. When would you use `or()`?

When the application can legitimately display one of several alternative elements, such as different button labels during an A/B test or phased UI rollout.

----------

### Q5. Why is filtering better than using indexes?

Filtering identifies elements by business meaning (for example, a product name or order number) rather than their position in the DOM. This makes tests more robust when the UI layout changes.

----------

# Summary

Advanced locator techniques are essential for automating real-world applications. By combining `filter()`, `hasText`, `has`, `hasNot`, and locator operators like `and()` and `or()`, you can express exactly which element you want without relying on brittle indexes or complex CSS selectors. These techniques produce tests that are easier to read, more resilient to UI changes, and better aligned with how users interact with the application.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTM5MTI5NzQxMl19
-->