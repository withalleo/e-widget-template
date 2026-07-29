# Import Footer - Generating importable Alleo E-widget files

You are generating a **single importable file** for an **Alleo E-widget**. Read every rule in this file before writing the footer. This document describes the **settings footer** that is appended to the HTML; for the rules that govern the HTML body itself, follow [`AI-INSTRUCTIONS.md`](./AI-INSTRUCTIONS.md) (the source of truth) and the SDK manual in [`LIBRARY.md`](./LIBRARY.md).

---

## What an importable file is

An importable file is **plain text** with two parts, in this exact order:

1. **The HTML content** - exactly the single self-contained HTML document you generated following `AI-INSTRUCTIONS.md`.
2. **The settings footer** - a single HTML comment, placed at the very end of the file, that carries the widget configuration as JSON.

When the user imports the file, Alleo:

- splits the text at the settings footer,
- stores everything **before** the footer as the widget's `htmlContent` and forces `sourceType` to `Html`,
- parses the JSON inside the footer and applies the whitelisted settings.

The recommended file name suffix is `.Alleo-eWidget.txt` (`.Alleo-eWidget` is also accepted). The `.txt` extension keeps the file uploadable as a plain document.

---

## Exact footer format

Append the footer at the **very end** of the file, after the closing `</html>`.

> **The footer is mandatory and so is every key in it.** Unlike older guidance, the footer is **not** optional decoration and you may **not** cherry-pick keys. Every widget you generate **must** emit the footer with the **complete set of required keys** below - present, explicitly set, every time. When a feature is not used by the widget, still include its key and set it to its "off" value (`false`, `[]`, or the documented empty/default value). Never omit a required key to "fall back to a default".

### Required keys - always present in every footer

These keys **must appear in every generated footer**, with explicit values:

