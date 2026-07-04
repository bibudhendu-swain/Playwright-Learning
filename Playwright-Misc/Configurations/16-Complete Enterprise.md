Excellent! This is the culmination of the handbook. We'll build a **production-ready enterprise `playwright.config.ts`** and explain every section. This isn't just a config file—it's an architecture template suitable for most enterprise Playwright projects.

----------

# Part 16 – Complete Enterprise `playwright.config.ts`

## Enterprise Folder Structure

Before writing the configuration, organize your project like this:

```text
playwright-framework/
│
├── playwright.config.ts
├── package.json
├── .env.qa
├── .env.uat
├── .env.prod
│
├── playwright/
│   ├── .auth/
│   │      admin.json
│   │      customer.json
│   │
│   ├── auth.setup.ts
│   └── global.teardown.ts
│
├── config/
│   ├── environment.ts
│   ├── constants.ts
│   └── users.ts
│
├── tests/
│   ├── ui/
│   ├── api/
│   ├── smoke/
│   └── regression/
│
├── pages/
├── utils/
├── fixtures/
├── test-data/
├── reports/
├── artifacts/
└── snapshots/

```

This keeps configuration, authentication, tests, fixtures, and generated artifacts well organized.

----------

# Production-Ready Configuration

```ts
import { defineConfig, devices } from '@playwright/test';
import dotenv from 'dotenv';

dotenv.config({
  path: `.env.${process.env.ENV || 'qa'}`
});

export default defineConfig({

  //---------------------------------------
  // Test Discovery
  //---------------------------------------

  testDir: './tests',

  testMatch: '**/*.spec.ts',

  //---------------------------------------
  // Execution
  //---------------------------------------

  fullyParallel: true,

  workers: process.env.CI ? 4 : undefined,

  retries: process.env.CI ? 2 : 0,

  timeout: 90000,

  globalTimeout: 4 * 60 * 60 * 1000,

  maxFailures: process.env.CI ? 20 : undefined,

  forbidOnly: !!process.env.CI,

  failOnFlakyTests: !!process.env.CI,

  //---------------------------------------
  // Expect
  //---------------------------------------

  expect: {

    timeout:10000

  },

  //---------------------------------------
  // Artifacts
  //---------------------------------------

  outputDir:'artifacts',

  preserveOutput:'failures-only',

  //---------------------------------------
  // Reports
  //---------------------------------------

  reporter:[

    ['list'],

    ['html',{

      open:'never'

    }],

    ['junit',{

      outputFile:'reports/results.xml'

    }]

  ],

  //---------------------------------------
  // Browser Context
  //---------------------------------------

  use:{

    baseURL:process.env.BASE_URL,

    headless:true,

    viewport:{

      width:1920,

      height:1080

    },

    locale:'en-US',

    timezoneId:'Asia/Kolkata',

    ignoreHTTPSErrors:true,

    storageState:

      `playwright/.auth/${process.env.USER_ROLE || 'admin'}.json`,

    screenshot:'only-on-failure',

    video:'retain-on-failure',

    trace:'retain-on-failure',

    acceptDownloads:true,

    actionTimeout:10000,

    navigationTimeout:30000

  },

  //---------------------------------------
  // Web Server
  //---------------------------------------

  webServer:{

    command:'npm run dev',

    url:process.env.BASE_URL,

    reuseExistingServer:!process.env.CI

  },

  //---------------------------------------
  // Projects
  //---------------------------------------

  projects:[

    {

      name:'Setup',

      testMatch:/auth\.setup\.ts/

    },

    {

      name:'Chrome',

      use:{

        ...devices['Desktop Chrome']

      },

      dependencies:['Setup']

    },

    {

      name:'Firefox',

      use:{

        ...devices['Desktop Firefox']

      },

      dependencies:['Setup']

    },

    {

      name:'Edge',

      use:{

        channel:'msedge'

      },

      dependencies:['Setup']

    },

    {

      name:'Mobile Chrome',

      use:{

        ...devices['Pixel 7']

      },

      dependencies:['Setup']

    }

  ],

  //---------------------------------------
  // Advanced
  //---------------------------------------

  metadata:{

    environment:process.env.ENV,

    build:process.env.BUILD_NUMBER,

    release:process.env.RELEASE_VERSION

  },

  reportSlowTests:{

    max:20,

    threshold:30000

  },

  testIdAttribute:'automation-id'

});

```

----------

# Section-by-Section Breakdown

## Test Discovery

```ts
testDir:'./tests'

testMatch:'**/*.spec.ts'

```

Determines:

-   where Playwright searches
    
-   which files are executed
    

----------

## Execution

```ts
workers

retries

fullyParallel

maxFailures

timeout

```

Responsible for:

-   parallel execution
    
-   retries
    
-   CI behavior
    

----------

## Browser Context

```ts
use:{}

```

Creates

```text
Fresh Browser Context

↓

Every Test

```

Contains:

-   viewport
    
-   storageState
    
-   screenshots
    
-   traces
    
-   downloads
    
-   locale
    

----------

## Web Server

