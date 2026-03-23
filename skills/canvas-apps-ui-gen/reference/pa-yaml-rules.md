# Power Apps Canvas App YAML Rules

> **Output format note for canvas-ui-gen skill**: Unlike the original web app that output raw YAML, this skill MUST wrap all output in a ` ```yaml ` code block for clean copy-paste in Claude Code chat.

---

## CRITICAL FORMAT RULES

- Root element starts with `- ` (dash + space)
- Every property value starts with `=` prefix: `Height: =205`, `Text: ="Hello"`
- Children arrays use `- ` (dash + space) for each child control
- Multiline expressions use `|-` with `=` on the next indented line
- Use Galleries with `Table()` for repeating layouts — NEVER hardcode duplicate controls
- Include `DropShadow`, `RadiusBottomLeft/Right`, `RadiusTopLeft/Right` properties on all containers

### The Colon/Hash Rule — Most Common PA1001 Error

**NEVER** use `:` (colon) or `#` (hash) inside single-line expressions. They break YAML parsing. ANY expression containing a colon or hash MUST use multiline `|-` format. This includes:
- Text strings like `"Goal: 5"` or `"Item #3"`
- String interpolation like `$"Goal: {value}"`
- SVG strings containing `xmlns='http://...'`
- Any expression with URLs

**CORRECT** (colon inside string → multiline):
```yaml
Text: |-
  =$"Goal: {ThisItem.Goal}"
```

**WRONG** (causes PA1001):
```yaml
Text: =$"Goal: {ThisItem.Goal}"
```

Before outputting any single-line expression: does it contain `:` or `#`? If YES → make it multiline with `|-`.

---

## GALLERY WRAP COUNT

Galleries have a `WrapCount` property for multi-row/column layouts. NEVER create multiple galleries when one gallery with WrapCount can do the same job.

### Horizontal Gallery wrap:
- Items fill COLUMN BY COLUMN (top → bottom), then next column (left → right)
- `WrapCount` = how many items stack vertically before wrapping to the next column
- `TemplateSize` sets the WIDTH of each item (independent of WrapCount)
- Example: 10 items, WrapCount 2: columns are `[1,2]`, `[3,4]`, `[5,6]`, `[7,8]`, `[9,10]`

### Vertical Gallery wrap:
- Items fill ROW BY ROW (left → right), then next row (top → bottom)
- `WrapCount` = how many items sit horizontally before wrapping
- `TemplateSize` sets the HEIGHT of each item
- Example: 10 items, WrapCount 2: rows are `[1,2]`, `[3,4]`, `[5,6]`, `[7,8]`, `[9,10]`

> **`TemplateSize` is write-only.** Use it to set the item size, but never reference it in formulas as `Self.TemplateSize` — it is not readable. To read the effective size back in a formula, use `Self.TemplateHeight` (vertical gallery) or `Self.TemplateWidth` (horizontal gallery). Example: `Height: =Self.AllItemsCount * Self.TemplateHeight`.

### Key rule:
If a design shows items in a grid pattern (multiple rows AND columns), use ONE gallery with appropriate WrapCount. If rows have the same card structure, they MUST be a single gallery — different text/colors/values are just different records in `Table()`, not a reason for separate galleries.

---

## GALLERY IN SIDEBARS AND NAVIGATION

### TemplateSize — setter only, cannot be read back

`TemplateSize` sets the item size on the Gallery control. It cannot be referenced as `Self.TemplateSize` in formulas — it is write-only. To read the effective size in a formula, always use:
- `Self.TemplateHeight` — for vertical galleries (nav sidebars)
- `Self.TemplateWidth` — for horizontal galleries

```yaml
# CORRECT
Height: =Self.AllItemsCount * Self.TemplateHeight

# WRONG — Self.TemplateSize does not exist as a readable property
Height: =Self.AllItemsCount * Self.TemplateSize
```

### FillPortions and Height are mutually exclusive on Gallery

**Never set both `FillPortions: =1` and a static `Height` formula on the same gallery.** They contradict each other:
- `FillPortions: =1` → the parent layout system controls the gallery's height — do NOT also set `Height`
- Static `Height` formula → set `FillPortions: =0` (fixed size, no flex fill)

### Three sidebar nav gallery patterns

**Pattern A — Gallery fills remaining sidebar height (use by default)**

The navGallery takes all remaining space after the logo area. No spacer needed.
```yaml
- navGallery:
    Control: Gallery@2.15.0
    Variant: Vertical
    Properties:
      FillPortions: =1        # fills remaining height — do NOT also set Height
      TemplateSize: =48
      TemplatePadding: =0
      ShowScrollbar: =false
      LayoutMinHeight: =0
      LayoutMinWidth: =0
```
Logo area and footer (divider + profile): `FillPortions: =0` with static heights.

**Pattern B — Gallery is exactly as tall as its items (compact, no scroll)**

Use when the design has so few nav items that the gallery doesn't need to fill remaining space, and you want elements below to follow immediately after.
```yaml
- navGallery:
    Control: Gallery@2.15.0
    Variant: Vertical
    Properties:
      FillPortions: =0        # fixed size
      Height: =Self.AllItemsCount * (Self.TemplateHeight + Self.TemplatePadding)
      TemplateSize: =48
      TemplatePadding: =0
      ShowScrollbar: =false
      LayoutMinHeight: =0
      LayoutMinWidth: =0
```

**Pattern C — Nav items centered vertically in the sidebar (special design)**

Only use when the design explicitly shows nav icons grouped and centered in the middle of the sidebar height. Wrap the gallery in a centering container.
```yaml
- navCenterWrapper:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      FillPortions: =1
      LayoutDirection: =LayoutDirection.Vertical
      LayoutJustifyContent: =LayoutJustifyContent.Center
      LayoutMinHeight: =0
      LayoutMinWidth: =0
      DropShadow: =DropShadow.None
      RadiusBottomLeft: =0
      RadiusBottomRight: =0
      RadiusTopLeft: =0
      RadiusTopRight: =0
    Children:
      - navGallery:
          Control: Gallery@2.15.0
          Variant: Vertical
          Properties:
            FillPortions: =0
            Height: =Self.AllItemsCount * Self.TemplateHeight
            TemplateSize: =48
            TemplatePadding: =0
            ShowScrollbar: =false
            LayoutMinHeight: =0
            LayoutMinWidth: =0
```

### ⚠️ Never add a `sidebarSpacer` between navGallery and footer

A `sidebarSpacer` with `FillPortions: =1` alongside a `navGallery` with `FillPortions: =1` splits remaining space 50/50 — the gallery only gets half the sidebar. Use Pattern A instead: `navGallery` takes `FillPortions: =1` alone, and there is no spacer.

---

## SIZING AND VIEWPORT — CRITICAL

Default Power Apps viewport width: **1366px**. Content must never be cut off unless there is an explicit scroll bar.

### Use flexible sizing (Stretch + FillPortions + small LayoutMin values) for:
- Labels with dynamic text (titles, values, descriptions)
- Content containers that fill available space
- Card templates inside galleries
- Any element that should adapt to screen size

### Use static sizing (fixed Width/Height, FillPortions: =0) for:
- Icons and icon containers (e.g., `Width: =35, Height: =35`)
- Custom buttons with specific dimensions
- Status pills and badges
- Progress bars and dividers/separators
- Gallery `TemplateSize`

### LayoutMinWidth / LayoutMinHeight — MOST CRITICAL DEFAULT TRAP

**Power Apps DEFAULT values are `LayoutMinWidth: =250` and `LayoutMinHeight: =100`.**

If you do NOT explicitly set these, Power Apps applies these defaults automatically, which WILL cause overflow and content being cut off in most layouts.

**You MUST explicitly set `LayoutMinWidth` and `LayoutMinHeight` on EVERY container and control.**

Typical values:
- Label `LayoutMinHeight`: `=20` to `=30` (depending on font size)
- Label `LayoutMinWidth`: `=50` to `=100` (depending on expected character count)
- Icon container: `LayoutMinWidth/Height` matches its fixed size
- Stretched child (`AlignInContainer.Stretch`): `LayoutMinWidth: =0` — it gets width from parent

**CRITICAL**: When a child has `AlignInContainer: =AlignInContainer.Stretch`, its `LayoutMinWidth` must be `=0`. A large `LayoutMinWidth` on a stretched item causes overflow.

### Rules:
- NEVER set a child's fixed Width that could overflow its parent
- Default to Stretch + FillPortions over fixed sizes
- If content is cut off: reduce `LayoutMinWidth` to `=0` and use Stretch or FillPortions
- Nothing should be cut off without an explicit `LayoutOverflowY: =LayoutOverflow.Scroll`

---

## CONTAINER DIRECTION LOGIC

For each group of children, ask: "Do these sit SIDE BY SIDE, or are they STACKED?"

- Side by side (left to right) → `LayoutDirection: =LayoutDirection.Horizontal`
- Stacked (top to bottom) → `LayoutDirection: =LayoutDirection.Vertical`
- Both directions needed → NEST containers

Example (card with icon row on top, title below, value below that):
```
Root = Vertical container (rows stack top to bottom)
  ├── Child 1: Horizontal container (icon left, percentage right — side by side)
  ├── Child 2: Title label
  ├── Child 3: Value label
  └── Child 4: Comparison label
```

Complex layouts require multiple levels of nesting — a horizontal inside a vertical inside a horizontal is normal.

### Layout Decomposition Process (for complex screens)

When a layout has multiple sections (header, sidebar, content areas), determine the root direction using this rule:

**Ask: "How can I break down this layout along ONLY ONE axis so that no individual section is split?"**

- Try the horizontal axis first (horizontal lines cutting the layout). If any section spans multiple horizontal slices, horizontal splitting won't work.
- Try the vertical axis (vertical lines cutting the layout). If every section fits cleanly in one column, use a horizontal root container.
- The axis that works cleanly = the root container direction.

Then recurse: apply the same question to each child container.

Example — A layout with a sidebar + (header, overview, tasks stacked vertically):
1. A vertical line splits it cleanly into 2 columns (sidebar left, content right) → root is Horizontal
2. Inside the content column, horizontal lines split it into header/overview/tasks → content container is Vertical
3. Inside the tasks row, a vertical line splits it into tasks/inspections → tasks container is Horizontal

---

## ALIGNINCONTAINER — ALWAYS SET EXPLICITLY ON EVERY CHILD

Every child inside an AutoLayout container **MUST** have `AlignInContainer` set explicitly.
Never rely on the parent's `LayoutAlignItems` as the default.

**Why:** If the parent has `LayoutAlignItems: Stretch` (common default), any child without its own explicit `AlignInContainer` silently stretches on the cross-axis, overriding its fixed size and breaking circles, icons, and badges.

### Cross-axis override rule

| Parent direction | Cross-axis | What `Stretch` overrides |
|---|---|---|
| `Horizontal` | Height | Child's fixed `Height` |
| `Vertical` | Width | Child's fixed `Width` |

### Decision guide — what to set on each child

Identify the parent's `LayoutDirection` first — it determines which axis `AlignInContainer` controls:
- **Horizontal parent** → AlignInContainer controls **height** allocation
- **Vertical parent** → AlignInContainer controls **width** allocation

| What the child should do | Set on child |
|---|---|
| Fill the full cross-axis (labels, content areas, inputs) | `AlignInContainer: =AlignInContainer.Stretch` |
| Labels needing vertical centering in a **horizontal** parent | `AlignInContainer: Stretch` (fills height) + `VerticalAlign: Middle` on the label — **NOT Center** |
| Labels needing horizontal centering in a **vertical** parent | `AlignInContainer: Stretch` (fills width) — **NOT Center** unless the label has an intentionally narrower explicit `Width` |
| Keep its own fixed size, centered on cross-axis | `AlignInContainer: =AlignInContainer.Center` |
| Keep its own fixed size, pinned to start | `AlignInContainer: =AlignInContainer.Start` |
| Keep its own fixed size, pinned to end | `AlignInContainer: =AlignInContainer.End` |

> **ANTI-PATTERN**: Do NOT use `AlignInContainer: Center` on a label to achieve vertical centering. A centered label only occupies its natural height (~27px for Size 11) and creates gaps above/below in taller rows. Use `AlignInContainer: Stretch` + `VerticalAlign: Middle` instead. `AlignInContainer` controls how much **space** the control occupies — it is not a text alignment property.

### Circle avatar pattern (profile photos, thumbnails)

