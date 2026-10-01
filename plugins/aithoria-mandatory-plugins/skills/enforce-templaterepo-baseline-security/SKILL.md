---
name: enforce-templaterepo-baseline-security
description: >-
  Keeps the governance baseline of aithoria-internal/templaterepo in the
  current git repository: the ignore files .gitignore, .cursorignore and
  .dockerignore (read live from GitHub), and template Cursor rules when they
  add value. Creates missing ignore files in own projects, proposes additions
  to existing ones, and writes a rule only after an explicit yes. Never changes
  a customer project on its own. Use once per day per repository, and again
  when a stack path such as infra/, bootstrap/, prisma/ or tests/ appears.
metadata:
  version: "3"
---

# Template baseline

The template repository `aithoria-internal/templaterepo` defines the minimum governance baseline: which files stay out of git, out of the AI context, and out of Docker images. This skill brings that baseline into a repository. It does not adopt the template. Code, workflows, Terraform, Docker, and package files stay as they are.

## When

- Once per calendar day per repository.
- Again the same day when a stack path appears that was absent at the last check: `infra/`, `bootstrap/`, `prisma/`, `tests/`, `src/app/api/`, `src/components/`, or a `project.json` that contains `templateRepo`. Also run before you create one of these paths yourself.
- Skip the template repository itself and any directory that is not a git repository.

Keep a note outside the repository, in the user agent store, as `template-baseline-stamps/<owner>--<repo>.txt`. If there is no store, keep it for the conversation. The note holds the date, the stack paths that exist, the project level, and the user's decisions. Never write it into the project.

Something the user declined stays declined. Do not ask again, do not ask why, and do not bring it up on a later day.

## 1. Project level

Decide this first. It controls everything below.

Read the template with `gh`:

```
gh api repos/aithoria-internal/templaterepo/contents/<file> -H "Accept: application/vnd.github.raw"
```

**Customer project** applies if either is true:

- The template cannot be read: no access, `gh` is not logged in, or it returns 404.
- The owner of the `origin` remote is not `aithoria`, `aithoria-internal`, or the user's own account (`gh api user --jq .login`). The same applies if the user says it is a customer or third-party project.

This is not an error. Be strictly defensive here: create nothing and change nothing on your own, not even a missing ignore file. Open with one short note: this is not an aithoria-internal or private project, so you will not change anything without explicit agreement. If the template is readable, offer the comparison (see 2) in the same note. Show it only as a read-only report. Write only the lines the user then names explicitly.

**Own project** means the template is readable and the owner is `aithoria`, `aithoria-internal`, or the user's own account. Decide each ignore file separately:

- **File missing:** create it without asking, with the current content from the template. Then name the created files in one sentence.
- **File present:** always offer the comparison with the template (see 2). Write only after the user confirms.

If there is no remote, ask once whether this is a customer project. Treat it as one until the user answers.

## 2. Ignore files

The files are `.gitignore`, `.cursorignore`, and `.dockerignore`. Always read them live from the template's default branch, because they are the secret boundary and must be current. Do not use a copy, and do not create a file the template does not have.

To compare an existing file, match trimmed pattern lines and ignore blank lines and comments. Show a short list per file:

- **Present:** template patterns the file already has.
- **Would be added:** template patterns it lacks.
- **Notes:** added lines starting with `!`, because a negation at the end can re-include paths the project ignores on purpose. Also list any tracked file a new pattern would match. Check that with `git ls-files -ci --exclude-from=<tmp>`.

Then ask: add all, add some, or add none. If everything is already present, say so in one sentence and do not ask.

When the user confirms, append only the confirmed lines to the end of the project's ignore file. Put this comment line directly above them, so it is clear where they came from: `# template baseline (aithoria-internal/templaterepo)`. Never write to the template repository. Never change, reorder, or remove project lines. A tracked file that becomes ignored stays tracked. Report it, especially `.env*`, `*.pem`, and `*.tfstate`, but do not remove it from the index.

## 3. Template rules

Do not include this in the daily pass. Use it only when a stack path is new, or when the user asks about rules.

Source: [references/template-rules.md](references/template-rules.md). This is a local copy of the template rules as of 2026-10-01. Do not read rules from GitHub.

Offer a rule only if it passes both tests:

1. **It helps this repository:** its "Fits when" condition is true here.
2. **The project does not already cover it:** compare the rule's behavior and target with what the project already has, regardless of file names. Look in `.cursor/rules/`, `.cursorrules`, `AGENTS.md`, `CLAUDE.md`, and `.cursor/skills/`, `.claude/skills/`, `.agents/skills/`, `.codex/skills/`. A rule with a different name but the same scope and target counts as coverage. For example, an existing `react-conventions.mdc` on `src/components/**` covers `components.mdc`. A matching file name is only a hint.

If no rule passes, say nothing about rules. If some pass, name each one with one sentence on why it helps. Do not list what is already covered.

Write a rule only after an explicit yes for that specific rule. Agreeing to hear suggestions is not agreement to write a file. Copy the rule from `references/rules/<name>.mdc` to `.cursor/rules/<name>.mdc`. Never overwrite, merge into, or delete an existing rule. In a customer project, bring up rules only if the user asks.

## After

Say in one or two sentences what changed: which files were created, which lines were added, and which rules were adopted. If nothing changed, say so briefly. Then continue with the user's actual task.
