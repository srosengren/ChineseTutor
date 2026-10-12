# 0004. Defer full-text English translation

- Status: Accepted
- Date: 2026-10-12

## Context

Translating whole sentences or paragraphs is the only planned feature that needs a machine
translation model. The options assessed were:

- Apple's Translation framework: on-device and free, but only for a native app.
- Chrome's built-in Translator API: on-device and free, but only in desktop Chrome.
- A translation model running in the browser (e.g. Transformers.js with `opus-mt-zh-en`):
  a download of roughly 100 MB, with mediocre quality.
- A serverless proxy to a translation or LLM API: needs a backend and a secret key, which
  conflicts with ADR-0003.

## Decision

We will leave full-text translation out of the MVP. Per-word definitions from the
dictionary cover the beginner reading use case.

## Alternatives considered

- **Ship a serverless proxy now.** Adds a backend, a secret, and running cost before the
  core reading experience is proven.
- **Chrome Translator API only.** Would leave most browsers and all mobile users without
  the feature, which is confusing for a toggleable helper.

## Consequences

- The MVP stays fully static.
- When revisited, the likely shape is per platform: Apple Translation on native, and the
  Chrome API, an opt-in proxy, or a bring-your-own-key option on web. That requires a new ADR.
