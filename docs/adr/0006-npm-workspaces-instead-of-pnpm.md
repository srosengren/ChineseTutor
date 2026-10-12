# 0006. npm workspaces instead of pnpm

- Status: Accepted
- Date: 2026-10-12
- Supersedes: the package-manager choice in ADR-0002 (the rest of ADR-0002 stands)

## Context

ADR-0002 chose a pnpm workspace. We prefer not to add development tooling beyond what ships
with Node. The repository will hold about four packages, so install speed and disk use are
not concerns. pnpm would arrive through corepack, which Node no longer bundles from Node 25
onwards, so it would eventually be a separate install.

## Decision

We will use npm workspaces, declared in the root `package.json`. npm ships with Node, so the
only required tool is the Node version in `.nvmrc`.

## Alternatives considered

- **pnpm.** Strict isolation stops a package from importing dependencies it doesn't declare,
  and installs are faster. It is an extra tool, and React Native's Metro bundler needs extra
  configuration for pnpm's symlinked `node_modules`.
- **Yarn or Bun.** The same extra-tool cost, with no advantage over pnpm for this repo.

## Consequences

- `node_modules` is hoisted, so a package can import a dependency it didn't declare. We guard
  `packages/core` with a tsconfig that excludes DOM types, plus a lint rule against
  undeclared imports once ESLint is set up.
- Metro works with npm's hoisted layout without extra configuration. React Native pins an
  exact React version, so if the web and mobile apps need different React versions, follow
  Expo's monorepo guide to keep each app on its own copy.
- Revisit if installs become slow or undeclared dependencies cause real bugs.
