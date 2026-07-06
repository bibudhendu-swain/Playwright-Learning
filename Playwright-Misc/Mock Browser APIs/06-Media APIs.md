This is one of the **most advanced Browser API topics** in Playwright.

Many enterprise applications depend on camera and microphone access, including:

-   Microsoft Teams
    
-   Zoom
    
-   Google Meet
    
-   Slack Huddles
    
-   Banking KYC
    
-   Passport Verification
    
-   QR Code Scanners
    
-   Barcode Readers
    
-   Face Verification
    
-   Voice Search
    

Testing these manually is difficult because they depend on hardware devices. Playwright, together with Chromium launch options, allows us to automate many of these scenarios reliably.

----------

# Part 17 – Mock Browser APIs

# Chapter 6 – Media APIs (Camera & Microphone)

----------

# Introduction

Many web applications need access to the user's:

-   Camera
    
-   Microphone
    

Typical workflow:

```text
Application

↓

Request Camera

↓

Browser Permission

↓

Camera Stream

↓

Application

```

Similarly,

```text
Application

↓

Request Microphone

↓

Browser Permission

↓

Audio Stream

↓

Application

```

Without permissions, access is denied.

----------

# What are Media APIs?

Browsers expose media devices through:

```typescript
navigator.mediaDevices

```

The most commonly used API is:

```typescript
navigator.mediaDevices.getUserMedia()

```

This API requests access to:

-   Video
    
-   Audio
    
-   Or both
    

----------

# Media API Flow

```text
Application

↓

getUserMedia()

↓

Browser

↓

Permission

↓

Camera/Microphone

↓

Media Stream

```

----------

# Browser Permissions

Before media devices can be used,

the browser must allow:

```text
Camera

Microphone

```

Playwright grants these permissions using:

```typescript
await context.grantPermissions([
    "camera",
    "microphone"
]);

```

----------

# Basic Camera Permission

```typescript
const context =
await browser.newContext();

await context.grantPermissions([
    "camera"
]);

```

----------

# Basic Microphone Permission

```typescript
await context.grantPermissions([
    "microphone"
]);

```

----------

# Camera + Microphone Together

Most conferencing applications require both.

```typescript
await context.grantPermissions([

    "camera",

    "microphone"

]);

```

----------

# getUserMedia()

Application code

```typescript
const stream =

await navigator.mediaDevices.getUserMedia({

    video: true,

    audio: true

});

```

Returns

```text
MediaStream

```

----------

# MediaStream

A MediaStream contains one or more tracks.

```text
MediaStream

│

├── Video Track

└── Audio Track

```

Applications use these tracks for:

-   Video calls
    
-   Recording
    
-   QR scanning
    
-   Voice recognition
    

----------

# Testing Camera Access

Verify the application successfully receives a stream.

```typescript
const streamAvailable =
await page.evaluate(async () => {

    try {

        const stream =
        await navigator.mediaDevices.getUserMedia({

            video: true

        });

        return stream.active;

    } catch {

        return false;

    }

});

expect(streamAvailable).toBeTruthy();

```

----------

# Testing Microphone Access

```typescript
const audioAvailable =
await page.evaluate(async () => {

    try {

        const stream =
        await navigator.mediaDevices.getUserMedia({

            audio: true

        });

        return stream.active;

    } catch {

        return false;

    }

});

expect(audioAvailable).toBeTruthy();

```

----------

# Fake Media Devices

One challenge is that CI servers often **don't have a physical camera or microphone**.

Chromium supports fake devices through launch arguments.

```typescript
const browser =
await chromium.launch({

    args: [

        "--use-fake-ui-for-media-stream",

        "--use-fake-device-for-media-stream"

    ]

});

```

Meaning:

```text
Application

↓

Browser

↓

Fake Camera

↓

Fake Microphone

↓

MediaStream

```

No physical hardware is required.

----------

# Fake Video File

Chromium can also simulate a camera using a video file.

```typescript
const browser =
await chromium.launch({

    args: [

        "--use-fake-ui-for-media-stream",

        "--use-file-for-fake-video-capture=test.y4m"

    ]

});

```

The browser treats the file as a live camera feed.

This is useful for:

-   Face verification
    
-   QR scanners
    
-   Barcode scanners
    
-   Identity verification
    

> **Note:** Chromium expects the fake video file to be in a supported raw format such as `.y4m`.

----------

# Fake Audio File

Similarly,

```typescript
"--use-file-for-fake-audio-capture=test.wav"

```

can simulate microphone input.

Useful for:

-   Voice recognition
    
-   Speech-to-text
    
-   Call automation
    

Support for fake audio capture depends on the Chromium version and platform.

----------

# Testing Permission Denied

Do not grant permissions.

```typescript
const context =
await browser.newContext();

```

Application

```typescript
navigator.mediaDevices.getUserMedia(...)

```

should reject.

Verify

```text
Camera Permission Required

```

or

```text
Microphone Permission Required

```

----------

# Mocking getUserMedia()

Sometimes you don't want to request any real media stream.

Override the API.

```typescript
await page.addInitScript(() => {

    navigator.mediaDevices.getUserMedia =
        async () => {

            return {} as MediaStream;

        };

});

```

Useful when the application only checks that the API succeeds.

> **Note:** Applications that inspect tracks, dimensions, or media properties may require a more complete mock than an empty object.

