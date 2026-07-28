<!--
  AUDIENCE: Reference only — analytics events emitted by the E-widget.
  This is background for the human/admin, NOT authoring instructions. An agent
  does not need to read this to build a widget. For authoring rules see
  ../../AGENTS.md.
-->
# Embed Browser - Tracked Analytics Events

This document lists every analytics event emitted by the **embed-browser** widget.

Events are sent through various channels:

- `console.log()`
- the default backend logging service (eg. Azure Logs)
- if a board Analytics widget is present

All events are automatically enriched with `widgetName`, `widgetId`, and `widgetEntryPoint`, so those fields are not repeated below.

> **Privacy note.** Event payloads are intentionally PII-light, but not URL-free. Most events use hostname-only fields (`urlHost` via `safeHost()`), while some lifecycle payloads also include a raw `url` value for diagnostics. HTML/file content is only logged as `contentLength`. Audio chunks are **never** logged.

---

## Lifecycle events

### `embed-iframe-rendered`

Emitted every time a fresh `<iframe>` is created and appended to the DOM. Captures the full security posture of that iframe so audits can be performed post-hoc.

| Field                       | Type                        | Description                                                 |
| --------------------------- | --------------------------- | ----------------------------------------------------------- |
| `sourceType`                | `'Html' \| 'Url' \| 'File'` | Which content source was used.                              |
| `urlHost`                   | `string \| undefined`       | Hostname of the loaded URL (only for `Url` source).         |
| `url`                       | `string \| undefined`       | The loaded URL (only for `Url` source).                     |
| `fileId`                    | `string \| undefined`       | Board file id backing the content (only for `File` source). |
| `sandbox`                   | `string`                    | Final `sandbox` attribute applied to the iframe.            |
| `referrerPolicy`            | `string`                    | Final `referrerPolicy` applied to the iframe.               |
| `allowUnsecure`             | `boolean`                   | Whether highly-unsecure permissions were eligible.          |
| `strictSecurity`            | `boolean`                   | Whether strict-security mode is forced by deployment.       |
| `enableIframeCommunication` | `boolean`                   | Whether the postMessage bridge is enabled.                  |
| `enableAddContent`          | `boolean`                   | Whether the iframe can call `addContent`.                   |
| `enableBoardObjectContent`  | `boolean`                   | Whether iframe ↔ board-object text APIs are enabled.        |
| `enableMic`                 | `boolean`                   | Whether microphone access is granted to the iframe.         |

### `embed-iframe-loaded`

Emitted when the iframe is (re)loaded because the widget came into the viewport (or unloading-when-off-screen is disabled).

| Field        | Type                        | Description                      |
| ------------ | --------------------------- | -------------------------------- |
| `sourceType` | `'Html' \| 'Url' \| 'File'` | Which content source was loaded. |

### `embed-iframe-unloaded`

Emitted when the iframe is removed from the DOM because the widget left the viewport.

| Field    | Type           | Description                   |
| -------- | -------------- | ----------------------------- |
| `reason` | `'off-screen'` | Why the iframe was torn down. |

### `embed-destroyed`

Emitted from `destroy()` when the widget instance is being torn down. No payload.

### `embed-migrated-triggers`

Emitted by the one-time `migrate()` pass that converts the legacy `actionTriggers` field into the new `outgoingActions` shape.

| Field   | Type     | Description                                   |
| ------- | -------- | --------------------------------------------- |
| `count` | `number` | Number of legacy triggers that were migrated. |

---

## Content & settings events

### `embed-replace-content`

Emitted whenever the public `replace-content` action is invoked (typically by another widget on the board).

| Field           | Type                        | Description                                       |
| --------------- | --------------------------- | ------------------------------------------------- |
| `sourceType`    | `'Html' \| 'Url' \| 'File'` | New source type.                                  |
| `contentLength` | `number`                    | Length of the content string (0 for non-strings). |
| `urlHost`       | `string \| undefined`       | Hostname of the new URL (only for `Url`).         |

### `embed-settings-dialog-closed`

Emitted whenever the settings dialog closes. Fires regardless of whether the user canceled or saved.

| Field                | Type       | Description                                  |
| -------------------- | ---------- | -------------------------------------------- |
| `didAnythingChanged` | `boolean`  | `true` if at least one stored field changed. |
| `changedProperties`  | `string[]` | Names of the settings fields that changed.   |
| `changedCount`       | `number`   | `changedProperties.length`.                  |

