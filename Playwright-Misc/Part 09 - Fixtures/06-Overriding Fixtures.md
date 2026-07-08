Excellent. This is where Playwright fixtures become **extremely powerful**.

Most engineers stop at creating fixtures like `loginPage` or `customerApi`. Enterprise frameworks, however, make extensive use of **fixture overriding** to modify or extend Playwright's built-in behavior.

This is one of the most frequently misunderstood topics in Playwright.

----------

# 📘 Playwright Fixtures Handbook

# Part 6 – Overriding Fixtures (Complete Guide)

> **Creating a fixture adds new functionality.**
> 
> **Overriding a fixture changes existing functionality.**

Think of it like subclassing in OOP, except you're extending fixture behavior instead of class behavior.

----------

# What is Fixture Overriding?

Suppose Playwright provides:

```text
page

```

You can replace or enhance how `page` is created.

Instead of:

```text
Playwright

↓

Creates Page

```

You can do:

```text
Playwright

↓

Your Code

↓

Creates Page

↓

Passes to Test

```

The test still requests:

```ts
page

```

But now it receives **your customized version**.

----------

# Why Override Fixtures?

Common enterprise use cases:

-   Automatically log users in
    
-   Automatically navigate to the application
    
-   Automatically accept cookie banners
    
-   Automatically set feature flags
    
-   Automatically inject test data
    
-   Automatically configure locale/timezone
    
-   Automatically mock APIs
    

Instead of writing this in every test:

```ts
await page.goto('/');

await login();

await acceptCookies();

```

you do it once in the fixture.

----------

# Basic Override Syntax

Suppose Playwright already provides:

```text
page

```

Override it:

```ts
import { test as base } from '@playwright/test';

export const test = base.extend({

  page: async ({ page }, use) => {

    // custom behavior

    await use(page);

  }

});

```

Notice:

The fixture name remains:

```text
page

```

You are replacing the default behavior.

----------

# Understanding the Parameters

This confuses many people.

```ts
page: async ({ page }, use) => {

}

```

Question:

> Why is `page` both the fixture name and a dependency?

Answer:

The **left-hand `page`** is the fixture you're overriding.

The **`{ page }`** parameter is the original Playwright fixture that you're extending.

Think of it as:

```text
Original Page

↓

My Custom Logic

↓

Final Page

```

----------

# Example 1 – Automatically Open Home Page

Without overriding:

```ts
test('Login', async ({ page }) => {

    await page.goto('/');

});

```

Every test repeats:

```ts
await page.goto('/');

```

----------

Override

```ts
page: async ({ page }, use) => {

    await page.goto('/');

    await use(page);

}

```

Now every test starts at:

```text
/

```

Automatically.

----------

# Execution Flow

```text
Create Page

↓

Navigate Home

↓

Run Test

↓

Cleanup

```

----------

# Example 2 – Auto Login

Without override

```ts
await page.goto('/login');

await loginPage.login();

```

Every test.

----------

Override

```ts
page: async ({ page }, use) => {

    await page.goto('/login');

    await page.fill('#user','admin');

    await page.fill('#password','password');

    await page.click('#login');

    await use(page);

}

```

Now:

```ts
test('Dashboard', async ({ page }) => {

});

```

Already logged in.

----------

# Better Approach

Although possible, enterprise Playwright projects usually prefer:

```text
storageState

```

or

```text
Setup Project

```

instead of UI login.

We'll compare them shortly.

----------

# Example 3 – Accept Cookie Banner

Instead of

```ts
await page.click('Accept');

```

Every test.

Override

```ts
page: async ({ page }, use)=>{

    await page.goto('/');

    await page.click('Accept');

    await use(page);

}

```

Now tests never see the popup.

----------

# Example 4 – Inject Feature Flags

```ts
context: async ({ context }, use) => {

    await context.addCookies([

        {

            name:'feature',

            value:'enabled'

        }

    ]);

    await use(context);

}

```

Every page now uses that cookie.

----------

# Overriding `context`

Example

```ts
context: async ({ context }, use)=>{

    await context.grantPermissions([

        'clipboard-read'

    ]);

    await use(context);

}

```

Every test automatically has clipboard permission.

----------

# Overriding `request`

```ts
request: async ({ request }, use)=>{

    request.setExtraHTTPHeaders({

        Authorization:'Bearer Token'

    });

    await use(request);

}

```

Every API call automatically sends the token.

----------

# Overriding Custom Fixtures

Suppose:

Original

```ts
loginPage

```

Override

```ts
loginPage: async ({

loginPage

}, use)=>{

// extra behavior

await use(loginPage);

}

```

You can extend your own fixtures exactly like built-in ones.

----------

# Override Flow

