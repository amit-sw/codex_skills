---
name: codebase-to-course
description: "Turn any codebase into a polished, interactive HTML course that teaches how the code works to non-technical and technical-curious learners. Use this skill whenever someone wants an interactive course, tutorial, walkthrough, or explainer generated from a repo or project. Also trigger when users mention 'turn this into a course,' 'explain this codebase interactively,' 'teach this code,' 'interactive tutorial from code,' 'codebase walkthrough,' or 'make a course from this project.' This skill produces a directory-based course with structured modules, animated visualizations, code-with-plain-English translations, and multi-level quizzes."
---

# Codebase-to-Course

Turn a codebase into a modern, interactive course that explains how it works. The output is a **directory** containing pre-built `styles.css` and `main.js`, per-module HTML files, and an assembled `index.html`. The course is designed for learners who want to understand software well enough to steer AI tools, debug issues, and make better technical decisions.

## Intake First

Before inspecting the repo, gather missing specification details. If the user already answered an item in their request, do not ask it again.

Ask for:
- **Codebase target**: local path, current repo, or GitHub URL
- **Explanation depth**: `Low`, `Medium`, or `Deep`
- **Questions per module**: default `5`
- **Audience**: who the course is for
- **Goal focus**: prioritize `architecture`, `debugging`, `AI steering`, or `product flow`
- **Optional emphasis / de-emphasis**: anything to especially cover or skip

Defaults when the user does not answer:
- Depth: `Medium`
- Questions per module: `5`
- Audience: non-technical or technical-curious builder who understands the product but not the code
- Goal focus: balanced, with extra weight on architecture and debugging

Do **not** ask the user to explain what the app does if that can be learned from the repo after intake.

If the skill is triggered without a codebase target yet, introduce yourself briefly and ask the missing intake questions in one compact message.

If the user provides a GitHub URL, clone it first into `/tmp/<repo-name>` before analysis. If they say "this codebase" or similar, use the current working directory.

## Who This Is For

The default learner is a vibe coder or technical operator: someone who ships software with AI assistance but wants better mental models for how systems actually work.

Assume the learner may have little formal CS background unless the intake answers say otherwise. Define terms aggressively, explain cause and effect in plain English, and keep every module tied to practical value:
- steering AI assistants better
- debugging faster
- recognizing architecture boundaries
- making smarter implementation requests

## Output Contract

The course output is a **directory**, not a single hand-written HTML blob.

```text
course-name/
  styles.css
  main.js
  _base.html
  _footer.html
  build.sh
  briefs/          # complex codebases only
  modules/
    01-intro.html
    02-actors.html
    ...
  index.html
```

Copy these reference files verbatim into the course directory:
- `references/styles.css`
- `references/main.js`
- `references/_footer.html`
- `references/build.sh`

Customize `references/_base.html` with:
- `COURSE_TITLE`
- `ACCENT_*` values
- `NAV_DOTS`

Then write only the module HTML and assemble with:

```bash
cd course-name && bash build.sh
```

## Course Shape

Most courses should be **4-6 modules**. Go to 7-8 only when the codebase genuinely warrants it.

Each module should include:
- 3-6 screens
- at least one code-to-English translation block
- at least one interactive element
- one or two high-value callouts
- one quiz with three difficulty tracks: `low`, `medium`, `deep`

Across the course, always include:
- code-to-English translation blocks
- glossary tooltips
- at least one group chat animation
- at least one message flow animation
- at least one quiz per module

## Depth And Quiz Rules

The selected **depth** changes explanation density, teaching pace, and how much architecture/debugging detail to include:
- `Low`: explain the big picture, avoid overloading, shorter translations and fewer layers
- `Medium`: balanced detail, enough internals to support decisions and debugging
- `Deep`: denser tracing, more design tradeoffs, more failure-mode coverage

The selected **questions per module** is a build-time setting. Apply it consistently across the course unless a module would become obviously bloated.

Quiz difficulty is different from explanation depth:
- Explanation depth is chosen during intake and stays fixed across the course
- Quiz difficulty is selectable inside the generated course UI
- Every module quiz must include `low`, `medium`, and `deep` tracks
- The course should default to the `medium` quiz track on first load

## Process

### Phase 1: Intake

Get the brief first. Ask only the missing high-impact questions listed above. Once the brief is complete enough, proceed without waiting for extra approval.

### Phase 2: Codebase Analysis

Read the important files, trace the main user journey, and identify:
- the main actors
- the product flow
- key APIs and data flows
- noteworthy patterns
- likely gotchas
- the tech stack and why it exists

Figure out what the app does yourself from README, entry points, UI code, and system boundaries.

### Phase 3: Curriculum Design

Start from the learner's visible experience, then progressively peel back layers. Keep each module tied to practical usefulness.

Typical arc:
1. What the product does and what happens during a core user action
2. Main actors and responsibilities
3. How data moves between the pieces
4. External systems and boundaries
5. Useful patterns and tradeoffs
6. Failure modes and debugging

Use fewer modules for simple codebases.

### Phase 4: Build Path Decision

- **Simple codebase**: write modules sequentially
- **Complex codebase**: create module briefs first, then write in parallel when helpful

For complex codebases, write a brief per module in `briefs/` using `references/module-brief-template.md`.

### Phase 5: Build

Write module files that contain only the `<section class="module" ...>` block. Do not include page boilerplate in module files.

When writing quizzes:
- emit one quiz container per module
- include three quiz tracks inside each quiz container: `data-quiz-level="low"`, `medium`, and `deep`
- keep the narrative and module body shared across all learner levels

Read these references only when needed:
- `references/content-philosophy.md`
- `references/gotchas.md`
- `references/design-system.md`
- `references/interactive-elements.md`
- `references/module-brief-template.md`

### Phase 6: Review

After building:
- run `build.sh`
- open `index.html`
- verify the nav dots, progress bar, quiz level selector, tooltips, and animations
- confirm that switching quiz difficulty changes the active question set without changing the explanatory content

## Design Direction

The frontend should feel like **editorial product documentation**:
- quiet neutral surfaces
- a single slate-blue accent
- modern, professional typography
- restrained motion
- fewer competing colors
- dark code panels with strong readability

Avoid the generic AI aesthetic: loud gradients, candy colors, purple-heavy palettes, or overly playful surfaces.

## Critical Rules

- Never regenerate `styles.css` or `main.js`; copy them from `references/`
- Do not ask the user for the product explanation if the repo can tell you
- Use `scroll-snap-type: y proximity`, never `mandatory`
- Use `min-height: 100dvh` with `100vh` fallback on modules
- Module files contain only section content
- Quiz difficulty is learner-selectable in the generated UI; explanatory depth is not
