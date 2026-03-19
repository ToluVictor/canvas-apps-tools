# Canvas App Controls Index

Master lookup table for all Power Apps Canvas App controls. Used in Phase 4 Step 1 to resolve the correct `Control: Type@version` string for any UI element.

**Version strings sourced from real PA YAML exports** — these are the confirmed version strings as of early 2026. If PA reports PA2105 for any control, apply the VERSION SELF-HEALING RULE in `pa-yaml-rules.md` to update both files.

---

## How to Use This Index

1. Identify the UI element to render (e.g., "checkbox", "text input", "dropdown")
2. Find the row in the relevant table below
3. Check the **Preferred** column — use that control type in the YAML
4. Look up the control's unique properties in `controls-reference.md`

**To change the preferred variant** for a control: update `**Classic**` → `**Modern**` (or vice versa) in the Preferred column. The skill will use the new preference on all future generations.

---

## Controls with Classic and Modern Variants

Both variants are real, separate controls — do NOT rename or replace one with the other. Use the **Preferred** column to select which one to generate.

| UI Element | Classic YAML Type | Classic Version | Modern YAML Type | Modern Version | Preferred |
|---|---|---|---|---|---|
| Text input / text field | `Classic/TextInput` | `@2.3.2` | `ModernTextInput` | `@1.0.0` | **Classic** |
| Dropdown (simple list) | `Classic/DropDown` | `@2.3.1` | _(none)_ | — | **Classic** |
| Combobox (searchable) | `Classic/ComboBox` | `@2.4.0` | `ModernCombobox` | `@1.0.0` | **Classic** |

> **Prefer `Classic/ComboBox` over `Classic/DropDown`.** ComboBox does everything DropDown does (single-select list) plus adds search and multi-select. Default to ComboBox for all dropdown/select UI elements unless the user explicitly requests a simple DropDown.
| Checkbox | `Classic/CheckBox` | `@2.1.0` | `CheckBox` | `@0.0.30` | **Classic** |
| Radio button group | `Classic/Radio` | `@2.3.0` | `ModernRadio` | `@1.0.0` | **Classic** |
| Toggle / switch | `Classic/Toggle` | `@2.1.0` | `Toggle` | `@1.1.5` | **Classic** |
| Slider (range input) | `Classic/Slider` | `@2.1.0` | `Slider` | `@1.0.32` | **Classic** |
| Date picker | `Classic/DatePicker` | `@2.6.0` | `ModernDatePicker` | `@1.0.0` | **Classic** |
| Button | `Classic/Button` | `@2.2.0` | `Button` | `@0.0.45` | **Classic** |
| Text label | `Label` | `@2.5.1` | `ModernText` | `@1.0.0` | **Label** |
| Number input | `NumberInput` | `@2.9.12` | `ModernNumberInput` | `@1.1.0` | **NumberInput** |

> **Note:** `Classic/Button@2.2.0` is also used as a transparent click overlay on top of custom-styled containers. `Button@0.0.45` (Modern) is the Fluent Design button with `Appearance`, `Icon`, and `Layout` properties.

---

## Common Controls (no Classic/Modern split)

Single control type — use as-is.

| UI Element | YAML Type | Version | Notes |
|---|---|---|---|
| Layout container | `GroupContainer` | `@1.5.0` | Variants: `AutoLayout`, `ManualLayout` |
| Scrollable list / data grid | `Gallery` | `@2.15.0` | Variants: `Vertical`, `Horizontal` |
| Image | `Image` | `@2.2.3` | Supports `RadiusTopLeft/Right/BottomLeft/Right` |
| Icon (500+ built-in SVG) | `Classic/Icon` | `@2.5.0` | `Icon` property uses `Icon.xxx` enum |
| Rectangle / divider | `Rectangle` | `@2.3.0` | **No border radius support** — use GroupContainer for rounded shapes |
| Circle / oval shape | `Circle` | `@2.3.0` | |
| Edit form | `Form` | `@2.4.4` | Connected to a data source |
| Display form (read-only) | `FormViewer` | `@2.3.4` | Read-only form display |
| Star rating | `Rating` | `@2.1.0` | |
| Multi-select list box | `ListBox` | `@2.2.0` | |
| Rich text editor | `RichTextEditor` | `@2.7.0` | |
| Timer | `Timer` | `@2.1.0` | |
| HTML text renderer | `HtmlViewer` | `@2.1.0` | |
| Image / media upload | `AddMedia` | `@2.2.1` | |
| Pen / signature input | `PenInput` | `@2.3.0` | |

---

## Modern-Only Controls (no Classic equivalent)

These controls exist only in the Modern Fluent Design family. Use when the UI specifically calls for them.