```yaml
- profileAvatar:
    Control: GroupContainer@1.5.0
    Variant: ManualLayout
    Properties:
      AlignInContainer: =AlignInContainer.Center   # explicit — preserves fixed Height in horizontal row
      FillPortions: =0
      Height: =28
      Width: =28                                   # Height = Width for a circle
      RadiusBottomLeft: =14                        # = Height ÷ 2
      RadiusBottomRight: =14
      RadiusTopLeft: =14
      RadiusTopRight: =14
      DropShadow: =DropShadow.None
```

Rules for circles:
- `Height = Width` (same value — square base)
- All four radius values = `Height ÷ 2` (produces a true circle)
- `AlignInContainer: Center` — always explicit, never Stretch

---

## FIELD GROUP CONTAINER HEIGHT — ALWAYS CALCULATE FROM CHILDREN

AutoLayout containers that group a label with an input (field groups) MUST have an explicit
`Height` formula summing their children. Never leave Height unset on a `FillPortions: =0` container —
PA Studio will default it to 200px instead of sizing to content.

**Labels must have `AutoHeight: =true`** so their `.Height` can be referenced in the parent's formula.

### Complete Height formula (vertical container)

```
Height = PaddingTop + PaddingBottom
       + sum of each child's effective height
       + (number_of_gaps × LayoutGap)    ← gaps = number of children - 1

Where each child's effective height =
    child.Height + (2 × Max(child.BorderThickness, child.FocusedBorderThickness))

PA IMPORTANT: Power Apps Height/Width properties do NOT include border rendering.
Borders render OUTSIDE the allocated size. A container's own BorderThickness does NOT
affect its own Height property — but when a PARENT sums this container as a child, it
must add 2 × Max(BorderThickness, FocusedBorderThickness) to account for the border
that renders outside the allocated Height.
```

### Pattern: label (no border) + text input (border=1, focusedBorder=2)
```yaml
- fieldGroup:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      FillPortions: =0
      LayoutDirection: =LayoutDirection.Vertical
      LayoutGap: =6
      Height: =lblField.Height + (inputField.Height + 2 * 2) + 6
      #                                                  ↑↑↑
      #              2 × FocusedBorderThickness(=2) adds 4px outside the input's Height
    Children:
      - lblField:
          Control: Label@2.5.1
          Properties:
            AutoHeight: =true          # required — lets Height be referenced in parent formula
            AlignInContainer: =AlignInContainer.Stretch
            FillPortions: =0
      - inputField:
          Control: Classic/TextInput@2.3.2
          Properties:
            AlignInContainer: =AlignInContainer.Stretch
            FillPortions: =0
            Height: =40
            BorderThickness: =1
            FocusedBorderThickness: =2
```

### Pattern: container WITH padding (no border on container itself)
```yaml
# 3 children, gap=8, padding top/bottom=16
# If any child has a border, add 2 × Max(child.BorderThickness, child.FocusedBorderThickness) per bordered child
Height: =16 + 16 + child1.Height + child2.Height + child3.Height + (2 * 8)
```

### Pattern: label + multiple sub-items (e.g., recommended category list, gap=4)
```yaml
# 4 children → 3 gaps at 4px each
Height: =lblTitle.Height + lbl1.Height + lbl2.Height + lbl3.Height + (3 * 4)
```

### Horizontal rows (children side by side, not stacked)
For horizontal containers, Height = the tallest child's height + top/bottom padding + 2×border.
Set a fixed Height equal to the known tallest child:
```yaml
Height: =32   # matches the radio/checkbox/button height in the row (no border/padding on this row)
```

### FillPortions vs. explicit Height — mutually exclusive on the same axis

| Situation | What to do |
|---|---|
| Size to children (field group, fixed section) | `FillPortions: =0` + explicit `Height` formula |
| Fill remaining space proportionally | `FillPortions: =1` — explicit `Height` is **ignored** |
| Fill with minimum size constraint | `FillPortions: =1` + `LayoutMinHeight: =N` |
| Fill with maximum size constraint | `FillPortions: =1` + `LayoutMaxHeight: =N` |

**Rule:** Once `FillPortions > 0`, any static Height/Width on that axis is overridden by fill.
Use `LayoutMinHeight`/`LayoutMaxHeight` to constrain fill behavior.
- Vertical AutoLayout → `FillPortions` controls **Height**
- Horizontal AutoLayout → `FillPortions` controls **Width**

### What NOT to do
- Do NOT omit Height on `FillPortions: =0` containers — PA Studio defaults them to 200px
- Do NOT set both `FillPortions: =1` AND a calculated Height — FillPortions wins, Height is ignored
- Do NOT hardcode a static guess — compute from children + gaps + padding + border

---

## BORDER CLIPPING — PARENT MUST ADD PADDING FOR BORDERED CHILDREN

When a child inside a vertical AutoLayout container has a border, the border renders OUTSIDE the
child's Width property on the left and right sides. Without padding on the parent, this border
is visually clipped at the container edge.

**Rule:** The direct parent of a bordered child MUST have:
- `PaddingLeft` and `PaddingRight` ≥ `Max(child.BorderThickness, child.FocusedBorderThickness)`

This applies in both directions:
- Vertical container → children stack vertically → left/right borders can be clipped → add PaddingLeft/Right
- Horizontal container → children sit side by side → top/bottom borders can be clipped → add PaddingTop/Bottom

### Pattern
```yaml
- fieldGroup:       # vertical container: must add left/right padding
    Properties:
      PaddingLeft: =2    # = Max(inputField.BorderThickness=1, inputField.FocusedBorderThickness=2)
      PaddingRight: =2   # same
    Children:
      - inputField:
          Properties:
            BorderThickness: =1
            FocusedBorderThickness: =2    # ← this drives the padding requirement
```

### What NOT to do
- Do NOT leave a vertical field group container with `PaddingLeft: =0` if it contains a bordered input
- This is a recurring bug: the focused border appears cut off on left and right sides in the app

---

## LAYOUT OVERFLOW — SCROLL VS. HIDE

When child content may exceed the container's height, set `LayoutOverflowY` on the container:

| Property value | Effect |
|---|---|
| `=LayoutOverflow.Scroll` | Container becomes vertically scrollable when children exceed its height |
| `=LayoutOverflow.Hide` | Content is clipped at the boundary (default — no need to set explicitly) |

Use `LayoutOverflowY: =LayoutOverflow.Scroll` on any container that:
- Has a bounded Height (FillPortions or fixed)
- May contain more content than fits at runtime (form bodies, list panels, etc.)

```yaml
- scrollableBody:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      FillPortions: =1
      LayoutDirection: =LayoutDirection.Vertical
      LayoutOverflowY: =LayoutOverflow.Scroll
```

Note: Only set this on bounded containers. A container that auto-sizes to its children
(`Height = sum of children`) can never overflow itself — no overflow setting needed.
Horizontal scrolling (`LayoutOverflowX`) exists but is rarely used.

---

## CHILD SIZING RULES — MOST COMMON SOURCE OF ERRORS

Every child in an AutoLayout container MUST have an explicit size strategy on the PRIMARY AXIS. Without this, children balloon or collapse unpredictably.

### In a VERTICAL AutoLayout parent (primary axis = vertical):
Every child needs an explicit HEIGHT strategy:
- `FillPortions: =0` + static `Height: =value` → exact height
- `FillPortions: =1` (or higher) → flexible height from remaining space; use `LayoutMinHeight` as floor

### In a HORIZONTAL AutoLayout parent (primary axis = horizontal):
Every child needs an explicit WIDTH strategy:
- `FillPortions: =0` + static `Width: =value` → exact width
- `FillPortions: =1` (or higher) → flexible width from remaining space; use `LayoutMinWidth` as floor

### How FillPortions works:
1. Container allocates space to all static-sized children first (`FillPortions: =0`)
2. Remaining space is divided among flexible children proportionally
3. `LayoutMinWidth/Height` acts as floor — child won't shrink below this

Example (horizontal container, width 1000):
- Child A: `FillPortions: =0, Width: =200` → gets exactly 200
- Child B: `FillPortions: =2` → gets 2/3 of remaining 800 = ~533
- Child C: `FillPortions: =1` → gets 1/3 of remaining 800 = ~267

**NEVER** leave a child without a size strategy. Every child must have either a static Height/Width + `FillPortions: =0`, OR `FillPortions: =1+` with a `LayoutMinHeight/Width`.

---

## ALIGNINCONTAINER — SECONDARY AXIS POSITIONING

`AlignInContainer` positions a child on the SECONDARY axis (perpendicular to parent direction).

### In a HORIZONTAL parent (secondary axis = vertical):
- `AlignInContainer.Start` → child at TOP, uses static Height
- `AlignInContainer.Center` → child CENTERED vertically, uses static Height
- `AlignInContainer.End` → child at BOTTOM, uses static Height
- `AlignInContainer.Stretch` → child STRETCHES to fill parent height, uses `LayoutMinHeight` as floor
- `AlignInContainer.SetByContainer` → inherits from parent's `LayoutAlignItems`

### In a VERTICAL parent (secondary axis = horizontal):
- `AlignInContainer.Start` → child at LEFT, uses static Width
- `AlignInContainer.Center` → child CENTERED horizontally, uses static Width
- `AlignInContainer.End` → child at RIGHT, uses static Width
- `AlignInContainer.Stretch` → child STRETCHES to fill parent width, uses `LayoutMinWidth` as floor

---

## ALWAYS SET ALL LAYOUT PROPERTIES EXPLICITLY

Every `GroupContainer` with `Variant: AutoLayout` must have ALL of the following properties explicitly set — never omit any, even when the default matches the intent. Power Apps defaults `LayoutJustifyContent` to `Start`, `LayoutAlignItems` to `Start`, and `LayoutWrap` to `false` — but omitting them is a latent bug waiting to surface.

**Required on every AutoLayout GroupContainer:**
- `LayoutDirection` — `=LayoutDirection.Vertical` or `=LayoutDirection.Horizontal`
- `LayoutJustifyContent` — controls main-axis spacing (see table below)
- `LayoutAlignItems` — controls secondary-axis alignment (`Start`, `Center`, `End`, `Stretch`)
- `LayoutGap` — explicit px value; use `=0` if no gap
- `LayoutWrap` — `=false` unless wrapping is intentional

> **Note:** `ManualLayout` containers do NOT use `LayoutJustifyContent`, `LayoutAlignItems`, `LayoutGap`, or `LayoutWrap`. Only set these on `Variant: AutoLayout`.

### LayoutJustifyContent Reference

| Intent | Value |
|---|---|
| Items stack from top/left (most containers) | `=LayoutJustifyContent.Start` |
| Items centered on the main axis | `=LayoutJustifyContent.Center` |
| Items pushed to the bottom/right edge | `=LayoutJustifyContent.End` |
| First item at start, last item at end | `=LayoutJustifyContent.SpaceBetween` |
| Equal space around each item | `=LayoutJustifyContent.SpaceAround` |

### Pattern — Cancel Left, Action Buttons Right

```yaml
- actionRow:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      LayoutDirection: =LayoutDirection.Horizontal
      LayoutJustifyContent: =LayoutJustifyContent.SpaceBetween
      LayoutAlignItems: =LayoutAlignItems.Center
      LayoutGap: =0
      LayoutWrap: =false
  Children:
    - btnCancel: ...          # FillPortions: =0, fixed Width
    - rightButtonGroup:       # FillPortions: =1 — fills remaining space
        Control: GroupContainer@1.5.0
        Variant: AutoLayout
        Properties:
          FillPortions: =1           # responsive — takes all remaining space
          LayoutMinWidth: =200       # floor: never collapse below 200px
          LayoutDirection: =LayoutDirection.Horizontal
          LayoutJustifyContent: =LayoutJustifyContent.End
          LayoutAlignItems: =LayoutAlignItems.Center
          LayoutGap: =10
          LayoutWrap: =false
      Children:
        - btnSave: ...
        - btnContinue: ...
```

---

## FILLPORTIONS — STATIC VS. RESPONSIVE SIZING

In any AutoLayout container, `FillPortions` controls how a child sizes itself on the **primary axis** (width for horizontal, height for vertical):

| `FillPortions` | Behavior |
|---|---|
| `=0` (or omitted) | **Static** — equivalent to a hard `Width`/`Height`. Must have an explicit `Width`/`Height` set, or the item collapses to zero and clips children. |
| `=1` or any positive value | **Responsive/flexible** — takes a proportional share of remaining space after all `FillPortions: =0` siblings have claimed theirs. |

**FillPortions defaults to zero when omitted** — always set it explicitly.

