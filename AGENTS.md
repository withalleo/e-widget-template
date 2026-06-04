# AGENTS.md — Build an Alleo E-widget from `idea.txt`

These are the operating instructions for any AI coding agent (Codex, Claude Code,
GitHub Copilot, etc.) working in this repository.

> This file is the single source of truth for the task and the coding rules.
> `CLAUDE.md` and `.github/copilot-instructions.md` point here. Read this whole
> file before writing any code.

---

## Your task

1. **Read `idea.txt`** in the repository root. It contains a plain-language
   description of the widget the user wants.
2. **Design and generate one self-contained Alleo E-widget** (a single HTML
   document) that fulfils that idea, following every rule in this file.
3. **Write the result to the `dist/` directory as a `.txt` file** that contains
   the full HTML. Use a short, kebab-case name derived from the widget's purpose,
   e.g. `dist/countdown-timer.txt`. If you cannot derive a good name, use
   `dist/widget.txt`.
    - The file **must** have a `.txt` extension, not `.html`. The Alleo E-widget's file picker expects `.txt` for
      uploaded HTML (see `docs/reference/HELP.md`).
    - The file content is a normal, complete HTML document.
4. **Write a short build note** to `dist/<same-name>.README.md` summarising what
   you built, the assumptions you made, every outgoing/incoming action, the board
   features the user must enable, and the default widget size.

If `idea.txt` is empty or missing, do not invent a random widget — stop and tell
the user to fill in `idea.txt` first (or ask them what to build if running
interactively).

---

## Working autonomously vs. asking questions

The reference instructions in `docs/AI-INSTRUCTIONS.md` ask the agent to
interview the user before generating. **In this repository the idea comes from
`idea.txt`, so prefer to proceed autonomously:**

- Derive the answers to the clarification questions below from `idea.txt`.
- When `idea.txt` is silent on a point, **pick a sensible default, proceed, and
  record the decision** in the build note. Do not block.
- Only stop to ask the user if the idea is genuinely contradictory or impossible.

### Decisions to make (with defaults)

1. **Hosting** — Default: hosted in Alleo, fully self-contained (use the
   `<EWidgetSDK />` tag). Only use an external URL if `idea.txt` demands it.
2. **Containment** — Default: no external network calls. Only call an external
   API if the idea requires live data and names the source.
3. **State synchronization** — Default: if the widget has any shared/interactive
   state and the idea implies collaboration, synchronize it via
   `alleo.setSyncedStatus` / `alleo.onSyncedStatusUpdate`. Otherwise keep
   instances independent.
4. **Board object content** — Default: off. Only read/write other board objects
   if the idea explicitly asks for it.

### Build note — required summary

Always produce the `dist/<name>.README.md` build note with:

1. **A plain-English description** (2–4 short paragraphs) of what the widget does.
2. **The action list**:

   **Outgoing actions** (fired on user interaction):
    - `onloaded` — **always first; always present.** Fires once after init.
    - `action-id` — when it fires and what data it carries.

   **Incoming actions** (handled to drive widget behavior):
    - `action-id` — what it does to the widget.

   **Board object content** (if used):
    - Whitelist index N — read / replace / append — what it does.

3. **Required widget settings** the user must enable (see the table below).
4. **Assumptions made** because `idea.txt` did not specify them.

`onloaded` is mandatory for every widget — never omit it.

---

## Output rules

- **Produce exactly one self-contained HTML document** per widget. No external
  stylesheets, no separate script files.
- All CSS in a `<style>` block; all JavaScript in `<script>` blocks — inside the
  single document.
- **All JavaScript must be valid, plain ES2020+.** No TypeScript, no JSX, no
  build step. It runs directly in the browser.
- **Do not include links or navigations that open content outside the iframe**
  (`<a href>` to external pages, `window.location`, `window.open`, redirects).
- The code must be production-ready: clear, correct, organized, bug-free.

## Design guidelines

- Prefer flat, material surfaces.
- Prefer a dark background (e.g. `#051825`) with light text (e.g. `#f9fff6`).
- Prefer the default sans-serif font.
- Use `#6da8ff` (blue) as the default primary color.
- Expose the main colors (background, text, primary) in the configuration block
  so they are easy to customize.

---

## Code structure and quality

### Class-based architecture

All widget logic **must** live in one or more JavaScript classes. No loose
top-level functions or unstructured procedural code.

- Name the main class after the widget's purpose (`CountdownTimer`,
  `VotingWidget`, `DataDashboard`).
- Use descriptive method names; the code should read like documentation.
- Keep logic at most two nesting levels deep; use early returns / helper methods.

### Configuration section

Every class begins with a clearly marked configuration block grouping all
tunable values, before the constructor.

