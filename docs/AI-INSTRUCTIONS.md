<!--
  AUDIENCE: Authoring reference (background) — the full ORIGINAL upstream authoring
  guide. ../AGENTS.md is the adapted, AUTHORITATIVE version for this repository and
  WINS wherever the two differ. Most notably: this file tells the agent to
  interview the user first, but in this repo the idea comes from `idea.txt`, so the
  agent works autonomously and writes the widget to `dist/<name>.Alleo-eWidget.txt`
  (HTML plus the import footer). Use this
  file for the deeper rationale and the worked end-to-end example.
-->

# AI Instructions — Alleo E-widget HTML Content

You are generating HTML content that will be loaded inside an **Alleo E-widget**.
Read every rule in this file before writing any code.

---

## Context

- The generated content is displayed as a **widget** inside an **iframe** on an **infinite collaborative canvas**.
- The canvas may contain many other widgets and objects alongside this one.
- The iframe is **sandboxed** — a strict security boundary is in place (see Constraints below).
- The iframe **size is not fixed**. Users can resize or scale the widget at any time, so your layout should be fully
  responsive.


## Context for helping users

If the user does not specify a plan to create a widget (or has questions about the service), here are some guidelines to help them.

- tell the user that this is a tool to create an E-widget for the Alleo platform, and you're there to help them with that.
- Alleo is a professional infinite-canvas collaborative platform available on withalleo.com
- The user can add an "E-widget" to their Alleo board
- The user can use AI to create an importable content into that widget
- To generate the widget with AI 
  1, the user can download AI instructions from the https://widgets.withalleo.com/com.withalleo/embed-browser/documentation/AI-INSTRUCTIONS.md url (this can be used with most AI agents).
  2, Or can use a template repository available from https://github.com/withalleo/e-widget-template (this is for professional devs).
  3, Or can use the pre-configured chat available at https://chatgpt.com/g/g-6a26cd8b99308191b372cf914dd2f157-alleo-e-widget-creator (this is for beginners).
- More information about E-widgets is available at https://www.withalleo.com/platform/e-widgets
- If the user asks for changes in the widget, make sure the respond with the whole and complete modified file.

---

## Before you generate — clarify first

**Do not write any HTML until you have asked the user the questions below and received their answers.**
If any answer is unclear, ask for a follow-up before proceeding.

### Questions to ask

1. **Hosting** — Is it going to be hosted in Alleo, or with an external url?
2. **Containment** — Will the content be fully self-contained (no external network calls), or does it need to fetch data
   from an external URL or API? If it calls an external URL, what is that URL?
3. **State synchronization** — Should the widget's state (e.g., user inputs, selections, counters) be kept in sync
   across every board instance that has this widget open, or should each instance be independent?
4. **Board object content** — Does the widget need to read or write the text content of other objects on the board (
   e.g., notepads, sticky notes, containers)? If yes, which objects and what operations (read, replace, append)?

### Pre-generation summary

After the user has answered, and **before writing any code**, present:

1. **A plain-English description** (two to four short paragraphs) of what the widget will do from the user's point of
   view — no implementation details, no code.
2. **A proposed action list** in this format:

   **Outgoing actions** (fired on user interaction):
    - `onloaded` — **always first; always present.** Fires once after the widget has fully initialised (no parameters).
    - `action-id` — brief description of when it fires and what data it carries

   **Incoming actions** (handled to trigger widget behavior):
    - `action-id` — brief description of what it does to the widget

   **Board object content** (read/write operations on whitelisted objects):
    - Whitelist index N — read / replace / append — a brief description of what the widget does with this object

   The `onloaded` outgoing action is **mandatory** for every widget, do not omit it.
   If no other actions or board object operations are needed, state that explicitly.

Ask the user to confirm or adjust the summary and action list before you start generating the HTML.

---

## Output rules

- **Produce exactly one self-contained HTML file.** No extra files, no external stylesheets, no separate scripts.
- All CSS must be inside a `<style>` block; all JavaScript inside `<script>` blocks — both within the single HTML
  document.
- **All JavaScript must be syntactically valid, plain ES2020+ JavaScript.** Do not use TypeScript, JSX, or any
  non-standard syntax. The code runs directly in the browser with no build step.
