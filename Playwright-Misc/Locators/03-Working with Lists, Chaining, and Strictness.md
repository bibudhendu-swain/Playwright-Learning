# Chapter: Locators (Part 3) – Working with Lists, Chaining, and Strictness

In the previous chapters, we learned how to locate elements and how to narrow them down using filters and locator operators.

This chapter focuses on one of the most common real-world scenarios in UI automation: **working with collections of elements**. Product grids, tables, search results, menus, notifications, cards, and lists all require techniques for counting, filtering, selecting specific items, and validating collections.

We'll also cover **strictness**, one of Playwright's most important concepts that distinguishes it from many other automation frameworks.

----------

# Working with Lists

Many pages contain multiple matching elements.

Example:

```html
<ul>
    <li>Apple</li>
    <li>Orange</li>
    <li>Mango</li>
    <li>Banana</li>
</ul>

```

Locator:

```typescript
const fruits = page.getByRole('listitem');

```

This locator represents **all four list items**, not a single one.

Unlike Selenium, a Playwright Locator can naturally represent **multiple elements**.

----------

# Count Items in a List

One of the most common validations.

Example:

```typescript
const products = page.locator('.product');

expect(await products.count()).toBe(10);

```

Another example:

```typescript
await expect(page.getByRole('listitem'))
    .toHaveCount(4);

```

This is preferred because Playwright automatically waits until the expected count is reached.

----------

## Real Example – Search Results

```html
<div class="product">Laptop</div>

<div class="product">Phone</div>

<div class="product">Tablet</div>

```

```typescript
const results = page.locator('.product');

await expect(results).toHaveCount(3);

```

----------

# Retrieve All Text from a List

Playwright provides an easy way to read all text values.

```typescript
const names =
await page.getByRole('listitem')
.allTextContents();

console.log(names);

```

Output

```text
[
 "Apple",
 "Orange",
 "Mango",
 "Banana"
]

```

----------

## allInnerTexts()

Another method:

```typescript
const values =
await page.locator('.price')
.allInnerTexts();

```

Difference:

Method

Reads

`allTextContents()`

`textContent`

`allInnerTexts()`

`innerText`

Generally:

-   `innerText` respects visibility and rendered layout.
    
-   `textContent` returns the raw text from the DOM.
    

For most UI validations, `innerText` better reflects what a user sees.

----------

# Assert All Text in a List

Suppose the page contains

```html
<ul>

<li>Home</li>

<li>Products</li>

<li>Contact</li>

</ul>

```

Validation

```typescript
await expect(
page.getByRole('listitem')
).toHaveText([
'Home',
'Products',
'Contact'
]);

```

Playwright verifies

-   Count
    
-   Order
    
-   Text
    

all together.

----------

Another example

```typescript
await expect(
page.locator('.menu-item')
).toContainText([
'Dashboard',
'Users',
'Reports'
]);

```

Useful when validating menus.

----------

# Iterate Through a List

Sometimes custom logic is needed.

```typescript
const items =
page.locator('.product');

const count =
await items.count();

for(let i=0;i<count;i++){

    console.log(
        await items
            .nth(i)
            .textContent()
    );

}

```

Useful when

-   Comparing values
    
-   Reading tables
    
-   Dynamic validations
    

----------

# Get a Specific Item

Playwright provides several methods.

----------

## first()

```typescript
await page.locator('.product')
.first()
.click();

```

Equivalent to

```typescript
await page.locator('.product')
.nth(0)
.click();

```

----------

## last()

```typescript
await page.locator('.product')
.last()
.click();

```

----------

## nth()

```typescript
await page.locator('.product')
.nth(3)
.click();

```

Remember:

`nth()` starts at **0**.

----------

## Example

```html
<div>Apple</div>

<div>Orange</div>

<div>Mango</div>

```

```typescript
await page.locator('div')
.nth(1)
.click();

```

Clicks

```text
Orange

```

----------

# When Should You Use `nth()`?

Sometimes indexes are unavoidable.

Examples:

-   Calendar dates
    
-   Grid layouts
    
-   Image galleries
    
-   Fixed dashboards
    

However, avoid using indexes when the element can be identified by meaningful content.

❌

```typescript
page.locator('.user')
.nth(5);

```

✔ Better

```typescript
page.locator('.user')
.filter({
hasText:'John'
});

```

----------

# Chaining Locators

Locators can be chained together.

Example

```html
<div class="card">

<h2>Laptop</h2>

<button>Add</button>

</div>

```

Locator

```typescript
await page
.locator('.card')
.getByRole('button',{
name:'Add'
})
.click();

```

The button search happens **inside the card**.

----------

# Chaining Filters

Filters can also be chained.

Example

```typescript
await page
.locator('.product')
.filter({
hasText:'Laptop'
})
.filter({
has:
page.getByRole('button')
})
.getByRole('button',{
name:'Add'
})
.click();

```

Each filter narrows the search.

Think of it like SQL:

```sql
SELECT *
FROM Product
WHERE Name='Laptop'
AND HasButton=true

```

----------

# Rare Use Cases

Although not needed every day, the following APIs are useful in specific scenarios.

----------

## locator.locator()

Search inside another locator.

```typescript
const dialog =
page.getByRole('dialog');

await dialog
.locator('input')
.fill('John');

```

Useful for

-   Dialogs
    
-   Cards
    
-   Panels
    
-   Nested layouts
    

----------

## locator.page()

Returns the owning page.

```typescript
const pageRef =
page.locator('#login')
.page();

```

Rarely used directly but helpful when writing reusable libraries.

----------

## Evaluate an Element

Execute JavaScript on the located element.

