# Module 3: Architectural Principles – DRY & SOLID at Scale
When scaling a Playwright framework past a dozen tests, arbitrary scripting leads to flaky test suites and high maintenance costs. To build a robust framework, we follow two engineering foundations: **DRY** and **SOLID**.
## 1. The DRY Principle (Don't Repeat Yourself)
### The Architectural Blueprint
DRY is more than avoiding duplicate lines of code; it is about **centralizing system knowledge**. If a business flow, a UI element locator, or an environment endpoint is duplicated across your files, a change in the application forces a breaking change across your entire codebase.
In a professional Playwright framework, we apply DRY across three distinct layers:
 1. **Configuration Layer:** Base URLs, timeouts, and browser contexts are declared once.
 2. **State/Hook Layer:** Reusable setup flows (like authenticating a session) are handled globally or via custom fixtures.
 3. **Action Layer:** Repeated UI interactions are componentized.
### Detailed Framework Code Example
#### ❌ The DRY-Violating Anti-Pattern
Imagine a scenario where three different feature suites (profile.spec.ts, billing.spec.ts, reports.spec.ts) require a standard authenticated session. A naive approach copies the authorization logic into every test file:
```typescript
// ❌ ANTI-PATTERN: Copying the exact same login sequence into separate files
// file: tests/billing.spec.ts
import { test, expect } from '@playwright/test';

test('Update credit card info', async ({ page }) => {
    // Duplicated Auth Logic
    await page.goto('https://qa-env.mycompany.com/login');
    await page.locator('#username-field').fill('finance_user');
    await page.locator('#password-field').fill('SecurePass123!');
    await page.locator('button[type="submit"]').click();
    await expect(page).toHaveURL(/.*dashboard/);

    // Actual Billing Test Logic
    await page.goto('/billing');
    await page.locator('#card-number').fill('4111222233334444');
    // ... remaining test code
});

```
#### 💻 The DRY-Compliant Framework Solution
To fix this globally, we isolate authentication out of individual test blocks. We can use a custom **Playwright Fixture** to inject an pre-authenticated page state cleanly.
```typescript
// file: src/fixtures/authFixture.ts
import { test as baseTest, expect } from '@playwright/test';
import * as fs from 'fs';

// Extend the base test context to provide a reusable, authenticated page instance
export const authenticatedTest = baseTest.extend<{ authPage: any }>({
    authPage: async ({ page, baseURL }, use) => {
        // Centralized Authentication Logic - written exactly ONCE
        await page.goto(`${baseURL}/login`);
        await page.locator('#username-field').fill(process.env.TEST_USER || 'standard_user');
        await page.locator('#password-field').fill(process.env.TEST_PASS || 'password');
        await page.locator('button[type="submit"]').click();
        await expect(page).toHaveURL(/.*dashboard/);

        // Hand control over to the test executing this fixture
        await use(page);
    }
});

```
```typescript
// file: tests/billing.spec.ts
//  COMPLIANT: Test scripts consume the auth state automatically without code duplication
import { authenticatedTest as test } from '../src/fixtures/authFixture';
import { expect } from '@playwright/test';

test('Update credit card info', async ({ authPage }) => {
    // The browser is already logged in here via the authPage fixture!
    await authPage.goto('/billing');
    await authPage.locator('#card-number').fill('4111222233334444');
    // ... brief, focused billing interaction logic
});

```
## 2. SOLID Principles in Test Automation
Let's unpack all five SOLID principles mapped strictly to a distributed automation framework layout.
### Framework Directory Structure Overview
Before looking at the code, here is where these decoupled files live in an enterprise-level automation tree:
```text
playwright-framework/
├── playwright.config.ts          # Core framework configuration (DIP)
├── .env                          # Local environment variables
├── src/
│   ├── interfaces/               # Segregated type definitions (ISP)
│   │   └── actions.interface.ts
│   ├── pages/                    # UI Page Objects (SRP / OCP / LSP)
│   │   ├── BasePage.ts
│   │   ├── LoginPage.ts
│   │   └── AdminDashboardPage.ts
│   └── fixtures/                 # Custom operational abstractions (DIP)
│       └── baseFixture.ts
└── tests/                        # Clean execution specifications
    └── login.spec.ts

```
### S – Single Responsibility Principle (SRP)
> **Rule:** A class or module should have one, and only one, reason to change.
> 
 * **In Framework Architecture:** Never mix **Locators/Actions**, **Assertions**, and **Environment Setup** in the same component.
