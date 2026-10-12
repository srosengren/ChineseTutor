# ChineseTutor

Reading aid for learners of Chinese (Simplified first). Paste or drop text and read it
segmented into words, with toggleable pinyin, word-boundary highlighting, and
tap-a-word dictionary entries (pinyin, definitions, word class, measure words).

Web app first, hosted on GitHub Pages; a React Native app comes later and shares the core.

## Status

Pre-implementation. No app code yet. Read `docs/architecture.md` for the planned shape,
`docs/adr/` for decisions already made, and `docs/specs/2026-10-12-v1-design.md` for the v1
design before proposing changes to any of them.

## Principles

- **Static-first.** Everything runs on the client; there is no backend. A feature that needs
  server-side compute or secrets needs an ADR first (ADR-0003).
- **Platform-agnostic core.** `packages/core` is pure TypeScript: no DOM, React, React Native
  or Node APIs. Data is passed in, never fetched by core (ADR-0002).
- **One lexicon.** The segmenter and dictionary lookup use the same word list, so every word
  the segmenter produces has an entry or an explicit fallback.
- **Build-time data.** Source dictionaries are processed into compact app data by
  `tools/dict-build` and committed to `packages/dict-data`; raw downloads are never committed.

## Repository layout (planned)

| Path | Purpose |
|---|---|
| `apps/web` | React + Vite web app, static build deployed to GitHub Pages |
| `apps/mobile` | Expo / React Native app (later) |
| `packages/core` | Segmentation, lookup, pinyin formatting. Shared by both apps |
| `packages/dict-data` | Built dictionary data (CC BY-SA 4.0), committed; regenerate with `npm run dict:build` |
| `tools/dict-build` | Node scripts that turn source dictionaries into app data |
| `docs/` | Architecture overview, ADRs, specs and plans (see `docs/CLAUDE.md`) |

npm workspaces (no other package manager, ADR-0002); Node version in `.nvmrc`.

## Working rules

- Decisions that are costly to reverse, span packages, or add an external service, dataset
  or licence get an ADR. Revise an ADR in place until code depends on it; after that,
  supersede it with a new one (details in `docs/CLAUDE.md`).
- Update `docs/architecture.md` in the same commit when the shape of the system changes.
- Any third-party data must be recorded in `docs/data-sources.md` with source, version,
  licence and required attribution. Code is Apache-2.0; never add NonCommercial-licensed
  data or GPL code (ADR-0005).
- `main` rejects force pushes and cannot be deleted.
