# Git Momentum

Review this repository as a Git Momentum repo when `.gitmomentum/policy.md` says `adoption: from-start` or `adoption: since-commit`. If `adoption` is `off`, do not file the findings below.

When `adoption` is `since-commit`, do not ask anyone to rewrite commits that are the cutoff SHA or ancestors of it. Flag only the pull request that would create a new in-scope commit.

The squash commit on `main` is the pull request title. Working-branch commit messages inside the branch are unrestricted. When a rule below matches, add a blocking Bug assigned to the PR author. Do not file findings for style issues these rules do not name.

## PR title

The pull request title must match `type: subject`.

- `type` is one of: feat, fix, refactor, docs, build, ci, chore
- the subject is imperative
- the first letter after the colon is lowercase
- the subject has no trailing period
- the title is 72 characters or fewer
- the subject describes the change, not a filename

If the title violates this, add a blocking Bug titled "PR title breaks the Git Momentum commit convention". Quote the title and the expected form in the body.

## Policy sentinel

If `.gitmomentum/policy.md` is absent from the repository, add a blocking Bug titled "Git Momentum policy folder missing" only when this Bugbot file is already present. Body: "Add .gitmomentum/policy.md with adoption from-start, since-commit, or off. A missing file is undecided. It is not an opt-out, and it is not permission to rewrite existing history."

## Derived history

If the diff adds a root file whose purpose is to store the product version or a hand-written changelog (`VERSION`, `CHANGELOG`, `CHANGELOG.md`), add a blocking Bug titled "Version or changelog must stay derived from Git". Body: "Versions are annotated tags. The changelog is generated from main. Do not commit either as a source file."

If the diff adds or changes a feature flag so that it is enabled for prod, add a blocking Bug titled "Feature flag must not target prod". Body: "prod is not a flag stage. Remove the flag and ship the behavior unconditionally."
