# Content Philosophy

Use this reference while designing section summaries and questions.

## Study Then Quiz

Each section should begin with a short study block before the questions start. The study block should:
- explain the core idea in plain language
- highlight what matters most
- prepare the learner for the questions without simply giving away the answers

Keep it compact. The study block is a launchpad, not a full lesson rewrite.

## Infer Natural Sections

Prefer real document structure:
- explicit headings first
- paragraph/topic shifts second
- fallback chunking only when the document has no usable structure

Very short documents can stay as one section. Long documents should be broken where meaning changes, not at arbitrary line counts.

If the document begins with recap or previous-session review material, do not turn that recap into its own quiz section unless the user explicitly wants cumulative review. Treat recap as non-quizable framing and move to the first section that teaches new material.

## Write Questions That Test Understanding

Good question types:
- identify the main claim
- distinguish fact from implication
- apply an idea to a new example
- compare two viewpoints
- spot the strongest supporting evidence
- infer what would happen if one assumption changes
- explain a principle or tradeoff that would still matter outside this one class session

Avoid:
- rote definition recall when the concept is obvious from the study card
- trivia that depends on one isolated phrase
- ambiguous opinion questions with no grounded answer
- questions whose only source is recap, review, or previous-session quiz material rather than the new instructional content
- questions that mostly test session-specific coverage details, such as the exact count or names of items listed in class, when the enduring idea is more important than the list itself

When deciding what to ask, prefer durable knowledge:
- what principle should the learner remember later
- what contrast or tradeoff matters beyond this one session
- what misunderstanding would still matter in a new context

## Difficulty Tracks

Every section must include `low`, `medium`, and `deep` tracks.

- `low`: recognition, summary, and clear-text comprehension
- `medium`: interpretation, application, and multi-step reasoning
- `deep`: tradeoffs, subtle distinctions, implications, and edge cases

All three tracks should test the same section topic, but at different cognitive loads.

## Explanations

Immediate feedback must teach, not just score.

- correct feedback should reinforce the underlying idea briefly
- incorrect feedback should explain why the chosen answer misses the point
- explanations should reference the section meaning, not just repeat the right option

## Multiple-Choice Option Quality

Most options should be plausible to a thoughtful learner who only partially understands the section.

Rules:
- make distractors specific enough to feel possible
- avoid one obviously verbose answer paired with thin throwaway distractors
- keep the options similar in length, precision, and tone
- let wrong answers fail for substantive reasons, not because they are flimsy

## Depth Setting

The chosen quiz depth changes the study blocks and explanations:
- `Low`: shorter summaries, lighter context, clearer restatement
- `Medium`: balanced summary plus concise reasoning
- `Deep`: denser synthesis, nuance, implications, and stronger explanation detail

Do not create three versions of the entire page body. Difficulty switches live in the quiz. Depth is fixed at generation time.
