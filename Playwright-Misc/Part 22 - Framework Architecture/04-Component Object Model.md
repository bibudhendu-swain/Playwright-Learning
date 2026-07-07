Large organizations go one step further and implement a **Component Object Model (COM)** because modern web applications are built using reusable UI components (React, Angular, Vue, etc.). Your automation framework should mirror that architecture.

----------

# Part 22 – Framework Best Practices

# Chapter 4 – Component Object Model (COM)

----------

# Introduction

Modern web applications are no longer built as a collection of independent pages.

Instead, pages are composed of reusable UI components such as:

-   Header
    
-   Footer
    
-   Navigation Menu
    
-   Search Box
    
-   Product Card
    
-   Shopping Cart
    
-   Modal Dialog
    
-   Data Table
    

The **Component Object Model (COM)** represents each reusable UI component as its own automation class.

Instead of duplicating the same locators and actions across multiple page objects, components encapsulate their own behavior.

----------

# Why Page Objects Are Not Enough

Consider an e-commerce application.

```text
Home

↓

Category

↓

Product

↓

Cart

↓

Checkout

```

Every page contains the same:

-   Header
    
-   Search Bar
    
-   Navigation Menu
    
-   User Menu
    
-   Shopping Cart Icon
    

Without components,

every page object duplicates the same code.

----------

# Without Component Objects

```text
HomePage

├── Header Locators

├── Search Locators

├── Menu Locators

----------------------

ProductPage

├── Header Locators

├── Search Locators

├── Menu Locators

----------------------

CartPage

├── Header Locators

├── Search Locators

├── Menu Locators

```

Problems:

-   Duplicate locators
    
-   Duplicate actions
    
-   Higher maintenance cost
    

----------

# With Component Objects

```text
Header Component

↓

Search Component

↓

Navigation Component

↓

Cart Component

↓

Pages

```

Each component is implemented once and reused across all pages.

----------

# Component Architecture

```text
Tests

↓

Pages

↓

Components

↓

Playwright

```

Pages compose reusable components rather than implementing everything themselves.

----------

# What is a Component?

A component represents a reusable UI section that appears in one or more pages.

Examples:

-   Header
    
-   Footer
    
-   Sidebar
    
-   Navigation Menu
    
-   Search Box
    
-   Notification Panel
    
-   User Profile Menu
    
-   Product Card
    
-   Shopping Cart Widget
    
-   Pagination
    

----------

# Enterprise Example

Instead of

```text
DashboardPage

↓

200 Locators

```

Use

```text
DashboardPage

├── Header

├── Sidebar

├── Notifications

├── UserMenu

└── DashboardContent

```

Each responsibility is isolated.

----------

# Folder Structure

```text
src/

├── pages/

├── components/

│     ├── Header.ts

│     ├── Footer.ts

│     ├── Navigation.ts

│     ├── SearchBar.ts

│     ├── ProductCard.ts

│     ├── Modal.ts

│     └── DataTable.ts

```

----------

# Example Header Component

```typescript
import { Page } from '@playwright/test';

export class Header {

    constructor(private readonly page: Page) {}

    private readonly searchBox =
        this.page.getByPlaceholder('Search');

    private readonly cartButton =
        this.page.getByRole('button', { name: 'Cart' });

    async search(text: string) {
        await this.searchBox.fill(text);
        await this.searchBox.press('Enter');
    }

    async openCart() {
        await this.cartButton.click();
    }

}

```

Notice that the component knows only about the header.

----------

# Using Components Inside Pages

```typescript
export class HomePage {

    readonly header: Header;

    constructor(private readonly page: Page) {

        this.header = new Header(page);

    }

}

```

Now tests can simply write:

```typescript
await homePage.header.search("Laptop");

```

----------

# Component Composition

Pages become compositions of components.

```text
HomePage

├── Header

├── Navigation

├── FeaturedProducts

├── Footer

```

Instead of hundreds of locators in a single class.

----------

# Nested Components

Components can contain other components.

Example

```text
Header

↓

UserMenu

↓

ProfileDropdown

```

Architecture

```text
Header

├── Logo

├── Search

├── UserMenu

       ↓

Profile Menu

```

----------

# Dynamic Components

Some components appear only under certain conditions.

Examples:

-   Success Toast
    
-   Error Banner
    
-   Loading Spinner
    
-   Notification Popup
    

These should still have dedicated component classes.

----------

# Modal Component

Example

```typescript
export class ConfirmationModal {

    constructor(private readonly page: Page) {}

    async confirm() {
        await this.page.getByRole('button', { name: 'Confirm' }).click();
    }

    async cancel() {
        await this.page.getByRole('button', { name: 'Cancel' }).click();
    }

}

```