Starts application automatically.

```text
npm run dev

↓

Wait

↓

Run Tests

```

----------

## Projects

Creates browser matrix.

```text
Chrome

Firefox

Edge

Mobile

```

Same test.

Multiple browsers.

----------

## Setup Project

Runs once.

```text
Authenticate

↓

Generate storageState

↓

Chrome

↓

Firefox

↓

Edge

```

No repeated login.

----------

## Reporters

Produces

```text
Console

↓

HTML

↓

JUnit

```

Suitable for local debugging and CI integration.

----------

## Metadata

Makes reports much richer.

```text
Environment

Release

Build Number

```

Very useful when viewing historical reports.

----------

# Typical Enterprise Execution Flow

```text
Load Environment

↓

Start Server

↓

Setup Project

↓

Authenticate

↓

Save Storage State

↓

Chrome Tests

↓

Firefox Tests

↓

Edge Tests

↓

Generate Reports

↓

Archive Artifacts

↓

Cleanup

```

----------

# How This Fits in CI/CD

## GitHub Actions

```text
Checkout

↓

Install

↓

Load Secrets

↓

Run Playwright

↓

Publish JUnit

↓

Upload HTML Report

↓

Upload Trace

```

----------

## Azure DevOps

```text
Checkout

↓

Install

↓

Run Tests

↓

Publish Test Results

↓

Publish HTML Report

↓

Publish Artifacts

```

----------

## Jenkins

```text
Checkout

↓

npm ci

↓

Playwright

↓

JUnit Plugin

↓

Archive Artifacts

```

----------

# Execution Hierarchy

One of the most common interview questions.

```text
CLI

↓

Project

↓

test.use()

↓

Global use

↓

Playwright Default

```

Example

```text
CLI

--headed

↓

Overrides

↓

headless:true

```

CLI wins.

----------

# Browser Architecture

```text
Chromium

│

├── Context 1

│      └── Page

│

├── Context 2

│      └── Page

│

└── Context 3

       └── Page

```

Every test gets a fresh context.

----------

# Worker Architecture

```text
Worker 1

↓

Chrome

↓

Login.spec

↓

Cart.spec



Worker 2

↓

Chrome

↓

Search.spec

↓

Checkout.spec

```

Workers are isolated.

----------

# Project Execution

```text
Setup

↓

Chrome

↓

Firefox

↓

Edge

↓

Mobile

```

Each project receives its own workers.

----------

# Common Enterprise Improvements

Most mature frameworks add:

### Custom Fixtures

```text
fixtures/

auth.fixture.ts

api.fixture.ts

db.fixture.ts

```

----------

### Custom Reporters

-   Slack notifications
    
-   Teams integration
    
-   Email reports
    

----------

### API Helpers

```text
api/

CustomerApi.ts

OrderApi.ts

```

----------

### Database Helpers

```text
db/

DatabaseHelper.ts

```

----------

### Environment Helper

```text
config/

environment.ts

```

Never use `process.env` directly in tests.

----------

### Test Data

```text
test-data/

customer.json

orders.json

```

----------

# Enterprise Folder Layout

```text
src/

pages/

components/

fixtures/

utils/

api/

db/

tests/

config/

playwright/

.auth/

reports/

artifacts/

snapshots/

```

----------

# Most Common Interview Questions

## Q1

**How do you authenticate once?**

Use:

-   Setup Project
    
-   storageState
    
-   dependencies
    

----------

## Q2

**How do you support multiple browsers?**

Projects.

----------

## Q3

**How do you support multiple environments?**

-   `.env`
    
-   baseURL
    
-   process.env
    

----------

## Q4

**How do you support multiple users?**

Multiple

```text
storageState

```

files.

----------

## Q5

**How do you reduce execution time?**

-   workers
    
-   fullyParallel
    
-   storageState
    
-   API login
    
-   Setup Project
    

----------

## Q6

**How do you debug failures?**

-   Trace
    
-   Screenshot
    
-   Video
    
-   HTML Report
    

----------

## Q7

**What is the biggest advantage of Playwright over Selenium?**

A strong answer could include:

-   Auto-waiting
    
-   Auto-retrying assertions
    
-   Built-in parallel execution
    
-   Browser context isolation
    
-   Trace Viewer
    
-   Cross-browser support out of the box
    
-   Rich configuration through `playwright.config.ts`
    

----------

# Final Enterprise Best Practices Checklist

Area

Recommendation

Authentication

Use Setup Project + `storageState`

URLs

Always use `baseURL`

Environments

`.env` + environment helper

Browsers

Projects

Mobile

`devices` presets

Debugging

`trace: 'retain-on-failure'`

Screenshots

`only-on-failure`

Videos

`retain-on-failure`

Reports

HTML + JUnit

CI

`retries`, `forbidOnly`, `maxFailures`

Test IDs

Configure `testIdAttribute`

Secrets

CI secret management, not source code

Performance

Worker tuning + `reportSlowTests`

Test Design

Independent, isolated, idempotent tests

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTE3NDk4MTMyOF19
-->