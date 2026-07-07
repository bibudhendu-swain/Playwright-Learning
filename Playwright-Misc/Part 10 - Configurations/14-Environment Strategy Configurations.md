# Part 14 – Environment Management & Configuration Strategy (Enterprise Guide)

> **This is one of the most important topics in enterprise Playwright automation.**
> 
> Almost every large organization has multiple environments:
> 
> -   Development
>     
> -   QA
>     
> -   SIT
>     
> -   UAT
>     
> -   Staging
>     
> -   Production
>     
> 
> The goal is to run **the same test suite** against different environments **without modifying test code**.

----------

# Why Environment Management Matters

❌ Bad approach

```ts
await page.goto('https://qa.company.com/login');

await page.fill('#username', 'admin');

await page.fill('#password', 'password');

```

Every environment change requires code changes.

----------

## Better Approach

```ts
await page.goto('/login');

```

Configuration decides:

```text
QA
↓

https://qa.company.com

```

or

```text
UAT
↓

https://uat.company.com

```

The test remains unchanged.

----------

# Enterprise Folder Structure

```text
project/

playwright.config.ts

.env

.env.qa

.env.uat

.env.prod

config/

env.ts

credentials.ts

urls.ts

```

----------

# Environment Variables

Node.js exposes environment variables through:

```ts
process.env

```

Example

```ts
console.log(process.env.BASE_URL);

```

Output

```text
https://qa.company.com

```

----------

# Using Environment Variables

Instead of:

```ts
baseURL:'https://qa.company.com'

```

Use:

```ts
baseURL:process.env.BASE_URL

```

Now Playwright reads:

```text
BASE_URL

```

at runtime.

----------

# Running Tests

Linux / macOS

```bash
BASE_URL=https://qa.company.com npx playwright test

```

Windows PowerShell

```powershell
$env:BASE_URL="https://qa.company.com"
npx playwright test

```

----------

# Using `.env`

Instead of typing:

```bash
BASE_URL=...

```

Create:

```text
.env

```

Example

```text
BASE_URL=https://qa.company.com

USERNAME=admin

PASSWORD=password

```

----------

# Loading `.env`

Install

```bash
npm install dotenv

```

Then:

```ts
import dotenv from 'dotenv';

dotenv.config();

```

Now:

```ts
process.env.BASE_URL

```

works automatically.

----------

# Different Environment Files

QA

```text
.env.qa

BASE_URL=https://qa.company.com

```

----------

UAT

```text
.env.uat

BASE_URL=https://uat.company.com

```

----------

PROD

```text
.env.prod

BASE_URL=https://company.com

```

----------

# Selecting Environment

Example

```bash
ENV=qa npx playwright test

```

Then:

```ts
dotenv.config({

path:`.env.${process.env.ENV}`

});

```

Execution

```text
ENV=qa

↓

.env.qa

```

----------

# Dynamic Base URL

```ts
use:{

baseURL:process.env.BASE_URL

}

```

Tests

```ts
await page.goto('/login');

```

Automatically uses

```text
QA

UAT

PROD

```

----------

# Dynamic Credentials

Instead of

```ts
page.fill('#user','admin');

```

Use

```ts
page.fill('#user',process.env.USERNAME!);

```

Password

```ts
process.env.PASSWORD

```

No secrets in code.

----------

# Creating an Environment Helper

Instead of accessing `process.env` everywhere, create a typed helper.

```ts
// config/env.ts

export const env = {

baseURL:process.env.BASE_URL!,

username:process.env.USERNAME!,

password:process.env.PASSWORD!

};

```

Usage

```ts
await page.goto(env.baseURL);

await page.fill('#user',env.username);

```

Cleaner and easier to maintain.

----------

# Environment-Specific Projects

```ts
projects:[

{

name:'QA',

use:{

baseURL:'https://qa.company.com'

}

},

{

name:'UAT',

use:{

baseURL:'https://uat.company.com'

}

}

]

```

One suite

↓

Two environments

----------

# Combining Browser + Environment

```text
Chrome QA

Chrome UAT

Firefox QA

Firefox UAT

```

Configuration

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

# Credentials Per Environment

QA

```text
qa_user

```

UAT

```text
uat_user

```

PROD

```text
readonly_user

```

Load dynamically using environment variables or separate storage state files.

----------

# Storage State Per Environment

```text
playwright/

.auth/

qa-admin.json

uat-admin.json

prod-admin.json

```

Project

```ts
storageState:

`playwright/.auth/${process.env.ENV}-admin.json`

```

----------

# Feature Flags

Some applications use custom headers.

Example

```ts
extraHTTPHeaders:{

'X-Feature':'NewCheckout'

}

```

Environment

QA

↓

Feature Enabled

Production

↓

Disabled

----------

# Multi-Tenant Applications

Suppose application supports:

```text
Tenant A

Tenant B

Tenant C

```

Projects

```ts
projects:[

{

name:'Tenant A',

use:{

extraHTTPHeaders:{

'Tenant':'A'

}

}

}
]

```

No duplicated tests.

----------

# GitHub Actions Secrets

Never commit

```text
PASSWORD=password

```

Instead

GitHub

