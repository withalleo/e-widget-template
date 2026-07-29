<!--
  AUDIENCE: Reference only — end-user help for the E-widget. This is
  advice for the human using Alleo, NOT authoring instructions. An agent does not
  need to read this to build a widget. For authoring rules see ../../AGENTS.md.
-->
## Context for helping users

If the user does not specify a plan to create a widget (or has questions about the service), use the guidance below.

This tool is for creating an **E-widget** for the Alleo platform, and you are here to help users with that.

Alleo is a professional infinite-canvas collaborative platform available at **withalleo.com**.

Users can:

- add an **E-widget** to their Alleo board
- use AI to create importable content for that widget 
 
This widget lets you **embed content inside your board** using a secure, built‑in browser frame (an iframe).

> This is a `beta` widget, which may change significantly. Use it with caution!

You can embed:

- **A website** (by URL)
- **Custom HTML** (a snippet of webpage code you paste in)
- **A file** you upload (for example, an `.html` file)

It's designed for common "show something live on the board" use cases, while still giving you control over **security**, **permissions**, and **user interaction**.

---

## Quick start

[Introduction to E-widgets](https://meet.withalleo.com/board/shbdwhhhwi7lg7rxks3dtmpf) - follow our interactive guide.

### Ways to generate the widget with AI

1. [Use the pre-configured chat](https://chatgpt.com/g/g-6a26cd8b99308191b372cf914dd2f157-alleo-e-widget-creator)

    This path is intended for beginners.

2. [Use the template repository](https://github.com/withalleo/e-widget-template)

    This path is intended for professional developers.

3. [Download AI instructions](https://widgets.withalleo.com/com.withalleo/embed-browser/documentation/AI-INSTRUCTIONS.md)

    This works with most AI agents.

After generating an e-widget with AI, you can use the import button to set it up.

> You can also customize your widget by editing the configuration on the top of your generated code.

### Embed custom HTML

1. **Settings → Source → Source = Custom HTML**.
2. Paste your markup into **Custom HTML** (full document or fragment).
3. Close settings.

If your markup uses JavaScript, enable **Allow scripts** under **Security**.

### Embed from a file

1. **Settings → Source → Source = File**.
2. Upload or select your file.

> If the picker rejects an `.html` file, rename it to `.txt` and upload the renamed copy.

### Embed a website (URL)

> Embedding a website can be tricky depending on your organization's security policies.

1. **Settings → Source → Source = URL**.
2. Paste an `https://` address into **URL**.
3. Close settings.

> `http://` is accepted but `https://` is strongly recommended. Non-HTTP schemes (e.g. `javascript:`) are rejected.

---

## Settings reference

### Source

| Setting         | Description                                                                                   |
| --------------- | --------------------------------------------------------------------------------------------- |
| **Source**      | Choose **Custom HTML**, **URL**, or **File**.                                                 |
| **Custom HTML** | Raw HTML rendered when Source = Custom HTML.                                                  |
| **URL**         | Address loaded when Source = URL. `https://` is strongly recommended (`http://` is accepted). |
| **File**        | Asset loaded when Source = File.                                                              |

### Display

| Setting                    | Description                                                                                                                                                                                                                                                                                        |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Unload when off-screen** | Removes embedded content while the widget is outside the viewport. Reduces resource usage; content reloads on return.                                                                                                                                                                              |
| **Enable UI selectors**    | Available once **Board features** are enabled. Shows a background/primary/text color picker and/or a font picker on the widget so board editors can adjust the theme without editing the HTML. The chosen values are pushed into the embedded content as CSS variables (see Colors & fonts below). |

### User interaction

| Setting                                            | Description                                                         |
| -------------------------------------------------- | ------------------------------------------------------------------- |
| **Disable all user interactions with the content** | Blocks clicks, typing, scrolling, and other input inside the embed. |
| **Disable scrolling the content**                  | Prevents scrolling only; other interactions remain enabled.         |

### Media permissions

| Setting                                    | Description                                                                                                                                                                                       |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Allow content access to the microphone** | Grants microphone permission to the embedded content. Custom HTML widgets reach the mic through the `alleo.mic` SDK bridge, which streams audio from the parent widget into the sandboxed iframe. |

The microphone option may be hidden when your organization enforces strict security. Browser-level permission prompts still apply. Camera access is **not** available inside the sandboxed iframe.

### Security

| Setting                                         | Description                                                                                         |
| ----------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| **Allow scripts**                               | Permits JavaScript execution inside the iframe. Required by most modern websites.                   |
| **Allow forms to be submitted**                 | Permits form submissions (logins, surveys, etc.).                                                   |
| **Allow the iframe to lock screen orientation** | Mobile-only. May be hidden in strict environments.                                                  |
| **Referrer policy**                             | Controls the `Referer` header sent when the embed loads external resources. Default: `no-referrer`. |

### External connections

| Setting                                                                                                    | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ---------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Allow use of protected backends**                                                                        | Lets the embedded content fetch anything from the internet via `alleo.fetchProtectedUrlJSON`/`fetchProtectedUrlText`/`fetchProtectedUrlBinary` - both public/unauthenticated fetches (`keyId: null`, routed through the CORS-bypass proxy) and authenticated calls to third-party APIs that require an **API key, token, or other credential**. This is the **only** supported way to reach the internet from inside the sandboxed iframe - the credential itself is never exposed to the iframe or embedded in the widget's HTML. |
| **Configure External Connection `<keyId>`** (context-menu action, appears once a connection is configured) | Opens a dialog to enter the real API key/credential for a given connection. Each connection also declares which exact endpoint URL(s) it may call - a call to any other URL is rejected.                                                                                                                                                                                                                                                                                                                                           |

---

## Hosting your content externally

When embedding a URL you control, the hosting server must return the correct HTTP headers or the browser will block the embed.

### Key headers

| Header                                     | Purpose                                                                                                                               |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------- |
| `Content-Security-Policy: frame-ancestors` | Declares which origins may embed the page. `frame-ancestors 'none'` or `X-Frame-Options: DENY/SAMEORIGIN` will produce a blank embed. |
| `Access-Control-Allow-Origin`              | Required for cross-origin `fetch`/`XMLHttpRequest` calls your page makes from inside the board.                                       |
| `Referrer-Policy`                          | Limits URL information leaked to third-party resources.                                                                               |
| `X-Content-Type-Options: nosniff`          | Prevents MIME-type sniffing. Recommended for all assets.                                                                              |
| `Permissions-Policy`                       | Explicitly denies or grants powerful features (camera, microphone, geolocation). Combine with the widget's permission toggles.        |

### Required origin

Use the Alleo deployment origin - `https://meet.withalleo.com` for the online service. On-prem deployments differ.

### Example response headers

```http
Content-Security-Policy: frame-ancestors https://meet.withalleo.com;
Access-Control-Allow-Origin: https://meet.withalleo.com
Referrer-Policy: strict-origin-when-cross-origin
X-Content-Type-Options: nosniff
Permissions-Policy: camera=(), microphone=()
```

- Replace the origin with your deployment if different.
- The widget never delegates camera or microphone permission to a URL embed via the iframe `allow` attribute. Camera is unavailable; microphone is only reachable in **Custom HTML** content through the `alleo.mic` SDK bridge (with the matching widget toggle enabled).
- `Access-Control-Allow-Origin` accepts a single origin or `*`. For multiple deployments, echo the request `Origin` after validating it against an allowlist.
- These headers must come from **your** server. If a third-party site blocks framing, the widget cannot override it.

---

## Board features & Alleo E-widget SDK

**Enable Alleo board features for enclosed content** (under **Board features**) to let your embedded page communicate with the board via the `alleo` object. Independently, **Share "\_\_embed\_\_" URL parameters with enclosed content** (under **Security**) exposes `alleo.getParams` even when Board features are off - either setting causes the SDK to be included.

### Including the SDK

**Custom HTML / File** - place the self-replacing tag in your markup:

```html
<EWidgetSDK />
```

**URL** - load from CDN and initialize manually:

```html
<script src="https://widgets.withalleo.com/com.withalleo/embed-browser/assets/widgetAssets/ewidget-utils.umd.js"></script>
<script>
    const alleo = AlleoEWidget.getEmbedWidgetMessenger()
</script>
```

### Feature matrix

| Feature                               | Required settings                                                                                                                          |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Trigger board actions (outgoing)      | Board features                                                                                                                             |
| Receive incoming actions              | Board features                                                                                                                             |
| Add content to the board              | Board features + **Enable adding new content**                                                                                             |
| Synced status                         | Board features + **Enable synchronizing status**                                                                                           |
| Read/write board object content       | Board features + **Enable reading/writing board object content** + populate **Whitelisted board objects**                                  |
| Read parent page URL parameters       | Board features + **Allow access to URL parameters**                                                                                        |
| Read the current user profile         | Board features + **Allow access to the current user profile**                                                                              |
| Call an authenticated third-party API | Board features + **Allow use of protected backends** + a configured endpoint allow-list and API key (see Calling authenticated APIs below) |

### Initializing your widget

Call `alleo.initialize()` once, right after the SDK loads. It replaces manually calling `alleo.requestSyncedStatus()` on load - it fetches the synced status **and** the widget's configured colors/font in one call:

```js
alleo.initialize()
```

### Colors & fonts

Each widget has a **background color**, a **text color**, a **primary color**, and a **font**, configurable via **Enable UI selectors** (see Display above) and the **Background/Primary/Text color** and **Font** pickers. After `alleo.initialize()` runs, these are applied as CSS variables on the embedded content's `<body>`:

- `--alleo-background-color`
- `--alleo-text-color`
- `--alleo-primary-color`
- `--alleo-font`

Custom HTML content should reference them with `var(--alleo-background-color)`, etc., and define fallback values at the top of its own stylesheet for when the widget runs standalone.

### Reading parent page URL parameters

Enable **Share "\_\_embed\_\_" URL parameters with enclosed content** (under **Security**) to expose selected query parameters of the page hosting Alleo to your embedded content - useful for passing a per-session ID or locale from the embedding site. Only parameters whose name starts with `__embed__` are shared, and the prefix is stripped from the keys.

```js
// Parent page URL: https://host.example.com/board?__embed__userId=42&theme=dark
alleo.initialize()
// Shortly after, once the parent has responded:
console.log(alleo.getParams) // { userId: '42' } - "theme" is not shared
```

The values are captured once and remain fixed for the widget's lifetime, even if the parent page's URL changes later.

### Reading the current user

Enable **Allow access to the current user profile** (under **Security**) to expose the profile of the logged-in board user to your embedded content - useful for personalizing the widget or pre-filling a name/email. The profile is read-only and captured once during `alleo.initialize()`.

```js
alleo.initialize()
// Shortly after, once the parent has responded:
console.log(alleo.user) // { firstName: 'Ada', lastName: 'Lovelace', email: 'ada@example.com', organization: { name: 'Alleo' }, ... }
```

The profile is an empty object until the parent responds, or when the feature is disabled.

### Outgoing triggers

Configure action IDs in **Settings → Actions → Enabled outgoing triggers**. Fire them from your page:

```js
alleo.triggerAction('my-action', { 'param-id': 'Hello from the board!' })
```

IDs are case-sensitive; only listed IDs are accepted.

### Incoming actions

Register a listener to receive actions triggered by other board objects:

```js
alleo.onIncomingAction(({ actionId, data }) => {
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

// Request current value on load - usually done via alleo.initialize() instead
alleo.requestSyncedStatus()

// Update (shallow-merged)
alleo.setSyncedStatus({ phase: 'ready' })
```

### Adding content

Supported types:

- `{ type: 'html', html: string }`
- `{ type: 'notepad', text?: string | string[], textFormat?: 'text' | 'markdown' | 'html' }`
- `{ type: 'sticky-note', text?: string, color?: string, outlineColor?: string, shape?: string }`
- `{ type: 'image', url: string }`
- `{ type: 'video', fileId: string }`

Use **Container for new content** to direct new objects into a specific group or frame, or leave it at **Do not add to container**.

### Fetching content from the internet

Custom HTML content **cannot** reliably fetch a third-party URL with a plain `fetch()` - it's usually blocked by CORS, and a request that requires an API key, bearer token, or other credential would expose that credential in the page source. Instead, enable **Allow use of protected backends** and call one of:

```js
const data = await alleo.fetchProtectedUrlJSON(keyId, url, init)
const text = await alleo.fetchProtectedUrlText(keyId, url, init)
const bytes = await alleo.fetchProtectedUrlBinary(keyId, url, init)
```

- Pass `keyId: null` for a **public, unauthenticated** URL - the request is routed through Alleo's CORS-bypass proxy. Prefer this over a plain `fetch()` for any public resource.
- Pass a `keyId` string to call an **authenticated** third-party API. The parent widget attaches the real credential to the request on your behalf; the iframe never sees it.
- Each authenticated connection also has an **allow-list of exact URLs** it may call - a request to any other URL is rejected, even if the iframe content asks for it.
- After importing a widget that declares a connection, open the widget's **context menu → "Configure External Connection `<keyId>`"** to enter the real API key/credential once.

This is the **only** supported way to reach the internet from inside the embed - never use a plain `fetch()`, and never a request that carries a secret.

### Reading & writing board object content

Whitelisted objects are referenced by **1-based index**.

```js
// Read
const lines = await alleo.getBoardObjectContent(1, { format: 'text' })

// Replace
alleo.replaceBoardObjectContent(1, '# Title\n\nBody.', { format: 'markdown' })

// Append
alleo.appendBoardObjectContent(1, '\n- Another item')
```

`replaceBoardObjectContent` and `appendBoardObjectContent` are fire-and-forget. Errors surface through `onError`.

### Error handling

```js
const unsubscribe = alleo.onError(({ index, error, requestId }) => {
    console.error(`Error on object #${index}:`, error)
})
```

Register early to catch issues from any board-object operation.

---

## Iframe concepts

| Term                | Meaning                                                                                                                                    |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| **iframe**          | An inline frame - a self-contained browser window rendered inside another page. The widget uses one to isolate embedded content.           |
| **Sandbox**         | Browser restrictions applied to the iframe (script execution, pop-ups, device access, navigation). Widget settings selectively relax them. |
| **Same-origin**     | A page's identity: scheme + domain + port (e.g. `https://example.com`). Embedded content is always cross-origin relative to the board.     |
| **Referrer policy** | Controls how much URL information the browser sends in the `Referer` header when the embed requests external resources.                    |

---

## Troubleshooting

| Symptom                             | Resolution                                                                                                                                                                                                                                                                   |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Embed is blank                      | Verify the URL starts with `https://`. Enable **Allow scripts** if the page requires JS. Check that **Unload when off-screen** isn't hiding content. The target site may block framing via `X-Frame-Options` or `frame-ancestors`                                            |
| Content reloads on scroll           | Expected when **Unload when off-screen** is enabled. Disable it to keep content persistent (higher resource usage).                                                                                                                                                          |
| Cannot click or scroll inside embed | Disable **Disable all user interactions with the content** and/or **Disable scrolling the content** under **Display**.                                                                                                                                                       |
| Microphone unavailable              | Enable **Allow content access to the microphone** under **Media permissions**, accept any browser prompts, and check whether your organisation's policy hides or locks the option. Sandbox restrictions may still apply. (Camera access is not available inside the iframe.) |
| HTML file won't upload              | Rename the file to `.txt` and upload the renamed copy.                                                                                                                                                                                                                       |

---

## FAQ

**Does the widget store my content?** Custom HTML and URLs are stored in the widget's settings. File content is loaded from the selected asset.

**Why do some websites refuse to load?** They send anti-framing headers (`X-Frame-Options`, `frame-ancestors`). If you don't control the server, the widget cannot override this.

**What is the safest configuration?** Use **URL** with trusted `https://` sites, leave **Allow scripts** off unless required, and avoid enabling extra permissions unnecessarily.

---

## Sample: SDK demo

Demonstrates outgoing triggers and synced status. Enable **Board features**, **synchronizing status**, and add an outgoing trigger with ID `demo` (with a Text parameter `demo-param`).

```html
<EWidgetSDK />
<button id="action-btn" style="border:none; padding: 16px 24px; font-size: 1rem; cursor: pointer;">START</button>

<script>
    const button = document.getElementById('action-btn')

    const randomColor = () =>
        '#' +
        Math.floor(Math.random() * 0xffffff)
            .toString(16)
            .padStart(6, '0')

    button.addEventListener('click', () => {
        alleo.triggerAction('demo', { 'demo-param': 'Hello World!' })
        alleo.setSyncedStatus({ color: randomColor() })
    })

    alleo.onSyncedStatusUpdate((status) => {
        if (status.color) button.style.backgroundColor = status.color
    })

    alleo.initialize()
</script>
```