**When to use `FillPortions: =1`:**
- Any child that must expand to fill remaining space in its parent
- The button group in a `SpaceBetween` row (fixed Cancel left, flexible group right)
- Any section that should grow/shrink with the container

**Ratio math:** Multiple flexible siblings split remaining space proportionally. E.g., `2:1:1` = 50% : 25% : 25%.

**`LayoutMinWidth` / `LayoutMinHeight`** — Sets the floor for a flexible item. When the floor is hit, the system treats this item as static at its minimum value and redistributes remaining flexible space among other items.

---

## SECONDARY AXIS ALIGNMENT — STATIC VS. STRETCH

On the axis **perpendicular** to `LayoutDirection`, each child can have a static size or stretch:

| `AlignInContainer` value | Child size on secondary axis | What to set |
|---|---|---|
| `Start`, `Center`, `End` | Static | `Width` (in vertical parent) or `Height` (in horizontal parent) |
| `Stretch` | Expands to fill container | Use `LayoutMinWidth`/`LayoutMinHeight` as floor — do NOT set a static size |

**Per-child override:** Set `AlignInContainer` on an individual child to override the parent's `LayoutAlignItems` for that item only.

**Rule:** When `LayoutAlignItems: =LayoutAlignItems.Stretch` is set on the parent, do NOT also set a static `Width`/`Height` on the child — it conflicts with stretch.

---

## OVERFLOW AND SCROLLABLE CONTAINERS

Define overflow behavior explicitly on any container where content might exceed its bounds:

| Property | Applies when |
|---|---|
| `LayoutOverflowVertical` | Content exceeds container height |
| `LayoutOverflowHorizontal` | Content exceeds container width |

| Value | Behavior |
|---|---|
| `=LayoutOverflow.Hide` | Extra content is clipped (default) |
| `=LayoutOverflow.Scroll` | Scrollbar is added automatically |

**Always set `LayoutOverflow.Scroll`** on:
- The main content pane / form body (scrolls vertically)
- Any container that wraps a gallery or list that can exceed available height

Set it on the **innermost scrollable wrapper**, not on a parent container above it.

```yaml
- scrollableContent:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      LayoutDirection: =LayoutDirection.Vertical
      LayoutOverflowVertical: =LayoutOverflow.Scroll
      LayoutOverflowHorizontal: =LayoutOverflow.Hide
```

---

## WRAP — REQUIREMENTS AND GOTCHAS

`LayoutWrap: =true` enables line-wrapping when children overflow the primary axis. Three conditions must all be met:

1. **Item width** set (static or formula)
2. **Item height** set (static or minimum height)
3. **Parent height** set to accommodate multiple rows

**Parent height formula when wrapping:**
```
(rows × item_height) + ((rows - 1) × LayoutGap) + PaddingTop + PaddingBottom
```

**Gotcha — hidden items still consume gap:** When `Visible: =false`, the gap beside the hidden item is still applied. Conditionally zero the gap:
```yaml
LayoutGap: =If(IsBlank(someValue), 0, 10)
```

**Gotcha — Stretch + Wrap:** When items have `AlignInContainer: Stretch` AND wrap is enabled, Power Apps uses `LayoutMinHeight` (not the flexible height) as the actual item height. Always set `LayoutMinHeight` explicitly on items inside a wrapping container.

---

## LAYOUT DECOMPOSITION PROCESS

When converting a design into containers, use the **single-axis decomposition test** at each level:

1. **Identify the major sections** — treat the design as a wireframe, ignore inner details.
2. **Test horizontal axis:** Does dividing with a horizontal line split any section?
   - No → use a **Vertical AutoLayout** container for this level.
   - Yes → test the vertical axis instead.
3. **Test vertical axis:** Does dividing with a vertical line split any section?
   - No → use a **Horizontal AutoLayout** container.
4. **Recurse** — repeat for each child container.
5. **Leaf rule:** If a section has only one control, or multiple controls all in the same direction as its parent, add them directly — no extra wrapper needed.

**Use ManualLayout only when:**
- Items need to **overlap** (e.g., background image + overlay button + label on top)
- The layout is genuinely unstructured with no single-axis decomposition
- Note: ManualLayout sacrifices easy responsiveness — prefer AutoLayout wherever possible

---

## GALLERY RESPONSIVE SIZING

**Template width — subtract 1 to prevent rounding scrollbars:**
```
TemplateWidth: =(Parent.Width / columnCount) - 1
```
Float division can produce a width 0.5px over the container, triggering an unwanted scrollbar. Subtract 1 to prevent it.

**Gallery root container must fill the template exactly:**
```yaml
X: =0
Y: =0
Width: =Parent.TemplateWidth
Height: =Parent.TemplateHeight
```

**Responsive column count via WrapCount:**
```yaml
WrapCount: =If(App.ActiveScreen.Size <= 2, 2, 4)
```

**Template padding:** Use the gallery's `TemplatePadding` property only for centered alignment. For custom positioning, set `TemplatePadding: =0` and position the root container manually.

---

## DROP SHADOW — ALWAYS CLEAR UNLESS EXPLICITLY NEEDED

Power Apps adds `DropShadow` to every new `GroupContainer` by default. **Clear it on every container** unless the design explicitly calls for a shadow (e.g., a floating modal or card).

```yaml
DropShadow: =DropShadow.None    # required on every container
```

---

## GALLERY TEMPLATE LAYERING

Children inside a gallery template behave like manual layout — siblings can OVERLAP each other.

### Card-style galleries (visible spacing, backgrounds, rounded corners)

Use 3-level nesting inside the gallery template:

**Level 1 — Gallery children (siblings that overlap):**
- `imgTemplateBG`: Image control as background
- `rootContainer`: Transparent AutoLayout, fills full gallery template, provides padding between cards

**Level 2 — Inside rootContainer:**
- `galCardTemplate`: The visible card container with border radius, its own padding, and background color

**Level 3 — Inside galCardTemplate:**
- Actual content (title, value, icons, etc.)

The background image's size and position come from `galCardTemplate` (not `rootContainer`) so it matches the visible card exactly. Border radius must also match.

**CORRECT card pattern:**
```yaml
Children:
  - imgTemplateBG:
      Control: Image@2.2.3
      Properties:
        Image: =Blank()
        Fill: =ThisItem.BG_Color
        Height: =galCardTemplate.Height
        Width: =galCardTemplate.Width
        X: =galCardTemplate.X
        Y: =galCardTemplate.Y
        RadiusBottomLeft: =20
        RadiusBottomRight: =20
        RadiusTopLeft: =20
        RadiusTopRight: =20
  - rootContainer:
      Control: GroupContainer@1.5.0
      Variant: AutoLayout
      Properties:
        DropShadow: =DropShadow.None
        Height: =Parent.TemplateHeight
        LayoutDirection: =LayoutDirection.Vertical
        PaddingBottom: =15
        PaddingLeft: =15
        PaddingRight: =15
        PaddingTop: =15
        Width: =Parent.TemplateWidth
      Children:
        - galCardTemplate:
            Control: GroupContainer@1.5.0
            Variant: AutoLayout
            Properties:
              Height: =Parent.Height
              Width: =Parent.Width
              LayoutDirection: =LayoutDirection.Vertical
              PaddingBottom: =20
              PaddingLeft: =20
              PaddingRight: =20
              PaddingTop: =20
              RadiusBottomLeft: =20
              RadiusBottomRight: =20
              RadiusTopLeft: =20
              RadiusTopRight: =20
            Children:
              # actual content here
```

### Table/list-style galleries (continuous rows, no gaps)
Content goes directly in the template root container. Each row has its own internal padding. No background image layer needed unless the design calls for it.

---

## BACKGROUND IMAGE CONTROLS

When using an Image control as a background layer:
- ALWAYS set `Image: =Blank()` to prevent the default SampleImage placeholder
- Set `Fill: =ThisItem.BG_Color` (or the color expression) for the background
- The Image control acts purely as a colored rectangle with rounded corners

---

## DROP SHADOW DEFAULT

Every container in Power Apps has drop shadow ON by default.

**ALWAYS set `DropShadow: =DropShadow.None` on every container** unless you specifically want a shadow. Forgetting this makes the UI look cluttered.

---

## CUSTOM BUTTON PATTERN

To create a clickable area in Power Apps, use a ManualLayout container with:
1. A `Label` for text
2. An `Image` for the icon
3. A transparent `Classic/Button@2.2.0` overlaid on top:
   ```yaml
   Fill: =RGBA(0, 0, 0, 0)
   HoverFill: =RGBA(0,0,0,0.1)
   PressedFill: =RGBA(0,0,0,0.2)
   Text: =""
   Width: =Parent.Width
   Height: =Parent.Height
   ```

The button captures clicks; the label and icon provide the visual appearance.

---

## BORDER RADIUS WORKAROUND

Controls that do NOT have border radius properties (applying RadiusBottomLeft/Right/TopLeft/Right directly will cause PA2108):
- `Label@2.5.1`
- `ComboBox@0.0.51`
- `DatePicker@0.0.46`
- `NumberInput@2.9.12`
- `Radio@0.0.25`
- `TextInput@0.0.54`

**To give these controls rounded corners:**
1. Wrap the control inside a `GroupContainer`
2. Apply `RadiusBottomLeft/Right` and `RadiusTopLeft/Right` to the CONTAINER
3. Set `Fill` on the container for the background color

---

## SVG ICON COLOR HANDLING — CRITICAL

SVGs are HTML text strings. Colors inside SVGs must be TEXT STRINGS, not Power Apps RGBA function calls.

### Pattern 1: Hardcoded color
```yaml
Image: |-
  ="data:image/svg+xml;utf8, " & EncodeUrl(
      Substitute(
          ThisItem.Icon, "currentColor", "RGBA(246,246,250,1)"
      )
  )
```
Note: `"RGBA(246,246,250,1)"` is a text string in quotes — NOT a function call.

### Pattern 2: Dynamic color from a field or variable

When the color comes from a data field (e.g., `ThisItem.KPIColor`) or a variable, use `JSON()` to serialize it to a string, then strip the surrounding quotes:

```yaml
Substitute(JSON(ThisItem.KPIColor), """", "")
```

This produces a string like `#22c55e` or `rgba(0, 142, 210, 1)` — a text value the SVG can use.

For a hardcoded color value: `Substitute(JSON(RGBA(34,197,94,1)), """", "")` produces `#22c55e`.

**WRONG — `Text()` on a Color type returns empty string or error:**
```yaml
Text(ThisItem.KPIColor)      # WRONG — Color is not a text-convertible type
```

**WRONG — passing a Color value directly:**
```yaml
RGBA(34,197,94,1)   # This is a Color value, not a string — SVG cannot use it
```

**Key rule**: Inside `EncodeUrl(Substitute(...))`, the replacement color must ALWAYS be a text string. Use `Substitute(JSON(colorValue), """", "")` for any dynamic/field color. Use a quoted literal like `"RGBA(246,246,250,1)"` for hardcoded colors.

---

## DESIGN QUALITY STANDARDS

### Spacing & Padding
- Card content containers: `PaddingTop/Bottom/Left/Right: =15` to `=25` — NEVER skip padding
- Outer wrapper containers: `PaddingTop/Bottom/Left/Right: =15` to `=20`
- Gallery `TemplatePadding: =0` — control spacing via inner container padding
- `LayoutGap: =8` to `=16` between child elements in AutoLayout containers
- Gallery `TemplateSize: =Self.Width/NumberOfCards` for equal-width cards
- Always check: "Would text touch the edge?" If yes, add more padding

### Gallery Template Padding
The visible card container inside a gallery MUST have padding:
```yaml
PaddingBottom: =20
PaddingLeft: =20
PaddingRight: =20
PaddingTop: =20
```

### Color Palette
- Dark backgrounds: `RGBA(35,36,47,1)` or `RGBA(42,43,71,1)`
- Blue accents: `RGBA(0,142,210,1)` or `RGBA(14,165,233,1)`
- Light text on dark: `RGBA(246,246,250,1)` at full opacity
- Subtle text on dark: `RGBA(246,246,250,0.7)` for secondary text
- Near-black text: `RGBA(15,23,42,1)` or `RGBA(35,36,47,1)`
- Gray text: `RGBA(100,116,139,1)` or `RGBA(131,141,157,1)`
- Avoid pure black `RGBA(0,0,0,1)` or pure white `RGBA(255,255,255,1)`
- Card fills should alternate between 2 colors for visual variety

### Font Hierarchy (must be clearly distinct)