↓

Secrets

↓

PASSWORD

Workflow

```yaml
env:

PASSWORD:${{ secrets.PASSWORD }}

```

Playwright

↓

```ts
process.env.PASSWORD

```

----------

# Azure DevOps

Variable Group

↓

```text
BASE_URL

USERNAME

PASSWORD

```

Pipeline

↓

Playwright

↓

process.env

----------

# Jenkins

Parameterized Build

↓

```text
Environment

↓

QA

UAT

PROD

```

Pipeline

↓

```bash
ENV=QA

npx playwright test

```

----------

# Enterprise Config

```ts
import dotenv from 'dotenv';

dotenv.config({

path:`.env.${process.env.ENV || 'qa'}`

});

export default defineConfig({

use:{

baseURL:process.env.BASE_URL,

storageState:

`playwright/.auth/${process.env.ENV}.json`

}

});

```

----------

# Configuration Factory (Advanced)

Instead of reading `process.env` throughout the codebase, centralize configuration.

```ts
// config/environment.ts

export function getEnvironment() {
  return {
    baseURL: process.env.BASE_URL!,
    username: process.env.USERNAME!,
    password: process.env.PASSWORD!,
    apiBaseURL: process.env.API_BASE_URL!
  };
}

```

Usage

```ts
const env = getEnvironment();

await page.goto(env.baseURL);

```

This makes testing and maintenance easier.

----------

# Avoid Hardcoded Values

❌

```ts
await page.goto('https://qa.company.com');

```

✅

```ts
await page.goto('/');

```

with

```ts
baseURL: process.env.BASE_URL

```

----------

# Environment Selection Flow

```text
CI Variable

↓

ENV=uat

↓

Load .env.uat

↓

Configure Playwright

↓

Run Tests

↓

Generate Report

```

----------

# Common Mistakes

## ❌ Committing Secrets

Never commit

```text
PASSWORD=admin123

```

Use:

-   GitHub Secrets
    
-   Azure DevOps Variable Groups
    
-   Jenkins Credentials
    
-   Secret managers (AWS Secrets Manager, Azure Key Vault, HashiCorp Vault, etc.)
    

----------

## ❌ Reading `process.env` Everywhere

Prefer:

```ts
config/environment.ts

```

One place.

----------

## ❌ Hardcoding URLs

Always use:

```ts
baseURL

```

----------

## ❌ Separate Repositories

Avoid

```text
QA Repo

UAT Repo

PROD Repo

```

Use one repository with configurable environments.

----------

## ❌ Different Test Code

The test should **never know** whether it runs in QA or UAT.

Only configuration changes.

----------

# Enterprise Folder Structure

```text
project/

config/

environment.ts

credentials.ts

playwright.config.ts

.env.qa

.env.uat

.env.prod

playwright/

.auth/

qa.json

uat.json

prod.json

tests/

pages/

utils/

```

----------

# Interview Questions

### Q1. How do you run the same Playwright suite against QA and UAT?

Use environment variables (or projects) to change configuration such as `baseURL`, credentials, and `storageState` without changing the test code.

----------

### Q2. Where should passwords be stored?

In secure secret stores or CI/CD secret management systems—not in source code or committed `.env` files containing production credentials.

----------

### Q3. Why use `.env` files?

They separate configuration from code and simplify switching between environments during local development.

----------

### Q4. What is `process.env`?

It is Node.js's interface for reading environment variables provided by the operating system or CI pipeline.

----------

### Q5. How do you switch environments?

Example:

```bash
ENV=uat npx playwright test

```

Then load:

```text
.env.uat

```

using your configuration logic.

----------

### Q6. Should tests contain environment-specific URLs?

No.

Use:

```ts
baseURL

```

and navigate with relative paths.

----------

### Q7. Where should environment logic live?

Ideally in a dedicated configuration layer such as:

```text
config/environment.ts

```

rather than scattered throughout the test code.

----------

# Best Practices

-   ✅ Keep **tests environment-agnostic**.
    
-   ✅ Use `baseURL` instead of hardcoded URLs.
    
-   ✅ Load environment-specific values from `.env` files or CI variables.
    
-   ✅ Centralize environment access in a configuration module.
    
-   ✅ Store credentials in secure secret management systems.
    
-   ✅ Use different `storageState` files per environment and user role.
    
-   ✅ Prefer API-based authentication for generating environment-specific authentication state.
    
-   ✅ Make environment selection a pipeline parameter rather than a code change.
    

----------

# ⭐ Enterprise Architecture (Recommended)

```text
                 CI Pipeline
                      │
              ENV = qa / uat / prod
                      │
          ┌───────────┴───────────┐
          │                       │
     Load .env File         Load Secrets
          │                       │
          └───────────┬───────────┘
                      │
            config/environment.ts
                      │
        ┌─────────────┼─────────────┐
        │             │             │
     baseURL     Credentials   storageState
        │             │             │
        └─────────────┴─────────────┘
                      │
              Playwright Projects
                      │
                 Execute Tests

```

This architecture scales well across multiple environments, browsers, and CI/CD platforms while keeping test code clean and reusable.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTc5ODcwNTg0Nl19
-->