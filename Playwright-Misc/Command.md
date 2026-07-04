# Playwright CLI Cheat Sheet (`npx playwright`)

This cheat sheet focuses primarily on the **Playwright CLI** commands (`npx playwright ...`) and groups them by category.

----------

# 1. Installation & Setup

Command

Description

`npm init playwright@latest`

Create a new Playwright project

`npx playwright install`

Install all browsers

`npx playwright install chromium`

Install Chromium only

`npx playwright install firefox`

Install Firefox only

`npx playwright install webkit`

Install WebKit only

`npx playwright install --with-deps`

Install browsers with OS dependencies

`npx playwright install-deps`

Install only OS dependencies

`npx playwright uninstall`

Remove installed browsers

`npx playwright uninstall chromium`

Remove Chromium

----------

# 2. Running Tests

Command

Description

`npx playwright test`

Run all tests

`npx playwright test login.spec.ts`

Run a specific file

`npx playwright test -g "Login"`

Run tests matching title

`npx playwright test tests/login.spec.ts:25`

Run a test from a specific line

`npx playwright test --headed`

Run in headed mode

`npx playwright test --debug`

Debug mode

`npx playwright test --ui`

Open Playwright UI Mode

`npx playwright test --workers=4`

Parallel execution

`npx playwright test --project=chromium`

Run only Chromium project

`npx playwright test --project=firefox`

Firefox only

`npx playwright test --project=webkit`

WebKit only

`npx playwright test --repeat-each=5`

Repeat every test

`npx playwright test --retries=2`

Retry failures

`npx playwright test --grep @smoke`

Run tagged tests

`npx playwright test --grep-invert @skip`

Exclude tagged tests

`npx playwright test --max-failures=3`

Stop after failures

`npx playwright test --timeout=60000`

Override timeout

`npx playwright test --trace=on`

Enable tracing

`npx playwright test --trace=retain-on-failure`

Save trace only on failure

`npx playwright test --video=on`

Record videos

`npx playwright test --screenshot=only-on-failure`

Screenshots on failure

`npx playwright test --reporter=html`

HTML reporter

`npx playwright test --list`

List tests only

`npx playwright test --shard=1/4`

Run first shard

----------

# 3. Code Generation

Command

Description

`npx playwright codegen`

Launch recorder

`npx playwright codegen https://example.com`

Record from URL

`npx playwright codegen --browser=firefox`

Firefox recorder

`npx playwright codegen --browser=webkit`

WebKit recorder

`npx playwright codegen --device="iPhone 15"`

Mobile recording

`npx playwright codegen --color-scheme=dark`

Dark mode

`npx playwright codegen --viewport-size=1280,720`

Custom viewport

`npx playwright codegen --save-storage=state.json`

Save login state

`npx playwright codegen --load-storage=state.json`

Use saved state

`npx playwright codegen --lang=ts`

Generate TypeScript

`npx playwright codegen --lang=js`

JavaScript

`npx playwright codegen --lang=python`

Python

`npx playwright codegen --lang=java`

Java

`npx playwright codegen --lang=csharp`

C#

----------

# 4. Trace Viewer

Command

Description

`npx playwright show-trace trace.zip`

Open trace

`npx playwright show-trace trace1.zip trace2.zip`

Compare traces

----------

# 5. HTML Report

Command

Description

`npx playwright show-report`

Open latest report

`npx playwright show-report playwright-report`

Open specific report

----------

# 6. Browser Commands

Command

Description

`npx playwright open https://example.com`

Launch browser

`npx playwright open --browser=firefox`

Firefox

`npx playwright open --browser=webkit`

WebKit

`npx playwright open --device="Pixel 7"`

Mobile emulation

----------

# 7. Screenshot Utility

Command

Description

`npx playwright screenshot https://example.com page.png`

Capture screenshot

`npx playwright screenshot --device="iPhone 15"`

Mobile screenshot

`npx playwright screenshot --wait-for-timeout=5000`

Wait before capture

`npx playwright screenshot --full-page`

Full page screenshot

----------

# 8. PDF Generation

_(Chromium only)_

Command

Description

`npx playwright pdf https://example.com page.pdf`

Save PDF

`npx playwright pdf --landscape`

Landscape mode

`npx playwright pdf --format=A4`

A4 paper

`npx playwright pdf --margin=20px`

Set margins

----------

# 9. Test Listing

Command

Description

`npx playwright test --list`

List all tests

`npx playwright test --grep login --list`

Filtered list

----------

# 10. Debugging

Command

Description

`npx playwright test --debug`

Inspector mode

`PWDEBUG=1 npx playwright test`

Pause execution

`PWDEBUG=console npx playwright test`

Console debugging

`npx playwright test --headed --workers=1`

Easier debugging

----------

# 11. Browser Server

Command

Description

`npx playwright launch-server`

Launch browser server

`npx playwright launch-server --browser=chromium`

Chromium server

----------

# 12. Browser Installation Management

Command

Description

`npx playwright install`

Install all browsers

`npx playwright install chromium firefox`

Install selected browsers

`npx playwright install --force`

Force reinstall

`npx playwright install --only-shell`

Install Chromium headless shell only

`npx playwright install --no-shell`

Skip Chromium headless shell

`npx playwright uninstall`

Remove browsers

`npx playwright uninstall --all`

Remove all browser binaries

----------

# 13. Browser Cache

Command

Description

`npx playwright clear-cache`

Clear Playwright browser cache _(available in newer versions)_

----------

# 14. Test Filtering

Command

Description

`npx playwright test --grep smoke`

Run matching tests

`npx playwright test --grep-invert slow`

