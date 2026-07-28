# AGENTS.md — Build an Alleo E-widget from `idea.txt`

These are the operating instructions for any AI coding agent (Codex, Claude Code,
GitHub Copilot, etc.) working in this repository.

> This file is the single source of truth for the task and the coding rules.
> `CLAUDE.md` and `.github/copilot-instructions.md` point here. Read this whole
> file before writing any code. For full detail, the authoring docs in `docs/`
> (`AI-INSTRUCTIONS.md`, `LIBRARY.md`, `IMPORT-FOOTER.md`) are authoritative.

---

## Your task

1. **Read `idea.txt`** in the repository root. It contains a plain-language
   description of the widget the user wants.
2. **Design and generate one self-contained Alleo E-widget** (a single HTML
   document) that fulfils that idea, following every rule in this file.
3. **Write the result to the `dist/` directory as a `.Alleo-eWidget.txt` file**
   that contains the full HTML. Use a short, kebab-case name derived from the
   widget's purpose, e.g. `dist/countdown-timer.Alleo-eWidget.txt`. If you cannot
   derive a good name, use `dist/widget.Alleo-eWidget.txt`.
    - The file **must** have a `.txt` extension, not `.html` (the recommended
      suffix is `.Alleo-eWidget.txt`). The Alleo E-widget's file picker expects
      `.txt` for uploaded HTML (see `docs/reference/HELP.md`).
    - The file content is a normal, complete HTML document.
4. **Write a short build note** to `dist/<same-name>.README.md` (the same
   kebab-case base name, e.g. `dist/countdown-timer.README.md`) summarizing what
   you built, the assumptions you made, every outgoing/incoming action, the board
   features the user must enable, and the default widget size.

If `idea.txt` is empty or missing, do not invent a random widget — stop and tell
the user to fill in `idea.txt` first (they can copy `idea.sample.txt` as a
starting point), or ask them what to build if running interactively.

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
2. **Containment** — Default: no external network calls. Only fetch live data if
   the idea requires it and names the source. When you do fetch, use
   `alleo.fetchProtectedUrlJSON` / `fetchProtectedUrlText` / `fetchProtectedUrlBinary`
   (never a plain `fetch()`).
3. **State synchronization** — Default: if the widget has any shared/interactive
   state and the idea implies collaboration, synchronize it via
   `alleo.setSyncedStatus` / `alleo.onSyncedStatusUpdate`. Otherwise keep
   instances independent.
4. **Board object content** — Default: off. Only read/write other board objects
   if the idea explicitly asks for it.
5. **Color & font pickers** — Decide yourself which of the 4 theme pickers
   (background, primary, text, font) to expose in Alleo's settings panel via the
   footer's `enabledColorPickers`. Default background/primary/text **on** for
   visual widgets, font **off** unless typography is central.

### Build note — required summary

Always produce the `dist/<name>.README.md` build note with:

1. **A plain-English description** (2–4 short paragraphs) of what the widget does.
2. **The action list**:

   **Outgoing actions** (fired on user interaction):
    - `onloaded` — **always first; always present.** Fired automatically by
      `alleo.initialize()` after init.
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
- **Do not use any external images** (logos, icons, background images). Use inline
  SVG for icons.
- The code must be production-ready: clear, correct, organized, bug-free.

## Design guidelines

- Prefer flat, material surfaces; minimalist design. Avoid unnecessary borders and
  shadows, and avoid border radius where possible.
- Prefer a dark background (e.g. `#051825`) with light text (e.g. `#f9fff6`); leave
  the background transparent where possible.
- Build the widget around **3 colors** — background, primary (`#6da8ff` blue by
  default), and text. Derive any extra shades from these three (e.g. with CSS
  `color-mix()`), not unrelated colors.
- Use the Alleo-provided font (`var(--alleo-font, sans-serif)`) with at least 16px
  base font size.
- Design touch-first: every action must be usable with touch. Optional keyboard
  support may be added on top. Do **not** use the native HTML `<select>` element —
  build a custom dropdown instead.
- Fit the content to the widget; avoid excessive margins. Keep content centered
  when the user resizes the widget.

### Alleo-provided CSS variables (required)

