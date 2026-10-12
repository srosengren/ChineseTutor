# 0002. Web first, with a shared TypeScript core for a later React Native app

- Status: Accepted
- Date: 2026-10-12

## Context

We want a web app now and a React Native app later, without rewriting the hard parts. The
hard parts are platform-independent: word segmentation, dictionary lookup, pinyin
formatting, and the build-time pipeline that produces dictionary data. The UI is
comparatively thin: an input area, a reading view, and a word-entry popover.

## Decision

We will use a single repository organised as a pnpm workspace:

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
- **npm workspaces.** Workable. pnpm was chosen for strict dependency isolation and speed.
  Expo supports pnpm monorepos. If Metro has trouble with symlinked dependencies,
  `node-linker=hoisted` in `.npmrc` is the known fallback.

## Consequences

- `packages/core` must stay platform-agnostic. Enforce this with a tsconfig that excludes
  DOM types and a lint rule against platform imports once the package exists.
- UI is implemented twice when the native app arrives.
- App data must load on both platforms: plain static files that the web app fetches and the
  native app bundles.
