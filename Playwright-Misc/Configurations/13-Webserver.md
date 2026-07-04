# Part 13 – `webServer` Configuration (Complete Guide)

One of the most useful Playwright features is its ability to **automatically start your application before running tests**.

Instead of remembering to do this manually:

```text
Terminal 1
-----------
npm run dev

Terminal 2
-----------
npx playwright test

```

Playwright can do it for you.

----------

# What is `webServer`?

The `webServer` configuration tells Playwright:

> **"Start this application, wait until it's ready, then execute the tests."**

After the tests complete, Playwright automatically stops the server (unless it reuses an existing one).

----------

# Basic Flow

```text
Start Web Server

↓

Wait Until Ready

↓

Run Tests

↓

Stop Server

```

----------

# Basic Configuration

```ts
export default defineConfig({

  webServer: {

    command: 'npm run dev',

    url: 'http://localhost:3000'

  }

});

```

Playwright executes:

```bash
npm run dev

```

Waits until:

```text
http://localhost:3000

```

responds successfully.

Then starts the tests.

----------

# Important Properties

Property

Purpose

`command`

Command used to start the application

`url`

URL Playwright waits for

`reuseExistingServer`

Reuse already-running server

`timeout`

Maximum startup wait time

`cwd`

Working directory

`env`

Environment variables

`stdout`

Console output behavior

`stderr`

Error output behavior

`gracefulShutdown`

Shutdown behavior

----------

# 1. `command`

The startup command.

React

```ts
command: 'npm run dev'

```

----------

Angular

```ts
command:'ng serve'

```

----------

Next.js

```ts
command:'npm run dev'

```

----------

Vite

```ts
command:'npm run dev'

```

----------

Production Build

```ts
command:'npm start'

```

----------

# 2. `url`

The URL Playwright checks.

```ts
url:'http://localhost:3000'

```

Playwright continuously polls:

```text
GET /

↓

200 OK

↓

Start Tests

```

----------

If the server never starts:

```text
Timeout

↓

Execution Fails

```

----------

# 3. `reuseExistingServer`

Probably the most useful property.

Suppose you're already running:

```bash
npm run dev

```

Without:

```ts
reuseExistingServer:true

```

Playwright attempts to start another server.

This often fails because the port is already in use.

----------

Configuration

```ts
reuseExistingServer:true

```

Execution

```text
Server Running?

↓

Yes

↓

Reuse

↓

Run Tests

```

----------

Typical Enterprise Configuration

```ts
reuseExistingServer:!process.env.CI

```

Local

```text
Reuse Existing

```

CI

```text
Always Start Fresh

```

----------

# 4. `timeout`

Default startup timeout:

```text
60 seconds

```

Increase it for slow applications.

```ts
timeout:120000

```

Now Playwright waits:

```text
2 Minutes

```

before failing.

----------

# 5. `cwd`

Run command from another folder.

Project

```text
repo/

frontend/

backend/

tests/

```

Configuration

```ts
cwd:'frontend'

```

Playwright executes

```bash
cd frontend

npm run dev

```

----------

# 6. `env`

Pass environment variables.

```ts
env:{

NODE_ENV:'test',

PORT:'3000'

}

```

Useful for:

-   QA
    
-   Development
    
-   Mock environments
    

----------

# 7. `stdout`

Options

```ts
stdout:'pipe'

```

Capture output.

----------

```ts
stdout:'ignore'

```

Hide output.

----------

# 8. `stderr`

Same options.

Useful when startup logs are too noisy.

----------

# 9. `gracefulShutdown`

Controls how Playwright stops the application.

Useful for:

-   Docker
    
-   Linux
    
-   Long-running Node.js servers
    

Most users keep the default behavior.

----------

# Complete Example

```ts
export default defineConfig({

webServer:{

command:'npm run dev',

url:'http://localhost:3000',

reuseExistingServer:!process.env.CI,

timeout:120000

}

});

```

----------

# React Example

```ts
webServer:{

command:'npm start',

url:'http://localhost:3000',

reuseExistingServer:true

}

```

----------

# Angular Example

```ts
webServer:{

command:'ng serve',

url:'http://localhost:4200'

}

```

----------

# Vite Example

```ts
webServer:{

command:'npm run dev',

url:'http://localhost:5173'

}

```

----------

# Next.js Example

```ts
webServer:{

command:'npm run dev',

url:'http://localhost:3000'

}

```

----------

# Running Production Build

Instead of

```text
Development Server

```

Run

```ts
command:'npm start'

```

Useful for production smoke testing.

----------

# Multiple Web Servers

Modern applications often have:

```text
Frontend

Backend

API

Authentication Server

```

