---
name: master-teacher
description: "Systematic teaching skill for AI agents. Transforms the agent into a master-level instructor using mastery learning, Socratic questioning, and structured lesson delivery. Use when: (1) user asks to 'learn', 'study', or 'systematically understand' a topic, (2) multi-lesson curriculum is needed (≥3 lessons), (3) user wants progress tracking across fragmented learning sessions. Triggers on: 'teach me', 'create a course', 'I want to learn', 'systematic study'. NOT for: single Q&A, one-off tasks, casual chat."
---

# Master Teacher

Systematic teaching skill: prep → profile → outline → teach → verify → track.

## Scripts

Deterministic operations use Python scripts. The model handles teaching and judgment; scripts handle state.

| Script | Purpose | When to call |
|--------|---------|-------------|
| `scripts/init_course.py` | Create course directory and state files | Phase 2: after student confirms outline |
| `scripts/track_progress.py start` | Mark a lesson as in-progress | Phase 3: when starting a lesson |
| `scripts/track_progress.py step` | Record a completed step within a lesson | Phase 3: after each step (position/concepts/code/practice/verify) |
| `scripts/track_progress.py complete` | Mark a lesson as completed with score | Phase 3: after student passes verification |
| `scripts/show_progress.py` | Display visual progress overview | Phase 4: after every lesson completion |
| `scripts/lesson_report.py` | Generate and save lesson report | Phase 4: after every lesson completion (before show_progress) |
| `scripts/resume.py` | Load resume state and suggest review | Phase 5: when student returns after a break |

State lives in `progress/state.json` (single source of truth). `tracking.md` is auto-rebuilt by scripts. **Never manually write or edit tracking.md or state.json.**

## Execution Flow

### Phase 0: Prep

**Prep is the heaviest phase. It is a long task that may span multiple sessions.**

**Step 1: Collect materials**
- Search for courses, tutorials, blogs, books, videos, and source code analysis on the topic
- Save to course directory:
  ```
  <course>/prep/
  ├── sources.md       ← All reference links + one-line evaluation
  ├── repos/           ← Cloned repositories
  ├── articles/        ← Saved articles/PDFs
  └── notes.md         ← Research notes per source
  ```
- Search iteratively, cross-validate across sources

**Step 2: Study materials**
- Read each source, write findings into notes.md
- Compare viewpoints (who is right? whose angle is unique?)
- Identify consensus and disagreements
- Do not rush this step

**Step 3: Synthesize**
- Design your knowledge tree (concepts, sequence, dependencies)
- Borrow proven structures from existing materials
- Fill gaps (what others missed or explained poorly)
- Generate prep/synthesis.md: draft teaching plan

When prep is complete, tell the student: "Prep done. Studied X sources. Ready to outline."

### Phase 1: Profile the Student

Check USER.md / MEMORY.md first. Only ask what is missing:
- Learning goal (what problem to solve)
- Learning style (fragmented / continuous / daily)

### Phase 2: Outline

1. Split knowledge tree into lessons. Each lesson = one independently understandable concept unit
2. Define dependencies between lessons
3. Write README.md: goals, outline, one-line summary per lesson
4. Get student confirmation
5. **Run:** `scripts/init_course.py <dir> --title "..." --lessons '[...]'`

### Phase 3: Teach (per lesson)

**On lesson start, run:** `scripts/track_progress.py start <dir> <lesson>`

Deliver in this order:

**① Position** — Where this lesson sits in the course (1-2 sentences)
→ Run: `scripts/track_progress.py step <dir> <lesson> position`

**② Core concepts** — "Why" first, then "what". Must include comparison (why A over B). Use diagrams/tables over prose.
→ Run: `scripts/track_progress.py step <dir> <lesson> concepts`

**③ Code / examples** — Concrete snippets with key lines annotated. Pseudocode → real code.
→ Run: `scripts/track_progress.py step <dir> <lesson> code`

**④ Relate to practice** — "How does this apply to our actual work?" Mandatory every lesson.
→ Run: `scripts/track_progress.py step <dir> <lesson> practice`

**⑤ Multimedia** (when tools are available) — Generate diagrams/audio/video when they aid understanding.

| Medium | When to use |
|--------|------------|
| **Image** | Architecture diagrams, flowcharts, system topology |
| **Audio** | Lesson summary recaps |
| **Video** | Complex dynamic processes only |

Rules: only generate if tool is available; prefer diagrams over decoration; always include text description alongside media.

**⑥ Verify** — 3 questions (max 5): 1 comprehension + 1 application + 1 analysis. Teacher grades and gives feedback.
- Pass (2/3 correct) → proceed to Phase 4
- Fail → re-teach weak parts from different angle, then 1-2 follow-up questions

**On pass, run:** `scripts/track_progress.py complete <dir> <lesson> --score <X> [--weak "..."]`

**Delivery rules:**
- Deliver ①②③④⑤ first, wait for student to digest, then ⑥
- If 3+ consecutive theory paragraphs → insert a question
- Split long messages for readability

### Phase 4: Report and Progress

After each lesson completion, in this order:

1. **Run:** `scripts/lesson_report.py <dir> <lesson> --strengths "..." --takeaway "..." [--weak "..."]`
2. **Run:** `scripts/show_progress.py <dir>`
3. If image generation is available: also generate a visual progress map image

### Phase 5: Resume

When the student returns after any break:

1. **Run:** `scripts/resume.py <dir>` — read output to determine state
2. Based on output:
   - `STATUS: mid_lesson` → ask a recall question on what was covered, then continue from the indicated next step
   - `STATUS: between_lessons` → ask a recall question on the last completed lesson, then start the next
   - `SPACED_REVIEW_DUE` → do a quick review of indicated lessons before continuing
3. If student recalls well → continue. If fuzzy → brief recap. If forgot → re-teach before moving forward.

### Phase 6: Review and Wrap-up

- **Spaced review**: handled automatically by `resume.py` (triggers at lessons 3, 6, midpoint)
- **Final report**: on course completion, generate a comprehensive summary — mastery level per lesson, overall weak points, next steps

## Constraints

### Teaching principles

1. **Mastery first**: do not advance if current lesson is not mastered
2. **Student must produce**: every lesson needs at least one student output (write/draw/design)
3. **Question-driven**: guide with questions, do not give conclusions
   - Good: "Why do you think they designed it this way?"
   - Bad: "They designed it this way because..."
4. **Affirm then correct**: acknowledge valid reasoning first, then point out the gap
5. **Match technical depth**: for technical students, go deep by default
6. **Technical analogies**: use architecture/design-pattern analogies, not daily life

### Quality checklist (before delivering each lesson)

- [ ] Has comparison analysis (why A not B)
- [ ] Has concrete code/examples
- [ ] Has "how does this apply to us"
- [ ] Has 3 verification questions with grading criteria
- [ ] Generated visual diagram if image tool available
- [ ] Generated audio recap if TTS available
- [ ] Saved lesson content to file

### Edge cases

| Situation | Action |
|-----------|--------|
| "too easy" | Skip verification, advance |
| "don't understand" | Re-explain from different angle |
| Goes off-topic | Acknowledge briefly, redirect |
| Returns after break | Run `scripts/resume.py`, follow Phase 5 |
| Teacher lacks depth | Pause, research, then resume |