**General cap: font size must never exceed 13 for ordinary UI text.** Canvas Apps render at ~96 DPI on a 1366px canvas, so sizes that look reasonable on screen appear very large in the rendered app. The only exception is large numeric KPI/stat values where the design explicitly calls for a prominent number.

| Element | Size range | Font |
|---|---|---|
| Page/screen title | `=13` to `=14` max | `Font.'Lato Black'`, Semibold |
| Section/card title | `=12` | `Font.Lato`, Semibold |
| Field labels, button text, tab labels | `=11` | `Font.Lato`, Semibold or Normal |
| Body text, input placeholder | `=10` to `=11` | `Font.Lato`, Normal |
| Secondary/muted text, breadcrumbs | `=9` to `=10` | `Font.Lato`, Normal, lighter color |
| Table headers | `=8` | `Font.'Lato Light'`, Semibold, gray |
| Table content, badge text | `=9` | `Font.Lato`, Normal |
| KPI/stat primary value (explicit design call) | `=20` to `=28` | `Font.'Lato Black'`, Bold |

**Never default to Size: =13 or higher for every label** — that produces UI that looks oversized on screen. Use the hierarchy above to maintain a clear visual scale.

### Never Leave Controls at Default Appearance

Every control you emit must be fully styled to match the target design. Power Apps default control appearance is plain, grey, and unstyled — it is never acceptable in generated output.

**For every control, always explicitly set:**
- `Font`, `Size`, `Color` / `Fill` — match the design's typography and palette
- Hover and pressed states (`HoverFill`, `HoverColor`, `PressedFill`, `PressedColor`) — make interactions feel polished
- `BorderColor`, `BorderStyle`, `BorderThickness` — match the design or use transparent borders for ghost elements
- `RadiusTopLeft/Right/BottomLeft/Right` — match the design's corner style; never leave sharp corners on inputs or buttons if the design is rounded
- `Font.Lato` as the default font — never leave `Font.OpenSansRegular` (the PA default)

This applies to every `Classic/TextInput`, `Classic/ComboBox`, `Classic/CheckBox`, `Classic/Radio`, `Classic/Button`, `Label`, `Gallery`, and any other control. If a control is in the output, it is styled. There are no exceptions.

### Radius Wrapper Pattern

Some controls do **not** support `RadiusTopLeft/Right/BottomLeft/Right` — adding these properties causes PA2108. Controls confirmed to lack radius support:

- `Classic/ComboBox@2.4.0`
- `Classic/DatePicker@2.6.0`
- `Classic/Radio@2.3.0`
- `Classic/CheckBox@2.1.0`
- `Label@2.5.1`
- `Rectangle@2.3.0`

**Rule:** When you need rounded corners on a control that doesn't support radius, wrap it in a `GroupContainer` with the radius and border set on the container. The GroupContainer's rounded corners visually clip the child control. The inner control must have its own border removed (`BorderStyle: =BorderStyle.None`, `BorderThickness: =0`) to avoid a double border.

**Pattern:**
```yaml
- controlNameFrame:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      BorderColor: =RGBA(210,210,220,1)
      BorderStyle: =BorderStyle.Solid
      BorderThickness: =1
      DropShadow: =DropShadow.None
      Fill: =RGBA(255,255,255,1)
      FillPortions: =0
      Height: =40                        # same as the inner control
      LayoutAlignItems: =LayoutAlignItems.Stretch
      LayoutDirection: =LayoutDirection.Vertical
      LayoutMinHeight: =0
      LayoutMinWidth: =0
      PaddingBottom: =2                  # >= inner control's FocusedBorderThickness
      PaddingLeft: =2
      PaddingRight: =2
      PaddingTop: =2
      RadiusBottomLeft: =8
      RadiusBottomRight: =8
      RadiusTopLeft: =8
      RadiusTopRight: =8
    Children:
      - controlName:
          Control: Classic/ComboBox@2.4.0   # or any non-radius control
          Properties:
            AlignInContainer: =AlignInContainer.Stretch
            BorderStyle: =BorderStyle.None    # ← no own border; frame provides it
            BorderThickness: =0
            Fill: =RGBA(0,0,0,0)             # ← transparent; frame provides background
            FocusedBorderColor: =RGBA(160,160,175,1)
            FocusedBorderThickness: =2
            # ... other control properties
```

**Naming convention:** Name the wrapper `[controlName]Frame` (e.g., `inputCategoryFrame` wraps `inputCategory`).

**Height formula:** Any container that references the inner control's height must reference the frame instead: `=lblField.Height + inputCategoryFrame.Height + 6`.

**Note:** The inner control's hover/pressed fill states still work normally (they apply to the control's interior). Border-related hover states (e.g., `HoverBorderColor`) are lost since the inner control has no border — accept this limitation, or use `OnHover` formulas for dynamic frame border color if required.

---

### Border Clipping Inside Zero-Padding Containers

When a child control has a visible border and its parent container has zero padding on the side where the border falls, the border is clipped by the container's edge. This also affects the **focused** border: when a control is focused, its border expands from `BorderThickness` to `FocusedBorderThickness`. If the container's padding is only equal to the base `BorderThickness`, the focused border overflows and clips.

**Rule:** The container's minimum padding must equal the child's **`FocusedBorderThickness`** (NOT just `BorderThickness`). This is the worst-case border width and guarantees no clipping in any state.

- If child has `FocusedBorderThickness: =2` → set container `PaddingTop/Bottom/Left/Right: =2`
- If child has `FocusedBorderThickness: =1` → `PaddingTop/Bottom/Left/Right: =1` is sufficient
- This is ONLY for preventing clipping — do NOT add padding to every container by default. Only apply when a bordered child would otherwise be flush against the container edge.

Example:
```yaml
- inputWrapper:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      PaddingLeft: =2    # >= child's FocusedBorderThickness (2) — prevents clipping in focused state
      PaddingRight: =2
      PaddingTop: =2
      PaddingBottom: =2
```

If the container already has meaningful padding (e.g., `PaddingLeft: =16`), no extra padding is needed — the border will not be clipped.

### FocusedBorderThickness Cap

**Rule:** `FocusedBorderThickness` must be **≤ 2** on ALL controls — inputs, buttons, checkboxes, radios, overlays, galleries — with no exceptions. This applies regardless of whether the focus ring is visible or invisible.

Always set `FocusedBorderThickness` explicitly. Never omit it — Power Apps defaults to 4, which is too thick.

Standard pattern:
```yaml
BorderThickness: =1          # base border
FocusedBorderColor: =RGBA(r,g,b,1)   # same or slightly darker than HoverBorderColor
FocusedBorderThickness: =2   # focus state — 1 step thicker, never more
```

Never use values of 3 or 4, even on transparent overlay buttons where the ring would be invisible. The rule is unconditional.

### Explicit Container Heights Must Include Padding and Gap

When a `GroupContainer` in `AutoLayout` mode uses an **explicit `Height`** formula that sums child heights, the formula must always include the container's own padding and gap. Omitting padding causes children to be clipped — the container's logical height is shorter than its visual content area.

**Formula pattern (vertical AutoLayout container — 2 children):**
```
Height: =child1.Height + child2.Height + Self.LayoutGap + Self.PaddingTop + Self.PaddingBottom
```

**Formula pattern (n children):**
```
Height: =child1.Height + ... + childN.Height + (N - 1) * Self.LayoutGap + Self.PaddingTop + Self.PaddingBottom
```

**Formula pattern (horizontal AutoLayout container — width-driven):**
```
Width: =child1.Width + ... + childN.Width + (N - 1) * Self.LayoutGap + Self.PaddingLeft + Self.PaddingRight
```

**Rules:**
- Always use `Self.PaddingTop` / `Self.PaddingBottom` (not hardcoded numbers) — if padding changes, the formula stays correct automatically.
- Always use `Self.LayoutGap` (not a hardcoded number) for the same reason.
- If `LayoutGap` is 0 or there is only one child, the gap term is 0 — omit it.
- If the container uses `FillPortions`-driven height or `Height: =Parent.Height`, this rule does not apply — Power Apps manages the space automatically.
- When a child is a Radius Wrapper frame (GroupContainer wrapping a no-radius control), the frame's height already accounts for its own inner padding — use `frame.Height` directly, not the inner control's height.

**Wrong — padding missing (causes bottom clipping):**
```yaml
Height: =lblTitle.Height + inputName.Height + 6
PaddingTop: =2
PaddingBottom: =2
LayoutGap: =6
```

**Correct:**
```yaml
Height: =lblTitle.Height + inputName.Height + Self.LayoutGap + Self.PaddingTop + Self.PaddingBottom
PaddingTop: =2
PaddingBottom: =2
LayoutGap: =6
```

### Always Set Explicit Values for Visual Properties

Power Apps applies its own default styling when a property is omitted. Those defaults are generic and will not match the design. For every control in the output, **explicitly set ALL properties** in the applicable lists below. Never omit them.

**Interactive controls** — `Classic/Button`, `Classic/TextInput`, `Classic/ComboBox`, `Classic/CheckBox`, `Classic/Radio`, `Gallery`:

Every instance must include ALL of:
- `HoverFill`, `HoverColor`, `HoverBorderColor`
- `PressedFill`, `PressedColor`, `PressedBorderColor`
- `DisabledFill`, `DisabledColor`, `DisabledBorderColor`
- `BorderStyle`, `BorderThickness`
- `FontWeight`
- `FocusedBorderColor`, `FocusedBorderThickness`
- `RadiusTopLeft`, `RadiusTopRight`, `RadiusBottomLeft`, `RadiusBottomRight`

Additionally:
- `Classic/CheckBox`: must also have `CheckboxSize`, `CheckboxBackgroundFill`, `CheckmarkFill`, `CheckboxBorderColor` (see dedicated section below)
- `Classic/Radio`: must have `Height: =39` minimum, plus `RadioSize`, `RadioBackgroundFill`, `RadioBorderColor`, `RadioSelectionFill` (see dedicated section below)
- `Classic/ComboBox`: must also have `ChevronFill`, `ChevronHoverFill`, `ChevronBackground`, `ChevronHoverBackground`, `SelectionColor`, `SelectionFill`, and `FocusedBorderThickness: =0` (see dedicated section below)

> **Important:** Not all controls support every property above. Always cross-check `controls-reference.md` for the exact control type. Only set properties that appear there. Never guess. Unlisted properties cause PA2108.

**Display controls** — `Label@2.5.1`:

Labels are non-interactive — do NOT add HoverColor, PressedColor, or similar states. Always set:
- `FontWeight` — `=FontWeight.Normal`, `.Semibold`, or `.Bold` (never omit)
- `Align` — `=Align.Left`, `.Center`, or `.Right`
- `VerticalAlign` — on fixed-height labels only; omit when `AutoHeight: =true`

**Structural controls** — `GroupContainer`, `Rectangle`, `Image`:
- `GroupContainer`: Always set `Fill` (use `=RGBA(0,0,0,0)` for transparent), `DropShadow: =DropShadow.None` unless shadow is intentional.
- `Rectangle`: Always set `BorderStyle`, `BorderThickness`, `BorderColor`.
- `Image`: Always set `BorderStyle: =BorderStyle.None`, `BorderThickness: =0`, `BorderColor: =RGBA(0,0,0,0)` for decorative images.

**Quick-reference default values when design has no specific guidance:**

| Property | Default value |
|---|---|
| `HoverFill` on inputs | `=RGBA(248,249,250,1)` |
| `HoverFill` on ghost/transparent buttons | `=RGBA(0,0,0,0.04)` |
| `PressedFill` on ghost/transparent buttons | `=RGBA(0,0,0,0.08)` |
| `DisabledFill` on inputs and solid buttons | `=RGBA(240,240,245,1)` |
| `DisabledColor` on inputs and solid buttons | `=RGBA(180,180,195,1)` |
| `DisabledBorderColor` on bordered controls | `=RGBA(220,220,230,1)` |
| `FontWeight` when not bold or semibold | `=FontWeight.Normal` |
| `Align` when not centered | `=Align.Left` |
| `BorderStyle` when no border | `=BorderStyle.None` with `BorderThickness: =0` |
| `HoverBorderColor` on inputs | `=Self.BorderColor` |
| `PressedBorderColor` on inputs | `=Self.BorderColor` |

### Visual Polish
- Border radius: 15–20 for cards, 10–15 for buttons/pills, 30+ for circular elements
- Icon containers: `Fill: =RGBA(255,255,255,0.5)`, 35×35 size, `RadiusBottomLeft/Right/TopLeft/Right: =10`
- Status pills: `BorderColor` set to accent color + `Fill: =ColorFade(Self.BorderColor, 80%)`

