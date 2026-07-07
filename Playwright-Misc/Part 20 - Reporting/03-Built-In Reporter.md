This chapter is very important because many engineers only know about the **HTML Reporter**, but Playwright actually ships with **multiple built-in reporters**, each designed for a different purpose.

For example:

-   During development → **List Reporter**
    
-   CI Console → **Line Reporter**
    
-   Very large pipelines → **Dot Reporter**
    
-   Jenkins/Azure DevOps → **JUnit Reporter**
    
-   Dashboards → **JSON Reporter**
    
-   Distributed execution → **Blob Reporter**
    

Knowing **when** to use each reporter is much more valuable than simply knowing **how** to configure them.

----------

# Part 20 – Reporting

# Chapter 3 – Built-in Reporters

----------

# Introduction

A **reporter** is responsible for presenting test execution results.

Playwright separates:

```text
Test Execution

↓

Reporter

↓

Output

```

The same execution can generate multiple outputs simultaneously.

For example,

```text
Run Tests

↓

HTML Report

+

Console Output

+

JUnit XML

+

JSON Results

```

This flexibility allows different stakeholders to consume the same execution in different ways.

----------

# Built-in Reporters

Playwright includes the following built-in reporters.

Reporter

Best For

List

Local development

Line

CI pipelines

Dot

Large test suites

HTML

Interactive debugging

JSON

Dashboards & automation

JUnit

Jenkins/Azure DevOps

Blob

Merging distributed reports

----------

# Reporter Selection Flow

```text
Run Tests

↓

Reporter

↓

Console

↓

Files

↓

Dashboards

↓

CI/CD

```

----------

# List Reporter

## Introduction

The **List Reporter** prints every test in the console.

Example

```text
✓ Login Test

✓ Create Customer

✓ Create Order

✗ Payment Test

✓ Logout Test

```

It is the default reporter when running Playwright locally.

----------

## Configuration

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({

    reporter: 'list'

});

```

----------

## Advantages

-   Easy to read
    
-   Shows every test
    
-   Good for debugging
    
-   Ideal for local execution
    

----------

## Disadvantages

Large test suites can generate very long console output.

----------

# Line Reporter

## Introduction

Instead of printing every test,

Line Reporter continuously updates the same console line.

Example

```text
Running 125/1000 tests...

```

When a failure occurs,

it prints the failure details.

----------

## Configuration

```typescript
export default defineConfig({

    reporter: 'line'

});

```

----------

## Advantages

-   Compact output
    
-   Cleaner CI logs
    
-   Less scrolling
    
-   Suitable for medium and large suites
    

----------

## Best Use Case

```text
GitHub Actions

Azure DevOps

GitLab

Jenkins

```

----------

# Dot Reporter

## Introduction

Dot Reporter is extremely compact.

Example

```text
................F..............

```

Meaning

```text
.

↓

Passed

F

↓

Failed

```

Each character represents one test.

----------

## Configuration

```typescript
export default defineConfig({

    reporter: 'dot'

});

```

----------

## Advantages

-   Very small output
    
-   Fast console rendering
    
-   Excellent for huge suites
    

----------

## Disadvantages

Provides very little information until the execution completes.

----------

# HTML Reporter

Already covered in detail.

Quick summary:

```text
Interactive Report

↓

Screenshots

↓

Trace

↓

Video

↓

Attachments

```

Best for

-   Developers
    
-   Testers
    
-   Debugging
    

----------

# JSON Reporter

## Introduction

Instead of human-readable output,

JSON Reporter creates a machine-readable file.

Example

```json
{
  "status":"passed",
  "tests":[]
}

```

----------

## Configuration

```typescript
export default defineConfig({

    reporter: [

        [

            'json',

            {

                outputFile:

                    'reports/results.json'

            }

        ]

    ]

});

```

----------

## Typical Uses

-   Dashboards
    
-   Analytics
    
-   Custom reports
    
-   Data processing
    
-   Trend analysis
    

----------

## Example Workflow

```text
Playwright

↓

JSON

↓

Dashboard

↓

Charts

↓

Metrics

```

----------

# JUnit Reporter

## Introduction

Many CI servers understand JUnit XML.

Playwright can generate it directly.

Example

```xml
<testsuite>

<testcase>

</testcase>

</testsuite>

```

----------

## Configuration

```typescript
export default defineConfig({

    reporter: [

        [

            'junit',

            {

                outputFile:

                    'reports/results.xml'

            }

        ]

    ]

});

```

----------

## Supported By

-   Jenkins
    
-   Azure DevOps
    
-   GitLab
    
-   Bamboo
    
-   TeamCity
    
-   CircleCI
    

Almost every CI tool understands JUnit XML.

----------

# Blob Reporter

## Introduction

Blob Reporter is designed for **distributed** or **parallel** execution.

Imagine:

```text
Machine 1

