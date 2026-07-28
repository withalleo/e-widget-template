# Alleo E-widget Template

A GitHub template for building **[Alleo](https://www.withalleo.com) E-widgets**
with AI coding agents (Codex, Claude Code, GitHub Copilot, and others).

An *E-widget* is a custom widget you embed on an Alleo board: a single,
self-contained HTML document that runs in a sandboxed iframe and talks to the
board through the `alleo` SDK.

## How it works

1. **Describe your widget** in [`idea.txt`](./idea.txt) in plain language. Not
   sure how to phrase it? Copy from the worked example in
   [`idea.sample.txt`](./idea.sample.txt).
2. **Ask your AI agent to build it.** The agent reads `idea.txt`, follows the
   rules in [`AGENTS.md`](./AGENTS.md), and generates the widget.
3. **Collect the output** from [`dist/`](./dist):
    - `dist/‹widget-name›.Alleo-eWidget.txt` — the importable widget (full HTML
      plus a `WIDGETSETTINGS:` footer, saved as `.txt`).
    - `dist/‹widget-name›.README.md` — a build note with assumptions, the
      widget's actions, and the board settings you need to enable.
4. **Add it to a board:** open the **E-widget** → **Settings → Source**, then
   paste the HTML as **Custom HTML** or upload the `.Alleo-eWidget.txt` file as
   **File** (importing the file also applies the footer settings automatically).

> The widget is saved with a `.txt` extension because the Alleo E-widget's
> file picker expects `.txt` for uploaded HTML. The recommended suffix is
> `.Alleo-eWidget.txt`.

## Quick start

```text
1. Use this template to create your own repo (green "Use this template" button).
2. Edit idea.txt with your widget idea (see idea.sample.txt for an example).
3. Run your agent of choice in the repo:
     • Codex / generic agents → read AGENTS.md
     • Claude Code            → read CLAUDE.md
     • GitHub Copilot         → reads .github/copilot-instructions.md
4. Find the generated widget in dist/.
```

## Repository layout

| Path                              | Purpose                                                                      |
|-----------------------------------|------------------------------------------------------------------------------|
| `idea.txt`                        | **You edit this.** Describes the widget to build.                            |
| `idea.sample.txt`                 | A filled-in example you can copy into `idea.txt` to get started.             |
| `AGENTS.md`                       | The single source of truth: the task + all coding rules for agents.          |
| `CLAUDE.md`                       | Pointer to `AGENTS.md` for Claude Code.                                      |
| `.github/copilot-instructions.md` | Pointer to `AGENTS.md` for GitHub Copilot.                                   |
| `dist/`                           | Generated widgets land here (`.Alleo-eWidget.txt`) with a build note.         |
| `docs/`                           | Authoring docs the agent reads while coding (SDK manual, types, footer).      |
| `docs/reference/`                 | User-facing reference about the E-widget (help, security, events, settings). |
| `samples/`                        | Importable example widgets (`.Alleo-eWidget.txt`) for each SDK capability.    |

## Documentation

See [`docs/README.md`](./docs/README.md) for the full index. In short:

**Authoring docs** (read while building a widget):

- [`docs/LIBRARY.md`](./docs/LIBRARY.md) — SDK methods, examples, and sandbox limitations.
- [`docs/IMPORT-FOOTER.md`](./docs/IMPORT-FOOTER.md) — the importable `WIDGETSETTINGS:` footer format and allowed keys.
- [`docs/AI-INSTRUCTIONS.md`](./docs/AI-INSTRUCTIONS.md) — the full upstream authoring guide.

**Reference docs** (user-facing advice about the E-widget):

- [`docs/reference/HELP.md`](./docs/reference/HELP.md) — end-user help for the E-widget.
- [`docs/reference/SECURITY.md`](./docs/reference/SECURITY.md) — sandbox tiers and security options.
- [`docs/reference/TRACKED-EVENTS.md`](./docs/reference/TRACKED-EVENTS.md) — analytics events emitted by the E-widget.

> The Alleo E-widget is currently a **beta** feature and may change.