- Mark it: `/* ═══════════════ Configuration ═══════════════ */`
- Each value gets a JSDoc comment with purpose, type, and constraints.
- Candidates: colors, labels, text, sizes, timing intervals, endpoints,
  thresholds, toggle flags.

```js
class ExampleWidget {
    /* ═══════════════ Configuration ═══════════════ */

    /** @type {string} Background color of the primary action button (CSS color). */
    buttonColor = '#6da8ff'

    /** @type {string} Text label shown on the primary action button. */
    buttonLabel = 'Submit'

    /** @type {number} Debounce interval in milliseconds for rapid clicks. */
    debounceMs = 300

    /* ═══════════════ End Configuration ═══════════════ */
}
```

### JSDoc & general quality

- Every class, constructor, method, property, and config value has a `/** … */`
  JSDoc block (`@param`, `@returns`, `@type`, `@fires` where relevant).
- Use `const` by default, `let` only when reassigning, never `var`.
- Prefer template literals; avoid magic numbers/strings (promote to config).
- Use meaningful names; no dead code or commented-out experiments.
- Log user actions with `console.log()`, problems with `console.warn()` /
  `console.error()`. Include the class and method name in the message.

---

## Required comment block (top of the HTML file)

Place a commented-out summary at the very top of the generated HTML. List every
outgoing action, every incoming action (with parameter names and types), board
object operations, and any warnings.

```html
<!--
  WIDGET SUMMARY
  ==============

  Outgoing actions (triggered by user interaction via alleo.triggerAction):
    - onloaded ()
    - action-id-1 (param1: string, param2: number)

  Incoming actions (handled via alleo.onIncomingAction):
    - action-id-3 (param1: string)

  Board object content:
    - Reads object at whitelist index 1
    - Writes (replace) object at whitelist index 1

  ⚠ WARNING: This widget requires "Enable reading/writing board object content"
             and board objects added to the whitelist in widget settings.

  ⚠ WARNING: This widget requests microphone access via alleo.mic. The user must
             enable "Allow content access to the microphone". (Camera access is
             not available inside the iframe.)
-->
```

- `onloaded` is **always the first outgoing action listed.**
- Include `⚠ WARNING` lines only when the widget genuinely needs those features.
- Omit sections that have no entries.
- This comment is the human-readable summary; the importable settings footer (see
  *Importable settings footer* below) is the machine-readable source of truth for
  the actions. Keep them consistent.

---

## Sandboxed iframe — blocked features

The iframe is sandboxed. The following are **unavailable** — do not use them:

| Blocked feature                                                               | Use instead                                            |
|-------------------------------------------------------------------------------|--------------------------------------------------------|
| `localStorage` / `sessionStorage`                                             | `alleo.setSyncedStatus` / `alleo.onSyncedStatusUpdate` |
| `document.cookie`, `IndexedDB`                                                | `alleo.setSyncedStatus` / `alleo.onSyncedStatusUpdate` |
| `window.open()`, top-level navigation                                         | — (not allowed)                                        |
| `alert()`, `confirm()`, `prompt()`                                            | build in-page UI instead                               |
| `navigator.geolocation`                                                       | —                                                      |
| `getUserMedia()` / `getDisplayMedia()`                                        | `alleo.mic` for microphone (no camera)                 |
| `Notification`, `navigator.clipboard.writeText()`                             | —                                                      |
| `requestFullscreen()`, `requestPointerLock()`                                 | —                                                      |
| Service workers, `PaymentRequest`, Web Bluetooth/USB, `navigator.credentials` | —                                                      |
| Device sensors, same-origin parent access                                     | —                                                      |

**Allowed:** JavaScript, HTML forms (incl. submission), `fetch` /
`XMLHttpRequest` to external APIs, `<canvas>`, Web Workers, WebSockets, Web Audio
(without mic input), and standard DOM APIs not listed above.

Full list: `docs/LIBRARY.md` → *Limitations*.

---

## Alleo E-widget SDK

You **must** use the Alleo SDK (the `alleo` global) whenever the widget contains
JavaScript. It bridges the iframe and the Alleo board.

### Including the SDK (self-contained, the default)

Add the self-replacing `<EWidgetSDK />` tag once, before any other script,
anywhere in `<body>`. **Never** add a closing `</EWidgetSDK>` tag.

```html

<body>
<EWidgetSDK/>
<!-- your content -->
<script>
    // alleo is available here
</script>
</body>
```

### Including the SDK (external URL hosting only)

If hosting externally, do **not** use `<EWidgetSDK />`. Instead:

```html

<script src="https://unpkg.com/@withalleo/ewidget-utils/dist/ewidget-utils.umd.cjs"></script>
<script>
    const alleo = AlleoEWidget.getEmbedWidgetMessenger({debug: false})
</script>
```

### SDK API reference (summary)

