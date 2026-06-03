# GitHub Copilot instructions

This repository is a template for building **Alleo E-widgets** with AI agents.

**Read [`AGENTS.md`](../AGENTS.md) in the repository root — it is the single
source of truth for the task and all coding rules.** Follow it exactly.

## Task summary

1. Read [`idea.txt`](../idea.txt) — it describes the widget to build.
2. Generate one self-contained Alleo E-widget (a single HTML document) following
   every rule in `AGENTS.md`.
3. Write the result to `dist/<widget-name>.txt` (full HTML content, `.txt`
   extension — the Alleo file picker expects `.txt`).
4. Write a build note to `dist/<widget-name>.README.md`.

If `idea.txt` is empty, stop and ask the user to fill it in.

## Key rules (see `AGENTS.md` for the full list)

- One self-contained HTML document. All CSS in `<style>`, all JS in `<script>`.
- Plain ES2020+ only — no TypeScript/JSX/build step.
- Organize logic into classes with a documented configuration block at the top.
- Use the `alleo` SDK; add `<EWidgetSDK />` once in `<body>` before any script.
- Fire `alleo.triggerAction('onloaded')` exactly once after full init.
- Fire outgoing actions on every meaningful interaction; handle incoming actions.
- The iframe is sandboxed — no `localStorage`, cookies, `window.open`, `alert`,
  `getUserMedia`, etc. Use the SDK equivalents.
- Authoring docs (read while coding): `docs/LIBRARY.md` (SDK manual),
  `docs/ADVANCED.md` (types/protocol). Index: `docs/README.md`.
- `docs/reference/` is user-facing advice about the E-widget (help, security,
  tracked events, settings) — not authoring instructions.
- Runnable examples are in `samples/`.
