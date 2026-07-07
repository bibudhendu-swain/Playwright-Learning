This chapter is one of the **most practical** browser mocking topics because almost every enterprise application behaves differently based on the browser environment.

Applications frequently change behavior based on:

-   🌐 Online / Offline status
    
-   🌍 Timezone
    
-   🌎 Locale
    
-   🗣️ Language
    
-   🌙 Light/Dark mode
    
-   📱 Mobile/Desktop device
    
-   💻 User Agent
    
-   📺 Screen Size
    

Playwright allows us to simulate all of these without changing the actual operating system.

----------

# Part 17 – Mock Browser APIs

# Chapter 7 – Browser State (Online/Offline, Timezone, Locale, Language, Color Scheme & User Agent)

----------

# Introduction

Every browser exposes information about its environment.

For example:

```text
Current Timezone

↓

Asia/Kolkata

```

or

```text
Browser Language

↓

English

```

or

```text
Network Status

↓

Offline

```

Applications often use this information to customize behavior.

Playwright allows us to control these browser characteristics for reliable testing.

----------

# Browser State Overview

A browser's state includes:

Property

Example

Online / Offline

Connected or disconnected

Timezone

Asia/Kolkata

Locale

en-IN

Language

English

Color Scheme

Dark Mode

User Agent

Chrome Desktop

Viewport

1920 × 1080

Device

iPhone 15

----------

# Browser State Flow

```text
Application

↓

Browser Environment

↓

Timezone

Locale

Language

Network

Color Scheme

↓

Application Behavior

```

----------

# Online vs Offline

Many applications need internet connectivity.

Examples:

-   Gmail
    
-   Teams
    
-   Slack
    
-   PWAs
    
-   Banking Apps
    

Normally

```text
Application

↓

Internet

↓

Server

```

Offline

```text
Application

↓

No Internet

↓

Offline Mode

```

----------

# Simulating Offline Mode

Playwright allows changing the browser's network state.

```typescript
await context.setOffline(true);

```

Now every request fails as if the network connection were lost.

----------

# Restoring Online Mode

```typescript
await context.setOffline(false);

```

Network connectivity returns.

----------

# Offline Workflow

```text
Application

↓

Offline

↓

Cached Content

↓

Offline Banner

↓

Reconnect

↓

Sync

```

----------

# Testing Offline Banner

Example

```typescript
await context.setOffline(true);

await page.reload();

await expect(

page.getByText("You're Offline")

).toBeVisible();

```

----------

# Testing Retry

```typescript
await context.setOffline(true);

await page.click("text=Retry");

await expect(

page.getByText("Unable to Connect")

).toBeVisible();

await context.setOffline(false);

await page.click("text=Retry");

await expect(

page.getByText("Connected")

).toBeVisible();

```

----------

# Timezone

Applications often display:

-   Current Date
    
-   Business Hours
    
-   Delivery Times
    
-   Scheduled Meetings
    

Timezone affects these values.

----------

# Setting Timezone

Timezone is configured when creating the browser context.

```typescript
const context = await browser.newContext({

    timezoneId:

        "Asia/Kolkata"

});

```

----------

# Common Timezones

Region

Timezone

India

Asia/Kolkata

London

Europe/London

New York

America/New_York

Los Angeles

America/Los_Angeles

Tokyo

Asia/Tokyo

Sydney

Australia/Sydney

----------

# Testing Multiple Timezones

```typescript
const context =
await browser.newContext({

    timezoneId:

        "America/New_York"

});

```

Verify

```text
09:00 AM

```

instead of

```text
06:30 PM

```

----------

# Locale

Locale determines:

-   Date format
    
-   Number format
    
-   Currency formatting
    
-   Sorting rules
    

----------

# Setting Locale

```typescript
const context =
await browser.newContext({

    locale:

        "fr-FR"

});

```

Application now behaves as if running in France.

----------

# Common Locales

Locale

Country

en-US

United States

en-IN

India

en-GB

United Kingdom

fr-FR

France

de-DE

Germany

ja-JP

Japan

----------

# Example

Date

US

```text
12/31/2026

```

France

```text
31/12/2026

```

Same application.

Different locale.

----------

# Language

Many applications support multiple languages.

Example

```text
English

↓

French

↓

German

↓

Japanese

```

Language preferences are commonly influenced by the browser's locale and `Accept-Language` headers.

----------

# Example

```typescript
const context =
await browser.newContext({

    locale:

        "de-DE"

});

```

Verify

```text
Willkommen

```

instead of

```text
Welcome

```

> **Note:** Whether the application changes language depends on how it determines the user's preferred language. Some applications use the browser locale, others rely on server-side settings or user preferences.

----------

# Color Scheme

Modern applications support:

```text
Light Mode

↓

Dark Mode

```

Playwright can emulate the user's preferred color scheme.

----------

# Light Mode

```typescript
const context =
await browser.newContext({

    colorScheme:

        "light"

});

```

----------

# Dark Mode

```typescript
const context =
await browser.newContext({

    colorScheme:

        "dark"

});

```

----------

# Testing Dark Mode

```typescript
await expect(

page.locator("body")

).toHaveCSS(

    "background-color",

    "rgb(18, 18, 18)"

);

```

