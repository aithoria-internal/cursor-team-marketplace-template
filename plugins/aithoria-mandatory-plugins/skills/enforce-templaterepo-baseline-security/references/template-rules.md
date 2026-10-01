# Template rules

Copied from `aithoria-internal/templaterepo` `.cursor/rules` on 2026-10-01, commit `d112334622fee45344857f3f7e328b9a167454f7`. Full texts are in `rules/`. Read a full text only after the user said yes to that rule.

The template has no skills.

| Rule | Target | Behavior | Fits when |
| --- | --- | --- | --- |
| `behavior.mdc` | always | Caution over speed: state assumptions and ask, minimal code, surgical changes only, define a verifiable success check before implementing. | always |
| `project.mdc` | always | Template identity and stack: `project.json` holds org values, generated files, Next.js / Prisma / NextAuth Entra / Azure, named exports, Zod, logger, no `console.log`. | `project.json` contains `templateRepo` |
| `commands.mdc` | always | Use the template's npm scripts (dev, lint, test, db, bootstrap, DNS PRs), never raw shell. | `project.json` contains `templateRepo` |
| `api-routes.mdc` | `src/app/api/**` | Zod validation, `auth()` check, `{ data } \| { error }` shape, correct status codes, no stack traces in responses. | `src/app/api/` exists |
| `components.mdc` | `src/components/**` | Design tokens + Tailwind, server components by default, `{Name}Props`, `cn()`, accessibility, named exports. | `src/components/` exists |
| `prisma.mdc` | `prisma/**` | PascalCase models, cuid `id` + `createdAt` + `updatedAt`, singleton client, migrations with snake_case names, SSL, `$transaction`. | `prisma/` exists |
| `terraform.mdc` | `infra/**`, `bootstrap/**` | `bootstrap/` = state + deploy identity, `infra/` = runtime; naming, tags, unique-name suffixes, no `-var` in PowerShell, managed identity for Postgres, no DB password. | `infra/` or `bootstrap/` exists |
| `testing.mdc` | `tests/**` | Vitest `*.test.ts`, Playwright `*.spec.ts`, mock DB and auth helpers, test behavior, accessible selectors. | `tests/` exists |
