---
name: auto-research-builder
description: |
  Interactive scaffolder that builds a complete auto-research environment for the user's specific task.
  Use this skill when the user wants to set up an iterative agent loop for optimization, experimentation,
  or research on any domain — code, data, content, parsing, or anything else.
allowed-tools:
  - Read
  - Write
  - Edit
  - WebSearch
  - WebFetch
  - Bash
---

# auto-research-builder

## What is auto-research?

Auto-research is a sandbox environment where an AI agent iterates autonomously to improve something.
It combines three things:

- **Data** — the raw material the agent works on (files, a database, a dataset, scraped content, etc.)
- **System** — the instructions that tell the agent what to do, what to modify, and how to reason
- **Validation** — the criteria that decide whether an iteration made things better or worse

The agent loop looks like this:

```
read current state
→ form a hypothesis
→ make a change
→ run / execute
→ measure the result
→ keep if better, discard if not
→ log the attempt
→ repeat
```

This pattern works across domains. The loop can optimize a data parser, a content rephraser, a prompt, a ranking algorithm,
or anything else where "better" can be defined.

---

## What does auto-research-builder do?

Auto-research-builder is a companion that interviews the user and generates the four files every
auto-research environment needs:

- `pull_data.py` — fetches or prepares the data the agent will work on, if the data is necessarily pulled from source.
- `program.md` — agent instructions: the loop, what to modify, what not to touch, how to log
- `skill.md` — domain knowledge the agent needs to reason well (researched if necessary)
- `eval.txt / eval.py / eval.md` — validation rules, stopping conditions, the metric or review protocol

The builder asks one question at a time and one chat at a time. It does not dump all questions at once.
After all information is gathered, it confirms the plan with the user before writing any file.

---

## Builder Flow

### Phase 1 — Task

Ask the user, one at a time:

1. What is the task? What is the agent trying to improve or produce?
2. What does the agent modify on each iteration? (a script, a file, a prompt, a config, etc.)

Wait for the answer to each before asking the next.

---

### Phase 2 — Data

Ask the user, one at a time:

1. What is the data the agent works on? Describe the type and format.
2. Where does the data come from — local files, a database, an API, the web, or something else?
3. Does the data need to be fetched/pulled before each run, or is it static and already local?

If the data needs pulling: plan a `pull_data.py` that fetches and structures it.
If data is already local: document its shape and location instead.

---

### Phase 3 — Specs and Constraints

Ask the user, one at a time:

1. What is the agent allowed to modify? Be specific — which files, which sections, which fields?
2. What is the agent NOT allowed to touch? (e.g. raw data, eval rules, pull scripts)
3. Are there any other constraints — runtime limits, output format requirements, tool restrictions?

---

### Phase 4 — Validation

Ask the user:

1. How do you know if an iteration made things better?

There are two types of validation — help the user identify which applies:

**Metric-based** — a number that can be computed automatically.
Examples: loss value, parse accuracy score, number of passing test cases, BLEU score.
The agent computes the metric after each run and decides keep/discard autonomously.

**Open-ended / manual** — the output requires human judgment.
Examples: does this rephrasing sound like me? is this summary accurate? does this feel right?
The agent produces output and logs it each iteration. The human reviews and decides keep/discard.
The agent still runs the loop and logs every attempt — it just flags the result for review instead of
auto-deciding.

If metric-based: ask what the metric is, how it's computed, and what the threshold/goal is.
If open-ended: ask what the agent should output per iteration so the human can review it easily.

Also ask: what are the stopping conditions? (e.g. metric hits a target, N iterations completed,
human is satisfied, all test cases pass)

---

### Phase 5 — Domain Knowledge

Assess whether the task requires specialized domain knowledge for the agent to reason well.

Ask the user:
1. Does this task require domain expertise that isn't obvious? (e.g. engineering drawing conventions,
   legal clause patterns, a specific coding standard, a niche content style)

If yes: use WebSearch and WebFetch to research the domain. Synthesize findings into `skill.md`.
If no: `skill.md` will be a lightweight file with just the task-specific rules the user provides.

---

### Phase 6 — Plan and Confirm

Before writing any files, present the user with a summary:

