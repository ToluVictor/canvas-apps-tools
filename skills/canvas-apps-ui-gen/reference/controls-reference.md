# Canvas App Controls Reference

Focused reference for property names and valid enum values used in YAML generation.
Use `controls-index.md` to resolve the correct control type string for any UI element.

**Structure:**
- Section A: Common properties (apply to most controls — listed once)
- Section B: Per-control unique properties (only what differs from Section A)
- Section C: PA2108 Trap Table (known invalid properties — never use)

---

## SECTION A — Common Properties

These apply to most controls. Do not repeat them in per-control sections.

### Layout & Sizing

| Property | Type | Notes |
|---|---|---|
| `Height` | Number/Formula | Distance between top and bottom edges |
| `Width` | Number/Formula | Distance between left and right edges |
| `X` | Number/Formula | Left edge distance from parent (ManualLayout only) |
| `Y` | Number/Formula | Top edge distance from parent (ManualLayout only) |
| `Visible` | Boolean | `=true` or `=false` or formula |
| `DisplayMode` | Enum | `=DisplayMode.Edit`, `=DisplayMode.View`, `=DisplayMode.Disabled` |

### Container child layout (set on the CHILD, not the container)

| Property | Type | Notes |
|---|---|---|
| `AlignInContainer` | Enum | `=AlignInContainer.Start`, `.Center`, `.End`, `.Stretch`, `.SetByContainer`. **Always set explicitly on every child — never rely on the parent's `LayoutAlignItems`.** `Stretch` overrides fixed cross-axis size: `Height` in horizontal containers, `Width` in vertical containers. Use `Center`/`Start`/`End` to preserve a fixed size (e.g., circle avatars, icons, badges). |
| `FillPortions` | Number | `=0` (fixed size), `=1`, `=2`, etc. (proportional flex fill) |
| `LayoutMinWidth` | Number | Floor for flex width; set `=0` when using `AlignInContainer.Stretch` |
| `LayoutMinHeight` | Number | Floor for flex height; usually `=0` to `=30` |
| `LayoutMaxWidth` | Number | Cap for flex width |

### Color & Border

| Property | Type | Notes |
|---|---|---|
| `Fill` | Color | `=RGBA(r,g,b,a)` or `=Color.White` etc. |
| `Color` | Color | Text/foreground color |
| `BorderColor` | Color | `=RGBA(r,g,b,a)` |
| `BorderStyle` | Enum | `=BorderStyle.None`, `.Solid`, `.Dashed`, `.Dotted` |
| `BorderThickness` | Number | Pixels |

### Interaction States

| Property | Type | Notes |
|---|---|---|
| `HoverFill` | Color | Background on hover |
| `HoverColor` | Color | Text color on hover |
| `HoverBorderColor` | Color | Border on hover |
| `PressedFill` | Color | Background when pressed/clicked |
| `PressedColor` | Color | Text color when pressed |
| `PressedBorderColor` | Color | Border when pressed |
| `DisabledFill` | Color | Background when disabled |
| `DisabledColor` | Color | Text when disabled |
| `DisabledBorderColor` | Color | Border when disabled |
| `FocusedBorderColor` | Color | Border when focused (keyboard nav) |
| `FocusedBorderThickness` | Number | Pixels |

### Interaction & Accessibility

| Property | Type | Notes |
|---|---|---|
| `OnSelect` | Behavior | Formula to run on tap/click |
| `Tooltip` | String | Tooltip text |
| `TabIndex` | Number | Tab order; `=0` included, `=-1` excluded |
| `AccessibleLabel` | String | Screen reader label |

### Padding

| Property | Type | Notes |
|---|---|---|
| `PaddingTop` | Number | Pixels |
| `PaddingBottom` | Number | Pixels |
| `PaddingLeft` | Number | Pixels |
| `PaddingRight` | Number | Pixels |

---

## SECTION B — Per-Control Unique Properties

Only properties that are unique to the control, not in Section A above.

---

## GroupContainer@1.5.0

Primary layout container. Variants: `AutoLayout` (children positioned by layout rules) and `ManualLayout` (children positioned by X/Y).

**Key difference from other controls:** `DropShadow` default is NOT None — always set it explicitly.

| Property | Type | Valid Values |
|---|---|---|
| `LayoutDirection` | Enum | `=LayoutDirection.Horizontal`, `=LayoutDirection.Vertical` |
| `LayoutAlignItems` | Enum | `=LayoutAlignItems.Start`, `.Center`, `.End`, `.Stretch` |
| `LayoutJustifyContent` | Enum | `=LayoutJustifyContent.Start`, `.End`, `.Center`, `.SpaceBetween`, `.SpaceAround`, `.SpaceEvenly` |
| `LayoutGap` | Number | Pixels between children |
| `LayoutWrap` | Boolean | `=true` wraps children to new row/column |
| `LayoutOverflowX` | Enum | `=LayoutOverflow.Scroll`, `=LayoutOverflow.Hidden` |
| `LayoutOverflowY` | Enum | `=LayoutOverflow.Scroll`, `=LayoutOverflow.Hidden` |
| `LayoutMinWidth` | Number | **Default is 250 — override to `=0` when using Stretch** |
| `LayoutMinHeight` | Number | **Default is 100 — override to `=0` when needed** |
| `DropShadow` | Enum | `=DropShadow.None`, `.Light`, `.Regular`, `.Bold`, `.ExtraBold` — **always set explicitly** |
| `RadiusTopLeft` | Number | Pixels — border radius top-left corner |
| `RadiusTopRight` | Number | Pixels — border radius top-right corner |
| `RadiusBottomLeft` | Number | Pixels — border radius bottom-left corner |
| `RadiusBottomRight` | Number | Pixels — border radius bottom-right corner |

> **Status badge pattern (rounded GroupContainer):** Use a ManualLayout GroupContainer with `Fill: =ColorFade(Self.BorderColor, 80%)` — the fill is automatically derived from the border color at 80% fade. Set `BorderColor` via a Switch/If based on status value. **Never hardcode both Fill and BorderColor separately** — the ColorFade derivation keeps them visually consistent:
> ```yaml
> Fill: =ColorFade(badgeName.BorderColor, 80%)
> BorderColor: =Switch(ThisItem.Status, "Active", RGBA(0,168,120,1), "Pending", RGBA(255,140,0,1), RGBA(160,160,180,1))
> BorderThickness: =1
> RadiusBottomLeft: =12
> RadiusTopLeft: =12
> RadiusBottomRight: =12
> RadiusTopRight: =12
> ```

---

## Gallery@2.15.0

Scrollable list. Variants: `Vertical`, `Horizontal`.

