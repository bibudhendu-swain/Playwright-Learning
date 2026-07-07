This chapter is aimed at **Automation Architects, Framework Developers, and Senior Automation Engineers**.

Most teams are happy with HTML or Allure reports. However, enterprise organizations often require custom reporting such as:

-   Store results in a database
    
-   Send Slack notifications
    
-   Send Microsoft Teams messages
    
-   Publish execution metrics
    
-   Generate custom HTML/PDF reports
    
-   Push results to internal dashboards
    
-   Integrate with Test Management tools
    

Playwright provides a **Reporter API** that allows you to build your own reporter.

----------

# Part 20 – Reporting

# Chapter 5 – Custom Reporters

----------

# Introduction

A **Reporter** is a component that listens to test execution events.

Instead of only generating HTML or JUnit reports,

you can generate:

-   Excel Reports
    
-   PDF Reports
    
-   Database Records
    
-   REST API Calls
    
-   Slack Notifications
    
-   Teams Notifications
    
-   Email Reports
    
-   Custom Dashboards
    

Think of it as an event listener for the Playwright test runner.

----------

# Reporter Architecture

```text
Playwright Test Runner

↓

Execution Events

↓

Reporter API

↓

Custom Reporter

↓

Database

Slack

Email

Dashboard

```

Every significant event during test execution is exposed through the Reporter API.

----------

# Reporter Lifecycle

The reporter receives callbacks in a predictable order.

```text
Execution Starts

↓

onBegin()

↓

onTestBegin()

↓

onStepBegin()

↓

onStepEnd()

↓

onTestEnd()

↓

onEnd()

```

----------

# Creating a Reporter

Create a file:

```text
reporters/

└── CustomReporter.ts

```

Basic implementation

```typescript
import {
    Reporter,
    FullConfig,
    Suite
} from "@playwright/test/reporter";

class CustomReporter implements Reporter {

    onBegin(config: FullConfig, suite: Suite) {

        console.log("Execution Started");

    }

}

export default CustomReporter;

```

----------

# Registering the Reporter

```typescript
import { defineConfig } from "@playwright/test";

export default defineConfig({

    reporter: [

        ["./reporters/CustomReporter.ts"]

    ]

});

```

Playwright loads your reporter automatically.

----------

# Reporter Lifecycle Methods

Method

Purpose

onBegin()

Test execution starts

onTestBegin()

Individual test starts

onStepBegin()

Test step starts

onStepEnd()

Test step ends

onStdOut()

Standard output received

onStdErr()

Standard error received

onError()

Framework error

onTestEnd()

Individual test finishes

onEnd()

Entire execution finishes

----------

# onBegin()

Called once.

```text
Execution

↓

Started

```

Example

```typescript
onBegin(config, suite){

    console.log(

        `Running ${suite.allTests().length} tests`

    );

}

```

Typical uses:

-   Create report folders
    
-   Initialize database connections
    
-   Start timers
    
-   Send "Execution Started" notifications
    

----------

# onTestBegin()

Called before every test.

```typescript
import { TestCase } from "@playwright/test/reporter";

onTestBegin(test: TestCase){

    console.log(

        test.title

    );

}

```

Example output

```text
Login Test

Create Customer

Delete Order

```

Useful for:

-   Logging
    
-   Live dashboards
    
-   Progress tracking
    

----------

# onStepBegin()

Called before every Playwright step.

```typescript
import { TestStep } from "@playwright/test/reporter";

onStepBegin(test, result, step){

    console.log(

        step.title

    );

}

```

Example

```text
Fill Username

Click Login

Verify Dashboard

```

----------

# onStepEnd()

Called after each step.

Example

```typescript
onStepEnd(test, result, step){

    console.log(

        `${step.title} completed`

    );

}

```

Useful for

-   Timing individual steps
    
-   Custom logs
    
-   Performance metrics
    

----------

# onStdOut()

Capture application output.

```typescript
onStdOut(chunk){

    console.log(

        chunk.toString()

    );

}

```

Useful when applications print useful diagnostic information.

----------

# onStdErr()

Capture errors.

```typescript
onStdErr(chunk){

    console.error(

        chunk.toString()

    );

}

```

----------

# onError()

Framework-level errors.

```typescript
onError(error){

    console.error(

        error.message

    );

}

```

Examples:

-   Reporter failure
    
-   Internal Playwright error
    
-   Unexpected framework exception
    

----------

# onTestEnd()

Most useful lifecycle method.

```typescript
onTestEnd(test, result){

    console.log(

        test.title,

        result.status

    );

}

```

Possible statuses

```text
passed

failed

timedOut

interrupted

skipped

```

----------

# onEnd()

Called once after execution finishes.

```typescript
onEnd(result){

    console.log(

        result.status

    );

}

```

Typical uses

-   Generate reports
    
-   Send email
    
-   Upload results
    
-   Close database connections
    

----------

# Custom Console Reporter

Example

```typescript
class ConsoleReporter {

    onTestEnd(test, result){

        console.log(

            `${test.title} : ${result.status}`

        );

    }

}

```

Output

```text
Login : passed

Checkout : failed

```

----------

# Creating a JSON Report

