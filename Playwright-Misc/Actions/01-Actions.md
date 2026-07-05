# Chapter: Actions in Playwright

Actions are one of the most fundamental parts of Playwright automation. Every UI test interacts with the application through actions such as clicking buttons, typing text, selecting options, uploading files, dragging elements, or pressing keyboard shortcuts.

Unlike many traditional automation tools, Playwright automatically waits before performing an action. This significantly reduces flaky tests because Playwright ensures the target element is ready before interacting with it.

----------

# Introduction

An **action** is any operation performed on a web element.

Examples include:

-   Clicking a button
    
-   Entering text
    
-   Selecting a dropdown option
    
-   Uploading a file
    
-   Pressing keyboard keys
    
-   Dragging an element
    
-   Scrolling the page
    

Example:

```typescript
await page.getByRole('button', { name: 'Login' }).click();

```

or

```typescript
await page.locator('#username').fill('admin');

```

----------

# Actionability Checks

Before performing most actions, Playwright automatically verifies that the element is ready.

It checks whether the element is:

-   Attached to the DOM
    
-   Visible
    
-   Stable (not animating)
    
-   Enabled
    
-   Receiving pointer events
    
-   Not covered by another element
    

Example:

```typescript
await page.locator('#submit').click();

```

If the button is still loading, Playwright waits automatically.

This is one of the biggest reasons Playwright tests are more reliable than Selenium tests.

----------

# Text Input

Playwright provides multiple ways to enter text.

## fill()

Replaces existing text.

```typescript
await page.locator('#username').fill('john');

```

Before:

```
Existing Value

```

After:

```
john

```

It clears the field first.

----------

## Example

```typescript
test('Login', async ({ page }) => {

    await page.goto('https://example.com');

    await page.locator('#username').fill('admin');

    await page.locator('#password').fill('password');

    await page.locator('#login').click();

});

```

----------

## clear()

Clears existing value.

```typescript
await page.locator('#username').clear();

```

----------

## inputValue()

Reads entered value.

```typescript
const value = await page.locator('#username').inputValue();

console.log(value);

```

----------

## Example

```typescript
await page.locator('#email').fill('test@gmail.com');

expect(await page.locator('#email').inputValue())
    .toBe('test@gmail.com');

```

----------

# Checkboxes and Radio Buttons

Playwright provides dedicated methods instead of using click().

## check()

```typescript
await page.locator('#rememberMe').check();

```

If already checked, Playwright does nothing.

----------

## uncheck()

```typescript
await page.locator('#rememberMe').uncheck();

```

----------

## isChecked()

```typescript
const checked =
    await page.locator('#rememberMe').isChecked();

expect(checked).toBeTruthy();

```

----------

## Example

```typescript
await page.locator('#newsletter').check();

expect(
    await page.locator('#newsletter').isChecked()
).toBe(true);

```

----------

## Radio Buttons

```typescript
await page.locator('#male').check();

```

No need for click().

Playwright ensures correct selection.

----------

# Select Options

Used for HTML `<select>` dropdowns.

----------

## Select by value

```typescript
await page.locator('#country')
    .selectOption('india');

```

HTML

```html
<option value="india">India</option>

```

----------

## Select by label

```typescript
await page.locator('#country')
    .selectOption({ label: 'India' });

```

----------

## Select by index

```typescript
await page.locator('#country')
    .selectOption({ index: 2 });

```

----------

## Multiple Selection

```typescript
await page.locator('#languages')
    .selectOption([
        'java',
        'typescript',
        'python'
    ]);

```

----------

## Example

```typescript
await page.locator('#country')
    .selectOption({ label: 'India' });

expect(
    await page.locator('#country').inputValue()
).toBe('india');

```

----------

# Mouse Click

The most commonly used action.

----------

## click()

```typescript
await page.locator('#login').click();

```

----------

## Double Click

```typescript
await page.locator('#edit').dblclick();

```

----------

## Right Click

```typescript
await page.locator('#file')
    .click({ button: 'right' });

```

----------

## Middle Click

```typescript
await page.locator('#link')
    .click({ button: 'middle' });

```

----------

## Click with Modifiers

```typescript
await page.locator('#link').click({
    modifiers: ['Control']
});

```

Useful for opening links in new tabs.

----------

## Force Click

Normally avoid.

```typescript
await page.locator('#submit')
    .click({ force: true });

```

This skips actionability checks.

Use only when absolutely necessary.

