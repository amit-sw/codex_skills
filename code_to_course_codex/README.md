# Codebase to Course for Codex

Codex skill package for turning a codebase into a polished, interactive HTML course.

The generated output is a **directory-based course**:
- `styles.css` and `main.js` are copied from the skill references
- modules are written as separate HTML sections
- `build.sh` assembles the final `index.html`

This Codex version adds:
- upfront intake questions for missing spec details
- build-time explanation depth selection: `Low`, `Medium`, `Deep`
- per-module quiz count control, default `5`
- learner-selectable in-course quiz difficulty: `Low`, `Medium`, `Deep`
- a calmer editorial frontend with neutral surfaces and a slate-blue accent

The Claude source package was reviewed and intentionally left unchanged. One inconsistency remains there: its README still describes a single offline HTML artifact, while the actual skill/package is directory-based and depends on Google Fonts.
