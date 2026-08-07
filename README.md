# DBD — Drill Baby Drill

A local-first, mobile-first drill engine for running AI-generated quizzes without directly integrating the website with an AI API.

DBD is designed around a simple separation of responsibilities:

> **The chat understands and generates. DBD conducts the drill and preserves the evidence.**

DBD does **not** attempt to become a material library, tutoring platform, syllabus manager, or AI frontend. Subject-specific chats remain responsible for understanding uploaded materials, teaching concepts, diagnosing weaknesses, and generating question packets. DBD is the execution layer.

Current version: **v0.9.3**

---

## Overview

DBD is a static HTML/CSS/JavaScript application intended to run on GitHub Pages or directly from a browser.

The normal workflow is:

1. Ask a subject chat to generate a DBD-compatible JSON packet.
2. Download or copy the JSON.
3. Import the packet into DBD.
4. Review the packet in preflight.
5. Run the drill.
6. Record correctness, confidence, skips, guesses, comments, timing, and error causes.
7. Review the full quiz and session report.
8. Copy either diagnostics or pure results back into the relevant subject chat.

No API key is required.

---

## Design principles

### Local-first

All drill data is stored in the browser under a stable local storage key:

```text
dbd_gazali
```

DBD can be used without a backend or account system.

### No direct AI integration

DBD will not directly integrate with:

- ChatGPT API
- OpenAI API
- Supabase
- Firebase
- cloud authentication
- a remote database

AI interaction remains manual through copied prompts and imported JSON files.

### Mobile-first

The interface is designed primarily for phone use:

- large touch targets
- JSON file import
- compact confidence controls
- fixed drill progress
- isolated question mode
- resumable sessions
- minimal navigation while drilling

### Evidence over question volume

DBD is not built to reward raw question counts.

The useful output is evidence about:

- what was correct
- what was wrong
- what was uncertain
- what was guessed
- what was not known
- what was skipped
- what took too long
- what the learner commented on
- which skills repeatedly failed

---

## Core workflow

```text
Subject chat
    ↓
Generate DBD JSON packet
    ↓
Download / copy JSON
    ↓
DBD preflight
    ↓
Run drill
    ↓
Session evidence
    ↓
Diagnostics / pure results
    ↓
Subject chat
```

DBD does not need to know the current chapter in advance. The subject chat determines the relevant scope from its own files, conversation history, recent work, and assessment context.

---

## Features

### Packet import

DBD supports:

- `.json` file import
- drag-and-drop import on desktop
- pasted JSON as a fallback
- persistent pending packets
- packet preflight before launch

For larger packets, file delivery is preferred over clipboard delivery.

### Preflight

Before a drill begins, DBD displays a centered mission briefing with:

- subject
- topic
- test type
- number of questions
- answer composition
- working style
- packet validation
- collapsed warnings
- collapsed validation details

Customization can be adjusted directly from preflight before launch.

### Test types

DBD supports six practical assessment purposes:

- **Coverage Scan** — broad sampling of what is known, shaky, or unknown
- **Focused Drill** — repeated work on a narrow skill
- **Deep Practice** — reasoning, transfer, interpretation, and application
- **Repair Drill** — fresh variants targeting previous mistakes
- **Mastery Check** — independent checkpoint after learning
- **Exam Simulation** — assessment-style performance under test conditions

### Working styles

- **No pen or paper** — conceptual and mental testing, especially useful for Math, Physics, and Chemistry
- **Full problem solving** — allows questions that require written working

### Answer formats

- **Multiple choice only**
- **Mixed**

Mixed packets may contain multiple-choice, short-answer, numeric, or other supported question types.

### Feedback timing

- **Immediate feedback**
- **After-session feedback**

Immediate feedback reveals correctness and explanation after submission.

After-session feedback keeps results hidden until the session is submitted.

### Timing

- **Off**
- **Record only**
- **AI-set limits**

For AI-set timing, each question may carry its own:

```json
"time_limit_seconds": 90
```

This allows easy and difficult questions to receive different limits.

### Honest-answer states

DBD distinguishes between correctness and confidence.

A learner can mark an answer as:

