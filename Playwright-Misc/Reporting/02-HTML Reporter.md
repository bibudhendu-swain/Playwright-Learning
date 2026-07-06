This is one of the most used features in Playwright.

If you ask 100 Playwright engineers which report they use the most, over **95%** will say:

> **HTML Reporter**

It is Playwright's flagship reporter because it provides an interactive UI with screenshots, traces, videos, errors, execution time, filters, retries, attachments, and much more.

----------

# Part 19 – Reporting

# Chapter 2 – HTML Reporter

----------

# Introduction

The HTML Reporter is Playwright's built-in interactive reporting system.

Unlike console logs, it provides a complete visual representation of your test execution.

It allows you to:

-   View test results
    
-   Navigate failed tests
    
-   Inspect errors
    
-   View screenshots
    
-   Watch videos
    
-   Open traces
    
-   Filter results
    
-   Search tests
    
-   Analyze retries
    

It is the primary debugging tool during Playwright development.

----------

# HTML Report Architecture

```text
Playwright Test Runner

↓

Execute Tests

↓

Collect Results

↓

Generate HTML Report

↓

Open in Browser

```

Unlike console reporters,

the HTML report persists after execution.

----------

# Default HTML Report

Enable HTML reporting.

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({

    reporter: 'html'

});

```

----------

# Multiple Reporters

Most enterprise projects use HTML together with other reporters.

```typescript
export default defineConfig({

    reporter: [

        ['list'],

        ['html']

    ]

});

```

This provides

```text
Console Output

+

Interactive HTML Report

```

----------

# Generate HTML Report

Simply run:

```bash
npx playwright test

```

If HTML reporting is enabled,

Playwright automatically generates the report.

----------

# Default Report Location

By default,

the report is created under

```text
playwright-report/

```

Project

```text
project/

├── tests/

├── playwright.config.ts

├── playwright-report/

└── test-results/

```

----------

# Open HTML Report

Open the report manually.

```bash
npx playwright show-report

```

Playwright starts a local server.

Example

```text
http://localhost:9323

```

The report opens automatically in the browser.

----------

# Opening a Custom Report

Suppose reports are stored elsewhere.

```bash
npx playwright show-report reports/html

```

Useful when different pipelines generate reports into different folders.

----------

# Configure Output Folder

```typescript
export default defineConfig({

    reporter: [

        [

            'html',

            {

                outputFolder:

                    'reports/html'

            }

        ]

    ]

});

```

Generated structure

```text
reports/

└── html/

```

----------

# Opening Automatically

During local development,

automatically open the report.

```typescript
reporter:[

['html',

{

open:'always'

}]

]

```

Options

Option

Meaning

always

Open every run

never

Never open

on-failure

Open only if tests fail

For CI/CD,

```text
never

```

is generally recommended.

----------

# HTML Report Overview

The dashboard typically displays:

```text
Execution Summary

↓

Passed

↓

Failed

↓

Skipped

↓

Duration

↓

Projects

↓

Browsers

```

Everything is available from one page.

----------

# Test Summary

Example

```text
150 Tests

↓

145 Passed

↓

4 Failed

↓

1 Skipped

```

Useful for quickly assessing overall execution health.

----------

# Test Navigation

The left panel displays

```text
Suite

↓

File

↓

Test

↓

Status

```

You can quickly navigate between tests.

----------

# Test Details

Selecting a test displays

```text
Test Name

↓

Duration

↓

Steps

↓

Errors

↓

Attachments

```

Everything related to that test is centralized.

----------

# Viewing Steps

Example

```text
Open Login

↓

Enter Username

↓

Enter Password

↓

Click Login

↓

Verify Dashboard

```

Each Playwright action appears as an execution step.

This makes debugging much easier.

----------

# Error Details

Failed tests display

```text
Expected

↓

Actual

↓

Stack Trace

↓

Source Location

```

Instead of only seeing

```text
Timeout

```

you see the complete failure context.

----------

# Stack Trace

Example

```text
LoginPage.ts

↓

line 45

↓

clickLogin()

```

You can jump directly to the failing code.

----------

# Screenshots

If configured,

failed tests include screenshots.

```text
Failure

↓

Screenshot

↓

Visual State

```

Useful for:

-   Missing elements
    
-   Wrong page
    
-   Broken UI
    
-   Layout issues
    

----------

# Configure Screenshots

```typescript
use:{

    screenshot:

        'only-on-failure'

}

```

Other options

```text
on

off

only-on-failure

```

----------

# Videos

Playwright can record videos.

```typescript
use:{

    video:

        'retain-on-failure'

}

```

Options

```text
on

off

retain-on-failure

on-first-retry

```

The report displays a video player.

----------

# Trace Viewer Integration

One of Playwright's strongest debugging features.

```typescript
use:{

trace:

'on-first-retry'

}

```

The HTML report provides

```text
Open Trace

```

Clicking it launches the Trace Viewer.

----------

# Trace Viewer

Trace Viewer provides

```text
Timeline

↓

DOM Snapshot

↓

Console Logs

↓

