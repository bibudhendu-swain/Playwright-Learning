Almost every automation engineer knows what a Page Object Model (POM) is, but **very few implement it well**. In enterprise projects, poorly designed page objects become the biggest maintenance burden.

This chapter focuses on **how experienced automation architects design Page Objects**, not just how to wrap locators and actions in a class.

----------

# Part 22 – Framework Best Practices

# Chapter 3 – Enterprise Page Object Model

----------

# Introduction

The **Page Object Model (POM)** is a design pattern where each page of an application is represented by a class.

The purpose of a page object is to:

-   Encapsulate UI interactions
    
-   Hide locator details
    
-   Improve test readability
    
-   Promote code reuse
    
-   Reduce maintenance
    

A page object is **not** a place to store all automation logic.

----------

# Why Use Page Objects?

Without Page Objects

```text
Test

↓

Locator

↓

Click

↓

Wait

↓

Locator

↓

Fill

↓

Locator

↓

Assert

```

Tests become repetitive and tightly coupled to the UI.

With Page Objects

```text
Test

↓

LoginPage.login()

↓

DashboardPage.verifyLoaded()

```

Tests focus on business behavior rather than implementation details.

----------

# Traditional Page Object

```typescript
export class LoginPage {

    constructor(private page: Page){}

    username = this.page.getByLabel("Username");

    password = this.page.getByLabel("Password");

    loginButton = this.page.getByRole("button", { name: "Login" });

    async login(user: string, password: string){

        await this.username.fill(user);

        await this.password.fill(password);

        await this.loginButton.click();

    }

}

```

This is a good starting point, but enterprise projects require more discipline.

----------

# Responsibility of a Page Object

A page object should only know:

-   UI elements
    
-   User interactions
    
-   Navigation
    
-   Page-specific validations
    

A page object should **not** know about:

-   Database operations
    
-   API requests
    
-   File processing
    
-   Business workflows spanning multiple pages
    

----------

# Good Architecture

```text
Test

↓

Page

↓

Application

```

The page object hides UI details.

----------

# Bad Architecture

```text
Test

↓

Page

↓

API

↓

Database

↓

Email

↓

Utilities

↓

File System

```

This violates the Single Responsibility Principle.

----------

# Thin Page Objects

Thin Page Objects contain:

-   Locators
    
-   UI actions
    
-   Simple validations
    

Example

```typescript
class LoginPage {

    async enterUsername(username: string){

        await this.username.fill(username);

    }

    async enterPassword(password: string){

        await this.password.fill(password);

    }

    async clickLogin(){

        await this.loginButton.click();

    }

}

```

Advantages:

-   Easy to maintain
    
-   Highly reusable
    
-   Easy to test
    

----------

# Fat Page Objects

Example

```typescript
class LoginPage{

    async login(){

        await database.connect();

        await api.createUser();

        await fs.writeFile(...);

        await page.click(...);

        await sendEmail();

    }

}

```

Problems:

-   Too many responsibilities
    
-   Difficult to maintain
    
-   Hard to reuse
    
-   Violates separation of concerns
    

----------

# What Belongs in a Page Object?

✅ Locators

```typescript
usernameTextbox

passwordTextbox

loginButton

```

----------

✅ Page Actions

```typescript
login()

logout()

search()

save()

delete()

```

----------

✅ Page Verification

```typescript
isLoaded()

hasErrorMessage()

isEmpty()

isVisible()

```

----------

# What Does NOT Belong?

❌ API Calls

```typescript
createUser()

```

----------

❌ Database Queries

```typescript
executeQuery()

```

----------

❌ File Utilities

```typescript
readJson()

```

----------

❌ Random Data Generation

```typescript
generateRandomEmail()

```

----------

❌ Business Workflows

Example

```text
Login

↓

Create Customer

↓

Approve Loan

↓

Generate Invoice

↓

Logout

```

This belongs in a higher-level orchestration layer or directly in the test if it represents the business scenario.

----------

# One Page = One Class

Good

```text
LoginPage

DashboardPage

CheckoutPage

OrdersPage

```

Avoid

```text
ApplicationPage

```

A single class representing the whole application becomes difficult to maintain.

----------

# Composition over Inheritance

Instead of

```text
BasePage

↓

HeaderPage

↓

CheckoutPage

```

Prefer

```text
CheckoutPage

↓

Header Component

↓

Cart Component

↓

Address Component

```

Composition provides greater flexibility.

----------

# Component Integration

Example

```text
CheckoutPage

├── Header

├── CartSummary

├── ShippingForm

└── PaymentSection

```

Each component manages its own locators and actions.

----------

# Navigation

Good

```typescript
async open(){

    await this.page.goto("/login");

}

```

Avoid navigation logic scattered across tests.

----------

# Assertions

Simple page-level assertions are acceptable.

