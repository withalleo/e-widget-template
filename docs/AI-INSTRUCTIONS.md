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
- The iframe **size is not fixed**. Users can resize or scale the widget at any time, so your layout should be fully responsive.

---

## Before you generate — clarify first

**Do not write any HTML until you have asked the user the questions below and received their answers.**
If any answer is unclear, ask for a follow-up before proceeding.

### Questions to ask

1. **Hosting** — Is it going to be hosted in Alleo, or with an external url?
2. **Containment** — Will the content be fully self-contained (no external network calls), or does it need to fetch data from an external URL or API? If it calls an external URL, what is that URL?
3. **State synchronization** — Should the widget's state (e.g., user inputs, selections, counters) be kept in sync across every board instance that has this widget open, or should each instance be independent?
4. **Board object content** — Does the widget need to read or write the text content of other objects on the board (e.g., notepads, sticky notes, containers)? If yes, which objects and what operations (read, replace, append)?

### Pre-generation summary

After the user has answered, and **before writing any code**, present:

1. **A plain-English description** (two to four short paragraphs) of what the widget will do from the user's point of view — no implementation details, no code.
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
- All CSS must be inside a `<style>` block; all JavaScript inside `<script>` blocks — both within the single HTML document.
- **All JavaScript must be syntactically valid, plain ES2020+ JavaScript.** Do not use TypeScript, JSX, or any non-standard syntax. The code runs directly in the browser with no build step.
- **Do not include any links** (`<a href>`, `window.location`, `window.open`, redirects, etc.) that open content outside the iframe.
- The code must be **production-ready**: clear, correct, organized, and free of bugs.

## Design guidelines

