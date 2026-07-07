Many teams adopt Playwright expecting it to be fast by default. While Playwright is significantly faster than Selenium, **poor framework design can still result in slow, resource-intensive test suites**.

This chapter focuses on building **high-performance, scalable Playwright frameworks** that can support thousands—or even tens of thousands—of tests.

----------

# Part 22 – Framework Best Practices

# Chapter 12 – Framework Performance & Scalability

----------

# Introduction

Performance optimization is not about making one test faster.

It is about ensuring that:

-   Thousands of tests execute efficiently
    
-   Infrastructure costs remain reasonable
    
-   Developers receive rapid feedback
    
-   CI pipelines stay reliable
    
-   The framework scales as applications grow
    

A scalable framework optimizes execution without sacrificing test isolation or maintainability.

----------

# Performance Goals

An enterprise framework should aim for:

-   Fast execution
    
-   High parallelism
    
-   Low memory usage
    
-   Minimal flakiness
    
-   Efficient resource utilization
    
-   Predictable execution times
    

----------

# Performance Architecture

```text
Tests

↓

Fixtures

↓

Workers

↓

Browser Contexts

↓

Browsers

↓

Application

```

Optimization can happen at every layer.

----------

# Performance Bottlenecks

Typical causes of slow execution:

-   Repeated UI logins
    
-   Excessive browser launches
    
-   Poor test data setup
    
-   Sequential execution
    
-   Static waits
    
-   Heavy page objects
    
-   Large screenshots/videos
    
-   Duplicate setup logic
    

Measure before optimizing.

----------

# Browser Reuse vs Browser Isolation

### Option 1 – New Browser Per Test

```text
Test 1

↓

Browser

----------------

Test 2

↓

Browser

```

Advantages:

-   Maximum isolation
    

Disadvantages:

-   Very slow
    
-   High memory usage
    

----------

### Option 2 – Shared Browser

```text
Browser

↓

Context 1

↓

Test 1

----------------

Context 2

↓

Test 2

```

This is Playwright's default approach and provides an excellent balance of speed and isolation.

----------

# Browser Context Strategy

Each test should receive a fresh browser context.

```text
Browser

↓

Context

↓

Page

```

Contexts are lightweight and isolated.

Avoid sharing browser contexts between unrelated tests.

----------

# Page Lifecycle

Recommended lifecycle:

```text
Context

↓

New Page

↓

Run Test

↓

Close Page

```

Keep page lifetimes short.

----------

# Worker Optimization

Workers execute tests in parallel.

Choose worker counts based on available CPU and memory.

Example

```typescript
workers:

process.env.CI ? 4 : undefined

```

Increasing workers beyond available resources often reduces performance.

----------

# Smart Parallelization

Not every test should run in parallel.

Suitable candidates:

-   Read-only tests
    
-   Independent CRUD operations
    
-   API tests
    

Sequential candidates:

-   Tests sharing external resources
    
-   Ordered business workflows
    
-   Legacy systems with concurrency limitations
    

----------

# Storage State Optimization

Avoid logging in through the UI for every test.

Slow

```text
Open Login

↓

Enter Username

↓

Enter Password

↓

Submit

↓

Dashboard

```

Fast

```text
Storage State

↓

Authenticated Context

↓

Dashboard

```

This often saves minutes across a large suite.

----------

# API-First Setup

Instead of:

```text
UI

↓

Create Customer

↓

2 Minutes

```

Use:

```text
API

↓

Create Customer

↓

2 Seconds

```

Reserve the UI for validating UI behavior.

----------

# Lazy Fixture Initialization

Fixtures are created only when requested.

```text
Test

↓

Requests

↓

loginPage

↓

Created

```

Unused fixtures consume no resources.

----------

# Reduce Duplicate Setup

Avoid:

```text
Login

↓

Create User

↓

Navigate

↓

Repeat

↓

Repeat

↓

Repeat

```

Move common setup into fixtures or API services.

----------

# Smart Waiting

Avoid

```typescript
waitForTimeout(5000);

```

Prefer Playwright's auto-waiting and explicit expectations.

```typescript
await expect(button).toBeVisible();

```

Static waits waste execution time.

----------

# Locator Optimization

Prefer stable, accessible locators.

Example

```typescript
getByRole()

getByLabel()

getByTestId()

```

Avoid brittle XPath expressions where better alternatives exist.

----------

# Network Optimization

When appropriate:

-   Mock unnecessary external services
    
-   Block analytics calls
    
-   Block advertisements
    
-   Block third-party tracking
    

Example

```text
Application

↓

Block Analytics

↓

Run Test

```

This reduces execution variability.

----------

# Test Data Optimization

Generate data through APIs instead of the UI.

Reuse immutable reference data.

Generate only the data required for the scenario.

----------

# Screenshot Strategy

Bad

```text
Screenshot

↓

Every Step

```

Good

```text
Screenshot

↓

Failure Only

```

