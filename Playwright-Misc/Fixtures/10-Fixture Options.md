Excellent. This is one of the **least understood** Playwright features.

Most developers think fixtures are only for creating objects like `LoginPage` or `CustomerApi`.

But Playwright also allows fixtures to represent **configuration values**.

This feature is called **Fixture Options** (`option: true`), and it powers many enterprise frameworks.

----------

# 📘 Playwright Fixtures Handbook

# Part 10 – Fixture Options (`option: true`) (Complete Guide)

> **Normal fixtures provide objects.**
> 
> **Option fixtures provide configuration.**

Think of them as **typed configuration values** that participate in Playwright's fixture system.

----------

# Why Do We Need Fixture Options?

Imagine your tests run against:

-   QA
    
-   UAT
    
-   PROD
    

Traditional approach:

```ts
process.env.BASE_URL

```

Everywhere.

Or:

```ts
const user = "admin";

```

Hardcoded.

Not ideal.

----------

Instead

Provide configuration through fixtures.

Example

```ts
test('Login', async ({

username,

password

}) => {

});

```

Looks like:

Normal fixture.

But:

They're configuration values.

----------

# What is an Option Fixture?

Normal fixture

↓

Creates

↓

Object

```text
LoginPage

```

Option fixture

↓

Provides

↓

Value

```text
admin

```

----------

# Creating an Option Fixture

Syntax

```ts
export const test = base.extend({

username:[

'admin',

{

option:true

}

]

});

```

Notice:

No

```text
async

```

required.

It's just a value.

----------

# Another Example

```ts
password:[

'Password123',

{

option:true

}

]

```

Test

```ts
test('Login', async ({

username,

password

})=>{

});

```

Injected automatically.

----------

# TypeScript Version

```ts
type Options = {

username:string;

password:string;

};

```

Configuration

```ts
export const test =

base.extend<Options>({

});

```

Strong typing.

----------

# Option Fixture vs Normal Fixture

Normal

```ts
loginPage

↓

Object

```

----------

Option

```ts
username

↓

String

```

----------

# Real Enterprise Example

```text
Environment

↓

Username

↓

Password

↓

Tenant

↓

Region

↓

API Version

```

All become option fixtures.

----------

# Using `test.use()`

One of the biggest advantages.

Suppose

Default

```ts
username:'admin'

```

One test

Needs

```text
manager

```

Override

```ts
test.use({

username:'manager'

});

```

Only this test changes.

----------

# Example

Default

```text
admin

```

Specific suite

```ts
test.describe('Manager',()=>{

test.use({

username:'manager'

});

});

```

Beautiful.

----------

# Multi-Tenant Example

Default

```text
tenant:'US'

```

Override

```ts
test.use({

tenant:'Europe'

});

```

Same tests.

Different tenant.

----------

# Browser Configuration

Suppose

```text
theme:'light'

```

Override

```ts
test.use({

theme:'dark'

});

```

Fixture

↓

Configures application.

----------

# Option Fixture Can Be Used By Other Fixtures

Example

```ts
loginPage:

async(

{

page,

username

},

use

)=>{

```

Dependency

```text
username

↓

loginPage

```

Perfectly valid.

----------

# Enterprise Example

```text
username

↓

Login Fixture

↓

Authenticated Page

```

Changing one option changes login behavior everywhere.

----------

# Complex Options

Instead of

```ts
username

```

Use

```ts
user

```

Example

```ts
user:[

{

username:'admin',

role:'administrator'

},

{

option:true

}

]

```

One object.

Many values.

----------

# Environment Option

```ts
environment:[

'QA',

{

option:true

}

]

```

Usage

```ts
test.use({

environment:'UAT'

});

```

----------

# API Version

```ts
apiVersion:[

'v2',

{

option:true

}

]

```

Different tests

↓

Different APIs.

----------

# Feature Flag

```ts
featureEnabled:[

true,

{

option:true

}

]

```

Override

```ts
test.use({

featureEnabled:false

});

```

Very useful.

----------

# Runtime Configuration

Suppose

```text
Customer A

```

Needs

```text
Region

↓

US

```

Customer B

↓

Europe

Only option changes.

