# 0003. Static hosting on GitHub Pages, no backend

- Status: Accepted
- Date: 2026-10-12

## Context

The MVP features (segmentation, pinyin, dictionary lookup) can run entirely in the client
against static data files. We want no servers to operate and no running cost, and the code
already lives on GitHub.

## Decision

We will deploy the web app as a static build to GitHub Pages from this repository, using a
GitHub Actions workflow as the Pages source. There is no backend. Any feature that needs
server-side compute or secrets requires its own ADR.

## Alternatives considered

- **Cloudflare Pages or Netlify.** Equally static and free, with features we don't need
  yet (edge functions, preview deploys). Adds a second account and integration.
- **A small server (e.g. a container on a VPS).** Running cost and maintenance for no
  current benefit.

## Consequences

- The site is served at `https://srosengren.github.io/ChineseTutor/`, so the Vite `base`
  must be `/ChineseTutor/`.
- No secrets in client code.
- Dictionary data ships as static assets. Keep the compressed payload to a few MB, load it
  lazily, and cache it for offline use.
- On the free GitHub plan, Pages and branch protection require a public repository, so the
  repository is public (licensing in ADR-0005). Any bundled third-party data must allow
  public redistribution.
- GitHub Pages limits (1 GB site, 100 MB per file, soft 100 GB/month bandwidth) are far
  above what this app needs.
