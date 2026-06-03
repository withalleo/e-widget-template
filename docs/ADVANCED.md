<!--
  AUDIENCE: Authoring reference (advanced) — full type definitions and the
  iframe<->widget postMessage protocol. Read this only when you need exact
  signatures or message shapes beyond LIBRARY.md. Coding rules: ../AGENTS.md.
-->

# Advanced API Reference

Full type definitions and advanced usage details for `@withalleo/ewidget-utils`.
For common usage examples see [README.md](README.md).

## Table of Contents

- [Helper Functions](#helper-functions)
- [AlleoEwidgetUtils](#alleoEwidgetutils)
- [EmbedWidgetMessenger](#embedwidgetmessenger)
- [AlleoMic](#alleomic)
- [Message Types](#message-types)
    - [Outbound Commands](#outbound-commands-iframe--widget)
    - [Inbound Messages](#inbound-messages-widget--iframe)
- [Notes](#notes)

---

## Helper Functions

### `getAlleoEwidgetUtils(options?: AlleoEwidgetUtilsOptions): AlleoEwidgetUtils`

Returns a singleton instance of `AlleoEwidgetUtils`. If a browser global singleton already exists, the existing instance
is reused.

### `getEmbedWidgetMessenger(options?: EmbedWidgetMessengerOptions): EmbedWidgetMessenger`

Convenience helper equivalent to `getAlleoEwidgetUtils().createEmbedWidgetMessenger(options)`.

---

## `AlleoEwidgetUtils`

Top-level manager that owns the browser-global singleton and creates `EmbedWidgetMessenger` instances.

### Constructor

```ts
new AlleoEwidgetUtils(options ? : AlleoEwidgetUtilsOptions)
```

Creates an instance and calls `initialize()`. In the browser, a global singleton is stored so that multiple bundle loads
reuse the same instance.

### Options (`AlleoEwidgetUtilsOptions`)

| Option        | Type      | Default | Description                                                              |
|---------------|-----------|---------|--------------------------------------------------------------------------|
| `autoDestroy` | `boolean` | `true`  | Attaches a `beforeunload` listener that automatically calls `destroy()`. |
| `debug`       | `boolean` | `false` | Enables debug logging via `console.debug`.                               |

### Properties

| Property        | Type      | Description                                    |
|-----------------|-----------|------------------------------------------------|
| `isInitialized` | `boolean` | `true` when initialized and not yet destroyed. |

### Methods

| Method                                 | Returns                | Description                                                                                  |
|----------------------------------------|------------------------|----------------------------------------------------------------------------------------------|
| `static getInstance(options?)`         | `AlleoEwidgetUtils`    | Returns the singleton instance, creating one if needed.                                      |
| `createEmbedWidgetMessenger(options?)` | `EmbedWidgetMessenger` | Creates a new `EmbedWidgetMessenger` with the provided options.                              |
| `initialize()`                         | `void`                 | Idempotent; sets initialized state and attaches lifecycle listeners.                         |
| `destroy()`                            | `void`                 | Detaches listeners, clears the global singleton reference, and marks the instance destroyed. |
| `log(message, data?)`                  | `void`                 | Debug logger gated by the `debug` option.                                                    |

---

## `EmbedWidgetMessenger`

Handles communication between the iframe and the Alleo widget via `postMessage`.

### Constructor

```ts
new EmbedWidgetMessenger(options ? : EmbedWidgetMessengerOptions)
```

Initializes message handling and, by default, calls `startListening()` automatically.

### Options (`EmbedWidgetMessengerOptions`)

| Option         | Type      | Default         | Description                                                           |
|----------------|-----------|-----------------|-----------------------------------------------------------------------|
| `targetOrigin` | `string`  | `'*'`           | Passed to `postMessage` for all outbound commands.                    |
| `parentWindow` | `Window`  | `window.parent` | The window that receives outbound commands.                           |
| `autoListen`   | `boolean` | `true`          | Automatically registers the `message` event listener on construction. |
| `debug`        | `boolean` | `false`         | Enables debug logging via `console.debug`.                            |

### Properties

| Property      | Type      | Description                                                       |
|---------------|-----------|-------------------------------------------------------------------|
| `isListening` | `boolean` | Whether the instance is currently listening for `message` events. |

### Lifecycle Methods

| Method             | Description                                                                                            |
|--------------------|--------------------------------------------------------------------------------------------------------|
| `startListening()` | Adds a `message` event listener if running in a browser.                                               |
| `stopListening()`  | Removes the `message` event listener.                                                                  |
| `destroy()`        | Stops listening, clears all registered handlers, and rejects pending `getBoardObjectContent` promises. |

### Public Methods

| Method                                             | Returns             | Description                                                                                                                 |
|----------------------------------------------------|---------------------|-----------------------------------------------------------------------------------------------------------------------------|
| `triggerAction(actionId, data?)`                   | `void`              | Sends a `triggerAction` command to the widget.                                                                              |
| `addContent(params)`                               | `void`              | Sends an `addContent` command to add content to the board.                                                                  |
| `requestSyncedStatus()`                            | `void`              | Requests the current synced status from the widget.                                                                         |
| `setSyncedStatus(status)`                          | `void`              | Shallow-merges the given status into the shared synced status.                                                              |
| `onSyncedStatusUpdate(handler)`                    | `() => void`        | Registers a handler for synced-status updates. Returns unsubscribe.                                                         |
| `onIncomingAction(handler)`                        | `() => void`        | Registers a handler for incoming action triggers. Returns unsubscribe.                                                      |
| `getBoardObjectContent(index, options?)`           | `Promise<string[]>` | Reads text content from a whitelisted board object. Resolves with an array of strings or rejects on error / timeout (30 s). |
| `replaceBoardObjectContent(index, text, options?)` | `void`              | Replaces the text content of a whitelisted board object (fire-and-forget).                                                  |
| `appendBoardObjectContent(index, text, options?)`  | `void`              | Appends text to a whitelisted board object (fire-and-forget).                                                               |
| `onError(handler)`                                 | `() => void`        | Registers a handler for errors. Returns unsubscribe.                                                                        |
| `onMessage(handler)`                               | `() => void`        | Registers a low-level handler for all inbound messages. Returns unsubscribe.                                                |

### Properties

| Property | Type                    | Description                                  |
|----------|-------------------------|----------------------------------------------|
| `mic`    | [`AlleoMic`](#alleomic) | Lazily-created microphone bridge. See below. |

---

## `AlleoMic`

Microphone bridge accessible via `messenger.mic` (i.e. `alleo.mic`). The iframe sandbox forbids
`navigator.mediaDevices.getUserMedia()`, so this class delegates capture to the parent widget over `postMessage`, then
reassembles the encoded chunks back into a live `MediaStream` inside the iframe using `MediaSource` + a hidden `<audio>`
element + `HTMLMediaElement.captureStream()`.

The parent slices the live recording into short chunks; chunk duration is **clamped to a maximum of 200 ms** by the
library before being sent to the parent.

### Constructor

```ts
new AlleoMic(messenger
:
EmbedWidgetMessenger
)
```

Normally not called directly — use `messenger.mic` (or `alleo.mic`).

### Properties

| Property   | Type      | Description                                            |
|------------|-----------|--------------------------------------------------------|
| `isActive` | `boolean` | `true` between `micStarted` and `micStopped` messages. |

### Methods

| Method                                            | Returns      | Description                                                                                                                     |
|---------------------------------------------------|--------------|---------------------------------------------------------------------------------------------------------------------------------|
| `start(options?: MicStartOptions)`                | `void`       | Sends a `micStart` command. The `timeslice` value is clamped to `[20, 200]` ms (default `100`).                                 |
| `stop()`                                          | `void`       | Sends a `micStop` command and tears down the local pipeline.                                                                    |
| `onTrack(handler: (stream: MediaStream) => void)` | `() => void` | Registers a handler that receives a live `MediaStream` once per capture session. Fires immediately if a stream is already live. |
| `onStarted(handler)`                              | `() => void` | Registers a handler called when the parent confirms capture started.                                                            |
| `onStopped(handler)`                              | `() => void` | Registers a handler called when the parent confirms capture stopped.                                                            |
| `onError(handler)`                                | `() => void` | Registers a handler for mic-specific errors (permission denial, missing browser API, etc.).                                     |
| `destroy()`                                       | `void`       | Sends `micStop`, tears down the pipeline, unsubscribes from messages, clears all handlers.                                      |

### `MicStartOptions`

| Field                | Type     | Default          | Description                                                 |
|----------------------|----------|------------------|-------------------------------------------------------------|
| `timeslice`          | `number` | `100`            | Chunk duration in ms; clamped to `[20, 200]`.               |
| `mimeType`           | `string` | _parent default_ | Preferred container/codec, e.g. `'audio/webm;codecs=opus'`. |
| `sampleRate`         | `number` | _parent default_ | Preferred sample rate in Hz.                                |
| `audioBitsPerSecond` | `number` | _parent default_ | Preferred audio bitrate.                                    |

### Stream pipeline

The chunks received over `postMessage` are not exposed directly. Instead the library:

1. On the first `micChunk` of a session, creates a `MediaSource` and a hidden, muted `<audio>` element.
2. Feeds each subsequent chunk into a `SourceBuffer` (in `'sequence'` mode) for that `<audio>` element.
3. Calls `audio.captureStream()` (or `mozCaptureStream()` on Firefox) and emits the resulting `MediaStream` to all
   `onTrack` handlers.
4. On `micStopped` / `micError` / `destroy()`, calls `endOfStream()` (best effort), stops all tracks on the captured
   stream, and removes the hidden element.

If `MediaSource`, the negotiated mime type, or `captureStream()` is unavailable in the host browser, an `onError`
callback is fired with a descriptive message and no stream is emitted.

### Constants

| Name                       | Value | Description                               |
|----------------------------|-------|-------------------------------------------|
| `MIC_MAX_TIMESLICE_MS`     | `200` | Hard upper bound on chunk size.           |
| `MIC_DEFAULT_TIMESLICE_MS` | `100` | Default chunk size when none is provided. |

---

## Message Types

### Outbound Commands (iframe → widget)

Commands sent from the iframe to the Alleo widget.

`EmbedWidgetCommandEnvelope` is the outbound message envelope:

```ts
{
    type: 'EmbedWidgetCommand', command
:
    EmbedWidgetCommand, params
}
```

`EmbedWidgetCommandMap` defines all command names and their parameter shapes:

| Command                     | Params                                                                                              |
|-----------------------------|-----------------------------------------------------------------------------------------------------|
| `triggerAction`             | `{ actionId: string; data?: Record<string, unknown> }`                                              |
| `addContent`                | See [`AddContentParams`](#addcontentparams)                                                         |
| `requestSyncedStatus`       | _(no params)_                                                                                       |
| `setSyncedStatus`           | `{ status: Record<string, unknown> }`                                                               |
| `getBoardObjectContent`     | `{ index: number; format?: BoardObjectContentFormat; requestId: string }`                           |
| `replaceBoardObjectContent` | `{ index: number; text: string \| string[]; format?: BoardObjectContentFormat; requestId: string }` |
| `appendBoardObjectContent`  | `{ index: number; text: string \| string[]; format?: BoardObjectContentFormat; requestId: string }` |
| `micStart`                  | `{ timeslice: number; mimeType?: string; sampleRate?: number; audioBitsPerSecond?: number }`        |
| `micStop`                   | _(no params)_                                                                                       |

#### `AddContentParams`

A union of the following shapes (discriminated by `type`):

| `type`          | Additional fields                                                            |
|-----------------|------------------------------------------------------------------------------|
| `'html'`        | `html: string`                                                               |
| `'notepad'`     | `text?: string \| string[]`, `textFormat?: 'text' \| 'markdown' \| 'html'`   |
| `'sticky-note'` | `text?: string`, `color?: string`, `outlineColor?: string`, `shape?: string` |
| `'image'`       | `url: string`                                                                |
| `'video'`       | `fileId: string`                                                             |

### Inbound Messages (widget → iframe)

Messages received by the iframe from the Alleo widget.

`EmbedWidgetMessageEnvelope` is the inbound message envelope:

```ts
{
    type: 'EmbedWidgetMessage', command
:
    EmbedWidgetMessageCommand, params
}
```

`EmbedWidgetMessageEvent` is the normalized event passed to `onMessage` handlers:

```ts
{
    command, params, raw
:
    EmbedWidgetMessageEnvelope
}
```

`EmbedWidgetMessageMap` defines all inbound command names and their parameter shapes:

| Command                    | Params                                                                                              |
|----------------------------|-----------------------------------------------------------------------------------------------------|
| `syncedStatusUpdate`       | `{ syncedStatus: Record<string, unknown> }`                                                         |
| `incomingAction`           | `{ actionId: string; data: Record<string, unknown> }`                                               |
| `boardObjectContentResult` | `{ index: number; content: string[]; requestId: string \| undefined }`                              |
| `boardObjectContentError`  | `{ index: number; error: string; requestId: string \| undefined }`                                  |
| `micStarted`               | `{ mimeType: string; sampleRate?: number; timeslice: number }`                                      |
| `micStopped`               | _(no params)_                                                                                       |
| `micChunk`                 | `{ data: ArrayBuffer; mimeType: string; timestamp: number; sequence: number; sampleRate?: number }` |
| `micError`                 | `{ error: string }`                                                                                 |

#### `BoardObjectContentFormat`

```ts
type BoardObjectContentFormat = 'text' | 'html' | 'markdown'
```

#### `ErrorPayload`

The payload passed to `onError` handlers:

```ts
type ErrorPayload = {
    index: number
    error: string
    requestId?: string
}
```

---

## Notes

- The `targetOrigin` option in `EmbedWidgetMessenger` should generally remain as `'*'` to allow communication with the
  widget regardless of the host origin.
- `getBoardObjectContent` is the only async method. It uses an internal `requestId` to correlate the outgoing command
  with the widget's response, allowing multiple calls to be in flight concurrently. Each call creates a unique
  `requestId`; the promise is resolved or rejected when a matching `boardObjectContentResult` or
  `boardObjectContentError` message arrives. A 30-second timeout rejects the promise automatically if no response is
  received.
- `replaceBoardObjectContent` and `appendBoardObjectContent` are fire-and-forget. They also generate a `requestId` so
  that any resulting `boardObjectContentError` message can be correlated, but there is no success callback. Use
  `onError` to catch errors from these methods.
- Calling `destroy()` on `EmbedWidgetMessenger` immediately rejects all pending `getBoardObjectContent` promises with an
  `'EmbedWidgetMessenger destroyed'` error.
- The microphone bridge (`alleo.mic`) requires the parent widget to implement the `micStart` / `micStop` commands and
  emit `micStarted`, `micStopped`, `micChunk` and `micError` messages. The library clamps `timeslice` to a hard maximum
  of **200 ms** (`MIC_MAX_TIMESLICE_MS`) before forwarding the request — the parent should respect this and never
  produce larger chunks. The raw `micChunk` envelope is internal: `AlleoMic` reassembles the chunks via `MediaSource`
  and exposes the result as a `MediaStream` through `onTrack`.
