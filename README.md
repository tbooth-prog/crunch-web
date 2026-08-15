# Crunch — website

The public site for [Crunch](https://play.google.com/store/apps/details?id=io.game.crunch),
a free daily number puzzle. Two static pages, published with GitHub Pages: the
privacy policy the app stores link to, and the URL that puts people on the right
store for their phone.

Plain HTML with inline CSS. No build step, no framework, no dependencies beyond
Google Fonts.

## Pages

| URL      | File                | What it does                                                                                                                             |
| -------- | ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `/`      | `index.html`        | Privacy policy. The URL the Play Store listing and the app's in-app link should point at — see the checklist below, neither does yet.    |
| `/play/` | `play/index.html`   | Store redirect. Android goes straight to the Play Store; iOS and desktop get a landing page. This is the URL to put on cards, bios, etc. |

The `/play/` redirect runs in a `<head>` script before any body content is
parsed, so Android visitors never paint a frame of the page, and it uses
`location.replace` so Back from the Play Store returns to wherever they came
from rather than to an empty page. `<noscript>` covers JavaScript being off.

## Local preview

```sh
python3 -m http.server 8000
# → http://localhost:8000/ and http://localhost:8000/play/
```

Everything is referenced with **relative** paths (`assets/…` from the root,
`../assets/…` from `play/`), so the site works at a custom domain and at a
GitHub Pages project sub-path alike. Keep it that way — a leading `/` breaks the
sub-path case.

## Design system

Colours and fonts are lifted from the app's `constants/theme.ts` so the site and
the app resolve to identical values. **Don't let them drift.** If a token changes
in the app, change it here.

Type: DM Mono for the logotype and numbers, Syne for headings and labels, Nunito
Sans for body. DM Mono stops at weight 500 — asking for more gets a synthesised
fake bold with smeared stems, so cap it at 500 as the app does.

One token is not from the app: `--muted-read: #9A9AB2`. The app's
`--muted: #6C6C82` measures 3.2–3.8:1 on these surfaces, which is fine for
labels and chrome but under AA for sentence-length text on a phone. Prose uses
the lifted tone; everything else keeps the app token.

## Assets

| File                        | Used by    | Notes                                                                                                                                                                                        |
| --------------------------- | ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `assets/favicon.svg`        | both pages | Authored here. 3 × 4 tile "C" on `#0C0C18`, same proportions and optical offset as the app icon, glyph scaled to ~75% of the canvas so it still reads at 16px in a tab. Covers modern browsers. |
| `assets/favicon.png`        | both pages | 256 × 256 fallback for browsers that don't take SVG favicons. Copied from the app repo.                                                                                                       |
| `assets/apple-touch-icon.png` | both pages | Home-screen icon when an iOS visitor saves the page — likely, since `/play/` is the iOS landing page. 180 × 180, opaque (iOS puts black behind transparency).                              |
| `assets/og-image.png`       | both pages | 1200 × 630 link preview card for Slack, iMessage, WhatsApp, X. Generated — see below.                                                                                                         |
| `assets/og-image.html`      | nothing    | Source for `og-image.png`. A rendering surface, not a page: it carries `noindex, nofollow`, and nothing links to it.                                                                          |

To re-copy the two icons from the app repo:

```sh
cp ~/Codebase/Crunch/assets/favicon.png assets/favicon.png
sips -Z 180 ~/Codebase/Crunch/assets/icon.png --out assets/apple-touch-icon.png
```

### Regenerating the OG image

Rendered at 2× and downsampled, which is what keeps the type crisp. With a
static server running at the repo root:

```sh
# 1. Render at 2400 × 1260 (any headless-Chrome runner will do)
npx playwright screenshot \
  --viewport-size=1200,630 --scale=css --wait-for-timeout=1000 \
  http://localhost:8000/assets/og-image.html assets/og-image.png

# 2. Downsample to the exact 1200 × 630 the meta tags promise
sips -z 630 1200 assets/og-image.png --out assets/og-image.png
```

The card is deliberately platform-agnostic — it names no store and no OS, so it
survives an iOS launch untouched. The puzzle on it is a legal draw under the
app's `constants/numbers.ts` (two large, four small, target within 100–500) and
solves exactly: (50 + 7) × 6 = 342. If you change the numbers, keep it
solvable — someone will check.

Regenerating needs network access for Google Fonts, and a future release of DM
Mono or Syne could shift metrics slightly. Fine for a preview card; don't expect
a byte-identical rebuild.

## Before publishing

- [ ] Replace `REPLACE-WITH-EMAIL@example.com` in `index.html` — it appears
      twice, in the `href` and in the link text, marked with an HTML comment.
- [ ] Update the policy's `<time datetime>` and its visible date if the copy has
      changed since it was written.

Then, once the site has a live URL, two things outside this repo point at it:

- [ ] **The app's in-app link.** `app/settings.tsx` in the Crunch repo opens
      `https://example.com/crunch-privacy` — a placeholder with a TODO on it.
      Until that is changed, the Privacy Policy row in the app's settings screen
      goes nowhere useful.
- [ ] **The Play Store listing.** Its privacy policy field should point at the
      same URL. Google requires one for published apps.

## When iOS ships

Every platform reference on the site is in `play/index.html`. The privacy policy
and the OG image name no platform and need nothing.

| Line  | What                                                                       |
| ----- | -------------------------------------------------------------------------- |
| `12`  | `<meta name="description">` — "Available now on Android, coming to iOS soon" |
| `25`  | `isIOS` — computed but unused today; this is the hook                       |
| `396` | The `<h1>` headline                                                        |
| `398` | The blurb                                                                  |
| `408` | The store card line, and the card becoming a two-platform choice           |

The redirect becomes an `else if (isIOS) location.replace(appStoreUrl)` next to
the Android branch, and the page underneath becomes the desktop fallback.
