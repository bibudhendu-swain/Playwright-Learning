# Playwright Test Structure & Annotations: Comprehensive Reference Manual

Playwright Test provides a robust, BDD-style framework to structure, organize, runtime-configure, and tag automated test suites. This reference guide details how to organize test layout patterns, manage nested hierarchies, modularize execution blocks, and leverage metadata annotations for production-grade test suites.

## 1. Core Structural Components

Playwright uses several foundational building blocks to manage test execution isolation, scoping, and readability:

-   **`test(title, async ({ page }) => { ... })`**: Defines a single test case.
    
-   **`test.describe(title, () => { ... })`**: Groups related tests logically into a block (e.g., a specific feature or page context).
    
-   **`test.step(title, async () => { ... })`**: Breaks a long test into nested, named logical sections. These steps report individually in the Playwright HTML report.
    
-   **Lifecycle Hooks (`beforeEach`, `afterEach`, `beforeAll`, `afterAll`)**: Set up state before tests run, or tear down environments afterward.
    

## 2. Monolithic vs. Structured Test Layout

To see the value of structured test code, compare a flat, unstructured test against one utilizing Playwright's structural APIs.

### The Unstructured Way (Hard to Maintain)

Without grouping or steps, debugging a failure in a long workflow is difficult because the HTML report treats the entire test as a single monolithic block of code.

TypeScript

```
import { test, expect } from '@playwright/test';

test('unstructured checkout flow', async ({ page }) => {
  await page.goto('https://example.com/login');
  await page.fill('#username', 'user1');
  await page.fill('#password', 'pass1');
  await page.click('#submit');
  
  await page.goto('https://example.com/shop');
  await page.click('.add-to-cart-btn');
  
  await page.goto('https://example.com/checkout');
  await page.click('#pay-now');
  await expect(page.locator('.success-msg')).toBeVisible();
});

```

### The Structured Way (Highly Maintainable)

By grouping related tests inside a `test.describe` block and dividing long actions into `test.step` annotations, your test suite immediately becomes self-documenting.

TypeScript

```
import { test, expect } from '@playwright/test';

// Grouping related functionalities under a specific feature block
test.describe('E-Commerce Checkout Pipeline', () => {

  // Lifecycle hook: This runs before every single test inside this describe block
  test.beforeEach(async ({ page }) => {
    await test.step('Navigate to Base URL and Login', async () => {
      await page.goto('https://example.com/login');
      await page.getByPlaceholder('Username').fill('standard_user');
      await page.getByPlaceholder('Password').fill('secret_sauce');
      await page.getByRole('button', { name: 'Login' }).click();
      await expect(page).toHaveURL(/.*inventory/);
    });
  });

  // Lifecycle hook: Runs after every test to clean up state
  test.afterEach(async ({ page }) => {
    await test.step('State teardown and cache clearing', async () => {
      await page.evaluate(() => localStorage.clear());
    });
  });

  test('Successful purchase of single item via credit card', async ({ page }) => {
    
    // Step 1: Inventory interactions
    await test.step('Add product to shopping cart', async () => {
      await page.locator('[data-test="add-to-cart-backpack"]').click();
      const cartBadge = page.locator('.shopping_cart_badge');
      await expect(cartBadge).toHaveText('1');
    });

    // Step 2: Navigating to cart view
    await test.step('Proceed through shopping cart details', async () => {
      await page.locator('.shopping_cart_link').click();
      await page.locator('[data-test="checkout"]').click();
    });

    // Step 3: Completing the forms
    await test.step('Fill out shipping information summary', async () => {
      await page.getByPlaceholder('First Name').fill('Alex');
      await page.getByPlaceholder('Last Name').fill('Tester');
      await page.getByPlaceholder('Zip/Postal Code').fill('90210');
      await page.locator('[data-test="continue"]').click();
    });

    // Step 4: Verification phase
    await test.step('Finalize order execution and verify receipt', async () => {
      await page.locator('[data-test="finish"]').click();
      const successHeader = page.locator('.complete-header');
      await expect(successHeader).toHaveText('Thank you for your order!');
    });
  });

  test('Attempt checkout with missing shipping information', async ({ page }) => {
    await test.step('Navigate directly to checkout stage', async () => {
      await page.locator('.shopping_cart_link').click();
      await page.locator('[data-test="checkout"]').click();
    });

    await test.step('Submit form without entering parameters', async () => {
      await page.locator('[data-test="continue"]').click();
      
      // Asserting error handling output
      const errorContainer = page.locator('[data-test="error"]');
      await expect(errorContainer).toBeVisible();
      await expect(errorContainer).toContainText('Error: First Name is required');
    });
  });

});

```