`alleo.initialize()` applies the widget's configured theme as inline styles on
`document.body`. **Always define these four variables with sensible defaults at
the very top of the first `<style>` block** so the widget looks right before
initialize responds or when opened standalone, and reference them everywhere:

- `--alleo-background-color` (e.g. `#051825`)
- `--alleo-text-color` (e.g. `#f9fff6`)
- `--alleo-primary-color` (e.g. `#6da8ff`)
- `--alleo-font` (e.g. `'PT Root UI', 'Ubuntu Sans', Helvetica, Verdana, sans-serif`)

Alleo sets its real values as inline styles, so they always win over your defaults
— no `!important` needed.

---

## Code structure and quality

### Class-based architecture

All widget logic **must** live in one or more JavaScript classes. No loose
top-level functions or unstructured procedural code.

- Name the main class after the widget's purpose (`CountdownTimer`,
  `VotingWidget`, `DataDashboard`).
- Use descriptive method names; the code should read like documentation.
- Keep logic at most two nesting levels deep; use early returns / helper methods.
- Follow YAGNI: implement only what the idea needs; keep it minimal.

### Configuration section

Group **all** tunable values in one clearly marked configuration section at the
top of the HTML `<head>`, before any other style or logic. Keep it as the single
place a user edits to customize the widget — separate from the widget logic.

- Mark it with HTML comments:
  `<!-- =============== Configuration =============== -->` …
  `<!-- ============= END Configuration ============= -->`
- Put **JS-tunable values** (labels, text, timing, endpoints, thresholds, toggle
  flags) in a `settings` object inside a `<script>`.
- Put **theme values** in CSS custom properties on `body` inside a `<style>`,
  starting with the four `--alleo-*` variables above; reference them with
  `var(--…)` in the rest of the CSS.

```html
<head>
    <!-- ...meta/title... -->
    <!-- =============== Configuration =============== -->
    <script>
        /** User-tunable values. Edit these to customize the widget. */
        const settings = {
            /** @type {string} Text label shown on the primary action button. */
            buttonLabel: 'Submit',
            /** @type {number} Debounce interval in milliseconds for rapid clicks. */
            debounceMs: 300,
        }
    </script>
    <style>
        body {
            /* Defaults — alleo.initialize() overrides these with the configured theme */
            --alleo-background-color: #051825;
            --alleo-text-color: #f9fff6;
            --alleo-primary-color: #6da8ff;
            --alleo-font: 'PT Root UI', 'Ubuntu Sans', Helvetica, Verdana, sans-serif;
        }
    </style>
    <!-- ============= END Configuration ============= -->
</head>
```

The main class still keeps a short in-class config block, but it **reads from
`settings`** (and the CSS variables) rather than hard-coding values — so the
top-of-document section stays the single source of truth.

```js
class ExampleWidget {
    /* =============== Configuration =============== */

    /** @type {string} Text label shown on the primary action button. */
    buttonLabel = settings.buttonLabel ?? 'Submit'

    /** @type {number} Debounce interval in milliseconds for rapid clicks. */
    debounceMs = settings.debounceMs ?? 200

    /* ============= End Configuration ============= */
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

## Required comment blocks (top of the HTML file)

Place three standalone HTML comment blocks at the very top of the generated file,
in this order: `WIDGETNAME:`, `WIDGETDESCRIPTION:`, `WIDGETHELP:`.

```html
<!-- WIDGETNAME:
Submit Button
-->
<!-- WIDGETDESCRIPTION:
A button that fires a submit action and syncs its last-clicked state across all board instances.
-->
<!-- WIDGETHELP:
## What it does

Short, end-user-facing Markdown help describing what the widget does, how to use
it, and any settings worth knowing about.
-->
```

- `WIDGETNAME:` and `WIDGETDESCRIPTION:` are **required**, plain text only (no
  markdown, no line breaks).
- `WIDGETHELP:` is optional but recommended: Markdown, **500–2000 characters**.
- None of these blocks may contain the characters `<!--` or `-->` in their content.
- Do **not** list the actions here — the importable settings footer (see below) is
  the machine-readable source of truth for actions.

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
| Plain `fetch()` to a third-party URL (blocked by CORS)                         | `alleo.fetchProtectedUrlJSON` / `Text` / `Binary`      |

**Allowed:** JavaScript, HTML forms (incl. submission), `<canvas>`, Web Workers,
WebSockets, Web Audio (without mic input), and standard DOM APIs not listed above.
For any internet fetch, use the `alleo.fetchProtectedUrl*` methods.

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
<script src="https://widgets.withalleo.com/com.withalleo/embed-browser/assets/widgetAssets/ewidget-utils.umd.js"></script>
<script>
    const alleo = AlleoEWidget.getEmbedWidgetMessenger({debug: false})
</script>
```