Playwright supports multiple servers.

Configuration

```ts
webServer:[

{

command:'npm run backend',

url:'http://localhost:5000'

},

{

command:'npm run frontend',

url:'http://localhost:3000'

}

]

```

Execution

```text
Backend

↓

Frontend

↓

Tests

```

----------

# Monorepo Example

```text
repo/

apps/

frontend/

backend/

tests/

```

Configuration

```ts
webServer:[

{

cwd:'apps/backend',

command:'npm run dev',

url:'http://localhost:5000'

},

{

cwd:'apps/frontend',

command:'npm run dev',

url:'http://localhost:3000'

}

]

```

----------

# Backend + Frontend Flow

```text
Backend

↓

Ready

↓

Frontend

↓

Ready

↓

Playwright

↓

Tests

```

----------

# Enterprise Example

```ts
export default defineConfig({

webServer:[

{

command:'npm run api',

url:'http://localhost:5000',

reuseExistingServer:!process.env.CI

},

{

command:'npm run web',

url:'http://localhost:3000',

reuseExistingServer:!process.env.CI

}

]

});

```

Very common.

----------

# Web Server vs `baseURL`

A common interview question.

`webServer`

```text
Starts Application

```

`baseURL`

```text
Navigates Application

```

Example

```ts
webServer:{

command:'npm run dev',

url:'http://localhost:3000'

},

use:{

baseURL:'http://localhost:3000'

}

```

----------

# Local Development

Developer already running

```bash
npm run dev

```

Playwright

↓

Reuse

↓

Run Tests

Fast.

----------

# CI Flow

```text
Pipeline

↓

Checkout

↓

Install

↓

Start Server

↓

Run Tests

↓

Stop Server

```

Everything automatic.

----------

# Common Mistakes

## ❌ Forgetting `reuseExistingServer`

Every test execution tries to start another server.

Result

```text
Port Already In Use

```

----------

## ❌ Wrong URL

Application

```text
localhost:5173

```

Config

```text
localhost:3000

```

Playwright waits forever until timeout.

----------

## ❌ Short Timeout

Large Angular app

Needs

```text
90 Seconds

```

Config

```text
30 Seconds

```

Failure.

----------

## ❌ Using `sleep`

Instead of

```text
wait 60 seconds

```

Use

```text
url

```

Playwright waits intelligently until the application is actually ready.

----------

# Interview Questions

### Q1. What does `webServer` do?

It starts one or more applications before the tests run and waits until they are ready.

----------

### Q2. What does `reuseExistingServer` do?

If the application is already running, Playwright reuses it instead of starting a new instance.

----------

### Q3. Why use:

```ts
reuseExistingServer:!process.env.CI

```

Local developers often already have the application running.

CI should always start a clean server.

----------

### Q4. Can Playwright start multiple applications?

Yes.

Example:

-   Backend
    
-   Frontend
    
-   Authentication
    

All before execution.

----------

### Q5. What happens if the server never starts?

Playwright waits until the configured timeout and then fails the test run.

----------

### Q6. What is the difference between `webServer.url` and `use.baseURL`?

`webServer.url`

`use.baseURL`

Health check used to determine when the application is ready

Base address used by `page.goto('/path')` and similar navigation methods

Used before tests start

Used during test execution

----------

### Q7. Does `webServer` automatically stop the application?

Yes. Playwright manages the lifecycle of the server it starts unless you're reusing an existing server.

----------

# Best Practices

-   ✅ Use `reuseExistingServer: !process.env.CI` for a smooth local development experience.
    
-   ✅ Always pair `webServer.url` with `use.baseURL` so startup and navigation point to the same application.
    
-   ✅ Increase `timeout` for applications with slow startup times.
    
-   ✅ Use multiple `webServer` entries for modern frontend/backend architectures.
    
-   ✅ Prefer health checks (`url`) over fixed delays or sleeps.
    
-   ✅ Set `cwd` when your application lives in a subdirectory (common in monorepos).
    

----------

# ⭐ Enterprise Configuration (Recommended)

```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({

  use: {
    baseURL: 'http://localhost:3000'
  },

  webServer: [
    {
      command: 'npm run api',
      cwd: './apps/api',
      url: 'http://localhost:5000',
      reuseExistingServer: !process.env.CI,
      timeout: 120000
    },
    {
      command: 'npm run web',
      cwd: './apps/web',
      url: 'http://localhost:3000',
      reuseExistingServer: !process.env.CI,
      timeout: 120000
    }
  ]

});

```

This pattern is commonly used for monorepos or applications with separate frontend and backend services.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2NTc0NjkxMDFdfQ==
-->