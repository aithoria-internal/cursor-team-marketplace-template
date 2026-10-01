# Git Momentum

adoption: since-commit

# Last legacy commit on main. Descendants of this SHA follow Git Momentum.
# Copy the SHA from the GitHub commit page before the adoption change.
since-commit: 46a614e198d1268d36d13cecdca0990961198848
since-commit-url: https://github.com/aithoria-internal/cursor-team-marketplace-template/commit/46a614e198d1268d36d13cecdca0990961198848

trunk: main
deployment-branches: testing, staging, prod
update-pattern: update/YYYYMMDD-slug
hotfix-pattern: hotfix/YYYYMMDD-slug

direct-push: forbidden

## Commit messages

Applies to the squash commit on main and to hotfix squash commits on prod, and only to descendants of since-commit. Commits on update and hotfix branches stay free. Commits at or before since-commit are not rewritten to match this format.

format: "type: subject"
types: feat, fix, refactor, docs, build, ci, chore
subject: imperative, lowercase first letter, no trailing period, 72 characters or fewer, describes the change rather than the filename
body: optional, blank line above, explains why

## Versions

pattern: v<major>.<minor>.<patch>
annotated: true
when: only when someone asks to release
confirm-before-tagging: true
lookup: git describe --tags --candidates=100 --match=v[0-9]* --abbrev=4
