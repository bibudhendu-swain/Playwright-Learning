# Part 7 – Screenshots, Videos & Traces (`use.screenshot`, `use.video`, `use.trace`)

One of Playwright's most powerful debugging features is its ability to automatically capture **screenshots**, **videos**, and **execution traces**. Together, these artifacts make it much easier to diagnose failures without reproducing them locally.

Think of them as three different levels of debugging:

Artifact

Best For

📸 Screenshot

Quick visual state of the page

🎥 Video

Watch the entire test execution

🔍 Trace

Step-by-step interactive debugging with DOM snapshots, network, console, and timeline

For most enterprise teams, **Trace Viewer** is the first tool to use when investigating a failed test.

----------

# 1. Screenshots (`use.screenshot`)

## Purpose

Automatically capture screenshots during test execution.

----------

## Configuration

```ts
export default defineConfig({

  use: {
    screenshot: 'only-on-failure'
  }

});

```

----------

## Available Options

Value

Description

`'off'`

Never capture screenshots (default)

`'on'`

Capture for every test

`'only-on-failure'`

Capture only failed tests

----------

## Example

```ts
use: {
  screenshot: 'only-on-failure'
}

```

Execution:

```text
Login ✅

Search ✅

Checkout ❌

```

Generated:

```text
test-results/

checkout-failed-1.png

```

----------

## Screenshot for Every Test

```ts
use: {

    screenshot: 'on'

}

```

Produces:

```text
Login.png

Search.png

Checkout.png

```

Useful for UI regression or design validation, but it increases storage usage.

----------

# Manual Screenshot

Sometimes you want a screenshot at a specific point.

```ts
await page.screenshot({

    path: 'screenshots/login-page.png'

});

```

----------

## Full Page Screenshot

```ts
await page.screenshot({

    path: 'fullpage.png',

    fullPage: true

});

```

Captures the entire scrollable page.

----------

## Screenshot of an Element

```ts
await page.locator('#checkout').screenshot({

    path: 'checkout-button.png'

});

```

Only the selected element is captured.

----------

# 2. Video Recording (`use.video`)

## Purpose

Record the complete browser session.

----------

## Configuration

```ts
use: {

    video: 'retain-on-failure'

}

```

----------

## Available Options

Value

Description

`'off'`

No video (default)

`'on'`

Record every test

`'retain-on-failure'`

Keep only failed test videos

`'on-first-retry'`

Record only the first retry

----------

## Example

```ts
video: 'retain-on-failure'

```

Execution:

```text
Login ✅

Checkout ❌

```

Generated:

```text
checkout.webm

```

The successful test's video is discarded.

----------

## Record Every Test

```ts
video: 'on'

```

Useful for:

-   demos
    
-   training
    
-   reproducing intermittent issues
    

Be aware that this can consume significant disk space.

----------

## Record Only on Retry

```ts
video: 'on-first-retry'

```

A common enterprise choice because:

-   first execution stays fast
    
-   video is available only when troubleshooting flaky failures
    

----------

# Manual Access to Video

Inside a test:

```ts
const video = page.video();

const path = await video?.path();

console.log(path);

```

You can also save or attach the video in custom reporting workflows.

----------

# 3. Trace Recording (`use.trace`)

## Purpose

Trace is Playwright's most comprehensive debugging artifact.

A trace captures:

-   every Playwright API call
    
-   DOM snapshots
    
-   screenshots
    
-   network requests
    
-   console logs
    
-   source code
    
-   timing information
    

It allows you to "replay" a test after execution.

----------

## Configuration

```ts
use: {

    trace: 'on'

}

```

----------

## Available Options

Value

Description

`'off'`

No trace

`'on'`

Record every test

`'retain-on-failure'`

Save traces only for failures

`'on-first-retry'`

Record only on first retry

----------

## Recommended Enterprise Setting

```ts
trace: 'retain-on-failure'

```

Provides detailed diagnostics without generating traces for successful tests.

----------

# Trace File

Generated file:

```text
trace.zip

```

----------

Open it:

```bash
npx playwright show-trace trace.zip

```

----------

# What's Inside a Trace?

When opened in Trace Viewer, you'll see:

-   Timeline
    
-   Action log
    
-   DOM snapshot
    
-   Network activity
    
-   Console messages
    
-   Source code
    
-   Test steps
    
-   Screenshots
    
-   Timing details
    

This makes it possible to understand exactly what happened without rerunning the test.

----------

# Trace Viewer

Example flow:

```text
Step 1

page.goto()

↓

DOM Snapshot

↓

Network

↓

Console

↓

Locator Click

↓

Screenshot

```

You can move backward and forward through each recorded action.

----------

# Why Trace Is Better Than Video

Video

Trace

Visual replay

Interactive debugging

Cannot inspect DOM

DOM snapshots

No network details

Full network log

No console

Console messages