## 3. Nesting Describe Blocks

`test.describe` blocks can be nested to mirror complex user workflows, match multi-tiered application components, or manage different browser contexts and user permission states inside a single file.

TypeScript

```
import { test, expect } from '@playwright/test';

test.describe('User Management System', () => {

  // Global hooks for all user tiers can go here if needed

  test.describe('Admin Permissions Context', () => {
    test.beforeEach(async ({ page }) => {
      await test.step('Authenticate as Administrator', async () => {
        // Administrative login logic here
      });
    });

    test('Can modify global organization settings', async ({ page }) => {
      // Admin specific action
    });

    test('Can delete inactive user profiles permanently', async ({ page }) => {
      // Admin specific action
    });
  });

  test.describe('Read-Only Permissions Context', () => {
    test.beforeEach(async ({ page }) => {
      await test.step('Authenticate as Guest Viewer', async () => {
        // Guest login logic here
      });
    });

    test('Can view dashboard data charts', async ({ page }) => {
      // Shared viewing action
    });

    test('Bypasses and hides administrative modification panel', async ({ page }) => {
      // Negative validation context for guests
    });
  });
});

```

## 4. Built-In Test Annotations

Annotations modify runtime execution rules either statically or dynamically based on environmental conditions.

### Execution Framework Modifiers

**Annotation**

**Behavior**

**Typical Use Case**

**`test.only()`**

Focuses execution _exclusively_ on this test or suite.

Debugging a single test in isolation.

**`test.skip()`**

Bypasses the test execution completely.

Temporarily disabling a test due to unconfigured environments.

**`test.fixme()`**

Skips execution; explicitly meant for broken/crashing tests.

Postponing a fix without blocking the CI pipeline.

**`test.fail()`**

Instructs Playwright that this test _must_ fail.

Validating bug presence or negative test paths.

**`test.slow()`**

Flags a test as long-running, tripling its default timeout.

Complex end-to-end multi-page user loops.

### Inline & Conditional Annotations

Annotations can accept conditional logic inline, allowing your test suite to adapt dynamically to different environments, devices, or browsers.

TypeScript

```
import { test, expect } from '@playwright/test';

// 1. Conditional Skip: Skip specific browsers
test('Download reports validation', async ({ page, browserName }) => {
  test.skip(browserName === 'firefox', 'Firefox file download bug is tracked in ticket #123');
  
  await page.goto('https://example.com/reports');
  // Remaining test steps...
});

// 2. Conditional FIXME inside a Hook to bypass broken execution flows
test.beforeEach(async ({ page, isMobile }) => {
  test.fixme(isMobile, 'Settings page does not work in mobile yet');
  await page.goto('http://localhost:3000/settings');
});

// 3. Runtime/Dynamic Annotation
test('Dynamic payment engine validation', async ({ page, browser }) => {
  await page.goto('https://example.com/pay');
  
  // Inject custom metadata directly into the test info metadata profile at runtime
  test.info().annotations.push({
    type: 'browser version',
    description: browser.version(),
  });
});

```

## 5. Test Tagging & Metadata Control

Tagging allows you to catalog and filter tests easily during execution or within Continuous Integration (CI) test execution runs.

### Assigning Tags to Test Suites

You can declare `@` token keywords inline within the text description or explicitly pass metadata array properties:

TypeScript

```
import { test, expect } from '@playwright/test';

// Method A: Meta Object Parameter Declaration
test('Verify accounting ledger outputs', {
  annotation: [
    { type: 'issue', description: 'https://github.com/microsoft/playwright/issues/23180' },
    { type: 'category', description: 'report' }
  ],
}, async ({ page }) => {
  // ...
});

// Method B: Inline String Token Grouping
test.describe('Payment Gateways @billing', () => {
  
  test('Credit card profile registration', { tag: '@fast' }, async ({ page }) => {
    // Carries both @billing and @fast tags
  });

  test('Bank transfer verification routing @slow', async ({ page }) => {
    // Carries both @billing and @slow tags
  });
});

```

### CLI Filtering Commands

Filter your test executions through the terminal based on your configured tags using `--grep` or `--grep-invert`:

Bash

```
# Run only tests containing the @fast tag
npx playwright test --grep @fast

# Run tests containing EITHER @fast OR @billing (Logical OR)
npx playwright test --grep "@fast|@billing"

# Run tests containing BOTH @fast AND @billing (Logical AND)
npx playwright test --grep "(?=.*@fast)(?=.*@billing)"

# Skip any tests carrying the @slow identifier
npx playwright test --grep-invert @slow

```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQ5NDAyMTQ3XX0=
-->