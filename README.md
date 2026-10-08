# Assistant

Personal repo for tracking my own TODOs and assignments, maintained together
with my coding agent. May grow to hold other personal-assistant material
over time (notes, journals, references).

## File map

- `TODO.md` — active tasks, grouped by priority.
- `ARCHIVE.md` — completed/cancelled tasks, grouped by completion month.
- `tasks/` — detail docs for tasks that need more than a one-liner.
- `notes/` — placeholder for future non-task content.
- `AGENTS.md` — rules for how the agent should read and update all of the above.

## Task line format

Every task is one line in `TODO.md` or `ARCHIVE.md`:

```
- [ ] [Customer] Title → tasks/T042-slug.md
```

- `[Customer]` is required. Internal work uses `[Internal]`.
- The `→ tasks/...` link is optional — only present when a detail doc exists.
- Nothing else goes on the line. All other metadata lives in the detail doc's frontmatter.

## Detail doc frontmatter (`tasks/T###-slug.md`)

| Field | Meaning |
|---|---|
| `id` | `T###`, matches the filename, never reused |
| `title` | matches the todo/archive line |
| `customer` | matches the `[Customer]` prefix |
| `priority` | 🔴 high · 🟡 medium · 🟢 low |
| `status` | `todo` `next` `wip` `blocked` `waiting` `done` `cancelled` |
| `due` | `YYYY-MM-DD` or blank |
| `effort` | `S` `M` `L` |
| `source` | who assigned it |
| `created` | `YYYY-MM-DD` |
| `completed` | `YYYY-MM-DD`, set on completion |

## Example prompts

- "Add a P1 task for Acme: fix batch scheduler drift"
- "Add a high priority task for Acme: fix batch scheduler drift"
- "What should I work on next?"
- "Mark T042 as done"
- "Show me everything for Acme"
- "I'm blocked on T042 waiting on Acme, update it"
