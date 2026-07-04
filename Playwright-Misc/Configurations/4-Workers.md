# Part 4 – Parallel Execution (`workers`, `fullyParallel`, `test.describe.configure()`, Serial vs Parallel)

Parallel execution is one of Playwright's biggest advantages over many older automation frameworks. By running tests concurrently across multiple **workers** (separate Node.js processes), Playwright can significantly reduce execution time while maintaining test isolation.

----------

# How Playwright Executes Tests

By default, Playwright uses multiple **worker processes**.

Each worker:

-   Is a separate Node.js process.
    
-   Launches its own browser instance (or reuses it within that worker, depending on your fixtures).
    
-   Executes tests independently.
    
-   Cannot share variables or memory with other workers.
    

Example:

```text
4 Workers

Worker 1
   ├── login.spec.ts
   └── profile.spec.ts

Worker 2
   ├── cart.spec.ts
   └── checkout.spec.ts

Worker 3
   ├── api.spec.ts

Worker 4
   ├── search.spec.ts

```

----------

# 1. `workers`

## Purpose

Controls the **maximum number of worker processes** Playwright uses.

----------

## Default

Locally:

```text
≈ Number of CPU cores

```

In CI (many templates):

```text
50% of available CPU cores

```

It's common to explicitly configure this for predictable behavior.

----------

## Configuration

```ts
export default defineConfig({
  workers: 4
});

```

Playwright will run up to four workers in parallel.

----------

## Example

Suppose you have:

```text
10 test files

Each takes 20 seconds

```

### 1 Worker

```text
20 × 10 = 200 seconds

```

### 2 Workers

```text
≈100 seconds

```

### 5 Workers

```text
≈40 seconds

```

(Actual execution depends on test duration and scheduling.)

----------

## Disable Parallelism

```ts
workers: 1

```

Equivalent CLI:

```bash
npx playwright test --workers=1

```

Useful for:

-   debugging
    
-   investigating flaky tests
    
-   applications with limited parallel support
    

----------

## Override from CLI

Configuration:

```ts
workers: 8

```

Run:

```bash
npx playwright test --workers=2

```

The CLI option overrides the configuration.

----------

# How Workers Pick Tests

Suppose:

```text
tests/

login.spec.ts
checkout.spec.ts
cart.spec.ts
profile.spec.ts
orders.spec.ts
payment.spec.ts

```

With:

```ts
workers: 3

```

Execution might look like:

```text
Worker 1

login.spec.ts

orders.spec.ts


Worker 2

checkout.spec.ts

profile.spec.ts


Worker 3

cart.spec.ts

payment.spec.ts

```

Playwright dynamically assigns the next available file to an idle worker.

----------

# Worker Restart on Failure

Suppose Worker 2 crashes.

Playwright automatically starts:

```text
Worker 2 (new process)

```

Remaining tests continue executing.

This isolation improves suite reliability.

----------

# 2. `fullyParallel`

## Purpose

By default, Playwright parallelizes **test files**.

Inside a single file, tests run sequentially.

Example:

```ts
test('Login', async () => {});

test('Checkout', async () => {});

test('Payment', async () => {});

```

Execution:

```text
Login

↓

Checkout

↓

Payment

```

----------

Enable:

```ts
fullyParallel: true

```

Now:

```text
Login

Checkout

Payment

```

can execute simultaneously (subject to available workers).

----------

## Configuration

```ts
export default defineConfig({
  fullyParallel: true
});

```

----------

## When to Use

Suitable when:

-   every test is independent
    
-   each test creates its own data
    
-   no shared mutable state exists
    

----------

## When Not to Use

Avoid if tests:

-   depend on execution order
    
-   reuse shared test data
    
-   modify the same records
    
-   share authenticated sessions in unsafe ways
    

----------

# Serial Execution

Sometimes tests must run in order.

Example:

```text
1. Create Customer

↓

2. Update Customer

↓

3. Delete Customer

```

Configuration:

```ts
test.describe.configure({
  mode: 'serial'
});

```

Example:

```ts
test.describe('Customer lifecycle', () => {

  test.describe.configure({
    mode: 'serial'
  });

  test('Create', async () => {});

  test('Update', async () => {});

  test('Delete', async () => {});

});

```

Execution:

```text
Create

↓

Update

↓

Delete

```

If **Create** fails:

```text
Update

Delete

```