| Property | Type | Notes |
|---|---|---|
| `Items` | Table/Collection | Data source or `Table({...},{...})` inline |
| `Default` | Record/Formula | Initial selected item or record from the data source — **this IS valid on Gallery** |
| `TemplateSize` | Number | **Setter only** — sets item height (Vertical gallery) or width (Horizontal gallery). Cannot be read back via `Self.TemplateSize` — use `Self.TemplateHeight` or `Self.TemplateWidth` instead |
| `TemplateHeight` | Number (read) | Effective item height — use in formulas: `Height: =Self.AllItemsCount * Self.TemplateHeight` |
| `TemplateWidth` | Number (read) | Effective item width — use in formulas for horizontal galleries |
| `TemplatePadding` | Number | Gap between templates — usually `=0` |
| `TemplateFill` | Color | Background color of each gallery template item |
| `WrapCount` | Number | Items per row (Vertical) or per column (Horizontal) |
| `LayoutMinWidth` | Number | **Must override — default 250 causes overflow** |
| `LayoutMinHeight` | Number | **Must override — default 100 causes overflow** |
| `DelayItemLoading` | Boolean | `=false` for immediate load |
| `Transition` | Enum | `=Transition.None`, `.Pop`, `.Push`, `.Fade` |
| `ShowScrollbar` | Boolean | |
| `ShowNavigation` | Boolean | `=true` shows navigation arrows at each end |
| `Selectable` | Boolean | `=true` allows gallery items to be selected (default true) |

Inside gallery template: `Parent.TemplateHeight` / `Parent.TemplateWidth` for dimensions; `ThisItem.FieldName` for data. **Never use `Parent.TemplateSize`** — it is write-only and cannot be read.

---

## Label@2.5.1

**No border radius support** — wrap in GroupContainer to achieve rounded corners.

| Property | Type | Notes |
|---|---|---|
| `Text` | String/Formula | `="Hello"` or `=ThisItem.Name` |
| `Font` | Enum | `=Font.Lato`, `=Font.'Lato Black'`, `=Font.'Lato Light'`, `=Font.'Open Sans'`, `=Font.Arial` |
| `Size` | Number | Font size in points (8–28 typical) |
| `FontWeight` | Enum | `=FontWeight.Normal`, `.Semibold`, `.Bold`, `.Lighter` |
| `Align` | Enum | `=Align.Left`, `.Center`, `.Right`, `.Justify` |
| `VerticalAlign` | Enum | `=VerticalAlign.Top`, `.Middle`, `.Bottom` |
| `AutoHeight` | Boolean | `=true` to auto-grow height to fit text content. **Required on every Label inside an AutoLayout field group container** — without it PA Studio shows labels at 200px default height, and the parent container cannot correctly compute its Height from children. |
| `Wrap` | Boolean | `=true` allows text to wrap to multiple lines. Use together with `AutoHeight: =true` for labels whose text may span more than one line. |

---

## Image@2.2.3

Supports border radius directly (unlike Label and Rectangle).

| Property | Type | Notes |
|---|---|---|
| `Image` | Image/Formula | `=SomeImage`, `=Blank()` (use Fill for color), or SVG formula. **Always set `=Blank()` when using the Image control as a colored rectangle/background — omitting it shows the ugly "SampleImage" placeholder.** |
| `ImagePosition` | Enum | `=ImagePosition.Fill`, `.Fit`, `.Stretch`, `.Center`, `.Tile` |
| `RadiusTopLeft` | Number | Supported directly |
| `RadiusTopRight` | Number | Supported directly |
| `RadiusBottomLeft` | Number | Supported directly |
| `RadiusBottomRight` | Number | Supported directly |

SVG pattern:
```yaml
Image: |-
  ="data:image/svg+xml;utf8, " & EncodeUrl(
      Substitute(ThisItem.Icon, "currentColor", "RGBA(246,246,250,1)")
  )
```

---

## Rectangle@2.3.0

Used for dividers and separators only. **Does NOT support border radius.**

| Property | Type | Notes |
|---|---|---|
| `Height` | Number | `=1` for a hairline divider |
| `Width` | Formula | Usually `=Parent.TemplateWidth` or `=Parent.Width` |

> **Divider dimension rule:**
> - **Horizontal divider** (inside a vertical AutoLayout container): `Height: =1`, `Width: =Parent.Width`, `AlignInContainer: =AlignInContainer.Stretch`. Never set `Height: =Parent.Height` on a horizontal divider — it fills the entire container and renders as a solid block.
> - **Vertical divider** (inside a horizontal AutoLayout container): `Width: =1`, `Height: =Parent.Height`, `AlignInContainer: =AlignInContainer.Stretch`.
> - Thickness is always 1 or 2 — never more.
>
> **⚠️ No radius support.** `RadiusTopLeft/Right/BottomLeft/Right` cause PA2108. For rounded separators, use a GroupContainer with `Fill` and radius properties.

---

## Circle@2.3.0

| Property | Type | Notes |
|---|---|---|
| `Height` | Number | Diameter |
| `Width` | Number | Diameter (match Height for a true circle) |

---

## Classic/Button@2.2.0

Full-featured classic button. Also used as transparent click overlay.

| Property | Type | Notes |
|---|---|---|
| `Text` | String | Button label; `=""` for invisible overlay |
| `FontWeight` | Enum | `=FontWeight.Bold`, `.Semibold`, `.Normal`, `.Lighter` — use this, not `Bold` |
| `Italic` | Boolean | |
| `Underline` | Boolean | |
| `Font` | Enum | See Label font enum |
| `Size` | Number | Font size |
| `Align` | Enum | `=Align.Left`, `.Center`, `.Right` |
| `VerticalAlign` | Enum | `=VerticalAlign.Top`, `.Middle`, `.Bottom` |
| `AutoDisableOnSelect` | Boolean | `=true` auto-disables button while `OnSelect` is running |
| `RadiusTopLeft` | Number | Supported |
| `RadiusTopRight` | Number | Supported |
| `RadiusBottomLeft` | Number | Supported |
| `RadiusBottomRight` | Number | Supported |

For transparent overlay pattern:
```yaml
Fill: =RGBA(0,0,0,0)
HoverFill: =RGBA(0,0,0,0.08)
PressedFill: =RGBA(0,0,0,0.16)
BorderStyle: =BorderStyle.None
BorderColor: =RGBA(0,0,0,0)
HoverBorderColor: =RGBA(0,0,0,0)
PressedBorderColor: =RGBA(0,0,0,0)
FocusedBorderColor: =ColorFade(Self.Fill, 75%)
FocusedBorderThickness: =2
DisabledBorderColor: =RGBA(0,0,0,0)
Width: =Parent.Width
Height: =Parent.Height
```

