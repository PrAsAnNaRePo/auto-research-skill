# auto-research-builder

a skill that helps you set up an auto-research environment for your specific use-case.

the idea is simple, an agent modifying code, running experiments, checking if a metric improved, and repeating. the pattern works for any optimization problem. it works for parsing pipelines, content optimization, prompt refinement, anything where "better" can be defined.

this skill scaffolds that environment for you. it asks you questions one at a time and generates the four files every auto-research loop needs.

---

## what it generates

- `pull_data.py` — fetches or preps the data the agent works on
- `program.md` — agent instructions: the loop, what to modify, what not to touch, how to log
- `skill.md` — domain knowledge so the agent reasons well (researched if needed)
- `eval.txt / eval.py / eval.md` — validation, stopping conditions, the metric or review protocol

---

## how to use it

you need [Claude Code](https://claude.ai/code) installed. (or codex, opencode etc.)

**1. add the skill to your project**

copy `BUILDER.md` into your project's `.claude/` folder or point Claude Code to it.

**2. invoke the skill**

```
/auto-research-builder
```

**3. answer the questions**

the builder will ask you one question at a time about task, data, constraints, validation. it won't dump everything at once.

**4. confirm before it writes**

before generating any files, it'll show you a summary of what it's about to create. you can correct anything there.

**5. run your environment**

once the files are generated:

```bash
# pull your data first
python pull_data.py

# then hand the agent the files
# program.md, skill.md, eval.*, and your data
# and let it run
```

---

## two types of validation

**metric-based** — the agent computes a score after each run and decides keep/discard on its own. fully automated. good for things like parse accuracy, loss values, test pass rates.

**open-ended** — the agent produces output each iteration, you review it, it refines. good for things like tone, style, content quality — anything that needs human judgment.

the builder will figure out which applies and design the eval accordingly.

---

## one thing to watch out for

if your validation is weak, the agent will game it. score goes up, actual output doesn't improve. this is reward hacking and it's common.

the builder has a section on this — it'll push you to tighten your eval before generating files. don't skip that part.

---

## feedback

if you try it, would love to know what use-case you built it for and where it broke.