### SDK API reference (summary)

| Method / property                                        | Purpose                                                                                          |
|----------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| `alleo.initialize(options?)`                             | Call once at startup. Requests synced status, params, user, and theme colors/font, then fires `onloaded` automatically. Returns a promise that always resolves. |
| `alleo.getParams`                                        | Read-only `Record<string,string>` of shared `__embed__` URL params (prefix stripped).            |
| `alleo.user`                                             | Read-only profile of the current board user.                                                    |
| `alleo.triggerAction(actionId, data?)`                   | Fire an outgoing action to the board.                                                            |
| `alleo.onIncomingAction(handler)`                        | Handle actions from other board objects. Returns unsubscribe.                                    |
| `alleo.setSyncedStatus(status)`                          | Shallow-merge key/values into shared state.                                                      |
| `alleo.requestSyncedStatus()`                            | Request current shared state (fires `onSyncedStatusUpdate` once). `initialize()` does this for you. |
| `alleo.onSyncedStatusUpdate(handler)`                    | Subscribe to shared-state changes (always full state). Returns unsubscribe.                      |
| `alleo.requestColorUpdate()`                             | Re-apply the configured colors/font as `--alleo-*` variables. `initialize()` does this for you.  |
| `alleo.addContent(params)`                               | Add a new object to the board (html / notepad / sticky-note / image / video).                    |
| `alleo.fetchProtectedUrlJSON/Text/Binary(keyId, input, init?)` | Fetch from the internet via the Alleo proxy. `keyId: null` for public URLs; a real `keyId` for authenticated APIs. |
| `alleo.getBoardObjectContent(index, options?)`           | Read text of a whitelisted object. Returns `Promise<string[]>`.                                  |
| `alleo.replaceBoardObjectContent(index, text, options?)` | Replace a whitelisted object's text (fire-and-forget).                                           |
| `alleo.appendBoardObjectContent(index, text, options?)`  | Append to a whitelisted object's text (fire-and-forget).                                         |
| `alleo.onError(handler)`                                 | Handle errors from board-object operations. Returns unsubscribe.                                 |
| `alleo.mic.*`                                            | Microphone bridge (`start`, `stop`, `onTrack`, `onStarted`, `onStopped`, `onError`, `isActive`). |

Full signatures, parameter shapes, and full samples: **`docs/LIBRARY.md`**.
Runnable examples: the **`samples/`** folder.

Key notes:

- Board object `index` is **1-based** (position in the whitelist), not an object id.
- For internet fetches, **always** prefer `alleo.fetchProtectedUrl*` over `fetch()`;
  pass `keyId: null` for public resources, a real `keyId` for authenticated APIs.
  **Never** hard-code an API key/secret in the HTML.
- For `alleo.mic`, register `onTrack` **before** `start()`, handle `onError`, and
  call `stop()` when audio is no longer needed.

---

## Actions — critical rules

### `onloaded` (mandatory, fired automatically)

Every widget signals readiness with the `onloaded` action exactly **once**, after
full initialization. **`alleo.initialize()` fires `onloaded` for you** once it
resolves — do **not** call `alleo.triggerAction('onloaded')` yourself. Make sure
all startup work (listeners attached, initial fetch, synced state applied) is done
before or awaited by `initialize()`.

`onloaded` is still a normal outgoing trigger from the board's point of view, so it
**must** have a matching `{ "id": "onloaded", "label": "Loaded" }` entry in the
footer's `outgoingActions`, or the board silently drops it.

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
- On startup, `await alleo.initialize()` (which requests the synced status and
  theme) and apply the returned state in `onSyncedStatusUpdate` **before**
  `onloaded` fires automatically.
