This chapter is one of the most important for **enterprise automation**.

A framework that only runs against **one environment** (e.g., QA) is not enterprise-ready. Large organizations need to execute the same test suite across multiple environments without modifying test code.

This chapter focuses on building a **configuration system that is scalable, secure, and easy to maintain**.

----------

# Part 22 – Framework Best Practices

# Chapter 8 – Configuration & Environment Management

----------

# Introduction

Every automation framework depends on configuration.

Examples include:

-   Base URLs
    
-   Browser settings
    
-   Credentials
    
-   Timeouts
    
-   API endpoints
    
-   Database connections
    
-   Feature flags
    
-   Environment-specific values
    

Hardcoding these values inside tests or page objects makes the framework difficult to maintain.

A mature framework centralizes configuration and allows tests to run against different environments without code changes.

----------

# Why Configuration Management Matters

Imagine this test:

```typescript
await page.goto("https://qa.company.com");

```

Tomorrow the same test must execute against:

-   QA
    
-   UAT
    
-   Staging
    
-   Performance
    
-   Production
    

Hardcoding URLs forces code changes.

Instead, tests should remain environment-independent.

----------

# Characteristics of Good Configuration

A good configuration system should be:

-   Centralized
    
-   Environment-aware
    
-   Secure
    
-   Easy to extend
    
-   Type-safe
    
-   Validated
    
-   Independent of test logic
    

----------

# Enterprise Architecture

```text
Tests

↓

Configuration

↓

Environment

↓

Application

```

Tests should never know which environment they are running against.

----------

# Folder Structure

```text
src/

├── config/

│     ├── config.ts

│     ├── qa.ts

│     ├── uat.ts

│     ├── staging.ts

│     ├── production.ts

│     ├── environments.ts

│     └── validator.ts

```

----------

# Environment Configuration

Example

```typescript
export default {

    baseUrl: "https://qa.company.com",

    apiUrl: "https://qa-api.company.com",

    timeout: 30000

};

```

Each environment has its own configuration file.

----------

# Multiple Environments

```text
QA

↓

UAT

↓

Staging

↓

Production

```

The same test suite executes against all environments.

----------

# Selecting an Environment

Example

```bash
ENV=qa npx playwright test

```

or

```bash
ENV=staging npx playwright test

```

The framework selects the appropriate configuration automatically.

----------

# Configuration Factory

Instead of

```typescript
if(environment==="QA")

```

throughout the codebase,

centralize the logic.

```typescript
import qa from './qa';
import uat from './uat';
import staging from './staging';

const configs = {
    qa,
    uat,
    staging
};

export const config =
    configs[process.env.ENV ?? "qa"];

```

Tests consume only `config`.

----------

# Using Configuration

Instead of

```typescript
await page.goto(

"https://qa.company.com"

);

```

Use

```typescript
await page.goto(

config.baseUrl

);

```

The test never changes.

----------

# Environment Variables

Some values belong in environment variables.

Examples

```text
ENV

BASE_URL

API_URL

USERNAME

PASSWORD

```

Access

```typescript
process.env.BASE_URL

```

----------

# Using .env Files

Development environments often use `.env` files.

Example

```text
.env.qa

.env.uat

.env.staging

.env.production

```

Each file contains environment-specific values.

Example

```text
BASE_URL=https://qa.company.com

API_URL=https://qa-api.company.com

```

----------

# Loading .env Files

Example

```typescript
import dotenv from "dotenv";

dotenv.config({

    path: `.env.${process.env.ENV}`

});

```

This loads the correct configuration automatically.

----------

# Secrets Management

Never store secrets in:

-   Git repositories
    
-   Test files
    
-   Page objects
    
-   Configuration files committed to source control
    

Instead, use:

-   GitHub Secrets
    
-   Azure Key Vault
    
-   Jenkins Credentials
    
-   Azure DevOps Variable Groups
    
-   GitLab CI Variables
    

----------

# Credentials

Avoid

```typescript
const password =

"Password123";

```

Prefer

```typescript
process.env.PASSWORD

```

----------

# Feature Flags

Modern applications often enable features selectively.

Configuration should expose them.

```typescript
config.features.newCheckout

```

Tests can adapt based on feature availability without hardcoding conditions.

----------

# Runtime Configuration

Configuration may depend on:

```text
Browser

↓

Environment

↓

Region

↓

Language

↓

Tenant

```