- What `pull_data.py` will do
- What `program.md` will instruct the agent to do (the loop, what to modify, logging format)
- What `skill.md` will contain
- What the eval file will define (metric or review protocol, stopping conditions)

Ask: "Does this look right? Anything to change before I generate the files?"

Only proceed after the user confirms.

---

### Phase 7 — Generate Files

Write all four files. Follow the structure below.

---

## File Structures

### `pull_data.py`

- Fetches or prepares the data the agent will work on
- Must be runnable standalone: `uv run pull_data.py` or `python pull_data.py`
- Outputs data to a predictable location (e.g. `data/`, `data.json`, `input/`)
- Must NOT be modified by the agent during the loop
- Include a brief docstring explaining what it fetches and what it produces

### `program.md`

Must contain:

1. **Objective** — one paragraph on what the agent is doing and why
2. **What you may modify** — explicit list of files/sections the agent can change
3. **What you must NOT modify** — explicit list (at minimum: `pull_data.py`, `eval.*`, `program.md`)
4. **The loop** — step by step:
   - Read current state of the modifiable file(s)
   - Form a hypothesis about what to improve
   - Make the change
   - Run the experiment / execute
   - Measure the result (metric or produce output for review)
   - Keep if better / discard if worse (or flag for human review if open-ended)
   - Log to `results.tsv`: timestamp, result, what changed, notes
   - Repeat — never stop unless stopping condition is met
5. **Logging format** — exact `results.tsv` columns
6. **Stopping conditions** — when the loop ends

### `skill.md`

- Domain knowledge the agent needs to reason well about the task
- Structured as reference material: rules, conventions, patterns, failure modes
- Should answer: "what does an expert know about this domain that a generalist wouldn't?"
- If researched via web: cite sources at the bottom
- If user-provided: organize and structure it clearly

### `eval.txt` / `eval.py` / `eval.md`

Choose the format based on the validation type:

- `eval.txt` — for scenario-based validation: list of named scenarios with expected outputs
- `eval.py` — for metric-based validation: a script that computes the score
- `eval.md` — for open-ended / manual validation: a review protocol the human follows

Must always include:
- The validation criteria (what "better" means)
- The stopping conditions
- For metric-based: the target threshold
- For open-ended: what to look for when reviewing, what questions to ask

---

## Logging

Every auto-research environment must have a `results.tsv` for tracking iteration history.
The agent writes one row per iteration.

Minimum columns:
```
timestamp   result   change_summary   notes
```

Add domain-specific columns as needed (e.g. `scenarios_passed`, `quality_score`, `val_bpb`).

---

## Rules for the Builder

- Ask one question at a time. Never list all questions in one message.
- Never generate files until Phase 6 confirmation is complete.
- If the domain needs research, do it before Phase 6 — bring the knowledge into the plan.
- If the user's validation is open-ended, do not force a metric. Design the loop around human review.
- Keep `program.md` honest: the loop instructions must match what is actually possible in the environment.
- The agent's loop in `program.md` should always end with "NEVER STOP unless stopping condition is met.
  NEVER ask for input mid-loop."

---

## Examples

---

### Example 1 — ML Training Optimization (Karpathy-style)

**Use-case:** Agent autonomously optimizes a GPT training script overnight on a single GPU.

---

**`pull_data.py`**
```python
# Downloads and tokenizes the training dataset (e.g. TinyShakespeare).
# Outputs: data/train.bin, data/val.bin
# Run once before starting the agent loop.
# DO NOT modify this file.
```

---

**`program.md`**
```
# GPT Training Optimizer

## Objective
Improve the validation bits-per-byte (val_bpb) of a small GPT model trained on TinyShakespeare.
Lower val_bpb = better. Each experiment runs for exactly 5 minutes regardless of hardware,
making results directly comparable across iterations.

## What you may modify
- train.py: model architecture, optimizer settings, hyperparameters, training loop logic

## What you must NOT modify
- prepare.py, data/, program.md, eval.txt

## The loop
1. Read the current train.py and results.tsv
2. Form a hypothesis: what change might reduce val_bpb?
3. Modify train.py
4. Run: python train.py — wait exactly 5 minutes, then read the output val_bpb
5. If val_bpb improved: keep the change
   If val_bpb did not improve: revert train.py to previous state
6. Log to results.tsv
7. Repeat. NEVER STOP unless stopping condition is met. NEVER ask for input.

## Logging
timestamp | val_bpb | change_summary | hypothesis | kept

## Stopping conditions
- val_bpb < 0.95 (target threshold), OR
- 100 iterations completed
```

