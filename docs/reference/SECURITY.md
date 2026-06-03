<!--
  AUDIENCE: Reference only — describes the E-widget's sandbox tiers and
  security/admin options. This is advice for the human/admin configuring Alleo,
  NOT authoring instructions. An agent does not need to read this to build a
  widget (the sandbox limits that matter are summarized in ../../AGENTS.md).
-->

# E-widget Security

The E-widget embeds external content inside a sandboxed `<iframe>`. This document describes every security-relevant
option, the three-tier enforcement model, and who can configure each setting.

---

## Configuration levels

Configuration is coming from the following sources:

| Source                    | Scope              | How it is set                                                                                                                |
|---------------------------|--------------------|------------------------------------------------------------------------------------------------------------------------------|
| **Manifest defaults**     | Widget (ship-time) | `config` block in `manifest.json` — defines the out-of-the-box baseline                                                      |
| **Organization settings** | Organization       | defaults to manifest file, but can be overriden by org admins and applied to every widget instance within their organization |

> The `RequireStrictSecurity` and `DisableHighlyUnsecureOptions` flags ALWAYS have priority over any other setting, and
> are enforced if set in **either** the manifest or in the organization settings.

---

## Security tiers

### Standard

Available to all widget editors. Can be disabled by org or deployment administrators.

### Restricted

Available by default, but disabled if the `RequireStrictSecurity` flag is set.

This flag can be set at organization level (`RequireStrictSecurity`) or deployment level (`RequireStrictSecurity`);
either being `true` is sufficient to enforce strict mode.

### Highly Unsecure

Disabled by default and not available unless every one of the following conditions is satisfied simultaneously:

1. `AllowHighlyUnsecureOptions: true` is set at **organization or deployment level**.
2. Neither `RequireStrictSecurity` nor `DisableHighlyUnsecureOptions` is `true` at organization **or** deployment level.
3. The current content source is in the org-level `HighlyUnsecureWhitelist`:
    - Source **HTML** → the literal string `"HTML"` must be present in the whitelist.
    - Source **File** → the literal string `"FILE"` must be present.
    - Source **URL** → the exact URL or a path-scoped wildcard (e.g. `https://example.com/app/*`) must match. The URL
      must use `https://`. Domain-only wildcards (e.g. `https://example.com/*`) are **not** accepted — the wildcard
      prefix must include at least one path segment.

> The widget manifest ships with `"DisableHighlyUnsecureOptions": true`, so highly unsecure options are **disabled by
default** for all deployments that do not explicitly override this.

### Microphone bridge note

Because the iframe is sandboxed, embedded scripts cannot call `navigator.mediaDevices.getUserMedia()` directly, and the
widget never grants a native `allow: microphone` permission to the iframe. The only way for embedded content to capture
audio is the `alleo.mic` bridge, which delegates capture to the parent widget and streams encoded audio chunks back over
`postMessage`. The bridge is gated by `iframeAllowMicrophone` and `RequireStrictSecurity`

---

## Iframe options reference

### Sandbox & permission options

| Key for default                     | iframe effect                                      | Security tier   | Default | Disabled / overridden by                                                         |
|-------------------------------------|----------------------------------------------------|-----------------|---------|----------------------------------------------------------------------------------|
| `DefaultIframeAllowScripts`         | `sandbox: allow-scripts`                           | Standard        | `true`  | `DisableScripts: true` (org / deployment)                                        |
| `DefaultIframeAllowForms`           | `sandbox: allow-forms`                             | Standard        | `true`  | `DisableForms: true` (org / deployment)                                          |
| `DefaultIframeAllowOrientationLock` | `sandbox: allow-orientation-lock`                  | Restricted      | `false` | Disabled when `RequireStrictSecurity`                                            |
| `DefaultIframeAllowMicrophone`      | Enables the `alleo.mic` parent-side capture bridge | Restricted      | `false` | Disabled when `RequireStrictSecurity`; also requires `enableIframeCommunication` |
| `DefaultIframeAllowSameOrigin`      | `sandbox: allow-same-origin`                       | Highly Unsecure | `false` | Disabled unless whitelist matches                                                |
| `DefaultIframeAllowModals`          | `sandbox: allow-modals`                            | Highly Unsecure | `false` | Disabled unless whitelist matches                                                |
| `DefaultIframeAllowTopNavigation`   | `sandbox: allow-top-navigation-by-user-activation` | Highly Unsecure | `false` | Disabled unless whitelist matches                                                |
| `DefaultIframeAllowFullscreen`      | `allowfullscreen` attribute                        | Highly Unsecure | `false` | Disabled unless whitelist matches                                                |
| `DefaultIframeAllowPopups`          | `sandbox: allow-popups`                            | Highly Unsecure | `false` | Disabled unless whitelist matches                                                |
| `DefaultIframeAllowPresentation`    | `sandbox: allow-presentation`                      | Highly Unsecure | `false` | Disabled unless whitelist matches                                                |
| `DefaultIframeAllowDownloads`       | `sandbox: allow-downloads`                         | Highly Unsecure | `false` | Disabled unless whitelist matches                                                |
| `DefaultIframeAllowPointerLock`     | `sandbox: allow-pointer-lock`                      | Highly Unsecure | `false` | Disabled unless whitelist matches                                                |

