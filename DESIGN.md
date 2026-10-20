---
name: haven
description: The visual system of the Haven chat client: one vocabulary of tokens, twenty-six themes over it.
colors:
  primary: "#7c5cfc"
  neutral-bg: "#191b28"
  surface: "#1e2035"
  surface-raised: "#252840"
  text: "#e2e4f0"
  text-muted: "#9498b3"
  accent-glow: "rgba(124, 92, 252, 0.25)"
  success: "#43b581"
  danger: "#f04747"
  warning: "#faa61a"
typography:
  body:
    fontFamily: "'Segoe UI', system-ui, -apple-system, sans-serif"
    fontSize: "16px"
    fontWeight: 400
    lineHeight: 1.5
  title:
    fontFamily: "'Segoe UI', system-ui, sans-serif"
    fontSize: "20px"
    fontWeight: 700
    lineHeight: 1.25
  mono:
    fontFamily: "'Consolas', 'Courier New', monospace"
    fontSize: "14px"
    fontWeight: 400
    lineHeight: 1.45
rounded:
  md: "8px"
  sm: "4px"
  pill: "12px"
spacing:
  xs: "4px"
  sm: "8px"
  md: "16px"
  lg: "28px"
---

# Design system: haven

## Overview

**Creative North Star: "one set of bones, many coats of paint"**

Haven's client is a single page with a single stylesheet, and the stylesheet
is built in two layers. The first layer is a vocabulary of custom
properties: the surfaces, the accent, the text colours, the borders, the
status colours. The second layer is a set of themes, each of which fills
that vocabulary with its own values. Twenty-six themes ship: the default one
called Haven, and twenty-five that borrow their names from other worlds,
Discord and Matrix among them.

The reason for the split is that the client has no build step. The browser
loads `public/css/style.css` as it is on disk, and a theme is a block of
custom properties on an attribute selector. Changing the look of the whole
application is a matter of setting `data-theme` on the root element, with no
JavaScript in the way and nothing to compile.

The default theme is the one that has to work on its own. The other
twenty-five are a palette over the same components, and a few of them add
their own rules on top: the Matrix theme paints a scanline over the whole
page, the Windows 95 theme replaces every radius with a bevel, and the
trench-coat themes put a gradient behind the wordmark.

Confirmed rejections: no CSS framework, no preprocessor, no utility classes,
no component library, and no custom properties outside the vocabulary. The
stylesheet is 14176 lines and it is one file.

Key characteristics:

- every colour in the interface comes from a token;
- every surface is a flat fill;
- depth comes from a border, a glow or a shadow, never from a gradient on a
  panel;
- the same component looks the same in every theme, because the theme only
  changes the values.

## Colors

The vocabulary has four groups, and a theme fills all of them. The values
below are the default theme's.

### Primary

- **Accent** (`#7c5cfc`): the colour of action. Primary buttons, the send
  button, the active channel marker, the mention, the inline code.
- **Accent hover** (`#9478ff`): the accent one step lighter, used on hover.
- **Accent dim** (`#5a3fd4`): the accent one step darker, used on a pressed
  state and on the glow behind an active element.
- **Accent glow** (`rgba(124, 92, 252, 0.25)`): a translucent accent, used
  for the halo behind a selected avatar and a selected reaction.

### Secondary

- **Text link** (`#82aaff`): the colour of a link in a message body. It is a
  separate token from the accent on purpose, so that a link stays legible
  when the accent is dark.
- **Success** (`#43b581`), **danger** (`#f04747`), **warning** (`#faa61a`):
  the three states, and the only colours in the interface that carry a
  meaning on their own.

### Neutral

- **Background primary** (`#191b28`): the chat surface, the ground of the
  main column.
- **Background secondary** (`#1e2035`): the sidebar, the right-hand panel,
  the channel header and the status bar.
- **Background tertiary** (`#252840`): one step further out. Cards, toasts,
  code blocks, reaction chips.