> **`FocusedBorderColor` pattern:** Use `=ColorFade(Self.Fill, 75%)` to derive the focus ring from the button's own fill color. This means a transparent overlay button gets a transparent focus ring (invisible), while a colored button gets a soft tinted ring — no hardcoded color needed.

---

## Button@0.0.45 (Modern Button)

Fluent Design button with icon support.

| Property | Type | Notes |
|---|---|---|
| `Text` | String | Button label |
| `Appearance` | Enum | `='ButtonCanvas.Appearance'.Transparent`, `.Subtle`, `.Outline`, `.Secondary`, `.Primary` |
| `Icon` | String | Icon name, e.g., `="Save"`, `="ArrowReset"`, `="Add"` |
| `Layout` | Enum | `='ButtonCanvas.Layout'.IconAfter`, `.IconBefore`, `.IconOnly` |
| `BasePaletteColor` | Color | Theme base palette color override |
| `Font` | Enum | Font family name |
| `FontSize` | Number | Font size in points |
| `FontColor` | Color | Font color |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `FontItalic` | Boolean | |
| `FontUnderline` | Boolean | |
| `FontStrikethrough` | Boolean | |

---

## ModernTextInput@1.0.0

Single-line or multi-line Fluent text input.

| Property | Type | Notes |
|---|---|---|
| `Default` | String/Formula | Initial value when control loads |
| `Text` | String (output) | Current user-entered text |
| `Placeholder` | String | Hint text when empty |
| `Type` | Enum | `=TextInputType.SingleLine`, `.Multiline`, `.Password`, `.Search` |
| `TriggerOutput` | Enum | `=TriggerOutput.Keypress`, `FocusOut`, `Delayed` |
| `MaxLength` | Number | `=0` means unlimited |
| `Required` | Boolean | Enables required validation styling |
| `ValidationState` | Enum | `=ValidationState.None` or `.Error` |
| `Appearance` | Enum | `=Appearance.Filled`, `=Appearance.FilledDarker`, `=Appearance.Outline` |
| `BasePaletteColor` | Color | Theme base palette color override |
| `Align` | Enum | `=Align.Left`, `.Center`, `.Right` |
| `Font` | Enum | Font family name |
| `Size` | Number | Font size in points |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `Italic` | Boolean | |
| `Underline` | Boolean | |
| `Strikethrough` | Boolean | |
| `RadiusTopLeft` | Number | Corner radius |
| `RadiusTopRight` | Number | Corner radius |
| `RadiusBottomLeft` | Number | Corner radius |
| `RadiusBottomRight` | Number | Corner radius |

---

## ModernCombobox@1.0.0

Searchable (optional multi-select) Fluent combobox.

| Property | Type | Notes |
|---|---|---|
| `Items` | Table/Collection | Data source for options |
| `ItemDisplayText` | Formula | Text displayed per item (use `ThisItem`) |
| `DefaultSelectedItems` | Table | Initial selection (must be a table of records) |
| `InputTextPlaceholder` | String | Placeholder when nothing selected |
| `SelectMultiple` | Boolean | `=false` for single selection |
| `IsSearchable` | Boolean | `=true` enables search |
| `DelayOutput` | Boolean | Delays `OnChange` until user clicks outside/presses Enter |
| `SelectedItems` | Table (output) | Selected items table |
| `Selected` | Record (output) | Selected record when `SelectMultiple=false` |
| `SearchText` | String (output) | Current search text |
| `MultiValueDelimiter` | String | Separator for multi selection display |
| `Required` | Boolean | Enables required validation styling |
| `ValidationState` | Enum | `ValidationState.Error` or `ValidationState.None` |
| `Appearance` | Enum | `=Appearance.Filled`, `.FilledDarker`, `.Outline` |
| `BasePaletteColor` | Color | Theme base palette color override |
| `Font` | Enum | Font family name |
| `Size` | Number | Font size in points |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `Italic` | Boolean | |
| `Underline` | Boolean | |
| `Strikethrough` | Boolean | |
| `RadiusTopLeft` | Number | Corner radius |
| `RadiusTopRight` | Number | Corner radius |
| `RadiusBottomLeft` | Number | Corner radius |
| `RadiusBottomRight` | Number | Corner radius |

---

## ModernRadio@1.0.0

Mutually exclusive Fluent radio group.

| Property | Type | Notes |
|---|---|---|
| `Items` | Table/Collection | Options for radio buttons |
| `ItemDisplayText` | Formula | Text displayed per item (use `ThisItem`) |
| `Default` | Record/Formula | Initial selected value (must match an item) |
| `Selected` | Record (output) | Currently selected item |
| `Layout` | Enum | `=Layout.Vertical` or `=Layout.Horizontal` |
| `TriggerOutput` | Enum | Forces output to update |
| `Required` | Boolean | Enables required validation styling |
| `RadioSize` | Number | Size of radio circles |
| `RadioBackgroundFill` | Color | Radio circle background |
| `RadioBorderColor` | Color | Radio circle border color |
| `RadioSelectionFill` | Color | Fill of selected radio circle |
| `HoverColor` | Color | Text color on hover |
| `PressedColor` | Color | Text color on press/click |
| `DisabledColor` | Color | Text color in Disabled mode |
| `RadiusTopLeft` | Number | Corner radius |
| `RadiusTopRight` | Number | Corner radius |
| `RadiusBottomLeft` | Number | Corner radius |
| `RadiusBottomRight` | Number | Corner radius |

---

## CheckBox@0.0.30 (Modern Checkbox)

Fluent checkbox control.

| Property | Type | Notes |
|---|---|---|
| `Checked` | Boolean | Current/initial checked state |
| `Label` | String | Text label |
| `OnCheck` | Behavior | Actions when checked |
| `OnSelect` | Behavior | Actions when selected |
| `OnUncheck` | Behavior | Actions when unchecked |
| `BasePaletteColor` | Color | Theme base palette color override |
| `Font` | Enum | Font family name |
| `FontSize` | Number | Font size in points |
| `FontColor` | Color | Label text color |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `FontItalic` | Boolean | |
| `FontUnderline` | Boolean | |
| `FontStrikethrough` | Boolean | |

---

## ModernDatePicker@1.0.0

Fluent date picker control.

