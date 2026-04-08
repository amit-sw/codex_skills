# Document to Quiz for Codex

Codex skill package for turning a document into an interactive HTML quiz experience.

The generated output is a **directory-based quiz**:
- `styles.css` and `main.js` come from the skill references
- section HTML files live in `sections/`
- `build.sh` assembles the final `index.html`

This package adds:
- document-first intake for pasted text, local files, and URLs
- inferred sectioning from headings or topic breaks
- study-summary blocks before each section quiz
- three global difficulty levels: `Low`, `Medium`, `Deep`
- immediate answer checking on click
- running section and overall correct/incorrect counts
- default `10` questions per section, configurable during generation