- Prefer flat, material surfaces.
- Prefer a dark background color (e.g., #051825), with light text colors (e.g., #f9fff6).
- Prefer the default sans-serif font.
- allow the user to adjust the main colors in the configuration section.
- use #6da8ff (a shade of blue) as a primary color by default (but add it to the configuration section)

---

## Code structure and quality

### Class-based architecture

All widget logic **must** be organized into one or more JavaScript classes. Do not write loose top-level functions or unstructured procedural code.

- Name the main class after the widget's purpose (e.g. `CountdownTimer`, `VotingWidget`, `DataDashboard`).
- Use descriptive method names that explain intent — the code should read like documentation.
- Keep logic at most two nesting levels deep; extract helper methods or use early returns to flatten complex branches.

### Configuration section

Every class must begin with a clearly marked **configuration block** that groups all tunable values together. This makes the widget easy to customize without reading through the entire codebase.

- Place the configuration block at the very top of the class body, before the constructor.
- Mark it with a prominent section comment: `/* ═══════════════ Configuration ═══════════════ */`
- Each configurable value must have a **JSDoc comment** explaining its purpose, expected type, and any constraints.
- Typical candidates: colors, labels, display text, sizes, timing intervals, API endpoints, thresholds, toggle flags.

Example:

```js
class ExampleWidget {
    /* ═══════════════ Configuration ═══════════════ */

    /** @type {string} Background color of the primary action button (CSS color value). */
    buttonColor = '#4f86f7'

    /** @type {string} Text label shown on the primary action button. */
    buttonLabel = 'Submit'

    /** @type {number} Debounce interval in milliseconds for rapid clicks. */
    debounceMs = 300

/* ═══════════════ End Configuration ═══════════════ */
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
- Write to the console.log() for user actions, use console.warn() and console.error() when needed. Add class and method names to the logs.

---

## Required comment block

Place a commented-out HTML block at the very top of the generated file. It does **not** need to list the individual actions — those are declared in the settings footer (see below). Instead, give a one-line description of the widget and a **short warning that lists the widget options/settings the user must enable** for it to work, plus any capability warnings (board object content, microphone).

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

> **Microphone:** although `getUserMedia()` is blocked, the SDK exposes `alleo.mic` — a bridge that asks the parent widget to capture the mic and streams the audio back into the iframe as a real `MediaStream`. See the section about alleo.mic below. (Warn the user that this requires the **Allow content access to the microphone** widget setting.)

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
<script src="https://unpkg.com/@withalleo/ewidget-utils/dist/ewidget-utils.umd.cjs"></script>
<script>
    const alleo = AlleoEWidget.getEmbedWidgetMessenger({ debug: false })
</script>
```

---

### SDK API reference

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

Replaces the entire text content of a whitelisted board object. Fire-and-forget — no success response. Errors are reported via `alleo.onError`.

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

- Register `onTrack` **before** calling `start()`. The handler receives the live `MediaStream` once the first chunk arrives — assign it to `audio.srcObject`, feed it to a `MediaRecorder`, an `AudioContext`, `RTCPeerConnection.addTrack()`, etc.
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

### Actions — critical rules

Actions are the primary integration mechanism between your widget and the Alleo board. Treat action design as a first-class concern — not an afterthought.

#### The `onloaded` action (mandatory)

Every widget **must** fire `alleo.triggerAction('onloaded')` exactly once, after the full widget fully and completely loaded. (The dom loaded, the listeners are set up, `alleo.requestSyncedStatus()` is called, all initial data fetch is completed.)

#### Outgoing actions — trigger on every user interaction

Fire `alleo.triggerAction` for **every meaningful user interaction**:

- Button clicks, form submissions, toggle switches.
- Text input changes (on commit — e.g. `change` or `blur`, not every keystroke).
- Selections from dropdowns, radio buttons, checkboxes.
- Drag-end, slider changes, color picks — anything the user intentionally does.

Use descriptive, lowercase, hyphenated action IDs (e.g. `item-selected`, `form-submitted`, `color-changed`). Include relevant data as the second argument so downstream automations have context.

#### Incoming actions — allow external control

Register `alleo.onIncomingAction` handlers so the widget can be **driven by other board objects or automations**:

- For every outgoing user action, consider whether an incoming counterpart makes sense (e.g. outgoing `item-selected` → incoming `select-item`).
- Provide at least a `reset` or `set-state` incoming action when the widget has mutable state.
- Always guard against missing or malformed `data` properties. Consider `data` properties optional - It is possible that only some are defined.

#### Action naming conventions

| Pattern       | Use for                            | Example                           |
| ------------- | ---------------------------------- | --------------------------------- |
| `onloaded`    | Widget ready signal                | `alleo.triggerAction('onloaded')` |
| `noun-verbed` | Outgoing (something happened)      | `item-selected`, `form-submitted` |
| `verb-noun`   | Incoming (command to do something) | `select-item`, `reset-form`       |

---

### State synchronization — keeping every viewer in sync

Alleo boards are collaborative — multiple people may view or interact with the same widget simultaneously from different browsers, devices, or sessions. If the user confirmed during the clarification step that **state should be synchronized**, every piece of visible or interactive state **must** be persisted through the synced-status API so that all viewers see identical content at all times.

#### When synchronization is required

- Every user interaction that changes what the widget displays (selections, input values, toggles, counters, sort order, active tabs, etc.) **must** call `alleo.setSyncedStatus(…)` immediately after updating the local UI.
- On startup, the widget **must** call `alleo.requestSyncedStatus()` and apply the returned state before firing `onloaded`, so a newly opened instance renders the latest shared state — not a blank or default view.
- The `alleo.onSyncedStatusUpdate` handler **must** cover every piece of state the widget writes. If the widget calls `alleo.setSyncedStatus({ selected: ... })`, then `onSyncedStatusUpdate` must read `status.selected` and update the UI accordingly.

#### Implementation rules

- **Store the minimal useful state.** Keep values small and JSON-serializable — IDs, indices, short strings, numbers, booleans. Do not store large blobs, DOM snapshots, or derived data that can be recomputed.
- **Always handle missing keys gracefully.** The first time the widget loads, there is no prior state; default values must produce a sensible initial UI.
- **Never split a single logical change into multiple `setSyncedStatus` calls.** Batch related keys into one call to avoid intermediate inconsistent states for other viewers.
- **Derive visual state from the synced state, not the other way around.** The synced-status object is the source of truth; the DOM is a projection of it. When `onSyncedStatusUpdate` fires, rebuild or patch the UI from the received status — do not try to reverse-engineer what changed.

#### Quick reference

```js
// Writing state (after any user interaction that changes visible content)
alleo.setSyncedStatus({ selectedIndex: 2, filterText: 'active' })

// Reading state on startup
alleo.onSyncedStatusUpdate((status) => this.onSyncedStatusChange(status))
alleo.requestSyncedStatus()
```

---

### Required settings (remind users in the summary comment)

| Capability               | Settings to enable                                                                                                                    |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------- |
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
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>Example Widget</title>
        <style>
            body {
                display: flex;
                align-items: center;
                justify-content: center;
                height: 100vh;
                margin: 0;
                font-family: Arial, sans-serif;
            }

            #action-btn {
                border: none;
                padding: 16px 24px;
                font-size: 1rem;
                cursor: pointer;
                border-radius: 8px;
                background: #4f86f7;
                color: #fff;
            }
        </style>
    </head>
    <body>
        <EWidgetSDK />
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
                defaultLabel = 'Submit'

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
    "enableIframeCommunication": true,
    "enableSyncedStatus": true,
    "outgoingActions": [
        {
            "id": "submit",
            "label": "Submit",
            "outputData": [
                { "id": "label", "label": "Label", "type": "Text" }
            ]
        }
    ],
    "incomingActions": [
        {
            "id": "reset",
            "label": "Reset"
        }
    ]
}
-->
```

---

## Importable settings footer

The widget can be **imported as a single file** that bundles both the HTML and its Alleo settings. To make the file importable, append a **settings footer** at the very end of the file, after the closing `</html>`. The footer is a single HTML comment whose JSON pre-configures the widget settings so they match the code you generated — without it, the user has to enable every capability by hand.

Always emit this footer when you finish the HTML. Keep the settings consistent with what the HTML actually does.

### Footer format

```html
<!--
WIDGETSETTINGS:
{
    "enableIframeCommunication": true,
    "outgoingActions": []
}
-->
```

Hard rules:

- The footer **must be the last content in the file**, after `</html>`; nothing but whitespace may follow `-->`.
- It **must** contain the literal token `WIDGETSETTINGS:` immediately before the JSON.
- The content between `WIDGETSETTINGS:` and `-->` **must be one valid JSON object** parseable by `JSON.parse` — double-quoted keys/strings, no trailing commas, no comments, no `undefined`.
- Never let the sequence `-->` appear inside a JSON string value.
- Include **only** keys from the allowed list below. Never emit `htmlContent`, `sourceType`, `url`, or `fileId` — those are derived on import. Any other key aborts the import.
- Every key is optional; omit a setting to keep its default. Only enable what the HTML actually uses.

### Allowed settings

| Key                                  | Type                    | Default         | Set it when…                                                                                                                    |
| ------------------------------------ | ----------------------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `enableIframeCommunication`          | `boolean`               | `false`         | the HTML uses `alleo.*` (any action, synced status, add content, board object content, mic). **Required** for all of the below. |
| `enableIframeFuncAddContent`         | `boolean`               | `false`         | the HTML calls `alleo.addContent(...)`.                                                                                         |
| `enableSyncedStatus`                 | `boolean`               | `false`         | the HTML uses `alleo.setSyncedStatus` / `requestSyncedStatus` / `onSyncedStatusUpdate`.                                         |
| `enableIframeFuncBoardObjectContent` | `boolean`               | `false`         | the HTML reads/writes board objects (`alleo.getBoardObjectContent`, etc.).                                                      |
| `boardObjectWhitelist`               | `string[]` (object ids) | `[]`            | leave `[]` — you cannot know real ids; tell the user to add objects in the 1-based order the HTML expects.                      |
| `newContentContainer`                | `string` (object id)    | omit            | omit — you cannot invent a valid id.                                                                                            |
| `iframeAllowMicrophone`              | `boolean`               | `false`         | the HTML uses `alleo.mic`.                                                                                                      |
| `outgoingActions`                    | `StoredActionTrigger[]` | `[]`            | the HTML calls `alleo.triggerAction('<id>', data)`. One entry per action id (excluding `onloaded`).                             |
| `incomingActions`                    | `StoredActionTrigger[]` | `[]`            | the HTML handles ids in `alleo.onIncomingAction`. One entry per handled id.                                                     |
| `iframeAllowScripts`                 | `boolean`               | `true`          | keep `true` for any widget with `<script>`.                                                                                     |
| `iframeAllowForms`                   | `boolean`               | `true`          | keep `true` if the HTML submits a `<form>`.                                                                                     |
| `iframeUnloadWhenOffScreen`          | `boolean`               | `true`          | keep `true` unless the widget must keep running off-screen.                                                                     |
| `iframeDisableUserActions`           | `boolean`               | `false`         | display-only widgets that must ignore input.                                                                                    |
| `iframeDisableScrolling`             | `boolean`               | `false`         | the content must never scroll.                                                                                                  |
| `backgroundColor`                    | `string` (CSS color)    | `"transparent"` | mirror the background you chose in the design (e.g. `"#051825"`).                                                               |
| `overwriteTextColor`                 | `boolean`               | `false`         | Alleo should force the text color.                                                                                              |
| `textColor`                          | `string` (CSS color)    | theme           | used only when `overwriteTextColor` is `true`.                                                                                  |
| `iframeReferrerPolicy`               | `string` enum           | `"no-referrer"` | one of `"no-referrer"`, `"origin"`, `"strict-origin"`.                                                                          |

> Do not emit any unsafe sandbox flags (same-origin, modals, top navigation, fullscreen, popups, presentation, downloads, pointer lock, orientation lock) — they are stripped on import by policy.

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
- The action `id` and every parameter `id` **must be identical** to the strings used in the HTML's `alleo.triggerAction` / `alleo.onIncomingAction` / `data` keys.
- Do **not** add `onloaded` to `outgoingActions`; it is a fixed lifecycle signal that does not need a footer entry.
- `type` is the case-sensitive socket type name: `"Text"` (string), `"Number"`, `"Boolean"`, `"Numbers"` (number array), `"AnyArray"` (any array), `"Any"` (avoid unless necessary).

### Deriving the footer from your HTML

Before writing the footer, scan the generated HTML and set flags accordingly:

1. Any `<script>` → keep `iframeAllowScripts: true`.
2. Any `alleo.` reference (or `<EWidgetSDK />`) → `enableIframeCommunication: true`.
3. Each `alleo.triggerAction('<id>', { <param>: ... })` → one `outgoingActions` entry with matching `id` and one `outputData` slot per `data` key.
4. Each id handled in `alleo.onIncomingAction` → one `incomingActions` entry (with `inputData` slots if it reads `data`).
5. `alleo.setSyncedStatus`/`requestSyncedStatus`/`onSyncedStatusUpdate` → `enableSyncedStatus: true`.
6. `alleo.addContent` → `enableIframeFuncAddContent: true`.
7. `alleo.getBoardObjectContent`/`replaceBoardObjectContent`/`appendBoardObjectContent` → `enableIframeFuncBoardObjectContent: true` (leave whitelist `[]`).
8. `alleo.mic` → `iframeAllowMicrophone: true`.
9. Form submission → keep `iframeAllowForms: true`.
10. Mirror the chosen design background in `backgroundColor`.

If a capability is not used, omit its flag.

---

## Responsive layout

- Always use fixed pixel dimensions. Assume that the default size is 1280x720 pixel. However assume that the user can resize or scale the widget to any size or ratio.
- The iframe may be small (a few hundred pixels) or very large — design for both.
- Use `box-sizing: border-box` globally to avoid overflow issues.
- Scrolling within the iframe is allowed – but only if needed. If possible, adjust the size to be large enough to show all content without scrolling. Warn the user if you need scrolling.

---

## Checklist before finalizing

- [ ] Clarified hosting mode (self-contained vs. external URL), synchronization preference, and board object content needs with the user before generating.
- [ ] Presented a plain-English summary and proposed action list; user confirmed before code was written.
- [ ] Single HTML file — no external file references (other than API calls).
- [ ] `<EWidgetSDK />` tag (or equal script for external url hosting) is present in `<body>` and loads before anything else.
- [ ] Summary comment at the top gives a one-line description and a short warning listing the widget options/settings the user must enable (no per-action listing required).
- [ ] Summary comment warns about board object content usage (and whitelist requirement) if applicable.
- [ ] If using the mic, a ⚠ Warning included in the summary comment if `alleo.mic` (microphone) is used and `onTrack` is registered before `start()`, `onError` is handled, and `stop()` is called when audio is no longer needed.
- [ ] ⚠ Warning included in the summary comment if board object content features are used, reminding the user to enable the setting and add objects to the whitelist.
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
- [ ] Widget logic is organized into classes with descriptive names — no loose top-level functions or unstructured procedural code.
- [ ] Configurable values are grouped at the top of the class in a marked `/* ═══ Configuration ═══ */` section with JSDoc comments.
- [ ] Every class, constructor, method, property, and configuration value has a `/** … */` JSDoc block.
- [ ] Every meaningful user interaction (click, input change, selection, etc.) fires an `alleo.triggerAction`.
- [ ] Incoming actions via `alleo.onIncomingAction` are provided to allow external control of widget state.
- [ ] If the user requested synchronized state: every visible/interactive state change calls `alleo.setSyncedStatus`, `onSyncedStatusUpdate` covers all stored keys, and `requestSyncedStatus` is called on startup before `onloaded`.
- [ ] An importable settings footer is appended as the **last** content in the file, after `</html>`.
- [ ] The footer is a single `<!-- ... -->` comment containing the literal `WIDGETSETTINGS:` token followed by a valid `JSON.parse`-able object (no trailing commas/comments/`undefined`, no `-->` inside strings).
- [ ] The footer contains only allowed keys — no `htmlContent`, `sourceType`, `url`, or `fileId`.
- [ ] `enableIframeCommunication` is `true` in the footer whenever the HTML uses `alleo.*`, and every feature flag matches the capabilities the HTML actually uses.
- [ ] Every `outgoingActions` / `incomingActions` entry (and each parameter `id`/`type`) in the footer matches the action ids and data keys used in the HTML; `onloaded` is not listed in `outgoingActions`.