- **Sure**
- **Not sure**
- **Ngasal**
- **I don't know**
- **Skip for now**
- **Flag for review**

This prevents lucky guesses from being treated as strong evidence of mastery.

### Question comments

Each question can store an optional learner comment, for example:

```text
I knew the theorem but confused the sign.
```

or:

```text
This wording feels ambiguous — ask the chat about it later.
```

Comments are included in diagnostics and full reports.

### Ongoing sessions

DBD can preserve multiple unfinished drills.

Ongoing sessions appear at the top of the homepage and may include:

- active drills
- packets waiting at preflight
- progress through the packet
- resume controls
- discard controls

### Question mode

During an active drill:

- History and Data navigation are hidden
- the question becomes the dominant visual element
- only the Pause control remains in the active toolbar
- the hazard stripe remains fixed at the top
- progress and questions remaining are integrated
- questions can be revisited through the navigator
- delayed-feedback answers may be changed before final submission
- immediate-feedback answers remain locked after feedback is revealed

### Full quiz history

History stores both summary and full-quiz views.

The full quiz view includes:

- every question
- stimulus
- choices
- learner answer
- correct answer
- correctness
- confidence state
- explanation
- error classification
- timing
- tags
- notes
- comments

### Reports

DBD produces two intentionally different outputs.

#### Copy Diagnostics

Designed for sending back to the subject chat.

Includes:

- overall performance
- weak skills
- error patterns
- uncertain answers
- guessed answers
- don't-know responses
- timing problems
- comments
- wrong-question details

It is written as evidence for diagnosis and repair.

#### Copy Pure Results

A neutral factual record.

Includes the session results without telling the receiving chat what to do.

Useful for:

- archiving
- comparison
- manual analysis
- sharing raw results

### Local data tools

The Data page supports:

- browser persistence request
- JSON backup export
- JSON backup import
- full local-data reset

---

## JSON packet format

A typical DBD packet looks like this:

```json
{
  "dbd_version": "0.9.3",
  "campaign": "School",
  "subject": "Matematika Tingkat Lanjut",
  "topic": "Polynomials",
  "source": "Current subject-chat materials",
  "test_type": "coverage_scan",
  "working_style": "no_paper",
  "answer_format": "mcq",
  "difficulty": "adaptive",
  "feedback": "immediate",
  "timing": "record_only",
  "questions": [
    {
      "id": "q1",
      "type": "mcq",
      "prompt": "Which expression is a polynomial in x?",
      "choices": {
        "A": "3x^2 - 2x + 1",
        "B": "x^-1 + 2",
        "C": "sqrt(x) + 1",
        "D": "1/(x+1)"
      },
      "answer": "A",
      "explanation": "A polynomial uses non-negative integer exponents of the variable.",
      "why_wrong": {
        "B": "The exponent -1 is negative.",
        "C": "sqrt(x) is x^(1/2), which has a fractional exponent.",
        "D": "The variable appears in the denominator."
      },
      "tags": [
        "polynomial definition"
      ],
      "difficulty": "easy",
      "paper_required": false
    }
  ]
}
```

### Supported question fields

Common fields include:

| Field | Purpose |
| --- | --- |
| `id` | Unique question identifier |
| `type` | Question type |
| `prompt` | Main question text |
| `stimulus` | Optional quote, passage, or context |
| `choices` | Multiple-choice options |
| `answer` | Correct answer |
| `accepted_answers` | Alternative valid typed answers |
| `tolerance` | Numeric tolerance |
| `explanation` | Post-answer explanation |
| `why_wrong` | Specific distractor explanations |
| `tags` | Skill/subtopic labels |
| `difficulty` | Easy, medium, or hard |
| `paper_required` | Whether written scratch work is expected |
| `time_limit_seconds` | AI-assigned time limit for the question |

---

## Running locally

DBD is a static application.

No build system is required.

### Option 1 — Open directly

Download `index.html` and open it in a browser.

### Option 2 — GitHub Pages

Place the application at the repository root:

```text
index.html
README.md
```

Then serve the repository through GitHub Pages.

### Option 3 — Local development server

Any simple static server works, for example:

