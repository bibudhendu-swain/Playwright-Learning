Every project eventually creates a folder called:

```text
utils/

```

Six months later it contains:

-   150 utility classes
    
-   800 helper methods
    
-   Random API methods
    
-   Database code
    
-   Excel code
    
-   Playwright wrappers
    
-   Business logic
    
-   Logging
    
-   Constants
    

Eventually it becomes the infamous **"God Utils"** folder.

The goal of this chapter is to prevent that.

----------

# Part 22 – Framework Best Practices

# Chapter 10 – Utility Layer & Shared Libraries

----------

# Introduction

Utilities provide reusable functionality that is **generic** and **independent of business logic**.

Examples include:

-   Date formatting
    
-   JSON parsing
    
-   File handling
    
-   Random value generation
    
-   String manipulation
    
-   Encryption
    
-   Validation
    

Utilities should solve **technical problems**, not business problems.

----------

# What is a Utility Layer?

The Utility Layer contains reusable helper functions that are not tied to:

-   A page
    
-   A component
    
-   A service
    
-   A test
    

Instead, they provide generic capabilities that can be used anywhere.

----------

# Architecture

```text
Tests

↓

Pages

↓

Services

↓

Utilities

↓

Node.js / Playwright

```

Utilities support the framework but do not contain application-specific knowledge.

----------

# Characteristics of Good Utilities

A good utility should be:

-   Generic
    
-   Reusable
    
-   Stateless
    
-   Small
    
-   Well-tested
    
-   Independent
    

If it knows about your application's business domain, it probably doesn't belong in the Utility Layer.

----------

# Folder Structure

```text
utilities/

├── DateUtil.ts

├── StringUtil.ts

├── FileUtil.ts

├── JsonUtil.ts

├── RandomUtil.ts

├── ValidationUtil.ts

├── EncryptionUtil.ts

├── RetryUtil.ts

└── WaitUtil.ts

```

Notice that each utility has a clear responsibility.

----------

# Date Utility

Instead of

```typescript
const today = new Date();

const tomorrow = new Date(

today.getTime() + 86400000

);

```

Create

```typescript
DateUtil.tomorrow();

```

Example

```typescript
class DateUtil {

    static tomorrow(): Date {

        const date = new Date();

        date.setDate(date.getDate() + 1);

        return date;

    }

}

```

----------

# String Utility

Examples

```typescript
StringUtil.capitalize();

StringUtil.slugify();

StringUtil.camelCase();

StringUtil.removeWhitespace();

```

These methods are reusable across many projects.

----------

# File Utility

Example

```typescript
FileUtil.readJson();

FileUtil.writeJson();

FileUtil.readCsv();

FileUtil.exists();

```

Keep all file operations centralized.

----------

# JSON Utility

Instead of repeating parsing logic,

create reusable helpers.

```typescript
JsonUtil.prettyPrint();

JsonUtil.parse();

JsonUtil.stringify();

```

----------

# Random Utility

Useful for generating unique values.

```typescript
RandomUtil.uuid();

RandomUtil.email();

RandomUtil.number();

RandomUtil.string();

```

Remember:

Randomness should be used only when uniqueness is required.

----------

# Validation Utility

Examples

```typescript
ValidationUtil.isEmail();

ValidationUtil.isUuid();

ValidationUtil.isDate();

ValidationUtil.isEmpty();

```

Generic validation belongs here.

Business validation does not.

----------

# Encryption Utility

Useful for

-   Hashing
    
-   Base64
    
-   Encryption
    
-   Decryption
    

Example

```typescript
EncryptionUtil.encrypt();

EncryptionUtil.decrypt();

EncryptionUtil.hash();

```

----------

# Retry Utility

Some operations require retries.

Example

```typescript
RetryUtil.execute(

operation,

3

);

```

Useful for temporary network failures.

Avoid using retries to hide application defects.

----------

# Wait Utility

Be careful.

Avoid wrapping Playwright's intelligent waiting unnecessarily.

Bad

```typescript
WaitUtil.waitFiveSeconds();

```

Better

```typescript
WaitUtil.waitForApiReady();

```

Only create utilities that add real value.

----------

# Environment Utility

Example

```typescript
EnvironmentUtil.isQA();

EnvironmentUtil.isProduction();

```

Avoid spreading environment checks throughout the framework.

----------

# Path Utility

Useful for

```typescript
PathUtil.reportFolder();

PathUtil.downloadFolder();

PathUtil.testDataFolder();

```

Centralized path management improves portability.

----------

# HTTP Utility

Generic HTTP helpers.

```typescript
HttpUtil.buildHeaders();

HttpUtil.createQueryString();

```

Business-specific API calls belong in services, not utilities.

----------

# Serialization Utility

Example

```typescript
SerializationUtil.serialize();

SerializationUtil.deserialize();

```

Useful for storing and restoring objects.

----------

# Screenshot Utility

Instead of

```typescript
await page.screenshot(...);

```

everywhere,

create

```typescript
ScreenshotUtil.capture();

```

The utility can standardize:

-   Naming
    
-   Folder location
    
