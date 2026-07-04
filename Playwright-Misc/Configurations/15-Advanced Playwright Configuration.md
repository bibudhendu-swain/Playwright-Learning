# Part 15 – Advanced Playwright Configuration (Enterprise & Interview Guide)

> This chapter covers the **less frequently used—but extremely powerful—configuration options** that you'll often see in enterprise Playwright frameworks.
> 
> These settings help with:
> 
> -   Better reporting
>     
> -   Snapshot organization
>     
> -   Build optimization
>     
> -   CI/CD
>     
> -   Debugging
>     
> -   Custom test IDs
>     
> -   Metadata
>     
> -   Large-scale framework maintenance
>     

----------

# Advanced Configuration Overview

```ts
export default defineConfig({

    outputDir: 'test-results',

    preserveOutput: 'failures-only',

    snapshotPathTemplate: '{testDir}/__screenshots__/{testFilePath}/{arg}{ext}',

    metadata: {

        team: 'QA',

        release: '2.3.0'

    },

    testIdAttribute: 'data-testid'

});

```

----------

# 1. `outputDir`

## Purpose

Specifies where Playwright stores generated test artifacts.

Artifacts include:

-   screenshots
    
-   videos
    
-   traces
    
-   attachments
    

----------

Default

```text
test-results/

```

----------

Configuration

```ts
outputDir:'artifacts'

```

Generated

```text
artifacts/

trace.zip

video.webm

failure.png

```

----------

## Why Change It?

Enterprise projects often organize reports separately.

Example

```text
reports/

artifacts/

screenshots/

videos/

allure-results/

```

----------

# 2. `preserveOutput`

Controls whether Playwright keeps artifacts after execution.

----------

Available Options

Option

Meaning

`always`

Keep everything

`never`

Delete everything

`failures-only`

Keep only failed test artifacts

----------

Configuration

```ts
preserveOutput:'failures-only'

```

Example

```text
100 Tests

↓

99 Pass

↓

1 Fail

```

Only failed test artifacts remain.

----------

Recommended

CI

```ts
preserveOutput:'failures-only'

```

----------

# 3. `snapshotPathTemplate`

One of the least-known but very useful features.

By default

```text
tests/

login.spec.ts

login.spec.ts-snapshots/

```

Snapshot folder sits next to the test.

----------

Custom structure

```ts
snapshotPathTemplate:

'{testDir}/__screenshots__/{testFilePath}/{arg}{ext}'

```

Generated

```text
tests/

__screenshots__/

login/

homepage.png

```

Much cleaner.

----------

Useful placeholders include:

Placeholder

Meaning

`{testDir}`

Test directory

`{testFilePath}`

Relative test file path

`{testFileName}`

Test filename

`{projectName}`

Project name

`{arg}`

Snapshot name

`{ext}`

File extension

----------

Enterprise Example

```ts
snapshotPathTemplate:

'{testDir}/snapshots/{projectName}/{testFilePath}/{arg}{ext}'

```

Generated

```text
snapshots/

Chrome/

Firefox/

Mobile/

```

----------

# 4. `metadata`

Adds custom information to reports.

Example

```ts
metadata:{

release:'2.5',

team:'Automation',

environment:'QA'

}

```

HTML report shows

```text
Release

2.5

```

Useful for

-   releases
    
-   sprint
    
-   build number
    
-   pipeline
    

----------

Dynamic Metadata

```ts
metadata:{

build:process.env.BUILD_NUMBER,

branch:process.env.GIT_BRANCH

}

```

----------

# 5. `updateSnapshots`

Controls snapshot updates.

Normally

```bash
npx playwright test --update-snapshots

```

updates snapshots.

Configuration

```ts
updateSnapshots:'missing'

```

Only missing snapshots are created.

----------

Options

Option

Meaning

`all`

Update everything

`missing`

Only create missing snapshots

`none`

Never update automatically

----------

# 6. `quiet`

Suppresses console output.

Configuration

```ts
quiet:true

```

Useful for

-   CI
    
-   clean logs
    

----------

# 7. `testIdAttribute`

Probably one of the most useful advanced options.

Default

```ts
getByTestId()

```

uses

```text
data-testid

```

HTML

```html
<button data-testid="login-btn">

```

Locator

```ts
page.getByTestId('login-btn')

```

----------

Custom Attribute

Suppose application uses

```html
<button automation-id="login">

```

Configure

```ts
testIdAttribute:'automation-id'

```

Now

```ts
page.getByTestId('login')

```

works automatically.

No locator changes.

----------

Enterprise Example

Many companies use

```text
qa-id

test-id

automation-id

data-test

```

instead of

```text
data-testid

```

----------

# 8. `tsconfig`

Specify custom TypeScript config.

Default

```text
tsconfig.json

```

Custom

```ts
tsconfig:'configs/tsconfig.playwright.json'

```

Useful when application and tests use different TypeScript configurations.

----------

# 9. `name`

Project name shown in reports.

Example

```ts
name:'Smoke Suite'

```

Report

```text
Smoke Suite

```

Instead of

```text
chromium

```

----------

# 10. `captureGitInfo` _(Recent Feature)_

Playwright can capture Git metadata automatically.

Example

