A good automation engineer should understand **what every installation command does**, **what gets installed**, **where it gets installed**, and **why**.

This chapter will build that understanding.

----------

# Part 2 – Installation & Project Setup

# Chapter 2 – Installing Playwright

----------

# Introduction

Installing Playwright is one of the easiest parts of getting started with browser automation. However, behind a few simple commands, Playwright performs several important tasks:

-   Downloads the Playwright library
    
-   Installs browser binaries
    
-   Creates a project structure
    
-   Configures the test runner
    
-   Generates sample tests (optional)
    
-   Sets up configuration files
    

Understanding what happens during installation helps troubleshoot issues and gives you confidence when setting up new projects.

----------

# Installation Workflow

At a high level, installing Playwright follows this process:

```text
Create Project

↓

Initialize Node.js

↓

Install Playwright

↓

Download Browsers

↓

Create Configuration

↓

Run First Test

```

Each step prepares a different part of the development environment.

----------

# Step 1 – Create a Project Folder

Create a new directory for your project.

Example

```bash
mkdir playwright-demo

cd playwright-demo

```

At this point the folder is empty.

```text
playwright-demo/

(empty)

```

----------

# Step 2 – Initialize a Node.js Project

Run

```bash
npm init

```

or

```bash
npm init -y

```

This creates a `package.json` file.

Example

```json
{
  "name": "playwright-demo",
  "version": "1.0.0"
}

```

----------

# Why package.json?

Think of it as the identity card of your project.

It contains:

-   Project name
    
-   Version
    
-   Dependencies
    
-   Scripts
    
-   Metadata
    

Every Node.js project contains one.

----------

# Step 3 – Install Playwright

There are two common installation approaches.

## Option 1 – Recommended

```bash
npm init playwright@latest

```

This command:

-   Creates the project
    
-   Installs Playwright
    
-   Downloads browsers
    
-   Generates configuration
    
-   Creates sample tests
    

This is the easiest option for beginners.

----------

## Option 2 – Manual Installation

Install the Playwright Test package.

```bash
npm install -D @playwright/test

```

Then install the browser binaries.

```bash
npx playwright install

```

This approach gives more control and is common in existing Node.js projects.

----------

# Understanding `-D`

Notice:

```bash
npm install -D @playwright/test

```

`-D` means:

```text
Development Dependency

```

Playwright is required only during development and testing.

It is not needed when deploying your production application.

----------

# What Gets Installed?

After installation:

```text
Project

↓

node_modules

↓

@playwright/test

↓

Playwright Libraries

```

The package contains:

-   Test Runner
    
-   Assertions
    
-   Fixtures
    
-   API Testing
    
-   Reporters
    
-   Browser automation APIs
    

----------

# Browser Installation

Installing Playwright libraries does **not** automatically install browser binaries unless you use the initialization wizard.

Browser installation is performed using:

```bash
npx playwright install

```

This downloads:

```text
Chromium

↓

Firefox

↓

WebKit

```

These browsers are managed by Playwright.

----------

# Installing a Specific Browser

Instead of downloading every browser,

you can install only one.

Example

Chromium

```bash
npx playwright install chromium

```

Firefox

```bash
npx playwright install firefox

```

WebKit

```bash
npx playwright install webkit

```

Useful when storage or CI time is limited.

----------

# Installing Multiple Browsers

Example

```bash
npx playwright install chromium firefox

```

Only the specified browsers are downloaded.

----------

# Installing System Dependencies (Linux)

On Linux,

additional operating system libraries may be required.

Playwright provides:

```bash
npx playwright install-deps

```

or

```bash
npx playwright install --with-deps

```

These commands install required system packages for browser execution.

----------

# What Happens During Installation?

Internally,

Playwright performs several steps.

```text
Download npm Package

↓

Resolve Dependencies

↓

Install Libraries

↓

Download Browsers

↓

Verify Installation

```

Most of these steps are automatic.

----------

# Project Structure After Installation

A typical project looks like this:

```text
playwright-demo/

├── node_modules/

├── tests/

├── playwright.config.ts

├── package.json

├── package-lock.json

└── .gitignore

```

We'll examine each file in detail in the next chapter.

----------

# Installing with pnpm

Some teams prefer **pnpm**.

Install:

```bash
pnpm add -D @playwright/test

```

Then

```bash
pnpm exec playwright install

```

----------

# Installing with Yarn

Install

```bash
yarn add -D @playwright/test

```