| Property | Type | Notes |
|---|---|---|
| `PlaceHolder` | String | Placeholder when no date selected |
| `Format` | Enum/String | `Short`, `LongAbbreviated`, `YearMonth`, or custom string |
| `Required` | Boolean | Enables required validation styling |
| `EndDate` | Date | Latest selectable date |
| `StartDate` | Date | Earliest selectable date |
| `SelectedDate` | Date (output) | Currently selected date (local time) |
| `StartOfWeek` | Enum | `=StartOfWeek.Sunday`, `.Monday` |
| `IsEditable` | Boolean | If false, user can only pick via calendar |
| `ValidationState` | Enum | `ValidationState.Error` or `ValidationState.None` |
| `OnChange` | Behavior | Actions when the value changes |
| `BasePaletteColor` | Color | Theme base palette color override |
| `Font` | Enum | Font family name |
| `FontSize` | Number | Font size in points |
| `FontColor` | Color | Text color |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `FontItalic` | Boolean | |
| `FontUnderline` | Boolean | |
| `FontStrikethrough` | Boolean | |

---

## Slider@1.0.32

Fluent slider.

| Property | Type | Notes |
|---|---|---|
| `Value` | Number | Current slider value |
| `Min` | Number | Minimum value |
| `Max` | Number | Maximum value |
| `OnChange` | Behavior | Actions when value changes |
| `Layout` | Enum | `=Layout.Vertical` or `=Layout.Horizontal` |
| `BasePaletteColor` | Color | Theme base palette color override |
| `AccessibleLabel` | String | Screen reader label |

---

## ModernNumberInput@1.1.0

Fluent numeric input with validation and precision.

| Property | Type | Notes |
|---|---|---|
| `Default` | Number | Initial value |
| `Value` | Number (output) | Current value |
| `HintText` | String | Placeholder/hint text |
| `Min` | Number | Minimum allowed value |
| `Max` | Number | Maximum allowed value |
| `Step` | Number | Increment/decrement step |
| `Precision` | Enum | `=DecimalPrecision.Auto`, `'0'`, `'1'`, `'2'`, `'3'` |
| `ValidationState` | Enum | `ValidationState.None` or `ValidationState.Error` |
| `Appearance` | Enum | `=Appearance.Filled`, `.FilledDarker`, `.Outline` |
| `BasePaletteColor` | Color | Theme base palette color override |
| `Font` | Enum | Font family name |
| `Size` | Number | Font size in points |
| `Color` | Color | Text color |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `Italic` | Boolean | |
| `Underline` | Boolean | |
| `Strikethrough` | Boolean | |
| `RadiusTopLeft` | Number | Corner radius |
| `RadiusTopRight` | Number | Corner radius |
| `RadiusBottomLeft` | Number | Corner radius |
| `RadiusBottomRight` | Number | Corner radius |

---

## ModernText@1.0.0

Fluent static text.

| Property | Type | Notes |
|---|---|---|
| `Text` | String/Formula | Text content |
| `OnSelect` | Behavior | Actions on tap/click |
| `Wrap` | Boolean | Whether text wraps |
| `AutoHeight` | Boolean | Auto-grow height when text overflows |
| `Align` | Enum | `=Align.Left`, `.Center`, `.Right`, `.Justify` |
| `VerticalAlign` | Enum | `=VerticalAlign.Top`, `.Middle`, `.Bottom` |
| `Font` | Enum | Font family name |
| `Size` | Number | Font size in points |
| `Color` | Color | Text color |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `Italic` | Boolean | |
| `Underline` | Boolean | |
| `Strikethrough` | Boolean | |
| `RadiusTopLeft` | Number | Corner radius |
| `RadiusTopRight` | Number | Corner radius |
| `RadiusBottomLeft` | Number | Corner radius |
| `RadiusBottomRight` | Number | Corner radius |

---

## Header@0.0.44

Modern app header.

| Property | Type | Notes |
|---|---|---|
| `Title` | String | Header title |
| `IsTitleVisible` | Boolean | |
| `Logo` | Image/Formula | Header logo |
| `IsLogoVisible` | Boolean | |
| `IsProfilePictureVisible` | Boolean | |
| `OnSelectLogo` | Behavior | Actions when logo is selected |
| `LogoMaxHeight` | Number | Max height for logo |
| `LogoToolTip` | String | Tooltip for the logo |
| `BasePaletteColor` | Color | Theme base palette override |
| `Style` | Enum | Header color variant |
| `Font` | Enum | Font family name |
| `TitleFontSize` | Number | Font size for title |
| `FontColor` | Color | Title text color |

---

## ModernTabList@1.0.0

Tabbed navigation.

| Property | Type | Notes |
|---|---|---|
| `Items` | Table/Collection | Tabs source |
| `Default` | String/Formula | Tab selected on load |
| `Selected` | Record (output) | Currently selected tab |
| `OnChange` | Behavior | Fires when selected tab changes |
| `OnSelect` | Behavior | Fires when any tab is selected |
| `Align` | Enum | `=Align.Left`, `.Center`, `.Right`, `.Justify` |
| `Alignment` | Enum | `=TabListAlignment.Start`, `.Center`, `.End` |
| `Appearance` | Enum | `=TabListAppearance.Transparent`, `.Subtle`, `.Underline`, `.Filled` |
| `Font` | Enum | Font family name |
| `Size` | Number | Font size in points |
| `Color` | Color | Tab text color |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `Italic` | Boolean | |
| `Underline` | Boolean | |
| `Strikethrough` | Boolean | |
| `BorderColor` | Color | Tab border color |
| `BorderStyle` | Enum | `=BorderStyle.Solid`, `.Dashed`, `.Dotted`, `.None` |
| `BorderThickness` | Number | Pixels |
| `RadiusTopLeft` | Number | Corner radius |
| `RadiusTopRight` | Number | Corner radius |
| `RadiusBottomLeft` | Number | Corner radius |
| `RadiusBottomRight` | Number | Corner radius |

---

## Avatar@1.0.40

User/entity avatar.

| Property | Type | Notes |
|---|---|---|
| `Name` | String | Used for initials and accessibility |
| `Image` | Image/Formula | Avatar image |
| `Badge` | String | Optional status badge (if supported by runtime) |
| `Shape` | Enum | Circular or square |
| `Appearance` | Enum | `Colorful`, `Neutral`, `Brand` |
| `BasePaletteColor` | Color | Theme base palette override |
| `Font` | Enum | Font family name |
| `FontSize` | Number | Font size in points |
| `FontColor` | Color | Text color |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `FontItalic` | Boolean | |
| `FontUnderline` | Boolean | |
| `FontStrikethrough` | Boolean | |

---

## Badge@0.0.24

Decorative status badge.

