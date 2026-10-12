# 0001. Record architecture decisions

- Status: Accepted
- Date: 2026-10-12

## Context

The project starts as a web app and later gains a native app sharing the same core. Choices
made now (repo layout, data formats, hosting) constrain that later work, and both humans and
coding agents need to know why things are the way they are without reconstructing it from
commit history.

## Decision

We will record significant decisions as Architecture Decision Records in `docs/adr/`, using
the template in `docs/adr/template.md` and the conventions in `docs/CLAUDE.md`.

## Alternatives considered

- **Decisions in commit messages and PR descriptions only.** Hard to find later, and the
  rejected alternatives are rarely written down.
- **A single design document.** Gets edited in place, so the history of why something changed
  is lost.

## Consequences

- Small overhead per significant decision.
- Agents working in the repo read `docs/adr/` before proposing changes to the architecture.
- Superseded decisions remain readable, so the reasoning trail is preserved.