---

## CLASSIC/CHECKBOX — REQUIRED SIZE AND STYLE PROPERTIES

**`CheckboxSize` — always set explicitly.** Power Apps defaults `CheckboxSize` to 40, which is far too large for compact forms. Without this property, every checkbox renders as a 40px square that clashes with the design.

Always set `CheckboxSize` to match the design. For compact forms (font size 10–12), typical values:
- `=16` — small, tight forms
- `=18` — standard compact forms (recommended default)
- `=20` — slightly larger, still compact

**Two separate border properties on `Classic/CheckBox`:**
- `CheckboxBorderColor` — border of the **tick box square only**
- `BorderColor` — border around the **entire control** (tick box + label text combined)

These are distinct. `BorderColor` should be set transparent (`=RGBA(0,0,0,0)`) since the outer control border is almost never visible in designs.

**Required properties on every `Classic/CheckBox@2.1.0`:**
```yaml
CheckboxSize: =18               # REQUIRED — default 40 is too large
CheckboxBackgroundFill: =If(Self.Value, RGBA(42,43,71,1), RGBA(255,255,255,1))
CheckmarkFill: =RGBA(255,255,255,1)
CheckboxBorderColor: =RGBA(180,180,195,1)  # tick box square border
Color: =RGBA(60,60,80,1)        # text label color
Fill: =RGBA(255,255,255,1)      # control background
BorderColor: =RGBA(0,0,0,0)     # outer control border — set transparent
BorderStyle: =BorderStyle.None
BorderThickness: =0
FocusedBorderColor: =RGBA(100,100,120,1)
FocusedBorderThickness: =2
HoverBorderColor: =RGBA(100,100,120,1)
HoverFill: =RGBA(0,0,0,0)
PressedBorderColor: =RGBA(100,100,120,1)
PressedColor: =RGBA(42,43,71,1)
PressedFill: =RGBA(0,0,0,0.04)
FontWeight: =FontWeight.Normal
```
> `Classic/CheckBox` does NOT support `DisabledBorderColor`, `DisabledColor`, or `DisabledFill` — omit those.

---

## CLASSIC/RADIO — MINIMUM HEIGHT TO PREVENT SCROLLBAR

**`Classic/Radio@2.3.0` (horizontal layout) has a minimum height that depends on both `RadioSize` and font `Size`.** Below this minimum, a scrollbar appears. The minimum is NOT a flat value — it scales with both properties.

**Formula (horizontal layout only):**
```
min_height = 39 + floor(max(0, RadioSize - 16) / 2) + max(0, fontSize - 10)
```

**Reference table:**

| RadioSize | Font Size | Min Height |
|---|---|---|
| ≤16 | ≤10 | 39 |
| ≤16 | 11 | 40 |
| ≤16 | 12 | 41 |
| 18 | ≤10 | 40 |
| 18 | 11 | 41 |
| 18 | 12 | 42 |
| 18 | 13 | 43 |
| 20 | ≤10 | 41 |
| 20 | 11 | 42 |

**Rule:** Always compute the correct minimum height using the formula above before setting `Height` on any `Classic/Radio@2.3.0` with `Layout: =Layout.Horizontal`. Any parent container with an explicit fixed `Height` must also be ≥ this minimum.

**Required properties on every `Classic/Radio@2.3.0`:**
```yaml
Height: =41                     # EXAMPLE for RadioSize=18, fontSize=11 — compute per formula above
RadioSize: =18                  # size of the radio circle
RadioBackgroundFill: =RGBA(255,255,255,1)
RadioBorderColor: =RGBA(160,160,180,1)
RadioSelectionFill: =RGBA(42,43,71,1)
Color: =RGBA(42,43,71,1)
Fill: =RGBA(255,255,255,1)
BorderStyle: =BorderStyle.None
BorderThickness: =0
BorderColor: =RGBA(180,180,195,1)
FocusedBorderColor: =RGBA(100,100,120,1)
FocusedBorderThickness: =2
HoverFill: =RGBA(0,0,0,0)
FontWeight: =FontWeight.Normal
```
> `Classic/Radio` does NOT support `HoverBorderColor` — omit it. Cross-check `controls-reference.md` before adding `PressedFill`, `PressedBorderColor`, `PressedColor`, `DisabledBorderColor`, `DisabledColor`, or `DisabledFill`.

### Classic/ComboBox Styling

Every `Classic/ComboBox@2.4.0` must have the following chevron and selection properties explicitly set, in addition to the standard interactive-control properties listed above.

```yaml
ChevronFill: =RGBA(100,100,120,1)
ChevronHoverFill: =RGBA(100,100,120,1)
ChevronBackground: =RGBA(0,0,0,0)
ChevronHoverBackground: =RGBA(0,0,0,0.04)
SelectionColor: =RGBA(42,43,71,1)
SelectionFill: =RGBA(0,0,0,0.05)
FocusedBorderThickness: =0
```

- `ChevronBackground: =RGBA(0,0,0,0)` makes the chevron area transparent for a modern look.
- `ChevronHoverFill` should match `ChevronFill` if no hover color change is desired.
- `FocusedBorderThickness: =0` removes the dotted border that appears when a ComboBox dropdown is opened. On other controls this property controls the focus ring thickness, but on ComboBox it produces an unwanted dotted outline.
- Always wrap in the **Radius Wrapper Pattern** for rounded corners — ComboBox does not support `RadiusXxx` properties.

### Dropdown-to-Radio/Checkbox Conversion

When generating a screen that contains ComboBox or DropDown controls with a small, fixed set of options, recommend converting them to reduce user clicks:

- **2–3 options (single-select):** Recommend `Classic/Radio@2.3.0` instead of a ComboBox/DropDown. A Radio lets the user select in one click instead of two (open dropdown + select). Present this to the user: "This field has only N options — would you like me to use a Radio instead of a ComboBox for fewer clicks?"
- **2–4 options (multi-select):** Recommend individual `Classic/CheckBox@2.1.0` controls instead of a multi-select ComboBox. Each checkbox is directly visible and clickable.

This is a **recommendation only** — do not auto-convert without asking the user. If the user declines, use the ComboBox as designed.

---

## RESPONSIVE DESIGN PATTERNS

### Screen Size System

Power Apps provides a `Size` property on every screen, calculated from the screen width and app breakpoints.

**Default breakpoints:**
| Size | ScreenSize enum | Width range |
|---|---|---|
| 1 | `ScreenSize.Small` | < 600px (mobile) |
| 2 | `ScreenSize.Medium` | 600–900px (tablet portrait) |
| 3 | `ScreenSize.Large` | 900–1200px (tablet landscape) |
| 4 | `ScreenSize.ExtraLarge` | > 1200px (desktop) |

Reference in formulas as `ScreenName.Size` (replace `ScreenName` with the actual screen name). Use `ScreenSize.Large` etc. for readable comparisons.

> **`ScreenName.Size` vs `App.ActiveScreen.Size`:**
> - **Use `ScreenName.Size`** whenever the user requests responsive design or the output is a full screen. This ties directly to the viewport and enables true responsive scaling. Ask the user for the screen name if it isn't known.
> - **`App.ActiveScreen.Size` is acceptable only** when generating a partial component or section (paste target a or c) AND the user has not mentioned responsiveness. In that case the screen name is unknown and `App.ActiveScreen.Size` is a reasonable fallback.
> - **Never use `App.ActiveScreen.Size` on a full-screen (paste target b) output** — always use the actual screen name there.
> - If you see `App.ActiveScreen.Size` in reference/template code, note this distinction and substitute `ScreenName.Size` in generated output whenever the screen name is known.

### Conditional Sizing Pattern

To toggle between flexible and static height/width based on screen size:

```yaml
# Flexible height on large screens, static on small:
FillPortions: =If(HomeScreen.Size >= ScreenSize.Large, 1, 0)
Height: =If(HomeScreen.Size >= ScreenSize.Large, 0, galleryContainer.Height + 20)
```

- `FillPortions: =1` = flexible (fills remaining space)
- `FillPortions: =0` = static (use the explicit Height/Width property)

### FillPortions Toggle for Responsive Height

For sections that should flex on desktop but have a fixed height on mobile:
```yaml
FillPortions: =If(HomeScreen.Size >= ScreenSize.Large, 1, 0)
LayoutMinHeight: =350
Height: =If(HomeScreen.Size < ScreenSize.Large, taskSection.Height + inspectionSection.Height, 0)
```

### Conditional Visibility
```yaml
Visible: =HomeScreen.Size >= ScreenSize.Large   # show only on tablet/desktop
Visible: =HomeScreen.Size < ScreenSize.Large    # show only on mobile/small
```

### Conditional Padding / Spacing
```yaml
PaddingLeft: =If(HomeScreen.Size >= ScreenSize.Large, 32, 24)
PaddingRight: =If(HomeScreen.Size >= ScreenSize.Large, 32, 24)
```

### Gallery Responsive Patterns

**Dynamic gallery height** (gallery expands to show all its items):
```yaml
Height: =Self.AllItemsCount * Self.TemplateHeight
```

**WrapCount by screen size** (2 columns on mobile, 4 on desktop):
```yaml
WrapCount: =If(HomeScreen.Size < ScreenSize.Large, 2, 4)
TemplateSize: =Self.Width / Self.WrapCount - 1
```
Note: subtract 1 from TemplateSize to prevent scroll caused by decimal rounding.

**Reduce visible items on small screens:**
```yaml
Items: =FirstN(colSampleData, If(HomeScreen.Size < ScreenSize.Large, 7, 14))
```

### Wrap Property — 3 Criteria

When using `LayoutWrap: =true` on a container, ALL THREE of these must be set or the layout will break:

1. **Width of each item** — must be set so wrapping knows when to move to next line
2. **Height of each item** — must be set or items collapse
3. **Height of the parent** — either static (add `LayoutOverflowY: =LayoutOverflow.Scroll`) or dynamic (sum of all child item heights + padding + gaps)

### Scrollable Content Pattern

For screens that need to scroll only the content area (fixed header + scrollable body):
```yaml
- staticHeaderContainer:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      Height: =75
      FillPortions: =0
      # ... header content
- scrollableContent:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      FillPortions: =1
      LayoutOverflowY: =LayoutOverflow.Scroll
      # ... scrollable content
```

### Screen Design Principle

There is no requirement that all content must be on one screen. If a form has many fields, use a multi-screen form. If a dashboard is complex, show only the most important KPIs on the main screen. Well-spaced, padded, modern layouts are more important than fitting everything in one view.

---

## COMPONENT PATTERNS

These are proven, reusable patterns extracted from real production Canvas App YAML. Use them directly — do not reinvent.

---

### Custom Button Pattern

The standard button pattern in Canvas Apps is NOT the native Button control. It is a `ManualLayout` GroupContainer styled as a button, with an SVG icon Image and a transparent `Classic/Button` overlay for interaction. This gives full control over shape, color, icon, and hover/press states.

```yaml
- customButtonName:
    Control: GroupContainer@1.5.0
    Variant: ManualLayout
    Properties:
      DropShadow: =DropShadow.None
      Fill: =RGBA(42,43,71,1)          # button background color
      Height: =35
      Width: =150
      RadiusBottomLeft: =10
      RadiusBottomRight: =10
      RadiusTopLeft: =10
      RadiusTopRight: =10
    Children:
      - btnLabel:
          Control: Label@2.5.1
          Properties:
            Color: =Color.White
            Font: =Font.Lato
            FontWeight: =FontWeight.Semibold
            Height: =Parent.Height
            Width: =Parent.Width
            Size: =10
            PaddingLeft: =5 + btnIcon.X + btnIcon.Width   # offsets text past the icon
            Text: ="Button Label"
      - btnIcon:
          Control: Image@2.2.3
          Properties:
            Height: =25
            Width: =25
            X: =10
            Y: =(Parent.Height - Self.Height)/2
            Image: |-
              ="data:image/svg+xml;utf8, " & EncodeUrl(
                  Substitute(
                      "<svg xmlns='http://www.w3.org/2000/svg' width='16' height='16' fill='currentColor' viewBox='0 0 16 16'>
                <path d='M8 4a.5.5 0 0 1 .5.5v3h3a.5.5 0 0 1 0 1h-3v3a.5.5 0 0 1-1 0v-3h-3a.5.5 0 0 1 0-1h3v-3A.5.5 0 0 1 8 4z'/>
              </svg>",
                      "currentColor", "RGBA(246,246,250,1)"
                  )
              )
      - btnClickOverlay:
          Control: Classic/Button@2.2.0
          Properties:
            Text: =""
            Fill: =RGBA(0,0,0,0)
            HoverFill: =RGBA(0,0,0,0.08)
            PressedFill: =RGBA(0,0,0,0.16)
            HoverColor: =RGBA(255,255,255,1)
            PressedColor: =RGBA(255,255,255,1)
            BorderStyle: =BorderStyle.None
            BorderColor: =RGBA(0,0,0,0)
            HoverBorderColor: =RGBA(0,0,0,0)
            PressedBorderColor: =ColorFade(RGBA(0,188,242,1), 50%)
            FocusedBorderColor: =ColorFade(Self.Fill, 75%)
            DisabledBorderColor: =RGBA(0,0,0,0)
            Height: =Parent.Height
            Width: =Parent.Width
            OnSelect: =false   # replace with actual action
```

