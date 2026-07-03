# lufs-audio

The umbrella front door for **lufs.audio** — a self-contained, persistent-audio SPA on the LUFS
design system. This repo consolidates the landing, `about`, `portfolio`, and `workchain` surfaces into
one codebase with a shared reactive shell, replacing the archived `lufs-website` (v1).

Design: **Amacher** · engine/streaming lane pairs with **Ciani**. Full plan lives in the shared KB at
`agent-knowledge` → `docs/product/lufs-audio-redesign/` (see especially **05 consolidation-and-rollout**
and **06 prototype-and-roadmap**).

---

## What's here (Phase A scaffold)

- **`index.html`** — the whole site. One document, hash-routed SPA, no build step.
  - **Persistent shell** (lives *outside* the routed view, so it never reloads across navigation):
    - a **WebGL vscope-fog hero** — the finalized, verified engine reused from
      `lufs-vectorscope-fog-demo` (decision **D-BG**). Full-strength on the landing, dimmed/calmer on
      inner pages per the brand rule.
    - a **corner audio player** (lower-left) — decision **D-PLAY**; audio survives route changes.
    - the **custom cursor** — uniform everywhere, orbit dot on the **landing route only**.
  - **Routes:** `#/`, `#/workchain`, `#/about`, `#/portfolio`, `#/portfolio/<lane>` (`sound-design`,
    `music-composition`, `technical-audio`).
- **`assets/lufs-tokens.css`** — the canonical brand token + component stylesheet (source of truth).
  `index.html` currently inlines an expanded component set derived from these tokens; keep the two in
  sync, and migrate to a linked stylesheet if/when the site grows past a single document.
- **`assets/favicon.svg`** — a LUFS four-color vectorscope motif on `#111` (teal leads).

## Audio + reactivity

The hero is audio-reactive off a Web Audio `AnalyserNode`. The player streams a catalog track via the
**media plane** — Ciani's standalone **`lufs-media-worker`** (the `stream.lufsaud.io/api/stream/<id>`
contract). Analyser reactivity from a cross-origin stream requires **CORS on the audio *bytes*** (R2),
not just on the redirect — if that isn't in place for the `lufs.audio` origin, audio still *plays* but
the analyser reads silence and the hero falls back to its calm idle wisp. See
`agent-knowledge` → `docs/product/lufs-audio-redesign/06` (D-PLAY) and the `lufs-catalog-streaming`
skill. Full-catalog search (over the public `catalog.lufs.audio/stream-manifest.json`, 103 tracks)
lands in **Phase D**.

## Routing

Hash routing is deliberate: zero server config on the static host, and audio genuinely survives
navigation because the document never reloads. To move to **path URLs** (`lufs.audio/workchain`) later,
switch the router to the History API and add a catch-all rewrite so every path serves `index.html`.
On the Apache/Hostinger host that is an `.htaccess`:

```apache
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^ index.html [L]
```

## Deploy

Hostinger push-on-merge-to-`main` (the `hostinger-static-deploy` skill). **Gotcha banked from the
landing + arcade deploys:** the first deploy needs a manual hPanel Git webhook / docroot swap before
push-on-merge takes over.

## ⚠️ Verify WebGL on the real deploy — never in a sandbox

The publish/preview/`/s/` share sandboxes block the external `three@0.128` CDN `<script>`, so
`window.THREE` is undefined there and only the non-WebGL fallback renders. **A green preview does not
prove the hero.** Verify the hero on the actual Hostinger deploy (or by inlining three.js into a test
copy). The hero and router are in **separate `<script>` blocks** so a THREE load failure can never
break navigation.

Dev tuning: **Shift+D** toggles a hidden dial panel (the `live-dial-tuning-panel` surface) for
re-tuning the hero by eye; "Copy settings" emits JSON to bake as new defaults.

## Roadmap (from KB doc 06)

- **A — scaffold** *(this commit)*: SPA shell + tokens + persistent hero/player/cursor + routes.
- **B — content**: About + Portfolio finished on the brand system; Workchain page (install stays a
  labeled preview until `@lufs/workchain` publishes).
- **C — portfolio manifest**: a typed `Project` schema + per-lane registries + a registry test
  (decision **D-PORT**, the arcade `comptia-study-games` pattern) — adding a project = one entry.
- **D — audio**: persistent catalog player + resume-style reactivity; corner player + catalog search
  over the public manifest; confirm manifest CORS + R2 bytes CORS.
- **E — cut over**: finish retiring `vscope.lufs.audio`; flip the landing from "Coming soon." to the
  full site; layer the product/install hero when Workchain distribution is real.

## Brand honesty rules (carried into the copy)

No public pricing (engagement by inquiry); the Workchain install is a **labeled preview** (no promise
that doesn't run); the trailing dot in `lufs.` is load-bearing.

## Credits

Built on the **LUFS Brand Design System**. Hero engine adapted from `lufs-vectorscope-fog-demo`.
© 2026 LUFS Audio.
