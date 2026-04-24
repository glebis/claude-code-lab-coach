---
name: lab-homework
description: Use when the user wants a practice assignment between Claude Code Lab meetings — a concrete task tied to a specific lab meeting, that combines their personal goals with what was actually covered in the presentation. Triggers on "give me homework", "generate a practice task", "what should I practice", "homework for meeting 3", "задание на неделю". Fetches the current cohort's curriculum from agency-lab.glebkalinin.com, uses the meeting summary as the presentation record, reads the lab vault (plus optional sandbox) for personal context, and writes the assignment as a new markdown file. Also updates progress.md and optionally installs recommended agents.
---

# Lab Homework

Generates a personal practice assignment tied to a specific Claude Code Lab meeting.

## Language

**Default: English.** Switch to Russian if the user wrote to you in Russian. The generated assignment file is written in the same language as the user's request.

## Inputs

### 1. Curriculum manifest

Fetched from the public API:

```
https://agency-lab.glebkalinin.com/api/curriculum/<cohort>.json
```

- Cohort slug stored in `<vault>/.config.md` under `cohort:`. On first run, ask the user via `AskUserQuestion` — options: `claude-code-lab-05` (**default**), `claude-code-lab-04`, `claude-code-lab-03`, `claude-code-lab-02`.
- **Cache:** write the response to `<vault>/.cache/curriculum-<cohort>.json` with an added top-level `_fetched_at` ISO timestamp. Reuse the cache if younger than 24h.
- **Refresh triggers:** user phrases "refresh curriculum", "update curriculum", "pull latest", "обнови программу", "обнови курс" force a re-fetch. If the cache is older than 7 days, offer a refresh before continuing.
- **Schema:** `schema_version: 2`. See `references/curriculum-schema.md`. If `schema_version` is missing or unknown, refuse and tell the user to update the skill.
- **Failure:** on 404 or network error, tell the user, suggest `refresh curriculum`, and offer to accept the meeting topic in one sentence as manual input. Do not fabricate content.

### 2. Meeting content

Each meeting in the manifest has two fields the skill uses:

- `summary_md` — the already-synthesized record of what was covered in that meeting.
- `has_content: boolean` — `true` when `summary_md` is real content, `false` when it's a pre-event stub.

**If `has_content` is true:** use `summary_md` as the source of truth. Do not fetch a separate transcript. Do not invent content.

**If `has_content` is false** (meeting hasn't happened yet): see "Handling future meetings" below.

**Optional per-meeting endpoint** (for direct drill-in without re-fetching the cohort list):

```
https://agency-lab.glebkalinin.com/api/curriculum/<cohort>/<meeting>.json
```

### 3. Personal vault

Read from the vault (see `lab-context` skill):
- `goals.md` — what the user wants
- `tools-learned.md` — what's already familiar (recent entries matter more)
- `projects.md` — what they're working on (best landing pad for tasks)
- `profile.md` — level, primary language
- `homework/` — existing assignments (to avoid duplicates)

If the vault is missing or empty, point to `lab-context` first. Don't invent personal context.

### 4. Sandbox context (optional)

Read `<vault>/sandbox/` if it exists. These files provide a simulated work environment (clients, customers, stakeholders, projects) that grounds homework in realistic scenarios.

- **Real projects beat sandbox.** If `projects.md` has an active project that fits the meeting topic, use it as the task landing pad.
- If no real project fits (or `projects.md` is empty), use sandbox entities to make the task concrete — reference specific clients, customers, or team members.
- **Never mix** sandbox and real-project context in the same task.
- Starter sandbox templates are bundled at `<skill-dir>/sandboxes/{company,consultant,freelancer,solopreneur}/`. If the user has no `<vault>/sandbox/` and asks for one, offer to copy one of these.

## Workflow

1. **Resolve cohort** — read `<vault>/.config.md` for `cohort:`. If missing, ask via `AskUserQuestion` (default `claude-code-lab-05`) and save it.
2. **Fetch or reuse the curriculum manifest** (see caching rules above).
3. **Ask which meeting** via `AskUserQuestion`, populated from `manifest.meetings[]` (number + title). Mark stubs visibly — append " (upcoming)" or " (not yet covered)" to titles where `has_content` is false. Single-select.
4. **Ask parameters** via a second `AskUserQuestion`:
   - **"How much time do you have?"** — 30 min / 1 hour / evening / weekend
   - **"Where should this land?"** — active project / playground / either
5. **Read vault + optional sandbox** (profile, goals, tools-learned, projects, existing homework, sandbox/).
6. **Generate the assignment** using the pedagogical rubric below, grounded in the selected meeting's `summary_md`.
   - If `has_content: false`, follow "Handling future meetings" below instead of improvising from the stub.
7. **Offer recommended agents** (optional step, silent when not applicable).
   - If the manifest has a top-level `agents[]` array AND the selected meeting has `recommended_agents`: look each up, tell the user "**[Agent Name]** could help — install to `.claude/agents/`?", confirm before installing.
   - Otherwise: check the bundled `<skill-dir>/agents/` directory. If bundled agents are topic-relevant (e.g. `prompt-coach` for prompting meetings), offer to install from local copies. Confirm first.
   - Write confirmed agents to `.claude/agents/<id>.md`. Skip any that already exist.
   - Never install automatically. If the user declines, move on and don't re-offer.
8. **Write the homework file** to `<vault>/homework/YYYY-MM-DD-meeting-<NN>-<slug>.md`.
9. **Update `<vault>/progress.md`:**
   - If the file doesn't exist, create it from `<skill-dir>/references/progress-template.md` (substitute `{{date}}` with today's ISO date).
   - Find the section for this meeting (e.g. `## Meeting 03 — Prompt Engineering`). If missing, create it.
   - Append a line: `- [ ] Homework: [[homework/YYYY-MM-DD-meeting-NN-slug.md]]`.
   - Update the `updated:` date in frontmatter and bump `homework_completed` by 0 (increment only when the user marks it done).
   - Use **Edit**, never overwrite the file wholesale.