```typescript
class JsonReporter {

    private results = [];

    onTestEnd(test, result){

        this.results.push({

            test: test.title,

            status: result.status

        });

    }

    async onEnd(){

        await fs.promises.writeFile(

            "results.json",

            JSON.stringify(this.results, null, 2)

        );

    }

}

```

Useful for custom dashboards.

----------

# Database Reporter

Workflow

```text
Test Ends

↓

Reporter

↓

Database

↓

Dashboard

```

Example

```typescript
onTestEnd(test, result){

    database.insert({

        name: test.title,

        status: result.status

    });

}

```

Real implementations should batch writes or use asynchronous queues to avoid slowing down test execution.

----------

# Slack Notification

Workflow

```text
Execution Ends

↓

Reporter

↓

Slack Webhook

↓

Team Notification

```

Example

```typescript
async onEnd(){

    await fetch(slackWebhook,{

        method:"POST",

        body:JSON.stringify({

            text:"Execution Completed"

        })

    });

}

```

----------

# Microsoft Teams Notification

Very similar.

```text
Execution

↓

Webhook

↓

Teams Channel

```

----------

# Email Reporter

Workflow

```text
Execution Ends

↓

Reporter

↓

SMTP

↓

Email

↓

QA Team

```

Example summary

```text
100 Tests

98 Passed

2 Failed

```

----------

# Publishing Results to REST API

Many organizations have internal dashboards.

Workflow

```text
Playwright

↓

Reporter

↓

POST /results

↓

Dashboard

```

Example

```typescript
await fetch(

    dashboardUrl,

    {

        method:"POST",

        body:JSON.stringify(results)

    }

);

```

----------

# Generating Custom HTML

Instead of HTML Reporter,

your reporter can generate

```text
report.html

```

Example

```typescript
const html = `

<h1>Execution Summary</h1>

`;

await fs.promises.writeFile(

"report.html",

html

);

```

----------

# Measuring Execution Time

```typescript
private start = Date.now();

async onEnd(){

    const duration =

    Date.now() - this.start;

}

```

Useful for dashboards.

----------

# Combining Reporters

Enterprise projects usually combine reporters.

```typescript
reporter:[

["html"],

["junit"],

["./reporters/SlackReporter.ts"],

["./reporters/DatabaseReporter.ts"]

]

```

One execution produces:

```text
HTML

+

JUnit

+

Slack

+

Database

```

----------

# Enterprise Architecture

```text
Playwright

↓

Reporter API

↓

Custom Reporters

├── Slack

├── Teams

├── Email

├── Dashboard

├── Database

└── HTML

```

Each reporter has a single responsibility.

----------

# Suggested Folder Structure

```text
reporters/

├── CustomReporter.ts

├── SlackReporter.ts

├── TeamsReporter.ts

├── EmailReporter.ts

├── DatabaseReporter.ts

├── HtmlReporter.ts

└── JsonReporter.ts

```

----------

# Best Practices

-   Keep reporters lightweight and non-blocking.
    
-   Separate notification logic from report generation.
    
-   Batch database writes for large executions.
    
-   Handle network failures gracefully so reporting issues don't affect test execution.
    
-   Use multiple reporters rather than building one reporter that does everything.
    

----------

# Common Mistakes

### ❌ Performing Heavy Work in `onTestEnd()`

Every test triggers this method.

Avoid expensive:

-   Database writes
    
-   File operations
    
-   Network requests
    

Prefer batching and processing in `onEnd()`.

----------

### ❌ Throwing Exceptions Inside Reporters

Reporter failures should not crash the test execution.

Always catch and log errors.

```typescript
try{

// reporting

}catch(error){

console.error(error);

}

```

----------

### ❌ Building One Huge Reporter

Instead of

```text
One Reporter

↓

Everything

```

Prefer

```text
Slack Reporter

Database Reporter

Email Reporter

HTML Reporter

```

Smaller reporters are easier to maintain.

----------

### ❌ Blocking the Test Runner

Avoid synchronous file or network operations inside lifecycle methods.

Use asynchronous operations wherever possible.

----------

# Interview Questions

### Q1. What is the purpose of a custom reporter?

A custom reporter listens to Playwright execution events and generates organization-specific outputs such as dashboards, database records, notifications, or custom reports.

----------

### Q2. Which lifecycle method is called once before test execution starts?

`onBegin()`

----------

### Q3. Which lifecycle method is commonly used to collect test results?

`onTestEnd()`

----------

### Q4. Why should expensive operations be avoided in `onTestEnd()`?

Because it runs for every test. Heavy operations can significantly slow down large test suites.

----------

### Q5. Why are multiple small reporters better than one large reporter?

They follow the Single Responsibility Principle, making them easier to develop, test, maintain, and reuse.

----------

# Summary

Playwright's Reporter API provides complete control over how test execution results are processed and presented. By implementing lifecycle callbacks such as `onBegin()`, `onTestEnd()`, and `onEnd()`, teams can build custom integrations for dashboards, databases, notifications, and reporting systems. Keeping reporters modular, asynchronous, and focused on a single responsibility results in scalable, enterprise-grade reporting solutions.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbNjgzODMyNTUzXX0=
-->