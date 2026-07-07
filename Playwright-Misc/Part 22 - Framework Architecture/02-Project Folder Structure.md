This is one of the most debated topics in automation.

If you ask 20 automation architects how they organize a Playwright project, you'll probably get 20 different answers.

The reality is:

> **There is no single perfect folder structure. The right structure depends on the project's size, team size, and complexity.**

In this chapter, we'll cover **how a framework evolves** from a small project to an enterprise-grade architecture.

----------

# Part 22 – Framework Best Practices

# Chapter 2 – Project Folder Structure

----------

# Introduction

A project structure defines how automation code is organized.

A well-designed structure makes it easier to:

-   Find files quickly
    
-   Add new features
    
-   Reduce duplication
    
-   Scale the framework
    
-   Onboard new team members
    
-   Maintain the project
    

A poor structure leads to confusion, duplicated utilities, and tightly coupled code.

----------

# Why Folder Structure Matters

Imagine a project after three years.

```text
4000 Tests

250 Pages

120 Components

60 Utilities

25 Engineers

```

Without a proper structure,

finding a single page object or utility becomes difficult.

Good organization improves both development speed and maintainability.

----------

# Framework Growth Journey

A Playwright framework typically evolves through stages.

```text
Small Project

↓

Medium Project

↓

Large Project

↓

Enterprise Framework

```

Trying to build an enterprise structure on day one is usually unnecessary, but planning for growth is important.

----------

# Small Project Structure

Suitable for:

-   Proof of Concepts (POCs)
    
-   Learning
    
-   Small internal tools
    
-   Fewer than 100 tests
    

```text
project/

├── tests/

├── pages/

├── playwright.config.ts

├── package.json

└── utils/

```

Advantages:

-   Very simple
    
-   Easy to understand
    
-   Quick setup
    

Disadvantages:

-   Doesn't scale well
    
-   Utilities become crowded
    
-   Difficult to separate responsibilities
    

----------

# Medium Project Structure

Suitable for:

-   100–500 tests
    
-   Small QA teams
    
-   One application
    

```text
project/

├── tests/

├── pages/

├── components/

├── fixtures/

├── test-data/

├── utils/

├── config/

├── reports/

├── playwright.config.ts

└── package.json

```

Benefits:

-   Better organization
    
-   Reusable components
    
-   Clear separation between tests and framework code
    

----------

# Enterprise Project Structure

Suitable for:

-   Thousands of tests
    
-   Multiple applications
    
-   Multiple teams
    

```text
project/

├── src/
│   ├── pages/
│   ├── components/
│   ├── services/
│   ├── fixtures/
│   ├── utilities/
│   ├── models/
│   ├── builders/
│   ├── constants/
│   ├── config/
│   └── types/
│
├── tests/
│
├── test-data/
│
├── reports/
│
├── scripts/
│
├── docs/
│
├── playwright.config.ts
│
└── package.json

```

Notice that framework code lives under **src**, while test cases live separately.

----------

# Layer-Based Organization

This is the approach I recommend for most enterprise Playwright frameworks.

```text
Tests

↓

Fixtures

↓

Pages

↓

Components

↓

Services

↓

Utilities

```

Each folder represents a technical responsibility.

Example:

```text
src/

├── pages/

├── components/

├── services/

├── fixtures/

├── utilities/

```

Advantages:

-   Clear responsibilities
    
-   Easy maintenance
    
-   Good reusability
    

----------

# Feature-Based Organization

Some teams organize by business domain.

```text
src/

├── authentication/

│     ├── pages/

│     ├── components/

│     └── services/

├── orders/

├── catalog/

├── checkout/

└── users/

```

Advantages:

-   Good domain ownership
    
-   Easier for product teams
    

Disadvantages:

-   Shared components may be duplicated
    
-   Cross-feature reuse requires discipline
    

----------

# Which One Should You Choose?

Project Type

Recommendation

Small POC

Layer-based

Single Application

Layer-based

Large Enterprise

Layer-based with feature modules where appropriate

Microservices

Feature-oriented modules with shared libraries

----------

# Tests Folder

Keep tests focused on business scenarios.

```text
tests/

├── smoke/

├── regression/

├── api/

├── integration/

└── e2e/

```

Avoid placing utilities or page objects inside the tests folder.

----------

# Pages Folder

Contains page-level interactions.

```text
pages/

├── LoginPage.ts

├── DashboardPage.ts

├── OrdersPage.ts

└── CheckoutPage.ts

```

Each page object should represent a single application page or view.

----------

# Components Folder

Reusable UI building blocks.

```text
components/

├── Header.ts

├── Footer.ts

├── Navigation.ts

├── ProductCard.ts

├── Modal.ts

└── SearchBar.ts

```

Components should be shared across multiple pages.

----------

# Fixtures Folder

Contains custom Playwright fixtures.

```text
fixtures/

├── base.fixture.ts

├── auth.fixture.ts

├── api.fixture.ts

└── admin.fixture.ts

```

Avoid putting business logic directly into fixtures.

----------

# Services Folder

Contains non-UI interactions.

```text
services/

├── AuthService.ts

├── UserService.ts

├── ProductService.ts

├── OrderService.ts

```

Responsibilities include:

-   API calls
    
-   Authentication
    
-   Database helpers
    
-   External integrations
    

----------

# Models Folder

Data models.

```text
models/

├── User.ts

├── Product.ts

├── Order.ts

```

Useful for TypeScript interfaces and domain objects.

----------

# Builders Folder

