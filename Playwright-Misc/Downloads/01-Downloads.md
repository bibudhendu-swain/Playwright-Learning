**Downloads** are a deceptively simple topic. Most beginners only learn how to download a file, but in enterprise automation you'll often need to verify:

-   Was the download triggered?
    
-   Did it complete successfully?
    
-   Was the correct filename used?
    
-   Was the downloaded file valid?
    
-   Was the file stored in the correct location?
    
-   Did the exported CSV/PDF/Excel contain the expected data?
    

We'll cover all of these.

----------

# Part 9 – Downloads

# Chapter 1 – Handling File Downloads

----------

# Introduction

Many web applications allow users to download files.

Common examples include:

-   Export Reports
    
-   Download Invoice
    
-   Export CSV
    
-   Export Excel
    
-   Download PDF
    
-   Download ZIP
    
-   Download Images
    
-   Download Logs
    

Unlike normal browser interactions, downloads happen **outside the webpage**.

----------

# Download Lifecycle

A typical download follows this sequence:

```text
User Clicks Download
        │
        ▼
Browser Sends Request
        │
        ▼
Server Returns File
        │
        ▼
Browser Starts Download
        │
        ▼
Download Completes

```

Playwright provides a **Download** object that represents this lifecycle.

----------

# How Playwright Handles Downloads

Downloads are exposed through the **download event**.

```text
Page

↓

Download Event

↓

Download Object

```

----------

# Download Object

The Download object provides several useful APIs.

Method

Purpose

`path()`

Temporary download location

`saveAs()`

Save to a custom location

`failure()`

Get download failure

`cancel()`

Cancel download

`delete()`

Delete downloaded file

`suggestedFilename()`

Suggested filename from server

`createReadStream()`

Read file contents

----------

# Waiting for a Download

This is the most common pattern.

```typescript
const downloadPromise =
    page.waitForEvent('download');

await page.getByRole('button', {
    name: 'Download'
}).click();

const download =
    await downloadPromise;

```

Notice:

The event is registered **before** clicking.

----------

# Why Register First?

Wrong

```typescript
await page.click('#download');

const download =
await page.waitForEvent('download');

```

The download may already have started.

Correct

```typescript
const downloadPromise =
page.waitForEvent('download');

await page.click('#download');

const download =
await downloadPromise;

```

----------

# Saving a Download

By default Playwright stores downloads in a temporary folder.

To save permanently:

```typescript
await download.saveAs(
'downloads/report.pdf'
);

```

----------

Example

```typescript
const downloadPromise =
page.waitForEvent('download');

await page.click('#export');

const download =
await downloadPromise;

await download.saveAs(
'downloads/orders.csv'
);

```

----------

# Getting the Suggested Filename

Servers often specify:

```http
Content-Disposition:
attachment;
filename="orders.csv"

```

Retrieve it:

```typescript
const name =
download.suggestedFilename();

console.log(name);

```

----------

Validation

```typescript
expect(
download.suggestedFilename()
).toBe(
'orders.csv'
);

```

Very common in enterprise testing.

----------

# Temporary Download Path

Playwright stores downloads temporarily.

Retrieve the path:

```typescript
const path =
await download.path();

```

Example

```text
/tmp/playwright-downloads/12345.pdf

```

Useful when another tool needs to process the file.

----------

# Verifying Download Success

Downloads can fail.

Check:

```typescript
const failure =
await download.failure();

```

If successful

```text
null

```

If failed

```text
Network Error

```

----------

Example

```typescript
expect(
await download.failure()
).toBeNull();

```

----------

# Cancelling a Download

```typescript
await download.cancel();

```

Useful when testing:

-   User cancellation
    
-   Interrupted downloads
    

----------

# Deleting a Download

```typescript
await download.delete();

```

Helps clean up temporary files after tests.

----------

# Reading Download Contents

Instead of saving to disk immediately, you can read the file as a stream.

```typescript
const stream =
await download.createReadStream();

```

Useful for:

-   Large files
    
-   Custom validation
    
-   Streaming pipelines
    

----------

# Real-World Example – Export CSV

```typescript
const downloadPromise =
page.waitForEvent('download');

await page.getByRole('button', {
    name: 'Export CSV'
}).click();

const download =
await downloadPromise;

expect(
download.suggestedFilename()
).toBe(
'customers.csv'
);

await download.saveAs(
'downloads/customers.csv'
);

```

----------

# Real-World Example – Download PDF

```typescript
const downloadPromise =
page.waitForEvent('download');

await page.click('#invoice');

const invoice =
await downloadPromise;

await invoice.saveAs(
'downloads/invoice.pdf'
);

```

