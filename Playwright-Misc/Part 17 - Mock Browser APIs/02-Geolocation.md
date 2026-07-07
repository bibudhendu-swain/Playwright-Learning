Geolocation is probably the **most commonly used Browser API** in enterprise applications.

Applications like:

-   Google Maps
    
-   Uber
    
-   Swiggy
    
-   Zomato
    
-   Amazon
    
-   Delivery apps
    
-   Banking apps
    
-   Travel apps
    
-   Weather apps
    

all depend on it.

Fortunately, Playwright makes geolocation testing extremely simple.

----------

# Part 17 – Mock Browser APIs

# Chapter 2 – Geolocation

----------

# Introduction

Many modern web applications need to know **where the user is located**.

For example:

```text
Open App

↓

Request Current Location

↓

Browser GPS

↓

Latitude & Longitude

↓

Nearby Results

```

Examples include:

-   Nearby restaurants
    
-   Nearby stores
    
-   Delivery availability
    
-   Taxi pickup
    
-   Weather
    
-   Regional pricing
    
-   Language selection
    

Testing these features manually would require physically moving to different locations.

Playwright allows us to simulate any location in the world.

----------

# What is Geolocation?

A browser's geolocation consists primarily of:

-   Latitude
    
-   Longitude
    
-   Accuracy
    

Example

```text
Latitude

28.6139

Longitude

77.2090

Accuracy

50 meters

```

These values are exposed through the browser's Geolocation API.

----------

# Browser Geolocation Flow

Normally

```text
Application

↓

Browser

↓

Operating System

↓

GPS / WiFi

↓

Coordinates

```

With Playwright

```text
Application

↓

Browser

↓

Playwright

↓

Mock Coordinates

```

No real GPS is required.

----------

# Browser Context Configuration

Geolocation is configured when creating the browser context.

```typescript
const context = await browser.newContext({

    geolocation: {

        latitude: 28.6139,

        longitude: 77.2090

    },

    permissions: ["geolocation"]

});

```

Two important things happen:

-   Location is mocked.
    
-   Permission is granted.
    

----------

# Why Permissions Matter

Browsers normally ask:

```text
Allow this site to know your location?

[Allow]

[Deny]

```

Automation cannot reliably interact with native browser permission dialogs.

Instead,

Playwright grants permission automatically.

```typescript
permissions: ["geolocation"]

```

----------

# Without Permission

Application

```typescript
navigator.geolocation.getCurrentPosition(...)

```

Result

```text
Permission Denied

```

----------

# With Permission

```typescript
permissions: ["geolocation"]

```

Result

```text
Latitude

↓

Longitude

↓

Application Works

```

----------

# Example – Mock New Delhi

```typescript
const context = await browser.newContext({

    geolocation: {

        latitude: 28.6139,

        longitude: 77.2090

    },

    permissions: ["geolocation"]

});

const page = await context.newPage();

await page.goto("https://example.com");

```

The application believes the user is in New Delhi.

----------

# Example – Mock New York

```typescript
const context = await browser.newContext({

    geolocation: {

        latitude: 40.7128,

        longitude: -74.0060

    },

    permissions: ["geolocation"]

});

```

Without moving,

the application thinks the user is in New York.

----------

# Common Coordinates

City

Latitude

Longitude

New Delhi

28.6139

77.2090

Mumbai

19.0760

72.8777

Bengaluru

12.9716

77.5946

Hyderabad

17.3850

78.4867

London

51.5072

-0.1276

New York

40.7128

-74.0060

Tokyo

35.6762

139.6503

Sydney

-33.8688

151.2093

These are useful for regional testing.

----------

# Changing Location During a Test

Unlike locale or timezone, **geolocation can be changed after the browser context has been created**.

Playwright provides:

```typescript
await context.setGeolocation({

    latitude: 19.0760,

    longitude: 72.8777

});

```

Now the application believes the user moved from Delhi to Mumbai.

----------

# Dynamic Location Flow

```text
Delhi

↓

Mumbai

↓

London

↓

New York

```

All within one automated test.

----------

# Example

```typescript
await context.setGeolocation({

    latitude: 28.6139,

    longitude: 77.2090

});

await page.reload();

await context.setGeolocation({

    latitude: 19.0760,

    longitude: 72.8777

});

await page.reload();

```

Useful for testing:

-   Delivery availability
    
-   Regional pricing
    
-   Store locator
    
-   Weather
    

----------

# Testing "Near Me"

Application

```text
Search

↓

Restaurants Near Me

↓

Location API

↓

Nearby Restaurants

```

