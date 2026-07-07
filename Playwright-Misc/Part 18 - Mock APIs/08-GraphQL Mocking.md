This chapter is extremely important because **GraphQL is fundamentally different from REST APIs**.

One of the biggest mistakes engineers make is trying to mock GraphQL the same way they mock REST.

With REST:

```text
GET /products
GET /orders
GET /customers

```

Each endpoint is different.

With GraphQL:

```text
POST /graphql

```

Everything goes through a **single endpoint**.

The only thing that changes is the **operation** inside the request body.

Understanding this difference is the key to successful GraphQL mocking.

----------

# Part 18 – Mock APIs

# Chapter 8 – GraphQL Mocking

----------

# Introduction

Many modern applications use GraphQL instead of REST APIs.

Popular platforms include:

-   Shopify
    
-   GitHub
    
-   Apollo
    
-   Hasura
    
-   Contentful
    
-   AWS AppSync
    

Unlike REST, GraphQL typically exposes a single endpoint.

Example

```text
POST /graphql

```

Whether you're fetching products, customers, or orders, the URL usually remains the same.

----------

# REST vs GraphQL

## REST

```text
GET /products

GET /orders

GET /customers

GET /users

```

Different endpoint for every resource.

----------

## GraphQL

```text
POST /graphql

```

Everything goes through one endpoint.

The request body determines the operation.

----------

# GraphQL Request Structure

A GraphQL request typically looks like:

```json
{
    "operationName": "GetProducts",

    "query": "...",

    "variables": {

        "category": "Laptop"

    }
}

```

Notice the important fields:

-   operationName
    
-   query
    
-   variables
    

These are what we'll use for mocking.

----------

# GraphQL Request Flow

```text
Browser

↓

POST /graphql

↓

Operation Name

↓

Resolver

↓

Response

```

Unlike REST,

the URL tells us very little.

----------

# Example Query

```graphql
query GetProducts {

    products {

        id

        name

    }

}

```

Request

```json
{
    "operationName": "GetProducts"
}

```

----------

# Example Mutation

```graphql
mutation CreateOrder {

    createOrder {

        id

    }

}

```

Request

```json
{
    "operationName": "CreateOrder"
}

```

----------

# Why GraphQL Mocking is Different

With REST,

we intercept:

```text
/products

```

With GraphQL,

we intercept:

```text
/graphql

```

Then inspect:

```text
operationName

```

----------

# Basic GraphQL Interception

```typescript
await page.route("**/graphql", async route => {

    const body =
        route.request().postDataJSON();

    console.log(body.operationName);

    await route.continue();

});

```

Output

```text
GetProducts

```

----------

# Mocking a Query

Suppose

```text
operationName

↓

GetProducts

```

Example

```typescript
await page.route("**/graphql", async route => {

    const request =
        route.request().postDataJSON();

    if (
        request.operationName ===
        "GetProducts"
    ) {

        await route.fulfill({

            json: {

                data: {

                    products: [

                        {

                            id: 1,

                            name: "Laptop"

                        }

                    ]

                }

            }

        });

        return;

    }

    await route.continue();

});

```

Notice the GraphQL response structure.

```json
{
    "data": {

    }
}

```

----------

# Mocking a Mutation

Example

```typescript
await page.route("**/graphql", async route => {

    const request =
        route.request().postDataJSON();

    if (

        request.operationName ===
        "CreateOrder"

    ) {

        await route.fulfill({

            json: {

                data: {

                    createOrder: {

                        id: 1001

                    }

                }

            }

        });

        return;

    }

    await route.continue();

});

```

----------

# GraphQL Response Format

GraphQL responses almost always follow this structure.

Success

```json
{
    "data": {

    }
}

```

Error

```json
{
    "errors":[

        {

            "message":"Unauthorized"

        }

    ]
}

```

Notice there is no

```json
{
    "success":true
}

```

unless your API specifically defines it.

----------

# Mocking GraphQL Errors

Example

```typescript
await route.fulfill({

    json: {

        errors: [

            {

                message:

                    "Access Denied"

            }

        ]

    }

});

```

Useful for

-   Authorization
    
-   Validation
    
-   Server errors
    

----------

# Mocking Based on Variables

Suppose

```graphql
query Product($id:Int){

}

```

Variables

```json
{

"id":10

}

```

Example

```typescript
await page.route("**/graphql", async route => {

    const body =
        route.request().postDataJSON();

    const id =
        body.variables.id;

    await route.fulfill({

        json: {

            data: {

                product: {

                    id,

                    name:
                        `Laptop ${id}`

                }

            }

        });

});

```

Now

```text
id=5

```

returns

```json
{

"id":5

}

```

----------

# Multiple Operations

One endpoint.

Many operations.

