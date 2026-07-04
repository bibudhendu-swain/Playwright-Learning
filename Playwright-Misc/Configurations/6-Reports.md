# Part 6 – Reporters (`reporter`) – Complete Guide

One of Playwright's strengths is its flexible reporting system. Reports help you understand:

-   Which tests passed or failed
    
-   Execution duration
    
-   Error messages
    
-   Screenshots, videos, and traces
    
-   Trends across builds
    
-   CI/CD integration
    

Playwright supports multiple built-in reporters and allows custom reporters.

----------

# What is a Reporter?

A **reporter** determines **how test results are displayed or stored**.

For example:

```bash
npx playwright test

```

might print a console summary, while another reporter generates:

-   HTML reports
    
-   JSON files
    
-   JUnit XML
    
-   GitHub annotations
    
-   Blob reports for report merging
    

The same test execution can produce multiple reports simultaneously.

----------

# Reporter Configuration

Basic configuration:

```ts
export default defineConfig({
  reporter: 'html'
});

```

----------

## Multiple Reporters

Playwright supports multiple reporters.

Example:

```ts
export default defineConfig({

  reporter: [

    ['list'],

    ['html'],

    ['json', {
      outputFile: 'results.json'
    }]

  ]

});

```

This generates:

-   Console output
    
-   HTML report
    
-   JSON report
    

----------

# Built-in Reporters

Playwright currently provides these built-in reporters:

Reporter

Purpose

Typical Use

`list`

Detailed console output

Local development

`line`

One updating line in terminal

CI

`dot`

Compact progress output

CI

`html`

Interactive HTML report

Debugging, sharing

`json`

Machine-readable JSON

Custom dashboards

`junit`

JUnit XML

Jenkins, Azure DevOps

`blob`

Binary report

Merge sharded reports

`github`

GitHub Actions annotations

GitHub Actions

----------

# 1. List Reporter

## Configuration

```ts
reporter: 'list'

```

----------

## Output

```text
✓ Login

✓ Search

✓ Checkout

✗ Payment

```

Shows every test with its status.

----------

## Best For

-   Local execution
    
-   Debugging
    
-   Developers
    

----------

# 2. Line Reporter

## Configuration

```ts
reporter: 'line'

```

Output continuously updates on one line.

Example:

```text
Running 248/1000

```

instead of printing every completed test.

----------

## Best For

Large suites in CI where minimizing console output is helpful.

----------

# 3. Dot Reporter

Configuration

```ts
reporter: 'dot'

```

Output

```text
................F............

```

Legend:

```text
.

= Passed

F

= Failed

```

----------

## Best For

Very large suites.

Minimal console output.

----------

# Comparison

Reporter

Console Size

List

Large

Line

Medium

Dot

Very Small

----------

# 4. HTML Reporter

Probably the most frequently used reporter.

----------

## Configuration

```ts
reporter: 'html'

```

----------

Run

```bash
npx playwright test

```

Open report

```bash
npx playwright show-report

```

----------

## Output Contains

-   Passed tests
    
-   Failed tests
    
-   Stack trace
    
-   Screenshots
    
-   Trace
    
-   Video
    
-   Attachments
    
-   Execution time
    
-   Retry information
    

----------

## Custom Folder

```ts
reporter: [

['html', {

outputFolder: 'playwright-report',

open: 'never'

}]

]

```

----------

Options

Option

Description

outputFolder

Report location

open

always / never / on-failure

----------

Open Modes

```ts
open: 'always'

```

Always opens report.

----------

```ts
open: 'on-failure'

```

Only after failures.

----------

```ts
open: 'never'

```

Never opens automatically.

Ideal for CI.

----------

# 5. JSON Reporter

Useful for dashboards.

Configuration

```ts
reporter: [

['json', {

outputFile: 'results.json'

}]

]

```

Generates:

```text
results.json

```

Contains:

-   Test name
    
-   Duration
    
-   Status
    
-   Retry count
    
-   Errors
    

Machine-readable.

----------

# 6. JUnit Reporter

Most enterprise CI systems understand JUnit XML.

Configuration

```ts
reporter: [

['junit', {

outputFile: 'results.xml'

}]

]

```

Generates

```text
results.xml

```

Compatible with

-   Jenkins
    
-   Azure DevOps
    
-   Bamboo
    
-   GitLab
    
-   TeamCity
    

----------

# 7. Blob Reporter

Introduced for report merging.

Configuration

```ts
reporter: 'blob'

```

Generates

```text
report.blob

```

Blob reports are useful when:

```text
Shard 1

Shard 2

Shard 3

```