No locator details

Playwright API steps

Passive viewing

Step-by-step investigation

When debugging Playwright tests, **start with the trace**. Videos are useful for visual context but provide less diagnostic detail.

----------

# Combining Screenshots, Videos & Traces

```ts
export default defineConfig({

    use: {

        screenshot: 'only-on-failure',

        video: 'retain-on-failure',

        trace: 'retain-on-failure'

    }

});

```

This is a solid default configuration for many projects.

----------

# Per-Test Overrides

You can override settings for a specific test:

```ts
test.use({

    trace: 'on',

    video: 'on'

});

```

Only tests in that scope use these settings.

----------

# Debugging a Failure

Suppose:

```text
Login

↓

Click Checkout

↓

Timeout

```

Artifacts generated:

```text
Screenshot

checkout.png

Video

checkout.webm

Trace

trace.zip

```

Open the trace:

```bash
npx playwright show-trace trace.zip

```

Inspect:

-   Was the button visible?
    
-   Was it enabled?
    
-   Did the network request complete?
    
-   Were there JavaScript errors?
    
-   What locator was used?
    

----------

# Attach Files to Reports

Playwright allows attaching files to the test result.

```ts
await testInfo.attach('Request Body', {

    path: 'request.json',

    contentType: 'application/json'

});

```

Or attach a screenshot:

```ts
await testInfo.attach('Screenshot', {

    path: 'login.png',

    contentType: 'image/png'

});

```

Attachments appear in supported reporters, such as the HTML report.

----------

# Output Directory

By default, artifacts are stored in:

```text
test-results/

```

You can customize the location:

```ts
export default defineConfig({

    outputDir: 'artifacts'

});

```

Generated structure:

```text
artifacts/

trace.zip

video.webm

failure.png

```

----------

# Storage Considerations

Approximate sizes:

Artifact

Typical Size

Screenshot

100 KB–500 KB

Video

2–20 MB

Trace

1–10 MB

For a suite of 5,000 tests:

```text
Video on

↓

Hundreds of GB

```

Prefer recording only failed tests or retries in CI.

----------

# Enterprise Configuration

```ts
export default defineConfig({

  use: {

    screenshot: 'only-on-failure',

    video: process.env.CI

      ? 'retain-on-failure'

      : 'off',

    trace: process.env.CI

      ? 'retain-on-failure'

      : 'on-first-retry'

  }

});

```

Typical behavior:

**Local**

-   Fast execution
    
-   Trace only when retrying
    

**CI**

-   Screenshot on failure
    
-   Video on failure
    
-   Trace on failure
    

----------

# Common Mistakes

## Mistake 1

```ts
video: 'on'

```

for a suite with thousands of tests.

This can quickly exhaust disk space.

----------

## Mistake 2

Not recording traces in CI.

Without traces, diagnosing intermittent failures becomes much harder.

----------

## Mistake 3

Using screenshots alone for debugging.

Screenshots show a single moment in time, while traces provide the sequence of events that led to the failure.

----------

## Mistake 4

Committing generated artifacts to Git.

Artifacts such as `playwright-report/`, `test-results/`, videos, and traces should usually be excluded from source control.

----------

# Interview Questions

### Q1. What is the default screenshot mode?

`off`

----------

### Q2. What is the default video mode?

`off`

----------

### Q3. What is the default trace mode?

`off`

----------

### Q4. Which mode is recommended for CI?

```ts
trace: 'retain-on-failure'

```

and

```ts
video: 'retain-on-failure'

```

----------

### Q5. How do you open a trace?

```bash
npx playwright show-trace trace.zip

```

----------

### Q6. Can you capture a screenshot of a locator?

Yes.

```ts
await page.locator('#submit').screenshot({
    path: 'submit.png'
});

```

----------

### Q7. What information does a trace contain?

A trace can include:

-   Playwright API calls
    
-   DOM snapshots
    
-   Screenshots
    
-   Network activity
    
-   Console logs
    
-   Timing information
    
-   Source code references
    
-   Test steps
    

----------

### Q8. Which is better for debugging: video or trace?

In most cases, **trace**. It provides interactive diagnostics and much more context than a video alone.

----------

# Best Practices

-   **Local development:** Use traces sparingly (e.g., `on-first-retry`) to avoid slowing down every run.
    
-   **CI pipelines:** Configure `trace: 'retain-on-failure'`, `video: 'retain-on-failure'`, and `screenshot: 'only-on-failure'`.
    
-   Use locator screenshots when validating specific UI components.
    
-   Customize `outputDir` if your CI pipeline expects artifacts in a particular location.
    
-   Archive HTML reports, traces, screenshots, and videos as CI artifacts so they can be downloaded after failed builds.
    
-   When investigating a failure, start with the **trace**, then use the screenshot and video for additional visual context.
    

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwMjg3NTg1NTldfQ==
-->