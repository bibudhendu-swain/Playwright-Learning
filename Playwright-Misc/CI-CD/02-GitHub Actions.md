This chapter is one of the most practical in the CI/CD section because **GitHub Actions** has become the default CI/CD platform for many Playwright projects.

----------

# Part 20 – CI/CD

# Chapter 2 – GitHub Actions

----------

# Introduction

GitHub Actions is GitHub's built-in Continuous Integration and Continuous Delivery (CI/CD) platform.

It enables you to automatically:

-   Build applications
    
-   Run Playwright tests
    
-   Generate reports
    
-   Upload artifacts
    
-   Deploy applications
    
-   Notify teams
    

Every time code is pushed to GitHub, GitHub Actions can automatically execute your Playwright test suite.

----------

# GitHub Actions Workflow

A typical workflow looks like this:

```text
Developer Push

↓

GitHub Repository

↓

Workflow Triggered

↓

GitHub Runner

↓

Install Dependencies

↓

Run Playwright Tests

↓

Generate Reports

↓

Upload Reports

```

----------

# GitHub Actions Terminology

Term

Description

Workflow

Complete automation pipeline

Event

Trigger for a workflow

Job

Collection of steps

Step

Individual command/task

Runner

Machine executing the workflow

Action

Reusable automation component

Artifact

Files generated during execution

----------

# Workflow File Location

GitHub Actions workflows are stored in:

```text
.github/

└── workflows/

    └── playwright.yml

```

Every workflow is a YAML file.

----------

# Basic Workflow Structure

```yaml
name: Playwright Tests

on:
  push:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Source
        uses: actions/checkout@v4

```

A workflow consists of:

```text
Workflow

↓

Trigger

↓

Jobs

↓

Steps

```

----------

# Workflow Triggers

GitHub Actions supports multiple triggers.

```yaml
on:
  push:

  pull_request:

  workflow_dispatch:

  schedule:

```

Meaning

Trigger

Description

push

Run after code push

pull_request

Run for PRs

workflow_dispatch

Manual execution

schedule

Run on a schedule

----------

# Manual Execution

Allow manual execution.

```yaml
on:
  workflow_dispatch:

```

GitHub displays a **Run Workflow** button.

Useful for:

-   Smoke testing
    
-   Regression
    
-   Production validation
    

----------

# Scheduled Execution

Nightly regression

```yaml
on:
  schedule:

    - cron: "0 2 * * *"

```

Meaning

```text
Every Day

↓

02:00 UTC

↓

Run Regression

```

----------

# GitHub Runner

Every workflow executes on a runner.

```yaml
runs-on: ubuntu-latest

```

Common runners

Runner

Usage

ubuntu-latest

Recommended

windows-latest

Windows testing

macos-latest

Safari/WebKit

----------

# Checking Out Code

Every workflow needs project source code.

```yaml
- name: Checkout Repository

  uses: actions/checkout@v4

```

----------

# Installing Node.js

```yaml
- name: Setup Node

  uses: actions/setup-node@v4

  with:

    node-version: 20

```

Always use an LTS version unless your project requires something else.

----------

# Installing Dependencies

```yaml
- name: Install Dependencies

  run: npm ci

```

Why `npm ci`?

-   Faster
    
-   Deterministic
    
-   Uses `package-lock.json`
    
-   Recommended for CI
    

----------

# Installing Playwright Browsers

```yaml
- name: Install Browsers

  run: npx playwright install --with-deps

```

On Linux,

this installs both browsers and required system dependencies.

----------

# Running Playwright Tests

```yaml
- name: Run Tests

  run: npx playwright test

```

Simple.

Every configured Playwright project executes.

----------

# Complete Minimal Workflow

```yaml
name: Playwright

on:

  push:

jobs:

  test:

    runs-on: ubuntu-latest

    steps:

      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4

        with:

          node-version: 20

      - run: npm ci

      - run: npx playwright install --with-deps

      - run: npx playwright test

```

----------

# Upload HTML Report

Reports should be published as artifacts.

```yaml
- name: Upload Report

  uses: actions/upload-artifact@v4

  with:

    name: playwright-report

    path: playwright-report/

```

After execution,

developers can download the report directly from GitHub.

----------

# Upload Test Results

```yaml
- name: Upload Results

  uses: actions/upload-artifact@v4

  with:

    name: test-results

    path: test-results/

```

Useful for:

-   Screenshots
    
-   Videos
    
-   Traces
    

----------

# Environment Variables

```yaml
env:

  BASE_URL: https://qa.company.com

```

Access inside Playwright

```typescript
process.env.BASE_URL

```

----------

# Using GitHub Secrets

Never hardcode credentials.

GitHub

```text
Repository

↓

Settings

↓

Secrets

↓

Actions

↓

Add Secret

```

Workflow

```yaml
env:

  USERNAME: ${{ secrets.USERNAME }}

  PASSWORD: ${{ secrets.PASSWORD }}

```