| Key                                  | "Off" / empty value when unused                                                    |
| ------------------------------------ | ---------------------------------------------------------------------------------- |
| `iframeAllowScripts`                 | `true` (keep `true` - every widget runs JS)                                        |
| `iframeAllowForms`                   | `false`                                                                            |
| `iframeAllowOrientationLock`         | `false`                                                                            |
| `iframeAllowPointerLock`             | `false`                                                                            |
| `iframeDisableUserActions`           | `false`                                                                            |
| `iframeDisableScrolling`             | `false`                                                                            |
| `iframeReferrerPolicy`               | `"no-referrer"`                                                                    |
| `iframeUnloadWhenOffScreen`          | `true`                                                                             |
| `enableIframeCommunication`          | `false`                                                                            |
| `enableGetParams`                    | `false`                                                                            |
| `enableGetUser`                      | `false`                                                                            |
| `enableIframeFuncAddContent`         | `false`                                                                            |
| `enableSyncedStatus`                 | `false`                                                                            |
| `enableIframeFuncBoardObjectContent` | `false`                                                                            |
| `enableBackendProxy`                 | `false`                                                                            |
| `incomingActions`                    | `[]`                                                                               |
| `outgoingActions`                    | `[]`                                                                               |
| `backgroundColor`                    | `"transparent"` (or your chosen background)                                        |
| `textColor`                          | `""` (or your chosen text color)                                                   |
| `primaryColor`                       | `""` (or your chosen primary/accent color)                                         |
| `font`                               | `""` (keeps the Alleo default font)                                                |
| `enabledColorPickers`                | `{ "background": false, "primary": false, "text": false, "font": false }`          |
| `iframeAllowMicrophone`              | `false`                                                                            |
| `version`                            | `"1.<YYYYMMDD>"` (today's date; bump the number and refresh the date on each edit) |
| `requiredAPIfunctions`               | `[]`                                                                               |

Use this canonical shape as the starting point and turn on only the flags the widget actually needs (see _Make the settings match the generated HTML_ below):

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
    "version": "1.20260625",
    "DefaultWidth": 1280,
    "DefaultHeight": 720
}
-->
```

Hard requirements - the import parser will silently reject the file (or ignore the footer) if any are violated:

- The footer **must be the last thing in the file.** Nothing but optional trailing whitespace may follow the closing `-->`.
- The comment **must** contain the literal token `WIDGETSETTINGS:` immediately before the JSON. Whitespace/newlines around it are fine.
- The content between `WIDGETSETTINGS:` and `-->` **must be a single valid JSON object** (`{ ... }`). It is parsed with `JSON.parse`; trailing commas, comments, single quotes, or `undefined` will make parsing fail and the whole import is rejected.
- The JSON **must not** contain the sequence `-->` anywhere inside string values, or the comment terminates early. Avoid embedding raw HTML comments in any string setting.
- Only include keys from the **allowed settings list** below. Any key outside that list throws an error and aborts the import - do **not** add `htmlContent`, `sourceType`, `url`, `fileId`, or `overwriteTextColor` (a legacy key that no longer exists); those are derived automatically or unsupported.
- **Every key in the "Required keys" table above is mandatory** - include all of them in every footer, with an explicit value. Set unused features to their "off" value (`false` / `[]` / empty), never omit them. Keys not in that table (e.g. `newContentContainer`, `boardObjectWhitelist`, `DefaultWidth`, `DefaultHeight`) remain optional and are added only when relevant.
- `DefaultWidth` and `DefaultHeight` are optional but **must be supplied together** as positive numbers when present; on import they set the widget's pixel size via `setSize`. Use the fixed design size you built for (default `1280`×`720`).

Separate the HTML and the footer with a blank line for readability (the parser trims trailing whitespace from the HTML automatically).

---

## Allowed settings

Only the following keys may appear in the footer JSON. Use the exact key names, casing, and value types shown. Anything else aborts the import.

### Board features & SDK

| Key                                  | Type                               | Default | Meaning                                                                                                                                                                                                                                                                                                                                                                                                        |
| ------------------------------------ | ---------------------------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `enableIframeCommunication`          | `boolean`                          | `false` | Master switch for the Alleo SDK bridge. **Must be `true`** if the HTML uses `alleo.*` (actions, synced status, add content, board object content, mic).                                                                                                                                                                                                                                                        |
| `enableIframeFuncAddContent`         | `boolean`                          | `false` | Allows `alleo.addContent(...)`. Requires `enableIframeCommunication: true`.                                                                                                                                                                                                                                                                                                                                    |
| `newContentContainer`                | `string` (board object id) or omit | -       | Optional container object that newly added content is placed into. Leave it out unless the user picked a specific container; you cannot invent a valid id.                                                                                                                                                                                                                                                     |
| `enableSyncedStatus`                 | `boolean`                          | `false` | Allows `alleo.setSyncedStatus` / `requestSyncedStatus` / `onSyncedStatusUpdate`. Requires `enableIframeCommunication: true`.                                                                                                                                                                                                                                                                                   |
| `enableIframeFuncBoardObjectContent` | `boolean`                          | `false` | Allows reading/writing whitelisted board objects. Requires `enableIframeCommunication: true`.                                                                                                                                                                                                                                                                                                                  |
| `boardObjectWhitelist`               | `string[]` (board object ids)      | `[]`    | Ordered list of board objects the widget may read/write. The 1-based index in this array is what the HTML passes to `alleo.getBoardObjectContent(index, ...)`. You usually cannot know real ids - leave it `[]` and tell the user to fill it in.                                                                                                                                                               |
| `iframeAllowMicrophone`              | `boolean`                          | `false` | Enables the `alleo.mic` bridge. Requires `enableIframeCommunication: true`.                                                                                                                                                                                                                                                                                                                                    |
| `enableGetParams`                    | `boolean`                          | `false` | Exposes `alleo.getParams` - the parent page's URL query parameters prefixed with `__embed__` (prefix stripped from the keys). Requires `enableIframeCommunication: true`; set `true` if the HTML reads `alleo.getParams`.                                                                                                                                                                                      |
| `enableGetUser`                      | `boolean`                          | `false` | Exposes `alleo.user` - the profile of the currently logged-in board user (name, email, organization, etc.). Requires `enableIframeCommunication: true`; set `true` if the HTML reads `alleo.user`.                                                                                                                                                                                                             |
| `enableBackendProxy`                 | `boolean`                          | `false` | Master switch for `alleo.fetchProtectedUrlJSON` / `fetchProtectedUrlText` / `fetchProtectedUrlBinary`. **Must be `true`** if the HTML calls any of them for any reason - including a public/unauthenticated fetch with `keyId: null` (CORS-bypass proxy), not just authenticated calls. Requires `enableIframeCommunication: true`. See [Protected backend connections](#protected-backend-connections) below. |

### Required API functions

| Key                    | Type       | Default | Meaning                                                                                                                                                                                                                                                                                                   |
| ---------------------- | ---------- | ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `requiredAPIfunctions` | `string[]` | `[]`    | Every `alleo.*` endpoint the HTML actually uses, listed **without** the `alleo.` prefix - this includes function calls **and** property reads like `alleo.getParams`/`alleo.user`, not just functions (e.g. `["initialize", "triggerAction", "onIncomingAction", "fetchProtectedUrlJSON", "getParams", "user"]`). List each name once even if used multiple times. Keep it in sync with the HTML whenever a reference is added or removed. |

### Protected backend connections

Use this feature whenever the generated HTML needs to fetch anything from the internet - it's the **only** supported way to do this, whether the request is public/unauthenticated (`keyId: null`, routed through a CORS-bypass proxy) or requires an **API key, bearer token, or any other credential** (a real `keyId`). See [`LIBRARY.md`](./LIBRARY.md#fetchprotectedurljson--fetchprotectedurltext--fetchprotectedurlbinary) for the full `alleo.fetchProtectedUrlJSON` / `fetchProtectedUrlText` / `fetchProtectedUrlBinary` reference. Always prefer this over a plain `fetch()`, and never:

- hard-code an API key/secret anywhere in the generated HTML/JS (it would be visible to anyone who opens the widget's settings or views the page source),
- ask the user to paste a secret into a `<script>` variable,
- or pass a secret through the `keyId: null` (public) path - it cannot carry credentials.

| Key                        | Type                                                                                                                                                               | Default | Meaning                                                                                                                                                                                                                                                                 |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `allowedProtectedBackends` | `Record<string, { backendSchema: { method: "QueryParam" \| "Header", template: string, connection: "Http" \| "WebSocket", allowedUrls: [string, ...string[]] } }>` | `{}`    | One entry per non-null `keyId` used in `alleo.fetchProtectedUrl*(keyId, url, init)`. Not needed for calls that pass `keyId: null` (public CORS-proxy fetch). Declares **which endpoints** that `keyId` may call - it never carries the actual secret value (see below). |

`backendSchema` fields, all **required** for every entry:

| Field         | Type                               | Meaning                                                                                                                                                                                                                                      |
| ------------- | ---------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `method`      | `"QueryParam"` \| `"Header"`       | Where the parent widget injects the credential into the request: as a query string parameter, or as a request header.                                                                                                                        |
| `template`    | `string`                           | The exact header/query-param assignment, using `{{secret.token}}` as the placeholder for the real credential. Format is `"<Name>={{secret.token}}"`, e.g. `"Authorization=Bearer {{secret.token}}"` or `"appid={{secret.token}}"`.           |
| `connection`  | `"Http"` \| `"WebSocket"`          | Use `"Http"` for one-off `fetch`-style calls (the vast majority of cases). `"WebSocket"` is for persistent connections and is not exercised by `fetchProtectedUrl*`.                                                                         |
| `allowedUrls` | `[string, ...string[]]` (1+ items) | **The exact URL(s) this `keyId` is permitted to call.** This is a hard security boundary enforced by the platform - a call to any URL not in this list is rejected before the credential is ever attached, no matter what the HTML requests. |

> ⚠️ **`allowedUrls` is mandatory and must be exact/narrow - this is not optional decoration.** List every specific endpoint the generated HTML actually calls (e.g. `"https://api.example.com/v1/forecast"`), not a wildcard or a bare domain you don't fully trust with every path under it. Omitting `allowedUrls`, leaving it empty, or making it overly broad defeats the entire purpose of the protected-backend feature and will be rejected or flagged during review.

Example - a widget that only needs public, unauthenticated fetches (`keyId: null`) - no `allowedProtectedBackends` entry is needed:

```jsonc
{
    // ...other required keys...
    "enableIframeCommunication": true,
    "enableBackendProxy": true,
}
```

Matching HTML call:

```js
const data = await alleo.fetchProtectedUrlJSON(null, 'https://api.example.com/data?id=42')
```

Example - a widget that calls one authenticated weather endpoint under the `keyId` `"weatherApi"`:

```jsonc
{
    // ...other required keys...
    "enableIframeCommunication": true,
    "enableBackendProxy": true,
    "allowedProtectedBackends": {
        "weatherApi": {
            "backendSchema": {
                "method": "QueryParam",
                "template": "appid={{secret.token}}",
                "connection": "Http",
                "allowedUrls": ["https://api.openweathermap.org/data/2.5/weather"],
            },
        },
    },
}
```

Matching HTML call:

```js
const data = await alleo.fetchProtectedUrlJSON('weatherApi', 'https://api.openweathermap.org/data/2.5/weather?q=Berlin')
```

Example - OpenAI Chat Completions with a bearer token under `keyId` `"openaiApi"`:

```jsonc
{
    // ...other required keys...
    "enableIframeCommunication": true,
    "enableBackendProxy": true,
    "allowedProtectedBackends": {
        "openaiApi": {
            "backendSchema": {
                "method": "Header",
                "template": "Authorization=Bearer {{secret.token}}",
                "connection": "Http",
                "allowedUrls": ["https://api.openai.com/v1/chat/completions"],
            },
        },
    },
}
```

Matching HTML call:

```js
const completion = await alleo.fetchProtectedUrlJSON('openaiApi', 'https://api.openai.com/v1/chat/completions', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
    },
    body: JSON.stringify({
        model: 'gpt-4.1-mini',
        messages: [{ role: 'user', content: 'Write a short summary' }],
    }),
})
```

**The footer never contains the actual API key/token.** After import, the board owner must open the widget's context menu and use the auto-added **"Configure External Connection `<keyId>`"** action to enter the real credential once; from then on every `fetchProtectedUrl*(keyId, …)` call is authenticated with it. If you configure `allowedProtectedBackends` in the footer but the user hasn't entered a credential yet, calls will fail until they do.

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
    "inputData": [/* same shape */], // only for incomingActions. Optional.
}
```