| Property | Type | Notes |
|---|---|---|
| `Content` | String/Formula | Badge content |
| `Shape` | Enum | `Circular`, `Square`, `Rounded` |
| `Appearance` | Enum | Badge appearance variant |
| `ThemeColor` | Enum | `Brand`, `Danger`, `Important`, `Informative`, `Severe`, `Subtle`, `Success`, `Warning` |
| `BasePaletteColor` | Color | Theme base palette override |
| `Font` | Enum | Font family name |
| `FontSize` | Number | Font size in points |
| `FontColor` | Color | Text color |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `FontItalic` | Boolean | |
| `FontUnderline` | Boolean | |
| `FontStrikethrough` | Boolean | |

---

## Progress@1.1.34

Progress bar.

| Property | Type | Notes |
|---|---|---|
| `Value` | Number | Current progress value |
| `Max` | Number | Max value for determinate state |
| `Indeterminate` | Boolean | If true, determinate `Value` and `Max` are ignored |
| `ProgressColor` | Enum | `Brand`, `Error`, `Warning`, `Success` |
| `Thickness` | Enum | `Medium`, `Large` |
| `Shape` | Enum | Bar and track shape variant |
| `OnChange` | Behavior | Actions when value changes |
| `DisplayMode` | Enum | `=DisplayMode.Edit`, `View`, `Disabled` |
| `BasePaletteColor` | Color | Theme base palette override |

---

## Spinner@1.4.6

Loading spinner.

| Property | Type | Notes |
|---|---|---|
| `Label` | String | Label text |
| `LabelPosition` | Enum | Position of the label relative to spinner |
| `AccessibleLabel` | String | Screen reader label |
| `SpinnerSize` | Enum | Spinner size |
| `Appearance` | Enum | `Primary` or `Inverted` |
| `OnChange` | Behavior | Actions when spinner changes (if applicable) |
| `Font` | Enum | Font family name |
| `FontSize` | Number | Font size |
| `FontColor` | Color | Text color |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `FontItalic` | Boolean | |
| `FontUnderline` | Boolean | |
| `FontStrikethrough` | Boolean | |

---

## ModernLink@1.0.0

Clickable hyperlink.

| Property | Type | Notes |
|---|---|---|
| `Text` | String/Formula | Display text |
| `Url` | String | Destination URL (must be safe protocol) |
| `Type` | Enum | `=LinkAppearance.Default` or `=LinkAppearance.Subtle` |
| `Wrap` | Boolean | Whether text wraps |
| `AutoHeight` | Boolean | Auto-grow height |
| `Align` | Enum | `=Align.Left`, `.Center`, `.Right`, `.Justify` |
| `VerticalAlign` | Enum | `=VerticalAlign.Top`, `.Middle`, `.Bottom` |
| `BasePaletteColor` | Color | Theme base palette override |
| `Font` | Enum | Font family name |
| `Size` | Number | Font size in points |
| `Color` | Color | Link text color |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `Italic` | Boolean | |
| `Underline` | Boolean | |
| `Strikethrough` | Boolean | |

---

## ModernCard@1.0.0

Card container.

| Property | Type | Notes |
|---|---|---|
| `Direction` | Enum | Vertical or horizontal card layout |
| `Title` | String/Formula | Card title |
| `Subtitle` | String/Formula | Supplemental text |
| `Description` | String/Formula | Body description |
| `Image` | Image/Formula | Preview image |
| `ImageAltText` | String/Formula | Alt text for the preview image |
| `HeaderImage` | Image/Formula | Small avatar/icon in the header |
| `HeaderImageAltText` | String/Formula | Alt text for the header image |
| `ImagePosition` | Enum | Image before/after header |
| `OnSelect` | Behavior | Action when card is selected |
| `Radius` | Number | Card corner roundness |
| `DropShadow` | Enum | Shadow style |
| `Visible` | Boolean | Shows or hides the card |
| `DisplayMode` | Enum | `=DisplayMode.Edit`, `View`, `Disabled` |

---

## ModernInformationButton@1.0.0

Info icon with flyout content.

| Property | Type | Notes |
|---|---|---|
| `Content` | String/Formula | Flyout content |
| `IconColor` | Color | Color of the info icon |
| `OnSelect` | Behavior | Actions beyond opening the flyout |
| `DisplayMode` | Enum | `=DisplayMode.Edit`, `View`, `Disabled` |
| `BasePaletteColor` | Color | Theme base palette override |
| `Font` | Enum | Font family name |
| `Size` | Number | Flyout font size |
| `Color` | Color | Flyout text color |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `Italic` | Boolean | |
| `Underline` | Boolean | |
| `Strikethrough` | Boolean | |

---

## Classic/TextInput@2.3.2

Full-featured text input. Supports `Default` and `Mode`.

| Property | Type | Notes |
|---|---|---|
| `Default` | String/Formula | Pre-filled value — **this control DOES support Default** |
| `HintText` | String | Placeholder/hint text |
| `Mode` | Enum | `=TextMode.SingleLine`, `.MultiLine`, `.Password` |
| `Format` | Enum | `=TextInputFormat.Text`, `.Number` |
| `MaxLength` | Number | Max character count (`=0` = unlimited) |
| `Clear` | Boolean | `=true` shows an X button to clear input |
| `DelayOutput` | Boolean | `=true` delays `OnChange` until user stops typing |
| `VirtualKeyboardMode` | Enum | `=VirtualKeyboardMode.Auto`, `.Numeric`, `.Text` |
| `RadiusTopLeft` | Number | Supported |
| `RadiusTopRight` | Number | Supported |
| `RadiusBottomLeft` | Number | Supported |
| `RadiusBottomRight` | Number | Supported |

---

## TextInput@0.0.54 (Previous-gen Modern)

**No `Default` property** — use `Classic/TextInput@2.3.2` when pre-fill is needed.

| Property | Type | Notes |
|---|---|---|
| `Appearance` | Enum | `='TextInputCanvas.Appearance'.FilledLighter`, `.FilledDarker`, `.Outline` |
| `Mode` | Enum | `='TextInputCanvas.Mode'.SingleLine`, `.Multiline`, `.Password` |
| `Placeholder` | String | Placeholder text |
| `Height` | Number | Usually `=40` (single line) or `=80`+ (multiline) |

---

## Classic/DropDown@2.3.1

Simple list selection (no search).

| Property | Type | Notes |
|---|---|---|
| `Default` | String/Formula | Pre-selected value |
| `Items` | Table | Data source or `["Option1","Option2"]` |
| `Value` | String | Column name to display, e.g., `="Value"` |
| `AllowEmptySelection` | Boolean | `=true` allows clearing selection |
| `SelectionColor` | Color | Text color of selected item |
| `SelectionFill` | Color | Background of selected item |
| `ChevronBackground` | Color | Background behind the chevron icon |
| `ChevronFill` | Color | Color of the chevron icon |