---

**`skill.md`**
```
# GPT Training Domain Knowledge

- val_bpb (validation bits per byte): measures how well the model predicts held-out text.
  Lower is better. Typical range for small models on TinyShakespeare: 1.0–1.5.

- Safe things to try: learning rate schedule, batch size, weight decay, dropout,
  attention head count, embedding dimension, activation function.

- Risky things: changing the tokenizer, modifying data loading, altering the 5-minute
  training cap — these break comparability across runs.

- A change that reduces val_bpb by >0.01 is meaningful. Less than 0.005 is noise.
```

---

**`eval.txt`**
```
Target: val_bpb < 0.95
Stopping condition: target reached OR 100 iterations completed
Baseline: record val_bpb from unmodified train.py as iteration 0
A run is valid only if it completed the full 5-minute window without error.
```

---

### Example 2 — Engineering Drawing Table Parser

**Use-case:** Agent iterates on `postprocess.py` to extract structured data (title blocks, BOMs,
revision tables) from raw HTML OCR output of engineering drawings.

---

**`pull_data.py`**
```python
# Downloads raw engineering drawing data from the Adeos pipeline (MongoDB + Azure Blob).
# Outputs: data/*.json — one file per drawing, containing raw HTML table strings.
# Run once. DO NOT modify.
```

---

**`program.md`**
```
# Engineering Drawing Table Parser

## Objective
Build postprocess.py so that structured output in structured/*.json is correct enough
to pass all 14 scenarios in eval.txt using plain Python — no LLM, no fuzzy logic.

## What you may modify
- postprocess.py and any helper modules you create and import from it

## What you must NOT modify
- data/, eval.txt, program.md, skill.md, pull_data.py

## The loop
1. Read postprocess.py — understand what is currently broken
2. Form a hypothesis about the root cause (not a symptom)
3. Modify postprocess.py
4. Run: uv run postprocess.py → output written to structured/
5. Compute: scenarios_passed, quality_score, unknown_count (see eval.txt)
6. If improved: keep. If not: revert.
7. Log to results.tsv
8. NEVER STOP until all stopping conditions are met. NEVER ask for input.

## Stopping conditions
- scenarios_passed = 14/14, AND
- unknown_count ≤ 3, AND
- quality_score is no longer improving across 3 consecutive iterations

## Logging
timestamp | scenarios_passed | quality_score | unknown_count | change_summary | notes
```

---

**`skill.md`**
```
# Engineering Drawing Domain Knowledge

## Title block
- Always in the bottom-right corner of the page. Strongest classification signal.
- Key fields: dwg_no, title, material, scale, drawn_by, sheet
- Common OCR artifacts: label and value in same cell ("Drawn: DTL"), rowspan duplicates

## BOM
- Right side of page, above title block
- Columns: ITEM NO., DESCRIPTION, QTY., PART NO., MATERIAL
- Item numbers are sequential integers starting at 1

## Revision table
- Top-right corner, above title block
- Columns: REV, ZONE, DESCRIPTION, DATE, APPROVED

## Failure modes
- A value that looks like a label = parsing failure (material: "DWG NO.")
- A growing alias dictionary = wrong approach; fix root parsing logic instead
```

---

**`eval.txt`**
```
14 scenarios across: title_block field extraction, BOM row extraction, revision parsing.

Scenario 1: dwg_no extracted correctly across all pages of 31.001-A-096.pdf
Scenario 2: title extracted correctly, not a label word
...

Stopping: scenarios_passed=14, unknown_count≤3, quality_score stable for 3 runs.

quality_score = sum of correctly extracted fields across all files (no ceiling).
unknown_count = total annotations still classified as "unknown" across all files.
```

---

### Example 3 — Content Tone Optimizer

**Use-case:** Agent iterates on a `skill.md` that teaches Claude to rephrase content in the user's
personal tone and slang. Validation is manual — the user reads each output and decides if it sounds
like them.

