# Architecture overview

## What the app does

MVP:

- Paste or drop Chinese text (Simplified first).
- Render it segmented into words.
- Toggleable helpers: pinyin under each word; word-boundary highlighting.
- Pinyin under the text shows spoken tones for 一 and 不 (一个 *yí gè*, 不是 *bú shì*);
  the entry popover shows dictionary tones. When the rule is unclear (ordinals like 第一,
  dates, counting) the dictionary tone is kept. Third-tone changes are not marked.
- Tap a word to see its entry: pinyin, definitions, word class, measure words.

Not in v1:

- Full-text English translation (ADR-0004).
- Manual correction of word splits (merge or split). The reading view is read-only; the
  dictionary-based segmenter is right often enough for v1.
- Guessing unknown words (jieba-style). Names that CC-CEDICT lacks split into single
  characters; well-known names (about 19k people, places and organisations) have entries.
- Explaining why a tone changed. Planned: `core` already reports, for each syllable, the
  dictionary tone, the spoken tone, and which rule changed it, so this is a UI-only addition.

## Shape

Everything runs on the client (ADR-0003). Work splits into a build-time stage and a runtime
stage.

```
BUILD TIME (tools/dict-build, run by hand; output committed to packages/dict-data)

  source dictionaries ──► normalise & merge ──► compact app data ──► static assets
  (downloaded, never                            (lexicon with
   committed)                                    frequencies, entries)

RUNTIME (packages/core, shared by web and native)

  text ──► normalise ──► segment ──► tokens ──► annotate ──► UI (apps/web, apps/mobile)
                            ▲                     ▲
                            └── lexicon ──────────┴── entries (pinyin, senses, word class)
```

The segmenter and the lookup share one lexicon, so every token resolves to an entry or to an
explicit fallback (for example, per-character entries for an unknown word).

## Packages

| Package | Responsibility | May depend on |
|---|---|---|
| `packages/core` | Segmentation, lookup, pinyin formatting | Nothing platform-specific |
| `packages/dict-data` | Built dictionary data (CC BY-SA 4.0), committed | Nothing |
| `apps/web` | Web UI, data loading and caching | `core`, React, browser APIs |
| `apps/mobile` | Native UI (later) | `core`, React Native, Expo |
| `tools/dict-build` | Build app data from source dictionaries | Node |

See ADR-0002 for why the split is drawn here.

## Data loading

`core` never fetches data. Each app loads the data files and hands them to `core`:

- Web: fetched from GitHub Pages; the word list at startup, entry files on demand. Browser
  caching avoids repeat downloads (offline use is not in v1).
- Native: bundled with the app.

Dictionary sources and the entry shape are set in ADR-0006. Word class is stored per word
and, optionally, per sense; v1 fills mostly the word level, and later upgrades fill the
sense level without changing the format.

## Design documents

- v1 design: `docs/specs/2026-10-12-v1-design.md`
