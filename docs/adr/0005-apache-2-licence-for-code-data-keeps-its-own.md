# 0005. Apache-2.0 for the code; third-party data keeps its own licence

- Status: Accepted
- Date: 2026-10-12

## Context

The repository is public so that GitHub Pages and branch protection work on the free plan
(ADR-0003). We want a licence that:

- does not limit which dictionary data we can use, and
- keeps open the option of taking future development private or commercial.

The likely dictionary sources are CC BY-SA 4.0 (CC-CEDICT, Wiktionary) and MIT (jieba).
CC BY-SA's share-alike applies to adaptations of the data, not to separate code that loads
and displays it.

## Decision

We will license the code under Apache-2.0. Third-party data, and data derived from it,
keeps its source licence and is documented in `docs/data-sources.md`. Data sources must
allow commercial use and redistribution: no NonCommercial (NC) licences, and no GPL code
bundled into the app.

## Alternatives considered

- **MIT.** Equally permissive and shorter, but has no explicit patent grant.
- **Functional Source License (FSL-1.1-ALv2).** Source-available: blocks competing
  commercial use for two years, after which each release becomes Apache-2.0. Useful if we
  want to stop others from commercialising a fork, but it is not an open-source licence.
  Not needed at this stage.
- **GPL / AGPL.** Copyleft would apply to forks, and accepting outside contributions would
  make going closed much harder.

## Consequences

- As sole copyright holder we can make future versions closed at any time. Code already
  published stays available under Apache-2.0.
- If we accept outside contributions, Apache-2.0 (inbound = outbound) lets us use them in a
  closed version. A CLA is not required for that, but is needed to relicense to anything
  non-permissive.
- The app must show attribution for CC BY-SA data, and our processed dictionary files are
  themselves CC BY-SA. Publishing those files openly keeps us compliant even if the app's
  code later goes closed.
- GitHub Pages terms disallow sites that are primarily commercial SaaS. If the app goes
  commercial, hosting moves elsewhere (superseding ADR-0003).
