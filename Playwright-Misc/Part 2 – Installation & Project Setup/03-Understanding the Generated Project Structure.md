For enterprise engineers, understanding the project structure is essential because eventually you'll customize almost every file.

----------

# Part 2 – Installation & Project Setup

# Chapter 3 – Understanding the Generated Project Structure

----------

# Introduction

After installing Playwright using:

```bash
npm init playwright@latest

```

Playwright generates a project containing several files and directories.

At first glance, the structure may seem overwhelming.

```text
playwright-demo/

├── node_modules/
├── tests/
├── playwright.config.ts
├── package.json
├── package-lock.json
├── .gitignore
└── ...

```

Every file serves a specific purpose.

Understanding these files early makes framework customization much easier later.

----------

# The Complete Project Structure

A typical Playwright project looks like this.

```text
playwright-demo/

├── node_modules/
│
├── tests/
│
├── playwright.config.ts
│
├── package.json
│
├── package-lock.json
│
├── .gitignore
│
├── playwright-report/
│
├── test-results/
│
└── README.md

```

As the framework grows, you'll add more folders such as:

```text
pages/

fixtures/

utils/

services/

data/

reports/

```

----------

# Understanding the Root Directory

The project root contains everything required for the automation framework.

```text
Project Root

↓

Configuration

↓

Source Code

↓

Dependencies

↓

Reports

```

Think of it as the home directory of your automation project.

----------

# node_modules

One of the largest folders.

```text
node_modules/

↓

Playwright

↓

TypeScript

↓

Libraries

↓

Dependencies

```

This folder contains every package installed through npm.

Examples include:

-   Playwright
    
-   TypeScript
    
-   Supporting libraries
    

----------

## Should You Modify node_modules?

No.

This folder is managed automatically by npm.

Never edit files inside it.

If something is wrong,

reinstall dependencies instead.

----------

## Should node_modules Be Committed?

No.

Reasons:

-   Very large
    
-   Can be recreated
    
-   Machine independent
    

Instead,

commit:

```text
package.json

package-lock.json

```

Then execute

```bash
npm install

```

on another machine.

----------

# tests Folder

Example

```text
tests/

├── login.spec.ts

├── cart.spec.ts

└── checkout.spec.ts

```

This is where Playwright discovers test files.

By default,

files ending with:

```text
.spec.ts

.test.ts

```

are executed.

----------

# Why Keep Tests Separate?

Separating test files improves:

-   Readability
    
-   Organization
    
-   Scalability
    

A large project may eventually contain hundreds or thousands of test files.

----------

# playwright.config.ts

This is one of the most important files.

It controls:

-   Browser selection
    
-   Timeouts
    
-   Reporters
    
-   Retries
    
-   Projects
    
-   Parallel execution
    
-   Base URL
    
-   Screenshots
    
-   Videos
    
-   Tracing
    

Architecture

```text
Tests

↓

playwright.config.ts

↓

Execution

```

Almost every framework customization starts here.

----------

# Why Only One Configuration File?

Instead of scattering settings,

Playwright centralizes configuration.

Benefits:

-   Easier maintenance
    
-   Consistent execution
    
-   Single source of truth
    

----------

# package.json

Every Node project contains:

```text
package.json

```

Example

```json
{
  "name": "playwright-demo",
  "scripts": {
    "test": "playwright test"
  }
}

```

It defines:

-   Project information
    
-   Dependencies
    
-   Scripts
    
-   Metadata
    

----------

# Dependencies

Inside package.json

```json
"devDependencies": {
  "@playwright/test": "^1.55.0"
}

```

This tells npm which packages the project requires.

----------

# Scripts

Instead of typing

```bash
npx playwright test

```

every time,

create scripts.

Example

```json
"scripts": {

"test":"playwright test",

"headed":"playwright test --headed",

"debug":"playwright test --debug"

}

```

Then execute

```bash
npm run test

```

Much easier.

----------

# package-lock.json

Many beginners ignore this file.

It records:

-   Exact package versions
    
-   Dependency tree
    
-   Resolution information
    

This guarantees:

```text
Developer A

↓

Install

↓

Same Versions

----------------

Developer B

↓

Install

↓

Same Versions

```

without unexpected differences.

----------

# Should package-lock.json Be Committed?

Yes.

Always.

It improves:

-   Build consistency
    
-   CI reliability
    
-   Team collaboration
    

----------

# .gitignore

This file tells Git

which files should NOT be committed.

Example

```text
node_modules/

playwright-report/

test-results/

```