---

## Classic/ComboBox@2.4.0

Searchable dropdown with optional multi-select. Supports `DefaultSelectedItems`.

| Property | Type | Notes |
|---|---|---|
| `DefaultSelectedItems` | Record/Table | Pre-selected item(s) — **use this, not `Default` (deprecated)** |
| `Items` | Table | Data source |
| `SearchFields` | List | Column name(s) to search on, e.g., `["Value"]` — **Classic/ComboBox DOES support SearchFields** |
| `IsSearchable` | Boolean | `=true` enables search (default true) |
| `SelectMultiple` | Boolean | `=true` enables multi-select |
| `DisplayFields` | List | Columns to show in dropdown, e.g., `["MyFirstColumn", "MySecondColumn"]` |
| `InputTextPlaceholder` | String | Placeholder when nothing selected |
| `SelectedItems` | Table (read-only) | List of selected items from user interaction |
| `Selected` | Record (read-only) | Last selected item |

**Visual / chevron properties:**

| Property | Type | Notes |
|---|---|---|
| `ChevronFill` | Color | Color of the chevron arrow icon |
| `ChevronHoverFill` | Color | Chevron icon color on hover |
| `ChevronBackground` | Color | Background behind the chevron — use `=RGBA(0,0,0,0)` for modern look |
| `ChevronHoverBackground` | Color | Chevron background on hover |
| `SelectionColor` | Color | Text color of a selected item in the dropdown list |
| `SelectionFill` | Color | Background of a selected item in the dropdown list |

> **`FocusedBorderThickness` on ComboBox** behaves differently than on other controls — it produces a dotted border when the dropdown is opened. Set `FocusedBorderThickness: =0` to remove it for a cleaner look.

> `ComboBox@0.0.51` (previous-gen) does NOT support `SearchFields`. Only `Classic/ComboBox@2.4.0` supports it.

> **`Default` is deprecated** on ComboBox — use `DefaultSelectedItems` instead. `SearchField` (singular) was the old name; the correct property name is `SearchFields` (plural).

> **No border radius support** — `RadiusTopLeft/Right/BottomLeft/Right` cause PA2108. Apply the **Radius Wrapper Pattern**: wrap in a GroupContainer with `RadiusXxx`, `BorderStyle: Solid`, `BorderThickness`, and `BorderColor`. Set the control's own `BorderStyle: =BorderStyle.None` and `BorderThickness: =0`. The container's rounded corners visually clip the child. See "Radius Wrapper Pattern" in `pa-yaml-rules.md`.

---

## ComboBox@0.0.51 (Previous-gen Modern)

**No `SearchField` property** — search is built-in automatically.

| Property | Type | Notes |
|---|---|---|
| `Appearance` | Enum | `='ComboboxCanvas.Appearance'.FilledLighter`, `.FilledDarker`, `.Outline` |
| `Items` | Table | Data source or inline table |
| `Height` | Number | Usually `=40` |

---

## Classic/CheckBox@2.1.0

Real checkbox control. **Use this instead of Rectangle for checkboxes.**

| Property | Type | Notes |
|---|---|---|
| `Default` | Boolean | `=true` (checked) or `=false` (unchecked) |
| `Value` | Boolean (read-only) | Current checked state — `=checkName.Value` |
| `Text` | String | Label text beside the checkbox |
| `CheckboxSize` | Number | Size of the checkbox in pixels |
| `CheckmarkFill` | Color | Color of the checkmark |
| `CheckboxBackgroundFill` | Color | Background of the checkbox box. Use `=If(checkboxName.Value, RGBA(42,43,71,1), RGBA(255,255,255,1))` to change color on checked state. |
| `CheckboxBorderColor` | Color | Border of the box |

> **`OnCheckFill` does NOT exist** — causes PA2108. To style the checked state, use `CheckboxBackgroundFill` with an `If(checkboxName.Value, ...)` formula.

---

## Classic/Radio@2.3.0

Radio button group. Supports `Default`.

| Property | Type | Notes |
|---|---|---|
| `Default` | String/Formula | Pre-selected value, e.g., `="Option1"` — **this control DOES support Default** |
| `Items` | Table | Options — usually `=["Option1","Option2"]` or inline Table |
| `Layout` | Enum | `=Layout.Vertical`, `=Layout.Horizontal` |
| `Value` | String (read-only) | Currently selected value |
| `RadioBackgroundFill` | Color | Background of each radio button circle |
| `RadioBorderColor` | Color | Border of each circle |
| `RadioSelectionFill` | Color | Fill of selected radio circle |
| `RadioSize` | Number | Diameter of the radio circle |

> **`HoverBorderColor` does NOT exist** on Classic/Radio — causes PA2108. Hover styling on Radio is limited to `HoverFill` and `HoverColor` only.

---

## Radio@0.0.25 (Previous-gen Modern)

**No `Default` property** — use `Classic/Radio@2.3.0` when pre-selection is needed.

| Property | Type | Notes |
|---|---|---|
| `Layout` | Enum | `='RadioGroupCanvas.Layout'.Horizontal`, `.Vertical` |
| `Items` | Table | Options — `Table({Value:"Yes"},{Value:"No"})` |
| `Height` | Number | Usually `=40` |

---

## Classic/Toggle@2.1.0

On/off toggle switch. Supports `Default`.

| Property | Type | Notes |
|---|---|---|
| `Default` | Boolean | `=true` (on) or `=false` (off) |
| `Value` | Boolean (read-only) | Current toggle state |
| `TrueText` | String | Label when on, e.g., `="Yes"` |
| `FalseText` | String | Label when off, e.g., `="No"` |
| `ShowLabel` | Boolean | `=true` shows `TrueText`/`FalseText` label |
| `TextPosition` | Enum | `=TextPosition.Left`, `=TextPosition.Right` |
| `TrueFill` | Color | Rail background when on |
| `FalseFill` | Color | Rail background when off |
| `TrueHoverFill` | Color | Rail on hover when on |
| `FalseHoverFill` | Color | Rail on hover when off |
| `HandleFill` | Color | Color of the sliding handle |
| `RailFill` | Color | Rail fill (base state) |
| `RailHoverFill` | Color | Rail fill on hover |

---

## Toggle@1.1.5 (Modern Toggle)