```text
POST /graphql

↓

GetProducts

↓

GetOrders

↓

GetCustomer

↓

UpdateUser

```

Example

```typescript
await page.route("**/graphql", async route => {

    const request =
        route.request().postDataJSON();

    switch (
        request.operationName
    ) {

        case "GetProducts":

            await route.fulfill({

                json: {

                    data: {

                        products: []

                    }

                }

            });

            break;

        case "GetOrders":

            await route.fulfill({

                json: {

                    data: {

                        orders: []

                    }

                }

            });

            break;

        default:

            await route.continue();

    }

});

```

----------

# Partial GraphQL Mocking

Enterprise teams often mock only one operation.

```text
/graphql

↓

GetProducts

↓

Mock

--------------

GetOrders

↓

Real Backend

--------------

Login

↓

Real Backend

```

This keeps important integrations intact while isolating the scenario under test.

----------

# Apollo Client Example

Apollo sends requests like:

```json
{

"operationName":"GetProducts",

"variables":{

"id":10

}

}

```

Your route handler works exactly the same.

----------

# Stateful GraphQL Mock

Example

```text
CreateOrder

↓

Order List Changes

```

Implementation

```typescript
const orders: any[] = [];

await page.route("**/graphql", async route => {

    const body =
        route.request().postDataJSON();

    if (

        body.operationName ===
        "CreateOrder"

    ) {

        orders.push({

            id: orders.length + 1

        });

        await route.fulfill({

            json: {

                data: {

                    createOrder:

                        orders.at(-1)

                }

            }

        });

        return;

    }

});

```

Now each mutation changes the mock state.

----------

# Enterprise GraphQL Architecture

Instead of

```text
Every Test

↓

Huge Switch Statement

```

Large frameworks use

```text
Tests

↓

GraphQL Mock Manager

↓

Operation Handlers

↓

Response Builders

```

----------

Example

```typescript
GraphQLMock.mockProducts(page);

GraphQLMock.mockOrders(page);

GraphQLMock.mockCustomer(page);

```

Each operation lives in its own module.

----------

# Suggested Folder Structure

```text
graphql/

├── operations/
│   ├── GetProducts.ts
│   ├── GetOrders.ts
│   ├── CreateOrder.ts
│   └── UpdateUser.ts
│
├── builders/
│   ├── ProductBuilder.ts
│   └── OrderBuilder.ts
│
├── data/
│   ├── products.json
│   └── orders.json
│
└── GraphQLMockManager.ts

```

This scales well as the number of operations grows.

----------

# REST vs GraphQL Mocking

REST

GraphQL

Match URL

Match `operationName`

Many endpoints

Usually one endpoint

Different routes

One route with branching logic

URL identifies resource

Request body identifies operation

----------

# Best Practices

-   Match GraphQL requests using `operationName`, not just the URL.
    
-   Keep response structures faithful to the GraphQL schema (`data` and `errors`).
    
-   Separate operation handlers into reusable modules.
    
-   Mock only the operations required by the current test.
    
-   Reuse builders and fixture data to keep responses consistent.
    

----------

# Common Mistakes

### ❌ Matching only `/graphql`

Every GraphQL request uses the same endpoint.

Always inspect:

```text
operationName

```

----------

### ❌ Returning REST-style responses

Incorrect

```json
{

"success":true

}

```

Correct

```json
{

"data":{

}

}

```

----------

### ❌ Ignoring variables

Many GraphQL operations depend heavily on request variables.

Read:

```typescript
body.variables

```

before generating the response.

----------

### ❌ Creating one enormous route handler

Split operation handling into dedicated modules or helper methods for maintainability.

----------

# Interview Questions

### Q1. Why is GraphQL mocking different from REST mocking?

Because GraphQL typically uses a single endpoint. The operation being executed is identified by the request body (`operationName`), not by the URL.

----------

### Q2. What field is most commonly used to identify a GraphQL operation?

`operationName`

----------

### Q3. What is the standard structure of a successful GraphQL response?

```json
{
  "data": {
    ...
  }
}

```

----------

### Q4. How are GraphQL errors typically returned?

```json
{
  "errors": [
    {
      "message": "..."
    }
  ]
}

```

----------

### Q5. Why should GraphQL operations be organized into separate handlers?

It improves readability, reusability, and maintainability, especially in large applications with many queries and mutations.

----------

# Summary

GraphQL mocking differs from REST because all operations typically share a single endpoint. Effective GraphQL mocking relies on inspecting the request body—particularly the `operationName` and `variables`—to determine which response to return. By organizing operation handlers, builders, and mock data into reusable modules, teams can create scalable, maintainable GraphQL test frameworks that closely resemble real production behavior.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjk5NjEwNzg3XX0=
-->