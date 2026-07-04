# Part 1 – Configuration File Basics

## What is `playwright.config.ts`?

The `playwright.config.ts` file is the **central configuration file** for the Playwright Test Runner. It defines how tests are discovered, executed, retried, reported, and how browser contexts are configured.

Think of it as the "brain" of your Playwright project.

A typical project structure looks like:

```text
project/
│
├── playwright.config.ts
├── package.json
├── tests/
│   ├── login.spec.ts
│   └── checkout.spec.ts
├── pages/
├── utils/
└── test-results/

```

Playwright automatically looks for `playwright.config.ts` in the project root when you run:

```bash
npx playwright test

```

No additional configuration is needed unless you want to use a different config file.

----------

# Basic Structure

The configuration is typically wrapped using `defineConfig()`:

```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({

});

```

### Why use `defineConfig()`?

Although you could export a plain object, `defineConfig()` provides:

-   TypeScript IntelliSense
    
-   Compile-time validation
    
-   Better auto-completion
    
-   Detection of invalid configuration properties
    

----------

# Minimal Configuration

```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests'
});

```

This tells Playwright to search for tests under the `tests` directory.

----------

# A Typical Starter Configuration

```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({

  testDir: './tests',

  timeout: 30000,

  retries: 1,

  workers: 4,

  reporter: 'html',

  use: {
    headless: true,
    baseURL: 'https://example.com'
  }

});

```

This configuration specifies:

-   Test location
    
-   Maximum test timeout
    
-   Retry failed tests once
    
-   Run up to four workers in parallel
    
-   Generate an HTML report
    
-   Use headless browser mode
    
-   Set a base URL
    

----------

# Using Multiple Configuration Files

You can maintain separate configurations for different environments.

Example:

```text
playwright.config.ts
playwright.dev.config.ts
playwright.qa.config.ts
playwright.prod.config.ts

```

Run a specific configuration:

```bash
npx playwright test --config=playwright.qa.config.ts

```

Another example:

```bash
npx playwright test --config=configs/mobile.config.ts

```

This is useful for switching between environments, browsers, or execution profiles.

----------

# Organizing Large Configurations

As projects grow, it's common to split configuration into multiple files.

Example structure:

```text
config/
    browsers.ts
    reporters.ts
    environments.ts
    projects.ts

playwright.config.ts

```

Then import the pieces:

```ts
import { defineConfig } from '@playwright/test';
import { projects } from './config/projects';

export default defineConfig({
  projects
});

```

This keeps the main configuration concise and maintainable.

----------

# How Playwright Reads the Configuration

When you run:

```bash
npx playwright test

```

Playwright performs the following steps:

1.  Reads `playwright.config.ts`.
    
2.  Validates the configuration.
    
3.  Initializes reporters.
    
4.  Creates browser projects.
    
5.  Starts the configured number of workers.
    
6.  Discovers test files.
    
7.  Executes tests according to the configuration.
    

----------

# Configuration Precedence

When the same setting is defined in multiple places, Playwright applies them in the following order (highest priority first):

1.  Command-line options (e.g., `--headed`, `--workers`)
    
2.  `test.use()` within a test file
    
3.  Project-specific configuration
    
4.  Global `use` configuration
    
5.  Playwright defaults
    

For example:

Global configuration:

```ts
use: {
  headless: true
}

```

Project configuration:

```ts
projects: [
  {
    name: 'chromium',
    use: {
      headless: false
    }
  }
]

```

Running with:

```bash
npx playwright test --headed

```

results in headed mode because the command-line option overrides the project and global settings.

----------

# Common Interview Questions

**Q: Is `playwright.config.ts` mandatory?**

No. Playwright can run without it, but you'll rely on default settings. In practice, almost every project includes a configuration file.

----------

**Q: Why use `defineConfig()` instead of exporting a plain object?**

It provides TypeScript type safety, validation, and IntelliSense, reducing configuration errors.

----------

**Q: Can a project have multiple configuration files?**

Yes. You can create separate configuration files for different environments or execution scenarios and specify the desired one using the `--config` option.

----------

# Best Practices

-   Keep global settings in the main configuration file.
    
-   Use the `use` section for browser context defaults.
    
-   Prefer projects over creating multiple repositories for different browsers.
    
-   Use separate configuration files for distinct environments (e.g., QA, UAT, Production).
    
-   Avoid hardcoding environment-specific values directly in the configuration; use environment variables where appropriate.
    
-   Split very large configurations into smaller modules for better maintainability.
    

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5MzkzNzMxMzNdfQ==
-->