are skipped.

----------

# Parallel Suite

Instead:

```ts
test.describe.configure({
  mode: 'parallel'
});

```

Now every test inside the `describe` block can run simultaneously.

----------

Example:

```ts
test.describe('Smoke', () => {

  test.describe.configure({
    mode: 'parallel'
  });

  test('Login', async () => {});

  test('Search', async () => {});

  test('Checkout', async () => {});

});

```

Execution:

```text
Login

Search

Checkout

```

all at once (depending on worker availability).

----------

# Default Mode

If you don't specify anything:

```text
mode: 'default'

```

Tests execute sequentially within the file.

----------

# File-Level Parallelism vs Test-Level Parallelism

Without `fullyParallel`:

```text
Worker 1

login.spec.ts

Test A

↓

Test B

↓

Test C

```

With `fullyParallel`:

```text
Worker 1

Test A

Worker 2

Test B

Worker 3

Test C

```

(Subject to scheduling and available workers.)

----------

# Projects and Parallelism

Suppose:

```ts
projects: [
  {
    name: 'Chromium'
  },
  {
    name: 'Firefox'
  }
]

```

Each project has its own workers.

Example:

```text
Chromium

Worker 1

Worker 2


Firefox

Worker 1

Worker 2

```

Parallelism applies independently to each project.

----------

# Worker Fixtures

Worker-scoped fixtures are created **once per worker**.

Example:

```ts
test.extend({

  database: [async ({}, use) => {

    // connect once

    await use(connection);

  }, { scope: 'worker' }]

});

```

If:

```text
workers = 4

```

The fixture initializes four times—once for each worker.

----------

# Common Mistakes

## Mistake 1

Assuming workers share variables.

```ts
let counter = 0;

```

Each worker has its own memory space. Changes in one worker are not visible to others.

----------

## Mistake 2

Using the same test account across parallel workers.

Example:

```text
Worker 1

Deletes customer


Worker 2

Updates same customer

```

This can lead to race conditions and flaky tests.

Use unique test data or isolated environments.

----------

## Mistake 3

Setting:

```ts
workers: 32

```

on a machine with:

```text
4 CPU cores

```

This often increases context switching and can reduce overall performance.

----------

# Enterprise Configuration Example

```ts
export default defineConfig({

  workers: process.env.CI ? 4 : undefined,

  fullyParallel: true,

  retries: process.env.CI ? 2 : 0

});

```

This allows local machines to use Playwright's default worker count while limiting concurrency in CI.

----------

# Performance Recommendations

Machine

Recommended Workers

Laptop (4 cores)

2–4

Laptop (8 cores)

4–8

CI Agent (8 cores)

4–6

CI Agent (16 cores)

8–12

Always benchmark with your application; the optimal value depends on browser startup cost, backend capacity, and test design.

----------

# Interview Questions

### Q1. What is a Playwright worker?

A worker is a separate Node.js process that executes tests independently with its own browser context and memory.

----------

### Q2. Do workers share global variables?

No. Each worker runs in a separate process with isolated memory.

----------

### Q3. What does `workers: 1` do?

It disables parallel execution and runs tests sequentially.

----------

### Q4. What is the difference between `workers` and `fullyParallel`?

`workers`

`fullyParallel`

Controls how many worker processes run

Controls whether tests within the same file can run in parallel

Affects concurrency across files

Affects concurrency within a file

----------

### Q5. When should you use serial mode?

When tests have intentional dependencies, such as a create → update → delete workflow, or when they share state that cannot be isolated.

----------

### Q6. If one test fails in a serial suite, what happens?

The remaining tests in that serial suite are skipped.

----------

### Q7. Is `fullyParallel` enabled by default?

No. By default, Playwright runs test files in parallel, but tests within the same file execute sequentially.

----------

# Best Practices

-   Design tests to be independent so they can safely run in parallel.
    
-   Use `workers: 1` only for debugging or when an application truly cannot support concurrent execution.
    
-   Avoid serial mode unless there is a genuine dependency between tests.
    
-   Use worker-scoped fixtures for expensive setup (e.g., database connections or authentication) that can be reused within a worker.
    
-   Avoid sharing test accounts or mutable data across parallel tests.
    
-   Tune the number of workers based on both your hardware and the application's capacity, not just the CPU count.
    

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTg0NzgxOTg1NV19
-->