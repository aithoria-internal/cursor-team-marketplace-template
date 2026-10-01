---
name: enforce-templaterepo-baseline-security
description: >-
  Checks the current git repository against the secret and ignore baseline from
  aithoria-internal/templaterepo. Creates missing .gitignore, .cursorignore and
  .dockerignore files, proposes additions to existing ones for confirmation,
  and mentions a template rule only when that rule would help this project.
  Never changes a customer project on its own. Use once per day in a repository, and again the same day
  when a stack path appears that was absent at the last check, such as infra/,
  bootstrap/, prisma/, or tests/.
metadata:
  version: "2"
---

# Template baseline

The template repository is the source of the minimum secret boundary. Projects that were not created from it, and projects that bypass it, can still receive that boundary. Read the files live from GitHub. Do not rely on a copy stored in this skill.

Source: `aithoria-internal/templaterepo`, default branch, via `gh`.

This does not adopt the template. Application code, workflows, Terraform, Docker, package files, Prisma, `.env.example`, Cursor rules, and skills stay as they are unless the user explicitly chooses otherwise.

## When

Once per calendar day for a repository, and again the same day when a stack path appears that was absent at the last check. Skip the template repository itself. Skip a directory that is not a git repository. If `gh` cannot read the template, stop and say so. Do not invent file contents.

Stack paths that trigger a new check: `infra/`, `bootstrap/`, `prisma/`, `tests/`, `src/app/api/`, `src/components/`, and a `project.json` that contains `templateRepo`.

Record the check outside the repository, in the user agent store, as `template-baseline-stamps/<owner>--<repo>.txt`. Write today's date, which of those paths existed, the project level (see below), and the user's decisions (for example "ignore additions declined", "rule comparison declined"). If no agent store is available, remember the same note for the conversation. Run when there is no note, the date is not today, or a path exists now that the note does not list. After a run, update the note. Do not put the note in the project and do not commit it.

Do not ask again on the same day about something the user already declined, unless a new stack path appeared or the template now proposes different lines.

Before creating one of those stack paths, run the check again even if today already ran. Then update the note.

Do not commit the baseline. If the user asks to commit, follow `use-git-momentum` when that repository adopts it.

## Project level

Decide the level before touching anything. It controls every step below.

1. **Customer project.** The `origin` remote belongs to an owner other than `aithoria` or `aithoria-internal`, or the user has said it is a customer or third-party project. Change nothing and create nothing. Mention in one sentence that a comparison against the template baseline is available on request. Only if the user asks, show the comparisons described below as a read-only report. Apply nothing unless the user then names exactly what to apply.
2. **Own project, ignore file missing.** Create the missing file from the template and tell the user which files were created.
3. **Own project, ignore file present.** Show the comparison list and wait for confirmation before writing.

Levels 2 and 3 apply per file: a repository can have `.gitignore` but no `.dockerignore`.

If ownership is unclear (no remote, a fork, a personal account), ask the user once whether it is a customer project and treat it as level 1 until they answer.

## Ignore files

Read these paths from the template:

- `.gitignore`
- `.cursorignore`
- `.dockerignore`

The template has no `.terraformignore` and no `.cursorrules`. Terraform state, plans, and `.terraform/` are patterns inside `.gitignore` and `.cursorignore`. Do not create a file the template does not have.

### Missing file (level 2)

Create it with the template contents. Afterwards, list the created files in one sentence.

### Existing file (level 3)

Compare by trimmed pattern lines; ignore blank lines and comments. Present, per file:

- **Already present:** template patterns the file already contains.
- **Would be added:** template patterns the file lacks.
- **Notes:** any added line that starts with `!` (a negation appended at the end can re-include paths the project ignores on purpose), and any already tracked file the new patterns would match.

Lines that exist only in the project are not listed and are never touched.

Keep the list compact; a short table or grouped list per file is enough. Then ask whether to apply all additions, a selection, or none. Do not write before the user answers.

On confirmation, append only the confirmed lines at the end under one comment: `# template baseline (aithoria-internal/templaterepo)`. Leave every existing line, including order and comments, untouched. Omit the block when nothing is confirmed. If every template pattern is already present, say so and skip the question.

### Tracked files

A newly ignored file that is already tracked stays tracked. Report it, including `.env`, `*.pem`, and `*.tfstate`. Do not remove it from the index.

## Rules

Never add, replace, merge into, or delete a Cursor rule on your own. Projects may have their own conventions. Leave a legacy `.cursorrules` file untouched.

Do not ask whether the user wants a comparison of the template rules. Read each template rule in `.cursor/rules` first, at least its description, and decide whether it would help this repository. Use the table for the rules listed there. A rule the template adds later follows the same test: `alwaysApply` and no stack names can fit any repository; otherwise the paths in its description or `globs` must already exist. Read the rule body only when the description is not enough to decide.

Offer a comparison only for rules that are missing in this repository and would help it. Name each one with why it fits. Skip every rule that does not fit. Do not mention Terraform rules when `infra/` and `bootstrap/` are absent, and do not mention the other stack rules when their path is absent. If no missing rule would help, say nothing about rules and continue the task. Do not inventory rules that are already present, and do not list rules that only the project has, unless the user asks.

In a customer project, do not offer this unless the user brings rules up. If the user declines, record that and do not ask again the same day.

The user decides what to take over. Apply only the rules they name, as new files. Change an existing rule file only when the user explicitly asks for that specific change.

| Template rule | Fits this repository when |
| --- | --- |
| `behavior.mdc` | always; it does not name a stack |
| `project.mdc` | `project.json` contains `templateRepo` |
| `commands.mdc` | `project.json` contains `templateRepo` |
| `api-routes.mdc` | `src/app/api/` exists |
| `components.mdc` | `src/components/` exists |
| `prisma.mdc` | `prisma/` exists |
| `terraform.mdc` | `infra/` or `bootstrap/` exists |
| `testing.mdc` | `tests/` exists |

## Skills

List `.agents/skills`, `.claude/skills`, and `.codex/skills` in the template. The template currently has none. If it gains one, treat it like a rule: read it, and mention it only when it would help this project and the project does not already have the same skill name in any of those three directories or in `.cursor/skills`. Copy it only when the user asks. Write it once, into the same directory the template used. Do not overwrite an existing skill.

## After

Say in one or two sentences what changed: created files, confirmed additions, and adopted rules. If nothing changed because the baseline was already present or the user declined, say that briefly and continue the task the user asked for.

## Changelog

History only; the sections above define the behavior.

- **v2** (feedback: Erik). The always-apply rule only points at this skill. It no longer repeats create, append, or copy steps. Template rules are read before any question. A comparison is offered only for missing rules that would help this repository. Rules that do not fit are not mentioned.
- **v1** (feedback: Denis). Added project levels: customer projects are not changed, and a comparison runs only on request. Missing ignore files are created and reported. Additions to existing ignore files are shown as a comparison list and applied only after confirmation. Cursor rules and template skills are no longer added automatically; the user is offered a comparison and decides. Declined decisions are stored in the stamp. Reason: template rules can conflict with a project's own conventions, and customer repositories must not change without consent.
- **v0**. Initial version. Created missing ignore files and appended missing patterns without asking. Added template Cursor rules and skills automatically when the filename was absent and the stack path existed.