#### ❌ The Bad (Violating SRP)
This page object class tries to manage UI locators, evaluate server environment endpoints, read files, and assert data structures simultaneously.
```typescript
// file: src/pages/BadLoginPage.ts
// ❌ VIOLATION: Page Object handles element finding, config mapping, and test assertions.
import { Page, expect } from '@playwright/test';

export class BadLoginPage {
    constructor(private page: Page) {}

    async executeLogin() {
        // Violating SRP: Environment resolution does not belong inside a Page Class
        const environmentUrl = process.env.STAGE === 'PROD' ? 'https://prod.app.com' : 'https://qa.app.com';
        await this.page.goto(environmentUrl);

        await this.page.locator('#user').fill('user');
        await this.page.locator('#pass').fill('pass');
        await this.page.locator('#submit').click();

        // Violating SRP: Assertions inside page objects lock the class into fixed validation schemes
        await expect(this.page.locator('#welcome-banner')).toBeVisible();
    }
}

```
#### 💻 The Good (Following SRP)
Split the file definitions clearly. The Page Object maps elements and performs actions. The test file handles configurations and assertions.
```typescript
// file: src/pages/LoginPage.ts
//  COMPLIANT: This class manages exactly one thing: interacting with the Login UI view
import { Page, Locator } from '@playwright/test';

export class LoginPage {
    private usernameInput: Locator;
    private passwordInput: Locator;
    private loginButton: Locator;

    constructor(private page: Page) {
        this.usernameInput = page.locator('#user');
        this.passwordInput = page.locator('#pass');
        this.loginButton = page.locator('#submit');
    }

    async login(user: string, pass: string): Promise<void> {
        await this.usernameInput.fill(user);
        await this.passwordInput.fill(pass);
        await this.loginButton.click();
    }
}

```
### O – Open/Closed Principle (OCP)
> **Rule:** Software artifacts should be open for extension, but closed for modification.
> 
 * **In Framework Architecture:** When adding fresh behaviors (e.g., handling regular login vs. Multi-Factor Authentication login), **extend** your base code instead of ripping into functional methods with messy conditional logic blocks.
#### ❌ The Bad (Violating OCP)
Modifying an active, working page object method to patch in support for a new user flow breaks legacy tests and risks side effects.
```typescript
// ❌ VIOLATION: Ripping open functional methods with fragile if/else statements
async completeLogin(username: string, pass: string, loginType: 'standard' | 'mfa' | 'sso') {
    await this.usernameInput.fill(username);
    await this.passwordInput.fill(pass);
    await this.submitButton.click();

    // Appending mutations manually violates OCP
    if (loginType === 'mfa') {
        await this.page.locator('#mfa-token-field').fill('123456');
        await this.page.locator('#mfa-submit').click();
    } else if (loginType === 'sso') {
        await this.page.locator('#sso-redirect-link').click();
    }
}

```
#### 💻 The Good (Following OCP)
Keep the working core class stable. If special treatment is required for MFA authentication, extend the base functionality safely using standard object inheritance.
```typescript
// file: src/pages/MFALoginPage.ts
//  COMPLIANT: Subclass extends the core behavior without mutating original baseline code
import { LoginPage } from './LoginPage';
import { Page, Locator } from '@playwright/test';

export class MFALoginPage extends LoginPage {
    private mfaInput: Locator;
    private mfaSubmitButton: Locator;

    constructor(page: Page) {
        super(page); // Inherits base locators seamlessly
        this.mfaInput = page.locator('#mfa-token-field');
        this.mfaSubmitButton = page.locator('#mfa-submit');
    }

    async loginWithMFAToken(user: string, pass: string, token: string): Promise<void> {
        // Reuse base methods completely closed for modification
        await this.login(user, pass); 
        
        // Extend functionality gracefully
        await this.mfaInput.fill(token);
        await this.mfaSubmitButton.click();
    }
}

```
### L – Liskov Substitution Principle (LSP)
> **Rule:** Objects of a superclass must be replaceable with objects of subclasses without breaking functional execution.
> 
 * **In Framework Architecture:** Subclasses or custom wrapped wrappers must honor the interface contract of their parent types. If a child component overrides a parent method, it must not alter input expectations, fail arbitrarily, or strip functions away.
#### ❌ The Bad (Violating LSP)
Creating an inherited page object that restricts access to base actions or changes operational assumptions breaks polymorphism.
```typescript
// Base Page
export class BasePage {
    constructor(protected page: Page) {}
    async navigateTo(path: string) {
        await this.page.goto(path);
    }
}

// ❌ VIOLATION: Subclass fundamentally alters or disables a parent capability
export class SecureSettingsPage extends BasePage {
    // Overriding the method to crash breaks substitution expectations
    async navigateTo(path: string) {
        throw new Error("Direct routing forbidden for security! Must use dashboard clicks.");
    }
}

```
If your framework relies on a global teardown or utility helper loop that processes an array of generic BasePage entities via .navigateTo('/home'), passing SecureSettingsPage will trigger a fatal runtime failure.
#### 💻 The Good (Following LSP)
Subclasses must accept the identical parameters and general execution pathways of their parent components, appending contextual logic transparently without crashing execution lines.
```typescript
// file: src/pages/SecureSettingsPage.ts
//  COMPLIANT: Subclass respects baseline parameters while enhancing access controls cleanly
export class SecureSettingsPage extends BasePage {
    async navigateTo(path: string) {
        // Safely intercept and handle specific initialization states, then fulfill parent contract
        console.log(`Auditing secure route request to: ${path}`);
        await this.page.locator('#security-handshake-spinner').waitFor({ state: 'detached' });
        await super.navigateTo(path); // Fulfills parent signature perfectly
    }
}

```
### I – Interface Segregation Principle (ISP)
> **Rule:** Clients should not be forced to depend upon interfaces or methods they do not actively employ.
> 
 * **In Framework Architecture:** Break apart monster custom framework interfaces into highly targeted, granular behavior contracts. Do not force page structures to implement interactions they lack the capability to perform.
