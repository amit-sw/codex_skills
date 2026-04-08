---
name: document-to-quiz
description: "Turn a document into a polished, interactive HTML quiz experience. Use this skill whenever someone wants to create a quiz, comprehension check, assessment, review activity, or study-and-quiz walkthrough from a document such as a paragraph, lesson, transcript, paper, article, notes, or URL. Also trigger when users mention 'turn this into a quiz,' 'quiz me on this lesson,' 'make an assessment from this transcript,' or 'create questions from this document.' This skill produces a directory-based HTML quiz with inferred sections, study summaries, three difficulty levels, immediate answer checking, and running section/global score counts."
---

# Document-to-Quiz

Turn a document into a professional interactive quiz. The output is a **directory** containing pre-built `styles.css` and `main.js`, per-section HTML files, and an assembled `index.html`.

The generated experience is designed for study and assessment:
- each section begins with a short study block
- each section then presents a quiz
- each question is graded immediately when clicked
- the learner sees both section-level and document-level correct/incorrect counts

## Intake First

Before reading the document, gather missing specification details. If the user already answered an item in their request, do not ask it again.

Ask for:
- **Document source**: pasted text, local file path, or URL
- **Explanation depth**: `Low`, `Medium`, or `Deep`
- **Questions per section**: default `10`
- **Audience**: who the quiz is for
- **Goal focus**: comprehension, recall, application, exam prep, discussion, or mixed
- **Optional emphasis / de-emphasis**: topics to stress or skip

Defaults when the user does not answer:
- Depth: `Medium`
- Questions per section: `10`
- Audience: general learner or technical-curious reader
- Goal focus: mixed, with more weight on comprehension and application

If the user gives a URL or file path, read the document directly. Do **not** ask them to paste the whole document unless access fails.

## Output Contract

The output is a **directory**, not a single HTML blob.

```text
quiz-name/
  styles.css
  main.js
  _base.html
  _footer.html
  build.sh
  briefs/          # optional for long or complex documents
  sections/
    01-intro.html
    02-core-idea.html
    ...
  index.html
```

Copy these reference files verbatim into the quiz directory:
- `references/styles.css`
- `references/main.js`
- `references/_footer.html`
- `references/build.sh`

Customize `references/_base.html` with:
- `QUIZ_TITLE`
- `ACCENT_*` values
- `NAV_DOTS`

Then assemble with:

```bash
cd quiz-name && bash build.sh
```

## Section Model

Infer sections from the document:
- prefer explicit headings
- otherwise use natural topic breaks
- if the document is unstructured, fall back to chunking by idea density and length

Exclude recap-only material from quiz generation:
- if a session opens with recap, review, warm-up, or previous-session quiz content, do not treat that material as a quizable section
- use it only as context for understanding the current session if needed
- start question generation from the new instructional content of the current session
- if a recap is mixed into a section, strip recap-specific material out of the study summary and question pool

Very short documents may become a single section. Longer documents usually become **3-8 sections**, depending on topic shifts.

Each section should include:
- a short study summary or lesson card
- optionally a short excerpt or key points list
- one quiz container
- one reset button for that section

## Difficulty And Question Count

The generated quiz must support three global difficulty levels:
- `Low`
- `Medium`
- `Deep`

Difficulty is controlled globally in the UI and switches every section quiz together.

Question count is a **build-time** setting:
- default `10` questions per section
- configurable during intake
- apply the same target count to all three difficulty tracks unless there is a strong document-specific reason to vary it

Difficulty and depth are different:
- **Depth** changes how rich the study summaries and explanations are
- **Difficulty** changes the cognitive load of the quiz questions

## Quiz Runtime Rules

Every section quiz must:
- grade the answer immediately when the learner selects an option
- show right/wrong feedback on the spot
- lock that question after grading
- keep the rest of the section active
- update both section-level and document-level correct/incorrect counts

Do not require a separate "Check answer" click for standard multiple-choice questions.

Provide:
- a section reset button
- a whole-page reset button in the shell

Changing difficulty should reset quiz progress and scores. Define this clearly in the generated experience and keep the behavior consistent.

## Process

### Phase 1: Intake

Get the missing brief information first. Ask only the missing high-impact questions listed above.

### Phase 2: Document Analysis

Read the source document and extract:
- major sections or topics
- key claims, arguments, or ideas
- terminology that needs defining
- likely misunderstandings
- facts vs interpretations vs implications
- recap/review material that should be excluded from quizzing

Build the quiz from the document itself. Do not ask the user to summarize the content if the document can be read directly.

### Phase 3: Section Design

For each inferred section, define:
- the section title
- a short study summary
- the most important knowledge to test
- question angles for `low`, `medium`, and `deep`

If a section contains both recap and new instruction, summarize only the new instruction and generate questions only from that new material.

Use `references/module-brief-template.md` as a section brief template if the document is long enough to benefit from explicit planning.

### Phase 4: Build

Write `sections/0N-slug.html` files containing only the `<section class="quiz-section" id="section-N">` block and its contents.

Use the reference files only as needed:
- `references/content-philosophy.md`
- `references/gotchas.md`
- `references/design-system.md`
- `references/interactive-elements.md`
- `references/module-brief-template.md`

### Phase 5: Review

After assembling:
- open `index.html`
- verify section navigation, difficulty switching, score counters, immediate grading, and resets
- confirm that question count and difficulty tracks match the intake brief

## Design Direction

The frontend should feel like an **editorial assessment interface**:
- parchment or stone neutrals
- a restrained non-blue accent, preferably forest, olive, copper, or charcoal-warm
- professional typography
- understated motion
- strong information hierarchy

Avoid the current slate-blue course look. This should feel related in quality but clearly different in identity.

## Critical Rules

- Never regenerate `styles.css` or `main.js`; copy them from `references/`
- Use `sections/`, not `modules/`
- Question count is configurable at generation time, default `10 per section`
- Difficulty is global in the runtime UI; question count is not
- Answers are checked immediately on click
- Section and global score counters must remain accurate after grading, resetting, and difficulty changes
- Do not generate quiz questions from upfront recap, review, or prior-session quiz material unless the user explicitly asks to be quizzed on that recap
