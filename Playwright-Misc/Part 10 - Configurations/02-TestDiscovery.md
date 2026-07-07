# Part 2 – Test Discovery (`testDir`, `testMatch`, `testIgnore`)

One of the first things Playwright does when you execute:

```bash
npx playwright test

```

is **discover which files are test files**.

Test discovery is controlled primarily by:

-   `testDir`
    
-   `testMatch`
    
-   `testIgnore`
    

Understanding these options is important because they determine **which files Playwright executes**.

----------

# 1. `testDir`

## Purpose

`testDir` specifies the **root directory where Playwright looks for test files**.

### Syntax

```ts
testDir: './tests'

```

----------

## Example Project

```
project/
│
├── playwright.config.ts
│
├── tests/
│     login.spec.ts
│     cart.spec.ts
│     checkout.spec.ts
│
└── pages/

```

Configuration:

```ts
export default defineConfig({
  testDir: './tests'
});

```

Playwright searches only inside the `tests` folder.

----------

## Nested Folders

```
tests/
    login/
        login.spec.ts
    checkout/
        payment.spec.ts
    api/
        users.spec.ts

```

All nested folders are searched automatically.

No additional configuration is needed.

----------

## Using Another Folder

```
automation/
    ui/
        login.spec.ts

```

```ts
export default defineConfig({
    testDir: './automation/ui'
});

```

----------

## Multiple Test Directories?

❌ Not directly.

Playwright accepts only **one** `testDir`.

If you have multiple locations:

```
ui-tests/
api-tests/
mobile-tests/

```

Use **Projects** instead:

```ts
projects: [
    {
        name: 'UI',
        testDir: './ui-tests'
    },
    {
        name: 'API',
        testDir: './api-tests'
    }
]

```

This is the recommended enterprise approach.

----------

# 2. `testMatch`

## Purpose

`testMatch` tells Playwright **which files inside `testDir` should be considered test files**.

By default, Playwright looks for files such as:

```
*.spec.ts
*.test.ts
*.spec.js
*.test.js

```

----------

## Example

```
tests/

login.spec.ts
login.ts
helper.ts
common.ts
cart.spec.ts

```

Only these execute:

```
login.spec.ts
cart.spec.ts

```

----------

## Custom Pattern

Suppose your organization names tests like:

```
login.e2e.ts
checkout.e2e.ts
payment.e2e.ts

```

Configure:

```ts
testMatch: '**/*.e2e.ts'

```

Now Playwright executes only:

```
login.e2e.ts
checkout.e2e.ts
payment.e2e.ts

```

----------

## Multiple Patterns

```ts
testMatch: [
    '**/*.spec.ts',
    '**/*.e2e.ts'
]

```

Matches:

```
login.spec.ts
checkout.e2e.ts

```

----------

## Using Regular Expressions

You can also use a regular expression:

```ts
testMatch: /.*\.spec\.ts/

```

Or:

```ts
testMatch: /login.*/

```

Matches:

```
login.spec.ts
loginSmoke.spec.ts
loginRegression.spec.ts

```

----------

## Enterprise Example

```
tests/

smoke/

login.smoke.ts

regression/

checkout.regression.ts

api/

users.api.ts

```

Configuration:

```ts
testMatch: [
    '**/*.smoke.ts',
    '**/*.regression.ts'
]

```

API tests are skipped.

----------

# 3. `testIgnore`

## Purpose

Ignore files or folders from execution.

----------

Example:

```
tests/

login.spec.ts
cart.spec.ts
old/
legacy.spec.ts

```

Configuration:

```ts
testIgnore: '**/old/**'

```

Result:

```
✓ login.spec.ts

✓ cart.spec.ts

✗ old/legacy.spec.ts

```

----------

## Ignore Multiple Directories

```ts
testIgnore: [
    '**/old/**',
    '**/draft/**'
]

```

----------

## Ignore by File Pattern

```ts
testIgnore: '**/*.broken.ts'

```

Skips:

```
payment.broken.ts
checkout.broken.ts

```

----------

## Ignore a Single File

