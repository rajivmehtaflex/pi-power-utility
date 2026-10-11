# rename-proc

A Pi extension that sets the current session's display name from the conversation in the format `word_word_word`.

## Behavior

- Names the session when it starts or reloads, then refreshes the title after each agent run.
- Uses the active model to choose a short, descriptive title from recent user/assistant conversation and session summaries.
- If model naming is unavailable or fails, derives a three-word title from conversation keywords.
- Uses a random three-word title only when the session has no usable conversation context.
- Registers `/rename-proc` to request a rename manually.

The extension sends a compacted excerpt of recent conversation text to the currently selected model for title generation. This is an additional model request after an agent run and may incur provider usage. Tool output is not included in the excerpt.

## Install

From a clone of this repository, install only this package:

```sh
pi install ./packages/rename-proc
```

Or install the monorepo root to load all resources declared by its manifest:

```sh
pi install git:github.com/rajivmehtaflex/pi-power-utility@main
```

To test the entry point without changing package settings:

```sh
pi --extension ./packages/rename-proc/extensions/rename-proc.ts
```

## Requirements

- Pi with the Extension API methods `setSessionName`, `ModelRegistry.streamSimple`, and session lifecycle events.
- A configured model for model-generated titles. If none is available, contextual keyword naming remains available; an empty session receives a random title.