---

**`pull_data.py`**
```python
# No external data source. Reads sample content from input/samples.txt —
# a set of 10 pieces of content the user wants rephrased (paragraphs, tweets, emails).
# The user writes these manually once. DO NOT modify this file.
```

---

**`program.md`**
```
# Content Tone Optimizer

## Objective
Refine skill.md so that when Claude uses it to rephrase content in input/samples.txt,
the output consistently sounds like the user — their vocabulary, rhythm, slang, and energy.
Each iteration produces a new output/rephrased-{n}.md for human review.

## What you may modify
- skill.md: tone rules, vocabulary lists, example rewrites, anti-patterns

## What you must NOT modify
- input/samples.txt, program.md, eval.md, pull_data.py

## The loop
1. Read skill.md and the last review notes in results.tsv
2. Form a hypothesis: what aspect of the tone is still off?
   (e.g. too formal, wrong slang, wrong sentence length, wrong energy)
3. Update skill.md with the refined tone rules
4. Apply skill.md to rephrase all samples in input/samples.txt
5. Write output to output/rephrased-{n}.md (n = iteration number)
6. Log to results.tsv and STOP — wait for human review
7. Human reviews output/rephrased-{n}.md, writes feedback into results.tsv, signals to continue
8. Resume from step 1 with new feedback

## Stopping conditions
- Human marks 3 consecutive iterations as "sounds right"

## Logging
timestamp | iteration | hypothesis | human_verdict | feedback_notes
```

---

**`skill.md`** *(starts sparse, agent refines it each iteration)*
```
# Tone Guide — [User Name]

## Voice
- [Agent fills this in and refines it over iterations]

## Vocabulary
- Preferred words: ...
- Words to avoid: ...

## Slang and phrases
- Common expressions: ...

## Rhythm
- Sentence length: ...
- Energy level: ...

## Example rewrites
- Original: "..."
  Rephrased: "..."
```

---

**`eval.md`**
```
# Review Protocol — Content Tone Optimizer

After each iteration, read output/rephrased-{n}.md and answer these questions:

1. Does it sound like you wrote it? (yes / almost / no)
2. What feels off? (too formal, wrong slang, too long, wrong energy, other)
3. Pick the sentence that feels most wrong. What would you have written instead?
4. Pick the sentence that feels most right.

Write your answers into results.tsv under feedback_notes.
Mark human_verdict as: "sounds right" | "getting closer" | "still off"

Stopping condition: 3 consecutive "sounds right" verdicts.
```

---

## Guidelines for a Good Auto-Research Environment

Share these with the user during the interview if relevant, and apply them when designing the files.

### Task clarity
The task must be specific. The narrower and more concrete the objective, the more predictable and
useful the agent's iterations will be. A vague task ("make the output better") produces aimless
iterations. A sharp task ("reduce val_bpb below 0.95 by modifying only the optimizer settings")
produces focused, comparable experiments.

If the user's task feels open-ended, push them to constrain it before generating files:
- What exactly is being modified each iteration?
- What exactly is being measured?
- What does "done" look like?

### Reward hacking
This is the most common failure mode in iterative agent loops. The agent finds a way to maximize
the metric without actually solving the problem — it games the eval rather than improving the output.

Examples:
- A parser that hardcodes known field values from the test set instead of parsing them structurally
- A tone optimizer that copies phrases verbatim from the user's samples instead of generalizing the style
- A training script that overfits to validation data by leaking it into training

How to prevent it when designing `eval`:
- Validation scenarios should cover cases the agent has NOT seen during iteration
- For metric-based eval: include a qualitative sanity check alongside the score
- For open-ended eval: the human reviewer should look for genuine generalization, not pattern matching
- In `program.md`, add an explicit rule: "Do not hardcode values. Solutions must generalize to unseen inputs."

### Validation design
Weak validation produces false progress. Before finalizing the eval file, ask:
- Does passing this eval mean the actual problem is solved, or just this specific test?
- Could the agent score well on this eval while producing output the user would reject?
- Is the stopping condition reachable without reward hacking?

If the answer to any of these is uncertain, tighten the eval before the agent starts iterating.
