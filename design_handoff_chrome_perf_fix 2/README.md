# Handoff to Claude Code — Chrome/Edge performance fix for the Braha portfolio

**Goal:** Get the two corrected files in this bundle (`index.html`, `site.css`) committed
and pushed to the GitHub repo that Vercel deploys, so the live site stops freezing/janking
on Chrome and Edge. After you push, Vercel auto-deploys.

This is **not** a "rebuild in a framework" task. The site is a static, no-build site
(plain HTML + CSS + a little vanilla JS). You are swapping two files and pushing.

---

## The repo

- **Repo:** `braha77/BRAHAi`
- **Branch:** `main`
- Vercel is connected via Git integration: a push to `main` triggers an automatic
  production redeploy. There is no build step (framework preset "Other", output dir = the
  site folder).

## ⚠️ Important: the repo has TWO folders — one is stale

The repo root currently contains:

```
README.md
michael-braha-portfolio/        <-- OLD, INCOMPLETE copy (missing most /assets screenshots)
michael-braha-portfolio 2/      <-- CURRENT, COMPLETE site (all screenshots). This matches the fixed files.
```

Vercel serves **only one** of these (whichever the project's **Root Directory** setting
points at). The fixed files in this bundle correspond to **`michael-braha-portfolio 2/`**
(same `index.html`/`site.css` lineage, with all asset screenshots present).

**Please do this:**

1. **Determine which folder Vercel actually serves.** Check, in order:
   - `vercel.json` or a `.vercel/project.json` in the repo (if present),
   - the Vercel dashboard → Project → **Settings → Build & Deployment → Root Directory**,
   - or just open the live URL and compare its HTML to each folder's `index.html`.
2. **Copy the two fixed files from this bundle into the folder that is actually served**,
   overwriting the existing `index.html` and `site.css` there. (Filenames stay the same;
   `image-slot.js` and `assets/` are unchanged — do not touch them.)
   - The complete/current folder is `michael-braha-portfolio 2/`, so unless the dashboard
     says otherwise, that is the target.
3. **Clean up the duplicate** (strongly recommended, to prevent future "I fixed it but it's
   still broken" confusion): once you've confirmed which folder is live, delete the other,
   stale folder in the same commit. Do NOT delete the live one.
4. **Commit and push to `main`:**
   ```
   git add -A
   git commit -m "Fix Chrome/Edge scroll performance (idle rAF, fixed-blend grain, will-change); remove stale duplicate folder"
   git push origin main
   ```
5. Wait ~30–60s for Vercel to redeploy, then hard-refresh the live site in Chrome
   (Ctrl/Cmd + Shift + R) and confirm scrolling is smooth.

> If the served folder turns out to be the *incomplete* `michael-braha-portfolio/` one,
> do NOT just drop these files in — its `assets/` is missing screenshots and images would
> break. In that case, make the **complete** `michael-braha-portfolio 2/` the deployed
> folder instead (point Vercel's Root Directory at it, or rename it), then apply these
> fixed files there. Flag this back to the user before pushing.

---

## What changed and why (the actual diff)

All three changes target Blink (Chrome/Edge) scroll/compositing cost. They are smooth on
macOS/Safari but lock up scrolling on average Windows Chrome/Edge hardware. No visual or
behavioral change is intended — the laptop scroll-zoom, dark "bloom," reveals, and Tweaks
panel all behave exactly as before.

### 1. `index.html` — the scroll animation loop now sleeps when idle
The scroll-driven laptop/zoom effect used a `requestAnimationFrame` loop that **ran forever**,
reading layout (`getBoundingClientRect()` ×3) and writing styles **every frame for the life
of the page** — even sitting still at the top. That is continuous forced layout at 60fps.

- Added a `running` flag + `wake()` helper. The loop now stops (`return` without scheduling
  another frame) once all interpolated values have settled, and is restarted by the
  `scroll`/`resize` listeners (`() => { compute(); wake(); }`).
- The per-frame bloom anchoring (`getBoundingClientRect()` on the i-dot + stage) is now
  **gated** behind `cBloom > 0.02 || tBloom > 0.02`, so those layout reads only happen
  while the orange bloom is actually visible.
- `compute()` now sets `laptop.style.willChange = (pin near viewport) ? 'transform' : 'auto'`
  so the laptop is only GPU-promoted near the pinned section.

### 2. `site.css` — `.grain` no longer uses a fixed-position blend layer
```css
/* before */
.grain{position:fixed;inset:0;z-index:-1;pointer-events:none;opacity:.5;mix-blend-mode:multiply; ...}
/* after */
.grain{position:fixed;inset:0;z-index:-1;pointer-events:none;opacity:.55; ...}
```
A `position:fixed` full-viewport element with `mix-blend-mode` forces the compositor to
re-blend the **entire screen** on every scroll frame — a well-known Chrome scroll killer.
Removing the blend (and nudging opacity .5 → .55 to keep the look) composites once. Visually
near-identical.

### 3. `site.css` — removed permanent `will-change` from `.laptop`
```css
/* before */ .laptop{position:relative;width:min(60vw,700px);will-change:transform}
/* after  */ .laptop{position:relative;width:min(60vw,700px)}
```
A permanent `will-change:transform` kept a large composited layer alive for the whole page.
It is now toggled from JS (see change #1) only while the pinned section is near the viewport.

---

## Files in this bundle

- `index.html` — fixed main page (drop into the deployed site folder, replacing its `index.html`)
- `site.css` — fixed styles (drop into the deployed site folder, replacing its `site.css`)
- `README.md` — this file

`image-slot.js` and everything under `assets/` are **unchanged** and are not included here —
leave the repo's existing copies in place.
