<!--
  AUDIENCE: Reference only. These files describe the E-widget (the host that runs your HTML content). They are advice for the human using/administering Alleo, NOT
  authoring instructions for an agent. To build a widget, follow ../../AGENTS.md
  and use ../LIBRARY.md / ../ADVANCED.md.
-->

# E-widget - _Alleo widget_

(beta) Create a custom widget using common HTML code. Display your content with complete flexibility, create
interactive, dynamic experiences tailored exactly to your needs.

## Documentation

> [Help for users](./HELP.md) - opens using the Help button

- [AI-INSTRUCTIONS](../AI-INSTRUCTIONS.md)
- [IMPORT-FOOTER](../IMPORT-FOOTER.md)
- [LIBRARY](../LIBRARY.md)
- [SECURITY](./SECURITY.md)
- [TRACKED-EVENTS](./TRACKED-EVENTS.md)

## Information

| []()         | []()                                                                                    |
|--------------|-----------------------------------------------------------------------------------------|
| Icon         | ![Icon](https://widgets.withalleo.com/com.withalleo/embed-browser/assets/icon-beta.png) |
| Version      | `1.0.79`                                                                                |
| Widget id    | `embed-browser`                                                                         |
| Author       | [Alleo](https://www.withalleo.com)                                                      |
| Publisher id | `com.withalleo`                                                                         |
| Default size | 1280 x 720                                                                              |

## Settings

### `defaultHTMLContent`

Default not set in configuration file

### `IframeCommandRateLimitWindowMs`

Default in configuration: `10000`

### `DefaultSourceType`

Default not set in configuration file

### `DefaultUrl`

Default not set in configuration file

### `DefaultHTML`

Default not set in configuration file

### `DefaultFileId`

Default not set in configuration file

### `DefaultIframeAllowScripts`

Default in configuration: `true`

### `DefaultIframeAlleoScripts`

Default not set in configuration file

### `DefaultIframeAllowForms`

Default in configuration: `true`

### `DefaultIframeAllowOrientationLock`

Default in configuration: `false`

### `DefaultIframeAllowPointerLock`

Default in configuration: `false`

### `DefaultIframeDisableUserActions`

Default in configuration: `false`

### `DefaultIframeDisableScrolling`

Default in configuration: `false`

### `DefaultIframeReferrerPolicy`

Default in configuration: `no-referrer`

### `DefaultIframeUnloadWhenOffScreen`

Default in configuration: `true`

### `DefaultEnableIframeCommunication`

Default in configuration: `false`

### `DefaultEnableGetParams`

Default in configuration: `false`

### `DefaultEnableGetUser`

Default in configuration: `false`

### `DefaultEnableIframeFuncAddContent`

Default not set in configuration file

### `DefaultNewContentContainer`

Default not set in configuration file

### `DefaultEnableSyncedStatus`

Default in configuration: `false`

### `DefaultEnableIframeFuncBoardObjectContent`

Default not set in configuration file

### `DefaultBoardObjectWhitelist`

Default not set in configuration file

### `DefaultIncomingActions`

Default not set in configuration file

### `DefaultOutgoingActions`

Default not set in configuration file

### `DefaultBackgroundColor`

Default in configuration: `transparent`

### `DefaultTextColor`

Default in configuration: `null`

### `DefaultPrimaryColor`

Default in configuration: `null`

### `DefaultFont`

Default in configuration: `null`

### `DefaultEnableBackendProxy`

Default in configuration: `false`

### `DefaultAllowedProtectedBackends`

Default in configuration:

```json
{}
```

### `DefaultVersion`

Default not set in configuration file

### `DefaultHelpContent`

Default not set in configuration file

### `DefaultIframeAllowSameOrigin`

Default in configuration: `false`

### `DefaultIframeAllowModals`

Default in configuration: `false`

### `DefaultIframeAllowTopNavigation`

Default in configuration: `false`

### `DefaultIframeAllowFullscreen`

Default in configuration: `false`

### `DefaultIframeAllowPopups`

Default in configuration: `false`

### `DefaultIframeAllowPresentation`

Default in configuration: `false`

### `DefaultIframeAllowDownloads`

Default in configuration: `false`

### `DefaultIframeAllowMicrophone`

Default in configuration: `false`

### `Mode`

Default not set in configuration file

### `AllowHighlyUnsecureOptions`

Default in configuration: `false`

### `RequireStrictSecurity`

Default in configuration: `false`

### `DisableHighlyUnsecureOptions`

Default in configuration: `true`

### `AllEWidgetsDisableHighlyUnsecureOptions`

Default not set in configuration file

### `HighlyUnsecureWhitelist`

Default in configuration:

```json
[]
```

### `AllEWidgetsRequireStrictSecurity`

Default not set in configuration file

### `EWidgetLibUrl`

Default not set in configuration file

### `ImportFile`

Default not set in configuration file

### `DisableScripts`

Default in configuration: `false`

### `DisableForms`

Default in configuration: `false`

### `DefaultIframeCommandRateLimits`

Default in configuration:

```json
{
  "triggerAction": 50,
  "addContent": 20,
  "requestSyncedStatus": 20,
  "setSyncedStatus": 50,
  "getBoardObjectContent": 50,
  "replaceBoardObjectContent": 20,
  "appendBoardObjectContent": 20,
  "micStart": 20,
  "micStop": 20
}
```

### `DefaultActionTriggers`

Default in configuration:

```json
[]
```

### `ExternalBackendConnections`

Default in configuration:

```json
{
  "CorsProxy": {
    "DisplayName": "Alleo CORS Proxy",
    "PrivacyPolicyUrl": "https://www.withalleo.com/privacy-policy",
    "EnableUI": false,
    "DefaultScope": "deployment",
    "UseOrganizationKeyOverDeploymentKey": true,
    "EnableChangingScope": true,
    "DeploymentSharedSecretId": "CorsProxy",
    "OrganizationSharedSecretId": "CorsProxy",
    "AllowDeploymentSharedSecret": true,
    "AllowOrganizationSharedSecret": true,
    "AllowCustomDeploymentSharedSecret": false,
    "AllowCustomOrganizationSharedSecret": false,
    "AllowCustomApiConnection": false
  }
}
```

## Additional permissions

### Viewer

- `iframeContentSyncedStatus`

## Scopes for shared settings

- `settings-dialog`
- `color-picker`
- `integrated-documentation`
- `default-colors`
- `widget-assets`
- `default-network`

## Changelog

### 1.0.2

- embed browser: add basic functionality

### 1.0.17

- add widgetId and publisherId to all widget manifest.json files
- add settings-dialog to settingsScopes for widgets having a setting dialog
- add settingsScopes to the widget manifests, to allow using settings based on scopes defined in the admin.

### 1.0.24

- embed browser: review

### 1.0.28

- embed browser: add additional settings around security and media support
- embed browser: add ability for iframe content to trigger action triggers
- embed browser: add support for transparent background
- embed browser: adjust verbiage and default values
- embed browser: add icon
- embed browser: add additional check to enforce iframe containment by default
- embed browser: adjust default settings
- embed browser: fix feature that disables scrolling
- embed browser: add option to add content to the board
- embed browser: fix types
- embed browser: change the default so it fades in
- embed browser: support uploaded files as a source (not just html or url)
- embed browser: change default size to 16:9
- embed browser: review settings dialog
- embed browser: add help
- embed browser: add ability to add sticky notes / notepads / images / videos from the embed
- embed browser: add integrated-documentation flag

### 1.0.30

- embed browser: extend documentation
- embed browser: adjust verbiage
- embed browser: change the default size
- embed browser: add support for SVG files, and add option to overwrite primary color
- embed browser: fix scrolling issues
- embed browser: add support for synchronized status for the iframe content
- embed browser: adjust defaults
- embed browser: fix synchronized content
- embed browser: add support for action flows with parameters
- embed browser: limit and review features due to security concerns
- embed browser: update api lib

### 1.0.32

- embed browser: update library include
- embed browser: update default
- embed browser: add DisableHighlyUnsecureOptions flag to disable unsecure options.
- embed browser: make updates with a single call to prevent rate limiting
- embed browser: improve documentation
- embed browser: adjust settings

### 1.0.38

- embed browser: improve documentation
- update scoped settings
- embed browser: add support editing board objects
- embed browser: update AI instructions
- embed browser: add support for mics

### 1.0.40

- embed browser: add support for mics
- embed browser: improve sandbox security
- embed browser: adjust menu UI
- embed browser: better logging

### 1.0.41

- embed browser: better AI documentation
- embed browser: review referrers
- embed browser: do less unneeded sync updates

### 1.0.42

- embed browser: add rate limiting
- embed browser: add better inline comments
- embed browser: make sure that the listeners for iframe messaging are properly destroyed

### 1.0.43

- embed browser: fix the name of the DefaultIframeAllowScripts flag
- embed browser: update documentation
- embed browser: improve logging

### 1.0.47

- embed browser: improve logging
- embed browser: properly block the iframe content from user interactions

### 1.0.49

- embed browser: add description
- embed browser: add "beta" tag to description
- embed browser: add "beta" tag to icon
- embed browser: update documentation to add "beta" warning
- embed browser: add options to import and export widget settings
- embed browser: update documentation
- embed browser: fix settings dialog bug with hiding the actions tab
- embed browser: adjust settings dialog

### 1.0.50

- embed browser: fix issues with import

### 1.0.51

- embed browser: fix bugs with mic access

### 1.0.52

- embed browser: add link to custom gpt assistant

### 1.0.53

- embed browser: adjust verbiage
- embed browser: adjust AI instructions

### 1.0.54

- embed browser: adjust help
- embed browser: adjust instructions to avoid keyboard controls
- embed browser: adjust instructions to respond with the complete modified file if the user asks for changes
- embed browser: fix hidden list that is not working properly.
- embed browser: update action flow related verbiage to correct terminology

### 1.0.56

- embed browser: update action trigger definition to prevent double-generating each
- embed browser: update help
- embed browser: allow direct mic access when unsecure on on-prem
- embed browser: do not allow the html select element
- embed browser: move the iframe library to a local install

### 1.0.58

- embed browser: add option to package e-widgets as widgets

### 1.0.59

- embed browser: add security flags that can be applied to ALL e-widgets

### 1.0.61

- review widget tags and categories
- embed browser: remove generated lib files from repo

### 1.0.62

- embed browser: add support for using alleo colors and fonts

### 1.0.63

- embed browser: add support for a default size, that is used in export and import
- embed browser: add version to export, show version in the settings dialog.

### 1.0.65

- embed browser: add support for CORS bypass proxy
- embed browser: update instructions to produce simpler code
- embed browser: add support for color and font pickers

### 1.0.66

- embed browser: add support to pass get params from the board

### 1.0.67

- embed browser: make alleo.initialize () async and automatically handle the onloaded trigger.
- embed browser: clarify API library code structure and description

### 1.0.68

- embed browser: adjust font sizes
- embed browser: fix font picker

### 1.0.69

- embed browser: update library names for clarity and consistency
- embed browser: reset widget settings to default when importing
- embed browser: add support for backend proxy

### 1.0.70

- embed browser: adjust UX process in code generation and import

### 1.0.71

- embed browser: improve key setup UX
- embed-browser: Remove local links from help.md #XXXXX
- Revert "embed-browser: Remove local links from help.md #XXXXX"
- embed browser: remove internal links from help

### 1.0.72

- embed browser: fix checkbox to disable the zoom button

### 1.0.73

- embed browser: adjust AI instructions, have a separate file to clarify user communication.
- embed browser: adjust AI instructions to have separate widget name, description and help sections.
- embed browser: add support for a custom name and help for the widget

### 1.0.74

- embed browser: add a permission dialog when importing

### 1.0.75

- embed browser: add a permission dialog when importing
- embed browser: remove duplicate action triggers

### 1.0.76

- embed browser: clarify `iframeDisableUserActions` documentation for display-only widgets
- embed browser: add some widget settings to the permission dialog when importing

### 1.0.77

- embed browser: adjust AI instructions
- embed browser: update verbiage
- embed browser: update mic use to better support mobile devices

### 1.0.78

- embed browser: review what counts as on-prem
- embed browser: update instructions to not allow the cors proxy on on-prem
- embed browser: use the backend proxy for authenticating with the cors proxy
- embed browser: less individual network writes during import
- embed browser: add tutorial content link to opening dialog, and help.

### 1.0.79

- embed browser: add information about the current user
