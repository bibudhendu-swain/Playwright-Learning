The next topic is **Extensibility**, specifically **Custom Selector Engines**.

This is an **advanced Playwright feature**. In my experience, **95% of automation engineers will never need to write a custom selector engine**, but understanding it is valuable if you're building an automation framework, testing custom web components, or working with proprietary UI libraries.

I'll also explain **when you should _not_ use custom selector engines**, because many people over-engineer solutions that are better solved with locators.

----------

# Part 3 – Extensibility

# Chapter 4 – Custom Selector Engines

----------

# Introduction

Playwright already provides powerful built-in locator strategies:

-   `getByRole()`
    
-   `getByText()`
    
-   `getByLabel()`
    
-   `getByPlaceholder()`
    
-   `getByTestId()`
    
-   CSS selectors
    
-   XPath selectors
    
-   Shadow DOM support
    

For almost every application, these are sufficient.

However, some applications use proprietary component frameworks or unique DOM structures where built-in selectors become difficult to use.

For these cases, Playwright allows you to create your own selector engine.

----------

# What is a Selector Engine?

A selector engine defines **how Playwright finds elements**.

Normally:

```typescript
await page.locator("#login");

```

uses the CSS selector engine.

Likewise:

```typescript
await page.getByRole("button", {
    name: "Login"
});

```

uses Playwright's accessibility engine.

A custom selector engine lets you define your own search logic.

----------

# Why Would You Need One?

Imagine an application built with custom components.

```html
<app-button
    component-id="save">
</app-button>

<app-button
    component-id="cancel">
</app-button>

```

There are:

-   No IDs
    
-   No roles
    
-   No labels
    
-   No text
    

Your team always identifies components by:

```text
component-id

```

Instead of repeatedly writing:

```typescript
page.locator(
    '[component-id="save"]'
);

```

you could create:

```typescript
page.locator(
    "component=save"
);

```

Much cleaner.

----------

# How Custom Selector Engines Work

Internally:

```text
Locator

↓

Custom Selector Engine

↓

Browser

↓

Matching Elements

```

The selector engine determines how matching elements are found.

----------

# Registering a Selector Engine

Selector engines are registered **before pages are created**.

Example:

```typescript
import { selectors } from "@playwright/test";

await selectors.register(
    "component",
    () => ({
        query(root, selector) {
            return root.querySelector(
                `[component-id="${selector}"]`
            );
        },

        queryAll(root, selector) {
            return [
                ...root.querySelectorAll(
                    `[component-id="${selector}"]`
                )
            ];
        }
    })
);

```

This registers a selector named:

```text
component

```

----------

# Using the Selector

Once registered:

```typescript
await page.locator(
    "component=save"
).click();

```

Internally:

```text
component=save

↓

component-id="save"

↓

Element Found

```

----------

# query()

Every selector engine implements:

```typescript
query(root, selector)

```

Purpose:

Return the **first matching element**.

Example:

```typescript
query(root, selector) {

    return root.querySelector(
        `[component-id="${selector}"]`
    );

}

```

----------

# queryAll()

Purpose:

Return **all matching elements**.

Example:

```typescript
queryAll(root, selector) {

    return [

        ...root.querySelectorAll(

            `[component-id="${selector}"]`

        )

    ];

}

```

----------

# Example – Data Attribute

Suppose your application uses:

```html
<div
    data-component="header">
</div>

```

Selector engine:

```typescript
query(root, selector) {

    return root.querySelector(
        `[data-component="${selector}"]`
    );

}

```

Now:

```typescript
await page.locator(
    "component=header"
);

```

----------

# Example – Legacy Framework

Imagine an old application:

```html
<legacy-field
    field-name="email">
</legacy-field>

```

Instead of:

```typescript
page.locator(
    '[field-name="email"]'
);

```

Use:

```typescript
page.locator(
    "field=email"
);

```

----------

# Example – Enterprise Component Library

Large organizations often have reusable components.

```text
CompanyButton

↓

CompanyInput

↓

CompanyTable

↓

CompanyDropdown

```

Every component exposes:

```text
component-id

```

A custom selector engine can standardize how all tests locate these components.

----------

# Selector Engine Lifecycle

```text
Register Engine

↓

Create Browser

↓

Create Context

↓

Create Page

↓

Use Locator

```

Register once.

Use everywhere.

----------

