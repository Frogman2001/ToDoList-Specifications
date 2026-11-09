---
name: ToDoList
description: Plain, quiet, dark-first task app. Areas are defined by lines and tone, not decoration. Little color; state is never shown by color.
status: final
updated: 2026-11-04
colors:
  background: '#14181a'
  surface: '#1c2224'
  line: '#2c3639'
  text: '#e6ece9'
  text-muted: '#8a9892'
  accent: '#5fb386'
  surface-picked: '#212a2c'
  error: '#d9776f'
typography:
  body:
    fontFamily: system sans-serif
    fontSize: 16px
    fontWeight: 400
  day-header:
    fontFamily: system sans-serif
    fontSize: 16px
    fontWeight: 600
  meta:
    fontFamily: system sans-serif
    fontSize: 13px
    fontWeight: 400
rounded:
  sm: 4px
spacing:
  '1': 4px
  '2': 8px
  '3': 12px
  '4': 16px
  '5': 24px
  '6': 32px
  row-min-height: 52px
  tap-target-min: 48px
components:
  day-box:
    backgroundColor: '{colors.surface}'
    borderColor: '{colors.line}'
    rounded: '{rounded.sm}'
  day-header:
    typography: '{typography.day-header}'
    textColor: '{colors.text}'
    borderBottomColor: '{colors.line}'
  day-header-today:
    typography: '{typography.day-header}'
    textColor: '{colors.accent}'
  item-row:
    typography: '{typography.body}'
    textColor: '{colors.text}'
    minHeight: '{spacing.row-min-height}'
    padding: '{spacing.4}'
    borderBottomColor: '{colors.line}'
  item-row-done:
    textColor: '{colors.text-muted}'
  item-row-picked:
    backgroundColor: '{colors.surface-picked}'
    outlineColor: '{colors.accent}'
  state-box:
    size: 22px
    strokeWidth: 2px
    rounded: '{rounded.sm}'
  button-outline:
    textColor: '{colors.accent}'
    borderColor: '{colors.accent}'
    rounded: '{rounded.sm}'
  button-solid:
    backgroundColor: '{colors.accent}'
    textColor: '{colors.background}'
    rounded: '{rounded.sm}'
  error-bar:
    textColor: '{colors.error}'
    borderColor: '{colors.line}'
    backgroundColor: '{colors.surface}'
  text-area:
    backgroundColor: '{colors.background}'
    borderColor: '{colors.line}'
    textColor: '{colors.text}'
    placeholderColor: '{colors.text-muted}'
    minHeight: '{spacing.tap-target-min}'
---

## Brand & Style

A plain, working tool. Nothing is decorative. Areas are separated by a thin line or a small shift in background tone, never by shadow or gradient. Buttons and links are words ("Add", "Sign In"), and icons appear only where an icon is truly the clearest way to say something. The one place that is, is the item state. The app is dark first. The look is expected to be replaceable with other themes later (web CSS swap, decided in architecture), so every value here is a named token.

## Colors

Little color on purpose. Color is never used to show state.

- **Background (`{colors.background}`)** is the canvas and the fill of text boxes.
- **Surface (`{colors.surface}`)** is the day box, one step lighter than the canvas.
- **Line (`{colors.line}`)** separates headers, rows, and edges.
- **Text (`{colors.text}`)** is for open items and headings: near white.
- **Text muted (`{colors.text-muted}`)** is for done items, placeholder text, and secondary text. A medium gray that makes open items carry more weight.
- **Accent (`{colors.accent}`)** is a calm green. It is used for the Add button, today's date header, and keyboard or picked-up focus. Nothing else.
- **Surface picked (`{colors.surface-picked}`)** is the slightly lighter tone of an item being moved.
- **Error (`{colors.error}`)** is a muted red, used only for the error message text. It is lighter than a true dark red on purpose: a dark red would be nearly invisible on a dark surface. The message is also words, so color is never the only signal.

Avoid: red or any color for overdue items or state; coloring items by state; using accent for decoration.

## Typography

One normal sans-serif per theme, using each platform's system font by default. The font is a theme token and may change per theme. Two sizes do almost all the work: body and day header (the same size, the header heavier), with a small meta size for secondary text. No all-caps labels. Text stays legible at large system font sizes without truncating controls.

## Layout & Spacing

Spacing is roomy. Scale: 4, 8, 12, 16, 24, 32 px. Item rows are at least 52px tall, and no tap target is under 48px. Day boxes sit side by side. Once the number of columns is chosen from the available width, the columns expand to fill all of it, so spacing grows with the screen and nothing is left empty at the edge. Column counts: phone portrait shows one day (plus Add new on the phone); phone landscape two; a small tablet two or three; wider browsers more, up to seven for a week. On narrow screens the week scrolls sideways.

## Elevation & Depth

None. Depth comes from the tonal step between background and surface, and from lines. No shadows. A popup is set apart by its border and surface tone.

## Shapes

One small corner radius (`{rounded.sm}`, 4px) on day boxes, text boxes, buttons, and state boxes. Nothing fully rounded.

## Components

- **Day box** — `{components.day-box}`. A header, then item rows, then an Add control. The header is a date such as `Fri Oct 30`.
- **Day header** — Date as weekday, month, day. Today's date is the only header in `{colors.accent}`. In the Today view, overdue items sit under their own due-date header; there is no "Overdue" or "Today" label.
- **Item row** — State box, then description. Separated from the next row by a line. Open items use `{colors.text}`.
- **Item row, done** — Same layout. Only the text color changes, to `{colors.text-muted}`. No background change and no strike-through.
- **State box** — Always a 22px box with a 2px outline. To do: empty. In progress: a centered dot. Done: a check mark inside the box. Never filled, and never color-coded.
- **Item row, picked up** — `{components.item-row-picked}`: a slightly lighter tone and an accent outline.
- **Buttons** — Words only. `button-outline` for Add in a day box; `button-solid` for the main action in a popup or on the Add new screen.
- **Text area** — A boxed field, never bare text. Placeholder `Add description...` in `{colors.text-muted}`. The date field next to it echoes the understood date in meta text below it.
- **Error bar** — `{components.error-bar}`: a lined bar at the top of the screen reading `Error reaching server`, in muted red text.
- **Loading animation** — The three state boxes cycling: empty, dot, check, repeat, on a blank list. Static when reduced motion is requested.

## Do's and Don'ts

| Do | Don't |
|---|---|
| Separate areas with a line or a small tone shift | Shadows, gradients, or decorative borders |
| Words for buttons and links ("Add", "Sign In") | Icons next to text, or icons that need guessing |
| Show state by box shape (empty, dot, check) | Show state by color |
| Gray the text of done items | Change the background of done items or strike them through |
| Grow spacing to fill the available width | Leave a gap at the edge of the screen |
| Animate only to explain (an item moving, a popup arriving, loading) | Animate for decoration |
| Use the accent rarely | Use accent for emphasis or ornament |
