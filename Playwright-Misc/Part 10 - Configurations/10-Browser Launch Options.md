# Part 10 – Browser Launch Options (`launchOptions`, `channel`, `args`, `slowMo`, `executablePath`) – Complete Guide

> **One of the most common Playwright interview questions is:**
> 
> **What is the difference between `use` and `launchOptions`?**
> 
> **Answer:**
> 
> -   `use` configures the **Browser Context**.
>     
> -   `launchOptions` configures the **Browser Process**.
>     

Think of it like this:

```text
Operating System
       │
       ▼
Browser Process  ← launchOptions
       │
       ├─────────────┐
       ▼             ▼
Browser Context   Browser Context ← use
       │
       ▼
Page

```

----------

# What are `launchOptions`?

`launchOptions` are passed to:

```ts
chromium.launch({...})

```

internally by Playwright.

They control **how the browser starts**, not how individual tests behave.

----------

# Where is `launchOptions` Used?

```ts
export default defineConfig({

    use: {

        launchOptions: {

        }

    }

});

```

Everything in this chapter belongs inside:

```ts
use.launchOptions

```

----------

# Browser Context vs Browser Launch

Browser Launch

Browser Context

Starts Chrome

Creates new context

One per worker

One per test

launchOptions

use

Browser-wide

Test-specific

----------

# 1. `channel`

## Purpose

Specify which installed browser should be launched.

Default:

```text
Playwright Chromium

```

----------

## Chrome

```ts
use: {

launchOptions: {

channel:'chrome'

}

}

```

Launches:

```text
Google Chrome

```

instead of Playwright Chromium.

----------

## Microsoft Edge

```ts
channel:'msedge'

```

----------

## Chrome Beta

```ts
channel:'chrome-beta'

```

----------

## Chrome Dev

```ts
channel:'chrome-dev'

```

----------

## Chrome Canary

```ts
channel:'chrome-canary'

```

----------

## Microsoft Edge Beta

```ts
channel:'msedge-beta'

```

----------

## Why Use Chrome Instead of Chromium?

Chromium

```text
Open Source

```

Chrome

```text
Real production browser

```

Some enterprise features are available only in Chrome.

Examples:

-   Corporate policies
    
-   Enterprise extensions
    
-   Certain DRM capabilities
    
-   Company-managed browsers
    

----------

# 2. `args`

Pass browser command-line arguments.

Example

```ts
launchOptions:{

args:[

'--start-maximized'

]

}

```

Browser starts maximized.

----------

## Multiple Arguments

```ts
args:[

'--start-maximized',

'--disable-web-security',

'--disable-dev-shm-usage'

]

```

----------

## Common Chrome Arguments

### Disable Notifications

```ts
args:['--disable-notifications']

```

----------

### Ignore Certificate Errors

```ts
args:['--ignore-certificate-errors']

```

----------

### Disable GPU

```ts
args:['--disable-gpu']

```

Mostly useful on some CI environments.

----------

### Disable Extensions

```ts
args:['--disable-extensions']

```

----------

### Incognito

```ts
args:['--incognito']

```

----------

### Disable Popup Blocking

```ts
args:['--disable-popup-blocking']

```

----------

### Fullscreen

```ts
args:['--start-fullscreen']

```

----------

### Remote Debugging

```ts
args:['--remote-debugging-port=9222']

```

Useful when attaching external debugging tools.

----------

# 3. `slowMo`

Probably the easiest setting.

Example

```ts
slowMo:500

```

Every Playwright action pauses:

```text
500 milliseconds

```

Execution

```text
click()

↓

500ms

↓

fill()

↓

500ms

↓

press()

↓

500ms

```

----------

## Why Use `slowMo`?

Great for:

-   demonstrations
    
-   debugging
    
-   learning Playwright
    

Not recommended for CI because it intentionally slows execution.

----------

# 4. `executablePath`

Launch a specific browser executable.

Example

```ts
executablePath:

'C:/Program Files/Google/Chrome/Application/chrome.exe'

```

Useful when:

-   using a custom Chromium build
    
-   testing a portable browser
    
-   validating against a company-managed browser
    

----------

# 5. `downloadsPath`

Specify where browser downloads are stored.

Example

```ts
downloadsPath:'downloads'

```

All downloads are placed in:

```text
downloads/

```

instead of the default temporary location.

----------

# 6. `devtools`

Automatically open DevTools.

```ts
devtools:true

```

Browser starts with:

```text
Chrome DevTools Open

```

Useful while debugging selectors, network requests, or console messages.

----------

# 7. `env`

Pass environment variables to the browser process.

Example

```ts
env:{

API_ENV:'QA'

}

```

Useful for browser extensions or browser-side scripts that read environment variables.

----------

# 8. `firefoxUserPrefs`

Firefox-only preferences.

Example

```ts
firefoxUserPrefs:{

'network.proxy.type':1

}

```

Useful for Firefox-specific customization.

----------

# 9. `chromiumSandbox`

Default

```text
true

```

Disable sandbox

```ts
chromiumSandbox:false

```

