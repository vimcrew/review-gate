# Ready-to-use review rules

Start with **base + one profile + one stack**. These are review criteria, not
instructions to run tools or change the reviewer's response format. Edit them to
match your supported versions, deployment model and documented contracts.

Project rule loading requires a Review Gate engine with project-rules support;
older engines reject `rules` and `max_rules_bytes`.
Use the [Action workflow](../examples/review-gate.yml) to configure vendor secrets
and base checkout. The YAML files below are gate configs, not Actions workflows.

## Choose a combination

| Layer | Choose | Purpose |
| --- | --- | --- |
| Base | [Review essentials](base/review-essentials.md) | Evidence, severity, correctness, security, tests, operations |
| Profile | [Corporate](profiles/corporate.md) | Shared services, compatibility, audit, governance and SLOs |
| Profile | [Individual](profiles/individual.md) | Small projects: bugs and security with little process |
| Stack | [Kotlin, Spring, PostgreSQL, DDD, hexagonal](stacks/kotlin-spring-postgres-ddd-hexagonal.md) | Domain boundaries, persistence, transactions and deployment |
| Stack | [Java / Spring Boot](stacks/java-spring-boot.md) | JVM services |
| Stack | [TypeScript / React / Next.js](stacks/typescript-react-nextjs.md) | Web applications |
| Stack | [TypeScript / Node / NestJS](stacks/typescript-node-nestjs.md) | Node services |
| Stack | [Python / FastAPI](stacks/python-fastapi.md), [Django](stacks/python-django.md) | Python APIs and applications |
| Stack | [Go](stacks/go-service.md), [.NET / ASP.NET](stacks/dotnet-aspnet.md), [Rust](stacks/rust-service.md) | Backend services |
| Stack | [Android / Kotlin](stacks/android-kotlin.md), [iOS / Swift](stacks/ios-swift.md) | Mobile applications |
| Companion | [Terraform / Kubernetes](stacks/infra-terraform-k8s.md) | Infrastructure changes |
| Companion | [SQL / migrations](stacks/data-sql-migrations.md) | Database and data changes |

Select only applicable stacks; a monorepo can add a companion for changed infra
or SQL. Do not load both profiles or the entire library with a broad glob. The
layers add criteria; later files do not mechanically override earlier ones.
Resolve conflicting rules in the committed project policy before enabling them.
Adopted, explicit project policy can justify `policy`; a preferred pattern alone
is a minor `refactor` finding unless it causes a concrete defect.

## Connect the rules

Download or clone this public repository first. From its root, concatenate your
selection and commit the result in your project's base branch:

```sh
mkdir -p /path/to/project/.vimcrew
cat rules/base/review-essentials.md rules/profiles/individual.md \
  rules/stacks/python-fastapi.md > /path/to/project/.vimcrew/review-rules.md
```

No `rules:` key is needed for this default file. Without an explicit list, the
engine loads existing `AGENTS.md`, `CLAUDE.md`, `.github/copilot-instructions.md`
and `.vimcrew/review-rules.md`, in that order, sharing the same budget.

Alternatively, copy this directory into your project as `.vimcrew/rules/` and
copy one [example config](examples/) to `.vimcrew/review-gate.yml`:

```sh
mkdir -p /path/to/project/.vimcrew/rules
cp -R rules/. /path/to/project/.vimcrew/rules/
cp rules/examples/individual-fastapi.yml /path/to/project/.vimcrew/review-gate.yml
```

`rules: [...]` replaces the default list. Add `AGENTS.md` explicitly if wanted.
Paths and globs are relative to the target repository root, not the YAML file or
the Action installation; they cannot reference this library remotely. Avoid
absolute paths and `..`. Explicit paths preserve list order; matches of each
glob are sorted and duplicates removed. `rules: []` disables project rules.

Rules in PR reviews come from **git objects at the base SHA**, regardless of the
checkout. Merge the rules/config setup first: a PR cannot activate its own new
rules. If base objects are unavailable, rules are omitted with a warning; the
exit code still follows the verdict. Check the summary's file list and any
truncation before relying on a review.

## Budget and finding vocabulary

Each rule file is at most **6000 UTF-8 bytes**. Every base/profile/stack triple
and each supplied example fits **24000 bytes**, including per-file headings.
`max_rules_bytes: 24000` is the default shared budget: contents, path headings
and truncation markers count; the fixed section instruction does not. Overflow
truncates the current file and stops later files. Other default files, extra
stacks and your edits also consume it. Keep headroom and measure bytes with
`wc -c`, not character counts. Rules increase every reviewer's input cost.

The engine's `GateFindingPolicy`, `ReviewGateConfig.KNOWN_CATEGORIES` and
`SeverityClassifier` define the vocabulary used by [the base rules](base/review-essentials.md).
Use explicit `blocking` or `minor` severity and canonical categories. Unknown
categories become minor; minor is never promoted solely by category. The gate
also applies quorum and `block_on` thresholds; prose rules do not change them.

Examples: [corporate Kotlin](examples/corporate-kotlin.yml),
[individual Next.js](examples/individual-nextjs.yml),
[individual FastAPI](examples/individual-fastapi.yml),
[corporate infrastructure + SQL](examples/corporate-platform.yml).
They use two vendors and contain no credentials; pass their API keys through
Action inputs. Start in comment mode when calibrating a new policy.
