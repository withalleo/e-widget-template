# Import Footer — Generating importable Alleo E-widget files

You are generating a **single importable file** for an **Alleo E-widget**.
Read every rule in this file before writing the footer. This document describes the **settings footer** that is appended to the HTML; for the rules that govern the HTML body itself, follow [`../AGENTS.md`](../AGENTS.md) (the source of truth) and the SDK manual in [`LIBRARY.md`](LIBRARY.md).

---

## What an importable file is

An importable file is **plain text** with two parts, in this exact order:

1. **The HTML content** — exactly the single self-contained HTML document you generated following `AI-INSTRUCTIONS.md`.
2. **The settings footer** — a single HTML comment, placed at the very end of the file, that carries the widget configuration as JSON.

When the user imports the file, Alleo:

- splits the text at the settings footer,
- stores everything **before** the footer as the widget's `htmlContent` and forces `sourceType` to `Html`,
- parses the JSON inside the footer and applies the whitelisted settings.

The recommended file name suffix is `.Alleo-eWidget.txt` (`.Alleo-eWidget` is also accepted). The `.txt` extension keeps the file uploadable as a plain document.

---

## Exact footer format

Append the footer at the **very end** of the file, after the closing `</html>`. Use this exact shape:

```html
<!--
WIDGETSETTINGS:
{
    "enableIframeCommunication": true,
    "outgoingActions": []
}
-->
```

Hard requirements — the import parser will silently reject the file (or ignore the footer) if any are violated:

- The footer **must be the last thing in the file.** Nothing but optional trailing whitespace may follow the closing `-->`.
- The comment **must** contain the literal token `WIDGETSETTINGS:` immediately before the JSON. Whitespace/newlines around it are fine.
- The content between `WIDGETSETTINGS:` and `-->` **must be a single valid JSON object** (`{ ... }`). It is parsed with `JSON.parse`; trailing commas, comments, single quotes, or `undefined` will make parsing fail and the whole import is rejected.
- The JSON **must not** contain the sequence `-->` anywhere inside string values, or the comment terminates early. Avoid embedding raw HTML comments in any string setting.
- Only include keys from the **allowed settings list** below. Any key outside that list throws an error and aborts the import — do **not** add `htmlContent`, `sourceType`, `url`, or `fileId`; those are derived automatically.
- Every key you include is optional. Omit a setting to fall back to its default; only include the settings your widget actually needs.

Separate the HTML and the footer with a blank line for readability (the parser trims trailing whitespace from the HTML automatically).

---

## Allowed settings

Only the following keys may appear in the footer JSON. Use the exact key names, casing, and value types shown. Anything else aborts the import.

### Board features & SDK

| Key                                  | Type                               | Default | Meaning                                                                                                                                                                                                                                          |
| ------------------------------------ | ---------------------------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `enableIframeCommunication`          | `boolean`                          | `false` | Master switch for the Alleo SDK bridge. **Must be `true`** if the HTML uses `alleo.*` (actions, synced status, add content, board object content, mic).                                                                                          |
| `enableIframeFuncAddContent`         | `boolean`                          | `false` | Allows `alleo.addContent(...)`. Requires `enableIframeCommunication: true`.                                                                                                                                                                      |
| `newContentContainer`                | `string` (board object id) or omit | —       | Optional container object that newly added content is placed into. Leave it out unless the user picked a specific container; you cannot invent a valid id.                                                                                       |
| `enableSyncedStatus`                 | `boolean`                          | `false` | Allows `alleo.setSyncedStatus` / `requestSyncedStatus` / `onSyncedStatusUpdate`. Requires `enableIframeCommunication: true`.                                                                                                                     |
| `enableIframeFuncBoardObjectContent` | `boolean`                          | `false` | Allows reading/writing whitelisted board objects. Requires `enableIframeCommunication: true`.                                                                                                                                                    |
| `boardObjectWhitelist`               | `string[]` (board object ids)      | `[]`    | Ordered list of board objects the widget may read/write. The 1-based index in this array is what the HTML passes to `alleo.getBoardObjectContent(index, ...)`. You usually cannot know real ids — leave it `[]` and tell the user to fill it in. |
| `iframeAllowMicrophone`              | `boolean`                          | `false` | Enables the `alleo.mic` bridge. Requires `enableIframeCommunication: true`.                                                                                                                                                                      |