```ts
captureGitInfo:{

commit:true,

diff:true

}

```

Reports may include:

-   Commit hash
    
-   Changed files
    
-   Git diff (where supported)
    

Very useful in CI.

----------

# 11. `build`

Allows configuring how Playwright handles TypeScript transpilation.

Example

```ts
build:{

external:[

'./node_modules'

]

}

```

Useful for large monorepos or optimizing build performance.

----------

# 12. `ignoreSnapshots`

Useful for API-only projects.

```ts
ignoreSnapshots:true

```

Snapshot assertions are skipped.

----------

# 13. `reportSlowTests`

Identifies slow tests.

Example

```ts
reportSlowTests:{

max:10,

threshold:30000

}

```

Report

```text
Top 10 Slow Tests

```

Very useful for performance tuning.

----------

# 14. `grep`

Instead of CLI

```bash
npx playwright test --grep @smoke

```

Config

```ts
grep:/@smoke/

```

Every execution

↓

Smoke only.

----------

# 15. `grepInvert`

Configuration

```ts
grepInvert:/@manual/

```

Manual tests never execute.

----------

# Complete Enterprise Configuration

```ts
export default defineConfig({

outputDir:'artifacts',

preserveOutput:'failures-only',

metadata:{

team:'Automation',

release:'2.5'

},

testIdAttribute:'automation-id',

reportSlowTests:{

max:20,

threshold:20000

}

});

```

----------

# Advanced Snapshot Organization

Instead of

```text
login.spec.ts-snapshots/

```

Enterprise

```text
snapshots/

Chrome/

Firefox/

Mobile/

Login/

Homepage.png

```

Much easier to maintain.

----------

# Real Enterprise Example

```ts
export default defineConfig({

outputDir:'artifacts',

preserveOutput:'failures-only',

snapshotPathTemplate:

'{testDir}/snapshots/{projectName}/{testFilePath}/{arg}{ext}',

metadata:{

build:process.env.BUILD,

environment:process.env.ENV,

release:'3.1.0'

},

testIdAttribute:'automation-id',

reportSlowTests:{

max:20,

threshold:30000

}

});

```

Very common.

----------

# Common Mistakes

## ❌ Keeping Every Artifact

```ts
preserveOutput:'always'

```

10,000 tests

↓

Huge disk usage.

----------

Better

```ts
failures-only

```

----------

## ❌ Hardcoding Test IDs

Instead of

```ts
locator('[automation-id="login"]')

```

Configure

```ts
testIdAttribute

```

Use

```ts
getByTestId()

```

----------

## ❌ Ignoring Slow Tests

Always enable

```ts
reportSlowTests

```

Large projects benefit greatly.

----------

## ❌ Flat Snapshot Folder

Organize snapshots by:

-   Project
    
-   Browser
    
-   Test
    

----------

# Interview Questions

### Q1. What does `outputDir` do?

It specifies where Playwright stores generated artifacts such as screenshots, videos, traces, and attachments.

----------

### Q2. What is `preserveOutput`?

It controls which test artifacts are kept after execution.

----------

### Q3. Why use `testIdAttribute`?

It allows `getByTestId()` to work with your application's custom test attribute instead of the default `data-testid`.

----------

### Q4. What is `metadata`?

Custom information added to reports, such as build numbers, release versions, team names, or environment details.

----------

### Q5. What is `snapshotPathTemplate`?

It controls how Playwright organizes snapshot files on disk.

----------

### Q6. What is `reportSlowTests`?

It identifies tests that exceed a configured duration threshold, helping teams optimize suite performance.

----------

### Q7. Why change `tsconfig`?

When the Playwright tests require a different TypeScript configuration than the application code.

----------

# Best Practices

✅ Use

```ts
outputDir:'artifacts'

```

to separate generated files.

----------

✅ Prefer

```ts
preserveOutput:'failures-only'

```

----------

✅ Configure

```ts
testIdAttribute

```

to match your application's test attribute.

----------

✅ Add

```ts
metadata

```

to reports.

----------

✅ Organize snapshots by

-   browser
    
-   project
    
-   feature
    

----------

✅ Monitor

```ts
reportSlowTests

```

regularly.

----------

# ⭐ Enterprise Configuration (Recommended)

```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({

  outputDir: 'artifacts',

  preserveOutput: 'failures-only',

  testIdAttribute: 'automation-id',

  metadata: {
    team: 'QA Automation',
    application: 'Customer Portal',
    environment: process.env.ENV,
    release: process.env.RELEASE_VERSION,
    build: process.env.BUILD_NUMBER
  },

  snapshotPathTemplate:
    '{testDir}/snapshots/{projectName}/{testFilePath}/{arg}{ext}',

  reportSlowTests: {
    max: 20,
    threshold: 30000
  },

  captureGitInfo: {
    commit: true,
    diff: true
  }

});

```

This configuration is typical of mature enterprise Playwright frameworks because it:

-   Produces clean artifact directories.
    
-   Minimizes disk usage.
    
-   Makes reports self-describing with build metadata.
    
-   Standardizes `getByTestId()` across the application.
    
-   Highlights slow tests for continuous optimization.
    
-   Organizes snapshots in a scalable directory structure.
    

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEzOTk3MDYyNDZdfQ==
-->