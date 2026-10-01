---
name: enforce-templaterepo-baseline-security
description: >-
  Checks the current git repository against the secret and ignore baseline from
  aithoria-internal/templaterepo. Creates missing .gitignore, .cursorignore and
  .dockerignore files, and proposes additions to existing ones for confirmation.
  Considers a template rule or skill only when a new stack path appears or the
  user asks, and only when this project does not already cover that behavior
  and target. Never changes a customer project on its own. Use once per day
  for the ignore files, and again the same day when a stack path appears that
  was absent at the last check, such as infra/, bootstrap/, prisma/, or tests/.
metadata:
  version: "3"
---

# Template baseline

The template repository is the source of the minimum secret boundary. Projects that were not created from it, and projects that bypass it, can still receive that boundary. Read the files live from GitHub. Do not rely on a copy stored in this skill.

Source: `aithoria-internal/templaterepo`, default branch, via `gh`.

This does not adopt the template. Application code, workflows, Terraform, Docker, package files, Prisma, `.env.example`, Cursor rules, and skills stay as they are unless the user explicitly chooses otherwise.

## When

Once per calendar day for a repository, check the ignore files. That daily pass does not read or offer template rules or skills. Consider rules and skills only when a stack path appears that was absent at the last check, or when the user asks. Skip the template repository itself. Skip a directory that is not a git repository. If `gh` cannot read the template, stop and say so. Do not invent file contents.

Stack paths that trigger a new check: `infra/`, `bootstrap/`, `prisma/`, `tests/`, `src/app/api/`, `src/components/`, and a `project.json` that contains `templateRepo`.

Record the check outside the repository, in the user agent store, as `template-baseline-stamps/<owner>--<repo>.txt`. Write today's date, which of those paths existed, the project level (see below), and the user's decisions (for example "ignore additions declined", "rule comparison declined"). If no agent store is available, remember the same note for the conversation. Run when there is no note, the date is not today, or a path exists now that the note does not list. After a run, update the note. Do not put the note in the project and do not commit it.

Do not ask again on the same day about ignore-file lines the user already declined, unless a new stack path appeared or the template now proposes different lines. A declined rule or skill stays declined until a new stack path appears or the user asks. The next day's ignore-file pass does not reopen it.

Before creating one of those stack paths, run the check again even if today already ran. Then update the note.

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

Do not ask whether the user wants a comparison of the template rules. On a daily ignore-file pass, do not open this section.

Template rules are only the files in `aithoria-internal/templaterepo` at `.cursor/rules`, on the default branch, read with `gh`:

`gh api repos/aithoria-internal/templaterepo/contents/.cursor/rules`

That tree is the source. The project's `.cursor/rules` is the thing you compare against, not the source.

Read each template rule there, at least its description. Then decide both of the following. Read the rule body only when the description is not enough.

1. **Would it help this repository?** Use the table. A rule the template adds later follows the same test: `alwaysApply` and no stack names can fit any repository; otherwise the paths in its description or `globs` must already exist. Skip Terraform when `infra/` and `bootstrap/` are absent, and skip the other stack rules when their path is absent.
2. **Does this project already cover that behavior for that target?** Compare content, behavior, and target with the project's rules and skills, whatever they are named. A component policy covers `components.mdc` even when it is not named `components.mdc`. A behavioral guideline covers `behavior.mdc` under any name. A Terraform rule or skill covers `terraform.mdc` even when the name does not contain `terraform`. Look at `.cursor/rules`, a legacy `.cursorrules`, and skills under `.agents/skills`, `.claude/skills`, `.codex/skills`, and `.cursor/skills`. The same filename is a hint, not the decision. If the project already manages that aspect, do not offer a second rule beside it.

Offer a comparison only when a template rule would help and nothing already covers that behavior and target. Name each one and why. If none qualify, say nothing about rules and continue the task. Do not inventory what is already covered.

In a customer project, do not offer this unless the user brings rules up. If the user declines, record that and do not ask again until a new stack path appears or the user asks.

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

Template skills, if any, are the skills in `aithoria-internal/templaterepo` under `.agents/skills`, `.claude/skills`, and `.codex/skills`, read with `gh` the same way as the rules. The template currently has none. On a daily ignore-file pass, do not open this section.

If the template gains one, use the same two tests as for rules: it would help this repository, and no project rule or skill already covers that behavior for that target, regardless of name. Copy it only when the user asks. Write it once, into the same directory the template used. Do not overwrite an existing skill.

## After

Say in one or two sentences what changed: created files, confirmed additions, and adopted rules. If nothing changed because the baseline was already present or the user declined, say that briefly and continue the task the user asked for.

## Changelog

History only; the sections above define the behavior.

- **v3** (feedback: Erik). The daily pass checks ignore files only. Template rules are read from `aithoria-internal/templaterepo` `.cursor/rules` with `gh`, not from the project's `.cursor/rules`. A rule or skill is treated as already featured when the project already covers the same behavior for the same target, whatever the file is named.
- **v2** (feedback: Erik). The always-apply rule only points at this skill. It no longer repeats create, append, or copy steps. Template rules are read before any question. A comparison is offered only for missing rules that would help this repository. Rules that do not fit are not mentioned.
- **v1** (feedback: Denis). Added project levels: customer projects are not changed, and a comparison runs only on request. Missing ignore files are created and reported. Additions to existing ignore files are shown as a comparison list and applied only after confirmation. Cursor rules and template skills are no longer added automatically; the user is offered a comparison and decides. Declined decisions are stored in the stamp. Reason: template rules can conflict with a project's own conventions, and customer repositories must not change without consent.
- **v0**. Initial version. Created missing ignore files and appended missing patterns without asking. Added template Cursor rules and skills automatically when the filename was absent and the stack path existed.