```typescript
const color =
await page.locator('#header')
.evaluate(
element=>
getComputedStyle(element)
.backgroundColor
);

```

Useful for

-   CSS validations
    
-   Canvas
    
-   Complex DOM inspection
    

----------

## Evaluate All Elements

```typescript
const prices =
await page
.locator('.price')
.evaluateAll(
elements=>
elements.map(
e=>e.textContent
)
);

```

Very useful for tables.

----------

# Strictness

One of Playwright's most important concepts.

## What is Strictness?

Playwright expects actions to operate on **exactly one element**.

If multiple elements match, Playwright throws an error.

Example

```html
<button>Save</button>

<button>Save</button>

```

Code

```typescript
await page
.getByText('Save')
.click();

```

Result

```text
Error:

strict mode violation

locator resolved to 2 elements

```

This is intentional.

----------

## Why Strictness Exists

Without strictness,

Playwright would randomly click

-   First button
    
-   Last button
    
-   Any matching element
    

leading to flaky tests.

Instead, Playwright forces you to write a better locator.

----------

## Correct Solution

Instead of

```typescript
page.getByText('Save');

```

Use

```typescript
page
.getByRole('dialog')
.getByText('Save');

```

or

```typescript
page
.locator('.toolbar')
.getByText('Save');

```

Now only one element matches.

----------

## Another Example

HTML

```html
<div class="user">

John

<button>Edit</button>

</div>

<div class="user">

David

<button>Edit</button>

</div>

```

Wrong

```typescript
await page
.getByText('Edit')
.click();

```

Better

```typescript
await page
.locator('.user')
.filter({
hasText:'David'
})
.getByRole('button',{
name:'Edit'
})
.click();

```

----------

# When Doesn't Strictness Apply?

Operations that naturally work with multiple elements do **not** require a single match.

Examples:

```typescript
await page.locator('.product')
.count();

```

```typescript
await page.locator('.price')
.allTextContents();

```

```typescript
await expect(
page.locator('.card')
).toHaveCount(5);

```

These APIs are designed for collections.

----------

# Handling Multiple Matches Intentionally

If you genuinely want the first or last match, make it explicit.

```typescript
await page
.getByRole('button', { name: 'Save' })
.first()
.click();

```

or

```typescript
await page
.getByRole('button', { name: 'Save' })
.last()
.click();

```

Using `.first()`, `.last()`, or `.nth()` tells Playwright that you intentionally chose one element from the collection.

----------

# Best Practices

-   Prefer meaningful filters over indexes.
    
-   Use `toHaveCount()` instead of `count()` when asserting the number of elements.
    
-   Use `toHaveText()` for validating complete lists in one assertion.
    
-   Chain locators to scope searches to a specific section of the page.
    
-   Treat strictness errors as a signal to improve your locator rather than suppressing them.
    
-   Use `.first()`, `.last()`, or `.nth()` only when the UI design guarantees a stable order.
    

----------

# Common Mistakes

### ❌ Looping when a single assertion is enough

```typescript
const items = page.locator('.menu');

for (let i = 0; i < await items.count(); i++) {
    expect(await items.nth(i).textContent()).toBe(...);
}

```

Better:

```typescript
await expect(page.locator('.menu')).toHaveText([
    'Home',
    'Products',
    'Contact'
]);

```

----------

### ❌ Ignoring strictness errors

```typescript
await page.getByText('Delete').click();

```

If multiple "Delete" buttons exist, don't silence the error by immediately adding `.first()`. Instead, ask _which_ Delete button you actually want and make the locator more specific.

----------

### ❌ Overusing `nth()`

```typescript
page.locator('.product').nth(7);

```

This is fragile if the list order changes. Prefer business identifiers like product names, order numbers, or user names whenever possible.

----------

# Interview Questions

### Q1. What is Playwright's strict mode?

Strictness means that actions such as `click()`, `fill()`, and `check()` require the locator to resolve to exactly one element. If multiple elements match, Playwright throws a strict mode violation.

----------

### Q2. Why does Playwright enforce strictness?

To prevent ambiguous interactions that could lead to flaky tests. It encourages developers to write precise, maintainable locators.

----------

### Q3. What is the difference between `count()` and `toHaveCount()`?

-   `count()` immediately returns the current number of matching elements.
    
-   `toHaveCount()` automatically waits until the locator matches the expected count, making it better suited for assertions.
    

----------

### Q4. When should `nth()` be used?

Use `nth()` only when the position of an element is meaningful and stable (for example, calendar cells or fixed dashboard widgets). Otherwise, prefer filters or semantic locators.

----------

### Q5. How do you validate all items in a list?

Use assertions like `toHaveText()` or `toContainText()` with an array of expected values. These assertions validate the collection more concisely than looping through each item.

----------

# Locator Decision Guide

Scenario

Recommended Approach

Click a button

`getByRole('button')`

Fill a form field

`getByLabel()`

Work with repeated cards

`filter({ hasText })`

Find a parent containing a child

`filter({ has })`

Validate a list

`toHaveText()`

Count elements

`toHaveCount()`

Choose one item intentionally

`first()`, `last()`, or `nth()`

Resolve strictness

Refine the locator instead of relying on indexes

----------

# Summary

Working with collections is an essential part of UI automation. Playwright's Locator API makes it easy to count elements, validate entire lists, iterate through collections, and narrow searches using chained locators and filters. Equally important is understanding **strictness**—a design choice that prevents ambiguous actions and encourages precise, maintainable tests. Rather than being an obstacle, strictness is one of the reasons Playwright tests tend to be more reliable than those written with less opinionated frameworks.
<!--stackedit_data:
eyJoaXN0b3J5IjpbOTk1ODMyODg2XX0=
-->