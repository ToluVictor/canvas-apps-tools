# Design Spec Formats

Reference file for Phase 4 Step 1. Read only the section matching the resolved mode.

---

## Replicate Mode

Extract directly from the screenshot. Be precise — measure colors, estimate pixel dimensions from proportions.

```
=== DESIGN SPEC (extracted from screenshot) ===
Mode: replicate
Paste target: [a/b/c]
Screen name: [ScreenName]
Control style: [classic / modern / auto→classic]

COLORS
  Background:       RGBA(249,250,251,1)
  Sidebar bg:       RGBA(255,255,255,1)
  Primary text:     RGBA(17,24,39,1)
  Secondary text:   RGBA(107,114,128,1)
  Accent:           RGBA(234,88,12,1)
  Active nav bg:    RGBA(255,247,237,1)
  Border:           RGBA(229,231,235,1)
  Input border:     RGBA(209,213,219,1)
  [add all colors visible in the mockup]

MEASUREMENTS
  Canvas:           1366×768 (tablet)
  Reference canvas width: ~[N]px  (estimate from screenshot — total width of the mockup image)
  Scale factor:           [1366/N] (apply to sidebar, card, column widths — see reference/sizing-reference.md §6)
  Sidebar width:    [ref × scale]px  (e.g. ~860px ref → 130 × 1.585 = 200px at 1366px)
  Top bar height:   64px
  Tab bar height:   42px
  Form card width:  [ref × scale]px  (e.g. ~860px ref → 393 × 1.585 = 620px at 1366px)
  Form card padding: 24px all sides
  Gap between form groups: 20px
  Input height:     40px
  Button height:    36px
  Nav item height:  40px
  [add all key measurements]

TYPOGRAPHY
  Logo:             13px, Lato Black, Bold
  Page title:       14px, Lato Black, Bold
  Card title:       13px, Lato Black, Bold
  Form labels:      11px, Lato, Semibold
  Input text:       11px, Lato, Regular
  Nav labels:       10px, Lato, Semibold (active) / Regular (inactive)
  Breadcrumb:       9px, Lato, Regular
  Button text:      11px, Lato, Normal
  [add all typography visible]

COMPONENT STATE DESCRIPTIONS
  Nav item (active):   orange bg tint (RGBA(255,247,237,1)), orange icon, orange label, semibold
  Nav item (default):  transparent bg, gray icon, gray label, regular weight
  Tab (active):        dark text, 2px underline at bottom
  Tab (default):       gray text, no underline
  Button "Save":       outlined — white fill, gray border
  Button "Continue":   filled dark — RGBA(17,24,39,1) fill, white text
  Button "Cancel":     text-only, underlined, no fill
  Input (focused):     border thickens to 2px, darkens to primary text color
  Input (hover):       border lightens slightly
  Checkbox (checked):  fill = primary text color, white checkmark
  [add all interactive state descriptions]

MODIFICATIONS (user-requested changes — apply these over the extracted spec above)
  [list each modification the user requested, or omit this section if none]
```

---

## Improvement Mode