### Actions

| Key               | Type                    | Default | Meaning                                                           |
| ----------------- | ----------------------- | ------- | ----------------------------------------------------------------- |
| `outgoingActions` | `StoredActionTrigger[]` | `[]`    | Actions the widget fires via `alleo.triggerAction('<id>', data)`. |
| `incomingActions` | `StoredActionTrigger[]` | `[]`    | Actions the widget handles via `alleo.onIncomingAction(...)`.     |

`StoredActionTrigger` shape:

```jsonc
{
    "id": "item-selected", // string, required. Matches the id used in the HTML. Pattern: ^[a-zA-Z0-9_-]+$
    "label": "Item selected", // string, required. Human-readable name shown in the board UI.
    "outputData": [
        // only for outgoingActions. Optional. Omit if the action carries no data.
        {
            "id": "value", // string, required. Pattern: ^[a-zA-Z0-9_-]+$. This is the key in the data object.
            "label": "Value", // string, required.
            "type": "Text", // one of: "Text" | "Number" | "Boolean" | "Numbers" | "AnyArray" | "Any"
        },
    ],
    "inputData": [
        /* same shape */
    ], // only for incomingActions. Optional.
}
```

Rules for actions:

- Use `outputData` **only** inside `outgoingActions`, and `inputData` **only** inside `incomingActions`.
- The parameter `type` value is the action socket data type, given as the exact (case-sensitive) enum name: `"Text"`, `"Number"`, `"Boolean"`, `"Numbers"` (number array), `"AnyArray"` (any array), or `"Any"` (avoid unless necessary).
- **Do not** declare the mandatory `onloaded` action here. `onloaded` is a fixed lifecycle signal that does not need an `outgoingActions` entry — list it only in the HTML summary comment, not in the footer.
- Each data slot `id` is the exact key the HTML reads/writes in the action `data` object — keep them identical.

### Display & appearance

| Key                         | Type      | Default         | Meaning                                                                                                       |
| --------------------------- | --------- | --------------- | ------------------------------------------------------------------------------------------------------------- |
| `iframeUnloadWhenOffScreen` | `boolean` | `true`          | Unload the iframe when scrolled off-screen. Keep `true` unless the widget must keep running while off-screen. |
| `iframeDisableUserActions`  | `boolean` | `false`         | Block all pointer/keyboard interaction with the content. Use only for display-only widgets.                   |
| `iframeDisableScrolling`    | `boolean` | `false`         | Prevent scrolling inside the iframe.                                                                          |
| `backgroundColor`           | `string`  | `"transparent"` | CSS color applied behind the content (e.g. `"#051825"`, `"transparent"`).                                     |
| `overwriteTextColor`        | `boolean` | `false`         | When `true`, Alleo overrides the content's text color with `textColor`.                                       |
| `textColor`                 | `string`  | theme primary   | CSS color used when `overwriteTextColor` is `true`.                                                           |

### Sandbox / security

| Key                          | Type          | Default         | Meaning                                                                              |
| ---------------------------- | ------------- | --------------- | ------------------------------------------------------------------------------------ |
| `iframeAllowScripts`         | `boolean`     | `true`          | Allow JavaScript to run. **Must be `true`** for any widget with `<script>`.          |
| `iframeAllowForms`           | `boolean`     | `true`          | Allow HTML form submission.                                                          |
| `iframeAllowOrientationLock` | `boolean`     | `false`         | Allow the content to lock screen orientation.                                        |
| `iframeAllowPointerLock`     | `boolean`     | `false`         | Allow the Pointer Lock API. Note: blocked by org security policy; do not rely on it. |
| `iframeReferrerPolicy`       | `string` enum | `"no-referrer"` | Referrer policy. One of `"no-referrer"`, `"origin"`, `"strict-origin"`.              |