Mock location

```text
Latitude

↓

Longitude

↓

Nearby Results

```

Verify

```typescript
await expect(

page.getByText("Nearby Restaurants")

).toBeVisible();

```

----------

# Testing Delivery Availability

Example

```text
Location A

↓

Delivery Available

-------------------

Location B

↓

Delivery Not Available

```

Same application.

Different mocked locations.

----------

# Testing Weather Applications

Mock

```text
London

```

Verify

```text
15°C

Cloudy

```

Then

Mock

```text
Delhi

```

Verify

```text
40°C

Sunny

```

----------

# Testing Maps

Application

```text
Google Maps

↓

Current Location

↓

Marker Appears

```

Mock coordinates.

Verify

-   Marker location
    
-   Route calculation
    
-   Nearby places
    

----------

# Testing Ride Booking

Applications like:

```text
Uber

Ola

Lyft

```

can be tested by mocking:

Pickup

↓

Destination

↓

Driver Search

----------

# Testing Permission Denied

Sometimes the application should handle users denying location access.

Instead of granting permission:

```typescript
const context = await browser.newContext();

```

Do **not** include:

```typescript
permissions:["geolocation"]

```

Now the application should display:

```text
Location Permission Required

```

----------

# Reading Current Position

Application code

```typescript
navigator.geolocation.getCurrentPosition(

position => {

    console.log(

        position.coords.latitude

    );

});

```

Playwright automatically supplies the mocked coordinates.

----------

# Accuracy

Geolocation supports an optional accuracy value.

```typescript
await context.setGeolocation({

    latitude: 28.6139,

    longitude: 77.2090,

    accuracy: 100

});

```

This represents the estimated accuracy in meters.

----------

# Enterprise Example

Food Delivery

```text
Open App

↓

Current Location

↓

Nearby Restaurants

↓

Estimated Delivery

↓

Checkout

```

No GPS required.

----------

# Enterprise Architecture

Instead of

```typescript
await context.setGeolocation(...);

```

inside every test,

use reusable helpers.

```typescript
await LocationFactory.india(context);

await LocationFactory.uk(context);

await LocationFactory.usa(context);

```

----------

Example

```typescript
class LocationFactory {

    static async india(context){

        await context.setGeolocation({

            latitude:28.6139,

            longitude:77.2090

        });

    }

}

```

Much easier to maintain.

----------

# Best Practices

-   Always grant geolocation permission when the application requires location access.
    
-   Use realistic coordinates that represent actual cities or business scenarios.
    
-   Change locations only when the scenario requires movement.
    
-   Verify business behavior rather than just checking the coordinates.
    
-   Centralize commonly used locations in helper classes or factories.
    

----------

# Common Mistakes

### ❌ Forgetting Permission

```typescript
geolocation:{
...
}

```

without

```typescript
permissions:["geolocation"]

```

The application may receive a permission denied error.

----------

### ❌ Using Invalid Coordinates

Latitude must be between **-90 and 90**.

Longitude must be between **-180 and 180**.

----------

### ❌ Hardcoding Coordinates Everywhere

Instead of

```typescript
28.6139

77.2090

```

Create reusable location helpers.

----------

### ❌ Testing Only One Region

Enterprise applications often behave differently depending on location.

Test multiple regions where business logic varies.

----------

# Interview Questions

### Q1. How do you mock geolocation in Playwright?

Configure the browser context with `geolocation` and grant the `"geolocation"` permission.

----------

### Q2. Why is the `permissions` option required?

Without it, the browser may deny access to the Geolocation API, preventing the application from receiving location data.

----------

### Q3. Can geolocation be changed after creating the browser context?

Yes. Use:

```typescript
await context.setGeolocation({
    latitude: ...,
    longitude: ...
});

```

to update the mocked location during a test.

----------

### Q4. What kinds of applications commonly use geolocation testing?

Maps, ride-sharing, food delivery, weather, retail store locators, travel, and regional pricing applications.

----------

### Q5. Can geolocation mocking replace testing on real GPS devices?

It covers most browser-based scenarios, but real-device testing is still valuable for validating hardware behavior, native integrations, and device-specific edge cases.

----------

# Summary

Geolocation mocking enables Playwright to simulate any location in the world without relying on physical GPS hardware. By configuring browser contexts with coordinates and appropriate permissions, you can reliably test location-aware features such as maps, nearby searches, delivery availability, weather, and regional business rules. Centralizing commonly used locations into reusable helpers further improves maintainability in enterprise automation frameworks.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTY2MTI5Nzg0Nl19
-->