Rules for actions:

- Use `outputData` **only** inside `outgoingActions`, and `inputData` **only** inside `incomingActions`.
- The parameter `type` value is the action socket data type, given as the exact (case-sensitive) enum name: `"Text"`, `"Number"`, `"Boolean"`, `"Numbers"` (number array), `"AnyArray"` (any array), or `"Any"` (avoid unless necessary).
- **Always declare the mandatory `onloaded` action here**: add `{ "id": "onloaded", "label": "Loaded" }` to `outgoingActions` (no `outputData`). The board only accepts an `alleo.triggerAction(actionId, …)` call whose `actionId` is listed in `outgoingActions`, and `onloaded` is no exception - omitting it means the widget's `onloaded` signal is silently dropped.
- Each data slot `id` is the exact key the HTML reads/writes in the action `data` object - keep them identical.

### Display & appearance / colors & fonts

| Key                         | Type                                           | Default         | Meaning                                                                                                                                                                                                                                              |
| --------------------------- | ---------------------------------------------- | --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `iframeUnloadWhenOffScreen` | `boolean`                                      | `true`          | Unload the iframe when scrolled off-screen. Keep `true` unless the widget must keep running while off-screen.                                                                                                                                        |
| `iframeDisableUserActions`  | `boolean`                                      | `false`         | Set `true` when the user cannot interact with the content using a pointer (click/tap/drag/hover) - i.e. display-only content. The built-in color/font pickers don't count toward this - having only those doesn't make a widget pointer-interactive. |
| `iframeDisableScrolling`    | `boolean`                                      | `false`         | Prevent scrolling inside the iframe.                                                                                                                                                                                                                 |
| `backgroundColor`           | `string`                                       | `"transparent"` | Applied as `--alleo-background-color` (e.g. `"#051825"`, `"transparent"`).                                                                                                                                                                           |
| `textColor`                 | `string`                                       | theme           | Applied as `--alleo-text-color` (e.g. `"#f9fff6"`).                                                                                                                                                                                                  |
| `primaryColor`              | `string`                                       | theme           | Applied as `--alleo-primary-color` (e.g. `"#6da8ff"`).                                                                                                                                                                                               |
| `font`                      | `string`                                       | `""`            | Applied as `--alleo-font`. Leave `""` to keep the Alleo default font stack.                                                                                                                                                                          |
| `enabledColorPickers`       | `{ background, primary, text, font: boolean }` | all `false`     | For each key, `true` shows a matching color/font picker in Alleo's own widget settings panel so board editors can override that value independently of the HTML. Decide per widget which of the 4 to enable - see `AI-INSTRUCTIONS.md`.              |

