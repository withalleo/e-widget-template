# AI Instructions - Generating Alleo E-widget

You are generating HTML content that will be loaded inside an **Alleo E-widget**. Read every rule in this file before writing any code.

## Step 0 - check you can actually generate the file

**Before anything else** (before asking clarifying questions, before planning), confirm that your environment lets you produce and hand off a single HTML file to the user (e.g. as a downloadable file, a code block they can save, or an equivalent artifact). If you cannot output a file the user can use, tell them immediately instead of proceeding with the rest of this workflow.

---

## Context

- The generated content is displayed as a **widget** inside an **iframe** on an **infinite collaborative canvas**.
- The canvas may contain many other widgets and objects alongside this one.
- The iframe is **sandboxed** - a strict security boundary is in place (see Constraints below).
- The iframe **size is not fixed**. Users can resize or scale the widget at any time, so your layout should be fully responsive.

## Required knowledge before generating

**Do not write any HTML until the items below are resolved.** Some of these might change based on the user's request).

| Knowledge needed       | Question it answers                                                                                            | Assumed default                   | Resolve how                   |
| ---------------------- | -------------------------------------------------------------------------------------------------------------- | --------------------------------- | ----------------------------- |
| Hosting                | Hosted in Alleo (ie. imported), or served from an external URL?                                                | Hosted in Alleo                   | Ask user, inform of default   |
| Containment            | Fully self-contained, or does it fetch data from an external URL/API (and which URL)?                          | Self-contained, no external calls | Decide yourself, warn if used |
| Alleo version          | Is it using the online or the on-premise version of Alleo?                                                     | Online version                    | Inform of default             |
| Public internet access | Will it use any online resources (not from Alleo? Some installations are offline)                              | Has full internet access          | Inform of default             |
| State synchronization  | Should widget state be kept in sync across every board instance, or independent per instance?                  | Independent (not synced)          | Decide yourself, ask the user |
| Board object content   | Does it need to read/write text content of other board objects - which ones, which operations?                 | No board object access            | Decide yourself, warn if used |
| Connection use         | Does it need network connections (WebSocket/HTTP), and does it need an API key?                                | No network connections            | Decide yourself, warn if used |
| Color & font pickers   | Should board editors be able to change background/primary/text color and/or font from Alleo's widget settings? | See Color & font pickers section  | Decide yourself               |

---

## Output rules

- **Produce exactly one self-contained HTML file.** No extra files, no external stylesheets, no separate scripts.
- All CSS must be inside a `<style>` block; all JavaScript inside `<script>` blocks - both within the single HTML document.
- **All JavaScript must be syntactically valid, plain ES2020+ JavaScript.** Do not use TypeScript, JSX, or any non-standard syntax. The code runs directly in the browser with no build step.
- **Do not include any links** (`<a href>`, `window.location`, `window.open`, redirects, etc.) that open content outside the iframe.
- The code must be **production-ready**: clear, correct, organized, and free of bugs.

## Design guidelines