```bash
python -m http.server 8000
```

Then open:

```text
http://localhost:8000
```

---

## Data persistence

DBD currently uses browser-local storage.

This means data normally persists across:

- page reloads
- browser restarts
- updates to `index.html` on the same origin

It does **not** automatically synchronize across browsers or devices.

For device migration, use:

```text
Data → Export Backup
```

and import that backup on the second device.

Cloud synchronization is intentionally outside the project scope.

---

## Project structure

The production application is intentionally small.

```text
/
├── index.html
└── README.md
```

The application is currently kept as a single HTML file containing:

- markup
- styles
- application logic

This keeps deployment simple and makes the project easy to host on GitHub Pages.

---

# Changelog

The changelog below tracks the major DBD releases and design changes that led to the current application.

## v0.9.3

### Added

- Per-question comments.
- **Copy Diagnostics** report action.
- **Copy Pure Results** report action.
- Integrated questions-left text into the progress display.
- Larger, more visually dominant question card.
- More stable question/feedback sizing to reduce layout jumps.
- Fixed top hazard stripe during scrolling.

### Changed

- Active question mode now hides History and Data navigation.
- DBD logo acts as the main Drill/Home navigation control.
- Active drill toolbar now uses Pause without a separate Exit button.
- Homepage no longer displays raw or secure accuracy.
- Primary homepage workflow simplified around packet import.
- **Import Packet** and **Copy Vanilla Instruction** moved above customization.
- Custom drill generation moved into a collapsed **Custom Drill** panel.
- Generate Chat Request moved inside Custom Drill.
- Removed unnecessary instructional copy from the main builder.
- Ongoing sessions remain prioritized at the top of the homepage.
- Question design revised toward a cleaner mobile quiz-app hierarchy.

---

## v0.9.2

### Added

- Collapsible homepage customization.
- **Copy Vanilla Instruction** workflow.
- Pending imported packets stored as ongoing sessions.
- Direct customization inside the preflight modal.
- Cleaner preflight validation sections.

### Changed

- Removed the daily DBD section.
- Ongoing sessions moved to the top of the homepage.
- Importing a packet updates expected question count from the packet itself.
- Success validation text moved into collapsed details.
- Warnings moved into a collapsed section.
- Preflight became more responsive on narrow mobile screens.
- Removed standalone Edit Customization and Replace Packet controls from preflight.
- Removed the dedicated previous-question footer button; navigation remains available through the question navigator.

### Fixed

- Eliminated false warnings such as a 30-question imported packet being compared against a stale five-question customization.
- Improved mobile preflight overflow and button layout.

---

## v0.9.1

### Added

- Six test types:
  - Coverage Scan
  - Focused Drill
  - Deep Practice
  - Repair Drill
  - Mastery Check
  - Exam Simulation
- Multiple ongoing sessions.
- Previous-question navigation and question navigator.
- **Sure**, **Not sure**, and **Ngasal** confidence states.
- **I don't know** response.
- **Skip for now**.
- **Flag for review**.
- Raw versus secure accuracy internally.
- Paperless conceptual testing mode.
- Daily DBD flow prototype.
- AI-assigned per-question timing support.
- Mobile-oriented session controls.

### Changed

- Multiple-choice selection no longer immediately submits an answer.
- Learner selects an answer, confidence state, and then submits.
- Question-count presets reduced to 5, 10, and 20 while retaining a custom count field.
- Answer format simplified to Multiple Choice Only or Mixed.
- Ongoing sessions moved below the primary builder in the initial v0.9.1 implementation.

---

## v0.9

Design milestone only. The planned v0.9 feature set was folded into v0.9.1 rather than released as a separate stable build.

Major concepts introduced during this milestone included:

- backward navigation
- confidence-aware answers
- skips
- paperless conceptual drilling
- daily drill flow
- multiple ongoing sessions

---

## v0.8.5

### Added

- Multiple-choice-only, typed-only, and mixed answer-format controls.
- AI-assigned per-question time limits.
- Full Quiz view in History.
- Downloadable full quiz reports.
- Question-level report details:
  - learner answer
  - correct answer
  - explanation
  - timing
  - tags
  - error cause
