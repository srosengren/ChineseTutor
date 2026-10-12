# 0007. Dictionary sources for v1, and a word-class model that allows per-sense detail later

- Status: Accepted
- Date: 2026-10-12

## Context

A word entry needs pinyin, definitions, word class and measure words, and the segmenter
needs word frequencies. What the free sources offer (checked 2026-10-12):

| Source | Licence | Size | Offers | Lacks |
|---|---|---|---|---|
| CC-CEDICT | CC BY-SA 4.0 | 125,262 entries | Readings, short separated senses, measure words | Word class |
| jieba `dict.txt` | MIT | 349,046 words | Frequency, one word-class tag per word | Pinyin, definitions |
| complete-hsk-vocabulary | MIT | 11,470 words | HSK levels, frequency rank, several word-class tags per word | Per-sense classes; its definitions are copied from CC-CEDICT |
| Wiktionary (kaikki.org extract) | CC BY-SA 4.0 | 307,224 words, 1.1 GB download | Word class per sense for multi-character words | Word class for single characters |

No free source gives word class per sense for single characters (了, 打, 白), which are the
words a beginner taps most.

## Decision

v1 uses three sources:

- **CC-CEDICT** for headwords, readings, senses and measure words.
- **jieba** for word frequencies, and for word class when the HSK list has no tags for a word.
- **complete-hsk-vocabulary** for HSK level, frequency rank and word-level classes. We do not
  use its definitions.

The segmentation lexicon is the set of CC-CEDICT headwords, weighted by jieba frequency, so
every segment has an entry.

Word class is stored at two levels:

- **Word level** (`Entry.wordClasses`): filled in v1 from the HSK list, falling back to jieba.
- **Sense level** (`Sense.wordClasses`): optional. In v1 it is filled only where CC-CEDICT's
  writing conventions make it certain: "to …" means verb, "classifier for …" means measure
  word, and glosses marked as particles mean particle.

The UI shows sense-level classes where present and word-level classes otherwise. Word
classes use our own small set of learner-facing labels (noun, verb, adjective, adverb,
measure word, particle, and so on). Each source's tag codes are mapped to those labels at
build time and never reach the app.

Illustrative shape (the exact format is specified with the data pipeline):

```ts
interface Entry {
  simplified: string;
  traditional: string;
  readings: Reading[];       // one per pronunciation, e.g. 了: le, liǎo
  wordClasses: WordClass[];  // word level
  hskLevel?: number;
  frequencyRank?: number;
}

interface Reading {
  pinyin: string;            // numbered syllables, e.g. "xue2 xi2"
  senses: Sense[];
  classifiers: string[];     // measure words, e.g. ["个"]
}

interface Sense {
  gloss: string;
  wordClasses?: WordClass[]; // sense level, sparse in v1
}
```

## Alternatives considered

- **Wiktionary as the main dictionary.** Gives per-sense classes for multi-character words,
  but adds a larger payload and two definition styles, and still has nothing for single
  characters.
- **Wiktionary word-level tags in v1.** Requires processing a 1.1 GB download for a small
  gain over the HSK list plus jieba. Deferred to the per-sense upgrade.
- **Tag every CC-CEDICT sense with an LLM at build time.** The best per-sense coverage,
  including single characters, and the runtime stays static. This is the planned upgrade,
  not v1.
- **jieba alone.** It has no pinyin and no definitions.

## Consequences

- In v1, word class is mostly per word (白 → adjective · adverb · verb), with some per-sense
  labels.
- Upgrading to per-sense word class (Wiktionary for multi-character words, LLM tagging for
  the rest) fills `Sense.wordClasses`. It is a data-pipeline change, with no change to the
  format or the UI.
- Built data derived from CC-CEDICT is CC BY-SA 4.0. The app shows attribution for
  CC-CEDICT, jieba and complete-hsk-vocabulary.
- Words that jieba knows but CC-CEDICT doesn't (mostly personal and place names) segment
  into single characters.
- `docs/data-sources.md` is created when `tools/dict-build` first downloads the sources, with
  pinned versions.