----------

# Simulating Camera Failure

```typescript
await page.addInitScript(() => {

    navigator.mediaDevices.getUserMedia =
        async () => {

            throw new Error(

                "Camera Unavailable"

            );

        };

});

```

Verify

```text
Unable to Access Camera

```

----------

# Simulating Microphone Failure

```typescript
await page.addInitScript(() => {

    navigator.mediaDevices.getUserMedia =
        async () => {

            throw new Error(

                "Microphone Blocked"

            );

        };

});

```

----------

# QR Scanner Testing

Workflow

```text
Open Scanner

↓

Camera

↓

QR Detected

↓

Navigate

```

Use fake camera input to present a QR code in the video stream.

Verify

-   Scanner opens
    
-   QR detected
    
-   Navigation occurs
    

----------

# Video Conferencing

Example

```text
Join Meeting

↓

Camera

↓

Microphone

↓

Video Preview

↓

Join

```

Verify

-   Preview appears
    
-   Join button enabled
    
-   Local stream active
    

----------

# Banking KYC

```text
Capture Passport

↓

Camera

↓

OCR

↓

Verification

```

A prerecorded video feed allows repeatable testing.

----------

# Face Verification

```text
Camera

↓

Capture Face

↓

Recognition

↓

Verification

```

Use fake video for consistent automation.

----------

# Voice Search

```text
Microphone

↓

Speech

↓

Search

↓

Results

```

Use fake audio input where supported to test speech workflows.

----------

# Enterprise Example – Teams

```text
Open Meeting

↓

Allow Camera

↓

Allow Microphone

↓

Join Meeting

```

Permissions

```typescript
await context.grantPermissions([

    "camera",

    "microphone"

]);

```

----------

# Enterprise Example – QR Scanner

```text
Camera

↓

Scan QR

↓

Open Product

```

Use

```text
Fake Camera Feed

```

----------

# Enterprise Example – Barcode Reader

```text
Open Scanner

↓

Camera

↓

Barcode

↓

Lookup Product

```

----------

# Enterprise Media Helper

Instead of repeating setup:

```typescript
class MediaHelper {

    static async allowMedia(context){

        await context.grantPermissions([

            "camera",

            "microphone"

        ]);

    }

}

```

Usage

```typescript
await MediaHelper.allowMedia(
    context
);

```

----------

# Suggested Folder Structure

```text
helpers/

├── MediaHelper.ts

├── PermissionManager.ts

├── BrowserContextFactory.ts

└── FakeMedia.ts

```

----------

# Media APIs vs Permissions

Permission

Media API

Grants access

Uses access

`grantPermissions()`

`getUserMedia()`

Security

Functionality

----------

# Browser Support

Feature

Chromium

Firefox

WebKit

Camera Permission

✅

✅

✅

Microphone Permission

✅

✅

✅

Fake Media Device Flags

✅

Limited

Limited

Fake Video File

✅

Limited

Limited

Chromium provides the most comprehensive support for fake media devices and is the preferred choice for media-heavy automation.

----------

# Best Practices

-   Grant only the permissions required by the scenario.
    
-   Use Chromium fake media flags for CI environments.
    
-   Test both successful access and permission-denied scenarios.
    
-   Use prerecorded media files for deterministic testing.
    
-   Separate media setup into reusable helpers or fixtures.
    

----------

# Common Mistakes

### ❌ Assuming permissions are enough

Granting permission does **not** create a camera.

CI machines often need:

```text
--use-fake-device-for-media-stream

```

----------

### ❌ Expecting a real webcam in CI

Most CI environments have no camera or microphone.

Always use fake devices.

----------

### ❌ Returning an empty MediaStream for every test

Some applications inspect:

-   Video tracks
    
-   Audio tracks
    
-   Resolution
    
-   Stream state
    

In these cases, use Chromium's fake media devices instead of simplistic mocks.

----------

### ❌ Ignoring browser differences

Fake media support is strongest in Chromium.

Cross-browser media automation may require browser-specific strategies.

----------

# Interview Questions

### Q1. Which browser API provides camera and microphone access?

```typescript
navigator.mediaDevices.getUserMedia()

```

----------

### Q2. How do you grant camera permission in Playwright?

```typescript
await context.grantPermissions([
    "camera"
]);

```

----------

### Q3. Why are fake media devices useful?

They allow camera and microphone workflows to run in environments without physical hardware, such as CI/CD pipelines.

----------

### Q4. Which Chromium launch flags are commonly used for media testing?

```text
--use-fake-ui-for-media-stream
--use-fake-device-for-media-stream

```

Optionally:

```text
--use-file-for-fake-video-capture=<file.y4m>
--use-file-for-fake-audio-capture=<file.wav>

```

----------

### Q5. Can you automate operating system camera permission dialogs?

No. Instead, grant browser permissions programmatically using Playwright and, where necessary, launch Chromium with fake media device flags.

----------

# Summary

Media APIs allow web applications to access cameras and microphones for video conferencing, document scanning, QR code recognition, and voice-based interactions. Playwright simplifies media automation by granting permissions programmatically and, in Chromium, supporting fake media devices and prerecorded media streams. By combining permission management, fake hardware, and reusable helpers, you can build reliable tests for even the most complex media-driven applications.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE4MzI0MDc3NTRdfQ==
-->