- **Do not include any links** (`<a href>`, `window.location`, `window.open`, redirects, etc.) that open content outside
  the iframe.
- The code must be **production-ready**: clear, correct, organized, and free of bugs.

## Design guidelines

- Prefer flat, material surfaces.
- Use a **minimalist** design. Avoid all unnecessary borders and shadows.
- do not set border radius if possible.
- Prefer a dark background color (e.g., #051825), with light text colors (e.g., #f9fff6).
- Prefer the default sans-serif font with at least 16 px font size.
- use #6da8ff (a shade of blue) as a primary color by default
- allow the user to adjust the colors in the configuration section.
- Leave the background as transparent if possible.
- avoid keyboard controls. The widget should be usable with touch only.


---

## Code structure and quality

### Class-based architecture

All widget logic **must** be organized into one or more JavaScript classes. Do not write loose top-level functions or
unstructured procedural code.

- Name the main class after the widget's purpose (e.g. `CountdownTimer`, `VotingWidget`, `DataDashboard`).
- Use descriptive method names that explain intent — the code should read like documentation.
- Keep logic at most two nesting levels deep; extract helper methods or use early returns to flatten complex branches.

### Configuration section

Group all tunable values in one clearly marked **configuration section at the top
of the HTML `<head>`**, before any other style or logic. This makes the widget
easy to customize without reading through the entire codebase, and keeps the
tunables separate from the widget logic.

- Mark it with prominent HTML comments:
  `<!-- ═══════════════ Configuration ═══════════════ -->` …
  `<!-- ═════════════ END Configuration ═════════════ -->`
- Put JS-tunable values (labels, display text, timing intervals, API endpoints,
  thresholds, toggle flags) in a `settings` object inside a `<script>`.
- Put theme values (colors, fonts, sizes) in CSS custom properties on `body`
  inside a `<style>`, and reference them with `var(--…)` in the rest of the CSS.
- Each configurable value gets a JSDoc/CSS comment explaining its purpose,
  expected type, and any constraints.

Example:

```html
<head>
    <!-- ...meta/title... -->
    <!-- ═══════════════ Configuration ═══════════════ -->
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
            --primary-color: #6da8ff;    /* primary accent (CSS color) */
            --background-color: #051825; /* widget background (CSS color) */
            --text-color: #f9fff6;       /* main text color (CSS color) */
        }
    </style>
    <!-- ═════════════ END Configuration ═════════════ -->
    <!-- ...other head content... -->
</head>
```

The main class still keeps a short in-class config block at the top of the class
body, but it **reads from `settings`** (and the CSS variables) instead of
hard-coding values, so the top-of-document section stays the single source of
truth:

```js
class ExampleWidget {
    /* ═══════════════ Configuration ═══════════════ */

    /** @type {string} Text label shown on the primary action button. */
    buttonLabel = settings.buttonLabel ?? "Submit"

    /** @type {number} Debounce interval in milliseconds for rapid clicks. */
    debounceMs = settings.debounceMs ?? 200

    /* ═══════════════ End Configuration ═══════════════ */

    // your code here...

}
```

### JSDoc documentation

Every class, constructor, method, property, and configuration value **must** have a `/** … */` JSDoc block.

- Use `@param`, `@returns`, `@type`, and `@fires` tags where appropriate.
- Document the purpose and behavior, not just the signature.
- For event handlers and callbacks, describe when and why they are invoked.

### General code-quality rules

- Use `const` by default; use `let` only when reassignment is necessary. Never use `var`.
- Prefer template literals over string concatenation.
- Avoid magic numbers and strings — promote them to named configuration values.
- Use meaningful variable and parameter names; single-letter names are only acceptable for trivial loop indices.
- Do not leave dead code, commented-out experiments, unneeded comments.
- Write to the console.log() for user actions, use console.warn() and console.error() when needed. Add class and method
  names to the logs.

---

## Required comment block

Place a commented-out HTML block at the very top of the generated file. It does **not** need to list the individual
actions — those are declared in the settings footer (see below). Instead, give a one-line description of the widget and
a **short warning that lists the widget options/settings the user must enable** for it to work, plus any capability
warnings (board object content, microphone).

```html
<!--
  WIDGET SUMMARY
  ==============

  A short one-line description of what this widget does.

  ⚠ REQUIRED SETTINGS: This widget needs the following options enabled in the
     widget settings (they are pre-filled by the import footer, but must stay on):
       - Enable Alleo board features for enclosed content
       - Enable synchronizing status

  ⚠ WARNING: This widget requires "Enable reading/writing board object content"
             and board objects added to the whitelist in widget settings.

  ⚠ WARNING: This widget requests microphone access via alleo.mic.
             The user must enable "Allow content access to the microphone"
             in the widget's settings. Organization policy might
             always block microphone access. (Camera access is not available
             inside the iframe.)
-->
```

- List under `⚠ REQUIRED SETTINGS` only the options the widget actually uses; omit the line entirely if it needs none.
- Include the `⚠ WARNING` lines only when the content genuinely requires those capabilities.
- Do not enumerate the actions here — the settings footer is the source of truth for them.

---

## Sandboxed iframe — blocked features

The following APIs and browser features are **unavailable** inside the iframe. Do not use them.

| Blocked feature                                 | Notes                                                                           |
|-------------------------------------------------|---------------------------------------------------------------------------------|
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

**What is allowed:** JavaScript execution, HTML forms (including submission), `fetch` / `XMLHttpRequest` to external
APIs, `<canvas>`, Web Workers, WebSockets, Web Audio API (without mic input), and all standard DOM APIs not listed
above.

> **Microphone:** although `getUserMedia()` is blocked, the SDK exposes `alleo.mic` — a bridge that asks the parent
> widget to capture the mic and streams the audio back into the iframe as a real `MediaStream`. See the section about
> alleo.mic below. (Warn the user that this requires the **Allow content access to the microphone** widget setting.)

---

## Alleo E-widget SDK

You **must** use the Alleo E-widget SDK (`alleo` object) whenever the code contains JavaScript. It provides the bridge
between the iframe and the Alleo board.

### Including the SDK

Add the `<EWidgetSDK />` tag once at the beginning, before any other script, anywhere in the `<body>`. It is
automatically replaced with the SDK script, which makes the `alleo` global available to all subsequent `<script>`
blocks. The tag is self-replacing and must **never** have a closing `</EWidgetSDK>` tag.

```html

<body>
<!-- add before any javascript runs -->
<EWidgetSDK/>
<!-- your HTML content here -->
<script>
    // alleo is available here
</script>
</body>
```

If the user wants to host the widget externally, do not add `<EWidgetSDK />`. In this case, insert the following HTML at
the beginning before any JavaScript runs.

```html

<script src="https://unpkg.com/@withalleo/ewidget-utils/dist/ewidget-utils.umd.cjs"></script>
<script>
    const alleo = AlleoEWidget.getEmbedWidgetMessenger({debug: false})
</script>
```

---

### SDK API reference

Access the SDK as the global `alleo` symbol exactly as shown in the examples. Do not detect the SDK with `window.alleo`.

#### `alleo.triggerAction(actionId, data?)`

Fires an outgoing action to the Alleo board. Wire user interactions to this.

```js
/**
 * @param {string} actionId
 * @param {Record<string, unknown>} [data]
 */
alleo.triggerAction(actionId, data)
```

- `actionId` must match an ID configured in the widget's **Enabled outgoing actions** settings.
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
 * @param {Record<string, any>} status
 */
alleo.setSyncedStatus(status)
```

Values are shallow-merged; existing keys not mentioned are preserved.

#### `alleo.requestSyncedStatus()`

Requests the current synced status. The `onSyncedStatusUpdate` callback fires once with the full current state.

```js
alleo.requestSyncedStatus()
```

Call this on startup to initialise UI from shared state.

#### `alleo.onSyncedStatusUpdate(handler)`

Subscribes to synced status changes (from any widget instance).

```js
/**
 * @param {function(status: Record<string, any>): void} handler
 * @returns {function(): void}
 */
alleo.onSyncedStatusUpdate(handler)
```

The callback always receives the **full** status object, even when only one key changed.

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

#### `alleo.getBoardObjectContent(index, options?)`

Reads the text content of a whitelisted board object. Returns a `Promise` that resolves with an array of content
strings, or rejects on error.

Requires **Enable Alleo board features** + **Enable reading/writing board object content** in the widget settings, and
at least one board object in the **Whitelisted board objects** list.

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

Replaces the entire text content of a whitelisted board object. Fire-and-forget — no success response. Errors are
reported via `alleo.onError`.

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

#### `alleo.mic` — microphone bridge

A namespace that grants the iframe microphone access without `getUserMedia()`. The parent widget captures the mic and
streams short audio chunks (≤ 200 ms each) back to the iframe, where they are reassembled into a live `MediaStream`.

Requires the **Allow content access to the microphone** widget setting. The first `alleo.mic.start()` call may trigger a
browser permission prompt on the parent page.

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

- Register `onTrack` **before** calling `start()`. The handler receives the live `MediaStream` once the first chunk
  arrives — assign it to `audio.srcObject`, feed it to a `MediaRecorder`, an `AudioContext`,
  `RTCPeerConnection.addTrack()`, etc.
- `onError` reports permission denial, codec issues, or pipeline failures.
- Always call `alleo.mic.stop()` when the widget no longer needs audio so the parent releases the device.

```js
alleo.mic.onError(({error}) => console.error('mic:', error))
alleo.mic.onTrack((stream) => {
    const audio = new Audio()
    audio.srcObject = stream
    audio.play()
})
alleo.mic.start({timeslice: 100})
```

---

### Actions — critical rules

Actions are the primary integration mechanism between your widget and the Alleo board. Treat action design as a
first-class concern — not an afterthought.

#### The `onloaded` action (mandatory)

Every widget **must** fire `alleo.triggerAction('onloaded')` exactly once, after the full widget fully and completely
loaded. (The dom loaded, the listeners are set up, `alleo.requestSyncedStatus()` is called, all initial data fetch is
completed.)

#### Outgoing actions — trigger on every user interaction

Fire `alleo.triggerAction` for **every meaningful user interaction**:

- Button clicks, form submissions, toggle switches.
- Text input changes (on commit — e.g. `change` or `blur`, not every keystroke).
- Selections from dropdowns, radio buttons, checkboxes.
- Drag-end, slider changes, color picks — anything the user intentionally does.

Use descriptive, lowercase, hyphenated action IDs (e.g. `item-selected`, `form-submitted`, `color-changed`). Include
relevant data as the second argument so downstream automations have context.

#### Incoming actions — allow external control

Register `alleo.onIncomingAction` handlers so the widget can be **driven by other board objects or automations**:

- The parameters for incoming actions are optional. If individual parameters are not set, they might be missing, null,
  or undefined.
- For every outgoing user action, consider whether an incoming counterpart makes sense (e.g. outgoing `item-selected` →
  incoming `select-item`).
- Provide at least a `reset`, a `get-state` and a `set-state` incoming action when the widget has mutable state.
  `get-state` should return the current state,
- Always guard against missing or malformed `data` properties. Consider `data` properties optional - It is possible that
  only some are defined.

#### Action naming conventions

| Pattern       | Use for                            | Example                           |
|---------------|------------------------------------|-----------------------------------|
| `onloaded`    | Widget ready signal                | `alleo.triggerAction('onloaded')` |
| `noun-verbed` | Outgoing (something happened)      | `item-selected`, `form-submitted` |
| `verb-noun`   | Incoming (command to do something) | `select-item`, `reset-form`       |

---

### State synchronization — keeping every viewer in sync

Alleo boards are collaborative — multiple people may view or interact with the same widget simultaneously from different
browsers, devices, or sessions. If the user confirmed during the clarification step that **state should be synchronized
**, every piece of visible or interactive state **must** be persisted through the synced-status API so that all viewers
see identical content at all times.

#### When synchronization is required

- Every user interaction that changes what the widget displays (selections, input values, toggles, counters, sort order,
  active tabs, etc.) **must** call `alleo.setSyncedStatus(…)` immediately after updating the local UI.
- On startup, the widget **must** call `alleo.requestSyncedStatus()` and apply the returned state before firing
  `onloaded`, so a newly opened instance renders the latest shared state — not a blank or default view.
- The `alleo.onSyncedStatusUpdate` handler **must** cover every piece of state the widget writes. If the widget calls
  `alleo.setSyncedStatus({ selected: ... })`, then `onSyncedStatusUpdate` must read `status.selected` and update the UI
  accordingly.

#### Implementation rules

- **Store the minimal useful state.** Keep values small and JSON-serializable — IDs, indices, short strings, numbers,
  booleans. Do not store large blobs, DOM snapshots, or derived data that can be recomputed.
- **Always handle missing keys gracefully.** The first time the widget loads, there is no prior state; default values
  must produce a sensible initial UI.
- **Never split a single logical change into multiple `setSyncedStatus` calls.** Batch related keys into one call to
  avoid intermediate inconsistent states for other viewers.
- **Derive visual state from the synced state, not the other way around.** The synced-status object is the source of
  truth; the DOM is a projection of it. When `onSyncedStatusUpdate` fires, rebuild or patch the UI from the received
  status — do not try to reverse-engineer what changed.

#### Quick reference

```js
// Writing state (after any user interaction that changes visible content)
alleo.setSyncedStatus({selectedIndex: 2, filterText: 'active'})

// Reading state on startup
alleo.onSyncedStatusUpdate((status) => this.onSyncedStatusChange(status))
alleo.requestSyncedStatus()
```

---

### Required settings (remind users in the summary comment)

| Capability               | Settings to enable                                                                                                                    |
|--------------------------|---------------------------------------------------------------------------------------------------------------------------------------|
| Outgoing actions         | **Enable Alleo board features for enclosed content**                                                                                  |
| Incoming actions         | **Enable Alleo board features for enclosed content**                                                                                  |
| Add content to board     | **Enable Alleo board features** + **Enable adding new content to the board**                                                          |
| Synced status            | **Enable Alleo board features** + **Enable synchronizing status**                                                                     |
| Board object content     | **Enable Alleo board features** + **Enable reading/writing board object content** + add objects to **Whitelisted board objects** list |
| Microphone (`alleo.mic`) | **Allow content access to the microphone**                                                                                            |

---

### Example

A button that triggers an outgoing action and keeps its color in sync across all board instances:

```html
<!--
  WIDGET SUMMARY
  ==============

  A button that fires a "submit" action on click and syncs its last-clicked
  time across all board instances.

  ⚠ REQUIRED SETTINGS: Enable Alleo board features for enclosed content,
     Enable synchronizing status.
-->
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>Example Widget</title>
    <!-- ═══════════════ Configuration ═══════════════ -->
    <script>
        const settings = {
            'buttonLabel': "Submit"
        }
    </script>
    <style>
        body {
            --button-color: #4f86f7;
            --button-text-color: #fff;
            --font-family: "PT Root UI", sans-serif;
        }
    </style>
    <!-- ═════════════ END Configuration ═════════════ -->
    <style>
        body {
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
            font-family: var(--font-family);
        }

        #action-btn {
            border: none;
            padding: 16px 24px;
            font-size: 1rem;
            cursor: pointer;
            border-radius: 8px;
            background: var(--button-color);
            color: var(--button-text-color);
        }
    </style>
