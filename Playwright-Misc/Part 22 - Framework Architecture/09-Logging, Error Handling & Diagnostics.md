In reality, **writing tests is only half the job**. The other half is **diagnosing failures quickly**.

A mature automation framework should answer these questions immediately when a test fails:

-   What failed?
    
-   Where did it fail?
    
-   Why did it fail?
    
-   Was it an application issue or an automation issue?
    
-   What evidence do we have?
    

The goal of this chapter is to design a framework where failures are **self-explanatory**.

----------

# Part 22 – Framework Best Practices

# Chapter 9 – Logging, Error Handling & Diagnostics

----------

# Introduction

Every automation test will eventually fail.

Failures can be caused by:

-   Application bugs
    
-   Environment issues
    
-   Network failures
    
-   Test data problems
    
-   Automation defects
    

Without proper logging and diagnostics, debugging becomes slow and expensive.

A mature framework captures enough information so that engineers can investigate failures without rerunning the test.

----------

# Why Logging Matters

Imagine a failed test.

Without logging:

```text
TimeoutError

↓

Test Failed

```

Questions remain unanswered:

-   Which page?
    
-   Which action?
    
-   Which user?
    
-   Which environment?
    

----------

With good logging:

```text
Login

↓

User: admin@test.com

↓

Page: Login

↓

Click Login

↓

Response: 500

↓

Screenshot

↓

Trace

↓

Video

```

Root cause analysis becomes much faster.

----------

# Logging Architecture

```text
Tests

↓

Framework Logger

↓

Console

↓

File

↓

CI Logs

↓

Artifacts

```

One logging system should serve both local execution and CI/CD.

----------

# Logging Levels

A framework should support multiple log levels.

Level

Purpose

TRACE

Very detailed execution information

DEBUG

Debugging framework behavior

INFO

Normal execution events

WARN

Recoverable issues

ERROR

Failures requiring attention

Choose the level appropriate for the audience and environment.

----------

# Example Log Flow

```text
INFO

↓

Open Login Page

↓

INFO

↓

Enter Username

↓

INFO

↓

Click Login

↓

ERROR

↓

Dashboard Not Visible

```

The sequence tells the execution story.

----------

# What Should Be Logged?

Useful information includes:

-   Test name
    
-   Browser
    
-   Environment
    
-   User
    
-   Page
    
-   Action
    
-   URL
    
-   Timestamp
    
-   Duration
    
-   Error message
    

Avoid logging sensitive information such as passwords or tokens.

----------

# Framework Logger

Instead of

```typescript
console.log("Login");

```

Create a reusable logger.

```typescript
Logger.info("Logging in user");

```

This allows consistent formatting and output destinations.

----------

# Structured Logging

Prefer structured messages over free-form text.

Example

```text
Action: Login

User: admin@test.com

Environment: QA

Browser: Chromium

Duration: 1200 ms

```

Structured logs are easier to search and analyze.

----------

# Correlation IDs

In distributed systems, assign a unique identifier to each test execution.

```text
Execution ID

↓

Test

↓

API Calls

↓

Logs

↓

Reports

```

This links related events across systems.

----------

# Timing Information

Capture durations for important actions.

Example

```text
Login

↓

850 ms

```

Performance trends become visible over time.

----------

# Error Classification

Not all failures are equal.

Error Type

Example

Application

HTTP 500, validation bug

Automation

Incorrect locator

Environment

Server unavailable

Test Data

Missing customer

Infrastructure

Browser crash

Categorizing failures speeds investigation.

----------

# Custom Error Classes

Instead of generic errors,

create meaningful exceptions.

```typescript
throw new AuthenticationError(

    "Login failed"

);

```

Examples:

-   `AuthenticationError`
    
-   `DataCreationError`
    
-   `ConfigurationError`
    
-   `EnvironmentError`
    

----------

# Exception Handling

Handle expected failures gracefully.

```typescript
try {

    await loginPage.login(user);

} catch (error) {

    Logger.error(error);

    throw error;

}

```

Avoid swallowing exceptions.

----------

# Screenshot Strategy

Capture screenshots:

-   On failure
    
-   On retries
    
-   Optionally at key checkpoints
    

```text
Failure

↓

Screenshot

↓

Artifact

```

Avoid capturing screenshots after every step unless there is a specific need, as this increases storage usage.

----------

# Trace Strategy

Playwright traces capture:

-   Actions
    
-   DOM snapshots
    
-   Network requests
    
-   Console logs
    

Recommended configuration:

