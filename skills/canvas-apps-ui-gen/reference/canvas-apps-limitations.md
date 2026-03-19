# Canvas Apps UI Limitations Reference

Compact reference of web/design elements that Canvas Apps cannot render natively,
with the closest Canvas Apps alternative for each.

The skill reads this file during Phase 3 (Analyze) in **all modes** and uses the
`Handle:` tag on each entry to decide how to proceed:

- **auto** — Apply the Canvas Apps alternative silently. Track in a "Substitutions applied" list shown at the end of output.
- **ask** — Surface the issue to the user with numbered options. Wait for their choice before proceeding.
- **skip** — Inform the user the element will be omitted. No user decision needed.

**Global rules (apply to all entries):**
- No PCF. This skill cannot generate Power Apps Component Framework components.
- No PowerBI Tile embed. Requires extra environment setup outside this skill's scope.
- No custom-built controls. Use the nearest native Canvas Apps control as-is.

Format per entry:
> **[Design element]**
> Handle: auto | ask | skip
> Limitation: [why Canvas Apps can't do it natively]
> Alternative: [Canvas Apps-native solution — multiple options listed for `ask` entries]

---

## CHARTS & DATA VISUALIZATION

**Donut / ring chart**
Handle: ask
Limitation: Canvas Apps has no native donut or ring chart control.
Alternative A: Two overlapping Ellipse controls with a formula-driven arc fill (static only).
Alternative B: Inline SVG arc path via Image@2.2.3 (more precise, still static).
Alternative C: Omit.

**Multi-axis / dual-axis chart**
Handle: ask
Limitation: Built-in Chart controls support a single Y-axis only.
Alternative A: Simplify to a single-axis line or column Chart control.
Alternative B: Stack two separate Chart controls sharing the same X labels.
Alternative C: Omit.

**Scatter / bubble chart**
Handle: ask
Limitation: No native scatter or bubble chart control.
Alternative A: Gallery with positioned dot Image controls in a ManualLayout container (complex but possible).
Alternative B: Simplify to a bar or column Chart control.
Alternative C: Omit.

**Animated / live-updating chart**
Handle: ask
Limitation: Chart controls are static renders — no built-in animation or real-time streaming.
Alternative A: Timer + UpdateContext to trigger a data refresh at set intervals; the Chart re-renders on each data change.
Alternative B: Static chart only (no refresh).
Alternative C: Omit.

**Gauge / speedometer chart**
Handle: ask
Limitation: No native gauge control.
Alternative A: Two Arc SVG shapes via Image@2.2.3 (inline SVG, static representation).
Alternative B: Simplified horizontal bar indicator using Rectangle controls.
Alternative C: Omit.

**Treemap / sunburst / funnel chart**
Handle: ask
Limitation: Not available as native chart types.
Alternative A: Simplified bar or column Chart control.
Alternative B: Gallery-based custom visualization using Rectangle controls sized proportionally.
Alternative C: Omit.

**Sparklines (inline mini-charts)**
Handle: ask
Limitation: No sparkline control; Chart controls are standalone full-size elements.
Alternative A: Stacked Rectangle controls inside a Gallery template, sized proportionally to values.
Alternative B: Text-based trend indicator (e.g., "+12%" or an up/down SVG arrow).
Alternative C: Omit.

---

## TYPOGRAPHY & FONTS

**Custom web fonts (Google Fonts, Adobe Fonts, @font-face)**
Handle: auto
Limitation: Canvas Apps cannot load fonts from external URLs.
Alternative: Use the closest available built-in font. Options include: Lato, Lato Black, Calibri, Arial, Segoe UI, Open Sans, Raleway, Patrick Hand, Dancing Script, Georgia, and others. Never reference a URL in a font property.

**Variable font weights (e.g., font-weight: 300, 400, 600, 700)**
Handle: auto
Limitation: Only discrete FontWeight enum values are available: Thin, Lighter, Light, Normal, Semibold, Bold, Bolder.
Alternative: Use the nearest available FontWeight enum value.

**Text gradients / outlined text stroke**
Handle: auto
Limitation: No gradient fill or stroke on text.
Alternative: Use the dominant solid color from the gradient's visible tone.

**Mixed inline styles within one label (e.g., bold a single word)**
Handle: auto
Limitation: Classic/Label applies one style to all its text. There is no inline span-level formatting.
Alternative: Use the HtmlText@0.0.45 control — it renders HTML markup and supports inline bold, italic, and color spans within a single control. NOTE: HtmlText is display-only — never use it for interactive elements (links, buttons, clicks).

**Hyperlink / clickable link**
Handle: auto
Limitation: HtmlText renders <a> tags visually but interactions coded on HtmlText can cause issues — it is a display-only control.
Alternative: Use a Classic/Button@2.2.0 (or Modern/Button) with the link text as its Text property and OnSelect: =Launch("<url>"). Style it as a text-only button (no fill/border) to match a visual link appearance.

**Text shadow**
Handle: ask
Limitation: No native text shadow property on any control.
Alternative A: Simulate with a duplicate Label placed slightly offset behind the visible label in a ManualLayout container, using a slightly different color.
Alternative B: Omit the shadow and use the plain label.

---

## LAYOUT & SCROLLING

**Flex-wrap / wrapping tile grid**
Handle: ask
Limitation: AutoLayout containers do not support flex-wrap — children cannot reflow to a new row when the container fills.
Alternative A: Nested horizontal AutoLayout containers (one per row) — rows are fixed.
Alternative B: Gallery with the WrapCount property for card or tile grids — layout is dynamic.

**Horizontal scroll on a GroupContainer**
Handle: auto
Limitation: LayoutOverflowX (horizontal scroll) is not supported on GroupContainers.
Alternative: Use a horizontal Gallery, which scrolls horizontally natively.

**CSS Grid / masonry layout**
Handle: ask
Limitation: No native grid or masonry layout engine.
Alternative A: Nested AutoLayout containers to simulate a fixed-column grid.
Alternative B: Gallery with WrapCount for a simpler repeating tile layout.

**Sticky / fixed header while body scrolls**
Handle: auto
Limitation: No CSS position:sticky equivalent.
Alternative: Place the header outside the scrollable container. The scrollable LayoutOverflowY container sits below the header in a parent Vertical AutoLayout — this is the standard pattern used throughout this skill's templates.

**Z-index overlapping within AutoLayout**
Handle: auto
Limitation: AutoLayout containers cannot render overlapping children.
Alternative: Switch the specific container to ManualLayout and use X/Y positioning. Or use a transparent Classic/Button as a top-layer overlay inside ManualLayout.

**Absolute / fixed positioned floating panels (e.g., toast, tooltip, modal)**
Handle: auto
Limitation: Controls cannot be positioned relative to the viewport. Everything is in document flow.
Alternative: Use a top-level ManualLayout GroupContainer as the outermost parent, place the floating panel there at fixed X/Y coordinates, and toggle its Visible property with a variable.

---

## ANIMATION & TRANSITIONS

**CSS transitions (hover animations, fade-ins, slide-ins)**
Handle: skip
Limitation: CSS animation is not supported in Canvas Apps. Animations are fundamentally incompatible with the Canvas Apps rendering model.
Alternative: None. Accept a static design. Interactive state changes (hover color, pressed state) are handled via control properties, not animation.

**Animated SVG (SMIL or CSS animation inside SVG)**
Handle: ask
Limitation: SVG animation support in Canvas Apps Image controls is inconsistent — SMIL animations may execute in some clients but not others.
Alternative A: Use an animated inline SVG via Image@2.2.3 for simple use cases (e.g., loading spinner, pulsing dot). Results may vary across devices. Acceptable for non-critical decorative animations.
Alternative B: Use a static SVG icon.
Alternative C: Omit.

**Lottie animations / animated illustrations**
Handle: skip
Limitation: No native Lottie player. Cannot be generated as plain Canvas Apps YAML.
Alternative: None available in this skill. Element will be omitted. Use a static SVG illustration as a replacement if needed.

**Screen / page transition animations**
Handle: auto
Limitation: Only the transitions built into Navigate() are available.
Alternative: Use Navigate() with the nearest available Transition enum: None, Fade, Cover, CoverRight, UnCover, UnCoverRight, Slide, SlideRight.

**Skeleton loading screens**
Handle: auto
Limitation: No native skeleton shimmer component.
Alternative: Use a Rectangle with a neutral fill (RGBA(229,231,235,1)) as a placeholder, Visible=true while data loads, Visible=false once the data-bound control is ready.

---

## INTERACTIVE STATES

**Hover state on GroupContainer**
Handle: auto
Limitation: GroupContainers do not have a HoverFill property.
Alternative: Overlay a transparent Classic/Button (same Width/Height) on top of the container in ManualLayout. Set HoverFill on the button. The button's OnSelect drives any click action.

**Focus ring / keyboard focus visible on GroupContainer**
Handle: auto
Limitation: GroupContainers are not natively focusable.
Alternative: The transparent Classic/Button overlay used for hover also handles keyboard focus and interaction.

**Right-click / long-press context menus**
Handle: ask
Limitation: No native right-click or long-press event. Canvas Apps only supports tap/click.
Alternative A: A visible context menu panel (GroupContainer + items) toggled by a variable, revealed by a standard Button tap. The interaction model changes from right-click to tap.
Alternative B: Omit the context menu entirely.

**Tooltip on GroupContainer or non-interactive control**
Handle: skip
Limitation: Tooltip is only available on controls that natively support the property. GroupContainers and most non-interactive controls do not. No workaround will be built.
Alternative: None. If the target control does not natively support Tooltip, the tooltip will be omitted from the generated YAML.

**Drag and drop between containers**
Handle: skip
Limitation: No native drag-and-drop between controls or lists in Canvas Apps.
Alternative: None generatable. Replace with action buttons (move up/down, transfer between galleries) if the interaction is critical — but this requires explicit user design input.

---

## MEDIA

**Autoplay video with no controls visible**
Handle: ask
Limitation: The Video control always renders with browser-native playback controls. There is no option to hide them or auto-start silently.
Alternative A: Use the Video control and accept the default player UI.
Alternative B: Omit the video element.

**Audio control with custom player UI**
Handle: ask
Limitation: The Audio control renders the browser's native audio element. It cannot be styled.
Alternative A: Accept the native audio player as-is.
Alternative B: Omit the audio element.

**Interactive embedded map (pan, zoom, markers)**
Handle: ask
Limitation: No native interactive map control in standard Canvas Apps. The Geospatial Map control is available only if the environment has the add-on enabled.
Alternative A: Use the Power Apps Geospatial Map control (requires the add-on to be enabled in the environment).
Alternative B: Use a static map image embedded via an Image control.
Alternative C: Omit the map.

**360 / VR / WebGL content**
Handle: skip
Limitation: No support for 3D, WebGL, or VR content types in Canvas Apps.
Alternative: None. Omit. Replace with a flat static image if a visual placeholder is needed.

---

## ICONS & GRAPHICS

**Icon font libraries (Font Awesome, Material Icons, Bootstrap Icons)**
Handle: auto
Limitation: Canvas Apps cannot load external icon fonts.
Alternative: Inline SVG data URIs via Image@2.2.3. This is the standard approach used throughout this skill — Classic/Icon is never used.

**SVG styling (gradients, multi-color fills)**
Handle: auto
Limitation: Not a hard limitation. Since SVG is inline HTML/XML text, both gradients (linearGradient, radialGradient) and multi-color paths are achievable via Image@2.2.3.
Alternative: Default to flat solid-color SVGs for simplicity and cross-device reliability. Use gradients or multi-color fills when the mockup explicitly shows them or the user requests it.

**Background image with CSS object-fit / cover / contain**
Handle: auto
Limitation: Image controls have Scale (fit, fill, stretch, tile, center) but behavior differs from CSS object-fit. Full-bleed background images behind overlapping content require ManualLayout.
Alternative: Image control with ImagePosition=ImagePosition.Fill for cover-style, placed behind other controls in ManualLayout with matching Width/Height.

---

## FORMS & INPUT CONTROLS

**Date/time picker with custom calendar styling**
Handle: auto
Limitation: The DatePicker control's calendar popup uses the system/browser default appearance and cannot be restyled.
Alternative: Use Classic/DatePicker as-is. No custom date picker will be built.

**Multi-select ComboBox showing selections as chip tags**
Handle: auto
Limitation: Classic/ComboBox with SelectMultiple=true shows selected items as comma-separated text, not as removable tag chips.
Alternative: Use Classic/ComboBox with SelectMultiple=true and accept the default display.

**Drag-and-drop file upload**
Handle: auto
Limitation: No drag-and-drop file upload target in Canvas Apps.
Alternative: Use the Attachment control for file selection via the OS file picker.

**Rich text editor with custom toolbar**
Handle: auto
Limitation: Classic/RichTextEditor has a fixed toolbar — bold, italic, lists, links. The toolbar cannot be customized.
Alternative: Use Classic/RichTextEditor and accept its default toolbar.

**Slider with custom-styled track and thumb**
Handle: ask
Limitation: The Slider control exposes only RailFill, ValueFill, HandleFill, and HandleActiveFill. The track shape and thumb shape are fixed.
Alternative A: Apply color overrides (RailFill/ValueFill/HandleFill) and accept the default shape.
Alternative B: Omit the slider and use a different input (e.g., TextInput with numeric validation).

**Input with inline leading/trailing icon inside the field boundary**
Handle: auto
Limitation: TextInput controls have no built-in slot for embedded icons inside the input border.
Alternative: ManualLayout GroupContainer as a wrapper: place the TextInput and an Image (icon) inside at overlapping X/Y positions, set the TextInput PaddingLeft to account for the icon width, and use the wrapper's border styling to simulate a unified input.

**Autocomplete / type-ahead dropdown beyond ComboBox**
Handle: auto
Limitation: Classic/ComboBox with IsSearchable=true provides basic type-ahead filtering. More advanced autocomplete (custom result formatting, async search, grouped results) is not natively supported.
Alternative: Use Classic/ComboBox with IsSearchable=true. This is the best available native option. Advanced autocomplete cannot be generated by this skill.

---

## KNOWN WORKAROUNDS & PATTERNS

**Simulating a card hover highlight**
Use a transparent Classic/Button overlay (matching the card's Width/Height, placed on top in ManualLayout) with HoverFill=RGBA(0,0,0,0.04). The card GroupContainer itself cannot receive hover events.

**Overlapping elements (modal dialog, floating badge, overlay panel)**
AutoLayout does not support overlap. Use a ManualLayout GroupContainer as the outermost container and use X/Y coordinates for all overlapping children. For full-screen modals, use a top-level ManualLayout container at X=0, Y=0, Width=Parent.Width, Height=Parent.Height.

**Collapsible / accordion section**
Use a variable (e.g., varSectionOpen) toggled by a Button OnSelect. The section container's Visible (or Height set to 0 vs. its natural height) responds to the variable. In AutoLayout, Visible=false on a child removes it from layout flow.

**Scrollable body with fixed header and footer**
Place header and footer as direct children of a Vertical AutoLayout parent with FillPortions=0. The scrollable body is a GroupContainer child with FillPortions=1 and LayoutOverflowY=LayoutOverflow.Scroll.

**Conditional field that collapses without a layout gap**
In AutoLayout, Visible=false on a control hides it but it still occupies space. To truly collapse: wrap the control in a GroupContainer and set the container's Visible=false, which removes it from layout. Alternatively, set Height=If(condition, naturalHeight, 0) on the control directly.
