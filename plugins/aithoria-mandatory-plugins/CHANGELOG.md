# Changelog

Index of how rules, skills, and agents in this plugin changed. Newest first.

## 2026-10-01

Plugin added, first as `aithoria-internal-plugins`, then renamed to `aithoria-mandatory-plugins`.

### rules/rule-template-baseline.mdc

Always-apply pointer. It tells the agent to read `enforce-templaterepo-baseline-security`. It does not itself create files, append ignore patterns, or copy rules.

### skills/enforce-templaterepo-baseline-security

- **v3** (this PR). Ignore files are read live from the template, because they are the secret boundary. Template rules are a local copy from 2026-10-01 (commit `d112334622fee45344857f3f7e328b9a167454f7`) in `references/template-rules.md` (target and behavior per rule) and `references/rules/*.mdc` (full texts, read only after a yes). A customer project is recognized when the template is not readable or the `origin` owner is foreign. It gets an opening note and an offered read-only comparison, and nothing is changed without agreement. Missing ignore files in own projects are created from the current template; existing ones always get a comparison. A template rule is offered only when it fits the stack and nothing in the project already covers its behavior and target, whatever the name, and is written only after an explicit yes. The always-apply rule only points at this skill.
- **v2** (PR #2). Plugin renamed from `aithoria-internal-plugins` to `aithoria-mandatory-plugins`. Skill content unchanged.
- **v1** (PR #1). First version as `template-baseline`. Customer projects are not changed. Missing ignore files are created. Additions to existing ignore files wait for confirmation. Optional comparison of Cursor rules.

### agents

None.
