# Part 11 – Projects (`projects`) – Complete Guide

If there is **one Playwright feature that truly differentiates it from frameworks like Selenium + TestNG**, it's **Projects**.

Projects allow you to run the **same tests** across:

-   Multiple browsers
    
-   Multiple devices
    
-   Multiple environments
    
-   Different user roles
    
-   Different locales
    
-   Different configurations
    

**...without duplicating test code.**

----------

# What are Projects?

A **Project** is simply a **named execution configuration**.

Instead of writing:

```text
Login Test (Chrome)

Login Test (Firefox)

Login Test (Safari)

Login Test (Mobile)

```

You write **one test**:

```ts
test('Login', async ({ page }) => {

});

```

Then configure multiple projects.

----------

# How Projects Work

```text
Tests
   │
   ▼
Login.spec.ts
Checkout.spec.ts
Search.spec.ts
   │
   ▼
Projects
   │
   ├── Chromium
   ├── Firefox
   ├── WebKit
   └── Mobile Chrome

```

Each project runs **the same tests** with different configurations.

----------

# Basic Project Structure

```ts
export default defineConfig({

    projects: [

    ]

});

```

----------

# Single Project

```ts
projects: [

{

name:'chromium',

use:{
browserName:'chromium'
}

}

]

```

Project name:

```text
chromium

```

Browser:

```text
Chromium

```

----------

# Multiple Browsers

```ts
projects:[

{

name:'Chromium',

use:{

browserName:'chromium'

}

},

{

name:'Firefox',

use:{

browserName:'firefox'

}

},

{

name:'WebKit',

use:{

browserName:'webkit'

}

}

]

```

Execution

```text
Chromium

↓

Firefox

↓

WebKit

```

Each browser executes the **entire suite**.

----------

# Browser Matrix

Suppose:

```text
50 Tests

```

Projects:

```text
Chrome

Firefox

Safari

```

Execution:

```text
50 × 3

=

150 Executions

```

----------

# Running a Single Project

CLI

```bash
npx playwright test --project=Chromium

```

Only Chromium runs.

----------

# Run Multiple Projects

```bash
npx playwright test --project=Chromium --project=Firefox

```

WebKit is skipped.

----------

# Project-Specific `use`

Global

```ts
use:{

baseURL:'https://qa.company.com'

}

```

Project

```ts
projects:[

{

name:'Firefox',

use:{

locale:'fr-FR'

}

}

]

```

Result

Global

```text
baseURL

```

Project

```text
locale

```

Project settings override global ones.

----------

# Mobile Testing

Import devices

```ts
import {

devices

}

from '@playwright/test';

```

----------

Example

```ts
projects:[

{

name:'Pixel 7',

use:{

...devices['Pixel 7']

}

}

]

```

Playwright configures:

-   Viewport
    
-   User Agent
    
-   Touch support
    
-   Device scale factor
    
-   Mobile mode
    

Automatically.

----------

# iPhone Example

```ts
{

name:'iPhone 15',

use:{

...devices['iPhone 15']

}

}

```

----------

# Desktop + Mobile

```ts
projects:[

{

name:'Desktop Chrome',

use:{

browserName:'chromium'

}

},

{

name:'iPhone',

use:{

...devices['iPhone 15']

}

}

]

```

Same tests

Desktop

AND

Mobile

----------

# Branded Browsers

```ts
{

name:'Google Chrome',

use:{

channel:'chrome'

}

}

```

----------

Microsoft Edge

```ts
{

name:'Microsoft Edge',

use:{

channel:'msedge'

}

}

```

----------

# Different Environments

QA

```ts
{

name:'QA',

use:{

baseURL:'https://qa.company.com'

}

}

```

----------

UAT

```ts
{

name:'UAT',

use:{

baseURL:'https://uat.company.com'

}

}

```

----------

Production

```ts
{

name:'PROD',

use:{

baseURL:'https://company.com'

}

}

```

----------

Execution

```text
QA

↓

UAT

↓

PROD

```

No test duplication.

----------

# Different Locales

```ts
projects:[

{

name:'English',

use:{

locale:'en-US'

}

},

{

name:'French',

use:{

locale:'fr-FR'

}

},

{

name:'German',

use:{

locale:'de-DE'

}

}

]

```

----------

# Different Timezones

```ts
{

name:'India',

use:{

timezoneId:'Asia/Kolkata'

}

}

```

----------

```ts
{

name:'New York',

use:{

timezoneId:'America/New_York'

}

}

```

----------

# Different User Roles

Admin

```ts
{

name:'Admin',

use:{

storageState:'admin.json'

}

}

```

Customer

```ts
{

name:'Customer',

use:{

storageState:'customer.json'

}

}

```

Guest

```ts
{

name:'Guest'

}

```

One suite

Three users

----------

# Different Viewports

Desktop

```ts
{

name:'Desktop',

use:{

viewport:{

width:1920,

height:1080

}

}

}

```

Tablet

```ts
{

name:'Tablet',

use:{

viewport:{

width:820,

height:1180

}

}

}

```

----------

# Combining Everything

