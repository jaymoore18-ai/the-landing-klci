# The Landing at KLCI

Marketing website for The Landing at KLCI — private aircraft hangars at Laconia
Municipal Airport, NH. Production domain: **luxuryhangars.com**.

This is a static, single-page site. There is no build step, no framework, and
no backend — it is one HTML file plus a folder of images and video. You can
open `index.html` directly in a browser, or deploy the folder as-is to any
static host.

## Folder structure

```
index.html              The entire site (markup, CSS, and JS in one file)
assets/
  video/
    hero-bg.mp4          Hero background video (MP4 — Safari/iOS fallback)
    hero-bg.webm         Hero background video (WebM — primary, smaller file)
    hero-poster.jpg       Poster frame shown before the video loads
  images/
    slide-2.jpg … slide-10.jpg / .png
                          The flattened portfolio slides (one per page of
                          the deck). These are pre-composed images — the
                          headlines, diagrams, and photography on each are
                          baked into the image file itself, not live text.
    archive/              Unused source variants kept for reference only
                          (not linked from index.html — safe to delete).
```

## How the page is built

The site is a vertical "deck" of full-bleed sections (`<section class="deck-slide">`
inside `<main class="deck">`), one per page: Opportunity, Hangars, Materials,
Lakes, Masterplan, Floorplans, Milestones, Ownership, Contact. CSS
`scroll-snap-type: y mandatory` makes the page snap to each section as the
visitor scrolls, and a small `IntersectionObserver` in the closing `<script>`
block (see `setActive()`) tracks which section is on screen, updates the nav
counter and dot indicators, and adds a one-time `.revealed` class to each
section the first time it comes into view — that class is what triggers the
gentle fade/scale-in on each slide's image. Nothing re-animates on repeat
scrolls.

Each section's real content is a full-bleed image (the flattened portfolio
slide) with a short visually-hidden paragraph next to it (`.slide-copy`)
carrying the same text as alt content for accessibility and SEO — that
hidden copy is the thing to edit if page wording needs to change, since the
visible imagery itself is flattened artwork, not live type.

### Responsive behavior — read before touching layout CSS

Desktop and mobile render the slide images differently, and this matters if
you ever add an overlay (like the phone-number redaction box, see below):

- **Desktop (>700px):** images use `object-fit: fill`, so each image is
  stretched to exactly match its container. Percentage-based CSS positioning
  (`left: 64%`, `top: 76%`, etc.) lines up correctly against the image
  because the container and the image share the same aspect ratio.
- **Mobile (≤700px):** images switch to `object-fit: contain` (see the
  `@media (max-width:700px)` block), so the image is letterboxed/centered
  inside its container instead of stretched. Fixed CSS percentages will
  *not* line up with the image content at this breakpoint — anything that
  must stay pixel-aligned to the image on both layouts needs to be
  positioned with JavaScript instead. See `positionPhoneRedact()` near the
  bottom of the closing `<script>` block for a working example: it reads
  the rendered image's actual box on screen and repositions an overlay
  `<div>` against it, recalculating on load and on resize.

### The flat panel color technique (`#292921`)

Several slides have a solid dark panel as part of their design. That exact
color (`#292921`) was sampled pixel-for-pixel from the source images with
PIL, so a plain CSS `<div>` filled with it and placed over a seam or a
sensitive detail is visually seamless against the real panel next to it —
no image editing required. Two examples already in the page:

- `.seam-align--floorplans` — a thin strip that hides a minor panel-width
  mismatch where the Masterplan and Floorplans slides meet.
- `.redact--phone` (on the Contact/Buyer Opportunities page) — covers a
  phone number that was baked into that slide's source image, positioned
  with `positionPhoneRedact()` as described above so it stays correctly
  placed on both desktop and mobile.

If you ever need to cover or align something else on a flattened slide, this
is the pattern: a `#292921`-filled `<div>`, positioned in CSS if it only
needs to work on desktop, or in JS (following `positionPhoneRedact()`) if it
must also hold on mobile.

### Design tokens

Colors, easing curves, and other repeated values are defined once as CSS
custom properties near the top of the `<style>` block (`--ease`,
`--gold-light`, `--on-dark`, etc.) — change a value there rather than
hunting for it throughout the file.

### Motion

All entrance motion is intentionally restrained: a slow opacity + slight
scale settle on each slide's image the first time it's scrolled into view,
plus a soft fade on the hero's headline/scroll-cue as the visitor leaves the
hero. It plays once per slide, never on repeat scrolls, and everything
respects `prefers-reduced-motion` (see the `@media (prefers-reduced-motion:
reduce)` block, which disables all of it for visitors who've asked for
reduced motion at the OS level).

## Mobile build-out

Below 700px, every section of the deck (the hero plus all nine portfolio
slides) shows a real, live-coded mobile layout instead of one flattened
desktop image, matching mockups provided page by page. **Desktop is
completely untouched** — it always shows the original flattened `slide-N`
images exactly as before; none of this changes anything above 700px.