----------

## Click at Position

```typescript
await page.locator('#canvas').click({
    position: {
        x: 150,
        y: 80
    }
});

```

Useful for canvas testing.

----------

# Type Characters

Unlike `fill()`, `pressSequentially()` (or the older `type()` API in earlier Playwright versions) enters characters one by one, making it useful when the application reacts to each keystroke.

> **Recommendation:** In modern Playwright, prefer `fill()` for standard text entry. Use sequential typing only when you specifically need key-by-key behavior (such as autocomplete, search suggestions, or input masking).

## pressSequentially()

```typescript
await page.locator('#search')
    .pressSequentially('Playwright');

```

Each character is typed separately.

----------

## Add Delay

```typescript
await page.locator('#search')
    .pressSequentially('Playwright', {
        delay: 100
    });

```

Characters are typed every 100 milliseconds.

----------

## Example

Autocomplete search.

```typescript
await page.goto('https://example.com');

await page.locator('#search')
    .pressSequentially('Laptop');

await expect(
    page.locator('.suggestions')
).toBeVisible();

```

----------

# Keys and Shortcuts

Keyboard interactions are handled through the keyboard API or locator methods.

----------

## Press a Key

```typescript
await page.keyboard.press('Enter');

```

----------

## Press Escape

```typescript
await page.keyboard.press('Escape');

```

----------

## Keyboard Shortcut

```typescript
await page.keyboard.press('Control+A');

await page.keyboard.press('Delete');

```

----------

## Copy

```typescript
await page.keyboard.press('Control+A');

await page.keyboard.press('Control+C');

```

----------

## Paste

```typescript
await page.keyboard.press('Control+V');

```

----------

## Hold Key

```typescript
await page.keyboard.down('Shift');

await page.keyboard.press('ArrowRight');

await page.keyboard.up('Shift');

```

----------

## Example

```typescript
await page.locator('#search').fill('Playwright');

await page.keyboard.press('Control+A');

await page.keyboard.press('Delete');

await expect(
    page.locator('#search')
).toHaveValue('');

```

----------

# Upload Files

Playwright uploads files without opening the operating system file picker.

----------

## Single File

```typescript
await page.locator('input[type=file]')
    .setInputFiles('sample.pdf');

```

----------

## Multiple Files

```typescript
await page.locator('input[type=file]')
    .setInputFiles([
        'resume.pdf',
        'photo.png'
    ]);

```

----------

## Remove Uploaded File

```typescript
await page.locator('input[type=file]')
    .setInputFiles([]);

```

----------

## Upload from Buffer

```typescript
await page.locator('input[type=file]')
.setInputFiles({
    name: 'hello.txt',
    mimeType: 'text/plain',
    buffer: Buffer.from('Hello Playwright')
});

```

Useful when generating files dynamically during tests.

----------

## Example

```typescript
await page.goto('/upload');

await page.locator('#upload')
    .setInputFiles('test-data/image.png');

await expect(
    page.locator('.success')
).toContainText('Upload Successful');

```

----------

# Focus Element

Moves keyboard focus to an element.

```typescript
await page.locator('#email').focus();

```

Useful when:

-   Testing focus behavior
    
-   Triggering validation
    
-   Keyboard navigation testing
    

----------

## Example

```typescript
await page.locator('#email').focus();

await page.keyboard.type('abc@test.com');

```

----------

# Drag and Drop

Playwright provides a high-level API for drag-and-drop interactions.

```typescript
await page.locator('#source')
    .dragTo(page.locator('#target'));

```

----------

## Example

```typescript
await page.goto('/dragdrop');

await page.locator('#item')
    .dragTo(page.locator('#cart'));

await expect(
    page.locator('#cart')
).toContainText('item');

```

----------

# Dragging Manually

Some applications (especially custom canvas controls or complex JavaScript frameworks) do not respond correctly to `dragTo()`. In those cases, simulate the drag with mouse events.

```typescript
const source = page.locator('#drag');
const target = page.locator('#drop');

await source.hover();

await page.mouse.down();

await target.hover();

await page.mouse.up();

```

----------

## Pixel-by-Pixel Drag

```typescript
const box =
    await page.locator('#slider')
        .boundingBox();

await page.mouse.move(
    box!.x + 10,
    box!.y + 10
);

await page.mouse.down();

await page.mouse.move(
    box!.x + 250,
    box!.y + 10
);

await page.mouse.up();

```