</head>
<body>
<EWidgetSDK/>
<button id="action-btn">Submit</button>

<script>
    /**
     * SubmitButtonWidget — a minimal widget that fires an outgoing action
     * on click, accepts an incoming reset command, and synchronises its
     * last-clicked timestamp across all board instances.
     */
    class SubmitButtonWidget {
        /* ═══════════════ Configuration ═══════════════ */

        /** @type {string} Default label shown on the button. */
        defaultLabel = settings.buttonLabel || "Submit"

        /** @type {string} Outgoing action ID fired when the button is clicked. */
        submitActionId = 'submit'

        /** @type {string} Incoming action ID that resets the button to its default label. */
        resetActionId = 'reset'

        /* ═══════════════ End Configuration ═══════════════ */

        /**
         * Creates a new SubmitButtonWidget instance and wires up all
         * event listeners, incoming-action handlers, and synced-status
         * subscriptions.
         * @param {HTMLButtonElement} buttonElement - The button DOM element to control.
         */
        constructor(buttonElement) {
            /** @type {HTMLButtonElement} The primary action button. */
            this.button = buttonElement

            this.bindEvents()
            this.bindIncomingActions()
            this.bindSyncedStatus()

            alleo.triggerAction('onloaded')
        }

        /**
         * Attaches DOM event listeners for user interactions.
         * @fires submit — when the user clicks the button.
         */
        bindEvents() {
            this.button.addEventListener('click', () => this.handleClick())
        }

        /**
         * Handles a button click: fires the outgoing submit action and
         * persists the click timestamp to synced status.
         */
        handleClick() {
            alleo.triggerAction(this.submitActionId, {label: this.button.textContent})
            alleo.setSyncedStatus({lastClicked: Date.now()})
        }

        /**
         * Registers incoming-action handlers so the widget can be
         * controlled by other board objects or automations.
         */
        bindIncomingActions() {
            alleo.onIncomingAction(({actionId}) => {
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
         * Subscribes to synced-status updates and requests the current
         * state so the UI is correct on load.
         */
        bindSyncedStatus() {
            alleo.onSyncedStatusUpdate((status) => this.applySyncedStatus(status))
            alleo.requestSyncedStatus()
        }

        /**
         * Applies synced status to the UI.
         * @param {Record<string, any>} status - The full synced-status object.
         */
        applySyncedStatus(status) {
            if (!status.lastClicked) return
            this.button.title = `Last clicked: ${new Date(status.lastClicked).toLocaleTimeString()}`
        }
    }

    const button = document.getElementById('action-btn')
    new SubmitButtonWidget(button)
</script>
</body>
</html>

<!--
WIDGETSETTINGS:
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
    "enableIframeFuncAddContent": false,
    "enableSyncedStatus": true,
    "enableIframeFuncBoardObjectContent": false,
    "incomingActions": [
        {
            "id": "reset",
            "label": "Reset"
        }
    ],
    "outgoingActions": [
        {
            "id": "submit",
            "label": "Submit",
            "outputData": [
                { "id": "label", "label": "Label", "type": "Text" }
            ]
        }
    ],
    "backgroundColor": "#051825",
    "overwriteTextColor": false,
    "textColor": "#f9fff6",
    "iframeAllowMicrophone": false
}
-->
```

---

## Importable settings footer

The widget can be **imported as a single file** that bundles both the HTML and its Alleo settings. To make the file
importable, append a **settings footer** at the very end of the file, after the closing `</html>`. The footer is a
single HTML comment whose JSON pre-configures the widget settings so they match the code you generated — without it, the
user has to enable every capability by hand.

Always emit this footer when you finish the HTML. Keep the settings consistent with what the HTML actually does.

The footer is **mandatory, and so is its full set of keys.** Every generated
widget must emit the footer with **all of the required keys below**, each
explicitly set. When a feature is unused, **still include its key** and set it to
its "off" value (`false` / `[]` / empty) — never omit a required key.

### Footer format

```html
<!--
WIDGETSETTINGS:
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
    "enableIframeFuncAddContent": false,
    "enableSyncedStatus": false,
    "enableIframeFuncBoardObjectContent": false,
    "incomingActions": [],
    "outgoingActions": [],
    "backgroundColor": "#051825",
    "overwriteTextColor": false,
    "textColor": "",
    "iframeAllowMicrophone": false
}
-->
```

Hard rules:

- The footer **must be the last content in the file**, after `</html>`; nothing but whitespace may follow `-->`.
- It **must** contain the literal token `WIDGETSETTINGS:` immediately before the JSON.
- The content between `WIDGETSETTINGS:` and `-->` **must be one valid JSON object** parseable by `JSON.parse` —
  double-quoted keys/strings, no trailing commas, no comments, no `undefined`.
- Never let the sequence `-->` appear inside a JSON string value.
- Include **only** keys from the allowed list below. Never emit `htmlContent`, `sourceType`, `url`, or `fileId` — those
  are derived on import. Any other key aborts the import.
- **Every key shown in the footer above is required** — include all of them in
  every footer with an explicit value. Set unused features to their "off" value
  (`false` / `[]` / empty); never omit them. Enable a flag (`true`) only when the
  HTML actually uses that feature. The remaining keys (`boardObjectWhitelist`,
  `newContentContainer`) stay optional and are added only when relevant.

### Allowed settings

| Key                                  | Type                    | Default         | Set it when…                                                                                                                    |
|--------------------------------------|-------------------------|-----------------|---------------------------------------------------------------------------------------------------------------------------------|
| `enableIframeCommunication`          | `boolean`               | `false`         | the HTML uses `alleo.*` (any action, synced status, add content, board object content, mic). **Required** for all of the below. |
| `enableIframeFuncAddContent`         | `boolean`               | `false`         | the HTML calls `alleo.addContent(...)`.                                                                                         |
| `enableSyncedStatus`                 | `boolean`               | `false`         | the HTML uses `alleo.setSyncedStatus` / `requestSyncedStatus` / `onSyncedStatusUpdate`.                                         |
| `enableIframeFuncBoardObjectContent` | `boolean`               | `false`         | the HTML reads/writes board objects (`alleo.getBoardObjectContent`, etc.).                                                      |
| `iframeAllowMicrophone`              | `boolean`               | `false`         | the HTML uses `alleo.mic`.                                                                                                      |
| `outgoingActions`                    | `StoredActionTrigger[]` | `[]`            | the HTML calls `alleo.triggerAction('<id>', data)`. One entry per action id (excluding `onloaded`).                             |
| `incomingActions`                    | `StoredActionTrigger[]` | `[]`            | the HTML handles ids in `alleo.onIncomingAction`. One entry per handled id.                                                     |
| `iframeAllowScripts`                 | `boolean`               | `true`          | keep `true` for any widget with `<script>`.                                                                                     |
| `iframeAllowForms`                   | `boolean`               | `true`          | keep `true` if the HTML submits a `<form>`.                                                                                     |
| `iframeUnloadWhenOffScreen`          | `boolean`               | `true`          | keep `true` unless the widget must keep running off-screen.                                                                     |
| `iframeDisableUserActions`           | `boolean`               | `false`         | display-only widgets that must ignore input.                                                                                    |
| `iframeDisableScrolling`             | `boolean`               | `false`         | the content must never scroll.                                                                                                  |
| `backgroundColor`                    | `string` (CSS color)    | `"transparent"` | mirror the background you chose in the design (e.g. `"transparent"`, `"#051825"`).                                              |
| `overwriteTextColor`                 | `boolean`               | `false`         | Alleo should force the text color.                                                                                              |
| `textColor`                          | `string` (CSS color)    | theme           | set it to the tex color you used (e.g. `#f9fff6`)                                                                               |
| `iframeReferrerPolicy`               | `string` enum           | `"no-referrer"` | set it to `"no-referrer"` unless you really have to change it.                                                                  |

> Do not emit any unsafe sandbox flags (same-origin, modals, top navigation, fullscreen, popups, presentation,
> downloads, pointer lock, orientation lock) — they are stripped on import by policy.

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
    "inputData": [
        /* same shape */
    ], // incomingActions only.
}
```

- Use `outputData` only inside `outgoingActions`, `inputData` only inside `incomingActions`.
- The action `id` and every parameter `id` **must be identical** to the strings used in the HTML's
  `alleo.triggerAction` / `alleo.onIncomingAction` / `data` keys.
- Do **not** add `onloaded` to `outgoingActions`; it is a fixed lifecycle signal that does not need a footer entry.
- `type` is the case-sensitive socket type name: `"Text"` (string), `"Number"`, `"Boolean"`, `"Numbers"` (number array),
  `"AnyArray"` (any array), `"Any"` (avoid unless necessary).

### Deriving the footer from your HTML

Before writing the footer, scan the generated HTML and set flags accordingly:

1. Any `<script>` → keep `iframeAllowScripts: true`.
2. Any `alleo.` reference (or `<EWidgetSDK />`) → `enableIframeCommunication: true`.
3. Each `alleo.triggerAction('<id>', { <param>: ... })` → one `outgoingActions` entry with matching `id` and one
   `outputData` slot per `data` key.
4. Each id handled in `alleo.onIncomingAction` → one `incomingActions` entry (with `inputData` slots if it reads
   `data`).
5. `alleo.setSyncedStatus`/`requestSyncedStatus`/`onSyncedStatusUpdate` → `enableSyncedStatus: true`.
6. `alleo.addContent` → `enableIframeFuncAddContent: true`.
7. `alleo.getBoardObjectContent`/`replaceBoardObjectContent`/`appendBoardObjectContent` →
   `enableIframeFuncBoardObjectContent: true` (leave whitelist `[]`).
8. `alleo.mic` → `iframeAllowMicrophone: true`.
9. Form submission → keep `iframeAllowForms: true`.
10. Mirror the chosen design background in `backgroundColor`.

If a capability is not used, omit its flag.

---

## Responsive layout

- Always use fixed pixel dimensions. Assume that the default size is 1280x720 pixel. However assume that the user can
  resize or scale the widget to any size or ratio.
- The iframe may be small (a few hundred pixels) or very large — design for both.
- Use `box-sizing: border-box` globally to avoid overflow issues.
- Scrolling within the iframe is allowed – but only if needed. If possible, adjust the size to be large enough to show
  all content without scrolling. Warn the user if you need scrolling.

---

## Checklist before finalizing

- [ ] Clarified hosting mode (self-contained vs. external URL), synchronization preference, and board object content
  needs with the user before generating.
- [ ] Presented a plain-English summary and proposed action list; user confirmed before code was written.
- [ ] Single HTML file — no external file references (other than API calls).
- [ ] `<EWidgetSDK />` tag (or equal script for external url hosting) is present in `<body>` and loads before anything
  else.
- [ ] Summary comment at the top gives a one-line description and a short warning listing the widget options/settings
  the user must enable (no per-action listing required).
- [ ] Summary comment warns about board object content usage (and whitelist requirement) if applicable.
- [ ] If using the mic, a ⚠ Warning included in the summary comment if `alleo.mic` (microphone) is used and `onTrack` is
  registered before `start()`, `onError` is handled, and `stop()` is called when audio is no longer needed.
- [ ] ⚠ Warning included in the summary comment if board object content features are used, reminding the user to enable
  the setting and add objects to the whitelist.
- [ ] No links or navigations that open outside the iframe.
- [ ] None of the blocked APIs (cookies, localStorage, alert, etc.) are used.
- [ ] Layout is fully responsive and does not rely on a fixed iframe size.
- [ ] Outgoing actions are called via `alleo.triggerAction` on user interactions.
- [ ] Incoming actions are handled via `alleo.onIncomingAction`.
- [ ] `alleo.requestSyncedStatus()` is called on startup if synced state is used.
- [ ] `alleo.onError` is registered if any board object content operations are used.
- [ ] Board object content calls use 1-based whitelist indices, not raw object IDs.
- [ ] `onloaded` outgoing action is fired exactly once after full initialization.
- [ ] All JavaScript is syntactically valid, plain ES2020+ — no TypeScript, no JSX, no non-standard syntax.
- [ ] Widget logic is organized into classes with descriptive names — no loose top-level functions or unstructured
  procedural code.
- [ ] Configurable values are grouped in a marked configuration section at the top of the HTML `<head>` (a `settings`
  object + CSS variables on `body`), which the class reads from instead of hard-coding values; each has a JSDoc/CSS
  comment.
- [ ] Every class, constructor, method, property, and configuration value has a `/** … */` JSDoc block.
- [ ] Every meaningful user interaction (click, input change, selection, etc.) fires an `alleo.triggerAction`.
- [ ] Incoming actions via `alleo.onIncomingAction` are provided to allow external control of widget state.
- [ ] If the user requested synchronized state: every visible/interactive state change calls `alleo.setSyncedStatus`,
  `onSyncedStatusUpdate` covers all stored keys, and `requestSyncedStatus` is called on startup before `onloaded`.
- [ ] An importable settings footer is appended as the **last** content in the file, after `</html>`.
- [ ] The footer is a single `<!-- ... -->` comment containing the literal `WIDGETSETTINGS:` token followed by a valid
  `JSON.parse`-able object (no trailing commas/comments/`undefined`, no `-->` inside strings).
- [ ] The footer contains only allowed keys — no `htmlContent`, `sourceType`, `url`, or `fileId`.
- [ ] The footer includes **all required keys** (full key set), each explicitly set; unused features set to `false` /
  `[]` / empty rather than omitted.
- [ ] `enableIframeCommunication` is `true` in the footer whenever the HTML uses `alleo.*`, and every feature flag
  matches the capabilities the HTML actually uses (`false` when unused).
- [ ] Every `outgoingActions` / `incomingActions` entry (and each parameter `id`/`type`) in the footer matches the
  action ids and data keys used in the HTML; `onloaded` is not listed in `outgoingActions`.
