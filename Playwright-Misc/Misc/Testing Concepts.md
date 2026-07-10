# 🚀 Software Testing Concepts
## 1. The Testing Landscapes (Environments)
An **environment** is simply the playground where the software lives at any given moment. We divide these into two major worlds:
### 🚫 Non-Production (The Safe Zones)
These are internal servers where code is written, tested, and broken safely. Real users can never see or access these.
 * **Dev Environment (Development):** Where developers write the initial code. It is highly unstable because things are constantly changing.
 * **QA Environment (Quality Assurance):** This is **your** main playground. A stable build is deployed here so QA engineers can run tests without developers changing the code mid-test.
 * **Staging / UAT Environment:** A near-perfect clone of the real world. We use it for final checks and User Acceptance Testing (UAT) to ensure everything works exactly as expected before the big launch.
### 🌍 Production (The Real World)
 * **Production (Prod):** This is the live application that actual customers use (like the live Amazon website or the real Instagram app).
 * **Golden Rule :** *Never test or manipulate data directly in Production unless explicitly instructed under strict guidelines.* A mistake here impacts real users and real company revenue.
## 2. Platforms & Device Testing (Where the App Runs)
Users will access your software from hundreds of different setups. Your job is to make sure it looks and works great everywhere.
 * **Platform Testing:** Checking how the app behaves on different Operating Systems (like Windows, macOS, Linux, iOS, or Android).
 * **Mobile Device Testing:** Testing explicitly on smartphones and tablets. You'll check things like battery drain, how the app handles incoming phone calls while open, and touch gestures (taps, swipes).
 * **Compatibility Testing:** Ensuring the application works seamlessly across different hardware configurations, screen resolutions, and networks (like 4G vs. slow Wi-Fi).
 * **Cross-Browser Testing:** A subset of compatibility testing for web apps. You ensure the website behaves identically on Google Chrome, Safari, Mozilla Firefox, and Microsoft Edge.
## 3. Core Terminologies (How and When We Test)
These four terms trip up almost every newcomer. Use this matrix to understand exactly how they differ from each other.
| Testing Type | What is it? (The Simple Definition) | When do you run it? | Analogy |
|---|---|---|---|
| **Smoke Testing** | Testing the absolute core functionalities to ensure the app isn't "on fire" and is stable enough for deeper testing. | As soon as you receive a fresh software build. | Turning the ignition key in a car just to see if the engine starts. |
| **Sanity Testing** | A quick, focused check on a specific component or bug fix to verify that it works and hasn't completely broken that specific module. | After minor changes or bug fixes are deployed. | Testing just the headlights after replacing a bulb. |
| **Retesting** | Specifically executing the exact same test case that failed before, just to confirm that the developer actually fixed the bug. | After a developer marks a reported bug as "Fixed." | Retesting a leaky pipe after the plumber says they patched it. |
| **Regression Testing** | Testing the *unchanged* parts of the application to ensure that new code changes or bug fixes didn't accidentally break existing features. | After any new feature is added or bug is fixed. | Checking that your car's brakes still work perfectly after you upgrade the music system. |

<!--stackedit_data:
eyJoaXN0b3J5IjpbODQxODI4OTcwXX0=
-->