Artifacts consume storage and pipeline time.

----------

# Video Strategy

Recommended

```typescript
video:

"retain-on-failure"

```

Recording every successful execution increases storage requirements.

----------

# Trace Strategy

Recommended

```typescript
trace:

"on-first-retry"

```

Provides diagnostics without excessive overhead.

----------

# Memory Optimization

Avoid:

-   Large global objects
    
-   Unused fixtures
    
-   Long-lived page references
    
-   Accumulating test data in memory
    

Release resources promptly.

----------

# Test Suite Partitioning

Large suites benefit from logical separation.

```text
Smoke

↓

API

↓

Regression

↓

Accessibility

↓

Performance

```

Execute only what is required.

----------

# Sharding

Distribute very large suites across multiple machines.

```text
Machine 1

↓

25%

----------------

Machine 2

↓

25%

----------------

Machine 3

↓

25%

----------------

Machine 4

↓

25%

```

Sharding complements worker-based parallelism.

----------

# Caching

Cache where appropriate:

-   Node modules
    
-   Playwright browsers
    
-   Build artifacts
    

Avoid caching mutable test data.

----------

# CI Optimization

A fast pipeline typically uses:

```text
Cache Dependencies

↓

Build

↓

Parallel Jobs

↓

Workers

↓

Artifacts

```

Every stage should contribute value.

----------

# Performance Metrics

Track:

Metric

Purpose

Total execution time

Overall pipeline health

Average test duration

Identify slow tests

Slowest 10 tests

Optimization targets

Worker utilization

Resource efficiency

Memory usage

Detect leaks

Retry rate

Flakiness indicator

----------

# Profiling Slow Tests

If one test is significantly slower than others:

Check:

-   Login strategy
    
-   API setup
    
-   Static waits
    
-   Network latency
    
-   Large uploads/downloads
    
-   Inefficient selectors
    

Optimize the root cause rather than adding more workers.

----------

# Scaling Strategy

Framework evolution:

```text
100 Tests

↓

500 Tests

↓

2000 Tests

↓

10000 Tests

```

As the suite grows:

-   Increase parallelism gradually
    
-   Introduce sharding
    
-   Optimize setup
    
-   Monitor infrastructure costs
    

----------

# Enterprise Performance Architecture

```text
CI

↓

Matrix

↓

Workers

↓

Browser

↓

Context

↓

Page

↓

Application

```

Optimization occurs at multiple layers.

----------

# Performance Checklist

```text
✓ API Setup

✓ Storage State

✓ Parallel Workers

✓ Sharding

✓ Smart Waiting

✓ Lazy Fixtures

✓ Failure-Only Artifacts

✓ Cached Dependencies

✓ Independent Tests

```

Use this checklist when reviewing framework performance.

----------

# Best Practices

-   Prefer browser context isolation over launching new browsers for every test.
    
-   Use storage state to avoid repetitive UI logins.
    
-   Perform setup through APIs whenever possible.
    
-   Eliminate static waits.
    
-   Optimize worker counts based on hardware.
    
-   Capture heavy artifacts only when useful.
    
-   Continuously monitor execution metrics and optimize the slowest tests first.
    
-   Scale horizontally with sharding before overloading a single machine.
    

----------

# Common Mistakes

### ❌ Logging in Through the UI for Every Test

This is one of the most common performance bottlenecks.

----------

### ❌ Adding More Workers Without Measuring

More workers can increase CPU contention and reduce throughput.

----------

### ❌ Recording Videos for Every Test

Store videos only when they provide debugging value.

----------

### ❌ Using Static Waits

They increase execution time and often hide synchronization problems.

----------

### ❌ Sharing Mutable Test Data

This creates contention, retries, and flaky failures that negate performance gains.

----------

# Interview Questions

### Q1. What is the biggest performance improvement most Playwright frameworks can make?

Replacing repeated UI-based setup with API-based setup and storage state authentication typically provides the largest improvement.

----------

### Q2. Why are browser contexts preferred over launching multiple browsers?

Browser contexts are lightweight, isolated, and significantly faster while still providing strong test isolation.

----------

### Q3. Should every test run in parallel?

No. Tests should run in parallel only when they are independent and do not compete for shared mutable resources.

----------

### Q4. Why are static waits harmful?

They unnecessarily increase execution time and often conceal synchronization issues that should be solved using Playwright's waiting mechanisms.

----------

### Q5. What metrics should be monitored to improve framework performance?

Execution time, slowest tests, worker utilization, memory consumption, retry rate, pass rate, and infrastructure usage are key indicators.

----------

# Summary

Framework performance is achieved through thoughtful architecture rather than isolated optimizations. By combining efficient browser context management, API-first setup, storage state authentication, intelligent parallelization, lazy fixture initialization, and continuous performance monitoring, Playwright frameworks can scale from hundreds to tens of thousands of tests while remaining fast, reliable, and cost-effective.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0MTU0MTczMThdfQ==
-->