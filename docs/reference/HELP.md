<!--
  AUDIENCE: Reference only — end-user help for the E-widget. This is
  advice for the human using Alleo, NOT authoring instructions. An agent does not
  need to read this to build a widget. For authoring rules see ../../AGENTS.md.
-->

This widget lets you **embed content inside your board** using a secure, built‑in browser frame (an iframe).

> This is a `beta` widget, which may change significantly. Use it with caution!

You can embed:

- **A website** (by URL)
- **Custom HTML** (a snippet of webpage code you paste in)
- **A file** you upload (for example, an `.html` file)

It's designed for common "show something live on the board" use cases, while still giving you control over **security**,
**permissions**, and **user interaction**.

---

## Quick start

### Embed a website (URL)

1. **Settings → Source → Source = URL**.
2. Paste an `https://` address into **URL**.
3. Close settings.

> `http://` is accepted but `https://` is strongly recommended. Non-HTTP schemes (e.g. `javascript:`) are rejected.

### Embed custom HTML

1. **Settings → Source → Source = Custom HTML**.
2. Paste your markup into **Custom HTML** (full document or fragment).
3. Close settings.

If your markup uses JavaScript, enable **Allow scripts** under **Security**.

### Embed from a file

1. **Settings → Source → Source = File**.
2. Upload or select your file.

> If the picker rejects an `.html` file, rename it to `.txt` and upload the renamed copy.

---

## Settings reference

### Source

| Setting         | Description                                            |
|-----------------|--------------------------------------------------------|
| **Source**      | Choose **Custom HTML**, **URL**, or **File**.          |
| **Custom HTML** | Raw HTML rendered when Source = Custom HTML.           |
| **URL**         | Address loaded when Source = URL. Must use `https://`. |
| **File**        | Asset loaded when Source = File.                       |

### Display

| Setting                    | Description                                                                                                           |
|----------------------------|-----------------------------------------------------------------------------------------------------------------------|
| **Unload when off-screen** | Removes embedded content while the widget is outside the viewport. Reduces resource usage; content reloads on return. |
| **Overwrite text color**   | Forces all text inside the embed to the chosen **Text color**. Useful for matching board themes on HTML/SVG content.  |

### User interaction

| Setting                                            | Description                                                         |
|----------------------------------------------------|---------------------------------------------------------------------|
| **Disable all user interactions with the content** | Blocks clicks, typing, scrolling, and other input inside the embed. |
| **Disable scrolling the content**                  | Prevents scrolling only; other interactions remain enabled.         |

### Media permissions

| Setting                                    | Description                                                                                                                                                                                                |
|--------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Allow content access to the microphone** | Grants microphone permission to the embedded content. Custom HTML widgets can also reach the mic through the `alleo.mic` SDK bridge, which streams audio from the parent widget into the sandboxed iframe. |

> Camera access is **no longer available** to embedded content. This option may be
> hidden when your organisation enforces strict security. Browser-level permission
> prompts still apply.

### Security

| Setting                                         | Description                                                                                         |
|-------------------------------------------------|-----------------------------------------------------------------------------------------------------|
| **Allow scripts**                               | Permits JavaScript execution inside the iframe. Required by most modern websites.                   |
| **Allow forms to be submitted**                 | Permits form submissions (logins, surveys, etc.).                                                   |
| **Allow the iframe to lock screen orientation** | Mobile-only. May be hidden in strict environments.                                                  |
| **Referrer policy**                             | Controls the `Referer` header sent when the embed loads external resources. Default: `no-referrer`. |

---

## Hosting your content externally

When embedding a URL you control, the hosting server must return the correct HTTP headers or the browser will block the
embed.

### Key headers

| Header                                     | Purpose                                                                                                                               |
|--------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------|
| `Content-Security-Policy: frame-ancestors` | Declares which origins may embed the page. `frame-ancestors 'none'` or `X-Frame-Options: DENY/SAMEORIGIN` will produce a blank embed. |
| `Access-Control-Allow-Origin`              | Required for cross-origin `fetch`/`XMLHttpRequest` calls your page makes from inside the board.                                       |
| `Referrer-Policy`                          | Limits URL information leaked to third-party resources.                                                                               |
| `X-Content-Type-Options: nosniff`          | Prevents MIME-type sniffing. Recommended for all assets.                                                                              |
| `Permissions-Policy`                       | Explicitly denies or grants powerful features (camera, microphone, geolocation). Combine with the widget's permission toggles.        |

### Required origin

Use the Alleo deployment origin — `https://meet.withalleo.com` for the online service. On-prem deployments differ.

### Example response headers

```http
Content-Security-Policy: frame-ancestors https://meet.withalleo.com;
Access-Control-Allow-Origin: https://meet.withalleo.com
Referrer-Policy: strict-origin-when-cross-origin
X-Content-Type-Options: nosniff
Permissions-Policy: camera=(), microphone=()
```

- Replace the origin with your deployment if different.
- To grant microphone access, adjust `Permissions-Policy` (e.g. `microphone=(self)`) **and** enable the
  matching widget toggle. (Camera access is no longer available to embedded content.)
- `Access-Control-Allow-Origin` accepts a single origin or `*`. For multiple deployments, echo the request `Origin`
  after validating it against an allowlist.
- These headers must come from **your** server. If a third-party site blocks framing, the widget cannot override it.

---

## Board features & Alleo E-widget SDK

**Enable Alleo board features for enclosed content** (under **Board features**) to let your embedded page communicate
with the board via the `alleo` object.

### Including the SDK

**Custom HTML / File** — place the self-replacing tag in your markup:

```html

<EWidgetSDK/>
```

**URL** — load from CDN and initialise manually:

