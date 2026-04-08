# Design System Reference

This package should feel like a professional assessment interface, distinct from the slate-blue course package.

## Visual Direction

Target:
- parchment and stone neutrals
- restrained forest, olive, copper, or charcoal accents
- editorial typography
- calm, high-legibility surfaces
- minimal decorative color

Avoid:
- blue accents
- candy colors
- dashboard-like density

## Core Tokens

```css
:root {
  --color-bg:             #F5F1E8;
  --color-bg-warm:        #EFE9DD;
  --color-bg-code:        #221D18;
  --color-text:           #2F2A24;
  --color-text-secondary: #655D54;
  --color-text-muted:     #8A8278;
  --color-border:         #DDD4C7;
  --color-border-light:   #ECE5DB;
  --color-surface:        #FCFAF6;
  --color-surface-warm:   #F7F2EA;

  --color-accent:         #4D6A4F;
  --color-accent-hover:   #3D5540;
  --color-accent-light:   #E9F0EA;
  --color-accent-muted:   #95A794;

  --font-display: 'Newsreader', Georgia, serif;
  --font-body:    'Instrument Sans', -apple-system, sans-serif;
  --font-mono:    'JetBrains Mono', monospace;
}
```

## Shell Components

The shell should include:
- title and short subtitle
- overall correct / incorrect score chips
- global difficulty selector
- global reset button
- section nav dots

## Section Layout

Each section should contain:
- a section header
- a study card
- one quiz container

Use clear spacing and keep the study block visually separate from the quiz.

## Quiz Layout

Every quiz container should support:
- section score readout
- a difficulty note
- three `.quiz-track` blocks with `data-quiz-level`
- immediate feedback panels
- a section reset action

Correct and incorrect states should be obvious without being loud.