- `onSyncedStatusUpdate` must cover every key the widget writes; rebuild the UI
  from the received status (the synced status is the source of truth).
- Store minimal, JSON-serializable state. Batch related keys into one
  `setSyncedStatus` call. Handle missing keys gracefully with sensible defaults.

```js
alleo.onSyncedStatusUpdate((status) => this.applySyncedStatus(status))
await alleo.initialize() // requests synced status + theme, then fires onloaded
// ...later, on interaction:
alleo.setSyncedStatus({selectedIndex: 2, filterText: 'active'})
```

---

## Required widget settings (list these in the build note)

| Capability                        | Settings the user must enable                                                                                   |
|-----------------------------------|-----------------------------------------------------------------------------------------------------------------|
| Outgoing / incoming actions       | **Enable Alleo board features for enclosed content**                                                            |
| Add content to board              | Board features + **Enable adding new content to the board**                                                     |
| Synced status                     | Board features + **Enable synchronizing status**                                                                |
| Board object content              | Board features + **Enable reading/writing board object content** + add objects to **Whitelisted board objects** |
| Networked fetches (`fetchProtectedUrl*`) | Board features + **Allow use of protected backends** (for public and authenticated calls; authenticated calls also need an `allowedProtectedBackends` entry + the owner entering the credential) |
| URL params / user profile         | **Share "\_\_embed\_\_" URL parameters** / **Allow access to the current user profile**                         |
| Microphone (`alleo.mic`)          | **Allow content access to the microphone**                                                                      |

---

## Importable settings footer

To make the widget **importable as a single file**, append a settings footer at
the **very end** of the file, after the closing `</html>`. The footer is one HTML
comment whose JSON pre-configures the Alleo settings so they match what the HTML
actually does. On import, Alleo stores everything before the footer as the
widget's HTML and applies the footer settings.

The footer is **mandatory and not optional**, and so is its full set of keys.
**Every generated widget must emit the footer with the complete required key set
below**, each key explicitly set. When a feature is unused, still include its key
and set it to its "off" value (`false` / `[]` / empty) — never omit a required
key.

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
    "enableBackendProxy": false,
    "incomingActions": [],
    "outgoingActions": [],
    "backgroundColor": "#051825",
    "textColor": "",
    "primaryColor": "",
    "font": "",
    "enabledColorPickers": { "background": false, "primary": false, "text": false, "font": false },
    "iframeAllowMicrophone": false,
    "requiredAPIfunctions": ["initialize"],
    "version": "1.20260728",
    "DefaultWidth": 1280,
    "DefaultHeight": 720
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
- Include **only** allowed keys. Never emit `htmlContent`, `sourceType`, `url`,
  `fileId`, or the legacy `overwriteTextColor` — those are derived on import or
  unsupported; any other key aborts the import.
- **All keys shown above are required in every footer** — present and explicitly
  set. Set `enableIframeCommunication: true` whenever the HTML uses `alleo.*`, and
  turn each feature flag (`enableSyncedStatus`, `enableIframeFuncAddContent`,
  `enableIframeFuncBoardObjectContent`, `enableBackendProxy`, `enableGetParams`,
  `enableGetUser`, `iframeAllowMicrophone`) `true` only when the HTML actually uses
  it — otherwise keep it `false`. Do not drop unused keys.
- Add one `outgoingActions` entry per `alleo.triggerAction('<id>', data)` (one
  `outputData` slot per `data` key) and one `incomingActions` entry per id handled
  in `alleo.onIncomingAction` (with `inputData` slots). The ids and parameter ids
  **must exactly match** the strings used in the HTML. **Do** list `onloaded` in
  `outgoingActions` (`{ "id": "onloaded", "label": "Loaded" }`, no `outputData`).
- Mirror the colors/font you used in `backgroundColor` / `textColor` /
  `primaryColor` / `font`, and set `enabledColorPickers` to the pickers you decided
  to expose.
- `requiredAPIfunctions` lists every distinct `alleo.*` function the HTML calls,
  without the `alleo.` prefix (e.g. `["initialize", "triggerAction"]`).
