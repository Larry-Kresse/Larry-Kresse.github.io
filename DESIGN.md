# Rev B design system — "LKR-2026"

This document defines the **Rev B** PCB design language used by `index2.html` and
`projects/tesseract_v1_2.html`. Use it when migrating the remaining project pages
(`psu_ui.html`, `beacon_V1–V3.html`, …) so every page reads as one fab run.

The conceit: every page is a printed circuit board. The page body is the solder
mask, chrome is silkscreen, accents are exposed copper/ENIG gold, and the footer
is a card-edge connector.

---

## 1. Colors

Declared as CSS custom properties in `:root` on every page:

```css
:root {
    --mask:        #0D674E;  /* board background — PCB solder-mask green */
    --mask-raised: #12745A;  /* raised cards (stackup layers, modules, detail views) */
    --mask-edge:   #08402F;  /* page background behind the board + mount-hole bores */
    --copper:      #c88a3c;  /* copper accents: borders, labels, small silkscreen */
    --gold:        #d9b25f;  /* ENIG gold: links, pads, button borders */
    --gold-bright: #ecd9a0;  /* headings, strong text, hover states */
    --silk:        #e8e6df;  /* silkscreen white — primary body text */
    --silk-dim:    #cfd4c9;  /* secondary body text (on green; index2 uses #a9aaa2) */
    --led-green:   #4cd964;  /* status LED only */
    --mono:        "Courier New", Courier, monospace;
    --heading:     "Raleway", Helvetica, sans-serif;
}
```

Rules of thumb:

- `--mask` / `--mask-raised` / `--mask-edge` are three shades of the same green;
  keep that relationship if the palette ever shifts (raised ≈ lighter, edge ≈ darker).
- The green `#0D674E` is deliberate — it matches the Rev A homepage so both
  revisions feel like siblings. Do not substitute black; that was the pre-revision look.
- Copper for structure and small labels, gold for interactive things, gold-bright
  for headings. Body text is always silkscreen white, never gold.

## 2. Typography

- **Headings:** Raleway 800 (h1/h2) and 600 (h3), uppercase, letter-spaced
  (`0.08–0.14em`). Loaded from Google Fonts (`Raleway:600,800`).
- **Silkscreen labels** (legends, designators, sheet numbers, pin numbers,
  buttons, footer): Courier New, uppercase, small (`0.66–0.78rem`), wide
  letter-spacing (`0.14–0.3em`), copper colored.
- **Body copy:** Helvetica/Arial stack, 16px base, `line-height: 1.7`,
  constrained to `max-width: 46rem`.
- Inline links in body copy: gold with a dotted copper underline; solid + brighter
  on hover.
- Beware wide letter-spacing on mobile: it overflows narrow viewports. Every page
  carries a `max-width: 720px` media query that tightens legend/tagline
  font-size and letter-spacing. Also: separators like `/` between words must have
  real spaces around them or the line can't wrap.

## 3. Board-frame structure

Every page is wrapped in a single `.board` element sitting on the dotted-grid
page background:

```
body                      ← --mask-edge + 24px radial-gradient copper dot grid
└── .board                ← --mask, 2px copper border, 14px radius, overflow hidden
    ├── .mount-hole ×4    ← corner circles: --mask-edge fill, 4px gold ring (tl/tr/bl/br)
    ├── .board-legend     ← centered mono fab line across the top
    ├── header.hero       ← status LED, refdes line, h1, tagline, hero nav
    ├── section ×N        ← content sheets, separated by dashed copper rules
    └── (last section ends with) .footer-note + .gold-fingers
```

Fixed furniture, top to bottom:

- **Board legend** — mono, centered:
  `LKR-2026 · <page id> · <page name> · Rev B` (landing page adds `· RoHS`).
- **Status LED** — blinking green dot + mono caption. Make the caption
  page-appropriate: landing page `PWR OK — SYSTEM NOMINAL`, Tesseract
  `SPINDLE OK — 100 LBS ROTATING`.
