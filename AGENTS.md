# Agent rules for this repo

This repo tracks Lars' personal TODOs and assignments. `todo.md` is the
single source of truth for what active work exists.

## Task rules

- Every task appears in `todo.md` exactly once. Completed or cancelled tasks
  move to `archive.md` — they never stay in `todo.md`.
- Every line starts with a customer prefix: `[Customer]`. Use `[Internal]`
  for internal work. Never omit it.
- Every task gets a `T###` id, shown on the line right after the customer
  prefix: `[Customer] T###: Title`, optionally followed by
  ` → tasks/T###-slug.md`. This applies even to bare one-off tasks with no
  detail doc — every task in `todo.md`/`archive.md` is referable by id.
- Keep `todo.md`/`archive.md` lines terse: `[Customer]`, `T###:`,
  title, optional ` → tasks/T###-slug.md` link. No checkboxes, no other
  metadata on the line.
- Only create a `tasks/T###-slug.md` detail doc when there is real detail,
  a due date, or a status worth tracking. A quick one-off task can stay a
  bare line with no detail doc, but still gets a `T###` id.
- IDs are `T###`, derived from the highest existing id in use across both
  `todo.md`/`archive.md` lines and `tasks/` filenames. New ID = highest
  existing number + 1. Never renumber or reuse an ID.
- When a detail doc exists, its frontmatter `title` and `customer` must
  match the todo/archive line exactly. Update both together.
- `todo.md` is grouped by priority section: `## 🔴`, `## 🟡`, `## 🟢`,
  `## Someday`. Moving a task between priorities means moving its line to
  the other section (and updating `priority` in its detail doc, if any).
  🔴/🟡/🟢 are spoken of as high/medium/low priority — treat those words as
  synonyms for the dots in conversation and when writing frontmatter.
- `archive.md` is grouped by completion month: `## YYYY-MM`, using the
  `completed` date.

## Completing a task

1. Move the line from `todo.md` to the correct `## YYYY-MM` section of
   `archive.md` (create the section if needed).
2. If a detail doc exists, set `status: done` and `completed: YYYY-MM-DD`
   in its frontmatter, and append a closing note to `## Log`.

## Adding or changing a task

- When a task is added or its details change through conversation, proactively
  gather metadata in a single multi-question ask (the opencode `question`
  framework) rather than one question at a time or a wall of text. Cover
  whatever of the following are unset or plausibly changed: customer,
  priority, due date, effort, source/assigner, status.
- Always suggest a value for priority (and, when relevant, the other fields)
  based on what's known — urgency words, due dates, customer importance,
  how it relates to other open tasks — instead of asking blind. Make your
  suggestion the recommended option, but let the customer field remain a
  real question if it isn't obvious; don't guess that one.
- Don't invent a customer — always ask if it isn't obvious from context.
- If priority truly can't be judged even with a suggestion, default to `🟡`
  and say so.
- Only create a detail doc if the task warrants one (see above). Creating
  one is itself worth a suggestion, not just a hard rule — if the
  conversation surfaces due dates, effort, or status worth tracking, propose
  creating the doc rather than silently skipping it.

## Prioritizing / "what should I work on"

Act like a personal assistant here, not a sorting function. Give an opinion,
not just a ranked dump.

- Baseline ranking signal: overdue detail docs → 🔴 → due soon → effort fit.
- But weigh soft signals too: how long a task has been sitting, how it was
  phrased when added (urgency, frustration, a hard external deadline implied
  in conversation), whether it blocks other work, and general judgement
  about what matters — not just the hard metadata.
- State your reasoning, including any soft signals that pushed a task up or
  down. Never silently reorder `todo.md` without being asked — reordering
  sections is a suggestion to confirm, not an automatic action.
- Read detail docs for due dates and effort when they exist for tasks in the
  running, since the todo.md lines don't carry them.

## Git — overrides global rules

My global `~/.config/opencode/AGENTS.md` forbids the agent from staging,
committing, or pushing. **That rule does not apply to this repository.**

- Auto-commit and push, without asking, any change to `todo.md`,
  `archive.md`, `tasks/**`, or `notes/**`.
- Changes to `README.md` or this `AGENTS.md` file still require my explicit
  approval before committing — write them, then wait to be told to commit.
- Work directly on `main` and push to `origin`. No feature branches, no
  pull requests.
- One commit per logical change; do not batch unrelated edits together.
- Commit message format: `<verb> <what>`, e.g.
  - `Add [Acme] scheduler drift task`
  - `Complete [Internal] certificate renewal`
  - `Update T042 status to wip`
- Never force-push, never amend a pushed commit, never rewrite history.
- If a push fails, `pull --rebase` and retry once; if it fails again, stop
  and report rather than forcing anything.