```typescript
async isLoaded(){

    await expect(this.heading).toBeVisible();

}

```

Avoid embedding complete business validations inside page objects.

----------

# Return Types

When navigation changes the page, return the next page object.

```typescript
async login(user: User){

    await this.loginButton.click();

    return new DashboardPage(this.page);

}

```

This creates a fluent and readable flow.

----------

# Method Naming

Good

```text
login()

logout()

search()

save()

delete()

```

Avoid

```text
clickLoginButton()

fillUsernameTextbox()

pressSearchIcon()

```

The implementation detail is already inside the page object. Method names should describe the user's intent.

----------

# Keep Locators Private

```typescript
private readonly loginButton =
    this.page.getByRole("button", { name: "Login" });

```

Expose behavior, not implementation.

----------

# Avoid Duplicated Locators

Bad

```typescript
page.getByRole("button",{name:"Save"})

```

Repeated across multiple methods.

Instead,

define it once.

----------

# Page Factory?

Some Selenium frameworks use a Page Factory pattern with reflection or annotations.

For Playwright, this is generally unnecessary because:

-   Locators are lazy by design.
    
-   Playwright locators are lightweight.
    
-   Native locator APIs already provide clean abstractions.
    

A simple constructor is usually sufficient.

----------

# Example of a Good Page

```typescript
export class LoginPage {

    constructor(private readonly page: Page){}

    private readonly username =
        this.page.getByLabel("Username");

    private readonly password =
        this.page.getByLabel("Password");

    private readonly loginButton =
        this.page.getByRole("button",{name:"Login"});

    async open(){

        await this.page.goto("/login");

    }

    async login(user: User){

        await this.username.fill(user.username);

        await this.password.fill(user.password);

        await this.loginButton.click();

        return new DashboardPage(this.page);

    }

}

```

This page has a single responsibility and a clean public API.

----------

# Common Anti-Patterns

## God Page

```text
ApplicationPage

↓

5000 Lines

```

One class containing every page interaction.

----------

## Duplicate Locators

Same locator copied across multiple page objects.

----------

## Utility Calls Everywhere

```typescript
RandomUtil

FileUtil

ExcelUtil

DatabaseUtil

```

inside every page object.

----------

## Assertions Everywhere

Pages should not become assertion libraries.

Keep validations focused on page state.

----------

## Exposing Locators

Bad

```typescript
loginPage.loginButton.click();

```

Tests should call page methods rather than manipulating locators directly.

----------

# Enterprise Example

```text
Test

↓

LoginPage

↓

DashboardPage

↓

OrdersPage

↓

CheckoutPage

```

Each page represents one logical screen and exposes business-friendly methods.

----------

# Best Practices

-   Keep page objects focused on a single page or view.
    
-   Expose user actions rather than low-level UI operations.
    
-   Prefer composition with reusable components over deep inheritance.
    
-   Return the next page object after navigation.
    
-   Keep locators private.
    
-   Avoid business logic, API calls, and database operations inside page objects.
    
-   Use meaningful method names that describe user intent.
    

----------

# Common Mistakes

### ❌ Creating One Huge Page Class

Split functionality into page objects and reusable components.

----------

### ❌ Treating Page Objects as Utility Classes

A page object should model the UI, not provide generic helper methods.

----------

### ❌ Returning Raw Locators

Expose behavior through methods instead of allowing tests to interact with locators directly.

----------

### ❌ Embedding End-to-End Business Flows

A page object should not orchestrate workflows spanning multiple pages.

----------

### ❌ Inheriting Everything from BasePage

Use inheritance sparingly. Prefer composition for shared UI elements.

----------

# Interview Questions

### Q1. What is the primary responsibility of a Page Object?

To encapsulate page-specific UI elements and interactions while hiding implementation details from tests.

----------

### Q2. What is the difference between a thin and a fat page object?

-   **Thin page objects** focus on UI interactions and simple page validations.
    
-   **Fat page objects** accumulate unrelated responsibilities such as API calls, database access, and business workflows, making them harder to maintain.
    

----------

### Q3. Why is composition preferred over inheritance in Page Object design?

Composition creates flexible, reusable building blocks and avoids the tight coupling and deep class hierarchies often introduced by inheritance.

----------

### Q4. Should page objects expose locators?

Generally, no. Page objects should expose behaviors through methods, keeping locators as implementation details.

----------

### Q5. Should page objects contain API calls?

No. API interactions belong in a dedicated service layer, allowing page objects to remain focused on UI behavior.

----------

# Summary

An enterprise Page Object Model is more than a collection of locators and click methods. Well-designed page objects encapsulate UI interactions, expose business-friendly behaviors, and maintain a single responsibility. By keeping page objects thin, favoring composition, hiding implementation details, and separating UI logic from services and utilities, teams can build automation frameworks that remain scalable and maintainable as applications evolve.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjY5ODcwMDY5XX0=
-->