- **Background hover** (`#2c2f4a`) and **background active** (`#333660`):
  the two interaction states of a row.
- **Background input** (`#15172a`): the composer and every text field.
- **Background card** (`#1c1e33`): the surface of a modal.
- **Text primary** (`#e2e4f0`), **text secondary** (`#9498b3`), **text
  muted** (`#5d6180`): the three levels of text.
- **Border** (`#2d3050`) and **border light** (`#383b5e`): the two rules
  that separate one region from the next.
- **Led on** (`#43b581`) and **led off** (`#555`): the presence dots.

### Named rules

**The Token Rule.** A colour in the interface comes from a custom property.
A literal hex value in a rule is a second source of truth, and the themes
cannot reach it.

**The One Accent Rule.** The accent marks what can be acted on. It is not
decoration, and it is never used for a second family of things.

**The Two Light Themes.** Twenty-four of the twenty-six themes are dark.
`daylight` and `cloudy` are the light ones, and they are built from the same
vocabulary rather than from a separate set of rules.

## Typography

**Display Font:** `'Segoe UI', system-ui, sans-serif`.
**Body Font:** `'Segoe UI', system-ui, -apple-system, sans-serif`.
**Mono Font:** `'Consolas', 'Courier New', monospace`.

Two families, and the third slot is reserved for code. The stylesheet also
imports VT323 and Share Tech Mono from Google Fonts, and neither is
referenced by a rule that ships.

- **Body** (400, 16px, 1.5): the default. `body` sets the family, the
  colour and the line height, and leaves the size to the browser.
- **Message** (400, 14px, 1.45): the text of a message, which is the most
  read text in the application and is set smaller than the body on purpose.
- **Author** (600, 14px): the name above a message.
- **Title** (700, 20px): the heading of a modal, and the family of the
  brand.
- **Section label** (700, 11px, uppercase, 0.8px of letter spacing): the
  headings inside the sidebar and the panels.
- **Status label** (600, 10px, uppercase, 0.3px): the labels in the status
  bar.
- **Code** (13px, monospace): a fenced block. Inline code is 0.9em of its
  surroundings and takes the accent colour.

### Named rules

**The Mono Slot.** Anything that is code, a key or a number is monospace.
The status bar's values use it, the code blocks use it and the inline code
uses it.

## Layout

Three columns and a status bar, in a flex row that fills the viewport. The
page does not scroll; the message list does.

- **Server bar** (56px): the icons of the servers this browser knows about.
- **Sidebar** (270px by default, between 200px and 400px, resizable by
  dragging the handle on its edge): the channels, the members and the
  settings.
- **Main** (flexible): the channel header, the message list and the
  composer.
- **Right sidebar** (260px by default, between 200px and 400px): the member
  list and the channel details. It can be moved to the left.
- **Status bar** (28px tall at minimum): the connection state, the version
  and the count of unread messages.

The rhythm inside a column is small and regular: 4px inside a row, 8px
between the parts of a row, 16px inside a section, 28px inside a modal.

The message list is the densest part. A message row is a flex line with a
12px gap: a 36px avatar, then the author and the content. A compact message
pads its left edge by 56px so that it lines up under the text of the
message above it.

## Elevation & Depth

Flat, with three exceptions that are deliberate rather than decorative.

- A panel is separated from the ground by a 1px border, never by a shadow.
- A modal lifts with `0 12px 48px rgba(0,0,0,0.5)` and a toast with
  `0 4px 16px rgba(0,0,0,0.3)`, because they float above the message list
  and have to read as temporary.
- Some themes add a glow. The default theme has none; the Matrix theme
  paints a 4px green halo around the messages and a scanline over the whole
  page, and the sci-fi themes put a coloured shadow under the send button.

The `--msg-glow` token is `none` in the default theme and a shadow in ten
others. It exists so that a theme can add atmosphere without touching the
message rule.

## Shapes

- **Radius md** (8px): the default. Cards, toasts, buttons and panels.
- **Radius small** (4px): message rows, icon buttons, code blocks and the
  small buttons.