No fixture changes.

----------

# Option Fixtures Are Read-Only

Think of them as configuration.

Don't modify them.

Wrong

```ts
username='guest';

```

Treat them as immutable inputs.

----------

# Enterprise Example

```ts
type Options={

tenant:string;

environment:string;

user:string;

};

```

Tests

```ts
test('Example', async ({

tenant,

environment,

user

})=>{

});

```

Everything configurable.

----------

# Combining With Fixtures

```text
Option Fixture

↓

Login Fixture

↓

Page Fixture

↓

Test

```

Very powerful.

----------

# Example

```ts
loginPage:

async(

{

page,

username,

password

},

use

)=>{

const login=

new LoginPage(page);

await login.login(

username,

password

);

await use(login);

}

```

Changing:

```text
username

```

changes login automatically.

----------

# Project Override

Different project

↓

Different option.

Example

```ts
projects:[

{

name:'Admin',

use:{

username:'admin'

}

},

{

name:'Manager',

use:{

username:'manager'

}

}

]

```

One test suite.

Two users.

----------

# Enterprise Flow

```text
Project

↓

Option Fixture

↓

Login Fixture

↓

Authenticated Test

```

----------

# Common Mistakes

## ❌ Using `process.env` Everywhere

Instead of

```ts
process.env.USER

```

Use

```text
username

```

fixture.

----------

## ❌ Creating Fixture Instead Of Option

Don't write

```ts
async()=>{

return 'admin';

}

```

Simple values should be option fixtures.

----------

## ❌ Modifying Options

Treat option fixtures as immutable configuration.

----------

## ❌ Hardcoding

Don't write

```ts
login('admin')

```

Inject

```text
username

```

instead.

----------

# Option vs Fixture

Normal Fixture

Option Fixture

Creates objects

Provides values

Async lifecycle

Usually static values

Uses `await use()`

Declared as a value with `option: true`

Examples: `page`, `loginPage`

Examples: `username`, `tenant`, `theme`

----------

# Interview Questions

### Q1. What is an option fixture?

A fixture declared with:

```ts
option:true

```

that provides configuration values instead of creating objects.

----------

### Q2. Why use option fixtures?

To provide typed, overridable configuration that integrates with Playwright's fixture system.

----------

### Q3. Can option fixtures be overridden?

Yes.

Using:

```ts
test.use()

```

or project configuration.

----------

### Q4. Can normal fixtures depend on option fixtures?

Yes.

A common example is a login fixture depending on `username` and `password`.

----------

### Q5. Are option fixtures asynchronous?

Usually no.

They're typically static configuration values.

----------

### Q6. When should you use option fixtures?

For configuration such as:

-   User roles
    
-   Tenants
    
-   Environment names
    
-   Themes
    
-   API versions
    
-   Feature flags
    

----------

# Best Practices

-   Use option fixtures for configuration, not object creation.
    
-   Prefer option fixtures over scattered `process.env` lookups throughout your tests.
    
-   Override options with `test.use()` or project-level `use` configuration.
    
-   Keep option fixtures immutable.
    
-   Group related values into typed configuration objects when appropriate.
    
-   Let normal fixtures consume option fixtures to build configurable services.
    

----------

# ⭐ Enterprise Example

A real framework often combines option fixtures with normal fixtures.

```ts
type Options = {
  user: {
    username: string;
    password: string;
    role: string;
  };
};

type Fixtures = {
  loginPage: LoginPage;
};

export const test = base.extend<Options & Fixtures>({

  user: [
    {
      username: 'admin',
      password: 'Password123',
      role: 'administrator'
    },
    {
      option: true
    }
  ],

  loginPage: async ({ page, user }, use) => {

    const loginPage = new LoginPage(page);

    await loginPage.login(
      user.username,
      user.password
    );

    await use(loginPage);

  }

});

```

Now different projects can simply override the user:

```ts
projects: [
  {
    name: 'Manager',
    use: {
      user: {
        username: 'manager',
        password: 'Manager123',
        role: 'manager'
      }
    }
  }
]

```

The tests never change—they simply receive a `loginPage` that's authenticated as the appropriate user.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTAzNjE2NzE3MV19
-->