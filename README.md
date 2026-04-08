# claude-code-lab-coach

Claude Code plugin marketplace hosting **lab-coach** — a personal trainer for participants of Claude Code Lab.

## Install

```
/plugin marketplace add glebis/claude-code-lab-coach
/plugin install lab-coach@claude-code-lab-coach
```

## What's in `lab-coach`

- **`lab-context`** — maintains your participant profile (goals, tools learned, projects) in an Obsidian-compatible markdown vault.
- **`lab-homework`** — generates practice assignments aligned with the current lab curriculum, fetched from a remote manifest on agency-docs. Updates when the program changes.
- **`lab-review`** — audits a Claude Code session (current, most recent, or by ID) and returns coaching-style feedback on your prompts and agent interaction patterns.

Default language: English. Switches to Russian if you write in Russian.

See [`lab-coach/skills/lab-homework/references/curriculum-schema.md`](lab-coach/skills/lab-homework/references/curriculum-schema.md) for the curriculum manifest schema you publish on agency-docs.