This approach is commonly used for:

-   Sliders
    
-   Signature pads
    
-   Canvas elements
    
-   Image editors
    
-   Custom drag-and-drop libraries
    

----------

# Scrolling

Playwright offers several ways to scroll.

----------

## Scroll Element into View

```typescript
await page.locator('#footer')
    .scrollIntoViewIfNeeded();

```

----------

## Scroll Page

```typescript
await page.mouse.wheel(0, 1000);

```

----------

## Scroll Using JavaScript

```typescript
await page.evaluate(() => {
    window.scrollTo(0, document.body.scrollHeight);
});

```

----------

## Scroll a Specific Container

```typescript
await page.locator('.results').evaluate((element) => {
    element.scrollTop = element.scrollHeight;
});

```

----------

## Infinite Scroll Example

```typescript
let previousHeight = 0;

while (true) {

    const currentHeight = await page.evaluate(() => {
        return document.body.scrollHeight;
    });

    if (currentHeight === previousHeight)
        break;

    previousHeight = currentHeight;

    await page.mouse.wheel(0, currentHeight);

    await page.waitForTimeout(1000);
}

```

This pattern is useful when testing applications that load more content as the user scrolls.

----------

# Best Practices

-   Prefer `Locator` APIs over `ElementHandle`.
    
-   Use `fill()` for standard text input; use `pressSequentially()` only when the application depends on individual keystrokes.
    
-   Use `check()` and `uncheck()` instead of `click()` for checkboxes and radio buttons.
    
-   Use `selectOption()` only for native `<select>` elements. For custom dropdowns (React, Angular, Material UI, etc.), interact with them using clicks and locators.
    
-   Avoid `force: true` unless you've confirmed that actionability checks are the only blocker.
    
-   Prefer `dragTo()` for standard HTML drag-and-drop, falling back to manual mouse events only when necessary.
    
-   Let Playwright's built-in auto-waiting work for you instead of adding unnecessary explicit waits.
    
-   Use accessible locators such as `getByRole()`, `getByLabel()`, and `getByPlaceholder()` whenever possible.
    

----------

# Common Mistakes

### Using `click()` for checkboxes

```typescript
await page.locator('#agree').click();

```

Better:

```typescript
await page.locator('#agree').check();

```

----------

### Using `fill()` for autocomplete testing

```typescript
await page.locator('#search').fill('Playwright');

```

If suggestions appear after each keypress, use:

```typescript
await page.locator('#search').pressSequentially('Playwright');

```

----------

### Using `selectOption()` on a custom dropdown

`selectOption()` only works with native `<select>` elements. For custom UI libraries, locate and click the dropdown trigger and then the desired option.

----------

### Excessive `waitForTimeout()`

```typescript
await page.waitForTimeout(5000);

```

In most cases, rely on Playwright's auto-waiting or explicit assertions (`expect(locator).toBeVisible()`) instead of fixed delays.

----------

# Interview Questions

### Q1. What is the difference between `fill()` and `pressSequentially()`?

-   `fill()` clears existing content and sets the value immediately.
    
-   `pressSequentially()` types one character at a time, triggering keyboard events for each character.
    

----------

### Q2. Why should `check()` be preferred over `click()` for checkboxes?

`check()` ensures the checkbox ends up in the checked state. If it's already checked, no unnecessary action is performed. A `click()` simply toggles the state.

----------

### Q3. Does Playwright wait before clicking?

Yes. Before performing most actions, Playwright automatically performs actionability checks to ensure the element is attached, visible, stable, enabled, and able to receive input.

----------

### Q4. When would you use manual dragging instead of `dragTo()`?

When interacting with custom drag-and-drop implementations, canvas-based controls, sliders, or JavaScript libraries that don't respond correctly to the standard HTML5 drag-and-drop events.

----------

### Q5. Can Playwright upload files without opening the native file picker?

Yes. Using `setInputFiles()`, Playwright sets the file directly on the `<input type="file">` element without interacting with the operating system dialog.

----------

## Summary

Actions are at the heart of every Playwright test. Understanding the differences between methods like `fill()` versus `pressSequentially()`, `click()` versus `check()`, and `dragTo()` versus manual mouse operations helps you build reliable, readable, and maintainable automation. Combined with Playwright's automatic waiting and actionability checks, these APIs allow tests to closely mimic real user interactions while remaining resilient against common UI timing issues.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwNDc0NjkzMjNdfQ==
-->