```typescript
use: {

    trace: "on-first-retry"

}

```

This balances diagnostic value with storage costs.

----------

# Video Strategy

Videos are helpful for:

-   UI timing issues
    
-   Animation problems
    
-   Flaky failures
    

Recommended:

```typescript
video: "retain-on-failure"

```

----------

# Network Diagnostics

Capture network requests for failed tests.

Useful information:

-   Request URL
    
-   Status code
    
-   Response time
    
-   Failed requests
    

This helps distinguish UI issues from backend failures.

----------

# Console Logs

Collect browser console output.

Example

```text
Console Error

↓

TypeError

↓

Application Stack Trace

```

This often reveals JavaScript errors affecting the UI.

----------

# Diagnostic Artifacts

A typical failure should include:

```text
Screenshot

↓

Trace

↓

Video

↓

Console Logs

↓

Network Logs

↓

Framework Logs

```

Together they provide a complete picture.

----------

# Retry Diagnostics

If a retry succeeds,

mark the test as flaky.

```text
Fail

↓

Retry

↓

Pass

↓

Flaky

```

Do not silently ignore intermittent failures.

----------

# Logging API Calls

Example

```text
POST /orders

↓

201 Created

↓

340 ms

```

This simplifies backend debugging.

----------

# Logging Database Operations

Log:

-   Query name
    
-   Duration
    
-   Result
    

Avoid logging sensitive data.

----------

# Logging Test Data

Useful:

```text
Customer ID

Order ID

Tenant

Region

```

Avoid logging:

-   Passwords
    
-   Tokens
    
-   Personal information
    

----------

# CI/CD Integration

Publish logs and artifacts with every pipeline run.

```text
Pipeline

↓

Artifacts

↓

Logs

↓

Trace

↓

HTML Report

```

Developers should not need to rerun a test just to investigate it.

----------

# Enterprise Logging Architecture

```text
Tests

↓

Framework Logger

↓

Playwright Reporter

↓

Artifacts

↓

CI Dashboard

```

----------

# Recommended Folder Structure

```text
logs/

├── framework/

├── api/

├── browser/

├── screenshots/

├── traces/

├── videos/

└── reports/

```

----------

# Best Practices

-   Use a centralized logging framework instead of scattered `console.log()` statements.
    
-   Log business actions rather than low-level implementation details.
    
-   Capture screenshots, traces, and videos on failures.
    
-   Use structured logs with consistent formatting.
    
-   Classify failures into meaningful categories.
    
-   Record timing information for important operations.
    
-   Protect sensitive information by masking or omitting it from logs.
    
-   Publish all diagnostic artifacts in CI/CD pipelines.
    

----------

# Common Mistakes

### ❌ Excessive Logging

Logging every locator interaction creates noise and makes useful information harder to find.

----------

### ❌ Logging Secrets

Never log:

-   Passwords
    
-   Access tokens
    
-   API keys
    
-   Session cookies
    

----------

### ❌ Swallowing Exceptions

Avoid:

```typescript
catch (error) {

    Logger.error(error);

}

```

without rethrowing or handling the failure appropriately.

----------

### ❌ No Diagnostic Artifacts

A failed test without screenshots, traces, or logs is much harder to investigate.

----------

### ❌ Using Only `console.log`

A centralized logging system provides consistent formatting, filtering, and integration with reports and CI tools.

----------

# Interview Questions

### Q1. Why is structured logging better than plain text logging?

Structured logs are easier to search, filter, analyze, and correlate across different systems and pipeline runs.

----------

### Q2. When should Playwright traces be enabled?

A common strategy is `trace: "on-first-retry"` or `trace: "retain-on-failure"` to balance diagnostic value with storage usage.

----------

### Q3. Why should passwords never appear in logs?

Logs are often stored and shared across systems. Sensitive information in logs creates security risks.

----------

### Q4. What diagnostic artifacts are most useful for debugging UI failures?

Screenshots, traces, videos, browser console logs, network logs, and framework logs together provide comprehensive debugging information.

----------

### Q5. Why should failures be classified?

Categorizing failures (application, automation, environment, test data, infrastructure) helps teams assign ownership and prioritize investigation.

----------

# Summary

Logging, error handling, and diagnostics are essential components of an enterprise Playwright framework. By implementing structured logging, meaningful error classification, secure logging practices, and comprehensive diagnostic artifact collection, teams can dramatically reduce the time required to investigate failures. A mature framework should make failures easy to understand without requiring repeated test executions.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTg2Njk5MDYxOF19
-->