### Sandbox / security

| Key                          | Type          | Default         | Meaning                                                                              |
| ---------------------------- | ------------- | --------------- | ------------------------------------------------------------------------------------ |
| `iframeAllowScripts`         | `boolean`     | `true`          | Allow JavaScript to run. **Must be `true`** for any widget with `<script>`.          |
| `iframeAllowForms`           | `boolean`     | `true`          | Allow HTML form submission.                                                          |
| `iframeAllowOrientationLock` | `boolean`     | `false`         | Allow the content to lock screen orientation.                                        |
| `iframeAllowPointerLock`     | `boolean`     | `false`         | Allow the Pointer Lock API. Note: blocked by org security policy; do not rely on it. |
| `iframeReferrerPolicy`       | `string` enum | `"no-referrer"` | Referrer policy. One of `"no-referrer"`, `"origin"`, `"strict-origin"`.              |

> Some unsafe sandbox flags (same-origin, modals, top navigation, fullscreen, popups, presentation, downloads) exist in the widget but are **not importable** and are stripped by policy. Never put them in the footer.

### Size

| Key             | Type              | Default | Meaning                                                                                                                      |
| --------------- | ----------------- | ------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `DefaultWidth`  | `number` (pixels) | `1280`  | Initial widget width applied via `setSize` on import. Optional; must be paired with `DefaultHeight`. Use your design width.  |
| `DefaultHeight` | `number` (pixels) | `720`   | Initial widget height applied via `setSize` on import. Optional; must be paired with `DefaultWidth`. Use your design height. |

