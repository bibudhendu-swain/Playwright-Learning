Excellent. This is probably **the most important section** of the entire Playwright configuration. In real projects, **80–90% of the configuration you'll modify lives inside the `use` section**.

----------

# Part 9 – Browser Context Configuration (`use`) – Complete Guide

## What is `use`?

The `use` section defines the **default Browser Context configuration** for every test.

Think of it this way:

```text
Browser
    │
    ├── Browser Context (Configured by `use`)
    │       │
    │       ├── Page 1
    │       ├── Page 2
    │       └── Page 3
    │
    └── Browser Context

```

Every test gets a **fresh Browser Context**, ensuring test isolation.

The `use` section defines how those contexts are created.

----------

# Basic Structure

```ts
export default defineConfig({

    use: {

    }

});

```

Everything we'll discuss belongs inside this block.

----------

# Configuration Hierarchy

The `use` configuration can be applied at different levels:

```
Global Config

↓

Project Config

↓

test.use()

↓

Single Test

```

The closest configuration overrides the previous one.

----------

# 1. `baseURL`

## Purpose

Defines the default URL for navigation.

Instead of:

```ts
await page.goto('https://qa.company.com/login');

```

You can configure:

```ts
use: {

    baseURL: 'https://qa.company.com'

}

```

Then:

```ts
await page.goto('/login');

```

Playwright automatically navigates to:

```
https://qa.company.com/login

```

----------

## Why use `baseURL`?

Without:

```ts
await page.goto('https://qa.company.com/login');

await page.goto('https://qa.company.com/cart');

await page.goto('https://qa.company.com/orders');

```

With:

```ts
await page.goto('/login');

await page.goto('/cart');

await page.goto('/orders');

```

Much cleaner.

----------

## Enterprise Example

```ts
baseURL: process.env.BASE_URL

```

Run:

```bash
BASE_URL=https://qa.company.com npx playwright test

```

Same tests can run against:

-   QA
    
-   UAT
    
-   PROD
    
-   Local
    

without code changes.

----------

# 2. `headless`

Controls browser visibility.

----------

## Headless

```ts
headless: true

```

Browser is invisible.

Fastest execution.

Recommended for:

-   CI
    
-   Nightly execution
    

----------

## Headed

```ts
headless: false

```

Browser opens.

Useful for:

-   debugging
    
-   demonstrations
    

----------

Override:

```bash
npx playwright test --headed

```

----------

# 3. `viewport`

Defines browser window size.

----------

Example

```ts
viewport: {

    width: 1920,

    height: 1080

}

```

Browser:

```
1920 × 1080

```

----------

Laptop Example

```ts
viewport: {

width:1366,

height:768

}

```

----------

Mobile Example

```ts
viewport: {

width:390,

height:844

}

```

----------

## Disable Viewport

```ts
viewport: null

```

Playwright uses the actual browser window size.

Useful for:

```ts
launchOptions: {

    args:['--start-maximized']

}

```

----------

# 4. `ignoreHTTPSErrors`

Suppose:

```
https://qa.company.com

```

has:

```
Self-signed certificate

```

Browser warning:

```
Your connection is not private

```

Configure:

```ts
ignoreHTTPSErrors:true

```

Playwright ignores SSL certificate errors.

Useful for:

-   QA
    
-   Internal environments
    

Avoid using this in production testing unless necessary.

----------

# 5. `locale`

Simulates browser language.

Example

```ts
locale:'fr-FR'

```

Website loads in:

```
French

```

----------

Other examples

```ts
locale:'de-DE'

```

German

----------

```ts
locale:'ja-JP'

```

Japanese

----------

Useful for localization testing.

----------

# 6. `timezoneId`

Simulates timezone.

Example

```ts
timezoneId:'Asia/Kolkata'

```

Browser behaves like India.

----------

Another example

```ts
timezoneId:'America/New_York'

```

Browser behaves like EST/EDT.

Useful when testing:

-   appointments
    
-   calendars
    
-   scheduling
    
-   reminders
    

----------

# 7. `permissions`

Grant browser permissions automatically.

Example

```ts
permissions:['clipboard-read']

```

----------

Multiple

```ts
permissions:[

'clipboard-read',

'clipboard-write',

'notifications'

]

```

Useful when testing:

-   Notifications
    
-   Clipboard
    
-   Geolocation
    
-   Camera
    
-   Microphone
    

----------

# 8. `geolocation`

Simulates GPS.

Example

```ts
geolocation:{

latitude:28.6139,

longitude:77.2090

}

```

Browser location:

```
New Delhi

```

Remember:

```ts
permissions:['geolocation']

```

must also be granted.

----------

# 9. `colorScheme`

Dark mode testing.

```ts
colorScheme:'dark'

```

Website loads in:

🌙 Dark Mode

----------

Light mode

```ts
colorScheme:'light'

```

----------

No preference

```ts
colorScheme:'no-preference'

```

----------

# 10. `userAgent`

Override browser user agent.

Example

```ts
userAgent:'Automation Test'

```

Useful for:

-   analytics testing
    
-   mobile simulation
    
-   custom identification
    

----------

# 11. `storageState`

Probably one of the most important Playwright features.

Suppose login takes:

```
20 seconds

```

Instead of logging in every test:

```ts
storageState:'state.json'

```

Playwright loads:

-   cookies
    
-   local storage
    
-   session storage (where applicable via storage state)
    

User is already logged in.

----------

Generate:

```bash
npx playwright codegen --save-storage=state.json

```

Reuse:

```ts
storageState:'state.json'

```

Enterprise frameworks often use a dedicated setup project to generate storage state automatically.

----------

# 12. `httpCredentials`

Basic Authentication.

Example

```ts
httpCredentials:{

username:'admin',

password:'password'

}

```

Playwright automatically authenticates against HTTP Basic Auth prompts.

----------

# 13. `proxy`

Example

```ts
proxy:{

server:'http://proxy.company.com:8080'

}

```

Useful inside corporate networks.

----------

Authenticated proxy

```ts
proxy:{

server:'http://proxy.company.com:8080',

username:'user',

password:'password'

}

```

----------

# 14. `acceptDownloads`

Example

```ts
acceptDownloads:true

```

Downloaded files are accepted automatically.

Common for:

-   invoices
    
-   reports
    
-   PDFs
    
-   CSV exports
    

----------

# 15. `offline`

Simulate network loss.

```ts
offline:true

```

Useful for testing:

-   offline mode
    
-   Progressive Web Apps (PWAs)
    
-   reconnect logic
    

----------

# 16. `javaScriptEnabled`

Disable JavaScript.

```ts
javaScriptEnabled:false

```

Useful for accessibility and progressive enhancement testing.

----------

# 17. `extraHTTPHeaders`

Automatically send headers.

```ts
extraHTTPHeaders:{

'X-Test':'Automation',

'Environment':'QA'

}

```

Useful for:

-   feature flags
    
-   tenant selection
    
-   API gateways
    
-   custom authentication
    

----------

# 18. `serviceWorkers`

Control service workers.

```ts
serviceWorkers:'block'

```

Useful when:

-   debugging caching issues
    
-   ensuring fresh resources
    
-   preventing service worker interference
    

----------

# Complete Enterprise Configuration

```ts
export default defineConfig({

use:{

baseURL:process.env.BASE_URL,

headless:true,

viewport:{

width:1920,

height:1080

},

ignoreHTTPSErrors:true,

locale:'en-US',

timezoneId:'Asia/Kolkata',

permissions:['clipboard-read'],

colorScheme:'light',

storageState:'playwright/.auth/user.json',

acceptDownloads:true,

trace:'retain-on-failure',

video:'retain-on-failure',

screenshot:'only-on-failure'

}

});

```

