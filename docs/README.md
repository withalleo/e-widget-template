<!--
  AUDIENCE: Index for both humans and AI agents. Start at ../AGENTS.md for the
  task and rules; come here to find the right detailed doc.
-->

# Documentation index

The single source of truth for the task and coding rules is
[`../AGENTS.md`](../AGENTS.md). These docs are the supporting detail.

They are split by audience so an agent knows what to read while building a widget
and what is background advice for the human user.

## Authoring docs — read these when writing widget code

| Doc                                          | Read it when you need to…                                                                         |
|----------------------------------------------|---------------------------------------------------------------------------------------------------|
| [`LIBRARY.md`](./LIBRARY.md)                 | Call the `alleo.*` SDK (methods, examples) and check sandbox limitations. **Primary SDK manual.** |
| [`ADVANCED.md`](./ADVANCED.md)               | Get exact type signatures or the iframe↔widget `postMessage` protocol.                            |
| [`AI-INSTRUCTIONS.md`](./AI-INSTRUCTIONS.md) | See the full original upstream guide and worked example. `../AGENTS.md` adapts and overrides it.  |

## Reference docs — advice for the user, not authoring instructions

Files in [`reference/`](./reference) describe the **E-widget** — the host that runs your HTML content. An agent does *
*not** need to read them to build a widget;
they help the human configure Alleo and understand its behavior.

| Doc                                                            | What it covers                                               |
|----------------------------------------------------------------|--------------------------------------------------------------|
| [`reference/HELP.md`](./reference/HELP.md)                     | End-user help: embedding content, settings, troubleshooting. |
| [`reference/SECURITY.md`](./reference/SECURITY.md)             | Sandbox tiers and security/admin options.                    |
| [`reference/TRACKED-EVENTS.md`](./reference/TRACKED-EVENTS.md) | Analytics events the E-widget emits.                         |

## Runnable examples

See [`../samples/`](../samples) for minimal, working widgets covering each SDK
capability (basic actions, synced status, add-content, board-object content, mic).