- `version` is `<version>.YYYYMMDD` — start at `1`, increment on every edit, and
  refresh the date to today. `DefaultWidth` / `DefaultHeight` are the design size.
- When `enableBackendProxy: true` and a non-null `keyId` is used, add a matching
  `allowedProtectedBackends[keyId]` entry with an exact, non-empty `allowedUrls`
  list. The footer never contains the actual secret.

The full key list, the `StoredActionTrigger` shape, protected-backend schema, and a
worked example are in **`docs/IMPORT-FOOTER.md`**. The `samples/*.Alleo-eWidget.txt`
files each show a complete HTML document plus a matching footer.

---

## Responsive layout

- Assume a default size of **1280×720**, but the user can resize/scale the widget
  to any size or ratio — design responsively. Mirror the design size in the
  footer's `DefaultWidth` / `DefaultHeight`.
- Use `box-sizing: border-box` globally to avoid overflow.
- Scroll inside the iframe only if needed; prefer fitting content. If scrolling is
  unavoidable, note it in the build note.

---

## Checklist before finalizing

- [ ] Read `idea.txt`; recorded any assumptions in the build note.
- [ ] Output written to `dist/<name>.Alleo-eWidget.txt` (HTML content + settings footer, `.txt` extension).
- [ ] Build note written to `dist/<name>.README.md`.
- [ ] Single self-contained HTML document; no external file references (other than API/CDN); no external images (inline SVG only).
- [ ] `<EWidgetSDK />` (or the external-URL script) loads before any other script.
- [ ] `WIDGETNAME:`, `WIDGETDESCRIPTION:`, `WIDGETHELP:` comment blocks present at the top, in that order.
- [ ] The four `--alleo-*` CSS variables are defined with defaults at the top of the first `<style>`.
- [ ] Summary/build note lists board object operations (whitelist indices) if used.
- [ ] `alleo.initialize()` awaited once on startup; `onloaded` fires automatically (not a manual `triggerAction('onloaded')`).
- [ ] No links/navigations leaving the iframe; no blocked APIs used; internet fetches use `alleo.fetchProtectedUrl*`.
- [ ] Layout is responsive and touch-first; no native `<select>`; does not rely on a fixed iframe size.
- [ ] Outgoing actions fire on user interactions; incoming actions are handled.
- [ ] If synced: state writes on every change, `onSyncedStatusUpdate` covers all keys, `initialize()` awaited on startup.
- [ ] `alleo.onError` registered if any board-object operations are used.
- [ ] If using mic: `onTrack` registered before `start()`, `onError` handled, `stop()` called when done, and a warning added to the build note.
- [ ] Valid plain ES2020+ only; class-based; tunables grouped in a top-of-document `<head>` configuration section (`settings` object + CSS variables) that the class reads from, with JSDoc; every member documented.
- [ ] Importable `WIDGETSETTINGS:` footer appended as the **last** content, after `</html>`; valid JSON, only allowed
  keys, **all required keys present and explicitly set** (unused features `false` / `[]` / empty), feature flags and
  action ids/params match the HTML, `onloaded` **is** listed in `outgoingActions`, `requiredAPIfunctions` and `version`
  present.

---

## Reference documentation (in this repo)

Start here: [`docs/README.md`](docs/README.md) is the index. The docs are split by
audience so you know what to read while building vs. what is user-facing advice.

**Authoring docs — consult while writing widget code:**

- `docs/LIBRARY.md` — SDK methods with full examples and the limitations list. **Primary SDK manual.**
- `docs/IMPORT-FOOTER.md` — the importable settings footer: format, allowed keys, protected backends, and worked example.
- `docs/AI-INSTRUCTIONS.md` — the full upstream authoring guide (this file is the adapted, authoritative version).

**Reference docs — user-facing advice, not authoring instructions (you usually do not need these):**

- `docs/reference/HELP.md` — end-user help for the E-widget.
- `docs/reference/SECURITY.md` — sandbox tiers and security options.
- `docs/reference/TRACKED-EVENTS.md` — analytics events emitted by the E-widget.

**Runnable examples:** the `samples/` folder — one importable
`*.Alleo-eWidget.txt` widget per SDK capability (each is full HTML plus a matching
`WIDGETSETTINGS:` footer).