| Method                                                   | Purpose                                                                                          |
|----------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| `alleo.triggerAction(actionId, data?)`                   | Fire an outgoing action to the board.                                                            |
| `alleo.onIncomingAction(handler)`                        | Handle actions from other board objects. Returns unsubscribe.                                    |
| `alleo.setSyncedStatus(status)`                          | Shallow-merge key/values into shared state.                                                      |
| `alleo.requestSyncedStatus()`                            | Request current shared state (fires `onSyncedStatusUpdate` once).                                |
| `alleo.onSyncedStatusUpdate(handler)`                    | Subscribe to shared-state changes (always full state). Returns unsubscribe.                      |
| `alleo.addContent(params)`                               | Add a new object to the board (html / notepad / sticky-note / image / video).                    |
| `alleo.getBoardObjectContent(index, options?)`           | Read text of a whitelisted object. Returns `Promise<string[]>`.                                  |
| `alleo.replaceBoardObjectContent(index, text, options?)` | Replace a whitelisted object's text (fire-and-forget).                                           |
| `alleo.appendBoardObjectContent(index, text, options?)`  | Append to a whitelisted object's text (fire-and-forget).                                         |
| `alleo.onError(handler)`                                 | Handle errors from board-object operations. Returns unsubscribe.                                 |
| `alleo.mic.*`                                            | Microphone bridge (`start`, `stop`, `onTrack`, `onStarted`, `onStopped`, `onError`, `isActive`). |

Full signatures, parameter shapes, and full samples: **`docs/LIBRARY.md`** and
**`docs/ADVANCED.md`**. Runnable examples: the **`samples/`** folder.

Key notes:

- Board object `index` is **1-based** (position in the whitelist), not an object id.
- For `alleo.mic`, register `onTrack` **before** `start()`, handle `onError`, and
  call `stop()` when audio is no longer needed.

---

## Actions — critical rules

### `onloaded` (mandatory)

Every widget must call `alleo.triggerAction('onloaded')` exactly **once**, after
it is fully initialized: DOM ready, listeners attached, `requestSyncedStatus()`
called, and any initial fetch completed.

### Outgoing actions

Fire `alleo.triggerAction` for **every meaningful user interaction** — button
clicks, form submits, toggles, committed input changes (`change`/`blur`, not each
keystroke), selections, slider/drag-end, color picks. Pass relevant data as the
second argument. Use lowercase, hyphenated `noun-verbed` ids (`item-selected`,
`form-submitted`).

### Incoming actions

Register `alleo.onIncomingAction` so the widget can be driven externally. For
each outgoing action consider an incoming counterpart (`item-selected` →
`select-item`). Provide at least a `reset` / `set-state` for mutable state. Always
treat `data` and its properties as optional and guard against missing values.
Use `verb-noun` ids for incoming actions (`select-item`, `reset-form`).

---

## State synchronization

If the widget has shared/interactive state and synchronization is appropriate:

- After **every** interaction that changes what the widget displays, call
  `alleo.setSyncedStatus(...)` immediately after updating the local UI.
- On startup, call `alleo.requestSyncedStatus()` and apply the returned state
  **before** firing `onloaded`.
- `onSyncedStatusUpdate` must cover every key the widget writes; rebuild the UI
  from the received status (the synced status is the source of truth).
- Store minimal, JSON-serializable state. Batch related keys into one
  `setSyncedStatus` call. Handle missing keys gracefully with sensible defaults.

```js
alleo.onSyncedStatusUpdate((status) => this.applySyncedStatus(status))
alleo.requestSyncedStatus()
// ...later, on interaction:
alleo.setSyncedStatus({selectedIndex: 2, filterText: 'active'})
```

---

## Required widget settings (list these in the build note)

| Capability                  | Settings the user must enable                                                                                   |
|-----------------------------|-----------------------------------------------------------------------------------------------------------------|
| Outgoing / incoming actions | **Enable Alleo board features for enclosed content**                                                            |
| Add content to board        | Board features + **Enable adding new content to the board**                                                     |
| Synced status               | Board features + **Enable synchronizing status**                                                                |
| Board object content        | Board features + **Enable reading/writing board object content** + add objects to **Whitelisted board objects** |
| Microphone (`alleo.mic`)    | **Allow content access to the microphone**                                                                      |

---

## Importable settings footer

To make the widget **importable as a single file**, append a settings footer at
the **very end** of the file, after the closing `</html>`. The footer is one HTML
comment whose JSON pre-configures the Alleo settings so they match what the HTML
actually does. On import, Alleo stores everything before the footer as the
widget's HTML and applies the footer settings. Without it, the user must enable
every capability by hand.

