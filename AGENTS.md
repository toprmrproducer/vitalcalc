# VitalCalc

Free, fully client-side BMI (Body Mass Index) calculator, monetized via Google AdSense.
This is a **deliberate single-purpose site** — one keyword-matched domain per tool is
the whole SEO strategy. Do NOT add other tools, a multi-tool nav, or links out to
sibling projects (anyconvert, scriptflow, etc). If a second tool idea comes up, it gets
its own standalone project/domain, not a page here.

## What it does

One page (`/`): enter height (cm or ft/in) and weight (kg or lb), get your BMI, its
WHO category (Underweight / Normal weight / Overweight / Obese), and a visual
segmented gauge showing where you land. Everything is computed client-side with plain
JavaScript — no upload, no backend, no API key, no per-use cost.

## Stack

- **AstroJS** (static output) + **Tailwind v4** (`@tailwindcss/vite` plugin, imported
  in `src/styles/global.css`)
- Plain TypeScript in a `<script>` block — no calculation dependencies at all (no
  ffmpeg.wasm, no ML models, nothing). This tool needs zero extra packages beyond
  Astro + Tailwind.
- **@astrojs/sitemap** for `sitemap-index.xml`.
- **Netlify** hosting.

## Design system (mandatory — do not deviate)

Premium cream/gold aesthetic, deliberately NOT the generic white/cyan AI-tool look:

- Background: `#FAF6EC` (warm cream)
- Card/surface: `#FFFFFF`, border `#E8DFC8` (warm, not gray)
- Headings text: `#2B2013` (warm espresso brown, not pure black)
- Body text: `#5C4F3D` (warm brown-gray)
- Accent/CTA/links/active state: `#C9982E` (warm gold), hover `#B8860B` (darker gold)
- Fonts: **Fraunces** (Google Font, soft-serif/display) for all headings — loaded via
  `<link>` in `Layout.astro`'s `<head>`, `font-display: swap`. **Inter** (Google Font)
  for body/labels/buttons/UI. Headings use weight 600–700 with slightly negative
  letter-spacing for a tightened, premium look.
- Generous whitespace, soft shadows only (never harsh), rounded-xl/2xl corners, no
  gradients, no dark mode toggle — cream-only is the whole point.

Tokens live in `src/styles/global.css` under `@theme` (`--color-cream`,
`--color-surface`, `--color-border-warm`, `--color-espresso`, `--color-brown`,
`--color-gold`, `--color-gold-dark`, `--font-display`, `--font-body`). Prefer the
arbitrary-value Tailwind classes (`text-[#2B2013]` etc.) already used throughout
`src/pages/*.astro` and `src/layouts/Layout.astro` to stay consistent with the
existing code rather than introducing a second token naming scheme.

## Structure

- `src/layouts/Layout.astro` — shared shell: simple header (brand name only, no nav to
  other tools by design), footer (About/Privacy/FAQ + copyright), SEO meta tags.
- `src/pages/index.astro` — the tool. Height/weight inputs with unit toggles (cm vs
  ft/in, kg vs lb) → BMI calculation → WHO category label → segmented visual gauge
  with a position marker.
- `src/pages/about.astro`, `privacy.astro`, `faq.astro`, `404.astro` — required
  AdSense-eligibility pages, linked from both the homepage body and the footer.
- `public/robots.txt`, `public/ads.txt` (placeholder — replace with the real AdSense
  publisher line once approved), `public/favicon.svg`.

## BMI logic (source of truth, ported from anyconvert's bmi-calculator.astro)

- Height → meters: cm input divided by 100, or `(ft * 12 + in) * 0.0254`.
- Weight → kg: raw value, or `lb * 0.45359237` if imperial.
- `BMI = weightKg / (heightM * heightM)`.
- WHO categories: `< 18.5` Underweight, `< 25` Normal weight, `< 30` Overweight,
  else Obese.
- Gauge: linear scale from 0 to 40 BMI mapped to 0–100% marker position, clamped.

## Security

Any user-controlled or computed text that could ever be routed through `innerHTML`
must go through an `escapeHtml()`-style helper first. The BMI value and category label
are set via `textContent` (already safe) but are still passed through `escapeHtml()`
first as defense-in-depth and as the canonical pattern to copy if this code is ever
adapted to use `innerHTML`. See the helper at the top of the `<script>` block in
`src/pages/index.astro`.

## Dev / test

- `npm run dev` — runs on port **4340** (anyconvert owns 4325, reelshift/scriptflow
  own 4331, stillmotion owns 4332 — keep VitalCalc on its own port since sibling tool
  projects may be running concurrently).
- Test by filling `#height-cm` and `#weight-input`, clicking `#calculate-btn`, and
  reading `#result-bmi` / `#result-category` / `#gauge-marker`'s `left` style —
  standard form interaction, no file picker or worker complexity involved.

## iCloud sync hygiene

This project lives in iCloud Drive (`~/iCloud/website/vitalcalc`). `node_modules` is
renamed to `node_modules.nosync` with a symlink back to `node_modules` so iCloud does
not try to sync build-tool dependency trees (it chokes on the file count). `dist/` and
`.astro/` are device-local and already gitignored.

`tsconfig.json`'s `exclude` array includes **both** `"node_modules"` and
`"node_modules.nosync"` — `tsc`/`astro check` does not recognize the `.nosync` suffix
as implicitly excluded, so without the explicit entry `astro check` crashes trying to
type-check into the dependency tree.

## Deploy

Netlify site `vitalcalc` (or whatever suffixed name Netlify assigned if that name was
taken — check `.netlify/state.json` for the actual site ID/URL). Deployed via:

```bash
source ~/.claude/credentials/netlify.env
export NETLIFY_AUTH_TOKEN=$NETLIFY_API_KEY
netlify deploy --prod --dir=dist
```

Run `npm run build` first and confirm no errors before deploying.

## Single-purpose site strategy

This is intentional, not an oversight: VitalCalc has no nav or footer links to any
other tool, no shared multi-tool homepage, and no "AnyConvert" branding anywhere. The
strategy across this site family is one narrow, keyword-matched tool per domain (this
one: BMI calculation), each independently indexable and rankable, rather than one big
multi-tool hub. Do not merge this back into anyconvert or add tool-switcher UI.
