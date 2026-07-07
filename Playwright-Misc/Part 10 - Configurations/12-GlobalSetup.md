Excellent. This is one of the **most important topics in modern Playwright** and one that many experienced automation engineers still misunderstand.

> **Interview Tip:** If someone asks, _"How do you perform login once and reuse it across all tests?"_, don't immediately answer **globalSetup**. Since Playwright v1.31+, the **recommended approach is Setup Projects with Project Dependencies**.

----------

# Part 12 – Global Setup, Global Teardown & Project Dependencies

We'll cover:

-   `globalSetup`
    
-   `globalTeardown`
    
-   Setup Projects (Recommended)
    
-   Project Dependencies
    
-   Project Teardown
    
-   Authentication (`storageState`)
    
-   Database Setup
    
-   API Setup
    
-   Environment Initialization
    
-   Best Practices
    

----------

# The Evolution of Playwright Setup

Historically:

```text
globalSetup()

↓

All Tests

↓

globalTeardown()

```

Modern Playwright:

```text
Setup Project

↓

Browser Projects

↓

Teardown Project

```

The **Setup Project** is now the preferred approach because it integrates naturally with Playwright's reporting, retries, traces, and dependencies.

----------

# Old Approach – `globalSetup`

## Purpose

Runs **once before** the test suite.

Configuration:

```ts
export default defineConfig({

    globalSetup: './global-setup.ts'

});

```

----------

Example

```ts
// global-setup.ts

import { chromium } from '@playwright/test';

export default async () => {

    const browser = await chromium.launch();

    const page = await browser.newPage();

    await page.goto('https://company.com');

    console.log('Setup Complete');

    await browser.close();

};

```

Execution:

```text
Global Setup

↓

All Tests

```

----------

# Typical Uses

Historically used for:

-   Login
    
-   Seed database
    
-   Create users
    
-   Generate tokens
    
-   Environment validation
    

----------

# Limitations of `globalSetup`

This is why Playwright recommends Setup Projects instead.

### ❌ Not a Playwright Test

No:

-   Trace
    
-   Screenshot
    
-   Video
    
-   HTML Report
    
-   Retry
    

----------

### ❌ Doesn't Appear in Reports

HTML report:

```text
Login Test

Checkout Test

```

Global setup is invisible.

----------

### ❌ Failure Diagnostics

If setup fails:

```text
Database Connection Failed

```

You don't get the rich debugging experience available for regular tests.

----------

# `globalTeardown`

Runs after all tests.

Configuration

```ts
globalTeardown:'./global-teardown.ts'

```

----------

Example

```ts
export default async ()=>{

console.log('Cleanup');

};

```

Typical usage:

-   Delete users
    
-   Reset database
    
-   Remove temp files
    
-   Cleanup cloud resources
    

----------

# Modern Approach – Setup Project

Instead of:

```text
globalSetup

```

Create:

```text
tests/

auth.setup.ts

```

----------

Example

```ts
import { test } from '@playwright/test';

test('Authenticate', async ({ page }) => {

    await page.goto('/login');

    await page.getByLabel('Username').fill('admin');

    await page.getByLabel('Password').fill('password');

    await page.getByRole('button', { name: 'Login' }).click();

    await page.context().storageState({

        path:'playwright/.auth/user.json'

    });

});

```

Notice:

This is a **normal Playwright test**.

----------

# Configure Setup Project

```ts
projects:[

{

name:'Setup',

testMatch:/auth.setup.ts/

}

]

```

----------

Then:

```ts
{

name:'Chrome',

dependencies:['Setup']

}

```

Execution:

```text
Setup

↓

Chrome

↓

Firefox

↓

WebKit

```

----------

# Why Setup Projects Are Better

Because the setup itself is a Playwright test.

Therefore it supports:

✅ Trace

✅ Screenshot

✅ Video

✅ Retry

✅ HTML Report

✅ Fixtures

Everything works.

----------

# Authentication Example

Setup Project

```ts
test('Authenticate', async ({ page }) => {

    await page.goto('/login');

    // Login

    await page.context().storageState({

        path:'playwright/.auth/admin.json'

    });

});

```

Chrome Project

```ts
{

name:'Chrome',

use:{

storageState:'playwright/.auth/admin.json'

},

dependencies:['Setup']

}

```

Execution

```text
Login

↓

Save Cookies

↓

All Tests Already Logged In

```

----------

# Multiple Users

Setup

```text
Generate

admin.json

customer.json

manager.json

```

Projects

```text
Admin

Customer

Manager

```

No repeated login.

----------

# API Login Instead of UI Login

Instead of:

```text
Open Browser

↓

Login Page

↓

Click Login

```

Use API:

```ts
import { request, test } from '@playwright/test';

test('Setup', async () => {

    const api = await request.newContext();

    await api.post('/login', {

        data: {

            username:'admin',

            password:'password'

        }

    });

});

```

Much faster.

Many enterprise frameworks use API authentication to generate storage state.

----------

# Project Dependencies

Configuration

```ts
projects:[

{

name:'Setup'

},

{

name:'Chrome',

dependencies:['Setup']

},

{

name:'Firefox',

dependencies:['Setup']

}

]

```