-   Timestamp
    
-   File format
    

----------

# Logging Utility

Avoid

```typescript
console.log(...)

```

Use

```typescript
Logger.info();

Logger.warn();

Logger.error();

```

A logger is a specialized utility.

----------

# Utility Composition

Utilities can collaborate.

Example

```text
FileUtil

↓

JsonUtil

↓

ValidationUtil

```

Each utility still has one responsibility.

----------

# Shared Libraries

Large organizations often extract utilities into internal packages.

Example

```text
company-playwright-utils

↓

Logging

↓

Retry

↓

Date

↓

Random

↓

Validation

```

Multiple automation projects reuse the same library.

----------

# Monorepo Example

```text
packages/

├── framework/

├── utilities/

├── reporting/

└── shared/

```

Utilities become organization-wide assets.

----------

# What Does NOT Belong in Utilities?

❌ Login

❌ Checkout

❌ Product Creation

❌ Customer Approval

❌ Order Placement

These belong in:

-   Pages
    
-   Components
    
-   Services
    

not utilities.

----------

# Business Logic

Bad

```typescript
Utility.createCustomer();

```

Good

```typescript
CustomerService.createCustomer();

```

Utilities should never know about business concepts.

----------

# Utility vs Service

Utility

Service

Generic

Business-specific

Stateless

Domain-aware

Technical helper

Application interaction

----------

# Utility vs Page

Utility

Page

Generic helper

UI interaction

Independent

Application-specific

----------

# Utility vs Component

Utility

Component

No UI

UI element

Generic

Page-specific behavior

----------

# Enterprise Utility Structure

```text
utilities/

├── date/

├── file/

├── json/

├── logging/

├── validation/

├── retry/

├── encryption/

├── random/

├── serialization/

└── path/

```

As the framework grows, grouping related utilities keeps them organized.

----------

# Avoid the God Utils Class

Bad

```text
CommonUtil

↓

200 Methods

```

Including:

-   File
    
-   JSON
    
-   API
    
-   Date
    
-   Database
    
-   Random
    
-   Logging
    

Split them into focused classes.

----------

# Static vs Instance Utilities

For stateless helpers, static methods are usually sufficient.

Example

```typescript
DateUtil.today();

StringUtil.capitalize();

```

If a utility needs configuration or dependencies (for example, a logger), an instance-based approach may be more appropriate.

----------

# Testing Utilities

Utilities should have their own unit tests.

Example

```text
DateUtil

↓

Unit Tests

```

Since utilities are reused extensively, defects here can affect many tests.

----------

# Enterprise Architecture

```text
Tests

↓

Pages

↓

Components

↓

Services

↓

Utilities

↓

Node.js APIs

```

Utilities remain the lowest reusable layer before external libraries.

----------

# Best Practices

-   Keep utilities generic and reusable.
    
-   Give each utility one clear responsibility.
    
-   Prefer small, focused utility classes over large helper collections.
    
-   Avoid wrapping Playwright APIs unless additional value is provided.
    
-   Unit test utility methods.
    
-   Share mature utilities across projects when appropriate.
    
-   Keep business logic out of the Utility Layer.
    

----------

# Common Mistakes

### ❌ God Utility Class

Avoid one massive helper class with unrelated methods.

----------

### ❌ Business Logic in Utilities

Bad

```typescript
Utility.placeOrder();

```

That belongs in a service.

----------

### ❌ Wrapping Every Playwright Method

Don't create wrappers like:

```typescript
ClickUtil.click(locator);

```

unless they add meaningful behavior such as logging, retry, or custom error handling.

----------

### ❌ Stateful Utilities

Utilities should generally not maintain mutable shared state.

----------

### ❌ Duplicate Helpers

Before creating a new utility, check whether similar functionality already exists.

----------

# Interview Questions

### Q1. What is the purpose of a Utility Layer?

The Utility Layer provides reusable, generic helper functions that solve technical problems independently of the application's business domain.

----------

### Q2. What is the difference between a utility and a service?

A utility is generic and domain-independent, while a service encapsulates business-specific interactions such as API calls or database operations.

----------

### Q3. Should Playwright methods always be wrapped in utilities?

No. Wrappers should be created only when they add meaningful value, such as standardized logging, retry logic, or consistent error handling.

----------

### Q4. Why should utilities be stateless?

Stateless utilities are easier to reuse, test, and execute safely in parallel without introducing shared-state issues.

----------

### Q5. What is the "God Utils" anti-pattern?

It is the practice of placing unrelated helper methods into one large utility class or folder, leading to poor organization, tight coupling, and maintenance challenges.

----------

# Summary

A well-designed Utility Layer provides reusable technical capabilities without becoming a dumping ground for unrelated code. By keeping utilities generic, focused, stateless, and independent of business logic, enterprise Playwright frameworks remain easier to maintain, test, and reuse across multiple projects. A disciplined utility strategy also prevents one of the most common architectural problems in long-lived automation frameworks: the "God Utils" anti-pattern.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbNjk5MDQ5NjcwLC0xOTY0OTI3MDM1XX0=
-->