| Property | Type | Notes |
|---|---|---|
| `Checked` | Boolean | Initial checked state |
| `Label` | String | Text label |
| `LabelPosition` | Enum | `='ToggleCanvas.LabelPosition'.Before`, `.After` |
| `OnCheck` | Behavior | Actions when toggled on |
| `OnSelect` | Behavior | Actions when selected |
| `OnUncheck` | Behavior | Actions when toggled off |
| `BasePaletteColor` | Color | Theme base palette color override |
| `Font` | Enum | Font family name |
| `FontSize` | Number | Font size in points |
| `FontColor` | Color | Label text color |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `FontItalic` | Boolean | |
| `FontUnderline` | Boolean | |
| `FontStrikethrough` | Boolean | |

---

## Classic/Slider@2.1.0

Numeric range slider. Supports `Default`.

| Property | Type | Notes |
|---|---|---|
| `Default` | Number | Initial value |
| `Min` | Number | Minimum value |
| `Max` | Number | Maximum value |
| `Value` | Number (read-only) | Current value |
| `Layout` | Enum | `=Layout.Horizontal`, `=Layout.Vertical` |
| `ShowValue` | Boolean | `=true` shows current value label |
| `ReadOnly` | Boolean | `=true` prevents interaction |
| `HandleFill` | Color | Handle circle color |
| `HandleHoverFill` | Color | Handle on hover |
| `HandleActiveFill` | Color | Handle when dragging |
| `HandleSize` | Number | Diameter of handle circle |
| `RailFill` | Color | Unfilled rail color |
| `RailHoverFill` | Color | Rail on hover |
| `ValueFill` | Color | Filled portion of rail |
| `ValueHoverFill` | Color | Filled rail on hover |

---

## Classic/DatePicker@2.6.0

Date picker with calendar popup. Supports `DefaultDate`.

| Property | Type | Notes |
|---|---|---|
| `DefaultDate` | Date/Formula | Pre-selected date, e.g., `=Today()` |
| `SelectedDate` | Date (read-only) | Currently selected date |
| `Format` | Enum/String | `="ShortDate"` (default), `="LongDate"`, or custom string like `="yyyy/mm/dd"` |
| `Language` | String | Locale string, e.g., `="en-US"` |
| `StartOfWeek` | Enum | `=StartOfWeek.Sunday`, `.Monday` |
| `IsEditable` | Boolean | `=false` disables manual text entry — user must use calendar popup |
| `DateTimeZone` | Enum | `="UTC"` or `="Local"` — whether to display date in UTC or local time |
| `StartYear` | Number | Earliest year selectable in the picker |
| `EndYear` | Number | Latest year selectable in the picker |
| `InputTextPlaceholder` | String | Instructional text when no date is entered |
| `IconFill` | Color | Foreground color of the calendar icon |
| `IconBackground` | Color | Background color of the calendar icon |

> **`IsReadOnly` does NOT exist on Classic/DatePicker** — the correct property is `IsEditable` (set to `=false` to make read-only). `IsReadOnly` causes PA2108.

---

## DatePicker@0.0.46 (Previous-gen Modern)

**No border radius support.**

| Property | Type | Notes |
|---|---|---|
| `Appearance` | Enum | `='DatePickerCanvas.Appearance'.FilledLighter`, `.FilledDarker`, `.Outline` |
| `Fill` | Color | `=Color.Transparent` to let container background show |
| `Height` | Number | Usually `=40` |

---

## NumberInput@2.9.12 (Previous-gen Modern)

**No border radius support.**

| Property | Type | Notes |
|---|---|---|
| `Appearance` | Enum | `='NumberInput.Appearance'.FilledLighter`, `.FilledDarker`, `.Outline` |
| `Height` | Number | Usually `=40` |

---

## Classic/Icon@2.5.0

> **⛔ Do not use in generated YAML.** `Classic/Icon` produces dated, flat icons that conflict with the modern aesthetic this skill targets. Always use `Image@2.2.3` with an inline Bootstrap Icons SVG instead (see SVG Inline Icon Pattern in `pa-yaml-rules.md`). This control is documented here for reference only — it must never appear in skill output.

500+ built-in SVG icons. Different from `Image` — icons are vector, not bitmap.

| Property | Type | Notes |
|---|---|---|
| `Icon` | Enum | `=Icon.ChevronRight`, `=Icon.Search`, `=Icon.Add`, `=Icon.Cancel`, etc. |
| `Rotation` | Number | Degrees (0–360) |
| `Color` | Color | Icon color — set `Color` not `Fill` for the icon tint |

Common icon values: `Icon.Add`, `Icon.Cancel`, `Icon.Check`, `Icon.ChevronLeft`, `Icon.ChevronRight`, `Icon.ChevronDown`, `Icon.ChevronUp`, `Icon.Search`, `Icon.Edit`, `Icon.Trash`, `Icon.Filter`, `Icon.Sort`, `Icon.Settings`, `Icon.People`, `Icon.Person`, `Icon.Upload`, `Icon.Download`, `Icon.Save`, `Icon.Back`, `Icon.Forward`, `Icon.Refresh`, `Icon.Home`, `Icon.Star`, `Icon.StarFilled`, `Icon.Bell`, `Icon.Lock`, `Icon.Unlock`, `Icon.Eye`, `Icon.EyeOff`, `Icon.Info`, `Icon.Warning`, `Icon.Error`

---

## Screen

| Property | Type | Notes |
|---|---|---|
| `Fill` | Color | Screen background color |
| `OnVisible` | Behavior | Runs when screen becomes visible — use for `ClearCollect` |
| `OnHidden` | Behavior | Runs when screen is navigated away from |
| `Size` | Number (read-only) | Responsive tier: 1=Small, 2=Medium, 3=Large, 4=ExtraLarge |

`Size` breakpoints (default): Small <600px, Medium 600–900px, Large 900–1200px, ExtraLarge >1200px.

Reference as `ScreenName.Size` in formulas.

Screen root YAML format (for paste target b — new screen):
```yaml
Screens:
  ScreenName:
    Properties:
      Fill: =RGBA(248,249,250,1)
      OnVisible: |-
        =ClearCollect(colSampleData, ...)
    Children:
      - mainContainer:
          Control: GroupContainer@1.5.0
          Variant: AutoLayout
          Properties:
            Height: =Parent.Height
            Width: =Parent.Width
```

> **WARNING: `Control: Screen` is NOT valid — causes PA2101. Always use the `Screens:` top-level key format above.**

---

## SECTION C — ⚠️ PA2108 Trap Table

Using any property in this table causes a PA2108 "Unknown property" error in PA Studio.