| UI Element | YAML Type | Version | Learn URL (relative) |
|---|---|---|---|
| App header bar | `Header` | `@0.0.44` | modern-controls-header |
| Tab list / navigation tabs | `ModernTabList` | `@1.0.0` | modern-control-tabs-or-tabs-list |
| Info button with tooltip | `ModernInformationButton` | `@1.0.0` | modern-control-info-button |
| User / entity avatar | `Avatar` | `@1.0.40` | modern-control-avatar |
| Status badge | `Badge` | `@0.0.24` | modern-controls-badge |
| Progress bar | `Progress` | `@1.1.34` | modern-control-progress-bar |
| Loading spinner | `Spinner` | `@1.4.6` | modern-control-spinner |
| Hyperlink | `ModernLink` | `@1.0.0` | modern-control-link |
| Card container | `ModernCard` | `@1.0.0` | modern-control-card |

---

## Modern Control Capabilities

The newer Modern variants (`ModernTextInput@1.0.0`, `ModernCombobox@1.0.0`, `ModernRadio@1.0.0`, etc.) are **fully featured** and support properties that the previous-generation controls lack. When the user selects Modern controls, prefer these over Classic — there are NO mandatory Classic fallbacks for `Default`, `IsSearchable`, or `SearchFields`.

| Modern YAML Type | Version | Key Capabilities |
|---|---|---|
| `ModernTextInput` | `@1.0.0` | Supports `Default`, `Appearance`, `BasePaletteColor`, `BorderStyle`, `BorderThickness`, `RadiusXxx`, `PaddingXxx` |
| `ModernCombobox` | `@1.0.0` | Supports `InputTextPlaceholder`, `IsSearchable`, `SelectMultiple`, `Items`, `ItemDisplayText` |
| `ModernRadio` | `@1.0.0` | Supports `Default` (record syntax: `={Value: "Option"}`), `Items` |
| `CheckBox` | `@0.0.30` | Modern Fluent checkbox — fixed Fluent token sizing, no `CheckboxSize` property |
| `Button` | `@0.0.45` | Supports `Appearance`, `Icon`, `Layout` — does NOT support `Fill`, `Color`, `BorderColor` (Fluent handles all colors internally) |

> **When user selects Modern:** Use `ModernTextInput`, `ModernCombobox`, `ModernRadio`, `CheckBox@0.0.30`, `Button@0.0.45` for all applicable controls. The ONLY mandatory Classic fallback is `Classic/Button@2.2.0` for transparent overlay buttons (no Modern equivalent for the overlay pattern).

---

## Previous-Generation Modern Controls

These are older modern controls. They are **separate controls** from both the Classic AND the newer Modern variants above — do NOT confuse them. They remain valid but have fewer properties.

| YAML Type | Version | Key Limitation |
|---|---|---|
| `TextInput` | `@0.0.54` | **No `Default` property** — use `ModernTextInput@1.0.0` or `Classic/TextInput@2.3.2` instead |
| `ComboBox` | `@0.0.51` | **No `SearchField` property** — use `ModernCombobox@1.0.0` or `Classic/ComboBox@2.4.0` instead |
| `Radio` | `@0.0.25` | **No `Default` property** — use `ModernRadio@1.0.0` or `Classic/Radio@2.3.0` instead |
| `DatePicker` | `@0.0.46` | No border radius support |
| `NumberInput` | `@2.9.12` | Older generation — `ModernNumberInput@1.1.0` is the newer variant |

---

## Microsoft Learn URLs

Base URL: `https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/controls/`

| Control | Page slug |
|---|---|
| `Classic/Button` | `control-button` |
| `Classic/TextInput` | `control-text-input` |
| `Classic/DropDown` | `control-drop-down` |
| `Classic/ComboBox` | `control-combo-box` |
| `Classic/CheckBox` | `control-check-box` |
| `Classic/Radio` | `control-radio` |
| `Classic/Toggle` | `control-toggle` |
| `Classic/Slider` | `control-slider` |
| `Classic/DatePicker` | `control-date-picker` |
| `Classic/Icon` | `control-shapes-icons` |
| `Gallery` | `control-gallery` |
| `Label` | `control-text-box` |
| `Image` | `control-image` |
| `Rectangle` | `control-shapes-icons` |
| `Button` (Modern) | `modern-control-button` |
| `ModernTextInput` | `modern-control-text-input` |
| `ModernCombobox` | `modern-control-combobox` |
| `CheckBox` (Modern) | `modern-control-checkbox` |
| `ModernRadio` | `modern-controls-radio-group` |
| `Toggle` (Modern) | `modern-control-toggle` |
| `Slider` (Modern) | `modern-control-slider` |
| `ModernDatePicker` | `modern-controls-date-picker` |
| `Header` | `modern-controls-header` |
| `ModernTabList` | `modern-control-tabs-or-tabs-list` |
| `Avatar` | `modern-control-avatar` |
| `Badge` | `modern-controls-badge` |
| `Progress` | `modern-control-progress-bar` |
| `Spinner` | `modern-control-spinner` |