### Content policy

| Manifest config key           | Shared variable        | Effect                              | Security tier | Default       | Notes                                                          |
|-------------------------------|------------------------|-------------------------------------|---------------|---------------|----------------------------------------------------------------|
| `DefaultIframeReferrerPolicy` | `iframeReferrerPolicy` | Sets the `referrerpolicy` attribute | Restricted    | `no-referrer` | Forced to `no-referrer` when `RequireStrictSecurity` is active |

### Display & interaction options

These options affect behaviour and UX only; they do not grant additional browser permissions to embedded content.

| Manifest config key                | Shared variable             | Effect                                                                          | Default |
|------------------------------------|-----------------------------|---------------------------------------------------------------------------------|---------|
| `DefaultIframeUnloadWhenOffScreen` | `iframeUnloadWhenOffScreen` | Removes the iframe from the DOM when it is scrolled off-screen                  | `true`  |
| `DefaultIframeDisableUserActions`  | `iframeDisableUserActions`  | Sets `pointer-events: none` on the iframe, blocking all mouse/touch interaction | `false` |
| `DefaultIframeDisableScrolling`    | `iframeDisableScrolling`    | Intercepts scroll events inside the iframe content, preventing scrolling        | `false` |

---

## Referrer policy values

Only the values exposed by the widget settings dialog are listed here. The remaining standard policies are intentionally
omitted from the UI.

| Value           | Behaviour                                                                              |
|-----------------|----------------------------------------------------------------------------------------|
| `no-referrer`   | No `Referer` header is sent _(default; forced when `RequireStrictSecurity` is on)_     |
| `origin`        | Only the origin (scheme + host + port) is sent                                         |
| `strict-origin` | Origin sent only when the protocol security level stays the same; nothing on downgrade |

---

## Organization-level configuration reference

Set via `WidgetSettings.settings` by org admins. These apply to every E-widget instance in the organization.

| Key                            | Type     | Manifest default | Effect                                                                                                                                                  |
|--------------------------------|----------|------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| `RequireStrictSecurity`        | boolean  | `false`          | Forces strict mode: resets all Restricted and Highly Unsecure options to `false` and forces `referrerpolicy` to `no-referrer`                           |
| `AllowHighlyUnsecureOptions`   | boolean  | `false`          | Master switch required to unlock any Highly Unsecure option; has no effect without a matching whitelist entry                                           |
| `HighlyUnsecureWhitelist`      | string[] | `[]`             | Source allowlist for Highly Unsecure options; accepts `"HTML"`, `"FILE"`, exact HTTPS URLs, or path-scoped wildcards (e.g. `https://example.com/app/*`) |
| `DisableHighlyUnsecureOptions` | boolean  | `true`           | When `true`, overrides `AllowHighlyUnsecureOptions` and blocks all Highly Unsecure options regardless of other settings                                 |
| `DisableScripts`               | boolean  | `false`          | When `true`, prevents the `allow-scripts` sandbox token from being added, regardless of the widget-level setting                                        |
| `DisableForms`                 | boolean  | `false`          | When `true`, prevents the `allow-forms` sandbox token from being added, regardless of the widget-level setting                                          |

---

## Deployment-level configuration overrides

Set via the manifest file by deployment administrators. These (if set) always override organization settings.

| Key                            | Type    | Manifest default | Effect                                                                            |
|--------------------------------|---------|------------------|-----------------------------------------------------------------------------------|
| `RequireStrictSecurity`        | boolean | `false`          | Same effect as the org-level key; either source being `true` enforces strict mode |
| `DisableHighlyUnsecureOptions` | boolean | `true`           | When `true`, blocks all Highly Unsecure options regardless of org settings        |

> The manifest ships `DisableHighlyUnsecureOptions: true` as the baseline. A deployment (ie. on-prem) admin must
> explicitly set it to `false` to allow orgs to unlock the Highly Unsecure tier.

---

## Security recommendations

- Keep `DisableHighlyUnsecureOptions: true` in the deployment config. Do not disable it ever.
- Enabling `RequireStrictSecurity` might be a good idea for organizations with elevated security or compliance
  considerations.
- Keep the `HighlyUnsecureWhitelist` as narrow as possible. Use path-scoped wildcards rather than domain-root wildcards,
  and only whitelist sources you control.
- Never enable `allow-same-origin`
- The default `no-referrer` policy is the safest choice; only relax it if needed.