```ts
testIgnore: '**/login.spec.ts'

```

----------

# Combining `testMatch` and `testIgnore`

```
tests/

login.spec.ts
login.old.spec.ts
cart.spec.ts

```

Configuration:

```ts
testMatch: '**/*.spec.ts',

testIgnore: '**/*.old.spec.ts'

```

Executed:

```
login.spec.ts

cart.spec.ts

```

Skipped:

```
login.old.spec.ts

```

----------

# Real-World Enterprise Example

```
tests/

smoke/
regression/
performance/
experimental/
api/

```

```ts
export default defineConfig({

    testDir: './tests',

    testMatch: [
        '**/*.spec.ts'
    ],

    testIgnore: [
        '**/experimental/**',
        '**/*.broken.ts'
    ]

});

```

This allows developers to keep experimental or broken tests in the repository without executing them.

----------

# How `testDir`, `testMatch`, and `testIgnore` Work Together

Suppose the project looks like:

```
tests/

login/
    login.spec.ts
    login.helper.ts

checkout/
    payment.spec.ts

experimental/
    newfeature.spec.ts

api/
    users.api.ts

```

Configuration:

```ts
export default defineConfig({

    testDir: './tests',

    testMatch: '**/*.spec.ts',

    testIgnore: '**/experimental/**'

});

```

### Discovery Process

1.  Search inside `tests/`
    
2.  Find all files recursively.
    
3.  Keep only files matching `**/*.spec.ts`.
    
4.  Exclude anything under `experimental/`.
    

Final execution:

```
✓ login.spec.ts

✓ payment.spec.ts

✗ newfeature.spec.ts

✗ users.api.ts

```

----------

# Difference Between `testMatch` and `--grep`

This is a common interview topic.

### `testMatch`

Filters **files** before they are loaded.

Example:

```ts
testMatch: '**/*.smoke.ts'

```

Only smoke test files are loaded.

----------

### `--grep`

Filters **individual test cases** after the file has been loaded.

Example:

```bash
npx playwright test --grep "@smoke"

```

Even if a file contains 100 tests, only those tagged with `@smoke` execute.

----------

# Performance Tip

If your project contains thousands of tests:

✅ Use `testMatch` to reduce the number of files Playwright loads.

Using `--grep` alone still requires Playwright to load every matching test file before filtering the tests.

----------

# Common Mistakes

### ❌ Mistake 1

```ts
testDir: './test'

```

Actual folder:

```
tests/

```

Result:

```
No tests found

```

----------

### ❌ Mistake 2

```ts
testMatch: '*.spec.ts'

```

This matches only the current directory.

Use:

```ts
'**/*.spec.ts'

```

to search recursively.

----------

### ❌ Mistake 3

Ignoring too much:

```ts
testIgnore: '**/*'

```

No tests will be executed.

----------

# Interview Questions

### Q1. Can Playwright have multiple `testDir` values?

No. `testDir` accepts a single directory. Use **Projects** to manage multiple test roots.

----------

### Q2. Which is faster: `testMatch` or `--grep`?

`testMatch` is generally faster because it filters files before Playwright loads them. `--grep` filters tests after the files are loaded.

----------

### Q3. Can `testMatch` accept arrays?

Yes.

```ts
testMatch: [
    '**/*.spec.ts',
    '**/*.e2e.ts'
]

```

----------

### Q4. Does `testIgnore` support glob patterns?

Yes.

Example:

```ts
testIgnore: '**/legacy/**'

```

----------

# Best Practices

-   Keep all tests under a single `testDir` unless there is a compelling reason to separate them into projects.
    
-   Use consistent naming conventions such as `*.spec.ts` or `*.test.ts`.
    
-   Prefer `testIgnore` over deleting or commenting out unstable tests.
    
-   Use `testMatch` to distinguish between different suites (e.g., smoke, regression, e2e) by filename if that aligns with your team's conventions.
    
-   Use tag-based filtering (`--grep`) for dynamic execution scenarios, and file-based filtering (`testMatch`) for organizing test assets.
    

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1MDc1NDk1MjFdfQ==
-->