```
=== DESIGN SPEC (improvement mode) ===
Mode: improvement
Paste target: [a/b/c]
Screen name: [ScreenName]
Control style: [classic / modern / auto→classic]

DESIGN TOKENS
─────────────────────────────────────────────────────────
PALETTE
  Background:       RGBA(...)   ← base screen/page bg
  Surface:          RGBA(...)   ← card/panel bg (slightly offset from bg)
  Border:           RGBA(...)   ← dividers, input borders
  Accent-Primary:   RGBA(...)   ← CTA buttons, active states, focus rings
  Accent-Secondary: RGBA(...)   ← optional 2nd accent (blank if none)
  Text-Primary:     RGBA(...)   ← headings, primary labels
  Text-Secondary:   RGBA(...)   ← metadata, helper text, placeholders
  Text-OnAccent:    RGBA(...)   ← text on accent-colored surfaces

TYPOGRAPHY
  Font:           [Lato / Segoe UI / Verdana / Georgia / Courier New]
  Heading-Size:   [PA size integer]  ← section titles, card headers
  Body-Size:      [PA size integer]  ← form labels, list items, body copy
  Caption-Size:   [PA size integer]  ← timestamps, metadata, helper text
  Heading-Weight: [Bold / Semibold]
  Body-Weight:    [Normal / Semibold]

DENSITY
  Profile:   [compact / balanced / generous]
  LayoutGap: [integer]  ← gap between siblings in AutoLayout containers
  PaddingH:  [integer]  ← PaddingLeft + PaddingRight on containers
  PaddingV:  [integer]  ← PaddingTop + PaddingBottom on containers

RADIUS
  Container:   [integer]  ← GroupContainer cards, panels
  Interactive: [integer]  ← buttons, inputs (same as Container or half)

STATE COLORS (all derived from Accent-Primary)
  HoverFill:     RGBA(...)  ← accent at ~8% opacity
  PressedFill:   RGBA(...)  ← accent at ~15% opacity
  FocusedBorder: RGBA(...)  ← accent-primary at full opacity
  DisabledFill:  RGBA(...)  ← near-background, very low contrast
  DisabledColor: RGBA(...)  ← text-secondary at ~38% opacity
─────────────────────────────────────────────────────────

PART A — PRESERVE (extracted from screenshot)
  Sections (keep in this order):
    1. [Section name and purpose]
    2. [Section name and purpose]
    [...]
  Field labels to keep verbatim:
    - "[exact label text]"
    - "[exact label text]"
  Navigation items (keep verbatim):
    - "[item name]"
    [...]
  Control purposes:
    - [controlName]: [what it does]
    [...]

PART B — IMPROVE (apply these; replace anything from PART A that conflicts)
  COLORS
    Background:     RGBA(249,250,251,1)
    Sidebar bg:     RGBA(255,255,255,1)
    Primary text:   RGBA(17,24,39,1)
    [full palette based on user's color answer]

  MEASUREMENTS
    [standardized values from reference/sizing-reference.md defaults]

  TYPOGRAPHY
    Font: Lato throughout
    [full type scale]

  CONTROL UPGRADES
    [list any controls being upgraded, e.g. "DropDown → Classic/ComboBox@2.4.0 with IsSearchable"]

  LAYOUT FIXES
    - Add LayoutMinWidth: =0 and LayoutMinHeight: =0 to all containers
    - Add DropShadow: =DropShadow.Light to card containers
    - Add 6px radius to all inputs and buttons
    - Add proper hover/pressed/focus states to all interactive controls

  SPECIFIC IMPROVEMENTS (from user's optional answer, if any)
    [list any user-specified improvements beyond the standard palette/spacing/control upgrades]

  COMPONENT STATE DESCRIPTIONS
    [same format as replicate mode, but using PART B colors]
```

---

## Text-Only Mode

```
=== DESIGN SPEC (synthesized from description + user answers) ===
Mode: text-only
Paste target: [a/b/c]
Screen name: [ScreenName]
Layout pattern: [from user's answer]
Primary purpose: [from user's answer]
Control style: [classic / modern / auto→classic]

DESIGN TOKENS
─────────────────────────────────────────────────────────
PALETTE
  Background:       RGBA(...)   ← base screen/page bg
  Surface:          RGBA(...)   ← card/panel bg (slightly offset from bg)
  Border:           RGBA(...)   ← dividers, input borders
  Accent-Primary:   RGBA(...)   ← CTA buttons, active states, focus rings
  Accent-Secondary: RGBA(...)   ← optional 2nd accent (blank if none)
  Text-Primary:     RGBA(...)   ← headings, primary labels
  Text-Secondary:   RGBA(...)   ← metadata, helper text, placeholders
  Text-OnAccent:    RGBA(...)   ← text on accent-colored surfaces

TYPOGRAPHY
  Font:           [Lato / Segoe UI / Verdana / Georgia / Courier New]
  Heading-Size:   [PA size integer]  ← section titles, card headers
  Body-Size:      [PA size integer]  ← form labels, list items, body copy
  Caption-Size:   [PA size integer]  ← timestamps, metadata, helper text
  Heading-Weight: [Bold / Semibold]
  Body-Weight:    [Normal / Semibold]

DENSITY
  Profile:   [compact / balanced / generous]
  LayoutGap: [integer]  ← gap between siblings in AutoLayout containers
  PaddingH:  [integer]  ← PaddingLeft + PaddingRight on containers
  PaddingV:  [integer]  ← PaddingTop + PaddingBottom on containers

RADIUS
  Container:   [integer]  ← GroupContainer cards, panels
  Interactive: [integer]  ← buttons, inputs (same as Container or half)

STATE COLORS (all derived from Accent-Primary)
  HoverFill:     RGBA(...)  ← accent at ~8% opacity
  PressedFill:   RGBA(...)  ← accent at ~15% opacity
  FocusedBorder: RGBA(...)  ← accent-primary at full opacity
  DisabledFill:  RGBA(...)  ← near-background, very low contrast
  DisabledColor: RGBA(...)  ← text-secondary at ~38% opacity
─────────────────────────────────────────────────────────

COLORS  [source: user selected "[palette answer]" — using standard light theme palette]
  [full palette, marked as "proposed"]

MEASUREMENTS  [source: reference/sizing-reference.md defaults]
  [standard values]

TYPOGRAPHY  [source: skill defaults — Lato font system]
  [full type scale]

COMPONENT STATE DESCRIPTIONS  [source: skill best practices]
  [standard state descriptions for the control types in this screen]
```