#### ❌ The Bad (Violating ISP)
Creating a single monolithic interface wrapping every action type under the sun results in bloat.
```typescript
// ❌ VIOLATION: A bloated interface design
interface ComponentInteractions {
    enterText(selector: string, val: string): Promise<void>;
    clickElement(selector: string): Promise<void>;
    uploadFile(selector: string, file: string): Promise<void>;
    sortDataGrid(colId: string): Promise<void>;
}

// A basic informational alert dialog page is forced to implement grid sorting and file uploads!
export class InformationalModal implements ComponentInteractions {
    async enterText() {}
    async clickElement() { /* actual code */ }
    
    // Extraneous code lines written just to appease compilation restrictions
    async uploadFile() { throw new Error("Modals cannot upload files!"); }
    async sortDataGrid() { throw new Error("Modals do not have grids!"); }
}

```
#### 💻 The Good (Following ISP)
Define lean, modular interfaces representing singular, explicit behavioral traits. Mix and match those components precisely where they apply.
```typescript
// file: src/interfaces/actions.interface.ts
//  COMPLIANT: Micro-focused, distinct interaction roles
export interface Clickable {
    clickElement(selector: string): Promise<void>;
}

export interface FileUploadable {
    uploadFile(selector: string, path: string): Promise<void>;
}

export interface GridSortable {
    sortDataGrid(columnId: string): Promise<void>;
}

// A simple page block implements exactly what fits its real UI footprint
export class MediaUploadDialog implements Clickable, FileUploadable {
    async clickElement(selector: string) { /* ... */ }
    async uploadFile(selector: string, path: string) { /* ... */ }
}

```
### D – Dependency Inversion Principle (DIP)
> **Rule:** High-level modules should not depend on low-level concrete implementations. Both should rely on abstractions.
> 
 * **In Framework Architecture:** Your actual test specification code (.spec.ts files) should never be tightly bound to concrete configuration strings, file streams, or lower-level backend databases directly inside the script block. Everything should be resolved via dynamic dependencies and dependency injection containers like Playwright custom fixtures.
#### ❌ The Bad (Violating DIP)
If a test spec explicitly constructs low-level infrastructure instances directly inside its execution body, changing your backend architecture instantly breaks the test suites.
```typescript
// ❌ VIOLATION: High-level test logic tightly coupled to low-level MySQL implementation
import { MySqlDatabaseDriver } from '../db/mysql-driver';

test('Verify reporting dashboard data', async ({ page }) => {
    const db = new MySqlDatabaseDriver(); // Tight coupling
    const data = await db.executeQuery('SELECT revenue FROM sales WHERE id = 99');
    
    await page.goto('/reports');
    await expect(page.locator('.revenue-label')).toHaveText(data.revenue);
});

```
#### 💻 The Good (Following DIP)
Invert the flow of dependencies. Define an abstracted interface layer or utilize Playwright's **Fixtures** engine to supply operational layers contextually at run time.
```typescript
// file: src/fixtures/baseFixture.ts
//  COMPLIANT: High-level tests interact with abstract helpers injected down by the fixture engine
import { test as baseTest } from '@playwright/test';

interface DataService {
    getSalesRevenue(id: number): Promise<string>;
}

// The fixture abstracts low-level construction. If you switch databases later, 
// you ONLY change this orchestrator file—none of your 500 test scripts require touchups.
export const test = baseTest.extend<{ salesService: DataService }>({
    salesService: async ({}, use) => {
        const structuralService: DataService = {
            getSalesRevenue: async (id) => {
                // Concrete data retrieval implementation safely hidden under abstraction
                return "150,000.00"; 
            }
        };
        await use(structuralService);
    }
});

```
```typescript
// file: tests/login.spec.ts
// Clean execution consuming abstracted operations via Dependency Inversion
import { test } from '../src/fixtures/baseFixture';
import { expect } from '@playwright/test';

test('Verify reporting dashboard data', async ({ page, salesService }) => {
    // High level test calls decoupled method signatures seamlessly
    const projectedRevenue = await salesService.getSalesRevenue(99);
    
    await page.goto('/reports');
    await expect(page.locator('.revenue-label')).toHaveText(projectedRevenue);
});

```

<!--stackedit_data:
eyJoaXN0b3J5IjpbOTIxNDE3NjM5XX0=
-->