The footer is **mandatory and not optional**, and so is its full set of keys.
**Every generated widget must emit the footer with the complete required key set
below**, each key explicitly set. When a feature is unused, still include its key
and set it to its "off" value (`false` / `[]` / empty) — never omit a required
key.

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

- The footer **must be the last content in the file**, after `</html>`; only
  whitespace may follow `-->`.
- It **must** contain the literal token `WIDGETSETTINGS:` immediately before the
  JSON, which must be one `JSON.parse`-able object (double-quoted keys/strings, no
  trailing commas, no comments, no `undefined`).
- Never let the sequence `-->` appear inside a JSON string value.
- Include **only** allowed keys. Never emit `htmlContent`, `sourceType`, `url`, or
  `fileId` — those are derived on import; any other key aborts the import.
- **All keys shown above are required in every footer** — present and explicitly
  set. Set `enableIframeCommunication: true` whenever the HTML uses `alleo.*`, and
  turn each feature flag (`enableSyncedStatus`, `enableIframeFuncAddContent`,
  `enableIframeFuncBoardObjectContent`, `iframeAllowMicrophone`) `true` only when
  the HTML actually uses it — otherwise keep it `false`. Do not drop unused keys.
- Add one `outgoingActions` entry per `alleo.triggerAction('<id>', data)` (one
  `outputData` slot per `data` key) and one `incomingActions` entry per id handled
  in `alleo.onIncomingAction` (with `inputData` slots). The ids and parameter ids
  **must exactly match** the strings used in the HTML. Do **not** list `onloaded`.

The full key list, the `StoredActionTrigger` shape, and a worked example are in
**`docs/IMPORT-FOOTER.md`**. The `samples/*.Alleo-eWidget.txt` files each show a
complete HTML document plus a matching footer.

---

## Responsive layout

- Assume a default size of **1280×720**, but the user can resize/scale the widget
  to any size or ratio — design responsively.
- Use `box-sizing: border-box` globally to avoid overflow.
- Scroll inside the iframe only if needed; prefer fitting content. If scrolling is
  unavoidable, note it in the build note.

---

## Checklist before finalizing

- [ ] Read `idea.txt`; recorded any assumptions in the build note.
- [ ] Output written to `dist/<name>.Alleo-eWidget.txt` (HTML content + settings footer, `.txt` extension).
- [ ] Build note written to `dist/<name>.README.md`.
- [ ] Single self-contained HTML document; no external file references (other than API/CDN).
- [ ] `<EWidgetSDK />` (or the external-URL script) loads before any other script.
- [ ] Top-of-file `WIDGET SUMMARY` comment lists all outgoing/incoming actions with params.
- [ ] Summary lists board object operations (whitelist indices) if used.
- [ ] `onloaded` fired exactly once after full init, and listed first.
- [ ] No links/navigations leaving the iframe; no blocked APIs used.
- [ ] Layout is responsive; does not rely on a fixed iframe size.
- [ ] Outgoing actions fire on user interactions; incoming actions are handled.
- [ ] If synced: state writes on every change, `onSyncedStatusUpdate` covers all keys, `requestSyncedStatus` called on
  startup before `onloaded`.
- [ ] `alleo.onError` registered if any board-object operations are used.
- [ ] If using mic: `onTrack` registered before `start()`, `onError` handled, `stop()` called when done, and a ⚠ warning
  added to the summary.
- [ ] Valid plain ES2020+ only; class-based; config block at top with JSDoc; every member documented.
- [ ] Importable `WIDGETSETTINGS:` footer appended as the **last** content, after `</html>`; valid JSON, only allowed
  keys, **all required keys present and explicitly set** (unused features `false` / `[]` / empty), feature flags and
  action ids/params match the HTML, `onloaded` not listed.

---

## Reference documentation (in this repo)

Start here: [`docs/README.md`](docs/README.md) is the index. The docs are split by
audience so you know what to read while building vs. what is user-facing advice.

**Authoring docs — consult while writing widget code:**

- `docs/LIBRARY.md` — SDK methods with full examples and the limitations list. **Primary SDK manual.**
- `docs/ADVANCED.md` — full type definitions and message protocol.
- `docs/IMPORT-FOOTER.md` — the importable settings footer: format, allowed keys, and worked example.
- `docs/AI-INSTRUCTIONS.md` — the full upstream authoring guide (this file is the adapted, authoritative version).

**Reference docs — user-facing advice, not authoring instructions (you usually do not need these):**

- `docs/reference/HELP.md` — end-user help for the E-widget.
- `docs/reference/SECURITY.md` — sandbox tiers and security options.
- `docs/reference/TRACKED-EVENTS.md` — analytics events emitted by the E-widget.

**Runnable examples:** the `samples/` folder — one importable
`*.Alleo-eWidget.txt` widget per SDK capability (each is full HTML plus a matching
`WIDGETSETTINGS:` footer).
