# CLAUDE.md

This repository is a template for building **Alleo E-widgets** with AI agents.

**👉 Read [`AGENTS.md`](./AGENTS.md) — it is the single source of truth for your
task and all coding rules.**

## TL;DR

1. Read [`idea.txt`](./idea.txt) — it describes the widget to build.
2. Generate one self-contained Alleo E-widget (a single HTML document) that
   fulfils the idea, following every rule in `AGENTS.md`.
3. Write the result to `dist/<widget-name>.txt` — the file content is full HTML,
   but the extension is `.txt` (the Alleo file picker expects `.txt`).
4. Write a build note to `dist/<widget-name>.README.md` summarising what you
   built, your assumptions, the actions, and the widget settings the user must
   enable.

If `idea.txt` is empty, stop and ask the user to fill it in.

Reference docs are indexed in [`docs/README.md`](./docs/README.md): authoring
docs (SDK manual, types) live in [`docs/`](./docs), and user-facing advice about
the E-widget lives in [`docs/reference/`](./docs/reference). Runnable examples
live in [`samples/`](./samples).