**Icon-only small button** (25×25 circle, used for row actions and nav):
```yaml
- iconButton:
    Control: GroupContainer@1.5.0
    Variant: ManualLayout
    Properties:
      Fill: =RGBA(42,43,71,1)
      Height: =25
      Width: =25
      RadiusBottomLeft: =10
      RadiusBottomRight: =10
      RadiusTopLeft: =10
      RadiusTopRight: =10
    Children:
      - iconImg:
          Control: Image@2.2.3
          Properties:
            Height: =Parent.Height
            Width: =Parent.Width
            PaddingTop: =3
            PaddingBottom: =3
            Image: |-
              ="data:image/svg+xml;utf8, " & EncodeUrl(
                  Substitute("<svg ...>...</svg>", "currentColor", "RGBA(246,246,250,1)")
              )
      - iconOverlay:
          Control: Classic/Button@2.2.0
          Properties:
            Text: =""
            Fill: =RGBA(0,0,0,0)
            HoverFill: =RGBA(0,0,0,0.1)
            PressedFill: =RGBA(0,0,0,0.2)
            BorderStyle: =BorderStyle.None
            Height: =Parent.Height
            Width: =Parent.Width
            OnSelect: =false
```

---

### SVG Inline Icon Pattern

**This is the ONLY approved icon method.** Never use `Classic/Icon@2.5.0` — its icon set looks dated and undermines the modern aesthetic this skill exists to produce. All icons must use `Image@2.2.3` with an inline SVG data URI.

Source preference: Bootstrap Icons (`bi-*`) for common UI icons; construct custom SVG paths for anything not in Bootstrap Icons. The `Substitute` call replaces `currentColor` with a color string so you can tint any SVG.

```yaml
Image: |-
  ="data:image/svg+xml;utf8, " & EncodeUrl(
      Substitute(
          "<svg xmlns='http://www.w3.org/2000/svg' width='16' height='16' fill='currentColor' viewBox='0 0 16 16'>
    <path d='...'/>
  </svg>",
          "currentColor", "RGBA(42,43,71,1)"
      )
  )
```

**SVG from a data field** (e.g. gallery item stores SVG string):
```yaml
Image: |-
  ="data:image/svg+xml;utf8, " & EncodeUrl(
      Substitute(ThisItem.Icon, "currentColor", "RGBA(42,43,71,1)")
  )
```

**Conditional icon tinting** (selected vs unselected state):
```yaml
Image: |-
  ="data:image/svg+xml;utf8, " & EncodeUrl(
      Substitute(
          If(CurrentMenuID = ThisItem.MenuID, ThisItem.MenuIcon_Selected, ThisItem.MenuIcon),
          "currentColor",
          If(CurrentMenuID = ThisItem.MenuID, "RGBA(42,43,71,1)", "RGBA(192,194,197,1)")
      )
  )
```

> **SVG rule:** SVG strings containing `xmlns='http://...'` always use `|-` multiline format because of the colon in the URL (see Colon/Hash Rule above).

---

### Collapsible Sidebar Pattern

Drive the entire sidebar with a single boolean variable `varMenuExpanded`. Set the initial state in screen `OnVisible`.

**Screen OnVisible:**
```yaml
OnVisible: |-
  =Set(varMenuExpanded, false);
  Set(CurrentMenuID, 1)
```

**Sidebar container** (fixed width, switches between collapsed/expanded):
```yaml
- sidebarContainer:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      FillPortions: =0
      Width: =If(varMenuExpanded, 200, 60)
      Height: =Parent.Height
      LayoutDirection: =LayoutDirection.Vertical
      DropShadow: =DropShadow.None
      Fill: =Color.White
```

**Hamburger toggle button** (collapses/expands on click):
```yaml
OnSelect: =Set(varMenuExpanded, !varMenuExpanded)
```

**Elements that react to sidebar state:**
```yaml
# Label: visible only when expanded
Visible: =varMenuExpanded

# Icon: centered when collapsed, left-aligned when expanded
X: =If(varMenuExpanded, 10, (Parent.Width - Self.Width)/2)

# Container alignment
LayoutJustifyContent: =If(varMenuExpanded, LayoutJustifyContent.Start, LayoutJustifyContent.Center)
PaddingLeft: =If(varMenuExpanded, 10, 0)

# Logo: swap between full logo and icon-only logo
Image: =If(varMenuExpanded, 'AppLogoFull', 'AppLogoIcon')
```

**Gallery-as-nav-menu** — store all nav items in an inline `Table()` inside the gallery `Items`:

```yaml
Items: |-
  =Table(
      {
          MenuIcon_Selected: "<svg ...filled version...>",
          MenuIcon: "<svg ...outline version...>",
          MenuName: "Home",
          MenuID: 1,
          PageNavigation: HomeScreen
      },
      {
          MenuIcon_Selected: "<svg ...>",
          MenuIcon: "<svg ...>",
          MenuName: "Tasks",
          MenuID: 2,
          PageNavigation: TasksScreen
      }
  )
TemplateSize: =60
TemplatePadding: =0
Height: =Self.AllItemsCount * (Self.TemplateHeight + Self.TemplatePadding)
```

**Active menu item highlight** — inside the gallery template:
```yaml
# Background: highlighted when active
Fill: =If(CurrentMenuID = ThisItem.MenuID, RGBA(240,239,248,1), RGBA(0,0,0,0))

# Label color
Color: =If(CurrentMenuID = ThisItem.MenuID, RGBA(42,43,71,1), RGBA(192,194,197,1))

# Navigation on click
OnSelect: =Navigate(ThisItem.PageNavigation); Set(CurrentMenuID, ThisItem.MenuID)
```

**Left accent bar** (5px pill indicator, right-radius only, visible only on active item):
```yaml
- activeAccentBar:
    Control: GroupContainer@1.5.0
    Variant: ManualLayout
    Properties:
      Fill: =RGBA(42,43,71,1)
      Width: =5
      Height: =Parent.Height * 80%
      Y: =(Parent.Height - Self.Height)/2
      RadiusTopRight: =20
      RadiusBottomRight: =20
      RadiusTopLeft: =0
      RadiusBottomLeft: =0
      Visible: =CurrentMenuID = ThisItem.MenuID
      DropShadow: =DropShadow.None
```

---

### Tab Bar Pattern (Horizontal Underline Tabs)

A horizontal tab bar that switches content sections on the same screen. Built from a `Gallery@2.15.0 (Horizontal)` inside a vertical AutoLayout container. Each tab item has a label, an active-state underline, and a transparent button overlay for tap handling. A 1px divider sits below the gallery.

**Template:** `templates/tab-bar.yaml`

**State variable:** `varActiveTab` (Integer) — holds the `TabID` of the currently active tab.

**Screen OnVisible — initialize state and tab data:**
```yaml
OnVisible: |-
  =Set(varActiveTab, 1);
  ClearCollect(colTabs,
      {TabName: "General", TabID: 1},
      {TabName: "Details", TabID: 2},
      {TabName: "Settings", TabID: 3}
  )
```

**Tab gallery — key properties:**
```yaml
Items: =colTabs
TemplateSize: =180
```
Use `TemplateSize: =Self.Width / Self.AllItemsCount - 1` for auto-fit equal-width tabs. Use a fixed `TemplateSize` (e.g., 180) when tab labels vary in length — size to the WIDEST label plus padding. Do NOT set a value too small for the longest tab name or it will be cut off.

**CRITICAL — tab item container width inside gallery template:**
Inside a horizontal gallery, the template item container MUST use `Width: =Parent.TemplateWidth` — NOT `Width: =Parent.Width`. `Parent.Width` gives the full gallery width (e.g., 1236px), making every tab item as wide as the entire gallery. `Parent.TemplateWidth` gives the correct per-item width (e.g., 180px).

```yaml
- tabItemContainer:
    Control: GroupContainer@1.5.0
    Variant: ManualLayout
    Properties:
      Width: =Parent.TemplateWidth   # CORRECT — per-template width
      Height: =44
      # Width: =Parent.Width        # WRONG — full gallery width, not template width
```

This applies to any gallery template root container. Always use `Parent.TemplateWidth` / `Parent.TemplateHeight` for the template item's own dimensions.

**Inside each gallery template item:**

- `tabLabel` — `Label@2.5.1` showing `ThisItem.TabName`. Color and FontWeight switch on active state:
  ```yaml
  Color: =If(varActiveTab = ThisItem.TabID, RGBA(30,30,35,1), RGBA(140,140,150,1))
  FontWeight: =If(varActiveTab = ThisItem.TabID, FontWeight.Semibold, FontWeight.Normal)
  ```

- `tabUnderline` — `Rectangle@2.3.0`, 2px tall, visible only on the active tab:
  ```yaml
  Visible: =varActiveTab = ThisItem.TabID
  Height: =2
  Y: =Parent.TemplateHeight - 2
  ```

- `tabOverlay` — transparent `Classic/Button@2.2.0` covering the full template cell. Sets the active tab on click:
  ```yaml
  OnSelect: =Set(varActiveTab, ThisItem.TabID)
  Fill: =RGBA(0,0,0,0)
  HoverFill: =RGBA(0,0,0,0.03)
  PressedFill: =RGBA(0,0,0,0.06)
  Text: =""
  ```

**Wiring tab content visibility:**

Each content section (container, form group, gallery, etc.) that belongs to a specific tab sets its `Visible` property to a comparison against `varActiveTab`:

```yaml
# Content for Tab 1 (General)
Visible: =varActiveTab = 1

# Content for Tab 2 (Details)
Visible: =varActiveTab = 2

# Content for Tab 3 (Settings)
Visible: =varActiveTab = 3
```