> The payload is also enriched with every field from `embed-iframe-rendered` (sourceType, urlHost, sandbox, etc.) so the post-save security posture is captured alongside the diff.

### `embed-settings-auto-opened`

Emitted when the settings dialog is auto-opened on widget creation (because no source has been configured yet). No payload.

### `embed-invalid-url`

Emitted when `fillIframe()` is asked to load a URL that does not start with `http://` or `https://`. The widget then throws to abort the render.

| Field     | Type                  | Description                                                  |
| --------- | --------------------- | ------------------------------------------------------------ |
| `urlHost` | `string \| undefined` | Hostname extracted from the invalid URL (often `undefined`). |

### `embed-file-load-failed`

Emitted when loading content from a board file fails.

| Field    | Type                                   | Description          |
| -------- | -------------------------------------- | -------------------- |
| `reason` | `'missing-file-id' \| 'empty-content'` | Why the load failed. |

---

## Security policy events

### `embed-unsecure-option-stripped`

Emitted when `disableUnsecureOptions()` forcibly clears one or more sandbox-loosening flags because deployment policy (or strict-security mode) forbids them. This is the key audit signal that a user attempted to enable a risky option and was overridden.

| Field    | Type                            | Description                                        |
| -------- | ------------------------------- | -------------------------------------------------- |
| `reason` | `'strict-security' \| 'policy'` | Which policy triggered the strip.                  |
| `keys`   | `string[]`                      | The setting keys that were forced back to `false`. |

### `embed-same-origin-enabled`

Emitted in `renderContent()` whenever a fresh iframe is created with `allow-same-origin` **and** `allow-scripts` actually applied. This combination effectively breaks the sandbox and is the single highest-risk configuration.

| Field     | Type                  | Description                                  |
| --------- | --------------------- | -------------------------------------------- |
| `urlHost` | `string \| undefined` | Hostname of the loaded URL (only for `Url`). |

> **Note.** Whitelist matches that grant unsecure permissions are also logged at `info` level under the message `unsecure-whitelist-match` (with `rule` and `urlHost`). They are intentionally _not_ sent through `trackEvent` to avoid noise; promote to a tracked event if you need aggregate metrics.

---

## Iframe ↔ host messaging events

All of these are emitted by `EmbedBrowserIframeMessageHandler` while processing `postMessage` commands sent from the embedded content.

### `embed-iframe-command`

Emitted once per **successful** privileged command. Payloads never include the data the iframe sent - only metadata about the call shape.

| Field      | Type                                                                                                                         | Description                                                              |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| `command`  | `'triggerAction' \| 'setSyncedStatus' \| 'replaceBoardObjectContent' \| 'appendBoardObjectContent' \| 'requestProtectedUrl'` | Which command was executed.                                              |
| `actionId` | `string \| undefined`                                                                                                        | Trigger id (only for `triggerAction`).                                   |
| `hasData`  | `boolean \| undefined`                                                                                                       | Whether a data payload was attached (only for `triggerAction`).          |
| `keys`     | `string[] \| undefined`                                                                                                      | Top-level keys of the synced-status update (only for `setSyncedStatus`). |
| `index`    | `number \| undefined`                                                                                                        | 1-based whitelist index (only for board-object commands).                |
| `format`   | `'text' \| 'markdown' \| 'html' \| undefined`                                                                                | Requested text format (only for board-object commands).                  |
| `keyId`    | `string \| undefined`                                                                                                        | Protected-backend key id (only for `requestProtectedUrl`).               |

### `embed-iframe-denied`

Emitted whenever a command is rejected by a capability check (feature disabled, action not enabled, whitelist empty, index out of range, mic not allowed, etc.).

| Field      | Type                                                                                                 | Description                                                                                      |
| ---------- | ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| `command`  | `'triggerAction' \| 'addContent' \| 'micStart' \| 'boardObjectContent'`                              | The command that was denied. `'boardObjectContent'` is used for all three board-object commands. |
| `reason`   | `'feature-disabled' \| 'not-enabled' \| 'empty-whitelist' \| 'index-out-of-range' \| 'mic-disabled'` | Why the call was rejected.                                                                       |
| `actionId` | `string \| undefined`                                                                                | Action id that was rejected (only for `triggerAction`).                                          |
| `index`    | `number \| undefined`                                                                                | Offending index (only for `boardObjectContent`).                                                 |