10. **Summarise briefly** in the chat: one sentence about the task + the reflection question. Don't dump the whole file.

## Handling future meetings

When the selected meeting has `has_content: false`, the manifest's `summary_md` is a stub and can't ground the task. Options, in order of preference:

1. **Use the title + description.** The frontmatter usually encodes intent (e.g. `Meeting 05: Subagents & Research (theory + demos)`). If that's enough to design a *preparatory* task, tell the user: "This meeting hasn't happened yet. I'll design a prep task based on the planned topic (<title>). Confirm before I write."
2. **Ask the user to describe the planned topic in one sentence** if the title is too vague.
3. **Redirect to a completed meeting** if the participant is unsure. Show only `has_content: true` meetings and ask again.

Never silently generate a task using fabricated content.

## Pedagogical rubric

- **An artifact is mandatory.** The participant must end up with a file / folder / script / commit they can show at the next meeting. "Read the docs" ≠ a task.
- **Open-ended, not a test.** Frame as "build X that does Y" — the *how* is the participant's call.
- **80% familiar + 20% new.** Most of the task leans on `tools-learned.md`. Exactly one new element — not two.
- **Ground it in the summary.** Pull specific topics / demos from `summary_md`. The task should feel like "continue what we did on screen".
- **Land it in a real project when possible.** Real project > sandbox > playground.
- **Provocation, not instruction.** "Figure out how the agent could verify its own work" > "create a hook that runs pytest after every Edit".
- **One reflection question** that can't be googled — only answered from experience.
- **Smaller than feels right.** 45 minutes finished > 3 hours abandoned.

## Output format

Save to `<vault>/homework/YYYY-MM-DD-meeting-<NN>-<slug>.md`:

````markdown
---
date: <YYYY-MM-DD>
meeting: <NN>
meeting_title: <title from manifest>
estimated_time: <30m | 1h | evening | weekend>
tools: [<tool1>, <tool2>]
related_goal: <quote from goals.md>
related_project: <[[projects#name]] or playground or sandbox/<entity>>
status: new
---

# <Task name>

## Why this matters for you
<1-2 sentences linking the task to the user's goal>

## Context from the meeting
<1-2 sentences summarising the relevant slice of `summary_md` — what the presenter showed>

## What you'll build
<clear description of the target — build, fix, or investigate>

## The new thing
<the single 20% — explicitly named>

## Done when
- [ ] <concrete observable result>
- [ ] <another one>

## Reflection
<one question worth asking yourself after you finish>
````

## Rules

- **One task per call.** If the user wants more — suggest picking one.
- **Never duplicate past homework.** Check `homework/` first. Continue an existing task as "part 2" or pick a different angle.
- **Concrete over abstract.** Not "practice MCP" but "write an MCP server with one tool `get_current_weather`, wire it into Claude Code, call it three times".
- **Never create files outside the vault.** For scratch dirs use `<vault>/homework/<slug>/`.
- **If the manifest fetch fails**, tell the user and offer `refresh curriculum` or accept a one-sentence topic. Don't fabricate.
- **If the cache is stale** (older than 7 days), warn and offer to refresh before generating.