----------

# User Agent

Applications sometimes detect:

-   Browser
    
-   Device
    
-   Platform
    

through the User-Agent header.

----------

# Setting User Agent

```typescript
const context =
await browser.newContext({

    userAgent:

"Mozilla/5.0 Test Browser"

});

```

Useful for:

-   Device-specific UI
    
-   Legacy browser handling
    
-   Analytics testing
    

----------

# Reading User Agent

```typescript
const ua =
await page.evaluate(() =>

navigator.userAgent

);

console.log(ua);

```

----------

# Viewport

Screen size affects responsive layouts.

```typescript
const context =
await browser.newContext({

    viewport: {

        width: 390,

        height: 844

    }

});

```

----------

# Device Emulation

Playwright provides predefined device profiles.

```typescript
import { devices } from '@playwright/test';

const context =
await browser.newContext({

    ...devices['iPhone 15']

});

```

The emulated device includes:

-   Viewport
    
-   User Agent
    
-   Touch support
    
-   Device scale factor
    
-   Mobile behavior
    

----------

# Testing Responsive UI

Desktop

```text
Sidebar Visible

```

Mobile

```text
Hamburger Menu

```

Example

```typescript
await expect(

page.getByRole("button", {

    name:"Menu"

})

).toBeVisible();

```

----------

# Enterprise Example – Banking

```text
India

↓

₹

----------------

USA

↓

$

```

Locale determines currency formatting.

----------

# Enterprise Example – Calendar

```text
New York

↓

Meeting

↓

09:00

----------------

London

↓

14:00

```

Timezone changes meeting display.

----------

# Enterprise Example – PWA

```text
Online

↓

Sync

↓

Offline

↓

Cached Data

↓

Reconnect

↓

Sync Pending Changes

```

Perfect for testing offline-first applications.

----------

# Enterprise Example – E-commerce

```text
Dark Mode

↓

Product Page

↓

Checkout

↓

Confirmation

```

Verify visual consistency across themes.

----------

# Browser Environment Factory

Instead of

```typescript
await browser.newContext({

    locale:"en-US",

    timezoneId:"America/New_York",

    colorScheme:"dark"

});

```

Create reusable factories.

```typescript
class BrowserEnvironment {

    static async us(browser){

        return browser.newContext({

            locale:"en-US",

            timezoneId:"America/New_York"

        });

    }

}

```

Usage

```typescript
const context =

await BrowserEnvironment.us(browser);

```

----------

# Suggested Folder Structure

```text
helpers/

├── BrowserEnvironment.ts

├── DeviceFactory.ts

├── ThemeHelper.ts

├── LocaleFactory.ts

└── NetworkHelper.ts

```

----------

# Browser State Summary

Feature

API

Offline

`context.setOffline()`

Timezone

`timezoneId`

Locale

`locale`

Color Scheme

`colorScheme`

User Agent

`userAgent`

Viewport

`viewport`

Device

`devices[]`

----------

# Best Practices

-   Configure browser state before creating pages whenever the setting is context-based.
    
-   Use realistic combinations of locale, timezone, and device profiles.
    
-   Test offline and reconnect scenarios for applications that cache data.
    
-   Use built-in Playwright device descriptors instead of manually recreating mobile configurations.
    
-   Create reusable browser environment factories for common business regions and devices.
    

----------

# Common Mistakes

### ❌ Changing timezone after context creation

Unlike geolocation, the timezone cannot be changed for an existing browser context.

Create a new context instead.

----------

### ❌ Confusing Locale with Language

Locale affects:

-   Date format
    
-   Number format
    
-   Regional preferences
    

Language affects:

-   Display text
    

Applications may use one, the other, or both.

----------

### ❌ Forgetting Offline Recovery

Always test:

```text
Offline

↓

Reconnect

↓

Application Recovers

```

instead of testing only the offline state.

----------

### ❌ Creating Custom Mobile Profiles

Prefer:

```typescript
devices["iPhone 15"]

```

over manually copying viewport and user agent values.

----------

# Interview Questions

### Q1. How do you simulate offline mode in Playwright?

```typescript
await context.setOffline(true);

```

----------

### Q2. Can the timezone be changed after creating a browser context?

No. The timezone is fixed when the browser context is created. Create a new context to test a different timezone.

----------

### Q3. What is the difference between locale and timezone?

-   **Locale** controls regional formatting such as dates, numbers, and currencies.
    
-   **Timezone** controls how dates and times are interpreted and displayed.
    

----------

### Q4. How do you emulate Dark Mode?

```typescript
const context =
await browser.newContext({

    colorScheme: "dark"

});

```

----------

### Q5. Why should you use Playwright's built-in device descriptors?

They provide a realistic combination of viewport, user agent, touch support, and other device characteristics, reducing manual configuration and improving test accuracy.

----------

# Summary

Browser state simulation is one of Playwright's most powerful capabilities for testing modern web applications. By controlling network connectivity, timezone, locale, color scheme, user agent, viewport, and device characteristics, you can validate how applications behave across different environments without modifying the operating system or using physical devices. Centralizing these configurations into reusable environment factories keeps enterprise test suites clean, consistent, and easy to maintain.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbODg0OTgzNjQyXX0=
-->