This works on any control or container. When `varActiveTab` changes (via the tab overlay's `OnSelect`), only the matching section is shown. All other sections hide instantly — no navigation required.

**Rules:**
- Always initialize `varActiveTab` in the screen's `OnVisible` so the first tab is selected on load.
- Use integer `TabID` values (1, 2, 3...) — not strings — for fast equality checks.
- Place all tab content sections as siblings inside the same parent container (typically the main vertical content area). Only one is visible at a time, so they effectively occupy the same space.
- If the tab bar colors need to match a custom theme, update the `tabLabel` Color, the `tabUnderline` Fill, and the `tabOverlay` hover/pressed fills accordingly.
- The tab divider (`tabDivider`) is a 1px `Rectangle@2.3.0` below the gallery, providing a visual baseline. Always include it.

---

### KPI Card Gallery Pattern

Auto-fit horizontal gallery where cards evenly divide available width regardless of count.

```yaml
- kpiGallery:
    Control: Gallery@2.15.0
    Variant: Horizontal
    Properties:
      LayoutMinHeight: =0
      LayoutMinWidth: =0
      TemplatePadding: =0
      TemplateSize: =Self.Width / Self.AllItemsCount - 1
      Items: |-
        =Table(
            {Title: "General Safety", Value: 12, Total: 17, Color: RGBA(191,168,255,1), Icon: "<svg ...>"},
            {Title: "Fire Safety",    Value: 8,  Total: 10, Color: RGBA(254,230,194,1), Icon: "<svg ...>"},
            {Title: "Electrical",     Value: 5,  Total: 9,  Color: RGBA(237,250,84,1),  Icon: "<svg ...>"},
            {Title: "Ward",           Value: 3,  Total: 6,  Color: RGBA(82,222,229,1),  Icon: "<svg ...>"}
        )
    Children:
      - kpiCardRoot:
          Control: GroupContainer@1.5.0
          Variant: AutoLayout
          Properties:
            Fill: =ThisItem.Color
            Width: =Parent.TemplateWidth * 90%
            Height: =Parent.TemplateHeight * 90%
            Y: =(Parent.TemplateHeight - Self.Height)/2
            LayoutDirection: =LayoutDirection.Vertical
            PaddingTop: =20
            PaddingBottom: =20
            PaddingLeft: =20
            PaddingRight: =20
            RadiusBottomLeft: =20
            RadiusBottomRight: =20
            RadiusTopLeft: =20
            RadiusTopRight: =20
            DropShadow: =DropShadow.None
```

> `TemplateSize: =Self.Width / Self.AllItemsCount - 1` — the `-1` prevents rounding-induced horizontal scroll.

---

### Custom Progress Bar Pattern

Two nested containers. No native Progress control needed.

```yaml
- progressTrack:
    Control: GroupContainer@1.5.0
    Variant: ManualLayout
    Properties:
      Fill: =RGBA(0,0,0,0.2)      # translucent dark track
      Height: =4
      LayoutMinWidth: =0
      DropShadow: =DropShadow.None
      RadiusBottomLeft: =10
      RadiusBottomRight: =10
      RadiusTopLeft: =10
      RadiusTopRight: =10
    Children:
      - progressFill:
          Control: GroupContainer@1.5.0
          Variant: ManualLayout
          Properties:
            Fill: =RGBA(42,43,71,1)   # filled portion color
            Height: =Parent.Height
            Width: =Parent.Width * (ThisItem.Completed / ThisItem.Total)
            DropShadow: =DropShadow.None
            RadiusBottomLeft: =10
            RadiusBottomRight: =10
            RadiusTopLeft: =10
            RadiusTopRight: =10
```

---

### Data Table Pattern (Header + Gallery)

A custom table built from a static header row and a vertical gallery. Use matching `FillPortions` on both to keep columns aligned.

```yaml
- tableWrapper:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      LayoutDirection: =LayoutDirection.Vertical
      BorderColor: =RGBA(131,141,157,1)
      BorderThickness: =1
      RadiusBottomLeft: =10
      RadiusBottomRight: =10
      RadiusTopLeft: =10
      RadiusTopRight: =10
      DropShadow: =DropShadow.None
    Children:
      - tableHeader:
          Control: GroupContainer@1.5.0
          Variant: AutoLayout
          Properties:
            Fill: =RGBA(246,246,250,1)
            FillPortions: =0
            Height: =50
            LayoutDirection: =LayoutDirection.Horizontal
            PaddingLeft: =10
            PaddingRight: =10
            DropShadow: =DropShadow.None
            RadiusBottomLeft: =0
            RadiusBottomRight: =0
            RadiusTopLeft: =0
            RadiusTopRight: =0
          Children:
            - hdrCategory:
                Control: Label@2.5.1
                Properties:
                  AlignInContainer: =AlignInContainer.Stretch
                  FillPortions: =1
                  LayoutMinHeight: =0
                  Color: =RGBA(131,141,157,1)
                  Font: =Font.'Lato Light'
                  FontWeight: =FontWeight.Semibold
                  Size: =8
                  Text: ="Category"
            - hdrTitle:
                Control: Label@2.5.1
                Properties:
                  AlignInContainer: =AlignInContainer.Stretch
                  FillPortions: =2            # wider column
                  LayoutMinHeight: =0
                  Color: =RGBA(131,141,157,1)
                  Font: =Font.'Lato Light'
                  FontWeight: =FontWeight.Semibold
                  Size: =8
                  Text: ="Title"
            - hdrDate:
                Control: Label@2.5.1
                Properties:
                  AlignInContainer: =AlignInContainer.Stretch
                  FillPortions: =1
                  LayoutMinHeight: =0
                  LayoutMinWidth: =90
                  Color: =RGBA(131,141,157,1)
                  Font: =Font.'Lato Light'
                  FontWeight: =FontWeight.Semibold
                  Size: =8
                  Text: ="Date"
            - hdrActionSpacer:
                Control: Rectangle@2.3.0
                Properties:
                  AlignInContainer: =AlignInContainer.Stretch
                  Fill: =RGBA(0,0,0,0)
                  LayoutMinHeight: =0
                  Width: =50              # matches action button column width in rows
      - tableDataContainer:
          Control: GroupContainer@1.5.0
          Variant: AutoLayout
          Properties:
            LayoutDirection: =LayoutDirection.Vertical
            PaddingLeft: =10
            PaddingRight: =10
            DropShadow: =DropShadow.None
            RadiusBottomLeft: =0
            RadiusBottomRight: =0
            RadiusTopLeft: =0
            RadiusTopRight: =0
          Children:
            - tableGallery:
                Control: Gallery@2.15.0
                Variant: Vertical
                Properties:
                  Items: =colSampleData
                  TemplateSize: =45
                  TemplatePadding: =0
                  LayoutMinHeight: =0
                  LayoutMinWidth: =0
                  DelayItemLoading: =false
                Children:
                  - rowContainer:
                      Control: GroupContainer@1.5.0
                      Variant: AutoLayout
                      Properties:
                        Height: =Parent.TemplateHeight
                        Width: =Parent.TemplateWidth
                        LayoutDirection: =LayoutDirection.Horizontal
                        DropShadow: =DropShadow.None
                        RadiusBottomLeft: =0
                        RadiusBottomRight: =0
                        RadiusTopLeft: =0
                        RadiusTopRight: =0
                      Children:
                        - cellCategory:
                            Control: Label@2.5.1
                            Properties:
                              AlignInContainer: =AlignInContainer.Stretch
                              FillPortions: =1     # MUST match header FillPortions
                              LayoutMinHeight: =0
                              Size: =9
                              Font: =Font.Lato
                              Text: =ThisItem.Category
                        - cellTitle:
                            Control: Label@2.5.1
                            Properties:
                              AlignInContainer: =AlignInContainer.Stretch
                              FillPortions: =2     # MUST match header FillPortions
                              LayoutMinHeight: =0
                              Size: =9
                              Font: =Font.Lato
                              Text: =ThisItem.Title
                        - cellDate:
                            Control: Label@2.5.1
                            Properties:
                              AlignInContainer: =AlignInContainer.Stretch
                              FillPortions: =1     # MUST match header FillPortions
                              LayoutMinHeight: =0
                              LayoutMinWidth: =90
                              Size: =9
                              Font: =Font.Lato
                              Text: =Text(ThisItem.Date, "mm/dd/yyyy")
                        - cellAction:
                            Control: GroupContainer@1.5.0
                            Variant: AutoLayout
                            Properties:
                              FillPortions: =0
                              Width: =50           # MUST match header spacer width
                              LayoutDirection: =LayoutDirection.Horizontal
                              LayoutAlignItems: =LayoutAlignItems.Center
                              LayoutMinHeight: =0
                              DropShadow: =DropShadow.None
                              RadiusBottomLeft: =0
                              RadiusBottomRight: =0
                              RadiusTopLeft: =0
                              RadiusTopRight: =0
                  - rowDivider:
                      Control: Rectangle@2.3.0
                      Properties:
                        Fill: =RGBA(246,246,250,1)
                        Height: =1
                        Width: =Parent.TemplateWidth
                        Y: =Parent.TemplateHeight - Self.Height
                        OnSelect: =Select(Parent)
```

> **Column alignment rule:** `FillPortions` values in the header row and data gallery template MUST be identical for each column. Mismatched values cause columns to not line up.
> **Row divider:** `Y: =Parent.TemplateHeight - Self.Height` pins the 1px Rectangle to the bottom of each gallery row.

---

### Status Badge Pattern

A pill-shaped label container where the border drives the color theme and the fill is auto-derived.

```yaml
- statusBadge:
    Control: GroupContainer@1.5.0
    Variant: ManualLayout
    Properties:
      BorderColor: |-
        =Switch(
            Text(ThisItem.Status),
            "Active",   RGBA(82,222,229,1),
            "Pending",  RGBA(254,230,194,1),
            "Closed",   RGBA(191,168,255,1),
                        RGBA(192,194,197,1)    # default
        )
      BorderThickness: =2
      Fill: =ColorFade(Self.BorderColor, 80%)   # auto-derives tinted fill from border
      Height: =25
      Width: =80
      RadiusBottomLeft: =10
      RadiusBottomRight: =10
      RadiusTopLeft: =10
      RadiusTopRight: =10
      DropShadow: =DropShadow.None
    Children:
      - badgeLabel:
          Control: Label@2.5.1
          Properties:
            Align: =Align.Center
            Color: =RGBA(35,36,47,1)
            Font: =Font.Lato
            FontWeight: =FontWeight.Semibold
            Height: =Parent.Height
            Width: =Parent.Width
            Size: =9
            Text: =ThisItem.Status
```

> `Fill: =ColorFade(Self.BorderColor, 80%)` — the fill is always a soft tinted version of the border. Never hardcode both separately; derive one from the other.

---

### Circular Container / Avatar Pattern

Force a perfect circle: set all four radius values equal to half the width.

```yaml
- avatarContainer:
    Control: GroupContainer@1.5.0
    Variant: ManualLayout
    Properties:
      Width: =40
      Height: =Self.Width          # always square
      BorderColor: =RGBA(42,43,71,1)
      BorderThickness: =2
      RadiusTopLeft: =20           # = Width / 2
      RadiusTopRight: =20
      RadiusBottomLeft: =20
      RadiusBottomRight: =20
      DropShadow: =DropShadow.None
    Children:
      - avatarImage:
          Control: Image@2.2.3
          Properties:
            Image: =User().Image    # or ThisItem.Photo
            Width: =Parent.Width
            Height: =Parent.Height
            RadiusTopLeft: =20
            RadiusTopRight: =20
            RadiusBottomLeft: =20
            RadiusBottomRight: =20
            X: =(Parent.Width - Self.Width)/2
            Y: =(Parent.Height - Self.Height)/2
```

---

### Self-Centering Formulas (ManualLayout)

In `ManualLayout` containers, use these to center a child without knowing exact dimensions:

```yaml
# Vertical center:
Y: =(Parent.Height - Self.Height)/2

# Horizontal center:
X: =(Parent.Width - Self.Width)/2

# Both centered:
X: =(Parent.Width - Self.Width)/2
Y: =(Parent.Height - Self.Height)/2
```

---

### String Interpolation

Use `$"..."` to embed expressions directly in strings — no `&` concatenation needed:

```yaml
Text: =$"You have {outstandingCount} tasks left this month!"
Text: =$"{ThisItem.Completed} of {ThisItem.Total} | {Text(ThisItem.Completed/ThisItem.Total * 100, '##')}%"
Text: =$"Welcome, {User().FullName}"
```

> **Rule:** If a `$"..."` string contains `:` or `#` inside the literal text (not inside `{}`), use `|-` multiline format to avoid PA1001.

---

### User() Functions

Built-in PA functions to get the currently signed-in user. No data source connection required.

```yaml
Image: =User().Image      # profile photo (use in Image control)
Text: =User().FullName    # display name
Text: =User().Email       # email address
```

---

### Color Palettes

**Dark Navy / Indigo palette** (from production app):
| Role | Value |
|---|---|
| Primary accent / button fill | `RGBA(42,43,71,1)` |
| App / screen background | `RGBA(246,246,250,1)` |
| White card background | `RGBA(255,255,255,1)` |
| Active menu item background | `RGBA(240,239,248,1)` |
| Inactive icon / muted text | `RGBA(192,194,197,1)` |
| Divider / border | `RGBA(214,221,224,1)` |
| Table header text | `RGBA(131,141,157,1)` |
| Dark body text | `RGBA(35,36,47,1)` |

**Category / status accent colors** (pastel):
| Color | Value |
|---|---|
| Purple | `RGBA(191,168,255,1)` |
| Orange | `RGBA(254,230,194,1)` |
| Yellow | `RGBA(237,250,84,1)` |
| Teal | `RGBA(82,222,229,1)` |

**Standard skill palette** (blue accent):
| Role | Value |
|---|---|
| Background | `RGBA(35,36,47,1)` |
| Accent | `RGBA(0,142,210,1)` |

---



See `controls-index.md` for the full control inventory (40+ controls). The table below lists the most commonly generated controls.

**Classic controls** (preferred by default — stable, highly customizable):

| Control | Version | Notes |
|---|---|---|
| `Classic/Button` | `@2.2.0` | Classic button — use for interactive/styled buttons |
| `Classic/TextInput` | `@2.3.2` | Classic text input — supports `Default`, `Mode`, `HintText`, `MaxLength` |
| `Classic/DropDown` | `@2.3.1` | Classic dropdown — simple list selection, supports `Default` |
| `Classic/ComboBox` | `@2.4.0` | Classic combobox — searchable + multi-select, supports `Default` |
| `Classic/CheckBox` | `@2.1.0` | Classic checkbox — use for checkboxes (NOT Rectangle) |
| `Classic/Radio` | `@2.3.0` | Classic radio group — supports `Default` |
| `Classic/Toggle` | `@2.1.0` | Classic toggle — supports `Default`, `TrueFill`, `FalseFill` |
| `Classic/Slider` | `@2.1.0` | Classic slider — supports `Default`, `Min`, `Max` |
| `Classic/DatePicker` | `@2.6.0` | Classic date picker — supports `DefaultDate` |
| `Classic/Icon` | `@2.5.0` | Classic icon — 500+ icons via `Icon.xxx` enum |

**Common controls** (no Classic/Modern split):

| Control | Version | Notes |
|---|---|---|
| `GroupContainer` | `@1.5.0` | Variants: `AutoLayout`, `ManualLayout` |
| `Gallery` | `@2.15.0` | Variants: `Vertical`, `Horizontal` |
| `Label` | `@2.5.1` | No border radius support |
| `Image` | `@2.2.3` | Supports radius via `RadiusTopLeft/Right/BottomLeft/Right` |
| `Rectangle` | `@2.3.0` | **No border radius support** — use GroupContainer for rounded shapes |
| `Circle` | `@2.3.0` | Circle shape |

**Previous-generation modern controls** (separate — do NOT rename to Classic or Modern):

| Control | Version | Notes |
|---|---|---|
| `TextInput` | `@0.0.54` | No `Default` property — use `Classic/TextInput@2.3.2` if `Default` needed |
| `ComboBox` | `@0.0.51` | No `SearchField` property — search is built-in |
| `Radio` | `@0.0.25` | No `Default` property — use `Classic/Radio@2.3.0` if `Default` needed |
| `DatePicker` | `@0.0.46` | No border radius support |
| `NumberInput` | `@2.9.12` | No border radius support |
| `Button` | `@0.0.45` | Modern Fluent button |

**Always use the exact name and version number shown above.** For the full list including modern controls, see `controls-index.md`. Versions drift as PA updates — if PA2105 fires, apply the VERSION SELF-ANNEALING RULE below. Note: PA2105 is a warning (not an error) and PA auto-upgrades to the current version, so it is non-fatal but must be corrected immediately to keep the skill accurate.

---

## ⚠️ PA2108 TRAP TABLE — Known Invalid Properties

Using any property in this table will cause a PA2108 error. **Never use these.**

| Control | Invalid Property — NEVER USE | Correct Approach |
|---|---|---|
| `TextInput@0.0.54` | `Default` | Omit it, or switch to `Classic/TextInput@2.3.2` which supports `Default` |
| `ComboBox@0.0.51` | `SearchField` | Omit it — search is built-in, no configuration needed |
| `Radio@0.0.25` | `Default` | Omit it, or switch to `Classic/Radio@2.3.0` which supports `Default` |
| `DatePicker@0.0.46` | `Default` | Omit it, or switch to `Classic/DatePicker@2.6.0` (`DefaultDate` property) |
| `Rectangle@2.3.0` | `RadiusTopLeft`, `RadiusTopRight`, `RadiusBottomLeft`, `RadiusBottomRight` | Rectangle does NOT support border radius. Wrap in `GroupContainer` with Fill+radius for a rounded shape instead |
| `Label@2.5.1` | `RadiusTopLeft`, `RadiusTopRight`, `RadiusBottomLeft`, `RadiusBottomRight` | Label does NOT support border radius. Wrap in `GroupContainer` with Fill+radius |
| `Gallery@2.15.0` | `Default` | Galleries have no Default — set pre-selection via `UpdateContext` or `Set` |
| `Classic/TextInput@2.3.2` | `Mode: =TextInputMode.___` | Wrong enum prefix. The correct enum is `TextMode`, not `TextInputMode`. Use `=TextMode.SingleLine`, `=TextMode.MultiLine`, or `=TextMode.Password` |

---

## VERSION SELF-HEALING RULE

If Power Apps Studio reports PA2105 for any control (e.g., "GroupContainer@1.5.0 has a version older than current version of @1.6.0"):

1. Note the control name and its new version from the PA2105 message.
2. Update the version in the **VALID CONTROL TYPES table** above.
3. Replace **ALL occurrences** of the old version string in:
   - This file (`pa-yaml-rules.md`) — table entry and every inline code example
   - `controls-reference.md` — section heading and any code examples
   - Every `.yaml` file in the `templates/` directory
4. Use `replace_all: true` on each file for efficiency.

The same PA2105 warning must never occur twice for the same control. This rule applies to every control, not just `GroupContainer`.

---

## WRAP=FALSE ON SINGLE-LINE LABELS

Every `Label@2.5.1` that is expected to display on **one line** MUST have `Wrap: =false` set explicitly.

Power Apps defaults `Wrap` to `true`. On a narrow container (logo area, sidebar nav item, tab bar, status badge), a label without `Wrap: =false` will split to two lines when the text fits on one line visually but the assigned width is slightly too narrow.

**Always add `Wrap: =false` to:**
- Logo text labels
- Navigation item labels (sidebar and top nav)
- Tab bar item labels
- Column headers in tables and galleries
- Status badge text
- KPI metric values and card titles
- Breadcrumb text labels
- Button-adjacent text labels

**Omit `Wrap` (leave as default `true`) only for:**
- Multi-line description fields
- Body copy paragraphs
- Error message labels where text length is variable and unknown

**Sizing rule for single-line labels in horizontal containers:**
Do NOT set a fixed `Width` that barely fits the expected text — even 1px too narrow triggers wrapping. Instead use `FillPortions: =1` (fills remaining container space) or `Width: =Parent.Width` with `Wrap: =false`. Only use a fixed `Width` when the label is a fixed-size element like a status pill or badge.

---

## OVERLAY BUTTON PATTERN — MANDATORY MANUALLAYOUT

A **transparent overlay button** is a `Classic/Button@2.2.0` with:
- `Fill: =RGBA(0,0,0,0)` — fully transparent
- `BorderStyle: =BorderStyle.None` / `BorderThickness: =0`
- `Width: =Parent.Width` and `Height: =Parent.Height` (or matching the parent exactly)
- Positioned at `X: =0`, `Y: =0` to intercept taps across the entire parent area

**RULE: Any `GroupContainer` that contains a transparent overlay button as a child MUST be `Variant: ManualLayout`.** Remove `LayoutDirection` and all AutoLayout properties from that container.

**Why:** In an AutoLayout container, children stack sequentially on the layout axis. An overlay button with `Height: =Parent.Height` (fixed, full container height) stacks alongside siblings and consumes the entire container height, leaving sibling controls with 0px height. They become invisible. The overlay appears to be the only thing in the container.

**How to convert:**
1. Change `Variant: AutoLayout` → `Variant: ManualLayout`
2. Remove: `LayoutDirection`, `LayoutAlignItems`, `LayoutJustifyContent`, `LayoutGap`, `LayoutWrap`, `LayoutMinWidth`, `LayoutMinHeight`, `PaddingLeft`, `PaddingRight`
3. Remove `AlignInContainer` from all children (not applicable in ManualLayout)
4. Add explicit `X`, `Y`, `Width`, `Height` to every child
5. Place the overlay button **LAST** in the `Children:` list — controls render in tree order, so last = on top

**Standard ManualLayout overlay pattern:**
```yaml
- itemContainer:          # ManualLayout — has overlay child
    Control: GroupContainer@1.5.0
    Variant: ManualLayout
    Properties:
      Width: =Parent.Width
      Height: =44
      Fill: =RGBA(0,0,0,0)
      DropShadow: =DropShadow.None
    Children:
      - itemLabel:        # visible content — explicit X/Y/W/H
          X: =12
          Y: =0
          Width: =Parent.Width - 24
          Height: =40
      - itemUnderline:    # decorative element — pinned to bottom
          X: =0
          Y: =42
          Width: =Parent.Width
          Height: =2
      - itemOverlay:      # LAST — renders on top of all siblings
          X: =0
          Y: =0
          Width: =Parent.Width
          Height: =44
```

**This pattern applies universally:** tab bar items, gallery nav items, card click areas, list row hit areas, and any other container where a click must be captured over visible content.

---

## SCROLL CONTAINER CHILDREN SIZING RULE

When a `GroupContainer` has `LayoutOverflowY: =LayoutOverflow.Scroll`, its **direct children that contain the scrollable content** MUST use `FillPortions: =0` (or have no `FillPortions` set). **NEVER** use `FillPortions: =1` on content that should be scrollable.

**Why `FillPortions: =1` defeats scrolling:**
`FillPortions: =1` sizes the child to exactly fill the scroll viewport height. The child cannot grow beyond the viewport — content is pinned to viewport size and overflows are clipped. The scroll viewport never activates.

**Correct scroll pattern:**
```yaml
- scrollableBody:          # bounded container — has LayoutOverflowY
    Variant: AutoLayout
    FillPortions: =1       # bounded by parent (scroll viewport)
    LayoutOverflowY: =LayoutOverflow.Scroll
    Children:
      - contentWrapper:    # FillPortions: =0 — must have explicit Height
          FillPortions: =0
          Height: =formCard.Height + 48   # REQUIRED: explicit Height = child.Height + padding
          AlignInContainer: =AlignInContainer.Center  # horizontal centering if needed
          Children:
            - formCard:    # the actual content (taller than viewport = scroll activates)
```

**Wrong pattern (content clipped, scroll never shows):**
```yaml
- scrollableBody:
    FillPortions: =1
    LayoutOverflowY: =LayoutOverflow.Scroll
    Children:
      - contentWrapper:
          FillPortions: =1   # WRONG — pins child to viewport height. Scroll never activates.
```

**The Three Sizing Contracts (reference):**

Every control in an AutoLayout container operates under one contract:

| Contract | Properties | When to use |
|----------|-----------|-------------|
| Fixed | `Height: =N` or `Width: =N` with `FillPortions: =0` | Control is always this size |
| Flexible | `FillPortions: =N` (N > 0) | Control fills proportional share of remaining space |
| Manual | `X`, `Y`, `Width`, `Height` | Parent is ManualLayout; control is absolutely positioned |

**Critical rule:** Fixed children in the same AutoLayout axis are allocated first. Flexible children divide what remains. If fixed children sum to ≥ parent size, flexible children collapse to 0px height/width and become invisible. Always verify that fixed children do not fully consume the parent before assigning `FillPortions: =1` to a sibling.

---

## FILLPORTIONS AND HEIGHT — THE COMPLETE SIZING RULE

This rule applies to **all children of AutoLayout containers**. The same logic applies to `Width` inside Horizontal direction containers.

| FillPortions value | Height required? | PA behaviour |
|---|---|---|
| `FillPortions > 0` | **No — do NOT set Height** | PA computes height proportionally from available parent space |
| `FillPortions = 0` or not set (default = 0) | **Yes — MUST set explicit Height** | PA uses static height; if absent, **defaults to 200px** |

**200px is never intentionally correct** for layout containers. The default fires silently — no error, no warning — containers just render at the wrong size.

### Correct pattern (FillPortions = 0):
```yaml
topBarInner:
  FillPortions: =0
  Height: =pageTitle.Height + pageBreadcrumb.Height + 28   # padding 12+12 + gap 4
```

### Correct pattern (FillPortions > 0):
```yaml
scrollableBody:
  FillPortions: =1
  # Height intentionally absent — PA computes it from FillPortions
  LayoutOverflowY: =LayoutOverflow.Scroll
```

### Wrong pattern (triggers 200px default):
```yaml
topBarInner:
  FillPortions: =0
  # Height missing → PA defaults to 200px
```

### Formula template for Vertical AutoLayout containers:
```
Height: =PaddingTop + child1.Height + LayoutGap + child2.Height + ... + PaddingBottom
```

For containers with many children, chain the field group Heights:
```
Height: =fieldGroup1.Height + fieldGroup2.Height + fieldGroup3.Height + (PaddingTop + PaddingBottom + (n-1) * LayoutGap)
```

### Circle/square controls:
Any control with equal `Width` and `Height` (circular avatar, square icon) MUST have `AlignInContainer: =AlignInContainer.Center`. The default `AlignInContainer` is `Stretch`, which overrides even a parent `LayoutAlignItems: Center` — the control stretches on the cross-axis, breaking its aspect ratio (circle → oval).