Then

```bash
yarn playwright install

```

----------

# Installing with Bun

Playwright also supports Bun.

Install

```bash
bun add -d @playwright/test

```

Then

```bash
bunx playwright install

```

----------

# Comparing Package Managers

Package Manager

Install

Execute

npm

`npm install`

`npx`

pnpm

`pnpm add`

`pnpm exec`

Yarn

`yarn add`

`yarn`

Bun

`bun add`

`bunx`

The Playwright APIs remain the same regardless of the package manager.

----------

# Verifying Installation

Run

```bash
npx playwright --version

```

Example output

```text
Version 1.55.0

```

This confirms Playwright is installed correctly.

----------

# Running the First Test

Execute

```bash
npx playwright test

```

If the installation was successful,

Playwright will:

```text
Discover Tests

↓

Launch Browser

↓

Execute

↓

Generate Report

```

Even if you only have the sample tests, this validates your setup.

----------

# Viewing the HTML Report

After execution,

open the report.

```bash
npx playwright show-report

```

This launches the generated HTML report in your browser.

----------

# Updating Playwright

Upgrade to the latest version.

```bash
npm install -D @playwright/test@latest

```

Then install matching browsers.

```bash
npx playwright install

```

Always review release notes before upgrading production projects.

----------

# Uninstalling Playwright

Remove the package.

```bash
npm uninstall @playwright/test

```

If desired,

remove browser binaries.

```bash
npx playwright uninstall

```

This frees disk space and removes Playwright-managed browsers.

----------

# Offline Installation

In restricted enterprise environments:

-   Use internal npm registries.
    
-   Mirror browser binaries where organizational policies require it.
    
-   Coordinate with infrastructure teams for proxy and certificate configuration.
    

This allows Playwright to be used even when direct internet access is unavailable.

----------

# Common Installation Problems

## Node.js Not Found

```text
'node' is not recognized...

```

Cause:

Node.js is not installed or not added to the system PATH.

----------

## npm Permission Errors

Usually caused by:

-   Insufficient permissions
    
-   Corporate restrictions
    
-   Incorrect Node.js installation
    

Avoid running installation commands with elevated privileges unless required by your environment.

----------

## Browser Download Failure

Possible causes:

-   Proxy server
    
-   Firewall
    
-   SSL inspection
    
-   No internet access
    

Verify network configuration before retrying.

----------

## Unsupported Node Version

Playwright supports specific Node.js versions.

Always verify compatibility before upgrading Node.js.

----------

# Installation Checklist

```text
✓ Node.js Installed

✓ npm Working

✓ Playwright Installed

✓ Browsers Installed

✓ Version Verified

✓ First Test Passed

✓ HTML Report Opened

```

If every item is complete,

your environment is ready for development.

----------

# Best Practices

-   Use `npm init playwright@latest` for new projects unless you need a custom setup.
    
-   Install only the browsers required by your project or CI environment.
    
-   Keep Playwright and browser binaries synchronized.
    
-   Commit `package.json` and lock files to source control.
    
-   Validate the installation by running the sample tests before adding your own code.
    
-   Use the package manager that aligns with your organization's standards.
    

----------

# Interview Questions

### Q1. What is the difference between `npm init playwright@latest` and manually installing `@playwright/test`?

`npm init playwright@latest` scaffolds a complete Playwright project, including configuration, sample tests, and browser installation. Installing `@playwright/test` manually adds only the Playwright package, giving you more control over project setup.

----------

### Q2. Why is Playwright typically installed as a development dependency?

Because it is required for development and test execution, not for running the production application.

----------

### Q3. What does `npx playwright install` do?

It downloads and installs the browser binaries (Chromium, Firefox, and WebKit) that Playwright uses for automation.

----------

### Q4. Can Playwright install only specific browsers?

Yes. You can install individual browsers, such as `chromium`, `firefox`, or `webkit`, instead of downloading all supported browsers.

----------

### Q5. How can you verify that Playwright was installed successfully?

Run `npx playwright --version` to verify the installation and `npx playwright test` to execute the sample tests and confirm that the environment is working correctly.

----------

# Summary

Installing Playwright involves more than adding a package to a project. It includes setting up the Node.js environment, installing Playwright libraries, downloading compatible browser binaries, generating a project structure, and validating the installation. By understanding what each command does and how the installation process works, you'll be better prepared to troubleshoot issues and build production-ready automation projects.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwNjM0MDU2OThdfQ==
-->