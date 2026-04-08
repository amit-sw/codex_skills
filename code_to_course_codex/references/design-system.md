# Design System Reference

Reference tokens and shell conventions for Codex-generated courses. The visual target is **editorial product documentation**: quiet neutrals, one restrained accent, crisp typography, subtle depth, and high readability.

## Color Palette

Use a neutral stone/slate foundation. Keep decorative color to a minimum.

```css
:root {
  --color-bg:             #F4F6FA;
  --color-bg-warm:        #EEF2F7;
  --color-bg-code:        #141C2B;
  --color-text:           #172131;
  --color-text-secondary: #4E5C72;
  --color-text-muted:     #7D8AA0;
  --color-border:         #D8E0EB;
  --color-border-light:   #E7EDF5;
  --color-surface:        #FFFFFF;
  --color-surface-warm:   #F8FAFD;

  --color-accent:         #4D6BA3;
  --color-accent-hover:   #3E5787;
  --color-accent-light:   #E9EEF8;
  --color-accent-muted:   #90A3CA;
}
```

Accent options for `_base.html`:
- `slate-blue`: `#4D6BA3 / #3E5787 / #E9EEF8 / #90A3CA`
- `ink-blue`: `#355C93 / #284874 / #E7EEF9 / #7994C0`
- `steel`: `#5C7598 / #4A5F7B / #EAF0F6 / #9BAEC7`
- `forest`: `#3C6A57 / #2F5445 / #E8F2ED / #83A996`
- `copper`: `#9A6548 / #7D523A / #F6EBE4 / #C39379`

Rules:
- alternate `--color-bg` and `--color-bg-warm` between modules for rhythm
- use accent sparingly for emphasis, controls, and active states
- keep actor colors muted and adjacent to the main palette, not rainbow-bright
- code always uses the dark code surface

## Typography

Use a modern, credible sans pairing.

```css
:root {
  --font-display: 'Manrope', 'Avenir Next', sans-serif;
  --font-body:    'Instrument Sans', -apple-system, sans-serif;
  --font-mono:    'JetBrains Mono', 'Fira Code', 'Consolas', monospace;
}
```

Google Fonts:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Instrument+Sans:ital,wght@0,400;0,500;0,600;0,700;1,400;1,500&family=JetBrains+Mono:wght@400;500;600&family=Manrope:wght@500;600;700;800&display=swap" rel="stylesheet">
```

Rules:
- module titles: display font, strong weight, tight tracking
- body copy: calm, readable, medium contrast
- labels and metadata: mono, uppercase, subtle
- avoid whimsical or retro type choices

## Layout And Rhythm

Keep modules spacious and easy to scan.

```css
:root {
  --content-width:      800px;
  --content-width-wide: 1000px;
  --nav-height:         72px;
  --radius-sm:          8px;
  --radius-md:          12px;
  --radius-lg:          16px;
}

.module {
  min-height: 100dvh;
  min-height: 100vh;
  scroll-snap-align: start;
  padding: var(--space-16) var(--space-6);
  padding-top: calc(var(--nav-height) + var(--space-12));
}
```

Rules:
- prefer one strong visual anchor per screen
- keep text blocks short
- let sections breathe rather than filling every gap

## Shell Components

The top navigation now includes:
- course title
- global quiz difficulty selector
- nav dots for modules

Quiz selector behavior:
- options are `Low`, `Medium`, `Deep`
- `Medium` is active by default
- changing quiz level swaps quiz tracks only
- explanatory body content stays the same

## Motion And Depth

Motion should feel informative, not decorative.

Rules:
- use subtle reveal animations and restrained hover elevation
- avoid loud bounces, heavy parallax, or long looping motion
- use soft shadows derived from slate/dark-ink values, never hard black

## Quiz Styling

Quiz cards should feel like product UI, not classroom worksheets:
- clean borders
- soft tinted surfaces
- clear active state
- compact toolbar with current level indicator

Every quiz container should support:
- a toolbar
- three `.quiz-track` blocks with `data-quiz-level`
- check and reset actions

## Code Presentation

Keep code panels dark and readable:
- no horizontal scrollbars
- wrap lines with `pre-wrap`
- preserve exact project code
- use syntax color sparingly; readability is more important than spectacle