Later merged into one report.

Common in distributed CI.

----------

# 8. GitHub Reporter

Configuration

```ts
reporter: 'github'

```

Instead of plain logs,

GitHub Actions displays annotations directly in the workflow summary.

Example

```text
❌ login.spec.ts

Line 42

```

becomes a clickable annotation.

----------

# Multiple Reporters Example

One of the most common enterprise setups:

```ts
reporter: [

['list'],

['html'],

['junit', {

outputFile: 'results.xml'

}]

]

```

Produces:

-   Nice local console output
    
-   HTML report
    
-   Jenkins XML
    

----------

# CI Example

```ts
reporter: process.env.CI

? [

['dot'],

['junit', {

outputFile: 'results.xml'

}]

]

: [

['list'],

['html']

]

```

Local:

```text
List

HTML

```

CI:

```text
Dot

JUnit

```

----------

# Reporter vs show-report

Configuration:

```ts
reporter: 'html'

```

Generates report.

To view:

```bash
npx playwright show-report

```

`show-report` does **not** generate the report—it only opens an existing HTML report.

----------

# Reporter Options

Example

```ts
reporter: [

['html', {

outputFolder: 'reports',

open: 'never'

}],

['json', {

outputFile: 'reports/result.json'

}]

]

```

----------

# Allure Integration

Although not built into Playwright, Allure is a popular reporting solution.

Install:

```bash
npm install -D allure-playwright

```

Configuration

```ts
reporter: [

['line'],

['allure-playwright']

]

```

Generate report

```bash
allure generate allure-results --clean

```

Open

```bash
allure open

```

----------

# Azure DevOps Example

```ts
reporter: [

['junit', {

outputFile: 'test-results.xml'

}]
]

```

Azure DevOps can publish the XML file as test results.

----------

# Jenkins Example

```ts
reporter: [

['junit', {

outputFile: 'junit.xml'

}]
]

```

Jenkins JUnit plugin reads the XML automatically.

----------

# GitHub Actions Example

```ts
reporter: [

['github'],

['html']

]

```

Produces:

-   Workflow annotations
    
-   Downloadable HTML report
    

----------

# Common Mistakes

## Mistake 1

Using only:

```ts
reporter: 'dot'

```

during local debugging.

The HTML report is often more useful for investigating failures.

----------

## Mistake 2

Committing large HTML report folders to source control.

These are generated artifacts and should typically be ignored (e.g., via `.gitignore`).

----------

## Mistake 3

Expecting `show-report` to generate a report.

It only opens an existing one.

----------

## Mistake 4

Generating JUnit XML without configuring the CI pipeline to publish or consume it.

----------

# Enterprise Configuration

```ts
export default defineConfig({

reporter: process.env.CI

? [

['dot'],

['junit', {

outputFile: 'test-results/results.xml'

}],

['blob']

]

: [

['list'],

['html', {

open: 'never'

}]

]

});

```

This provides:

-   Minimal CI console logs
    
-   JUnit XML for CI integration
    
-   Blob reports for sharding
    
-   HTML reports for local debugging
    

----------

# Interview Questions

### Q1. Can Playwright use multiple reporters?

Yes.

```ts
reporter: [

['list'],

['html'],

['json']

]

```

----------

### Q2. Which reporter generates XML?

`junit`

----------

### Q3. Which reporter generates JSON?

`json`

----------

### Q4. Which reporter produces the interactive report?

`html`

----------

### Q5. Which reporter is best for Jenkins?

`junit`

----------

### Q6. Which reporter is best for GitHub Actions?

`github`

----------

### Q7. What does the Blob reporter do?

It generates a binary report that can be merged later, making it ideal for sharded or distributed test execution.

----------

### Q8. How do you open an HTML report?

```bash
npx playwright show-report

```

----------

### Q9. Can `show-report` create a report?

No. It only opens a report that has already been generated by the HTML reporter.

----------

# Best Practices

-   **Local development:** Use `list` + `html` for readable console output and easy debugging.
    
-   **CI pipelines:** Use `dot` or `line` to keep logs concise, plus `junit` for test result publishing.
    
-   **Sharded execution:** Include the `blob` reporter so reports can be merged later.
    
-   **GitHub Actions:** Add the `github` reporter for inline annotations.
    
-   Store generated reports in dedicated directories (e.g., `playwright-report/` or `test-results/`) and exclude them from version control unless you have a specific archival requirement.
    
-   Generate HTML reports for failed builds and make them available as downloadable CI artifacts.
    

----------

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3MzQyNDIyMDhdfQ==
-->