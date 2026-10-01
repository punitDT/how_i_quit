# Quit Smoke — Brand Spec

Direction: **Quit Smoke × SpaceX.** Aerospace-industrial, achromatic, cinematic.
The product's two rules are written the way a spacecraft is labelled: uppercase, tracked, unadorned.

Contract source: `design-systems/spacex/tokens.css` + `DESIGN.md`. All 56 brand tokens are
pasted verbatim into the first `<style>` block of every file. One token is added —
`--scrim` — because DESIGN.md §2 mandates a `rgba(0,0,0,0.5)` photography overlay that the
shipped token block does not carry.

---

## 1. Color

Zero hues. The palette is achromatic in both themes; state is carried by **fill, outline,
inversion, weight and size** — never by colour.

### Dark — the default (SpaceX-native)

| Role | Value | Use |
|---|---|---|
| `--bg` | `#000000` | the void; every screen |
| `--surface` | `#000000` | identical to `--bg` — there are no cards, only hairlines |
| `--fg` | `#f0f0fa` | spectral white, never `#ffffff` for text |
| `--muted` | `rgba(240,240,250,0.7)` | labels, captions, helper copy |
| `--border` | `rgba(240,240,250,0.35)` | the ghost border — the only visible edge |
| `--border-soft` | `rgba(240,240,250,0.1)` | row separators, unselected segments |
| `--accent` | `#f0f0fa` | filled primary CTA, timer numerals, progress |
| `--accent-on` | `#000000` | text on an accent fill |
| `--scrim` | `rgba(0,0,0,0.55)` | photography legibility (DESIGN.md §2) |

### Light — the inversion, not a mirror

| Role | Value |
|---|---|
| `--bg` / `--surface` | `#FFFFFF` |
| `--fg` | `#0A0A0A` |
| `--muted` | `rgba(10,10,10,0.62)` — 5.65:1 on white |
| `--border` | `rgba(10,10,10,0.30)` |
| `--border-soft` | `rgba(10,10,10,0.10)` |
| `--accent` | `#0A0A0A` · `--accent-on` `#FFFFFF` |

Photography frames stay dark in both themes, exactly as on the SpaceX site. A dark photo
panel inside a white screen is intentional contrast, not a bug.

Derived tones use `color-mix(in oklch, var(--fg) N%, transparent)` so every ghost surface
inverts correctly with the theme. No second palette.

---

## 2. Typography

One family: **D-DIN**, two weights, no exceptions (DESIGN.md §7 *Don't*: "Use D-DIN
exclusively"). D-DIN is proprietary, so the artifacts declare `@font-face` aliases that
resolve `local("D-DIN")` first and fall back to a locally-hosted DIN-heritage webfont —
**Saira** (`assets/fonts/saira-var.woff2`, SIL Open Font License 1.1). Nothing is hotlinked.

| Role | Face | Size | Weight | Tracking | Case |
|---|---|---|---|---|---|
| Timer / numerals | `D-DIN-Cond` (Saira Condensed) | `clamp(52px, 20vw, 92px)` | 700 | 0.01em | tabular |
| Screen title | `D-DIN-Bold` | 26–32px | 700 | 0.02em | UPPER |
| Section label | `D-DIN` | 10px | 400 | 0.1em | UPPER |
| Button | `D-DIN-Bold` | 12–13px | 700 | 0.12em | UPPER |
| Body | `D-DIN` | 15–16px | 400 | 0.01em | sentence |
| Caption | `D-DIN` | 12px | 400 | 0.06em | UPPER |

`D-DIN-Cond` is an instrument face, reserved for the timer readout and for Live Activity /
widget readouts where horizontal space is tight. It is the same superfamily at a narrower
width axis, not a second brand.

**The one exception to universal uppercase** is running body copy. Explanatory sentences
(onboarding paragraphs, helper text) stay sentence-case at 0.01em tracking, because 10px
letter-spaced caps in a paragraph is unreadable and the goal outranks the styling rule.
Every label, title, button, stat and numeral remains uppercase.

---

## 3. Shape and depth

- **Radius:** `4px` on rows, cards and fields. `32px` (pill) on buttons — the ghost button
  is the only rounded element in the brand, so it stays the only pill.
- **Elevation:** none. `--elev-raised: none`. No shadows anywhere, ever.
- **Surfaces:** a 1px hairline plus at most a 3–4% `--fg` tint. Depth comes from the
  10-step grey ramp, not from blur, glass or gradients.
- **No cards-within-cards.** One hairline level, then rows.

---

## 4. The seven rules

1. **One filled accent per screen.** The centre craving pill in the tab bar is the fill on
   the five tab screens. Inside Craving Escape the tab bar is hidden and the step's own
   primary CTA takes the fill. Never both.
2. **State = fill vs outline vs inversion.** Done days are solid, today is ringed, future
   days are a bare hairline. Text and icons always carry the meaning too.
3. **Uppercase + positive tracking** on all chrome. No negative tracking anywhere.
4. **Zero shadows, zero glass, zero gradients** — except the black photography scrim that
   DESIGN.md §2 requires.
5. **Photography is the whitespace.** Full-bleed, grayscale, 55–78% black scrim, type
   sitting directly on the frame with no card behind it.
6. **44px minimum targets**, 48dp on Android. Primary actions live in the bottom 40% of
   the screen; the tab bar floats above the home indicator.
7. **Motion is 150–350ms**, spring easing, and always disabled under
   `prefers-reduced-motion: reduce`.

---

## 5. The one flourish

The **running timer**: condensed tabular numerals at up to 92px, unchanging in width
(`tabular-nums`, no layout shift), with a 2px accent rule filling edge to edge beneath it.
It is the only kinetic element in the product, and it is the reason the app is called
Quit Smoke.

---

## 6. Data honesty

Demo state is internally consistent and stated on screen: **20 cigarettes/day, $12.00 a
pack → $12.00 a day → $36.00 over 3 smoke-free days, $360.00 a month, $4,380.00 a year.**
No other number appears that did not come from the brief. Empty, loading and success
states are designed, not implied.

---

## 7. Files

| File | What it is |
|---|---|
| `index.html` | Launcher + light/dark priority-screen collage |
| `mobile-onboarding.html` | 8-step onboarding: the problem → the two tools → the fix → plan |
| `mobile-app.html` | Home, Smoke-Free Plan, Craving Escape, Progress, Trigger Zones, Settings |
| `mobile-system-surfaces.html` | Widgets, Live Activity, Dynamic Island, lock screen, notifications, states |
| `brand-spec.md` | This file |

---

## 8. Image credits

Real photography, downloaded and stored in the project at `assets/photos/`. No hotlinks,
no generated or look-alike substitutes, no cigarette, smoke or lung imagery anywhere.

| File | Subject | Source | Licence |
|---|---|---|---|
| `street-corner-night.jpg` | Street corner at night in the rain | Wikimedia Commons, Rio de Janeiro | CC0 (public domain) |
| `pedestrian-street.jpg` | Pedestrian street, Athens | Wikimedia Commons, julesvernex2 | CC BY-SA 4.0 |
| `city-sunrise.jpg` | Chicago skyline at sunrise | Wikimedia Commons | CC BY-SA 4.0 |

All three are rendered grayscale at reduced brightness: they are atmosphere, not content,
and the achromatic palette admits no colour.