Exclude tests

`npx playwright test login.spec.ts`

Single file

`npx playwright test tests/login.spec.ts:42`

Single test by line

`npx playwright test --project=chromium`

Specific project

----------

# 15. Environment Variables (used with `npx playwright`)

Variable

Purpose

`PWDEBUG=1`

Inspector mode

`PWDEBUG=console`

Console debugger

`CI=true`

CI execution

`PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1`

Skip browser download

`PLAYWRIGHT_BROWSERS_PATH=0`

Store browsers locally

`PLAYWRIGHT_DOWNLOAD_HOST=<url>`

Use custom download host

----------

# 16. Useful Combinations

```bash
# Debug a single test
npx playwright test login.spec.ts --debug

# Run smoke tests in Chromium
npx playwright test --project=chromium --grep @smoke

# Run headed with trace
npx playwright test --headed --trace=on

# Run one test repeatedly
npx playwright test login.spec.ts --repeat-each=10

# Generate code using saved login
npx playwright codegen --load-storage=state.json

# Record login state
npx playwright codegen --save-storage=state.json

# Run in UI Mode
npx playwright test --ui

# Open latest HTML report
npx playwright show-report

# View a trace
npx playwright show-trace trace.zip

# Capture a full-page screenshot
npx playwright screenshot --full-page https://example.com page.png

```

## Most commonly used commands in day-to-day automation

```bash
npx playwright test
npx playwright test --ui
npx playwright test --debug
npx playwright test --headed
npx playwright test --project=chromium
npx playwright test --grep @smoke
npx playwright codegen
npx playwright show-trace trace.zip
npx playwright show-report
npx playwright install
npx playwright install --with-deps
npx playwright open https://example.com
npx playwright screenshot https://example.com page.png

```

Here are the **tag filtering commands** you can add to the **Test Filtering** section of your cheat sheet.

----------

## Test Filtering with Tags (AND / OR / NOT Logic)

Command

Logic

Description

`npx playwright test --grep "@smoke"`

Single Tag

Run tests tagged with `@smoke`

`npx playwright test --grep "@smoke|@regression"`

**OR**

Run tests having either `@smoke` or `@regression`

`npx playwright test --grep "(?=.*@smoke)(?=.*@regression)"`

**AND**

Run tests containing both `@smoke` and `@regression`

`npx playwright test --grep "(?=.*@smoke)(?=.*@api)(?=.*@critical)"`

Multiple AND

Run tests containing all specified tags

`npx playwright test --grep-invert "@slow"`

NOT

Exclude all `@slow` tests

`npx playwright test --grep "@smoke" --grep-invert "@flaky"`

AND + NOT

Run smoke tests except flaky ones

`npx playwright test --grep "@smoke|@sanity" --grep-invert "@skip"`

OR + NOT

Run smoke or sanity tests, excluding skipped ones

`npx playwright test --grep "(?=.*@smoke)(?=.*@ui)" --grep-invert "@flaky"`

AND + NOT

Run UI smoke tests except flaky tests

`npx playwright test --grep "(?=.*@regression)(?=.*@chrome)"`

AND

Run Chrome regression tests

`npx playwright test --grep "@api|@ui|@mobile"`

Multiple OR

Run API, UI, or Mobile tests

----------

## Regular Expression Logic Used

### OR (`|`)

```bash
npx playwright test --grep "@smoke|@regression"

```

Equivalent to:

```
@smoke OR @regression

```

----------

### AND (Positive Lookahead)

```bash
npx playwright test --grep "(?=.*@smoke)(?=.*@regression)"

```

Equivalent to:

```
@smoke AND @regression

```

----------

### NOT (`--grep-invert`)

```bash
npx playwright test --grep-invert "@slow"

```

Equivalent to:

```
NOT @slow

```

----------

### OR + NOT

```bash
npx playwright test --grep "@smoke|@sanity" --grep-invert "@skip"

```

Equivalent to:

```
(@smoke OR @sanity) AND NOT @skip

```

----------

### AND + NOT

```bash
npx playwright test --grep "(?=.*@smoke)(?=.*@api)" --grep-invert "@flaky"

```

Equivalent to:

```
(@smoke AND @api) AND NOT @flaky

```

----------

## Examples

```bash
# Smoke tests
npx playwright test --grep "@smoke"

# Smoke OR Regression
npx playwright test --grep "@smoke|@regression"

# Smoke AND Regression
npx playwright test --grep "(?=.*@smoke)(?=.*@regression)"

# Smoke AND UI AND Critical
npx playwright test --grep "(?=.*@smoke)(?=.*@ui)(?=.*@critical)"

# Exclude Slow tests
npx playwright test --grep-invert "@slow"

# Smoke but not Flaky
npx playwright test --grep "@smoke" --grep-invert "@flaky"

# Smoke or Sanity but not Skip
npx playwright test --grep "@smoke|@sanity" --grep-invert "@skip"

# API or UI or Mobile
npx playwright test --grep "@api|@ui|@mobile"

# Chrome Regression
npx playwright test --grep "(?=.*@chrome)(?=.*@regression)"

# Login tests only
npx playwright test --grep "Login"

# Login smoke tests
npx playwright test --grep "(?=.*Login)(?=.*@smoke)"

```

### Tip

-   `|` = **OR**
    
-   `(?=.*tag)` = **AND** (positive lookahead)
    
-   `--grep-invert` = **NOT**
    
-   You can combine `--grep` and `--grep-invert` to build complex expressions such as **(A OR B) AND NOT C** or **(A AND B) AND NOT C**.
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTk0NDAwMzE4MV19
-->