- Edit Customization from preflight.
- Revalidation of an already imported packet after customization changes.

### Changed

- Replaced broad test-purpose abstractions with more explicit drill configuration during the first v0.8.5 design.
- Timing options simplified to:
  - Off
  - Record only
  - AI-set limits
- Removed fixed universal question/session timers.
- Preflight validates requested answer format.
- Large packets are instructed to use downloadable JSON files instead of relying on phone clipboard capacity.

### Fixed

- Immediate feedback now appears correctly on the final question before the drill ends.
- Delayed-feedback sessions no longer intentionally reveal self-check feedback mid-session.

---

## v0.8

### Added

- JSON file import.
- Drag-and-drop JSON import.
- Packet preflight.
- Packet validation and warning system.
- Immediate versus end-of-session feedback.
- Quote, passage, and text-context stimulus support.
- Pause and resume.
- Session autosave.
- Wrong-question retry.
- Wrong-and-slow-question retry.
- Stronger session reporting.
- Downloadable TXT and JSON reports.
- Timing statistics.
- Accuracy by skill tag and difficulty.
- Mobile-first drill controls.

### Changed

- DBD became a more complete standalone drill runner rather than only a JSON question viewer.
- Packet validity is checked before entering the session.

---

## v0.7

### Added

- Single-screen drill builder.
- Subject dropdown.
- Drill mode selection.
- Question-count customization.
- Difficulty selection.
- Optional focus instructions.
- Saved local drill configuration.
- Local browser persistence.
- Stable `dbd_gazali` storage key.
- Backup import/export.
- Initial prompt-generation workflow.
- Material-selection gate in generated chat instructions.

### Changed

- Removed the dedicated Subjects page.
- Current subject material was no longer hard-coded into DBD.
- Subject chats became responsible for deciding what material should be tested.
- DBD focused more strictly on execution and evidence.

---

## v0.6

### Added

- Gazali-specific campaign structure.
- Stable browser storage.
- Migration from earlier v0.5 storage keys.
- Data page.
- Persistent-storage request.
- Backup export/import.
- Reset controls.

### Changed

- Subject entries stopped being permanently tied to the chapter being studied at the moment.
- The relevant subject chat became responsible for judging current material.
- Permanent campaign list was reduced to genuinely active DBD lanes.
- DBD's role was clarified as:
  - import question packet
  - drill
  - record evidence
  - generate report
  - return evidence to the subject chat

### Removed

- Heavy material-library behavior.
- Hard-coded syllabus assumptions.
- Inactive permanent campaign entries.
- Direct dependency on NotebookLM.

---

## v0.5

First clearly defined Gazali-specific DBD MVP.

### Added

- LocalStorage-based persistence.
- AI prompt generation.
- JSON packet import.
- Question-by-question drills.
- Mistake logging.
- Basic timing.
- Session reports.
- Subject/campaign organization.

### Established

The core DBD loop:

```text
Learn in chat
→ generate packet
→ drill in DBD
→ record mistakes
→ return report to chat
```

This version established the basic architecture that later releases refined.

---

## Pre-v0.5 prototype

The earliest prototype focused on proving the basic concept:

- use an AI-generated question packet
- run questions in a lightweight webpage
- record answers and mistakes
- keep the execution environment separate from the teaching chat

The early prototype was progressively replaced by the Gazali-specific v0.5 architecture.

---

## Roadmap

Potential future work should continue to respect the project's local-first boundary.

Reasonable future improvements include:

- richer local analytics
- more robust packet schema validation
- better question navigation
- safe structured table rendering
- image-based stimulus packages
- improved local backup/version migration
- accessibility improvements
- more refined mobile animation and interaction design

Explicitly out of scope:

- direct ChatGPT integration
- AI API calls from DBD
- Supabase
- cloud accounts
- server-side user profiles
- automatic cross-device synchronization

---

## Status

DBD is an evolving personal learning tool rather than a general-purpose commercial product.

The current design target is simple:

> Make it extremely easy to import a good question packet, answer honestly, preserve the evidence, and close the learning loop.
