### Module 1 – Project Structure Migration

**Visual:** Two project trees side by side.

**Left:** Selenium Java (Maven)

```text
Selenium Framework
├── src/main/java
│   ├── pages
│   ├── utils
│   ├── base
│   └── drivers
├── src/test/java
│   ├── tests
│   └── runners
├── pom.xml
└── testng.xml
```

**Right:** Playwright TypeScript

```text
Playwright Framework
├── pages
├── tests
├── fixtures
├── utils
├── playwright.config.ts
├── package.json
└── tsconfig.json
```

Then below the diagram, a mapping table:

| Selenium      | Playwright                      |
| ------------- | ------------------------------- |
| pom.xml       | package.json                    |
| Maven         | npm                             |
| TestNG        | Playwright Test                 |
| testng.xml    | playwright.config.ts            |
| WebDriver     | Browser / BrowserContext / Page |
| Java Packages | TypeScript Modules              |

---

### Module 2 – Coding Translation

Show exactly the same login scenario.

| Selenium Java           | Playwright TypeScript       |
| ----------------------- | --------------------------- |
| driver.findElement(...) | page.locator(...)           |
| sendKeys()              | fill()                      |
| click()                 | click()                     |
| getText()               | textContent() / innerText() |
| getAttribute()          | getAttribute()              |

Not just syntax, but highlight *why* Playwright does it differently.

---

### Module 3 – Locator Translation

| Selenium       | Playwright            |
| -------------- | --------------------- |
| By.id          | getByTestId / locator |
| By.xpath       | locator               |
| By.cssSelector | locator               |
| By.name        | getByLabel / locator  |
| By.linkText    | getByRole             |

Then add a note like:

> "In Selenium we ask *how* to locate an element. In Playwright we prefer locating it the way a user perceives it (Role, Label, Text)."

---

### Module 4 – Actions Translation

| Selenium                | Playwright     |
| ----------------------- | -------------- |
| sendKeys()              | fill()         |
| clear()                 | clear()        |
| submit()                | press('Enter') |
| Actions.moveToElement() | hover()        |
| Select class            | selectOption() |

---

### Module 5 – Assertions

| Selenium            | Playwright                    |
| ------------------- | ----------------------------- |
| Assert.assertEquals | expect().toHaveText()         |
| isDisplayed()       | expect(locator).toBeVisible() |
| isEnabled()         | expect(locator).toBeEnabled() |
| isSelected()        | expect(locator).toBeChecked() |

---

### Module 6 – Waits

This deserves its own visual.

```
Selenium

click()
↓
Thread.sleep()
↓
Explicit Wait
↓
ExpectedConditions
↓
Interaction
```

versus

```
Playwright

click()
↓
Auto Wait
↓
Actionability Checks
↓
Interaction
```

This is probably the biggest mindset shift.

---

### Module 7 – Framework Components

A high-level architecture diagram mapping:

```
BaseTest        → Fixtures
WebDriver       → Page
DriverFactory   → BrowserContext
PageFactory     → Page Objects
TestNG          → Playwright Test
```

---
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEzOTE4MjQ5NTldfQ==
-->