# Gotchas

## Bad Sectioning

Do not split mechanically if the document has meaningful headings or topic boundaries. Poor sectioning creates arbitrary quizzes that feel disconnected from the source.

## Quizzing The Recap

Many lesson transcripts and session notes begin with recap, review questions, or a summary of the prior session. Do not generate the new section quiz from that recap material unless the user explicitly asks for cumulative review. The quiz should focus on the new instructional content of the current session.

## Study Blocks That Are Too Long

The section intro should prepare the learner, not rewrite the entire document. Keep it compact and focused on what matters for comprehension.

## Recall-Only Questions

If all questions are low-value recall, the quiz feels shallow. Even at `low` difficulty, some questions should test understanding rather than verbatim memory.

## Session-Specific Trivia

Questions like "which four items were listed" are usually weak unless the list itself is the enduring concept. Prefer asking about the underlying principle, comparison, advantage, limitation, or implication that the learner should retain later.

## Difficulty Tracks That Barely Differ

If `low`, `medium`, and `deep` look interchangeable, the global difficulty selector becomes meaningless. Increase the reasoning load clearly as difficulty rises.

## Delayed Grading

This package is built around immediate answer checking. Do not reintroduce a “Check answers” workflow for standard multiple-choice questions.

## Broken Score Counts

Every answer selection must update:
- the section correct / incorrect count
- the global correct / incorrect count

Resetting or changing difficulty must also bring those counters back into sync.

## Longest-Answer Bias

If the correct option is consistently the longest, learners will spot the pattern quickly. Wrong options should also be complete, plausible, and similar in length and specificity to the correct answer.

## Difficulty Switching Without Reset Rules

Changing the global difficulty must have deterministic behavior. In this package, switching difficulty resets progress and scores so the learner is never comparing mixed tracks.