- Prefer flat, material surfaces.
- Use a **minimalist** design. Avoid all unnecessary borders and shadows.
- do not set a border radius if possible.
- Prefer a dark background color (e.g., #051825), with light text colors (e.g., #f9fff6).
- Use the Alleo-provided font (see CSS variables below) with at least 16 px font size.
- Build the widget around these **3 colors** - background, primary, and text - using them for exactly that purpose (surfaces, accents/interactive elements, and copy respectively). If you need extra shades (hover states, borders, disabled states, subtle backgrounds, etc.), **calculate them as variants of these 3 colors** (e.g. with CSS `color-mix()`, or a lighter/darker mix) instead of introducing unrelated colors.
- allow the user to adjust the colors in the configuration section.
- Leave the background as transparent if possible.
- avoid keyboard controls. The widget should be usable with touch only.
- Do not use the HTML select element. You can use a custom dropdown, but not the native HTML one.
- Do not add excessive margins to the content. Set the widget size to fit the content. But if the user resizes the widget, the content should not grow or shrink, and it should stay centered.
- **Do not use any external images** (e.g., logos, icons, or background images). If you need an icon, use a simple inline SVG instead.
- All user actions should be available using touch. (you can add additional keyboard support.)

### Alleo-provided CSS variables

Calling [`alleo.initialize()`](#sdk-api-reference) (see below) fetches the widget's configured theme and applies the following CSS variables to the iframe's `body` element:

- `--alleo-background-color` - The widget's background color (e.g., `#051825`)
- `--alleo-text-color` - The widget's text color (e.g., `#f9fff6`)
- `--alleo-primary-color` - The widget's primary accent color (e.g., `#6da8ff`)
- `--alleo-font` - The platform's font stack (e.g., `'PT Root UI', "Ubuntu sans", Helvetica, Verdana, sans-serif`)

**Always define these 4 CSS variables yourself, with sensible default values, at the very top of the first `<style>` block** (in the configuration section - see below). This keeps the widget looking correct before `alleo.initialize()` responds, or when the HTML is opened standalone outside of Alleo. Alleo applies its real values as **inline styles** on `document.body`, so they always take priority over your defaults once the widget initializes - you don't need `!important` or any special selector.

**Use these variables directly in your CSS** to ensure your widget respects the user's configured theme. You can reference them with `var(--alleo-background-color)`, `var(--alleo-text-color)`, `var(--alleo-primary-color)`, and `var(--alleo-font, sans-serif)`.

Always use `var(--alleo-font, sans-serif)` for your font-family declarations to match the platform typography with a sensible fallback.

You may also define your own CSS variables in the configuration section for widget-specific colors (e.g., secondary colors, button colors, etc.), but always prefer the Alleo-provided variables for main theme colors and typography, and calculate them as variants of the 3 Alleo colors rather than inventing unrelated colors.

### Color & font pickers

Deciding whether to expose color/font pickers for a widget is **part of your job when generating the widget** - do not ask the user about it as a clarification question; decide it yourself based on the design and mention your decision in the e-widget plan.

Each of the 4 theme values can independently get its own picker, shown in Alleo's own widget settings panel (separate from anything in your HTML) via the footer's `enabledColorPickers` object (see [Importable settings footer](#importable-settings-footer)):

- `background` - enable when the widget has a visible background surface that plausibly needs to match different board themes.
- `primary` - enable when the widget has an accent/interactive element (buttons, highlights, progress) whose color is meaningful customization.
- `text` - enable when the widget renders text directly against the background (rather than e.g. only inside form controls with their own styling).
- `font` - enable only when typography is a meaningful part of the design (e.g. a display/heading widget). Leave it off for small utility widgets where the default Alleo font is enough.

Default to **on** for `background`, `primary`, and `text` for most visual widgets, and **off** for `font` unless typography is central to the widget. Never enable a picker whose corresponding CSS variable your HTML does not actually use.

---

## Code structure and quality

### Programming principles

- **YAGNI (You Aren't Gonna Need It):** Only implement what's explicitly requested. Don't add features "just in case."
- **Keep it minimal:** Write the smallest amount of code that fully solves the problem.
- **No speculative code:** Don't add abstractions, utilities, or flexibility until they're needed.
- **Start simple:** Implement the simplest solution first. Refactor only when requirements demand it.

### Class-based architecture

All widget logic **must** be organized into one or more JavaScript classes. Do not write loose top-level functions or unstructured procedural code.

- Name the main class after the widget's purpose (e.g. `CountdownTimer`, `VotingWidget`, `DataDashboard`).
- Use descriptive method names that explain intent - the code should read like documentation.
- Keep logic at most two nesting levels deep; extract helper methods or use early returns to flatten complex branches.
- Only create additional classes or helper methods when complexity genuinely requires them.

### Configuration section

Group all tunable values in one clearly marked **configuration section at the top of the HTML `<head>`**, before any other style or logic. This makes the widget easy to customize without reading through the entire codebase and keeps the tunables separate from the widget logic.

- Mark it with prominent HTML comments: `<!-- =============== Configuration =============== -->` … `<!-- ============= END Configuration ============= -->`
- Put JS-tunable values (labels, display text, timing intervals, API endpoints, thresholds, toggle flags) in a `settings` object inside a `<script>`.
- Put theme values (colors, fonts, sizes) in CSS custom properties on `body` inside a `<style>`, and reference them with `var(--…)` in the rest of the CSS.
- **Always define the 4 Alleo CSS variables - `--alleo-background-color`, `--alleo-text-color`, `--alleo-primary-color`, `--alleo-font` - with default values at the very top of this first `<style>` section.** `alleo.initialize()` overrides them at runtime with the widget's real configured values (see CSS variables above); your defaults just keep the widget looking right before that response arrives or when running standalone.
- Each configurable value gets a JSDoc/CSS comment explaining its purpose, expected type, and any constraints.

Example:

```html
<head>
    <!-- ...meta/title... -->
    <!-- =============== Configuration =============== -->
    <script>
        const settings = {
            /** @type {string} Text label shown on the primary action button. */
            buttonLabel: 'Submit',
            /** @type {number} Debounce interval in milliseconds for rapid clicks. */
            debounceMs: 300,
        }
    </script>
    <style>
        body {
            /* Defaults - alleo.initialize() overrides these with the widget's configured theme */
            --alleo-background-color: #051825;
            --alleo-text-color: #f9fff6;
            --alleo-primary-color: #6da8ff;
            --alleo-font: 'PT Root UI', Roboto, 'Helvetica Neue', 'Ubuntu Sans', Helvetica, Verdana, sans-serif;

            /* Define any additional widget-specific variables here if needed, */
            /* calculated as variants of the 3 colors above rather than unrelated colors */
            --secondary-color: color-mix(in srgb, var(--alleo-primary-color) 70%, black);
        }
    </style>
    <!-- ============= END Configuration ============= -->
    <!-- ...other head content... -->
</head>
```

The main class still keeps a short in-class config block at the top of the class body, but it **reads from `settings`** (and the CSS variables) instead of hard-coding values, so the top-of-document section stays the single source of truth:

```js
class ExampleWidget {
    /* =============== Configuration =============== */

    /** @type {string} Text label shown on the primary action button. */
    buttonLabel = settings.buttonLabel ?? 'Submit'

    /** @type {number} Debounce interval in milliseconds for rapid clicks. */
    debounceMs = settings.debounceMs ?? 200

    /* =============== End Configuration =============== */

    // your code here...
}
```

### JSDoc documentation

Every class, constructor, method, property, and configuration value **must** have a `/** … */` JSDoc block.

- Skip the `@param`, `@returns`, `@type`, and `@fires` tags if the type is obvious from the signature, but always include a short description of what it does.
- Document the purpose and behavior, not just the signature.
- For event handlers and callbacks, describe when and why they are invoked.

### General code-quality rules

- Use `const` by default; use `let` only when reassignment is necessary. Never use `var`.
- Prefer template literals over string concatenation.
- Avoid magic numbers and strings - promote them to named configuration values when they're reused or non-obvious.
- Use meaningful variable and parameter names; single-letter names are only acceptable for trivial loop indices.
- Do not leave dead code, commented-out experiments, or unneeded comments.
- Write to console.log () for user actions; use console.warn () and console.error () when needed. Add class and method names to the logs.
- **Inline simple logic:** Don't extract a method for something used once unless it genuinely improves clarity.

---

## Required comment blocks

Place three commented-out HTML blocks at the very top of the generated file, in this order: `WIDGETNAME:`, `WIDGETDESCRIPTION:`, `WIDGETHELP:`. They do **not** need to list the individual actions - those are declared in the settings footer (see below). Warnings and required-settings reminders are **not** required in these blocks.

Each block is its own standalone HTML comment, formatted as `<!-- <TOKEN>:\n<content>\n-->`.

```html
<!-- WIDGETNAME:
Submit Button
-->
<!-- WIDGETDESCRIPTION:
A button that fires a submit action and syncs its last-clicked state across all board instances.
-->
<!-- WIDGETHELP:
## What it does

This widget shows a single button. Clicking it fires a "submit" action that other
board automations can react to, and keeps the last-clicked time synced across every
instance of the widget on the board.

## How to use it

1. Drop the widget onto the board.
2. Click the button whenever you want to trigger the connected action.
3. Anyone else viewing the board sees the same last-clicked state, since it is synced.

## Settings

You can adjust the button label and colors from the widget's settings panel. Board
editors can also enable color pickers for background, primary, and text colors if
those were configured during generation.

## Notes

No external services or credentials are required for this widget to work.
-->
```

- `WIDGETNAME:` and `WIDGETDESCRIPTION:` are simple plain text - no markdown, no line breaks, no formatting.
- `WIDGETHELP:` is Markdown-formatted, end-user-facing help text, **500–2000 characters**, explaining what the widget does, how to use it, and anything else useful to a board editor.
- `WIDGETNAME:` and `WIDGETDESCRIPTION:` are **required**; `WIDGETHELP:` is optional but recommended.
- These MUST NOT contain the characters `<!--` or `-->` inside the content, as that would break the HTML comment.
- Do not list the actions here - the settings footer is the source of truth for them.

---

## Sandboxed iframe - blocked features

The following APIs and browser features are **unavailable** inside the iframe. Do not use them.

| Blocked feature                                 | Notes                                                                           |
| ----------------------------------------------- | ------------------------------------------------------------------------------- |
| `localStorage` / `sessionStorage`               | Unavailable. Use `alleo.setSyncedStatus` / `alleo.onSyncedStatusUpdate` instead |
| `document.cookie`                               | Unavailable. Use `alleo.setSyncedStatus` / `alleo.onSyncedStatusUpdate` instead |
| `IndexedDB`                                     | Unavailable. Use `alleo.setSyncedStatus` / `alleo.onSyncedStatusUpdate` instead |
| `window.open()`                                 | Opening new windows or tabs is blocked                                          |
| `alert()`, `confirm()`, `prompt()`              | Unavailable                                                                     |
| Top-level navigation                            | Never navigate the parent page                                                  |
| `navigator.geolocation`                         | Unavailable                                                                     |
| `navigator.mediaDevices.getUserMedia()`         | Unavailable. Use `alleo.mic` for microphone access (camera unavailable)         |
| `navigator.mediaDevices.getDisplayMedia()`      | Unavailable                                                                     |
| `Notification` API                              | Unavailable                                                                     |
| `navigator.clipboard.writeText()`               | Unavailable                                                                     |
| `element.requestFullscreen()`                   | Unavailable                                                                     |
| `element.requestPointerLock()`                  | Unavailable                                                                     |
| `navigator.serviceWorker.register()`            | Unavailable                                                                     |
| `PaymentRequest`                                | Unavailable                                                                     |
| `navigator.bluetooth` / `navigator.usb`         | Unavailable                                                                     |
| `navigator.credentials`                         | Unavailable                                                                     |
| Device sensors (accelerometer, gyroscope, etc.) | Unavailable                                                                     |
| Same-origin parent access                       | Unavailable                                                                     |

**What is allowed:** JavaScript execution, HTML forms (including submission), `fetch` / `XMLHttpRequest` to external APIs, `<canvas>`, Web Workers, WebSockets, Web Audio API (without mic input), and all standard DOM APIs not listed above.

> **Fetching URLs and files:** the iframe is served from a sandboxed origin, so a plain `fetch()` to a third-party URL is usually blocked by the browser's same-origin / CORS policy. You **must always prefer** `alleo.fetchProtectedUrlJSON` / `fetchProtectedUrlText` / `fetchProtectedUrlBinary` over the native `fetch()` when retrieving anything from the internet - never a plain `fetch()`. Pass `keyId: null` for a **public, unauthenticated** URL or file; this routes the request through the Alleo CORS-bypass proxy, same as an authenticated call but without attaching any credential. If the API requires authentication, pass the real `keyId` string instead (see the SDK reference below) - that is the **only** way to reach an authenticated third-party API from inside the iframe.

> **Microphone:** although `getUserMedia()` is blocked, the SDK exposes `alleo.mic` - a bridge that asks the parent widget to capture the mic and streams the audio back into the iframe as a real `MediaStream`. See the section about alleo.mic below. (Warn the user that this requires the **Allow content access to the microphone** widget setting.)

> **Parent page URL parameters:** the sandboxed iframe cannot read `window.top.location` directly. If the widget needs query parameters from the page hosting Alleo (e.g. a per-session `userId` passed by the embedding site), use `alleo.getParams` instead - see the SDK reference below. Requires the **Share "\_\_embed\_\_" URL parameters** widget setting.

> **Current user:** to personalise the widget for the logged-in board user (name, email, organization, etc.), read `alleo.user` instead of trying to detect the user yourself - see the SDK reference below. Requires the **Allow access to the current user profile** widget setting.

### Features limited in some deployment versions

Alleo can also run in **offline** and **on-prem** deployments, which lack the cloud infrastructure some SDK calls depend on. Do not rely on the following in those versions:

| Feature                                                                                                                                                                    | Cloud | On-Prem | Offline |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----- | ------- | ------- |
| `alleo.fetchProtectedUrlJSON` / `fetchProtectedUrlText` / `fetchProtectedUrlBinary` (both the CORS-proxied `keyId: null` case and authenticated calls with a real `keyId`) | ✅    | ❌      | ❌      |

---

## Alleo E-widget SDK

You **must** use the Alleo E-widget SDK (`alleo` object) whenever the code contains JavaScript. It provides the bridge between the iframe and the Alleo board.

### Including the SDK

Add the `<EWidgetSDK />` tag once at the beginning, before any other script, anywhere in the `<body>`. It is automatically replaced with the SDK script, which makes the `alleo` global available to all subsequent `<script>` blocks. The tag is self-replacing and must **never** have a closing `</EWidgetSDK>` tag.

```html
<body>
    <!-- add before any javascript runs -->
    <EWidgetSDK />
    <!-- your HTML content here -->
    <script>
        // alleo is available here
    </script>
</body>
```

If the user wants to host the widget externally, do not add `<EWidgetSDK />`. In this case, insert the following HTML at the beginning before any JavaScript runs.

```html
<script src="https://widgets.withalleo.com/com.withalleo/embed-browser/assets/widgetAssets/ewidget-utils.umd.js"></script>
<script>
    const alleo = AlleoEWidget.getEmbedWidgetMessenger({ debug: false })
</script>
```

---

### SDK API reference

Access the SDK as the global `alleo` symbol exactly as shown in the examples. Do not detect the SDK with `window.alleo`.

#### `alleo.initialize()`

Call this once, right after the SDK loads, to initialize the widget. It requests the synced status, the shared URL parameters, the current user profile, and the widget's configured colors/font (equivalent to calling `alleo.requestSyncedStatus()` and `alleo.requestColorUpdate()`).

It returns a promise that resolves once every response has arrived. The synced status, URL parameters, user profile, and colors/font always come back (synced status, URL parameters, and user profile are an empty `{}` when their feature is disabled or unset), and a timeout bounds the wait as a safety net, so the promise **always resolves and never rejects**. `await` it to guarantee the initially synced state, theme, `alleo.getParams`, and `alleo.user` are applied before you continue.

**When `initialize()` finishes it automatically fires the `onloaded` action for you** (after the DOM has settled) - do not call `alleo.triggerAction('onloaded')` yourself.

```js
await alleo.initialize()
```

Call it once at startup. You don't fire `onloaded` manually - `initialize()` does it once it completes.

#### `alleo.getParams`

`Record<string, string>` - read-only property.

The parent page's URL query parameters that were shared with this embed. Only parameters whose name starts with `__embed__` are exposed, and that prefix is stripped from the keys. Requires **Share "\_\_embed\_\_" URL parameters with enclosed content** to be enabled in the widget's Security settings (`enableGetParams: true` in the footer); otherwise it stays an empty object.

```js
await alleo.initialize()
console.log(alleo.getParams) // e.g. { userId: '42', lang: 'en' }
```

The values are captured once and stay stable for the lifetime of the embed - they never change afterward, even if the parent page's URL changes. `await alleo.initialize()` before reading `alleo.getParams`. If you don't await it, read it slightly after startup (e.g. inside `onSyncedStatusUpdate`, or after a short delay), not synchronously on the same tick.

#### `alleo.user`

`EmbedUserProfile` - read-only property.

The profile of the currently logged-in board user: `{ firstName?, lastName?, displayName?, email?, imageUrl?, organization?: { name?, id? }, lastActivityAt?, lastLoginAt?, displayColor? }`. Requires **Allow access to the current user profile** to be enabled in the widget's Security settings (`enableGetUser: true` in the footer); otherwise it stays an empty object.

```js
await alleo.initialize()
console.log(alleo.user.email) // e.g. "ada@example.com"
```

The value is captured once and stays stable for the lifetime of the embed. `await alleo.initialize()` before reading `alleo.user`. If you don't await it, read it slightly after startup (e.g. inside `onSyncedStatusUpdate`, or after a short delay), not synchronously on the same tick.

#### `alleo.triggerAction(actionId, data?)`

Fires an outgoing trigger to the Alleo board. Wire user interactions to this.

```js
/**
 * @param {string} actionId
 * @param {Record<string, unknown>} [data]
 */
alleo.triggerAction(actionId, data)
```

- `actionId` must match an ID configured in the widget's **Enabled outgoing triggers** settings.
- `data` is an optional key/value map of output parameters.

#### `alleo.onIncomingAction(handler)`

Registers a callback for incoming actions sent from other board objects.

```js
/**
 * @param {function(payload: { actionId: string; data: Record<string, unknown> }): void} handler
 * @returns {function(): void}
 */
alleo.onIncomingAction(handler)
```

- `data` may be empty; always guard before accessing properties.
- Returns an unsubscribe function.

#### `alleo.setSyncedStatus(status)`

Writes key/value pairs to the widget's shared state. All open instances of the widget receive the update.

```js
/**
 * @param {Record<string, unknown>} status
 */
alleo.setSyncedStatus(status)
```

Values are shallow-merged; existing keys not mentioned are preserved.

#### `alleo.requestSyncedStatus()`

Requests the current synced status. The `onSyncedStatusUpdate` callback fires once with the full current state.

```js
alleo.requestSyncedStatus()
```

You normally don't need to call this directly on startup - `alleo.initialize()` calls it for you. Call it directly only if you need to re-sync mid-session.

#### `alleo.onSyncedStatusUpdate(handler)`

Subscribes to synced status changes (from any widget instance).

```js
/**
 * @param {function(status: Record<string, unknown>): void} handler
 * @returns {function(): void}
 */
alleo.onSyncedStatusUpdate(handler)
```

The callback always receives the **full** status object, even when only one key is changed.

#### `alleo.addContent(params)`

Adds a new object to the Alleo board. Requires **Enable adding new content to the board** in the widget's settings.

```js
/**
 * @param {({ type: 'html', html: string }|{ type: 'notepad', text?: string | string[], textFormat?: 'text' | 'markdown' | 'html' }|{ type: 'sticky-note', text?: string, color?: string, outlineColor?: string, shape?: string }|{ type: 'image', url: string }|{ type: 'video', fileId: string })} params
 */
alleo.addContent(params)
```

`params` is one of:

- `{ type: 'html', html: string }`
- `{ type: 'notepad', text?: string | string[], textFormat?: 'text' | 'markdown' | 'html' }`
- `{ type: 'sticky-note', text?: string, color?: string, outlineColor?: string, shape?: string }`
- `{ type: 'image', url: string }`
- `{ type: 'video', fileId: string }`

For Sticky Notes your text content should be very short. If the user didn't ask for it, only define the `text` (not the colors or shape).

For Notepads prefer the `markdown` format. Do not define fix font sizes unless the user explicitly asked for it.

#### `alleo.fetchProtectedUrlJSON(keyId, input, init?)` / `fetchProtectedUrlText(...)` / `fetchProtectedUrlBinary(...)`

**This is the only supported way to fetch anything from the internet - always prefer it over the native `fetch()`.** A plain `fetch()` to a third-party origin will usually fail with a CORS error inside the iframe. `keyId` controls which mode is used:

- **`keyId: null`** - fetches a **public, unauthenticated** URL or file through the Alleo CORS-bypass proxy (JSON APIs, RSS/Atom feeds, images, text, etc.). Use this whenever the request does **not** carry a secret, token, password, or API key.
- **`keyId: '<string>'`** - calls a third-party API that requires an API key, bearer token, or any other credential. This is the **only** supported way to reach an authenticated third-party API from inside the iframe - never any other approach.

```js
/**
 * @param {string | null} keyId - null for a public/unauthenticated CORS-proxied fetch, or a configured backend keyId for an authenticated call
 * @param {RequestInfo | URL} input
 * @param {RequestInit} [init]
 * @returns {Promise<unknown>}   // fetchProtectedUrlJSON
 * @returns {Promise<string>}    // fetchProtectedUrlText
 * @returns {Promise<ArrayBuffer>} // fetchProtectedUrlBinary
 */
alleo.fetchProtectedUrlJSON(keyId, input, init)
alleo.fetchProtectedUrlText(keyId, input, init)
alleo.fetchProtectedUrlBinary(keyId, input, init)
```

- For a non-null `keyId`, it must match a key declared in the footer's `allowedProtectedBackends` (see [Importable settings footer](#importable-settings-footer)).
- The credential itself is **never** visible to, sent by, or embedded in the iframe's HTML/JS. The parent widget attaches it server-side after checking the request URL against that connection's configured `allowedUrls`; a call to any other URL is rejected.
- **Never** hard-code an API key/secret anywhere in the generated HTML/JS, never ask the user to paste one into a `<script>` variable, and never pass a secret through the `keyId: null` (public) path - it cannot carry credentials.
- Requires `enableIframeCommunication: true` and `enableBackendProxy: true` for both the `keyId: null` and authenticated cases. An authenticated call additionally needs a matching `allowedProtectedBackends[keyId]` entry in the footer with the **exact endpoint URL (s)** the widget calls in `allowedUrls` - see [Importable settings footer](#importable-settings-footer) for the required schema format. After import, the board owner still has to enter the real credential once via the widget's "Configure External Connection" action; the footer only declares which endpoints are allowed, never the secret value.
- Sometimes these keys will be available without the user setting them up. Prefer these keys: OpenWeatherMapApiKey (https://api.openweathermap.org/data/2.5/weather?), GoogleMapsApiKey (https://maps.googleapis.com/maps/api/js), SendGridApiKey (https://api.sendgrid.com/v3/mail/send), ResendApiKey (https://api.resend.com/emails), AiService, AiService-ImageGenerate, AiService-SpeechRecognition, AiService-TextToSpeech (https://api.openai.com/ or other compatible AI provider)

```js
// Public, unauthenticated fetch through the CORS-bypass proxy
const response = await alleo.fetchProtectedUrlJSON(null, 'https://api.example.com/data?id=42')
```

```js
// Authenticated fetch using a configured backend connection
const forecast = await alleo.fetchProtectedUrlJSON('weatherApi', 'https://api.example.com/v1/forecast?city=Berlin')
```

```js
const completion = await alleo.fetchProtectedUrlJSON('openai', 'https://api.openai.com/v1/chat/completions', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ model: 'gpt-4o-mini', messages: [{ role: 'user', content: 'Hi!' }] }),
})
```

#### `alleo.getBoardObjectContent(index, options?)`

Reads the text content of a whitelisted board object. Returns a `Promise` that resolves with an array of content strings, or rejects on error.

Requires **Enable Alleo board features** + **Enable reading/writing board object content** in the widget settings, and at least one board object in the **Whitelisted board objects** list.

```js
/**
 * @param {number} index
 * @param {{ format?: 'text' | 'html' | 'markdown' }} [options]
 * @returns {Promise<string[]>}
 */
alleo.getBoardObjectContent(index, options)
```

- `index` is the **1-based position** of the target object in the whitelist (1 = first object, 2 = second, etc.).
- `format` defaults to `'text'`.

#### `alleo.replaceBoardObjectContent(index, text, options?)`

Replaces the entire text content of a whitelisted board object. Fire-and-forget - no success response. Errors are reported via `alleo.onError`.

```js
/**
 * @param {number} index
 * @param {string | string[]} text
 * @param {{ format?: 'text' | 'html' | 'markdown' }} [options]
 */
alleo.replaceBoardObjectContent(index, text, options)
```

#### `alleo.appendBoardObjectContent(index, text, options?)`

Appends text to the existing content of a whitelisted board object. Same parameters as `replaceBoardObjectContent`.

```js
/**
 * @param {number} index
 * @param {string | string[]} text
 * @param {{ format?: 'text' | 'html' | 'markdown' }} [options]
 */
alleo.appendBoardObjectContent(index, text, options)
```

#### `alleo.onError(handler)`

Registers a callback that fires whenever the widget reports an error (e.g. from board object content operations).

```js
/**
 * @param {function(payload: { error: string; requestId?: string; index?: number }): void} handler
 * @returns {function(): void}
 */
alleo.onError(handler)
```

- Returns an unsubscribe function.
- For `getBoardObjectContent` errors, the promise is rejected first, then `onError` handlers are called.

#### `alleo.mic` - microphone bridge

A namespace that grants the iframe microphone access without `getUserMedia()`. The parent widget captures the mic and streams short audio chunks (≤ 200 ms each) back to the iframe, where they are reassembled into a live `MediaStream`.

Requires the **Allow content access to the microphone** widget setting. The first `alleo.mic.start()` call may trigger a browser permission prompt on the parent page.

```js
/**
 * @param {{ timeslice?: number, mimeType?: string, sampleRate?: number, audioBitsPerSecond?: number }} [options]
 */
alleo.mic.start(options)

alleo.mic.stop()

/**
 * @param {function(stream: MediaStream): void} handler
 * @returns {function(): void}
 */
alleo.mic.onTrack(handler)

/**
 * @param {function(info: { mimeType: string; sampleRate?: number; timeslice: number }): void} handler
 * @returns {function(): void}
 */
alleo.mic.onStarted(handler)

/**
 * @param {function(): void} handler
 * @returns {function(): void}
 */
alleo.mic.onStopped(handler)

/**
 * @param {function(info: { error: string }): void} handler
 * @returns {function(): void}
 */
alleo.mic.onError(handler)

/** @type {boolean} */
alleo.mic.isActive
```

- Register `onTrack` **before** calling `start()`. The handler receives the live `MediaStream` once the first chunk arrives - assign it to `audio.srcObject`, feed it to a `MediaRecorder`, an `AudioContext`, `RTCPeerConnection.addTrack()`, etc.
- `onError` reports permission denial, codec issues, or pipeline failures.
- Always call `alleo.mic.stop()` when the widget no longer needs audio so the parent releases the device.

```js
alleo.mic.onError(({ error }) => console.error('mic:', error))
alleo.mic.onTrack((stream) => {
    const audio = new Audio()
    audio.srcObject = stream
    audio.play()
})
alleo.mic.start({ timeslice: 100 })
```

---

### Actions - critical rules

Actions are the primary integration mechanism between your widget and the Alleo board. Treat action design as a first-class concern - not an afterthought.

#### The `onloaded` action (mandatory)

Every widget signals readiness with the `onloaded` action exactly once, after it has fully loaded (the DOM is ready, listeners are set up, the initial state is applied, and all initial data fetching is complete). **`alleo.initialize()` fires `onloaded` for you automatically once it resolves - do not call `alleo.triggerAction('onloaded')` yourself.** Make sure every piece of startup work you need before the ready signal is finished before (or awaited by) `initialize()`.

`onloaded` is still a regular outgoing trigger from the board's point of view - it **must** have a matching `{ "id": "onloaded", "label": "Loaded" }` entry in the footer's `outgoingActions` (see [Importable settings footer](#importable-settings-footer)), otherwise the board silently rejects the automatically fired call and automations never see it fire.

#### Outgoing triggers - trigger on every user interaction

Fire `alleo.triggerAction` for **every meaningful user interaction**:

- Button clicks, form submissions, toggle switches.
- Text input changes (on commit - e.g. `change` or `blur`, not every keystroke).
- Selections from dropdowns, radio buttons, checkboxes.
- Drag-end, slider changes, color picks - anything the user intentionally does.

Use descriptive, lowercase, hyphenated action IDs (e.g. `item-selected`, `form-submitted`, `color-changed`). Include relevant data as the second argument so downstream automations have context.

#### Incoming actions - allow external control

Register `alleo.onIncomingAction` handlers so the widget can be **driven by other board objects or automations**:

- The parameters for incoming actions are optional. If individual parameters are not set, they might be missing, null, or undefined.
- **Only add incoming actions if requested or obviously needed.** Don't create symmetrical incoming actions "just in case."
- Provide a `reset` incoming action when the widget has mutable state that users might want to clear.
- Incoming actions also serve as a way to control the widget. Provide them as alternatives for inputs (e.g., buttons, inputs, and other interactive UI elements should have matching incoming actions).
- Always guard against missing or malformed `data` properties. Consider `data` properties optional - It is possible that only some are defined.

#### Action naming conventions

| Pattern       | Use for                                                           | Example                           |
| ------------- | ----------------------------------------------------------------- | --------------------------------- |
| `onloaded`    | Widget ready signal (fired automatically by `alleo.initialize()`) | `await alleo.initialize()`        |
| `noun-verbed` | Outgoing (something happened)                                     | `item-selected`, `form-submitted` |
| `verb-noun`   | Incoming (command to do something)                                | `select-item`, `reset-form`       |

---

### State synchronization - keeping every viewer in sync

Alleo boards are collaborative - multiple people may view or interact with the same widget simultaneously from different browsers, devices, or sessions. If the user confirmed during the clarification step that **state should be synchronized**, every piece of visible or interactive state **must** be persisted through the synced-status API so that all viewers see identical content at all times.

#### When synchronization is required

- Every user interaction that changes what the widget displays (selections, input values, toggles, counters, sort order, active tabs, etc.) **must** call `alleo.setSyncedStatus(…)` immediately after updating the local UI.
- On startup, the widget **must** `await alleo.initialize()` (which requests the synced status and the theme colors/font, then fires `onloaded` automatically) and its `onSyncedStatusUpdate` handler applies the returned state, so a newly opened instance renders the latest shared state - not a blank or default view.
- The `alleo.onSyncedStatusUpdate` handler **must** cover every piece of state the widget writes. If the widget calls `alleo.setSyncedStatus({ selected: ... })`, then `onSyncedStatusUpdate` must read `status.selected` and update the UI accordingly.

#### Implementation rules

- **Store only what's needed for sync.** Keep values small and JSON-serializable - IDs, indices, short strings, numbers, booleans. Don't store derived data that can be recomputed.
- **Handle missing keys gracefully.** The first time the widget loads, there is no prior state; default values must produce a sensible initial UI.
- **Batch related changes.** Never split a single logical change into multiple `setSyncedStatus` calls to avoid intermediate inconsistent states for other viewers.
- **Synced state is the source of truth.** When `onSyncedStatusUpdate` fires, rebuild or patch the UI from the received status - don't try to reverse-engineer what changed.

#### Quick reference

```js
// Writing state (after any user interaction that changes visible content)
alleo.setSyncedStatus({ selectedIndex: 2, filterText: 'active' })

// Reading state on startup
alleo.onSyncedStatusUpdate((status) => this.onSyncedStatusChange(status))
await alleo.initialize()
```

---

### Required settings by capability (reference)

| Capability                                                        | Settings to enable                                                                                                                                                                                                                                                                                                                                                                                 |
| ----------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Outgoing triggers                                                 | **Enable Alleo board features for enclosed content**                                                                                                                                                                                                                                                                                                                                               |
| Incoming actions                                                  | **Enable Alleo board features for enclosed content**                                                                                                                                                                                                                                                                                                                                               |
| Add content to board                                              | **Enable Alleo board features** + **Enable adding new content to the board**                                                                                                                                                                                                                                                                                                                       |
| Synced status                                                     | **Enable Alleo board features** + **Enable synchronizing status**                                                                                                                                                                                                                                                                                                                                  |
| Board object content                                              | **Enable Alleo board features** + **Enable reading/writing board object content** + add objects to **Whitelisted board objects** list                                                                                                                                                                                                                                                              |
| Microphone (`alleo.mic`)                                          | **Allow content access to the microphone**                                                                                                                                                                                                                                                                                                                                                         |
| Networked fetches (`alleo.fetchProtectedUrlJSON`/`Text`/`Binary`) | **Enable Alleo board features** + **Allow use of protected backends** (required for both public `keyId: null` fetches and authenticated calls). An authenticated call additionally needs a matching `allowedProtectedBackends` entry (with the exact `allowedUrls`) in the footer + the board owner entering the real API key via the widget's "Configure External Connection" action after import |

---

### Example

A button that triggers an outgoing trigger and keeps its color in sync across all board instances:

```html
<!-- WIDGETNAME:
Submit Button
-->
<!-- WIDGETDESCRIPTION:
A button that fires a submit action on click and syncs its last-clicked time across all board instances.
-->
<!-- WIDGETHELP:
## What it does

This widget shows a single button. Clicking it fires a "submit" action that other
board automations can react to, and keeps the last-clicked time synced across every
instance of the widget on the board.

## How to use it

1. Drop the widget onto the board.
2. Click the button to trigger the connected action.
3. The last-clicked time is synced automatically to every viewer.
-->
<!doctype html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>Example Widget</title>
        <!-- =============== Configuration =============== -->
        <script>
            const settings = {
                'buttonLabel': 'Submit',
            }
        </script>
        <style>
            body {
                --alleo-background-color: #051825;
                --alleo-text-color: #f9fff6;
                --alleo-primary-color: #6da8ff;
                --alleo-font: 'PT Root UI', Roboto, 'Helvetica Neue', 'Ubuntu Sans', Helvetica, Verdana, sans-serif;
            }
        </style>
        <!-- ============= END Configuration ============= -->
        <style>
            body {
                display: flex;
                align-items: center;
                justify-content: center;
                height: 100vh;
                margin: 0;
                font-family: var(--alleo-font, sans-serif);
                background: var(--alleo-background-color);
                color: var(--alleo-text-color);
            }

            #action-btn {
                border: none;
                padding: 16px 24px;
                font-size: 1rem;
                cursor: pointer;
                border-radius: 8px;
                background: var(--alleo-primary-color);
                color: var(--alleo-text-color);
            }
        </style>
    </head>
    <body>
        <EWidgetSDK />
        <button id="action-btn">Submit</button>

        <script>
            /**
             * SubmitButtonWidget - a minimal widget that fires an outgoing trigger
             * on click, accepts an incoming reset command, and synchronises its
             * last-clicked timestamp across all board instances.
             */
            class SubmitButtonWidget {
                /* =============== Configuration =============== */

                /** @type {string} Default label shown on the button. */
                defaultLabel = settings.buttonLabel || 'Submit'

                /** @type {string} Outgoing trigger ID fired when the button is clicked. */
                submitActionId = 'submit'

                /** @type {string} Incoming action ID that resets the button to its default label. */
                resetActionId = 'reset'

                /* =============== End Configuration =============== */

                /**
                 * Creates a new SubmitButtonWidget instance and wires up all
                 * event listeners, incoming-action handlers, and synced-status
                 * subscriptions.
                 */
                constructor(buttonElement) {
                    /** @type {HTMLButtonElement} The primary action button. */
                    this.button = buttonElement

                    this.bindEvents()
                    this.bindIncomingActions()
                    this.bindSyncedStatus()

                    // initialize() applies the initial synced state and theme, then fires `onloaded` automatically.
                    alleo.initialize()
                }

                /**
                 * Attaches DOM event listeners for user interactions.
                 */
                bindEvents() {
                    this.button.addEventListener('click', () => this.handleClick())
                }

                /**
                 * Handles a button click: fires the outgoing submit action and
                 * persists the click timestamp to synced status.
                 */
                handleClick() {
                    alleo.triggerAction(this.submitActionId, { label: this.button.textContent })
                    alleo.setSyncedStatus({ lastClicked: Date.now() })
                }

                /**
                 * Registers incoming-action handlers so the widget can be
                 * controlled by other board objects or automations.
                 */
                bindIncomingActions() {
                    alleo.onIncomingAction(({ actionId }) => {
                        if (actionId === this.resetActionId) this.resetButton()
                    })
                }

                /**
                 * Resets the button text to its default label.
                 */
                resetButton() {
                    this.button.textContent = this.defaultLabel
                }

                /**
                 * Subscribes to synced-status updates and initializes the widget
                 * (requests synced status + theme colors/font) so the UI is
                 * correct on load.
                 */
                bindSyncedStatus() {
                    alleo.onSyncedStatusUpdate((status) => this.applySyncedStatus(status))
                }

                /**
                 * Applies synced status to the UI.
                 * @param {Record<string, unknown>} status - The full synced-status object.
                 */
                applySyncedStatus(status) {
                    if (!status?.lastClicked) return
                    this.button.title = `Last clicked: ${new Date(status.lastClicked).toLocaleTimeString()}`
                }
            }

            const button = document.getElementById('action-btn')
            new SubmitButtonWidget(button)
        </script>
    </body>
</html>

<!-- WIDGETSETTINGS:
{
    "iframeAllowScripts": true,
    "iframeAllowForms": false,
    "iframeAllowOrientationLock": false,
    "iframeAllowPointerLock": false,
    "iframeDisableUserActions": false,
    "iframeDisableScrolling": false,
    "iframeReferrerPolicy": "no-referrer",
    "iframeUnloadWhenOffScreen": true,
    "enableIframeCommunication": true,
    "enableGetParams": false,
    "enableGetUser": false,
    "enableIframeFuncAddContent": false,
    "enableSyncedStatus": true,
    "enableIframeFuncBoardObjectContent": false,
    "enableBackendProxy": false,
    "incomingActions": [
        {
            "id": "reset",
            "label": "Reset"
        }
    ],
    "outgoingActions": [
        {
            "id": "onloaded",
            "label": "Loaded"
        },
        {
            "id": "submit",
            "label": "Submit",
            "outputData": [
                { "id": "label", "label": "Label", "type": "Text" }
            ]
        }
    ],
    "backgroundColor": "#051825",
    "textColor": "#f9fff6",
    "primaryColor": "#6da8ff",
    "font": "",
    "enabledColorPickers": { "background": true, "primary": true, "text": true, "font": false },
    "iframeAllowMicrophone": false,
    "requiredAPIfunctions": ["initialize", "triggerAction", "setSyncedStatus", "onIncomingAction", "onSyncedStatusUpdate"],
    "version": "1.20260625",
    "DefaultWidth": 1280,
    "DefaultHeight": 720
}
-->
```

---

## Importable settings footer

The widget can be **imported as a single file** that bundles both the HTML and its Alleo settings. To make the file importable, append a **settings footer** at the very end of the file, after the closing `</html>`. The footer is a single HTML comment whose JSON pre-configures the widget settings so they match the code you generated - without it, the user has to enable every capability by hand.

Always emit this footer when you finish the HTML. Keep the settings consistent with what the HTML actually does.

The footer is **mandatory, and so is its full set of keys.** Every generated widget must emit the footer with **all of the required keys below**, each explicitly set. When a feature is unused, **still include its key** and set it to its "off" value (`false` / `[]` / empty) - never omit a required key.

### Footer format

```html
<!-- WIDGETSETTINGS:
{
    "iframeAllowScripts": true,
    "iframeAllowForms": false,
    "iframeAllowOrientationLock": false,
    "iframeAllowPointerLock": false,
    "iframeDisableUserActions": false,
    "iframeDisableScrolling": false,
    "iframeReferrerPolicy": "no-referrer",
    "iframeUnloadWhenOffScreen": true,
    "enableIframeCommunication": true,
    "enableGetParams": false,
    "enableGetUser": false,
    "enableIframeFuncAddContent": false,
    "enableSyncedStatus": false,
    "enableIframeFuncBoardObjectContent": false,
    "incomingActions": [],
    "outgoingActions": [],
    "backgroundColor": "#051825",
    "textColor": "",
    "primaryColor": "",
    "font": "",
    "enabledColorPickers": { "background": false, "primary": false, "text": false, "font": false },
    "iframeAllowMicrophone": false,
    "enableBackendProxy": false,
    "requiredAPIfunctions": ["initialize"],
    "version": "1.20260625",
    "DefaultWidth": 1280,
    "DefaultHeight": 720
}
-->
```

Hard rules:

- The footer **must be the last content in the file**, after `</html>`; nothing but whitespace may follow `-->`.
- It **must** contain the literal token `WIDGETSETTINGS:` immediately before the JSON.
- The content between `WIDGETSETTINGS:` and `-->` **must be one valid JSON object** parseable by `JSON.parse` - double-quoted keys/strings, no trailing commas, no comments, no `undefined`.
- Never let the sequence `-->` appear inside a JSON string value.
- Include **only** keys from the allowed list below. Never emit `htmlContent`, `sourceType`, `url`, or `fileId` - those are derived on import or unsupported. Any other key aborts the import.
- **Every key shown in the footer above is required** - include all of them in every footer with an explicit value. Set unused features to their "off" value (`false` / `[]` / empty); never omit them. Enable a flag (`true`) only when the HTML actually uses that feature. The remaining keys (`boardObjectWhitelist`, `newContentContainer`, `allowedProtectedBackends`) stay optional and are added only when relevant.
- `DefaultWidth` and `DefaultHeight` **must be supplied** as positive numbers. Set them to the fixed design size you built for (eg. `1280`×`720`).
- **Whenever `enableBackendProxy: true` is set, `allowedProtectedBackends` MUST be included with a correctly formatted, non-empty `allowedUrls` list for every `keyId` the HTML calls** (see below) - an authenticated API call with no matching, correctly scoped entry will simply fail after import.

### Allowed settings

| Key                                  | Type                                                                                                                                                               | Default              | Set it when…                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `enableIframeCommunication`          | `boolean`                                                                                                                                                          | `false`              | the HTML uses `alleo.*` (any action, synced status, add content, board object content, mic, protected fetch). **Required** for all of the below.                                                                                                                                                                                                                                                                                                                                                           |
| `enableIframeFuncAddContent`         | `boolean`                                                                                                                                                          | `false`              | the HTML calls `alleo.addContent(...)`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `enableSyncedStatus`                 | `boolean`                                                                                                                                                          | `false`              | the HTML uses `alleo.setSyncedStatus` / `requestSyncedStatus` / `onSyncedStatusUpdate`.                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `enableIframeFuncBoardObjectContent` | `boolean`                                                                                                                                                          | `false`              | the HTML reads/writes board objects (`alleo.getBoardObjectContent`, etc.).                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `iframeAllowMicrophone`              | `boolean`                                                                                                                                                          | `false`              | the HTML uses `alleo.mic`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `enableGetParams`                    | `boolean`                                                                                                                                                          | `false`              | the HTML reads `alleo.getParams`. Independent of `enableIframeCommunication`.                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `enableGetUser`                      | `boolean`                                                                                                                                                          | `false`              | the HTML reads `alleo.user`. Requires `enableIframeCommunication: true`.                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `enableBackendProxy`                 | `boolean`                                                                                                                                                          | `false`              | the HTML calls `alleo.fetchProtectedUrlJSON` / `fetchProtectedUrlText` / `fetchProtectedUrlBinary` **at all** - including public/unauthenticated fetches with `keyId: null`. **This is the only way to fetch anything from the internet, and the only way to reach an API that requires an API key/token/credential**; a non-null `keyId` must also be paired with `allowedProtectedBackends` below.                                                                                                       |
| `allowedProtectedBackends`           | `Record<string, { backendSchema: { method: "QueryParam" \| "Header", template: string, connection: "Http" \| "WebSocket", allowedUrls: [string, ...string[]] } }>` | `{}` (optional)      | one entry per non-null `keyId` passed to `alleo.fetchProtectedUrl*(keyId, ...)`. Not needed for calls that pass `keyId: null` (public CORS-proxy fetch). **`allowedUrls` MUST list the exact endpoint(s) the HTML calls for that `keyId`** - this is the security boundary that stops the widget from being turned into an open proxy; never leave it empty, omit it, or use a bare/wildcard domain. See [Protected backend connections](#protected-backend-connections-footer) below for the full format. |
| `outgoingActions`                    | `StoredActionTrigger[]`                                                                                                                                            | `[]`                 | the HTML calls `alleo.triggerAction('<id>', data)`. One entry per action id, **including `onloaded`** (with no `outputData`) - the board only accepts a trigger whose id is listed here.                                                                                                                                                                                                                                                                                                                   |
| `incomingActions`                    | `StoredActionTrigger[]`                                                                                                                                            | `[]`                 | the HTML handles ids in `alleo.onIncomingAction`. One entry per handled id.                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `iframeAllowScripts`                 | `boolean`                                                                                                                                                          | `true`               | keep `true` for any widget with `<script>`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `iframeAllowForms`                   | `boolean`                                                                                                                                                          | `true`               | keep `true` if the HTML submits a `<form>`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `iframeUnloadWhenOffScreen`          | `boolean`                                                                                                                                                          | `true`               | keep `true` unless the widget must keep running off-screen.                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `iframeDisableUserActions`           | `boolean`                                                                                                                                                          | `false`              | Set `true` whenever the user cannot interact with the content using a pointer (click/tap/drag/hover). The built-in color/font pickers don't count toward this - a widget whose only interaction is through those pickers still needs `true`.                                                                                                                                                                                                                                                               |
| `iframeDisableScrolling`             | `boolean`                                                                                                                                                          | `false`              | the content must never scroll.                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `backgroundColor`                    | `string` (CSS color)                                                                                                                                               | `"transparent"`      | mirror the background you chose in the design, applied as `--alleo-background-color` (e.g. `"transparent"`, `"#051825"`).                                                                                                                                                                                                                                                                                                                                                                                  |
| `textColor`                          | `string` (CSS color)                                                                                                                                               | theme                | mirror the text color you used, applied as `--alleo-text-color` (e.g. `#f9fff6`).                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `primaryColor`                       | `string` (CSS color)                                                                                                                                               | theme                | mirror the accent/primary color you used, applied as `--alleo-primary-color` (e.g. `#6da8ff`).                                                                                                                                                                                                                                                                                                                                                                                                             |
| `font`                               | `string` (CSS font stack)                                                                                                                                          | `""` (Alleo default) | mirror the font stack you used, applied as `--alleo-font`. Leave `""` to keep the Alleo default font.                                                                                                                                                                                                                                                                                                                                                                                                      |
| `enabledColorPickers`                | `{ background, primary, text, font: boolean }`                                                                                                                     | all `false`          | for each key, `true` when board editors should get a picker for that value in Alleo's own settings panel - see [Color & font pickers](#color--font-pickers).                                                                                                                                                                                                                                                                                                                                               |
| `iframeReferrerPolicy`               | `string` enum                                                                                                                                                      | `"no-referrer"`      | set it to `"no-referrer"` unless you really have to change it.                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `DefaultWidth`                       | `number` (pixels)                                                                                                                                                  | `1280`               | the widget's initial pixel width. Set it to the design width you built for.                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `DefaultHeight`                      | `number` (pixels)                                                                                                                                                  | `720`                | the widget's initial pixel height. Set it to the design height you built for.                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `version`                            | `string`                                                                                                                                                           | `"1.<today>"`        | **always required.** Version stamp in the form `<version>.YYYYMMDD` (see below).                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `requiredAPIfunctions`               | `string[]`                                                                                                                                                         | `[]`                 | **always required.** List every `alleo.*` function the HTML actually calls, without the `alleo.` prefix (e.g. `["initialize", "triggerAction", "onIncomingAction", "fetchProtectedUrlJSON"]`). Keep it in sync with the HTML - add an entry whenever you add a new `alleo.*` call, remove it if the call is removed.                                                                                                                                                                                       |

> Do not emit any unsafe sandbox flags (same-origin, modals, top navigation, fullscreen, popups, presentation, downloads, pointer lock, orientation lock) - they are stripped on import by policy.

### Protected backend connections (footer)

Set `enableBackendProxy: true` whenever the widget calls `alleo.fetchProtectedUrlJSON` / `fetchProtectedUrlText` / `fetchProtectedUrlBinary` for **any** reason - including a public, unauthenticated fetch with `keyId: null` through the CORS-bypass proxy. **Always prefer this over a plain `fetch()` to download anything from the internet.** Additionally add one `allowedProtectedBackends` entry for every non-null `keyId` used - **this is the only supported way for a generated widget to reach an API that requires an API key, bearer token, or any other credential.** Never invent an alternative (embedding the key in the HTML, asking the user to paste it into a form field you control, routing an authenticated call through the `keyId: null` path, etc.).

Each entry has the shape:

```jsonc
"allowedProtectedBackends": {
    "<keyId>": {
        "backendSchema": {
            "method": "QueryParam", // or "Header" - where the credential is injected
            "template": "appid={{secret.token}}", // "<Name>={{secret.token}}"; use e.g. "Authorization=Bearer {{secret.token}}" for a header
            "connection": "Http", // "Http" for normal fetch-style calls (almost always)
            "allowedUrls": ["https://api.example.com/v1/forecast"] // REQUIRED, must be exact/narrow - see below
        }
    }
}
```

- `<keyId>` must exactly match the string passed as the first argument to every authenticated `alleo.fetchProtectedUrl*(keyId, ...)` call in your HTML for that connection. Calls that pass `keyId: null` (public CORS-proxy fetch) don't need an entry here.
- `method` / `template` describe **where and how** the (never-visible-to-you) credential gets attached - pick `"Header"` + `"Authorization=Bearer {{secret.token}}"` for typical bearer-token APIs, or `"QueryParam"` + `"<param>={{secret.token}}"` for APIs that take the key as a query string parameter.
- `connection` is almost always `"Http"`.
- **`allowedUrls` is mandatory and is a hard security boundary - it is not optional decoration.** It must list the **exact** endpoint URL (s) the HTML actually calls for that `keyId`. Never omit it, leave it empty, or set it to a bare/wildcard domain "just in case" - any request to a URL outside this list is rejected by the platform before the credential is ever attached, and an overly broad list defeats the entire point of the feature.
- The footer **never** contains the actual secret/API key value - only the schema describing which endpoints are allowed and how the credential (entered later by a human) gets attached. After import, the board owner must open the widget's context menu and use **"Configure External Connection `<keyId>`"** to enter the real credential once.

---

### Version stamp (`version`)

Every footer **must** include a `version` string in the exact form `<version>.YYYYMMDD`:

- `<version>` is an integer that **starts at `1`** for a brand-new widget and is **incremented by one every time you modify the widget** (e.g. `1` → `2` → `3`).
- `YYYYMMDD` is the **last update date** of the widget. Always update it to the current date.
- Example: a widget first created on 25 June 2026 ships as `"1.20260625"`. After two edits three days later it becomes `"3.20260628"`.
- If no prior version is present, treat the edit as version `1` and use today's date.

### `StoredActionTrigger` shape

```jsonc
{
    "id": "item-selected", // string, required. EXACTLY matches the id used in the HTML. Pattern: ^[a-zA-Z0-9_-]+$
    "label": "Item selected", // string, required. Human-readable name shown on the board.
    "outputData": [
        // outgoingActions only. Omit when the action carries no data.
        {
            "id": "value", // string, required. The key in the data object. Pattern: ^[a-zA-Z0-9_-]+$
            "label": "Value", // string, required.
            "type": "Text", // one of: "Text" | "Number" | "Boolean" | "Numbers" | "AnyArray" | "Any"
        },
    ],
    "inputData": [/* same shape */], // incomingActions only.
}
```

- Use `outputData` only inside `outgoingActions`, `inputData` only inside `incomingActions`.
- The action `id` and every parameter `id` **must be identical** to the strings used in the HTML's `alleo.triggerAction` / `alleo.onIncomingAction` / `data` keys.
- **Always add an `onloaded` entry to `outgoingActions`** (`{ "id": "onloaded", "label": "Loaded" }`, no `outputData`) - the board rejects `alleo.triggerAction(actionId, …)` calls whose `actionId` isn't listed in `outgoingActions`, and `onloaded` is no exception.
- `type` is the case-sensitive socket type name: `"Text"` (string), `"Number"`, `"Boolean"`, `"Numbers"` (number array), `"AnyArray"` (any array), `"Any"` (avoid unless necessary).

### Deriving the footer from your HTML

Before writing the footer, scan the generated HTML and set flags accordingly:

1. Any `<script>` → keep `iframeAllowScripts: true`.
2. Any `alleo.` reference (or `<EWidgetSDK />`) → `enableIframeCommunication: true`.
3. Each `alleo.triggerAction('<id>', { <param>: ... })` → one `outgoingActions` entry with matching `id` and one `outputData` slot per `data` key. **This includes `onloaded`** - add `{ "id": "onloaded", "label": "Loaded" }` (no `outputData`) even though it's fired automatically by `alleo.initialize()`.
4. Each id handled in `alleo.onIncomingAction` → one `incomingActions` entry (with `inputData` slots if it reads `data`).
5. `alleo.setSyncedStatus`/`requestSyncedStatus`/`onSyncedStatusUpdate` → `enableSyncedStatus: true`.
6. `alleo.addContent` → `enableIframeFuncAddContent: true`.
7. `alleo.getBoardObjectContent`/`replaceBoardObjectContent`/`appendBoardObjectContent` → `enableIframeFuncBoardObjectContent: true` (leave whitelist `[]`).
8. `alleo.mic` → `iframeAllowMicrophone: true`.
9. Form submission → keep `iframeAllowForms: true`.
10. `alleo.getParams` → `enableGetParams: true`; `alleo.user` → `enableGetUser: true`.
11. Any `alleo.fetchProtectedUrlJSON`/`fetchProtectedUrlText`/`fetchProtectedUrlBinary` call (public `keyId: null` fetch **or** authenticated `('<keyId>', ...)` call) → `enableBackendProxy: true`. For an authenticated call, also add one `allowedProtectedBackends['<keyId>']` entry with a correctly-formatted `backendSchema`, **`allowedUrls` listing the exact endpoint (s) called for that `keyId`** (see [Protected backend connections](#protected-backend-connections-footer)). This is the only way to fetch anything from the internet or reach an authenticated API - never invent an alternative.
12. Mirror the background/primary/text colors and font you used in `backgroundColor`/`primaryColor`/`textColor`/`font`, and set `enabledColorPickers` to the background/primary/text/font pickers you decided to enable (see [Color & font pickers](#color--font-pickers) - this is your decision, not something derived from the HTML alone).
13. Set `DefaultWidth`/`DefaultHeight` to the fixed pixel size you designed for (default `1280`×`720`).
14. Collect every distinct `alleo.<function>` call found in the HTML (from steps 2–11 above and any others, e.g. `alleo.initialize`) into `requiredAPIfunctions`, listing each function name **without** the `alleo.` prefix.

If a capability is not used, omit its flag.

---

## Responsive layout

- Always use fixed pixel dimensions. Assume that the default size is 1280x720 pixels. However, assume that the user can resize or scale the widget to any size or ratio.
- Mirror the fixed pixel size you designed for in the footer's `DefaultWidth`/`DefaultHeight` so the imported widget opens at that size.
- The iframe may be small (a few hundred pixels) or very large - design for both.
- Use `box-sizing: border-box` globally to avoid overflow issues.
- Scrolling within the iframe is allowed – but only if needed. If possible, adjust the size to be large enough to show all content without scrolling. Warn the user if you need scrolling.

---

## Checklist before finalizing

- [ ] Resolved every row of the [Required knowledge before generating](#required-knowledge-before-generating) table (hosting, containment, synchronization, board object content, connection use, color/font pickers).
- [ ] Decided (yourself) which color/font pickers to enable and mentioned that decision in the e-widget plan.
- [ ] Presented the e-widget plan (plain-English summary + proposed action list) per AI-USER-COMMUNICATION.md; user confirmed before code was written.
- [ ] Single HTML file - no external file references (other than API calls).
- [ ] All URL/file fetches use `alleo.fetchProtectedUrlJSON`/`fetchProtectedUrlText`/`fetchProtectedUrlBinary` instead of a plain `fetch()` - always prefer the CORS proxy (`keyId: null`) for public resources, and the real `keyId` for anything requiring an API key/token/credential; never a plain `fetch()`, and never a hard-coded secret.
- [ ] `<EWidgetSDK />` tag (or equal script for external url hosting) is present in `<body>` and loads before anything else.
- [ ] `WIDGETNAME:`, `WIDGETDESCRIPTION:`, and `WIDGETHELP:` comment blocks are present at the very top of the file, in that order.
- [ ] `WIDGETNAME:` and `WIDGETDESCRIPTION:` are plain text (no markdown/line breaks); `WIDGETHELP:` is Markdown-formatted, end-user help text between 500 and 2000 characters.
- [ ] If using the mic, `onTrack` is registered before `start()`, `onError` is handled, and `stop()` is called when audio is no longer needed.
- [ ] No links or navigations that open outside the iframe.
- [ ] None of the blocked APIs (cookies, localStorage, alert, etc.) are used.
- [ ] Layout is fully responsive and does not rely on a fixed iframe size.
- [ ] Outgoing triggers are called via `alleo.triggerAction` on user interactions.
- [ ] Incoming actions are handled via `alleo.onIncomingAction`.
- [ ] `alleo.initialize()` is awaited once on startup if synced state and/or theme colors and/or `alleo.getParams` and/or `alleo.user` are used.
- [ ] `alleo.onError` is registered if any board object content operations are used.
- [ ] Board object content calls use 1-based whitelist indices, not raw object IDs.
- [ ] `onloaded` fires exactly once after full initialization - via `alleo.initialize()` (automatic), not a manual `alleo.triggerAction('onloaded')` call.
- [ ] All JavaScript is syntactically valid, plain ES2020+ - no TypeScript, no JSX, no non-standard syntax.
- [ ] Widget logic is organized into classes with descriptive names - no loose top-level functions or unstructured procedural code.
- [ ] Configurable values are grouped in a marked configuration section at the top of the HTML `<head>` (a `settings` object + CSS variables on `body`), which the class reads from instead of hard-coding values; each has a JSDoc/CSS comment.
- [ ] The 4 Alleo CSS variables (`--alleo-background-color`, `--alleo-text-color`, `--alleo-primary-color`, `--alleo-font`) are defined with default values at the top of the first `<style>` block.
- [ ] Every class, constructor, method, property, and configuration value has a `/** … */` JSDoc block.
- [ ] Every meaningful user interaction (click, input change, selection, etc.) fires an `alleo.triggerAction`.
- [ ] Incoming actions via `alleo.onIncomingAction` are provided to allow external control of widget state.
- [ ] If the user requested synchronized state: every visible/interactive state change calls `alleo.setSyncedStatus`, `onSyncedStatusUpdate` covers all stored keys, and `alleo.initialize()` is awaited on startup (which fires `onloaded` automatically).
- [ ] An importable settings footer is appended as the **last** content in the file, after `</html>`.
- [ ] The footer is a single `<!-- ... -->` comment containing the literal `WIDGETSETTINGS:` token followed by a valid `JSON.parse`-able object (no trailing commas/comments/`undefined`, no `-->` inside strings).
- [ ] The footer contains only allowed keys - no `htmlContent`, `sourceType`, `url`, `fileId`, or `overwriteTextColor`.
- [ ] If present, `DefaultWidth` and `DefaultHeight` are supplied together as positive numbers matching the design size.
- [ ] The footer includes **all required keys** (full key set), each explicitly set; unused features set to `false` / `[]` / empty rather than omitted.
- [ ] The footer's `backgroundColor`, `textColor`, `primaryColor`, and `font` mirror the theme actually used, and `enabledColorPickers` matches the color/font picker decision made during pre-generation.
- [ ] The footer includes a `version` stamp in the form `<version>.YYYYMMDD` - `<version>` starts at `1` and is incremented on every modification; `YYYYMMDD` is refreshed to today's date on every modification.
- [ ] `enableIframeCommunication` is `true` in the footer whenever the HTML uses `alleo.*`, and every feature flag matches the capabilities the HTML actually uses (`false` when unused).
- [ ] Every `outgoingActions` / `incomingActions` entry (and each parameter `id`/`type`) in the footer matches the action ids and data keys used in the HTML; `onloaded` **is** listed in `outgoingActions` (with no `outputData`).
- [ ] If the HTML calls `alleo.fetchProtectedUrlJSON`/`fetchProtectedUrlText`/`fetchProtectedUrlBinary` (public `keyId: null` fetch or authenticated call), `enableBackendProxy: true` is set; for each non-null `keyId` used, `allowedProtectedBackends` has one correctly formatted entry with a non-empty, exact/narrow `allowedUrls` list - never omitted, empty, or wildcarded.
- [ ] The footer's `requiredAPIfunctions` lists every `alleo.*` function actually called in the HTML (without the `alleo.` prefix), and contains no unused entries.