Runtime configuration allows the framework to adapt dynamically.

----------

# Browser Configuration

Example

```typescript
config.browser =

"chromium";

```

Or

```text
Chrome

Firefox

WebKit

```

Configuration determines execution without changing tests.

----------

# Timeout Configuration

Instead of

```typescript
waitForTimeout(5000)

```

Use

```typescript
config.timeouts.default

```

Centralizing timeouts makes adjustments easier.

----------

# API Configuration

```typescript
config.api.baseUrl

config.api.timeout

config.api.headers

```

Keep UI and API configuration together when appropriate.

----------

# Database Configuration

Example

```typescript
config.database.host

config.database.port

config.database.schema

```

Useful for service-layer integrations.

----------

# Tenant Configuration

Multi-tenant applications often require:

```text
Tenant A

Tenant B

Tenant C

```

Configuration can select the correct tenant at runtime.

----------

# Region Configuration

Global applications may support:

```text
US

EU

APAC

```

Each region may have different endpoints and data.

----------

# Configuration Validation

Fail fast if required values are missing.

Example

```typescript
if (!process.env.BASE_URL) {

    throw new Error(

        "BASE_URL is required"

    );

}

```

Validation prevents confusing runtime failures.

----------

# Type-Safe Configuration

Define an interface.

```typescript
interface Config {

    baseUrl: string;

    apiUrl: string;

    timeout: number;

}

```

This provides compile-time safety.

----------

# Configuration Ownership

Layer

Responsibility

Config Files

Environment values

Factory

Select configuration

Validator

Validate required settings

Tests

Consume configuration only

----------

# Enterprise Configuration Flow

```text
ENV Variable

↓

Configuration Factory

↓

Validation

↓

Playwright Config

↓

Tests

```

The flow remains consistent regardless of the target environment.

----------

# Recommended Folder Structure

```text
config/

├── config.ts

├── environments.ts

├── validator.ts

├── qa.ts

├── uat.ts

├── staging.ts

├── production.ts

├── featureFlags.ts

└── browser.ts

```

----------

# Best Practices

-   Centralize all configuration in dedicated modules.
    
-   Keep tests environment-agnostic.
    
-   Use environment variables for sensitive values.
    
-   Validate required configuration at startup.
    
-   Prefer type-safe configuration objects.
    
-   Separate secrets from version-controlled configuration.
    
-   Support multiple environments without changing test code.
    
-   Treat feature flags as configuration rather than business logic.
    

----------

# Common Mistakes

### ❌ Hardcoding URLs

Bad

```typescript
await page.goto(

"https://qa.company.com"

);

```

Always use configuration.

----------

### ❌ Committing Secrets

Never commit:

-   Passwords
    
-   API Keys
    
-   Client Secrets
    
-   Tokens
    

to source control.

----------

### ❌ Environment Checks Throughout the Code

Bad

```typescript
if (process.env.ENV === "QA") {
    // ...
}

```

Centralize environment selection inside a configuration factory.

----------

### ❌ Duplicate Configuration

Avoid copying the same values across multiple files.

Extract shared configuration where appropriate.

----------

### ❌ Missing Validation

Don't allow the framework to start with incomplete configuration.

Validate required values during startup.

----------

# Interview Questions

### Q1. Why should configuration be centralized?

Centralized configuration reduces duplication, simplifies maintenance, and allows tests to run across multiple environments without modification.

----------

### Q2. Why should secrets not be stored in configuration files?

Configuration files are often version-controlled. Secrets should be managed through secure secret stores or CI/CD credential management systems.

----------

### Q3. What is the purpose of a configuration factory?

A configuration factory selects the appropriate environment-specific configuration and exposes a single interface to the rest of the framework.

----------

### Q4. Why validate configuration during startup?

Validation detects missing or invalid settings early, preventing difficult-to-diagnose runtime failures.

----------

### Q5. How should feature flags be handled in an automation framework?

Feature flags should be exposed through configuration so tests can adapt to enabled features without hardcoded environment-specific logic.

----------

# Summary

Configuration and environment management are foundational to enterprise Playwright frameworks. By centralizing configuration, separating secrets, supporting multiple environments, validating settings at startup, and exposing a consistent configuration interface, teams can build automation that is secure, maintainable, and portable across QA, UAT, staging, and production environments without changing test code.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3NzQ5NzY5MTJdfQ==
-->