```ts
projects:[

{

name:'Chrome QA',

use:{

channel:'chrome',

baseURL:'https://qa.company.com'

}

},

{

name:'Chrome UAT',

use:{

channel:'chrome',

baseURL:'https://uat.company.com'

}

}

]

```

----------

# Using Devices

Available devices

```ts
devices['Desktop Chrome']

devices['Desktop Edge']

devices['Pixel 7']

devices['Pixel 5']

devices['Galaxy S24']

devices['iPhone 15']

devices['iPad Pro']

devices['Desktop Safari']

```

Simply spread

```ts
...devices['iPhone 15']

```

----------

# Project Dependencies (NEWER FEATURE)

Projects can depend on another project.

Example

```text
Setup Project

↓

Chrome Tests

↓

Firefox Tests

```

Configuration

```ts
{

name:'setup',

testMatch:/global.setup.ts/

},

{

name:'chromium',

dependencies:['setup']

}

```

Setup executes once.

Then Chromium begins.

----------

# Teardown Project

```text
Setup

↓

Tests

↓

Cleanup

```

Enterprise frameworks use this for:

-   Database cleanup
    
-   User deletion
    
-   Environment reset
    

----------

# Project Output

HTML Report

```text
Chromium

✓ Login

✓ Checkout

```

Firefox

```text
✓ Login

❌ Checkout

```

Projects are grouped separately.

----------

# Enterprise Configuration

```ts
export default defineConfig({

projects:[

{

name:'Chromium',

use:{

browserName:'chromium'

}

},

{

name:'Firefox',

use:{

browserName:'firefox'

}

},

{

name:'WebKit',

use:{

browserName:'webkit'

}

},

{

name:'Mobile Chrome',

use:{

...devices['Pixel 7']

}

}

]

});

```

Very common.

----------

# Enterprise Matrix

```text
Chrome QA

Chrome UAT

Firefox QA

Firefox UAT

Edge QA

Edge UAT

```

One test

Runs

Six times

----------

# Common Mistakes

## ❌ Copying Tests

Instead of

```text
login.chrome.spec.ts

login.firefox.spec.ts

```

Use

Projects.

----------

## ❌ Hardcoding URLs

Instead of

```ts
page.goto('QA URL')

```

Use

```text
baseURL

```

inside each project.

----------

## ❌ Using Separate Repositories

One repository

↓

Projects

↓

Multiple browsers

----------

## ❌ Not Naming Projects Properly

Bad

```text
Project1

Project2

```

Good

```text
Chrome QA

Firefox QA

Mobile Chrome

```

----------

# Interview Questions

### Q1. What are Projects?

Projects allow the same test suite to run with different configurations (browser, device, environment, locale, authentication state, etc.) without duplicating test code.

----------

### Q2. Can each Project have different `use`?

Yes.

Each project overrides the global `use`.

----------

### Q3. Can Projects run in parallel?

Yes.

Each project gets its own workers.

----------

### Q4. Can one Project depend on another?

Yes.

Using:

```ts
dependencies:[]

```

----------

### Q5. What is the most common use of Projects?

Cross-browser testing.

----------

### Q6. Can Projects run different environments?

Yes.

QA

UAT

PROD

----------

### Q7. Can Projects emulate mobile devices?

Yes.

```ts
...devices['Pixel 7']

```

----------

### Q8. Can Projects use different storageState?

Absolutely.

Very common.

Admin

Customer

Guest

----------

# Best Practices

✅ Keep one test suite.

Don't duplicate tests.

----------

✅ Use Projects for

-   Browsers
    
-   Devices
    
-   Roles
    
-   Locales
    
-   Environments
    

----------

✅ Give meaningful names.

----------

✅ Prefer Projects over

Multiple config files

whenever possible.

----------

✅ Use

```ts
devices

```

instead of manually creating mobile viewports.

----------

# ⭐ Enterprise Project Structure (Recommended)

This is what you will commonly see in enterprise Playwright frameworks:

```ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  use: {
    baseURL: process.env.BASE_URL,
    trace: 'retain-on-failure',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure'
  },

  projects: [
    {
      name: 'Setup',
      testMatch: /auth\.setup\.ts/
    },

    {
      name: 'Chrome',
      use: {
        ...devices['Desktop Chrome'],
        storageState: 'playwright/.auth/user.json'
      },
      dependencies: ['Setup']
    },

    {
      name: 'Firefox',
      use: {
        ...devices['Desktop Firefox'],
        storageState: 'playwright/.auth/user.json'
      },
      dependencies: ['Setup']
    },

    {
      name: 'Edge',
      use: {
        channel: 'msedge',
        storageState: 'playwright/.auth/user.json'
      },
      dependencies: ['Setup']
    },

    {
      name: 'Mobile Chrome',
      use: {
        ...devices['Pixel 7'],
        storageState: 'playwright/.auth/user.json'
      },
      dependencies: ['Setup']
    }
  ]
});

```

This pattern is widely used because it:

-   Executes authentication once through a setup project.
    
-   Reuses the authenticated state across all browser projects.
    
-   Supports cross-browser and mobile execution from the same test suite.
    
-   Produces well-organized reports grouped by project.
    

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbNDI5Nzk1NjU0XX0=
-->