
# Reference Guide: Playwright Test Generator (`codegen`)

The Playwright Test Generator simplifies test creation by recording manual browser interactions and translating them into resilient, locator-based Playwright code.

## 1. Core Workflows

### A. The VS Code Extension

Manage recordings directly within the IDE using the **Testing** sidebar:

-   **Record new:** Creates a `test-1.spec.ts` file and launches an automated browser instance.
    
-   **Record at Cursor:** Appends new actions directly into an existing test file at the current cursor position.
    
-   **Pick Locator:** Highlights the most resilient locator when hovering over any element in the browser. Clicking the element captures the locator code for quick copying or direct insertion.
    

### B. The CLI Interface

For standalone recording or operations outside of VS Code:

-   **Basic Command:** `npx playwright codegen <URL>`
    
-   **The Playwright Inspector:** Running the command opens two windows—the target browser window for interactions and the Inspector window to view, pause, and copy the generated scripts.
    

## 2. Detailed End-to-End Example: E-Commerce Flow

This step-by-step example demonstrates what actions to perform in the browser and the exact TypeScript code the generator outputs.

### Scenario: Logging in, searching for an item, adding it to the cart, and verifying the cart count.

### The Automated Workflow

TypeScript

```
import { test, expect } from '@playwright/test';

test('complete e-commerce purchase flow', async ({ page }) => {
  // 1. Navigate to the target web application
  await page.goto('https://example-shop.com/');

  // 2. Click the login button using its role and name
  await page.getByRole('link', { name: 'Log In' }).click();

  // 3. Fill out the authentication form
  await page.getByPlaceholder('Enter your email').fill('user@example.com');
  await page.getByPlaceholder('Enter your password').fill('SecurePassword123');
  await page.getByRole('button', { name: 'Submit' }).click();

  // 4. Search for a specific product
  await page.getByRole('searchbox', { name: 'Search products' }).fill('Mechanical Keyboard');
  await page.getByRole('searchbox', { name: 'Search products' }).press('Enter');

  // 5. Select the item from the search results
  await page.getByRole('heading', { name: 'RGB Mechanical Keyboard v2' }).click();

  // 6. Add the selected item to the shopping cart
  await page.getByRole('button', { name: 'Add to Cart' }).click();

  // --- ASSERTIONS MADE VIA THE CODEGEN TOOLBAR ---

  // 7. Clicked 'Assert Visibility' on the success banner
  await expect(page.getByText('Item successfully added to your cart!')).toBeVisible();

  // 8. Clicked 'Assert Text' on the cart badge to ensure it updated to '1'
  await expect(page.locator('.cart-badge')).toContainText('1');
});

```

## 3. Environment Emulation and Context Control

The generator supports testing under varied device parameters and states without modifying project configuration files beforehand.

### Device and Environment Emulation

Launch the generator to mimic specific user environments using CLI flags:

-   **Device Profiles:** `npx playwright codegen --device="iPhone 13" https://example-shop.com`
    
-   **Custom Viewports:** `npx playwright codegen --viewport-size="800,600" https://example-shop.com`
    
-   **Color Schemes:** `npx playwright codegen --color-scheme=dark https://example-shop.com`
    
-   **Localization & Geolocation:** ```bash
    
    npx playwright codegen --timezone="Europe/Rome" --geolocation="41.890221,12.492348" --lang="it-IT" https://google.com/maps
    

### Authentication and Session Persistence

To avoid recording multi-step login sequences repeatedly, preserve and reuse the browser's authenticated state:

1.  **Save the Session:** Run the command below, perform the manual login steps, and close the browser. The session cookies, local storage, and IndexedDB data will save directly to a local file.
    
    Bash
    
    ```
    npx playwright codegen https://example-shop.com/login --save-storage=auth.json
    
    ```
    
2.  **Load the Session:** Run the command below for subsequent tests. The browser bypasses the login screen, restoring the authenticated state instantly so you can record inner-app functionalities directly.
    
    Bash
    
    ```
    npx playwright codegen https://example-shop.com/dashboard --load-storage=auth.json
    
    ```
    
    > ⚠️ **Security Warning:** `auth.json` contains sensitive session tokens. Ensure this file name is added to your `.gitignore` to prevent it from being committed to version control.
    

## 4. Implementation Best Practices

-   **Embed Structural Assertions:** Clicks and navigation alone do not constitute a complete test. Utilize the built-in toolbar icons (**Assert visibility**, **Assert text**, **Assert value**) during recording to automatically generate standard `expect()` verification blocks.
    
-   **Refactor Generated Code:** Code generation serves as a foundational scaffold. Once recorded, the code should be refactored to fit project architecture, such as migrating selectors into Page Object Models (POMs) or replacing hardcoded strings with environment variables.
    
-   **Intercept Custom Setups via `page.pause()`:** For environments requiring custom middleware, network routing (`browserContext.route()`), or mock APIs, initialize the setup programmatically in a script and invoke `await page.pause();`. This launches the Inspector controls mid-execution on the configured page context.
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTE1NzE4MjQyNV19
-->