These folders are generated automatically.

----------

# Why Ignore Generated Files?

Generated folders can become very large.

They:

-   Change frequently
    
-   Are machine-specific
    
-   Can be regenerated
    

Keeping them out of source control keeps repositories clean.

----------

# playwright-report

After execution,

Playwright creates

```text
playwright-report/

```

Contents:

-   HTML report
    
-   Assets
    
-   Screenshots
    
-   Navigation
    

Open using

```bash
npx playwright show-report

```

----------

# Should It Be Committed?

Usually,

No.

Reports are generated artifacts.

CI pipelines often publish them separately.

----------

# test-results

Example

```text
test-results/

↓

Failed Tests

↓

Screenshots

↓

Videos

↓

Trace Files

```

Useful for debugging.

Not intended for source control.

----------

# README.md

Most generated projects include:

```text
README.md

```

Document:

-   Project setup
    
-   Execution commands
    
-   Framework overview
    
-   Team conventions
    

Good documentation reduces onboarding time.

----------

# Future Project Structure

A production framework might evolve into:

```text
project/

├── tests/

├── pages/

├── components/

├── fixtures/

├── services/

├── data/

├── utils/

├── config/

├── reports/

├── playwright.config.ts

└── package.json

```

We'll build this gradually throughout the book.

----------

# File Relationships

```text
package.json

↓

Install Dependencies

↓

node_modules

↓

Playwright

↓

Tests

↓

Reports

```

Everything begins with package.json.

----------

# Source-Control Strategy

## Commit

```text
✓ package.json

✓ package-lock.json

✓ tests/

✓ pages/

✓ config/

✓ fixtures/

```

----------

## Do Not Commit

```text
✗ node_modules/

✗ playwright-report/

✗ test-results/

✗ Temporary Files

```

----------

# Project Lifecycle

```text
Clone Repository

↓

npm install

↓

Playwright Installed

↓

Run Tests

↓

Generate Reports

```

This is the typical onboarding flow for a new team member.

----------

# Common Mistakes

## ❌ Editing node_modules

Never modify installed libraries directly.

----------

## ❌ Deleting package-lock.json

Unless there is a specific reason, keep it under source control.

----------

## ❌ Committing Reports

Reports are generated artifacts.

Store them in CI, not Git.

----------

## ❌ Putting Everything Inside tests

As projects grow,

separate:

-   Pages
    
-   Components
    
-   Fixtures
    
-   Utilities
    
-   Services
    
-   Test Data
    

into dedicated folders.

----------

## ❌ Ignoring README

A good README saves hours of onboarding effort.

Document:

-   Installation
    
-   Commands
    
-   Framework architecture
    
-   Team conventions
    

----------

# Enterprise Best Practices

-   Keep the project root clean and organized.
    
-   Use dedicated folders for pages, components, fixtures, services, and utilities as the framework grows.
    
-   Commit configuration and source files, but ignore generated artifacts.
    
-   Treat `playwright.config.ts` as the central configuration point.
    
-   Use npm scripts to standardize common commands.
    
-   Document project setup and execution steps in the `README.md`.
    

----------

# Interview Questions

### Q1. What is the purpose of `playwright.config.ts`?

It is the central configuration file that controls Playwright's execution settings, including browsers, projects, reporters, retries, timeouts, tracing, screenshots, and parallel execution.

----------

### Q2. Why should `node_modules` not be committed to Git?

Because it contains generated dependencies that can be recreated from `package.json` and the lock file, making it unnecessary and inefficient to store in source control.

----------

### Q3. What is the difference between `package.json` and `package-lock.json`?

`package.json` defines the project's dependencies and scripts, while `package-lock.json` records the exact resolved versions of all dependencies to ensure consistent installations across environments.

----------

### Q4. What is the purpose of the `tests` directory?

The `tests` directory contains Playwright test files. The test runner discovers and executes files that match the configured naming patterns, such as `.spec.ts` or `.test.ts`.

----------

### Q5. Should `playwright-report` and `test-results` be committed?

No. They are generated artifacts created during test execution and should typically be excluded from source control using `.gitignore`.

----------

# Summary

A Playwright project is more than a collection of test files—it is a structured Node.js application with clearly defined responsibilities for configuration, dependencies, source code, reports, and generated artifacts. Understanding the purpose of each file and directory helps engineers maintain cleaner repositories, onboard new team members more efficiently, and build automation frameworks that scale from small projects to large enterprise solutions.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyOTcxMzY0MTddfQ==
-->