> `DefaultWidth`/`DefaultHeight` are exported automatically from the widget's current size. When you author a footer by hand, set them together (both positive numbers) to the fixed size the HTML was designed for, or omit both to keep the user's current size.

### Version

| Key       | Type     | Default       | Meaning                                                                                                                      |
| --------- | -------- | ------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `version` | `string` | `"1.<today>"` | **Always required.** Version stamp in the form `<version>.YYYYMMDD`. Displayed in the widget's settings dialog after import. |

> The `version` value has two parts joined by a dot. `<version>` is an integer that **starts at `1`** for a new widget and is **incremented by one every time the widget is modified** (`1` → `2` → `3`). `YYYYMMDD` is the **date of the most recent change** - update it to today's date every time you modify the widget, not just when it's first created. Example: a widget first created on 25 June 2026 ships as `"1.20260625"`; after two edits three days later it becomes `"3.20260628"`. If no prior version exists, treat the edit as version `1` and use today's date.

---

## Make the settings match the generated HTML

The footer is not decoration - it must reflect what the HTML code actually does. Before writing it, scan your generated HTML and set the flags accordingly:

1. **Scripts:** if the HTML has any `<script>`, set `iframeAllowScripts: true` (it is the default, but be explicit when the widget depends on JS).
2. **SDK usage:** if the HTML references `alleo.` anywhere (including `<EWidgetSDK />`), set `enableIframeCommunication: true`. Without it, every `alleo.*` call is a no-op.
3. **Actions:** for every `alleo.triggerAction('<id>', { <param>: ... })` in the HTML, add a matching entry to `outgoingActions` with the same `id` and one `outputData` slot per `data` key (matching `id` and a sensible `type`) - **including `onloaded`**, which needs its own `{ "id": "onloaded", "label": "Loaded" }` entry with no `outputData`. For every action id handled in `alleo.onIncomingAction`, add a matching `incomingActions` entry, with `inputData` slots for each expected `data` key. **The footer's action ids and parameter ids must be identical to the strings used in the HTML.**
4. **Synced status:** if the HTML calls `alleo.setSyncedStatus` / `requestSyncedStatus` / `onSyncedStatusUpdate`, set `enableSyncedStatus: true`.
5. **Add content:** if the HTML calls `alleo.addContent`, set `enableIframeFuncAddContent: true`.
6. **Board object content:** if the HTML calls `alleo.getBoardObjectContent` / `replaceBoardObjectContent` / `appendBoardObjectContent`, set `enableIframeFuncBoardObjectContent: true`. Leave `boardObjectWhitelist` as `[]` (you cannot know real object ids) and tell the user to add the objects in the order the HTML's 1-based indices expect.
7. **Microphone:** if the HTML uses `alleo.mic`, set `iframeAllowMicrophone: true` (and `enableIframeCommunication: true`).
8. **Forms:** if the HTML submits a `<form>`, keep `iframeAllowForms: true`.
9. **URL parameters:** if the HTML reads `alleo.getParams`, set `enableGetParams: true`. If the HTML reads `alleo.user`, set `enableGetUser: true`.
10. **Internet fetches:** if the HTML calls `alleo.fetchProtectedUrlJSON` / `fetchProtectedUrlText` / `fetchProtectedUrlBinary` for any reason - public (`keyId: null`) or authenticated - set `enableBackendProxy: true`. For each non-null `keyId` used, also add an `allowedProtectedBackends` entry with `allowedUrls` listing the exact endpoint (s) that `keyId` calls (see [Protected backend connections](#protected-backend-connections)). Never embed the credential itself anywhere in the footer or HTML.
11. **Appearance:** mirror the background/primary/text colors and font you used in `backgroundColor`/`primaryColor`/`textColor`/`font` (each applied as the matching `--alleo-*` CSS variable) so the iframe frame matches the content.
12. **Color/font pickers:** set `enabledColorPickers` to the background/primary/text/font pickers you decided to expose in Alleo's own settings panel - this is a decision you make while designing the widget (see `AI-INSTRUCTIONS.md`), not something mechanically derived from the HTML.
13. **Size:** set `DefaultWidth`/`DefaultHeight` to the fixed pixel size you designed for (both positive numbers, default `1280`×`720`), or omit both to keep the user's current size.
14. **Version:** set `version` to `<version>.YYYYMMDD` - start `<version>` at `1` for a new widget and increment it on every modification; set `YYYYMMDD` to today's date every time you modify the widget (it tracks the most recent change, not the original creation date).
15. **Required API functions:** collect every distinct `alleo.*` reference found anywhere in the HTML (from steps 2–10 above and any others, e.g. `alleo.initialize`) into `requiredAPIfunctions`, listing each name once, without the `alleo.` prefix. This includes property reads such as `alleo.getParams` and `alleo.user`, not just function calls.

If a capability is **not** used by the HTML, **still include its key** in the footer and set it to its "off" value (`false` / `[]` / empty) - see the _Required keys_ table. Do not omit it, and do not enable permissions the widget does not need.

---

## Validate before finishing

- [ ] The footer is the **last** content in the file, after `</html>`.
- [ ] The footer is a single `<!-- ... -->` comment containing the literal `WIDGETSETTINGS:` token.
- [ ] The JSON parses with `JSON.parse` - double-quoted keys/strings, no trailing commas, no comments, no `undefined`.
- [ ] No `-->` appears inside any JSON string value.
- [ ] Only allowed keys are present; no `htmlContent`, `sourceType`, `url`, `fileId`, or `overwriteTextColor`.
- [ ] If present, `DefaultWidth` and `DefaultHeight` are supplied together as positive numbers matching the design size.
- [ ] **All required keys are present** (the full _Required keys_ table), each with an explicit value; unused features set to `false` / `[]` / empty rather than omitted.
- [ ] `backgroundColor`, `textColor`, `primaryColor`, and `font` mirror the theme actually used, and `enabledColorPickers` matches the color/font picker decision made while designing the widget.
- [ ] `version` is present in the form `<version>.YYYYMMDD` - number starts at `1` and is incremented on every modification; `YYYYMMDD` is refreshed to today's date on every modification.
- [ ] `enableIframeCommunication` is `true` whenever the HTML uses `alleo.*`.
- [ ] Every `outgoingActions` / `incomingActions` `id` and parameter `id` exactly matches the HTML.
- [ ] Each feature flag (`enableSyncedStatus`, `enableIframeFuncAddContent`, `enableIframeFuncBoardObjectContent`, `iframeAllowMicrophone`, `enableGetParams`, `enableGetUser`) is `true` only when the HTML actually uses that feature, and `false` otherwise.
- [ ] `onloaded` **is** added to `outgoingActions` (`{ "id": "onloaded", "label": "Loaded" }`, no `outputData`).
- [ ] `requiredAPIfunctions` lists every `alleo.*` endpoint actually used in the HTML (without the `alleo.` prefix) - including property reads like `getParams`/`user`, not just function calls - with no unused entries.

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
