# Part 2 – Installation & Project Setup

## Chapter 1 – Prerequisites

----------

# Introduction

Before writing the first Playwright test, it is important to prepare the development environment correctly.

A Playwright project depends on several technologies working together. Understanding each one helps avoid common installation issues and makes troubleshooting much easier.

By the end of this chapter, you'll understand:

-   Why Playwright requires Node.js
    
-   What npm and npx are
    
-   Why TypeScript is recommended
    
-   Which IDEs are best suited for Playwright
    
-   Required browser dependencies
    
-   Operating system considerations
    
-   Recommended software versions
    

Think of this chapter as building the **foundation** upon which the rest of the Playwright framework will stand.

----------

# Development Environment Overview

A typical Playwright development environment looks like this:

```text
Developer

↓

IDE (VS Code)

↓

Node.js

↓

npm / npx

↓

Playwright

↓

Browser

↓

Application

```

Every layer has a specific responsibility.

----------

# Required Software

Before installing Playwright, ensure the following software is available.

Software

Required

Purpose

Node.js

✅ Yes

Runtime environment

npm

✅ Yes

Package management

Playwright

✅ Yes

Browser automation

VS Code

⭐ Recommended

Development IDE

Git

⭐ Recommended

Version control

TypeScript

⭐ Recommended

Type-safe automation

----------

# Understanding Node.js

One of the first questions beginners ask is:

> **"Why do I need Node.js when Playwright is a testing framework?"**

The answer is simple.

Playwright itself is a **Node.js package**.

Without Node.js:

-   JavaScript cannot execute outside the browser.
    
-   TypeScript cannot be compiled.
    
-   npm cannot install packages.
    
-   Playwright cannot run.
    

----------

# What is Node.js?

Node.js is a JavaScript runtime built on Google's V8 JavaScript engine.

Traditionally,

JavaScript executed only inside browsers.

```text
JavaScript

↓

Browser

```

Node.js changed that.

```text
JavaScript

↓

Node.js

↓

Operating System

```

Now JavaScript can:

-   Read files
    
-   Create servers
    
-   Execute scripts
    
-   Install packages
    
-   Run Playwright
    

----------

# Why Playwright Uses Node.js

Playwright is implemented as a Node.js library.

When you execute:

```bash
npx playwright test

```

Node.js:

-   Loads Playwright
    
-   Executes your test
    
-   Launches browsers
    
-   Generates reports
    

Without Node.js, none of this is possible.

----------

# Understanding npm

npm stands for:

> **Node Package Manager**

It is responsible for:

-   Installing packages
    
-   Updating packages
    
-   Removing packages
    
-   Managing dependencies
    

Example:

```bash
npm install

```

This installs every dependency listed in your project.

----------

# Understanding npx

Many beginners confuse **npm** and **npx**.

They serve different purposes.

### npm

Installs packages.

Example

```bash
npm install @playwright/test

```

----------

### npx

Executes packages.

Example

```bash
npx playwright test

```

Think of it this way:

```text
npm

↓

Install

------------------

npx

↓

Run

```

----------

# Node.js LTS vs Current

Node.js provides two release channels.

Version

Recommended?

Purpose

LTS (Long-Term Support)

✅ Yes

Production projects

Current

Sometimes

Latest features

For enterprise automation projects,

always prefer **LTS** versions.

Benefits:

-   Stable
    
-   Better ecosystem compatibility
    
-   Longer support lifecycle
    

----------

# Checking Node.js Installation

Verify Node.js:

```bash
node --version

```

Example output:

```text
v22.18.0

```

Verify npm:

```bash
npm --version

```

Example:

```text
10.9.0

```

Both commands should execute successfully.

----------

# Why TypeScript?

Although Playwright supports JavaScript,

Microsoft recommends **TypeScript**.

Advantages include:

-   Static typing
    
-   Better IntelliSense
    
-   Early error detection
    
-   Easier refactoring
    
-   Improved maintainability
    

Example

JavaScript

```javascript
const username = 123;

```

TypeScript

```typescript
const username: string = "admin";

```

TypeScript catches type errors during development instead of at runtime.

----------

# Do You Need TypeScript?

Technically,

No.

Practically,

Almost every enterprise Playwright project uses TypeScript because of:

-   Better IDE support
    
-   Safer refactoring
    
-   Cleaner architecture
    
-   Easier maintenance
    

Throughout this handbook, TypeScript will be the primary language.

Where useful, Java comparisons will also be provided.

----------

# Choosing an IDE

Any editor can write Playwright code,

but some provide a much better experience.

### Recommended

-   Visual Studio Code
    

Also supported:

-   IntelliJ IDEA
    
-   WebStorm
    
-   Visual Studio
    
-   Cursor
    
-   Windsurf
    

VS Code remains the reference IDE because it offers first-party Playwright support.

----------

# Why VS Code?

The Playwright VS Code extension provides:

-   Test Explorer
    
-   One-click execution
    
-   Debugging
    
-   Code Generation
    
-   Trace Viewer integration
    
-   IntelliSense
    
-   Auto-completion
    

These features significantly improve developer productivity.

----------

# Git

Every Playwright project should use Git.

Benefits:

-   Version control
    
-   Collaboration
    
-   Branching
    
-   Pull requests
    
-   CI/CD integration
    

A typical workflow:

```text
Developer

↓

Git

↓

Repository

↓

Pipeline

```

----------

# Supported Operating Systems

Playwright officially supports:

-   Windows
    
-   macOS
    
-   Linux
    

All examples in this book work across these platforms unless noted otherwise.

----------

# Browser Requirements

Playwright manages compatible browser binaries automatically.

Supported engines include:

-   Chromium
    
-   Firefox
    
-   WebKit
    

You do **not** need to manually download browser drivers.

----------

# Internet Connection

An internet connection is typically required during the initial setup to:

-   Download Playwright packages
    
-   Download browser binaries
    
-   Install dependencies
    

After installation, many tasks can be performed offline.

----------

# Corporate Networks

Enterprise environments may introduce additional challenges such as:

-   Proxy servers
    
-   Internal package registries
    
-   SSL inspection
    
-   Restricted internet access
    

If downloads fail, coordinate with your IT team to configure npm or your package manager appropriately.

----------

# Minimum Hardware Recommendations

For learning:

-   8 GB RAM
    
-   Dual-core CPU
    

For enterprise automation:

-   16 GB+ RAM
    
-   SSD storage
    
-   Multi-core CPU
    

Parallel execution benefits from additional CPU cores.

----------

# Recommended Software Stack

Component

Recommendation

Node.js

Latest LTS

Package Manager

npm (or pnpm if preferred)

IDE

VS Code

Language

TypeScript

Version Control

Git

Browser

Playwright-managed browsers

----------

# Environment Validation Checklist

Before installing Playwright, verify:

```text
✓ Node.js Installed

✓ npm Available

✓ Git Installed

✓ VS Code Installed

✓ Internet Connection

✓ Sufficient Disk Space

✓ Supported Operating System

```

----------

# Common Beginner Mistakes

### ❌ Installing the Current Node.js Version for Production

Use the latest **LTS** release unless you have a specific reason not to.

----------

### ❌ Confusing npm with npx

Remember:

```text
npm → Install

npx → Execute

```

----------

### ❌ Skipping TypeScript

While JavaScript works, TypeScript provides significant long-term benefits for Playwright projects.

----------

### ❌ Installing Browser Drivers Manually

Unlike Selenium, Playwright manages compatible browser binaries for you.

----------

### ❌ Ignoring IDE Features

A properly configured IDE with the Playwright extension can save considerable development time.

----------

# Best Practices

-   Use the latest supported LTS version of Node.js.
    
-   Prefer TypeScript for new Playwright projects.
    
-   Use VS Code with the official Playwright extension.
    
-   Keep Git enabled from the beginning of the project.
    
-   Let Playwright manage browser binaries.
    
-   Verify your environment before installing dependencies.
    

----------

# Interview Questions

### Q1. Why does Playwright require Node.js?

Because Playwright is distributed as a Node.js package. Node.js provides the runtime required to execute Playwright, manage dependencies, and run automation scripts.

----------

### Q2. What is the difference between `npm` and `npx`?

`npm` installs and manages packages, while `npx` executes packages without requiring a global installation.

----------

### Q3. Why is TypeScript recommended for Playwright?

TypeScript provides static typing, better IntelliSense, compile-time error detection, and easier maintenance, making it well suited for enterprise automation projects.

----------

### Q4. Do you need to install browser drivers manually for Playwright?

No. Playwright downloads and manages compatible browser binaries automatically.

----------

### Q5. Why is the LTS version of Node.js recommended?

LTS releases provide long-term stability, better compatibility with libraries, and are better suited for production and enterprise environments.

----------

# Summary

A successful Playwright project starts with a well-prepared development environment. Node.js provides the runtime, npm manages dependencies, npx executes Playwright commands, TypeScript improves code quality, and VS Code enhances developer productivity. By understanding the role of each prerequisite rather than simply installing software, you'll build a stronger foundation for the rest of your Playwright journey.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjA5NzQ3MzU5Nl19
-->