----------

# Matrix Builds

Test multiple browsers simultaneously.

```yaml
strategy:

  matrix:

    browser:

      - chromium

      - firefox

      - webkit

```

Execution

```text
Chromium

↓

Firefox

↓

WebKit

```

Runs in parallel.

----------

# Passing Matrix Value

```yaml
- name: Run Tests

  run: npx playwright test --project=${{ matrix.browser }}

```

One workflow,

multiple browsers.

----------

# Multiple Node Versions

```yaml
strategy:

  matrix:

    node:

      - 18

      - 20

```

Useful for library compatibility testing.

----------

# Caching Dependencies

Avoid downloading dependencies every run.

```yaml
- uses: actions/setup-node@v4

  with:

    node-version: 20

    cache: npm

```

Benefits:

-   Faster pipeline
    
-   Reduced network usage
    

----------

# Parallel Jobs

Instead of one job,

execute multiple.

```text
Build

↓

API Tests

↓

UI Tests

↓

Security Scan

```

Independent jobs run simultaneously.

----------

# Job Dependencies

Example

```yaml
jobs:

  build:

  test:

    needs: build

```

Workflow

```text
Build

↓

Test

```

----------

# Conditional Execution

Example

```yaml
if: github.ref == 'refs/heads/main'

```

Only execute on

```text
main

```

branch.

----------

# Continue on Failure

Sometimes you still want reports.

```yaml
- name: Run Tests

  run: npx playwright test

  continue-on-error: true

```

The workflow continues,

allowing reports to be uploaded.

----------

# Enterprise Workflow

```text
Checkout

↓

Install

↓

Cache

↓

Install Browsers

↓

Run Smoke

↓

Run Regression

↓

Generate HTML

↓

Upload Reports

↓

Notify Slack

```

----------

# Sample Enterprise Workflow

```yaml
name: Enterprise Playwright

on:

  pull_request:

  workflow_dispatch:

jobs:

  playwright:

    runs-on: ubuntu-latest

    steps:

      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4

      - run: npm ci

      - run: npx playwright install --with-deps

      - run: npx playwright test

      - uses: actions/upload-artifact@v4

        with:

          name: HTML Report

          path: playwright-report/

```

----------

# Recommended Folder Structure

```text
.github/

└── workflows/

    ├── smoke.yml

    ├── regression.yml

    ├── nightly.yml

    └── release.yml

```

Separate workflows are easier to maintain than one very large pipeline.

----------

# GitHub Actions Best Practices

-   Use `npm ci` instead of `npm install`.
    
-   Cache Node dependencies to reduce execution time.
    
-   Store secrets in GitHub Secrets.
    
-   Upload reports and test artifacts on every run.
    
-   Use matrix builds for browser coverage.
    
-   Split large workflows into smaller, focused workflows.
    
-   Use `continue-on-error` only when you intentionally want later steps (such as report upload) to run after test failures.
    

----------

# Common Mistakes

### ❌ Using `npm install`

Prefer

```bash
npm ci

```

for faster and deterministic builds.

----------

### ❌ Hardcoding Credentials

Never write

```yaml
PASSWORD: admin123

```

Use GitHub Secrets.

----------

### ❌ Not Uploading Reports

Failed pipelines without reports are difficult to debug.

Always upload:

-   HTML Reports
    
-   Screenshots
    
-   Videos
    
-   Traces
    

----------

### ❌ One Massive Workflow

Instead of

```text
1000-Line Workflow

```

Create

```text
Smoke

Regression

Nightly

Release

```

Separate workflows.

----------

### ❌ No Caching

Without caching,

every execution downloads packages again.

----------

# Interview Questions

### Q1. Why is `npm ci` preferred over `npm install` in CI?

`npm ci` performs a clean, reproducible installation based on `package-lock.json`, making builds faster and more reliable.

----------

### Q2. Why should GitHub Secrets be used?

To securely store sensitive information such as credentials, API keys, and tokens without exposing them in the repository.

----------

### Q3. What is a matrix build?

A matrix build runs the same workflow against multiple configurations (such as browsers, operating systems, or Node.js versions) in parallel.

----------

### Q4. Why should reports be uploaded as artifacts?

Artifacts allow developers to download reports, screenshots, traces, and videos after the workflow completes, making failure analysis much easier.

----------

### Q5. What is the purpose of `workflow_dispatch`?

It enables manual execution of a workflow directly from the GitHub Actions interface.

----------

# Summary

GitHub Actions provides a powerful and flexible platform for running Playwright tests as part of an automated CI/CD pipeline. By combining workflow triggers, runners, caching, matrix builds, secure secret management, artifact publishing, and parallel execution, teams can build reliable, scalable, and maintainable automation pipelines that deliver rapid feedback on every code change.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQwMTM2OTk2XX0=
-->