```html

<script src="https://unpkg.com/@withalleo/ewidget-utils/dist/ewidget-utils.umd.cjs"></script>
<script>
    const alleo = AlleoEWidget.getEmbedWidgetMessenger({debug: false})
</script>
```

### Feature matrix

| Feature                          | Required settings                                                                                         |
|----------------------------------|-----------------------------------------------------------------------------------------------------------|
| Trigger board actions (outgoing) | Board features                                                                                            |
| Receive incoming actions         | Board features                                                                                            |
| Add content to the board         | Board features + **Enable adding new content to the board**                                               |
| Synced status                    | Board features + **Enable synchronizing status**                                                          |
| Read/write board object content  | Board features + **Enable reading/writing board object content** + populate **Whitelisted board objects** |

### Outgoing actions

Configure action IDs in **Settings → Actions → Enabled outgoing actions**. Fire them from your page:

```js
alleo.triggerAction('my-action', {'param-id': 'Hello from the board!'})
```

IDs are case-sensitive; only listed IDs are accepted.

### Incoming actions

Register a listener to receive actions triggered by other board objects:

```js
alleo.onIncomingAction(({actionId, data}) => {
    console.log('incoming action', actionId, data)
})
```

### Synced status

A shared key/value map visible to all open instances of the widget.

```js
// Subscribe
alleo.onSyncedStatusUpdate((status) => {
    document.getElementById('display').textContent = status.phase || '-'
})

// Request current value on load
alleo.requestSyncedStatus()

// Update (shallow-merged)
alleo.setSyncedStatus({phase: 'ready'})
```

### Adding content

Supported types:

- `{ type: 'html', html: string }`
- `{ type: 'notepad', text?: string | string[], textFormat?: 'text' | 'markdown' | 'html' }`
- `{ type: 'sticky-note', text?: string, color?: string, outlineColor?: string, shape?: string }`
- `{ type: 'image', url: string }`
- `{ type: 'video', fileId: string }`

Use **Container for new content** to direct new objects into a specific group or frame, or leave it at **Do not add to
container**.

### Reading & writing board object content

Whitelisted objects are referenced by **1-based index**.

```js
// Read
const lines = await alleo.getBoardObjectContent(1, {format: 'text'})

// Replace
alleo.replaceBoardObjectContent(1, '# Title\n\nBody.', {format: 'markdown'})

// Append
alleo.appendBoardObjectContent(1, '\n- Another item')
```

`replaceBoardObjectContent` and `appendBoardObjectContent` are fire-and-forget. Errors surface through `onError`.

### Error handling

```js
const unsubscribe = alleo.onError(({index, error, requestId}) => {
    console.error(`Error on object #${index}:`, error)
})
```

Register early to catch issues from any board-object operation.

---

## Iframe concepts

| Term                | Meaning                                                                                                                                    |
|---------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| **iframe**          | An inline frame — a self-contained browser window rendered inside another page. The widget uses one to isolate embedded content.           |
| **Sandbox**         | Browser restrictions applied to the iframe (script execution, pop-ups, device access, navigation). Widget settings selectively relax them. |
| **Same-origin**     | A page's identity: scheme + domain + port (e.g. `https://example.com`). Embedded content is always cross-origin relative to the board.     |
| **Referrer policy** | Controls how much URL information the browser sends in the `Referer` header when the embed requests external resources.                    |

---

## Troubleshooting

| Symptom                             | Resolution                                                                                                                                                                                                                        |
|-------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Embed is blank                      | Verify the URL starts with `https://`. Enable **Allow scripts** if the page requires JS. Check that **Unload when off-screen** isn't hiding content. The target site may block framing via `X-Frame-Options` or `frame-ancestors` |
| Content reloads on scroll           | Expected when **Unload when off-screen** is enabled. Disable it to keep content persistent (higher resource usage).                                                                                                               |
| Cannot click or scroll inside embed | Disable **Disable all user interactions with the content** and/or **Disable scrolling the content** under **Display**.                                                                                                            |
| Microphone unavailable              | Enable **Allow content access to the microphone** under **Media permissions**, accept any browser prompts, and check whether your organisation's policy hides or locks the option. Sandbox restrictions may still apply. (Camera access is no longer available.)                   |
| HTML file won't upload              | Rename the file to `.txt` and upload the renamed copy.                                                                                                                                                                            |

---

## FAQ

**Does the widget store my content?**
Custom HTML and URLs are stored in the widget's settings. File content is loaded from the selected asset.

**Why do some websites refuse to load?**
They send anti-framing headers (`X-Frame-Options`, `frame-ancestors`). If you don't control the server, the widget
cannot override this.

**What is the safest configuration?**
Use **URL** with trusted `https://` sites, leave **Allow scripts** off unless required, and avoid enabling extra
permissions unnecessarily.

---

## Sample: SDK demo

Demonstrates outgoing actions and synced status. Enable **Board features**, **synchronizing status**, and add an
outgoing action with ID `demo` (with a Text parameter `demo-param`).

```html

<EWidgetSDK/>
<button id="action-btn" style="border:none; padding: 16px 24px; font-size: 1rem; cursor: pointer;">START</button>

<script>
    const button = document.getElementById('action-btn')

    const randomColor = () =>
            '#' +
            Math.floor(Math.random() * 0xffffff)
                    .toString(16)
                    .padStart(6, '0')

    button.addEventListener('click', () => {
        alleo.triggerAction('demo', {'demo-param': 'Hello World!'})
        alleo.setSyncedStatus({color: randomColor()})
    })

    alleo.onSyncedStatusUpdate((status) => {
        if (status.color) button.style.backgroundColor = status.color
    })

    alleo.requestSyncedStatus()
</script>
```