```text
Original Fixture

↓

Custom Logic

↓

Same Fixture Name

↓

Test

```

The test never changes.

----------

# Enterprise Example – Logged-In Page

Instead of

```text
Test

↓

Login

↓

Dashboard

```

Override

```text
Page Fixture

↓

Login

↓

Dashboard

↓

Test

```

Every test starts authenticated.

----------

# But Wait...

Should you override `page` for login?

Usually:

**No.**

Better:

```text
Setup Project

↓

storageState

↓

page

```

Why?

Because UI login:

-   Slower
    
-   More fragile
    
-   More maintenance
    

Storage state is faster and cleaner.

----------

# Good Override Examples

✅ Navigate Home

✅ Grant Permissions

✅ Configure Locale

✅ Set Cookies

✅ Mock APIs

✅ Configure Headers

----------

# Poor Override Examples

❌ Login through UI

❌ Create Test Data

❌ Perform Business Operations

Fixtures should prepare the environment, not execute business scenarios.

----------

# Overriding vs Creating

Creating

```ts
loginPage

```

New fixture.

----------

Overriding

```ts
page

```

Existing fixture.

----------

# Multiple Overrides

```ts
export const test = base.extend({

page:async(...){},

context:async(...){},

request:async(...){}

});

```

All three are customized.

----------

# Dependency Graph

```text
Browser

↓

Context (Overridden)

↓

Page (Overridden)

↓

LoginPage

```

Everything still works.

----------

# Enterprise Example

```ts
page: async ({ page }, use)=>{

    await page.goto('/');

    await page.evaluate(() => {

        localStorage.setItem(

            'theme',

            'dark'

        );

    });

    await use(page);

}

```

Every test starts in:

🌙 Dark Mode

----------

# Common Mistakes

## ❌ Forgetting `await use()`

Wrong

```ts
page: async ({ page })=>{

}

```

The test never receives the fixture.

----------

## ❌ Cleanup Before `use`

Wrong

```ts
close();

await use(page);

```

Correct

```text
Setup

↓

use

↓

Cleanup

```

----------

## ❌ Business Logic in Fixtures

Don't do:

```text
Place Order

Delete Customer

Checkout

```

inside fixtures.

Fixtures should prepare state, not perform the scenario under test.

----------

## ❌ Overriding Everything

Not every fixture needs overriding.

Only when behavior genuinely needs to change.

----------

# Storage State vs Override

Override Login

Storage State

UI Login Every Run

Reuse Existing Login

Slow

Fast

Fragile

Stable

More Maintenance

Less Maintenance

Enterprise frameworks usually choose:

```text
Setup Project

↓

storageState

```

----------

# Interview Questions

### Q1. What is fixture overriding?

It replaces or extends the behavior of an existing fixture while keeping the same fixture name.

----------

### Q2. Can built-in fixtures be overridden?

Yes.

Examples:

-   `page`
    
-   `context`
    
-   `request`
    

----------

### Q3. Can custom fixtures be overridden?

Yes.

The same mechanism applies.

----------

### Q4. Is overriding `page` for login recommended?

Usually not.

A Setup Project with `storageState` is generally preferred because it is faster and more reliable.

----------

### Q5. What's the difference between creating and overriding?

Creating

Overriding

Adds a new fixture

Changes an existing fixture

New fixture name

Same fixture name

Used for new dependencies

Used to customize behavior

----------

### Q6. What are good uses for overriding?

-   Automatic navigation
    
-   Permission grants
    
-   Default headers
    
-   Cookies
    
-   Feature flags
    
-   Test environment setup
    

----------

# Best Practices

-   Override fixtures only when you need to change default behavior.
    
-   Keep overrides focused on environment preparation.
    
-   Avoid embedding business workflows in fixture overrides.
    
-   Prefer `storageState` over UI login in overridden `page` fixtures.
    
-   Always call `await use(...)`; otherwise the fixture is never passed to the test.
    
-   Place cleanup logic after `await use(...)`.
    

----------

# ⭐ Enterprise Example

A common enterprise override configures the browser context consistently for every test:

```ts
import { test as base } from '@playwright/test';

export const test = base.extend({

  context: async ({ context }, use) => {

    await context.grantPermissions([
      'clipboard-read',
      'clipboard-write'
    ]);

    await context.addCookies([
      {
        name: 'featureFlag',
        value: 'new-checkout',
        domain: 'localhost',
        path: '/'
      }
    ]);

    await use(context);

  }

});

```

Every test now starts with:

-   Clipboard permissions granted
    
-   Feature flag enabled
    
-   No repeated setup code
    

The tests remain simple:

```ts
test('Checkout', async ({ page }) => {
  // Test only the checkout flow
});

```

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjk4MzM1NzA3XX0=
-->