### `embed-iframe-rate-limited`

Emitted when an incoming command is dropped because it exceeded the configured sliding-window rate limit. Watch this metric to detect runaway scripts or DoS attempts.

| Field      | Type     | Description                                       |
| ---------- | -------- | ------------------------------------------------- |
| `command`  | `string` | The rate-limited command.                         |
| `limit`    | `number` | Max allowed invocations per window.               |
| `windowMs` | `number` | Window length in milliseconds (default `10_000`). |

### `embed-iframe-unknown-command`

Emitted when the iframe posts a command name the handler does not recognize. Useful both for diagnosing SDK version drift and for spotting probing of the API.

| Field     | Type     | Description                    |
| --------- | -------- | ------------------------------ |
| `command` | `string` | The unrecognised command name. |

> **See also.** Messages that fail the `IframeMessaging.onMessage()` shape filters (wrong source, non-object data, missing `type`, missing `command`) are recorded at `debug` level as `iframe-message-rejected` with a `reason` and `origin`. They are intentionally _not_ sent through `trackEvent` because volume can be high; consider aggregating in a tracker widget if you need metrics.

---

## Add-content events (iframe → board mutations)

Emitted by `EmbedBrowserAddContentHandler` whenever the iframe creates new board objects via the `addContent` command.

### `embed-add-content`

Emitted **after** a content object has been successfully created.

| Field          | Type                                                         | Description                                                    |
| -------------- | ------------------------------------------------------------ | -------------------------------------------------------------- |
| `type`         | `'html' \| 'notepad' \| 'sticky-note' \| 'image' \| 'video'` | What was created.                                              |
| `hasContainer` | `boolean`                                                    | Whether the new object was placed into a configured container. |

### `embed-add-content-unknown-type`

Emitted when the iframe asks for a `type` value the handler does not support.

| Field  | Type                  | Description            |
| ------ | --------------------- | ---------------------- |
| `type` | `string \| undefined` | The unrecognised type. |

### `embed-add-content-failed`

Emitted when a known content type cannot be created due to a runtime precondition.

| Field    | Type                 | Description            |
| -------- | -------------------- | ---------------------- |
| `type`   | `'html'`             | Which add path failed. |
| `reason` | `'widget-not-found'` | Why it failed.         |

### `embed-add-content-invalid-params`

Emitted when required parameters for an add-content call are missing.

| Field    | Type                                | Description                  |
| -------- | ----------------------------------- | ---------------------------- |
| `type`   | `'image' \| 'video'`                | Which add path was called.   |
| `reason` | `'missing-url' \| 'missing-fileId'` | Which parameter was missing. |

---

## Microphone events

Emitted by `EmbedBrowserMicHandler`. Individual audio chunks are **never** tracked.

### `embed-mic-started`

Emitted after `MediaRecorder.start()` succeeds.

| Field        | Type                  | Description                                             |
| ------------ | --------------------- | ------------------------------------------------------- |
| `mimeType`   | `string`              | Codec the recorder negotiated.                          |
| `sampleRate` | `number \| undefined` | Requested sample rate, if any.                          |
| `timeslice`  | `number`              | Chunk emission interval in ms (clamped to `[20, 200]`). |

### `embed-mic-permission-denied`

Emitted when `navigator.mediaDevices.getUserMedia()` rejects (user denial, no device, secure-context failure, etc.).

| Field   | Type     | Description                   |
| ------- | -------- | ----------------------------- |
| `error` | `string` | Human-readable error message. |

### `embed-mic-error`

Emitted when the `MediaRecorder` raises an error or recorder setup fails after `getUserMedia` succeeded.

| Field   | Type                  | Description                                             |
| ------- | --------------------- | ------------------------------------------------------- |
| `error` | `string \| undefined` | Human-readable error message (only for setup failures). |

### `embed-mic-stopped`

Emitted when capture is stopped.

| Field        | Type     | Description                                  |
| ------------ | -------- | -------------------------------------------- |
| `chunkCount` | `number` | Number of chunks emitted during the session. |