- **Radius pill** (12px): the reaction chips, which are half their own
  height.
- **Radius 12px**: a modal, the one shape that is not one of the two
  tokens.
- **Radius 0**: the inline code, the channel badge and the Windows 95
  theme, which sets `border-radius: 0` on everything.

Avatars come in five shapes, and they are the only place a clip path is
used: circle, rounded square at 6px, squircle at 28%, hexagon and diamond.

## Components

### Buttons

- **Shape:** 8px radius, no border in the default theme.
- **Primary:** the accent as the background, white text, 11px by 20px of
  padding, 15px type at weight 600.
- **Small:** the tertiary background, a 1px border, 14px type, 5px by 10px
  of padding.
- **Icon:** transparent, the muted text colour, 4px of padding, the small
  radius.
- **Hover and focus:** the background moves to `--bg-hover` or to
  `--accent-hover`. The transition is 0.15s ease.

### Inputs and fields

- **Style:** the input background, no border in the default theme, the
  small radius. The Windows 95 theme replaces it with a 3px inset bevel.
- **Composer:** the input background, the small radius, with the action
  buttons in a box above it.
- **Focus:** the browser's own ring. The stylesheet removes the outline
  from buttons and inputs and does not replace it.

### Message row

- **Shape:** the small radius, 4px by 8px of padding, a 12px gap.
- **Avatar:** 36px square, a circle by default, the first letter in white
  at weight 700 when there is no image.
- **Content:** 14px, 1.45 line height, the primary text colour, wrapping
  at any word.
- **Compact:** the same, padded on the left by 56px so that a run of
  messages from one person reads as a block.

### Channel item

- **Shape:** a flex row, 8px by 16px of padding, no radius of its own.
- **Active:** the active background, a 3px accent bar on the left edge, and
  the primary text colour.
- **Private:** the name and the hash at 60% opacity until the row is
  hovered.

### Badges

- **Channel badge:** the danger background, white text, 18px tall and at
  least 18px wide, a 9px radius, 10px type at weight 700.
- **Reaction chip:** the tertiary background, a transparent border, a 12px
  radius, 13px type. A reaction the person has sent carries the accent.

### Modal and toast

- **Modal:** the card background, a 1px light border, a 12px radius, 28px
  of padding, at most 400px wide and 85% of the viewport tall, with the
  large shadow.
- **Toast:** the tertiary background, a 1px light border, the 8px radius,
  13px type, a 360px maximum width. It fades in over 0.25s and out after
  3.5 seconds.

### Status bar

- **Shape:** a flex row with 16px gaps, 5px by 16px of padding, 11px type
  in the muted colour.
- **Labels:** 10px, uppercase, weight 600.
- **Values:** monospace, 11px, the secondary text colour.

## Do's and Don'ts

### Do:

- **Do** read every colour from a token, and add a token rather than a
  literal when a new colour is needed.
- **Do** fill the whole vocabulary when you write a theme. A theme that
  leaves `--border` out inherits the default theme's border, which is a
  silent half-applied theme.
- **Do** keep the dark themes dark and the two light themes light. A theme
  is a mood, and the interface is the same in all of them.
- **Do** test a change in the default theme first and in one themed theme
  second.

### Don't:

- **Don't** add a hex value to a rule. The themes cannot reach it.
- **Don't** add a second font family outside the three slots.
- **Don't** put a gradient on a panel. A gradient belongs to a wordmark or
  to a themed accent, and never to a surface.
- **Don't** add a shadow to a panel. Borders separate panels, and a shadow
  means the panel floats.
- **Don't** invent a spacing value outside the scale. The columns and the
  rows are already on it.
- **Don't** give a component a radius of its own. It takes `--radius` or
  `--radius-sm`, and the Windows 95 theme takes everything to zero.
- **Don't** edit `public/css/style.css` to add a theme. A theme is a block
  of custom properties at the top of the file, next to the others.
