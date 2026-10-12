# 0002. Web first, with a shared TypeScript core for a later React Native app

- Status: Accepted
- Date: 2026-10-12

## Context

We want a web app now and a React Native app later, without rewriting the hard parts. The
hard parts are platform-independent: word segmentation, dictionary lookup, pinyin
formatting, and the build-time pipeline that produces dictionary data. The UI is
comparatively thin: an input area, a reading view, and a word-entry popover.

We prefer not to add development tooling beyond what ships with Node. The repository will
hold about four packages, so install speed and disk use are not concerns.

## Decision

We will use a single repository organised as npm workspaces, declared in the root
`package.json`. npm ships with Node, so the only required tool is the Node version in
`.nvmrc`.

- `packages/core`: pure TypeScript with no DOM, React, React Native or Node APIs. It receives
  dictionary data through interfaces and never fetches anything itself.
- `apps/web`: React + Vite, built to static files.
- `apps/mobile`: Expo (React Native), added when native work starts.
- `tools/dict-build`: Node scripts that build app data from source dictionaries.

React hooks and state shared by both apps move into a `packages/react-shared` package once
the second app exists. Until then they live in `apps/web`. UI components are written per
platform.

## Alternatives considered

- **Expo with react-native-web for both platforms (one UI codebase).** Maximises sharing,
  but the reading view relies on HTML text features (`<ruby>` for pinyin annotations, native
  text selection and copy) that react-native-web abstracts away, and plain React iterates
  faster for a web-first product. The logic worth sharing lives in `core` either way.
  Revisit if the UI grows large.
- **Separate repositories for web and native.** Shared code would have to be published and
  versioned as a package.
- **pnpm workspaces.** Strict isolation stops a package from importing dependencies it
  doesn't declare, which would help keep `core` platform-agnostic, and installs are faster.
  But it is an extra tool: corepack, which used to provide it, is no longer bundled from
  Node 25. React Native's Metro bundler also needs extra configuration for pnpm's symlinked
  `node_modules`.
- **Yarn or Bun workspaces.** The same extra-tool cost, with no advantage over pnpm here.

## Consequences

- `packages/core` must stay platform-agnostic. npm hoists `node_modules`, so a package can
  import a dependency it didn't declare. Guard against this with a tsconfig for `core` that
  excludes DOM types, plus a lint rule against undeclared and platform imports once ESLint
  is set up.
- Metro works with npm's hoisted layout without extra configuration. React Native pins an
  exact React version, so if the web and mobile apps need different React versions, follow
  Expo's monorepo guide to keep each app on its own copy.
- UI is implemented twice when the native app arrives.
- App data must load on both platforms: plain static files that the web app fetches and the
  native app bundles.
- Revisit the package manager if installs become slow or undeclared dependencies cause real
  bugs.