↓

Tests A

----------------

Machine 2

↓

Tests B

----------------

Machine 3

↓

Tests C

```

Each machine creates a **blob report**.

Later,

they are merged into one final report.

----------

## Configuration

```typescript
export default defineConfig({

    reporter: 'blob'

});

```

----------

## Merge Workflow

```text
Machine 1

↓

Blob

-------------

Machine 2

↓

Blob

-------------

Machine 3

↓

Blob

↓

Merge

↓

Single HTML Report

```

This is especially useful for distributed CI/CD pipelines.

----------

# Multiple Reporters

One execution can generate multiple reports.

```typescript
export default defineConfig({

    reporter: [

        ['list'],

        ['html'],

        [

            'json',

            {

                outputFile:

                    'reports/results.json'

            }

        ],

        [

            'junit',

            {

                outputFile:

                    'reports/results.xml'

            }

        ]

    ]

});

```

One test execution produces:

```text
Console

+

HTML

+

JSON

+

JUnit

```

----------

# Choosing the Right Reporter

Scenario

Recommended Reporter

Local development

List

Debugging

HTML

CI console

Line

Very large suites

Dot

Dashboards

JSON

Jenkins/Azure DevOps

JUnit

Distributed execution

Blob

----------

# Reporter Comparison

Reporter

Human Readable

Machine Readable

Interactive

List

✅

❌

❌

Line

✅

❌

❌

Dot

✅

❌

❌

HTML

✅

❌

✅

JSON

❌

✅

❌

JUnit

❌

✅

❌

Blob

❌

Intermediate format

❌

----------

# Enterprise Reporting Strategy

A common enterprise configuration is:

```typescript
reporter: [

    ['line'],

    ['html'],

    ['junit'],

    ['json']

]

```

Workflow

```text
Playwright

↓

Console

↓

HTML

↓

JUnit

↓

JSON

↓

CI Pipeline

↓

Dashboard

```

Each reporter serves a different audience.

----------

# Blob Report Merging

After distributed execution,

merge blob reports.

```bash
npx playwright merge-reports blob-report

```

Generate the final report.

```bash
npx playwright show-report

```

This is commonly used in large CI/CD pipelines where tests are split across multiple machines.

----------

# Suggested Folder Structure

```text
reports/

├── html/

├── results.json

├── results.xml

├── blob-report/

└── merged-report/

```

----------

# Reporter Performance

Reporter

Performance Impact

Dot

Very Low

Line

Low

List

Medium

JSON

Low

JUnit

Low

HTML

Medium

Blob

Low

The differences are usually small, but HTML reporting generates additional assets and is therefore slightly heavier.

----------

# Best Practices

-   Use **List** during local development.
    
-   Use **Line** for CI logs to keep output concise.
    
-   Generate **HTML** reports for debugging.
    
-   Publish **JUnit XML** for CI integration.
    
-   Use **JSON** when feeding dashboards or analytics tools.
    
-   Use **Blob** for distributed execution and merge reports afterward.
    
-   Configure multiple reporters to satisfy different audiences.
    

----------

# Common Mistakes

### ❌ Using HTML Reporter Alone

HTML reports are excellent for developers,

but CI systems generally require:

-   JUnit
    
-   JSON
    

for automated processing.

----------

### ❌ Using List Reporter for Huge Suites

Large suites may produce thousands of console lines.

Prefer:

```text
Line

or

Dot

```

----------

### ❌ Forgetting Blob Merge

Generating blob reports without merging them results in fragmented execution results.

Always merge distributed reports before publishing.

----------

### ❌ Using JSON for Manual Debugging

JSON is intended for tools and dashboards.

Use the HTML report for human investigation.

----------

# Interview Questions

### Q1. Which Playwright reporter is best for local development?

**List Reporter**, because it shows detailed test progress in the console.

----------

### Q2. Which reporter is most commonly used by CI systems?

**JUnit Reporter**, because its XML format is widely supported by CI/CD tools such as Jenkins, Azure DevOps, GitLab, and TeamCity.

----------

### Q3. Why would you use the JSON Reporter?

To generate machine-readable results for dashboards, analytics, custom reporting, or integrations.

----------

### Q4. What problem does the Blob Reporter solve?

It enables reports from distributed or sharded test executions to be merged into a single consolidated report.

----------

### Q5. Can Playwright use multiple reporters simultaneously?

Yes. A single test execution can generate multiple report formats, allowing different consumers (developers, CI systems, dashboards) to use the format best suited to their needs.

----------

# Summary

Playwright's built-in reporters address a wide range of reporting needs, from concise console output to interactive debugging and CI/CD integration. By selecting the appropriate reporter—or combining multiple reporters—you can deliver meaningful execution results to developers, testers, managers, and automated systems without changing your test execution process.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5OTgwNzE2OTJdfQ==
-->