Network

↓

Source

↓

Actions

```

It allows you to replay the execution step-by-step.

----------

# Attachments

Playwright supports custom attachments.

Example

```typescript
import { test } from "@playwright/test";

test("API Test", async ({}, testInfo) => {

    await testInfo.attach("Response", {

        body: JSON.stringify({

            id: 10

        }),

        contentType: "application/json"

    });

});

```

The attachment appears in the HTML report.

----------

# Search

Large projects may contain

```text
5000 Tests

```

The HTML report allows searching by

-   Test name
    
-   File
    
-   Suite
    

making navigation much easier.

----------

# Filtering

Common filters include:

```text
Passed

Failed

Skipped

Flaky

Retried

```

You can quickly isolate failures.

----------

# Retry Information

Example

```text
Retry 1

↓

Failed

Retry 2

↓

Passed

```

Very useful for identifying flaky tests.

----------

# Project Information

When testing multiple browsers,

the report groups results by project.

Example

```text
Chromium

Firefox

WebKit

```

This makes browser-specific issues easy to identify.

----------

# Execution Time

Each test displays:

```text
Start Time

↓

Duration

↓

End Time

```

Useful for spotting slow tests.

----------

# Report Assets

Typical report structure

```text
playwright-report/

├── index.html

├── data/

├── trace/

├── assets/

└── screenshots/

```

The exact internal structure may vary slightly between Playwright versions, but these are the common report components.

----------

# Sharing Reports

Reports can be shared by:

-   Publishing CI artifacts
    
-   Hosting on an internal web server
    
-   Uploading to cloud storage
    
-   Attaching to release pipelines
    

Developers can open the report without rerunning the tests.

----------

# HTML Reports in CI/CD

Typical workflow

```text
Pipeline

↓

Execute Tests

↓

Generate HTML Report

↓

Upload Artifact

↓

Download Report

↓

Debug Failure

```

This is a standard practice in enterprise environments.

----------

# Enterprise Debugging Workflow

```text
Pipeline Failed

↓

Download HTML Report

↓

Open Failed Test

↓

View Screenshot

↓

Open Trace

↓

Watch Video

↓

Fix Bug

```

This workflow significantly reduces debugging time.

----------

# HTML Reporter Configuration

```typescript
export default defineConfig({

    reporter: [

        [

            'html',

            {

                outputFolder:

                    'playwright-report',

                open:

                    'on-failure'

            }

        ]

    ]

});

```

----------

# HTML Reporter vs Console

Console

HTML Reporter

Temporary

Persistent

Text only

Interactive UI

Limited information

Rich diagnostics

Difficult to navigate

Searchable

No screenshots

Screenshots supported

No traces

Trace integration

No videos

Video playback

----------

# Best Practices

-   Use the HTML reporter for all local debugging.
    
-   Configure screenshots and traces for failures rather than every test to reduce storage usage.
    
-   Combine the HTML reporter with JUnit or JSON reporters for CI/CD integrations.
    
-   Publish HTML reports as pipeline artifacts.
    
-   Retain reports long enough for debugging and trend analysis.
    

----------

# Common Mistakes

### ❌ Recording Everything

Recording screenshots, videos, and traces for every successful test can dramatically increase execution time and storage requirements.

Prefer:

```text
only-on-failure

retain-on-failure

on-first-retry

```

----------

### ❌ Not Publishing Reports

Generating reports without making them available to the team defeats their purpose.

Always publish them as CI/CD artifacts.

----------

### ❌ Ignoring Trace Viewer

Many engineers rely only on screenshots.

The Trace Viewer provides:

-   Timeline
    
-   DOM snapshots
    
-   Network
    
-   Console
    
-   Source
    
-   Action history
    

making it one of the most powerful debugging tools available.

----------

### ❌ Opening Reports in CI

Using

```typescript
open: "always"

```

in CI environments is unnecessary.

Use

```typescript
open: "never"

```

for automated pipelines.

----------

# Interview Questions

### Q1. What is the default Playwright HTML report folder?

```text
playwright-report/

```

----------

### Q2. Which command opens an existing HTML report?

```bash
npx playwright show-report

```

----------

### Q3. Can Playwright HTML reports include screenshots, videos, and traces?

Yes. When configured, the HTML report can display screenshots, video recordings, trace files, and custom attachments.

----------

### Q4. Which `open` options are available for the HTML reporter?

-   `always`
    
-   `never`
    
-   `on-failure`
    

----------

### Q5. Why is the HTML Reporter preferred for debugging?

Because it provides an interactive view of test execution with detailed steps, error messages, screenshots, videos, traces, retries, and attachments, making root-cause analysis significantly easier.

----------

# Summary

The HTML Reporter is Playwright's primary reporting solution for local development and failure analysis. It combines execution summaries, detailed test steps, screenshots, videos, traces, attachments, and search capabilities into a single interactive interface. When integrated with CI/CD pipelines and artifact publishing, it becomes an indispensable tool for debugging and maintaining enterprise-scale automation suites.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwMzYwMzY0NjFdfQ==
-->