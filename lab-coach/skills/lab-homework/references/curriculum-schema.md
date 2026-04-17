# Curriculum Manifest Schema (v2)

The `lab-homework` skill fetches a **curriculum manifest** from:

    https://agency-lab.glebkalinin.com/api/curriculum/<cohort>.json

No auth. Cache-Control: `public, max-age=60`. The endpoint reads MDX files from agency-docs at request time, so editing a meeting's MDX is immediately reflected.

## Cohort slug

Use the public route slug, not the internal content-directory name:

| Public slug (use this) | Content dir (internal) |
|---|---|
| `claude-code-lab-02` | `claude-code-internal-02` |
| `claude-code-lab-03` | `claude-code-internal-03` |
| `claude-code-lab-04` | `claude-code-internal-04` |
| `claude-code-lab-05` | `claude-code-internal-05` |

## Response shape

```json
{
  "schema_version": 2,
  "cohort": "claude-code-lab-03",
  "title": "Claude Code Lab 03",
  "language": "ru",
  "updated_at": "2026-04-18",
  "meetings": [
    {
      "number": "01",
      "title": "Встреча 01: Знакомство и введение",
      "description": "Введение в Claude Code Lab...",
      "date": "2026-03-03",
      "slides_url": "https://agency-lab.glebkalinin.com/claude-code-lab-03/20260303-presentation.html",
      "video_url": "https://www.youtube.com/watch?v=0y7AGq9foBM",
      "summary_md": "## Claude Code Lab 03 — Meeting 01...\n\n### О ведущем\n..."
    }
  ]
}
```

## Field reference

| Field | Type | Notes |
|---|---|---|
| `schema_version` | integer | Must be `2`. Skill refuses unknown versions. |
| `cohort` | string | Echoes the URL slug. |
| `title` | string | From the cohort's `meta.json`. |
| `language` | `"ru"` or `"en"` | Hint for homework output language. |
| `updated_at` | ISO date | Newest mtime across the cohort's meeting MDX files. |
| `meetings[]` | array | Sorted ascending by `number`. |
| `meetings[].number` | string | Zero-padded from filename (`"01"`, `"03"`, `"11"`). |
| `meetings[].title` | string | From frontmatter `title`. |
| `meetings[].description` | string \| null | From frontmatter `description`. |
| `meetings[].date` | ISO date \| null | Labelled `Дата:` / `Date:` in MDX body, else first ISO date in the opening 500 chars. |
| `meetings[].slides_url` | URL \| null | Markdown link under `/<cohort>/...presentation.html`, made absolute. |
| `meetings[].video_url` | URL \| null | First YouTube link (watch, embed, or youtu.be). |
| `meetings[].summary_md` | string | MDX body with frontmatter / iframes / JSX components stripped. |

## Errors

| Status | Body | When |
|---|---|---|
| 200 | manifest JSON | cohort exists |
| 404 | `{ "error": "cohort_not_found", "known": [...] }` | unknown cohort |
| 5xx | Next.js default | implementation bug — retry or report |

## Minimal local test fixture

Drop this at `<vault>/.cache/curriculum-test.json` and point the skill at cohort `test` (requires a local override; not recommended for normal use):

```json
{
  "schema_version": 2,
  "cohort": "test",
  "title": "Test cohort",
  "language": "en",
  "updated_at": "2026-04-18",
  "meetings": [
    {
      "number": "01",
      "title": "Test meeting",
      "description": null,
      "date": "2026-04-18",
      "slides_url": null,
      "video_url": null,
      "summary_md": "Test summary."
    }
  ]
}
```
