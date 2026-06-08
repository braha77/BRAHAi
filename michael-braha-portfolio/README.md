# Michael Braha — Portfolio

A static, scroll-driven portfolio site. No build step, no dependencies — just HTML, CSS, and a little vanilla JS.

## Structure

```
index.html        Main page (everything renders from here)
site.css          All styles
image-slot.js     Tiny web component for drag-to-fill image placeholders
assets/           Images + video (project screenshots, Drift GIF, Choppr screen recording)
```

Fonts (Spectral + Helvetica Neue/system grotesk) load from Google Fonts at runtime.

## Run locally

It's a static site, so any static server works:

```bash
# Python
python3 -m http.server 8000

# or Node
npx serve .
```

Then open http://localhost:8000.

> Opening `index.html` directly via `file://` will not play the Choppr video
> (the page fetches the clip as a Blob). Use a local server.

## Deploy to Vercel

1. Push this folder to a GitHub repo.
2. In Vercel: **New Project → Import** the repo.
3. Framework preset: **Other**. Build command: _none_. Output directory: `.` (root).
4. Deploy.

Because `index.html` sits at the repo root, Vercel serves it with zero config.

## Notes

- **Choppr** and **Drift** show looping screen recordings; every other project card
  shows a static screenshot (or a placeholder where a real screenshot still needs to drop in).
- Replace the placeholder contact links in `index.html` (`#contact` section) and the
  `hello@michaelbraha.com` email with real ones before going live.
