This is **one of the most misunderstood topics in Playwright**.

Almost every beginner eventually asks questions like:

-   Why can't I use my local variable inside `evaluate()`?
    
-   Why can't I return a DOM element?
    
-   What is "serialization"?
    
-   What is a `JSHandle`?
    
-   Why does Playwright say _"Unexpected value"_?
    

Once you understand this chapter, **90% of `evaluate()` confusion disappears**.

----------

# Part 13 – Evaluating JavaScript

# Chapter 2 – Evaluation Arguments & Serialization

----------

# Introduction

Earlier we learned:

```typescript
const title = await page.evaluate(() => {
    return document.title;
});

```

Simple enough.

But what about this?

```typescript
const username = "admin";

await page.evaluate(() => {
    console.log(username);
});

```

It **fails**.

Why?

Because the browser **cannot see variables from Node.js**.

Understanding this requires understanding **serialization**.

----------

# Two Separate Worlds

Remember this diagram.

```text
+--------------------------------------+
|        Node.js (Playwright Test)     |
|                                      |
| const username = "admin";            |
| const total = 100;                   |
+-------------------▲------------------+
                    │
          Serialization
                    │
                    ▼
+--------------------------------------+
|        Browser (evaluate())          |
|                                      |
| window                               |
| document                             |
| localStorage                         |
+--------------------------------------+

```

These environments **cannot directly share memory**.

They communicate by **serializing** data.

----------

# What is Serialization?

Serialization means converting data into a transferable format.

Example:

Node.js

```typescript
const user = {
    id: 10,
    name: "John"
};

```

Playwright serializes it,

sends it to the browser,

then reconstructs it there.

Think of it as:

```text
Node Object

↓

Serialized

↓

Browser Object

```

----------

# Passing Arguments to `evaluate()`

Instead of trying to access outer variables,

pass them explicitly.

Wrong

```typescript
const username = "admin";

await page.evaluate(() => {

    console.log(username);

});

```

Error:

```text
username is not defined

```

----------

Correct

```typescript
const username = "admin";

await page.evaluate((name) => {

    console.log(name);

}, username);

```

Now

```text
admin

```

----------

# Syntax

```typescript
await page.evaluate(

(argument) => {

},

argument

);

```

The second argument is serialized and passed into the browser.

----------

# Multiple Values

You can pass an object.

```typescript
await page.evaluate(

(user) => {

    console.log(user.name);

    console.log(user.age);

},

{

    name: "John",

    age: 30

}

);

```

----------

# Arrays

```typescript
await page.evaluate(

(numbers) => {

    return numbers.reduce(

        (a, b) => a + b

    );

},

[10, 20, 30]

);

```

Returns:

```text
60

```

----------

# Complex Objects

```typescript
const user = {

    id: 100,

    name: "Alice",

    role: "Admin"

};

await page.evaluate(

(data) => {

    console.log(data.role);

},

user

);

```

Works because it is serializable.

----------

# Returning Values

Returning data works the same way.

```typescript
const result = await page.evaluate(() => {

    return {

        title: document.title,

        url: location.href

    };

});

```

Playwright serializes the object back to Node.js.

----------

# What Can Be Serialized?

These values can safely cross the boundary.

Type

Supported

String

✅

Number

✅

Boolean

✅

Null

✅

Array

✅

Plain Object

✅

Date

✅ (serialized as a Date value)

> **Note:** `Date` objects are preserved by Playwright's serialization. However, if you serialize data yourself (for example with `JSON.stringify()`), dates become strings.

----------

# Unsupported Values

These **cannot** be transferred directly.

Type

Supported

DOM Element

❌

Function

❌

Promise

❌ (return its resolved value instead)

Class Instance

❌ (unless represented as plain data)

----------

# Why DOM Elements Cannot Be Returned

Suppose

```typescript
await page.evaluate(() => {

    return document.querySelector("#login");

});

```

Why does it fail?

Because

```text
DOM Element

↓

Lives Inside Browser

↓

Cannot Exist

↓

Inside Node.js

```

The DOM node belongs to the browser process.

----------

# Correct Approach

Instead of returning the element,

return its value.

```typescript
const text = await page.evaluate(() => {

    return document
        .querySelector("h1")
        ?.textContent;

});

```

----------

# Another Example

Instead of

```typescript
return document.querySelector("input");

```

Return

