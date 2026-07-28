# Alleo E-widget utility library

A JavaScript library that can be used within an Alleo e-widget to provide additional functionalities.

## Insert library

To use the library, you have to insert the `<EWidgetSDK />` tag in the beginning of your code (it doesn't have a closing tag). This defines the `alleo` class.

```html
<EWidgetSDK />
<button id="button">Add a notepad</button>
<script>
    document.getElementById('button').onclick = () =>
        alleo.addContent({ type: 'notepad', textFormat: 'markdown', text: '# Status\n\nAll systems go.' })
</script>
```

Alternatively when using an url (not recommended) instead of the `<EWidgetSDK />` tag, you have to link the library directly.

```html
<script src="https://widgets.withalleo.com/com.withalleo/embed-browser/assets/widgetAssets/ewidget-utils.umd.js"></script>
<script>
    const alleo = AlleoEWidget.getEmbedWidgetMessenger({ debug: false })
</script>
```

## Colors & fonts

Every e-widget is configured with a **background color**, a **text color**, a **primary color**, and a **font**. Once you call [`alleo.initialize()`](#initialize) (or [`alleo.requestColorUpdate()`](#requestcolorupdate)), the library applies them as CSS custom properties on `document.body`:

| CSS variable               | Purpose                                      |
| -------------------------- | -------------------------------------------- |
| `--alleo-background-color` | The widget's configured background color     |
| `--alleo-text-color`       | The widget's configured text color           |
| `--alleo-primary-color`    | The widget's configured primary/accent color |
| `--alleo-font`             | The widget's configured font stack           |

Define fallback values for all four at the top of your first `<style>` block, so the widget still looks right before Alleo applies the real values (or when running standalone, outside of Alleo). Alleo sets its values as **inline styles** on `document.body`, so they always take priority over your CSS fallback once `initialize()` / `requestColorUpdate()` responds - you don't need `!important` or any special selector.

```css
body {
    --alleo-background-color: #051825;
    --alleo-text-color: #f9fff6;
    --alleo-primary-color: #6da8ff;
    --alleo-font: 'PT Root UI', Roboto, 'Helvetica Neue', 'Ubuntu Sans', Helvetica, Verdana, sans-serif;
}
```

Reference them with `var(--alleo-background-color)`, `var(--alleo-text-color)`, `var(--alleo-primary-color)`, and `var(--alleo-font, sans-serif)`. If you need extra shades (hover states, borders, etc.), derive them from these three colors (e.g. with `color-mix()`) instead of hard-coding unrelated colors.

## Methods

### initialize

`initialize(options?: { timeoutMs?: number }): Promise<void>`

Call this once, right after the SDK loads, to set up your widget. It replaces manually calling `requestSyncedStatus()` on load: it requests the synced status, the forwarded URL parameters, the current user profile **and** the widget's configured colors/font (applying them as the CSS variables described above).

It returns a promise that resolves once every response has been received. Responses that never arrive - for example, when a feature is disabled in the widget settings - are bounded by `timeoutMs` (default `500` ms), so the promise **always resolves and never rejects**. You can safely `await` it before reading `getParams` or `user`.

**Once the promise resolves, `initialize()` automatically fires the `onloaded` action for you** (after the DOM has settled) - do not call `alleo.triggerAction('onloaded')` yourself.

```js
await alleo.initialize()
```

Equivalent to calling both of the following, but preferred for startup:

```js
alleo.requestSyncedStatus()
alleo.requestColorUpdate()
```

Call it once at startup - you don't need to fire `onloaded` manually, `initialize()` does it for you once it completes.

---

### getParams

`getParams: Record<string, string>` _(read-only property)_

The URL parameters forwarded from the page that hosts Alleo. When **Share "\_\_embed\_\_" URL parameters with enclosed content** is enabled in the widget's **Security** settings, every query parameter of the parent page whose name starts with `__embed__` is exposed here with the `__embed__` prefix stripped from the key.

The values are captured once, when you call [`alleo.initialize()`](#initialize) (which requests them from the widget), and stay stable for the lifetime of the embed. The property returns an empty object until the parent responds, or when the feature is disabled.

```js
await alleo.initialize()

// Parent page URL: https://host.example.com/board?__embed__userId=42&__embed__lang=en&theme=dark
console.log(alleo.getParams) // { userId: '42', lang: 'en' }  (note: "theme" is not shared)
```

> Because the values arrive asynchronously, `await alleo.initialize()` before reading `getParams`. If you don't await it, read `getParams` once the SDK has had a chance to receive the response (for example inside `alleo.onSyncedStatusUpdate(...)` or after a short delay), not synchronously on the same tick as `initialize()`.

---

### user

`user: EmbedUserProfile` _(read-only property)_

The profile of the currently logged-in board user, forwarded from the widget. When **Allow access to the current user profile** is enabled in the widget's **Security** settings, this exposes the user's identity:

```ts
interface EmbedUserProfile {
    firstName?: string
    lastName?: string
    displayName?: string
    email?: string
    imageUrl?: string
    organization?: { name?: string; id?: string }
    lastActivityAt?: Date
    lastLoginAt?: Date
    displayColor?: string
}
```

The value is captured once, when you call [`alleo.initialize()`](#initialize) (which requests it from the widget), and stays stable for the lifetime of the embed. The property returns an empty object until the parent responds, or when the feature is disabled.

```js
await alleo.initialize()
console.log(alleo.user) // { firstName: 'Ada', lastName: 'Lovelace', email: 'ada@example.com', organization: { name: 'Alleo' }, ... }
```

> Because the value arrives asynchronously, `await alleo.initialize()` before reading `user`. If you don't await it, read `user` once the SDK has had a chance to receive the response (for example inside `alleo.onSyncedStatusUpdate(...)` or after a short delay), not synchronously on the same tick as `initialize()`.

---

### triggerAction

`triggerAction(actionId: string, data?: Record<string, unknown>): void`

Triggers an object action on the Alleo board with optional parameters.

It is commonly used on user actions within your e-widget, since you can set up functionalities on your board that reacts to these.

```js
alleo.triggerAction('demo', {
    'demo-param': 'Hello from iframe',
})
```

<details>
<summary>Full sample</summary>

```html
<!doctype html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>Alleo E-widget - Basic actions</title>
        <style>
            body {
                font-family: Arial, sans-serif;
                padding: 16px;
            }

            button {
                padding: 8px 12px;
            }

            pre {
                background: #f4f4f4;
                padding: 8px;
            }
        </style>
    </head>
    <body>
        <EWidgetSDK />
        <h1>Alleo E-widget - Basic actions</h1>
        <p>This page is intended to run inside the E-widget iframe.</p>

        <button id="trigger">Trigger action</button>
        <button id="request">Request synced status</button>

        <h2>Log</h2>
        <pre id="log"></pre>

        <script>
            const logEl = document.getElementById('log')
            const log = (message, data) => {
                const line = data ? `${message} ${JSON.stringify(data)}` : message
                logEl.textContent = `${line}\n${logEl.textContent}`
            }

            alleo.onSyncedStatusUpdate((status) => {
                log('syncedStatusUpdate:', status)
            })

            alleo.onIncomingAction(({ actionId, data }) => {
                log(`incomingAction: ${actionId}`, data)
            })

            document.getElementById('trigger').addEventListener('click', () => {
                alleo.triggerAction('demo', {
                    'demo-param': 'Hello from iframe',
                })
                log('triggerAction sent')
            })

            document.getElementById('request').addEventListener('click', () => {
                alleo.requestSyncedStatus()
                log('requestSyncedStatus sent')
            })
        </script>
    </body>
</html>
```

</details>

---

### addContent

`addContent(params): void`

Adds content to your board.

```js
alleo.addContent({
    type: 'notepad',
    textFormat: 'markdown',
    text: '# Status\n\nAll systems go.',
})
```

#### Parameters

Union of the following shapes:

- `{ type: 'html', html: string }`
- `{ type: 'notepad', text?: string | string[], textFormat?: 'text' | 'markdown' | 'html' }`
- `{ type: 'sticky-note', text?: string, color?: string, outlineColor?: string, shape?: string }`
- `{ type: 'image', url: string }`
- `{ type: 'video', fileId: string }`

<details>
<summary>Full sample</summary>

```html
<!doctype html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>Alleo E-widget - Add content</title>
        <style>
            body {
                font-family: Arial, sans-serif;
                padding: 16px;
            }

            button {
                padding: 8px 12px;
                margin-right: 8px;
                margin-bottom: 8px;
            }
        </style>
    </head>
    <body>
        <EWidgetSDK />
        <h1>Alleo E-widget - Add content</h1>
        <p>These commands require "Enable adding new content to the board" in settings.</p>

        <button id="add-notepad">Add notepad</button>
        <button id="add-sticky">Add sticky note</button>
        <button id="add-html">Add HTML embed</button>

        <script>
            document.getElementById('add-notepad').addEventListener('click', () => {
                alleo.addContent({
                    type: 'notepad',
                    textFormat: 'markdown',
                    text: '# New note\n\nAdded from iframe.',
                })
            })

            document.getElementById('add-sticky').addEventListener('click', () => {
                alleo.addContent({
                    type: 'sticky-note',
                    text: 'Remember to review the draft',
                    color: '#FFE680',
                    outlineColor: '#B38F00',
                    shape: 'square',
                })
            })

            document.getElementById('add-html').addEventListener('click', () => {
                alleo.addContent({
                    type: 'html',
                    html: '<!doctype html><html><body><h2>Hello from iframe</h2></body></html>',
                })
            })
        </script>
    </body>
</html>
```

</details>

---

### requestSyncedStatus

`requestSyncedStatus(): void`

Requests the current synchronized status of the e-widget.

After calling, the onSyncedStatusUpdate callback will trigger.

```js
alleo.requestSyncedStatus()
```

> You normally don't need to call this directly on startup - [`alleo.initialize()`](#initialize) calls it for you. Call it directly only if you need to re-sync mid-session.

---

### requestColorUpdate

`requestColorUpdate(): void`

Requests the widget's currently configured colors and font. When the response arrives, the library automatically applies them as the `--alleo-background-color`, `--alleo-text-color`, `--alleo-primary-color`, and `--alleo-font` CSS variables on `document.body` - there's no callback to register, just read the CSS variables in your stylesheet (see [Colors & fonts](#colors--fonts)).

```js
alleo.requestColorUpdate()
```

> You normally don't need to call this directly on startup - [`alleo.initialize()`](#initialize) calls it for you.

---

### setSyncedStatus

`setSyncedStatus(status: Record<string, unknown>): void`

Sets the synchronized status of the widget.

The status is a record (an object). You can set these values individually.

If the value exists, it will be overwritten.

```js
alleo.setSyncedStatus({
    color: '#FFAA33',
})
```

<details>
<summary>Full sample</summary>

```html
<!doctype html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>Alleo E-widget - Synced status</title>
        <style>
            body {
                font-family: Arial, sans-serif;
                padding: 16px;
            }

            button {
                padding: 8px 12px;
                border: none;
                color: #111;
            }
        </style>
    </head>
    <body>
        <EWidgetSDK />
        <h1>Alleo E-widget - Synced status</h1>
        <p>Click the button to set a random color in synced status.</p>

        <button id="color">Set random color</button>

        <script>
            const button = document.getElementById('color')

            const applyColor = (status) => {
                if (!status || !status.color) return
                button.style.backgroundColor = status.color
            }

            alleo.onSyncedStatusUpdate((status) => {
                applyColor(status)
            })

            button.addEventListener('click', () => {
                const color = `#${Math.floor(Math.random() * 0xffffff)
                    .toString(16)
                    .padStart(6, '0')}`
                alleo.setSyncedStatus({ color })
            })

            alleo.initialize()
        </script>
    </body>
</html>
```

</details>

---

### onSyncedStatusUpdate

`onSyncedStatusUpdate(handler: (status: Record<string, unknown>) => void): () => void`

Triggers a callback when the synchronized status changes. (ie. `setSyncedStatus` is called on any instance on the widget)

The returned status is always the full status, even if `setSyncedStatus` is called with a partial status.

```js
alleo.onSyncedStatusUpdate((status) => {
    console.log('synced status', status)
})
```

---

### onIncomingAction

`onIncomingAction(handler: (payload: { actionId: string; data: Record<string, unknown> }) => void): () => void`

Triggers a callback when any of the widget's incoming action triggers is triggered.

Note: `data` is optional and might be empty.

```js
alleo.onIncomingAction(({ actionId, data }) => {
    console.log('incoming action', actionId, data)
})
```

---

### getBoardObjectContent

`getBoardObjectContent(index: number, options?: { format?: 'text' | 'html' | 'markdown' }): Promise<string[]>`

Retrieves the text content of a whitelisted board object. The `index` is 1-based and refers to the position of the object in the widget's **Board object whitelist** setting.

Returns a `Promise` that resolves with an array of content strings. The promise rejects if the widget reports an error or if the request times out (30 s).

```js
const content = await alleo.getBoardObjectContent(1, { format: 'text' })
console.log(content.join('\n'))
```

#### Parameters

| Parameter        | Type                             | Default  | Description                                            |
| ---------------- | -------------------------------- | -------- | ------------------------------------------------------ |
| `index`          | `number`                         | -        | 1-based position of the board object in the whitelist. |
| `options.format` | `'text' \| 'html' \| 'markdown'` | `'text'` | The format to retrieve content in.                     |

<details>
<summary>Full sample</summary>

```html
<!doctype html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>Alleo E-widget - Board object content</title>
        <style>
            body {
                font-family: Arial, sans-serif;
                padding: 16px;
            }

            button {
                padding: 8px 12px;
                margin-right: 8px;
                margin-bottom: 8px;
            }

            textarea {
                width: 100%;
                min-height: 80px;
                font-family: monospace;
            }

            select,
            input[type='number'] {
                padding: 4px 8px;
            }

            pre {
                background: #f4f4f4;
                padding: 8px;
                max-height: 200px;
                overflow: auto;
            }

            .controls {
                margin-bottom: 16px;
            }

            .controls label {
                display: inline-block;
                margin-right: 12px;
            }
        </style>
    </head>
    <body>
        <EWidgetSDK />
        <h1>Alleo E-widget - Board object content</h1>
        <p>
            These commands require <strong>Enable reading/writing board object content</strong>
            and at least one whitelisted board object in settings.
        </p>

        <div class="controls">
            <label>
                Index:
                <input type="number" id="index" value="1" min="1" style="width:60px" />
            </label>
            <label>
                Format:
                <select id="format">
                    <option value="text">text</option>
                    <option value="markdown">markdown</option>
                    <option value="html">html</option>
                </select>
            </label>
        </div>

        <button id="get">Get content</button>
        <button id="replace">Replace content</button>
        <button id="append">Append content</button>

        <h2>Text</h2>
        <textarea id="text" placeholder="Enter text to replace or append…"></textarea>

        <h2>Log</h2>
        <pre id="log"></pre>

        <script>
            const logEl = document.getElementById('log')
            const log = (message, data) => {
                const line = data !== undefined ? `${message} ${JSON.stringify(data)}` : message
                logEl.textContent = `${line}\n${logEl.textContent}`
            }

            const getIndex = () => Number(document.getElementById('index').value) || 1
            const getFormat = () => document.getElementById('format').value
            const getText = () => document.getElementById('text').value

            document.getElementById('get').addEventListener('click', async () => {
                const index = getIndex()
                const format = getFormat()
                log(`getBoardObjectContent(${index}, { format: '${format}' }) …`)
                try {
                    const content = await alleo.getBoardObjectContent(index, { format })
                    log('Content received:', content)
                    document.getElementById('text').value = content.join('\n')
                } catch (err) {
                    log('Error: ' + err.message)
                }
            })

            document.getElementById('replace').addEventListener('click', () => {
                const index = getIndex()
                const format = getFormat()
                const text = getText()
                alleo.replaceBoardObjectContent(index, text, { format })
                log(`replaceBoardObjectContent(${index}) sent`)
            })

            document.getElementById('append').addEventListener('click', () => {
                const index = getIndex()
                const format = getFormat()
                const text = getText()
                alleo.appendBoardObjectContent(index, text, { format })
                log(`appendBoardObjectContent(${index}) sent`)
            })

            const unsubscribe = alleo.onError(({ index, error, requestId }) => {
                log(`boardObjectContentError #${index} (requestId=${requestId}):`, error)
            })
        </script>
    </body>
</html>
```

</details>

---

### replaceBoardObjectContent

`replaceBoardObjectContent(index: number, text: string | string[], options?: { format?: 'text' | 'html' | 'markdown' }): void`

Replaces the text content of a whitelisted board object. This is a fire-and-forget operation - there is no success response. Errors are reported asynchronously via `onError`.

```js
alleo.replaceBoardObjectContent(1, '# New content', { format: 'markdown' })
```

#### Parameters

| Parameter        | Type                             | Default  | Description                                                |
| ---------------- | -------------------------------- | -------- | ---------------------------------------------------------- |
| `index`          | `number`                         | -        | 1-based position of the board object in the whitelist.     |
| `text`           | `string \| string[]`             | -        | The replacement text. Accepts a single string or an array. |
| `options.format` | `'text' \| 'html' \| 'markdown'` | `'text'` | The format of the provided text.                           |

---

### appendBoardObjectContent

`appendBoardObjectContent(index: number, text: string | string[], options?: { format?: 'text' | 'html' | 'markdown' }): void`

Appends text to the existing content of a whitelisted board object. Identical to `replaceBoardObjectContent` except that the new text is appended instead of replacing the existing content.

```js
alleo.appendBoardObjectContent(1, '\nExtra line')
```

#### Parameters

| Parameter        | Type                             | Default  | Description                                              |
| ---------------- | -------------------------------- | -------- | -------------------------------------------------------- |
| `index`          | `number`                         | -        | 1-based position of the board object in the whitelist.   |
| `text`           | `string \| string[]`             | -        | The text to append. Accepts a single string or an array. |
| `options.format` | `'text' \| 'html' \| 'markdown'` | `'text'` | The format of the provided text.                         |

---

### onError

`onError(handler: (payload: { index: number; error: string; requestId?: string }) => void): () => void`

Registers a callback that fires whenever an error occurs. Errors may originate from `getBoardObjectContent`, `replaceBoardObjectContent`, or `appendBoardObjectContent`.

For `getBoardObjectContent` errors, the corresponding promise is rejected **first**, then all registered error handlers are called.

Returns an unsubscribe function.

```js
const unsubscribe = alleo.onError(({ index, error, requestId }) => {
    console.error(`Object #${index} error (requestId=${requestId}):`, error)
})

// Later: stop listening
unsubscribe()
```

---

### `fetchProtectedUrlJSON` / `fetchProtectedUrlText` / `fetchProtectedUrlBinary`

```ts
fetchProtectedUrlJSON(keyId: string | null, input: RequestInfo | URL, init?: RequestInit): Promise<unknown>
fetchProtectedUrlText(keyId: string | null, input: RequestInfo | URL, init?: RequestInit): Promise<string>
fetchProtectedUrlBinary(keyId: string | null, input: RequestInfo | URL, init?: RequestInit): Promise<ArrayBuffer>
```

**Always prefer these over a plain `fetch()` whenever the widget needs to download anything from the internet.** The iframe is served from a sandboxed origin, so a plain `fetch()` to a third-party URL is usually blocked by CORS. `keyId` controls the mode:

- **`keyId: null`** - a **public, unauthenticated** fetch, routed through the Alleo CORS-bypass proxy. Use this for any request that doesn't carry a secret, token, password, or API key.
- **`keyId: '<string>'`** - **the only supported way for embedded content to call a third-party API that requires authentication** (an API key, bearer token, or any other credential). The iframe never sees, holds, or sends the credential itself - the parent widget attaches it server-side and enforces which URLs may be called.

Never hard-code an API key/secret in the HTML/JS you generate, and never attempt to pass a secret through the `keyId: null` (public) path or a plain `fetch()`; those only work for public, unauthenticated resources.

```js
// Public, unauthenticated fetch through the CORS-bypass proxy
const data = await alleo.fetchProtectedUrlJSON(null, 'https://api.example.com/data?id=42')
```

```js
// Authenticated fetch using a configured backend connection
const data = await alleo.fetchProtectedUrlJSON('weatherApi', 'https://api.example.com/v1/forecast?city=Berlin')
```

- `keyId` is `null` for a public/unauthenticated fetch through the CORS-bypass proxy, or a string identifying **which** configured backend connection to use for an authenticated call. A non-null `keyId` must match one of the keys of the `allowedProtectedBackends` object in the widget's [import footer](./IMPORT-FOOTER.md#protected-backend-connections) (or configured directly in the widget's settings).
- `input` must be an `https://` URL. For a non-null `keyId` it is checked against that connection's configured `allowedUrls` - a request to any other URL is rejected. There is **no way around this allow-list** from inside the iframe.
- `init` is forwarded like a normal `fetch()` (`method`, `headers`, `body`, etc.) - omit any `Authorization`/API-key header yourself for authenticated calls; the parent widget injects the real credential according to the connection's configured `method`/`template`.
- Requires **Enable Alleo board features for enclosed content** + **Allow use of protected backends** in the widget's settings for both the `keyId: null` and authenticated cases. An authenticated call additionally needs at least one entry in **Allowed protected backends** matching `keyId`, and a board owner must enter the actual credential once via the widget's "Configure External Connection" context-menu action - the footer/settings only declare **which** endpoints are allowed, never the secret value itself.

```js
// POST with a JSON body - the auth header/query param is added automatically by the parent widget
const text = await alleo.fetchProtectedUrlText('crmApi', 'https://api.example.com/v1/contacts', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ name: 'Ada Lovelace' }),
})
```

`fetchProtectedUrlBinary` resolves with the raw `ArrayBuffer` - use it for images, audio, or other non-text payloads; `fetchProtectedUrlText` decodes the body as UTF-8 text; `fetchProtectedUrlJSON` additionally parses it with `JSON.parse`. All three reject if the response status is outside the `2xx` range, if the request times out (60 s), or if a non-null `keyId` has no matching, correctly-configured backend.

---

## Microphone (`alleo.mic`)

The iframe runs in a sandboxed origin and cannot call `navigator.mediaDevices.getUserMedia()` itself. The `alleo.mic` namespace delegates capture to the parent Alleo widget over `postMessage`. The parent records the microphone, slices the stream into short audio chunks (**maximum 200 ms per chunk**) and forwards them back to the iframe, where the library reassembles them into a live `MediaStream` you can use just like a normal `getUserMedia` result.

Requires **Allow content access to the microphone** in the widget settings. The first call to `alleo.mic.start()` triggers a browser permission prompt on the parent page.

### `mic.start`

`mic.start(options?: { timeslice?: number; mimeType?: string; sampleRate?: number; audioBitsPerSecond?: number }): void`

Asks the parent to start capturing audio. The resulting `MediaStream` is delivered to handlers registered via `mic.onTrack`. Errors (including permission denial) are reported via `mic.onError`.

| Option               | Type     | Default | Description                                                  |
| -------------------- | -------- | ------- | ------------------------------------------------------------ |
| `timeslice`          | `number` | `100`   | Chunk duration in **milliseconds**, clamped to `[20, 200]`.  |
| `mimeType`           | `string` | _auto_  | Preferred container/codec (e.g. `'audio/webm;codecs=opus'`). |
| `sampleRate`         | `number` | _auto_  | Preferred sample rate in Hz (e.g. `48000`, `16000`).         |
| `audioBitsPerSecond` | `number` | _auto_  | Preferred audio bitrate.                                     |

```js
alleo.mic.start({ timeslice: 100 })
```

### `mic.stop`

`mic.stop(): void`

Asks the parent to stop capturing. The live `MediaStream` is ended and its tracks are stopped.

### `mic.onTrack`

`mic.onTrack(handler: (stream: MediaStream) => void): () => void`

Registers a callback that receives a live `MediaStream` whenever the microphone starts capturing. The callback fires **once per capture session**, as soon as the first audio chunk arrives. Use it just like a stream from `getUserMedia`:

```js
alleo.mic.onTrack((stream) => {
    const audio = new Audio()
    audio.srcObject = stream
    audio.play()

    console.log('Live mic stream received')
})

alleo.mic.start()
```

You can also pipe the stream into the Web Audio API, `RTCPeerConnection.addTrack()`, `MediaRecorder`, etc.

If a stream is already live when the handler is registered, it is invoked immediately with the existing stream. Returns an unsubscribe function.

> **Note:** internally the library reassembles the parent's encoded chunks via `MediaSource` + a hidden `<audio>` element and exposes `HTMLMediaElement.captureStream()`. This means the browser must support both APIs (Chrome, Edge, Firefox, Opera all do).

### `mic.onStarted` / `mic.onStopped` / `mic.onError`

```js
alleo.mic.onStarted(({ mimeType, sampleRate, timeslice }) => {
    console.log('capturing', mimeType, sampleRate, timeslice)
})

alleo.mic.onStopped(() => {
    console.log('mic stopped')
})

alleo.mic.onError(({ error }) => {
    console.error('mic error', error)
})
```

Each returns an unsubscribe function.

<details>
<summary>Full sample</summary>

```html
<!doctype html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <title>Alleo E-widget - Microphone</title>
    </head>
    <body>
        <EWidgetSDK />
        <button id="start">Start mic</button>
        <button id="stop">Stop mic</button>
        <audio id="monitor" controls></audio>

        <script>
            const monitor = document.getElementById('monitor')

            alleo.mic.onError(({ error }) => console.error(error))
            alleo.mic.onTrack((stream) => {
                monitor.srcObject = stream
                monitor.play()
                console.log('Live mic stream received')
            })

            document.getElementById('start').onclick = () => alleo.mic.start({ timeslice: 100 })
            document.getElementById('stop').onclick = () => alleo.mic.stop()
        </script>
    </body>
</html>
```

</details>

---

## Limitations

Due to security limitations (sandboxed iframe), the e-widget code cannot use some html/javascript features.

This includes:

- **`localStorage` / `sessionStorage`** - Web Storage APIs are blocked; use `setSyncedStatus` / `onSyncedStatusUpdate` to persist and share state instead.
- **`document.cookie`** - Cookie access is blocked.
- **`IndexedDB`** - The IndexedDB API is unavailable.
- **`window.open()`** - Opening new windows or tabs is blocked.
- **`alert()`, `confirm()`, `prompt()`** - Modal dialogs are blocked.
- **Top-level navigation** - The iframe cannot navigate the parent page (e.g. `window.top.location = ...`).
- **Geolocation** - `navigator.geolocation` is unavailable.
- **Camera / Microphone** - `navigator.mediaDevices.getUserMedia()` is blocked. Microphone access is available indirectly through the [`alleo.mic`](#microphone-alleomic) bridge.
- **Screen capture** - `navigator.mediaDevices.getDisplayMedia()` is blocked.
- **Notifications** - The `Notification` API is blocked.
- **Clipboard write** - `navigator.clipboard.writeText()` and similar write operations are blocked.
- **Fullscreen** - `element.requestFullscreen()` is blocked.
- **Pointer Lock** - `element.requestPointerLock()` is blocked.
- **Service Workers** - Registering a service worker (`navigator.serviceWorker.register()`) is blocked.
- **Payment Request API** - `new PaymentRequest(...)` is blocked.
- **Web Bluetooth / Web USB** - `navigator.bluetooth` and `navigator.usb` are unavailable.
- **WebAuthn / Credential Management** - `navigator.credentials` is blocked.
- **Device sensors** - Accelerometer, gyroscope, ambient light sensor, etc. are unavailable.
- **Shared origin** - The iframe runs in a unique opaque origin, so it cannot access resources from the parent page via `document.domain` or same-origin APIs.