> Some unsafe sandbox flags (same-origin, modals, top navigation, fullscreen, popups, presentation, downloads) exist in the widget but are **not importable** and are stripped by policy. Never put them in the footer.

---

## Make the settings match the generated HTML

The footer is not decoration — it must reflect what the HTML code actually does. Before writing it, scan your generated HTML and set the flags accordingly:

1. **Scripts:** if the HTML has any `<script>`, set `iframeAllowScripts: true` (it is the default, but be explicit when the widget depends on JS).
2. **SDK usage:** if the HTML references `alleo.` anywhere (including `<EWidgetSDK />`), set `enableIframeCommunication: true`. Without it, every `alleo.*` call is a no-op.
3. **Actions:** for every `alleo.triggerAction('<id>', { <param>: ... })` in the HTML, add a matching entry to `outgoingActions` with the same `id` and one `outputData` slot per `data` key (matching `id` and a sensible `type`). For every action id handled in `alleo.onIncomingAction`, add a matching `incomingActions` entry, with `inputData` slots for each expected `data` key. **The footer's action ids and parameter ids must be identical to the strings used in the HTML.**
4. **Synced status:** if the HTML calls `alleo.setSyncedStatus` / `requestSyncedStatus` / `onSyncedStatusUpdate`, set `enableSyncedStatus: true`.
5. **Add content:** if the HTML calls `alleo.addContent`, set `enableIframeFuncAddContent: true`.
6. **Board object content:** if the HTML calls `alleo.getBoardObjectContent` / `replaceBoardObjectContent` / `appendBoardObjectContent`, set `enableIframeFuncBoardObjectContent: true`. Leave `boardObjectWhitelist` as `[]` (you cannot know real object ids) and tell the user to add the objects in the order the HTML's 1-based indices expect.
7. **Microphone:** if the HTML uses `alleo.mic`, set `iframeAllowMicrophone: true` (and `enableIframeCommunication: true`).
8. **Forms:** if the HTML submits a `<form>`, keep `iframeAllowForms: true`.
9. **Appearance:** if you chose a specific background in the design, mirror it in `backgroundColor` so the iframe frame matches the content.

If a capability is **not** used by the HTML, leave its flag at the default (omit it). Do not enable permissions the widget does not need.

---

## Validate before finishing

- [ ] The footer is the **last** content in the file, after `</html>`.
- [ ] The footer is a single `<!-- ... -->` comment containing the literal `WIDGETSETTINGS:` token.
- [ ] The JSON parses with `JSON.parse` — double-quoted keys/strings, no trailing commas, no comments, no `undefined`.
- [ ] No `-->` appears inside any JSON string value.
- [ ] Only allowed keys are present; no `htmlContent`, `sourceType`, `url`, or `fileId`.
- [ ] `enableIframeCommunication` is `true` whenever the HTML uses `alleo.*`.
- [ ] Every `outgoingActions` / `incomingActions` `id` and parameter `id` exactly matches the HTML.
- [ ] Each feature flag (`enableSyncedStatus`, `enableIframeFuncAddContent`, `enableIframeFuncBoardObjectContent`, `iframeAllowMicrophone`) is set only when the HTML actually uses that feature.
- [ ] `onloaded` is documented in the HTML summary comment but **not** added to `outgoingActions`.

---

## Complete example

A button widget that fires `submit` (with a `label` string) and accepts an incoming `reset`, synchronising its state. Only the footer is shown here; the HTML above it follows `AI-INSTRUCTIONS.md`.

```html
<!doctype html>
<html lang="en">
    <!-- ... full self-contained widget HTML per AI-INSTRUCTIONS.md ... -->
</html>

<!--
WIDGETSETTINGS:
{
    "enableIframeCommunication": true,
    "enableSyncedStatus": true,
    "iframeAllowScripts": true,
    "backgroundColor": "#051825",
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