```typescript
return document
    .querySelector("input")
    ?.value;

```

----------

# Passing Multiple Values

Best practice:

```typescript
await page.evaluate(

(data) => {

    console.log(data.user);

    console.log(data.password);

},

{

    user: "admin",

    password: "secret"

}

);

```

----------

# Destructuring

```typescript
await page.evaluate(

({ username, age }) => {

    console.log(username);

    console.log(age);

},

{

    username: "John",

    age: 30

}

);

```

Very readable.

----------

# Nested Objects

```typescript
await page.evaluate(

(company) => {

    console.log(

        company.address.city

    );

},

{

    address: {

        city: "London"

    }

}

);

```

----------

# Passing Large Objects

Possible.

But avoid passing:

```text
100 MB Object

```

Only send the data you actually need.

----------

# Using Browser Objects

Inside `evaluate()`

you have access to

```text
window

document

navigator

location

history

localStorage

sessionStorage

```

Exactly like browser JavaScript.

----------

# Real-World Example – Theme Validation

```typescript
const expectedTheme = "dark";

const actual = await page.evaluate(

(expected) => {

    return localStorage.getItem("theme") === expected;

},

expectedTheme

);

expect(actual).toBeTruthy();

```

----------

# Real-World Example – Product Count

```typescript
const expected = 10;

const actual = await page.evaluate(

(count) => {

    return document
        .querySelectorAll(".product")
        .length === count;

},

expected

);

expect(actual).toBeTruthy();

```

----------

# Real-World Example – Dynamic Configuration

```typescript
const config = {

    retries: 5,

    timeout: 10000

};

await page.evaluate(

(settings) => {

    window.appConfig = settings;

},

config

);

```

----------

# Serialization Flow

```text
Node.js Object

↓

Serialize

↓

Browser

↓

Run JavaScript

↓

Serialize Result

↓

Node.js

```

Everything crossing the boundary must be serializable.

----------

# Common Serialization Errors

## Error 1

```typescript
const user = document;

```

Fails.

There is no `document` in Node.js.

----------

## Error 2

```typescript
return document.body;

```

Fails.

DOM elements cannot be returned.

----------

## Error 3

```typescript
return function () {};

```

Functions cannot be serialized.

----------

# Best Practices

-   Pass data explicitly through the second argument to `evaluate()`.
    
-   Return only serializable values.
    
-   Prefer plain objects over complex class instances.
    
-   Return properties (text, values, attributes) instead of DOM elements.
    
-   Keep data transferred between Node.js and the browser as small as practical.
    

----------

# Common Mistakes

### ❌ Using outer variables directly

```typescript
await page.evaluate(() => {

    console.log(username);

});

```

Always pass the value as an argument.

----------

### ❌ Returning DOM elements

```typescript
return document.querySelector("#save");

```

Return the element's data instead.

----------

### ❌ Returning functions

```typescript
return () => {};

```

Functions cannot be serialized.

----------

### ❌ Passing very large objects

Only send the data needed for the browser-side logic to avoid unnecessary overhead.

----------

# Interview Questions

### Q1. Why can't `evaluate()` access local variables directly?

Because `evaluate()` runs in the browser, while local variables exist in the Node.js test environment. Values must be passed explicitly through the evaluation argument.

----------

### Q2. What is serialization?

Serialization is the process of converting data into a transferable format so it can move between the Node.js test process and the browser process.

----------

### Q3. Can `evaluate()` return a DOM element?

No. DOM elements belong to the browser process and cannot be transferred directly. Return serializable data or use handles (covered in the next chapter).

----------

### Q4. What data types can be passed to `evaluate()`?

Common supported types include strings, numbers, booleans, `null`, arrays, plain objects, and dates.

----------

### Q5. What is the best way to pass multiple values?

Wrap them in a single object:

```typescript
await page.evaluate(
    ({ username, password }) => {
        console.log(username);
        console.log(password);
    },
    {
        username: "admin",
        password: "secret"
    }
);

```

----------

# Summary

`page.evaluate()` bridges two isolated JavaScript environments: your Playwright test running in Node.js and the browser page running in its own process. Because these environments cannot share memory directly, Playwright serializes values that cross the boundary. Understanding what can and cannot be serialized—and how to pass arguments explicitly—is essential for using `evaluate()` effectively and avoiding common errors.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbODg5NzQ5NzE0XX0=
-->