----------

# Real-World Example – Export Excel

```typescript
const downloadPromise =
page.waitForEvent('download');

await page.click('#excel');

const excel =
await downloadPromise;

expect(
excel.suggestedFilename()
).toContain('.xlsx');

```

----------

# Download Directory

You can configure Playwright to keep downloads.

Example

```typescript
const browser =
await chromium.launch();

const context =
await browser.newContext({

acceptDownloads: true

});

```

Now downloads are automatically accepted.

----------

# Download vs File Save Dialog

Many people ask:

Can Playwright automate the browser Save As dialog?

Answer:

No.

Instead,

Playwright captures the download **before** the browser Save As dialog appears.

You save the file using:

```typescript
await download.saveAs(...);

```

No operating system dialog interaction is required.

----------

# Downloading Multiple Files

Example

```typescript
for (let i = 0; i < 5; i++) {

    const promise =
    page.waitForEvent('download');

    await page.click('#download');

    const file =
    await promise;

    console.log(
        file.suggestedFilename()
    );

}

```

----------

# Validating File Existence

After saving

```typescript
await download.saveAs(
'downloads/report.csv'
);

```

You can verify using Node.js.

```typescript
import fs from 'fs';

expect(
fs.existsSync(
'downloads/report.csv'
)
).toBeTruthy();

```

----------

# Validating File Content

Example

```typescript
import fs from 'fs';

const content =
fs.readFileSync(
'downloads/report.csv',
'utf8'
);

expect(content)
.toContain(
'Customer Name'
);

```

Very common in reporting tests.

----------

# Download Pattern

The recommended pattern is:

```text
Register Download Event

↓

Click Download

↓

Wait for Download

↓

Validate Filename

↓

Save File

↓

Validate Content

↓

Cleanup

```

----------

# Real-World Example – Monthly Report

```typescript
const downloadPromise =
page.waitForEvent('download');

await page.getByRole('button', {
    name: 'Monthly Report'
}).click();

const report =
await downloadPromise;

expect(
report.suggestedFilename()
).toContain(
'Monthly'
);

await report.saveAs(
'downloads/monthly.pdf'
);

expect(
await report.failure()
).toBeNull();

```

----------

# Best Practices

-   Always register `waitForEvent('download')` **before** triggering the download.
    
-   Validate the suggested filename whenever it is part of the business requirement.
    
-   Save downloads to a known location if you need to inspect them later.
    
-   Verify download failures using `failure()`.
    
-   Clean up downloaded files after the test unless they are required for debugging.
    
-   Validate file contents for reports, invoices, or exported data—not just the existence of the file.
    

----------

# Common Mistakes

### ❌ Waiting after clicking

```typescript
await page.click('#download');

await page.waitForEvent('download');

```

The download may already have started.

----------

### ❌ Assuming the file downloaded successfully

Always verify:

```typescript
expect(
await download.failure()
).toBeNull();

```

----------

### ❌ Ignoring the file contents

Checking that a file exists doesn't prove that it contains the expected business data.

----------

### ❌ Trying to automate the OS Save As dialog

Playwright bypasses this entirely using the `Download` API.

----------

# Interview Questions

### Q1. How do you handle downloads in Playwright?

Register a download event before triggering the action:

```typescript
const downloadPromise =
page.waitForEvent('download');

await page.click('#download');

const download =
await downloadPromise;

```

----------

### Q2. Why should `waitForEvent('download')` be registered first?

Because the download can start immediately after the user action. Registering first ensures the event isn't missed.

----------

### Q3. How do you verify that a download succeeded?

Use:

```typescript
expect(
await download.failure()
).toBeNull();

```

A `null` value indicates the download completed successfully.

----------

### Q4. Can Playwright automate the operating system's Save As dialog?

No. Instead, Playwright captures the download and allows you to save it programmatically using `saveAs()`.

----------

### Q5. How do you verify a downloaded report?

A robust verification typically includes:

1.  Confirm the download completed (`failure()` returns `null`).
    
2.  Verify the suggested filename.
    
3.  Save the file.
    
4.  Check that the file exists.
    
5.  Validate the file's contents (CSV, PDF, Excel, etc.) according to the business requirements.
    

----------

# Summary

Playwright's download API provides a reliable, browser-independent way to automate file downloads without interacting with operating system dialogs. By registering download events before user actions, validating filenames and download status, and inspecting downloaded content when needed, you can confidently automate export features such as reports, invoices, spreadsheets, and PDFs.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTM2NTUyNzAwMl19
-->