Any page displaying this modal can reuse it.

----------

# Table Component

Instead of creating table methods inside every page,

create a reusable table.

```typescript
class DataTable {

    async selectRow(index: number) {}

    async getCell(row: number, column: number) {}

    async sort(column: string) {}

}

```

Now every application table behaves consistently.

----------

# Pagination Component

```typescript
class Pagination {

    async nextPage(){}

    async previousPage(){}

    async goto(page:number){}

}

```

Reusable across reports, grids, and search results.

----------

# Search Component

```typescript
class SearchBar {

    async search(text:string){}

    async clear(){}

}

```

Every page with a search field reuses the same component.

----------

# Product Card Component

E-commerce example

```text
Product Card

↓

Image

↓

Title

↓

Price

↓

Add to Cart

```

Methods

```typescript
class ProductCard {

    async addToCart(){}

    async openProduct(){}

    async getPrice(){}

}

```

----------

# Sidebar Component

```text
Sidebar

↓

Orders

↓

Products

↓

Customers

↓

Reports

```

Instead of duplicating navigation methods.

----------

# Notification Component

```typescript
class Notification {

    async successMessage(){}

    async errorMessage(){}

    async close(){}

}

```

----------

# Generic Components

Some components are reusable across completely different projects.

Examples

```text
Modal

Table

Dropdown

Date Picker

Tree View

Accordion

```

These become framework-level components.

----------

# Component Communication

Good

```text
Page

↓

Component

↓

UI

```

Avoid

```text
Component

↓

Page

↓

Component

```

Components should not know about other pages.

----------

# Component Responsibilities

A component should contain:

-   Component locators
    
-   Component actions
    
-   Component validation
    

Nothing more.

----------

# What Should NOT Be Inside Components?

❌ API Calls

❌ Database Queries

❌ File Operations

❌ Business Workflows

❌ Random Utilities

Components represent UI only.

----------

# Composition vs Inheritance

Avoid

```text
BaseComponent

↓

Header

↓

MegaHeader

↓

AdminHeader

```

Prefer

```text
Header

↓

Search

↓

User Menu

↓

Notification

```

Composition is more flexible and avoids deep inheritance hierarchies.

----------

# Real Enterprise Architecture

```text
Tests

↓

Pages

├── HomePage

├── ProductPage

├── CheckoutPage

↓

Components

├── Header

├── Navigation

├── Footer

├── Modal

├── DataTable

├── SearchBar

├── Pagination

├── Toast

└── UserMenu

```

----------

# Benefits

Benefit

Description

Reusability

Write once, use everywhere

Maintainability

Update one component instead of many pages

Readability

Smaller, focused classes

Scalability

Easy to add new pages

Consistency

Uniform behavior across the framework

----------

# Best Practices

-   Extract any UI element used on multiple pages into a component.
    
-   Keep components focused on a single UI responsibility.
    
-   Compose pages from components instead of duplicating locators.
    
-   Keep components independent of business workflows.
    
-   Use descriptive names such as `Header`, `Modal`, or `DataTable`.
    
-   Reuse generic components across projects where appropriate.
    

----------

# Common Mistakes

### ❌ Duplicating Header Locators

If five pages contain the same header, implement it once.

----------

### ❌ Creating Huge Components

A component should represent one logical UI element, not an entire page.

----------

### ❌ Business Logic Inside Components

Components should interact with the UI only.

----------

### ❌ Components Calling Other Pages

A component should not navigate between pages or create page objects.

----------

### ❌ Deep Inheritance Hierarchies

Favor composition over inheritance for reusable UI building blocks.

----------

# Interview Questions

### Q1. What is the Component Object Model?

It is a design pattern where reusable UI elements are represented as independent classes that can be composed into page objects.

----------

### Q2. Why use components in addition to page objects?

Components eliminate duplicated UI logic, improve reuse, and keep page objects smaller and easier to maintain.

----------

### Q3. What kinds of UI elements should become components?

Reusable elements such as headers, footers, navigation menus, modals, tables, search bars, pagination controls, and notification panels.

----------

### Q4. Can a component be reused across different pages?

Yes. Reusability is the primary purpose of the Component Object Model.

----------

### Q5. Should components contain API calls or business workflows?

No. Components should encapsulate only UI interactions and validations related to that specific component.

----------

# Summary

The Component Object Model extends the Page Object Model by introducing reusable UI building blocks that mirror modern web application architecture. Instead of duplicating common locators and actions across page objects, components encapsulate shared behavior, improving maintainability, readability, and scalability. By composing pages from reusable components, enterprise Playwright frameworks become significantly easier to evolve as applications grow.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEzMjYxNjQxMTRdfQ==
-->