Often required inside some Docker containers or restricted Linux environments where the Chromium sandbox cannot be used.

----------

# 10. `handleSIGINT`

Default

```text
true

```

Allows graceful shutdown when pressing:

```text
CTRL + C

```

Generally left unchanged.

----------

# 11. `handleSIGTERM`

Useful in CI.

Allows Playwright to shut down cleanly when the operating system sends a termination signal.

----------

# 12. `handleSIGHUP`

Relevant mostly for Linux/Unix systems.

Again, most users keep the default value.

----------

# Launch Options Example

```ts
export default defineConfig({

use:{

launchOptions:{

channel:'chrome',

slowMo:200,

args:[

'--start-maximized',

'--disable-notifications'

]

}

}

});

```

----------

# Maximized Browser

Many newcomers write:

```ts
viewport:null

```

Only.

Browser still opens at its default window size.

Correct configuration:

```ts
use:{

viewport:null,

launchOptions:{

args:['--start-maximized']

}

}

```

Both settings are required:

-   `viewport: null` → Don't force a viewport size.
    
-   `--start-maximized` → Maximize the browser window.
    

----------

# Chrome vs Chromium

Chromium

Chrome

Bundled with Playwright

Installed separately

Open Source

Official Google browser

Default

Requires `channel`

Great for automation

Better matches end-user environments

----------

# Headless vs Headed

```ts
headless:true

```

Controls whether the browser UI is visible.

```ts
slowMo:500

```

Controls the delay between Playwright actions.

These settings are independent.

Example:

```text
Headless + slowMo

```

The browser is invisible but actions are still slowed down.

----------

# Enterprise Example

```ts
export default defineConfig({

use:{

launchOptions:{

channel:'chrome',

args:[

'--disable-notifications',

'--disable-popup-blocking',

'--start-maximized'

],

slowMo:0

},

headless:true,

viewport:null

}

});

```

This configuration runs tests in the installed Chrome browser, starts maximized, suppresses notifications, and runs at full speed.

----------

# Common Mistakes

## ❌ Using `slowMo` in CI

```ts
slowMo:1000

```

A suite with 10,000 actions would spend nearly three hours waiting due to the artificial delay alone.

----------

## ❌ Confusing `headless` with `slowMo`

`headless`

```text
Controls browser visibility

```

`slowMo`

```text
Controls execution speed

```

----------

## ❌ Hardcoding `executablePath`

Hardcoded paths can break across different operating systems or developer machines.

Prefer:

```ts
channel:'chrome'

```

when possible.

----------

## ❌ Using `viewport:null` Without Maximizing

Many assume this automatically maximizes the browser.

It simply tells Playwright not to override the window size. If you want a maximized window, combine it with:

```ts
args:['--start-maximized']

```

----------

# Interview Questions

### Q1. What is the difference between `use` and `launchOptions`?

`use` configures the browser context (viewport, locale, storage state, permissions, etc.).

`launchOptions` configures the browser process (browser channel, CLI arguments, slow motion, executable path, etc.).

----------

### Q2. What does `channel` do?

It tells Playwright to launch a specific installed browser, such as Chrome or Microsoft Edge, instead of the bundled Chromium.

Example:

```ts
channel:'chrome'

```

----------

### Q3. What is `slowMo`?

It introduces an artificial delay between Playwright actions, primarily for debugging or demonstrations.

----------

### Q4. How do you launch Microsoft Edge?

```ts
channel:'msedge'

```

----------

### Q5. How do you maximize the browser?

```ts
viewport:null,

launchOptions:{

args:['--start-maximized']

}

```

----------

### Q6. What is `executablePath` used for?

It launches a specific browser executable instead of the default browser installation.

----------

### Q7. Is `slowMo` recommended for CI?

No. It unnecessarily increases execution time and should generally be reserved for local debugging.

----------

# Best Practices

-   Prefer `channel` over `executablePath` whenever possible for better portability.
    
-   Use Chrome (`channel: 'chrome'`) if you need to validate behavior in the production browser rather than Playwright's bundled Chromium.
    
-   Reserve `slowMo` for local debugging or demos.
    
-   Keep browser arguments to the minimum required; avoid disabling security features unless your test environment demands it.
    
-   Combine `viewport: null` with `--start-maximized` when testing desktop layouts in a maximized window.
    
-   Avoid machine-specific paths in shared configuration files.
    

----------

# 💡 Real Interview Scenario

**Question:**

> Your application works correctly in Playwright Chromium but fails in Google Chrome. How would you investigate?

A strong answer would include:

1.  Run the tests using the installed Chrome browser:
    

```ts
launchOptions: {
  channel: 'chrome'
}

```

2.  Compare behavior between Chromium and Chrome.
    
3.  Enable tracing and inspect the failure:
    

```ts
trace: 'retain-on-failure'

```

4.  Run in headed mode with DevTools if needed:
    

```ts
headless: false,
launchOptions: {
  devtools: true
}

```

5.  Review browser console, network requests, and trace artifacts to identify browser-specific differences.
    

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbNDE4MjQ5OTI1XX0=
-->