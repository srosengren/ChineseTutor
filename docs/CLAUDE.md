# Documentation conventions

| Path | What goes there |
|---|---|
| `architecture.md` | Living overview of the system. Keep it current; update it in the same commit as the change. |
| `adr/` | Architecture Decision Records, one decision per file. |
| `specs/` | Feature designs: `YYYY-MM-DD-<topic>-design.md`. Create the folder with the first spec. |
| `plans/` | Implementation plans: `YYYY-MM-DD-<topic>-plan.md`. Create the folder with the first plan. |
| `data-sources.md` | Every third-party dataset: source URL, version or download date, licence, attribution text. Create it with the first dataset. |

## ADRs

Write one when a decision is costly to reverse, affects more than one package, or adds an
external service, dataset or licence obligation.

1. Copy `adr/template.md` to `adr/NNNN-kebab-case-title.md`, using the next free 4-digit number.
2. Fill in every section. "Alternatives considered" must name at least one real alternative.
3. Add a row to the index in `adr/README.md` in the same commit.

Status moves `Proposed` → `Accepted`. Later an ADR can become `Superseded by NNNN` or
`Deprecated`.

Until code depends on an ADR, revise it in place when the decision changes. For example,
a choice reversed in the same discussion is edited into the original ADR, with the old
option moved to "Alternatives considered". Once implementation relies on an ADR, never
rewrite its decision: write a new ADR that supersedes it and update the status line of
both.
