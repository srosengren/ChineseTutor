# 0006. Dictionary sources for v1, and a word-class model that allows per-sense detail later

- Status: Accepted
- Date: 2026-10-12

## Context

A word entry needs pinyin, definitions, word class and measure words, and the segmenter
needs word frequencies. What the free sources offer (checked 2026-10-12):

| Source | Licence | Size | Offers | Lacks |
|---|---|---|---|---|
| CC-CEDICT | CC BY-SA 4.0 | 125,262 entries | Readings, short separated senses, measure words | Word class |
| jieba `dict.txt` | MIT | 349,046 words | Frequency, one word-class tag per word | Pinyin, definitions |
| complete-hsk-vocabulary | MIT | 11,470 words | HSK levels, several word-class tags per word | Per-sense classes; its definitions are copied from CC-CEDICT |
| Wiktionary (kaikki.org extract) | CC BY-SA 4.0 | 307,224 words, 1.1 GB download | Word class per sense for multi-character words | Word class for single characters |

No free source gives word class per sense for single characters (了, 打, 白), which are the
words a beginner taps most.

## Decision

v1 uses three sources:

- **CC-CEDICT** for headwords, readings, senses and measure words.
- **jieba** for word frequencies, and for word class when the HSK list has no tags for a word.
- **complete-hsk-vocabulary** for HSK level and word-level classes. We do not use its
  definitions (copied from CC-CEDICT) or its frequencies (taken from a third project with no
  clear licence).

The segmentation lexicon is the set of CC-CEDICT headwords, weighted by jieba frequency, so
every segment has an entry.

Word class is stored at two levels:

- **Word level** (`Entry.wordClasses`): filled in v1 from the HSK list, falling back to jieba.
- **Sense level** (`Sense.wordClasses`): optional. In v1 it is filled only where CC-CEDICT's
  writing conventions make it certain: "to …" means verb, "classifier for …" means measure
  word, glosses marked as particles mean particle, and capitalised pinyin or a "surname …"
  gloss means proper noun.

The UI shows word-level classes at the top of an entry and sense-level classes next to the
senses that have them. Word
classes use our own small set of learner-facing labels (noun, verb, adjective, adverb,
measure word, particle, and so on). Each source's tag codes are mapped to those labels at
build time and never reach the app.

Entry shape (full definition in `docs/specs/2026-10-12-v1-design.md`):

```ts
interface Entry {
  word: string;              // simplified headword
  readings: Reading[];       // one per pronunciation, e.g. 了: le, liǎo
  wordClasses: WordClass[];  // word level
  hskLevel?: number;
}

interface Reading {
  pinyin: string;            // numbered syllables, e.g. "xue2 xi2"
  traditional: string;       // per reading: 了 is 了 for le but 瞭 for one liǎo reading
  senses: Sense[];
  classifiers: Classifier[]; // measure words, e.g. { word: "个", pinyin: "ge4" }
  notes: string[];           // e.g. "Taiwan pr. [da3]"
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
- `tools/dict-build` writes `docs/data-sources.md` with the pinned source versions each time
  it runs.
