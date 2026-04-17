---
name: lab-homework
description: Use when the user wants a practice assignment between Claude Code Lab meetings — a concrete task tied to a specific lab meeting, that combines their personal goals with what was actually covered in the presentation. Triggers on "give me homework", "generate a practice task", "what should I practice", "homework for meeting 3", "задание на неделю". Fetches the current cohort's curriculum from agency-lab.glebkalinin.com, uses the meeting summary as the presentation record, reads the lab vault for personal context, and writes the assignment as a new markdown file.
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

- Cohort slug stored in `<vault>/.config.md` under `cohort:`. On first run, ask the user via `AskUserQuestion` — options: `claude-code-lab-04` (**default**), `claude-code-lab-03`, `claude-code-lab-05`, `claude-code-lab-02`.
- **Cache:** write the response to `<vault>/.cache/curriculum-<cohort>.json` with an added top-level `_fetched_at` ISO timestamp. Reuse the cache if younger than 24h.
- **Refresh triggers:** user phrases "refresh curriculum", "update curriculum", "pull latest", "обнови программу", "обнови курс" force a re-fetch. If the cache is older than 7 days, offer a refresh before continuing.
- **Schema:** `schema_version: 2`. See `references/curriculum-schema.md`. If `schema_version` is missing or unknown, refuse and tell the user to update the skill.
- **Failure:** on 404 or network error, tell the user, suggest `refresh curriculum`, and offer to accept the meeting topic in one sentence as manual input. Do not fabricate content.

### 2. Meeting content

Each meeting in the manifest has a `summary_md` field — the already-synthesized record of what was covered. This is the skill's source of truth for meeting content. **Do not** fetch a separate transcript. Do not invent content.

### 3. Personal vault

Read from the vault (see `lab-context` skill):
- `goals.md` — what the user wants
- `tools-learned.md` — what's already familiar (recent entries matter more)
- `projects.md` — what they're working on (best landing pad for tasks)
- `profile.md` — level, primary language
- `homework/` — existing assignments (to avoid duplicates)

If the vault is missing or empty, point to `lab-context` first. Don't invent personal context.

## Workflow

1. **Resolve cohort** — read `<vault>/.config.md` for `cohort:`. If missing, ask via `AskUserQuestion` (default `claude-code-lab-04`) and save it.
2. **Fetch or reuse the curriculum manifest** (see caching rules above).
3. **Ask which meeting** via `AskUserQuestion`, populated from `manifest.meetings[]` (number + title). Single-select.
4. **Ask parameters** via a second `AskUserQuestion`:
   - **"How much time do you have?"** — 30 min / 1 hour / evening / weekend
   - **"Where should this land?"** — active project / playground / either
5. **Read the vault** (profile, goals, tools-learned, projects, existing homework).
6. **Generate the assignment** using the pedagogical rubric below, grounded in the selected meeting's `summary_md` so the task matches topics the user actually saw.
7. **Write the file** to `<vault>/homework/YYYY-MM-DD-meeting-<NN>-<slug>.md`.
8. **Summarise briefly** in the chat: one sentence about the task and the reflection question. Don't dump the whole file.

## Pedagogical rubric

A good Claude Code Lab homework task is structured like this:

- **An artifact is mandatory.** The participant must end up with a file / folder / script / commit they can show at the next meeting. No artifact, no homework. "Read the docs" ≠ a task.
- **Open-ended, not a test.** There's no single correct solution. Frame tasks as "build X that does Y" — the *how* is the participant's call. This mirrors real work with an agent far better than checklist exercises.
- **80% familiar + 20% new.** Most of the task leans on things already in `tools-learned.md`. Exactly one new element: an unfamiliar combination, a new MCP, a subagent they haven't tried, an unusual mode (plan/think). Not two, not three — that tips over into frustration.
- **Ground it in the summary.** Pull the specific topic, demo, or tool from `summary_md` for that meeting. The task should feel like "continue what we did on screen" — not a generic MCP tutorial.
- **Land it in a real project when possible.** Check `projects.md`. If an active project fits — ship the task there, artifact = commit or PR. Playground is the fallback when nothing fits.
- **Provocation, not instruction.** The phrasing should force the participant to *decide*, not *execute*. Good: "figure out how the agent could verify its own work in your project". Bad: "create a hook that runs pytest after every Edit".
- **One reflection question.** A single question at the end that can't be googled — only answered from your own experience. This *is* the success check: if the participant can answer it in depth, the insight landed.
- **Smaller than feels right.** Participants overestimate what they'll do between meetings. A 45-minute task that gets finished beats a 3-hour task that gets abandoned halfway.

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
related_project: <[[projects#name]] or playground>
status: new
---

# <Task name>

## Why this matters for you
<1-2 sentences linking the task to the user's goal from goals.md>

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

- **One task per call.** Never a bundle of five. If the user wants more — suggest picking one.
- **Never duplicate past homework.** Check `homework/` first. If a similar task exists, either continue it (part 2) or pick a different angle.
- **Concrete over abstract.** Not "practice MCP" but "write an MCP server with one tool `get_current_weather`, wire it into Claude Code, call it three times".
- **Never create files outside the vault.** If the task needs a scratch folder, create `<vault>/homework/<slug>/` — not `$HOME`, not the root of the user's active git project.
- **If the manifest fetch fails**, tell the user and ask them to either provide the meeting topic in one sentence or run `refresh curriculum`. Don't fabricate.
- **If the cache is stale** (older than 7 days), warn the user and offer to refresh before generating.