Execution

```text
Setup

↓

Chrome

↓

Firefox

```

Setup runs once.

----------

# Multiple Dependencies

Example

```text
Database Setup

↓

Authentication

↓

Tests

```

Configuration

```ts
{

name:'Database'

},

{

name:'Authentication',

dependencies:['Database']

},

{

name:'Chrome',

dependencies:['Authentication']

}

```

Execution order:

```text
Database

↓

Authentication

↓

Chrome

```

----------

# Project Teardown

Newer Playwright versions support teardown projects.

Example

```text
Setup

↓

Tests

↓

Cleanup

```

Configuration

```ts
{

name:'Setup',

teardown:'Cleanup'

},

{

name:'Cleanup'

}

```

----------

Cleanup Test

```ts
test('Cleanup', async ()=>{

// delete users

// reset database

});

```

----------

# Real Enterprise Flow

```text
Database

↓

Create Test Data

↓

Login

↓

Generate Storage State

↓

Chrome Tests

↓

Firefox Tests

↓

Edge Tests

↓

Cleanup

```

Everything becomes visible in the HTML report.

----------

# Setup vs GlobalSetup

Setup Project

globalSetup

Playwright Test

Plain Function

HTML Report

❌

Trace

❌

Retry

❌

Video

❌

Screenshot

❌

Fixtures

✅

Recommended

✅

----------

# Authentication Folder

Recommended structure

```text
playwright/

.auth/

admin.json

customer.json

manager.json

```

Usually ignored in Git:

```text
playwright/.auth/

```

----------

# Enterprise Example

```ts
projects:[

{

name:'Setup',

testMatch:/auth.setup.ts/

},

{

name:'Chrome',

dependencies:['Setup'],

use:{

storageState:'playwright/.auth/admin.json'

}

},

{

name:'Firefox',

dependencies:['Setup'],

use:{

storageState:'playwright/.auth/admin.json'

}

}

]

```

Very common.

----------

# Common Mistakes

## ❌ Logging in Every Test

```ts
test('Login', async()=>{

// login

});

```

Every test.

Slow.

----------

Better

Setup Project

↓

storageState

----------

## ❌ Using `globalSetup` for Authentication

Still works.

But Playwright recommends:

Setup Projects.

----------

## ❌ Committing Authentication Files

Never commit

```text
admin.json

```

Usually ignored.

----------

## ❌ UI Login When API Exists

If authentication API is available:

Use it.

It's much faster.

----------

# Interview Questions

### Q1. What is the difference between `globalSetup` and a Setup Project?

globalSetup

Setup Project

Runs as a plain function before the suite

Runs as a Playwright test

Not shown in reports

Appears in reports

No retry, trace, video, or screenshot support

Supports retries, traces, videos, screenshots, fixtures

Legacy approach

Recommended approach

----------

### Q2. How do you log in once for all tests?

Recommended flow:

1.  Create a **Setup Project**.
    
2.  Log in (preferably through an API if available).
    
3.  Save the browser state using `storageState`.
    
4.  Configure browser projects to use the saved `storageState`.
    

----------

### Q3. Can multiple projects depend on one setup project?

Yes.

Example:

```text
Setup

↓

Chrome

Firefox

Edge

Mobile

```

All browser projects can depend on the same setup project.

----------

### Q4. Can a setup project create multiple storage states?

Yes.

Example:

```text
admin.json

customer.json

manager.json

```

Each browser project or test suite can use the appropriate authentication state.

----------

### Q5. Where should authentication files be stored?

A common convention is:

```text
playwright/.auth/

```

and the folder should usually be added to `.gitignore`.

----------

### Q6. Should you prefer UI login or API login in the setup project?

If a stable authentication API exists, API login is generally preferred because it is faster and less brittle than driving the UI.

----------

# Best Practices

-   ✅ Prefer **Setup Projects** over `globalSetup` for authentication and environment preparation.
    
-   ✅ Use `storageState` to avoid repeated logins and significantly reduce execution time.
    
-   ✅ Use API-based authentication when available.
    
-   ✅ Store authentication files in `playwright/.auth/` and exclude them from version control.
    
-   ✅ Use project dependencies to model setup sequences (e.g., database → authentication → browser tests).
    
-   ✅ Reserve `globalSetup` for tasks that truly cannot be expressed as Playwright tests (rare in modern projects).
    

----------

# ⭐ Enterprise Authentication Architecture

```text
                 Setup Project
                       │
          ┌────────────┴────────────┐
          │                         │
     API Login                 Seed Test Data
          │                         │
          └────────────┬────────────┘
                       │
             Generate storageState
                       │
        playwright/.auth/admin.json
                       │
      ┌────────┬────────┬────────┐
      │        │        │        │
   Chrome   Firefox   Edge   Mobile
      │        │        │        │
      └────────┴────────┴────────┘
                   │
              Test Execution
                   │
             Cleanup Project

```

This pattern is widely adopted because it is fast, scalable, debuggable, and fully integrated with Playwright's reporting and retry mechanisms.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTUzNjYzODAyMF19
-->