This is very close to what you'll see in many enterprise Playwright frameworks.

----------

# Overriding `use`

Global:

```ts
use:{

locale:'en-US'

}

```

Project:

```ts
projects:[

{

name:'French',

use:{

locale:'fr-FR'

}

}

]

```

Single suite:

```ts
test.use({

locale:'ja-JP'

});

```

The suite runs with the Japanese locale while the rest of the project uses the global setting.

----------

# Common Mistakes

### ❌ Hardcoding URLs

```ts
page.goto('https://qa.company.com');

```

Prefer:

```ts
baseURL

```

----------

### ❌ Logging in before every test

Use:

```ts
storageState

```

to reuse an authenticated state when appropriate.

----------

### ❌ Using `headless:false` in CI

Headless execution is generally faster and more suitable for CI pipelines.

----------

### ❌ Ignoring SSL errors in production tests

Reserve:

```ts
ignoreHTTPSErrors:true

```

for environments where self-signed or internal certificates are expected.

----------

### ❌ Forgetting Geolocation Permission

```ts
geolocation:{...}

```

without:

```ts
permissions:['geolocation']

```

The application may still deny location access.

----------

# Interview Questions

### Q1. What is the purpose of the `use` section?

It defines the default browser context configuration applied to all tests unless overridden.

----------

### Q2. What does `baseURL` do?

It allows relative URLs in methods such as `page.goto()`, reducing duplication and simplifying environment changes.

----------

### Q3. What is `storageState`?

It stores and restores authentication-related browser state (such as cookies and local storage) so tests can start in a logged-in state.

----------

### Q4. How do you test dark mode?

```ts
colorScheme:'dark'

```

----------

### Q5. How do you simulate another country?

Configure:

-   `geolocation`
    
-   `permissions: ['geolocation']`
    
-   `locale`
    
-   `timezoneId`
    

as needed to match the target region.

----------

### Q6. How do you disable JavaScript?

```ts
javaScriptEnabled:false

```

----------

### Q7. Can `use` be overridden?

Yes. It can be overridden at the project, `test.use()`, or test level.

----------

### Q8. Which `use` property is most commonly used in enterprise frameworks?

Typically:

-   `baseURL`
    
-   `storageState`
    
-   `trace`
    
-   `video`
    
-   `screenshot`
    
-   `viewport`
    

----------

# Best Practices

-   **Always use `baseURL`** instead of hardcoding hostnames in tests.
    
-   **Use `storageState`** to avoid repeated login flows and speed up execution.
    
-   Keep **headless mode enabled in CI** and use `--headed` locally when debugging.
    
-   Standardize viewport sizes across the suite unless you're intentionally testing responsive layouts.
    
-   Configure locale, timezone, and geolocation only for tests that require them.
    
-   Store authentication files (e.g., `playwright/.auth/user.json`) outside your test folders and regenerate them through a setup project.
    
-   Use `extraHTTPHeaders` for environment-specific headers or feature flags instead of modifying application code.
    

----------

# 💡 Interview Tip

A common interview question is:

> **What's the difference between `launchOptions`, `Browser`, and `Browser Context (use)`?**

A concise answer is:

-   **Browser (`browser`)**: The actual browser process (Chromium, Firefox, WebKit).
    
-   **`launchOptions`**: Controls how the browser process starts (e.g., CLI arguments, slow motion, browser channel, executable path).
    
-   **`use` (Browser Context)**: Controls the isolated environment each test runs in (viewport, locale, storage state, permissions, geolocation, etc.).
    

Think of it as:

```text
Browser Process
        │
        │  (Configured by launchOptions)
        ▼
Chromium
        │
        ├───────────────┐
        ▼               ▼
Browser Context A   Browser Context B
(Configured by use) (Configured by use)
        │               │
      Page            Page

```

This distinction is frequently asked in Playwright interviews because it demonstrates an understanding of Playwright's architecture.

----------
   
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTU0NTk5MzM0XX0=
-->