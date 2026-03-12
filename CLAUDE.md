# conspiracyos.github.io

Static HTML documentation site. No build step — raw HTML served via GitHub Pages.

## Key Rules
- **No Jekyll.** `.nojekyll` is present. Do NOT add `.md` content files, `_config.yml`, `Gemfile`, or Jekyll layouts/themes. All pages are hand-written HTML.
- **Branding:** Always "ConspiracyOS" (capital C, capital OS). Never "conspiracyos" in user-facing text.
- **Styling:** All pages use `style.css`. No external CSS frameworks.
- **Pushing:** Never push directly. Use `./scripts/sync-repos.sh website` from the monorepo root (`~/Developer/ConspiracyOS/`).

## Files
- `index.html` — landing page
- `docs.html` — full documentation (sidebar + content)
- `how-it-works.html` — architecture overview
- `contributing.html` — contributor guide
- `style.css` — shared styles
- `docs/index.html` — redirect from /docs/ to /docs.html