Implements the Builder Pattern for test data.

```text
builders/

├── UserBuilder.ts

├── ProductBuilder.ts

└── OrderBuilder.ts

```

Instead of manually creating large objects in every test.

----------

# Test Data Folder

Separate test data from test logic.

```text
test-data/

├── users.json

├── products.json

├── orders.json

```

This keeps tests cleaner and allows data reuse.

----------

# Config Folder

Environment-specific configuration.

```text
config/

├── qa.ts

├── uat.ts

├── prod.ts

└── config.ts

```

Never hardcode URLs or credentials in tests.

----------

# Constants Folder

Shared constants.

```text
constants/

├── urls.ts

├── timeouts.ts

├── roles.ts

└── messages.ts

```

Centralizing constants reduces duplication.

----------

# Utilities Folder

Generic helper functions.

```text
utilities/

├── DateUtil.ts

├── FileUtil.ts

├── JsonUtil.ts

├── RandomUtil.ts

```

Utilities should be generic and independent of business logic.

----------

# Scripts Folder

Automation support scripts.

```text
scripts/

├── generate-storage-state.ts

├── create-test-users.ts

├── cleanup-data.ts

```

These are not Playwright tests but supporting tools.

----------

# Reports Folder

Keep generated artifacts together.

```text
reports/

├── html/

├── allure/

├── junit/

├── screenshots/

├── traces/

└── videos/

```

This makes artifact management simpler.

----------

# Monorepo Structure

For organizations with multiple applications.

```text
repo/

├── apps/

│     ├── admin/

│     ├── storefront/

│     └── mobile-web/

├── shared/

│     ├── framework/

│     ├── components/

│     └── utilities/

└── tests/

```

Shared code is reused across applications.

----------

# Multi-Project Framework

Using Playwright projects.

```text
projects/

├── chromium/

├── firefox/

├── webkit/

├── mobile/

└── api/

```

Or configure them in `playwright.config.ts` while sharing the same source structure.

----------

# Naming Conventions

Prefer descriptive names.

Good:

```text
LoginPage.ts

CheckoutPage.ts

OrderService.ts

UserBuilder.ts

```

Avoid:

```text
Page1.ts

Helper.ts

Common.ts

Util.ts

```

Names should communicate intent.

----------

# Folder Naming Guidelines

-   Use lowercase folder names.
    
-   Use singular names when appropriate (`page` vs `pages` depends on team conventions, but be consistent).
    
-   Avoid abbreviations.
    
-   Keep names meaningful.
    
-   Follow a single naming convention across the repository.
    

Consistency is more important than the specific convention chosen.

----------

# Recommended Enterprise Structure

```text
project/

├── src/

│   ├── components/

│   ├── pages/

│   ├── services/

│   ├── fixtures/

│   ├── builders/

│   ├── models/

│   ├── utilities/

│   ├── constants/

│   ├── config/

│   └── types/

├── tests/

│   ├── smoke/

│   ├── regression/

│   ├── api/

│   └── e2e/

├── test-data/

├── scripts/

├── reports/

├── docs/

├── playwright.config.ts

└── package.json

```

This structure balances separation of concerns with discoverability and scales well for most enterprise teams.

----------

# Best Practices

-   Separate framework code from test cases.
    
-   Group code by responsibility rather than by convenience.
    
-   Extract reusable UI into components.
    
-   Keep test data outside test files.
    
-   Centralize configuration and constants.
    
-   Use meaningful names for files and folders.
    
-   Review and refactor the structure as the framework grows.
    

----------

# Common Mistakes

### ❌ Putting Everything in `utils`

A `utils` folder should not become a dumping ground for unrelated code.

Instead, create specific folders such as:

-   `services`
    
-   `builders`
    
-   `config`
    
-   `constants`
    

----------

### ❌ Business Logic Inside Tests

Tests should describe scenarios, not contain API implementations or data creation logic.

----------

### ❌ Deep Folder Nesting

Avoid structures like:

```text
src/
 └── pages/
      └── common/
           └── shared/
                └── base/

```

Excessive nesting reduces discoverability.

----------

### ❌ Duplicating Components

If multiple pages use the same navigation menu or modal, implement it once as a reusable component.

----------

### ❌ Hardcoding Test Data

Keep test data in builders, factories, APIs, or external data files rather than embedding it directly in tests.

----------

# Interview Questions

### Q1. Why should framework code be separated from test cases?

It improves maintainability, reuse, and readability while allowing framework changes without affecting business scenarios.

----------

### Q2. What is the difference between a page and a component?

-   A **page** represents a complete screen or view.
    
-   A **component** represents a reusable UI element shared across one or more pages.
    

----------

### Q3. Why is a dedicated services folder useful?

It centralizes API interactions and other non-UI operations, keeping page objects focused solely on UI behavior.

----------

### Q4. When should a project move from a simple to an enterprise folder structure?

As the number of tests, applications, reusable components, or contributors grows, additional structure helps improve scalability and maintainability.

----------

### Q5. Is feature-based or layer-based organization always better?

Neither is universally better. Layer-based organization works well for many automation frameworks, while feature-based organization can be advantageous for domain-driven or microservice-oriented teams. The choice should match the project's complexity and team structure.

----------

# Summary

A well-designed folder structure is the foundation of a maintainable Playwright framework. By separating tests from framework code, organizing files by clear responsibilities, and planning for future growth, teams can build automation projects that remain easy to understand, extend, and maintain as they scale from a few tests to thousands.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTI3NTIyMTc4N119
-->