| Control | Invalid Property | PA Error | Correct Approach |
|---|---|---|---|
| `TextInput@0.0.54` | `Default` | PA2108 | Omit it, or use `Classic/TextInput@2.3.2` which supports `Default` |
| `ComboBox@0.0.51` | `SearchField` | PA2108 | Omit it — search is built-in, no config needed |
| `Classic/ComboBox@2.4.0` | `SearchField` | PA2108 | Old singular name — correct property is `SearchFields` (plural) |
| `Classic/ComboBox@2.4.0` | `Default` | PA2108 | Deprecated — use `DefaultSelectedItems` instead |
| `Classic/ComboBox@2.4.0` | `NoSelectionText` | PA2108 | Does not exist — use `InputTextPlaceholder` for placeholder text |
| `Radio@0.0.25` | `Default` | PA2108 | Omit it, or use `Classic/Radio@2.3.0` which supports `Default` |
| `DatePicker@0.0.46` | `Default` | PA2108 | Omit it, or use `Classic/DatePicker@2.6.0` which uses `DefaultDate` |
| `Classic/DatePicker@2.6.0` | `IsReadOnly` | PA2108 | Does not exist — use `IsEditable: =false` to prevent manual text entry |
| `Rectangle@2.3.0` | `RadiusTopLeft` | PA2108 | Rectangle does NOT support radius — use GroupContainer with Fill |
| `Rectangle@2.3.0` | `RadiusTopRight` | PA2108 | Same — use GroupContainer |
| `Rectangle@2.3.0` | `RadiusBottomLeft` | PA2108 | Same — use GroupContainer |
| `Rectangle@2.3.0` | `RadiusBottomRight` | PA2108 | Same — use GroupContainer |
| `Label@2.5.1` | `RadiusTopLeft/Right/BottomLeft/Right` | PA2108 | Label does NOT support radius — wrap in GroupContainer |
| `Classic/CheckBox@2.1.0` | `OnCheckFill` | PA2108 | Does not exist. Use `CheckboxBackgroundFill: =If(checkName.Value, checkedColor, uncheckedColor)` instead |
| `Classic/Button@2.2.0` | `Bold` | PA2108 | Does not exist as a standalone property. Use `FontWeight: =FontWeight.Bold` instead |
| `Classic/ComboBox@2.4.0` | `RadiusTopLeft`, `RadiusTopRight`, `RadiusBottomLeft`, `RadiusBottomRight` | PA2108 | Classic/ComboBox does NOT support border radius. Use the Radius Wrapper Pattern (see `pa-yaml-rules.md`): wrap in a GroupContainer with `RadiusXxx`, border, and fill; set the control's `BorderStyle: None` and `BorderThickness: 0`. |
| `Classic/DatePicker@2.6.0` | `RadiusTopLeft`, `RadiusTopRight`, `RadiusBottomLeft`, `RadiusBottomRight` | PA2108 | Classic/DatePicker does NOT support border radius. Use the Radius Wrapper Pattern. |
| `Classic/Radio@2.3.0` | `HoverBorderColor` | PA2108 | Classic/Radio does NOT support `HoverBorderColor`. Remove it — radio hover is styled via `HoverFill` and `HoverColor` only. |
| `Classic/Radio@2.3.0` | `PressedBorderColor` | PA2108 | Classic/Radio does NOT support `PressedBorderColor`. Remove it — radio pressed state is styled via `PressedFill` and `PressedColor` only. |
| `ModernRadio@1.0.0` | `HoverBorderColor` | PA2108 | ModernRadio does NOT support `HoverBorderColor`. Remove it — radio hover is styled via `HoverFill` and `HoverColor` only. |
| `ModernRadio@1.0.0` | `PressedBorderColor` | PA2108 | ModernRadio does NOT support `PressedBorderColor`. Remove it — radio pressed state is styled via `PressedFill` and `PressedColor` only. |
| `Button@0.0.45` | `Fill` | PA2108 | Modern Button does NOT support `Fill`. Use `Appearance` (`Primary`, `Secondary`, `Outline`) instead — Fluent handles all colors internally. |
| `Button@0.0.45` | `Color` | PA2108 | Modern Button does NOT support `Color`. Use `Appearance` instead — Fluent handles text color internally based on appearance. |
| `Button@0.0.45` | `BorderColor` | PA2108 | Modern Button does NOT support `BorderColor`. Use `Appearance` instead — Fluent handles borders internally. |
| `Classic/CheckBox@2.1.0` | `RadiusTopLeft`, `RadiusTopRight`, `RadiusBottomLeft`, `RadiusBottomRight` | PA2108 | Classic/CheckBox does NOT support border radius. Use the Radius Wrapper Pattern if rounded corners are needed. |
| `Gallery@2.15.0` | `Self.TemplateSize` in formulas | Wrong value | `TemplateSize` is write-only — cannot be read back. Use `Self.TemplateHeight` (vertical) or `Self.TemplateWidth` (horizontal) in any formula referencing gallery item size. |
| `Gallery@2.15.0` | `DropShadow` | PA2108 | Gallery does NOT support `DropShadow`. Remove it entirely — galleries have no shadow property. |

---

## Enum Quick Reference

| Enum | Values |
|---|---|
| `LayoutDirection` | `Horizontal`, `Vertical` |
| `LayoutAlignItems` | `Start`, `Center`, `End`, `Stretch` |
| `LayoutJustifyContent` | `Start`, `End`, `Center`, `SpaceBetween`, `SpaceAround`, `SpaceEvenly` |
| `AlignInContainer` | `Start`, `Center`, `End`, `Stretch`, `SetByContainer` |
| `LayoutOverflow` | `Scroll`, `Hidden` |
| `DropShadow` | `None`, `Light`, `Regular`, `Bold`, `ExtraBold` |
| `BorderStyle` | `None`, `Solid`, `Dashed`, `Dotted` |
| `Align` (text) | `Left`, `Center`, `Right`, `Justify` |
| `VerticalAlign` | `Top`, `Middle`, `Bottom` |
| `FontWeight` | `Normal`, `Semibold`, `Bold`, `Lighter` |
| `ImagePosition` | `Fill`, `Fit`, `Stretch`, `Center`, `Tile` |
| `ScreenSize` | `Small` (1), `Medium` (2), `Large` (3), `ExtraLarge` (4) |
| `Transition` | `None`, `Pop`, `Push`, `Fade` |
| `Font` | `Lato`, `'Lato Black'`, `'Lato Light'`, `'Open Sans'`, `Arial`, `Calibri` |
| `DisplayMode` | `Edit`, `View`, `Disabled` |
| `TextMode` | `SingleLine`, `MultiLine`, `Password` |
| `TextInputFormat` | `Text`, `Number` |
| `Layout` (Classic Radio/Slider) | `Horizontal`, `Vertical` |