- **Refdes line** — mono kicker above the h1: `U100 — MULTI-DISCIPLINARY ENGINEER`
  on the landing page; `MOD.xx — <descriptor>` on project pages.
- **Hero nav** — `.pad-link` buttons: mono, copper 1px border, transparent fill;
  hover inverts to solid gold with green text. Project pages include
  `← Main Board` (to `../index2.html`).
- **Section separators** — `1px dashed rgba(200, 138, 60, 0.35)`.
- **Section headers** (`.sec-head`) — mono sheet number + Raleway h2 + a dashed
  copper rule stretching to the right margin.
- **Gold fingers** — full-bleed repeating-gradient strip at the very bottom
  (negative side margins to escape board padding), 2px copper top border.
- **Footer note** — mono copyright + cross-links, directly above the fingers.

Component patterns available (copy from the existing pages):

- `.layer` — stackup rows (landing page expertise section).
- `.module` — project cards with fiducial crosshair + `MOD.xx` refdes.
- `.sub-section` — raised detail-view cards on project pages (see §5).
- `.gallery-grid` — copper-bordered thumbnails (220px, object-fit cover) that
  link to the full-size image.
- `.pinout` — contact list rows: `PIN n` + gold pad swatch + icon + link.
- `.figure` / `.hero-img` — single full-width images, copper border, 5px radius.

Pages are **standalone**: all CSS inline in `<head>`, no shared stylesheet, no
jQuery/template scripts. Only external deps are Font Awesome
(`assets/css/fontawesome-all.min.css`) and the Raleway Google-Fonts stylesheet.

## 4. MOD numbering

Project pages carry the module designator assigned by their card on the
`index2.html` "Assembly Modules" section:

| Designator | Project              | Rev A page                  | Rev B page                    |
| ---------- | -------------------- | --------------------------- | ----------------------------- |
| `MOD.01`   | Tesseract 3D Display | `projects/tesseract_v1.html`| `projects/tesseract_v1_2.html`|
| `MOD.02`   | Keithley 2304A UI    | `projects/psu_ui.html`      | `projects/psu_ui_2.html`      |
| `MOD.03`   | Celestial Navigator  | `projects/beacon_V3.html`   | —                             |

The designator appears in three places on a project page: the board legend, the
hero refdes line, and (on the landing page) the module card. New projects take
the next free `MOD.xx`. The landing page itself is `U100`.

## 5. Sheet and detail naming

Sections are numbered like schematic sheets, subsections like detail views on a
fab drawing:

- **Sheets:** each top-level section header carries `SHT n/N` (mono, copper),
  where `N` is the page's total. The landing page runs SHT 1/4 (About) through
  SHT 4/4 (Edge Connector). Renumber when adding sections.
- **Details:** subsections within a sheet are lettered `DET A`, `DET B`, `DET C`…
  with a short mono descriptor, e.g. `DET A — ARCHITECTURE`,
  `DET B — LED LAYOUT`, `DET C — ENCLOSURE` (Tesseract page). Each renders as a
  raised `.sub-section` card with the DET tag above its h3.
- Section vocabulary is PCB-flavored where it fits naturally: "Layer Stackup"
  (expertise), "Assembly Modules" (projects), "Edge Connector" (contact). Don't
  force it — "Background" is fine as "Background".

## 6. File conventions

- Rev B pages live **alongside** their Rev A originals with a `_2` suffix
  (`index2.html`, `tesseract_v1_2.html`); originals are left untouched.
- Every Rev B page cross-links: project pages → `../index2.html` ("Main Board")
  and their own Rev A; the landing page footer → `index.html` ("View Rev A").
- Asset paths from `projects/` are relative: `images/…` for project photos,
  `../assets/…` for shared assets.

## 7. Known content gotcha

`projects/tesseract_v1.html` (Rev A) contains three trailing sections
("Electronic Hardware Design", "Board Bring-up…", "Bonus content: project case
assembly") that are duplicated from `psu_ui.html` and are not about the
Tesseract. The Rev B Tesseract page intentionally omits them. When migrating
other Rev A pages, check for similar copy-paste leftovers before carrying
content over.
