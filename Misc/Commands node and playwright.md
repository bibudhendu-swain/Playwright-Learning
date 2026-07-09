This end-to-end command blueprint serves as a logical structured guide for your colleagues. It maps the complete progression from setting up Node.js on a machine all the way through local execution, interactive debugging, and final CI/CD deployment.
## Phase 1: Environment & Node.js Setup
Before touching Playwright, the team needs a solid JavaScript/TypeScript runtime.
### 1. Verification Commands
Always check if Node.js is already installed.
```bash
# Verify if Node.js is installed and check its version
node -v

# Verify if npm (Node Package Manager) is available
npm -v

```
### 2. Upgrading Node.js / Working with Multiple Versions
If they need to upgrade or switch between multiple versions (e.g., if another project requires an older version, but Playwright needs modern Node.js), recommend using **NVM** (Node Version Manager).
```bash
# Install a specific LTS version of Node.js
nvm install 22

# Switch to the specified Node.js version for the current terminal session
nvm use 22

# Set the global default version of Node.js for the machine
nvm alias default 22

```
## Phase 2: Project Initialization
This is how your team creates the scaffolding for a standardized Playwright TypeScript framework.
```bash
# Navigate to your automation workspace
cd path/to/your/workspace

# Initialize a brand-new Playwright project environment
# (This starts an interactive wizard wizard prompting for TypeScript, folder names, etc.)
npm init playwright@latest

```
## Phase 3: Package & Binary Management
Once the project is initialized, managing the lifecycle of dependencies and browser binaries becomes a routine task.
```bash
# Install a new custom dependency and save it as a development-only tool (e.g., prettier)
npm install --save-dev prettier

# Explicitly download or update the specific browser binaries used by Playwright
npx playwright install

# Download browser binaries AND install missing OS-level system dependencies (Critical for Linux/CI)
npx playwright install --with-deps

# Upgrade the core Playwright framework version to the latest release
npm install -D @playwright/test@latest

```
## Phase 4: Local Test Execution CLI
The core commands used daily to run scripts locally. Teach your team the power of CLI flags.
```bash
# Run all tests across all configured browsers headlessly
npx playwright test

# Run tests showing the browser windows visually
npx playwright test --headed

# Run a single specific test spec file
npx playwright test tests/login.spec.ts

# Filter and run only the tests matching a particular string text or tag in their title
npx playwright test -g "admin dashboard"

# Run tests exclusively on a single browser engine profile defined in your config
npx playwright test --project=chromium

# Run tests and limit the parallel execution to a specific number of worker threads
npx playwright test --workers=3

# Re-run only the test cases that failed in the immediate previous test execution run
npx playwright test --last-failed

```
## Phase 5: Advanced Tooling & Debugging
Playwright's world-class diagnostic tools are accessed straight through these CLI entry points.
```bash
# Launch the graphical UI Mode (live test trees, watch mode, time-travel step snapshots)
npx playwright test --ui

# Launch Codegen: Record browser interactions visually and auto-generate TypeScript code
npx playwright codegen https://example.com

# Open the Playwright Inspector to step through code execution line-by-line manually
npx playwright test tests/login.spec.ts --debug

# Instantly serve and view the full local static HTML test report generated from the last run
npx playwright show-report

# Open a recorded trace zip file directly inside the offline Trace Viewer app
npx playwright show-trace path/to/trace.zip

```
## Phase 6: Pipeline Execution (CI/CD)
When executing tests inside server pipelines (like Jenkins, GitHub Actions, or GitLab CI), the execution parameters shift to maximize reliability.
```bash
# Clean Install: Installs packages relying strictly on package-lock.json (deterministic)
npm ci

# Execute the test suite inside headless environments while forcing trace capture on retry phases
npx playwright test --trace on-first-retry --forbid-only

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTMyNjcxNDIwM119
-->