**Hero video:** phones load a separate, ~70% smaller encode of the same
footage (`assets/video/hero-bg-mobile.mp4` / `.webm`, 480x270) instead of
the desktop file (1280x720), picked automatically via `<source
media="(max-width:700px)">` on the `<video>` element — no JS needed for
the video itself. A matching small poster (`hero-poster-mobile.jpg`) is
set in a tiny inline script instead of a `poster` attribute, since
`<video>` has no responsive-poster mechanism. If the source video is ever
replaced, regenerate the mobile pair from the new file, e.g.:

```
ffmpeg -i hero-bg.mp4 -vf "scale=480:270:flags=lanczos" -c:v libx264 \
  -profile:v main -pix_fmt yuv420p -crf 28 -preset slow -an \
  -movflags +faststart hero-bg-mobile.mp4

ffmpeg -i hero-bg.webm -vf "scale=480:270:flags=lanczos" -c:v libvpx-vp9 \
  -b:v 0 -crf 38 -an hero-bg-mobile.webm
```

**The nine portfolio slides:** each one's mobile version (`.opp-mobile`,
`.hangars-mobile`, `.materials-mobile`, `.lakes-mobile`,
`.masterplan-mobile`, `.floorplans-mobile`, `.milestones-mobile`,
`.ownership-mobile`, `.contact-mobile` in `index.html`) is built from
photos cropped out of that slide's own flattened source image (see the
`assets/images/*.jpg`/`.png` files named after each section — e.g.
`hangars-photo-1.jpg`, `masterplan-diagram.jpg`, `contact-headshot.jpg`),
with headlines, specs and body copy as live text instead of baked
artwork. A few sections (Milestones' timeline, Ownership's lines,
Contact's phone/email) reuse copy or CSS classes that already existed
elsewhere in this file for the same content on desktop. On Contact
specifically, the phone number is shown as plain live text on mobile —
`.redact--phone` (the flat-color cover used on desktop because the
number is baked into that slide's image) is switched off at this
breakpoint, since there's nothing left to redact.

Because this content generally runs taller than one phone screen, these
sections also drop the "one slide = one screen height" sizing the deck
used before every page had a live mobile version — they just flow at
their natural height. The page-counter in the bottom-right corner
("03 / 09" etc.) tracks that with a scroll-position heuristic on mobile
(see `currentSlideByScroll()` in the closing `<script>` block) rather
than the simpler "55% of this slide is visible" rule desktop still uses,
since most mobile slides are now taller than the viewport.

Any new photo crop added to `assets/images/` for a future mobile section
should get explicit `width`/`height` attributes on its `<img>` tag (its
real pixel dimensions) even though it's also `loading="lazy"` — without
them the browser can't reserve the right amount of space before the
image loads, which briefly under-reports the page's height and (on a
slow connection) shifts content around as images pop in.

## Editing content

- **Wording on a page:** the visible imagery is flattened artwork, so
  changing what a slide *says* means editing the source image (outside this
  repo) and replacing the corresponding file in `assets/images/`, and
  updating the matching hidden `.slide-copy` text in `index.html` to match.
- **Contact info, nav labels, meta/SEO text:** these are live text/HTML in
  `index.html` and can be edited directly — search for the text you want to
  change.
- **Video:** replace `assets/video/hero-bg.mp4` and `hero-bg.webm` (keep
  both formats for browser compatibility) and `assets/video/hero-poster.jpg`
  (a still frame shown before the video loads).

## Deployment

This is a static site — any static host works. A few common options:

**GitHub Pages**
1. Push this folder to a GitHub repository.
2. In the repo's Settings → Pages, set the source to the branch/folder
   containing `index.html` (root, or a `/docs` folder if you prefer).
3. Point your domain's DNS at GitHub Pages per GitHub's custom-domain docs,
   or use the default `*.github.io` URL.

**Netlify / Vercel**
1. Connect the GitHub repository, or drag-and-drop this folder in their
   dashboard.
2. No build command is needed — set the publish directory to the repo root.
3. Add your custom domain in the host's domain settings.

**Traditional web host (cPanel, FTP, etc.)**
1. Upload the entire contents of this folder (index.html and the assets/
   folder) to your host's public web directory (often `public_html` or
   `www`).
2. Point your domain at that hosting account per your host's instructions.

In every case, keep `index.html` and the `assets/` folder together with the
same relative paths shown above — the page references `assets/video/...`
and `assets/images/...` as relative paths, so as long as that folder
structure is preserved, the site will work wherever it's hosted.

## A note on Claude Artifacts

At an earlier stage of building this site, a temporary preview link was
published using Claude's Artifact tool so the animation and layout could be
reviewed in a browser. That link is a preview convenience only — it is not
part of this repository, it is not referenced anywhere in this code, and
the site does not depend on it in any way. This repository is fully
self-contained and can be deployed to any host independent of Claude.