# Combining with Other Locators

Custom selector engines integrate with Locator APIs.

```typescript
await page
    .locator("component=save")
    .click();

```

Or

```typescript
await page
    .locator("component=row")
    .nth(2)
    .click();

```

Everything else works normally.

----------

# Custom Selector vs getByTestId()

Many teams ask:

Should we build a selector engine for test IDs?

Usually:

**No.**

Example:

```html
<button
    data-testid="login">
</button>

```

Playwright already supports:

```typescript
await page.getByTestId("login");

```

No custom engine needed.

----------

# Custom Selector vs CSS

Suppose:

```typescript
page.locator(
    '[component-id="save"]'
);

```

works perfectly.

Creating a selector engine just to shorten this to:

```typescript
page.locator(
    "component=save"
);

```

may not justify the additional maintenance.

----------

# When Should You Create a Custom Selector Engine?

Good candidates:

-   Proprietary UI frameworks
    
-   Large enterprise component libraries
    
-   Thousands of repeated selectors
    
-   Internal automation platforms
    
-   Organization-wide testing standards
    

----------

# When Should You Avoid It?

Avoid if:

-   CSS selectors already solve the problem.
    
-   `getByRole()` or `getByTestId()` works.
    
-   Only one project needs it.
    
-   The selector logic is simple.
    

Adding a custom selector engine increases framework complexity.

----------

# Real-World Example – Design System

Imagine every button in your company uses:

```html
<company-button
    name="submit">
</company-button>

```

Instead of:

```typescript
page.locator(
    'company-button[name="submit"]'
);

```

You could expose:

```typescript
page.locator(
    "company=submit"
);

```

Every test becomes consistent.

----------

# Architecture Example

```text
Playwright Test

        │

        ▼

Custom Selector Engine

        │

        ▼

Enterprise Component Library

        │

        ▼

DOM Elements

```

----------

# Performance Considerations

A selector engine executes browser-side JavaScript.

Good implementation:

```text
Simple querySelector()

↓

Fast

```

Poor implementation:

```text
Loop Entire DOM

↓

Complex Filtering

↓

Slow Tests

```

Keep selector logic efficient.

----------

# Best Practices

-   Register selector engines during framework initialization.
    
-   Keep selector implementations simple and deterministic.
    
-   Prefer semantic locators (`getByRole()`, `getByLabel()`, `getByTestId()`) whenever possible.
    
-   Document custom selector syntax so the entire team uses it consistently.
    
-   Treat selector engines as framework infrastructure rather than test-specific utilities.
    

----------

# Common Mistakes

### ❌ Creating a selector engine for simple CSS selectors

If CSS or Playwright locators are already readable, a custom selector engine adds unnecessary complexity.

----------

### ❌ Hiding poor application testability

If the application can expose stable accessibility attributes or test IDs, prefer those over introducing a custom selector engine.

----------

### ❌ Writing expensive selector logic

Avoid scanning the entire DOM manually when `querySelector()` or `querySelectorAll()` can do the work efficiently.

----------

### ❌ Creating multiple overlapping selector engines

Keep the number of custom engines small and focused.

----------

# Interview Questions

### Q1. What is a custom selector engine?

A custom selector engine allows Playwright to locate elements using user-defined selection logic instead of the built-in selector strategies.

----------

### Q2. When should you create one?

Primarily when testing proprietary UI frameworks, enterprise component libraries, or applications with consistent custom DOM structures that aren't well served by built-in locators.

----------

### Q3. What methods must a selector engine implement?

Typically:

-   `query()` – returns the first matching element.
    
-   `queryAll()` – returns all matching elements.
    

----------

### Q4. Should custom selector engines replace `getByRole()` or `getByTestId()`?

No. Built-in semantic locators should remain the default choice. Custom selector engines are intended for specialized scenarios.

----------

### Q5. What is the biggest downside of a custom selector engine?

It adds framework complexity and long-term maintenance. Teams should ensure the benefits outweigh the additional abstraction.

----------

# Summary

Custom selector engines extend Playwright's locator system for specialized applications and enterprise frameworks. While they provide a powerful way to standardize element lookup across large projects, they should be used sparingly. In most cases, Playwright's built-in locator strategies are more maintainable, expressive, and aligned with modern testing best practices.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTc0NDk2OTI1OSwtODM4NjkwODk3XX0=
-->