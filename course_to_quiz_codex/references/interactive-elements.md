# Interactive Elements Reference

This package is intentionally focused. The core interaction is a study block followed by an immediate-feedback quiz.

## Section Shell

Each section file should contain one section block:

```html
<section class="quiz-section" id="section-1">
  <div class="section-content">
    <header class="section-header">
      <span class="section-number">01</span>
      <h1 class="section-title">Section Title</h1>
      <p class="section-subtitle">One-sentence orientation for the learner.</p>
    </header>

    <article class="study-card animate-in">
      <span class="study-kicker">Study</span>
      <h2 class="study-title">What this section is really about</h2>
      <p>Short summary paragraph.</p>
      <ul class="study-points">
        <li>Key point one</li>
        <li>Key point two</li>
        <li>Key point three</li>
      </ul>
    </article>

    <!-- quiz container goes here -->
  </div>
</section>
```

## Quiz Container

Every section quiz should use one container with three difficulty tracks.

```html
<div class="quiz-container" id="quiz-section1">
  <div class="quiz-toolbar">
    <div>
      <span class="quiz-kicker">Section Quiz</span>
      <p class="quiz-level-note">Current difficulty: <span data-quiz-level-label>Medium</span></p>
    </div>
    <div class="section-score" aria-label="Section score">
      <span class="score-pill score-pill-correct"><strong data-section-correct>0</strong> correct</span>
      <span class="score-pill score-pill-incorrect"><strong data-section-incorrect>0</strong> incorrect</span>
    </div>
  </div>

  <div class="quiz-track" data-quiz-level="low">
    <!-- low questions -->
  </div>

  <div class="quiz-track active" data-quiz-level="medium">
    <!-- medium questions -->
  </div>

  <div class="quiz-track" data-quiz-level="deep">
    <!-- deep questions -->
  </div>

  <div class="quiz-actions">
    <button class="btn" type="button" onclick="resetQuiz('quiz-section1')">Reset Section</button>
  </div>
</div>
```

## Question Block

Questions are graded immediately when the learner clicks an option.

```html
<div class="quiz-question-block"
     data-correct="option-b"
     data-explanation-right="Exactly. This answer best captures the section's main point."
     data-explanation-wrong="Not quite. Re-check the study summary and the relationship between the key ideas.">
  <h3 class="quiz-question">Question text here?</h3>
  <div class="quiz-options">
    <button class="quiz-option" data-value="option-a" type="button" onclick="selectOption(this)">
      <div class="quiz-option-radio"></div>
      <span>Answer A</span>
    </button>
    <button class="quiz-option" data-value="option-b" type="button" onclick="selectOption(this)">
      <div class="quiz-option-radio"></div>
      <span>Answer B</span>
    </button>
    <button class="quiz-option" data-value="option-c" type="button" onclick="selectOption(this)">
      <div class="quiz-option-radio"></div>
      <span>Answer C</span>
    </button>
  </div>
  <div class="quiz-feedback"></div>
</div>
```

Rules:
- no extra check button per question
- after selection, that question locks
- explanations appear immediately
- difficulty tracks should contain parallel question sets for the same section topic

## Glossary Terms

If the study block uses jargon, wrap it with the standard tooltip pattern:

```html
<span class="term" data-definition="A short plain-English definition.">Term</span>
```
